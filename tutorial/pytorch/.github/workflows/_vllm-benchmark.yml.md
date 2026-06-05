# _vllm-benchmark.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/_vllm-benchmark.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-17 / 第 1-17 行

````yaml
name: vllm-benchmark

on:
  workflow_call:
    inputs:
      runner:
        required: true
        type: string
        description: The runner to use.
      docker_image:
        required: true
        type: string
        description: The name of the base docker image to use.
      build_environment:
        required: true
        type: string
        description: The build environment name, e.g. linux-jammy-cuda13.0-py3.12-gcc11
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 18-30 / 第 18-30 行

````yaml
      pytorch_branch:
        required: false
        type: string
        description: The PyTorch branch to checkout. Default to the current checkout.
      pytorch_commit:
        required: false
        type: string
        description: The PyTorch commit to checkout. Default to the current checkout.
      models:
        required: false
        type: string
        description: The list of models to benchmark. Default to all models.
        default: ''
````

- EN: This section describes repository automation behavior for `.github/workflows/_vllm-benchmark.yml`.
- CN: 该部分描述 `.github/workflows/_vllm-benchmark.yml` 的仓库自动化行为。

### Lines 31-47 / 第 31-47 行

````yaml
      compilation_config:
        required: false
        type: string
        description: A JSON string for vLLM --compilation-config, applied to all benchmark tests.
        default: ''
      is_nightly:
        required: false
        type: boolean
        description: Whether this is a nightly scheduled run. When true, TRANSFORMERS_OFFLINE is disabled.
        default: false

jobs:
  benchmark:
    runs-on: ${{ inputs.runner }}
    permissions:
      id-token: write
      contents: read
````

- EN: This section lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 48-61 / 第 48-61 行

````yaml
    container:
      image: ${{ inputs.docker_image }}
      options: --gpus all --ipc=host --tty
      volumes:
        - /mnt/hf_cache:/mnt/hf_cache
    steps:
      - name: Install system dependencies
        run: |
          set -eux
          sudo apt-get update
          sudo apt-get install -y git unzip

      - name: Checkout PyTorch
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
````

- EN: This section reuses actions and step building blocks such as `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 62-74 / 第 62-74 行

````yaml
        with:
          ref: ${{ inputs.pytorch_commit || inputs.pytorch_branch }}
          submodules: recursive
          show-progress: false

      - name: Get vLLM pinned commit
        id: vllm-pinned-commit
        run: |
          VLLM_PINNED_COMMIT=$(cat .github/ci_commit_pins/vllm.txt)
          echo "commit=${VLLM_PINNED_COMMIT}" >> "${GITHUB_OUTPUT}"

      - name: Checkout vLLM
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
````

- EN: This section reuses actions and step building blocks such as `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 75-89 / 第 75-89 行

````yaml
        with:
          repository: vllm-project/vllm
          path: vllm-project/vllm
          ref: ${{ steps.vllm-pinned-commit.outputs.commit }}
          submodules: recursive
          show-progress: false

      - name: Authenticate with AWS
        if: ${{ always() && contains(inputs.runner, 'b200') }}
        uses: aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722 # v4.1.0
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_upload-benchmark-results
          # The max duration enforced by the server side
          role-duration-seconds: 18000
          aws-region: us-east-1
````

- EN: This section reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 91-106 / 第 91-106 行

````yaml
      - name: Download build artifacts
        uses: ./.github/actions/download-build-artifacts
        with:
          name: ${{ inputs.build_environment }}
          s3-bucket: gha-artifacts

      - uses: pytorch/test-infra/.github/actions/setup-uv@main
        with:
          python-version: "3.12"
          activate-environment: "true"

      - name: Install build artifacts
        shell: bash
        run: |
          set -eux
          ls -laR dist
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/download-build-artifacts`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 108-122 / 第 108-122 行

````yaml
          # Need to install pip because there might be parts that are not using
          # uv yet
          uv pip install pip==25.3

          uv pip install \
            dist/torch-*.whl \
            dist/vision/torchvision-*.whl \
            dist/audio/torchaudio-*.whl \
            dist/ao/torchao-*.whl \
            dist/vllm/vllm-*.whl \
            dist/deepgemm/deep_gemm-*.whl \
            --extra-index-url https://download.pytorch.org/whl/cu130 \
            --index-strategy unsafe-best-match

      - name: Print some debug information
````

- EN: This section describes repository automation behavior for `.github/workflows/_vllm-benchmark.yml`.
- CN: 该部分描述 `.github/workflows/_vllm-benchmark.yml` 的仓库自动化行为。

### Lines 123-139 / 第 123-139 行

````yaml
        env:
          VLLM_PINNED_COMMIT: ${{ steps.vllm-pinned-commit.outputs.commit }}
        run: |
          uv pip list

          PYTORCH_WHL=$(uv pip list | grep torch)
          VLLM_WHL=$(uv pip list | grep vllm)

          {
            echo "### Run vLLM benchmark with the following packages:"
            echo "PyTorch:"
            echo "${PYTORCH_WHL}"
            echo "vLLM: ${VLLM_WHL}"
          } >> "${GITHUB_STEP_SUMMARY}"

      - name: Checkout pytorch-integration-testing repository
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
````

- EN: This section reuses actions and step building blocks such as `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 140-157 / 第 140-157 行

