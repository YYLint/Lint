---
description: write out own detector
---

# 自定义规则

通过自定义规则我们可以根据项目的实际情况来规范团队的代码输出。不同版本的 **Lint** api变化非常大，我们编写规则时使用的版本是：

```groovy
compileOnly "com.android.tools.lint:lint-api:26.1.2"
compileOnly "com.android.tools.lint:lint-checks:26.1.2"
testImplementation "com.android.tools.lint:lint:26.1.2"
testImplementation "com.android.tools.lint:lint-tests:26.1.2"
```

这个版本应该也是官方团队从 `Java`到 `Kotlin` 过渡的一个版本，从底层的实现可以看到既有 `Java` 的代码也有 `Kotlin` 的代码，而且 `Kotlin` 部分的语法树对开发者的友好性是远不如 `Java`。这些你都可以在实际开发的过程中体会得到。

