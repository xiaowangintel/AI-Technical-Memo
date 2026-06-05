# feature-request.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/ISSUE_TEMPLATE/feature-request.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Provides GitHub issue templates that guide reporters toward the repository's expected triage metadata.
- **用途 (CN)**: 提供 GitHub issue 模板，引导提单者填写仓库所需的分类与排障信息。

## Content Analysis / 内容分析
### Lines 1-9 / 第 1-9 行

````yaml
name: 🚀 Feature request
description: Submit a proposal/request for a new PyTorch feature

body:
- type: markdown
  attributes:
    value: >
      #### Before submitting, please review the [contribution guide](https://github.com/pytorch/pytorch/wiki/The-Ultimate-Guide-to-PyTorch-Contributions) and [AI-Assisted Development](https://github.com/pytorch/pytorch/blob/main/CONTRIBUTING.md#ai-assisted-development) policy. Issues that do not follow these practices will be automatically closed and users breaking these rules repeatedly may be banned.
- type: markdown
````

- EN: This section describes repository automation behavior for `.github/ISSUE_TEMPLATE/feature-request.yml`.
- CN: 该部分描述 `.github/ISSUE_TEMPLATE/feature-request.yml` 的仓库自动化行为。

### Lines 10-17 / 第 10-17 行

````yaml
  attributes:
    value: >
      #### Note: Please write your feature request in English to ensure it can be understood and addressed by the development team.
- type: textarea
  attributes:
    label: 🚀 The feature, motivation and pitch
    description: >
      A clear and concise description of the feature proposal. Please outline the motivation for the proposal. Is your feature request related to a specific problem? e.g., *"I'm working on X and would like Y to be possible"*. If this is related to another GitHub issue, please link here too.
````

- EN: This section describes repository automation behavior for `.github/ISSUE_TEMPLATE/feature-request.yml`.
- CN: 该部分描述 `.github/ISSUE_TEMPLATE/feature-request.yml` 的仓库自动化行为。

### Lines 18-25 / 第 18-25 行

````yaml
  validations:
    required: true
- type: textarea
  attributes:
    label: Alternatives
    description: >
      A description of any alternative solutions or features you've considered, if any.
- type: textarea
````

- EN: This section describes repository automation behavior for `.github/ISSUE_TEMPLATE/feature-request.yml`.
- CN: 该部分描述 `.github/ISSUE_TEMPLATE/feature-request.yml` 的仓库自动化行为。

### Lines 26-30 / 第 26-30 行

````yaml
  attributes:
    label: Additional context
    description: >
      Add any other context or screenshots about the feature request.
- type: markdown
````

- EN: This section describes repository automation behavior for `.github/ISSUE_TEMPLATE/feature-request.yml`.
- CN: 该部分描述 `.github/ISSUE_TEMPLATE/feature-request.yml` 的仓库自动化行为。

### Lines 31-33 / 第 31-33 行

````yaml
  attributes:
    value: >
      Thanks for contributing 🎉!
````

- EN: This section describes repository automation behavior for `.github/ISSUE_TEMPLATE/feature-request.yml`.
- CN: 该部分描述 `.github/ISSUE_TEMPLATE/feature-request.yml` 的仓库自动化行为。


## Key Concepts / 关键概念
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。
- EN: Captures repository governance and contributor-facing policies in declarative configuration.
  CN: 以声明式配置的形式记录仓库治理规则和面向贡献者的策略。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: none
- Inline commands / 内联命令: `A`, `Add`, `Thanks`
- Environment variables / 环境变量: `CONTRIBUTING`
- Named jobs or sections / 命名作业或章节: `body`, `attributes`, `validations`
