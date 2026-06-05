# inductor-perf-test-nightly-h100.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/inductor-perf-test-nightly-h100.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-13 / 第 1-13 行

````yaml
name: inductor-perf-nightly-h100

on:
  schedule:
    - cron: 15 0 * * 1-6
    - cron: 0 7 * * 0
  workflow_dispatch:
    inputs:
      training:
        description: Run training (on by default)?
        required: false
        type: boolean
        default: true
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 14-28 / 第 14-28 行

````yaml
      inference:
        description: Run inference (on by default)?
        required: false
        type: boolean
        default: true
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

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly-h100.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly-h100.yml` 的仓库自动化行为。

### Lines 29-43 / 第 29-43 行

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
      freezing_cudagraphs:
        description: Run inductor_cudagraphs with freezing for inference?
        required: false
        type: boolean
        default: false
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly-h100.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly-h100.yml` 的仓库自动化行为。

### Lines 44-58 / 第 44-58 行

````yaml
      aotinductor:
        description: Run aot_inductor for inference?
        required: false
        type: boolean
        default: false
      maxautotune:
        description: Run inductor_max_autotune?
        required: false
        type: boolean
        default: false
      deterministic_perf:
        description: Run benchmarks with deterministic mode enabled?
        required: false
        type: boolean
        default: false
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly-h100.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly-h100.yml` 的仓库自动化行为。

### Lines 59-73 / 第 59-73 行

````yaml
      batch_invariant_accuracy:
        description: Run accuracy benchmarks with batch-invariant mode enabled?
        required: false
        type: boolean
        default: false
      benchmark_configs:
        description: The list of configs used the benchmark
        required: false
        type: string
        default: inductor_huggingface_perf_cuda_h100,inductor_timm_perf_cuda_h100,inductor_torchbench_perf_cuda_h100
  pull_request:
    # Changing these files guarantees that this workflow needs to be run
    paths:
      - .github/workflows/inductor-perf-test-nightly-h100.yml
      - .ci/docker/ci_commit_pins/huggingface-requirements.txt
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 75-88 / 第 75-88 行

````yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.sha }}-${{ github.event_name == 'workflow_dispatch' }}-${{ github.event_name == 'schedule' }}
  cancel-in-progress: true

permissions:
  id-token: write
  contents: read
  actions: read

jobs:
  get-label-type:
    name: get-label-type
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
    if: ${{ (github.event_name != 'schedule' || github.repository == 'pytorch/pytorch') && github.repository_owner == 'pytorch' }}
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 89-104 / 第 89-104 行

````yaml
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}
      opt_out_experiments: lf

  build:
    name: build
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      # Use a bigger runner here because CUDA_ARCH 9.0 is only built for H100
      # or newer GPUs, so it doesn't benefit much from existing compiler cache
      # from trunk. Also use a memory-intensive runner here because memory is
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 105-122 / 第 105-122 行

````yaml
      # usually the bottleneck
      runner: linux.12xlarge.memory
      build-environment: linux-jammy-cuda13.0-py3.10-gcc11-sm90
      docker-image-name: ci-image:pytorch-linux-jammy-cuda13.0-cudnn9-py3-gcc11-inductor-benchmarks
      cuda-arch-list: '9.0'
      test-matrix: |
        { include: [
          { config: "inductor_huggingface_perf_cuda_h100", shard: 1, num_shards: 5, runner: "linux.aws.h100" },
          { config: "inductor_huggingface_perf_cuda_h100", shard: 2, num_shards: 5, runner: "linux.aws.h100" },
          { config: "inductor_huggingface_perf_cuda_h100", shard: 3, num_shards: 5, runner: "linux.aws.h100" },
          { config: "inductor_huggingface_perf_cuda_h100", shard: 4, num_shards: 5, runner: "linux.aws.h100" },
          { config: "inductor_huggingface_perf_cuda_h100", shard: 5, num_shards: 5, runner: "linux.aws.h100" },
          { config: "inductor_timm_perf_cuda_h100", shard: 1, num_shards: 7, runner: "linux.aws.h100" },
          { config: "inductor_timm_perf_cuda_h100", shard: 2, num_shards: 7, runner: "linux.aws.h100" },
          { config: "inductor_timm_perf_cuda_h100", shard: 3, num_shards: 7, runner: "linux.aws.h100" },
          { config: "inductor_timm_perf_cuda_h100", shard: 4, num_shards: 7, runner: "linux.aws.h100" },
          { config: "inductor_timm_perf_cuda_h100", shard: 5, num_shards: 7, runner: "linux.aws.h100" },
          { config: "inductor_timm_perf_cuda_h100", shard: 6, num_shards: 7, runner: "linux.aws.h100" },
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly-h100.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly-h100.yml` 的仓库自动化行为。

### Lines 123-136 / 第 123-136 行

