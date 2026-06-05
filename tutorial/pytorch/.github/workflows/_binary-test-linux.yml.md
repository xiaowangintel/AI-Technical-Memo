# _binary-test-linux.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/_binary-test-linux.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-17 / 第 1-17 行

````yaml
name: linux-binary-test

on:
  workflow_call:
    inputs:
      build_name:
        required: true
        type: string
        description: The build's name
      build_environment:
        required: true
        type: string
        description: The build environment
      ALPINE_IMAGE:
        required: false
        type: string
        default: "308535385114.dkr.ecr.us-east-1.amazonaws.com/tool/alpine"
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 18-33 / 第 18-33 行

````yaml
      PYTORCH_ROOT:
        required: true
        type: string
        description: Root directory for the pytorch/pytorch repository
      PACKAGE_TYPE:
        required: true
        type: string
        description: Package type
      DESIRED_CUDA:
        required: true
        type: string
        description: Desired Cuda version
      GPU_ARCH_VERSION:
        required: false
        type: string
        description: GPU Arch version
````

- EN: This section describes repository automation behavior for `.github/workflows/_binary-test-linux.yml`.
- CN: 该部分描述 `.github/workflows/_binary-test-linux.yml` 的仓库自动化行为。

### Lines 34-49 / 第 34-49 行

````yaml
      GPU_ARCH_TYPE:
        required: true
        type: string
        description: GPU Arch type
      DOCKER_IMAGE:
        required: true
        type: string
        description: Docker image to use
      DOCKER_IMAGE_TAG_PREFIX:
        required: true
        type: string
        description: Docker image tag to use
      LIBTORCH_CONFIG:
        required: false
        type: string
        description: Desired libtorch config (for libtorch builds only)
````

- EN: This section describes repository automation behavior for `.github/workflows/_binary-test-linux.yml`.
- CN: 该部分描述 `.github/workflows/_binary-test-linux.yml` 的仓库自动化行为。

### Lines 50-66 / 第 50-66 行

````yaml
      LIBTORCH_VARIANT:
        required: false
        type: string
        description: Desired libtorch variant (for libtorch builds only)
      DESIRED_PYTHON:
        required: false
        type: string
        description: Desired python version
      runner_prefix:
        required: false
        default: ""
        type: string
        description: prefix for runner label
      runs_on:
        required: true
        type: string
        description: Hardware to run this job on. Valid values are linux.4xlarge, linux.g4dn.4xlarge.nvidia.gpu, linux.arm64.2xlarge, and linux.rocm.gpu
````

- EN: This section describes repository automation behavior for `.github/workflows/_binary-test-linux.yml`.
- CN: 该部分描述 `.github/workflows/_binary-test-linux.yml` 的仓库自动化行为。

### Lines 67-82 / 第 67-82 行

````yaml
    secrets:
      github-token:
        required: true
        description: Github Token

permissions:
  id-token: write

jobs:
  test:
    runs-on: ${{ inputs.runner_prefix}}${{ inputs.runs_on }}
    timeout-minutes: 240
    env:
      PYTORCH_ROOT: ${{ inputs.PYTORCH_ROOT }}
      PACKAGE_TYPE: ${{ inputs.PACKAGE_TYPE }}
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 83-99 / 第 83-99 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: ${{ inputs.DESIRED_CUDA }}
      GPU_ARCH_VERSION: ${{ inputs.GPU_ARCH_VERSION }}
      GPU_ARCH_TYPE: ${{ inputs.GPU_ARCH_TYPE }}
      DOCKER_IMAGE: ${{ inputs.DOCKER_IMAGE }}
      SKIP_ALL_TESTS: 1
      LIBTORCH_CONFIG: ${{ inputs.LIBTORCH_CONFIG }}
      LIBTORCH_VARIANT: ${{ inputs.LIBTORCH_VARIANT }}
      DESIRED_PYTHON: ${{ inputs.DESIRED_PYTHON }}
      ALPINE_IMAGE: ${{ inputs.ALPINE_IMAGE }}
      AWS_DEFAULT_REGION: us-east-1
      BINARY_ENV_FILE: /tmp/env
      BUILD_ENVIRONMENT: ${{ inputs.build_environment }}
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      PR_NUMBER: ${{ github.event.pull_request.number }}
      PYTORCH_FINAL_PACKAGE_DIR: /artifacts
      SHA1: ${{ github.event.pull_request.head.sha || github.sha }}
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 100-115 / 第 100-115 行

