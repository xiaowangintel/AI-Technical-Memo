# pass-by-value.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/modernize/pass-by-value.rst`
- Title / 标题: `modernize-pass-by-value`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `modernize-pass-by-value` check. With move semantics added to the language and the standard library updated with move constructors added for many types it is now interesting to take an argument directly by value, instead of by const-reference, and then copy.
- CN: 本文档介绍 `modernize-pass-by-value` 检查。它会检测并标记特定的代码或 API 使用模式。 原文摘要：`With move semantics added to the language and the standard library updated with move constructors added for many types it is now interestin…`。

### Structure / 结构
- EN: The page uses a compact reference layout with 4 secondary heading(s), 9 bullet item(s), 5 code example block(s), and 2 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 4 个次级标题、9 个列表项、5 个代码示例块，以及 2 个交叉引用。

### Notable Details / 重点细节
- EN: The page includes code examples that illustrate the diagnostic and any suggested transformation. Representative terms include `Foo(const std::string &Copied, const std::string &ReadOnly)`, `: Copied(Copied), ReadOnly(ReadOnly)`, `Foo(std::string Copied, const std::string &ReadOnly)`.
- CN: 页面包含代码示例，用于展示诊断触发方式以及建议的代码转换。 代表性术语包括 `Foo(const std::string &Copied, const std::string &ReadOnly)`、`: Copied(Copied), ReadOnly(ReadOnly)`、`Foo(std::string Copied, const std::string &ReadOnly)`。

### References / 引用
- EN: External links: `https://clang.llvm.org/docs/UsersManual.html#microsoft-extensions`, `https://web.archive.org/web/20140205194657/http://cpp-next.com/archive/2009/08/want-speed-pass-by-value/`
- CN: 外部链接：`https://clang.llvm.org/docs/UsersManual.html#microsoft-extensions`、`https://web.archive.org/web/20140205194657/http://cpp-next.com/archive/2009/08/want-speed-pass-by-value/`

## Key Concepts / 关键概念
- `modernize-pass-by-value`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `std::move()`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `-fdelayed-template-parsing`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `-fno-delayed-template-parsing`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `Move semantics`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: External specification or landing page: `https://clang.llvm.org/docs/UsersManual.html#microsoft-extensions`, `https://web.archive.org/web/20140205194657/http://cpp-next.com/archive/2009/08/want-speed-pass-by-value/`
  - CN: 外部规范或落地页：`https://clang.llvm.org/docs/UsersManual.html#microsoft-extensions`、`https://web.archive.org/web/20140205194657/http://cpp-next.com/archive/2009/08/want-speed-pass-by-value/`
- EN: Related APIs / symbols discussed: `std::move()`, `-fdelayed-template-parsing`, `Foo(const std::string &Copied, const std::string &ReadOnly)`, `: Copied(Copied), ReadOnly(ReadOnly)`
  - CN: 涉及的 API / 符号：`std::move()`、`-fdelayed-template-parsing`、`Foo(const std::string &Copied, const std::string &ReadOnly)`、`: Copied(Copied), ReadOnly(ReadOnly)`
