Title: BRHC: Backend-driven Reactive Hypermedia Controls with a Statically Typed Kotlin DSL

# Introduction

The Web is the most widely accessible application platform, serving as the
primary computing environment for billions of users worldwide. Modern digital
services continue to rely heavily on web technologies, with JavaScript playing
a central role across modern websites. At the same time, modern web
applications have become increasingly complex, relying heavily on client-side
execution and large JavaScript codebases, while median mobile page weight has
continued to increase steadily over time, reflecting a substantial growth in
resource requirements over the past decade. The rapid adoption of JavaScript
progressively transformed the Web from a document navigation system into a
programmable application platform. Client-side libraries such as jQuery
simplified DOM manipulation and asynchronous communication with servers,
encouraging richer client-side interaction patterns. Techniques such as
partial page updates and transclusion—where fragments of remote documents are
dynamically inserted into existing pages—enabled more interactive user
interfaces without full page reloads. Over time, the complexity of client-side
logic grew significantly, culminating in the widespread adoption of the
Single-Page Application (SPA) model.

In recent years, there has been a renewed interest in the Hypermedia-Driven
Application (HDA) approach as an architectural alternative to JavaScript-heavy
SPAs. This shift is motivated by concerns related to complexity, steep
learning curves, and long-term maintenance costs. Rather than continuing the
progressive migration of application logic to the client, these approaches re-
emphasize server-centric interaction models and aim to reduce the reliance on
client-side JavaScript where possible.

The hypermedia-driven approach builds upon the original Web architecture
principles of Hypermedia as the Engine of Application State (HATEOAS), a core
constraint of REST. In this context, *hypermedia controls* such as links,
forms, and buttons, act as the primary interaction mechanism between client
and server.

*Hypermedia controls* were originally defined by Fielding as elements based on
affordances that provide users with actionable interaction choices within a
hypermedia system. Building on this foundation, Gross refined the concept into
a more formal model in which a *hypermedia control* is understood as an
element that (1) responds to an event trigger, (2) issues an HTTP request to a
target URL, and (3) integrates the resulting response into the user interface
at a designated location. JavaScript libraries such as htmx, Turbo, and
Datastar leverage the hypermedia control model by extending HTML with custom
attributes such as `hx-`, `turbo-`, respectively, or data attributes (i.e.,
`data-`) for Datastar, which augment standard HTML elements with declarative
behavior for client-server interaction. While this formalization clarifies the
interaction model of *hypermedia controls*, practical implementations are
typically realized through untyped or weakly typed mechanisms embedded in
HTML.

Our work extends the HtmlFlow Kotlin DSL (Domain-Specific Languages) by
leveraging Kotlin as a statically typed host language to suppress the issues
associated with untyped or string-based expressions in hypermedia control
attributes. The proposed model eliminates JavaScript expressions by replacing
them with statically typed expressions defined directly in the backend host
language, namely Kotlin. We demonstrate the applicability of the approach
through a catalog of *Backend-driven Reactive Hypermedia Control* (BRHC)
patterns (as popularized by htmx examples and also used in Datastar) and a
case study based on the well-known Petclinic Spring MVC web application.

# Related Work

Modern web development has increasingly shifted toward client-side
architectures based on Single-Page Applications (SPAs). While these
architectures enable rich interactive experiences, they also introduce
challenges related to frontend complexity, duplicated business logic, and
long-term maintainability, as well as hydration costs inherent to client-side
rendering.

One prominent response to these limitations is the emergence of *disappearing
frameworks*. These frameworks represent a paradigm shift in modern web
development by attempting to minimize their own presence in the final
application. Rather than extending the traditional SPA model, these approaches
focus on reducing the amount of JavaScript shipped to the client through
techniques such as server-side rendering, progressive enhancement, and
selective hydration. By prioritizing minimal client-side execution and
deferring computation whenever possible, they aim to improve performance,
simplify application delivery, and reduce frontend complexity.

Related approaches have explored alternative rendering strategies to address
the costs of client-side hydration. Islands Architecture restricts hydration
to isolated interactive regions while preserving server-rendered content,
thereby reducing JavaScript execution and improving page performance.
Similarly, Incremental Static Regeneration (ISR), popularized by Next.js,
combines static site generation with selective server-side revalidation,
enabling applications to retain many of the performance benefits of static
content while supporting dynamic updates.

Another architectural response to SPA complexity is the adoption of micro-
frontends, which decompose large frontend applications into independently
developed and deployed units. Unlike disappearing frameworks and islands-based
architectures, micro-frontends primarily address organizational scalability
and team autonomy rather than reducing JavaScript execution in the browser.
Nevertheless, they illustrate the broader effort to manage the growing
complexity of modern web applications.

While disappearing frameworks, Islands Architecture, ISR, and related
approaches differ in their implementation strategies, they share a common
objective: reducing the complexity, performance costs, and maintenance burden
associated with modern web applications. These approaches primarily focus on
improving how applications are rendered, delivered, and executed, while
preserving the rich interactive experiences expected by users.

A different line of work approaches the problem from an architectural
perspective by revisiting the principles of the original Web. Rather than
focusing on rendering and hydration strategies, these approaches return to the
REST architectural style proposed by Fielding, particularly the *Hypermedia
Constraint* (HATEOAS). Within this context, *Hypermedia Systems* rely on
*hypermedia controls*, which define how clients transition between application
states through server-provided representations.

Recent developments in this area build directly on the formalization of
hypermedia controls proposed by Gross, in which controls are defined as
elements that (1) respond to events, (2) issue HTTP requests, and (3)
integrate server responses into designated locations within the user
interface. In REST terms, these controls act as *affordances*: they tell the
client which application transitions are currently available and how they can
be activated next. This is central to HATEOAS, where the server does not only
return data, but also represents the valid next actions through links, forms,
buttons, and related controls embedded in the response. Frameworks and
libraries such as htmx, Turbo, and Datastar extend this model by enriching
*hypermedia controls* with additional client-side capabilities, including
fine-grained partial page updates, declarative DOM placement strategies, and
reactive behavior coordination. These systems maintain the server as the
primary source of application state and interaction logic while delegating
only minimal orchestration responsibilities to the client, thereby reducing
reliance on full client-side application frameworks.

The htmx example below illustrates how the hypermedia control model can be
expressed directly through HTML attributes. The `hx-trigger="click"` attribute
specifies the event that activates the control, while `hx-post="/clicked"`
defines the HTTP `POST` request issued to the `/clicked` endpoint. The `hx-
target="#parent-div"` attribute identifies the location where the server
response is integrated. The `hx-swap` attribute provides several options for
controlling how htmx integrates the server response into the DOM. In this
example, `outerHTML` replaces the entire target element with the returned HTML
fragment.

```html
<button hx-post="/clicked"
    hx-trigger="click"
    hx-target="#parent-div"
    hx-swap="outerHTML">
    Click Me!
</button>
```

The listing below shows the corresponding HTTP response containing the HTML
fragment inserted into the target element. Thus, the button encapsulates event
handling, request dispatch, and response integration without requiring
explicit client-side scripting.

```http
HTTP/1.1 200 OK
Content-Type: text/html

<div id="parent-div">
    <p>Hello world!</p>
</div>
```

