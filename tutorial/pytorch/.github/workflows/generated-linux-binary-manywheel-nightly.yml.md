# generated-linux-binary-manywheel-nightly.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/generated-linux-binary-manywheel-nightly.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-23 / 第 1-23 行

````yaml
# @generated DO NOT EDIT MANUALLY

# Template is at:    .github/templates/linux_binary_build_workflow.yml.j2
# Generation script: .github/scripts/generate_ci_workflows.py
name: linux-binary-manywheel


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
      - 'ciflow/binaries_libtorch/*'
      - 'ciflow/binaries_wheel/*'
  workflow_dispatch:

permissions:
  id-token: write
````

- EN: This section declares workflow triggers or event bindings; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；控制令牌权限范围或环境变量以实现安全执行。

### Lines 25-45 / 第 25-45 行

````yaml
env:
  # Needed for conda builds
  ALPINE_IMAGE: "308535385114.dkr.ecr.us-east-1.amazonaws.com/tool/alpine"
  AWS_DEFAULT_REGION: us-east-1
  BINARY_ENV_FILE: /tmp/env
  BUILD_ENVIRONMENT: linux-binary-manywheel
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  PR_NUMBER: ${{ github.event.pull_request.number }}
  PYTORCH_FINAL_PACKAGE_DIR: /artifacts
  PYTORCH_ROOT: /pytorch
  SHA1: ${{ github.event.pull_request.head.sha || github.sha }}
  SKIP_ALL_TESTS: 0
concurrency:
  group: linux-binary-manywheel-${{ github.event.pull_request.number || github.ref_name }}-${{ github.ref_type == 'branch' && github.sha }}-${{ github.event_name == 'workflow_dispatch' }}
  cancel-in-progress: true

jobs:
  get-label-type:
    if: github.repository_owner == 'pytorch'
    name: get-label-type
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 46-67 / 第 46-67 行

````yaml
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}
  manywheel-py3_10-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.10"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_10-cpu
      build_environment: linux-binary-manywheel
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 68-90 / 第 68-90 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_10-cpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-cpu-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cpu
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.4xlarge
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 91-109 / 第 91-109 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_10-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_10-cpu-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cpu
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 110-124 / 第 110-124 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_10-cuda12_6-build:
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

### Lines 125-147 / 第 125-147 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.10"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_10-cuda12_6
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==12.6.3; platform_system == 'Linux' | cuda-bindings>=12.9.4,<13; platform_system == 'Linux' | nvidia-cudnn-cu12==9.10.2.21; platform_system == 'Linux' | nvidia-cusparselt-cu12==0.7.1; platform_system == 'Linux' | nvidia-nccl-cu12==2.29.3; platform_system == 'Linux' | nvidia-nvshmem-cu12==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_10-cuda12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-cuda12_6-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 148-170 / 第 148-170 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cuda12_6
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_10-cuda12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_10-cuda12_6-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 171-193 / 第 171-193 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cuda12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_10-cuda13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 194-214 / 第 194-214 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.10"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_10-cuda13_0
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_10-cuda13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-cuda13_0-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 215-237 / 第 215-237 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cuda13_0
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_10-cuda13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_10-cuda13_0-test
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 238-260 / 第 238-260 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cuda13_0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_10-cuda13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 261-282 / 第 261-282 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.10"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_10-cuda13_2
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==13.2.1; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_10-cuda13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-cuda13_2-build
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 283-303 / 第 283-303 行

````yaml
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cuda13_2
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_10-cuda13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 304-325 / 第 304-325 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_10-cuda13_2-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cuda13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 327-347 / 第 327-347 行

````yaml
  manywheel-py3_10-rocm7_1-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.10"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      timeout-minutes: 420
      build_name: manywheel-py3_10-rocm7_1
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 349-371 / 第 349-371 行

````yaml
  manywheel-py3_10-rocm7_1-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-rocm7_1-build
      - get-label-type
    runs-on: linux.rocm.gpu.gfx942.1
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.10"
    permissions:
      id-token: write
      contents: read
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 372-391 / 第 372-391 行

````yaml
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
      - name: Setup ROCm
        uses: ./.github/actions/setup-rocm
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_10-rocm7_1
          path: "${{ runner.temp }}/artifacts/"
      - name: ROCm set GPU_FLAG
        run: |
          echo "GPU_FLAG=--device=/dev/mem --device=/dev/kfd --device=/dev/dri --group-add video --group-add daemon" >> "${GITHUB_ENV}"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`, `./.github/actions/setup-rocm`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 392-413 / 第 392-413 行

````yaml
      - name: configure aws credentials
        id: aws_creds
        if: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') }}
        uses: aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a # v4
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
          aws-region: us-east-1
          role-duration-seconds: 18000
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: rocm7.1
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
````

- EN: This section reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 414-435 / 第 414-435 行

````yaml
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown ROCm
        uses: ./.github/actions/teardown-rocm
  manywheel-py3_10-rocm7_1-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_10-rocm7_1-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-rocm7_1
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/teardown-rocm`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 436-450 / 第 436-450 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_10-rocm7_2-build:
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

### Lines 451-471 / 第 451-471 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.10"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      timeout-minutes: 420
      build_name: manywheel-py3_10-rocm7_2
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_10-rocm7_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-rocm7_2-build
      - get-label-type
    runs-on: linux.rocm.gpu.gfx942.1
    timeout-minutes: 240
````

- EN: This section describes repository automation behavior for `.github/workflows/generated-linux-binary-manywheel-nightly.yml`.
- CN: 该部分描述 `.github/workflows/generated-linux-binary-manywheel-nightly.yml` 的仓库自动化行为。

### Lines 472-493 / 第 472-493 行

````yaml
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.10"
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 494-515 / 第 494-515 行

````yaml
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Setup ROCm
        uses: ./.github/actions/setup-rocm
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_10-rocm7_2
          path: "${{ runner.temp }}/artifacts/"
      - name: ROCm set GPU_FLAG
        run: |
          echo "GPU_FLAG=--device=/dev/mem --device=/dev/kfd --device=/dev/dri --group-add video --group-add daemon" >> "${GITHUB_ENV}"
      - name: configure aws credentials
        id: aws_creds
        if: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') }}
        uses: aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a # v4
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
          aws-region: us-east-1
          role-duration-seconds: 18000
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/setup-rocm`, `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 516-535 / 第 516-535 行

````yaml
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: rocm7.2
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown ROCm
        uses: ./.github/actions/teardown-rocm
  manywheel-py3_10-rocm7_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/test-pytorch-binary`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 536-557 / 第 536-557 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_10-rocm7_2-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-rocm7_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 559-581 / 第 559-581 行

````yaml
  manywheel-py3_10-xpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.10"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_10-xpu
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: intel-cmplr-lib-rt==2025.3.2 | intel-cmplr-lib-ur==2025.3.2 | intel-cmplr-lic-rt==2025.3.2 | intel-sycl-rt==2025.3.2 | oneccl-devel==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | oneccl==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | impi-rt==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | onemkl-license==2025.3.1 | onemkl-sycl-blas==2025.3.1 | onemkl-sycl-dft==2025.3.1 | onemkl-sycl-lapack==2025.3.1 | onemkl-sycl-rng==2025.3.1 | onemkl-sycl-sparse==2025.3.1 | dpcpp-cpp-rt==2025.3.2 | intel-opencl-rt==2025.3.2 | mkl==2025.3.1 | intel-openmp==2025.3.2 | tbb==2022.3.1 | tcmlib==1.4.1 | umf==1.0.3 | intel-pti==0.16.0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_10-xpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 582-603 / 第 582-603 行

````yaml
    needs:
      - manywheel-py3_10-xpu-build
      - get-label-type
    runs-on: linux.idc.xpu
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.10"
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
````

- EN: This section reuses actions and step building blocks such as `actions/checkout@v4`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 604-623 / 第 604-623 行

````yaml
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Setup XPU
        uses: pytorch/pytorch/.github/actions/setup-xpu@main
      - name: Login to ECR
        uses: pytorch/pytorch/.github/actions/ecr-login@main
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_10-xpu
          path: "${{ runner.temp }}/artifacts/"
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/setup-xpu@main`, `pytorch/pytorch/.github/actions/ecr-login@main`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 624-644 / 第 624-644 行

