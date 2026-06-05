# _mac-build.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/_mac-build.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-17 / 第 1-17 行

````yaml
name: mac-build

on:
  workflow_call:
    inputs:
      build-environment:
        required: true
        type: string
        description: Top-level label for what's being built/tested.
      runner-type:
        required: true
        type: string
        description: Name of the GitHub-managed runner type to use for the build.
      build-generates-artifacts:
        required: true
        type: boolean
        description: If set, upload generated build artifacts.
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 18-29 / 第 18-29 行

````yaml
      xcode-version:
        required: false
        type: string
        default: ""
        description: What xcode version to build with.
      sync-tag:
        required: false
        type: string
        default: ""
        description: |
          If this is set, our linter will use this to make sure that every other
          job with the same `sync-tag` is identical.
````

- EN: This section describes repository automation behavior for `.github/workflows/_mac-build.yml`.
- CN: 该部分描述 `.github/workflows/_mac-build.yml` 的仓库自动化行为。

### Lines 30-42 / 第 30-42 行

````yaml
      python-version:
        required: false
        type: string
        default: "3.12"
        description: |
          The python version to be used. Will be 3.9 by default
      test-matrix:
        required: false
        type: string
        description: |
          An option JSON description of what test configs to run later on. This
          is moved here from the Linux test workflow so that we can apply filter
          logic using test-config labels earlier and skip unnecessary builds
````

- EN: This section describes repository automation behavior for `.github/workflows/_mac-build.yml`.
- CN: 该部分描述 `.github/workflows/_mac-build.yml` 的仓库自动化行为。

### Lines 43-58 / 第 43-58 行

````yaml
      sccache-use-gha:
        required: false
        type: boolean
        default: false
        description: If true, use the Github cache as the storage option for sccache instead of S3.

    outputs:
      test-matrix:
        value: ${{ jobs.build.outputs.test-matrix }}
        description: An optional JSON description of what test configs to run later on.
      build-outcome:
        value: ${{ jobs.build.outputs.build-outcome }}
        description: The outcome of the build step. This is used to influence test filtering logic later on.
      build-environment:
        value: ${{ jobs.build.outputs.build-environment }}
        description: Top-level label for what's being built/tested.
````

- EN: This section describes repository automation behavior for `.github/workflows/_mac-build.yml`.
- CN: 该部分描述 `.github/workflows/_mac-build.yml` 的仓库自动化行为。

### Lines 60-76 / 第 60-76 行

````yaml
jobs:
  build:
    # Don't run on forked repos.
    if: github.repository_owner == 'pytorch'
    runs-on: ${{ inputs.runner-type }}
    env:
      BUILD_ENVIRONMENT: ${{ inputs.build-environment }}
      SCCACHE_USE_GHA: ${{ inputs.sccache-use-gha }}  # this is placed here instead of the sccache step to appease actionlint
    outputs:
      build-outcome: ${{ steps.build.outcome }}
      test-matrix: ${{ steps.filter.outputs.test-matrix }}
      build-environment: ${{ inputs.build-environment }}
    steps:
      - name: Clean up disk space before running MacOS workflow
        uses: pytorch/test-infra/.github/actions/check-disk-space@main

      # [see note: pytorch repo ref]
````

- EN: This section lays out job topology or execution stages; reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/check-disk-space@main`; controls token scope or environment variables for safe execution.
- CN: 该部分铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 77-92 / 第 77-92 行

````yaml
      - name: Checkout PyTorch
        uses: pytorch/pytorch/.github/actions/checkout-pytorch@main

      - name: Set xcode version
        env:
          XCODE_VERSION: ${{ inputs.xcode-version }}
        run: |
          if [ -n "${XCODE_VERSION}" ]; then
            echo "DEVELOPER_DIR=/Applications/Xcode_${XCODE_VERSION}.app/Contents/Developer" >> "${GITHUB_ENV}"
          fi

      - name: Setup Python
        uses: pytorch/test-infra/.github/actions/setup-python@main
        with:
          python-version: ${{ inputs.python-version }}
          pip-requirements-file: .ci/docker/requirements-ci.txt
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `pytorch/test-infra/.github/actions/setup-python@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 94-106 / 第 94-106 行

````yaml
      - name: Install sccache (only for non-forked PRs, and pushes to trunk)
        uses: nick-fields/retry@7152eba30c6575329ac0576536151aca5a72780e # v3.0.0
        if: ${{ github.event_name == 'push' || github.event.pull_request.head.repo.full_name == github.repository }}
        with:
          timeout_minutes: 5
          max_attempts: 3
          retry_wait_seconds: 90
          command: |
            set -ex

            DOWNLOAD_SCCACHE=0
            SCCACHE_VERSION="0.4.1"
            LOCAL_PATH="/usr/local/bin"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `nick-fields/retry@7152eba30c6575329ac0576536151aca5a72780e`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 108-121 / 第 108-121 行