Datastar adopts a different API style based on the concept of *actions*, which
are embedded within JavaScript-like expressions assigned to Datastar
attributes. Actions represent triggered behaviors, typically initiated by user
interactions or lifecycle events, and are denoted by the `@` prefix. These
expressions declaratively specify the operations to be performed, such as
issuing HTTP requests or updating reactive state, while remaining embedded
within HTML attributes. The listing below presents the equivalent Datastar
implementation of the htmx example.

```html
<button data-on:click="@post('/clicked')">
    Click Me!
</button>
```

Instead of specifying the target element directly in the button attributes,
Datastar follows a different approach in which the backend determines both the
resulting HTML fragment and the location where that fragment should be
integrated into the document. To support this model, Datastar defines a
patching protocol on top of Server-Sent Events (SSE), using the `text/event-
stream` MIME type. SSE represents each message as a sequence of key–value
fields in plain text. In this format, each message begins with an `event:`
field that specifies the type of operation being sent by the server, followed
by one or more `data:` fields that carry the associated payload. Each `data:`
line represents a fragment of the message, and multiple lines are concatenated
in order to reconstruct the full structured payload on the client side.

The listing below illustrates an example of an HTTP response that patches the
HTML element identified by `parent-div`. The event type is `datastar-patch-
elements`, indicating that the server is instructing the client to update a
portion of the DOM. The subsequent `data: elements` entries define the HTML
fragment to be applied, split across multiple lines for streaming and
formatting purposes. When combined, these lines reconstruct the complete HTML
fragment: `<div id="parent-div"><p>Hello world!<p></div>`, which represents
the content to be inserted or used as a replacement for the selected element.

```http
HTTP/1.1 200 OK
Content-Type: text/event-stream

event: datastar-patch-elements
data: elements <div id="parent-div">
data: elements     <p>Hello world!<p>
data: elements </div>
```

Datastar provides several backend SDKs for commonly used technologies such as
Java, Python, Go, and Kotlin, which are designed to simplify the generation of
Datastar-specific Server-Sent Events (SSE).

# HTML Templating and Safe Reactive Signal Management

This section establishes the architectural foundation for modern *web
templates*, drawing a distinction between *external* and *internal* templating
idioms. A *web template* serves as a reusable layout for an HTML document that
an engine combines with *input data models* to produce standard HTML as
output.

In a typical web templating workflow using Thymeleaf, the template contains
placeholders and expressions, such as `th:text="${artistName}"` and
`th:text="${'Founded: ' + musicBrainz.year}"`, which define how dynamic
content should be inserted into the HTML document. At runtime, the template is
combined with an input data model containing information about an artist,
e.g., David Bowie and associated MusicBrainz data. During the rendering
process, Thymeleaf evaluates the template expressions against the provided
data model and replaces them with the corresponding values. The resulting
output is a complete HTML document in which all template directives have been
resolved.

Traditional templating approaches such as Thymeleaf, JSP, Handlebars, Pebble,
or FreeMarker embed dynamic expressions into HTML using string-based markup
extensions like `<%`, `{{}}`, `${}`, which lack native type safety and can
introduce runtime vulnerabilities. In contrast, an *internal* Domain Specific
Language (DSL) approach embeds HTML templating directly into a statically
compiled programming language, providing structural verification at compile
time. Building upon this paradigm, an approach that pairs statically typed
templates with backend-driven state updates provides robust guarantees.

## Internal Domain Specific Language (DSL) for HTML

An *internal* Domain Specific Language (DSL) like HtmlFlow, Hiccup, ScalaTags,
KotlinX.Html, or others, leverages its host programming language (e.g., Java,
Clojure, Scala, Kotlin) as the core dialect to define web templates. Although
HtmlFlow was originally designed for Java, it also provides an idiomatic
Kotlin API. Specifically, HtmlFlow supports HTML builders using *function
literals with receiver*.

In Kotlin, a block of code enclosed in curly braces `{...}` is known as a
*lambda*, and can be used as an argument to a function that expects a
*function literal*. When we write, for example, `body { div { hr() } }`, we
are invoking the `body` function with a lambda as its argument. This lambda,
in turn, calls the `div` function with another lambda as an argument that
creates a horizontal row (i.e., `hr`). Each call to an HTML builder (e.g.,
`body`, `div`, `hr`) creates the child element within the element generated by
the outer function call.

The listing below illustrates the HtmlFlow Kotlin idiom for constructing an
HTML template equivalent to the Thymeleaf template described previously. The
provided `HtmlView` highlights a statically typed binding with the `Artist`
domain model. Within this boundary, properties like `m.name` and the
aggregated `m.musicBrainz` fields (`year`, `from` and `genres`) are resolved
dynamically yet checked at compile time, guaranteeing that data bindings
conform directly to the domain model. Rather than parsing unverified text
attributes, this view exclusively leverages standard Kotlin programming
constructs.

```kotlin
val artistView: HtmlView<Artist> = view<Artist> {
    html {
        body {
            h1 { dyn { m: Artist ->  text(m.name) } }
            h2 { text("MusicBrainz info:") }
            ul { dyn { m: Artist -> with(m.musicBrainz) {
                li { text("Founded: " + year) }
                li { text("From: " + from) }
                li { text("Genre: " + genres) }
            }}}
        }
    }
}
```

The `dyn` builder is employed to seamlessly integrate Kotlin code into the
definition of web templates and has the following signature:

```kotlin
Element<T>.dyn(cons: T.(M) -> Unit): T
```

It takes a function literal (i.e., `cons`) with a receiver corresponding to
the parent element (i.e., `T`) and an argument representing the model (i.e.,
`M`). The `dyn` builder returns the same parent element (i.e., `T`).

Given an HtmlFlow view, e.g., `artistView`, the model instance (e.g., `bowie`)
is later passed to its `write(model: T)` method to produce the resulting HTML.
In the following example, the view `artistView` is first bound to an output
stream `outStream`, and then resolved with the `bowie` instance so that the
rendered HTML is written directly to it:
`artistView.setOut(outStream).write(bowie)`.

## Foundations of Reactive Signals

Reactive signals originate from the broader class of *reactive programming*
models. Signals are a programming abstraction used to model state that can be
observed and propagated through a system in a declarative manner. Early work
on functional reactive programming (FRP) introduced the notion of time-varying
values and event-driven propagation as first-class abstractions for
structuring interactive systems. In these models, a signal is typically
understood as a continuous or discrete value that maintains dependencies on
other signals, enabling automatic re-evaluation when upstream data changes.

Formally, a signal can be seen as a function of time $S : t \rightarrow v$,
where changes in the underlying value $v$ trigger the re-evaluation of any
computation that depends on $S$. In contrast to traditional mutable variables,
signals introduce a dependency-tracking mechanism that builds a directed graph
of relationships between producers (state sources) and consumers (derived
computations or views). When a signal is updated, only the affected downstream
nodes in this dependency graph are recomputed, enabling efficient incremental
updates.

Computed signals define derived values as pure functions over one or more
input signals. Formally, a computed signal $C$ can be defined as $C(t) =
f(S_1(t), S_2(t), \dots, S_n(t))$, where $f$ is a pure function and $S_i$ are
the input signals on which $C$ depends. Any change in one of the input signals
$S_i$ automatically triggers a recomputation of $C$, preserving consistency
across the dependency graph while maintaining referential transparency of
derived state.

