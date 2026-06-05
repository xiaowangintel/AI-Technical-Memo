# trunk-rocm-sandbox.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/trunk-rocm-sandbox.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-10 / 第 1-10 行

````yaml
name: trunk-rocm-sandbox

on:
  #push:
  #  branches:
  #    - main
  #    - release/*
  workflow_dispatch:
  schedule:
    - cron: 0 0,3,6,9,12,15,18,21 * * * # run every 3 hours for regression coverage
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

### Lines 22-30 / 第 22-30 行

````yaml
  llm-td:
    if: github.repository_owner == 'pytorch'
    name: before-test
    uses: ./.github/workflows/llm_td_retrieval.yml

  target-determination:
    name: before-test
    uses: ./.github/workflows/target_determination.yml
    needs: llm-td
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/llm_td_retrieval.yml`, `./.github/workflows/target_determination.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 32-40 / 第 32-40 行

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
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 42-55 / 第 42-55 行

````yaml
  linux-jammy-rocm-py3_10-build:
    name: linux-jammy-rocm-py3.10
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-rocm-py3.10
      docker-image-name: ci-image:pytorch-linux-jammy-rocm-n-py3
      sync-tag: rocm-build
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 6, runner: "linux.rocm.gpu.mi210.1.test" },
          { config: "default", shard: 2, num_shards: 6, runner: "linux.rocm.gpu.mi210.1.test" },
          { config: "default", shard: 3, num_shards: 6, runner: "linux.rocm.gpu.mi210.1.test" },
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 56-65 / 第 56-65 行

````yaml
          { config: "default", shard: 4, num_shards: 6, runner: "linux.rocm.gpu.mi210.1.test" },
          { config: "default", shard: 5, num_shards: 6, runner: "linux.rocm.gpu.mi210.1.test" },
          { config: "default", shard: 6, num_shards: 6, runner: "linux.rocm.gpu.mi210.1.test" },
          { config: "inductor", shard: 1, num_shards: 2, runner: "linux.rocm.gpu.mi210.1.test" },
          { config: "inductor", shard: 2, num_shards: 2, runner: "linux.rocm.gpu.mi210.1.test" },
          { config: "distributed", shard: 1, num_shards: 3, runner: "linux.rocm.gpu.mi210.2.test" },
          { config: "distributed", shard: 2, num_shards: 3, runner: "linux.rocm.gpu.mi210.2.test" },
          { config: "distributed", shard: 3, num_shards: 3, runner: "linux.rocm.gpu.mi210.2.test" },
        ]}
    secrets: inherit
````

- EN: This section describes repository automation behavior for `.github/workflows/trunk-rocm-sandbox.yml`.
- CN: 该部分描述 `.github/workflows/trunk-rocm-sandbox.yml` 的仓库自动化行为。

### Lines 67-72 / 第 67-72 行

````yaml
  linux-jammy-rocm-py3_10-test:
    name: linux-jammy-rocm-py3.10
    uses: ./.github/workflows/_rocm-test.yml
    needs:
      - linux-jammy-rocm-py3_10-build
      - target-determination
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_rocm-test.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 73-77 / 第 73-77 行

````yaml
    with:
      build-environment: ${{ needs.linux-jammy-rocm-py3_10-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-rocm-py3_10-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-rocm-py3_10-build.outputs.test-matrix }}
    secrets: inherit
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。


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
- EN: Manual workflow triggering — maintainers can run this automation on demand from GitHub Actions.
  CN: 手动工作流触发——维护者可以在 GitHub Actions 中按需执行该自动化流程。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: `./.github/workflows/llm_td_retrieval.yml`, `./.github/workflows/target_determination.yml`, `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `./.github/workflows/_linux-build.yml`, `./.github/workflows/_rocm-test.yml`
- Inline commands / 内联命令: `]}`
- Environment variables / 环境变量: none
- Named jobs or sections / 命名作业或章节: `on`, `workflow_dispatch`, `schedule`, `concurrency`, `permissions`, `jobs`, `llm-td`, `target-determination`, `get-label-type`, `with`, ...
