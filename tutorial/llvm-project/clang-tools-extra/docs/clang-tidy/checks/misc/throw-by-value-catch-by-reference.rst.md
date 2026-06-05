# throw-by-value-catch-by-reference.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/misc/throw-by-value-catch-by-reference.rst`
- Title / 标题: `misc-throw-by-value-catch-by-reference`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This document describes the `misc-throw-by-value-catch-by-reference` check. cert-err09-cpp and cert-err61-cpp redirect here as aliases of this check.
- CN: 本文档介绍 `misc-throw-by-value-catch-by-reference` 检查。它会检测并标记特定的代码或 API 使用模式。 原文摘要：`cert-err09-cpp and cert-err61-cpp redirect here as aliases of this check.`。

### Structure / 结构
- EN: The page uses a compact reference layout with 1 secondary heading(s), 0 bullet item(s), 0 code example block(s), and 3 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 1 个次级标题、0 个列表项、0 个代码示例块，以及 3 个交叉引用。

### Notable Details / 重点细节
- EN: The document explicitly ties the check to coding-standard guidance through referenced rule identifiers.
- CN: 文档通过引用的规则标识，把该检查与编码规范指导明确联系起来。

### References / 引用
- EN: Referenced rules: `ERR61-CPP`, `ERR09-CPP` External links: `https://wiki.sei.cmu.edu/confluence/display/cplusplus/ERR61-CPP.+Catch+exceptions+by+lvalue+reference`
- CN: 引用规则：`ERR61-CPP`、`ERR09-CPP` 外部链接：`https://wiki.sei.cmu.edu/confluence/display/cplusplus/ERR61-CPP.+Catch+exceptions+by+lvalue+reference`

## Key Concepts / 关键概念
- `misc-throw-by-value-catch-by-reference`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `ERR61-CPP`
  - EN: Coding-standard rule explicitly connected to the check.
  - CN: 与该检查显式关联的编码规范规则。
- `ERR09-CPP`
  - EN: Coding-standard rule explicitly connected to the check.
  - CN: 与该检查显式关联的编码规范规则。
- `size_t`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `Coding-standard mapping`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: Rule mapping: `ERR61-CPP`, `ERR09-CPP`
  - CN: 规则映射：`ERR61-CPP`、`ERR09-CPP`
- EN: External specification or landing page: `https://wiki.sei.cmu.edu/confluence/display/cplusplus/ERR61-CPP.+Catch+exceptions+by+lvalue+reference`
  - CN: 外部规范或落地页：`https://wiki.sei.cmu.edu/confluence/display/cplusplus/ERR61-CPP.+Catch+exceptions+by+lvalue+reference`
- EN: Related APIs / symbols discussed: `char`, `wchar_t`
  - CN: 涉及的 API / 符号：`char`、`wchar_t`
