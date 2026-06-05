# ci-sev.md — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/ISSUE_TEMPLATE/ci-sev.md`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Provides GitHub issue templates that guide reporters toward the repository's expected triage metadata.
- **用途 (CN)**: 提供 GitHub issue 模板，引导提单者填写仓库所需的分类与排障信息。

## Content Analysis / 内容分析
### Lines 1-8 / 第 1-8 行

````markdown
---
name: "⚠️ CI SEV"
about: Tracking incidents for PyTorch's CI infra.
title: ''
labels: ''
assignees: ''

---
````

- EN: This section explains repository context in prose form.
- CN: 该部分以文字形式解释仓库上下文。

### Lines 10-16 / 第 10-16 行

````markdown
> NOTE: Remember to label this issue with "`ci: sev`"
>       If you want autorevert to be disabled, keep the ci: disable-autorevert label

 <!-- Add the `merge blocking` label to this PR to prevent PRs from being merged while this issue is open -->

## Current Status
*Status could be: preemptive, ongoing, mitigated, closed. Also tell people if they need to take action to fix it (i.e. rebase)*.
````

- EN: This section introduces headings that organize the narrative or instructions.
- CN: 该部分通过标题组织叙述或操作说明。

### Lines 18-25 / 第 18-25 行

````markdown
## Error looks like
*Provide some way users can tell that this SEV is causing their issue.*

## Incident timeline (all times pacific)
*Include when the incident began, when it was detected, mitigated, root caused, and finally closed.*

## User impact
*How does this affect users of PyTorch CI?*
````

- EN: This section introduces headings that organize the narrative or instructions.
- CN: 该部分通过标题组织叙述或操作说明。

### Lines 27-31 / 第 27-31 行

````markdown
## Root cause
*What was the root cause of this issue?*

## Mitigation
*How did we mitigate the issue?*
````

- EN: This section introduces headings that organize the narrative or instructions.
- CN: 该部分通过标题组织叙述或操作说明。

### Lines 33-34 / 第 33-34 行

````markdown
## Prevention/followups
*How do we prevent issues like this in the future?*
````

- EN: This section introduces headings that organize the narrative or instructions.
- CN: 该部分通过标题组织叙述或操作说明。


## Key Concepts / 关键概念
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Captures repository governance and contributor-facing policies in declarative configuration.
  CN: 以声明式配置的形式记录仓库治理规则和面向贡献者的策略。

## Dependencies / 依赖关系
- Headings or named sections / 标题或命名章节: `Current Status`, `Error looks like`, `Incident timeline (all times pacific)`, `User impact`, `Root cause`, `Mitigation`, `Prevention/followups`
- Referenced paths / 引用路径: `Prevention/followups`
- URLs / 链接: none