````yaml
    steps:
      - name: Make the env permanent during this workflow (but not the secrets)
        shell: bash
        run: |
          {
            echo "PYTORCH_ROOT=${{ env.PYTORCH_ROOT }}"
            echo "PACKAGE_TYPE=${{ env.PACKAGE_TYPE }}"

            echo "DESIRED_CUDA=${{ env.DESIRED_CUDA }}"
            echo "GPU_ARCH_VERSION=${{ env.GPU_ARCH_VERSION }}"
            echo "GPU_ARCH_TYPE=${{ env.GPU_ARCH_TYPE }}"
            echo "DOCKER_IMAGE=${{ env.DOCKER_IMAGE }}"
            echo "SKIP_ALL_TESTS=${{ env.SKIP_ALL_TESTS }}"
            echo "LIBTORCH_CONFIG=${{ env.LIBTORCH_CONFIG }}"
            echo "LIBTORCH_VARIANT=${{ env.LIBTORCH_VARIANT }}"
            echo "DESIRED_PYTHON=${{ env.DESIRED_PYTHON }}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 117-133 / 第 117-133 行

````yaml
            echo "ALPINE_IMAGE=${{ env.ALPINE_IMAGE }}"
            echo "AWS_DEFAULT_REGION=${{ env.AWS_DEFAULT_REGION }}"
            echo "BINARY_ENV_FILE=${{ env.BINARY_ENV_FILE }}"
            echo "BUILD_ENVIRONMENT=${{ env.BUILD_ENVIRONMENT }}"
            echo "PR_NUMBER=${{ env.PR_NUMBER }}"
            echo "PYTORCH_FINAL_PACKAGE_DIR=${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
            echo "SHA1=${{ env.SHA1 }}"
          } >> "${GITHUB_ENV} }}"

      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        if: inputs.build_environment != 'linux-s390x-binary-manywheel'
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.github-token }}

        # Setup the environment
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 134-149 / 第 134-149 行

````yaml
      - name: Checkout PyTorch
        uses: pytorch/pytorch/.github/actions/checkout-pytorch@main
        with:
          no-sudo: ${{ inputs.build_environment == 'linux-aarch64-binary-manywheel' || inputs.build_environment == 'linux-s390x-binary-manywheel' }}

      - name: Setup Linux
        if: inputs.build_environment != 'linux-s390x-binary-manywheel'
        uses: pytorch/pytorch/.github/actions/setup-linux@main

      - name: Login to ECR
        if: inputs.build_environment != 'linux-s390x-binary-manywheel'
        uses: ./.github/actions/ecr-login

      - name: Chown workspace
        if: inputs.build_environment != 'linux-s390x-binary-manywheel'
        uses: ./.github/actions/chown-workspace
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `pytorch/pytorch/.github/actions/setup-linux@main`, `./.github/actions/ecr-login`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 150-165 / 第 150-165 行

````yaml
        with:
          ALPINE_IMAGE: ${{ inputs.ALPINE_IMAGE }}

      - name: Checkout PyTorch to pytorch dir
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
          path: pytorch

      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
        working-directory: pytorch
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 167-181 / 第 167-181 行

````yaml
      - name: Check if the job is disabled
        id: filter
        uses: ./pytorch/.github/actions/filter-test-configs
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          # NB: Use a mock test matrix with a default value here. After filtering, if the
          # returned matrix is empty, it means that the job is disabled
          test-matrix: |
            { include: [
              { config: "default" },
            ]}

      - name: Download Build Artifacts
        if: ${{ steps.filter.outputs.is-test-matrix-empty == 'False' }}
        uses: actions/download-artifact@65a9edc5881444af0b9093a5e628f2fe47ea3b2e # v4.1.7
