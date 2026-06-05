# action.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/actions/setup-xpu/action.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a reusable composite GitHub Action, including declared inputs, step composition, and shell execution.
- **用途 (CN)**: 定义可复用的复合型 GitHub Action，包括输入声明、步骤组合以及 shell 执行逻辑。

## Content Analysis / 内容分析
### Lines 1-12 / 第 1-12 行

````yaml
name: Setup XPU host

description: Set up XPU host for CI

runs:
  using: composite
  steps:
    - name: Clean all stopped docker containers
      if: always()
      shell: bash
      run: |
        # Prune all stopped containers.
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 13-26 / 第 13-26 行

````yaml
        # If other runner is pruning on this node, will skip.
        nprune=$(ps -ef | grep -c "docker container prune")
        if [[ $nprune -eq 1 ]]; then
          docker container prune -f
        fi

    - name: Runner health check system info
      if: always()
      shell: bash
      run: |
        cat /etc/os-release || true
        cat /etc/apt/sources.list.d/oneAPI.list || true
        cat /etc/apt/sources.list.d/intel-gpu-jammy.list || true
        whoami
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 28-41 / 第 28-41 行

````yaml
    - name: Runner health check xpu-smi
      if: always()
      shell: bash
      run: |
        timeout 30 xpu-smi discovery || true

    - name: Runner health check GPU count
      if: always()
      shell: bash
      run: |
        ngpu=$(timeout 30 xpu-smi discovery | grep -c -E 'Device Name' || true)
        msg="Please file an issue on pytorch/pytorch reporting the faulty runner. Include a link to the runner logs so the runner can be identified"
        if [[ $ngpu -eq 0 ]]; then
          echo "Error: Failed to detect any GPUs on the runner"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 42-54 / 第 42-54 行

````yaml
          echo "$msg"
          exit 1
        fi

    - name: Runner diskspace health check
      uses: pytorch/pytorch/.github/actions/diskspace-cleanup@main
      if: always()

    - name: Runner health check disconnect on failure
      if: ${{ failure() }}
      shell: bash
      run: |
        killall runsvc.sh
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/diskspace-cleanup@main`; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 56-67 / 第 56-67 行

````yaml
    - name: Setup useful environment variables
      shell: bash
      run: |
        RUNNER_ARTIFACT_DIR="${RUNNER_TEMP}/artifacts"
        rm -rf "${RUNNER_ARTIFACT_DIR}"
        mkdir -p "${RUNNER_ARTIFACT_DIR}"
        echo "RUNNER_ARTIFACT_DIR=${RUNNER_ARTIFACT_DIR}" >> "${GITHUB_ENV}"

        RUNNER_TEST_RESULTS_DIR="${RUNNER_TEMP}/test-results"
        rm -rf "${RUNNER_TEST_RESULTS_DIR}"
        mkdir -p "${RUNNER_TEST_RESULTS_DIR}"
        echo "RUNNER_TEST_RESULTS_DIR=${RUNNER_TEST_RESULTS_DIR}" >> "${GITHUB_ENV}"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 69-79 / 第 69-79 行

````yaml
        RUNNER_DOCS_DIR="${RUNNER_TEMP}/docs"
        rm -rf "${RUNNER_DOCS_DIR}"
        mkdir -p "${RUNNER_DOCS_DIR}"
        echo "RUNNER_DOCS_DIR=${RUNNER_DOCS_DIR}" >> "${GITHUB_ENV}"

    - name: Preserve github env variables for use in docker
      shell: bash
      run: |
        env | grep '^GITHUB' >> "${RUNNER_TEMP}/github_env_${GITHUB_RUN_ID}"
        env | grep '^CI' >> "${RUNNER_TEMP}/github_env_${GITHUB_RUN_ID}"
        env | grep '^RUNNER' >> "${RUNNER_TEMP}/github_env_${GITHUB_RUN_ID}"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 81-86 / 第 81-86 行

````yaml
    - name: XPU set GPU_FLAG
      shell: bash
      run: |
        # Add render group for container creation.
        render_gid=`cat /etc/group | grep render | cut -d: -f3`
        echo "GPU_FLAG=--device=/dev/mem --device=/dev/dri --group-add video --group-add $render_gid" >> "${GITHUB_ENV}"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 88-89 / 第 88-89 行

````yaml
    - name: Login to ECR
      uses: pytorch/pytorch/.github/actions/ecr-login@main
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/ecr-login@main`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/actions/diskspace-cleanup@main`, `pytorch/pytorch/.github/actions/ecr-login@main`
- Inline commands / 内联命令: `docker`, `cat`, `whoami`, `timeout`, `echo`, `exit`, `killall`, `rm`, ...
- Environment variables / 环境变量: `XPU`, `GPU`, `RUNNER_ARTIFACT_DIR`, `RUNNER_TEMP`, `GITHUB_ENV`, `RUNNER_TEST_RESULTS_DIR`, `RUNNER_DOCS_DIR`, `GITHUB`, `GITHUB_RUN_ID`, `RUNNER`, ...
- Named jobs or sections / 命名作业或章节: `runs`, `steps`
