# build-triton-wheel.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/build-triton-wheel.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-17 / 第 1-17 行

````yaml
name: Build Triton wheels

on:
  push:
    branches:
      - main
    tags:
      # NOTE: Binary build pipelines should only get triggered on release candidate builds
      # Release candidate tags look like: v1.11.0-rc1
      - v[0-9]+.[0-9]+.[0-9]+-rc[0-9]+
      - 'ciflow/triton_binaries/*'
    paths:
      - .github/workflows/build-triton-wheel.yml
      - .github/scripts/build_triton_wheel.py
      - .github/ci_commit_pins/triton.txt
      - .github/scripts/windows/install_vs2022.ps1
      - .github/scripts/windows/build_triton.bat
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 18-33 / 第 18-33 行

````yaml
      - .ci/docker/ci_commit_pins/triton.txt
      - .ci/docker/ci_commit_pins/triton-xpu.txt
  workflow_dispatch:
  pull_request:
    paths:
      - .github/workflows/build-triton-wheel.yml
      - .github/scripts/build_triton_wheel.py
      - .github/ci_commit_pins/triton.txt
      - .github/scripts/windows/install_vs2022.ps1
      - .github/scripts/windows/build_triton.bat
      - .ci/docker/ci_commit_pins/triton.txt
      - .ci/docker/ci_commit_pins/triton-xpu.txt

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.sha }}-${{ github.event_name == 'workflow_dispatch' }}
  cancel-in-progress: true
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 35-51 / 第 35-51 行

````yaml
jobs:
  get-label-type:
    if: github.repository_owner == 'pytorch'
    name: get-label-type
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}

  build-wheel:
    name: "Build Triton Wheel"
    needs: get-label-type
    runs-on: ${{ matrix.runs_on }}
    strategy:
      fail-fast: false
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 52-65 / 第 52-65 行

````yaml
      matrix:
        py_vers: [ "3.10", "3.11", "3.12", "3.13", "3.13t", "3.14", "3.14t" ]
        device: ["cuda", "rocm", "xpu", "aarch64"]
        docker-image: ["pytorch/manylinux2_28-builder:cpu"]
        include:
          - device: "rocm"
            rocm_version: "7.2"
            runs_on: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge"
          - device: "cuda"
            rocm_version: ""
            runs_on: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge"
          - device: "xpu"
            rocm_version: ""
            runs_on: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge"
````

- EN: This section describes repository automation behavior for `.github/workflows/build-triton-wheel.yml`.
- CN: 该部分描述 `.github/workflows/build-triton-wheel.yml` 的仓库自动化行为。

### Lines 66-82 / 第 66-82 行

