# _binary-build-flash-attention-wheel-windows.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/_binary-build-flash-attention-wheel-windows.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-13 / 第 1-13 行

````yaml
name: Build Flash Attention 3 wheels (Windows)

on:
  workflow_dispatch:
    inputs:
      test:
        description: 'Build test wheels with date suffix in version'
        required: false
        default: true
        type: boolean
  schedule:
    - cron: '0 0 1,15 * *'
  pull_request:
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 14-25 / 第 14-25 行

````yaml
    paths:
      - .github/workflows/_binary-build-flash-attention-wheel-windows.yml

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.sha }}-${{ github.event_name == 'workflow_dispatch' }}
  cancel-in-progress: true

jobs:
  get-label-type:
    if: github.repository_owner == 'pytorch'
    name: get-label-type
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程。

### Lines 26-38 / 第 26-38 行

````yaml
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}

  build-wheel:
    name: "Build FA3 Windows ${{ matrix.cuda_version }}"
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}${{ matrix.runner }}"
    strategy:
      fail-fast: false
      matrix:
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 39-51 / 第 39-51 行

````yaml
        include:
          - cuda_version: "12.8.1"
            cuda_short: "128"
            torch_cuda_arch_list: "8.0;8.6;9.0"
            runner: "windows.12xlarge"
            # python 3.10 because 3.9 is not supported by torch >= 2.9; FA3 is ABI stable after 2.9
            python_version: "3.10"
            pytorch_version: "2.10.0"
            pytorch_min_version: "2.9.0"
            einops_version: "0.8.2"
            ninja_version: "1.13.0"
            numpy_version: "2.2.6"
    timeout-minutes: 1440
````

- EN: This section describes repository automation behavior for `.github/workflows/_binary-build-flash-attention-wheel-windows.yml`.
- CN: 该部分描述 `.github/workflows/_binary-build-flash-attention-wheel-windows.yml` 的仓库自动化行为。

### Lines 52-65 / 第 52-65 行

````yaml
    defaults:
      run:
        shell: bash
    env:
      CUDA_VERSION: ${{ matrix.cuda_short }}
      TORCH_CUDA_ARCH_LIST: ${{ matrix.torch_cuda_arch_list }}
      PYTHON_VERSION: ${{ matrix.python_version }}
      PYTORCH_VERSION: ${{ matrix.pytorch_version }}
      PYTORCH_MIN_VERSION: ${{ matrix.pytorch_min_version }}
      EINOPS_VERSION: ${{ matrix.einops_version }}
      NINJA_VERSION: ${{ matrix.ninja_version }}
      NUMPY_VERSION: ${{ matrix.numpy_version }}
      VC_YEAR: "2022"
      DISTUTILS_USE_SDK: 1
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 66-76 / 第 66-76 行

````yaml
    steps:
      - name: Enable git long paths and symlinks on Windows
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true
          git config --global core.ignorecase false
          git config --global core.fsmonitor false
      - name: Setup SSH (Click me for login details)
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 78-91 / 第 78-91 行

````yaml
      - name: Checkout PyTorch
        uses: pytorch/pytorch/.github/actions/checkout-pytorch@main
        with:
          no-sudo: true
          submodules: true

      - name: Set CUDA environment
        run: |
          CUDA_VER="${{ matrix.cuda_version }}"
          CUDA_VER_SHORT="${CUDA_VER%.*}"
          CUDA_PATH="/c/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v${CUDA_VER_SHORT}"
          echo "Using CUDA ${CUDA_VER_SHORT} at ${CUDA_PATH}"
          echo "CUDA_HOME=${CUDA_PATH}" >> "${GITHUB_ENV}"
          echo "${CUDA_PATH}/bin" >> "${GITHUB_PATH}"
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/checkout-pytorch@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 92-102 / 第 92-102 行

````yaml
      - name: Setup MSVC
        uses: ilammy/msvc-dev-cmd@0b201ec74fa43914dc39ae48a89fd1d8cb592756 # v1

      - name: Remove link.exe conflict
        run: rm -f /usr/bin/link

      - name: Setup Windows
        uses: ./.github/actions/setup-win
        with:
          cuda-version: ${{ matrix.cuda_version }}
          python-version: ${{ env.PYTHON_VERSION }}