Modern reactive systems refine this idea by introducing *fine-grained
reactivity*, where dependencies are tracked at the level of individual
variables or expressions rather than coarse execution units. Frameworks such
as SolidJS and similar signal-based architectures formalize signals as mutable
cells with explicit dependency tracking and minimal recomputation overhead.
This shift improves performance by ensuring that only the affected portions of
the computation graph are updated when a signal changes. This design direction
is aligned with the emerging TC39 Signals proposal, which aims to standardize
a minimal reactive primitive for JavaScript. Although still under active
specification, this proposal reflects a convergence toward a unified model of
fine-grained reactivity across modern JavaScript runtimes.

## Signal-based Statically Typed DSL for HTML

Datastar is a JavaScript library that implements a backend-driven hypermedia
approach. Complementing Datastar attributes are *actions* and *signals*, which
together define the dynamic aspects of the system. To propagate state changes
efficiently, Datastar relies on a push-based notification protocol provided
through a modular plugin architecture. By default, Datastar uses a Server-Sent
Events (SSE)-based communication protocol that streams incremental updates
from the server to the client. Through this *unidirectional* channel, the
server can issue targeted patch operations that update both signal values and
HTML elements without requiring a full page reload.

The Datastar example below illustrates the combination of *signals* and
*actions* to implement a simple reactive counter. Note that other optimized
and recommended implementations of this example are possible; however, this
approach is intentionally used to illustrate the complete end-to-end flow of
signal propagation. The root element declares a signal state using `data-
signals="{count: 0}"`, which initializes a reactive variable `count` with the
value 0. This signal is bound to the user interface through the expression
`data-text="$count"`, ensuring that the content of the `span` element is
automatically updated whenever the signal changes. User interaction is handled
through action-based event bindings on the buttons. The attributes `data-
on:click="@post('/decrement')"` and `data-on:click="@post('/increment')"`
define actions that are executed when the user clicks the corresponding
buttons. These actions dispatch a `POST` request to the specified path
(`/decrement` and `/increment`), triggering the corresponding server-side
handler responsible for updating the shared state.

```html
<div data-signals="{count: 0}">
    <h1>Counting Stars - via Signals</h1>
    <div data-init="@get('/counter-signals/events')">
        <span id="counter" data-text="$count"></span>
    </div>
    <div>
        <button id="decrement" data-on:click="@post('/decrement')">-</button>
        <button id="increment" data-on:click="@post('/increment')">+</button>
    </div>
</div>
```

The reactive lifecycle is initialized using the `data-init` attribute, which
triggers a server request via the action `@get('/counter-signals/events')`.
This establishes an SSE connection between client and server, enabling
continuous backend-driven updates to be pushed to the UI. Importantly, the
counter state is shared across all connected frontend clients, meaning that
any update performed by one client is immediately reflected in every other
subscribed client through server-side propagation. When a button is activated,
the bound Hypermedia Control dispatches a `POST` request to the corresponding
endpoint. The server mutates its authoritative shared state and immediately
pushes the updated value to all connected clients using the `datastar-patch-
signals` event, as shown in the listing below. This SSE payload represents a
minimal patch message where the event type `datastar-patch-signals` indicates
a signal update operation. The `data` field contains a partial state update,
specifying that the `count` signal should be updated to the value 1. Upon
receiving this event, each client reconciles the patch with its local signal
graph, resulting in an automatic update of all bound UI elements, such as the
`span` displaying `$count`.

```http
event: datastar-patch-signals
data: signals { count: 1 }
```

To support reactive *hypermedia controls*, we extend the HtmlFlow DSL with a
signal-based API inspired by Reactive Signal models. In this approach, signals
are represented as statically typed symbolic references embedded directly into
the HTML DSL. Rather than behaving as locally reactive variables, signals act
as typed identifiers used to establish bindings between *hypermedia controls*,
request parameters, and backend-driven UI updates.

The listing below presents the definition of the `Signal<T>` abstraction. A
signal encapsulates a typed symbolic reference associated with a name, where
the signal name must be unique within the template view definition to avoid
ambiguity in bindings and control resolution. The signal itself does not
maintain reactive state on the server runtime; instead, it provides a
statically typed representation that can be composed within Hypermedia Control
expressions and translated into the underlying Datastar-compatible syntax.

```kotlin
class Signal<T>(val name: String)  {...}
```

Signals are introduced into HtmlFlow templates through the `dataSignal(...)`
function, which creates typed signal bindings that can be referenced
throughout the HTML DSL. The listing below illustrates the equivalent HtmlFlow
definition of the counter example. Unlike the raw Datastar sample, which
relies on string interpolation or inline JavaScript expressions, the proposed
API enables signal composition through native Kotlin expressions and typed
builders that are verified at compile time. No string literals are used for
signal binding or endpoint routing. Instead, signal binding is performed
through direct variable references (e.g., `count`), ensuring that renaming the
signal is consistently reflected across the entire template at compile time.
Similarly, server interactions are not defined using string-based URL paths,
but through function references using Kotlin's `::` operator, which binds
directly to backend route handlers within the project scope. In this model,
`get` and `post` are first-class functions rather than string-encoded HTTP
verbs, further reinforcing compile-time safety and structural consistency
between template and server logic.

```kotlin
div {
    val count: Signal = dataSignal("count", 0)
    div {
      dataInit { get(::getCounterEventsSignals) }
      span {
        attrId("counter")
        dataText{ +count }
      }
    }
    div {
      button { dataOn(Click) { post(::decrementCounterViaSignals) }}
      button { dataOn(Click) { post(::incrementCounterViaSignals) }}
    }
}
```

This approach allows Hypermedia Control definitions, signal bindings, and HTML
structure to coexist within a single homogeneous programming model. By
embedding signals directly into the HtmlFlow DSL, the proposed model improves
type safety, reduces reliance on string-based expressions, and enables
declarative backend-driven reactivity without requiring application-specific
client-side JavaScript.

# Backend-driven Reactive Hypermedia Control Patterns

This section presents a set of Hypermedia Control patterns based on a backend-
driven reactive model. In this approach, some controls are bound to signals
that represent application state, and changes in these signals may trigger
control reactions. A control reacts to a signal change by dispatching an HTTP
request, whose response may update the same signal and/or partially patch the
corresponding HTML region. This establishes a bidirectional loop in which
signals drive UI behavior, while the server responses continuously refine both
state and presentation. The resulting model unifies state management and
interaction through hypermedia, reducing the need for client-side imperative
logic and enabling declarative, backend-driven reactivity at the level of
individual controls.

The following patterns illustrate how this reactive model can be applied to
different interaction scenarios:


1. Backend-Driven Signal Update
2. Action-Triggered HTML Patch
3. Signal-Based HTML Patch
4. Signal-Based Incremental HTML Patch
5. Signal-based Formless Submission

Together, the Backend-driven Reactive Hypermedia Control (BRHC) patterns cover
a range of backend-driven behaviors, from simple state synchronization
mechanisms to more advanced interactions that combine signal propagation,
partial HTML updates, and Hypermedia Control-driven requests.

## Backend-Driven Signal Update

The first pattern illustrates how reactive UI behavior can be coordinated
through backend-driven signal updates. Instead of polling or issuing explicit
refresh requests, the client establishes a persistent SSE connection through
which the server pushes signal updates whenever application state changes. In
this pattern, the signal named `count` is initialized with the value 0. The
`dataInit` attribute triggers the browser to open a persistent SSE connection
to `getCounterEventsSignals` as soon as the element is mounted. The `dataText`
binding ensures that the `span` element is automatically re-rendered whenever
`count` changes. The bound UI fragment is updated transparently without
requiring explicit client-side DOM manipulation or imperative rendering logic.
The signal therefore acts as a reactive synchronization mechanism between the
authoritative server state and the rendered UI.

