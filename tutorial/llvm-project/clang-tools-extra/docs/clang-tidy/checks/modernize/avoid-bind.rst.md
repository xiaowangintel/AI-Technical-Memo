# avoid-bind.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/modernize/avoid-bind.rst`
- Title / 标题: `modernize-avoid-bind`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `modernize-avoid-bind` check. The check finds uses of std::bind and boost::bind and replaces them with lambdas.
- CN: 本文档介绍 `modernize-avoid-bind` 检查。它会查找目标模式，并在安全时给出替换或重命名建议。 原文摘要：`The check finds uses of std::bind and boost::bind and replaces them with lambdas.`。

### Structure / 结构
- EN: The page uses a compact reference layout with 1 secondary heading(s), 0 bullet item(s), 6 code example block(s), and 0 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 1 个次级标题、0 个列表项、6 个代码示例块，以及 0 个交叉引用。

### Notable Details / 重点细节
- EN: The page includes code examples that illustrate the diagnostic and any suggested transformation. Representative terms include `std::bind`, `boost::bind`, `std::ref`.
- CN: 页面包含代码示例，用于展示诊断触发方式以及建议的代码转换。 代表性术语包括 `std::bind`、`boost::bind`、`std::ref`。

### References / 引用
- EN: This page is largely self-contained inside the clang-tidy documentation tree.
- CN: 该页面基本在 clang-tidy 文档树内部自洽。

## Key Concepts / 关键概念
- `modernize-avoid-bind`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `std::ref`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `std::bind`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `boost::ref`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `Modernization guidance`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: Related APIs / symbols discussed: `std::bind`, `boost::bind`
  - CN: 涉及的 API / 符号：`std::bind`、`boost::bind`
