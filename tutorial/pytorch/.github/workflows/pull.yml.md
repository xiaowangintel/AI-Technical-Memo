# pull.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/pull.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-20 / 第 1-20 行

````yaml
name: pull

on:
  pull_request:
    branches-ignore:
      - nightly
  push:
    branches:
      - main
      - release/*
      - landchecks/*
    tags:
      - ciflow/pull/*
  workflow_dispatch:
    inputs:
      jobs-to-include:
        description: "Space-separated list of job display names to run (empty = all)"
        required: false
        default: ""
        type: string
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 21-43 / 第 21-43 行

````yaml
      tests-to-include:
        description: "Space-separated tests to include (passed to test runner; empty = default)"
        required: false
        default: ""
        type: string
  schedule:
    - cron: 29 8 * * *  # about 1:29am PDT

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.sha }}-${{ github.event_name == 'workflow_dispatch' && github.run_id }}-${{ github.event_name == 'schedule' }}
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
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; reuses actions and step building blocks such as `./.github/workflows/job-filter.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 44-65 / 第 44-65 行

````yaml
    with:
      jobs-to-include: ${{ github.event.inputs.jobs-to-include || '' }}

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
      check_experiments: arc,lf
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `./.github/workflows/llm_td_retrieval.yml`, `./.github/workflows/target_determination.yml`, `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 67-90 / 第 67-90 行

````yaml
  # ╠══════════════════════════════════════════════════════════════════════╣
  # ║ linux-jammy-py3.10-gcc11 (build + test)                              ║
  # ╠══════════════════════════════════════════════════════════════════════╣

  linux-jammy-py3_10-gcc11-build:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-py3.10-gcc11 ') || contains(needs.job-filter.outputs.jobs, ' linux-docs ') }}
    name: linux-jammy-py3.10-gcc11
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-py3.10-gcc11
      docker-image-name: ci-image:pytorch-linux-jammy-py3.10-clang18
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 2, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 3, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 4, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 5, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "docs_test", shard: 1, num_shards: 1,  runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          { config: "jit_legacy", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 91-113 / 第 91-113 行

````yaml
          { config: "backwards_compat", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.c7i.2xlarge" },
          # Use m7i instead of m7a to match the existing (Intel Xeon) numerics. DTensor crossref tests like linalg.multi_dot have tight
          # float32 tolerances sensitive to different FMA/reduction order across CPU vendors.
          { config: "distributed", shard: 1, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge.amx" },
          { config: "distributed", shard: 2, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge.amx" },
          { config: "distributed", shard: 3, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge.amx" },
          { config: "numpy_2_x", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.c7i.2xlarge" },
          { config: "libtorch_agnostic_targetting", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          { config: "openreg", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
        ]}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: gcc11
    secrets: inherit

  linux-jammy-py3_10-gcc11-test:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-py3.10-gcc11 ') }}
    name: linux-jammy-py3.10-gcc11
    uses: ./.github/workflows/_linux-test.yml
    needs:
      - linux-jammy-py3_10-gcc11-build
      - target-determination
      - job-filter
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 114-133 / 第 114-133 行

````yaml
      - get-label-type
    with:
      build-environment: ${{ needs.linux-jammy-py3_10-gcc11-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-py3_10-gcc11-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-py3_10-gcc11-build.outputs.test-matrix }}
      tests-to-include: ${{ github.event.inputs.tests-to-include || '' }}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: gcc11
    secrets: inherit

  # ╠══════════════════════════════════════════════════════════════════════╣
  # ║ linux-jammy-aarch64-py3.10-gcc11 (build + test)                      ║
  # ╠══════════════════════════════════════════════════════════════════════╣
  linux-jammy-aarch64-py3_10-gcc13-build:
    name: linux-jammy-aarch64-py3.10
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 134-156 / 第 134-156 行

````yaml
    with:
      runner_prefix: ${{ needs.get-label-type.outputs.label-type }}
      build-environment: linux-jammy-aarch64-py3.10
      docker-image-name: ci-image:pytorch-linux-jammy-aarch64-py3.10-gcc13
      runner: linux.arm64.m8g.4xlarge
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.arm64.m8g.4xlarge" },
          { config: "default", shard: 2, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.arm64.m8g.4xlarge" },
          { config: "default", shard: 3, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.arm64.m8g.4xlarge" },
          { config: "default", shard: 4, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.arm64.m8g.4xlarge" },
          { config: "default", shard: 5, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.arm64.m8g.4xlarge" },
          { config: "openreg", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.arm64.m8g.4xlarge" },
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

### Lines 157-177 / 第 157-177 行

````yaml
      - linux-jammy-aarch64-py3_10-gcc13-build
      - target-determination
      - job-filter
      - get-label-type
    with:
      build-environment: ${{ needs.linux-jammy-aarch64-py3_10-gcc13-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-aarch64-py3_10-gcc13-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-aarch64-py3_10-gcc13-build.outputs.test-matrix }}
      tests-to-include: ${{ github.event.inputs.tests-to-include || '' }}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: gcc13
    secrets: inherit

  # ╠══════════════════════════════════════════════════════════════════════╣
  # ║ linux-jammy-py3.14t-clang18 (build + test)                           ║
  # ╠══════════════════════════════════════════════════════════════════════╣
  linux-jammy-py3_14t-clang18-build:
    name: linux-jammy-py3.14t-clang18
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 178-201 / 第 178-201 行

````yaml
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-py3.14t-clang18
      docker-image-name: ci-image:pytorch-linux-jammy-py3.14t-clang18
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 2, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 3, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 4, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 5, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "crossref", shard: 1, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          { config: "crossref", shard: 2, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          # Free-threaded Python 3.14t has ~20-50% higher per-object memory overhead from biased reference counting and
          # per-object locks, and test_nn (531 tests) under dynamo wrapping with compiled autograd consistently OOMs at 64GB.
          { config: "dynamo_wrapped", shard: 1, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "dynamo_wrapped", shard: 2, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "dynamo_wrapped", shard: 3, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "einops", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          { config: "openreg", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
        ]}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.14t"
      compiler: clang18
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 202-222 / 第 202-222 行

````yaml
    secrets: inherit

  linux-jammy-py3_14t-clang18-test:
    name: linux-jammy-py3.14t-clang18
    uses: ./.github/workflows/_linux-test.yml
    needs:
      - linux-jammy-py3_14t-clang18-build
      - target-determination
      - get-label-type
    with:
      build-environment: linux-jammy-py3.14t-clang18
      docker-image: ${{ needs.linux-jammy-py3_14t-clang18-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-py3_14t-clang18-build.outputs.test-matrix }}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.14t"
      compiler: clang18
    secrets: inherit

  # ╠══════════════════════════════════════════════════════════════════════╣
  # ║ linux-docs                                                           ║
  # ╠══════════════════════════════════════════════════════════════════════╣
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 224-244 / 第 224-244 行

````yaml
  linux-docs:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-docs ') }}
    name: linux-docs
    uses: ./.github/workflows/_docs.yml
    needs:
      - linux-jammy-py3_10-gcc11-build
      - job-filter
      - get-label-type
    with:
      build-environment: ${{ needs.linux-jammy-py3_10-gcc11-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-py3_10-gcc11-build.outputs.docker-image }}
      run-doxygen: true
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      python-version: "3.10"
      compiler: gcc11
    secrets: inherit

  # ╠══════════════════════════════════════════════════════════════════════╣
  # ║ linux-jammy-py3.10-gcc11-no-ops (build only)                         ║
  # ╠══════════════════════════════════════════════════════════════════════╣
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_docs.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 246-268 / 第 246-268 行

````yaml
  linux-jammy-py3_10-gcc11-no-ops:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-py3.10-gcc11-no-ops ') }}
    name: linux-jammy-py3.10-gcc11-no-ops
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-py3.10-gcc11-no-ops
      docker-image-name: ci-image:pytorch-linux-jammy-py3.10-clang18
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 1 },
        ]}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: gcc11
    secrets: inherit

  # ╠══════════════════════════════════════════════════════════════════════╣
  # ║ linux-jammy-py3.10-clang18-asan (build + test)                       ║
  # ╠══════════════════════════════════════════════════════════════════════╣
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 270-293 / 第 270-293 行

````yaml
  linux-jammy-py3_10-clang18-asan-build:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-py3.10-clang18-asan ') }}
    name: linux-jammy-py3.10-clang18-asan
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
    with:
      runner: linux.c7i.4xlarge
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-py3.10-clang18-asan
      docker-image-name: ci-image:pytorch-linux-jammy-py3.10-clang18
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 7, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 2, num_shards: 7, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 3, num_shards: 7, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 4, num_shards: 7, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 5, num_shards: 7, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 6, num_shards: 7, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 7, num_shards: 7, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "openreg", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
        ]}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 294-316 / 第 294-316 行

````yaml
      python-version: "3.10"
      compiler: clang18
      # TODO (huydhn): Add this back once other workflow migrates
      # sync-tag: asan-build
    secrets: inherit

  linux-jammy-py3_10-clang18-asan-test:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-py3.10-clang18-asan ') }}
    name: linux-jammy-py3.10-clang18-asan
    uses: ./.github/workflows/_linux-test.yml
    needs:
      - linux-jammy-py3_10-clang18-asan-build
      - target-determination
      - job-filter
      - get-label-type
    with:
      build-environment: ${{ needs.linux-jammy-py3_10-clang18-asan-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-py3_10-clang18-asan-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-py3_10-clang18-asan-build.outputs.test-matrix }}
      tests-to-include: ${{ github.event.inputs.tests-to-include || '' }}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: clang18
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 317-331 / 第 317-331 行

````yaml
      # TODO (huydhn): Add this back once other workflow migrates
      # sync-tag: asan-test
    secrets: inherit

  # ╠══════════════════════════════════════════════════════════════════════╣
  # ║ linux-jammy-py3.10-clang18 (build + test)                            ║
  # ╠══════════════════════════════════════════════════════════════════════╣

  linux-jammy-py3_10-clang18-build:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-py3.10-clang18 ') }}
    name: linux-jammy-py3.10-clang18
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 332-355 / 第 332-355 行

````yaml
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-py3.10-clang18
      docker-image-name: ci-image:pytorch-linux-jammy-py3.10-clang18
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 2, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 3, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 4, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 5, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "crossref", shard: 1, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          { config: "crossref", shard: 2, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          { config: "dynamo_wrapped", shard: 1, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          { config: "dynamo_wrapped", shard: 2, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          { config: "dynamo_wrapped", shard: 3, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          { config: "einops", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          { config: "openreg", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          { config: "onnx", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
        ]}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: clang18
    secrets: inherit
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 357-378 / 第 357-378 行

````yaml
  linux-jammy-py3_10-clang18-test:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-py3.10-clang18 ') }}
    name: linux-jammy-py3.10-clang18
    uses: ./.github/workflows/_linux-test.yml
    needs:
      - linux-jammy-py3_10-clang18-build
      - target-determination
      - job-filter
      - get-label-type
    with:
      build-environment: ${{ needs.linux-jammy-py3_10-clang18-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-py3_10-clang18-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-py3_10-clang18-build.outputs.test-matrix }}
      tests-to-include: ${{ github.event.inputs.tests-to-include || '' }}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: clang18
    secrets: inherit

  # ╠══════════════════════════════════════════════════════════════════════╣
  # ║ linux-jammy-py3.14-clang18 (build + test)                            ║
  # ╠══════════════════════════════════════════════════════════════════════╣
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 380-403 / 第 380-403 行

````yaml
  linux-jammy-py3_14-clang18-build:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-py3.14-clang18 ') }}
    name: linux-jammy-py3.14-clang18
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-py3.14-clang18
      docker-image-name: ci-image:pytorch-linux-jammy-py3.14-clang18
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 2, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 3, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 4, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "default", shard: 5, num_shards: 5, runner: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge" },
          { config: "crossref", shard: 1, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          { config: "crossref", shard: 2, num_shards: 2, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          { config: "dynamo_wrapped", shard: 1, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          { config: "dynamo_wrapped", shard: 2, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          { config: "dynamo_wrapped", shard: 3, num_shards: 3, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
          { config: "einops", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 404-418 / 第 404-418 行

````yaml
          { config: "openreg", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge" },
        ]}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.14"
      compiler: clang18
    secrets: inherit

  linux-jammy-py3_14-clang18-test:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-py3.14-clang18 ') }}
    name: linux-jammy-py3.14-clang18
    uses: ./.github/workflows/_linux-test.yml
    needs:
      - linux-jammy-py3_14-clang18-build
      - job-filter
      - get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 419-439 / 第 419-439 行

````yaml
    with:
      build-environment: ${{ needs.linux-jammy-py3_14-clang18-build.outputs.build-environment }}
      docker-image: ${{ needs.linux-jammy-py3_14-clang18-build.outputs.docker-image }}
      test-matrix: ${{ needs.linux-jammy-py3_14-clang18-build.outputs.test-matrix }}
      tests-to-include: ${{ github.event.inputs.tests-to-include || '' }}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.14"
      compiler: clang18
    secrets: inherit

  # ╠══════════════════════════════════════════════════════════════════════╣
  # ║ linux-jammy-cuda12.8-cudnn9-py3.10-clang18 (build only)              ║
  # ╠══════════════════════════════════════════════════════════════════════╣

  linux-jammy-cuda12_8-cudnn9-py3_10-clang18-build:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-cuda12.8-cudnn9-py3.10-clang18 ') }}
    name: linux-jammy-cuda12.8-cudnn9-py3.10-clang18
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 440-462 / 第 440-462 行

````yaml
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-cuda12.8-cudnn9-py3.10-clang18
      docker-image-name: ci-image:pytorch-linux-jammy-cuda12.8-cudnn9-py3.10-clang18
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 1 },
        ]}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: clang18
      cuda-version: "12.8"
    secrets: inherit

  # ╠══════════════════════════════════════════════════════════════════════╣
  # ║ linux-jammy-py3.10-gcc11-mobile-lightweight-dispatch (build only)    ║
  # ╠══════════════════════════════════════════════════════════════════════╣

  linux-jammy-py3_10-gcc11-mobile-lightweight-dispatch-build:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-py3.10-gcc11-mobile-lightweight-dispatch-build ') }}
    name: linux-jammy-py3.10-gcc11-mobile-lightweight-dispatch-build
    uses: ./.github/workflows/_linux-build.yml
    needs:
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 463-484 / 第 463-484 行

````yaml
      - get-label-type
      - job-filter
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-py3.10-gcc11-mobile-lightweight-dispatch-build
      docker-image-name: ci-image:pytorch-linux-jammy-py3.10-clang18
      build-generates-artifacts: false
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 1 },
        ]}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
      compiler: gcc11
    secrets: inherit

  # ╠══════════════════════════════════════════════════════════════════════╣
  # ║ linux-jammy-rocm-py3.10 (build)                                      ║
  # ╠══════════════════════════════════════════════════════════════════════╣

  linux-jammy-rocm-py3_10-build:
    if: ${{ github.event_name == 'pull_request' || (needs.job-filter.outputs.jobs != '' && contains(needs.job-filter.outputs.jobs, ' linux-jammy-rocm-py3.10 ')) }}
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 485-507 / 第 485-507 行

````yaml
    # don't run build twice on main
    name: linux-jammy-rocm-py3.10
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-rocm-py3.10
      docker-image-name: ci-image:pytorch-linux-jammy-rocm-n-py3
      # TODO (huydhn): Add this back once other workflow migrates
      # sync-tag: rocm-build
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 3, runner: "linux.rocm.gpu.2" },
          { config: "default", shard: 2, num_shards: 3, runner: "linux.rocm.gpu.2" },
          { config: "default", shard: 3, num_shards: 3, runner: "linux.rocm.gpu.2" },
        ]}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
    secrets: inherit

  # ╠══════════════════════════════════════════════════════════════════════╣
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 508-531 / 第 508-531 行

````yaml
  # ║ linux-jammy-xpu-n-py3.10 (build)                                     ║
  # ╠══════════════════════════════════════════════════════════════════════╣

  linux-jammy-xpu-n-py3_10-build:
    if: ${{ needs.job-filter.outputs.jobs == '' || contains(needs.job-filter.outputs.jobs, ' linux-jammy-xpu-n-py3.10 ') }}
    name: linux-jammy-xpu-n-py3.10
    uses: ./.github/workflows/_linux-build.yml
    needs:
      - get-label-type
      - job-filter
    with:
      runner_prefix: ${{ needs.get-label-type.outputs.label-type }}
      runner: linux.c7i.4xlarge
      build-environment: linux-noble-xpu-n-py3.10
      docker-image-name: ci-image:pytorch-linux-noble-xpu-n-py3
      test-matrix: |
        { include: [
          { config: "default", shard: 1, num_shards: 4, runner: "linux.idc.xpu" },
          { config: "default", shard: 2, num_shards: 4, runner: "linux.idc.xpu" },
          { config: "default", shard: 3, num_shards: 4, runner: "linux.idc.xpu" },
          { config: "default", shard: 4, num_shards: 4, runner: "linux.idc.xpu" },
        ]}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.10"
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 532-553 / 第 532-553 行

````yaml
    secrets: inherit

  # ╠══════════════════════════════════════════════════════════════════════╣
  # ║ dynamo-cpython (build + test)                                        ║
  # ╠══════════════════════════════════════════════════════════════════════╣

  dynamo-cpython-build:
    name: dynamo-cpython-build
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-py3.13-clang18
      docker-image-name: ci-image:pytorch-linux-jammy-py3.13-clang18
      test-matrix: |
        { include: [
          { config: "dynamo_cpython", shard: 1, num_shards: 1, runner: "${{ needs.get-label-type.outputs.label-type }}linux.c7i.2xlarge" },
        ]}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.13"
      compiler: clang18
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 555-558 / 第 555-558 行

````yaml
  dynamo-cpython-test:
    name: dynamo-cpython-test
    uses: ./.github/workflows/_linux-test.yml
    needs: [get-label-type, dynamo-cpython-build]
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_linux-test.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 559-566 / 第 559-566 行

````yaml
    with:
      build-environment: linux-jammy-py3.13-clang18
      docker-image: ${{ needs.dynamo-cpython-build.outputs.docker-image }}
      test-matrix: ${{ needs.dynamo-cpython-build.outputs.test-matrix }}
      use-arc: ${{ needs.get-label-type.outputs.use-arc == 'true' }}
      python-version: "3.13"
      compiler: clang18
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
- Reusable actions / 复用 Action: `./.github/workflows/job-filter.yml`, `./.github/workflows/llm_td_retrieval.yml`, `./.github/workflows/target_determination.yml`, `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `./.github/workflows/_linux-build.yml`, `./.github/workflows/_linux-test.yml`, `./.github/workflows/_docs.yml`
- Inline commands / 内联命令: `]}`
- Environment variables / 环境变量: `PDT`, `FMA`, `CPU`, `TODO`
- Named jobs or sections / 命名作业或章节: `on`, `pull_request`, `branches-ignore`, `push`, `branches`, `tags`, `workflow_dispatch`, `inputs`, `schedule`, `concurrency`, ...
