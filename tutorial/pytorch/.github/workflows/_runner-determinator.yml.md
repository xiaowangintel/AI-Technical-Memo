# _runner-determinator.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/_runner-determinator.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-11 / 第 1-11 行

````yaml
name: Check whether the workflow owner can use ARC runners

on:
  workflow_call:
    inputs:
      check_experiments:
        required: false
        type: string
        description: |
          Comma-separated list of non-default experiments to opt into for this workflow.
          These are added on top of all default experiments. If not defined, only default experiments are included.
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 12-23 / 第 12-23 行

````yaml
      opt_out_experiments:
        required: false
        type: string
        description: Comma-separated list of experiments this workflow will opt-out of.
      triggering_actor:
        required: true
        type: string
        description: The triggering_actor for the workflow. Use github.triggering_actor
      issue_owner:
        required: true
        type: string
        description: The owner of the issue. Use github.event.pull_request.user.login || github.event.issue.user.login
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 24-32 / 第 24-32 行

````yaml
      curr_branch:
        required: true
        type: string
        description: Current branch or tag.
      curr_ref_type:
        required: false
        type: string
        default: branch
        description: The value of "github.ref_type", "branch" or "tag"
````

- EN: This section describes repository automation behavior for `.github/workflows/_runner-determinator.yml`.
- CN: 该部分描述 `.github/workflows/_runner-determinator.yml` 的仓库自动化行为。

### Lines 33-44 / 第 33-44 行

````yaml
      issue_number:
        required: false
        type: string
        default: "5132"
        description: |
          Fetch's GitHub Issue from pytorch/test-infra
          Example: https://github.com/pytorch/test-infra/issues/5132
      runner_config:
        required: false
        type: string
        default: ""
        description: Runner configuration used by the caller to derive runner-type.
````

- EN: This section describes repository automation behavior for `.github/workflows/_runner-determinator.yml`.
- CN: 该部分描述 `.github/workflows/_runner-determinator.yml` 的仓库自动化行为。

### Lines 46-58 / 第 46-58 行

````yaml
    outputs:
      label-type:
        description: Type of runners to use
        value: ${{ jobs.runner-determinator.outputs.label-type }}
      runner-config:
        description: Normalized runner configuration derived from the caller input
        value: ${{ jobs.runner-determinator.outputs.runner-config }}
      runner-type:
        description: Runner suffix to use for workflow-specific runner selection
        value: ${{ jobs.runner-determinator.outputs.runner-type }}
      runner-label:
        description: Fully qualified runner label derived from runner_config
        value: ${{ jobs.runner-determinator.outputs.runner-label }}
````

- EN: This section describes repository automation behavior for `.github/workflows/_runner-determinator.yml`.
- CN: 该部分描述 `.github/workflows/_runner-determinator.yml` 的仓库自动化行为。

### Lines 59-67 / 第 59-67 行

````yaml
      use-arc:
        description: Whether to use ARC runners
        value: ${{ jobs.runner-determinator.outputs.use-arc }}

jobs:
  runner-determinator:
    # Don't run on forked repos
    if: github.repository_owner == 'pytorch'
    runs-on: ubuntu-latest
````

- EN: This section lays out job topology or execution stages.
- CN: 该部分铺排作业拓扑或执行阶段。

### Lines 68-81 / 第 68-81 行

````yaml
    outputs:
      label-type: ${{ steps.set-condition.outputs.label-type }}
      runner-config: ${{ steps.set-runner-info.outputs.runner-config }}
      runner-type: ${{ steps.set-runner-info.outputs.runner-type }}
      runner-label: ${{ steps.set-runner-info.outputs.runner-label }}
      use-arc: ${{ steps.set-condition.outputs.use-arc }}
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      ISSUE_NUMBER: ${{ inputs.issue_number }}
      TRIGGERING_ACTOR: ${{ inputs.triggering_actor }}
      ISSUE_OWNER: ${{ inputs.issue_owner }}
      CHECK_EXPERIMENTS: ${{ inputs.check_experiments }}
      OPT_OUT_EXPERIMENTS: ${{ inputs.opt_out_experiments }}
      PR_NUMBER: ${{ github.event.pull_request.number }}
