# generated-linux-s390x-binary-manywheel-nightly.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/generated-linux-s390x-binary-manywheel-nightly.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-23 / 第 1-23 行

````yaml
# @generated DO NOT EDIT MANUALLY

# Template is at:    .github/templates/linux_binary_build_workflow.yml.j2
# Generation script: .github/scripts/generate_ci_workflows.py
name: linux-s390x-binary-manywheel


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
      - 'ciflow/s390/*'
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
  ALPINE_IMAGE: "docker.io/s390x/alpine"
  AWS_DEFAULT_REGION: us-east-1
  BINARY_ENV_FILE: /tmp/env
  BUILD_ENVIRONMENT: linux-s390x-binary-manywheel
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  PR_NUMBER: ${{ github.event.pull_request.number }}
  PYTORCH_FINAL_PACKAGE_DIR: /artifacts
  PYTORCH_ROOT: /pytorch
  SHA1: ${{ github.event.pull_request.head.sha || github.sha }}
  SKIP_ALL_TESTS: 0
concurrency:
  group: linux-s390x-binary-manywheel-${{ github.event.pull_request.number || github.ref_name }}-${{ github.ref_type == 'branch' && github.sha }}-${{ github.event_name == 'workflow_dispatch' }}
  cancel-in-progress: true

jobs:
  get-label-type:
    if: github.repository_owner == 'pytorch'
    name: get-label-type
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 46-69 / 第 46-69 行

````yaml
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}
  manywheel-py3_10-cpu-s390x-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    uses: ./.github/workflows/_binary-build-linux.yml
    needs: get-label-type
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.10"
      runs_on: linux.s390x
      ALPINE_IMAGE: "docker.io/s390x/alpine"
      timeout-minutes: 420
      build_name: manywheel-py3_10-cpu-s390x
      build_environment: linux-s390x-binary-manywheel
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `./.github/workflows/_binary-build-linux.yml`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 70-92 / 第 70-92 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_10-cpu-s390x-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_10-cpu-s390x-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cpu-s390x
      build_environment: linux-s390x-binary-manywheel
      runs_on: linux.s390x
      ALPINE_IMAGE: "docker.io/s390x/alpine"
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 93-111 / 第 93-111 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_10-cpu-s390x-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_10-cpu-s390x-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.10"
      build_name: manywheel-py3_10-cpu-s390x
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

  manywheel-py3_11-cpu-s390x-build:
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
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.11"
      runs_on: linux.s390x
      ALPINE_IMAGE: "docker.io/s390x/alpine"
      timeout-minutes: 420
      build_name: manywheel-py3_11-cpu-s390x
      build_environment: linux-s390x-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_11-cpu-s390x-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_11-cpu-s390x-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 150-172 / 第 150-172 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cpu-s390x
      build_environment: linux-s390x-binary-manywheel
      runs_on: linux.s390x
      ALPINE_IMAGE: "docker.io/s390x/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_11-cpu-s390x-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_11-cpu-s390x-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 173-194 / 第 173-194 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.11"
      build_name: manywheel-py3_11-cpu-s390x
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_12-cpu-s390x-build:
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

### Lines 195-217 / 第 195-217 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.12"
      runs_on: linux.s390x
      ALPINE_IMAGE: "docker.io/s390x/alpine"
      timeout-minutes: 420
      build_name: manywheel-py3_12-cpu-s390x
      build_environment: linux-s390x-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_12-cpu-s390x-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_12-cpu-s390x-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 218-240 / 第 218-240 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cpu-s390x
      build_environment: linux-s390x-binary-manywheel
      runs_on: linux.s390x
      ALPINE_IMAGE: "docker.io/s390x/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_12-cpu-s390x-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_12-cpu-s390x-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 241-262 / 第 241-262 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.12"
      build_name: manywheel-py3_12-cpu-s390x
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_13-cpu-s390x-build:
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

