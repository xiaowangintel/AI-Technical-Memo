# missing-std-forward.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/cppcoreguidelines/missing-std-forward.rst`
- Title / 标题: `cppcoreguidelines-missing-std-forward`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `cppcoreguidelines-missing-std-forward` check. Warns when a forwarding reference parameter is not forwarded inside the function body.
- CN: 本文档介绍 `cppcoreguidelines-missing-std-forward` 检查。它会对目标模式发出诊断警告。 原文摘要：`Warns when a forwarding reference parameter is not forwarded inside the function body.`。

### Structure / 结构
- EN: The page uses a compact reference layout with 2 secondary heading(s), 0 bullet item(s), 1 code example block(s), and 1 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 2 个次级标题、0 个列表项、1 个代码示例块，以及 1 个交叉引用。

### Notable Details / 重点细节
- EN: The page includes code examples that illustrate the diagnostic and any suggested transformation. Representative terms include `this Self&&`, `::std::forward`.
- CN: 页面包含代码示例，用于展示诊断触发方式以及建议的代码转换。 代表性术语包括 `this Self&&`、`::std::forward`。

### References / 引用
- EN: External links: `http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rf-forward`
- CN: 外部链接：`http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rf-forward`

## Key Concepts / 关键概念
- `cppcoreguidelines-missing-std-forward`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `::std::forward`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `this Self&&`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `Check behavior`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: External specification or landing page: `http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rf-forward`
  - CN: 外部规范或落地页：`http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rf-forward`
- EN: Related APIs / symbols discussed: `this Self&&`, `::std::forward`
  - CN: 涉及的 API / 符号：`this Self&&`、`::std::forward`
