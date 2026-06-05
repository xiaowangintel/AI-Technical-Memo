# slow.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/slow.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-15 / 第 1-15 行

````yaml
# This workflow is dedicated to host slow jobs that are run only periodically because
# they are too slow to run in every commit.  The list of slow tests can be found in
# https://github.com/pytorch/test-infra/blob/generated-stats/stats/slow-tests.json
name: slow

on:
  push:
    branches:
      - main
      - release/*
    tags:
      - ciflow/slow/*
  schedule:
    - cron: 29 8 * * *  # about 1:29am PDT, for mem leak check and rerun disabled tests
  workflow_dispatch:
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 17-30 / 第 17-30 行

````yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref_name }}-${{ github.ref_type == 'branch' && github.sha }}-${{ github.event_name == 'workflow_dispatch' && github.run_id }}-${{ github.event_name == 'schedule' }}-${{ github.event.schedule }}
  cancel-in-progress: true

permissions:
  id-token: write
  contents: read
  actions: read

jobs:
  llm-td:
    if: github.repository_owner == 'pytorch'
    name: before-test
    uses: ./.github/workflows/llm_td_retrieval.yml
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; reuses actions and step building blocks such as `./.github/workflows/llm_td_retrieval.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 32-45 / 第 32-45 行

````yaml
  target-determination:
    name: before-test
    uses: ./.github/workflows/target_determination.yml
    needs: llm-td

  get-label-type:
    name: get-label-type
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
    if: ${{ (github.event_name != 'schedule' || github.repository == 'pytorch/pytorch') && github.repository_owner == 'pytorch' }}
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `./.github/workflows/target_determination.yml`, `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 47-62 / 第 47-62 行

````yaml
  linux-jammy-cuda12_8-py3_10-gcc11-sm86-build:
    name: linux-jammy-cuda12.8-py3.10-gcc11-sm86
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-cuda12.8-py3.10-gcc11-sm86
      docker-image-name: ci-image:pytorch-linux-jammy-cuda12.8-cudnn9-py3-gcc11
      cuda-arch-list: 8.6
      test-matrix: |
        { include: [
          { config: "slow", shard: 1, num_shards: 3, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "slow", shard: 2, num_shards: 3, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "slow", shard: 3, num_shards: 3, runner: "linux.g5.4xlarge.nvidia.gpu" },
        ]}
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 64-79 / 第 64-79 行

````yaml
  linux-jammy-cuda12_8-py3_10-gcc11-sm86-test:
    name: linux-jammy-cuda12.8-py3.10-gcc11-sm86
    uses: ./.github/workflows/_linux-test.yml
    needs:
      - linux-jammy-cuda12_8-py3_10-gcc11-sm86-build
      - target-determination
    with:
      build-environment: ${{ needs.linux-jammy-cuda12_8-py3_10-gcc11-sm86-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-cuda12_8-py3_10-gcc11-sm86-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-cuda12_8-py3_10-gcc11-sm86-build.outputs.test-matrix }}
    secrets: inherit

  linux-jammy-cuda13_0-py3_10-gcc11-sm86-build:
    name: linux-jammy-cuda13.0-py3.10-gcc11-sm86
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`, `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 80-96 / 第 80-96 行

````yaml
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-cuda13.0-py3.10-gcc11-sm86
      docker-image-name: ci-image:pytorch-linux-jammy-cuda13.0-cudnn9-py3-gcc11
      cuda-arch-list: 8.6
      test-matrix: |
        { include: [
          { config: "slow", shard: 1, num_shards: 3, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "slow", shard: 2, num_shards: 3, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "slow", shard: 3, num_shards: 3, runner: "linux.g5.4xlarge.nvidia.gpu" },
        ]}
    secrets: inherit

  linux-jammy-cuda13_0-py3_10-gcc11-sm86-test:
    name: linux-jammy-cuda13.0-py3.10-gcc11-sm86
    uses: ./.github/workflows/_linux-test.yml
    needs:
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 97-108 / 第 97-108 行

````yaml
      - linux-jammy-cuda13_0-py3_10-gcc11-sm86-build
      - target-determination
    with:
      build-environment: linux-jammy-cuda13.0-py3.10-gcc11-sm86
      docker-image: ${{ needs.linux-jammy-cuda13_0-py3_10-gcc11-sm86-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-cuda13_0-py3_10-gcc11-sm86-build.outputs.test-matrix }}
    secrets: inherit

  linux-jammy-py3_10-clang18-build:
    name: linux-jammy-py3.10-clang18
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 109-125 / 第 109-125 行

````yaml
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-py3.10-clang18
      docker-image-name: ci-image:pytorch-linux-jammy-py3.10-clang18
      test-matrix: |
        { include: [
          { config: "slow", shard: 1, num_shards: 2, runner: "linux.2xlarge" },
          { config: "slow", shard: 2, num_shards: 2, runner: "linux.2xlarge" },
        ]}
    secrets: inherit

  linux-jammy-py3_10-clang18-test:
    name: linux-jammy-py3.10-clang18
    uses: ./.github/workflows/_linux-test.yml
    needs:
      - linux-jammy-py3_10-clang18-build
      - target-determination
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 126-143 / 第 126-143 行

````yaml
    with:
      build-environment: ${{ needs.linux-jammy-py3_10-clang18-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-py3_10-clang18-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-py3_10-clang18-build.outputs.test-matrix }}
    secrets: inherit

  linux-jammy-py3_10-clang18-asan-build:
    name: linux-jammy-py3.10-clang18-asan
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
    with:
      runner: linux.c7i.4xlarge
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-py3.10-clang18-asan
      docker-image-name: ci-image:pytorch-linux-jammy-py3.10-clang18
      test-matrix: |
        { include: [
          { config: "slow", shard: 1, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 144-155 / 第 144-155 行

````yaml
          { config: "slow", shard: 2, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "slow", shard: 3, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
        ]}
      sync-tag: asan-build
    secrets: inherit

  linux-jammy-py3_10-clang18-asan-test:
    name: linux-jammy-py3.10-clang18-asan
    uses: ./.github/workflows/_linux-test.yml
    needs:
      - linux-jammy-py3_10-clang18-asan-build
      - target-determination
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 156-161 / 第 156-161 行

````yaml
    with:
      build-environment: ${{ needs.linux-jammy-py3_10-clang18-asan-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-py3_10-clang18-asan-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-py3_10-clang18-asan-build.outputs.test-matrix }}
      sync-tag: asan-test
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
- Named jobs or sections / 命名作业或章节: `on`, `push`, `branches`, `tags`, `schedule`, `workflow_dispatch`, `concurrency`, `permissions`, `jobs`, `llm-td`, ...