````yaml
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: xpu
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown XPU
        uses: ./.github/actions/teardown-xpu
  manywheel-py3_10-xpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_10-xpu-test
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/test-pytorch-binary`, `./.github/actions/teardown-xpu`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 645-666 / 第 645-666 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-xpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_11-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 667-688 / 第 667-688 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.11"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_11-cpu
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_11-cpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_11-cpu-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 689-710 / 第 689-710 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cpu
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.4xlarge
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_11-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_11-cpu-test
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 711-732 / 第 711-732 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_11-cuda12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 733-754 / 第 733-754 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.11"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_11-cuda12_6
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==12.6.3; platform_system == 'Linux' | cuda-bindings>=12.9.4,<13; platform_system == 'Linux' | nvidia-cudnn-cu12==9.10.2.21; platform_system == 'Linux' | nvidia-cusparselt-cu12==0.7.1; platform_system == 'Linux' | nvidia-nccl-cu12==2.29.3; platform_system == 'Linux' | nvidia-nvshmem-cu12==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_11-cuda12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_11-cuda12_6-build
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 755-775 / 第 755-775 行

````yaml
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cuda12_6
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_11-cuda12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 776-797 / 第 776-797 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_11-cuda12_6-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cuda12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 799-819 / 第 799-819 行

````yaml
  manywheel-py3_11-cuda13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.11"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_11-cuda13_0
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 821-841 / 第 821-841 行

````yaml
  manywheel-py3_11-cuda13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_11-cuda13_0-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cuda13_0
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 842-861 / 第 842-861 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_11-cuda13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_11-cuda13_0-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cuda13_0
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 862-876 / 第 862-876 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_11-cuda13_2-build:
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

### Lines 877-899 / 第 877-899 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.11"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_11-cuda13_2
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==13.2.1; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_11-cuda13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_11-cuda13_2-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 900-922 / 第 900-922 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cuda13_2
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_11-cuda13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_11-cuda13_2-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 923-945 / 第 923-945 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cuda13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_11-rocm7_1-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 946-967 / 第 946-967 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.11"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      timeout-minutes: 420
      build_name: manywheel-py3_11-rocm7_1
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_11-rocm7_1-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_11-rocm7_1-build
      - get-label-type
    runs-on: linux.rocm.gpu.gfx942.1
    timeout-minutes: 240
````

- EN: This section describes repository automation behavior for `.github/workflows/generated-linux-binary-manywheel-nightly.yml`.
- CN: 该部分描述 `.github/workflows/generated-linux-binary-manywheel-nightly.yml` 的仓库自动化行为。

### Lines 968-989 / 第 968-989 行

````yaml
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.11"
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 990-1011 / 第 990-1011 行

````yaml
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Setup ROCm
        uses: ./.github/actions/setup-rocm
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_11-rocm7_1
          path: "${{ runner.temp }}/artifacts/"
      - name: ROCm set GPU_FLAG
        run: |
          echo "GPU_FLAG=--device=/dev/mem --device=/dev/kfd --device=/dev/dri --group-add video --group-add daemon" >> "${GITHUB_ENV}"
      - name: configure aws credentials
        id: aws_creds
        if: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') }}
        uses: aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a # v4
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
          aws-region: us-east-1
          role-duration-seconds: 18000
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/setup-rocm`, `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1012-1031 / 第 1012-1031 行

````yaml
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: rocm7.1
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown ROCm
        uses: ./.github/actions/teardown-rocm
  manywheel-py3_11-rocm7_1-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/test-pytorch-binary`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1032-1053 / 第 1032-1053 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_11-rocm7_1-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-rocm7_1
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1055-1075 / 第 1055-1075 行

````yaml
  manywheel-py3_11-rocm7_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.11"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      timeout-minutes: 420
      build_name: manywheel-py3_11-rocm7_2
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1077-1099 / 第 1077-1099 行

````yaml
  manywheel-py3_11-rocm7_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_11-rocm7_2-build
      - get-label-type
    runs-on: linux.rocm.gpu.gfx942.1
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.11"
    permissions:
      id-token: write
      contents: read
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1100-1119 / 第 1100-1119 行

````yaml
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
      - name: Setup ROCm
        uses: ./.github/actions/setup-rocm
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_11-rocm7_2
          path: "${{ runner.temp }}/artifacts/"
      - name: ROCm set GPU_FLAG
        run: |
          echo "GPU_FLAG=--device=/dev/mem --device=/dev/kfd --device=/dev/dri --group-add video --group-add daemon" >> "${GITHUB_ENV}"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`, `./.github/actions/setup-rocm`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1120-1141 / 第 1120-1141 行

````yaml
      - name: configure aws credentials
        id: aws_creds
        if: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') }}
        uses: aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a # v4
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
          aws-region: us-east-1
          role-duration-seconds: 18000
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: rocm7.2
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
````

- EN: This section reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1142-1163 / 第 1142-1163 行

````yaml
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown ROCm
        uses: ./.github/actions/teardown-rocm
  manywheel-py3_11-rocm7_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_11-rocm7_2-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-rocm7_2
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/teardown-rocm`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1164-1178 / 第 1164-1178 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_11-xpu-build:
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

### Lines 1179-1201 / 第 1179-1201 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.11"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_11-xpu
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: intel-cmplr-lib-rt==2025.3.2 | intel-cmplr-lib-ur==2025.3.2 | intel-cmplr-lic-rt==2025.3.2 | intel-sycl-rt==2025.3.2 | oneccl-devel==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | oneccl==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | impi-rt==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | onemkl-license==2025.3.1 | onemkl-sycl-blas==2025.3.1 | onemkl-sycl-dft==2025.3.1 | onemkl-sycl-lapack==2025.3.1 | onemkl-sycl-rng==2025.3.1 | onemkl-sycl-sparse==2025.3.1 | dpcpp-cpp-rt==2025.3.2 | intel-opencl-rt==2025.3.2 | mkl==2025.3.1 | intel-openmp==2025.3.2 | tbb==2022.3.1 | tcmlib==1.4.1 | umf==1.0.3 | intel-pti==0.16.0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_11-xpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_11-xpu-build
      - get-label-type
    runs-on: linux.idc.xpu
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section controls token scope or environment variables for safe execution.
- CN: 该部分控制令牌权限范围或环境变量以实现安全执行。

### Lines 1202-1223 / 第 1202-1223 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.11"
    permissions:
      id-token: write
      contents: read
    steps:
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
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1224-1246 / 第 1224-1246 行

````yaml
      - name: Setup XPU
        uses: pytorch/pytorch/.github/actions/setup-xpu@main
      - name: Login to ECR
        uses: pytorch/pytorch/.github/actions/ecr-login@main
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_11-xpu
          path: "${{ runner.temp }}/artifacts/"
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: xpu
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/setup-xpu@main`, `pytorch/pytorch/.github/actions/ecr-login@main`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1247-1267 / 第 1247-1267 行

````yaml
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown XPU
        uses: ./.github/actions/teardown-xpu
  manywheel-py3_11-xpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_11-xpu-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-xpu
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/teardown-xpu`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1268-1282 / 第 1268-1282 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_12-cpu-build:
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

