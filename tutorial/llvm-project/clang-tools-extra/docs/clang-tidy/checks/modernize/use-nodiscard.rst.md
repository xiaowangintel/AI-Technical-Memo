# use-nodiscard.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/modernize/use-nodiscard.rst`
- Title / 标题: `modernize-use-nodiscard`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `modernize-use-nodiscard` check. Adds [[nodiscard]] attributes (introduced in C++17) to member functions in order to highlight at compile time which return values should not be ignored.
- CN: 本文档介绍 `modernize-use-nodiscard` 检查。它说明了该检查关注的代码模式与使用约束。 原文摘要：`Adds [[nodiscard]] attributes (introduced in C++17) to member functions in order to highlight at compile time which return values should no…`。

### Structure / 结构
- EN: The page uses a compact reference layout with 3 secondary heading(s), 12 bullet item(s), 4 code example block(s), and 1 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 3 个次级标题、12 个列表项、4 个代码示例块，以及 1 个交叉引用。

### Notable Details / 重点细节
- EN: The page includes code examples that illustrate the diagnostic and any suggested transformation. Representative terms include `no [[nodiscard]], [[noreturn]],`, `non-void return type,`, `non-template return types,`.
- CN: 页面包含代码示例，用于展示诊断触发方式以及建议的代码转换。 代表性术语包括 `no [[nodiscard]], [[noreturn]],`、`non-void return type,`、`non-template return types,`。

### References / 引用
- EN: External links: `https://clang.llvm.org/docs/AttributeReference.html#nodiscard-warn-unused-result`
- CN: 外部链接：`https://clang.llvm.org/docs/AttributeReference.html#nodiscard-warn-unused-result`

## Key Concepts / 关键概念
- `modernize-use-nodiscard`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `[[gcc::warn_unused_result]]`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `[[clang::warn_unused_result]]`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `__attribute__`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `Return-value checking`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: External specification or landing page: `https://clang.llvm.org/docs/AttributeReference.html#nodiscard-warn-unused-result`
  - CN: 外部规范或落地页：`https://clang.llvm.org/docs/AttributeReference.html#nodiscard-warn-unused-result`
- EN: Related APIs / symbols discussed: `[[nodiscard]]`, `[[noreturn]]`, `no [[nodiscard]], [[noreturn]],`, `non-void return type,`
  - CN: 涉及的 API / 符号：`[[nodiscard]]`、`[[noreturn]]`、`no [[nodiscard]], [[noreturn]],`、`non-void return type,`
