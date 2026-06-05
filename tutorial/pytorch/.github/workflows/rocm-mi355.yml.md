# rocm-mi355.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/rocm-mi355.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-13 / 第 1-13 行

````yaml
name: rocm-mi355

on:
  push:
    tags:
      - ciflow/rocm-mi355/*
  workflow_dispatch:
  schedule:
    - cron: 29 8 * * *  # about 1:29am PDT

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref_name }}-${{ github.ref_type == 'branch' && github.sha }}-${{ github.event_name == 'workflow_dispatch' }}-${{ github.event_name == 'schedule' }}
  cancel-in-progress: true
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 15-24 / 第 15-24 行

````yaml
permissions:
  id-token: write
  contents: read
  actions: read

jobs:
  target-determination:
    if: github.repository_owner == 'pytorch'
    name: before-test
    uses: ./.github/workflows/target_determination.yml
````

- EN: This section lays out job topology or execution stages; reuses actions and step building blocks such as `./.github/workflows/target_determination.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 26-34 / 第 26-34 行

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

### Lines 36-49 / 第 36-49 行

````yaml
  linux-noble-rocm-py3_12-build:
    if: ${{ (github.event_name != 'schedule' || github.repository == 'pytorch/pytorch') && github.repository_owner == 'pytorch' }}
    name: linux-noble-rocm-py3.12-mi355
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-noble-rocm-py3.12-mi355
      docker-image-name: ci-image:pytorch-linux-noble-rocm-n-py3
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 6, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "default", shard: 2, num_shards: 6, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "default", shard: 3, num_shards: 6, runner: "linux.rocm.gpu.gfx950.1" },
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 50-61 / 第 50-61 行

````yaml
          { config: "default", shard: 4, num_shards: 6, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "default", shard: 5, num_shards: 6, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "default", shard: 6, num_shards: 6, runner: "linux.rocm.gpu.gfx950.1" },
        ]}
    secrets: inherit

  linux-noble-rocm-py3_12-test:
    name: linux-noble-rocm-py3.12-mi355
    uses: ./.github/workflows/_rocm-test.yml
    needs:
      - linux-noble-rocm-py3_12-build
      - target-determination
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_rocm-test.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 62-66 / 第 62-66 行

````yaml
    with:
      build-environment: ${{ needs.linux-noble-rocm-py3_12-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-noble-rocm-py3_12-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-noble-rocm-py3_12-build.outputs.test-matrix }}
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
- Reusable actions / 复用 Action: `./.github/workflows/target_determination.yml`, `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `./.github/workflows/_linux-build.yml`, `./.github/workflows/_rocm-test.yml`
- Inline commands / 内联命令: `]}`
- Environment variables / 环境变量: `PDT`
- Named jobs or sections / 命名作业或章节: `on`, `push`, `tags`, `workflow_dispatch`, `schedule`, `concurrency`, `permissions`, `jobs`, `target-determination`, `get-label-type`, ...
