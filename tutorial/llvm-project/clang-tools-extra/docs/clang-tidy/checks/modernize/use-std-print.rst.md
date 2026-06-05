# use-std-print.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/modernize/use-std-print.rst`
- Title / 标题: `modernize-use-std-print`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `modernize-use-std-print` check. Converts calls to printf, fprintf, absl::PrintF and absl::FPrintf to equivalent calls to C++23's std::print or std::println as appropriate, modifying the format string appropriately.
- CN: 本文档介绍 `modernize-use-std-print` 检查。它会查找目标模式，并在安全时给出替换或重命名建议。 原文摘要：`Converts calls to printf, fprintf, absl::PrintF and absl::FPrintf to equivalent calls to C++23's std::print or std::println as appropriate,…`。

### Structure / 结构
- EN: The page uses a compact reference layout with 1 secondary heading(s), 6 bullet item(s), 4 code example block(s), and 1 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 1 个次级标题、6 个列表项、4 个代码示例块，以及 1 个交叉引用。

### Notable Details / 重点细节
- EN: The page includes code examples that illustrate the diagnostic and any suggested transformation. Representative terms include `At the point that the check runs, the AST contains a single`, `The %' flag for thousands separators.`, `The glibc extension %m.`.
- CN: 页面包含代码示例，用于展示诊断触发方式以及建议的代码转换。 代表性术语包括 `At the point that the check runs, the AST contains a single`、`The %' flag for thousands separators.`、`The glibc extension %m.`。

### References / 引用
- EN: Internal documentation refs: `../readability/redundant-string-cstr`
- CN: 内部文档引用：`../readability/redundant-string-cstr`

## Key Concepts / 关键概念
- `modernize-use-std-print`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `std::string::c_str()`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `absl::PrintF`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `absl::FPrintf`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `Modernization guidance`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: Internal references: `../readability/redundant-string-cstr`
  - CN: 内部引用：`../readability/redundant-string-cstr`
- EN: Related APIs / symbols discussed: `printf`, `fprintf`, `At the point that the check runs, the AST contains a single`, `The %' flag for thousands separators.`
  - CN: 涉及的 API / 符号：`printf`、`fprintf`、`At the point that the check runs, the AST contains a single`、`The %' flag for thousands separators.`
