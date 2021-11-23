---
title: Effective Kotlin-35：用DSL创建复杂对象
tags: Kotlin
date: 2021-11-23 20:19:39
categories: Kotlin
description:
---
#### 一、概述：
利用Kotlin的语言特性可以创建可配置的领域语言-DSL（Domain Specific Language）。这种DSL用于表示复杂对象和对象继承结构很有用，它能隐藏模板代码和复杂性，让使用者更好的表达自己的意图。

例如，用Kotlin DSL的方式表示HTML的结构：

```kotlin
body {
    div {
        a("https://kotlinlang.org") {
            target = ATarget.blank
            +"Main site"
        }
    }
    +"Some content"
}
```

其他平台的视图也可以使用DSL来定义。下面是Android上使用Anko库定义的界面：
```kotlin
verticalLayout {
    val name = editText()
    button("Say Hello") {
        onClick { toast("Hello, ${name.text}!") }
    }
}
```

同样的，使用TornadorFX库定义的桌面应用如下：
```kotlin
class HelloWorld : View() {
    override val root = hbox {
        label("Hello world") {
            addClass(heading)
        }

        textfield {
            promptText = "Enter your name"
        }
    }
}
```

DSL通常也被用于定义数据和配置。下面是Ktor定义的API：
```kotlin
fun Routing.api() {
    route("news") {
        get {
            val newsData = NewsUseCase.getAcceptedNews()
            call.respond(newsData)
        }
        get("propositions") {
            requireSecret()
            val newsData = NewsUseCase.getPropositions()
            call.respond(newsData)
        }
    }
    // ...
}
```

这是在Kotlin Test中定义的测试用例：
```kotlin
class MyTests : StringSpec({
    "length should return size of string" {
        "hello".length shouldBe 5
    }
    "startsWith should test for a prefix" {
        "world" should startWith("wor")
    }
})
```

我们甚至可以使用Gradle DSL定义Gradle配置：
```kotlin
plugins {
    `java-library`
}

dependencies {
    api("junit:junit:4.12")
    implementation("junit:junit:4.12")
    testImplementation("junit:junit:4.12")
}

configurations {
    implementation {
        resolutionStrategy.failOnVersionConflict()
    }
}

sourceSets {
    main {
        java.srcDir("src/core/java")
    }
}

java {
    sourceCompatibility = JavaVersion.VERSION_11
    targetCompatibility = JavaVersion.VERSION_11
}

tasks {
    test {
        testLogging.showExceptions = true
    }
}
```

使用DSL简化了层次结构和复杂对象的创建。在DSL构建块中，你可以使用Kotlin语法和代码提示。也许你用过Kotlin DSL，但你更应该学会如何定义它。


#### 二、定义自己的DSL

在理解如何定义DSL前，先明白带接收者的函数类型。首先什么是函数类型呢？它是一个能被当成函数使用的对象。例如：`filter`函数，参数predicate决定集合是否包含此元素。
```kotlin
inline fun <T> Iterable<T>.filter(
    predicate: (T) -> Boolean
): List<T> {
    val list = arrayListOf<T>()
    for (elem in this) {
        if (predicate(elem)) {
            list.add(elem)
        }
    }
    return list
}
```

函数类型的例子：  
+ ()->Unit - 无参函数，返回Unit
+ (Int)->Unit - 单参Int函数，返回Unit
+ (Int)->Int - 单参Int函数，返回Int
+ (Int, Int)->Int - 双参Int函数，返回Int。
+ (Int)->()->Unit - 单参Int函数，返回无参函数。 
+ (()->Unit)->Unit - 参数为函数的函数，返回Unit


创建函数类型实例的方式有：  
+ 使用lambda表达式
+ 使用匿名函数
+ 使用函数引用


例如，下面的函数
````kotlin
fun plus(a: Int, b: Int) = a + b
````
同样可以声明成这样：
```kotlin
val plus1: (Int, Int) -> Int = { a, b -> a + b }
val plus2: (Int, Int) -> Int = fun(a, b) = a + b
val plus3: (Int, Int) -> Int = ::plus
```

上面的例子中，由于指明了属性的类型，所以lambda和匿名函数中的参数能被推导出来。换一种方式说：如果指定参数类型，函数类型可以被推导。

```kotlin
val plus4 = { a: Int, b: Int -> a + b }
val plus5 = fun(a: Int, b: Int) = a + b
```

函数类型以对象来表达函数，匿名函数也是一个普通函数只是没名字而已。lambda则是匿名函数的简短书写方式。

我们可以使用函数类型去表示函数，那拓展函数呢？
```kotlin
fun Int.myPlus(other: Int) = this + other
```

之前提到匿名函数只是没名字而已，匿名拓展函数也一样：
```kotlin
fun Int.myPlus(other: Int) = this + other
```

那此函数的类型是什么呢？一种表示拓展函数的特殊类型，叫：带接收者的函数类型。它看起来和普通函数很像，只是在参数之前额外的指定了接收者的类型，他们之间用一个点分割开来。
```kotlin
val myPlus: Int.(Int) -> Int =
    fun Int.(other: Int) = this + other
```

这种函数能通过lambda表达式定义，尤其是一个带接收者的lambda表达式，因为作用域内部的this关键字引用了拓展接收者。（此例中是一个Int类型的实例）：
```kotlin
val myPlus: Int.(Int)->Int = { this + it }
```

用匿名拓展函数或带接收者的lambda表达式创建的对象能被三种方式调用：
+ 像对象一样使用`invoke`方法调用
+ 像非拓展函数一样
+ 和普通的拓展函数一样

