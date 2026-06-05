# _mac-test.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/_mac-test.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-13 / 第 1-13 行

````yaml
name: mac-test

on:
  workflow_call:
    inputs:
      build-environment:
        required: true
        type: string
        description: Top-level label for what's being built/tested.
      test-matrix:
        required: true
        type: string
        description: JSON description of what test configs to run.
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 14-26 / 第 14-26 行

````yaml
      sync-tag:
        required: false
        type: string
        default: ""
        description: |
          If this is set, our linter will use this to make sure that every other
          job with the same `sync-tag` is identical.
      python-version:
        required: false
        type: string
        default: "3.12"
        description: |
          The python version to be used. Will be 3.9 by default
````

- EN: This section describes repository automation behavior for `.github/workflows/_mac-test.yml`.
- CN: 该部分描述 `.github/workflows/_mac-test.yml` 的仓库自动化行为。

### Lines 27-40 / 第 27-40 行

````yaml
      timeout-minutes:
        required: false
        type: number
        default: 270
        description: |
          Set the maximum (in minutes) how long the workflow should take to finish
      disable-monitor:
        description: |
          [Experimental] Disable utilization monitoring for tests.
          Currently, by default we disable the monitor job and only look for specific tests,
          since we are investigating the behaviour of the monitor script with different tests.
        required: false
        type: boolean
        default: true
````

- EN: This section describes repository automation behavior for `.github/workflows/_mac-test.yml`.
- CN: 该部分描述 `.github/workflows/_mac-test.yml` 的仓库自动化行为。

### Lines 41-57 / 第 41-57 行

````yaml
      monitor-log-interval:
        description: |
          Set the interval for the monitor script to log utilization.
        required: false
        type: number
        default: 5
      monitor-data-collect-interval:
        description: |
          Set the interval for the monitor script to collect data.
        required: false
        type: number
        default: 1
    secrets:
      HUGGING_FACE_HUB_TOKEN:
        required: false
        description: |
          HF Auth token to avoid rate limits when downloading models or datasets from hub
````

- EN: This section describes repository automation behavior for `.github/workflows/_mac-test.yml`.
- CN: 该部分描述 `.github/workflows/_mac-test.yml` 的仓库自动化行为。

### Lines 59-70 / 第 59-70 行

````yaml
jobs:
  test:
    # Don't run on forked repos or empty test matrix
    if: github.repository_owner == 'pytorch' && toJSON(fromJSON(inputs.test-matrix).include) != '[]'
    defaults:
      run:
        shell: bash -e -l {0}
    strategy:
      matrix: ${{ fromJSON(inputs.test-matrix) }}
      fail-fast: false
    runs-on: ${{ matrix.runner }}
    timeout-minutes: ${{ matrix.mem_leak_check == 'mem_leak_check' && 600 || inputs.timeout-minutes }}
````

- EN: This section lays out job topology or execution stages; embeds shell commands that perform the actual side effects.
- CN: 该部分铺排作业拓扑或执行阶段；内嵌执行实际副作用的 shell 命令。

### Lines 71-81 / 第 71-81 行

````yaml
    env:
      GIT_DEFAULT_BRANCH: ${{ github.event.repository.default_branch }}
      BUILD_ENVIRONMENT: ${{ inputs.build-environment }}
      TEST_CONFIG: ${{ matrix.config }}
      SHARD_NUMBER: ${{ matrix.shard }}
      NUM_TEST_SHARDS: ${{ matrix.num_shards }}
      PR_BODY: ${{ github.event.pull_request.body }}
    steps:
      - name: Print runner OS/HW info
        run: |
          sysctl machdep.cpu.brand_string kern.osproductversion
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 83-97 / 第 83-97 行

````yaml
      - name: Clean up leftover processes on MacOS pet runner
        continue-on-error: true
        run: |
          for PROCESS in "python" "conda" "ninja" "clang"; do
            echo "Cleaning up all remaining ${PROCESS} process"
            pkill "${PROCESS}" || true
          done

      - name: Clean up brew miniconda, if installed
        continue-on-error: true
        run: |
          if brew list miniconda; then
            brew uninstall miniconda
            echo "REINSTALL_BREW_MINICONDA=1" >> "${GITHUB_ENV}"
          fi
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 99-112 / 第 99-112 行

