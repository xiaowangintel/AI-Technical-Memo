# generated-macos-arm64-binary-wheel-nightly.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/generated-macos-arm64-binary-wheel-nightly.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-20 / 第 1-20 行

````yaml
# @generated DO NOT EDIT MANUALLY

# Template is at:    .github/templates/macos_binary_build_workflow.yml.j2
# Generation script: .github/scripts/generate_ci_workflows.py
name: macos-arm64-binary-wheel

on:
# TODO: Migrate to new ciflow trigger, reference https://github.com/pytorch/pytorch/pull/70321
  push:
    # NOTE: Meta Employees can trigger new nightlies using: https://fburl.com/trigger_pytorch_nightly_build
    branches:
      - nightly
    tags:
      # NOTE: Binary build pipelines should only get triggered on release candidate builds
      # Release candidate tags look like: v1.11.0-rc1
      - v[0-9]+.[0-9]+.[0-9]+-rc[0-9]+
      - 'ciflow/binaries/*'
      - 'ciflow/binaries_libtorch/*'
      - 'ciflow/binaries_wheel/*'
  workflow_dispatch:
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 22-41 / 第 22-41 行

````yaml
env:
  ALPINE_IMAGE: "308535385114.dkr.ecr.us-east-1.amazonaws.com/tool/alpine"
  AWS_DEFAULT_REGION: us-east-1
  BUILD_ENVIRONMENT: macos-arm64-binary-wheel
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  PR_NUMBER: ${{ github.event.pull_request.number }}
  SKIP_ALL_TESTS: 0
concurrency:
  group: macos-arm64-binary-wheel-${{ github.event.pull_request.number || github.ref_name }}-${{ github.ref_type == 'branch' && github.sha }}-${{ github.event_name == 'workflow_dispatch' }}
  cancel-in-progress: true

jobs:
  wheel-py3_10-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    runs-on: macos-26-xlarge
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 42-63 / 第 42-63 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.10"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          # shellcheck disable=SC2129
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          # shellcheck disable=SC2129
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          # shellcheck disable=SC2129
          echo "MAC_PACKAGE_WORK_DIR=${RUNNER_TEMP}" >> "${GITHUB_ENV}"
      - name: Setup Python
        uses: actions/setup-python@v6
        with:
````

- EN: This section reuses actions and step building blocks such as `actions/setup-python@v6`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 64-85 / 第 64-85 行

````yaml
          # .4 version is min minor for 3.10, and also no-gil version of 3.13 needs at least 3.13.3
          python-version: "3.10.4"
          freethreaded: false
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        run: |
          set -eux -o pipefail
          # shellcheck disable=SC1090
          source "${BINARY_ENV_FILE:-/Users/distiller/project/env}"
          mkdir -p "$PYTORCH_FINAL_PACKAGE_DIR"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 87-106 / 第 87-106 行

````yaml
          # Build
          USE_PYTORCH_METAL_EXPORT=1
          USE_COREML_DELEGATE=1
          TORCH_PACKAGE_NAME="${TORCH_PACKAGE_NAME//-/_}"
          export USE_PYTORCH_METAL_EXPORT
          export USE_COREML_DELEGATE
          export TORCH_PACKAGE_NAME
          "${PYTORCH_ROOT}/.ci/wheel/install_libomp.sh"
          "${PYTORCH_ROOT}/.ci/wheel/build_wheel.sh"
      - name: Test PyTorch wheel
        run: |
          set -eux -o pipefail
          # shellcheck disable=SC1090
          source "${BINARY_ENV_FILE:-/Users/distiller/project/env}"
          pip uninstall -y "$TORCH_PACKAGE_NAME" || true
          pip uninstall -y "$TORCH_PACKAGE_NAME" || true

          # Create new "clean" conda environment for testing

          SMOKE_TEST_PARAMS=""
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 108-130 / 第 108-130 行

