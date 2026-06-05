# build-vllm-wheel.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/build-vllm-wheel.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-17 / 第 1-17 行

````yaml
name: Build vLLM wheels

on:
  push:
    branches:
      - main
    paths:
      - .github/workflows/build-vllm-wheel.yml
      - .github/ci_commit_pins/vllm.txt
  workflow_dispatch:
  pull_request:
    paths:
      - .github/workflows/build-vllm-wheel.yml
      - .github/ci_commit_pins/vllm.txt
  schedule:
    # every morning at 01:30PM UTC, 9:30AM EST, 6:30AM PST
    - cron: 30 13 * * *
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 19-32 / 第 19-32 行

````yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.sha }}-${{ github.event_name == 'workflow_dispatch' }}
  cancel-in-progress: true

jobs:
  build-wheel:
    if: github.repository_owner == 'pytorch'
    strategy:
      fail-fast: false
      matrix:
        python-version: [ '3.12' ]
        platform: [ 'manylinux_2_28_x86_64', 'manylinux_2_28_aarch64' ]
        device: [ 'cu130', 'cu132' ]
        include:
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段。

### Lines 33-44 / 第 33-44 行

````yaml
          - platform: manylinux_2_28_x86_64
            device: cu130
            manylinux-image: 'pytorch/manylinux2_28-builder:cuda13.0'
            runner: linux.12xlarge.memory
          - platform: manylinux_2_28_x86_64
            device: cu132
            manylinux-image: 'pytorch/manylinux2_28-builder:cuda13.2'
            runner: linux.12xlarge.memory
          - platform: manylinux_2_28_aarch64
            device: cu130
            manylinux-image: 'pytorch/manylinuxaarch64-builder:cuda13.0'
            runner: linux.arm64.r7g.12xlarge.memory
````

- EN: This section describes repository automation behavior for `.github/workflows/build-vllm-wheel.yml`.
- CN: 该部分描述 `.github/workflows/build-vllm-wheel.yml` 的仓库自动化行为。

### Lines 45-61 / 第 45-61 行

````yaml
          - platform: manylinux_2_28_aarch64
            device: cu132
            manylinux-image: 'pytorch/manylinuxaarch64-builder:cuda13.2'
            runner: linux.arm64.r7g.12xlarge.memory
    name: "Build ${{ matrix.device }} vLLM wheel on ${{ matrix.platform }}"
    runs-on: ${{ matrix.runner }}
    timeout-minutes: 480
    env:
      PY_VERS: ${{ matrix.python-version }}
      MANYLINUX_IMAGE: ${{ matrix.manylinux-image }}
      PLATFORM: ${{ matrix.platform }}
      BUILD_DEVICE: ${{ matrix.device }}
    steps:
      - name: Setup SSH (Click me for login details)
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 63-74 / 第 63-74 行

