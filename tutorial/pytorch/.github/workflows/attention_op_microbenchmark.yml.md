# attention_op_microbenchmark.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/attention_op_microbenchmark.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-10 / 第 1-10 行

````yaml
name: attention_op_microbenchmark

on:
  push:
    tags:
      - ciflow/op-benchmark/*
  workflow_dispatch:
  schedule:
    # Run at 06:00 UTC everyday
    - cron: 0 7 * * *
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 12-21 / 第 12-21 行

````yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref_name }}-${{ github.ref_type == 'branch' && github.sha }}-${{ github.event_name == 'workflow_dispatch' }}-${{ github.event_name == 'schedule' }}
  cancel-in-progress: true

permissions:
  id-token: write
  contents: read
  actions: read

jobs:
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 22-31 / 第 22-31 行

````yaml
  get-label-type:
    name: get-label-type
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
    if: ${{ (github.event_name != 'schedule' || github.repository == 'pytorch/pytorch') && github.repository_owner == 'pytorch' }}
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}
      check_experiments: arc,lf
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 33-46 / 第 33-46 行

````yaml
  attn-microbenchmark-build:
    if: github.repository_owner == 'pytorch'
    uses: ./.github/workflows/_linux-build.yml
    with:
      runner: linux.12xlarge.memory
      build-environment: linux-jammy-cuda12.8-py3.10-gcc11-sm80
      docker-image-name: ci-image:pytorch-linux-jammy-cuda12.8-cudnn9-py3-gcc11
      cuda-arch-list: '8.0 9.0'
      test-matrix: |
        { include: [
          { config: "attention_microbenchmark_test", shard: 1, num_shards: 1, runner: "linux.aws.a100" },
          { config: "attention_microbenchmark_test", shard: 1, num_shards: 1, runner: "linux.aws.h100" },
        ]}
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 48-59 / 第 48-59 行

````yaml
  attn-microbenchmark-test:
    name: attn-microbenchmark-test
    uses: ./.github/workflows/_linux-test.yml
    needs: attn-microbenchmark-build
    with:
      timeout-minutes: 500
      build-environment: ${{ needs.attn-microbenchmark-build.outputs.build-environment }}
      docker-image: ${{ needs.attn-microbenchmark-build.outputs.docker-image }}
      test-matrix: ${{ needs.attn-microbenchmark-build.outputs.test-matrix }}
    secrets: inherit

  # B200 runner (OSDC), always use OSDC runner to test this workflow
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 60-73 / 第 60-73 行

````yaml
  opmicrobenchmark-build-b200:
    if: github.repository_owner == 'pytorch'
    name: opmicrobenchmark-build-b200
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
    with:
      runner_prefix: "mt-"
      runner: linux.r7i.4xlarge
      build-environment: linux-jammy-cuda12.8-py3.10-gcc11-sm100
      docker-image-name: ci-image:pytorch-linux-jammy-cuda12.8-cudnn9-py3-gcc11
      cuda-arch-list: '10.0'
      test-matrix: |
        { include: [
          { config: "operator_microbenchmark_test", shard: 1, num_shards: 1, runner: "linux.dgx.b200" },
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 74-86 / 第 74-86 行

````yaml
        ]}
      use-arc: true
      python-version: "3.10"
      compiler: gcc11
      cuda-version: "12.8"
    secrets: inherit

  opmicrobenchmark-test-b200:
    name: opmicrobenchmark-test-b200
    uses: ./.github/workflows/_linux-test.yml
    needs:
      - opmicrobenchmark-build-b200
      - get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 87-96 / 第 87-96 行

````yaml
    with:
      timeout-minutes: 500
      build-environment: ${{ needs.opmicrobenchmark-build-b200.outputs.build-environment }}
      docker-image: ${{ needs.opmicrobenchmark-build-b200.outputs.docker-image }}
      test-matrix: ${{ needs.opmicrobenchmark-build-b200.outputs.test-matrix }}
      use-arc: true
      python-version: "3.10"
      compiler: gcc11
      cuda-version: "12.8"
    secrets: inherit
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Configures native build inputs, optional backends, and dependency discovery at configure time.
  CN: 在配置阶段设置原生构建输入、可选后端以及依赖发现逻辑。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `./.github/workflows/_linux-build.yml`, `./.github/workflows/_linux-test.yml`
- Inline commands / 内联命令: `]}`
- Environment variables / 环境变量: `UTC`, `B200`, `OSDC`
- Named jobs or sections / 命名作业或章节: `on`, `push`, `tags`, `workflow_dispatch`, `schedule`, `concurrency`, `permissions`, `jobs`, `get-label-type`, `with`, ...