### Lines 1283-1304 / 第 1283-1304 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.12"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_12-cpu
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_12-cpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_12-cpu-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1305-1326 / 第 1305-1326 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cpu
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.4xlarge
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_12-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_12-cpu-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1327-1348 / 第 1327-1348 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_12-cuda12_6-build:
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

### Lines 1349-1371 / 第 1349-1371 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.12"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_12-cuda12_6
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==12.6.3; platform_system == 'Linux' | cuda-bindings>=12.9.4,<13; platform_system == 'Linux' | nvidia-cudnn-cu12==9.10.2.21; platform_system == 'Linux' | nvidia-cusparselt-cu12==0.7.1; platform_system == 'Linux' | nvidia-nccl-cu12==2.29.3; platform_system == 'Linux' | nvidia-nvshmem-cu12==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_12-cuda12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_12-cuda12_6-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1372-1394 / 第 1372-1394 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cuda12_6
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_12-cuda12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_12-cuda12_6-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1395-1417 / 第 1395-1417 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cuda12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_12-cuda13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1418-1438 / 第 1418-1438 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.12"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_12-cuda13_0
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_12-cuda13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_12-cuda13_0-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 1439-1461 / 第 1439-1461 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cuda13_0
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_12-cuda13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_12-cuda13_0-test
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1462-1484 / 第 1462-1484 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cuda13_0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_12-cuda13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1485-1506 / 第 1485-1506 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.12"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_12-cuda13_2
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==13.2.1; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_12-cuda13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_12-cuda13_2-build
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 1507-1527 / 第 1507-1527 行

````yaml
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cuda13_2
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_12-cuda13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1528-1549 / 第 1528-1549 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_12-cuda13_2-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cuda13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1551-1571 / 第 1551-1571 行

````yaml
  manywheel-py3_12-rocm7_1-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.12"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      timeout-minutes: 420
      build_name: manywheel-py3_12-rocm7_1
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1573-1595 / 第 1573-1595 行

````yaml
  manywheel-py3_12-rocm7_1-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_12-rocm7_1-build
      - get-label-type
    runs-on: linux.rocm.gpu.gfx942.1
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.12"
    permissions:
      id-token: write
      contents: read
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1596-1615 / 第 1596-1615 行

````yaml
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
      - name: Setup ROCm
        uses: ./.github/actions/setup-rocm
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_12-rocm7_1
          path: "${{ runner.temp }}/artifacts/"
      - name: ROCm set GPU_FLAG
        run: |
          echo "GPU_FLAG=--device=/dev/mem --device=/dev/kfd --device=/dev/dri --group-add video --group-add daemon" >> "${GITHUB_ENV}"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`, `./.github/actions/setup-rocm`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1616-1637 / 第 1616-1637 行

````yaml
      - name: configure aws credentials
        id: aws_creds
        if: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') }}
        uses: aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a # v4
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
          aws-region: us-east-1
          role-duration-seconds: 18000
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: rocm7.1
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
````

- EN: This section reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1638-1659 / 第 1638-1659 行

````yaml
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown ROCm
        uses: ./.github/actions/teardown-rocm
  manywheel-py3_12-rocm7_1-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_12-rocm7_1-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-rocm7_1
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/teardown-rocm`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1660-1674 / 第 1660-1674 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_12-rocm7_2-build:
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

### Lines 1675-1695 / 第 1675-1695 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.12"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      timeout-minutes: 420
      build_name: manywheel-py3_12-rocm7_2
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_12-rocm7_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_12-rocm7_2-build
      - get-label-type
    runs-on: linux.rocm.gpu.gfx942.1
    timeout-minutes: 240
````

- EN: This section describes repository automation behavior for `.github/workflows/generated-linux-binary-manywheel-nightly.yml`.
- CN: 该部分描述 `.github/workflows/generated-linux-binary-manywheel-nightly.yml` 的仓库自动化行为。

### Lines 1696-1717 / 第 1696-1717 行

````yaml
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.12"
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1718-1739 / 第 1718-1739 行

````yaml
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Setup ROCm
        uses: ./.github/actions/setup-rocm
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_12-rocm7_2
          path: "${{ runner.temp }}/artifacts/"
      - name: ROCm set GPU_FLAG
        run: |
          echo "GPU_FLAG=--device=/dev/mem --device=/dev/kfd --device=/dev/dri --group-add video --group-add daemon" >> "${GITHUB_ENV}"
      - name: configure aws credentials
        id: aws_creds
        if: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') }}
        uses: aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a # v4
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
          aws-region: us-east-1
          role-duration-seconds: 18000
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/setup-rocm`, `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1740-1759 / 第 1740-1759 行

````yaml
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: rocm7.2
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown ROCm
        uses: ./.github/actions/teardown-rocm
  manywheel-py3_12-rocm7_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/test-pytorch-binary`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1760-1781 / 第 1760-1781 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_12-rocm7_2-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-rocm7_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1783-1805 / 第 1783-1805 行

````yaml
  manywheel-py3_12-xpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.12"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_12-xpu
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: intel-cmplr-lib-rt==2025.3.2 | intel-cmplr-lib-ur==2025.3.2 | intel-cmplr-lic-rt==2025.3.2 | intel-sycl-rt==2025.3.2 | oneccl-devel==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | oneccl==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | impi-rt==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | onemkl-license==2025.3.1 | onemkl-sycl-blas==2025.3.1 | onemkl-sycl-dft==2025.3.1 | onemkl-sycl-lapack==2025.3.1 | onemkl-sycl-rng==2025.3.1 | onemkl-sycl-sparse==2025.3.1 | dpcpp-cpp-rt==2025.3.2 | intel-opencl-rt==2025.3.2 | mkl==2025.3.1 | intel-openmp==2025.3.2 | tbb==2022.3.1 | tcmlib==1.4.1 | umf==1.0.3 | intel-pti==0.16.0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_12-xpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1806-1827 / 第 1806-1827 行

````yaml
    needs:
      - manywheel-py3_12-xpu-build
      - get-label-type
    runs-on: linux.idc.xpu
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.12"
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
````

- EN: This section reuses actions and step building blocks such as `actions/checkout@v4`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1828-1847 / 第 1828-1847 行

````yaml
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Setup XPU
        uses: pytorch/pytorch/.github/actions/setup-xpu@main
      - name: Login to ECR
        uses: pytorch/pytorch/.github/actions/ecr-login@main
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_12-xpu
          path: "${{ runner.temp }}/artifacts/"
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/setup-xpu@main`, `pytorch/pytorch/.github/actions/ecr-login@main`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1848-1868 / 第 1848-1868 行

