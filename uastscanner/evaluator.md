---
description: evaluator
---

# 代码求值

**Lint** 有多个 **Evaluator** 辅助我们完成代码检测，可以"运行"一部分静态的代码并获得结果。

我们在[类型解析](resolve.md)中使用过 **JavaEvaluator**，这个类可以帮助我们获得一些类的层级信息。比如

```kotlin
abstract fun extendsClass(
        cls: PsiClass?,
        className: String,
        strict: Boolean): Boolean

abstract fun implementsInterface(
        cls: PsiClass,
        interfaceName: String,
        strict: Boolean): Boolean

open fun isMemberInSubClassOf(
        method: PsiMember,
        className: String,
        strict: Boolean): Boolean

open fun isMemberInClass(
        method: PsiMember?,
        className: String): Boolean 
```

 **ConstantEvaluator** 可以帮助我们直接获取某个常量的值，包括常量的运算表达式的值。我们甚至可以用这个工具类来判断一些 if 表达式的真假。

**TypeEvaluator** 是专门用于类型解析的工具类。这个类可以快速提取 **UElement** 的 **PsiType**，比如方法的返回值，变量的值，或者是整个表达式的结果。类型解析强大之处在于猜测变量的运行时类型，变量可能是一个抽象类类型或者接口类型，在运行过程中被赋值为具体的实现类型，我们可以去猜测并获得具体的实现类型。

**ResourceEvaluator** 用于判断节点元素引用着哪个资源，获得一个 **ResourceUrl** 对象。

还有其他的一些工具类比如：**LintUtils**, **SdkUtils**, **UastLintUtils**, **XmlUtils**。这些工具类都会逐渐被改写成 Kotlin 的扩展函数，所以在以后的 Lint 版本中就不需要再专门翻阅这些工具类的文档。

