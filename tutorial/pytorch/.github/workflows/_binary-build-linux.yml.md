# _binary-build-linux.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/_binary-build-linux.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-13 / 第 1-13 行

````yaml
name: linux-binary-build

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
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 14-28 / 第 14-28 行

````yaml
      runner_prefix:
        required: false
        default: ""
        type: string
        description: prefix for runner label
      runs_on:
        required: false
        default: linux.12xlarge.memory.ephemeral
        type: string
        description: Hardware to run this "build" job on, linux.12xlarge or linux.arm64.2xlarge.
      timeout-minutes:
        required: false
        default: 240
        type: number
        description: timeout for the job
````

- EN: This section describes repository automation behavior for `.github/workflows/_binary-build-linux.yml`.
- CN: 该部分描述 `.github/workflows/_binary-build-linux.yml` 的仓库自动化行为。

### Lines 29-45 / 第 29-45 行

````yaml
      ALPINE_IMAGE:
        required: false
        type: string
        default: "308535385114.dkr.ecr.us-east-1.amazonaws.com/tool/alpine"
        description: Alpine image to use
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
````

- EN: This section describes repository automation behavior for `.github/workflows/_binary-build-linux.yml`.
- CN: 该部分描述 `.github/workflows/_binary-build-linux.yml` 的仓库自动化行为。

### Lines 46-61 / 第 46-61 行

````yaml
      GPU_ARCH_VERSION:
        required: false
        type: string
        description: GPU Arch version
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
````

- EN: This section describes repository automation behavior for `.github/workflows/_binary-build-linux.yml`.
- CN: 该部分描述 `.github/workflows/_binary-build-linux.yml` 的仓库自动化行为。

### Lines 62-78 / 第 62-78 行

````yaml
      LIBTORCH_CONFIG:
        required: false
        type: string
        description: Desired libtorch config (for libtorch builds only)
      LIBTORCH_VARIANT:
        required: false
        type: string
        description: Desired libtorch variant (for libtorch builds only)
      DESIRED_PYTHON:
        required: false
        type: string
        description: Desired python version
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS:
        required: false
        type: string
        description: Extra install requirements
        default: ""
````

- EN: This section describes repository automation behavior for `.github/workflows/_binary-build-linux.yml`.
- CN: 该部分描述 `.github/workflows/_binary-build-linux.yml` 的仓库自动化行为。

### Lines 79-94 / 第 79-94 行

````yaml
    secrets:
      github-token:
        required: true
        description: Github Token

permissions:
  id-token: write

jobs:
  build:
    runs-on: ${{ inputs.runner_prefix}}${{ inputs.runs_on }}
    timeout-minutes: ${{ inputs.timeout-minutes }}
    env:
      PYTORCH_ROOT: ${{ inputs.PYTORCH_ROOT }}
      PACKAGE_TYPE: ${{ inputs.PACKAGE_TYPE }}
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 95-112 / 第 95-112 行

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
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: ${{ inputs.PYTORCH_EXTRA_INSTALL_REQUIREMENTS }}
      ALPINE_IMAGE: ${{ inputs.ALPINE_IMAGE }}
      AWS_DEFAULT_REGION: us-east-1
      BINARY_ENV_FILE: /tmp/env
      BUILD_ENVIRONMENT: ${{ inputs.build_environment }}
      GITHUB_TOKEN: ${{ secrets.github-token }}
      PR_NUMBER: ${{ github.event.pull_request.number }}
      PYTORCH_FINAL_PACKAGE_DIR: /artifacts
      SHA1: ${{ github.event.pull_request.head.sha || github.sha }}
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 113-130 / 第 113-130 行

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
            echo "PYTORCH_EXTRA_INSTALL_REQUIREMENTS=${{ env.PYTORCH_EXTRA_INSTALL_REQUIREMENTS }}"
            echo "ALPINE_IMAGE=${{ env.ALPINE_IMAGE }}"
            echo "AWS_DEFAULT_REGION=${{ env.AWS_DEFAULT_REGION }}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 131-146 / 第 131-146 行