````yaml
      - name: Setup Linux
        uses: pytorch/pytorch/.github/actions/setup-linux@main
        with:
          submodules: false

      - name: Login to ECR
        uses: ./.github/actions/ecr-login

      - name: Get latest PyTorch nightly
        shell: bash
        run: |
          set -eux
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/setup-linux@main`, `./.github/actions/ecr-login`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 76-93 / 第 76-93 行

````yaml
          # Determine python executable for given version (copied from build-triton-wheel)
          case $PY_VERS in
          3.10)
            PYTHON_EXECUTABLE=/opt/python/cp310-cp310/bin/python
            ;;
          3.11)
            PYTHON_EXECUTABLE=/opt/python/cp311-cp311/bin/python
            ;;
          3.12)
            PYTHON_EXECUTABLE=/opt/python/cp312-cp312/bin/python
            ;;
          3.13)
            PYTHON_EXECUTABLE=/opt/python/cp313-cp313/bin/python
            ;;
          3.13t)
            PYTHON_EXECUTABLE=/opt/python/cp313-cp313t/bin/python
            ;;
          3.14)
````

- EN: This section describes repository automation behavior for `.github/workflows/build-vllm-wheel.yml`.
- CN: 该部分描述 `.github/workflows/build-vllm-wheel.yml` 的仓库自动化行为。

### Lines 94-107 / 第 94-107 行

````yaml
            PYTHON_EXECUTABLE=/opt/python/cp314-cp314/bin/python
            ;;
          3.14t)
            PYTHON_EXECUTABLE=/opt/python/cp314-cp314t/bin/python
            ;;
          *)
            echo "Unsupported python version ${PY_VERS}"
            exit 1
            ;;
          esac

          # Keep PyTorch nightly wheel here so that we can install it later during
          # vLLM build process
          mkdir -p "${RUNNER_TEMP}/artifacts/"
````

- EN: This section describes repository automation behavior for `.github/workflows/build-vllm-wheel.yml`.
- CN: 该部分描述 `.github/workflows/build-vllm-wheel.yml` 的仓库自动化行为。

### Lines 109-124 / 第 109-124 行

````yaml
          container_name=$(docker run \
            --tty \
            --detach \
            -e PLATFORM \
            -e PYTHON_EXECUTABLE="${PYTHON_EXECUTABLE}" \
            -v "${GITHUB_WORKSPACE}:/pytorch" \
            -v "${RUNNER_TEMP}/artifacts:/artifacts" \
            -w /artifacts/ \
            "${MANYLINUX_IMAGE}"
          )

          docker exec -t "${container_name}" "${PYTHON_EXECUTABLE}" -mpip install \
            --pre torch torchvision torchaudio \
            --index-url "https://download.pytorch.org/whl/nightly/${BUILD_DEVICE}"

          # I wonder if there is a command to both download and install the wheels
````

- EN: This section describes repository automation behavior for `.github/workflows/build-vllm-wheel.yml`.
- CN: 该部分描述 `.github/workflows/build-vllm-wheel.yml` 的仓库自动化行为。

### Lines 125-140 / 第 125-140 行

````yaml
          # in one go
          docker exec -t "${container_name}" "${PYTHON_EXECUTABLE}" -mpip download \
            --pre torch torchvision torchaudio \
            --index-url "https://download.pytorch.org/whl/nightly/${BUILD_DEVICE}"

          # Save this for later
          echo "container_name=${container_name}" >> "$GITHUB_ENV"

      - name: Build vLLM wheel
        uses: ./.github/actions/build-external-packages
        with:
          build-targets: vllm
          docker-image: ${{ env.MANYLINUX_IMAGE }}
          cuda-arch-list: '8.0;8.9;9.0;10.0;12.0'
          torch-wheel-dir: ${{ runner.temp }}/artifacts
          output-dir: ${{ runner.temp }}/artifacts/externals
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/build-external-packages`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 142-155 / 第 142-155 行

````yaml
      - name: Prepare vLLM wheel
        shell: bash
        run: |
          set -eux

          # Get these wheels ready, the vllm renaming logic is copied from its .buildkite/scripts/upload-wheels.sh
          docker exec -t "${container_name}" bash -c /pytorch/.github/scripts/prepare_vllm_wheels.sh
          docker exec -t "${container_name}" chown -R 1000:1000 /artifacts

      - uses: actions/upload-artifact@50769540e7f4bd5e21e526ee35c689e35e0d6874 # v4.4.0
        with:
          name: vllm-wheel-${{ matrix.device }}-${{ matrix.platform }}-${{ matrix.python-version }}
          if-no-files-found: error
          path: ${{ runner.temp }}/artifacts/externals/vllm/wheels/*.whl
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 157-171 / 第 157-171 行

````yaml
      - name: Teardown Linux
        uses: pytorch/test-infra/.github/actions/teardown-linux@main
        if: always()

  # Copied from build-triton-wheel workflow (mostly)
  upload-wheel:
    name: "Upload ${{ matrix.device }} vLLM wheel on ${{ matrix.platform }}"
    needs:
      - build-wheel
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        platform: [ 'manylinux_2_28_x86_64', 'manylinux_2_28_aarch64' ]
        device: [ 'cu130', 'cu132' ]
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/teardown-linux@main`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 172-186 / 第 172-186 行

````yaml
    env:
      PLATFORM: ${{ matrix.platform }}
      BUILD_DEVICE: ${{ matrix.device }}
    permissions:
      id-token: write
      contents: read
    container:
      image: continuumio/miniconda3:4.12.0
    environment: ${{ ((github.event_name == 'push' && github.event.ref == 'refs/heads/main') || github.event_name == 'schedule' || github.event_name == 'workflow_dispatch') && 'nightly-wheel-upload' || '' }}
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2

      - name: Configure AWS credentials(PyTorch account) for main
        if: ${{ (github.event_name == 'push' && github.event.ref == 'refs/heads/main') || github.event_name == 'schedule' || github.event_name == 'workflow_dispatch' }}
        uses: aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722 # v4.1.0
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 187-202 / 第 187-202 行

````yaml
        with:
          role-to-assume: arn:aws:iam::749337293305:role/gha_workflow_nightly_build_wheels
          aws-region: us-east-1

      - name: Configure AWS credentials(PyTorch account) for RC builds
        if: ${{ github.event_name == 'push' &&  (startsWith(github.event.ref, 'refs/tags/') && !startsWith(github.event.ref, 'refs/tags/ciflow/')) }}
        uses: aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722 # v4.1.0
        with:
          role-to-assume: arn:aws:iam::749337293305:role/gha_workflow_test_build_wheels
          aws-region: us-east-1

      - name: Download Build Artifacts
        uses: actions/download-artifact@65a9edc5881444af0b9093a5e628f2fe47ea3b2e # v4.1.7
        with:
          # Download all available artifacts
          path: ${{ runner.temp }}/artifacts-all
````

- EN: This section reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`, `actions/download-artifact@65a9edc5881444af0b9093a5e628f2fe47ea3b2e`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 204-215 / 第 204-215 行

````yaml
      - name: Select Wheel Artifacts
        shell: bash
        run: |
          set -eux
          mkdir -p "${RUNNER_TEMP}/artifacts/"
          mv "${RUNNER_TEMP}"/artifacts-all/vllm-wheel-"${BUILD_DEVICE}"-"${PLATFORM}"-*/* "${RUNNER_TEMP}/artifacts/"

      - name: Set DRY_RUN
        if: ${{ (github.event_name == 'push' && (github.event.ref == 'refs/heads/main' || startsWith(github.event.ref, 'refs/tags/v'))) || github.event_name == 'schedule' || github.event_name == 'workflow_dispatch' }}
        shell: bash
        run: |
          echo "DRY_RUN=disabled" >> "$GITHUB_ENV"