```kotlin
myPlus.invoke(1, 2)
myPlus(1, 2)
1.myPlus(2)
```

带接收者的函数类型最大的特点是，它改变了`this`的指向。这种特点如何使用呢？假设一个类需要依次设置属性：
```kotlin
class Dialog {
    var title: String = ""
    var text: String = ""
    fun show() { /*...*/ }
}

fun main() {
    val dialog = Dialog()
    dialog.title = "My dialog"
    dialog.text = "Some text"
    dialog.show()
}
```

重复引用此dialog很不方便，如果我们使用带接收者的lambda，this就指向dialog，我们可以省略this，因为接收者能够隐式调用
```kotlin
class Dialog {
    var title: String = ""
    var text: String = ""
    fun show() { /*...*/ }
}

fun main() {
    val dialog = Dialog()
    val init: Dialog.() -> Unit = {
        title = "My dialog"
        text = "Some text"
    }
    init.invoke(dialog)
    dialog.show()
}
```

按照这个思路，你可以定义一个包含创建此dialog对象所需的所有公共部分的逻辑，只把需要变化的属性的设置给调用者。
```kotlin
class Dialog {
    var title: String = ""
    var text: String = ""
    fun show() { /*...*/ }
}

fun showDialog(init: Dialog.() -> Unit) {
    val dialog = Dialog()
    init.invoke(dialog)
    dialog.show()
}

fun main() {
    showDialog {
        title = "My dialog"
        text = "Some text"
    }
}
```

这就是最简单的DSL例子。因为大部分这些构建函数都是重复的，已经被抽取到一个apply函数内，能被直接使用，不用再定义DSL构建器去设置属性了。

```kotlin
inline fun <T> T.apply(block: T.() -> Unit): T {
    this.block()
    return this
}

Dialog().apply {
    title = "My dialog"
    text = "Some text"
}.show()
```

对于DSL来说，带接收者的函数类型是最基本的构建块。我们一起来写一个简单的DSL用于创建下面的HTML表格：
```kotlin
fun createTable(): TableBuilder = table {
    tr {
        for (i in 1..2) {
            td {
                +"This is column $i"
            }
        }
    }
}
```

在DSL开头我们定义了table函数。由于是最外层定义，所以它没用接收者。在table的内部可以使用tr，所以tr函数只能用于table内部。同理，td只能用于tr内部。
```kotlin
fun table(init: TableBuilder.()->Unit): TableBuilder {
    //...
}

class TableBuilder {
    fun tr(init: TrBuilder.() -> Unit) { /*...*/ }
}

class TrBuilder {
    fun td(init: TdBuilder.()->Unit) { /*...*/ }
}

class TdBuilder {
    var text = ""

    operator fun String.unaryPlus() {
        text += this
    }
}
```

现在，我们定义好了DSL。为了让它运行，每一步我们都需要创建一个builder并用参数里面的函数初始化它（下面例子中的`init`）。builder将包含所有init函数指定的数据。这些数据是我们需要的。我们可以直接返回这个构建器也可以返回另一个包含这些数据的新对象。此例中我们直接返回构建器。下面是`table`函数的定义：
```kotlin
fun table(init: TableBuilder.()->Unit): TableBuilder {
    val tableBuilder = TableBuilder()
    init.invoke(tableBuilder)
    return tableBuilder
}
```
我们可以使用apply函数简化函数：
```kotlin
fun table(init: TableBuilder.()->Unit) =
    TableBuilder().apply(init)
```

其他函数也使用apply简化：
```kotlin
class TableBuilder {
    var trs = listOf<TrBuilder>()

    fun tr(init: TrBuilder.()->Unit) {
        trs = trs + TrBuilder().apply(init)
    }
}

class TrBuilder {
    var tds = listOf<TdBuilder>()

    fun td(init: TdBuilder.()->Unit) {
        tds = tds + TdBuilder().apply(init)
    }
}
```


#### 三、何时使用？  
DSL给了我们一种定义信息的方式。它能用于表达你所想表达的东西。但是对于用户而言这些信息之后如何使用不是很清晰。在Anko、TornadoFX或HTML DSL中，我们相信视图会根据我们的定义形式被构建，但很难跟踪准确跟踪如何构建。一些复杂的使用方式很难发现。用法也会让那些不习惯的人感到困惑。更不用说维护了。它们的定义方式可能是一种成本——在开发人员困惑和性能方面都是如此。当我们可以使用其他更简单的特性时，dsl就太过了。虽然当我们需要表达下面的内容时，它们真的很有用:
+ 复杂的数据结构
+ 继承结构
+ 大量数据

描述事物不止DSL方式，还可以用builder、或只用构造器代替。dsl是关于此类结构的模板代码消除。当您看到可重复的样板代码，并且没有更简单的Kotlin特性可以提供帮助时，您应该考虑使用DSL。



#### 四、总结：  
DSL是语言中的一种特殊形式。它可以非常简单地创建复杂的对象，甚至整个对象层次结构，如HTML代码或复杂的配置文件。另一方面，DSL实现可能会让新开发人员感到困惑或困难。它们也很难定义。这就是为什么只有当它们提供真正的价值时才应该使用它们的原因。例如，用于创建一个真正复杂的对象，或者可能用于复杂的对象层次结构。这就是为什么最好在库中而不是在项目中定义它们的原因。制作一个好的DSL并不容易，但是一个定义良好的DSL可以让我们的项目做得更好。

参考：  
[Effective Kotlin Item 35: Consider defining a DSL for complex object creation](https://kt.academy/article/ek-dsl)