````yaml
            echo "BINARY_ENV_FILE=${{ env.BINARY_ENV_FILE }}"
            echo "BUILD_ENVIRONMENT=${{ env.BUILD_ENVIRONMENT }}"
            echo "BUILD_NAME=${{ env.BUILD_NAME }}"
            echo "PR_NUMBER=${{ env.PR_NUMBER }}"
            echo "PYTORCH_FINAL_PACKAGE_DIR=${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
            echo "SHA1=${{ env.SHA1 }}"
          } >> "${GITHUB_ENV} }}"

      - name: List the env
        shell: bash
        run: env

      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        if: inputs.build_environment != 'linux-s390x-binary-manywheel'
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 147-161 / 第 147-161 行

````yaml
        with:
          github-secret: ${{ secrets.github-token }}

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
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `pytorch/pytorch/.github/actions/setup-linux@main`, `./.github/actions/ecr-login`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 163-177 / 第 163-177 行

````yaml
      - name: Chown workspace
        if: inputs.build_environment != 'linux-s390x-binary-manywheel'
        uses: ./.github/actions/chown-workspace
        with:
          ALPINE_IMAGE: ${{ inputs.ALPINE_IMAGE }}

      - name: Clean workspace
        shell: bash
        run: |
          set -eux

          if [[ ${{ inputs.build_environment }} == 'linux-aarch64-binary-manywheel' ]] || [[ ${{ inputs.build_environment }} == 'linux-s390x-binary-manywheel' ]] ; then
            rm -rf "${RUNNER_TEMP}/artifacts"
            mkdir "${RUNNER_TEMP}/artifacts"
          fi
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/chown-workspace`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 179-192 / 第 179-192 行

````yaml
      - name: Checkout PyTorch to pytorch dir
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          fetch-depth: 2
          submodules: recursive
          path: pytorch
          show-progress: false

      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
        working-directory: pytorch
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 194-209 / 第 194-209 行

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

      - name: configure aws credentials
        id: aws_creds
        if: ${{ steps.filter.outputs.is-test-matrix-empty == 'False' && inputs.build_environment != 'linux-s390x-binary-manywheel' && startsWith(github.event.ref, 'refs/tags/ciflow/') }}
        uses: aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722 # v4.1.0
````

- EN: This section reuses actions and step building blocks such as `./pytorch/.github/actions/filter-test-configs`, `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 210-225 / 第 210-225 行

````yaml
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
          aws-region: us-east-1
          role-duration-seconds: 18000

      - name: Calculate docker image
        id: calculate-docker-image
        if: ${{ steps.filter.outputs.is-test-matrix-empty == 'False' && inputs.build_environment != 'linux-s390x-binary-manywheel' }}
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          # If doing this in main or release branch, use docker.io. Otherwise
          # use ECR
          docker-registry: ${{ startsWith(github.event.ref, 'refs/tags/ciflow/') && '308535385114.dkr.ecr.us-east-1.amazonaws.com' || 'docker.io' }}
          docker-image-name: ${{ inputs.DOCKER_IMAGE }}
          custom-tag-prefix: ${{ inputs.DOCKER_IMAGE_TAG_PREFIX }}
          # The build.sh script in this folder is not actually the correct one,
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/calculate-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 226-238 / 第 226-238 行

