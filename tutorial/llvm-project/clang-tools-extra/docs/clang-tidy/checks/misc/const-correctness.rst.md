# const-correctness.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/misc/const-correctness.rst`
- Title / 标题: `misc-const-correctness`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `misc-const-correctness` check. Finds local variables and function parameters which could be declared as const but are not.
- CN: 本文档介绍 `misc-const-correctness` 检查。它会检测并标记特定的代码或 API 使用模式。 原文摘要：`Finds local variables and function parameters which could be declared as const but are not.`。

### Structure / 结构
- EN: The page uses a compact reference layout with 2 secondary heading(s), 0 bullet item(s), 11 code example block(s), and 3 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 2 个次级标题、0 个列表项、11 个代码示例块，以及 3 个交叉引用。

### Notable Details / 重点细节
- EN: The page includes code examples that illustrate the diagnostic and any suggested transformation. Representative terms include `const`, `int i = 42;`, `int &ref = i;`.
- CN: 页面包含代码示例，用于展示诊断触发方式以及建议的代码转换。 代表性术语包括 `const`、`int i = 42;`、`int &ref = i;`。

### References / 引用
- EN: Internal documentation refs: `../readability/isolate-declaration`, `../cppcoreguidelines/avoid-non-const-global-variables` External links: `https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#es25-declare-an-object-const-or-constexpr-unless-you-want-to-modify-its-value-later-on`
- CN: 内部文档引用：`../readability/isolate-declaration`、`../cppcoreguidelines/avoid-non-const-global-variables` 外部链接：`https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#es25-declare-an-object-const-or-constexpr-unless-you-want-to-modify-its-value-later-on`

## Key Concepts / 关键概念
- `misc-const-correctness`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `namespace::Type`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `[Rr]ef(erence)?$`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `::`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `Check behavior`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: Internal references: `../readability/isolate-declaration`, `../cppcoreguidelines/avoid-non-const-global-variables`
  - CN: 内部引用：`../readability/isolate-declaration`、`../cppcoreguidelines/avoid-non-const-global-variables`
- EN: External specification or landing page: `https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#es25-declare-an-object-const-or-constexpr-unless-you-want-to-modify-its-value-later-on`
  - CN: 外部规范或落地页：`https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#es25-declare-an-object-const-or-constexpr-unless-you-want-to-modify-its-value-later-on`
- EN: Related APIs / symbols discussed: `const`, `int i = 42;`
  - CN: 涉及的 API / 符号：`const`、`int i = 42;`
