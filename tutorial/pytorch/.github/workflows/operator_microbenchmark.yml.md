# operator_microbenchmark.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/operator_microbenchmark.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-11 / 第 1-11 行

````yaml
name: operator_microbenchmark
# NOTE: When adding a new test, please update README: ../../benchmarks/operator_benchmark/README.md

on:
  push:
    tags:
      - ciflow/op-benchmark/*
  workflow_dispatch:
  schedule:
    # Run at 06:00 UTC everyday
    - cron: 0 6 * * *
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

### Lines 23-34 / 第 23-34 行

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
      check_experiments: arc,lf

  # H100 A100 runners
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 35-48 / 第 35-48 行

````yaml
  opmicrobenchmark-build:
    if: github.repository_owner == 'pytorch'
    name: opmicrobenchmark-build
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
    with:
      runner: linux.12xlarge.memory
      build-environment: linux-jammy-cuda12.8-py3.10-gcc11-sm80
      docker-image-name: ci-image:pytorch-linux-jammy-cuda12.8-cudnn9-py3-gcc11
      cuda-arch-list: '8.0 9.0'
      test-matrix: |
        { include: [
          { config: "operator_microbenchmark_test", shard: 1, num_shards: 1, runner: "linux.aws.h100" },
          { config: "operator_microbenchmark_test", shard: 1, num_shards: 1, runner: "linux.aws.a100" },
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 49-61 / 第 49-61 行

````yaml
        ]}
    secrets: inherit

  opmicrobenchmark-test:
    name: opmicrobenchmark-test
    uses: ./.github/workflows/_linux-test.yml
    needs: opmicrobenchmark-build
    with:
      timeout-minutes: 500
      build-environment: ${{ needs.opmicrobenchmark-build.outputs.build-environment }}
      docker-image: ${{ needs.opmicrobenchmark-build.outputs.docker-image }}
      test-matrix: ${{ needs.opmicrobenchmark-build.outputs.test-matrix }}
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 63-76 / 第 63-76 行

````yaml
  # B200 runner (OSDC), always use OSDC runner to test this workflow
  opmicrobenchmark-build-b200:
    if: github.repository_owner == 'pytorch'
    name: opmicrobenchmark-build-b200
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
    with:
      runner_prefix: "mt-"
      runner: linux.r7i.4xlarge
      build-environment: linux-jammy-cuda12.8-py3.10-gcc11-sm100
      docker-image-name: ci-image:pytorch-linux-jammy-cuda12.8-cudnn9-py3-gcc11
      cuda-arch-list: '10.0'
      test-matrix: |
        { include: [
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 77-89 / 第 77-89 行

````yaml
          { config: "operator_microbenchmark_test", shard: 1, num_shards: 1, runner: "linux.dgx.b200" },
        ]}
      use-arc: true
      python-version: "3.10"
      compiler: gcc11
      cuda-version: "12.8"
    secrets: inherit

  opmicrobenchmark-test-b200:
    name: opmicrobenchmark-test-b200
    uses: ./.github/workflows/_linux-test.yml
    needs:
      - opmicrobenchmark-build-b200
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 90-102 / 第 90-102 行

````yaml
      - get-label-type
    with:
      timeout-minutes: 500
      build-environment: ${{ needs.opmicrobenchmark-build-b200.outputs.build-environment }}
      docker-image: ${{ needs.opmicrobenchmark-build-b200.outputs.docker-image }}
      test-matrix: ${{ needs.opmicrobenchmark-build-b200.outputs.test-matrix }}
      use-arc: true
      python-version: "3.10"
      compiler: gcc11
      cuda-version: "12.8"
    secrets: inherit

  # ROCM MI300 runner
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 103-114 / 第 103-114 行

````yaml
  opmicrobenchmark-build-rocm:
    if: github.repository_owner == 'pytorch'
    name: opmicrobenchmark-build-rocm
    uses: ./.github/workflows/_linux-build.yml
    with:
      build-environment: linux-jammy-rocm-py3_10
      docker-image-name: ci-image:pytorch-linux-jammy-rocm-n-py3-benchmarks
      test-matrix: |
        { include: [
          { config: "operator_microbenchmark_test", shard: 1, num_shards: 1, runner: "linux.rocm.gpu.gfx950.1" },
        ]}
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 116-126 / 第 116-126 行

````yaml
  opmicrobenchmark-test-rocm:
    name: opmicrobenchmark-test-rocm
    uses: ./.github/workflows/_rocm-test.yml
    needs: opmicrobenchmark-build-rocm
    with:
      # TODO (huydo): Need to implement sharding for ops micro benchmark
      timeout-minutes: 900
      build-environment: ${{ needs.opmicrobenchmark-build-rocm.outputs.build-environment }}
      docker-image: ${{ needs.opmicrobenchmark-build-rocm.outputs.docker-image }}
      test-matrix: ${{ needs.opmicrobenchmark-build-rocm.outputs.test-matrix }}
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_rocm-test.yml`; threads configuration values into later steps.
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
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `./.github/workflows/_linux-build.yml`, `./.github/workflows/_linux-test.yml`, `./.github/workflows/_rocm-test.yml`
- Inline commands / 内联命令: `]}`
- Environment variables / 环境变量: `NOTE`, `README`, `UTC`, `H100`, `A100`, `B200`, `OSDC`, `ROCM`, `MI300`, `TODO`
- Named jobs or sections / 命名作业或章节: `on`, `push`, `tags`, `workflow_dispatch`, `schedule`, `concurrency`, `permissions`, `jobs`, `get-label-type`, `with`, ...