````yaml
          # this is just needed for sha calculation
          docker-build-dir: .ci/docker
          docker-build-script: ${{ contains(inputs.DOCKER_IMAGE, 'manylinux') && 'manywheel/build.sh' || 'build.sh' }}
          working-directory: pytorch

      - name: Pull Docker image
        if: ${{ steps.filter.outputs.is-test-matrix-empty == 'False' && inputs.build_environment != 'linux-s390x-binary-manywheel' }}
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}

      - name: Build PyTorch binary
        if: ${{ steps.filter.outputs.is-test-matrix-empty == 'False' }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/pull-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 239-256 / 第 239-256 行

````yaml
        env:
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image || format('{0}:{1}', inputs.DOCKER_IMAGE, inputs.DOCKER_IMAGE_TAG_PREFIX) }}
        run: |
          set -x
          mkdir -p artifacts/
          container_name=$(docker run \
            -e BINARY_ENV_FILE \
            -e BUILD_ENVIRONMENT \
            -e DESIRED_CUDA \
            -e DESIRED_PYTHON \
            -e GITHUB_ACTIONS \
            -e GPU_ARCH_TYPE \
            -e GPU_ARCH_VERSION \
            -e LIBTORCH_VARIANT \
            -e PACKAGE_TYPE \
            -e PYTORCH_FINAL_PACKAGE_DIR \
            -e PYTORCH_ROOT \
            -e SKIP_ALL_TESTS \
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 257-272 / 第 257-272 行

````yaml
            -e PYTORCH_EXTRA_INSTALL_REQUIREMENTS \
            --tty \
            --detach \
            -v "${GITHUB_WORKSPACE}:/pytorch" \
            -v "${RUNNER_TEMP}/artifacts:/artifacts" \
            -w / \
            "${DOCKER_IMAGE}"
          )
          docker exec -t -w "${PYTORCH_ROOT}" "${container_name}" bash -c "bash .ci/pytorch/binary_populate_env.sh"
          # Unified build script for all architectures (x86_64, aarch64, s390x)
          docker exec -t "${container_name}" bash -c "source ${BINARY_ENV_FILE} && bash /pytorch/.ci/${{ inputs.PACKAGE_TYPE }}/build.sh"

      - name: Chown artifacts
        if: ${{ steps.filter.outputs.is-test-matrix-empty == 'False' && inputs.build_environment != 'linux-s390x-binary-manywheel' }}
        shell: bash
        run: |
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 273-286 / 第 273-286 行

````yaml
          # Ensure the working directory gets chowned back to the current user
          docker run --rm -v "${RUNNER_TEMP}/artifacts:/v" -w /v "${ALPINE_IMAGE}" chown -R "$(id -u):$(id -g)" .

      - uses: actions/upload-artifact@50769540e7f4bd5e21e526ee35c689e35e0d6874 # v4.4.0
        if: ${{ steps.filter.outputs.is-test-matrix-empty == 'False' }}
        with:
          name: ${{ inputs.build_name }}
          if-no-files-found: error
          path:
            ${{ runner.temp }}/artifacts/*

      - name: Teardown Linux
        if: always() && inputs.build_environment != 'linux-s390x-binary-manywheel'
        uses: pytorch/test-infra/.github/actions/teardown-linux@main
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/teardown-linux@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 288-299 / 第 288-299 行

````yaml
      - name: Chown workspace
        if: always() && inputs.build_environment != 'linux-s390x-binary-manywheel'
        uses: ./pytorch/.github/actions/chown-workspace
        with:
          ALPINE_IMAGE: ${{ inputs.ALPINE_IMAGE }}

      - name: Cleanup docker
        if: always() && inputs.build_environment == 'linux-s390x-binary-manywheel'
        shell: bash
        run: |
          # on s390x stop the container for clean worker stop
          # ignore expansion of "docker ps -q" since it could be empty
````

- EN: This section reuses actions and step building blocks such as `./pytorch/.github/actions/chown-workspace`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 300-301 / 第 300-301 行

````yaml
          # shellcheck disable=SC2046
          docker stop $(docker ps -q) || true
````

- EN: This section describes repository automation behavior for `.github/workflows/_binary-build-linux.yml`.
- CN: 该部分描述 `.github/workflows/_binary-build-linux.yml` 的仓库自动化行为。


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
- Reusable actions / 复用 Action: `pytorch/test-infra/.github/actions/setup-ssh@main`, `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `pytorch/pytorch/.github/actions/setup-linux@main`, `./.github/actions/ecr-login`, `./.github/actions/chown-workspace`, `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`, `./pytorch/.github/actions/filter-test-configs`, `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`, ...
- Inline commands / 内联命令: `echo`, `set`, `rm`, `mkdir`, `git`, `]}`, `e`, `tty`, ...
- Environment variables / 环境变量: `ALPINE_IMAGE`, `PYTORCH_ROOT`, `PACKAGE_TYPE`, `DESIRED_CUDA`, `GPU_ARCH_VERSION`, `GPU`, `GPU_ARCH_TYPE`, `DOCKER_IMAGE`, `DOCKER_IMAGE_TAG_PREFIX`, `LIBTORCH_CONFIG`, ...
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `inputs`, `secrets`, `permissions`, `jobs`, `build`, `env`, `steps`