````yaml
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: xpu
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown XPU
        uses: ./.github/actions/teardown-xpu
  manywheel-py3_12-xpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_12-xpu-test
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/test-pytorch-binary`, `./.github/actions/teardown-xpu`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1869-1890 / 第 1869-1890 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-xpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_13-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1891-1912 / 第 1891-1912 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.13"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_13-cpu
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13-cpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13-cpu-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1913-1934 / 第 1913-1934 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cpu
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.4xlarge
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13-cpu-test
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1935-1956 / 第 1935-1956 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_13-cuda12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1957-1978 / 第 1957-1978 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.13"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_13-cuda12_6
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==12.6.3; platform_system == 'Linux' | cuda-bindings>=12.9.4,<13; platform_system == 'Linux' | nvidia-cudnn-cu12==9.10.2.21; platform_system == 'Linux' | nvidia-cusparselt-cu12==0.7.1; platform_system == 'Linux' | nvidia-nccl-cu12==2.29.3; platform_system == 'Linux' | nvidia-nvshmem-cu12==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13-cuda12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13-cuda12_6-build
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 1979-1999 / 第 1979-1999 行

````yaml
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cuda12_6
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13-cuda12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2000-2021 / 第 2000-2021 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13-cuda12_6-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cuda12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2023-2043 / 第 2023-2043 行

````yaml
  manywheel-py3_13-cuda13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.13"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_13-cuda13_0
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2045-2065 / 第 2045-2065 行

````yaml
  manywheel-py3_13-cuda13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13-cuda13_0-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cuda13_0
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2066-2085 / 第 2066-2085 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13-cuda13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13-cuda13_0-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cuda13_0
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2086-2100 / 第 2086-2100 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_13-cuda13_2-build:
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

### Lines 2101-2123 / 第 2101-2123 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.13"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_13-cuda13_2
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==13.2.1; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13-cuda13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13-cuda13_2-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2124-2146 / 第 2124-2146 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cuda13_2
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13-cuda13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13-cuda13_2-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2147-2169 / 第 2147-2169 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cuda13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_13-rocm7_1-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2170-2191 / 第 2170-2191 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.13"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      timeout-minutes: 420
      build_name: manywheel-py3_13-rocm7_1
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13-rocm7_1-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13-rocm7_1-build
      - get-label-type
    runs-on: linux.rocm.gpu.gfx942.1
    timeout-minutes: 240
````

- EN: This section describes repository automation behavior for `.github/workflows/generated-linux-binary-manywheel-nightly.yml`.
- CN: 该部分描述 `.github/workflows/generated-linux-binary-manywheel-nightly.yml` 的仓库自动化行为。

### Lines 2192-2213 / 第 2192-2213 行

````yaml
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.13"
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2214-2235 / 第 2214-2235 行

````yaml
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Setup ROCm
        uses: ./.github/actions/setup-rocm
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_13-rocm7_1
          path: "${{ runner.temp }}/artifacts/"
      - name: ROCm set GPU_FLAG
        run: |
          echo "GPU_FLAG=--device=/dev/mem --device=/dev/kfd --device=/dev/dri --group-add video --group-add daemon" >> "${GITHUB_ENV}"
      - name: configure aws credentials
        id: aws_creds
        if: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') }}
        uses: aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a # v4
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
          aws-region: us-east-1
          role-duration-seconds: 18000
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/setup-rocm`, `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2236-2255 / 第 2236-2255 行

````yaml
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: rocm7.1
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown ROCm
        uses: ./.github/actions/teardown-rocm
  manywheel-py3_13-rocm7_1-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/test-pytorch-binary`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2256-2277 / 第 2256-2277 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13-rocm7_1-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-rocm7_1
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2279-2299 / 第 2279-2299 行

````yaml
  manywheel-py3_13-rocm7_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.13"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      timeout-minutes: 420
      build_name: manywheel-py3_13-rocm7_2
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2301-2323 / 第 2301-2323 行

````yaml
  manywheel-py3_13-rocm7_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13-rocm7_2-build
      - get-label-type
    runs-on: linux.rocm.gpu.gfx942.1
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.13"
    permissions:
      id-token: write
      contents: read
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2324-2343 / 第 2324-2343 行

````yaml
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
      - name: Setup ROCm
        uses: ./.github/actions/setup-rocm
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_13-rocm7_2
          path: "${{ runner.temp }}/artifacts/"
      - name: ROCm set GPU_FLAG
        run: |
          echo "GPU_FLAG=--device=/dev/mem --device=/dev/kfd --device=/dev/dri --group-add video --group-add daemon" >> "${GITHUB_ENV}"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`, `./.github/actions/setup-rocm`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2344-2365 / 第 2344-2365 行

````yaml
      - name: configure aws credentials
        id: aws_creds
        if: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') }}
        uses: aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a # v4
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
          aws-region: us-east-1
          role-duration-seconds: 18000
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: rocm7.2
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
````

- EN: This section reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2366-2387 / 第 2366-2387 行

````yaml
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown ROCm
        uses: ./.github/actions/teardown-rocm
  manywheel-py3_13-rocm7_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13-rocm7_2-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-rocm7_2
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/teardown-rocm`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2388-2402 / 第 2388-2402 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_13-xpu-build:
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

### Lines 2403-2425 / 第 2403-2425 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.13"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_13-xpu
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: intel-cmplr-lib-rt==2025.3.2 | intel-cmplr-lib-ur==2025.3.2 | intel-cmplr-lic-rt==2025.3.2 | intel-sycl-rt==2025.3.2 | oneccl-devel==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | oneccl==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | impi-rt==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | onemkl-license==2025.3.1 | onemkl-sycl-blas==2025.3.1 | onemkl-sycl-dft==2025.3.1 | onemkl-sycl-lapack==2025.3.1 | onemkl-sycl-rng==2025.3.1 | onemkl-sycl-sparse==2025.3.1 | dpcpp-cpp-rt==2025.3.2 | intel-opencl-rt==2025.3.2 | mkl==2025.3.1 | intel-openmp==2025.3.2 | tbb==2022.3.1 | tcmlib==1.4.1 | umf==1.0.3 | intel-pti==0.16.0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13-xpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13-xpu-build
      - get-label-type
    runs-on: linux.idc.xpu
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section controls token scope or environment variables for safe execution.
- CN: 该部分控制令牌权限范围或环境变量以实现安全执行。

### Lines 2426-2447 / 第 2426-2447 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.13"
    permissions:
      id-token: write
      contents: read
    steps:
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
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2448-2470 / 第 2448-2470 行

````yaml
      - name: Setup XPU
        uses: pytorch/pytorch/.github/actions/setup-xpu@main
      - name: Login to ECR
        uses: pytorch/pytorch/.github/actions/ecr-login@main
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_13-xpu
          path: "${{ runner.temp }}/artifacts/"
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: xpu
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/setup-xpu@main`, `pytorch/pytorch/.github/actions/ecr-login@main`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2471-2491 / 第 2471-2491 行

````yaml
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown XPU
        uses: ./.github/actions/teardown-xpu
  manywheel-py3_13-xpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13-xpu-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-xpu
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/teardown-xpu`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2492-2506 / 第 2492-2506 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_13t-cpu-build:
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

### Lines 2507-2528 / 第 2507-2528 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.13t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_13t-cpu
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13t-cpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13t-cpu-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2529-2550 / 第 2529-2550 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cpu
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.4xlarge
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13t-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13t-cpu-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2551-2572 / 第 2551-2572 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_13t-cuda12_6-build:
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

### Lines 2573-2595 / 第 2573-2595 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.13t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_13t-cuda12_6
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==12.6.3; platform_system == 'Linux' | cuda-bindings>=12.9.4,<13; platform_system == 'Linux' | nvidia-cudnn-cu12==9.10.2.21; platform_system == 'Linux' | nvidia-cusparselt-cu12==0.7.1; platform_system == 'Linux' | nvidia-nccl-cu12==2.29.3; platform_system == 'Linux' | nvidia-nvshmem-cu12==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13t-cuda12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13t-cuda12_6-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2596-2618 / 第 2596-2618 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cuda12_6
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13t-cuda12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13t-cuda12_6-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2619-2641 / 第 2619-2641 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cuda12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_13t-cuda13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2642-2662 / 第 2642-2662 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.13t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_13t-cuda13_0
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13t-cuda13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13t-cuda13_0-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 2663-2685 / 第 2663-2685 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cuda13_0
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13t-cuda13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13t-cuda13_0-test
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2686-2708 / 第 2686-2708 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cuda13_0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_13t-cuda13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2709-2730 / 第 2709-2730 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.13t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_13t-cuda13_2
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==13.2.1; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13t-cuda13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13t-cuda13_2-build
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 2731-2751 / 第 2731-2751 行

