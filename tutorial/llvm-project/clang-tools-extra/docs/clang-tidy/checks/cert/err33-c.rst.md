# err33-c.rst — Documentation Analysis / 文档分析

## Source / 来源
- File / 文件: `clang-tools-extra/docs/clang-tidy/checks/cert/err33-c.rst`
- Title / 标题: `cert-err33-c`
- Format / 格式: `.rst`

## Content Analysis / 内容分析
### Overview / 概览
- EN: This page is an alias entry for `../bugprone/unused-return-value`. Warns on unused function return values.
- CN: 该页面是 `../bugprone/unused-return-value` 的别名入口。 它的核心行为可概括为：`Warns on unused function return values.`。

### Structure / 结构
- EN: The page uses a compact reference layout with 0 secondary heading(s), 177 bullet item(s), 0 code example block(s), and 3 cross-reference(s).
- CN: 页面采用紧凑的参考手册结构，包含 0 个次级标题、177 个列表项、0 个代码示例块，以及 3 个交叉引用。

### Notable Details / 重点细节
- EN: The file mainly preserves a stable alias name and forwards readers to `../bugprone/unused-return-value` instead of duplicating a full rule description. It also keeps the rule mapping to `ERR33-C`.
- CN: 该文件主要用于保留稳定的别名名称，并把读者转发到 `../bugprone/unused-return-value`，而不是重复完整的规则说明。 同时它也保留了到 `ERR33-C` 的规则映射。

### References / 引用
- EN: Internal documentation refs: `../bugprone/unused-return-value` Referenced rules: `ERR33-C` External links: `https://wiki.sei.cmu.edu/confluence/display/c/ERR33-C.+Detect+and+handle+standard+library+errors`
- CN: 内部文档引用：`../bugprone/unused-return-value` 引用规则：`ERR33-C` 外部链接：`https://wiki.sei.cmu.edu/confluence/display/c/ERR33-C.+Detect+and+handle+standard+library+errors`

## Key Concepts / 关键概念
- `cert-err33-c`
  - EN: Check or document identifier used by the page.
  - CN: 页面使用的检查或文档标识。
- `../bugprone/unused-return-value`
  - EN: Canonical documentation target referenced by this alias page.
  - CN: 该别名页指向的规范文档目标。
- `ERR33-C`
  - EN: Coding-standard rule explicitly connected to the check.
  - CN: 与该检查显式关联的编码规范规则。
- `gets_s()`
  - EN: Representative API, symbol, or term mentioned by the document.
  - CN: 文档提到的代表性 API、符号或术语。
- `Alias mapping`
  - EN: Main technical theme inferred from the summary.
  - CN: 根据摘要归纳出的主要技术主题。

## Dependencies / 依赖关系
- EN: Documentation hierarchy: this page lives inside the clang-tidy check reference tree.
  - CN: 文档层级：该页面位于 clang-tidy 检查参考文档树中。
- EN: Canonical target: `../bugprone/unused-return-value` supplies the authoritative rule description.
  - CN: 规范目标：`../bugprone/unused-return-value` 提供权威的规则说明。
- EN: Rule mapping: `ERR33-C`
  - CN: 规则映射：`ERR33-C`
- EN: External specification or landing page: `https://wiki.sei.cmu.edu/confluence/display/c/ERR33-C.+Detect+and+handle+standard+library+errors`
  - CN: 外部规范或落地页：`https://wiki.sei.cmu.edu/confluence/display/c/ERR33-C.+Detect+and+handle+standard+library+errors`
- EN: Related APIs / symbols discussed: `void`, `NULL`, `aligned_alloc()`, `asctime_s()`
  - CN: 涉及的 API / 符号：`void`、`NULL`、`aligned_alloc()`、`asctime_s()`
