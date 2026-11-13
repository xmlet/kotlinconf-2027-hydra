The presentation *Full Stream Ahead: Breaking the Protocol Barrier with http4k*,
delivered at KotlinConf 2025, explores how http4k extends functional programming
principles to support different communication protocols, including traditional
HTTP, Server-Sent Events (SSE), and WebSockets. It explains the limitations of
the traditional HTTP request-response model when developing real-time
applications and demonstrates how SSE enables continuous server-to-client
communication, while WebSockets support bidirectional communication. The
presentation highlights http4k's functional approach, using composable and
testable Kotlin functions to handle different communication mechanisms. It also
demonstrates the integration of DataStar to build reactive web interfaces using
HTML attributes, signals, and SSE. Through the `merge-fragments` and
`merge-signals` events, DataStar can dynamically update HTML fragments and
reactive client-side state without reloading the entire page. Finally, the
presentation challenges the complexity of traditional Single Page Applications
(SPAs), proposing an alternative architecture based on server-side rendering and
server-driven UI. It illustrates how Kotlin, http4k, and DataStar can be
combined to build modern, interactive, real-time web applications while
significantly reducing frontend complexity.

Based on this ideas we have built HtmlFlow-datastar described in:
* htmlflow-datastar-intro.md
* htmlflow-datastar-guide.md

Our work extends the HtmlFlow Kotlin DSL (Domain-Specific Languages) by
leveraging Kotlin as a statically typed host language to suppress the issues
associated with untyped or string-based expressions in hypermedia control
attributes. The proposed model eliminates JavaScript expressions by replacing
them with statically typed expressions defined directly in the backend host
language, namely Kotlin. We demonstrate the applicability of the approach
through a catalog of *Backend-driven Reactive Hypermedia Control* (BRHC)
patterns (as popularized by htmx examples and also used in Datastar) and a
case study based on the well-known Petclinic Spring MVC web application.

This work is presented in paper-2026-BRHC.md.