````yaml
            if [ ! -f "${LOCAL_PATH}/sccache" ]; then
              DOWNLOAD_SCCACHE=1
            else
              LOCAL_VERSION=$("${LOCAL_PATH}/sccache" --version | cut -d" " -f2)

              if [ "${LOCAL_VERSION}" != "${SCCACHE_VERSION}" ]; then
                DOWNLOAD_SCCACHE=1
              fi
            fi

            if [ "${DOWNLOAD_SCCACHE}" == "1" ]; then
              sudo curl --retry 3 --retry-all-errors "https://s3.amazonaws.com/ossci-macos/sccache/sccache-v0.4.1-${RUNNER_ARCH}" --output "${LOCAL_PATH}/sccache"
              sudo chmod +x "${LOCAL_PATH}/sccache"
            fi
````

- EN: This section describes repository automation behavior for `.github/workflows/_mac-build.yml`.
- CN: 该部分描述 `.github/workflows/_mac-build.yml` 的仓库自动化行为。

### Lines 123-135 / 第 123-135 行

````yaml
            if [[ "${SCCACHE_USE_GHA}" == "true" ]]; then
              echo "ACTIONS_CACHE_URL=${ACTIONS_CACHE_URL}" >> "${GITHUB_ENV}"
              echo "ACTIONS_RUNTIME_TOKEN=${ACTIONS_RUNTIME_TOKEN}" >> "${GITHUB_ENV}"
              echo "SCCACHE_GHA_ENABLED=on" >> "${GITHUB_ENV}"
            else
              # The runner has access to the S3 bucket via IAM profile without the need
              # for any credential
              echo "SCCACHE_BUCKET=ossci-compiler-cache-circleci-v2" >> "${GITHUB_ENV}"
              echo "SCCACHE_S3_KEY_PREFIX=${GITHUB_WORKFLOW}" >> "${GITHUB_ENV}"
            fi

            # This is needed so that later build script could find sccache (which sccache)
            echo "${LOCAL_PATH}" >> $GITHUB_PATH
````

- EN: This section describes repository automation behavior for `.github/workflows/_mac-build.yml`.
- CN: 该部分描述 `.github/workflows/_mac-build.yml` 的仓库自动化行为。

### Lines 137-151 / 第 137-151 行

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

### Lines 153-166 / 第 153-166 行

````yaml
      - name: Build
        if: steps.filter.outputs.is-test-matrix-empty == 'False' || inputs.test-matrix == ''
        id: build
        env:
          OUR_GITHUB_JOB_ID: ${{ steps.get-job-id.outputs.job-id }}
        run: |
          # TODO: Remove me later, and properly activate venv
          PATH="$VENV_PATH/bin:$PATH"
          export PATH

          # NB: Same trick as Linux, there is no need to initialize sccache with the risk of getting
          # it hangs or timeout at initialization. The cache will be started automatically
          export SKIP_SCCACHE_INITIALIZATION=1
          .ci/pytorch/macos-build.sh
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 168-183 / 第 168-183 行

````yaml
      - name: Archive artifacts into zip
        if: inputs.build-generates-artifacts && steps.build.outcome != 'skipped'
        run: |
          zip -1 -r artifacts.zip dist/ build/.ninja_log build/compile_commands.json .additional_ci_files

      - name: Store PyTorch Build Artifacts on GHA
        uses: actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 # v4.6.2
        if: inputs.build-generates-artifacts && steps.build.outcome != 'skipped'
        with:
          name: ${{ env.BUILD_ENVIRONMENT }}
          retention-days: 14
          if-no-files-found: error
          path: artifacts.zip

      - name: Upload sccache stats to GHA
        uses: actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 # v4.6.2
````

- EN: This section reuses actions and step building blocks such as `actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02`, `actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 184-190 / 第 184-190 行

````yaml
        # Only if sccache is installed, see above
        if: ${{ (github.event_name == 'push' || github.event.pull_request.head.repo.full_name == github.repository) && steps.build.outcome != 'skipped' }}
        with:
          name: sccache-stats-${{ inputs.build-environment }}-runattempt${{ github.run_attempt }}-${{ steps.get-job-id.outputs.job-id }}
          retention-days: 14
          if-no-files-found: warn
          path: sccache-stats-*.json
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 192-195 / 第 192-195 行

````yaml
      - name: Clean up disk space
        if: always()
        continue-on-error: true
        uses: pytorch/test-infra/.github/actions/check-disk-space@main
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/check-disk-space@main`.
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
- Reusable actions / 复用 Action: `pytorch/test-infra/.github/actions/check-disk-space@main`, `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `pytorch/test-infra/.github/actions/setup-python@main`, `nick-fields/retry@7152eba30c6575329ac0576536151aca5a72780e`, `./.github/actions/get-workflow-job-id`, `./.github/actions/filter-test-configs`, `actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02`
- Inline commands / 内联命令: `If`, `job`, `The`, `An`, `is`, `logic`, `echo`, `set`, ...
- Environment variables / 环境变量: `JSON`, `BUILD_ENVIRONMENT`, `SCCACHE_USE_GHA`, `XCODE_VERSION`, `DEVELOPER_DIR`, `GITHUB_ENV`, `DOWNLOAD_SCCACHE`, `SCCACHE_VERSION`, `LOCAL_PATH`, `LOCAL_VERSION`, ...
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `inputs`, `outputs`, `jobs`, `build`, `env`, `steps`
