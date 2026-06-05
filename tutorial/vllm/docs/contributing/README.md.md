# README.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/contributing/README.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This is the central contributor handbook for vLLM, covering development environment setup, linting, documentation, testing, issue reporting, PR expectations, kernel work, and the review process.
- **CN:** 这是 vLLM 的核心贡献者手册，覆盖开发环境搭建、代码检查、文档、测试、问题反馈、PR 要求、内核开发以及评审流程。
- **EN:** A notable modern section is the policy for AI-assisted contributions: humans must stay responsible for review and validation, disclose AI use in PRs, and add attribution trailers in commits.
- **CN:** 其中一个很突出的现代化部分是 AI 辅助贡献政策：人类提交者必须继续负责审查与验证，在 PR 中披露 AI 使用情况，并在提交中添加归属 trailer。

## Key Concepts / 关键概念
- **EN:** Development setup — The guide explains how to clone the repo, prepare Python environments, and choose different install paths for Python-only or CUDA/C++ development.
- **CN:** 开发环境搭建 — 文档说明了如何克隆仓库、准备 Python 环境，以及针对纯 Python 或 CUDA/C++ 开发选择不同安装路径。
- **EN:** Tooling workflow — It standardizes linting with `pre-commit`, docs preview with MkDocs, and testing with `pytest`.
- **CN:** 工具工作流 — 文档统一了使用 `pre-commit` 做检查、用 MkDocs 预览文档、以及用 `pytest` 运行测试的流程。
- **EN:** Issue and security path — Normal bugs go through GitHub issues, while security problems must follow the private security process.
- **CN:** 问题与安全反馈路径 — 普通问题通过 GitHub issue 提交，安全问题则必须走私密安全流程。
- **EN:** DCO and AI disclosure — Contributors need signed-off commits, and AI-assisted work requires disclosure plus commit attribution.
- **CN:** DCO 与 AI 披露 — 贡献者需要带签署信息的提交，而 AI 辅助工作还必须披露并附加归属信息。
- **EN:** PR classification — PR titles must use approved prefixes such as `[Bugfix]`, `[Model]`, or `[Kernel]`.
- **CN:** PR 分类 — PR 标题必须使用约定前缀，例如 `[Bugfix]`、`[Model]` 或 `[Kernel]`。
- **EN:** Review expectations — The document sets expectations for reviewer assignment, status updates, CI scope, and contributor responsiveness.
- **CN:** 评审预期 — 文档说明了 reviewer 分配、状态更新、CI 覆盖范围以及贡献者响应时效的预期。
