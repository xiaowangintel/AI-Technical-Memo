# disable-ci-jobs.md — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/ISSUE_TEMPLATE/disable-ci-jobs.md`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Provides GitHub issue templates that guide reporters toward the repository's expected triage metadata.
- **用途 (CN)**: 提供 GitHub issue 模板，引导提单者填写仓库所需的分类与排障信息。

## Content Analysis / 内容分析
### Lines 1-8 / 第 1-8 行

````markdown
---
name: Disable CI jobs (PyTorch Dev Infra only)
about: Use this template to disable CI jobs
title: DISABLED [WORKFLOW_NAME] / [PLATFORM_NAME] / [JOB_NAME]
labels: 'module: ci'
assignees: ''

---
````

- EN: This section explains repository context in prose form.
- CN: 该部分以文字形式解释仓库上下文。

### Lines 10-16 / 第 10-16 行

````markdown
> For example, DISABLED pull / win-vs2022-cpu-py3 / test (default). Once
> created, the job will be disabled within 15 minutes. You can check the
> list of disabled jobs at https://ossci-metrics.s3.amazonaws.com/disabled-jobs.json

> If you need to get this out ASAP instead of waiting for 15 minutes,
> you can manually trigger the workflow at https://github.com/pytorch/test-infra/actions/workflows/update_disabled_tests.yml
> once the issue is created to update the above JSON list right away.
````

- EN: This section links to external references or repository resources.
- CN: 该部分链接到外部参考资料或仓库资源。

### Lines 18-19 / 第 18-19 行

````markdown
> Noted: you need to have write access to PyTorch repo to disable CI
> jobs. The issue will be rejected otherwise.
````

- EN: This section explains repository context in prose form.
- CN: 该部分以文字形式解释仓库上下文。

### Lines 21-22 / 第 21-22 行

````markdown
## Reason
*Provide a reason why this is needed and when this can be resolved*.
````

- EN: This section introduces headings that organize the narrative or instructions.
- CN: 该部分通过标题组织叙述或操作说明。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。
- EN: Captures repository governance and contributor-facing policies in declarative configuration.
  CN: 以声明式配置的形式记录仓库治理规则和面向贡献者的策略。

## Dependencies / 依赖关系
- Headings or named sections / 标题或命名章节: `Reason`
- Referenced paths / 引用路径: `//ossci-metrics.s3.amazonaws.com/disabled-jobs.json`, `//github.com/pytorch/test-infra/actions/workflows/update_disabled_tests.yml`
- URLs / 链接: `https://ossci-metrics.s3.amazonaws.com/disabled-jobs.json`, `https://github.com/pytorch/test-infra/actions/workflows/update_disabled_tests.yml`