````

- EN: This section reuses actions and step building blocks such as `ilammy/msvc-dev-cmd@0b201ec74fa43914dc39ae48a89fd1d8cb592756`, `./.github/actions/setup-win`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 104-113 / 第 104-113 行

````yaml
      - name: Apply Flash Attention patches for Windows
        run: |
          cd third_party/flash-attention/hopper
          sed -i 's/bare_metal_version != Version("12.8")/& and not is_offline_build()/' setup.py
          sed -i '/flags.append(f'"'"'ldflags = /i\    ldflags.remove("/LTCG")' setup.py

          sed -i "s/python_requires=\">=3.8\"/python_requires=\">=${PYTHON_VERSION}\"/" setup.py
          sed -i "s/\"torch\",/\"torch>=${PYTORCH_MIN_VERSION}\",/" setup.py

      - name: Build Flash Attention 3 wheel
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 114-127 / 第 114-127 行

````yaml
        env:
          FA_FINAL_PACKAGE_DIR: ${{ runner.temp }}/artifacts
          PYTORCH_ROOT: ${{ github.workspace }}
          FLASH_ATTENTION_FORCE_BUILD: "TRUE"
          FLASH_ATTENTION_OFFLINE_BUILD: "TRUE"
          NVCC_THREADS: "1"
          MAX_JOBS: "4"
          FLASH_ATTENTION_DISABLE_SOFTCAP: "TRUE"
        run: |
          set -x
          mkdir -p "${FA_FINAL_PACKAGE_DIR}"
          nvcc --version
          python -m pip install torch==${PYTORCH_VERSION} --index-url "https://download.pytorch.org/whl/cu${CUDA_VERSION}"
          python -m pip install einops==${EINOPS_VERSION} ninja==${NINJA_VERSION} numpy==${NUMPY_VERSION}
````

- EN: This section lays out job topology or execution stages; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分铺排作业拓扑或执行阶段；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 128-140 / 第 128-140 行

````yaml
          if [[ "${{ inputs.test }}" == "true" ]]; then
            BUILD_DATE=$(date +%Y%m%d)
            export FLASH_ATTN_LOCAL_VERSION="${BUILD_DATE}.cu${CUDA_VERSION}"
          fi
          cd third_party/flash-attention/hopper
          python setup.py bdist_wheel -d "${FA_FINAL_PACKAGE_DIR}" -k --plat-name win_amd64
          echo "Wheel built:"
          ls -la "${FA_FINAL_PACKAGE_DIR}"/*.whl
      - uses: actions/upload-artifact@50769540e7f4bd5e21e526ee35c689e35e0d6874 # v4.4.0
        with:
          name: flash-attn-3-wheel-cu${{ matrix.cuda_short }}-windows_amd64
          if-no-files-found: error
          path: ${{ runner.temp }}/artifacts/*.whl
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 141-144 / 第 141-144 行

````yaml
      - name: Teardown Windows
        uses: ./.github/actions/teardown-win
        if: always()
        timeout-minutes: 120
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/teardown-win`.
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
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `pytorch/test-infra/.github/actions/setup-ssh@main`, `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `ilammy/msvc-dev-cmd@0b201ec74fa43914dc39ae48a89fd1d8cb592756`, `./.github/actions/setup-win`, `./.github/actions/teardown-win`
- Inline commands / 内联命令: `git`, `echo`, `cd`, `sed`, `set`, `mkdir`, `nvcc`, `python`, ...
- Environment variables / 环境变量: `FA3`, `ABI`, `CUDA_VERSION`, `TORCH_CUDA_ARCH_LIST`, `PYTHON_VERSION`, `PYTORCH_VERSION`, `PYTORCH_MIN_VERSION`, `EINOPS_VERSION`, `NINJA_VERSION`, `NUMPY_VERSION`, ...
- Named jobs or sections / 命名作业或章节: `on`, `workflow_dispatch`, `inputs`, `schedule`, `pull_request`, `paths`, `concurrency`, `jobs`, `get-label-type`, `with`, ...
