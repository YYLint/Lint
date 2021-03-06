---
description: Issue Category
---

# 类别

**Lint** 规则提供了以下几种 `Category`

* **Correctness**
* **Usability**
* **Security **
* **Performance**
* **Accessibility**
* **Internationalization**
* **Lint** （针对规则的规则）
* 上面几种的子类或是自定义 `Category` （**Icons**、**Typography ...**）

我们可以根据 **Issue** 的性质来作简单的分类。在 Lint 官方实现了海量的规则，在这些规则当中：

* `"SimpleDataFormat"`,`"ValidFragment"`,`"MissingCallSuper"`...类别是 **Correctness**
* `"HandlerLeak"` `"FieldGetter"`... 类别是 **Performance**
* `"JavascriptInterface"`, `"SecureRandom"` ...类别是 **Security**
* ...

当我们需要为自己定义的规则进行分类时，可结合官方的例子，根据可用性、易用性、正确性几个角度来考虑。

