# periodic.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/periodic.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-13 / 第 1-13 行

````yaml
name: periodic

on:
  schedule:
    # We have several schedules so jobs can check github.event.schedule to activate only for a fraction of the runs.
    # Also run less frequently on weekends.
    - cron: 45 0,8,16 * * 1-5
    - cron: 45 4 * * 0,6
    - cron: 45 4,12,20 * * 1-5
    - cron: 45 12 * * 0,6
    - cron: 29 8 * * *  # about 1:29am PDT, for mem leak check and rerun disabled tests
  push:
    tags:
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 14-26 / 第 14-26 行

````yaml
      - ciflow/periodic/*
    branches:
      - release/*
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref_name }}-${{ github.ref_type == 'branch' && github.sha }}-${{ github.event_name == 'workflow_dispatch' && github.run_id }}-${{ github.event_name == 'schedule' }}-${{ github.event.schedule }}
  cancel-in-progress: true

permissions:
  id-token: write
  contents: read
  actions: read
````

- EN: This section declares workflow triggers or event bindings; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；控制令牌权限范围或环境变量以实现安全执行。

### Lines 28-37 / 第 28-37 行

````yaml
jobs:
  llm-td:
    if: github.repository_owner == 'pytorch'
    name: before-test
    uses: ./.github/workflows/llm_td_retrieval.yml

  target-determination:
    name: before-test
    uses: ./.github/workflows/target_determination.yml
    needs: llm-td
````

- EN: This section lays out job topology or execution stages; reuses actions and step building blocks such as `./.github/workflows/llm_td_retrieval.yml`, `./.github/workflows/target_determination.yml`.
- CN: 该部分铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程。

### Lines 39-47 / 第 39-47 行

````yaml
  get-label-type:
    name: get-label-type
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
    if: (github.event_name != 'schedule' || github.repository == 'pytorch/pytorch') && github.repository_owner == 'pytorch'
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 49-62 / 第 49-62 行

````yaml
  linux-jammy-cuda13_0-py3_10-gcc11-build:
    name: linux-jammy-cuda13.0-py3.10-gcc11
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-cuda13.0-py3.10-gcc11
      docker-image-name: ci-image:pytorch-linux-jammy-cuda13.0-cudnn9-py3-gcc11
      cuda-arch-list: '7.5 8.6 8.9'
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu" },
          { config: "default", shard: 2, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu" },
          { config: "default", shard: 3, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu" },
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 63-76 / 第 63-76 行

````yaml
          { config: "default", shard: 4, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu" },
          { config: "default", shard: 5, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu" },
          { config: "distributed", shard: 1, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g4dn.12xlarge.nvidia.gpu" },
          { config: "distributed", shard: 2, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g4dn.12xlarge.nvidia.gpu" },
          { config: "distributed", shard: 3, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g4dn.12xlarge.nvidia.gpu" },
          { config: "nogpu_AVX512", shard: 1, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "nogpu_AVX512", shard: 2, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "nogpu_AVX512", shard: 3, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "nogpu_NO_AVX2", shard: 1, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "nogpu_NO_AVX2", shard: 2, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "jit_legacy", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g5.4xlarge.nvidia.gpu" },
          { config: "multigpu", shard: 1, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g5.12xlarge.nvidia.gpu", owners: ["oncall:distributed"] },
          { config: "multigpu", shard: 2, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g5.12xlarge.nvidia.gpu", owners: ["oncall:distributed"] },
        ]}
````

- EN: This section describes repository automation behavior for `.github/workflows/periodic.yml`.
- CN: 该部分描述 `.github/workflows/periodic.yml` 的仓库自动化行为。

### Lines 77-89 / 第 77-89 行

````yaml
    secrets: inherit

  linux-jammy-cuda13_0-py3_10-gcc11-test:
    name: linux-jammy-cuda13.0-py3.10-gcc11
    uses: ./.github/workflows/_linux-test.yml
    needs:
      - linux-jammy-cuda13_0-py3_10-gcc11-build
      - target-determination
    with:
      build-environment: ${{ needs.linux-jammy-cuda13_0-py3_10-gcc11-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-cuda13_0-py3_10-gcc11-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-cuda13_0-py3_10-gcc11-build.outputs.test-matrix }}
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 91-104 / 第 91-104 行

````yaml
  linux-jammy-cuda13_0-py3_10-gcc11-debug-build:
    name: linux-jammy-cuda13.0-py3.10-gcc11-debug
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-cuda13.0-py3.10-gcc11-debug
      docker-image-name: ci-image:pytorch-linux-jammy-cuda13.0-cudnn9-py3-gcc11
      cuda-arch-list: 8.9
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 7, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu", owners: ["oncall:debug-build"] },
          { config: "default", shard: 2, num_shards: 7, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu", owners: ["oncall:debug-build"] },
          { config: "default", shard: 3, num_shards: 7, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu", owners: ["oncall:debug-build"] },
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 105-117 / 第 105-117 行

````yaml
          { config: "default", shard: 4, num_shards: 7, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu", owners: ["oncall:debug-build"] },
          { config: "default", shard: 5, num_shards: 7, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu", owners: ["oncall:debug-build"] },
          { config: "default", shard: 6, num_shards: 7, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu", owners: ["oncall:debug-build"] },
          { config: "default", shard: 7, num_shards: 7, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu", owners: ["oncall:debug-build"] },
        ]}
    secrets: inherit

  linux-jammy-cuda13_0-py3_10-gcc11-debug-test:
    name: linux-jammy-cuda13.0-py3.10-gcc11-debug
    uses: ./.github/workflows/_linux-test.yml
    needs:
      - linux-jammy-cuda13_0-py3_10-gcc11-debug-build
      - target-determination
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 118-127 / 第 118-127 行

````yaml
    with:
      build-environment: ${{ needs.linux-jammy-cuda13_0-py3_10-gcc11-debug-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-cuda13_0-py3_10-gcc11-debug-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-cuda13_0-py3_10-gcc11-debug-build.outputs.test-matrix }}
    secrets: inherit

  linux-jammy-cuda13_0-py3-gcc11-slow-gradcheck-build:
    name: linux-jammy-cuda13.0-py3-gcc11-slow-gradcheck
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 128-141 / 第 128-141 行

````yaml
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-cuda13.0-py3-gcc11-slow-gradcheck
      docker-image-name: ci-image:pytorch-linux-jammy-cuda13.0-cudnn9-py3-gcc11
      cuda-arch-list: 8.6
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 8, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g5.4xlarge.nvidia.gpu", owners: ["module:slowgradcheck"] },
          { config: "default", shard: 2, num_shards: 8, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g5.4xlarge.nvidia.gpu", owners: ["module:slowgradcheck"] },
          { config: "default", shard: 3, num_shards: 8, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g5.4xlarge.nvidia.gpu", owners: ["module:slowgradcheck"] },
          { config: "default", shard: 4, num_shards: 8, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g5.4xlarge.nvidia.gpu", owners: ["module:slowgradcheck"] },
          { config: "default", shard: 5, num_shards: 8, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g5.4xlarge.nvidia.gpu", owners: ["module:slowgradcheck"] },
          { config: "default", shard: 6, num_shards: 8, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g5.4xlarge.nvidia.gpu", owners: ["module:slowgradcheck"] },
          { config: "default", shard: 7, num_shards: 8, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g5.4xlarge.nvidia.gpu", owners: ["module:slowgradcheck"] },
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 142-151 / 第 142-151 行

````yaml
          { config: "default", shard: 8, num_shards: 8, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g5.4xlarge.nvidia.gpu", owners: ["module:slowgradcheck"] },
        ]}
    secrets: inherit

  linux-jammy-cuda13_0-py3-gcc11-slow-gradcheck-test:
    name: linux-jammy-cuda13.0-py3-gcc11-slow-gradcheck
    uses: ./.github/workflows/_linux-test.yml
    needs:
      - linux-jammy-cuda13_0-py3-gcc11-slow-gradcheck-build
      - target-determination
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 152-157 / 第 152-157 行

````yaml
    with:
      build-environment: ${{ needs.linux-jammy-cuda13_0-py3-gcc11-slow-gradcheck-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-cuda13_0-py3-gcc11-slow-gradcheck-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-cuda13_0-py3-gcc11-slow-gradcheck-build.outputs.test-matrix }}
      timeout-minutes: 300
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
- Reusable actions / 复用 Action: `./.github/workflows/llm_td_retrieval.yml`, `./.github/workflows/target_determination.yml`, `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `./.github/workflows/_linux-build.yml`, `./.github/workflows/_linux-test.yml`
- Inline commands / 内联命令: `]}`
- Environment variables / 环境变量: `PDT`
- Named jobs or sections / 命名作业或章节: `on`, `schedule`, `push`, `tags`, `branches`, `workflow_dispatch`, `concurrency`, `permissions`, `jobs`, `llm-td`, ...
