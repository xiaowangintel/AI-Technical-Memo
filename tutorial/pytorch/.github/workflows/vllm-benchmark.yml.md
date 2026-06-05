# vllm-benchmark.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/vllm-benchmark.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-11 / 第 1-11 行

````yaml
name: vLLM Benchmark

on:
  workflow_dispatch:
    inputs:
      pytorch_branch:
        description: |
          PyTorch branch (main or refs/pull/PR_NUMBER/head for pull request)
        required: true
        type: string
        default: main
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 12-22 / 第 12-22 行

````yaml
      pytorch_commit:
        description: |
          PyTorch commit (optional, default to use the latest commit from the branch)
        required: false
        type: string
      models:
        description: |
          A comma-separated list of models from pytorch-integration-testing repo (optional, default to run everything)
        required: false
        type: string
        default: ''
````

- EN: This section describes repository automation behavior for `.github/workflows/vllm-benchmark.yml`.
- CN: 该部分描述 `.github/workflows/vllm-benchmark.yml` 的仓库自动化行为。

### Lines 23-35 / 第 23-35 行

````yaml
      compilation_config:
        description: |
          A JSON string for vLLM --compilation-config, applied to all benchmark tests, e.g. {"cudagraph_mode": "PIECEWISE", "use_inductor_graph_partition": true}
        required: false
        type: string
        default: ''
      runners:
        description: |
          A comma-separated list of runners from .github/scripts/generate_vllm_benchmark_matrix.py to run the benchmark (optional, default to run everything)
        required: true
        type: string
        default: h100,b200
  schedule:
````

- EN: This section describes repository automation behavior for `.github/workflows/vllm-benchmark.yml`.
- CN: 该部分描述 `.github/workflows/vllm-benchmark.yml` 的仓库自动化行为。

### Lines 36-46 / 第 36-46 行

````yaml
    # Run daily at 5:15 AM PST
    - cron: '15 13 * * *'

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.sha }}-${{ github.event_name == 'workflow_dispatch' }}-${{ github.event_name == 'schedule' }}
  cancel-in-progress: true

jobs:
  set-parameters:
    if: ${{ !github.event.pull_request.head.repo.fork && github.repository_owner == 'pytorch' }}
    runs-on: linux.c7i.2xlarge
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段。

### Lines 47-57 / 第 47-57 行

````yaml
    outputs:
      benchmark_matrix: ${{ steps.set-parameters.outputs.benchmark_matrix }}
      docker_image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      torch_cuda_arch_list: '8.0 8.9 9.0 10.0 12.0'
      build_environment: linux-jammy-cuda13.0-py3.12-gcc11
    steps:
      - uses: pytorch/test-infra/.github/actions/setup-uv@main
        with:
          python-version: "3.12"
          activate-environment: "true"
          ignore-empty-workdir: "true"
````

- EN: This section reuses actions and step building blocks such as named actions; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 59-70 / 第 59-70 行

````yaml
      - name: Checkout pytorch-integration-testing repository
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
        with:
          repository: pytorch/pytorch-integration-testing
          path: pytorch/pytorch-integration-testing
          ref: main

      - name: Set parameters
        working-directory: pytorch/pytorch-integration-testing
        id: set-parameters
        env:
          MODELS: ${{ inputs.models || '' }}
````

- EN: This section reuses actions and step building blocks such as `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 71-83 / 第 71-83 行

````yaml
          # Only need CUDA for now, we can add ROCm later if needed
          RUNNERS: ${{ inputs.runners || 'h100,b200' }}
        run: |
          set -eux

          # The generated matrix is grouped by model and runner
          python .github/scripts/generate_vllm_benchmark_matrix.py \
            --benchmark-configs-dir vllm-benchmarks/benchmarks \
            --models "${MODELS}" \
            --runners "${RUNNERS}"

      - name: Checkout PyTorch
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
````

- EN: This section reuses actions and step building blocks such as `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 84-94 / 第 84-94 行

````yaml
        with:
          path: pytorch/pytorch
          ref: ${{ inputs.pytorch_commit || inputs.pytorch_branch }}
          show-progress: false

      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          working-directory: pytorch/pytorch
          docker-image-name: ci-image:pytorch-linux-jammy-cuda13.0-cudnn9-py3.12-gcc11-vllm
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/calculate-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 96-108 / 第 96-108 行

````yaml
  build:
    name: Build PyTorch and vLLM
    needs:
      - set-parameters
    uses: ./.github/workflows/_vllm-build.yml
    with:
      runner: linux.24xlarge.memory
      docker_image: ${{ needs.set-parameters.outputs.docker_image }}
      build_environment: ${{ needs.set-parameters.outputs.build_environment }}
      pytorch_branch: ${{ inputs.pytorch_branch }}
      pytorch_commit: ${{ inputs.pytorch_commit }}
      torch_cuda_arch_list: ${{ needs.set-parameters.outputs.torch_cuda_arch_list }}
    secrets: inherit
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_vllm-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 110-121 / 第 110-121 行

````yaml
  benchmarks:
    name: Run vLLM benchmarks
    needs:
      - set-parameters
      - build
    strategy:
      matrix: ${{ fromJson(needs.set-parameters.outputs.benchmark_matrix) }}
      fail-fast: false
    uses: ./.github/workflows/_vllm-benchmark.yml
    permissions:
      id-token: write
      contents: read
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_vllm-benchmark.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 122-131 / 第 122-131 行

````yaml
    with:
      runner: ${{ matrix.runner }}
      docker_image: ${{ needs.set-parameters.outputs.docker_image }}
      build_environment: ${{ needs.set-parameters.outputs.build_environment }}
      pytorch_branch: ${{ inputs.pytorch_branch }}
      pytorch_commit: ${{ inputs.pytorch_commit }}
      models: ${{ matrix.models }}
      compilation_config: ${{ inputs.compilation_config }}
      is_nightly: ${{ github.event_name == 'schedule' }}
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
- Reusable actions / 复用 Action: `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `./.github/workflows/_vllm-build.yml`, `./.github/workflows/_vllm-benchmark.yml`
- Inline commands / 内联命令: `PyTorch`, `A`, `set`, `python`, `benchmark-configs-dir`, `models`, `runners`
- Environment variables / 环境变量: `PR_NUMBER`, `JSON`, `PIECEWISE`, `PST`, `MODELS`, `CUDA`, `RUNNERS`
- Named jobs or sections / 命名作业或章节: `on`, `workflow_dispatch`, `inputs`, `schedule`, `concurrency`, `jobs`, `set-parameters`, `outputs`, `steps`, `build`, ...
