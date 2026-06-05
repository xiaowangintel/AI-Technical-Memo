# _win-test.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/_win-test.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-17 / 第 1-17 行

````yaml
name: win-test

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
      test-matrix:
        required: true
        type: string
        description: JSON description of what test configs to run.
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 18-30 / 第 18-30 行

````yaml
      sync-tag:
        required: false
        type: string
        default: ""
        description: |
          If this is set, our linter will use this to make sure that every other
          job with the same `sync-tag` is identical.
      timeout-minutes:
        required: false
        type: number
        default: 240
        description: |
          Set the maximum (in minutes) how long the workflow should take to finish
````

- EN: This section describes repository automation behavior for `.github/workflows/_win-test.yml`.
- CN: 该部分描述 `.github/workflows/_win-test.yml` 的仓库自动化行为。

### Lines 31-44 / 第 31-44 行

````yaml
      disable-monitor:
        description: |
          [Experimental] Disable utilization monitoring for tests.
          Currently, by default we disable the monitor job and only look for specific tests,
          since we are investigating the behaviour of the monitor script with different tests.
        required: false
        type: boolean
        default: true
      monitor-log-interval:
        description: |
          Set the interval for the monitor script to log utilization.
        required: false
        type: number
        default: 5
````

- EN: This section describes repository automation behavior for `.github/workflows/_win-test.yml`.
- CN: 该部分描述 `.github/workflows/_win-test.yml` 的仓库自动化行为。

### Lines 45-57 / 第 45-57 行

````yaml
      monitor-data-collect-interval:
        description: |
          Set the interval for the monitor script to collect data.
        required: false
        type: number
        default: 1
env:
  GIT_DEFAULT_BRANCH: ${{ github.event.repository.default_branch }}

jobs:
  test:
    # Don't run on forked repos or empty test matrix
    if: github.repository_owner == 'pytorch' && toJSON(fromJSON(inputs.test-matrix).include) != '[]'
````

- EN: This section lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 58-73 / 第 58-73 行

````yaml
    strategy:
      matrix: ${{ fromJSON(inputs.test-matrix) }}
      fail-fast: false
    runs-on: ${{ matrix.runner }}
    timeout-minutes: ${{ matrix.mem_leak_check == 'mem_leak_check' && 600 || inputs.timeout-minutes }}
    defaults:
      run:
        shell: bash
    steps:
      # Duplicated in win-build because this MUST go before a checkout
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true
          git config --global core.ignorecase false
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 75-89 / 第 75-89 行

````yaml
          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
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

            To start tests locally, change working folder to \actions-runner\_work\pytorch\pytorch\test,
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 90-105 / 第 90-105 行

````yaml
            Activate miniconda and Visual Studio environment and set PYTHON_PATH, by running:
              call C:\Jenkins\Miniconda3\Scripts\activate.bat C:\Jenkins\Miniconda3
              call "C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvarsall.bat" x64
              set PYTHONPATH=C:\actions-runner\_work\pytorch\pytorch\build\win_tmp\build

      # [see note: pytorch repo ref]
      - name: Checkout PyTorch
        uses: pytorch/pytorch/.github/actions/checkout-pytorch@main
        with:
          no-sudo: true
          submodules: false

      - name: Setup Windows
        uses: ./.github/actions/setup-win
        with:
          cuda-version: ${{ inputs.cuda-version }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `./.github/actions/setup-win`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 107-124 / 第 107-124 行

````yaml
      - name: Get workflow job id
        id: get-job-id
        uses: ./.github/actions/get-workflow-job-id
        if: always()
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}

      - name: Start monitoring script
        id: monitor-script
        env:
          JOB_ID: ${{ steps.get-job-id.outputs.job-id }}
          JOB_NAME: ${{ steps.get-job-id.outputs.job-name }}
          WORKFLOW_NAME: ${{ github.workflow }}
          WORKFLOW_RUN_ID: ${{github.run_id}}
          MONITOR_LOG_INTERVAL: ${{ inputs.monitor-log-interval }}
          MONITOR_DATA_COLLECT_INTERVAL: ${{ inputs.monitor-data-collect-interval }}
        shell: bash
        if: ${{ !inputs.disable-monitor }}
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/get-workflow-job-id`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 125-141 / 第 125-141 行

````yaml
        continue-on-error: true
        run: |
          # Windows conda doesn't have python3 binary, only python, but it's python3
          ${CONDA_RUN} python -m pip install psutil==5.9.8 dataclasses_json==0.6.7 nvidia-ml-py==11.525.84
          ${CONDA_RUN} python -m tools.stats.monitor --log-interval "$MONITOR_LOG_INTERVAL" --data-collect-interval "$MONITOR_DATA_COLLECT_INTERVAL" > usage_log.txt 2>&1 &
          echo "monitor-script-pid=${!}" >> "${GITHUB_OUTPUT}"

      - name: Download PyTorch Build Artifacts
        uses: seemethere/download-artifact-s3@1da556a7aa0a088e3153970611f6c432d58e80e6 # v4.2.0
        with:
          name: ${{ inputs.build-environment }}
          path: C:\${{ github.run_id }}\build-results

      - name: Check build-results folder
        shell: powershell
        run: |
          tree /F C:\$Env:GITHUB_RUN_ID\build-results
````

- EN: This section reuses actions and step building blocks such as `seemethere/download-artifact-s3@1da556a7aa0a088e3153970611f6c432d58e80e6`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 143-156 / 第 143-156 行

````yaml
      - name: Download TD artifacts
        continue-on-error: true
        uses: ./.github/actions/download-td-artifacts

      - name: Check for keep-going label and re-enabled test issues
        # This uses the filter-test-configs action because it conveniently
        # checks for labels and re-enabled test issues.  It does not actually do
        # any filtering.  All filtering is done in the build step.
        id: keep-going
        uses: ./.github/actions/filter-test-configs
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          test-matrix: ${{ inputs.test-matrix }}
          job-name: ${{ steps.get-job-id.outputs.job-name }}
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/download-td-artifacts`, `./.github/actions/filter-test-configs`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 158-169 / 第 158-169 行

