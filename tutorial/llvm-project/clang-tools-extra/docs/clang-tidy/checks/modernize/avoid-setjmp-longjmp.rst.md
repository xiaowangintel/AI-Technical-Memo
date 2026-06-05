# avoid-setjmp-longjmp.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/modernize/avoid-setjmp-longjmp.rst`
- Title / 标题: `modernize-avoid-setjmp-longjmp`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `modernize-avoid-setjmp-longjmp` check. Flags all call expressions involving setjmp() and longjmp() in C++ code.
- CN: 本文档介绍 `modernize-avoid-setjmp-longjmp` 检查。它会检测并标记特定的代码或 API 使用模式。 原文摘要：`Flags all call expressions involving setjmp() and longjmp() in C++ code.`。

### Structure / 结构
- EN: The page uses a compact reference layout with 1 secondary heading(s), 0 bullet item(s), 0 code example block(s), and 2 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 1 个次级标题、0 个列表项、0 个代码示例块，以及 2 个交叉引用。

### Notable Details / 重点细节
- EN: The document explicitly ties the check to coding-standard guidance through referenced rule identifiers.
- CN: 文档通过引用的规则标识，把该检查与编码规范指导明确联系起来。

### References / 引用
- EN: Referenced rules: `ERR52-CPP` External links: `https://www.securecoding.cert.org/confluence/pages/viewpage.action?pageId=88046492`
- CN: 引用规则：`ERR52-CPP` 外部链接：`https://www.securecoding.cert.org/confluence/pages/viewpage.action?pageId=88046492`

## Key Concepts / 关键概念
- `modernize-avoid-setjmp-longjmp`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `ERR52-CPP`
  - EN: Coding-standard rule explicitly connected to the check.
  - CN: 与该检查显式关联的编码规范规则。
- `setjmp()`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `longjmp()`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `Modernization guidance`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: Rule mapping: `ERR52-CPP`
  - CN: 规则映射：`ERR52-CPP`
- EN: External specification or landing page: `https://www.securecoding.cert.org/confluence/pages/viewpage.action?pageId=88046492`
  - CN: 外部规范或落地页：`https://www.securecoding.cert.org/confluence/pages/viewpage.action?pageId=88046492`
- EN: Related APIs / symbols discussed: `setjmp()`, `longjmp()`
  - CN: 涉及的 API / 符号：`setjmp()`、`longjmp()`
