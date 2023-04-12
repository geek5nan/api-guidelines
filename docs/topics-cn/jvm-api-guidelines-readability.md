[//]: # (title: Readability)

本章节主要关注 [API 连贯性](#API-连贯性)，同时包含以下最佳实践：
* [使用构造者风格的领域特定语言](#使用构造者风格的领域特定语言)
* [尽可能的使用构造函数风格](#use-constructor-like-functions-where-applicable)
* [合理使用成员变量与扩展函数](#use-member-and-extension-functions-appropriately)
* [避免在函数形参列表中使用布尔类型的参数](#avoid-using-boolean-arguments-in-functions)

## API 连贯性

> API Consistent

<details><summary>查看原文</summary>A consistent and well-documented API is crucial for a good development experience. The same is valid for argument order, 
overall naming scheme, and overloads. Also, it's worth documenting all conventions.
</details>
一份 API 的文档是否完善，是否保持了连贯性对开发体验起着关键性的作用。同样的道理也适用于函数形参的顺序、纵观全局的命名规范以及重载函数的选用。此外，在文档中记录一些惯用的约定(译者注：[约定优先于配置](https://zh.wikipedia.org/wiki/%E7%BA%A6%E5%AE%9A%E4%BC%98%E4%BA%8E%E9%85%8D%E7%BD%AE))也能够提升开发体验。

<details><summary>查看原文</summary>For example, if one of your methods accepts `offset` and `length` as parameters, then so should other methods instead of 
accepting `startIndex` and `endIndex`. Parameters like these are most likely of  `Int` or `Long` type, and thus it's 
very easy to confuse them.
</details>

举个例子，如有你的某个方法接收 `offset` 和 `length` 作为形式参数，那么其他相关的方法也应该接收这两个参数（译者注：保持连贯性），而非 `startIndex` 和 `endIndex`。因为这些参数的类型可能都是 `Int` 或 `Long`，非常容易对开发者造成混淆。

<details><summary>查看原文</summary>The same works for parameter order: Keep it consistent between methods and overloads. Otherwise, users of your library 
might guess incorrectly the order they should pass arguments.
<br/>
Here is an example of both preserving the parameter order and consistent naming:
</details>

同样的，形式参数的顺序也很重要：在多个方法与多个重载函数之间要保持形参的顺序；否则开发者很可能会猜错形参的顺序，传递出错误的参数。

如下是一个保持了形参顺序和命名的示例：

```kotlin
fun String.chop(length: Int): String = substring(0, length)
fun String.chop(length: Int, startIndex: Int) =substring(startIndex, length + startIndex)
```

<details><summary>查看原文</summary>If you have many lookalike methods, name them consistently and predictably. This is how the `stdlib` API works: 
there are methods `first()` and `firstOrNull()`, `single()` and `singleOrNull()`, and so on. It's clear from their names 
that they are all pairs, and some of them might return `null` while others might throw an exception.
</details>

如果你有许多看起来相似的方法，尽量给它们统一连贯且可预测的命名。这正是 `stdlib` 库的做法：

`stdlib` 中有 `first()` 和 `firstOrNull()`, `single()` 和`singleOrNull()`等函数，这些函数清晰易懂见名知意，并且总是成对出现 --- 其中有些会返回 `null` 有些会抛出 `Exception`。

## 使用构造者风格的领域特定语言

> Use builder-DSL

<details><summary>查看原文</summary>"Builder" is a well-known pattern in development. It allows you to build a complex entity, not in a single expression, 
but gradually while getting more information. When you need to use a builder, it's better to write it using builder DSL — 
this is binary-compatible and more idiomatic.</details>

["构造者模式"](](https://en.wikipedia.org/wiki/Builder_pattern#:~:text=The%20builder%20pattern%20is%20a,Gang%20of%20Four%20design%20patterns) ) 是一个众所周知的设计模式，它能够让我们渐进的创建一个复杂对象实体，而不是通过某个表达式一次性的创建一个复杂对象。当你需要使用 builder 模式时，更好的做法是使用 builder DSL --- 这种做法可以保证二进制兼容性，同时也更符合 kotlin 语言的习惯。

<details><summary>查看原文</summary>A canonical example of a Kotlin builder DSL is `kotlinx.html`. Consider this example:</details>

`koltinx.html` 库是一个典型的范例，例如下边这个示例：

```kotlin
header("modal-card-head") {
    p("modal-card-title") {
        +book.book.name
    }
    button(classes = "delete") {
        attributes["aria-label"] = "close"
        attributes["_"] = closeModalScript
    }
}
```

<details><summary>查看原文</summary>It could be implemented as a traditional builder. But this is considerably more verbose:</details>

上述代码也可以使用传统的 builder 实现，但看起来会有点啰唆：

```kotlin
headerBuilder()
    .addClasses("modal-card-head")
    .addElement(
        pBuilder()
            .addClasses("modal-card-title")
            .addContent(book.book.name)
            .build()
    )
    .addElement(
        buttonBuilder()
            .addClasses("delete")
            .addAttribute("aria-label", "close")
            .addAttribute("_", closeModalScript)
            .build()
    )
    .build()
```

<details><summary>查看原文</summary>It has too many details that you don't necessarily need to know and requires you to build each entity at the end.</details>

上边这种代码相比 builder-dsl ，多了一些无须关注的细节，并且还需要显式地创建每一个实体。

<details><summary>查看原文</summary>The situation worsens further if you need to generate something dynamically in a loop. In this scenario, you have 
  to instantiate a variable and dynamically overwrite it:</details>

假如需要在循环中动态的生成对象的话，情况只会更糟。在这个场景下，你不得不在循环内实例化一个变量并且动态的覆写这个变量：

```kotlin
var buttonBuilder = buttonBuilder()
    .addClasses("delete")
for ((attributeName, attributeValue) in attributes) {
    buttonBuilder = buttonBuilder.addAttribute(attributeName, attributeValue)
}
buttonBuilder.build()
```

<details><summary>查看原文</summary>Inside the builder DSL, you can directly call a loop and all necessary DSL calls:</details>

如果使用 builder DSL 的话，可以直接在循环内调用必要的 DSL 函数

```kotlin
div("tags") {
    for (genre in book.genres) {
        span("tag is-rounded is-normal is-info is-light") {
            +genre
        }
    }
}
```

<details><summary>查看原文</summary>Keep in mind that inside curly braces it's impossible to check at compile time if you have set all the required attributes. 
To avoid this, put required arguments as function arguments, not as builder's properties. For example, if you want `href`
  to be a mandatory HTML attribute, your function will look like:</details>

有一点值得注意，DSL 大括号内部是无法在编译期间对你所需的参数进行检测的。为了规避这个情况，请把必要的参数作为函数形参，而不是builder 的某个属性。

举个例子，如果你打算让 `href` 作为某个 HTML 元素的主要属性的话，你的函数应该是这个样子：

```kotlin
fun a(href: String, block: A.() -> Unit): A
```

而不是:

```kotlin
fun a(block: A.() -> Unit): A
```

> <details><summary>查看原文</summary>Builder DSLs are [backward compatible](jvm-api-guidelines-backward-compatibility.md) as long as you don't delete anything 
>   from them. Typically this isn't a problem because most developers just add more properties to their builder classes over time.</details>
>
> Builder DSL 是后向兼容的，只要你不删除其中代码的话。通常这不是一个问题，因为大多数开发者只会往 builder 中添加越来越多的属性（译者注：开闭原则）

## Use constructor-like functions where applicable

Sometimes, it makes sense to simplify your API's appearance by using constructor-like functions. A constructor-like function 
is a function whose name starts with a capital letter so it looks like a constructor. This approach can make your library 
easier to understand.

For example, you want to introduce some [Option type](https://en.wikipedia.org/wiki/Option_type) in your library:

```kotlin
sealed interface Option<T>
class Some<T : Any>(val t: T) : Option<T>
object None : Option<Nothing>
```

You can define implementations of all the `Option` interface methods — `map()`, `flatMap()`, and so on. But each time 
your API users create such an `Option`, they must write some logic and check what they create. For example:

```kotlin
fun findById(id: Int): Option<Person> {
    val person = db.personById(id)
    return if (person == null) None else Some(person)
}
```

Instead of having to write the same check each time, you can add just one line to your API:

```kotlin
fun <T> Option(t: T?): Option<out T & Any> =
    if (t == null) None else Some(t)

// Usage of the code above:
fun findById(id: Int): Option<Person> = Option(db.personById(id))
```

Now, creating a valid `Option` is a no-brainer: just call `Option(x)` and you have a null-safe, purely functional Option idiom.

Another use case for using a constructor-like function is when you need to return some "hidden" things: a private instance, 
or some internal object. For example, let's look at a method from the standard library:

```kotlin
public fun <T> listOf(vararg elements: T): List<T> =
    if (elements.isNotEmpty()) elements.asList() else emptyList()
```

In the code above, `emptyList()` returns the following:

```kotlin
internal object EmptyList : List<Nothing>, Serializable, RandomAccess
```

You can write a constructor-like function to lower the [cognitive complexity](jvm-api-guidelines-introduction.md#cognitive-complexity) 
of your code and reduce the size of your API:

```kotlin
fun <T> List(): List<T> = EmptyList

// Usage of the code above:
public fun <T> listOf(vararg elements: T): List<T> =
    if (elements.isNotEmpty()) elements.asList() else List()
```

## Use member and extension functions appropriately

Write only the very core of the API as [member functions](functions.md#member-functions) and everything else as 
[extension functions](extensions.md#extension-functions). It allows you to clearly show to the reader what's 
the core functionality and what's not.

For example, consider a class for a graph:

```kotlin
class Graph {
    private val _vertices: MutableSet<Int> = mutableSetOf()
    private val _edges: MutableMap<Int, MutableSet<Int>> = mutableMapOf()

    fun addVertex(vertex: Int) {
        _vertices.add(vertex)
    }

    fun addEdge(vertex1: Int, vertex2: Int) {
        _vertices.add(vertex1)
        _vertices.add(vertex2)
        _edges.getOrPut(vertex1) { mutableSetOf() }.add(vertex2)
        _edges.getOrPut(vertex2) { mutableSetOf() }.add(vertex1)
    }

    val vertices: Set<Int> get() = _vertices
    val edges: Map<Int, Set<Int>> get() = _edges
}
```

There is a bare minimum of vertices and edges as private variables, functions to add vertices and edges, and 
accessor functions that return an immutable representation of the current state.

You can add all the remaining functionality outside the class:

```kotlin
fun Graph.getNumberOfVertices(): Int = vertices.size
fun Graph.getNumberOfEdges(): Int = edges.size
fun Graph.getDegree(vertex: Int): Int = edges[vertex]?.size ?: 0
```

So, only properties, overrides, and accessors should be members.

## Avoid using Boolean arguments in functions

It's almost impossible to understand what the purpose of a `Boolean` argument is just by reading code anywhere except 
in IDEs, for example, on some version control system sites. Using [named arguments](functions.md#named-arguments) can help 
to clarify this, but for now in IDEs, there is no way to force developers to use them. Another option is to create a function 
that contains the action of the `Boolean` argument and give this function a descriptive name.

For example, in the standard library there are two functions for `map()`:

```kotlin
map(transform: (T) -> R)

mapNotNull(transform: (T) -> R?)
```

It was possible to add something like `map(filterNulls: Boolean)` and write code like this:

```kotlin
listOf(1, null, 2).map(false) { it.toString() }
```

From reading this code, it's tough to infer what `false` actually means. However, if you use the `mapNotNull()` function, 
you can understand the logic at first glance:

```kotlin
listOf(1, null, 2).mapNotNull { it.toString() } 
```

## What's next?

Learn about API's:
* [Predictability](jvm-api-guidelines-predictability.md)
* [Debuggability](jvm-api-guidelines-debuggability.md)
* [Backward compatibility](jvm-api-guidelines-backward-compatibility.md)
