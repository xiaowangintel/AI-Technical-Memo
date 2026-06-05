# avoid-spinlock.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/darwin/avoid-spinlock.rst`
- Title / 标题: `darwin-avoid-spinlock`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `darwin-avoid-spinlock` check. Finds usages of OSSpinlock, which is deprecated due to potential livelock problems.
- CN: 本文档介绍 `darwin-avoid-spinlock` 检查。它会检测并标记特定的代码或 API 使用模式。 原文摘要：`Finds usages of OSSpinlock, which is deprecated due to potential livelock problems.`。

### Structure / 结构
- EN: The page uses a compact reference layout with 0 secondary heading(s), 3 bullet item(s), 0 code example block(s), and 1 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 0 个次级标题、3 个列表项、0 个代码示例块，以及 1 个交叉引用。

### Notable Details / 重点细节
- EN: A notable part of the page is its explicit enumeration of affected APIs, patterns, or subcases. Examples include `OSSpinlockLock`, `OSSpinlockTry`, `OSSpinlockUnlock`.
- CN: 页面的一个显著特点是明确枚举了受影响的 API、模式或子场景。 示例包括 `OSSpinlockLock`、`OSSpinlockTry`、`OSSpinlockUnlock`。

### References / 引用
- EN: External links: `https://blog.postmates.com/why-spinlocks-are-bad-on-ios-b69fc5221058`
- CN: 外部链接：`https://blog.postmates.com/why-spinlocks-are-bad-on-ios-b69fc5221058`

## Key Concepts / 关键概念
- `darwin-avoid-spinlock`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `OSSpinlock`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `OSSpinlockTry`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `OSSpinlockLock`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `Deprecated API migration`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: External specification or landing page: `https://blog.postmates.com/why-spinlocks-are-bad-on-ios-b69fc5221058`
  - CN: 外部规范或落地页：`https://blog.postmates.com/why-spinlocks-are-bad-on-ios-b69fc5221058`
- EN: Related APIs / symbols discussed: `OSSpinlock`, `OSSpinlockLock`, `OSSpinlockTry`
  - CN: 涉及的 API / 符号：`OSSpinlock`、`OSSpinlockLock`、`OSSpinlockTry`