````yaml
          - device: "aarch64"
            rocm_version: ""
            runs_on: "${{ needs.get-label-type.outputs.label-type }}linux.arm64.2xlarge"
    timeout-minutes: 40
    env:
      DOCKER_IMAGE: ${{ matrix.device == 'rocm' && format('pytorch/manylinux2_28-builder:rocm{0}', matrix.rocm_version) || matrix.device == 'aarch64' && 'pytorch/manylinux2_28_aarch64-builder:cpu-aarch64' || matrix.docker-image }}
      PY_VERS: ${{ matrix.py_vers }}
      BUILD_DEVICE: ${{ matrix.device }}
      PLATFORM: 'manylinux_2_28_x86_64'
    steps:
      - name: Setup SSH (Click me for login details)
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}

      - name: Setup Linux
        uses: pytorch/pytorch/.github/actions/setup-linux@main
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`, `pytorch/pytorch/.github/actions/setup-linux@main`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 83-94 / 第 83-94 行

````yaml
        with:
          submodules: false

      - name: Login to ECR
        uses: ./.github/actions/ecr-login

      - name: Pull Docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ env.DOCKER_IMAGE }}

      - name: Build Triton wheel
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/ecr-login`, `pytorch/test-infra/.github/actions/pull-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 95-107 / 第 95-107 行

````yaml
        env:
          IS_RELEASE_TAG: ${{ startsWith(github.event.ref, 'refs/tags/v') }}
        run: |
          set -x
          mkdir -p "${RUNNER_TEMP}/artifacts/"
          container_name=$(docker run \
            --tty \
            --detach \
            -v "${GITHUB_WORKSPACE}:/pytorch" \
            -v "${RUNNER_TEMP}/artifacts:/artifacts" \
            -w /artifacts/ \
            "${DOCKER_IMAGE}"      \
          )
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 109-126 / 第 109-126 行

````yaml
          # Determine python executable for given version
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

- EN: This section describes repository automation behavior for `.github/workflows/build-triton-wheel.yml`.
- CN: 该部分描述 `.github/workflows/build-triton-wheel.yml` 的仓库自动化行为。

### Lines 127-142 / 第 127-142 行

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

          RELEASE=""
          WITH_CLANG_LDD=""
          if [[ "${IS_RELEASE_TAG}" == true ]]; then
            RELEASE="--release"
          fi
````

- EN: This section describes repository automation behavior for `.github/workflows/build-triton-wheel.yml`.
- CN: 该部分描述 `.github/workflows/build-triton-wheel.yml` 的仓库自动化行为。

### Lines 144-160 / 第 144-160 行

````yaml
          docker exec -t "${container_name}" yum install -y zlib-devel zip
          docker exec -t "${container_name}" "${PYTHON_EXECUTABLE}"  -m pip install -U setuptools==78.1.0 pybind11==3.0.1 auditwheel wheel
          set +e
          docker exec -t "${container_name}" command -v pip
          has_pip=$?
          set -e
          if [ $has_pip -eq 0 ] ; then
              docker exec -t "${container_name}" pip install -U cmake --force-reinstall
          else
              docker exec -t "${container_name}" "${PYTHON_EXECUTABLE}"  -m pip install -U cmake --force-reinstall
          fi

          if [[ ("${{ matrix.device }}" == "cuda" || "${{ matrix.device }}" == "rocm" || "${{ matrix.device }}" == "aarch64" ) ]]; then
            # With this install, it gets clang 16.0.6.
            docker exec -t "${container_name}" dnf install clang lld -y
            WITH_CLANG_LDD="--with-clang-ldd"
          fi
````

- EN: This section describes repository automation behavior for `.github/workflows/build-triton-wheel.yml`.
- CN: 该部分描述 `.github/workflows/build-triton-wheel.yml` 的仓库自动化行为。

### Lines 162-176 / 第 162-176 行

````yaml
          docker exec -t "${container_name}" bash -c "${PYTHON_EXECUTABLE} /pytorch/.github/scripts/build_triton_wheel.py --device=$BUILD_DEVICE $RELEASE $WITH_CLANG_LDD"

          if [[ ("${{ matrix.device }}" == "cuda" || "${{ matrix.device }}" == "xpu") ]]; then
            docker exec -t "${container_name}"  bash -c "auditwheel repair --plat ${PLATFORM} --exclude libtriton.so //artifacts/*.whl"
          else
            docker exec -t "${container_name}"  bash -c "mkdir //artifacts/wheelhouse"
            docker exec -t "${container_name}"  bash -c "mv //artifacts/*.whl //artifacts/wheelhouse/"
          fi
          docker exec -t "${container_name}" chown -R 1000.1000 /artifacts/wheelhouse

      - uses: actions/upload-artifact@50769540e7f4bd5e21e526ee35c689e35e0d6874 # v4.4.0
        with:
          name: triton-wheel-${{ matrix.py_vers }}-${{ matrix.device }}-${{ env.PLATFORM }}
          if-no-files-found: error
          path: ${{ runner.temp }}/artifacts/wheelhouse/*
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 178-191 / 第 178-191 行

````yaml
      - name: Teardown Linux
        uses: pytorch/test-infra/.github/actions/teardown-linux@main
        if: always()

  build-wheel-win:
    name: "Build Triton Windows Wheel"
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.4xlarge"
    strategy:
      fail-fast: false
      matrix:
        py_vers: [ "3.10", "3.11", "3.12", "3.13", "3.13t", "3.14", "3.14t" ]
        device: ["xpu"]
    timeout-minutes: 40
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/teardown-linux@main`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 192-202 / 第 192-202 行

````yaml
    env:
      PY_VERS: ${{ matrix.py_vers }}
      BUILD_DEVICE: ${{ matrix.device }}
      VC_INSTALL_PATH: "C:\\MSVC-BuildTools-2022"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 203-215 / 第 203-215 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 216-231 / 第 216-231 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true
          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      - name: Checkout PyTorch
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: false
          path: pytorch
          show-progress: false
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 232-245 / 第 232-245 行

````yaml
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
        working-directory: pytorch
      - name: Enable long paths on Windows and install VS2022 17.13.2
        env:
          VC_YEAR: 2022
          VC_VERSION: 17.13.2
        shell: bash
        working-directory: pytorch
        run: |
          powershell .github/scripts/windows/install_vs2022.ps1
      - name: Build Triton wheel
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 246-259 / 第 246-259 行