````yaml
      - name: Set Test step time
        id: test-timeout
        shell: bash
        env:
          JOB_TIMEOUT: ${{ matrix.mem_leak_check == 'mem_leak_check' && 600 || inputs.timeout-minutes }}
        run: |
          echo "timeout=$((JOB_TIMEOUT-30))" >> "${GITHUB_OUTPUT}"

      - name: Test
        id: test
        shell: bash
        timeout-minutes: ${{ fromJson(steps.test-timeout.outputs.timeout) }}
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 170-187 / 第 170-187 行

````yaml
        env:
          USE_CUDA: ${{ inputs.cuda-version != 'cpu' && '1' || '0' }}
          INSTALL_WINDOWS_SDK: 1
          PYTHON_VERSION: "3.10"
          CONTINUE_THROUGH_ERROR: ${{ steps.keep-going.outputs.keep-going }}
          VERBOSE_TEST_LOGS: ${{ steps.keep-going.outputs.ci-verbose-test-logs }}
          TEST_SHOWLOCALS: ${{ steps.keep-going.outputs.ci-test-showlocals }}
          NO_TEST_TIMEOUT: ${{ steps.keep-going.outputs.ci-no-test-timeout }}
          NO_TD: ${{ steps.keep-going.outputs.ci-no-td }}
          VC_PRODUCT: "BuildTools"
          VC_VERSION: ""
          VS_VERSION: "17.4.1"
          VC_YEAR: "2022"
          AWS_DEFAULT_REGION: us-east-1
          PR_NUMBER: ${{ github.event.pull_request.number }}
          GITHUB_REPOSITORY: ${{ github.repository }}
          GITHUB_WORKFLOW: ${{ github.workflow }}
          GITHUB_JOB: ${{ github.job }}
````

- EN: This section declares workflow triggers or event bindings; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；控制令牌权限范围或环境变量以实现安全执行。

### Lines 188-205 / 第 188-205 行

````yaml
          GITHUB_RUN_ID: ${{ github.run_id }}
          GITHUB_RUN_NUMBER: ${{ github.run_number }}
          GITHUB_RUN_ATTEMPT: ${{ github.run_attempt }}
          JOB_ID: ${{ steps.get-job-id.outputs.job-id }}
          JOB_NAME: ${{ steps.get-job-id.outputs.job-name }}
          SHA1: ${{ github.event.pull_request.head.sha || github.sha }}
          CUDA_VERSION: ${{ inputs.cuda-version }}
          PYTORCH_FINAL_PACKAGE_DIR: /c/${{ github.run_id }}/build-results/
          BUILD_ENVIRONMENT: ${{ inputs.build-environment }}
          ALPINE_IMAGE: "308535385114.dkr.ecr.us-east-1.amazonaws.com/tool/alpine"
          SHARD_NUMBER: ${{ matrix.shard }}
          NUM_TEST_SHARDS: ${{ matrix.num_shards }}
          TEST_CONFIG: ${{ matrix.config }}
          REENABLED_ISSUES: ${{ github.event.pull_request.reenabled-issues }}
          TORCH_CUDA_ARCH_LIST: "8.6"
          PYTORCH_TEST_CUDA_MEM_LEAK_CHECK: ${{ matrix.mem_leak_check && '1' || '0' }}
          PYTORCH_TEST_RERUN_DISABLED_TESTS: ${{ matrix.rerun_disabled_tests && '1' || '0' }}
        run: |
