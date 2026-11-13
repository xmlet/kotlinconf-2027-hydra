## 1. Session Type

- [X] Brand new session
- [ ] Session from your speaker profile or another event

## 2. Session Details

### Session Title

Type-Safe Hypermedia Controls: Building Reactive Web UIs with Kotlin and
DataStar

### Description / Abstract

Interactive web applications often put UI behavior in JavaScript expressions
embedded in HTML attributes. Hypermedia libraries such as DataStar make this
approach declarative, but those expressions and signal names are still easy to
mistype and difficult for Kotlin tooling to check. What if we could describe
those interactions in Kotlin and have the compiler help us?

In this session, we’ll introduce HtmlFlow-Datastar, a Kotlin DSL that brings
DataStar’s reactive, hypermedia-first model into HtmlFlow. We’ll see how typed
signals, event builders, modifiers, and composable expressions let Kotlin
describe browser interactions without hand-built JavaScript expression
strings. We’ll follow those controls from an event and HTTP request through
server-rendered HTML fragments or signal updates delivered with
Server-Sent Events.

Then we’ll apply the approach to real workflows from Spring PetClinic:
active owner search, adding a pet, and editing records. These examples show
how the server can remain responsible for application state and UI updates
while the browser applies partial changes instead of reloading whole pages.
We’ll also share a catalog of backend-driven reactive hypermedia patterns and
compare the PetClinic implementation with Thymeleaf and React. In the
evaluated workflows, HtmlFlow-Datastar had the lowest interaction latency;
initial page-load performance was similar to Thymeleaf. The benchmark used a
single sequential thread, so these results are not a claim about concurrent
load or every application.

You’ll leave with practical patterns for expressing reactive web behavior in
type-safe Kotlin, a clearer understanding of the trade-offs between full-page
navigation and server-driven partial updates, and criteria for deciding when
this approach is a good fit. This is not a case for replacing every SPA: it
is a demonstration of how Kotlin teams can build interactive, server-centric
applications with less application-specific JavaScript.

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
[Enter number of years]

## 5. Learning Outcomes

### What are the key takeaways from the talk?

- Use typed signals, events, modifiers, and expressions to catch mistakes
  earlier and make UI-to-handler wiring navigable and refactorable with Kotlin
  IDE tooling.
- Reduce frontend glue code and client-side state handling while keeping
  interaction behavior and UI responses in the Kotlin backend.
- Use server-sent events to update a specific HTML fragment or reactive
  signal without replacing the whole page.
- Recognize backend-driven reactive hypermedia patterns in workflows such as
  active search and inline create/edit operations.
- Compare this server-centric approach with conventional server-rendered and
  SPA implementations, including the limits of the available benchmark.

### What unique perspective do you bring to the topic?

This session connects hands-on DSL design with an applied case study. It
shows how Kotlin’s type system can make DataStar hypermedia controls safer to
author, then demonstrates the approach in Spring PetClinic workflows. A
pattern catalog and a comparison with Thymeleaf and React provide both
reusable design ideas and practical trade-offs, rather than presenting
server-driven UI as a universal replacement for SPAs.

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
- [ ] Kotlin experience is specified.
- [X] Key takeaways are clearly described.
- [X] Unique perspective is explained.
- [ ] Previous talk recordings are included, if available.
- [ ] Rehearsal preference is selected.
- [ ] Lightning talk preference is selected.
- [ ] Additional notes are provided, if necessary.
- [ ] Speaker and co-speaker details are correct.
