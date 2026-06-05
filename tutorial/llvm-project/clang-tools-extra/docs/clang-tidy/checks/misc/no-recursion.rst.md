# no-recursion.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/misc/no-recursion.rst`
- Title / 标题: `misc-no-recursion`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `misc-no-recursion` check. Finds strongly connected functions (by analyzing the call graph for SCC's (Strongly Connected Components) that are loops), diagnoses each function in the cycle, and displays one example of a possible call graph loop (recursion).
- CN: 本文档介绍 `misc-no-recursion` 检查。它会检测并标记特定的代码或 API 使用模式。 原文摘要：`Finds strongly connected functions (by analyzing the call graph for SCC's (Strongly Connected Components) that are loops), diagnoses each f…`。

### Structure / 结构
- EN: The page uses a compact reference layout with 1 secondary heading(s), 1 bullet item(s), 0 code example block(s), and 3 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 1 个次级标题、1 个列表项、0 个代码示例块，以及 3 个交叉引用。

### Notable Details / 重点细节
- EN: A notable part of the page is its explicit enumeration of affected APIs, patterns, or subcases. Examples include `The check does not handle C++ destructors`.
- CN: 页面的一个显著特点是明确枚举了受影响的 API、模式或子场景。 示例包括 `The check does not handle C++ destructors`。

### References / 引用
- EN: Referenced rules: `DCL56-CPP` External links: `https://wiki.sei.cmu.edu/confluence/display/cplusplus/DCL56-CPP.+Avoid+cycles+during+initialization+of+static+objects`, `https://www.khronos.org/registry/OpenCL/specs/opencl-1.2.pdf`
- CN: 引用规则：`DCL56-CPP` 外部链接：`https://wiki.sei.cmu.edu/confluence/display/cplusplus/DCL56-CPP.+Avoid+cycles+during+initialization+of+static+objects`、`https://www.khronos.org/registry/OpenCL/specs/opencl-1.2.pdf`

## Key Concepts / 关键概念
- `misc-no-recursion`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `DCL56-CPP`
  - EN: Coding-standard rule explicitly connected to the check.
  - CN: 与该检查显式关联的编码规范规则。
- `Coding-standard mapping`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: Rule mapping: `DCL56-CPP`
  - CN: 规则映射：`DCL56-CPP`
- EN: External specification or landing page: `https://wiki.sei.cmu.edu/confluence/display/cplusplus/DCL56-CPP.+Avoid+cycles+during+initialization+of+static+objects`, `https://www.khronos.org/registry/OpenCL/specs/opencl-1.2.pdf`
  - CN: 外部规范或落地页：`https://wiki.sei.cmu.edu/confluence/display/cplusplus/DCL56-CPP.+Avoid+cycles+during+initialization+of+static+objects`、`https://www.khronos.org/registry/OpenCL/specs/opencl-1.2.pdf`
- EN: Related APIs / symbols discussed: `The check does not handle C++ destructors`
  - CN: 涉及的 API / 符号：`The check does not handle C++ destructors`