````yaml
        with:
          repository: pytorch/pytorch-integration-testing
          path: pytorch/pytorch-integration-testing
          ref: main

      - name: Check if the device is supported
        run: |
          set -eux

          if command -v nvidia-smi; then
            DEVICE_NAME=cuda
            nvidia-smi
          elif command -v rocm-smi; then
            DEVICE_NAME=rocm
            rocm-smi
          elif command -v hl-smi; then
            DEVICE_NAME=hpu
            hl-smi
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 158-173 / 第 158-173 行

````yaml
          else
            lscpu
            arch=$(uname -m)

            case "$arch" in
              aarch64|arm64)
                DEVICE_NAME=arm64-cpu
                ;;
              *)
                DEVICE_NAME=cpu
                ;;
            esac
          fi
          echo "DEVICE_NAME=$DEVICE_NAME" >> "${GITHUB_ENV}"

      - name: Setup benchmark tests
````

- EN: This section describes repository automation behavior for `.github/workflows/_vllm-benchmark.yml`.
- CN: 该部分描述 `.github/workflows/_vllm-benchmark.yml` 的仓库自动化行为。

### Lines 174-185 / 第 174-185 行

````yaml
        env:
          MODELS: ${{ inputs.models }}
          COMPILATION_CONFIG: ${{ inputs.compilation_config }}
        shell: bash
        run: |
          set -eux

          pushd vllm-project/vllm
          rm .buildkite/performance-benchmarks/tests/*.json || true
          popd

          pushd pytorch/pytorch-integration-testing
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 186-200 / 第 186-200 行

````yaml
          # Set the list of benchmarks we want to cover in this runner
          python .github/scripts/setup_vllm_benchmark.py \
            --from-benchmark-configs-dir vllm-benchmarks/benchmarks \
            --to-benchmark-configs-dir ../../vllm-project/vllm/.buildkite/performance-benchmarks/tests \
            --models "${MODELS}" \
            --device "${DEVICE_NAME}" \
            --include-eager-mode \
            --include-inductor-graph-partition \
            --compilation-config "${COMPILATION_CONFIG}"
          popd

          pushd vllm-project/vllm
          ls -lah .buildkite/performance-benchmarks/tests
          find .buildkite/performance-benchmarks/tests -type f -exec cat {} \;
          popd
````

- EN: This section describes repository automation behavior for `.github/workflows/_vllm-benchmark.yml`.
- CN: 该部分描述 `.github/workflows/_vllm-benchmark.yml` 的仓库自动化行为。

### Lines 202-218 / 第 202-218 行

````yaml
      - name: Run vLLM benchmark
        working-directory: vllm-project/vllm
        env:
          FLASHINFER_WORKSPACE_BASE: /mnt/hf_cache
          HF_TOKEN: ${{ secrets.VLLM_TEST_HUGGING_FACE_TOKEN }}
          HF_HOME: /mnt/hf_cache
          # Nightly runs download from HF to periodically refresh the local cache
          TRANSFORMERS_OFFLINE: ${{ inputs.is_nightly && '0' || '1' }}
          # vLLM-related environment variables
          ENGINE_VERSION: v1
          SAVE_TO_PYTORCH_BENCHMARK_FORMAT: 1
        shell: bash
        run: |
          set -eux
          bash .buildkite/performance-benchmarks/scripts/run-performance-benchmarks.sh

      - name: Check the benchmark results
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 219-235 / 第 219-235 行

````yaml
        env:
          BENCHMARK_RESULTS: vllm-project/vllm/benchmarks/results
        run: |
          set -eux

          # Fail when there is no result or if the metrics are all zero
          python3 pytorch/pytorch-integration-testing/.github/scripts/check_benchmark_results.py \
            --benchmark-results "${BENCHMARK_RESULTS}" \
            --strict

      - name: Authenticate with AWS
        uses: aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722 # v4.1.0
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_upload-benchmark-results
          # The max duration enforced by the server side
          role-duration-seconds: 18000
          aws-region: us-east-1
````

- EN: This section reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 237-238 / 第 237-238 行

````yaml
      - name: Upload the benchmark results to OSS benchmark database for the dashboard
        uses: pytorch/test-infra/.github/actions/upload-benchmark-results@main
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/upload-benchmark-results@main`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 239-244 / 第 239-244 行

````yaml
        with:
          benchmark-results-dir: vllm-project/vllm/benchmarks/results
          benchmark-name: 'PyTorch x vLLM benchmark'
          dry-run: false
          github-token: ${{ secrets.GITHUB_TOKEN }}
          if-no-files-found: error
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。


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
- Reusable actions / 复用 Action: `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`, `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`, `./.github/actions/download-build-artifacts`, `pytorch/test-infra/.github/actions/upload-benchmark-results@main`
- Inline commands / 内联命令: `set`, `sudo`, `echo`, `ls`, `uv`, `dist/torch-*.whl`, `dist/vision/torchvision-*.whl`, `dist/audio/torchaudio-*.whl`, ...
- Environment variables / 环境变量: `JSON`, `TRANSFORMERS_OFFLINE`, `VLLM_PINNED_COMMIT`, `GITHUB_OUTPUT`, `AWS`, `PYTORCH_WHL`, `VLLM_WHL`, `GITHUB_STEP_SUMMARY`, `DEVICE_NAME`, `GITHUB_ENV`, ...
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `inputs`, `jobs`, `benchmark`, `permissions`, `container`, `steps`
