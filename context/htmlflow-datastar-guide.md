## data-\*

[DataStar](https://data-star.dev/) is a hypermedia framework that builds
interactive web applications by extending HTML with declarative **`data-*`
attributes**. Rather than introducing a template language or a client-side
component model, it enhances standard HTML with reactive behavior, making it a
natural fit for HtmlFlow.

The **`htmlflow-datastar`** module extends **`htmlflow-kotlin`** with a
**type-safe DSL** for DataStar's attributes and expressions. Instead of writing
`data-*` attributes and JavaScript expressions as strings, they are represented
through Kotlin APIs that provide compile-time validation, IDE completion, and
seamless integration with the existing HtmlFlow Kotlin DSL.

For complete examples and demo applications, please visit the
[HtmlFlow-Datastar-Examples](https://github.com/xmlet/HtmlFlow-Datastar-Examples)
and
[spring-petclinic-benchmark](https://github.com/xmlet/spring-petclinic-benchmark)
repositories.

### 1. Dependency

Add the **`htmlflow-datastar`** module to your project.

```groovy
implementation 'com.github.xmlet:htmlflow-datastar:1.2.0'
```

### 2. Type-safe signals and attributes

DataStar manages reactive state through **`signals`**, which are created and
consumed by different **`data-*`** attributes. In HtmlFlow Datastar, a signal is
created once using **`dataSignal`** and represented as a typed **`Signal<T>`**
object that can be reused throughout the view. Instead of referencing signals
through string-based **`data-*`** attributes, builders such as **`dataText`**,
**`dataShow`**, and **`dataClass`** accept **`Signal<T>`** objects directly,
providing type safety, IDE completion, and consistent signal references. As
shown below, the **`count`** signal is created once and then passed to
**`dataText`** to display its value.

```kotlin
div {
  val count: Signal<Int> = dataSignal("count", 0)
  div {
     span {
      attrId("counter")
      dataText { +count }
    }
  }
}
```

This produces the equivalent DataStar markup:

```html
<div data-signals="{count: 0}">
  <span id="counter" data-text="$count"></span>
</div>
```

Reusing the same **`Signal<T>`** object avoids misspelled signal names and keeps
references consistent.

### 3. Type-safe events and modifiers

User interactions are defined through **`data-on:*`** attributes. HtmlFlow
Datastar exposes these as type-safe event builders, allowing the event type
(such as **`Click`** or **`Input`**), the action, and any modifiers to be
expressed using Kotlin APIs instead of manually constructing strings.

In the following example, the input field is bound to the **`search`** signal.
Every **`Input`** event triggers a GET request, but only after the user has
stopped typing for **200 ms**, thanks to the **`debounce`** modifier. This
reduces the number of requests sent while the user is typing and improves the
overall user experience.

Notice that the DataStar **`get`** action accepts a Kotlin function reference.
During rendering, this reference is resolved to the URL declared in the
function's route annotation, eliminating the need to hardcode endpoint paths and
ensuring they remain synchronized with the server-side route definitions.

```kotlin
div {
  val search: Signal<String> = dataSignal("search", "")
  input {
    attrType(EnumTypeInputType.TEXT)
    attrPlaceholder("Search...")
    dataBind(search)
    dataOn(Input) {
      get(::search)
      modifiers { debounce(200.milliseconds) }
    }
  }
}
```

Which produces:

```html
<div data-signals="{search: ''}">
  <input type="text" placeholder="Search..." data-bind="search" data-on:input__debounce="@get('/examples/search')" />
</div>
```

Typed events and modifiers eliminate invalid event names and malformed modifier
syntax.

### 4. Type-safe expressions

Many DataStar attributes accept JavaScript-like expressions. Instead of writing
these expressions as strings, HtmlFlow Datastar represents them as Kotlin
objects that can be combined using familiar operators such as **`and`**,
**`or`**, and **`not`**.

In the following example, **`dataIndicator`** creates a signal named
**`_fetching`**. The leading **`_`** follows a DataStar convention indicating
that the signal is local and is therefore not sent to the server when an HTTP
request is made. The signal's value is automatically set to **`true`** while a
request is in progress and back to **`false`** when the request completes.

The **`dataAttr`** attribute binds the value of any HTML attribute to an
expression, keeping it automatically synchronized with the expression's result.
In this case, binding the button's disabled attribute to **`_fetching`** ensures
that the button is disabled while the request is in progress.

The same signal is also used in the **`dataOn`** expression to prevent multiple
requests from being sent if the user clicks the button before the browser has
applied the disabled attribute. This provides an additional safeguard against
duplicate requests.

```kotlin
button {
  val fetching: Signal<Boolean> = dataIndicator("_fetching")
  dataAttr("disabled") { +fetching }
  dataOn(Click) {
      !fetching and get(::clickToLoadMore)
  }
  text("Load More")
}
```

Which generates:

```html
<button
  data-indicator:_fetching
  data-attr:disabled="$_fetching"
  data-on:click="!$_fetching && @get('/examples/click_to_load/more')"
>
  Load More
</button>
```

Expressions can be composed and refactored using Kotlin operators instead of
string concatenation.

**HtmlFlow Datastar** generates standard **DataStar** HTML, enabling developers
to build reactive user interfaces declaratively with a fully **type-safe**
Kotlin DSL while preserving compatibility with the **DataStar** runtime.

### 5. Structured signal values

`dataSignal` and `dataSignals` support Kotlin values beyond primitive types.
Maps and collections are serialized as JavaScript object and array literals, and
classes annotated with
[`@Serializable`](https://kotlinlang.org/docs/serialization.html) (from
`kotlinx.serialization`) are serialized as object literals, with properties in
declaration order. Nested properties can then be accessed in a type-safe way
through `Signal<T>.on(property)`.

:::note Serializing your own classes requires the `kotlinx.serialization`
compiler plugin in your project (`kotlin("plugin.serialization")`). A class
without a serializer is not converted to an object literal: its `toString()` is
emitted as-is. :::

```kotlin
@Serializable
data class Person(
    val name: String,
    val age: Int,
)

div {
    val (person) = dataSignals(
        "person" to Person("John Doe", 18)
    )

    input {
        attrType(EnumTypeInputType.TEXT)
        dataBind(person.on(Person::name))
    }

    p {
        dataText { +person.on(Person::age) }
    }
}
```

This produces:

```html
<div data-signals="{person: {name: 'John Doe', age: 18}}">
  <input type="text" data-bind="person.name" />
  <p data-text="$person.age"></p>
</div>
```

The Kotlin type is what matters. A `String` that happens to contain JavaScript
object syntax remains a string and is not interpreted as a structured value:

```kotlin
dataSignals(
    "person" to "{name: 'John Doe', age: 18}"
)
```

produces:

```html
<div data-signals="{person: '{name: \'John Doe\', age: 18}'}"></div>
```

So use a Kotlin model rather than a string with object syntax, and `Signal.on`
rather than a hand-written string path such as `"person.name"`.

#### Quote style and raw JavaScript

String values are written with single quotes by default. `dataSignal` and
`dataSignals` accept a `quoteStyle` parameter to switch to double quotes:

```kotlin
dataSignal("title", "O'Reilly", quoteStyle = QuoteStyle.DOUBLE)
// signal value: {title: "O'Reilly"}
```

When an initial value must be a JavaScript expression rather than a literal,
wrap it with `asRawJS()`. The string is then emitted unchanged:

```kotlin
val (selections) = dataSignals(
    "selections" to "Array(4).fill(false)".asRawJS()
)
// data-signals="{selections: Array(4).fill(false)}"
```

### 6. Assigning signal values

Inside an expression block, such as `dataOn`, `Signal.setValue` can receive
either a Kotlin value or another signal/expression. These represent different
kinds of assignments.

Assigning a Kotlin value produces a literal, serialized with the same rules as
signal initialization:

```kotlin
dataOn(Click) {
    person.setValue(Person("Jane Doe", 21))
}
```

```text
$person = {name: 'Jane Doe', age: 21}
```

Assigning another signal keeps the assignment as a DataStar expression:

```kotlin
dataOn(Click) {
    person.setValue(otherPerson)
}
```

```text
$person = $otherPerson
```

The first form describes a concrete Kotlin value; the second refers to another
reactive signal, evaluated on the client.