````yaml
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cuda13_2
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13t-cuda13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2752-2773 / 第 2752-2773 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13t-cuda13_2-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cuda13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2775-2795 / 第 2775-2795 行

````yaml
  manywheel-py3_13t-rocm7_1-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.13t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      timeout-minutes: 420
      build_name: manywheel-py3_13t-rocm7_1
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2797-2819 / 第 2797-2819 行

````yaml
  manywheel-py3_13t-rocm7_1-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13t-rocm7_1-build
      - get-label-type
    runs-on: linux.rocm.gpu.gfx942.1
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.13t"
    permissions:
      id-token: write
      contents: read
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2820-2839 / 第 2820-2839 行

````yaml
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
      - name: Setup ROCm
        uses: ./.github/actions/setup-rocm
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_13t-rocm7_1
          path: "${{ runner.temp }}/artifacts/"
      - name: ROCm set GPU_FLAG
        run: |
          echo "GPU_FLAG=--device=/dev/mem --device=/dev/kfd --device=/dev/dri --group-add video --group-add daemon" >> "${GITHUB_ENV}"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`, `./.github/actions/setup-rocm`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2840-2861 / 第 2840-2861 行

````yaml
      - name: configure aws credentials
        id: aws_creds
        if: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') }}
        uses: aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a # v4
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
          aws-region: us-east-1
          role-duration-seconds: 18000
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: rocm7.1
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
````

- EN: This section reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2862-2883 / 第 2862-2883 行

````yaml
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown ROCm
        uses: ./.github/actions/teardown-rocm
  manywheel-py3_13t-rocm7_1-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13t-rocm7_1-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-rocm7_1
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/teardown-rocm`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2884-2898 / 第 2884-2898 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_13t-rocm7_2-build:
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

### Lines 2899-2919 / 第 2899-2919 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.13t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      timeout-minutes: 420
      build_name: manywheel-py3_13t-rocm7_2
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13t-rocm7_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13t-rocm7_2-build
      - get-label-type
    runs-on: linux.rocm.gpu.gfx942.1
    timeout-minutes: 240
````

- EN: This section describes repository automation behavior for `.github/workflows/generated-linux-binary-manywheel-nightly.yml`.
- CN: 该部分描述 `.github/workflows/generated-linux-binary-manywheel-nightly.yml` 的仓库自动化行为。

### Lines 2920-2941 / 第 2920-2941 行

````yaml
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.13t"
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2942-2963 / 第 2942-2963 行

````yaml
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Setup ROCm
        uses: ./.github/actions/setup-rocm
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_13t-rocm7_2
          path: "${{ runner.temp }}/artifacts/"
      - name: ROCm set GPU_FLAG
        run: |
          echo "GPU_FLAG=--device=/dev/mem --device=/dev/kfd --device=/dev/dri --group-add video --group-add daemon" >> "${GITHUB_ENV}"
      - name: configure aws credentials
        id: aws_creds
        if: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') }}
        uses: aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a # v4
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
          aws-region: us-east-1
          role-duration-seconds: 18000
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/setup-rocm`, `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2964-2983 / 第 2964-2983 行

````yaml
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: rocm7.2
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown ROCm
        uses: ./.github/actions/teardown-rocm
  manywheel-py3_13t-rocm7_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/test-pytorch-binary`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2984-3005 / 第 2984-3005 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13t-rocm7_2-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-rocm7_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3007-3029 / 第 3007-3029 行

````yaml
  manywheel-py3_13t-xpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.13t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_13t-xpu
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: intel-cmplr-lib-rt==2025.3.2 | intel-cmplr-lib-ur==2025.3.2 | intel-cmplr-lic-rt==2025.3.2 | intel-sycl-rt==2025.3.2 | oneccl-devel==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | oneccl==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | impi-rt==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | onemkl-license==2025.3.1 | onemkl-sycl-blas==2025.3.1 | onemkl-sycl-dft==2025.3.1 | onemkl-sycl-lapack==2025.3.1 | onemkl-sycl-rng==2025.3.1 | onemkl-sycl-sparse==2025.3.1 | dpcpp-cpp-rt==2025.3.2 | intel-opencl-rt==2025.3.2 | mkl==2025.3.1 | intel-openmp==2025.3.2 | tbb==2022.3.1 | tcmlib==1.4.1 | umf==1.0.3 | intel-pti==0.16.0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13t-xpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3030-3051 / 第 3030-3051 行

````yaml
    needs:
      - manywheel-py3_13t-xpu-build
      - get-label-type
    runs-on: linux.idc.xpu
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.13t"
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
````

- EN: This section reuses actions and step building blocks such as `actions/checkout@v4`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3052-3071 / 第 3052-3071 行

````yaml
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Setup XPU
        uses: pytorch/pytorch/.github/actions/setup-xpu@main
      - name: Login to ECR
        uses: pytorch/pytorch/.github/actions/ecr-login@main
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_13t-xpu
          path: "${{ runner.temp }}/artifacts/"
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/setup-xpu@main`, `pytorch/pytorch/.github/actions/ecr-login@main`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3072-3092 / 第 3072-3092 行

````yaml
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: xpu
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown XPU
        uses: ./.github/actions/teardown-xpu
  manywheel-py3_13t-xpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13t-xpu-test
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/test-pytorch-binary`, `./.github/actions/teardown-xpu`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3093-3114 / 第 3093-3114 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-xpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_14-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3115-3136 / 第 3115-3136 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.14"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_14-cpu
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14-cpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14-cpu-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3137-3158 / 第 3137-3158 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cpu
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.4xlarge
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14-cpu-test
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3159-3180 / 第 3159-3180 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_14-cuda12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3181-3202 / 第 3181-3202 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.14"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_14-cuda12_6
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==12.6.3; platform_system == 'Linux' | cuda-bindings>=12.9.4,<13; platform_system == 'Linux' | nvidia-cudnn-cu12==9.10.2.21; platform_system == 'Linux' | nvidia-cusparselt-cu12==0.7.1; platform_system == 'Linux' | nvidia-nccl-cu12==2.29.3; platform_system == 'Linux' | nvidia-nvshmem-cu12==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14-cuda12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14-cuda12_6-build
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 3203-3223 / 第 3203-3223 行

````yaml
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cuda12_6
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14-cuda12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3224-3245 / 第 3224-3245 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14-cuda12_6-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cuda12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3247-3267 / 第 3247-3267 行

````yaml
  manywheel-py3_14-cuda13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.14"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_14-cuda13_0
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3269-3289 / 第 3269-3289 行

````yaml
  manywheel-py3_14-cuda13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14-cuda13_0-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cuda13_0
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3290-3309 / 第 3290-3309 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14-cuda13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14-cuda13_0-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cuda13_0
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3310-3324 / 第 3310-3324 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_14-cuda13_2-build:
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

### Lines 3325-3347 / 第 3325-3347 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.14"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_14-cuda13_2
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==13.2.1; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14-cuda13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14-cuda13_2-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3348-3370 / 第 3348-3370 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cuda13_2
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14-cuda13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14-cuda13_2-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3371-3393 / 第 3371-3393 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cuda13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_14-rocm7_1-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3394-3415 / 第 3394-3415 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.14"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      timeout-minutes: 420
      build_name: manywheel-py3_14-rocm7_1
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14-rocm7_1-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14-rocm7_1-build
      - get-label-type
    runs-on: linux.rocm.gpu.gfx942.1
    timeout-minutes: 240
````

- EN: This section describes repository automation behavior for `.github/workflows/generated-linux-binary-manywheel-nightly.yml`.
- CN: 该部分描述 `.github/workflows/generated-linux-binary-manywheel-nightly.yml` 的仓库自动化行为。

### Lines 3416-3437 / 第 3416-3437 行

````yaml
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.14"
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3438-3459 / 第 3438-3459 行

````yaml
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Setup ROCm
        uses: ./.github/actions/setup-rocm
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_14-rocm7_1
          path: "${{ runner.temp }}/artifacts/"
      - name: ROCm set GPU_FLAG
        run: |
          echo "GPU_FLAG=--device=/dev/mem --device=/dev/kfd --device=/dev/dri --group-add video --group-add daemon" >> "${GITHUB_ENV}"
      - name: configure aws credentials
        id: aws_creds
        if: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') }}
        uses: aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a # v4
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
          aws-region: us-east-1
          role-duration-seconds: 18000
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/setup-rocm`, `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3460-3479 / 第 3460-3479 行

````yaml
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: rocm7.1
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown ROCm
        uses: ./.github/actions/teardown-rocm
  manywheel-py3_14-rocm7_1-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/test-pytorch-binary`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3480-3501 / 第 3480-3501 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14-rocm7_1-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-rocm7_1
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3503-3523 / 第 3503-3523 行

