# virtual-class-destructor.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/cppcoreguidelines/virtual-class-destructor.rst`
- Title / 标题: `cppcoreguidelines-virtual-class-destructor`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `cppcoreguidelines-virtual-class-destructor` check. Finds virtual classes whose destructor is neither public and virtual nor protected and non-virtual.
- CN: 本文档介绍 `cppcoreguidelines-virtual-class-destructor` 检查。它会检测并标记特定的代码或 API 使用模式。 原文摘要：`Finds virtual classes whose destructor is neither public and virtual nor protected and non-virtual.`。

### Structure / 结构
- EN: The page uses a compact reference layout with 1 secondary heading(s), 0 bullet item(s), 2 code example block(s), and 1 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 1 个次级标题、0 个列表项、2 个代码示例块，以及 1 个交叉引用。

### Notable Details / 重点细节
- EN: The page includes code examples that illustrate the diagnostic and any suggested transformation.
- CN: 页面包含代码示例，用于展示诊断触发方式以及建议的代码转换。

### References / 引用
- EN: External links: `http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rc-dtor-virtual`
- CN: 外部链接：`http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rc-dtor-virtual`

## Key Concepts / 关键概念
- `cppcoreguidelines-virtual-class-destructor`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `Check behavior`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: External specification or landing page: `http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rc-dtor-virtual`
  - CN: 外部规范或落地页：`http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rc-dtor-virtual`
