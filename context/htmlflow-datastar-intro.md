# HtmlFlow-Datastar

HtmlFlow-Datastar Kotlin DSL that provides a Type-Safe Hypermedia-First DSL for
Reactive Backend-Driven Web Applications with
[Data-Star](https://data-star.dev/). 


HtmlFlow-Datastar Kotlin DSL provides a type-safe way to define Datastar
attributes in Kotlin. Next is a sample of the Counter example with a strongly
typed signal and another one from Active Search using events and modifiers:


<table class="table">
<tr>
<td>


```kotlin
div {
  val count: Signal = dataSignal("count", 0)
  div {
    dataInit { get(::getCounterEvents) }
    span {
      attrId("counter")
      dataText{ +count }
    }
  }
}
```

</td>
<td>


```kotlin
div {
  attrId("demo")
  input {
    attrType(EnumTypeInputType.TEXT)
    attrPlaceholder("Search...")
    dataBind("search")
    dataOn(Input) {
      get(::search)
      modifiers { debounce(200.milliseconds) }
    }
  }
}
```

</td>
</tr>
<tr>
    <td>
        Note that the <code>count</code> variable is of type <code>Signal</code>
        and simply binds to the data-text in a type-safe way,
        regardless of the name passed to the <code>Signal</code> constructor.
    </td>
    <td>
        Modifiers such as <code>debounce</code> are added inside a lambda using 
        builders, following an idiomatic Kotlin style.
    </td>
</tr>
</table>

Also, HtmlFlow DSL supports strongly typed DataStar expressions, allowing their
composition with the infix operator `and`, such as in the expression `!fetching
and get(::clickToLoadMore)`. Note how the JavaScript expression for the
`onclick` event handler (right side) is expressed in Kotlin through HtmlFlow in
a type-safe way:


<table>
<tr>
<td>
    
```kotlin
button {
  val fetching = dataIndicator("_fetching")
  dataAttr("disabled") { +fetching }
  dataOn(Click) {
      !fetching and get(::clickToLoadMore)
  }
  text("Load More")
}
```

</td>
<td>

```html
<button
  data-indicator:_fetching
  data-attr:disabled="$_fetching"
  data-on:click="!$_fetching && @get('/examples/click_to_load/more')"
>
    Load More
</button>
```

</td>
</tr>
</table>