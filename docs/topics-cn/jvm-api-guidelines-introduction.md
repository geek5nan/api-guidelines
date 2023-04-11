[//]: # (title: Introduction)

优秀的公共库通常具备以下特性:
* 后向兼容性
* 易于理解，同时有完善的文档
* 尽可能的降低认知复杂度
* 统一的 API

这份指南包含了对于编写公共库的最佳实践的总结，同时也提出了一些写公共库时值得考虑的点。

这份指南由以下几个章节组成：
* [可读性](jvm-api-guidelines-readability.md)
* [可预测性](jvm-api-guidelines-predictability.md)
* [可调试性](jvm-api-guidelines-debuggability.md)
* [后向兼容性](jvm-api-guidelines-backward-compatibility.md)

由于各章节中的许多最佳实践都有对降低 API 认知复杂度的建议，因此在介绍这些实践之前，本指南先对 认知复杂度 进行一个简单的解释。

### 认知复杂度

认知复杂度是一个关于心智负担的衡量标准，譬如需要花费多少精力来理解一份代码。如果代码仓库的认知复杂度较高，那么这份代码就较难理解和维护，而这样的代码可能会为开发过程带来更多的 bugs 和延期。

举例来讲，如果一个类或模块具有多份职责且不遵循单一职责原则（SRP），那它就是一份高认知复杂度的代码。
一个做了太多事情的类或模块是很难理解的修改的；与之相反的，如果一个类或模块有着清晰和良好的定义的职责，那工作起来会简单许多。

函数也可能具有较高的认知复杂度。一些「Bad Written」的函数通常有以下特征：
* 有着较多的参数、变量或者循环
* 有着较为复杂的逻辑嵌套在 if-else 语句中

带有这样特征的函数要比有着清晰和简单逻辑的函数（少量的参数，简单且易于理解的控制流）难维护。

一个高认知复杂度函数的示例：

```kotlin
fun processData(
    data: List<String>,
    delimiter: String,
    ignoreCase: Boolean,
    sort: Boolean,
    maxLength: Int
) {
    // Some complex processing logic
}
```

通过分解这个函数的功能来降低它的认知复杂度：

```kotlin
fun delimit(data: List<String>, delimiter: String) { … }
fun ignoreCase(data: List<String>) { … }
fun sortAscending(data: List<String>) { … }
fun sortDescending(data: List<String>) { … }
fun maxLength(data: List<String>, maxLength: Int) { … }
```

也可以通过扩展函数来进一步简化上述代码：
```kotlin
fun List<String>.delimit(delimiter: String): List<String> { … }
fun List<String>.sortAscending(): List<String> { … }
fun List<String>.sortDescending(): List<String> { … }
fun List<String>.maxLength(maxLength: Int): List<String> { … }
…
```
