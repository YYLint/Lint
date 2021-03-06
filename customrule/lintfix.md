---
description: Lint Fix
---

# 快速修复

**Lint** 提供了一种快速修复的建议手段。我们在指出问题的同时，可以给出问题的解决办法，也就是 **Lint** **Fix**。在 IDE 中可以通过 `Quick Fix / Show Intention Action` 的快捷键 （默认Alt+Enter）来触发：

![replace NavigationUtils with ARouter](../.gitbook/assets/quickfixsample.gif)

**Lint Fix** 的核心就是字符串替换：

* 我们希望在原来的位置上新增代码，可以用新代码替换一个空字符串""
* 希望替换方法调用，用新的方法的字符串替换原来的方法
* 希望删除多余的代码，用空字符串""替换原来多余的代码
* ....

```kotlin
val quickFix = LintFix.create()
        .name("replace with ARouter")
        .replace()
        .all()
        .with(aRouterCode)
        .shortenNames()
        .reformat(true)
        .build()
context.report(ISSUE_AWFUL_NAVIGATION, node, node.nameLocation, msg, quickFix)
```

 `name()` 方法可以给当前 **Lint Fix** 方案命名。`replace().all().with(aRouterCode)` 表示用 _‘aRouterCode‘_  这个变量的字符串内容替换原来整句代码。`shortenNames()` 会自动 import 新代码所需要的包，从而精简代码长度。`reformat()` 则是对代码格式化。这个 **Lint Fix** 对象会作为 `JavaContext.report` 方法的最后一个参数，传递给 IDE。

你也可以指定只替换原来的一部分代码：

```kotlin
val quicFix = LintFix.create()
        .replace()
        //.pattern(".*old(s)+code.*")
        //.range(context.getRangeLocation(node, 0, 1))
        .text("old code")
        .with("new code")
        .build()
```

 可以通过 `text()` 方法指定原来代码中的一段字符串，或者是通过 `pattern()` 方法来匹配。`range()` 方法可以传递一个 **Location** 对象，主要用于修改位置与 **Issue** 报告位置不同的情况，比如要修改一个变量上的注解，**Issue** 的范围通常是变量的位置，而 **Lint Fix** 的范围则是注解的位置。

```kotlin
val fix1 = LintFix.create().build()
val fix2 = LintFix.create().build()
val quickFix = LintFix.create()
        //.composite(fix1_step1, fix1_step2)
        .group(fix1, fix2)
```

 当我们有多个 **Lint Fix** 方案时，可以通过 `group()` 方法或者 `composite()` 方法组合起来。

* 如果每个 **Lint Fix** 是单独的解决建议，那么用 `group()` 方法，在 IDE 中会显示多条修改建议。
* 如果是同一个解决建议，由多个 **Lint Fix** 的步骤共同作用完成，那么用 `composite()` 方法，在 IDE 中只会显示一条修改建议，而且要注意多个 **Lint Fix** 的作用可能会重叠。

{% hint style="info" %}
在 Kotlin 代码文件中，只有 Android Studio 3.1 以上版本才会有 **Lint Fix**
{% endhint %}

