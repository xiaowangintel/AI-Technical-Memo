# generated-linux-aarch64-binary-manywheel-nightly.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/generated-linux-aarch64-binary-manywheel-nightly.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-22 / 第 1-22 行

````yaml
# @generated DO NOT EDIT MANUALLY

# Template is at:    .github/templates/linux_binary_build_workflow.yml.j2
# Generation script: .github/scripts/generate_ci_workflows.py
name: linux-aarch64-binary-manywheel


on:
  push:
    # NOTE: Meta Employees can trigger new nightlies using: https://fburl.com/trigger_pytorch_nightly_build
    branches:
      - nightly
    tags:
      # NOTE: Binary build pipelines should only get triggered on release candidate builds
      # Release candidate tags look like: v1.11.0-rc1
      - v[0-9]+.[0-9]+.[0-9]+-rc[0-9]+
      - 'ciflow/binaries/*'
      - 'ciflow/binaries_wheel/*'
  workflow_dispatch:

permissions:
  id-token: write
````

- EN: This section declares workflow triggers or event bindings; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；控制令牌权限范围或环境变量以实现安全执行。

### Lines 24-44 / 第 24-44 行

````yaml
env:
  # Needed for conda builds
  ALPINE_IMAGE: "arm64v8/alpine"
  AWS_DEFAULT_REGION: us-east-1
  BINARY_ENV_FILE: /tmp/env
  BUILD_ENVIRONMENT: linux-aarch64-binary-manywheel
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  PR_NUMBER: ${{ github.event.pull_request.number }}
  PYTORCH_FINAL_PACKAGE_DIR: /artifacts
  PYTORCH_ROOT: /pytorch
  SHA1: ${{ github.event.pull_request.head.sha || github.sha }}
  SKIP_ALL_TESTS: 0
concurrency:
  group: linux-aarch64-binary-manywheel-${{ github.event.pull_request.number || github.ref_name }}-${{ github.ref_type == 'branch' && github.sha }}-${{ github.event_name == 'workflow_dispatch' }}
  cancel-in-progress: true

jobs:
  get-label-type:
    if: github.repository_owner == 'pytorch'
    name: get-label-type
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 45-68 / 第 45-68 行

````yaml
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}
  manywheel-py3_10-cpu-aarch64-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.10"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_10-cpu-aarch64
      build_environment: linux-aarch64-binary-manywheel
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 69-92 / 第 69-92 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_10-cpu-aarch64-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-cpu-aarch64-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cpu-aarch64
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 93-111 / 第 93-111 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_10-cpu-aarch64-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_10-cpu-aarch64-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cpu-aarch64
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 112-126 / 第 112-126 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_10-cuda-aarch64-12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 127-149 / 第 127-149 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.10"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_10-cuda-aarch64-12_6
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==12.6.3; platform_system == 'Linux' | cuda-bindings>=12.9.4,<13; platform_system == 'Linux' | nvidia-cudnn-cu12==9.10.2.21; platform_system == 'Linux' | nvidia-cusparselt-cu12==0.7.1; platform_system == 'Linux' | nvidia-nccl-cu12==2.29.3; platform_system == 'Linux' | nvidia-nvshmem-cu12==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_10-cuda-aarch64-12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-cuda-aarch64-12_6-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 150-169 / 第 150-169 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cuda-aarch64-12_6
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_10-cuda-aarch64-12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 170-191 / 第 170-191 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_10-cuda-aarch64-12_6-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cuda-aarch64-12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 193-214 / 第 193-214 行

````yaml
  manywheel-py3_10-cuda-aarch64-13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.10"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_10-cuda-aarch64-13_0
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 215-238 / 第 215-238 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_10-cuda-aarch64-13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-cuda-aarch64-13_0-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cuda-aarch64-13_0
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 239-259 / 第 239-259 行

