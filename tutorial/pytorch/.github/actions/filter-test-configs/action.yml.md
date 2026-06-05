# action.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/actions/filter-test-configs/action.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a reusable composite GitHub Action, including declared inputs, step composition, and shell execution.
- **用途 (CN)**: 定义可复用的复合型 GitHub Action，包括输入声明、步骤组合以及 shell 执行逻辑。

## Content Analysis / 内容分析
### Lines 1-15 / 第 1-15 行

````yaml
name: Filter test configs matrix

description: |
  Apply filter to the test configs matrix to keep only entries specified
  by the PR test-config labels. If no test-config label is set, the same
  test configs matrix is returned untouched.

inputs:
  github-token:
    description: GITHUB_TOKEN
    required: true
  test-matrix:
    required: true
    type: string
    description: JSON description of what test configs to run.
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 16-31 / 第 16-31 行

````yaml
  selected-test-configs:
    required: false
    type: string
    description: |
      A comma-separated list of test configurations from the test matrix to keep,
      The empty list means we are going to keep every configurations by defaults
    default: ""
  job-name:
    type: string
    required: false
    default: ""

outputs:
  test-matrix:
    description: The filtered test configs matrix.
    value: ${{ steps.filter.outputs.test-matrix }}
````

- EN: This section describes repository automation behavior for `.github/actions/filter-test-configs/action.yml`.
- CN: 该部分描述 `.github/actions/filter-test-configs/action.yml` 的仓库自动化行为。

### Lines 32-46 / 第 32-46 行

````yaml
  is-test-matrix-empty:
    description: True if the filtered test configs matrix is empty. False otherwise.
    value: ${{ steps.filter.outputs.is-test-matrix-empty }}
  keep-going:
    description: True if keep-going label was on PR or [keep-going] in PR body.
    value: ${{ steps.filter.outputs.keep-going }}
  reenabled-issues:
    description: Comma separated list of issue numbers that should correspond to disable test issues that the PR fixes
    value: ${{ steps.filter.outputs.reenabled-issues }}
  ci-verbose-test-logs:
    description: True if ci-verbose-test-logs label was on PR or [ci-verbose-test-logs] in PR body.
    value: ${{ steps.filter.outputs.ci-verbose-test-logs }}
  ci-test-showlocals:
    description: True if ci-test-showlocals label was on PR or [ci-test-showlocals] in PR body.
    value: ${{ steps.filter.outputs.ci-test-showlocals }}
````

- EN: This section describes repository automation behavior for `.github/actions/filter-test-configs/action.yml`.
- CN: 该部分描述 `.github/actions/filter-test-configs/action.yml` 的仓库自动化行为。

### Lines 47-62 / 第 47-62 行

````yaml
  ci-no-test-timeout:
    description: True if ci-no-test-timeout label was on PR or [ci-no-test-timeout] in PR body.
    value: ${{ steps.filter.outputs.ci-no-test-timeout }}
  ci-no-td:
    description: True if ci-no-td label was on PR or [ci-no-td] in PR body.
    value: ${{ steps.filter.outputs.ci-no-td }}
  ci-td-distributed:
    description: True if ci-td-distributed label was on PR or [ci-td-distributed] in PR body.
    value: ${{ steps.filter.outputs.ci-td-distributed }}
  labels:
    description: The list of labels from the PR
    value: ${{ steps.filter.outputs.labels }}

runs:
  using: composite
  steps:
````

- EN: This section reuses actions and step building blocks such as named actions.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 63-79 / 第 63-79 行

````yaml
    - name: Setup uv
      if: ${{ !env.UV_PYTHON }}
      uses: pytorch/test-infra/.github/actions/setup-uv@main
      with:
        python-version: "3.12"

    - uses: nick-fields/retry@v3.0.0
      name: Setup dependencies
      env:
        GITHUB_TOKEN: ${{ inputs.github-token }}
      with:
        shell: bash
        timeout_minutes: 10
        max_attempts: 5
        retry_wait_seconds: 30
        command: |
          set -eux
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-uv@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 80-91 / 第 80-91 行

````yaml
          # Pre-fetch dependencies to cache them for later uv run calls
          uv run --no-project --with requests==2.27.1 --with pyyaml==6.0.2 python -c "import requests, yaml; print('Dependencies ready')"

    - name: Parse ref
      id: parse-ref
      shell: bash
      run: |
        set -x

        # Use relative path here as this could be checked out anywhere, not necessarily
        # in runner workspace
        uv run --no-project --with requests==2.27.1 --with pyyaml==6.0.2 python "${GITHUB_ACTION_PATH}/../../scripts/parse_ref.py"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 93-108 / 第 93-108 行

