# _win-build.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/_win-build.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-13 / 第 1-13 行

````yaml
name: windows-build

on:
  workflow_call:
    inputs:
      build-environment:
        required: true
        type: string
        description: Top-level label for what's being built/tested.
      cuda-version:
        required: true
        type: string
        description: What CUDA version to build with, "cpu" for none.
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 14-27 / 第 14-27 行

````yaml
      use-xpu:
        required: false
        type: boolean
        default: false
        description: If set, build with XPU support.
      xpu-version:
        required: false
        type: string
        description: The version of XPU support package.
      vc-year:
        required: false
        type: string
        default: "2022"
        description: The Visual Studio year to use for building.
````

- EN: This section describes repository automation behavior for `.github/workflows/_win-build.yml`.
- CN: 该部分描述 `.github/workflows/_win-build.yml` 的仓库自动化行为。

### Lines 28-39 / 第 28-39 行

````yaml
      build-with-debug:
        required: false
        type: boolean
        default: false
        description: If set, build in debug mode.
      sync-tag:
        required: false
        type: string
        default: ""
        description: |
          If this is set, our linter will use this to make sure that every other
          job with the same `sync-tag` is identical.
````

- EN: This section describes repository automation behavior for `.github/workflows/_win-build.yml`.
- CN: 该部分描述 `.github/workflows/_win-build.yml` 的仓库自动化行为。

### Lines 40-54 / 第 40-54 行

````yaml
      test-matrix:
        required: false
        type: string
        description: |
          An option JSON description of what test configs to run later on. This
          is moved here from the Linux test workflow so that we can apply filter
          logic using test-config labels earlier and skip unnecessary builds
      runner:
        required: false
        type: string
        default: "windows.4xlarge.nonephemeral"
        description: |
          Label of the runner this job should run on.

    outputs:
````

- EN: This section describes repository automation behavior for `.github/workflows/_win-build.yml`.
- CN: 该部分描述 `.github/workflows/_win-build.yml` 的仓库自动化行为。

### Lines 55-70 / 第 55-70 行

````yaml
      test-matrix:
        value: ${{ jobs.build.outputs.test-matrix }}
        description: An optional JSON description of what test configs to run later on.
      build-environment:
        value: ${{ jobs.build.outputs.build-environment }}
        description: Top-level label for what's being built/tested.

env:
  GIT_DEFAULT_BRANCH: ${{ github.event.repository.default_branch }}

jobs:
  build:
    # Don't run on forked repos.
    if: github.repository_owner == 'pytorch'
    runs-on: ${{ inputs.runner }}
    timeout-minutes: 240
````

- EN: This section lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 71-87 / 第 71-87 行

````yaml
    outputs:
      test-matrix: ${{ steps.filter.outputs.test-matrix }}
      build-environment: ${{ inputs.build-environment }}
    defaults:
      run:
        shell: bash
    steps:
      # Duplicated in win-test because this MUST go before a checkout
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true
          git config --global core.ignorecase false

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 88-103 / 第 88-103 行

````yaml
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false

      - name: Setup SSH (Click me for login details)
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
          instructions: |
            To forward remote desktop on your local machine ssh as follows:
              ssh -L 3389:localhost:3389 %%username%%@%%hostname%%
            And then change password using `passwd` command.

            To start build locally, change working folder to \actions-runner\_work\pytorch\pytorch,
            Activate miniconda and Visual Studio environment, by running:
              call C:\Jenkins\Miniconda3\Scripts\activate.bat C:\Jenkins\Miniconda3
              call "C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvarsall.bat" x64
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 105-119 / 第 105-119 行

````yaml
      # [see note: pytorch repo ref]
      - name: Checkout PyTorch
        uses: pytorch/pytorch/.github/actions/checkout-pytorch@main
        with:
          no-sudo: true

      - name: Setup Windows
        uses: ./.github/actions/setup-win
        with:
          cuda-version: ${{ inputs.cuda-version }}

      - name: Parse ref
        id: parse-ref
        shell: bash
        run: python3 .github/scripts/parse_ref.py
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `./.github/actions/setup-win`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 121-135 / 第 121-135 行

````yaml
      - name: Get workflow job id
        id: get-job-id
        uses: ./.github/actions/get-workflow-job-id
        if: always()
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}

      # Apply the filter logic to the build step too if the test-config label is already there
      - name: Select all requested test configurations (if the test matrix is available)
        id: filter
        uses: ./.github/actions/filter-test-configs
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          test-matrix: ${{ inputs.test-matrix }}
          job-name: ${{ steps.get-job-id.outputs.job-name }}
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/get-workflow-job-id`, `./.github/actions/filter-test-configs`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 137-147 / 第 137-147 行

````yaml
      - name: Download pytest cache
        uses: ./.github/actions/pytest-cache-download
        continue-on-error: true
        with:
          cache_dir: .pytest_cache
          job_identifier: ${{ github.workflow }}_${{ inputs.build-environment }}

      - name: Build
        if: steps.filter.outputs.is-test-matrix-empty == 'False' || inputs.test-matrix == ''
        id: build
        shell: bash
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/pytest-cache-download`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 148-165 / 第 148-165 行