The listing below presents the backend route responsible for maintaining a
persistent connection and emitting signal updates. The `MutableStateFlow` is
initialized with an initial value of 0 and acts as a hot, observable stream
that holds the current authoritative value of the counter while immediately
propagating any state updates to all active collectors. The route defined by
`getCounterEvents` responds with a `text/event-stream` content type and
suspends execution while collecting values from a shared `MutableStateFlow`
representing the counter state. Each time the `counter` flow is updated, the
backend resumes execution of `getCounterEvents` and invokes `patchSignals`,
which serializes the updated state and writes it to the response channel as a
Server-Sent Event. This ensures that all connected clients remain synchronized
with the latest counter value without requiring polling, explicit refresh
requests, or client-side state reconciliation logic.

```kotlin
private val counter: MutableStateFlow<Int> = MutableStateFlow(0)

private suspend fun RoutingContext.getCounterEvents() {
    call.respondBytesWriter(ContentType.Text.EventStream, OK) {
        with(ServerSentEventGenerator(response(this))) {
            counter.collect {
                patchSignals("{count: ${counter.value}}")
            }
        }
    }
}
```

In this interaction flow, the main participants involved are the user
interface, the Hypermedia Control, and the server. In this context, a
Hypermedia Control corresponds to an HTML element enhanced with a Datastar
attribute capable of triggering HTTP interactions.

## Action-Triggered HTML Patch

This pattern illustrates a Hypermedia Control that triggers a backend-driven
UI update in response to an action. When the Hypermedia Control is activated,
the client does not determine how the interface should change. Instead, it
dispatches the HTTP request defined by the action, while the backend processes
the request and determines the corresponding UI update.

To illustrate a use case of this pattern, consider a table that includes a
delete button on each row, allowing the corresponding table row to be removed.
The function `hfTableRow` presented in the listing below is an extension over
a table body element and generates a `tr` element whose identifier is
dynamically assigned using the index parameter (i.e., `row-$index`), enabling
precise targeting of individual rows. Each row contains two data cells that
display the user's name and email address, respectively, followed by a third
cell that defines an interactive control. The `button` element is bound to a
backend-driven hypermedia action through `dataOn(Click)`, which issues an HTTP
`DELETE` request to the path `/delete-row/$index` when activated, mutating the
authoritative state, and determining how the UI should be updated afterward.

```kotlin
fun Tbody<*>.hfTableRow(index: Int, user: TableUser) = tr {
    attrId("row-$index")
    td { text(user.name) }
    td { text(user.email) }
    td {
        button {
            dataOn(Click) { delete("/delete-row/$index") }
            text("Delete")
        }
    }
}
```

The listing below presents the backend route responsible for handling the
deletion request. When the client activates the delete control, the endpoint
responds with a `text/event-stream` payload containing a Datastar element
patch operation. The `PatchElementsOptions` object specifies both the target
element and the patching strategy. In this example, the CSS selector
`#row-$rowIndex` identifies the table row to be modified, while the
`ElementPatchMode.Remove` mode instructs the Datastar client to remove the
matching element from the DOM. Upon receiving the corresponding SSE event,
Datastar locates the element with the identifier given by `row-$rowIndex` and
removes it from the rendered page.

```kotlin
suspend fun RoutingContext.deleteRow() {
    call.respondBytesWriter(ContentType.Text.EventStream, OK) {
        with(ServerSentEventGenerator(response(this))) {
            val rowIndex = call.pathParameters["index"]
            removeUser(rowIndex)
            patchElements(
                PatchElementsOptions(
                    "#row-$rowIndex",
                    ElementPatchMode.Remove
                )
            )
        }
    }
}
```

The listing below shows a server response containing an element patch
operation that instructs the client to remove the element identified by
`row-2`.

```http
event: datastar-patch-elements
data: selector #row-2
data: mode remove
```

Compared with the previous patterns, this one does not rely on signals as
state carriers. It establishes a baseline in which a Hypermedia Control
supports interactive behavior through a direct action-response cycle alone.
The control dispatches a request, and the server response patches the affected
HTML region.

## Signal-Based HTML Patch

This pattern extends *hypermedia controls* with reactive user input. Rather
than relying solely on discrete user actions such as button clicks, the
control reacts to continuous input events and uses the resulting client-side
state to parameterize server requests. In this case, the state is represented
by a search term stored in a reactive signal, which is automatically included
in outgoing requests whenever the control is triggered. The responsibility for
evaluating the search criteria and generating the updated user interface
remains entirely on the server. The client only captures user input and
declares the reactive behavior, while the server performs the filtering
operation and produces the corresponding HTML representation.

To illustrate this pattern, consider an active-search interface composed of a
contact table and a search input field. As the user types a search term, the
current value is synchronized through a signal and used to retrieve the
matching contacts from the server.

The listing below presents the HtmlFlow code for the reactive contact
filtering interface, consisting of a search input and a results table. The
`input` element defines an event-driven Hypermedia Control that dispatches an
HTTP request whenever the `Input` event is triggered and no subsequent input
occurs within a debounce interval of 200 milliseconds. This mechanism ensures
that requests are issued only after the user pauses typing, reducing
unnecessary server interactions.

The `dataBind` attribute binds the current value of the input field to the
`search` signal. This signal is automatically included in outgoing requests,
such as the HTTP `GET` request issued to the backend route handler
`getSearchContacts`, and represents the current query state of the user
interface. Rather than directly initiating requests, the signal functions as a
shared state carrier between client and server, allowing the server to process
the most recent user input when the control is activated.

```kotlin
div {
    input {
        val search: Signal  = dataBind("search")
        dataOn(Input) {
            get(::getSearchContacts)
            modifiers { debounce(200.milliseconds) }
        }
    }
    table {
        thead { ... }
        tbody {
            attrId("contacts")
            ...
        }
    }
}
```

The listing below presents the backend route responsible for handling search
requests. When a Hypermedia Control triggers an HTTP request, it automatically
includes a query parameter named `datastar`, whose value is a JSON
representation of all currently defined signals, excluding local signals whose
names begin with an underscore. In turn, the endpoint responds with a
`text/event-stream` payload containing an HTML patch operation. It first
retrieves the `datastar` query parameter and decodes the `search` signal using
the `ActSearchSignals` data class. After that, it processes the search query
by invoking `getContacts(search)` to retrieve the matching contacts and then
renders the resulting table body using the reusable HtmlFlow fragment
`activeSearchTbody`. The generated HTML is stored in `tableBodyFragment`,
which is subsequently passed to `patchElements`. This operation serializes the
rendered fragment into a Datastar element patch and streams it to the client
through the SSE connection, causing the corresponding portion of the page to
be updated with the filtered search results.

```kotlin
data class ActSearchSignals(val search: String)

private suspend fun RoutingContext.searchContacts() {
    call.respondBytesWriter(ContentType.Text.EventStream, OK) {
        with(ServerSentEventGenerator(response(this))){
            val dsQueryArg = call.request.queryParameters["datastar"]
            val (search) =
                Json.decodeFromString<ActSearchSignals>(dsQueryArg)
            val tableBodyFragment =
                activeSearchTbody.render(getContacts(search))
            patchElements(tableBodyFragment)
        }
    }
}
```