````yaml
          # shellcheck disable=SC2086
          python -mvenv test_venv
          source test_venv/bin/activate
          pip install "$PYTORCH_FINAL_PACKAGE_DIR"/*.whl numpy -v

          # shellcheck disable=SC2086
          python "${PYTORCH_ROOT}/.ci/pytorch/smoke_test/smoke_test.py" --package torchonly ${SMOKE_TEST_PARAMS}
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_10-cpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
  wheel-py3_10-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_10-cpu-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: wheel
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 131-153 / 第 131-153 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.10"
      build_name: wheel-py3_10-cpu
      use_s3: False
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_11-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    runs-on: macos-26-xlarge
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 154-175 / 第 154-175 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.11"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          # shellcheck disable=SC2129
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          # shellcheck disable=SC2129
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          # shellcheck disable=SC2129
          echo "MAC_PACKAGE_WORK_DIR=${RUNNER_TEMP}" >> "${GITHUB_ENV}"
      - name: Setup Python
        uses: actions/setup-python@v6
        with:
````

- EN: This section reuses actions and step building blocks such as `actions/setup-python@v6`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 176-197 / 第 176-197 行

````yaml
          # .4 version is min minor for 3.10, and also no-gil version of 3.13 needs at least 3.13.3
          python-version: "3.11.4"
          freethreaded: false
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        run: |
          set -eux -o pipefail
          # shellcheck disable=SC1090
          source "${BINARY_ENV_FILE:-/Users/distiller/project/env}"
          mkdir -p "$PYTORCH_FINAL_PACKAGE_DIR"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 199-218 / 第 199-218 行

````yaml
          # Build
          USE_PYTORCH_METAL_EXPORT=1
          USE_COREML_DELEGATE=1
          TORCH_PACKAGE_NAME="${TORCH_PACKAGE_NAME//-/_}"
          export USE_PYTORCH_METAL_EXPORT
          export USE_COREML_DELEGATE
          export TORCH_PACKAGE_NAME
          "${PYTORCH_ROOT}/.ci/wheel/install_libomp.sh"
          "${PYTORCH_ROOT}/.ci/wheel/build_wheel.sh"
      - name: Test PyTorch wheel
        run: |
          set -eux -o pipefail
          # shellcheck disable=SC1090
          source "${BINARY_ENV_FILE:-/Users/distiller/project/env}"
          pip uninstall -y "$TORCH_PACKAGE_NAME" || true
          pip uninstall -y "$TORCH_PACKAGE_NAME" || true

          # Create new "clean" conda environment for testing

          SMOKE_TEST_PARAMS=""
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 220-242 / 第 220-242 行

````yaml
          # shellcheck disable=SC2086
          python -mvenv test_venv
          source test_venv/bin/activate
          pip install "$PYTORCH_FINAL_PACKAGE_DIR"/*.whl numpy -v

          # shellcheck disable=SC2086
          python "${PYTORCH_ROOT}/.ci/pytorch/smoke_test/smoke_test.py" --package torchonly ${SMOKE_TEST_PARAMS}
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_11-cpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
  wheel-py3_11-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_11-cpu-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: wheel
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 243-265 / 第 243-265 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.11"
      build_name: wheel-py3_11-cpu
      use_s3: False
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_12-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    runs-on: macos-26-xlarge
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 266-287 / 第 266-287 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.12"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          # shellcheck disable=SC2129
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          # shellcheck disable=SC2129
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          # shellcheck disable=SC2129
          echo "MAC_PACKAGE_WORK_DIR=${RUNNER_TEMP}" >> "${GITHUB_ENV}"
      - name: Setup Python
        uses: actions/setup-python@v6
        with:
````

- EN: This section reuses actions and step building blocks such as `actions/setup-python@v6`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 288-309 / 第 288-309 行

````yaml
          # .4 version is min minor for 3.10, and also no-gil version of 3.13 needs at least 3.13.3
          python-version: "3.12.4"
          freethreaded: false
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        run: |
          set -eux -o pipefail
          # shellcheck disable=SC1090
          source "${BINARY_ENV_FILE:-/Users/distiller/project/env}"
          mkdir -p "$PYTORCH_FINAL_PACKAGE_DIR"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 311-330 / 第 311-330 行

````yaml
          # Build
          USE_PYTORCH_METAL_EXPORT=1
          USE_COREML_DELEGATE=1
          TORCH_PACKAGE_NAME="${TORCH_PACKAGE_NAME//-/_}"
          export USE_PYTORCH_METAL_EXPORT
          export USE_COREML_DELEGATE
          export TORCH_PACKAGE_NAME
          "${PYTORCH_ROOT}/.ci/wheel/install_libomp.sh"
          "${PYTORCH_ROOT}/.ci/wheel/build_wheel.sh"
      - name: Test PyTorch wheel
        run: |
          set -eux -o pipefail
          # shellcheck disable=SC1090
          source "${BINARY_ENV_FILE:-/Users/distiller/project/env}"
          pip uninstall -y "$TORCH_PACKAGE_NAME" || true
          pip uninstall -y "$TORCH_PACKAGE_NAME" || true

          # Create new "clean" conda environment for testing

          SMOKE_TEST_PARAMS=""
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 332-354 / 第 332-354 行

````yaml
          # shellcheck disable=SC2086
          python -mvenv test_venv
          source test_venv/bin/activate
          pip install "$PYTORCH_FINAL_PACKAGE_DIR"/*.whl numpy -v

          # shellcheck disable=SC2086
          python "${PYTORCH_ROOT}/.ci/pytorch/smoke_test/smoke_test.py" --package torchonly ${SMOKE_TEST_PARAMS}
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_12-cpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
  wheel-py3_12-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_12-cpu-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: wheel
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 355-377 / 第 355-377 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.12"
      build_name: wheel-py3_12-cpu
      use_s3: False
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_13-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    runs-on: macos-26-xlarge
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 378-399 / 第 378-399 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          # shellcheck disable=SC2129
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          # shellcheck disable=SC2129
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          # shellcheck disable=SC2129
          echo "MAC_PACKAGE_WORK_DIR=${RUNNER_TEMP}" >> "${GITHUB_ENV}"
      - name: Setup Python
        uses: actions/setup-python@v6
        with:
````

- EN: This section reuses actions and step building blocks such as `actions/setup-python@v6`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 400-421 / 第 400-421 行

````yaml
          # .4 version is min minor for 3.10, and also no-gil version of 3.13 needs at least 3.13.3
          python-version: "3.13.4"
          freethreaded: false
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        run: |
          set -eux -o pipefail
          # shellcheck disable=SC1090
          source "${BINARY_ENV_FILE:-/Users/distiller/project/env}"
          mkdir -p "$PYTORCH_FINAL_PACKAGE_DIR"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 423-442 / 第 423-442 行

````yaml
          # Build
          USE_PYTORCH_METAL_EXPORT=1
          USE_COREML_DELEGATE=1
          TORCH_PACKAGE_NAME="${TORCH_PACKAGE_NAME//-/_}"
          export USE_PYTORCH_METAL_EXPORT
          export USE_COREML_DELEGATE
          export TORCH_PACKAGE_NAME
          "${PYTORCH_ROOT}/.ci/wheel/install_libomp.sh"
          "${PYTORCH_ROOT}/.ci/wheel/build_wheel.sh"
      - name: Test PyTorch wheel
        run: |
          set -eux -o pipefail
          # shellcheck disable=SC1090
          source "${BINARY_ENV_FILE:-/Users/distiller/project/env}"
          pip uninstall -y "$TORCH_PACKAGE_NAME" || true
          pip uninstall -y "$TORCH_PACKAGE_NAME" || true

          # Create new "clean" conda environment for testing

          SMOKE_TEST_PARAMS=""
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 444-466 / 第 444-466 行

````yaml
          # shellcheck disable=SC2086
          python -mvenv test_venv
          source test_venv/bin/activate
          pip install "$PYTORCH_FINAL_PACKAGE_DIR"/*.whl numpy -v

          # shellcheck disable=SC2086
          python "${PYTORCH_ROOT}/.ci/pytorch/smoke_test/smoke_test.py" --package torchonly ${SMOKE_TEST_PARAMS}
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_13-cpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
  wheel-py3_13-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_13-cpu-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: wheel
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 467-489 / 第 467-489 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.13"
      build_name: wheel-py3_13-cpu
      use_s3: False
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_13t-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    runs-on: macos-26-xlarge
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 490-511 / 第 490-511 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13t"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          # shellcheck disable=SC2129
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          # shellcheck disable=SC2129
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          # shellcheck disable=SC2129
          echo "MAC_PACKAGE_WORK_DIR=${RUNNER_TEMP}" >> "${GITHUB_ENV}"
      - name: Setup Python
        uses: actions/setup-python@v6
        with:
````

- EN: This section reuses actions and step building blocks such as `actions/setup-python@v6`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 512-533 / 第 512-533 行

````yaml
          # .4 version is min minor for 3.10, and also no-gil version of 3.13 needs at least 3.13.3
          python-version: "3.13.4"
          freethreaded: true
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        run: |
          set -eux -o pipefail
          # shellcheck disable=SC1090
          source "${BINARY_ENV_FILE:-/Users/distiller/project/env}"
          mkdir -p "$PYTORCH_FINAL_PACKAGE_DIR"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 535-554 / 第 535-554 行

````yaml
          # Build
          USE_PYTORCH_METAL_EXPORT=1
          USE_COREML_DELEGATE=1
          TORCH_PACKAGE_NAME="${TORCH_PACKAGE_NAME//-/_}"
          export USE_PYTORCH_METAL_EXPORT
          export USE_COREML_DELEGATE
          export TORCH_PACKAGE_NAME
          "${PYTORCH_ROOT}/.ci/wheel/install_libomp.sh"
          "${PYTORCH_ROOT}/.ci/wheel/build_wheel.sh"
      - name: Test PyTorch wheel
        run: |
          set -eux -o pipefail
          # shellcheck disable=SC1090
          source "${BINARY_ENV_FILE:-/Users/distiller/project/env}"
          pip uninstall -y "$TORCH_PACKAGE_NAME" || true
          pip uninstall -y "$TORCH_PACKAGE_NAME" || true

          # Create new "clean" conda environment for testing

          SMOKE_TEST_PARAMS=""
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 556-578 / 第 556-578 行

````yaml
          # shellcheck disable=SC2086
          python -mvenv test_venv
          source test_venv/bin/activate
          pip install "$PYTORCH_FINAL_PACKAGE_DIR"/*.whl numpy -v

          # shellcheck disable=SC2086
          python "${PYTORCH_ROOT}/.ci/pytorch/smoke_test/smoke_test.py" --package torchonly ${SMOKE_TEST_PARAMS}
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_13t-cpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
  wheel-py3_13t-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_13t-cpu-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: wheel
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 579-601 / 第 579-601 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.13t"
      build_name: wheel-py3_13t-cpu
      use_s3: False
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_14-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    runs-on: macos-26-xlarge
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 602-623 / 第 602-623 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          # shellcheck disable=SC2129
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          # shellcheck disable=SC2129
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          # shellcheck disable=SC2129
          echo "MAC_PACKAGE_WORK_DIR=${RUNNER_TEMP}" >> "${GITHUB_ENV}"
      - name: Setup Python
        uses: actions/setup-python@v6
        with:
````

- EN: This section reuses actions and step building blocks such as `actions/setup-python@v6`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 624-645 / 第 624-645 行

````yaml
          # .4 version is min minor for 3.10, and also no-gil version of 3.13 needs at least 3.13.3
          python-version: "3.14.0"
          freethreaded: false
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        run: |
          set -eux -o pipefail
          # shellcheck disable=SC1090
          source "${BINARY_ENV_FILE:-/Users/distiller/project/env}"
          mkdir -p "$PYTORCH_FINAL_PACKAGE_DIR"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 647-666 / 第 647-666 行

````yaml
          # Build
          USE_PYTORCH_METAL_EXPORT=1
          USE_COREML_DELEGATE=1
          TORCH_PACKAGE_NAME="${TORCH_PACKAGE_NAME//-/_}"
          export USE_PYTORCH_METAL_EXPORT
          export USE_COREML_DELEGATE
          export TORCH_PACKAGE_NAME
          "${PYTORCH_ROOT}/.ci/wheel/install_libomp.sh"
          "${PYTORCH_ROOT}/.ci/wheel/build_wheel.sh"
      - name: Test PyTorch wheel
        run: |
          set -eux -o pipefail
          # shellcheck disable=SC1090
          source "${BINARY_ENV_FILE:-/Users/distiller/project/env}"
          pip uninstall -y "$TORCH_PACKAGE_NAME" || true
          pip uninstall -y "$TORCH_PACKAGE_NAME" || true

          # Create new "clean" conda environment for testing

          SMOKE_TEST_PARAMS=""
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 668-690 / 第 668-690 行

````yaml
          # shellcheck disable=SC2086
          python -mvenv test_venv
          source test_venv/bin/activate
          pip install "$PYTORCH_FINAL_PACKAGE_DIR"/*.whl numpy -v

          # shellcheck disable=SC2086
          python "${PYTORCH_ROOT}/.ci/pytorch/smoke_test/smoke_test.py" --package torchonly ${SMOKE_TEST_PARAMS}
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_14-cpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
  wheel-py3_14-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_14-cpu-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: wheel
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 691-713 / 第 691-713 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.14"
      build_name: wheel-py3_14-cpu
      use_s3: False
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_14t-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    runs-on: macos-26-xlarge
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 714-735 / 第 714-735 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14t"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          # shellcheck disable=SC2129
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          # shellcheck disable=SC2129
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          # shellcheck disable=SC2129
          echo "MAC_PACKAGE_WORK_DIR=${RUNNER_TEMP}" >> "${GITHUB_ENV}"
      - name: Setup Python
        uses: actions/setup-python@v6
        with:
````

- EN: This section reuses actions and step building blocks such as `actions/setup-python@v6`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 736-757 / 第 736-757 行

````yaml
          # .4 version is min minor for 3.10, and also no-gil version of 3.13 needs at least 3.13.3
          python-version: "3.14.0"
          freethreaded: true
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        run: |
          set -eux -o pipefail
          # shellcheck disable=SC1090
          source "${BINARY_ENV_FILE:-/Users/distiller/project/env}"
          mkdir -p "$PYTORCH_FINAL_PACKAGE_DIR"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 759-778 / 第 759-778 行

````yaml
          # Build
          USE_PYTORCH_METAL_EXPORT=1
          USE_COREML_DELEGATE=1
          TORCH_PACKAGE_NAME="${TORCH_PACKAGE_NAME//-/_}"
          export USE_PYTORCH_METAL_EXPORT
          export USE_COREML_DELEGATE
          export TORCH_PACKAGE_NAME
          "${PYTORCH_ROOT}/.ci/wheel/install_libomp.sh"
          "${PYTORCH_ROOT}/.ci/wheel/build_wheel.sh"
      - name: Test PyTorch wheel
        run: |
          set -eux -o pipefail
          # shellcheck disable=SC1090
          source "${BINARY_ENV_FILE:-/Users/distiller/project/env}"
          pip uninstall -y "$TORCH_PACKAGE_NAME" || true
          pip uninstall -y "$TORCH_PACKAGE_NAME" || true

          # Create new "clean" conda environment for testing

          SMOKE_TEST_PARAMS=""
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 780-802 / 第 780-802 行

````yaml
          # shellcheck disable=SC2086
          python -mvenv test_venv
          source test_venv/bin/activate
          pip install "$PYTORCH_FINAL_PACKAGE_DIR"/*.whl numpy -v

          # shellcheck disable=SC2086
          python "${PYTORCH_ROOT}/.ci/pytorch/smoke_test/smoke_test.py" --package torchonly ${SMOKE_TEST_PARAMS}
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_14t-cpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
  wheel-py3_14t-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_14t-cpu-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: wheel
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 803-825 / 第 803-825 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.14t"
      build_name: wheel-py3_14t-cpu
      use_s3: False
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  libtorch-cpu-shared-with-deps-release-extract:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: wheel-py3_10-cpu-build
    runs-on: macos-26-xlarge
    timeout-minutes: 60
    env:
      DESIRED_CUDA: cpu
      LIBTORCH_VARIANT: shared-with-deps
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 826-843 / 第 826-843 行

````yaml
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          sparse-checkout: .ci/libtorch/
          show-progress: false
      - name: Setup Python
        uses: actions/setup-python@v6
        with:
          # .4 version is min minor for 3.10, and also no-gil version of 3.13 needs at least 3.13.3
          python-version: "3.10.4"
          freethreaded: false
      - uses: actions/download-artifact@v4.1.7
        name: Download Wheel Artifact
        with:
          name: wheel-py3_10-cpu
          path: "${{ runner.temp }}/wheel_artifact/"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`, `actions/setup-python@v6`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 844-866 / 第 844-866 行

````yaml
      - name: Extract libtorch from wheel
        run: |
          set -eux
          mkdir -p "${{ runner.temp }}/libtorch_output"
          python3 .ci/libtorch/extract_libtorch_from_wheel.py \
            --wheel-dir "${{ runner.temp }}/wheel_artifact" \
            --output-dir "${{ runner.temp }}/libtorch_output" \
            --platform macos \
            --desired-cuda "$DESIRED_CUDA" \
            --libtorch-variant "$LIBTORCH_VARIANT"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: libtorch-cpu-shared-with-deps-release
          retention-days: 14
          if-no-files-found: error
          path: "${{ runner.temp }}/libtorch_output/"
  libtorch-cpu-shared-with-deps-release-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: libtorch-cpu-shared-with-deps-release-extract
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 867-875 / 第 867-875 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: libtorch
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      LIBTORCH_CONFIG: release
      LIBTORCH_VARIANT: shared-with-deps
      build_name: libtorch-cpu-shared-with-deps-release
      use_s3: False
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 876-881 / 第 876-881 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。


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
- Reusable actions / 复用 Action: `actions/setup-python@v6`, `actions/checkout@v4`, `./.github/workflows/_binary-upload.yml`
- Inline commands / 内联命令: `echo`, `git`, `"${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"`, `set`, `source`, `mkdir`, `"${PYTORCH_ROOT}/.ci/wheel/install_libomp.sh"`, `"${PYTORCH_ROOT}/.ci/wheel/build_wheel.sh"`, ...
- Environment variables / 环境变量: `NOT`, `EDIT`, `MANUALLY`, `TODO`, `NOTE`, `ALPINE_IMAGE`, `AWS_DEFAULT_REGION`, `BUILD_ENVIRONMENT`, `GITHUB_TOKEN`, `PR_NUMBER`, ...
- Named jobs or sections / 命名作业或章节: `on`, `push`, `branches`, `tags`, `workflow_dispatch`, `env`, `concurrency`, `jobs`, `wheel-py3_10-cpu-build`, `steps`, ...
