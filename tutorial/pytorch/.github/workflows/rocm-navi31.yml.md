# rocm-navi31.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/rocm-navi31.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-12 / 第 1-12 行

````yaml
name: rocm-navi31

on:
  push:
    tags:
      - ciflow/rocm-navi31/*
  workflow_dispatch:
  schedule:
    # We have several schedules so jobs can check github.event.schedule to activate only for a fraction of the runs.
    # Also run less frequently on weekends.
    - cron: 45 */2 * * 1-5
    - cron: 45 4,12 * * 0,6
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 14-23 / 第 14-23 行

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
  target-determination:
    if: github.repository_owner == 'pytorch'
    name: before-test
    uses: ./.github/workflows/target_determination.yml

  get-label-type:
    name: get-label-type
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
    if: ${{ (github.event_name != 'schedule' || github.repository == 'pytorch/pytorch') && github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/target_determination.yml`, `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 33-43 / 第 33-43 行

````yaml
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}

  linux-jammy-rocm-py3_10-build:
    if: ${{ (github.event_name != 'schedule' || github.repository == 'pytorch/pytorch') && github.repository_owner == 'pytorch' }}
    name: linux-jammy-rocm-py3.10-navi31
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 44-54 / 第 44-54 行

````yaml
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-rocm-py3.10-navi31
      docker-image-name: ci-image:pytorch-linux-jammy-rocm-n-py3
      sync-tag: rocm-build
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 2, runner: "linux.rocm.gpu.gfx1100" },
          { config: "default", shard: 2, num_shards: 2, runner: "linux.rocm.gpu.gfx1100" },
        ]}
    secrets: inherit
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 56-69 / 第 56-69 行

````yaml
  linux-jammy-rocm-py3_10-test:
    name: linux-jammy-rocm-py3.10-navi31
    uses: ./.github/workflows/_rocm-test.yml
    needs:
      - linux-jammy-rocm-py3_10-build
      - target-determination
    with:
      build-environment: ${{ needs.linux-jammy-rocm-py3_10-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-rocm-py3_10-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-rocm-py3_10-build.outputs.test-matrix }}
      tests-to-include: >-
         test_autograd inductor/test_torchinductor inductor/test_kernel_benchmark
         inductor/test_pad_mm inductor/test_benchmark_fusion inductor/test_aot_inductor
         inductor/test_torchinductor inductor/test_decompose_mem_bound_mm
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_rocm-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 70-71 / 第 70-71 行

````yaml
         inductor/test_flex_attention inductor/test_max_autotune
    secrets: inherit
````

- EN: This section describes repository automation behavior for `.github/workflows/rocm-navi31.yml`.
- CN: 该部分描述 `.github/workflows/rocm-navi31.yml` 的仓库自动化行为。


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
- Inline commands / 内联命令: `]}`, `test_autograd`, `inductor/test_pad_mm`, `inductor/test_torchinductor`, `inductor/test_flex_attention`
- Environment variables / 环境变量: none
- Named jobs or sections / 命名作业或章节: `on`, `push`, `tags`, `workflow_dispatch`, `schedule`, `concurrency`, `permissions`, `jobs`, `target-determination`, `get-label-type`, ...