An example of the emitted patch event is shown in the listing below.

```http
event: datastar-patch-elements
data: elements <tbody id="contacts">
data: elements <tr><td> ... </td></tr>
```

The generated HTML fragment contains a `tbody` element with the identifier
`contacts`, allowing Datastar's morphing mechanism to match it against the
corresponding element already present in the DOM. This identifier is therefore
required for the patch operation to be applied correctly. Upon receiving the
event, the client replaces the contents of the existing `#contacts` table body
with the newly generated rows.

Compared with the first pattern, where the server is responsible solely for
emitting signal updates while the client handles the corresponding UI updates,
this pattern shifts the responsibility of rendering back to the server.
Instead of relying on client-side signal reconciliation, the server directly
produces the HTML updates required to reflect the new state.

## Signal-Based Incremental HTML Patch

This pattern combines two complementary mechanisms: backend-driven signal
updates and partial HTML patching. In this approach, the client maintains a
small reactive state through signals, which are continuously updated by the
server and later reused by *hypermedia controls* to drive subsequent
interactions. Although signals are stored on the client, the overall model
remains backend-driven because both the signal values and the HTML patches are
generated and controlled by the server. The client only acts as a temporary
holder of interaction state required to parameterize future requests.

To illustrate this pattern, consider an infinite-scroll interaction model in
which a table is progressively populated as the user scrolls toward the bottom
of the page. The listing below presents the code for this model. Two signals,
`offset` and `limit`, are initialized through `dataSignals` and represent the
current pagination state. The initial table body is populated with the first
ten rows, and a sentinel div element at the bottom of the page carries the
Hypermedia Control that drives subsequent data loading. The Hypermedia Control
is defined through the `dataOnIntersect` attribute. When the associated
element enters the viewport, the control automatically dispatches an HTTP
request to the server. The request implicitly includes the current signal
values, allowing the server to determine which additional rows must be
retrieved and appended to the table.

```kotlin
div {
    dataSignals("offset" to 10, "limit" to 5)
    table {
        thead {
            tr {
                th { text("Name") }
                th { text("Email") }
                th { text("ID") }
            }
        }
        tbody {
            attrId("agents")
            // First 10 rows (Agent Smith 0-9)
            for (i in 0..9) {
                tr { /* Agents Content */ }
            }
        }
    }
}
div { dataOnIntersect { get(::getMoreAgents) } }
```

The listing below presents the backend route responsible for implementing the
infinite-scroll interaction.

```kotlin
data class Signals(val offset: Int, val limit: Int)

suspend fun RoutingContext.getMoreAgents() {
    call.respondBytesWriter(ContentType.Text.EventStream, OK) {
        with(ServerSentEventGenerator(response(this))) {
            val datastarArg = call.request.queryParameters["datastar"]

            // Decode the signals from the datastar query argument
            // and update the signals for the next request
            val (offset, limit) =
                Json.decodeFromString<Signals>(datastarArg)
            patchSignals("{offset: ${offset + limit}}")

            // Generate the new rows to be added to the table
            // and send the patch to the client
            val agents = newAgents(offset, offset + limit)
            val htmlRows = generateRows(agents)
            patchElements(
                htmlRows,
                PatchElementsOptions("#agents", Append)
            )
        }
    }
}
```

The route receives the current client signals through the `datastar` query
parameter. The parameter contains a JSON representation of the `offset` and
`limit` signals, which are deserialized into a `Signals` instance. After
obtaining the current pagination state, it updates the `offset` signal by
adding the current `limit` value and emits the new signal value through
`patchSignals`, which emits a `datastar-patch-signals` SSE event, as
illustrated in the listing below. This ensures that subsequent requests will
continue from the correct position in the dataset. After that, it retrieves
the next batch of agents by invoking `newAgents(offset, offset + limit)` and
generates the corresponding HTML table rows through `generateRows`. Finally,
the generated rows are streamed back to the client using `patchElements`,
which emits a `datastar-patch-elements` event, as shown in the subsequent
listing. The patch operation targets the element identified by the CSS
selector `#agents` and uses the `Append` mode, causing the newly generated
rows to be appended to the existing table contents rather than replacing them.

```http
event: datastar-patch-signals
data: signals { offset: 15 }
```

```http
event: datastar-patch-elements
data: selector #agents
data: mode append
data: elements <tr><td> Agent Smith 10 </td><td> ... </td></tr>
...
data: elements <tr><td> Agent Smith 14 </td><td> ... </td></tr>
```

This interaction establishes a reactive feedback loop in which server-updated
signals continuously drive future Hypermedia Control interactions while the
HTML interface is progressively refined through partial patches. Compared with
the Backend-Driven HTML Patch pattern, where the server directly replaces a
rendered region in response to a discrete user action, this pattern introduces
client-side signal state that persists across multiple interactions. Rather
than discarding intermediate state after each response, the client retains the
updated signal values and reuses them to parameterize the next request. The
server therefore does not need to embed pagination context within the HTML
itself; instead, it delegates that responsibility to the signal store, keeping
both the HTML fragments and the backend routes free of session-specific state.

## Signal-based Formless Submission

Traditionally, data submission is performed through HTML forms, which act as
containers responsible for collecting and serializing input values before
sending them to the server. In this pattern, that responsibility is delegated
to reactive signals. Input elements are directly bound to signals, and
*hypermedia controls* automatically include the current signal values when
dispatching requests. As a result, form-like interactions can be implemented
without relying on explicit `form` elements or manual client-side
serialization logic.

Two interface states are involved in the interaction: the first corresponds to
a read-only view of the user data, while the second exposes editable input
fields bound to reactive signals. The server controls transitions between
these states by emitting HTML patch operations that replace the current
representation with the appropriate one.

The listing below presents the HtmlFlow code responsible for the read-only
representation. This view displays the current contact information and
provides two *hypermedia controls* that allow the user to transition to a
different presentation state. The `Edit` control requests the editable
representation, while the `Reset` control restores the original contact
information. In both cases, the server responds with an HTML patch that
replaces the corresponding interface region with the appropriate
representation, without the need for full page reloads.

```kotlin
div { attrId("edit-contact")
    p { text("First Name: John") }
    p { text("Last Name: Doe") }
    p { text("Email: joe@blow.com") }
    div {
        button { 
            dataOn(Click) { get(::editContact) }
            text("Edit")
        }
        button { 
            dataOn(Click) { patch(::clickToEditSignalsReset) }
            text("Reset")
        }
    }
}
```

The listing below presents the backend route responsible for handling the edit
request. The endpoint retrieves the contact information from a data access
layer and then renders the HTML, which remains the authoritative application
state. After persisting the updated values, the endpoint responds with a
`text/event-stream` payload containing an HTML patch of the read-only
representation. This patch replaces the element identified by `edit-contact`,
allowing the interface to transition back to the read-only view while
reflecting the updated contact information.

```kotlin
private suspend fun RoutingContext.editContact() {
    call.respondBytesWriter(ContentType.Text.EventStream, OK) {
        with(ServerSentEventGenerator(response(this))) {
            val contactInfo = getContact()
            patchElements(hfReadOnlyFragment.render(contactInfo))
        }
    }
}
```