````yaml
      - name: Clean up leftover local python3 site-packages on MacOS pet runner
        continue-on-error: true
        run: |
          for dir in  ~/.local/lib/python3.*/site-packages; do
            echo "Cleaning up ${dir}"
            rm -rf "${dir}"
          done

      - name: Clean up disk space before running MacOS workflow
        uses: pytorch/test-infra/.github/actions/check-disk-space@main

      # [see note: pytorch repo ref]
      - name: Checkout PyTorch
        uses: pytorch/pytorch/.github/actions/checkout-pytorch@main
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/check-disk-space@main`, `pytorch/pytorch/.github/actions/checkout-pytorch@main`; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 114-130 / 第 114-130 行

````yaml
      - name: Get workflow job id
        id: get-job-id
        uses: ./.github/actions/get-workflow-job-id
        if: always()
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}

      - name: Setup Python
        uses: pytorch/test-infra/.github/actions/setup-python@main
        with:
          python-version: ${{ inputs.python-version }}
          pip-requirements-file: .ci/docker/requirements-ci.txt

      - name: Start monitoring script
        id: monitor-script
        if: ${{ !inputs.disable-monitor }}
        continue-on-error: true
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/get-workflow-job-id`, `pytorch/test-infra/.github/actions/setup-python@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 131-147 / 第 131-147 行

````yaml
        env:
          JOB_ID: ${{ steps.get-job-id.outputs.job-id }}
          JOB_NAME: ${{ steps.get-job-id.outputs.job-name }}
          WORKFLOW_NAME: ${{ github.workflow }}
          WORKFLOW_RUN_ID: ${{github.run_id}}
          MONITOR_LOG_INTERVAL: ${{ inputs.monitor-log-interval }}
          MONITOR_DATA_COLLECT_INTERVAL: ${{ inputs.monitor-data-collect-interval }}
        run: |
          "$VENV_PATH/bin/python3" -m pip install psutil==5.9.8 dataclasses_json==0.6.7
          "$VENV_PATH/bin/python3" -m tools.stats.monitor --log-interval "$MONITOR_LOG_INTERVAL" --data-collect-interval "$MONITOR_DATA_COLLECT_INTERVAL" > usage_log.txt 2>&1 &
          echo "monitor-script-pid=${!}" >> "${GITHUB_OUTPUT}"

      - name: Download build artifacts
        uses: ./.github/actions/download-build-artifacts
        with:
          name: ${{ inputs.build-environment }}
          use-gha: true
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/download-build-artifacts`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 149-164 / 第 149-164 行