````yaml
          { config: "inductor_timm_perf_cuda_h100", shard: 7, num_shards: 7, runner: "linux.aws.h100" },
          { config: "inductor_torchbench_perf_cuda_h100", shard: 1, num_shards: 9, runner: "linux.aws.h100" },
          { config: "inductor_torchbench_perf_cuda_h100", shard: 2, num_shards: 9, runner: "linux.aws.h100" },
          { config: "inductor_torchbench_perf_cuda_h100", shard: 3, num_shards: 9, runner: "linux.aws.h100" },
          { config: "inductor_torchbench_perf_cuda_h100", shard: 4, num_shards: 9, runner: "linux.aws.h100" },
          { config: "inductor_torchbench_perf_cuda_h100", shard: 5, num_shards: 9, runner: "linux.aws.h100" },
          { config: "inductor_torchbench_perf_cuda_h100", shard: 6, num_shards: 9, runner: "linux.aws.h100" },
          { config: "inductor_torchbench_perf_cuda_h100", shard: 7, num_shards: 9, runner: "linux.aws.h100" },
          { config: "inductor_torchbench_perf_cuda_h100", shard: 8, num_shards: 9, runner: "linux.aws.h100" },
          { config: "inductor_torchbench_perf_cuda_h100", shard: 9, num_shards: 9, runner: "linux.aws.h100" },
        ]}
      selected-test-configs: ${{ inputs.benchmark_configs }}
      build-additional-packages: "vision audio fbgemm torchao"
    secrets: inherit
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly-h100.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly-h100.yml` 的仓库自动化行为。

### Lines 138-148 / 第 138-148 行

````yaml
  test-periodically:
    name: test-periodically
    uses: ./.github/workflows/_linux-test.yml
    needs: build
    if: github.event.schedule == '15 0 * * 1-6'
    with:
      build-environment: ${{ needs.build.outputs.build-environment }}
      dashboard-tag: training-true-inference-true-default-true-dynamic-true-cudagraphs-true-cppwrapper-true-aotinductor-true-freezing_cudagraphs-true-deterministic_perf-true
      docker-image: ${{ needs.build.outputs.docker-image }}
      test-matrix: ${{ needs.build.outputs.test-matrix }}
      timeout-minutes: 720
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 149-161 / 第 149-161 行

````yaml
      # disable monitor in perf tests, next step is to enable it
      disable-monitor: false
      monitor-log-interval: 15
      monitor-data-collect-interval: 4
      export-profiler-trace: "1"
      enable-torch-trace: "1"
    secrets: inherit

  test-weekly:
    name: test-weekly
    uses: ./.github/workflows/_linux-test.yml
    needs: build
    if: github.event.schedule == '0 7 * * 0'
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 162-174 / 第 162-174 行

````yaml
    with:
      build-environment: ${{ needs.build.outputs.build-environment }}
      dashboard-tag: training-true-inference-true-default-true-dynamic-true-cudagraphs-true-cppwrapper-true-aotinductor-true-freezing_cudagraphs-true-maxautotune-true-freeze_autotune_cudagraphs-true-deterministic_perf-true
      docker-image: ${{ needs.build.outputs.docker-image }}
      test-matrix: ${{ needs.build.outputs.test-matrix }}
      timeout-minutes: 1440
      # disable monitor in perf tests, next step is to enable it
      disable-monitor: false
      monitor-log-interval: 15
      monitor-data-collect-interval: 4
      export-profiler-trace: "1"
      enable-torch-trace: "1"
    secrets: inherit
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 176-188 / 第 176-188 行

````yaml
  test:
    name: test
    uses: ./.github/workflows/_linux-test.yml
    needs: build
    # The pull_request trigger is used in PR to bump transformers pin which always
    # needs one round of benchmark
    if: ${{ github.event_name == 'workflow_dispatch' || github.event_name == 'pull_request' }}
    with:
      build-environment: ${{ needs.build.outputs.build-environment }}
      dashboard-tag: training-${{ inputs.training || 'true' }}-inference-${{ inputs.inference || 'true' }}-default-${{ inputs.default || 'true' }}-dynamic-${{ inputs.dynamic || 'true' }}-cudagraphs-${{ inputs.cudagraphs || 'true' }}-cppwrapper-${{ inputs.cppwrapper || 'false' }}-aotinductor-${{ inputs.aotinductor || 'false' }}-maxautotune-${{ inputs.maxautotune || 'false' }}-freezing_cudagraphs-${{ inputs.freezing_cudagraphs || 'false' }}-deterministic_perf-${{ inputs.deterministic_perf || 'false' }}-batch_invariant_accuracy-${{ inputs.batch_invariant_accuracy || 'false' }}
      docker-image: ${{ needs.build.outputs.docker-image }}
      test-matrix: ${{ needs.build.outputs.test-matrix }}
      timeout-minutes: 720
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 189-195 / 第 189-195 行

````yaml
      # disable monitor in perf tests for more investigation
      disable-monitor: false
      monitor-log-interval: 15
      monitor-data-collect-interval: 4
      export-profiler-trace: "1"
      enable-torch-trace: "1"
    secrets: inherit
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly-h100.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly-h100.yml` 的仓库自动化行为。


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
- Environment variables / 环境变量: `CUDA_ARCH`, `H100`
- Named jobs or sections / 命名作业或章节: `on`, `schedule`, `workflow_dispatch`, `inputs`, `pull_request`, `paths`, `concurrency`, `permissions`, `jobs`, `get-label-type`, ...