````

- EN: This section declares workflow triggers or event bindings; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；控制令牌权限范围或环境变量以实现安全执行。

### Lines 82-94 / 第 82-94 行

````yaml
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2

      - name: Install dependencies
        run: python3 -m pip install urllib3==1.26.18 PyGithub==2.3.0

      - name: Get the workflow type for the current user
        id: set-condition
        run: |
          curr_branch="${{ inputs.curr_branch }}"
          curr_ref_type="${{ inputs.curr_ref_type }}"
          echo "Current branch is '$curr_branch'"
````

- EN: This section reuses actions and step building blocks such as `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 96-106 / 第 96-106 行

````yaml
          python3 .github/scripts/runner_determinator.py \
            --github-token "$GITHUB_TOKEN" \
            --github-issue "$ISSUE_NUMBER" \
            --github-branch "$curr_branch" \
            --github-actor "$TRIGGERING_ACTOR" \
            --github-issue-owner "$ISSUE_OWNER" \
            --github-ref-type "$curr_ref_type" \
            --github-repo "$GITHUB_REPOSITORY" \
            --eligible-experiments "$CHECK_EXPERIMENTS" \
            --opt-out-experiments "$OPT_OUT_EXPERIMENTS" \
            --pr-number "${PR_NUMBER}"
````

- EN: This section describes repository automation behavior for `.github/workflows/_runner-determinator.yml`.
- CN: 该部分描述 `.github/workflows/_runner-determinator.yml` 的仓库自动化行为。

### Lines 108-121 / 第 108-121 行

````yaml
      - name: Determine runner configuration
        id: set-runner-info
        run: |
          case "${{ inputs.runner_config }}" in
            "")
              runner_config="m8g"
              runner_type="metal-24xl"
              ;;
            "m8g")
              runner_config="m8g"
              runner_type="metal-24xl"
              ;;
            "m7g")
              runner_config="m7g"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 122-128 / 第 122-128 行

````yaml
              runner_type="metal"
              ;;
            *)
              echo "Unsupported runner_config: ${{ inputs.runner_config }}"
              exit 1
              ;;
          esac
````

- EN: This section describes repository automation behavior for `.github/workflows/_runner-determinator.yml`.
- CN: 该部分描述 `.github/workflows/_runner-determinator.yml` 的仓库自动化行为。

### Lines 130-133 / 第 130-133 行

````yaml
          runner_label="linux.arm64.${runner_config}.${runner_type}"
          echo "runner-config=${runner_config}" >> "$GITHUB_OUTPUT"
          echo "runner-type=${runner_type}" >> "$GITHUB_OUTPUT"
          echo "runner-label=${runner_label}" >> "$GITHUB_OUTPUT"
````

- EN: This section describes repository automation behavior for `.github/workflows/_runner-determinator.yml`.
- CN: 该部分描述 `.github/workflows/_runner-determinator.yml` 的仓库自动化行为。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`
- Inline commands / 内联命令: `Comma-separated`, `These`, `Fetch's`, `echo`, `python3`, `github-token`, `github-issue`, `github-branch`, ...
- Environment variables / 环境变量: `ARC`, `GITHUB_TOKEN`, `ISSUE_NUMBER`, `TRIGGERING_ACTOR`, `ISSUE_OWNER`, `CHECK_EXPERIMENTS`, `OPT_OUT_EXPERIMENTS`, `PR_NUMBER`, `GITHUB_REPOSITORY`, `GITHUB_OUTPUT`
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `inputs`, `outputs`, `jobs`, `runner-determinator`, `env`, `steps`
