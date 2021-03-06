---
description: create a detector
---

# 创建扫描器

声明好了 **Issue**，我们需要一个具体的执行者去完成扫描的逻辑。下面我们通过 `MyDetector` 这个类来扫描所有源代码文件当中，是否存在名为 "helloWorld" 的方法。

```kotlin
class MyDetector : Detector(), SourceCodeScanner {

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
            )
        )
    }

    override fun getApplicableMethodNames() = listOf("helloWorld")

    override fun visitMethod(context: JavaContext, node: UCallExpression, method: PsiMethod) {
        System.out.println("yymobile: method: $node return: ${node.returnType}")
    }
}
```

 上面实现非常简单，`MyDetector` 必须继承 `Detector` 类并实现 `SourceCodeScanner` 接口，通过 `getApplicableMethodNames` 方法返回我们感兴趣的一个或多个方法名，然后就可以在 `visitMethod` 方法中处理扫描回调，这里只是简单的把回调参数打印为日志。

在 _:app module_ 中，也就是被静态扫描的项目模块，有分别用 `Java` 和 `Kotlin` 实现的代码，如下：

```java
class Demo {

    void method1() {
        helloWorld("whatever");
    }

    int helloWorld(String param1) {
        return 3;
    }
}
```

上面的是 `Java` 版实现，`Kotlin`版实现也是相同的行为 。然后我们执行 _:app-&gt;verification-&gt;lintDebug_ 的 Gradle Task，对 `Demo` 类做一次扫描，然后可以看到有下面的日志打印结果：

> yymobile: method: org.jetbrains.uast.kotlin.KotlinUFunctionCallExpression@72c40423 return: PsiType:int
>
> yymobile: method: helloWorld\("whatever"\) return: PsiType:int

第一行是扫描 `Kotlin` 类打印的日志，第二行是扫描 `Java`类打印的日志。通过日志我们可以发现 `MyDetector` 已经成功地扫描到了 `helloWorld` 这个方法，并且知道返回类型是 `int` 类型。通过第二行`Java`类的日志，我们可以更清晰地看到 `helloWorld("whatever")` 这个方法调用和参数都打印了出来，而 `Kotlin` 的Expression类根本没有实现 `toString` 方法。这也可以看出 **Lint** 对 `Kotlin`的支持还有非常大的进步空间，对`Java`的语法树处理会更成熟一点，相信以后的版本会有改进。

```kotlin
fun visitMethod(context: JavaContext, node: UCallExpression, method: PsiMethod) 
```

我们可以发现 `visitMethod` 回调中有三个参数，`JavaContext` 类型是静态代码扫描的上下文，这个类非常重要，地位相当于 Android 中的 `android.content.Context` 类，后面可以看到更多关于这个类的作用和用法。`UCallExpression` 和 `PsiMethod` 都是语法树的节点类型，它们分别属于两个系列的节点：U开头的 `UElement` 系列节点和 Psi 开头的 `PsiElement` 系列节点。更详细的介绍会在语法树这章说明。

除了 `visitMethod` 以外，还有很多其他节点我们可以去关心。比如有以下这些：

| 列出关心的元素 | 语法树回调 |
| --- | --- | --- | --- | --- | --- | --- |
| fun getApplicableMethodNames\(\): List&lt;String&gt;? | fun visitMethod\(context: JavaContext, node: UCallExpression, method: PsiMethod\) |
| fun getApplicableConstructorTypes\(\): List&lt;String&gt;? | fun visitConstructor\(context: JavaContext, node: UCallExpression, constructor: PsiMethod\) |
| fun getApplicableReferenceNames\(\): List&lt;String&gt;? | fun visitReference\(context: JavaContext, reference: UReferenceExpression, referenced: PsiElement\) |
| fun appliesToResourceRefs\(\): Boolean | fun visitResourceReference\(context: JavaContext, node: UElement, type: ResourceType, name: String, isFramework: Boolean\) |
| fun applicableSuperClasses\(\): List&lt;String&gt;? | fun visitClass\(context: JavaContext, declaration: UClass\) |
| ... | ... |

很多诸如此类的 visit... 方法，可以通过这些方法来访问自己关心的节点。如何知道想扫描的代码句属于哪种语法树节点？这个会在[语法树章节](../uastscanner/uast.md)详细说明。

然而事实上，这些 visit... 的方法**并不是最常用的方法**。他们相当于是一种快捷方式，当你很明确要扫描的类名或者方法名具体是什么，才能使用这样的方式。但如果是更复杂的规则，比如要扫描所有形如A.B\(\)这样格式的方法调用，且A的类型是C类的子类，visit... 的方法就会捉襟见肘。

更多的情况下会重写 `fun createUastHandler(context: JavaContext): UElementHandler?` 方法来返回一个自定义的 'visitor'，这个访问器会遍历语法树并回调：

```kotlin
class MyDetector : Detector(), SourceCodeScanner {

    companion object {
        //Issue = ...   
    }

    override fun getApplicableUastTypes(): List<Class<out UElement>>? =
        listOf(UMethod::class.java, UClass::class.java)

    override fun createUastHandler(context: JavaContext): UElementHandler? = MyVisitor(context)

    private class MyVisitor(context: JavaContext) : UElementHandler() {
        override fun visitClass(node: UClass) {
            System.out.println("visitClass ${node.qualifiedName}")
        }

        override fun visitMethod(node: UMethod) {
            System.out.println("visitMethod ${node.name}")
        }
    }
}
```

 重写 `getApplicableUastTypes` 方法并返回关心的类型，比如这里对所有的类声明（`UClass`）和所有的方法声明\(`UMethod`\) 进行访问。然后重写 `createUastHandler` 方法返回一个自定义的实现类 `MyVisitor`，在这个类中重写 `visitClass` 和 `visitMethod` 方法，也是简单的日志打印：

> visitClass com.yy.mobile.lintdemo.Demo
>
> visitMethod method1
>
> visitMethod helloWorld

我们的访问器成功地遍历到了整个 `Demo` 类。