The editable representation, shown in the listing below, presents the HtmlFlow
code responsible for editing the contact information. The view contains three
input elements, each bound to a corresponding reactive signal through
`dataBind`. When the `Save` control is activated, the *hypermedia control*
dispatches an HTTP request. The current values of all bound signals are
automatically serialized and included in the request body, without requiring
an explicit HTML form or imperative JavaScript serialization logic.

```kotlin
div {
    attrId("edit-contact")
    p { text("First Name"); input { dataBind(firstName) } }
    p { text("Last Name"); input { dataBind(lastName) } }
    p { text("Email"); input { dataBind(email) } }
    div {
        button {
            dataOn(Click) { put(::saveClickToEdit) }
            text("Save")
        }
        button {
            dataOn(Click) { get(::cancelClickToEdit) }
            text("Cancel")
        }
    }
}
```

The listing below presents the backend route responsible for handling the save
request. The endpoint decodes the serialized signals received in the request
body, updates the server-side contact information, and responds with a
`text/event-stream` payload containing an HTML patch of the read-only
representation. This patch replaces the element identified by `edit-contact`,
causing the interface to return to the read-only view with the updated contact
information.

```kotlin
class Signals(val first: String, val last: String, val email: String)

private suspend fun RoutingContext.saveClickToEdit() {
    call.respondBytesWriter(ContentType.Text.EventStream, OK) {
        with(ServerSentEventGenerator(response(this))) {
            val datastarBodyArgs = call.request.call.receiveText()
            val updatedSignals =
                Json.decodeFromString<Signals>(datastarBodyArgs)
            saveNewContactInfo(updatedSignals)
            patchElements(hfReadOnlyFragment.render(updatedSignals))
        }
    }
}
```

In this pattern, the UI is structured around two reactive view modes, read-
only and edit, whose visibility and behavior are governed by the server
through HTML patches rather than traditional form submissions or full page
reloads.

# PetClinic Case Study

Here we analyze a case study based on the official Kotlin version of Spring
PetClinic. Spring PetClinic is an open-source application originally developed
in the Java ecosystem and commonly used to demonstrate web development
concepts, architectural patterns, and framework capabilities. It was inspired
by the PetStore application, which illustrated the use of J2EE technologies
for developing enterprise e-commerce web applications. The application was
selected as a case study because it represents a realistic backend-driven web
application with conventional server-side rendering patterns, including
controllers, domain models, repositories, and HTML views. Its structure
provides a representative baseline for evaluating how existing web application
workflows can be transformed into reactive *hypermedia controls* while
preserving the backend as the primary source of application state and
interaction logic. The original application uses Thymeleaf for server-side
templates; in our case study, those views were reimplemented using the
HtmlFlow Datastar DSL, and the Spring MVC controllers were adapted to return
completed views, partial views, and signal patches.

The migration changes how HTML representations are created and updated, with
the goal of demonstrating how a backend-driven application can express
reactive *hypermedia controls* through a statically typed Kotlin DSL while
following a hypermedia-first approach. The first change concerns the owner
search workflow. In the original implementation, the user enters a last name
in a form and submits it as shown in the listing below. The browser then
issues a `GET` request to the `/owners` resource, with the last name encoded
as a query parameter. This interaction results in the server generating a new
HTML representation containing the query results, which replaces the current
document displayed by the browser.

```html
<form th:object="${owner}" th:action="@{/owners}" method="get">
    <div>
        <input th:field="*{lastName}"/>
    </div>
    <div>
        <button type="submit">Find Owner</button>
    </div>
</form>
```

Through the HtmlFlow Datastar API, the same workflow is redefined as an active
search, as shown in the listing below. In this approach, the input element
acts as a Hypermedia Control that triggers requests during user interaction,
while the backend response updates only the affected results table fragment
instead of replacing the complete page.

```kotlin
val lastName = dataSignal(lastName)  
input {
    dataBind(lastName)
    dataOn(Input) {
        get("/owners/find")
    }
}
```

The input element in the listing above remains a standard HTML element, but
the DSL enriches it into a Hypermedia Control: the `Input` event defines the
interaction trigger, `get("/owners/find")` specifies the HTTP request and
target resource, and the server response determines how the resulting fragment
is integrated through a Datastar patch operation. The corresponding request
handler decodes the Datastar signal payload included with the request and
extracts the current `lastName` value. This value is then used to query the
repository. Instead of generating and returning a complete HTML page, the
handler emits a Server-Sent Event containing an element patch that updates
only the table body fragment with the matching results. The key change is not
only that the interface becomes more interactive, but that the interaction
remains hypermedia-first. The server continues to define the available
operations, process the requests, and determine the resulting UI
representation, while the client is responsible only for applying the received
updates.

The same pattern appears in owner and pet edit operations, and in pet
creation. In the original application, these interactions navigated to
separate pages, requiring a full-page render for each. In the migrated
version, they no longer require navigation to separate pages. The controls
send requests to the server with the relevant signal states, and the server
responds with partial element patches that replace only the affected owner or
pet region. For example, the owner-detail page includes a control for
initiating pet creation, as shown in the listing below. The corresponding
request handler responds with an element patch that updates the pets table by
inserting a provisional row. This row contains signal-bound input fields and a
save control, as shown in the subsequent listing.

```kotlin
button {
    dataOn(Click) { get("/pets/new") }
    text("Add New Pet")
}
```

```kotlin
val petName = dataSignal("petName")  
input { dataBind(petName) }
button {
    dataOn(Click) { post("/pets/new") }
    text("save")
}
```

When the save control is activated, the handler receives the current pet
signals (in this example `petName`) in the request body, validates and
persists the new pet, and returns another element patch. This patch replaces
the editing row with a default row for an existing pet.

These cases show how Reactive *hypermedia controls* can provide interactive
behavior usually implemented with application-specific JavaScript, while
preserving Hypermedia as the Engine of Application State. The DSL reduces
complexity through statically-typed expressions for *hypermedia controls*,
following a server-driven approach.

# Performance Evaluation

To evaluate the performance of the proposed approach, two complementary
benchmarking methodologies were employed. Lighthouse was used to measure
initial page-load performance under a standardized browser environment, while
JMeter, together with the WebDriver Sampler plug-in and Selenium WebDriver,
was used to benchmark complete interactive workflows. Since both the Datastar
and React implementations rely on client-side JavaScript execution, browser-
based measurements were required. Although Playwright supports browser
automation, it does not provide built-in metrics for throughput, network
transfer, or repeated performance measurements, making it less suitable for
the comparative workload evaluation conducted in this study.

The benchmark is based on selected use cases from the Spring PetClinic case
study. The benchmark with JMeter covers two operations: creating a pet, and
finding owners. The create pet operation exercises the Signal-based Formless
Submission pattern, and the find owners operation exercises the Signal-based
HTML patch pattern. These operations are measured across three implementations
(Thymeleaf, HtmlFlow Datastar, and React), each representing a different
architectural paradigm.

Thymeleaf follows a traditional SSR model in which every user interaction
triggers a full server round-trip and a complete page reload. The HtmlFlow
Datastar implementation follows the Backend-Driven Reactive Hypermedia Control
(BRHC) patterns by combining SSR with transclusion-based updates. The initial
response is fully rendered on the server using HtmlFlow, while subsequent
interactions are handled through *hypermedia controls* that trigger server-
side processing and receive partial HTML or signal updates via Datastar, which
are transcluded into the existing document without a full page reload. React
implements a pure CSR model following the SPA architecture, in which the
server exposes a JSON API and all view logic executes in the browser. The goal
of this evaluation is to compare their relative performance under identical
conditions.

