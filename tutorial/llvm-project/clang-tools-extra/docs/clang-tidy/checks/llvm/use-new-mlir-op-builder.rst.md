# use-new-mlir-op-builder.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/llvm/use-new-mlir-op-builder.rst`
- Title / 标题: `llvm-mlir-op-builder`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `llvm-mlir-op-builder` check. Checks for uses of MLIR's old/to be deprecated OpBuilder::create form and suggests using T::create instead.
- CN: 本文档介绍 `llvm-mlir-op-builder` 检查。它会检测并标记特定的代码或 API 使用模式。 原文摘要：`Checks for uses of MLIR's old/to be deprecated OpBuilder::create form and suggests using T::create instead.`。

### Structure / 结构
- EN: The page uses a compact reference layout with 1 secondary heading(s), 0 bullet item(s), 2 code example block(s), and 0 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 1 个次级标题、0 个列表项、2 个代码示例块，以及 0 个交叉引用。

### Notable Details / 重点细节
- EN: The page includes code examples that illustrate the diagnostic and any suggested transformation. Representative terms include `OpBuilder::create<T>`, `T::create`.
- CN: 页面包含代码示例，用于展示诊断触发方式以及建议的代码转换。 代表性术语包括 `OpBuilder::create<T>`、`T::create`。

### References / 引用
- EN: This page is largely self-contained inside the clang-tidy documentation tree.
- CN: 该页面基本在 clang-tidy 文档树内部自洽。

## Key Concepts / 关键概念
- `llvm-mlir-op-builder`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `T::create`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `OpBuilder::create<T>`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `Deprecated API migration`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: Related APIs / symbols discussed: `OpBuilder::create<T>`, `T::create`
  - CN: 涉及的 API / 符号：`OpBuilder::create<T>`、`T::create`
