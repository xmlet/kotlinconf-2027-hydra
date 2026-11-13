## 1. Session Type

- [X] Brand new session
- [ ] Session from your speaker profile or another event

## 2. Session Details

### Session Title
Taming the Frontend Hydra: Type-Safe Reactive Hypermedia with Kotlin and
Datastar

### Description / Abstract
Modern web development has become increasingly dominated by Single-Page
Applications (SPAs) and heavy client-side JavaScript.
While SPAs deliver fluid user experiences, they impose high cognitive and
maintenance costs: bloated bundles, hydration penalties, duplicated business
logic, and disjointed client-server architectures.
Hypermedia-Driven Applications (HDAs) like htmx and Datastar have revived
server-centric architectures, but their reliance on untyped HTML attributes,
inline JavaScript, and magic string URLs introduces fragility and escapes
compiler verification.

What if we could achieve the reactive responsiveness of modern SPAs with the
simplicity of server-side rendering, fully verified by Kotlin's type system?
In this session, we introduce a type-safe hypermedia architecture that
integrates the HtmlFlow Kotlin DSL with Datastar and Server-Sent Events (SSE).
By bringing hypermedia controls directly into Kotlin, we eliminate handwritten
JavaScript and string-based event declarations.
Instead, reactive signals, event modifiers, and server interactions are
expressed through statically typed Kotlin constructs and compile-time function
references to backend route handlers.

We present a catalog of Backend-driven Reactive Hypermedia Control (BRHC)
patterns, ranging from live SSE state synchronization and active search to
formless submissions and infinite scrolling.
To demonstrate real-world feasibility, we migrated the official Kotlin Spring
PetClinic application from traditional Thymeleaf page reloads to reactive
hypermedia controls.
We share empirical benchmark results comparing our approach against
traditional SSR and a React SPA: HtmlFlow with Datastar achieved near-instant
initial load times matching SSR, while delivering significantly lower latency
and reduced network transfer during interactive workflows.

Attendees will learn how to build real-time, interactive web applications in
pure Kotlin without an SPA framework or a complex frontend build toolchain.
Come discover how to tame the frontend Hydra, eliminate client-side hydration
overhead, and keep your application logic where it belongs—on the server,
backed by Kotlin's type safety.

### Session Format
- [X] Regular session (45 minutes, including optional Q&A)
- [ ] Lightning talk (15 minutes)

### Level
Intermediate;

### Theme
Web development; Kotlin libraries and ecosystem;

## 3. Abstract Preparation

### How did you use AI to prepare this abstract?
[Select the applicable option]

- [ ] AI used to improve or restructure my own text
- [ ] Other: [Specify]

### Please briefly describe whether and how you used AI tools to prepare your abstract.
[Explain how AI tools were used. This information is visible only to the program committee and will not be published.]

## 4. Kotlin Experience

### How many years have you been working with Kotlin?
8 years

## 5. Learning Outcomes

### What are the key takeaways from the talk?
1. **Architectural Principles of Reactive Hypermedia:**
   Understand the Hypermedia-Driven Application (HDA) paradigm as a practical
   alternative to complex SPAs, combining server-side rendering with fine-
   grained DOM patching and reactive signals via Server-Sent Events (SSE).

2. **Type-Safe Hypermedia DSL Design:**
   Learn how to eliminate stringly typed HTML attributes and JavaScript
   snippets using Kotlin's type system, builders, and function references
   (`::routeHandler`) to achieve compile-time verification and safe
   refactoring.

3. **The BRHC Pattern Catalog:**
   Gain practical techniques for implementing common interactive patterns—such
   as live server push, active search with debouncing, infinite scrolling, and
   formless editing—without writing custom client-side JavaScript.

4. **Empirical Performance and Migration Insights:**
   Learn from a realistic case study migrating Spring PetClinic from Thymeleaf,
   backed by reproducible benchmark metrics (latency, throughput, payload size,
   and Lighthouse scores) comparing SSR, reactive hypermedia, and SPAs.

### What unique perspective do you bring to the topic?
As the authors of the HtmlFlow Kotlin DSL and researchers behind the
Backend-driven Reactive Hypermedia Controls (BRHC) framework, we bring both
deep language design experience and empirical research to web development.
While many hypermedia advocates rely on untyped, string-based tools like htmx,
our work directly tackles the missing piece: compile-time type safety and
developer ergonomics in statically typed languages.
Rather than presenting toy examples, our insights are grounded in peer-reviewed
research, a full migration of the Spring PetClinic benchmark, and rigorous
browser automation performance tests (Lighthouse and JMeter WebDriver)
evaluating latency, throughput, and payload costs against traditional SSR and
React SPAs.

## 6. Previous Speaking Experience

### Recordings of previous talks
[Provide links to recordings of previous presentations, if available.]

## 7. Speaker Support and Session Alternatives

### Want to rehearse your talk with us?
- [ ] Yes
- [ ] No

### Would you consider giving a lightning talk?
- [ ] Yes
- [ ] No

> Note: Lightning talk speakers are not eligible for travel and accommodation expenses.

## 10. Submission Checklist

- [X] Session title is clear and accurate.
- [X] Abstract is approximately 200–400 words.
- [X] Session level is appropriate.
- [X] Up to two relevant themes are selected.
- [X] AI usage is disclosed.
- [X] Kotlin experience is specified.
- [X] Key takeaways are clearly described.
- [X] Unique perspective is explained.
- [X] Previous talk recordings are included, if available.
- [X] Rehearsal preference is selected.
- [X] Lightning talk preference is selected.
- [X] Additional notes are provided, if necessary.
- [X] Speaker and co-speaker details are correct.
