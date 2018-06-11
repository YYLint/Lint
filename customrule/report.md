---
description: report an issue
---

# 报告问题

当 **Detector** 在扫描中命中了一定的规则的时候，也就是说发现了代码中的问题，我们需要把这个**Issue** 报告出来。通过 `JavaContext.report` 方法，可以把 **Issue** 报告给 **LintClient**，然后由具体的 IDE 或者是其他工具把 report 表现出来。比如 IDE 的实现是按照 **Issue** 的严重程度在代码上直接标识出来，而 Gradle 命令则是会生成 XML 或者 HTML 的文件报告。

我们让 `MyDetector` 把所有扫描到的 `helloWord` 方法都报告出一个 **Issue**：

```kotlin
class MyDetector : Detector(), Detector.UastScanner {

    companion object {
        val ISSUE_HELLO_WORLD: Issue = IssueFactory.create(
                "HelloWorld",
                "Don't use method helloWorld",
                "Don't use method helloWorld, because it's too simple",
                Category.CORRECTNESS,
                10,
                Severity.ERROR,
                Implementation(
                        MyDetector::class.java,
                        EnumSet.of(Scope.JAVA_FILE)
                ))
                .setEnabledByDefault(true)
    }

    override fun getApplicableMethodNames() = listOf("helloWorld")

    override fun visitMethod(context: JavaContext,
                             node: UCallExpression,
                             method: PsiMethod) {
        context.report(ISSUE_HELLO_WORLD,
                node,
                context.getNameLocation(node),
                "hello world!")
    }
}
```

编写完 **Detector** 后，可以点击 IDE 上的 Sync 按钮，让修改同步到被检阅的项目的 Jar 包当中。然后执行 _:app -&gt; verification -&gt; lintDebug_ 的 Gradle Task，在 Gradle 脚本运行结束后，可以在 LintConfig 配置的目录下的报告（默认是./build/reports/lint-results-debug.xml）看到对应的 **Issue**：

```markup
<issue
    id="HelloWorld"
    severity="Error"
    message="hello world!"
    category="Correctness"
    priority="10"
    summary="Don&apos;t use method helloWorld"
    explanation="Don&apos;t use method helloWorld, because it&apos;s too simple"
    errorLine1="        helloWorld(&quot;whatever&quot;);"
    errorLine2="        ~~~~~~~~~~">
    <location
        file="/Users/zhangyu/AndroidStudioProjects/yylint/app/src/main/java/co/getpicks/lintdemo/Demo.java"
        line="12"
        column="9"/>
</issue>
```

如果幸运的话，还能在 IDE 中看到对应的标识：

![](../.gitbook/assets/lintreport.bin)

如果遇到了在 XML/HTML 报告中有对应 **Issue**，但 IDE 中却没有对应提示的情况，可以尝试：

* 检查 **Issue** 的id是否用驼峰命名法命名
* 检查是否用了过旧或者过新的接口，**Lint** 每个版本的 api 变化很大，而且要跟 Gradle 的版本相对应
* 如果 **Detector** 中有 `System.out.println` 的代码，删掉
* clean 一下项目
* 重启一下 IDE
* 调整一下心态，有些事情不能强求

