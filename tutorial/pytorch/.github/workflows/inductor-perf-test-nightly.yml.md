# inductor-perf-test-nightly.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/inductor-perf-test-nightly.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-13 / 第 1-13 行

````yaml
name: inductor-A100-perf-nightly

on:
  schedule:
    - cron: 0 7 * * 1-6
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

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly.yml` 的仓库自动化行为。

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

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly.yml` 的仓库自动化行为。

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
      benchmark_configs:
        description: The list of configs used the benchmark
        required: false
        type: string
        default: inductor_huggingface_perf,inductor_timm_perf,inductor_torchbench_perf,cachebench
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly.yml` 的仓库自动化行为。

### Lines 60-73 / 第 60-73 行

````yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref_name }}-${{ github.ref_type == 'branch' && github.sha }}-${{ github.event_name == 'workflow_dispatch' }}-${{ github.event_name == 'schedule' }}
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

### Lines 74-86 / 第 74-86 行

````yaml
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}
      opt_out_experiments: lf

  build:
    name: cuda13.0-py3.10-gcc11-sm80
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 87-104 / 第 87-104 行

````yaml
      # Every bit to make perf run faster helps
      runner: linux.12xlarge.memory
      build-environment: linux-jammy-cuda13.0-py3.10-gcc11-sm80
      docker-image-name: ci-image:pytorch-linux-jammy-cuda13.0-cudnn9-py3-gcc11-inductor-benchmarks
      cuda-arch-list: '8.0'
      test-matrix: |
        { include: [
          { config: "inductor_huggingface_perf", shard: 1, num_shards: 5, runner: "linux.aws.a100" },
          { config: "inductor_huggingface_perf", shard: 2, num_shards: 5, runner: "linux.aws.a100" },
          { config: "inductor_huggingface_perf", shard: 3, num_shards: 5, runner: "linux.aws.a100" },
          { config: "inductor_huggingface_perf", shard: 4, num_shards: 5, runner: "linux.aws.a100" },
          { config: "inductor_huggingface_perf", shard: 5, num_shards: 5, runner: "linux.aws.a100" },
          { config: "inductor_timm_perf", shard: 1, num_shards: 6, runner: "linux.aws.a100" },
          { config: "inductor_timm_perf", shard: 2, num_shards: 6, runner: "linux.aws.a100" },
          { config: "inductor_timm_perf", shard: 3, num_shards: 6, runner: "linux.aws.a100" },
          { config: "inductor_timm_perf", shard: 4, num_shards: 6, runner: "linux.aws.a100" },
          { config: "inductor_timm_perf", shard: 5, num_shards: 6, runner: "linux.aws.a100" },
          { config: "inductor_timm_perf", shard: 6, num_shards: 6, runner: "linux.aws.a100" },
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly.yml` 的仓库自动化行为。

### Lines 105-116 / 第 105-116 行

````yaml
          { config: "inductor_torchbench_perf", shard: 1, num_shards: 6, runner: "linux.aws.a100" },
          { config: "inductor_torchbench_perf", shard: 2, num_shards: 6, runner: "linux.aws.a100" },
          { config: "inductor_torchbench_perf", shard: 3, num_shards: 6, runner: "linux.aws.a100" },
          { config: "inductor_torchbench_perf", shard: 4, num_shards: 6, runner: "linux.aws.a100" },
          { config: "inductor_torchbench_perf", shard: 5, num_shards: 6, runner: "linux.aws.a100" },
          { config: "inductor_torchbench_perf", shard: 6, num_shards: 6, runner: "linux.aws.a100" },
          { config: "cachebench", shard: 1, num_shards: 2, runner: "linux.aws.a100" },
          { config: "cachebench", shard: 2, num_shards: 2, runner: "linux.aws.a100" },
        ]}
      selected-test-configs: ${{ inputs.benchmark_configs }}
      build-additional-packages: "vision audio fbgemm torchao"
    secrets: inherit
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-perf-test-nightly.yml`.
- CN: 该部分描述 `.github/workflows/inductor-perf-test-nightly.yml` 的仓库自动化行为。

### Lines 118-132 / 第 118-132 行

````yaml
  test-nightly:
    name: cuda13.0-py3.10-gcc11-sm80
    uses: ./.github/workflows/_linux-test.yml
    needs: build
    if: github.event.schedule == '0 7 * * 1-6'
    with:
      build-environment: ${{ needs.build.outputs.build-environment }}
      dashboard-tag: training-true-inference-true-default-true-dynamic-true-cudagraphs-true-cppwrapper-true-aotinductor-true-freezing_cudagraphs-true
      docker-image: ${{ needs.build.outputs.docker-image }}
      test-matrix: ${{ needs.build.outputs.test-matrix }}
      timeout-minutes: 720
      disable-monitor: false
      monitor-log-interval: 15
      monitor-data-collect-interval: 4
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 134-149 / 第 134-149 行

````yaml
  test-weekly:
    name: cuda13.0-py3.10-gcc11-sm80
    uses: ./.github/workflows/_linux-test.yml
    needs: build
    if: github.event.schedule == '0 7 * * 0'
    with:
      build-environment: ${{ needs.build.outputs.build-environment }}
      dashboard-tag: training-true-inference-true-default-true-dynamic-true-cudagraphs-true-cppwrapper-true-aotinductor-true-freezing_cudagraphs-true-maxautotune-true-freeze_autotune_cudagraphs-true
      docker-image: ${{ needs.build.outputs.docker-image }}
      test-matrix: ${{ needs.build.outputs.test-matrix }}
      timeout-minutes: 1440
      # disable monitor in perf tests, next step is to enable it
      disable-monitor: false
      monitor-log-interval: 15
      monitor-data-collect-interval: 4
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 151-165 / 第 151-165 行

````yaml
  test:
    name: cuda13.0-py3.10-gcc11-sm80
    uses: ./.github/workflows/_linux-test.yml
    needs: build
    if: github.event_name == 'workflow_dispatch'
    with:
      build-environment: ${{ needs.build.outputs.build-environment }}
      dashboard-tag: training-${{ inputs.training }}-inference-${{ inputs.inference }}-default-${{ inputs.default }}-dynamic-${{ inputs.dynamic }}-cudagraphs-${{ inputs.cudagraphs }}-cppwrapper-${{ inputs.cppwrapper }}-aotinductor-${{ inputs.aotinductor }}-maxautotune-${{ inputs.maxautotune }}-freezing_cudagraphs-${{ inputs.freezing_cudagraphs }}
      docker-image: ${{ needs.build.outputs.docker-image }}
      test-matrix: ${{ needs.build.outputs.test-matrix }}
      timeout-minutes: 720
      disable-monitor: false
      monitor-log-interval: 15
      monitor-data-collect-interval: 4
    secrets: inherit
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。


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
- Environment variables / 环境变量: `A100`
- Named jobs or sections / 命名作业或章节: `on`, `schedule`, `workflow_dispatch`, `inputs`, `concurrency`, `permissions`, `jobs`, `get-label-type`, `with`, `build`, ...