````yaml
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_10-cuda-aarch64-13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_10-cuda-aarch64-13_0-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cuda-aarch64-13_0
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 260-274 / 第 260-274 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_10-cuda-aarch64-13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 275-297 / 第 275-297 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.10"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_10-cuda-aarch64-13_2
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==13.2.1; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_10-cuda-aarch64-13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-cuda-aarch64-13_2-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 298-317 / 第 298-317 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cuda-aarch64-13_2
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_10-cuda-aarch64-13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 318-339 / 第 318-339 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_10-cuda-aarch64-13_2-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cuda-aarch64-13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 341-361 / 第 341-361 行

````yaml
  manywheel-py3_11-cpu-aarch64-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.11"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_11-cpu-aarch64
      build_environment: linux-aarch64-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 363-383 / 第 363-383 行

````yaml
  manywheel-py3_11-cpu-aarch64-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_11-cpu-aarch64-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cpu-aarch64
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 384-402 / 第 384-402 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_11-cpu-aarch64-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_11-cpu-aarch64-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cpu-aarch64
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 403-417 / 第 403-417 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_11-cuda-aarch64-12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 418-440 / 第 418-440 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.11"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_11-cuda-aarch64-12_6
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==12.6.3; platform_system == 'Linux' | cuda-bindings>=12.9.4,<13; platform_system == 'Linux' | nvidia-cudnn-cu12==9.10.2.21; platform_system == 'Linux' | nvidia-cusparselt-cu12==0.7.1; platform_system == 'Linux' | nvidia-nccl-cu12==2.29.3; platform_system == 'Linux' | nvidia-nvshmem-cu12==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_11-cuda-aarch64-12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_11-cuda-aarch64-12_6-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 441-460 / 第 441-460 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cuda-aarch64-12_6
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_11-cuda-aarch64-12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 461-482 / 第 461-482 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_11-cuda-aarch64-12_6-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cuda-aarch64-12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 484-505 / 第 484-505 行

````yaml
  manywheel-py3_11-cuda-aarch64-13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.11"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_11-cuda-aarch64-13_0
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 506-529 / 第 506-529 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_11-cuda-aarch64-13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_11-cuda-aarch64-13_0-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cuda-aarch64-13_0
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 530-550 / 第 530-550 行

````yaml
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_11-cuda-aarch64-13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_11-cuda-aarch64-13_0-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cuda-aarch64-13_0
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 551-565 / 第 551-565 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_11-cuda-aarch64-13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 566-588 / 第 566-588 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.11"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_11-cuda-aarch64-13_2
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==13.2.1; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_11-cuda-aarch64-13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_11-cuda-aarch64-13_2-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 589-608 / 第 589-608 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cuda-aarch64-13_2
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_11-cuda-aarch64-13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 609-630 / 第 609-630 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_11-cuda-aarch64-13_2-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cuda-aarch64-13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 632-652 / 第 632-652 行

````yaml
  manywheel-py3_12-cpu-aarch64-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.12"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_12-cpu-aarch64
      build_environment: linux-aarch64-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 654-674 / 第 654-674 行

````yaml
  manywheel-py3_12-cpu-aarch64-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_12-cpu-aarch64-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cpu-aarch64
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 675-693 / 第 675-693 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_12-cpu-aarch64-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_12-cpu-aarch64-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cpu-aarch64
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 694-708 / 第 694-708 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_12-cuda-aarch64-12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 709-731 / 第 709-731 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.12"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_12-cuda-aarch64-12_6
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==12.6.3; platform_system == 'Linux' | cuda-bindings>=12.9.4,<13; platform_system == 'Linux' | nvidia-cudnn-cu12==9.10.2.21; platform_system == 'Linux' | nvidia-cusparselt-cu12==0.7.1; platform_system == 'Linux' | nvidia-nccl-cu12==2.29.3; platform_system == 'Linux' | nvidia-nvshmem-cu12==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_12-cuda-aarch64-12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_12-cuda-aarch64-12_6-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 732-751 / 第 732-751 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cuda-aarch64-12_6
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_12-cuda-aarch64-12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 752-773 / 第 752-773 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_12-cuda-aarch64-12_6-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cuda-aarch64-12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 775-796 / 第 775-796 行