````

- EN: This section reuses actions and step building blocks such as `./pytorch/.github/actions/filter-test-configs`, `actions/download-artifact@65a9edc5881444af0b9093a5e628f2fe47ea3b2e`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 182-198 / 第 182-198 行

````yaml
        with:
          name: ${{ inputs.build_name }}
          path: "${{ runner.temp }}/artifacts/"

      - name: Install nvidia driver, nvidia-docker runtime, set GPU_FLAG
        id: install-nvidia-driver
        uses: pytorch/test-infra/.github/actions/setup-nvidia@main
        if: ${{ steps.filter.outputs.is-test-matrix-empty == 'False' }}

      - name: configure aws credentials
        id: aws_creds
        if: ${{ steps.filter.outputs.is-test-matrix-empty == 'False' && inputs.build_environment != 'linux-s390x-binary-manywheel' && startsWith(github.event.ref, 'refs/tags/ciflow/') }}
        uses: aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722 # v4.1.0
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
          aws-region: us-east-1
          role-duration-seconds: 18000
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-nvidia@main`, `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 200-215 / 第 200-215 行

````yaml
      - name: Calculate docker image
        id: calculate-docker-image
        if: ${{ steps.filter.outputs.is-test-matrix-empty == 'False' && inputs.build_environment != 'linux-s390x-binary-manywheel' }}
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: ${{ inputs.DOCKER_IMAGE }}
          custom-tag-prefix: ${{ inputs.DOCKER_IMAGE_TAG_PREFIX }}
          docker-build-dir: .ci/docker
          working-directory: pytorch

      - name: Pull Docker image
        if: ${{ steps.filter.outputs.is-test-matrix-empty == 'False' && inputs.build_environment != 'linux-s390x-binary-manywheel' }}
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 217-229 / 第 217-229 行

````yaml
      - name: Test Pytorch binary
        if: ${{ steps.filter.outputs.is-test-matrix-empty == 'False' }}
        uses: ./pytorch/.github/actions/test-pytorch-binary
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image || format('{0}:{1}', inputs.DOCKER_IMAGE, inputs.DOCKER_IMAGE_TAG_PREFIX) }}

      - name: Teardown Linux
        if: always() && inputs.build_environment != 'linux-s390x-binary-manywheel'
        uses: pytorch/test-infra/.github/actions/teardown-linux@main

      - name: Chown workspace
        if: always() && inputs.build_environment != 'linux-s390x-binary-manywheel'
        uses: ./pytorch/.github/actions/chown-workspace
````

- EN: This section reuses actions and step building blocks such as `./pytorch/.github/actions/test-pytorch-binary`, `pytorch/test-infra/.github/actions/teardown-linux@main`, `./pytorch/.github/actions/chown-workspace`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 230-231 / 第 230-231 行

````yaml
        with:
          ALPINE_IMAGE: ${{ inputs.ALPINE_IMAGE }}
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
- Reusable actions / 复用 Action: `pytorch/test-infra/.github/actions/setup-ssh@main`, `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `pytorch/pytorch/.github/actions/setup-linux@main`, `./.github/actions/ecr-login`, `./.github/actions/chown-workspace`, `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`, `./pytorch/.github/actions/filter-test-configs`, `actions/download-artifact@65a9edc5881444af0b9093a5e628f2fe47ea3b2e`, ...
- Inline commands / 内联命令: `echo`, `git`, `]}`
- Environment variables / 环境变量: `ALPINE_IMAGE`, `PYTORCH_ROOT`, `PACKAGE_TYPE`, `DESIRED_CUDA`, `GPU_ARCH_VERSION`, `GPU`, `GPU_ARCH_TYPE`, `DOCKER_IMAGE`, `DOCKER_IMAGE_TAG_PREFIX`, `LIBTORCH_CONFIG`, ...
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `inputs`, `secrets`, `permissions`, `jobs`, `test`, `env`, `steps`
