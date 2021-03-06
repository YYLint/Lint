---
description: Issue Location
---

# 报告位置

通过 `JavaContext.report` 方法来报告一个 **Issue** ，我们需要提供 **Issue** 的位置。大部分情况，我们都可以通过 `JavaContext.getNameLocation` 或者 `JavaContext.getLocation` 方法来获取语法树节点的位置。

**Location** 中有几个重要的属性，其中 `start` 和 `end` 之间的范围就是该 **Issue** 在 IDE 中提示的代码位置：

```kotlin
/**
 * Returns the file containing the warning. Note that the file *itself* may
 * not yet contain the error. When editing a file in the IDE for example,
 * the tool could generate warnings in the background even before the
 * document is saved. However, the file is used as a identifying token for
 * the document being edited, and the IDE integration can map this back to
 * error locations in the editor source code.
 *
 * @return the file handle for the location
 */
val file: File
/**
 * The start position of the range
 *
 * @return the start position of the range, or null
 */
val start: Position?
/**
 * The end position of the range
 *
 * @return the start position of the range, may be null for an empty range
 */
val end: Position?
/**
 * Returns a secondary location associated with this location (if
 * applicable), or null.
 */
open var secondary: Location? = null 
```

` secondary` 可以帮我们连接两个 **Location** ，比如我们检测资源文件的使用时，既要定位出代码中的行数，也要定位出资源文件中的位置：

![Linked Location](../.gitbook/assets/image.png)

```java
Location secondary = context.getLocation(previous);
secondary.setMessage("另一个位置");
location.setSecondary(secondary);
```

除了从 `UElement` 或者 `PsiElement` 中直接获取 **Location** ，当然我们也可以自己去计算和定义具体的行数和列数：

```java
Pair<Integer, Integer> offsets = getOffsets(node, context);
int fromLine = node.getLineNumber() - 1;
int fromColumn = node.getColumnNumber() - 1;
int toLine = node.getLastLineNumber() - 1;
int toColumn = node.getLastColumnNumber() - 1;
return Location.create(context.file,
        new DefaultPosition(fromLine, fromColumn, offsets.getFirst()),
        new DefaultPosition(toLine, toColumn, offsets.getSecond()));
```

 

