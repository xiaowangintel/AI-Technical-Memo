# action.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/actions/setup-rocm/action.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a reusable composite GitHub Action, including declared inputs, step composition, and shell execution.
- **用途 (CN)**: 定义可复用的复合型 GitHub Action，包括输入声明、步骤组合以及 shell 执行逻辑。

## Content Analysis / 内容分析
### Lines 1-12 / 第 1-12 行

````yaml
name: Setup ROCm host

description: Set up ROCm host for CI

runs:
  using: composite
  steps:
    - name: Runner ROCm version
      if: always()
      shell: bash
      run: |
        dpkg -l | grep -E "  rocm"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 14-22 / 第 14-22 行

````yaml
    - name: Stop all running docker containers
      if: always()
      shell: bash
      run: |
        # ignore expansion of "docker ps -q" since it could be empty
        # shellcheck disable=SC2046
        docker stop $(docker ps -q) || true
        # Prune all stopped containers.
        docker container prune -f
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 24-31 / 第 24-31 行

````yaml
    - name: Runner health check system info
      if: always()
      shell: bash
      run: |
        cat /etc/os-release || true
        cat /etc/apt/sources.list.d/rocm.list || true
        cat /opt/rocm/.info/version || true
        whoami
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 33-43 / 第 33-43 行

````yaml
    - name: Runner health check amdgpu info
      if: always()
      shell: bash
      run: |
        dpkg -l | grep -E "  amdgpu"

    - name: Runner health check rocm-smi
      if: always()
      shell: bash
      run: |
        rocm-smi
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 45-58 / 第 45-58 行

````yaml
    - name: Runner health check rocminfo
      if: always()
      shell: bash
      run: |
        rocminfo

    - name: Runner health check GPU count
      if: always()
      shell: bash
      run: |
        ngpu=$(rocminfo | grep -c -E 'Name:.*\sgfx')
        msg="Please file an issue on pytorch/pytorch reporting the faulty runner. Include a link to the runner logs so the runner can be identified"
        if [[ $ngpu -eq 0 ]]; then
            echo "Error: Failed to detect any GPUs on the runner"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 59-71 / 第 59-71 行

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

### Lines 73-84 / 第 73-84 行

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

### Lines 86-95 / 第 86-95 行

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
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 97-109 / 第 97-109 行

````yaml
    - name: ROCm set GPU_FLAG
      shell: bash
      run: |
        # All GPUs are visible to the runner; visibility, if needed, will be set by run_test.py.
        # Add render group for container creation.
        render_gid=`cat /etc/group | grep render | cut -d: -f3`
        # Ensure GPU isolation if pod is part of kubernetes setup with DEVICE_FLAG.
        if [ -f "/etc/podinfo/gha-render-devices" ]; then
          DEVICE_FLAG=$(cat /etc/podinfo/gha-render-devices)
        else
          DEVICE_FLAG="--device /dev/dri"
        fi
        # The --group-add daemon and --group-add bin are needed in the Ubuntu 24.04 and Almalinux OSs respectively.
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 110-123 / 第 110-123 行

````yaml
        # This is due to the device files (/dev/kfd & /dev/dri) being owned by video group on bare metal.
        # This video group ID maps to subgid 1 inside the docker image due to the /etc/subgid entries.
        # The group name corresponding to group ID 1 can change depending on the OS, so both are necessary.
        echo "GPU_FLAG=--device=/dev/mem --device=/dev/kfd $DEVICE_FLAG --group-add video --group-add $render_gid --group-add daemon --group-add bin --cap-add=SYS_PTRACE --security-opt seccomp=unconfined" >> "${GITHUB_ENV}"

    - name: Login to ECR
      uses: pytorch/pytorch/.github/actions/ecr-login@main

    - name: Preserve github env variables for use in docker
      shell: bash
      run: |
        env | grep '^GITHUB' >> "${RUNNER_TEMP}/github_env_${GITHUB_RUN_ID}"
        env | grep '^CI' >> "${RUNNER_TEMP}/github_env_${GITHUB_RUN_ID}"
        env | grep '^RUNNER' >> "${RUNNER_TEMP}/github_env_${GITHUB_RUN_ID}"
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/ecr-login@main`; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。


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
- Inline commands / 内联命令: `dpkg`, `docker`, `cat`, `whoami`, `rocm-smi`, `rocminfo`, `echo`, `exit`, ...
- Environment variables / 环境变量: `SC2046`, `GPU`, `RUNNER_ARTIFACT_DIR`, `RUNNER_TEMP`, `GITHUB_ENV`, `RUNNER_TEST_RESULTS_DIR`, `RUNNER_DOCS_DIR`, `GITHUB`, `GITHUB_RUN_ID`, `GPU_FLAG`, ...
- Named jobs or sections / 命名作业或章节: `runs`, `steps`