````yaml
  manywheel-py3_14-rocm7_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.14"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      timeout-minutes: 420
      build_name: manywheel-py3_14-rocm7_2
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3525-3547 / 第 3525-3547 行

````yaml
  manywheel-py3_14-rocm7_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14-rocm7_2-build
      - get-label-type
    runs-on: linux.rocm.gpu.gfx942.1
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.14"
    permissions:
      id-token: write
      contents: read
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3548-3567 / 第 3548-3567 行

````yaml
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
      - name: Setup ROCm
        uses: ./.github/actions/setup-rocm
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_14-rocm7_2
          path: "${{ runner.temp }}/artifacts/"
      - name: ROCm set GPU_FLAG
        run: |
          echo "GPU_FLAG=--device=/dev/mem --device=/dev/kfd --device=/dev/dri --group-add video --group-add daemon" >> "${GITHUB_ENV}"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`, `./.github/actions/setup-rocm`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3568-3589 / 第 3568-3589 行

````yaml
      - name: configure aws credentials
        id: aws_creds
        if: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') }}
        uses: aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a # v4
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
          aws-region: us-east-1
          role-duration-seconds: 18000
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: rocm7.2
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
````

- EN: This section reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3590-3611 / 第 3590-3611 行

````yaml
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown ROCm
        uses: ./.github/actions/teardown-rocm
  manywheel-py3_14-rocm7_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14-rocm7_2-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-rocm7_2
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/teardown-rocm`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3612-3626 / 第 3612-3626 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_14-xpu-build:
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

### Lines 3627-3649 / 第 3627-3649 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.14"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_14-xpu
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: intel-cmplr-lib-rt==2025.3.2 | intel-cmplr-lib-ur==2025.3.2 | intel-cmplr-lic-rt==2025.3.2 | intel-sycl-rt==2025.3.2 | oneccl-devel==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | oneccl==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | impi-rt==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | onemkl-license==2025.3.1 | onemkl-sycl-blas==2025.3.1 | onemkl-sycl-dft==2025.3.1 | onemkl-sycl-lapack==2025.3.1 | onemkl-sycl-rng==2025.3.1 | onemkl-sycl-sparse==2025.3.1 | dpcpp-cpp-rt==2025.3.2 | intel-opencl-rt==2025.3.2 | mkl==2025.3.1 | intel-openmp==2025.3.2 | tbb==2022.3.1 | tcmlib==1.4.1 | umf==1.0.3 | intel-pti==0.16.0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14-xpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14-xpu-build
      - get-label-type
    runs-on: linux.idc.xpu
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section controls token scope or environment variables for safe execution.
- CN: 该部分控制令牌权限范围或环境变量以实现安全执行。

### Lines 3650-3671 / 第 3650-3671 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.14"
    permissions:
      id-token: write
      contents: read
    steps:
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
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3672-3694 / 第 3672-3694 行

````yaml
      - name: Setup XPU
        uses: pytorch/pytorch/.github/actions/setup-xpu@main
      - name: Login to ECR
        uses: pytorch/pytorch/.github/actions/ecr-login@main
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_14-xpu
          path: "${{ runner.temp }}/artifacts/"
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: xpu
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/setup-xpu@main`, `pytorch/pytorch/.github/actions/ecr-login@main`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3695-3715 / 第 3695-3715 行

````yaml
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown XPU
        uses: ./.github/actions/teardown-xpu
  manywheel-py3_14-xpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14-xpu-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-xpu
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/teardown-xpu`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3716-3730 / 第 3716-3730 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_14t-cpu-build:
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

### Lines 3731-3752 / 第 3731-3752 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.14t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_14t-cpu
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14t-cpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14t-cpu-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3753-3774 / 第 3753-3774 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cpu
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.4xlarge
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14t-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14t-cpu-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3775-3796 / 第 3775-3796 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_14t-cuda12_6-build:
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

### Lines 3797-3819 / 第 3797-3819 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.14t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_14t-cuda12_6
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==12.6.3; platform_system == 'Linux' | cuda-bindings>=12.9.4,<13; platform_system == 'Linux' | nvidia-cudnn-cu12==9.10.2.21; platform_system == 'Linux' | nvidia-cusparselt-cu12==0.7.1; platform_system == 'Linux' | nvidia-nccl-cu12==2.29.3; platform_system == 'Linux' | nvidia-nvshmem-cu12==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14t-cuda12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14t-cuda12_6-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3820-3842 / 第 3820-3842 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cuda12_6
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14t-cuda12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14t-cuda12_6-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3843-3865 / 第 3843-3865 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda12.6
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cuda12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_14t-cuda13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3866-3886 / 第 3866-3886 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.14t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_14t-cuda13_0
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14t-cuda13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14t-cuda13_0-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 3887-3909 / 第 3887-3909 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cuda13_0
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14t-cuda13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14t-cuda13_0-test
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3910-3932 / 第 3910-3932 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.0
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cuda13_0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_14t-cuda13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`, `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3933-3954 / 第 3933-3954 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.14t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_14t-cuda13_2
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cublas,cufile,nvjitlink,nvtx]==13.2.1; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14t-cuda13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14t-cuda13_2-build
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 3955-3975 / 第 3955-3975 行

````yaml
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cuda13_2
      build_environment: linux-binary-manywheel
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      runs_on: linux.g4dn.4xlarge.nvidia.gpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14t-cuda13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3976-3997 / 第 3976-3997 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14t-cuda13_2-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: cuda13.2
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cuda13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3999-4019 / 第 3999-4019 行

````yaml
  manywheel-py3_14t-rocm7_1-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.14t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      timeout-minutes: 420
      build_name: manywheel-py3_14t-rocm7_1
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 4021-4043 / 第 4021-4043 行

````yaml
  manywheel-py3_14t-rocm7_1-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14t-rocm7_1-build
      - get-label-type
    runs-on: linux.rocm.gpu.gfx942.1
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.14t"
    permissions:
      id-token: write
      contents: read
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4044-4063 / 第 4044-4063 行

````yaml
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
      - name: Setup ROCm
        uses: ./.github/actions/setup-rocm
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_14t-rocm7_1
          path: "${{ runner.temp }}/artifacts/"
      - name: ROCm set GPU_FLAG
        run: |
          echo "GPU_FLAG=--device=/dev/mem --device=/dev/kfd --device=/dev/dri --group-add video --group-add daemon" >> "${GITHUB_ENV}"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`, `./.github/actions/setup-rocm`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4064-4085 / 第 4064-4085 行

````yaml
      - name: configure aws credentials
        id: aws_creds
        if: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') }}
        uses: aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a # v4
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
          aws-region: us-east-1
          role-duration-seconds: 18000
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: rocm7.1
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
````

- EN: This section reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 4086-4107 / 第 4086-4107 行

````yaml
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown ROCm
        uses: ./.github/actions/teardown-rocm
  manywheel-py3_14t-rocm7_1-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14t-rocm7_1-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.1
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-rocm7_1
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/teardown-rocm`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4108-4122 / 第 4108-4122 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_14t-rocm7_2-build:
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

