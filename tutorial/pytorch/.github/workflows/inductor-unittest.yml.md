# inductor-unittest.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/inductor-unittest.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-16 / 第 1-16 行

````yaml
# Workflow: Inductor Unit Test
# 1. runs unit tests for inductor.
# 2. performs daily memory leak checks and reruns of disabled tests, scheduled at `29 8 * * *`.
name: inductor-unittest

on:
  workflow_call:
  schedule:
    - cron: 29 8 * * * # about 1:29am PDT, for mem leak check and rerun disabled tests.
  pull_request:
    paths:
      - .github/workflows/inductor-unittest.yml

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.sha }}-${{ github.event_name == 'workflow_dispatch' }}-unittest
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

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 35-52 / 第 35-52 行

````yaml
  inductor-build:
    name: inductor-build
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
    with:
      build-environment: linux-jammy-cuda13.0-py3.10-gcc11-sm86
      docker-image-name: ci-image:pytorch-linux-jammy-cuda13.0-cudnn9-py3-gcc11-inductor-benchmarks
      cuda-arch-list: '8.6'
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      test-matrix: |
        { include: [
          { config: "inductor", shard: 1, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g5.4xlarge.nvidia.gpu" },
          { config: "inductor", shard: 2, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g5.4xlarge.nvidia.gpu" },
          { config: "inductor_distributed", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g5.12xlarge.nvidia.gpu" },
          { config: "inductor_cpp_wrapper", shard: 1, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g5.4xlarge.nvidia.gpu" },
          { config: "inductor_cpp_wrapper", shard: 2, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.g5.4xlarge.nvidia.gpu" },
        ]}
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 54-67 / 第 54-67 行

````yaml
  inductor-test:
    name: inductor-test
    uses: ./.github/workflows/_linux-test.yml
    needs: inductor-build
    with:
      build-environment: ${{ needs.inductor-build.outputs.build-environment }}
      docker-image: ${{ needs.inductor-build.outputs.docker-image }}
      test-matrix: ${{ needs.inductor-build.outputs.test-matrix }}
    secrets: inherit

  inductor-halide-build:
    name: inductor-halide-build
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`, `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 68-81 / 第 68-81 行

````yaml
    with:
      build-environment: linux-jammy-py3.12-gcc11-halide
      docker-image-name: ci-image:pytorch-linux-jammy-py3.12-halide
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      test-matrix: |
        { include: [
          { config: "inductor-halide", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.12xlarge" },
        ]}
    secrets: inherit

  inductor-halide-test:
    name: inductor-halide-test
    uses: ./.github/workflows/_linux-test.yml
    needs: inductor-halide-build
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 82-99 / 第 82-99 行

````yaml
    with:
      build-environment: ${{ needs.inductor-halide-build.outputs.build-environment }}
      docker-image: ${{ needs.inductor-halide-build.outputs.docker-image }}
      test-matrix: ${{ needs.inductor-halide-build.outputs.test-matrix }}
    secrets: inherit

  inductor-pallas-cpu-build:
    name: inductor-pallas-cpu-build
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
    with:
      build-environment: linux-jammy-py3.12-gcc11-pallas
      docker-image-name: ci-image:pytorch-linux-jammy-py3.12-pallas
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      test-matrix: |
        { include: [
          { config: "inductor-pallas-cpu", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.12xlarge" },
        ]}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 100-115 / 第 100-115 行

````yaml
    secrets: inherit

  inductor-pallas-cpu-test:
    name: inductor-pallas-cpu-test
    uses: ./.github/workflows/_linux-test.yml
    needs: inductor-pallas-cpu-build
    with:
      build-environment: ${{ needs.inductor-pallas-cpu-build.outputs.build-environment }}
      docker-image: ${{ needs.inductor-pallas-cpu-build.outputs.docker-image }}
      test-matrix: ${{ needs.inductor-pallas-cpu-build.outputs.test-matrix }}
    secrets: inherit

  inductor-triton-cpu-build:
    name: inductor-triton-cpu-build
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`, `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 116-129 / 第 116-129 行

````yaml
    with:
      build-environment: linux-jammy-py3.12-gcc11-triton-cpu
      docker-image-name: ci-image:pytorch-linux-jammy-py3.12-triton-cpu
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      test-matrix: |
        { include: [
          { config: "inductor-triton-cpu", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.12xlarge" },
        ]}
    secrets: inherit

  inductor-triton-cpu-test:
    name: linux-jammy-cpu-py3.12-gcc11-inductor-triton-cpu
    uses: ./.github/workflows/_linux-test.yml
    needs: inductor-triton-cpu-build
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 130-147 / 第 130-147 行

````yaml
    with:
      build-environment: ${{ needs.inductor-triton-cpu-build.outputs.build-environment }}
      docker-image: ${{ needs.inductor-triton-cpu-build.outputs.docker-image }}
      test-matrix: ${{ needs.inductor-triton-cpu-build.outputs.test-matrix }}
    secrets: inherit

  inductor-cpu-build:
    name: inductor-cpu-build
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
    with:
      build-environment: linux-jammy-py3.10-gcc11-build
      docker-image-name: ci-image:pytorch-linux-jammy-py3-gcc11-inductor-benchmarks
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      test-matrix: |
        { include: [
          { config: "inductor_amx", shard: 1, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge.amx" },
          { config: "inductor_amx", shard: 2, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge.amx" },
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 148-164 / 第 148-164 行

````yaml
        ]}
    secrets: inherit

  inductor-cpu-test:
    name: inductor-cpu-test
    uses: ./.github/workflows/_linux-test.yml
    needs: inductor-cpu-build
    with:
      build-environment: ${{ needs.inductor-cpu-build.outputs.build-environment }}
      docker-image: ${{ needs.inductor-cpu-build.outputs.docker-image }}
      test-matrix: ${{ needs.inductor-cpu-build.outputs.test-matrix }}
    secrets: inherit

  inductor-cpu-core-build:
    name: inductor-cpu-core-build
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`, `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 165-177 / 第 165-177 行

````yaml
    strategy:
      matrix:
        python-version: ['3.11', '3.12', '3.13']
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-py${{ matrix.python-version }}-clang18
      docker-image-name: ci-image:pytorch-linux-jammy-py${{ matrix.python-version }}-clang18
      test-matrix: |
        { include: [
          { config: "inductor_core", shard: 1, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.c7i.2xlarge" },
          { config: "inductor_core", shard: 2, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.c7i.2xlarge" },
        ]}
    secrets: inherit
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 179-194 / 第 179-194 行

````yaml
  inductor-cpu-core-test:
    name: inductor-cpu-core-test
    uses: ./.github/workflows/_linux-test.yml
    needs: [get-label-type, inductor-cpu-core-build]
    strategy:
      matrix:
        python-version: ['3.11', '3.12', '3.13']
    with:
      build-environment: linux-jammy-py${{ matrix.python-version }}-clang18
      docker-image: ci-image:pytorch-linux-jammy-py${{ matrix.python-version }}-clang18
      test-matrix: |
        { include: [
          { config: "inductor_core", shard: 1, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.c7i.2xlarge" },
          { config: "inductor_core", shard: 2, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.c7i.2xlarge" },
        ]}
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
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
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `./.github/workflows/_linux-build.yml`, `./.github/workflows/_linux-test.yml`
- Inline commands / 内联命令: `]}`
- Environment variables / 环境变量: `PDT`
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `schedule`, `pull_request`, `paths`, `concurrency`, `permissions`, `jobs`, `get-label-type`, `with`, ...