````

- EN: This section declares workflow triggers or event bindings; embeds shell commands that perform the actual side effects.
- CN: 该部分声明工作流触发条件或事件绑定；内嵌执行实际副作用的 shell 命令。

### Lines 206-222 / 第 206-222 行

````yaml
          pushd "${PYTORCH_FINAL_PACKAGE_DIR}"
          # shellcheck disable=SC2046,SC2102
          python3 -mpip install $(echo *.whl)[opt-einsum,optree] optree==0.13.0
          popd

          .ci/pytorch/win-test.sh

      - name: Upload pytest cache if tests failed
        uses: ./.github/actions/pytest-cache-upload
        continue-on-error: true
        if: failure() && steps.test.conclusion && steps.test.conclusion == 'failure'
        with:
          cache_dir: .pytest_cache
          shard: ${{ matrix.shard }}
          sha: ${{ github.event.pull_request.head.sha || github.sha }}
          test_config: ${{ matrix.config }}
          job_identifier: ${{ github.workflow }}_${{ inputs.build-environment }}
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `./.github/actions/pytest-cache-upload`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 224-237 / 第 224-237 行

````yaml
      - name: Print remaining test logs
        shell: bash
        if: always() && steps.test.conclusion
        run: |
          cat test/**/*_toprint.log || true

      - name: Stop monitoring script
        if: ${{ always() && steps.monitor-script.outputs.monitor-script-pid }}
        shell: bash
        continue-on-error: true
        env:
          MONITOR_SCRIPT_PID: ${{ steps.monitor-script.outputs.monitor-script-pid }}
        run: |
          kill "$MONITOR_SCRIPT_PID"
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 239-254 / 第 239-254 行

````yaml
      - name: Upload test artifacts
        uses: ./.github/actions/upload-test-artifacts
        if: always() && steps.test.conclusion && steps.test.conclusion != 'skipped'
        with:
          file-suffix: ${{ github.job }}-${{ matrix.config }}-${{ matrix.shard }}-${{ matrix.num_shards }}-${{ matrix.runner }}_${{ steps.get-job-id.outputs.job-id }}

      - name: Upload utilization stats
        if: ${{ always() && steps.test.conclusion && steps.test.conclusion != 'skipped' && !inputs.disable-monitor }}
        continue-on-error: true
        uses: ./.github/actions/upload-utilization-stats
        with:
          job_id: ${{ steps.get-job-id.outputs.job-id }}
          job_name: ${{ steps.get-job-id.outputs.job-name }}
          workflow_name: ${{ github.workflow }}
          workflow_run_id: ${{github.run_id}}
          workflow_attempt: ${{github.run_attempt}}
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/upload-test-artifacts`, `./.github/actions/upload-utilization-stats`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 256-259 / 第 256-259 行

````yaml
      - name: Parse ref
        id: parse-ref
        shell: bash
        run: python3 .github/scripts/parse_ref.py
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 261-264 / 第 261-264 行

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
- Reusable actions / 复用 Action: `pytorch/test-infra/.github/actions/setup-ssh@main`, `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `./.github/actions/setup-win`, `./.github/actions/get-workflow-job-id`, `seemethere/download-artifact-s3@1da556a7aa0a088e3153970611f6c432d58e80e6`, `./.github/actions/download-td-artifacts`, `./.github/actions/filter-test-configs`, `./.github/actions/pytest-cache-upload`, ...
- Inline commands / 内联命令: `If`, `job`, `Set`, `[Experimental]`, `Currently,`, `since`, `git`, `To`, ...
- Environment variables / 环境变量: `CUDA`, `JSON`, `GIT_DEFAULT_BRANCH`, `MUST`, `GHA`, `SSH`, `GITHUB_TOKEN`, `PYTHON_PATH`, `PYTHONPATH`, `JOB_ID`, ...
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `inputs`, `env`, `jobs`, `test`, `strategy`, `defaults`, `steps`
