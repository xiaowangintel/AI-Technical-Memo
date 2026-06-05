# inductor-periodic.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/inductor-periodic.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-15 / 第 1-15 行

````yaml
name: inductor-periodic

on:
  push:
    tags:
      - ciflow/inductor-periodic/*
  workflow_dispatch:
  schedule:
    # Run every 4 hours during the week and every 12 hours on the weekend
    - cron: 45 0,4,8,12,16,20 * * 1-5
    - cron: 45 4,12 * * 0,6

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref_name }}-${{ github.ref_type == 'branch' && github.sha }}-${{ github.event_name == 'workflow_dispatch' && github.run_id }}
  cancel-in-progress: true
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 18-33 / 第 18-33 行

````yaml
permissions:
  id-token: write
  contents: read
  actions: read

jobs:
  get-default-label-prefix:
    name: get-default-label-prefix
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
    if: ${{ (github.event_name != 'schedule' || github.repository == 'pytorch/pytorch') && github.repository_owner == 'pytorch' }}
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}
      opt_out_experiments: lf
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 35-52 / 第 35-52 行

````yaml
  periodic-dynamo-benchmarks-build:
    name: periodic-dynamo-benchmarks-build
    uses: ./.github/workflows/_linux-build.yml
    needs: get-default-label-prefix
    with:
      runner_prefix: "${{ needs.get-default-label-prefix.outputs.label-type }}"
      runner: linux.4xlarge.memory
      build-environment: linux-jammy-cuda13.0-py3.10-gcc11-sm86
      docker-image-name: ci-image:pytorch-linux-jammy-cuda13.0-cudnn9-py3-gcc11-inductor-benchmarks
      cuda-arch-list: '8.0;8.6'
      test-matrix: |
        { include: [
          { config: "dynamo_eager_torchbench", shard: 1, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "dynamo_eager_torchbench", shard: 2, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "dynamo_eager_huggingface", shard: 1, num_shards: 1, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "dynamo_eager_timm", shard: 1, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "dynamo_eager_timm", shard: 2, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "aot_eager_torchbench", shard: 1, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 53-70 / 第 53-70 行

````yaml
          { config: "aot_eager_torchbench", shard: 2, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "aot_eager_huggingface", shard: 1, num_shards: 1, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "aot_eager_timm", shard: 1, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "aot_eager_timm", shard: 2, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "dynamic_aot_eager_torchbench", shard: 1, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "dynamic_aot_eager_torchbench", shard: 2, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "dynamic_aot_eager_huggingface", shard: 1, num_shards: 1, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "dynamic_aot_eager_timm", shard: 1, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "dynamic_aot_eager_timm", shard: 2, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "dynamic_inductor_huggingface", shard: 1, num_shards: 1, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "inductor_huggingface_unbacked_parity", shard: 1, num_shards: 1, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "inductor_huggingface_unbacked_parity", shard: 1, num_shards: 1, runner: "linux.aws.a100" },
          { config: "inductor_huggingface_unbacked_parity", shard: 1, num_shards: 1, runner: "linux.aws.h100" },
          { config: "dynamic_inductor_timm", shard: 1, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "dynamic_inductor_timm", shard: 2, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "dynamic_inductor_torchbench", shard: 1, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "dynamic_inductor_torchbench", shard: 2, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "aot_inductor_huggingface", shard: 1, num_shards: 1, runner: "linux.aws.a100" },
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-periodic.yml`.
- CN: 该部分描述 `.github/workflows/inductor-periodic.yml` 的仓库自动化行为。

### Lines 71-87 / 第 71-87 行

````yaml
          { config: "aot_inductor_timm", shard: 1, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "aot_inductor_timm", shard: 2, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "aot_inductor_torchbench", shard: 1, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
          { config: "aot_inductor_torchbench", shard: 2, num_shards: 2, runner: "linux.g5.4xlarge.nvidia.gpu" },
        ]}
      build-additional-packages: "vision audio fbgemm torchao"
    secrets: inherit

  periodic-dynamo-benchmarks-test:
    name: periodic-dynamo-benchmarks-test
    uses: ./.github/workflows/_linux-test.yml
    needs: periodic-dynamo-benchmarks-build
    with:
      build-environment: ${{ needs.periodic-dynamo-benchmarks-build.outputs.build-environment }}
      docker-image: ${{ needs.periodic-dynamo-benchmarks-build.outputs.docker-image }}
      test-matrix: ${{ needs.periodic-dynamo-benchmarks-build.outputs.test-matrix }}
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 89-106 / 第 89-106 行

````yaml
  rocm-periodic-dynamo-benchmarks-build:
    if: github.repository_owner == 'pytorch'
    name: rocm-periodic-dynamo-benchmarks-build
    uses: ./.github/workflows/_linux-build.yml
    with:
      build-environment: linux-jammy-rocm-py3.10-mi355
      docker-image-name: ci-image:pytorch-linux-jammy-rocm-n-py3-benchmarks
      test-matrix: |
        { include: [
          { config: "dynamo_eager_torchbench", shard: 1, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "dynamo_eager_torchbench", shard: 2, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "dynamo_eager_huggingface", shard: 1, num_shards: 1, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "dynamo_eager_timm", shard: 1, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "dynamo_eager_timm", shard: 2, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "aot_eager_torchbench", shard: 1, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "aot_eager_torchbench", shard: 2, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "aot_eager_huggingface", shard: 1, num_shards: 1, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "aot_eager_timm", shard: 1, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 107-124 / 第 107-124 行

````yaml
          { config: "aot_eager_timm", shard: 2, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "dynamic_aot_eager_torchbench", shard: 1, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "dynamic_aot_eager_torchbench", shard: 2, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "dynamic_aot_eager_huggingface", shard: 1, num_shards: 1, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "dynamic_aot_eager_timm", shard: 1, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "dynamic_aot_eager_timm", shard: 2, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "dynamic_inductor_huggingface", shard: 1, num_shards: 1, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "dynamic_inductor_timm", shard: 1, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "dynamic_inductor_timm", shard: 2, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "dynamic_inductor_torchbench", shard: 1, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "dynamic_inductor_torchbench", shard: 2, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "aot_inductor_huggingface", shard: 1, num_shards: 1, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "aot_inductor_timm", shard: 1, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "aot_inductor_timm", shard: 2, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "aot_inductor_torchbench", shard: 1, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "aot_inductor_torchbench", shard: 2, num_shards: 2, runner: "linux.rocm.gpu.gfx950.1" },
        ]}
    secrets: inherit
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-periodic.yml`.
- CN: 该部分描述 `.github/workflows/inductor-periodic.yml` 的仓库自动化行为。

### Lines 126-140 / 第 126-140 行

````yaml
  rocm-periodic-dynamo-benchmarks-test:
    name: rocm-periodic-dynamo-benchmarks-test
    uses: ./.github/workflows/_rocm-test.yml
    needs: rocm-periodic-dynamo-benchmarks-build
    with:
      build-environment: ${{ needs.rocm-periodic-dynamo-benchmarks-build.outputs.build-environment }}
      docker-image: ${{ needs.rocm-periodic-dynamo-benchmarks-build.outputs.docker-image }}
      test-matrix: ${{ needs.rocm-periodic-dynamo-benchmarks-build.outputs.test-matrix }}
    secrets: inherit

  inductor-smoke-build:
    name: inductor-smoke-build
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-default-label-prefix
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_rocm-test.yml`, `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 141-157 / 第 141-157 行

````yaml
    with:
      runner_prefix: "${{ needs.get-default-label-prefix.outputs.label-type }}"
      runner: linux.4xlarge.memory
      build-environment: linux-jammy-cuda13.0-py3.10-gcc11-sm80
      docker-image-name: ci-image:pytorch-linux-jammy-cuda13.0-cudnn9-py3-gcc11-inductor-benchmarks
      cuda-arch-list: '8.0'
      test-matrix: |
        { include: [
          { config: "inductor_torchbench_smoketest_perf", shard: 1, num_shards: 1, runner: "linux.aws.a100" },
        ]}
      build-additional-packages: "vision audio fbgemm torchao"
    secrets: inherit

  inductor-smoke-test:
    name: inductor-smoke-test
    uses: ./.github/workflows/_linux-test.yml
    needs: inductor-smoke-build
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 158-168 / 第 158-168 行

````yaml
    with:
      build-environment: ${{ needs.inductor-smoke-build.outputs.build-environment }}
      docker-image: ${{ needs.inductor-smoke-build.outputs.docker-image }}
      test-matrix: ${{ needs.inductor-smoke-build.outputs.test-matrix }}
    secrets: inherit


  periodic-dynamo-benchmarks-cpu-build:
    name: periodic-dynamo-benchmarks-cpu-build
    uses: ./.github/workflows/_linux-build.yml
    needs: get-default-label-prefix
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 169-186 / 第 169-186 行

````yaml
    with:
      build-environment: linux-jammy-py3.10-gcc11-build
      docker-image-name: ci-image:pytorch-linux-jammy-py3-gcc11-inductor-benchmarks
      runner_prefix: "${{ needs.get-default-label-prefix.outputs.label-type }}"
      test-matrix: |
        { include: [
          { config: "inductor_avx2", shard: 1, num_shards: 2, runner: "linux.2xlarge.avx2" },
          { config: "inductor_avx2", shard: 2, num_shards: 2, runner: "linux.2xlarge.avx2" },
          { config: "dynamic_cpu_inductor_huggingface", shard: 1, num_shards: 1, runner: "linux.8xlarge.amx" },
          { config: "dynamic_cpu_inductor_timm", shard: 1, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "dynamic_cpu_inductor_timm", shard: 2, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "dynamic_cpu_inductor_huggingface", shard: 1, num_shards: 1, runner: "linux.8xlarge.amx" },
          { config: "dynamic_cpu_inductor_timm", shard: 1, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "dynamic_cpu_inductor_timm", shard: 2, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "dynamic_cpu_inductor_torchbench", shard: 1, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "dynamic_cpu_inductor_torchbench", shard: 2, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "cpu_inductor_freezing_avx2_huggingface", shard: 1, num_shards: 1, runner: "linux.10xlarge.avx2" },
          { config: "cpu_inductor_freezing_avx2_torchbench", shard: 1, num_shards: 2, runner: "linux.10xlarge.avx2" },
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 187-204 / 第 187-204 行

````yaml
          { config: "cpu_inductor_freezing_avx2_torchbench", shard: 2, num_shards: 2, runner: "linux.10xlarge.avx2" },
          { config: "cpu_inductor_freezing_avx2_timm", shard: 1, num_shards: 2, runner: "linux.10xlarge.avx2" },
          { config: "cpu_inductor_freezing_avx2_timm", shard: 2, num_shards: 2, runner: "linux.10xlarge.avx2" },
          { config: "cpu_inductor_freezing_huggingface", shard: 1, num_shards: 1, runner: "linux.8xlarge.amx" },
          { config: "cpu_inductor_freezing_timm", shard: 1, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "cpu_inductor_freezing_timm", shard: 2, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "cpu_inductor_freezing_torchbench", shard: 1, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "cpu_inductor_freezing_torchbench", shard: 2, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "cpu_inductor_amp_freezing_huggingface", shard: 1, num_shards: 1, runner: "linux.8xlarge.amx" },
          { config: "cpu_inductor_amp_freezing_timm", shard: 1, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "cpu_inductor_amp_freezing_timm", shard: 2, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "cpu_inductor_amp_freezing_torchbench", shard: 1, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "cpu_inductor_amp_freezing_torchbench", shard: 2, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "cpu_aot_inductor_freezing_huggingface", shard: 1, num_shards: 1, runner: "linux.8xlarge.amx" },
          { config: "cpu_aot_inductor_freezing_timm", shard: 1, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "cpu_aot_inductor_freezing_timm", shard: 2, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "cpu_aot_inductor_freezing_torchbench", shard: 1, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "cpu_aot_inductor_freezing_torchbench", shard: 2, num_shards: 2, runner: "linux.8xlarge.amx" },
````

- EN: This section describes repository automation behavior for `.github/workflows/inductor-periodic.yml`.
- CN: 该部分描述 `.github/workflows/inductor-periodic.yml` 的仓库自动化行为。

### Lines 205-218 / 第 205-218 行

````yaml
          { config: "cpu_aot_inductor_amp_freezing_torchbench", shard: 1, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "cpu_aot_inductor_amp_freezing_torchbench", shard: 2, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "dynamic_cpu_aot_inductor_freezing_torchbench", shard: 1, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "dynamic_cpu_aot_inductor_freezing_torchbench", shard: 2, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "dynamic_cpu_aot_inductor_amp_freezing_torchbench", shard: 1, num_shards: 2, runner: "linux.8xlarge.amx" },
          { config: "dynamic_cpu_aot_inductor_amp_freezing_torchbench", shard: 2, num_shards: 2, runner: "linux.8xlarge.amx" },
        ]}
      build-additional-packages: "vision audio torchao"
    secrets: inherit

  periodic-dynamo-benchmarks-cpu-test:
    name: periodic-dynamo-benchmarks-cpu-test
    uses: ./.github/workflows/_linux-test.yml
    needs: periodic-dynamo-benchmarks-cpu-build
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 219-223 / 第 219-223 行

````yaml
    with:
      build-environment: ${{ needs.periodic-dynamo-benchmarks-cpu-build.outputs.build-environment }}
      docker-image: ${{ needs.periodic-dynamo-benchmarks-cpu-build.outputs.docker-image }}
      test-matrix: ${{ needs.periodic-dynamo-benchmarks-cpu-build.outputs.test-matrix }}
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
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `./.github/workflows/_linux-build.yml`, `./.github/workflows/_linux-test.yml`, `./.github/workflows/_rocm-test.yml`
- Inline commands / 内联命令: `]}`
- Environment variables / 环境变量: none
- Named jobs or sections / 命名作业或章节: `on`, `push`, `tags`, `workflow_dispatch`, `schedule`, `concurrency`, `permissions`, `jobs`, `get-default-label-prefix`, `with`, ...
