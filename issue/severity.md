---
description: Issue Severity
---

# 严重级别

**Issue** 有以下几种严重级别，`Error` 以上的级别会导致编译阻断，在IDE上默认设置会以红色下划线标示。`Warning` 和 `Information` 则会是浅色的弱提示。 `Ignore` 级别的 **Issue** 不会在报告上出现。

```kotlin
/**
 * Fatal: Use sparingly because a warning marked as fatal will be
 * considered critical and will abort Export APK etc in ADT
 */
FATAL("Fatal"),

/**
 * Errors: The issue is known to be a real error that must be addressed.
 */
ERROR("Error"),

/**
 * Warning: Probably a problem.
 */
WARNING("Warning"),

/**
 * Information only: Might not be a problem, but the check has found
 * something interesting to say about the code.
 */
INFORMATIONAL("Information"),

/**
 * Ignore: The user doesn't want to see this issue
 */
IGNORE("Ignore");
```

 