````yaml
        env:
          IS_RELEASE_TAG: ${{ startsWith(github.event.ref, 'refs/tags/v') }}
        working-directory: pytorch
        shell: bash
        run: |
          set -x
          export RELEASE=""
          if [[ "${IS_RELEASE_TAG}" == true ]]; then
            export RELEASE="--release"
          fi
          .github/scripts/windows/build_triton.bat
          mkdir -p "${RUNNER_TEMP}/artifacts/"
          mv ./*.whl "${RUNNER_TEMP}/artifacts/"
      - uses: actions/upload-artifact@50769540e7f4bd5e21e526ee35c689e35e0d6874 # v4.4.0
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 260-276 / 第 260-276 行

````yaml
        with:
          name: triton-wheel-${{ matrix.py_vers }}-${{ matrix.device }}
          if-no-files-found: error
          path: ${{ runner.temp }}/artifacts/*


  upload-wheel:
    runs-on: ubuntu-22.04
    needs:
      - build-wheel
      - build-wheel-win
    permissions:
      id-token: write
      contents: read
    container:
      image: continuumio/miniconda3:4.12.0
    environment: ${{ (github.event_name == 'push' && github.event.ref == 'refs/heads/main') && 'nightly-wheel-upload' || '' }}
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 277-292 / 第 277-292 行

````yaml
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2

      - name: Configure AWS credentials(PyTorch account) for main
        if: ${{ github.event_name == 'push' && github.event.ref == 'refs/heads/main' }}
        uses: aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722 # v4.1.0
        with:
          role-to-assume: arn:aws:iam::749337293305:role/gha_workflow_nightly_build_wheels
          aws-region: us-east-1

      - name: Configure AWS credentials(PyTorch account) for RC builds
        if: ${{ github.event_name == 'push' &&  (startsWith(github.event.ref, 'refs/tags/') && !startsWith(github.event.ref, 'refs/tags/ciflow/')) }}
        uses: aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722 # v4.1.0
        with:
          role-to-assume: arn:aws:iam::749337293305:role/gha_workflow_test_build_wheels
          aws-region: us-east-1
````

- EN: This section reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`, `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 294-305 / 第 294-305 行

````yaml
      - name: Download Build Artifacts
        uses: actions/download-artifact@65a9edc5881444af0b9093a5e628f2fe47ea3b2e # v4.1.7
        with:
          # Download all available artifacts
          path: ${{ runner.temp }}/artifacts-all

      - name: Select Wheel Artifacts
        shell: bash
        run: |
          set -x
          mkdir -p "${RUNNER_TEMP}/artifacts/"
          mv "${RUNNER_TEMP}"/artifacts-all/triton-wheel-*/* "${RUNNER_TEMP}/artifacts/"
````

- EN: This section reuses actions and step building blocks such as `actions/download-artifact@65a9edc5881444af0b9093a5e628f2fe47ea3b2e`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 307-322 / 第 307-322 行

````yaml
      - name: Set DRY_RUN (only for tagged pushes)
        if: ${{ github.event_name == 'push' && (github.event.ref == 'refs/heads/main' || startsWith(github.event.ref, 'refs/tags/v')) }}
        shell: bash
        run: |
          echo "DRY_RUN=disabled" >> "$GITHUB_ENV"

      - name: Set UPLOAD_CHANNEL (only for tagged pushes)
        if: ${{ github.event_name == 'push' && startsWith(github.event.ref, 'refs/tags/v') }}
        shell: bash
        run: |
          set -ex

          # reference ends with an RC suffix
          if [[ "${GITHUB_REF_NAME}" = *-rc[0-9]* ]]; then
            echo "UPLOAD_CHANNEL=test" >> "$GITHUB_ENV"
          fi
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 324-339 / 第 324-339 行

````yaml
      # NB: This step is gated by DRY_RUN, which is enabled everywhere except main and release branches
      - name: Upload binaries
        env:
          PACKAGE_TYPE: wheel
          # The UPLOAD_SUBFOLDER needs to be empty here so that triton wheels are uploaded
          # to nightly or test
          UPLOAD_SUBFOLDER: ""
          PKG_DIR: ${{ runner.temp }}/artifacts
          R2_UPLOAD: ${{ (github.event_name == 'push' && github.event.ref == 'refs/heads/main') && 'true' || '' }}
          R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
          R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
          R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
        shell: bash
        run: |
          set -ex
          bash .ci/pytorch/binary_upload.sh
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。


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
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `pytorch/test-infra/.github/actions/setup-ssh@main`, `pytorch/pytorch/.github/actions/setup-linux@main`, `./.github/actions/ecr-login`, `pytorch/test-infra/.github/actions/pull-docker-image@main`, `pytorch/test-infra/.github/actions/teardown-linux@main`, `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`, `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`, ...
- Inline commands / 内联命令: `set`, `mkdir`, `tty`, `detach`, `v`, `w`, `"${DOCKER_IMAGE}"`, `3.10`, ...
- Environment variables / 环境变量: `NOTE`, `DOCKER_IMAGE`, `PY_VERS`, `BUILD_DEVICE`, `PLATFORM`, `SSH`, `GITHUB_TOKEN`, `ECR`, `IS_RELEASE_TAG`, `RUNNER_TEMP`, ...
- Named jobs or sections / 命名作业或章节: `on`, `push`, `branches`, `tags`, `paths`, `workflow_dispatch`, `pull_request`, `concurrency`, `jobs`, `get-label-type`, ...
