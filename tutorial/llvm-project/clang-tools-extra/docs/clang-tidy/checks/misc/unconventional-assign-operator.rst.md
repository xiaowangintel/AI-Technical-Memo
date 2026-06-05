# unconventional-assign-operator.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/misc/unconventional-assign-operator.rst`
- Title / 标题: `misc-unconventional-assign-operator`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `misc-unconventional-assign-operator` check. Finds declarations of assign operators with the wrong return and/or argument types and definitions with good return type but wrong return statements.
- CN: 本文档介绍 `misc-unconventional-assign-operator` 检查。它会检测并标记特定的代码或 API 使用模式。 原文摘要：`Finds declarations of assign operators with the wrong return and/or argument types and definitions with good return type but wrong return s…`。

### Structure / 结构
- EN: The page uses a compact reference layout with 0 secondary heading(s), 3 bullet item(s), 0 code example block(s), and 0 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 0 个次级标题、3 个列表项、0 个代码示例块，以及 0 个交叉引用。

### Notable Details / 重点细节
- EN: A notable part of the page is its explicit enumeration of affected APIs, patterns, or subcases. Examples include `The return type must be Class&.`, `Private and deleted operators are ignored.`, `The operator must always return *this.`.
- CN: 页面的一个显著特点是明确枚举了受影响的 API、模式或子场景。 示例包括 `The return type must be Class&.`、`Private and deleted operators are ignored.`、`The operator must always return *this.`。

### References / 引用
- EN: This page is largely self-contained inside the clang-tidy documentation tree.
- CN: 该页面基本在 clang-tidy 文档树内部自洽。

## Key Concepts / 关键概念
- `misc-unconventional-assign-operator`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `Class&`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `int`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `*this`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `Check behavior`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: Related APIs / symbols discussed: `return`, `Class&`, `The return type must be Class&.`, `Private and deleted operators are ignored.`
  - CN: 涉及的 API / 符号：`return`、`Class&`、`The return type must be Class&.`、`Private and deleted operators are ignored.`