An automated script coordinates the full lifecycle: starting all server
instances, waiting for their availability, executing a warm-up phase to
stabilise the runtime environment, and then running the main benchmark of
sequential iterations. The test plan is defined in a single JMeter file
covering the full workflow for all three implementations. The benchmark
deliberately uses a single execution thread, as introducing concurrency would
add variables that are unrelated to the comparison. Multi-user concurrency is
therefore deliberately excluded and reserved for future work.

Each implementation runs inside a separate Docker container with host
networking and no resource limits, orchestrated through Docker Compose.
Service containers use OpenJDK (Eclipse Temurin) or Node.js; the JMeter
container is based on Debian with Apache JMeter and Chromium.

## Initial Load

The initial-load benchmark was conducted using Lighthouse in navigation mode
under the desktop performance profile. The evaluated page corresponds to the
owner details view, which renders the owner's information, the associated pets
table, and interactive controls for editing the owner, adding new pets, and
managing existing pets. This page was selected because it represents one of
the most feature-rich views in the application, combining server-rendered
content with multiple *hypermedia controls*. The evaluation focuses on three
performance metrics: First Contentful Paint (FCP), which measures when the
first visible content is rendered; Largest Contentful Paint (LCP), which
measures when the main page content finishes rendering; and Speed Index (SI),
which estimates how quickly the page becomes visually complete.

HtmlFlow-Datastar and Thymeleaf exhibit nearly identical initial-load
performance. Both deliver a fully server-rendered HTML document, with the only
additional client-side resource in the HtmlFlow-Datastar implementation being
the lightweight Datastar runtime. In contrast, the React implementation must
download and execute a large JavaScript bundle before the interface can be
rendered.

Consequently, both HtmlFlow-Datastar and Thymeleaf achieve substantially
faster First Contentful Paint, Largest Contentful Paint, and Speed Index
compared to React. Both server-rendered approaches reach visual completeness
promptly, whereas React requires noticeable bundle parsing and execution time.

These results indicate that integrating Datastar into a server-rendered
application introduces negligible overhead during the initial page load. The
small additional runtime required by Datastar does not significantly affect
page rendering, allowing the HtmlFlow Datastar implementation to retain the
initial load characteristics of a traditional SSR application while providing
the reactive interaction model evaluated in the subsequent benchmarks.

## Create Pet

The create pet benchmark evaluates a representative user workflow, which
differs across implementations due to their respective architectural models:


* Thymeleaf (Multi-Page Application): Each user action triggers a full page
  reload driven by the server. The browser loads the owner details page as a
  full HTML document. The user clicks "Add New Pet", navigating to a new URL
  where the server returns a full HTML form page. The user fills the form and
  clicks "Save", submitting a form POST request; the server validates and
  persists the pet, returning a redirect. Finally, the browser follows the
  redirect, and the server returns the updated owner details page.
* HtmlFlow Datastar (Hypermedia-first): The initial load returns a full HTML
  page. Subsequent interactions use Server-Sent Events to deliver partial DOM
  patches. The browser loads the owner details page containing Datastar
  client-side attributes. When the user clicks "Add New Pet", a Datastar
  hypermedia control sends a GET request, and the server responds with an SSE
  stream containing an HTML fragment that is patched into the DOM, inserting
  the input row. The user fills the input fields and clicks "Save", causing
  the Datastar control to send a POST request with the form signals; the
  server validates, persists the pet, and responds with an SSE stream
  replacing the input row with the new pet row.
* React (SPA): The client loads the application entry point, then communicates
  with the server exclusively through JSON API calls, with all view logic
  executing in the browser. The browser loads the SPA entry point from the
  frontend server, and React Router renders the owner details component after
  fetching data via the API. When the user clicks "Add New Pet", client-side
  routing activates the pet component, which fetches auxiliary data and owner
  information in parallel before rendering the form. When the user fills the
  form and clicks "Save", the client sends a JSON POST request; upon receiving
  confirmation, the client navigates back to the owner details and re-fetches
  the owner data to display the new pet.

HtmlFlow Datastar achieved the lowest latency among the three implementations,
outperforming both Thymeleaf and React. This result is particularly noteworthy
given that the three implementations employ fundamentally different
interaction models. Thymeleaf performs three full page loads throughout the
workflow (owner details, pet form, and owner details after redirect). In
contrast, HtmlFlow Datastar performs a single full page load followed by two
SSE exchanges that patch the DOM in place. React, on the other hand, relies on
multiple sequential API interactions combined with client-side rendering.

HtmlFlow Datastar exhibited variability comparable to the other
implementations.

The network transfer volume differed across implementations. Thymeleaf and
HtmlFlow Datastar transferred modest data volumes per request, reflecting the
efficiency of server-rendered HTML payloads, whereas React required
substantially higher network volume due to client-side bundles and JSON API
communication. In all three implementations, the number of pets displayed in
the owner details view was limited to prevent unbounded page growth as pets
accumulated during the benchmark.

## Find Owners

The find owners benchmark assesses owner lookup, where there are also clear
differences in implementation and workflow:


* Thymeleaf (Multi-Page Application): Each user action triggers a full page
  reload driven by the server. The browser loads the find owners page as a
  full HTML document. The user enters a search string to filter by owner last
  name and clicks "Find Owner", triggering a GET request that causes the
  server to return a newly rendered page with the filtered owners.
* HtmlFlow Datastar (Hypermedia-first): The initial load returns a full HTML
  page containing Datastar client-side attributes, with an initially empty
  owner table. As the user enters a search string, the Datastar hypermedia
  control automatically sends a GET request. The server responds with an SSE
  stream containing an HTML fragment that is patched into the DOM, replacing
  the empty table body with the matching results.
* React (SPA): The client loads the SPA entry point, and React Router renders
  the find owners component, which fetches all owners via the JSON API. The
  user enters a search string and clicks "Find Owner", sending a GET request
  to the JSON API, after which React re-renders the table with the filtered
  results.

HtmlFlow Datastar achieved the lowest latency, outperforming both Thymeleaf
and React. This advantage reflects the efficiency of the SSE-based hypermedia
model: the input event triggers a single lightweight SSE request that patches
only the table body, whereas Thymeleaf requires a complete page reload and
React incurs the overhead of client-side rendering and JSON API serialization.
React exhibited consistent client-side rendering cost for a simple table
update independent of server-side processing time.

Regarding network transfer, Thymeleaf and HtmlFlow Datastar transferred
minimal data per request for server-rendered HTML tables, while React's
network volume was substantially higher.

## Summary

The results indicate that HtmlFlow Datastar consistently achieved the best
overall performance across the evaluated interaction benchmarks, delivering
the lowest latency while sustaining high throughput among the three
implementations. Regarding the initial page load, Lighthouse showed that
HtmlFlow Datastar exhibited performance comparable to Thymeleaf, with nearly
identical First Contentful Paint, Largest Contentful Paint, and Speed Index
values. This indicates that integrating the Datastar runtime introduces
negligible overhead during the initial rendering of a server-rendered page.
React, in contrast, exhibited substantially higher rendering times, reflecting
the additional cost of downloading and executing the client-side application
bundle before the interface becomes interactive. During the interaction
benchmarks, React exhibited the highest latency in the create pet benchmark.
In the find owners benchmark, React again exhibited the highest latency,
followed by Thymeleaf, with HtmlFlow Datastar achieving the lowest. In terms
of network transfer, Thymeleaf and HtmlFlow Datastar transferred modest
volumes in both workflows, reflecting the efficiency of server-rendered HTML,
whereas React's network volume was substantially higher across both
benchmarks.