### Lines 4123-4143 / 第 4123-4143 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.14t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      timeout-minutes: 420
      build_name: manywheel-py3_14t-rocm7_2
      build_environment: linux-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14t-rocm7_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14t-rocm7_2-build
      - get-label-type
    runs-on: linux.rocm.gpu.gfx942.1
    timeout-minutes: 240
````

- EN: This section describes repository automation behavior for `.github/workflows/generated-linux-binary-manywheel-nightly.yml`.
- CN: 该部分描述 `.github/workflows/generated-linux-binary-manywheel-nightly.yml` 的仓库自动化行为。

### Lines 4144-4165 / 第 4144-4165 行

````yaml
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.14t"
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4166-4187 / 第 4166-4187 行

````yaml
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Setup ROCm
        uses: ./.github/actions/setup-rocm
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_14t-rocm7_2
          path: "${{ runner.temp }}/artifacts/"
      - name: ROCm set GPU_FLAG
        run: |
          echo "GPU_FLAG=--device=/dev/mem --device=/dev/kfd --device=/dev/dri --group-add video --group-add daemon" >> "${GITHUB_ENV}"
      - name: configure aws credentials
        id: aws_creds
        if: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') }}
        uses: aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a # v4
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
          aws-region: us-east-1
          role-duration-seconds: 18000
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/setup-rocm`, `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4188-4207 / 第 4188-4207 行

````yaml
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: rocm7.2
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown ROCm
        uses: ./.github/actions/teardown-rocm
  manywheel-py3_14t-rocm7_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/test-pytorch-binary`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4208-4229 / 第 4208-4229 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14t-rocm7_2-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: rocm7.2
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-rocm7_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4231-4253 / 第 4231-4253 行

````yaml
  manywheel-py3_14t-xpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.14t"
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build_name: manywheel-py3_14t-xpu
      build_environment: linux-binary-manywheel
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: intel-cmplr-lib-rt==2025.3.2 | intel-cmplr-lib-ur==2025.3.2 | intel-cmplr-lic-rt==2025.3.2 | intel-sycl-rt==2025.3.2 | oneccl-devel==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | oneccl==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | impi-rt==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | onemkl-license==2025.3.1 | onemkl-sycl-blas==2025.3.1 | onemkl-sycl-dft==2025.3.1 | onemkl-sycl-lapack==2025.3.1 | onemkl-sycl-rng==2025.3.1 | onemkl-sycl-sparse==2025.3.1 | dpcpp-cpp-rt==2025.3.2 | intel-opencl-rt==2025.3.2 | mkl==2025.3.1 | intel-openmp==2025.3.2 | tbb==2022.3.1 | tcmlib==1.4.1 | umf==1.0.3 | intel-pti==0.16.0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14t-xpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 4254-4275 / 第 4254-4275 行

````yaml
    needs:
      - manywheel-py3_14t-xpu-build
      - get-label-type
    runs-on: linux.idc.xpu
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.14t"
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
````

- EN: This section reuses actions and step building blocks such as `actions/checkout@v4`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4276-4295 / 第 4276-4295 行

````yaml
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Setup XPU
        uses: pytorch/pytorch/.github/actions/setup-xpu@main
      - name: Login to ECR
        uses: pytorch/pytorch/.github/actions/ecr-login@main
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: manywheel-py3_14t-xpu
          path: "${{ runner.temp }}/artifacts/"
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/setup-xpu@main`, `pytorch/pytorch/.github/actions/ecr-login@main`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4296-4316 / 第 4296-4316 行

