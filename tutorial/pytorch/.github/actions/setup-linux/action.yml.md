# action.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/actions/setup-linux/action.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a reusable composite GitHub Action, including declared inputs, step composition, and shell execution.
- **用途 (CN)**: 定义可复用的复合型 GitHub Action，包括输入声明、步骤组合以及 shell 执行逻辑。

## Content Analysis / 内容分析
### Lines 1-17 / 第 1-17 行

````yaml
name: Setup Linux

description: Set up Linux workspace on EC2 or OSDC ARC runners

inputs:
  use-arc:
    description: Whether the runner is an OSDC ARC runner (returned by runner_determinator)
    required: false
    default: ''
  python-version:
    description: Python version to install (e.g. "3.12"). Empty string keeps the default.
    required: false
    default: ''
  compiler:
    description: Compiler name and version (e.g. "gcc11", "clang15"). Empty string keeps the base image default.
    required: false
    default: ''
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 18-34 / 第 18-34 行

````yaml
  cuda-version:
    description: CUDA version to activate (e.g. "13.0"). Empty string keeps the default.
    required: false
    default: ''
  submodules:
    description: Submodule checkout mode passed to checkout-pytorch (default "recursive", use "false" for test jobs).
    required: false
    default: 'recursive'
  github-token:
    description: GITHUB_TOKEN, needed to retrieve the workflow job id.
    required: false
    default: ''

outputs:
  branch:
    description: Parsed branch name from GITHUB_REF
    value: ${{ steps.parse-ref.outputs.branch }}
````

- EN: This section describes repository automation behavior for `.github/actions/setup-linux/action.yml`.
- CN: 该部分描述 `.github/actions/setup-linux/action.yml` 的仓库自动化行为。

### Lines 35-48 / 第 35-48 行

````yaml
  tag:
    description: Parsed tag name from GITHUB_REF (if applicable)
    value: ${{ steps.parse-ref.outputs.tag }}
  job-id:
    description: The workflow job id
    value: ${{ steps.get-job-id.outputs.job-id }}
  job-name:
    description: The workflow job name
    value: ${{ steps.get-job-id.outputs.job-name }}

runs:
  using: composite
  steps:
    # ── ARC-only steps ──────────────────────────────────────────────────
````

- EN: This section reuses actions and step building blocks such as named actions.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 49-65 / 第 49-65 行

````yaml
    - name: Fix workspace permissions
      if: ${{ inputs.use-arc == 'true' }}
      shell: bash
      run: |
        # GH runner image has switched to uid 1001 https://github.com/actions/runner-images/issues/10936
        # while current PyTorch CI image are still using uid 1000 (ec2-user). We
        # can update the uid to 1001 eventually when everything migrates to ARC.
        # In the meantime, this is a quick fix to ensure that CI has the permission
        # to use the GITHUB_WORKSPACE while still allowing the GH hook (uid 1001,
        # gid 1001) to clean up the directory after the job
        sudo chmod -R 777 "$GITHUB_WORKSPACE"

    - name: Ack Git cache ownership
      if: ${{ inputs.use-arc == 'true' }}
      shell: bash
      run: |
        git config --global --add safe.directory "$GITHUB_WORKSPACE"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 67-84 / 第 67-84 行

````yaml
    # ── Common steps (shared) ────────────────────────────────────────
    - name: Checkout PyTorch
      uses: pytorch/pytorch/.github/actions/checkout-pytorch@main
      with:
        no-sudo: true
        checkout-mode: treeless
        submodules: ${{ inputs.submodules }}

    - name: Parse ref
      id: parse-ref
      shell: bash
      run: |
        if [ -f "${{ github.action_path }}/../../scripts/parse_ref.py" ]; then
          python3 "${{ github.action_path }}/../../scripts/parse_ref.py"
        elif [ -f .github/scripts/parse_ref.py ]; then
          python3 .github/scripts/parse_ref.py
        else
          echo "ERROR: parse_ref.py not found" >&2
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/checkout-pytorch@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 85-101 / 第 85-101 行

````yaml
          exit 1
        fi

    - name: Get workflow job id
      id: get-job-id
      if: ${{ always() && inputs.github-token != '' }}
      uses: pytorch/pytorch/.github/actions/get-workflow-job-id@main
      with:
        github-token: ${{ inputs.github-token }}

    # ── EC2-only steps ──────────────────────────────────────────────────
    - name: Display EC2 information
      if: ${{ inputs.use-arc != 'true' }}
      shell: bash
      run: |
        set -euo pipefail
        function get_ec2_metadata() {
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/get-workflow-job-id@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 102-118 / 第 102-118 行

````yaml
          # Pulled from instance metadata endpoint for EC2
          # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
          category=$1
          # If it is GCP runner (runner name contains gcp), do not run this
          runner_name_str=${{ runner.name }}
          if [[ -f /.inarc ]]; then
            echo "ARC Runner, no info on ec2 metadata"
          elif [[ $runner_name_str == *"gcp"* || $runner_name_str == *"google"* ]]; then
            echo "Runner is from Google Cloud Platform, No info on ec2 metadata"
          else
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          fi
        }
        echo "ami-id: $(get_ec2_metadata ami-id)"
        echo "instance-id: $(get_ec2_metadata instance-id)"
        echo "instance-type: $(get_ec2_metadata instance-type)"
        echo "system info $(uname -a)"
