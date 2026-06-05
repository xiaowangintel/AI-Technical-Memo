# claude-code.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/claude-code.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-9 / 第 1-9 行

````yaml
name: Claude Code

on:
  issue_comment:
    types: [created]
  issues:
    types: [opened]

jobs:
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段。

### Lines 10-17 / 第 10-17 行

````yaml
  claude-code:
    uses: pytorch/test-infra/.github/workflows/_claude-code.yml@main
    permissions:
      contents: read
      pull-requests: write
      issues: write
      id-token: write
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/workflows/_claude-code.yml@main`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 18-22 / 第 18-22 行

````yaml
    with:
      additional_claude_args: '--allowedTools Skill'
      append_system_prompt: |
        When asked to review a PR, always use the /pr-review skill first.
        It contains PyTorch-specific review guidelines, output format, and critical checks.
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: `pytorch/test-infra/.github/workflows/_claude-code.yml@main`
- Inline commands / 内联命令: `When`, `It`
- Environment variables / 环境变量: none
- Named jobs or sections / 命名作业或章节: `on`, `issue_comment`, `issues`, `jobs`, `claude-code`, `permissions`, `with`