````yaml
    - name: Get the job name
      id: get-job-name
      if: inputs.job-name == ''
      continue-on-error: true
      shell: bash
      run: |
        set -x

        # TODO: This is a very hacky way to get the job name. GitHub runner has the info
        # but doesn't expose it in anyway. The job name is part of the job message the
        # runner receives, so it's there and printed out to the diag log. Below is the
        # code responsible for printing it. Need to check with GitHub to see if they can
        # expose this variable as part of GitHub context.
        # https://github.com/actions/runner/blob/main/src/Runner.Worker/JobExtension.cs#L345
        pushd "${{ runner.workspace }}/../../_diag"
        pwd
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 110-125 / 第 110-125 行

````yaml
        LOG_FILE=$(grep -l -r "${{ github.sha }}" *.log | tail -n 1)
        if [ -n "${LOG_FILE}" ]; then
          # For some reasons, awk {print $2} on Linux and Windows (bash) work correctly while it
          # needs to be awk {print $3} on MacOS
          case ${RUNNER_OS} in
            macOS)
              JOB_NAME=$(grep -r "\"jobDisplayName\"" "${LOG_FILE}" | awk -F '[:]' '{print $3}' | sed 's/"//g' | xargs)
              ;;
            *)
              JOB_NAME=$(grep -r "\"jobDisplayName\"" "${LOG_FILE}" | awk -F '[:]' '{print $2}' | sed 's/"//g' | xargs)
              ;;
          esac
          echo "job-name=${JOB_NAME}" >> "${GITHUB_OUTPUT}"
        fi

        popd
````

- EN: This section describes repository automation behavior for `.github/actions/filter-test-configs/action.yml`.
- CN: 该部分描述 `.github/actions/filter-test-configs/action.yml` 的仓库自动化行为。

### Lines 127-142 / 第 127-142 行

````yaml
    - name: Select all requested test configurations
      shell: bash
      env:
        GITHUB_TOKEN: ${{ inputs.github-token }}
        JOB_NAME: ${{ inputs.job-name == '' && steps.get-job-name.outputs.job-name || inputs.job-name }}
        PR_NUMBER: ${{ github.event.pull_request.number }}
        TAG: ${{ steps.parse-ref.outputs.tag }}
        EVENT_NAME: ${{ github.event_name }}
        SCHEDULE: ${{ github.event.schedule }}
        HEAD_BRANCH: ${{ steps.parse-ref.outputs.branch }}
      id: filter
      run: |
        echo "Workflow: ${GITHUB_WORKFLOW}"
        echo "Job name: ${JOB_NAME}"

        # Use relative path here as this could be checked out anywhere, not necessarily
````

- EN: This section declares workflow triggers or event bindings; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 143-159 / 第 143-159 行

````yaml
        # in runner workspace
        uv run --no-project --with requests==2.27.1 --with pyyaml==6.0.2 python "${GITHUB_ACTION_PATH}/../../scripts/filter_test_configs.py" \
          --workflow "${GITHUB_WORKFLOW}" \
          --job-name "${JOB_NAME}" \
          --test-matrix "${{ inputs.test-matrix }}" \
          --selected-test-configs "${{ inputs.selected-test-configs }}" \
          --pr-number "${PR_NUMBER}" \
          --tag "${TAG}" \
          --event-name "${EVENT_NAME}" \
          --schedule "${SCHEDULE}" \
          --branch "${HEAD_BRANCH}"

    - name: Print the filtered test matrix
      shell: bash
      run: |
        echo "Filtered matrix:"
        echo "${{ steps.filter.outputs.test-matrix }}"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 161-168 / 第 161-168 行

````yaml
        echo
        echo "Is the current job unstable? ${{ steps.filter.outputs.is-unstable }}"

        echo
        echo "Is keep-going label set? ${{ steps.filter.outputs.keep-going }}"

        echo
        echo "Is ci-no-td label set? ${{ steps.filter.outputs.ci-no-td }}"
````

- EN: This section describes repository automation behavior for `.github/actions/filter-test-configs/action.yml`.
- CN: 该部分描述 `.github/actions/filter-test-configs/action.yml` 的仓库自动化行为。

### Lines 170-171 / 第 170-171 行

````yaml
        echo
        echo "Reenabled issues? ${{ steps.filter.outputs.reenabled-issues }}"
````

- EN: This section describes repository automation behavior for `.github/actions/filter-test-configs/action.yml`.
- CN: 该部分描述 `.github/actions/filter-test-configs/action.yml` 的仓库自动化行为。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: `pytorch/test-infra/.github/actions/setup-uv@main`
- Inline commands / 内联命令: `Apply`, `by`, `A`, `The`, `set`, `uv`, `pushd`, `pwd`, ...
- Environment variables / 环境变量: `GITHUB_TOKEN`, `JSON`, `UV_PYTHON`, `GITHUB_ACTION_PATH`, `TODO`, `L345`, `LOG_FILE`, `RUNNER_OS`, `JOB_NAME`, `GITHUB_OUTPUT`, ...
- Named jobs or sections / 命名作业或章节: `inputs`, `github-token`, `test-matrix`, `selected-test-configs`, `job-name`, `outputs`, `is-test-matrix-empty`, `keep-going`, `reenabled-issues`, `ci-verbose-test-logs`, ...