### Lines 263-285 / 第 263-285 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.13"
      runs_on: linux.s390x
      ALPINE_IMAGE: "docker.io/s390x/alpine"
      timeout-minutes: 420
      build_name: manywheel-py3_13-cpu-s390x
      build_environment: linux-s390x-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13-cpu-s390x-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13-cpu-s390x-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 286-308 / 第 286-308 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cpu-s390x
      build_environment: linux-s390x-binary-manywheel
      runs_on: linux.s390x
      ALPINE_IMAGE: "docker.io/s390x/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13-cpu-s390x-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13-cpu-s390x-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 309-330 / 第 309-330 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.13"
      build_name: manywheel-py3_13-cpu-s390x
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_13t-cpu-s390x-build:
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

### Lines 331-353 / 第 331-353 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.13t"
      runs_on: linux.s390x
      ALPINE_IMAGE: "docker.io/s390x/alpine"
      timeout-minutes: 420
      build_name: manywheel-py3_13t-cpu-s390x
      build_environment: linux-s390x-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_13t-cpu-s390x-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_13t-cpu-s390x-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 354-376 / 第 354-376 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cpu-s390x
      build_environment: linux-s390x-binary-manywheel
      runs_on: linux.s390x
      ALPINE_IMAGE: "docker.io/s390x/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_13t-cpu-s390x-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_13t-cpu-s390x-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 377-398 / 第 377-398 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.13t"
      build_name: manywheel-py3_13t-cpu-s390x
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_14-cpu-s390x-build:
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

### Lines 399-421 / 第 399-421 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.14"
      runs_on: linux.s390x
      ALPINE_IMAGE: "docker.io/s390x/alpine"
      timeout-minutes: 420
      build_name: manywheel-py3_14-cpu-s390x
      build_environment: linux-s390x-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14-cpu-s390x-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14-cpu-s390x-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 422-444 / 第 422-444 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cpu-s390x
      build_environment: linux-s390x-binary-manywheel
      runs_on: linux.s390x
      ALPINE_IMAGE: "docker.io/s390x/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14-cpu-s390x-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14-cpu-s390x-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 445-466 / 第 445-466 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.14"
      build_name: manywheel-py3_14-cpu-s390x
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml

  manywheel-py3_14t-cpu-s390x-build:
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

### Lines 467-489 / 第 467-489 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.14t"
      runs_on: linux.s390x
      ALPINE_IMAGE: "docker.io/s390x/alpine"
      timeout-minutes: 420
      build_name: manywheel-py3_14t-cpu-s390x
      build_environment: linux-s390x-binary-manywheel
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}

  manywheel-py3_14t-cpu-s390x-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - manywheel-py3_14t-cpu-s390x-build
      - get-label-type
    uses: ./.github/workflows/_binary-test-linux.yml
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-test-linux.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 490-512 / 第 490-512 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cpu-s390x
      build_environment: linux-s390x-binary-manywheel
      runs_on: linux.s390x
      ALPINE_IMAGE: "docker.io/s390x/alpine"
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
  manywheel-py3_14t-cpu-s390x-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: manywheel-py3_14t-cpu-s390x-test
    with:
      PYTORCH_ROOT: /pytorch
      PACKAGE_TYPE: manywheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 513-519 / 第 513-519 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu-s390x
      DOCKER_IMAGE: pytorch/manylinuxs390x-builder
      DOCKER_IMAGE_TAG_PREFIX: cpu-s390x
      DESIRED_PYTHON: "3.14t"
      build_name: manywheel-py3_14t-cpu-s390x
````

- EN: This section describes repository automation behavior for `.github/workflows/generated-linux-s390x-binary-manywheel-nightly.yml`.
- CN: 该部分描述 `.github/workflows/generated-linux-s390x-binary-manywheel-nightly.yml` 的仓库自动化行为。

### Lines 520-525 / 第 520-525 行

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
