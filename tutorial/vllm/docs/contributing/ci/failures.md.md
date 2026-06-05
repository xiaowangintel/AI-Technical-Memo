# failures.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/contributing/ci/failures.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** Defines the expected triage flow for CI failures on pull requests: first check the shared dashboard, then decide whether the failure is already known or needs a new report.
- **CN:** 本文定义了 PR 上出现 CI 失败时的标准排查流程：先看共享看板，再判断该失败是否已知，还是需要新建报告。
- **EN:** It standardizes issue filing, log collection, reproduction, bisection, and follow-up PR labeling so flaky or unrelated failures are tracked systematically.
- **CN:** 文档把 issue 提交、日志整理、失败复现、问题二分以及后续 PR 标记都规范化，使 flaky 或无关失败也能被系统追踪。

## Key Concepts / 关键概念
- **EN:** CI failures dashboard — The dashboard is the first stop for deciding whether a failure is already known.
- **CN:** CI 失败看板 — 这是判断某个失败是否已知问题的第一入口。
- **EN:** Issue template — The document prescribes title, environment text, failing tests, and log attachments for new reports.
- **CN:** Issue 模板 — 文档规定了新报告应包含的标题、环境描述、失败测试以及日志附件。
- **EN:** Log wrangling — Helper scripts and shell commands are provided to fetch, clean, and share Buildkite logs.
- **CN:** 日志整理 — 文档提供脚本与命令，用于获取、清洗并分享 Buildkite 日志。
- **EN:** Reproduction and labeling — Contributors are encouraged to rerun flaky tests, bisect main, and label fixing PRs with `ci-failure`.
- **CN:** 复现与标记 — 贡献者应重复运行 flaky 测试、在 main 上二分定位，并给修复 PR 打上 `ci-failure` 标签。