The benchmark was intentionally executed using a single sequential thread to
isolate per-request latency and throughput. Concurrency behavior depends
heavily on the underlying server execution model, whether based on thread
pools or event-loop architectures, and these characteristics vary across the
evaluated implementations. Including such factors in the analysis would
conflate application-level interaction costs with server-side scheduling
behavior. Consequently, the scope of this evaluation is limited to single-user
interaction performance and does not address multi-user workloads, resource
contention, or scalability under concurrent access, which would require a
separate study.

Despite these limitations, the results support the central hypothesis of this
work: a backend-driven, hypermedia-first architecture based on reactive
signals can achieve competitive performance while avoiding some of the latency
and throughput overheads commonly associated with SPA architectures. At the
same time, it preserves the simplicity of a server-centric programming model,
where application state, interaction logic, and UI updates remain primarily
coordinated by the backend.

# Discussion

## Interpretation of the Results

The results indicate that Backend-driven Reactive *hypermedia controls* can be
effectively expressed through a statically typed Kotlin DSL without relying on
application-level JavaScript expressions. More importantly, the approach
shifts a substantial part of the interaction definition from stringly typed
HTML attributes into compiler-checked Kotlin constructs. Existing libraries
such as htmx and Datastar make server-driven interaction practical, but their
native programming model still depends on string-based signal names, URL
references, selectors, event declarations, and inline expressions. These
mechanisms are flexible, but difficult to verify before execution. The primary
benefit of the proposed DSL is therefore not only the reduction of handwritten
JavaScript, but also the movement of reactive hypermedia control definitions
into a statically typed environment where signal bindings, backend handler
references, and expression composition can be checked, refactored, and reused.

The practical applicability of these ideas was evaluated through the Spring
PetClinic migration and the performance benchmark. The migration demonstrated
that workflows originally implemented through Thymeleaf templates and full-
page navigation can be reformulated as reactive *hypermedia controls* based on
partial updates. Furthermore, the benchmark showed lower latency and higher
throughput than the evaluated Thymeleaf and React implementations. Together,
these findings suggest that backend-driven reactive architectures can provide
both expressive interaction models and competitive runtime performance.

The main trade-off is that the DSL shifts complexity from handwritten client-
side scripts and external templates into the backend type system and DSL
design. This trade-off is most relevant in contexts where compiler support,
backend ownership of state, and reduced frontend/backend heterogeneity are
desirable architectural properties. It may be less suitable when teams require
the flexibility of dedicated frontend frameworks or when significant portions
of the interaction logic are naturally client-local.

The approach also has practical limitations. Since it is fundamentally server-
driven and relies on HTTP requests and Server-Sent Events, it assumes network
connectivity for authoritative state transitions. Although Service Workers
could provide partial offline support through asset caching and deferred
synchronization, workflows involving server-pushed updates, conflict
resolution, or strongly consistent state remain challenging. Offline
capabilities should therefore be regarded as future work rather than an
inherent property of the proposed model.

## Threats to Validity

The evaluation presented in this work may be subject to threats to validity.
Although the pattern catalog, the Spring PetClinic migration, and the
performance evaluation provide evidence supporting the proposed approach, the
results should be interpreted within the scope of the adopted methodology,
implementation, and experimental setting.

**Construct validity.** The evaluation focuses on the elimination of
application-level JavaScript expressions and the improvement of type safety
through a statically typed DSL. While the pattern catalog and Petclinic
migration provide evidence that these goals can be achieved, the study does
not directly measure developer productivity, maintainability, defect rates, or
comprehension. Consequently, improvements in these dimensions are inferred
rather than empirically demonstrated.

**Internal validity.** The implementation and evaluation are based on a
specific technology stack consisting of Kotlin, HtmlFlow, Spring MVC, and
Datastar. The observed results may therefore be influenced by characteristics
of these technologies rather than exclusively by the DSL design itself. In
addition, the performance benchmarks were conducted in a controlled
environment and primarily reflect single-user workloads, limiting the ability
to attribute all observed performance differences solely to the architectural
model.

**External validity.** The pattern catalog covers representative reactive
*hypermedia controls* but is not exhaustive. Furthermore, the Petclinic
migration represents a single case study in a well-understood domain. The
results may not generalize to larger systems, domains with substantially
different interaction requirements, or organizations with different
frontend/backend responsibilities.

# Conclusions

This paper presented an approach for expressing Backend-driven Reactive
*hypermedia controls* through a statically typed Kotlin DSL built on top of
HtmlFlow and Datastar. The central idea is to move Hypermedia Control
definitions, signal bindings, and request composition from string-based HTML
attributes into compiler-checked Kotlin constructs, while preserving a
hypermedia-first interaction model in which the server remains the
authoritative source of application state and UI evolution. To support this
model, we defined a catalog of Backend-driven Reactive Hypermedia Control
patterns, showed how they can be expressed without application-specific
JavaScript expressions, and examined their structure in terms of signals,
triggers, requests, and server-emitted updates.

Regarding the expression of *hypermedia controls* and reactive signal bindings
through a statically typed Kotlin DSL while eliminating application-level
JavaScript expressions, the results show that the proposed DSL is expressive
enough to capture the studied reactive interaction patterns without requiring
application-specific JavaScript in the view layer. The runtime still depends
on Datastar in the browser to interpret generated attributes, maintain client-
side signal state, and apply Server-Sent Event patches, but the interaction
logic itself is authored in Kotlin rather than in stringly typed client-side
expressions.

Regarding type safety and the unification of frontend and backend programming
models, the proposed DSL contributes to both objectives by allowing signal
bindings, backend handler references, and HTML construction to coexist within
the same statically typed language and toolchain. Furthermore, the Spring
PetClinic case study demonstrated that workflows originally implemented with
Thymeleaf templates and full-page navigation can be migrated to a backend-
driven reactive model based on partial HTML updates and signal-aware
*hypermedia controls*.

Taken together, these results indicate that hypermedia-first architectures
remain a viable alternative to SPA-centric development for a substantial class
of interactive web applications. By combining server-driven interactions with
a statically typed DSL, the proposed approach reduces frontend/backend
heterogeneity while preserving competitive runtime performance and a compiler-
assisted development experience.

Several future directions follow naturally from this work. First, the approach
should be evaluated on larger and more heterogeneous applications to assess
its scalability across richer interaction models and development teams.
Second, stronger verification mechanisms could be explored for selectors,
patch targets, and signal payload contracts. Third, comparative studies with
template-based and SPA-based implementations are needed to directly evaluate
developer productivity, maintainability, and comprehension. Fourth, future
work could investigate offline-capable variants based on Service Workers,
deferred synchronization, and conflict-resolution strategies. Finally, an
important direction is the integration of this DSL into AI-assisted web
development workflows, where statically typed hypermedia control definitions
could serve as a structured and verifiable target for code generation,
reasoning, and automatic refactoring in LLM-based development tools,
potentially improving both correctness and consistency of generated web
interactions.
