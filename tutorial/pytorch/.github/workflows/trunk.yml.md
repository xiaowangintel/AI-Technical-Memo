# trunk.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/trunk.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-23 / 第 1-23 行

````yaml
name: trunk

on:
  push:
    branches:
      - main
      - release/*
      - landchecks/*
    tags:
      - ciflow/trunk/*
  workflow_dispatch:
    inputs:
      jobs-to-include:
        description: "Space-separated list of job display names to run (empty = all)"
        required: false
        default: ""
        type: string
      tests-to-include:
        description: "Space-separated tests to include (passed to test runner; empty = default)"
        required: false
        default: ""
        type: string
  schedule:
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 24-42 / 第 24-42 行

````yaml
    - cron: 29 8 * * *  # about 1:29am PDT

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref_name }}-${{ github.ref_type == 'branch' && github.sha }}-${{ github.event_name == 'workflow_dispatch' && github.run_id }}-${{ github.event_name == 'schedule' }}
  cancel-in-progress: true

permissions:
  id-token: write
  contents: read
  actions: read

jobs:
  # See job-filter.yml for rules on adding job filter conditions
  job-filter:
    if: github.repository_owner == 'pytorch'
    name: job-filter
    uses: ./.github/workflows/job-filter.yml
    with:
      jobs-to-include: ${{ github.event.inputs.jobs-to-include || '' }}
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; reuses actions and step building blocks such as `./.github/workflows/job-filter.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 44-63 / 第 44-63 行

````yaml
  llm-td:
    if: github.repository_owner == 'pytorch'
    name: before-test
    uses: ./.github/workflows/llm_td_retrieval.yml

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
      check_experiments: arc,lf
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `./.github/workflows/llm_td_retrieval.yml`, `./.github/workflows/target_determination.yml`, `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 65-86 / 第 65-86 行

````yaml
  libtorch-linux-jammy-cuda12_8-py3_10-gcc11-debug-build:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' libtorch-linux-jammy-cuda12.8-py3.10-gcc11-debug ') }}
    name: libtorch-linux-jammy-cuda12.8-py3.10-gcc11-debug
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
    with:
      build-environment: libtorch-linux-jammy-cuda12.8-py3.10-gcc11
      docker-image-name: ci-image:pytorch-linux-jammy-cuda12.8-cudnn9-py3-gcc11
      build-generates-artifacts: false
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runner: "linux.r7i.4xlarge"
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 1 },
        ]}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: gcc11
      cuda-version: "12.8"
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 88-111 / 第 88-111 行

````yaml
  linux-jammy-cuda12_8-py3_10-gcc11-build:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-cuda12.8-py3.10-gcc11 ') }}
    name: linux-jammy-cuda12.8-py3.10-gcc11
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-cuda12.8-py3.10-gcc11
      docker-image-name: ci-image:pytorch-linux-jammy-cuda12.8-cudnn9-py3-gcc11
      cuda-arch-list: '7.5 8.9'
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu" },
          { config: "default", shard: 2, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu" },
          { config: "default", shard: 3, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu" },
          { config: "default", shard: 4, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu" },
          { config: "default", shard: 5, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu" },
          { config: "distributed", shard: 1, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g4dn.12xlarge.nvidia.gpu" },
          { config: "distributed", shard: 2, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g4dn.12xlarge.nvidia.gpu" },
          { config: "distributed", shard: 3, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g4dn.12xlarge.nvidia.gpu" },
          { config: "libtorch_agnostic_targetting", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g4dn.metal.nvidia.gpu" },
        ]}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 112-129 / 第 112-129 行

````yaml
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: gcc11
      cuda-version: "12.8"
    secrets: inherit

  # CUDA 12.8 GPU tests moved to periodic.yml to reduce per-commit compute.
  # CUDA 13.0 (the newer shipping version) remains per-commit for forward-looking coverage.
  # The 12.8 build job is kept because cross-compile-linux-test depends on it.
  # See P2188981399 for the full CI workflow analysis.

  linux-jammy-cuda13_0-py3_10-gcc11-build:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-cuda13.0-py3.10-gcc11 ') || contains(needs.job-filter.outputs.jobs, ' cross-compile-linux-test-cuda13 ') }}
    name: linux-jammy-cuda13.0-py3.10-gcc11
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 130-145 / 第 130-145 行

````yaml
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-cuda13.0-py3.10-gcc11
      docker-image-name: ci-image:pytorch-linux-jammy-cuda13.0-cudnn9-py3-gcc11
      cuda-arch-list: '7.5 8.9'
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu" },
          { config: "default", shard: 2, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu" },
          { config: "default", shard: 3, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu" },
          { config: "default", shard: 4, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu" },
          { config: "default", shard: 5, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu" },
          { config: "distributed", shard: 1, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g4dn.12xlarge.nvidia.gpu" },
          { config: "distributed", shard: 2, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g4dn.12xlarge.nvidia.gpu" },
          { config: "distributed", shard: 3, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g4dn.12xlarge.nvidia.gpu" },
          { config: "pr_time_benchmarks", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g4dn.metal.nvidia.gpu" },
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 146-164 / 第 146-164 行

````yaml
          # Test cross-compiled models with Windows libs extracted from wheel (CUDA 13.0)
          { config: "aoti_cross_compile_for_windows", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g6.4xlarge.experimental.nvidia.gpu", win_torch_wheel_artifact: "win-vs2022-cuda13.0-py3" },
        ]}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: gcc11
      cuda-version: "13.0"
    secrets: inherit

  linux-jammy-cuda13_0-py3_10-gcc11-test:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-cuda13.0-py3.10-gcc11 ') }}
    name: linux-jammy-cuda13.0-py3.10-gcc11
    uses: ./.github/workflows/_linux-test.yml
    needs:
      - linux-jammy-cuda13_0-py3_10-gcc11-build
      - win-vs2022-cuda13_0-py3-build
      - target-determination
      - job-filter
      - get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 165-185 / 第 165-185 行

````yaml
    with:
      timeout-minutes: 360
      build-environment: linux-jammy-cuda13.0-py3.10-gcc11
      docker-image: ${{ needs.linux-jammy-cuda13_0-py3_10-gcc11-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-cuda13_0-py3_10-gcc11-build.outputs.test-matrix }}
      tests-to-include: ${{ github.event.inputs.tests-to-include || '' }}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: gcc11
      cuda-version: "13.0"
    secrets: inherit

  # no-ops builds test USE_PER_OPERATOR_HEADERS=0 where ATen/ops is not generated
  # CUDA 12.8 no-ops build moved to periodic (only 13.0 remains per-commit)
  linux-jammy-cuda13_0-py3_10-gcc11-no-ops-build:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-cuda13.0-py3.10-gcc11-no-ops ') }}
    name: linux-jammy-cuda13.0-py3.10-gcc11-no-ops
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 186-205 / 第 186-205 行

````yaml
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-cuda13.0-py3.10-gcc11-no-ops
      cuda-arch-list: '7.5 8.9'
      docker-image-name: ci-image:pytorch-linux-jammy-cuda13.0-cudnn9-py3-gcc11
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 1 },
        ]}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: gcc11
      cuda-version: "13.0"
    secrets: inherit

  macos-py3-arm64-build:
    if: ${{ github.repository_owner == 'pytorch' && (needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' macos-py3-arm64 ')) }}
    name: macos-py3-arm64
    uses: ./.github/workflows/_mac-build.yml
    needs: job-filter
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_mac-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 206-228 / 第 206-228 行

````yaml
    with:
      sync-tag: macos-py3-arm64-build
      build-environment: macos-py3-arm64
      runner-type: macos-m1-stable
      build-generates-artifacts: true
      # To match the one pre-installed in the m1 runners
      python-version: 3.12.7
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 3, runner: "macos-m1-stable" },
          { config: "default", shard: 2, num_shards: 3, runner: "macos-m1-stable" },
          { config: "default", shard: 3, num_shards: 3, runner: "macos-m1-stable" },
          { config: "mps", shard: 1, num_shards: 1, runner: "macos-m1-14" },
          { config: "mps", shard: 1, num_shards: 1, runner: "macos-m2-15" },
          { config: "openreg", shard: 1, num_shards: 1, runner: "macos-m1-stable" },
        ]}
    secrets: inherit

  macos-py3-arm64-test:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' macos-py3-arm64 ') }}
    name: macos-py3-arm64
    uses: ./.github/workflows/_mac-test.yml
    needs:
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_mac-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 229-248 / 第 229-248 行

````yaml
      - macos-py3-arm64-build
      - target-determination
      - job-filter
    with:
      build-environment: ${{ needs.macos-py3-arm64-build.outputs.build-environment }}
      # Same as the build job
      python-version: 3.12.7
      test-matrix: ${{ needs.macos-py3-arm64-build.outputs.test-matrix }}
      disable-monitor: false
    secrets: inherit

  # NB: Windows runners are not available in OSDC, so we run this on Meta account
  # for now until we get OSDC deployed to LF account
  win-vs2022-cpu-py3-build:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' win-vs2022-cpu-py3 ') }}
    name: win-vs2022-cpu-py3
    uses: ./.github/workflows/_win-build.yml
    needs:
      - get-label-type
      - job-filter
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_win-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 249-270 / 第 249-270 行

````yaml
    with:
      build-environment: win-vs2022-cpu-py3
      cuda-version: cpu
      runner: windows.4xlarge.nonephemeral
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 4, runner: "windows.4xlarge.nonephemeral" },
          { config: "default", shard: 2, num_shards: 4, runner: "windows.4xlarge.nonephemeral" },
          { config: "default", shard: 3, num_shards: 4, runner: "windows.4xlarge.nonephemeral" },
          { config: "default", shard: 4, num_shards: 4, runner: "windows.4xlarge.nonephemeral" },
          { config: "openreg", shard: 1, num_shards: 1, runner: "windows.4xlarge.nonephemeral" },
        ]}
    secrets: inherit

  win-vs2022-cpu-py3-test:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' win-vs2022-cpu-py3 ') }}
    name: win-vs2022-cpu-py3
    uses: ./.github/workflows/_win-test.yml
    needs:
      - win-vs2022-cpu-py3-build
      - target-determination
      - job-filter
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_win-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 271-291 / 第 271-291 行

````yaml
    with:
      build-environment: ${{ needs.win-vs2022-cpu-py3-build.outputs.build-environment }}
      cuda-version: cpu
      test-matrix: ${{ needs.win-vs2022-cpu-py3-build.outputs.test-matrix }}
      disable-monitor: false
    secrets: inherit

  # NB: Windows runners are not available in OSDC, so we run this on Meta account
  # for now until we get OSDC deployed to LF account
  win-vs2022-cuda13_0-py3-build:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' win-vs2022-cuda13.0-py3 ') || contains(needs.job-filter.outputs.jobs, ' cross-compile-linux-test-cuda13 ') }}
    name: win-vs2022-cuda13.0-py3
    uses: ./.github/workflows/_win-build.yml
    needs:
      - get-label-type
      - job-filter
    with:
      build-environment: win-vs2022-cuda13.0-py3
      cuda-version: "13.0"
      runner: windows.4xlarge.nonephemeral
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_win-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 293-316 / 第 293-316 行

````yaml
  linux-jammy-rocm-py3_10-build:
    name: linux-jammy-rocm-py3.10-mi355
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-rocm-py3.10-mi355
      docker-image-name: ci-image:pytorch-linux-jammy-rocm-n-py3
      # TODO (huydhn): Add this back once other workflow migrates
      # sync-tag: rocm-build
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 6, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "default", shard: 2, num_shards: 6, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "default", shard: 3, num_shards: 6, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "default", shard: 4, num_shards: 6, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "default", shard: 5, num_shards: 6, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "default", shard: 6, num_shards: 6, runner: "linux.rocm.gpu.gfx950.1" },
          { config: "distributed", shard: 1, num_shards: 3, runner: "linux.rocm.gpu.gfx950.2" },
          { config: "distributed", shard: 2, num_shards: 3, runner: "linux.rocm.gpu.gfx950.2" },
          { config: "distributed", shard: 3, num_shards: 3, runner: "linux.rocm.gpu.gfx950.2" },
        ]}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 317-339 / 第 317-339 行

````yaml
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
    secrets: inherit

  linux-jammy-rocm-py3_10-test:
    name: linux-jammy-rocm-py3.10-mi355
    uses: ./.github/workflows/_rocm-test.yml
    needs:
      - linux-jammy-rocm-py3_10-build
      - target-determination
      - job-filter
    with:
      build-environment: ${{ needs.linux-jammy-rocm-py3_10-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-rocm-py3_10-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-rocm-py3_10-build.outputs.test-matrix }}
      tests-to-include: ${{ github.event.inputs.tests-to-include || '' }}
    secrets: inherit

  inductor-build:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' inductor-build ') }}
    name: inductor-build
    uses: ./.github/workflows/_linux-build.yml
    needs:
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_rocm-test.yml`, `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 340-359 / 第 340-359 行

````yaml
      - get-label-type
      - job-filter
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-cuda13.0-py3.12-gcc11-sm80
      docker-image-name: ci-image:pytorch-linux-jammy-cuda13.0-cudnn9-py3-gcc11-inductor-benchmarks
      cuda-arch-list: '8.0'
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.12"
      compiler: gcc11
      cuda-version: "13.0"
    secrets: inherit

  verify-cachebench-cpu-build:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' verify-cachebench-cpu-build ') || contains(needs.job-filter.outputs.jobs, ' verify-cachebench-cpu-test ') }}
    name: verify-cachebench-cpu-build
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 360-381 / 第 360-381 行

````yaml
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-py3.10-gcc11
      docker-image-name: ci-image:pytorch-linux-jammy-py3-gcc11-inductor-benchmarks
      test-matrix: |
        { include: [
          { config: "verify_cachebench", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
        ]}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: gcc11
    secrets: inherit

  verify-cachebench-cpu-test:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' verify-cachebench-cpu-test ') }}
    name: verify-cachebench-cpu-test
    uses: ./.github/workflows/_linux-test.yml
    needs:
      - verify-cachebench-cpu-build
      - target-determination
      - job-filter
      - get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 382-399 / 第 382-399 行

````yaml
    with:
      build-environment: ${{ needs.verify-cachebench-cpu-build.outputs.build-environment }}
      docker-image: ${{ needs.verify-cachebench-cpu-build.outputs.docker-image }}
      test-matrix: ${{ needs.verify-cachebench-cpu-build.outputs.test-matrix }}
      tests-to-include: ${{ github.event.inputs.tests-to-include || '' }}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: gcc11
    secrets: inherit

  linux-jammy-py3-clang18-executorch-build:
#    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-py3-clang18-executorch ') }}
    name: linux-jammy-py3-clang18-executorch
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
    if: false # Has been broken for a while
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 400-422 / 第 400-422 行

````yaml
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-py3-clang18-executorch
      docker-image-name: ci-image:pytorch-linux-jammy-py3-clang18-executorch
      test-matrix: |
        { include: [
          { config: "executorch", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
        ]}
    secrets: inherit

  linux-jammy-py3-clang18-executorch-test:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-py3-clang18-executorch ') }}
    name: linux-jammy-py3-clang18-executorch
    uses: ./.github/workflows/_linux-test.yml
    needs:
      - linux-jammy-py3-clang18-executorch-build
      - job-filter
    with:
      build-environment: ${{ needs.linux-jammy-py3-clang18-executorch-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-py3-clang18-executorch-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-py3-clang18-executorch-build.outputs.test-matrix }}
      tests-to-include: ${{ github.event.inputs.tests-to-include || '' }}
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 424-444 / 第 424-444 行

````yaml
  linux-jammy-py3_10-gcc11-full-debug-build-only:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-py3.10-gcc11-full-debug-build-only ') }}
    name: linux-jammy-py3.10-gcc11-full-debug-build-only
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runner: linux.r7i.2xlarge
      build-environment: linux-jammy-py3.10-gcc11-full-debug-build-only
      docker-image-name: ci-image:pytorch-linux-jammy-py3.10-clang18
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: gcc11
    secrets: inherit

  linux-jammy-aarch64-py3_10-gcc13-build:
    name: linux-jammy-aarch64-py3.10
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`, `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 445-467 / 第 445-467 行

````yaml
    with:
      runner_prefix: ${{ needs.get-label-type.outputs.label-type }}
      build-environment: linux-jammy-aarch64-py3.10
      docker-image-name: ci-image:pytorch-linux-jammy-aarch64-py3.10-gcc13
      runner: linux.arm64.m7g.4xlarge
      # Periodic AArch64 tests for SVE256 coverage
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.arm64.m7g.4xlarge" },
          { config: "default", shard: 2, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.arm64.m7g.4xlarge" },
          { config: "default", shard: 3, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.arm64.m7g.4xlarge" },
          { config: "default", shard: 4, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.arm64.m7g.4xlarge" },
          { config: "default", shard: 5, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.arm64.m7g.4xlarge" },
        ]}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: gcc13
    secrets: inherit

  linux-jammy-aarch64-py3_10-gcc13-test:
    name: linux-jammy-aarch64-py3.10
    uses: ./.github/workflows/_linux-test.yml
    needs:
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 468-469 / 第 468-469 行

````yaml
      - linux-jammy-aarch64-py3_10-gcc13-build
      - get-label-type
````

- EN: This section describes repository automation behavior for `.github/workflows/trunk.yml`.
- CN: 该部分描述 `.github/workflows/trunk.yml` 的仓库自动化行为。

### Lines 470-477 / 第 470-477 行

````yaml
    with:
      build-environment: ${{ needs.linux-jammy-aarch64-py3_10-gcc13-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-aarch64-py3_10-gcc13-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-aarch64-py3_10-gcc13-build.outputs.test-matrix }}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: gcc13
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
- Reusable actions / 复用 Action: `./.github/workflows/job-filter.yml`, `./.github/workflows/llm_td_retrieval.yml`, `./.github/workflows/target_determination.yml`, `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `./.github/workflows/_linux-build.yml`, `./.github/workflows/_linux-test.yml`, `./.github/workflows/_mac-build.yml`, `./.github/workflows/_mac-test.yml`, ...
- Inline commands / 内联命令: `]}`
- Environment variables / 环境变量: `PDT`, `CUDA`, `GPU`, `P2188981399`, `USE_PER_OPERATOR_HEADERS`, `OSDC`, `TODO`, `SVE256`
- Named jobs or sections / 命名作业或章节: `on`, `push`, `branches`, `tags`, `workflow_dispatch`, `inputs`, `schedule`, `concurrency`, `permissions`, `jobs`, ...