````yaml
  manywheel-py3_12-cuda-aarch64-13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.12"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_12-cuda-aarch64-13_0
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 797-820 / 第 797-820 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_12-cuda-aarch64-13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_12-cuda-aarch64-13_0-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cuda-aarch64-13_0
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 821-841 / 第 821-841 行

````yaml
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_12-cuda-aarch64-13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_12-cuda-aarch64-13_0-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cuda-aarch64-13_0
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 842-856 / 第 842-856 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_12-cuda-aarch64-13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 857-879 / 第 857-879 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.12"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_12-cuda-aarch64-13_2
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==13.2.1; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_12-cuda-aarch64-13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_12-cuda-aarch64-13_2-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 880-899 / 第 880-899 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cuda-aarch64-13_2
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_12-cuda-aarch64-13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 900-921 / 第 900-921 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_12-cuda-aarch64-13_2-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cuda-aarch64-13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 923-943 / 第 923-943 行

````yaml
  manywheel-py3_13-cpu-aarch64-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.13"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_13-cpu-aarch64
      build_environment: linux-aarch64-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 945-965 / 第 945-965 行

````yaml
  manywheel-py3_13-cpu-aarch64-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13-cpu-aarch64-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cpu-aarch64
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 966-984 / 第 966-984 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13-cpu-aarch64-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13-cpu-aarch64-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cpu-aarch64
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 985-999 / 第 985-999 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_13-cuda-aarch64-12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1000-1022 / 第 1000-1022 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.13"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_13-cuda-aarch64-12_6
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==12.6.3; platform_system == 'Linux' | cuda-bindings>=12.9.4,<13; platform_system == 'Linux' | nvidia-cudnn-cu12==9.10.2.21; platform_system == 'Linux' | nvidia-cusparselt-cu12==0.7.1; platform_system == 'Linux' | nvidia-nccl-cu12==2.29.3; platform_system == 'Linux' | nvidia-nvshmem-cu12==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13-cuda-aarch64-12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13-cuda-aarch64-12_6-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 1023-1042 / 第 1023-1042 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cuda-aarch64-12_6
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13-cuda-aarch64-12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 1043-1064 / 第 1043-1064 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13-cuda-aarch64-12_6-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cuda-aarch64-12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1066-1087 / 第 1066-1087 行

````yaml
  manywheel-py3_13-cuda-aarch64-13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.13"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_13-cuda-aarch64-13_0
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1088-1111 / 第 1088-1111 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13-cuda-aarch64-13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13-cuda-aarch64-13_0-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cuda-aarch64-13_0
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1112-1132 / 第 1112-1132 行

````yaml
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13-cuda-aarch64-13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13-cuda-aarch64-13_0-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cuda-aarch64-13_0
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1133-1147 / 第 1133-1147 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_13-cuda-aarch64-13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1148-1170 / 第 1148-1170 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.13"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_13-cuda-aarch64-13_2
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==13.2.1; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13-cuda-aarch64-13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13-cuda-aarch64-13_2-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 1171-1190 / 第 1171-1190 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cuda-aarch64-13_2
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13-cuda-aarch64-13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 1191-1212 / 第 1191-1212 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13-cuda-aarch64-13_2-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cuda-aarch64-13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1214-1234 / 第 1214-1234 行

````yaml
  manywheel-py3_13t-cpu-aarch64-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.13t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_13t-cpu-aarch64
      build_environment: linux-aarch64-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1236-1256 / 第 1236-1256 行