````yaml
        env:
          PYTORCH_FINAL_PACKAGE_DIR: /c/${{ github.run_id }}/build-results/
          BRANCH: ${{ steps.parse-ref.outputs.branch }}
          BUILD_ENVIRONMENT: ${{ inputs.build-environment }}
          BUILD_WHEEL: 1
          MAX_JOBS: 8
          CUDA_VERSION: ${{ inputs.cuda-version }}
          PYTHON_VERSION: "3.10"
          SCCACHE_BUCKET: "ossci-compiler-cache"
          SCCACHE_S3_KEY_PREFIX: ${{ github.workflow }}
          SCCACHE_REGION: us-east-1
          VC_PRODUCT: "BuildTools"
          VC_VERSION: ""
          VC_YEAR: "${{ inputs.vc-year }}"
          ALPINE_IMAGE: "308535385114.dkr.ecr.us-east-1.amazonaws.com/tool/alpine"
          AWS_DEFAULT_REGION: us-east-1
          PR_NUMBER: ${{ github.event.pull_request.number }}
          SHA1: ${{ github.event.pull_request.head.sha || github.sha }}
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 166-180 / 第 166-180 行

````yaml
          DEBUG: ${{ inputs.build-with-debug && '1' || '0' }}
          TORCH_CUDA_ARCH_LIST: "8.6"
          USE_CUDA: ${{ inputs.cuda-version != 'cpu' && '1' || '0' }}
          USE_XPU: ${{ inputs.use-xpu == true && '1' || '0' }}
          XPU_VERSION: "${{ inputs.xpu-version }}"
          OUR_GITHUB_JOB_ID: ${{ steps.get-job-id.outputs.job-id }}
        run: |
          .ci/pytorch/win-build.sh

      # Collect Windows torch libs and CUDA libs for cross-compilation
      - name: Collect Windows CUDA libs for cross-compilation
        if: steps.build.outcome != 'skipped' && inputs.cuda-version != 'cpu'
        shell: bash
        run: |
          set -ex
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 182-194 / 第 182-194 行

````yaml
          # Create directory structure if does not exist
          mkdir -p /c/${{ github.run_id }}/build-results

          # Copy CUDA libs
          CUDA_PATH="/c/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v${{ inputs.cuda-version }}"

          if [ -f "${CUDA_PATH}/lib/x64/cuda.lib" ]; then
            cp "${CUDA_PATH}/lib/x64/cuda.lib" /c/${{ github.run_id }}/build-results/
          fi

          if [ -f "${CUDA_PATH}/lib/x64/cudart.lib" ]; then
            cp "${CUDA_PATH}/lib/x64/cudart.lib" /c/${{ github.run_id }}/build-results/
          fi
````

- EN: This section describes repository automation behavior for `.github/workflows/_win-build.yml`.
- CN: 该部分描述 `.github/workflows/_win-build.yml` 的仓库自动化行为。

### Lines 196-210 / 第 196-210 行

````yaml
          # Copy CUDA runtime DLL (needed for MinGW import lib generation on CUDA 13.0+)
          echo "Searching for CUDA runtime DLLs in ${CUDA_PATH}/bin/:"
          ls -la "${CUDA_PATH}"/bin/cudart*.dll 2>/dev/null || echo "No cudart*.dll found in ${CUDA_PATH}/bin/"
          for dll in "${CUDA_PATH}"/bin/cudart64_*.dll "${CUDA_PATH}"/bin/cudart.dll; do
            if [ -f "$dll" ]; then
              cp "$dll" /c/${{ github.run_id }}/build-results/
              echo "Copied $(basename $dll)"
            fi
          done

          # List collected files
          echo "Collected CUDA libs and DLLs:"
          ls -lah /c/${{ github.run_id }}/build-results/*.lib /c/${{ github.run_id }}/build-results/*.dll 2>/dev/null || true

      # Upload to github so that people can click and download artifacts
````

- EN: This section describes repository automation behavior for `.github/workflows/_win-build.yml`.
- CN: 该部分描述 `.github/workflows/_win-build.yml` 的仓库自动化行为。

### Lines 211-224 / 第 211-224 行

````yaml
      - name: Upload artifacts to s3
        if: steps.build.outcome != 'skipped'
        uses: seemethere/upload-artifact-s3@baba72d0712b404f646cebe0730933554ebce96a # v5.1.0
        with:
          retention-days: 14
          if-no-files-found: error
          name: ${{ inputs.build-environment }}
          path: C:\${{ github.run_id }}\build-results

      - name: Upload sccache stats
        if: steps.build.outcome != 'skipped'
        uses: ./.github/actions/upload-sccache-stats
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `seemethere/upload-artifact-s3@baba72d0712b404f646cebe0730933554ebce96a`, `./.github/actions/upload-sccache-stats`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 226-229 / 第 226-229 行

````yaml
      - name: Teardown Windows
        uses: ./.github/actions/teardown-win
        if: always()
        timeout-minutes: 120
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/teardown-win`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 230-231 / 第 230-231 行

````yaml
        with:
          extra-delete-dir: /c/${{ github.run_id }}/build-results/
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
- Reusable actions / 复用 Action: `pytorch/test-infra/.github/actions/setup-ssh@main`, `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `./.github/actions/setup-win`, `./.github/actions/get-workflow-job-id`, `./.github/actions/filter-test-configs`, `./.github/actions/pytest-cache-download`, `seemethere/upload-artifact-s3@baba72d0712b404f646cebe0730933554ebce96a`, `./.github/actions/upload-sccache-stats`, ...
- Inline commands / 内联命令: `If`, `job`, `An`, `is`, `logic`, `Label`, `git`, `To`, ...
- Environment variables / 环境变量: `CUDA`, `XPU`, `JSON`, `GIT_DEFAULT_BRANCH`, `MUST`, `GHA`, `SSH`, `GITHUB_TOKEN`, `PYTORCH_FINAL_PACKAGE_DIR`, `BRANCH`, ...
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `inputs`, `outputs`, `env`, `jobs`, `build`, `defaults`, `steps`
