# forbidden-subclassing.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/objc/forbidden-subclassing.rst`
- Title / 标题: `objc-forbidden-subclassing`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `objc-forbidden-subclassing` check. Finds Objective-C classes which are subclasses of classes which are not designed to be subclassed.
- CN: 本文档介绍 `objc-forbidden-subclassing` 检查。它会检测并标记特定的代码或 API 使用模式。 原文摘要：`Finds Objective-C classes which are subclasses of classes which are not designed to be subclassed.`。

### Structure / 结构
- EN: The page uses a compact reference layout with 1 secondary heading(s), 0 bullet item(s), 0 code example block(s), and 1 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 1 个次级标题、0 个列表项、0 个代码示例块，以及 1 个交叉引用。

### Notable Details / 重点细节
- EN: The page is concise and relies on linked material for additional background.
- CN: 该页面较为简洁，更多背景信息依赖其链接到的材料。

### References / 引用
- EN: External links: `https://clang.llvm.org/docs/AttributeReference.html#objc-subclassing-restricted`
- CN: 外部链接：`https://clang.llvm.org/docs/AttributeReference.html#objc-subclassing-restricted`

## Key Concepts / 关键概念
- `objc-forbidden-subclassing`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `@interface`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `before your`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `Check behavior`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: External specification or landing page: `https://clang.llvm.org/docs/AttributeReference.html#objc-subclassing-restricted`
  - CN: 外部规范或落地页：`https://clang.llvm.org/docs/AttributeReference.html#objc-subclassing-restricted`
- EN: Related APIs / symbols discussed: `__attribute__((objc_subclassing_restricted))`, `@interface`
  - CN: 涉及的 API / 符号：`__attribute__((objc_subclassing_restricted))`、`@interface`