````yaml
  manywheel-py3_13t-cpu-aarch64-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13t-cpu-aarch64-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cpu-aarch64
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1257-1275 / 第 1257-1275 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13t-cpu-aarch64-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13t-cpu-aarch64-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cpu-aarch64
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1276-1290 / 第 1276-1290 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_13t-cuda-aarch64-12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1291-1313 / 第 1291-1313 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.13t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_13t-cuda-aarch64-12_6
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==12.6.3; platform_system == 'Linux' | cuda-bindings>=12.9.4,<13; platform_system == 'Linux' | nvidia-cudnn-cu12==9.10.2.21; platform_system == 'Linux' | nvidia-cusparselt-cu12==0.7.1; platform_system == 'Linux' | nvidia-nccl-cu12==2.29.3; platform_system == 'Linux' | nvidia-nvshmem-cu12==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13t-cuda-aarch64-12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13t-cuda-aarch64-12_6-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 1314-1333 / 第 1314-1333 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cuda-aarch64-12_6
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13t-cuda-aarch64-12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 1334-1355 / 第 1334-1355 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13t-cuda-aarch64-12_6-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cuda-aarch64-12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1357-1378 / 第 1357-1378 行

````yaml
  manywheel-py3_13t-cuda-aarch64-13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.13t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_13t-cuda-aarch64-13_0
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1379-1402 / 第 1379-1402 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13t-cuda-aarch64-13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13t-cuda-aarch64-13_0-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cuda-aarch64-13_0
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1403-1423 / 第 1403-1423 行

````yaml
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13t-cuda-aarch64-13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13t-cuda-aarch64-13_0-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cuda-aarch64-13_0
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1424-1438 / 第 1424-1438 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_13t-cuda-aarch64-13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1439-1461 / 第 1439-1461 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.13t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_13t-cuda-aarch64-13_2
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==13.2.1; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13t-cuda-aarch64-13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13t-cuda-aarch64-13_2-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 1462-1481 / 第 1462-1481 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cuda-aarch64-13_2
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13t-cuda-aarch64-13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 1482-1503 / 第 1482-1503 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13t-cuda-aarch64-13_2-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cuda-aarch64-13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1505-1525 / 第 1505-1525 行

````yaml
  manywheel-py3_14-cpu-aarch64-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.14"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_14-cpu-aarch64
      build_environment: linux-aarch64-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1527-1547 / 第 1527-1547 行

````yaml
  manywheel-py3_14-cpu-aarch64-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14-cpu-aarch64-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cpu-aarch64
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1548-1566 / 第 1548-1566 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14-cpu-aarch64-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14-cpu-aarch64-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cpu-aarch64
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1567-1581 / 第 1567-1581 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_14-cuda-aarch64-12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1582-1604 / 第 1582-1604 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.14"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_14-cuda-aarch64-12_6
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==12.6.3; platform_system == 'Linux' | cuda-bindings>=12.9.4,<13; platform_system == 'Linux' | nvidia-cudnn-cu12==9.10.2.21; platform_system == 'Linux' | nvidia-cusparselt-cu12==0.7.1; platform_system == 'Linux' | nvidia-nccl-cu12==2.29.3; platform_system == 'Linux' | nvidia-nvshmem-cu12==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14-cuda-aarch64-12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14-cuda-aarch64-12_6-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 1605-1624 / 第 1605-1624 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cuda-aarch64-12_6
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14-cuda-aarch64-12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 1625-1646 / 第 1625-1646 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14-cuda-aarch64-12_6-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cuda-aarch64-12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1648-1669 / 第 1648-1669 行

````yaml
  manywheel-py3_14-cuda-aarch64-13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.14"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_14-cuda-aarch64-13_0
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1670-1693 / 第 1670-1693 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14-cuda-aarch64-13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14-cuda-aarch64-13_0-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cuda-aarch64-13_0
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1694-1714 / 第 1694-1714 行

````yaml
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14-cuda-aarch64-13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14-cuda-aarch64-13_0-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cuda-aarch64-13_0
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1715-1729 / 第 1715-1729 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_14-cuda-aarch64-13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1730-1752 / 第 1730-1752 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.14"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_14-cuda-aarch64-13_2
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==13.2.1; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14-cuda-aarch64-13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14-cuda-aarch64-13_2-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 1753-1772 / 第 1753-1772 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cuda-aarch64-13_2
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14-cuda-aarch64-13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 1773-1794 / 第 1773-1794 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14-cuda-aarch64-13_2-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cuda-aarch64-13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1796-1816 / 第 1796-1816 行

