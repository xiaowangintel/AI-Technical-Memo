# fix_issue.md — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/PULL_REQUEST_TEMPLATE/fix_issue.md`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Provides pull-request templates that standardize contributor checklists, release notes, and reviewer context.
- **用途 (CN)**: 提供 Pull Request 模板，统一贡献者检查项、发布说明以及评审上下文。

## Content Analysis / 内容分析
### Lines 1-9 / 第 1-9 行

````markdown
# Fixing an Issue

Before submitting, please review:
- [The Ultimate Guide to PyTorch Contributions](https://github.com/pytorch/pytorch/wiki/The-Ultimate-Guide-to-PyTorch-Contributions#getting-started-with-pull-requests)
- [AI-Assisted Development](CONTRIBUTING.md#ai-assisted-development) policy

---

## Issue
````

- EN: This section introduces headings that organize the narrative or instructions; lists actionable items, options, or policy bullets; links to external references or repository resources.
- CN: 该部分通过标题组织叙述或操作说明；列出可执行事项、选项或策略要点；链接到外部参考资料或仓库资源。

### Lines 11-19 / 第 11-19 行

````markdown
Fixes #<!-- Issue number. PRs without a linked issue may be automatically closed. -->

## Summary

<!-- Point to the issue for relevant design discussion. If not discussed there (rare), add up to one paragraph. Overly verbose descriptions will be considered spam. -->

## Checklist

- [ ] Passes lint (`spin fixlint`)
````

- EN: This section introduces headings that organize the narrative or instructions; lists actionable items, options, or policy bullets.
- CN: 该部分通过标题组织叙述或操作说明；列出可执行事项、选项或策略要点。

### Lines 20-24 / 第 20-24 行

````markdown
- [ ] Added/updated tests
- [ ] Updated documentation (if applicable)
- [ ] Included benchmark results (for PRs impacting perf)

## BC-breaking?
````

- EN: This section introduces headings that organize the narrative or instructions; lists actionable items, options, or policy bullets.
- CN: 该部分通过标题组织叙述或操作说明；列出可执行事项、选项或策略要点。

### Lines 26-26 / 第 26-26 行

````markdown
<!-- If this change breaks backward compatibility, describe the impact and migration path. Otherwise, write "No". -->
````

- EN: This section explains repository context in prose form.
- CN: 该部分以文字形式解释仓库上下文。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Captures repository governance and contributor-facing policies in declarative configuration.
  CN: 以声明式配置的形式记录仓库治理规则和面向贡献者的策略。

## Dependencies / 依赖关系
- Headings or named sections / 标题或命名章节: `Fixing an Issue`, `Issue`, `Summary`, `Checklist`, `BC-breaking?`
- Referenced paths / 引用路径: `//github.com/pytorch/pytorch/wiki/The-Ultimate-Guide-to-PyTorch-Contributions`, `Added/updated`
- URLs / 链接: `https://github.com/pytorch/pytorch/wiki/The-Ultimate-Guide-to-PyTorch-Contributions#getting-started-with-pull-requests`
