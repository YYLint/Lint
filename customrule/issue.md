---
description: create an issue
---

# 创建问题

我们可以通过 **Lint** 的 api 来完成 **Issue** 的定义：

```java
public static Issue create(
        String id,
        String briefDescription,
        String explanation,
        Category category,
        int priority,
        Severity defaultSeverity,
        Implementation implementation)
```

 但是在实际的开发过程中发现，在 `Kotlin` 中调用这个方法会导致编译失败。所以这里简单地规避一下，用 `Java` 编写一个 `IssueFactory` 类来调用 `Issue.create()` 方法，然后在 `Kotlin` 中调用 `IssueFactory` 的方法来完成 **Issue** 的创建。而且我们可以在 `IssueFactory` 中完成一些切片编程，比如我在每一个 `IssueFactory` 创建出来的 **Issue** 的详细解释\(`explanation`参数\)中，都会加上 

```text
"**如果你认为这是一个误报，请联系手Y主程序组。**"
```

 这样的一句话。

然后我们来编写一个简单的代码扫描器：

```kotlin
class MyDetector : Detector() {

    companion object {
        val ISSUE_HELLO_WORLD: Issue = IssueFactory.create(
                "HelloWorld",
                "Don't use method helloWorld",
                "Don't use method helloWorld, because it's too simple",
                Category.CORRECTNESS,
                10,
                Severity.WARNING,
                Implementation(
                        MyDetector::class.java,
                        EnumSet.of(Scope.JAVA_FILE)
                ))
    }
}
```

 从 `create` 方法中我们可以大概了解参数的含义：第一个参数是 **Issue** 的id，这个id必须是全局唯一的，我们可以在 `Gradle` 通过这个id来指定这个 **Issue** 做一些特定的配置，比如修改严重等级或者是disable掉。而且这个id最好是**驼峰命名**规则的字符串，不要把id写为 "~~HELLO\_WORLD~~" ，这样的id虽然不会影响生成报告，但会有一定概率失去 IDE 的标示提醒。第二个参数和第三个参数分别是简短说明和详细说明，详细说明可以用一些简单的 markdown 语法来修饰，具体请看 [TextFormat](../other.md#wen-ben-ge-shi)。第四参数是类别，请参考[ Issue Category](../issue/category.md)。第五个参数填的是10，表示优先级，0最低，10最高。第六个参数是严重级别，请参考[Issue Severity](../issue/severity.md)。第七个参数是 `Implementation` 类型，用于指定具体的 `Detector` 的实现类，以及扫描的文件范围，请参考[ Scope](../other.md#sao-miao-fan-wei)。

接下来把这个 **Issue** 注册到 **Lint** 当中：

```kotlin
class MyIssueRegistry : IssueRegistry() {

    override val issues: List<Issue>
        get() = listOf(MyDetector.ISSUE_HELLO_WORLD)

    override val api = CURRENT_API
}
```

 在 _module_ 级的 _build.gradle_ 中声明这个 `IssueRegistry`：

```groovy
dependencies {
    compileOnly "com.android.tools.lint:lint-api:26.1.2"
    compileOnly "com.android.tools.lint:lint-checks:26.1.2"
    testImplementation "com.android.tools.lint:lint:26.1.2"
    testImplementation "com.android.tools.lint:lint-tests:26.1.2"
}

jar {
    manifest {
        attributes("Lint-Registry": "com.yy.mobile.lint.MyIssueRegistry")
    }
}
```

 这样就成功地把我们的自定义规则 **ISSUE\_HELLO\_WORLD** 添加到了 **Lint** 的扫描中。

