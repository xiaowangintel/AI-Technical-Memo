# avoid-capturing-lambda-coroutines.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/cppcoreguidelines/avoid-capturing-lambda-coroutines.rst`
- Title / 标题: `cppcoreguidelines-avoid-capturing-lambda-coroutines`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `cppcoreguidelines-avoid-capturing-lambda-coroutines` check. Flags C++20 coroutine lambdas with non-empty capture lists that may cause use-after-free errors and suggests avoiding captures or ensuring the lambda closure object has a guaranteed lifetime.
- CN: 本文档介绍 `cppcoreguidelines-avoid-capturing-lambda-coroutines` 检查。它会检测并标记特定的代码或 API 使用模式。 原文摘要：`Flags C++20 coroutine lambdas with non-empty capture lists that may cause use-after-free errors and suggests avoiding captures or ensuring…`。

### Structure / 结构
- EN: The page uses a compact reference layout with 1 secondary heading(s), 0 bullet item(s), 2 code example block(s), and 1 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 1 个次级标题、0 个列表项、2 个代码示例块，以及 1 个交叉引用。

### Notable Details / 重点细节
- EN: The page includes code examples that illustrate the diagnostic and any suggested transformation. Representative terms include `sharedFoo`, `lambda()`, `this auto`.
- CN: 页面包含代码示例，用于展示诊断触发方式以及建议的代码转换。 代表性术语包括 `sharedFoo`、`lambda()`、`this auto`。

### References / 引用
- EN: External links: `https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rcoro-capture`
- CN: 外部链接：`https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rcoro-capture`

## Key Concepts / 关键概念
- `cppcoreguidelines-avoid-capturing-lambda-coroutines`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `lambda()`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `sharedFoo`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `true`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `Check behavior`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: External specification or landing page: `https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rcoro-capture`
  - CN: 外部规范或落地页：`https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rcoro-capture`
- EN: Related APIs / symbols discussed: `sharedFoo`, `lambda()`
  - CN: 涉及的 API / 符号：`sharedFoo`、`lambda()`
