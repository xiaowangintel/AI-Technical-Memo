# xpu.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/xpu.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-11 / 第 1-11 行

````yaml
name: xpu

on:
  push:
    tags:
      - ciflow/xpu/*
  workflow_dispatch:
  schedule:
    # Run 3 times on weekdays and less frequently on weekends.
    - cron: 45 0,8,16 * * 1-5
    - cron: 45 4 * * 0,6
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 13-22 / 第 13-22 行

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

### Lines 24-32 / 第 24-32 行

````yaml
  get-label-type:
    if: github.repository_owner == 'pytorch'
    name: get-label-type
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 34-47 / 第 34-47 行

````yaml
  linux-jammy-xpu-n-1-py3_10-build:
    name: linux-jammy-xpu-n-1-py3.10
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
    with:
      sync-tag: linux-xpu-n-1-build
      runner_prefix: ${{ needs.get-label-type.outputs.label-type }}
      build-environment: linux-jammy-xpu-n-1-py3.10
      docker-image-name: ci-image:pytorch-linux-jammy-xpu-n-1-py3
      runner: linux.c7i.12xlarge
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 6, runner: "linux.idc.xpu" },
          { config: "default", shard: 2, num_shards: 6, runner: "linux.idc.xpu" },
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 48-58 / 第 48-58 行

````yaml
          { config: "default", shard: 3, num_shards: 6, runner: "linux.idc.xpu" },
          { config: "default", shard: 4, num_shards: 6, runner: "linux.idc.xpu" },
          { config: "default", shard: 5, num_shards: 6, runner: "linux.idc.xpu" },
          { config: "default", shard: 6, num_shards: 6, runner: "linux.idc.xpu" },
        ]}
    secrets: inherit

  linux-noble-xpu-n-py3_10-build:
    name: linux-noble-xpu-n-py3.10
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 59-72 / 第 59-72 行

````yaml
    with:
      sync-tag: linux-xpu-n-build
      runner_prefix: ${{ needs.get-label-type.outputs.label-type }}
      build-environment: linux-noble-xpu-n-py3.10
      docker-image-name: ci-image:pytorch-linux-noble-xpu-n-py3
      runner: linux.c7i.12xlarge
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 12, runner: "linux.idc.xpu" },
          { config: "default", shard: 2, num_shards: 12, runner: "linux.idc.xpu" },
          { config: "default", shard: 3, num_shards: 12, runner: "linux.idc.xpu" },
          { config: "default", shard: 4, num_shards: 12, runner: "linux.idc.xpu" },
          { config: "default", shard: 5, num_shards: 12, runner: "linux.idc.xpu" },
          { config: "default", shard: 6, num_shards: 12, runner: "linux.idc.xpu" },
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 73-85 / 第 73-85 行

````yaml
          { config: "default", shard: 7, num_shards: 12, runner: "linux.idc.xpu" },
          { config: "default", shard: 8, num_shards: 12, runner: "linux.idc.xpu" },
          { config: "default", shard: 9, num_shards: 12, runner: "linux.idc.xpu" },
          { config: "default", shard: 10, num_shards: 12, runner: "linux.idc.xpu" },
          { config: "default", shard: 11, num_shards: 12, runner: "linux.idc.xpu" },
          { config: "default", shard: 12, num_shards: 12, runner: "linux.idc.xpu" },
        ]}
    secrets: inherit

  linux-noble-xpu-n-py3_10-test:
    name: linux-noble-xpu-n-py3.10
    uses: ./.github/workflows/_xpu-test.yml
    needs: linux-noble-xpu-n-py3_10-build
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_xpu-test.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 86-95 / 第 86-95 行

````yaml
    with:
      build-environment: ${{ needs.linux-noble-xpu-n-py3_10-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-noble-xpu-n-py3_10-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-noble-xpu-n-py3_10-build.outputs.test-matrix }}
    secrets: inherit

  linux-noble-xpu-n-py3_10-client-build:
    name: linux-noble-xpu-n-py3.10-client
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 96-106 / 第 96-106 行

````yaml
    with:
      sync-tag: linux-xpu-n-client-build
      runner_prefix: ${{ needs.get-label-type.outputs.label-type }}
      build-environment: linux-noble-xpu-n-py3.10-client
      docker-image-name: ci-image:pytorch-linux-noble-xpu-n-py3-client
      runner: linux.c7i.12xlarge
      test-matrix: |
        { include: [
          { config: "smoke_xpu", shard: 1, num_shards: 1, runner: "linux.client.xpu" },
        ]}
    secrets: inherit
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 108-116 / 第 108-116 行

````yaml
  linux-noble-xpu-n-py3_10-client-test:
    name: linux-noble-xpu-n-py3.10-client
    uses: ./.github/workflows/_xpu-test.yml
    needs: linux-noble-xpu-n-py3_10-client-build
    with:
      build-environment: ${{ needs.linux-noble-xpu-n-py3_10-client-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-noble-xpu-n-py3_10-client-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-noble-xpu-n-py3_10-client-build.outputs.test-matrix }}
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_xpu-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 118-128 / 第 118-128 行

````yaml
  windows-xpu-n-1-build:
    if: github.repository_owner == 'pytorch'
    name: win-vs2022-xpu-n-1-py3
    uses: ./.github/workflows/_win-build.yml
    with:
      build-environment: win-vs2022-xpu-n-1-py3
      cuda-version: cpu
      use-xpu: true
      xpu-version: '2025.2'
      vc-year: '2022'
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_win-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 130-140 / 第 130-140 行

````yaml
  windows-xpu-n-build:
    if: github.repository_owner == 'pytorch'
    name: win-vs2022-xpu-n-py3
    uses: ./.github/workflows/_win-build.yml
    with:
      build-environment: win-vs2022-xpu-n-py3
      cuda-version: cpu
      use-xpu: true
      xpu-version: '2025.3'
      vc-year: '2022'
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_win-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。


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
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `./.github/workflows/_linux-build.yml`, `./.github/workflows/_xpu-test.yml`, `./.github/workflows/_win-build.yml`
- Inline commands / 内联命令: `]}`
- Environment variables / 环境变量: none
- Named jobs or sections / 命名作业或章节: `on`, `push`, `tags`, `workflow_dispatch`, `schedule`, `concurrency`, `permissions`, `jobs`, `get-label-type`, `with`, ...