````yaml
      - name: Download TD artifacts
        continue-on-error: true
        uses: ./.github/actions/download-td-artifacts
        with:
          use-gha: true

      - name: Parse ref
        id: parse-ref
        run: .github/scripts/parse_ref.py

      - name: Check for keep-going label and re-enabled test issues
        # This uses the filter-test-configs action because it conveniently
        # checks for labels and re-enabled test issues.  It does not actually do
        # any filtering.  All filtering is done in the build step.
        id: keep-going
        uses: ./.github/actions/filter-test-configs
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/download-td-artifacts`, `./.github/actions/filter-test-configs`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 165-180 / 第 165-180 行

````yaml
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          test-matrix: ${{ inputs.test-matrix }}
          job-name: ${{ steps.get-job-id.outputs.job-name }}

      - name: Set Test step time
        id: test-timeout
        shell: bash
        env:
          JOB_TIMEOUT: ${{ matrix.mem_leak_check == 'mem_leak_check' && 600 || inputs.timeout-minutes }}
        run: |
          echo "timeout=$((JOB_TIMEOUT-30))" >> "${GITHUB_OUTPUT}"

      - name: Test
        id: test
        timeout-minutes: ${{ fromJson(steps.test-timeout.outputs.timeout) }}
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 181-198 / 第 181-198 行

````yaml
        env:
          PYTORCH_TEST_CUDA_MEM_LEAK_CHECK: ${{ matrix.mem_leak_check && '1' || '0' }}
          PYTORCH_TEST_RERUN_DISABLED_TESTS: ${{ matrix.rerun_disabled_tests && '1' || '0' }}
          CONTINUE_THROUGH_ERROR: ${{ steps.keep-going.outputs.keep-going }}
          VERBOSE_TEST_LOGS: ${{ steps.keep-going.outputs.ci-verbose-test-logs }}
          TEST_SHOWLOCALS: ${{ steps.keep-going.outputs.ci-test-showlocals }}
          NO_TEST_TIMEOUT: ${{ steps.keep-going.outputs.ci-no-test-timeout }}
          NO_TD: ${{ steps.keep-going.outputs.ci-no-td }}
          PIP_REQUIREMENTS_FILE: .github/requirements/pip-requirements-${{ runner.os }}.txt
          GITHUB_REPOSITORY: ${{ github.repository }}
          GITHUB_WORKFLOW: ${{ github.workflow }}
          GITHUB_JOB: ${{ github.job }}
          GITHUB_RUN_ID: ${{ github.run_id }}
          GITHUB_RUN_NUMBER: ${{ github.run_number }}
          GITHUB_RUN_ATTEMPT: ${{ github.run_attempt }}
          JOB_ID: ${{ steps.get-job-id.outputs.job-id }}
          JOB_NAME: ${{ steps.get-job-id.outputs.job-name }}
          REENABLED_ISSUES: ${{ steps.keep-going.outputs.reenabled-issues }}
````

- EN: This section controls token scope or environment variables for safe execution.
- CN: 该部分控制令牌权限范围或环境变量以实现安全执行。

### Lines 199-214 / 第 199-214 行

````yaml
          HUGGING_FACE_HUB_TOKEN: ${{ secrets.HUGGING_FACE_HUB_TOKEN }}
        run: |
          # shellcheck disable=SC1090
          set -ex

          # TODO: Remove me later, and properly activate venv
          PATH="$VENV_PATH/bin:$PATH"
          export PATH

          # Print out some information about the test environment
          for tool in python3 python; do
            which $tool
            $tool --version
          done

          python3 -mpip install --no-index --no-deps dist/*.whl
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 216-229 / 第 216-229 行

````yaml
          set +e
          pushd "${RUNNER_TEMP}"
          # Install pip dependencies if they are not found. This is to mitigate a peculiar
          # flaky missing dependencies on MacOS
          python3 -c "import torch"
          RC=$?
          popd

          if [ "${RC}" -ne 0 ]; then
            python3 -mpip install --ignore-installed -r "${PIP_REQUIREMENTS_FILE}"
          fi
          set -e

          .ci/pytorch/macos-test.sh
````

- EN: This section describes repository automation behavior for `.github/workflows/_mac-test.yml`.
- CN: 该部分描述 `.github/workflows/_mac-test.yml` 的仓库自动化行为。

### Lines 231-246 / 第 231-246 行

````yaml
      - name: Print remaining test logs
        shell: bash
        if: always() && steps.test.conclusion
        run: |
          cat test/**/*_toprint.log || true

      - name: Run OP benchmark
        shell: bash
        if: ${{ contains(steps.get-job-id.outputs.job-name, 'mps') }}
        run: |
          python3 test/bench_mps_ops.py


      - name: Stop monitoring script
        if: ${{ always() && steps.monitor-script.outputs.monitor-script-pid }}
        continue-on-error: true
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 247-260 / 第 247-260 行

````yaml
        env:
          MONITOR_SCRIPT_PID: ${{ steps.monitor-script.outputs.monitor-script-pid }}
        run: |
          kill "$MONITOR_SCRIPT_PID"

      - name: Upload test artifacts
        uses: ./.github/actions/upload-test-artifacts
        if: always() && steps.test.conclusion && steps.test.conclusion != 'skipped'
        with:
          use-gha: true
          file-suffix: ${{ github.job }}-${{ matrix.config }}-${{ matrix.shard }}-${{ matrix.num_shards }}-${{ matrix.runner }}_${{ steps.get-job-id.outputs.job-id }}

      - name: Upload the benchmark results
        uses: pytorch/test-infra/.github/actions/upload-benchmark-results@main
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/upload-test-artifacts`, `pytorch/test-infra/.github/actions/upload-benchmark-results@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 261-277 / 第 261-277 行

````yaml
        with:
          benchmark-results-dir: test/test-reports
          dry-run: false
          schema-version: v3
          github-token: ${{ secrets.GITHUB_TOKEN }}

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
          local_path: usage_log.txt
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/upload-utilization-stats`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 279-285 / 第 279-285 行

````yaml
      - name: Reinstall brew miniconda, if was installed
        if: always()
        continue-on-error: true
        run: |
          if [[ -n "$REINSTALL_BREW_MINICONDA" ]]; then
              brew install --cask miniconda
          fi
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 287-290 / 第 287-290 行

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
- Reusable actions / 复用 Action: `pytorch/test-infra/.github/actions/check-disk-space@main`, `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `./.github/actions/get-workflow-job-id`, `pytorch/test-infra/.github/actions/setup-python@main`, `./.github/actions/download-build-artifacts`, `./.github/actions/download-td-artifacts`, `./.github/actions/filter-test-configs`, `./.github/actions/upload-test-artifacts`, ...
- Inline commands / 内联命令: `If`, `job`, `The`, `Set`, `[Experimental]`, `Currently,`, `since`, `HF`, ...
- Environment variables / 环境变量: `JSON`, `HUGGING_FACE_HUB_TOKEN`, `GIT_DEFAULT_BRANCH`, `BUILD_ENVIRONMENT`, `TEST_CONFIG`, `SHARD_NUMBER`, `NUM_TEST_SHARDS`, `PR_BODY`, `PROCESS`, `REINSTALL_BREW_MINICONDA`, ...
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `inputs`, `secrets`, `jobs`, `test`, `defaults`, `strategy`, `env`, `steps`
