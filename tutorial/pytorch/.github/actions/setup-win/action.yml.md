# action.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/actions/setup-win/action.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a reusable composite GitHub Action, including declared inputs, step composition, and shell execution.
- **用途 (CN)**: 定义可复用的复合型 GitHub Action，包括输入声明、步骤组合以及 shell 执行逻辑。

## Content Analysis / 内容分析
### Lines 1-14 / 第 1-14 行

````yaml
name: Setup Windows

description: Set up for windows jobs

inputs:
  cuda-version:
    description: which cuda version to install, 'cpu' for none
    required: true
  python-version:
    required: false
    type: string
    default: "3.10"
    description: |
      The python version to be used. Will be 3.10 by default
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 16-24 / 第 16-24 行

````yaml
runs:
  using: composite
  steps:
    - name: Display EC2 information
      shell: bash
      run: |
        set -euo pipefail
        function get_ec2_metadata() {
          # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 25-34 / 第 25-34 行

````yaml
          # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
          category=$1
          curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
        }
        echo "ami-id: $(get_ec2_metadata ami-id)"
        echo "instance-id: $(get_ec2_metadata instance-id)"
        echo "instance-type: $(get_ec2_metadata instance-type)"
        echo "system info $(uname -a)"

    # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
````

- EN: This section describes repository automation behavior for `.github/actions/setup-win/action.yml`.
- CN: 该部分描述 `.github/actions/setup-win/action.yml` 的仓库自动化行为。

### Lines 35-44 / 第 35-44 行

````yaml
    - name: Enable long paths on Windows
      shell: powershell
      run: |
        Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1

    - name: Setup conda
      shell: bash
      run: |
        # Windows conda is baked into the AMI at this location
        CONDA="C:\Jenkins\Miniconda3\condabin\conda.bat"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 46-59 / 第 46-59 行

````yaml
        {
          echo "CONDA=${CONDA}";
          echo "CONDA_RUN=${CONDA} run --no-capture-output";
          echo "CONDA_BUILD=${CONDA} run conda-build";
          echo "CONDA_INSTALL=${CONDA} install";
        } >> "${GITHUB_ENV}"

    - name: Setup Python3
      env:
          PYTHON_VERSION: ${{ inputs.python-version }}
      shell: bash
      run: |
        set +e
        set -x
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 61-73 / 第 61-73 行

````yaml
        # Create new py_tmp env with python-version
        ${CONDA} create -y -n py_tmp python=${PYTHON_VERSION} intel-openmp libuv

        PYTHON3=$(${CONDA_RUN} -n py_tmp which python3)
        EXIT_CODE=$?

        if [[ "${EXIT_CODE}" == "0" ]]; then
          echo "Found Python3 at ${PYTHON3}, adding it into GITHUB_PATH"

          PYTHON_PATH=$(dirname "${PYTHON3}")
          echo "${PYTHON_PATH}" >> "${GITHUB_PATH}"
        else
          # According to https://docs.conda.io/en/latest/miniconda.html, we are using the Miniconda3
````

- EN: This section describes repository automation behavior for `.github/actions/setup-win/action.yml`.
- CN: 该部分描述 `.github/actions/setup-win/action.yml` 的仓库自动化行为。

### Lines 74-86 / 第 74-86 行

````yaml
          # installation, which is Python 3 based. Its Python is default to Python 3. Further, there
          # is also the Miniconda installation that is Python 2 based, and both can be installed if
          # needed. In both cases, Python binary is just called python
          PYTHON=$(${CONDA_RUN} -n py_tmp which python)
          EXIT_CODE=$?

          if [[ "${EXIT_CODE}" == "0" ]]; then
            echo "Found Python at ${PYTHON}, set Python3 alias and add it into GITHUB_PATH"

            PYTHON3=$(echo "${PYTHON}" | sed "s/python/python3/")
            # It's difficult to setup alias across GitHub action steps, so I just add a softlink
            # here pointing to Python
            ln -s "${PYTHON}" "${PYTHON3}"
````

- EN: This section describes repository automation behavior for `.github/actions/setup-win/action.yml`.
- CN: 该部分描述 `.github/actions/setup-win/action.yml` 的仓库自动化行为。

### Lines 88-99 / 第 88-99 行

````yaml
            PYTHON_PATH=$(dirname "${PYTHON}")
            echo "${PYTHON_PATH}" >> "${GITHUB_PATH}"
          else
            echo "Found no Python using ${CONDA_RUN}"
          fi
        fi

    - name: Get temporary directory used by Windows Python
      shell: bash
      run: |
        TMPDIR=$(python -c 'import tempfile; print(tempfile.gettempdir());')
        echo "TMPDIR=${TMPDIR}" >> "${GITHUB_ENV}"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 101-110 / 第 101-110 行

````yaml
    # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
    # removed once Windows Defender is removed from the AMI
    - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
      continue-on-error: true
      shell: powershell
      run: |
        Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TMPDIR,"C:\Jenkins\Miniconda3" -ErrorAction Ignore
        # Let's both exclude the path and disable Windows Defender completely just to be sure
        # that it doesn't interfere
        Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 112-117 / 第 112-117 行

````yaml
    - name: Install sysinternals handle tool
      continue-on-error: true
      shell: powershell
      run: |
        choco install handle -y
        handle C:\actions-runner\_work\
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。


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
- Reusable actions / 复用 Action: none
- Inline commands / 内联命令: `The`, `set`, `curl`, `echo`, `Set-ItemProperty`, `ln`, `Add-MpPreference`, `Set-MpPreference`, ...
- Environment variables / 环境变量: `EC2`, `AWSEC2`, `PUT`, `HKLM`, `SYSTEM`, `AMI`, `CONDA`, `CONDA_RUN`, `CONDA_BUILD`, `CONDA_INSTALL`, ...
- Named jobs or sections / 命名作业或章节: `inputs`, `cuda-version`, `python-version`, `runs`, `steps`
