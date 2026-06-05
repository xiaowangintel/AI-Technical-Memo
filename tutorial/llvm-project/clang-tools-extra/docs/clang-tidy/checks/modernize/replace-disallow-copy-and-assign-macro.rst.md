# replace-disallow-copy-and-assign-macro.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/modernize/replace-disallow-copy-and-assign-macro.rst`
- Title / 标题: `modernize-replace-disallow-copy-and-assign-macro`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `modernize-replace-disallow-copy-and-assign-macro` check. Finds macro expansions of DISALLOW_COPY_AND_ASSIGN(Type) and replaces them with a deleted copy constructor and a deleted assignment operator.
- CN: 本文档介绍 `modernize-replace-disallow-copy-and-assign-macro` 检查。它会查找目标模式，并在安全时给出替换或重命名建议。 原文摘要：`Finds macro expansions of DISALLOW_COPY_AND_ASSIGN(Type) and replaces them with a deleted copy constructor and a deleted assignment operato…`。

### Structure / 结构
- EN: The page uses a compact reference layout with 2 secondary heading(s), 0 bullet item(s), 2 code example block(s), and 2 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 2 个次级标题、0 个列表项、2 个代码示例块，以及 2 个交叉引用。

### Notable Details / 重点细节
- EN: The page includes code examples that illustrate the diagnostic and any suggested transformation. Representative terms include `DISALLOW_COPY_AND_ASSIGN(Type)`, `delete`, `private`.
- CN: 页面包含代码示例，用于展示诊断触发方式以及建议的代码转换。 代表性术语包括 `DISALLOW_COPY_AND_ASSIGN(Type)`、`delete`、`private`。

### References / 引用
- EN: Internal documentation refs: `../modernize/use-equals-delete` External links: `https://en.cppreference.com/w/cpp/language/function#Deleted_functions`
- CN: 内部文档引用：`../modernize/use-equals-delete` 外部链接：`https://en.cppreference.com/w/cpp/language/function#Deleted_functions`

## Key Concepts / 关键概念
- `modernize-replace-disallow-copy-and-assign-macro`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `DISALLOW_COPY_AND_ASSIGN(Type)`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `DISALLOW_COPY_AND_ASSIGN`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `delete`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `Modernization guidance`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: Internal references: `../modernize/use-equals-delete`
  - CN: 内部引用：`../modernize/use-equals-delete`
- EN: External specification or landing page: `https://en.cppreference.com/w/cpp/language/function#Deleted_functions`
  - CN: 外部规范或落地页：`https://en.cppreference.com/w/cpp/language/function#Deleted_functions`
- EN: Related APIs / symbols discussed: `DISALLOW_COPY_AND_ASSIGN(Type)`, `delete`
  - CN: 涉及的 API / 符号：`DISALLOW_COPY_AND_ASSIGN(Type)`、`delete`