````

- EN: This section declares workflow triggers or event bindings; embeds shell commands that perform the actual side effects.
- CN: 该部分声明工作流触发条件或事件绑定；内嵌执行实际副作用的 shell 命令。

### Lines 217-227 / 第 217-227 行

````yaml
      - name: Set UPLOAD_CHANNEL
        if: ${{ github.event_name == 'push' && startsWith(github.event.ref, 'refs/tags/v') }}
        shell: bash
        run: |
          set -ex

          if [[ "${GITHUB_REF_NAME}" = *-rc[0-9]* ]]; then
            echo "UPLOAD_CHANNEL=test" >> "$GITHUB_ENV"
          fi

      - name: Upload binaries
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 228-239 / 第 228-239 行

````yaml
        env:
          PACKAGE_TYPE: wheel
          UPLOAD_SUBFOLDER: ${{ env.BUILD_DEVICE }}
          PKG_DIR: ${{ runner.temp }}/artifacts
          R2_UPLOAD: ${{ ((github.event_name == 'push' && github.event.ref == 'refs/heads/main') || github.event_name == 'schedule' || github.event_name == 'workflow_dispatch') && 'true' || '' }}
          R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
          R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
          R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
        shell: bash
        run: |
          set -ex
          bash .ci/pytorch/binary_upload.sh
````

- EN: This section declares workflow triggers or event bindings; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。


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
- Reusable actions / 复用 Action: `pytorch/test-infra/.github/actions/setup-ssh@main`, `pytorch/pytorch/.github/actions/setup-linux@main`, `./.github/actions/ecr-login`, `./.github/actions/build-external-packages`, `pytorch/test-infra/.github/actions/teardown-linux@main`, `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`, `actions/download-artifact@65a9edc5881444af0b9093a5e628f2fe47ea3b2e`
- Inline commands / 内联命令: `set`, `3.10`, `3.11`, `3.12`, `3.13`, `3.13t`, `3.14`, `3.14t`, ...
- Environment variables / 环境变量: `UTC`, `EST`, `PST`, `PY_VERS`, `MANYLINUX_IMAGE`, `PLATFORM`, `BUILD_DEVICE`, `SSH`, `GITHUB_TOKEN`, `ECR`, ...
- Named jobs or sections / 命名作业或章节: `on`, `push`, `branches`, `paths`, `workflow_dispatch`, `pull_request`, `schedule`, `concurrency`, `jobs`, `build-wheel`, ...