````yaml
  manywheel-py3_14t-cpu-aarch64-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.14t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_14t-cpu-aarch64
      build_environment: linux-aarch64-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1818-1838 / 第 1818-1838 行

````yaml
  manywheel-py3_14t-cpu-aarch64-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14t-cpu-aarch64-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cpu-aarch64
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1839-1857 / 第 1839-1857 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14t-cpu-aarch64-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14t-cpu-aarch64-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-aarch64
      DOCKER_IMAGE: manylinux2_28_aarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-aarch64
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cpu-aarch64
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1858-1872 / 第 1858-1872 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_14t-cuda-aarch64-12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1873-1895 / 第 1873-1895 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.14t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_14t-cuda-aarch64-12_6
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==12.6.3; platform_system == 'Linux' | cuda-bindings>=12.9.4,<13; platform_system == 'Linux' | nvidia-cudnn-cu12==9.10.2.21; platform_system == 'Linux' | nvidia-cusparselt-cu12==0.7.1; platform_system == 'Linux' | nvidia-nccl-cu12==2.29.3; platform_system == 'Linux' | nvidia-nvshmem-cu12==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14t-cuda-aarch64-12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14t-cuda-aarch64-12_6-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 1896-1915 / 第 1896-1915 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cuda-aarch64-12_6
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14t-cuda-aarch64-12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 1916-1937 / 第 1916-1937 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14t-cuda-aarch64-12_6-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cuda-aarch64-12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1939-1960 / 第 1939-1960 行

````yaml
  manywheel-py3_14t-cuda-aarch64-13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.14t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_14t-cuda-aarch64-13_0
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1961-1984 / 第 1961-1984 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14t-cuda-aarch64-13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14t-cuda-aarch64-13_0-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cuda-aarch64-13_0
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1985-2005 / 第 1985-2005 行

````yaml
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14t-cuda-aarch64-13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14t-cuda-aarch64-13_0-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cuda-aarch64-13_0
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2006-2020 / 第 2006-2020 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_14t-cuda-aarch64-13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2021-2043 / 第 2021-2043 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.14t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.r7g.12xlarge.memory
      ALPINE_IMAGE: "arm64v8/alpine"
      build_name: manywheel-py3_14t-cuda-aarch64-13_2
      build_environment: linux-aarch64-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==13.2.1; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
      timeout-minutes: 420
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14t-cuda-aarch64-13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14t-cuda-aarch64-13_2-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 2044-2063 / 第 2044-2063 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cuda-aarch64-13_2
      build_environment: linux-aarch64-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.arm64.2xlarge
      ALPINE_IMAGE: "arm64v8/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14t-cuda-aarch64-13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 2064-2079 / 第 2064-2079 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14t-cuda-aarch64-13_2-build
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2-aarch64"
      GPU_ARCH_TYPE: cuda-aarch64
      DOCKER_IMAGE: manylinuxaarch64-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cuda-aarch64-13_2
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2080-2085 / 第 2080-2085 行

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
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `./.github/workflows/_binary-build-linux.yml`, `./.github/workflows/_binary-test-linux.yml`, `./.github/workflows/_binary-upload.yml`
- Inline commands / 内联命令: none
- Environment variables / 环境变量: `NOT`, `EDIT`, `MANUALLY`, `NOTE`, `ALPINE_IMAGE`, `AWS_DEFAULT_REGION`, `BINARY_ENV_FILE`, `BUILD_ENVIRONMENT`, `GITHUB_TOKEN`, `PR_NUMBER`, ...
- Named jobs or sections / 命名作业或章节: `on`, `push`, `branches`, `tags`, `workflow_dispatch`, `permissions`, `env`, `concurrency`, `jobs`, `get-label-type`, ...
