# new-delete-overloads.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/misc/new-delete-overloads.rst`
- Title / 标题: `misc-new-delete-overloads`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `misc-new-delete-overloads` check. The check flags overloaded operator new() and operator delete() functions that do not have a corresponding free store function defined within the same scope.
- CN: 本文档介绍 `misc-new-delete-overloads` 检查。它会检测并标记特定的代码或 API 使用模式。 原文摘要：`The check flags overloaded operator new() and operator delete() functions that do not have a corresponding free store function defined with…`。

### Structure / 结构
- EN: The page uses a compact reference layout with 0 secondary heading(s), 0 bullet item(s), 0 code example block(s), and 2 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 0 个次级标题、0 个列表项、0 个代码示例块，以及 2 个交叉引用。

### Notable Details / 重点细节
- EN: The page also acts as the canonical landing page for alias names such as `cert-dcl54-cpp`.
- CN: 该页面也承担别名落点页的作用，例如 `cert-dcl54-cpp`。

### References / 引用
- EN: Referenced rules: `DCL54-CPP` External links: `https://www.securecoding.cert.org/confluence/display/cplusplus/DCL54-CPP.+Overload+allocation+and+deallocation+functions+as+a+pair+in+the+same+scope`
- CN: 引用规则：`DCL54-CPP` 外部链接：`https://www.securecoding.cert.org/confluence/display/cplusplus/DCL54-CPP.+Overload+allocation+and+deallocation+functions+as+a+pair+in+the+same+scope`

## Key Concepts / 关键概念
- `misc-new-delete-overloads`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `DCL54-CPP`
  - EN: Coding-standard rule explicitly connected to the check.
  - CN: 与该检查显式关联的编码规范规则。
- `new()`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `delete()`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `Coding-standard mapping`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: Rule mapping: `DCL54-CPP`
  - CN: 规则映射：`DCL54-CPP`
- EN: External specification or landing page: `https://www.securecoding.cert.org/confluence/display/cplusplus/DCL54-CPP.+Overload+allocation+and+deallocation+functions+as+a+pair+in+the+same+scope`
  - CN: 外部规范或落地页：`https://www.securecoding.cert.org/confluence/display/cplusplus/DCL54-CPP.+Overload+allocation+and+deallocation+functions+as+a+pair+in+the+same+scope`
- EN: Related APIs / symbols discussed: `new()`, `delete()`
  - CN: 涉及的 API / 符号：`new()`、`delete()`