````

- EN: This section describes repository automation behavior for `.github/actions/setup-linux/action.yml`.
- CN: 该部分描述 `.github/actions/setup-linux/action.yml` 的仓库自动化行为。

### Lines 120-136 / 第 120-136 行

````yaml
    - name: Check if in a container runner
      if: ${{ inputs.use-arc != 'true' }}
      shell: bash
      id: check_container_runner
      run: echo "IN_CONTAINER_RUNNER=$(if [ -f /.inarc ] || [ -f /.incontainer ]; then echo true ; else echo false; fi)" >> "$GITHUB_OUTPUT"

    - name: Start docker if docker daemon is not running
      shell: bash
      if: ${{ steps.check_container_runner.outputs.IN_CONTAINER_RUNNER == 'false' }}
      run: |
        if ! docker version >/dev/null 2>/dev/null; then
          if systemctl is-active --quiet docker; then
              echo "Docker daemon is running...";
          else
              echo "Starting docker daemon..." && sudo systemctl start docker;
          fi
        fi
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 138-154 / 第 138-154 行

````yaml
    - name: Install uv (EC2)
      uses: pytorch/test-infra/.github/actions/setup-uv@main
      if: ${{ steps.check_container_runner.outputs.IN_CONTAINER_RUNNER == 'false' }}
      with:
        python-version: "3.12"

    - name: Install pip
      shell: bash
      if: ${{ steps.check_container_runner.outputs.IN_CONTAINER_RUNNER == 'false' }}
      run: |
        set -euo pipefail
        if command -v pip3 &> /dev/null; then
          echo "pip3 is already installed."
        else
          echo "pip3 not found, installing..."
          uv tool install pip
        fi
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-uv@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 156-171 / 第 156-171 行

````yaml
    - name: Kill any existing containers, clean up images
      if: ${{ steps.check_container_runner.outputs.IN_CONTAINER_RUNNER == 'false' }}
      shell: bash
      run: |
        # ignore expansion of "docker ps -q" since it could be empty
        # shellcheck disable=SC2046
        docker stop $(docker ps -q) || true
        # Prune all of the docker images
        docker system prune -af

    - name: Check that the docker daemon is running
      shell: bash
      continue-on-error: true
      if: ${{ steps.check_container_runner.outputs.IN_CONTAINER_RUNNER == 'true' }}
      run: |
        set +x
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 173-189 / 第 173-189 行

````yaml
        max_attempts=30
        delay=10
        attempt=1

        for attempt in $(seq 1 $max_attempts); do
          echo "Attempt $attempt of $max_attempts: Checking if Docker daemon is running..."
          if docker info > /dev/null 2>&1; then
            echo "Docker is running. Proceeding with the next steps"
            exit 0
          else
            echo "Docker is not running yet."
            echo "Retrying in $delay seconds..."
            sleep $delay
          fi
        done
        echo "Reached maximum attempts to connect to Docker. Exiting."
        exit 1
````

- EN: This section describes repository automation behavior for `.github/actions/setup-linux/action.yml`.
- CN: 该部分描述 `.github/actions/setup-linux/action.yml` 的仓库自动化行为。

### Lines 191-194 / 第 191-194 行

````yaml
    # ── Shared steps ────────────────────────────────────────────────────
    - name: Print GPU info (if present)
      shell: bash
      run: if [ -f /usr/bin/nvidia-smi ]; then nvidia-smi; fi
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 196-200 / 第 196-200 行

````yaml
    - name: Preserve github env variables for use in docker
      shell: bash
      run: |
        env | grep '^GITHUB' >> "/tmp/github_env_${GITHUB_RUN_ID}"
        env | grep '^CI' >> "/tmp/github_env_${GITHUB_RUN_ID}"
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
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `pytorch/pytorch/.github/actions/get-workflow-job-id@main`, `pytorch/test-infra/.github/actions/setup-uv@main`
- Inline commands / 内联命令: `sudo`, `git`, `python3`, `echo`, `exit`, `set`, `curl`, `uv`, ...
- Environment variables / 环境变量: `EC2`, `OSDC`, `ARC`, `CUDA`, `GITHUB_TOKEN`, `GITHUB_REF`, `GITHUB_WORKSPACE`, `ERROR`, `AWSEC2`, `GCP`, ...
- Named jobs or sections / 命名作业或章节: `inputs`, `use-arc`, `python-version`, `compiler`, `cuda-version`, `submodules`, `github-token`, `outputs`, `branch`, `tag`, ...
