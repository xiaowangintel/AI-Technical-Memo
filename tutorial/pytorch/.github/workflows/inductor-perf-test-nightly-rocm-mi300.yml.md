# inductor-perf-test-nightly-rocm-mi300.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/inductor-perf-test-nightly-rocm-mi300.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-10 / 第 1-10 行

````yaml
name: inductor-perf-nightly-rocm-mi300

on:
  push:
    tags:
      - ciflow/inductor-perf-test-nightly-rocm-mi300/*
  schedule:
    - cron: 15 0 * * *
  workflow_dispatch:
    inputs:
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 11-20 / 第 11-20 行

````yaml
      training:
        description: Run training (on by default)?
        required: false
        type: boolean
        default: true
      inference:
        description: Run inference (on by default)?
        required: false
        type: boolean
        default: true
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly-rocm-mi300.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly-rocm-mi300.yml` 的仓库自动化行为。

### Lines 21-30 / 第 21-30 行

````yaml
      default:
        description: Run inductor_default?
        required: false
        type: boolean
        default: false
      dynamic:
        description: Run inductor_dynamic_shapes?
        required: false
        type: boolean
        default: false
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly-rocm-mi300.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly-rocm-mi300.yml` 的仓库自动化行为。

### Lines 31-40 / 第 31-40 行

````yaml
      cppwrapper:
        description: Run inductor_cpp_wrapper?
        required: false
        type: boolean
        default: false
      cudagraphs:
        description: Run inductor_cudagraphs?
        required: false
        type: boolean
        default: true
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly-rocm-mi300.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly-rocm-mi300.yml` 的仓库自动化行为。

### Lines 41-50 / 第 41-50 行

````yaml
      freezing_cudagraphs:
        description: Run inductor_cudagraphs with freezing for inference?
        required: false
        type: boolean
        default: false
      aotinductor:
        description: Run aot_inductor for inference?
        required: false
        type: boolean
        default: false
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly-rocm-mi300.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly-rocm-mi300.yml` 的仓库自动化行为。

### Lines 51-60 / 第 51-60 行

````yaml
      maxautotune:
        description: Run inductor_max_autotune?
        required: false
        type: boolean
        default: false
      benchmark_configs:
        description: The list of configs used the benchmark
        required: false
        type: string
        default: inductor_huggingface_perf_rocm_mi300,inductor_timm_perf_rocm_mi300,inductor_torchbench_perf_rocm_mi300
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly-rocm-mi300.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly-rocm-mi300.yml` 的仓库自动化行为。

### Lines 62-71 / 第 62-71 行

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

### Lines 72-81 / 第 72-81 行

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
      opt_out_experiments: lf
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 83-96 / 第 83-96 行

````yaml
  linux-jammy-rocm-py3_10-inductor-benchmark-build:
    if: github.repository_owner == 'pytorch'
    name: linux-jammy-rocm-py3.10-mi300
    uses: ./.github/workflows/_linux-build.yml
    with:
      build-environment: linux-jammy-rocm-py3.10-mi300
      docker-image-name: ci-image:pytorch-linux-jammy-rocm-n-py3-benchmarks
      test-matrix: |
        { include: [
          { config: "inductor_huggingface_perf_rocm_mi300", shard: 1, num_shards: 5, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_huggingface_perf_rocm_mi300", shard: 2, num_shards: 5, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_huggingface_perf_rocm_mi300", shard: 3, num_shards: 5, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_huggingface_perf_rocm_mi300", shard: 4, num_shards: 5, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_huggingface_perf_rocm_mi300", shard: 5, num_shards: 5, runner: "linux.rocm.gpu.gfx942.1" },
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 97-110 / 第 97-110 行

````yaml
          { config: "inductor_timm_perf_rocm_mi300", shard: 1, num_shards: 7, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_timm_perf_rocm_mi300", shard: 2, num_shards: 7, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_timm_perf_rocm_mi300", shard: 3, num_shards: 7, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_timm_perf_rocm_mi300", shard: 4, num_shards: 7, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_timm_perf_rocm_mi300", shard: 5, num_shards: 7, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_timm_perf_rocm_mi300", shard: 6, num_shards: 7, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_timm_perf_rocm_mi300", shard: 7, num_shards: 7, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_torchbench_perf_rocm_mi300", shard: 1, num_shards: 9, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_torchbench_perf_rocm_mi300", shard: 2, num_shards: 9, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_torchbench_perf_rocm_mi300", shard: 3, num_shards: 9, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_torchbench_perf_rocm_mi300", shard: 4, num_shards: 9, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_torchbench_perf_rocm_mi300", shard: 5, num_shards: 9, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_torchbench_perf_rocm_mi300", shard: 6, num_shards: 9, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_torchbench_perf_rocm_mi300", shard: 7, num_shards: 9, runner: "linux.rocm.gpu.gfx942.1" },
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly-rocm-mi300.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly-rocm-mi300.yml` 的仓库自动化行为。

### Lines 111-119 / 第 111-119 行

````yaml
          { config: "inductor_torchbench_perf_rocm_mi300", shard: 8, num_shards: 9, runner: "linux.rocm.gpu.gfx942.1" },
          { config: "inductor_torchbench_perf_rocm_mi300", shard: 9, num_shards: 9, runner: "linux.rocm.gpu.gfx942.1" },
        ]}
    secrets: inherit

  linux-jammy-rocm-py3_10-inductor-benchmark-test:
    name: linux-jammy-rocm-py3.10-mi300
    uses: ./.github/workflows/_rocm-test.yml
    needs: linux-jammy-rocm-py3_10-inductor-benchmark-build
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_rocm-test.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 120-125 / 第 120-125 行

````yaml
    with:
      build-environment: ${{ needs.linux-jammy-rocm-py3_10-inductor-benchmark-build.outputs.build-environment }}
      dashboard-tag: training-true-inference-true-default-true-dynamic-true-cudagraphs-true-cppwrapper-true-aotinductor-true-freezing_cudagraphs-true
      docker-image: ${{ needs.linux-jammy-rocm-py3_10-inductor-benchmark-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-rocm-py3_10-inductor-benchmark-build.outputs.test-matrix }}
      timeout-minutes: 720
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 126-130 / 第 126-130 行

````yaml
      # Disable monitor in perf tests for more investigation
      disable-monitor: true
      monitor-log-interval: 10
      monitor-data-collect-interval: 2
    secrets: inherit
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly-rocm-mi300.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly-rocm-mi300.yml` 的仓库自动化行为。


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
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `./.github/workflows/_linux-build.yml`, `./.github/workflows/_rocm-test.yml`
- Inline commands / 内联命令: `]}`
- Environment variables / 环境变量: none
- Named jobs or sections / 命名作业或章节: `on`, `push`, `tags`, `schedule`, `workflow_dispatch`, `inputs`, `concurrency`, `permissions`, `jobs`, `get-label-type`, ...
