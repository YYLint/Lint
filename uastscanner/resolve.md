---
description: resolve
---

# 类型解析

每一个表达式，也就是 **UExpression** 对象，可以通过 `getExpressionType` 方法来获得对应的 **PsiType**。我们可以根据 **PsiType** 来做类型判断。**PsiType** 定义好了一些基础数据类型的常量：

```java
public static final PsiPrimitiveType BYTE = new PsiPrimitiveType("byte", "java.lang.Byte");
public static final PsiPrimitiveType CHAR = new PsiPrimitiveType("char", "java.lang.Character");
public static final PsiPrimitiveType DOUBLE = new PsiPrimitiveType("double", "java.lang.Double");
public static final PsiPrimitiveType FLOAT = new PsiPrimitiveType("float", "java.lang.Float");
public static final PsiPrimitiveType INT = new PsiPrimitiveType("int", "java.lang.Integer");
public static final PsiPrimitiveType LONG = new PsiPrimitiveType("long", "java.lang.Long");
public static final PsiPrimitiveType SHORT = new PsiPrimitiveType("short", "java.lang.Short");
public static final PsiPrimitiveType BOOLEAN = new PsiPrimitiveType("boolean", "java.lang.Boolean");
public static final PsiPrimitiveType VOID = new PsiPrimitiveType("void", "java.lang.Void");
public static final PsiPrimitiveType NULL = new PsiPrimitiveType("null", (String)null);
```

 也可以通过 `getCanonicalText` 方法来获取 **PsiType** 的完整类名，根据类名做字符串匹配从而判断是否符合条件。而如果我们需要进一步判断类的继承关系，则需要获取 **PsiClass** 对象。比如

```kotlin
val cls: PsiClass? = (exp.getExpressionType() as? PsiClassType)
        ?.resolve()
```

 如果 **PsiType** 是 **PsiClassType** 类型，通过 `resolve` 方法就可以获取一个 **PsiClass** 实例。**PsiClass** 具有 `getSuperClass` 和 `getInterfaces` 方法可以查询类的继承关系。更多时候，我们可以通过[求值工具](evaluator.md)来判断，比如：

```kotlin
val isString:Boolean = context.evaluator.extendsClass(cls, 
        "java.lang.CharSequence", false)
```

 `JavaEvaluator.extendsClass` 方法第一个参数是要判断的 **PsiClass** 对象，第二个参数是父类的完整类名，第三个参数为是否要求直接继承。