````yaml
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: manylinux2_28-builder
          custom-tag-prefix: xpu
          docker-build-dir: .ci/docker
      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Test Pytorch binary
        uses: ./.github/actions/test-pytorch-binary
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
      - name: Teardown XPU
        uses: ./.github/actions/teardown-xpu
  manywheel-py3_14t-xpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14t-xpu-test
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/test-pytorch-binary`, `./.github/actions/teardown-xpu`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4317-4338 / 第 4317-4338 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DOCKER_IMAGE: manylinux2_28-builder
      DOCKER_IMAGE_TAG_PREFIX: xpu
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-xpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  libtorch-cpu-shared-with-deps-release-extract:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-cpu-build
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 4339-4356 / 第 4339-4356 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge"
    timeout-minutes: 60
    env:
      DESIRED_CUDA: cpu
      LIBTORCH_VARIANT: shared-with-deps
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          sparse-checkout: .ci/libtorch/
          show-progress: false
      - uses: actions/download-artifact@v4.1.7
        name: Download Wheel Artifact
        with:
          name: manywheel-py3_10-cpu
          path: "${{ runner.temp }}/wheel_artifact/"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4357-4379 / 第 4357-4379 行

````yaml
      - name: Extract libtorch from wheel
        run: |
          set -eux
          mkdir -p "${{ runner.temp }}/libtorch_output"
          python3 .ci/libtorch/extract_libtorch_from_wheel.py \
            --wheel-dir "${{ runner.temp }}/wheel_artifact" \
            --output-dir "${{ runner.temp }}/libtorch_output" \
            --platform linux \
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

### Lines 4380-4401 / 第 4380-4401 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: libtorch
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      LIBTORCH_CONFIG: release
      LIBTORCH_VARIANT: shared-with-deps
      build_name: libtorch-cpu-shared-with-deps-release
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  libtorch-cuda12_6-shared-with-deps-release-extract:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-cuda12_6-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge"
    timeout-minutes: 60
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 4402-4416 / 第 4402-4416 行

````yaml
    env:
      DESIRED_CUDA: cu126
      LIBTORCH_VARIANT: shared-with-deps
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          sparse-checkout: .ci/libtorch/
          show-progress: false
      - uses: actions/download-artifact@v4.1.7
        name: Download Wheel Artifact
        with:
          name: manywheel-py3_10-cuda12_6
          path: "${{ runner.temp }}/wheel_artifact/"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4417-4439 / 第 4417-4439 行

````yaml
      - name: Extract libtorch from wheel
        run: |
          set -eux
          mkdir -p "${{ runner.temp }}/libtorch_output"
          python3 .ci/libtorch/extract_libtorch_from_wheel.py \
            --wheel-dir "${{ runner.temp }}/wheel_artifact" \
            --output-dir "${{ runner.temp }}/libtorch_output" \
            --platform linux \
            --desired-cuda "$DESIRED_CUDA" \
            --libtorch-variant "$LIBTORCH_VARIANT"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: libtorch-cuda12_6-shared-with-deps-release
          retention-days: 14
          if-no-files-found: error
          path: "${{ runner.temp }}/libtorch_output/"
  libtorch-cuda12_6-shared-with-deps-release-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: libtorch-cuda12_6-shared-with-deps-release-extract
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4440-4462 / 第 4440-4462 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: libtorch
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      LIBTORCH_CONFIG: release
      LIBTORCH_VARIANT: shared-with-deps
      build_name: libtorch-cuda12_6-shared-with-deps-release
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  libtorch-cuda13_0-shared-with-deps-release-extract:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-cuda13_0-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge"
    timeout-minutes: 60
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 4463-4477 / 第 4463-4477 行

````yaml
    env:
      DESIRED_CUDA: cu130
      LIBTORCH_VARIANT: shared-with-deps
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          sparse-checkout: .ci/libtorch/
          show-progress: false
      - uses: actions/download-artifact@v4.1.7
        name: Download Wheel Artifact
        with:
          name: manywheel-py3_10-cuda13_0
          path: "${{ runner.temp }}/wheel_artifact/"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4478-4500 / 第 4478-4500 行

````yaml
      - name: Extract libtorch from wheel
        run: |
          set -eux
          mkdir -p "${{ runner.temp }}/libtorch_output"
          python3 .ci/libtorch/extract_libtorch_from_wheel.py \
            --wheel-dir "${{ runner.temp }}/wheel_artifact" \
            --output-dir "${{ runner.temp }}/libtorch_output" \
            --platform linux \
            --desired-cuda "$DESIRED_CUDA" \
            --libtorch-variant "$LIBTORCH_VARIANT"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: libtorch-cuda13_0-shared-with-deps-release
          retention-days: 14
          if-no-files-found: error
          path: "${{ runner.temp }}/libtorch_output/"
  libtorch-cuda13_0-shared-with-deps-release-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: libtorch-cuda13_0-shared-with-deps-release-extract
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4501-4523 / 第 4501-4523 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: libtorch
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      LIBTORCH_CONFIG: release
      LIBTORCH_VARIANT: shared-with-deps
      build_name: libtorch-cuda13_0-shared-with-deps-release
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  libtorch-cuda13_2-shared-with-deps-release-extract:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-cuda13_2-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge"
    timeout-minutes: 60
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 4524-4538 / 第 4524-4538 行

````yaml
    env:
      DESIRED_CUDA: cu132
      LIBTORCH_VARIANT: shared-with-deps
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          sparse-checkout: .ci/libtorch/
          show-progress: false
      - uses: actions/download-artifact@v4.1.7
        name: Download Wheel Artifact
        with:
          name: manywheel-py3_10-cuda13_2
          path: "${{ runner.temp }}/wheel_artifact/"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4539-4561 / 第 4539-4561 行

````yaml
      - name: Extract libtorch from wheel
        run: |
          set -eux
          mkdir -p "${{ runner.temp }}/libtorch_output"
          python3 .ci/libtorch/extract_libtorch_from_wheel.py \
            --wheel-dir "${{ runner.temp }}/wheel_artifact" \
            --output-dir "${{ runner.temp }}/libtorch_output" \
            --platform linux \
            --desired-cuda "$DESIRED_CUDA" \
            --libtorch-variant "$LIBTORCH_VARIANT"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: libtorch-cuda13_2-shared-with-deps-release
          retention-days: 14
          if-no-files-found: error
          path: "${{ runner.temp }}/libtorch_output/"
  libtorch-cuda13_2-shared-with-deps-release-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: libtorch-cuda13_2-shared-with-deps-release-extract
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4562-4584 / 第 4562-4584 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: libtorch
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      LIBTORCH_CONFIG: release
      LIBTORCH_VARIANT: shared-with-deps
      build_name: libtorch-cuda13_2-shared-with-deps-release
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  libtorch-rocm7_1-shared-with-deps-release-extract:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-rocm7_1-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge"
    timeout-minutes: 60
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 4585-4599 / 第 4585-4599 行

````yaml
    env:
      DESIRED_CUDA: rocm7.1
      LIBTORCH_VARIANT: shared-with-deps
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          sparse-checkout: .ci/libtorch/
          show-progress: false
      - uses: actions/download-artifact@v4.1.7
        name: Download Wheel Artifact
        with:
          name: manywheel-py3_10-rocm7_1
          path: "${{ runner.temp }}/wheel_artifact/"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4600-4622 / 第 4600-4622 行

````yaml
      - name: Extract libtorch from wheel
        run: |
          set -eux
          mkdir -p "${{ runner.temp }}/libtorch_output"
          python3 .ci/libtorch/extract_libtorch_from_wheel.py \
            --wheel-dir "${{ runner.temp }}/wheel_artifact" \
            --output-dir "${{ runner.temp }}/libtorch_output" \
            --platform linux \
            --desired-cuda "$DESIRED_CUDA" \
            --libtorch-variant "$LIBTORCH_VARIANT"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: libtorch-rocm7_1-shared-with-deps-release
          retention-days: 14
          if-no-files-found: error
          path: "${{ runner.temp }}/libtorch_output/"
  libtorch-rocm7_1-shared-with-deps-release-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: libtorch-rocm7_1-shared-with-deps-release-extract
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4623-4645 / 第 4623-4645 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: libtorch
      DESIRED_CUDA: rocm7.1
      GPU_ARCH_VERSION: "7.1"
      GPU_ARCH_TYPE: rocm
      LIBTORCH_CONFIG: release
      LIBTORCH_VARIANT: shared-with-deps
      build_name: libtorch-rocm7_1-shared-with-deps-release
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  libtorch-rocm7_2-shared-with-deps-release-extract:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-rocm7_2-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge"
    timeout-minutes: 60
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 4646-4660 / 第 4646-4660 行

````yaml
    env:
      DESIRED_CUDA: rocm7.2
      LIBTORCH_VARIANT: shared-with-deps
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          sparse-checkout: .ci/libtorch/
          show-progress: false
      - uses: actions/download-artifact@v4.1.7
        name: Download Wheel Artifact
        with:
          name: manywheel-py3_10-rocm7_2
          path: "${{ runner.temp }}/wheel_artifact/"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4661-4683 / 第 4661-4683 行

````yaml
      - name: Extract libtorch from wheel
        run: |
          set -eux
          mkdir -p "${{ runner.temp }}/libtorch_output"
          python3 .ci/libtorch/extract_libtorch_from_wheel.py \
            --wheel-dir "${{ runner.temp }}/wheel_artifact" \
            --output-dir "${{ runner.temp }}/libtorch_output" \
            --platform linux \
            --desired-cuda "$DESIRED_CUDA" \
            --libtorch-variant "$LIBTORCH_VARIANT"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: libtorch-rocm7_2-shared-with-deps-release
          retention-days: 14
          if-no-files-found: error
          path: "${{ runner.temp }}/libtorch_output/"
  libtorch-rocm7_2-shared-with-deps-release-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: libtorch-rocm7_2-shared-with-deps-release-extract
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4684-4692 / 第 4684-4692 行

````yaml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: libtorch
      DESIRED_CUDA: rocm7.2
      GPU_ARCH_VERSION: "7.2"
      GPU_ARCH_TYPE: rocm
      LIBTORCH_CONFIG: release
      LIBTORCH_VARIANT: shared-with-deps
      build_name: libtorch-rocm7_2-shared-with-deps-release
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 4693-4698 / 第 4693-4698 行

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
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `./.github/workflows/_binary-build-linux.yml`, `./.github/workflows/_binary-test-linux.yml`, `./.github/workflows/_binary-upload.yml`, `actions/checkout@v4`, `./.github/actions/setup-rocm`, `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`, ...
- Inline commands / 内联命令: `git`, `echo`, `set`, `mkdir`, `python3`, `wheel-dir`, `output-dir`, `platform`, ...
- Environment variables / 环境变量: `NOT`, `EDIT`, `MANUALLY`, `NOTE`, `ALPINE_IMAGE`, `AWS_DEFAULT_REGION`, `BINARY_ENV_FILE`, `BUILD_ENVIRONMENT`, `GITHUB_TOKEN`, `PR_NUMBER`, ...
- Named jobs or sections / 命名作业或章节: `on`, `push`, `branches`, `tags`, `workflow_dispatch`, `permissions`, `env`, `concurrency`, `jobs`, `get-label-type`, ...
