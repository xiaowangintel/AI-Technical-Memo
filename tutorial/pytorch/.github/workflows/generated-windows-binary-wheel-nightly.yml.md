# generated-windows-binary-wheel-nightly.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/generated-windows-binary-wheel-nightly.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-21 / 第 1-21 行

````yaml
# @generated DO NOT EDIT MANUALLY

# Template is at:    .github/templates/windows_binary_build_workflow.yml.j2
# Generation script: .github/scripts/generate_ci_workflows.py
name: windows-binary-wheel

on:
  push:
    # NOTE: Meta Employees can trigger new nightlies using: https://fburl.com/trigger_pytorch_nightly_build
    branches:
      - nightly
    tags:
      # NOTE: Binary build pipelines should only get triggered on release candidate builds
      # Release candidate tags look like: v1.11.0-rc1
      - v[0-9]+.[0-9]+.[0-9]+-rc[0-9]+
      - 'ciflow/binaries/*'
      - 'ciflow/binaries_libtorch/*'
      - 'ciflow/binaries_wheel/*'
  workflow_dispatch:

env:
````

- EN: This section declares workflow triggers or event bindings; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；控制令牌权限范围或环境变量以实现安全执行。

### Lines 22-44 / 第 22-44 行

````yaml
  # Needed for conda builds
  ALPINE_IMAGE: "308535385114.dkr.ecr.us-east-1.amazonaws.com/tool/alpine"
  AWS_DEFAULT_REGION: us-east-1
  BUILD_ENVIRONMENT: windows-binary-wheel
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  PR_NUMBER: ${{ github.event.pull_request.number }}
  SHA1: ${{ github.event.pull_request.head.sha || github.sha }}
  SKIP_ALL_TESTS: 1
  OS: windows
concurrency:
  group: windows-binary-wheel-${{ github.event.pull_request.number || github.ref_name }}-${{ github.ref_type == 'branch' && github.sha }}-${{ github.event_name == 'workflow_dispatch' }}
  cancel-in-progress: true

jobs:
  get-label-type:
    if: github.repository_owner == 'pytorch'
    name: get-label-type
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 45-63 / 第 45-63 行

````yaml
  wheel-py3_10-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.10"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
````

- EN: This section reuses actions and step building blocks such as named actions; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 64-86 / 第 64-86 行

````yaml
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 87-105 / 第 87-105 行

````yaml
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 106-127 / 第 106-127 行

````yaml
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 128-149 / 第 128-149 行

````yaml
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_10-cpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 151-173 / 第 151-173 行

````yaml
  wheel-py3_10-cpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_10-cpu-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.4xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.10"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 174-196 / 第 174-196 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 197-219 / 第 197-219 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 220-239 / 第 220-239 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_10-cpu
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 240-260 / 第 240-260 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_10-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_10-cpu-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 261-283 / 第 261-283 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DESIRED_PYTHON: "3.10"
      build_name: wheel-py3_10-cpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_10-cuda12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 284-306 / 第 284-306 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.10"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 307-329 / 第 307-329 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 330-352 / 第 330-352 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 353-375 / 第 353-375 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_10-cuda12_6
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 376-398 / 第 376-398 行

````yaml
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_10-cuda12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_10-cuda12_6-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.10"
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 399-418 / 第 399-418 行

````yaml
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 419-441 / 第 419-441 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 442-464 / 第 442-464 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 465-481 / 第 465-481 行

````yaml
        with:
          name: wheel-py3_10-cuda12_6
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 482-502 / 第 482-502 行

````yaml
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_10-cuda12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_10-cuda12_6-test
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.10"
      build_name: wheel-py3_10-cuda12_6
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 503-525 / 第 503-525 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_10-cuda13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.10"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 526-547 / 第 526-547 行

````yaml
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 548-569 / 第 548-569 行

````yaml
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 570-591 / 第 570-591 行

````yaml
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 592-613 / 第 592-613 行

````yaml
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_10-cuda13_0
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 615-637 / 第 615-637 行

````yaml
  wheel-py3_10-cuda13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_10-cuda13_0-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.10"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 638-659 / 第 638-659 行

````yaml
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 660-682 / 第 660-682 行

````yaml
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 683-704 / 第 683-704 行

````yaml
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_10-cuda13_0
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 705-725 / 第 705-725 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_10-cuda13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_10-cuda13_0-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 726-746 / 第 726-746 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.10"
      build_name: wheel-py3_10-cuda13_0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_10-cuda13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 747-766 / 第 747-766 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.10"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 767-785 / 第 767-785 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 786-808 / 第 786-808 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 809-830 / 第 809-830 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 831-851 / 第 831-851 行

````yaml
        with:
          name: wheel-py3_10-cuda13_2
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_10-cuda13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_10-cuda13_2-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 852-871 / 第 852-871 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.10"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 872-894 / 第 872-894 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 895-917 / 第 895-917 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 918-937 / 第 918-937 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_10-cuda13_2
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 938-958 / 第 938-958 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_10-cuda13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_10-cuda13_2-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 959-979 / 第 959-979 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.10"
      build_name: wheel-py3_10-cuda13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_10-xpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 980-999 / 第 980-999 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.10"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: intel-cmplr-lib-rt==2025.3.2 | intel-cmplr-lib-ur==2025.3.2 | intel-cmplr-lic-rt==2025.3.2 | intel-sycl-rt==2025.3.2 | oneccl-devel==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | oneccl==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | impi-rt==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | onemkl-license==2025.3.1 | onemkl-sycl-blas==2025.3.1 | onemkl-sycl-dft==2025.3.1 | onemkl-sycl-lapack==2025.3.1 | onemkl-sycl-rng==2025.3.1 | onemkl-sycl-sparse==2025.3.1 | dpcpp-cpp-rt==2025.3.2 | intel-opencl-rt==2025.3.2 | mkl==2025.3.1 | intel-openmp==2025.3.2 | tbb==2022.3.1 | tcmlib==1.4.1 | umf==1.0.3 | intel-pti==0.16.0
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1000-1018 / 第 1000-1018 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1019-1041 / 第 1019-1041 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1042-1063 / 第 1042-1063 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1064-1084 / 第 1064-1084 行

````yaml
        with:
          name: wheel-py3_10-xpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_10-xpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_10-xpu-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1085-1103 / 第 1085-1103 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.4xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.10"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1104-1126 / 第 1104-1126 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1127-1149 / 第 1127-1149 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1150-1169 / 第 1150-1169 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_10-xpu
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1170-1190 / 第 1170-1190 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_10-xpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_10-xpu-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1191-1213 / 第 1191-1213 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DESIRED_PYTHON: "3.10"
      build_name: wheel-py3_10-xpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_11-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1214-1236 / 第 1214-1236 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.11"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 1237-1259 / 第 1237-1259 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1260-1282 / 第 1260-1282 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1283-1305 / 第 1283-1305 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_11-cpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1306-1328 / 第 1306-1328 行

````yaml
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_11-cpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_11-cpu-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.4xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.11"
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1329-1347 / 第 1329-1347 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1348-1370 / 第 1348-1370 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1371-1393 / 第 1371-1393 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1394-1410 / 第 1394-1410 行

````yaml
        with:
          name: wheel-py3_11-cpu
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1411-1430 / 第 1411-1430 行

````yaml
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_11-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_11-cpu-test
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DESIRED_PYTHON: "3.11"
      build_name: wheel-py3_11-cpu
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1431-1453 / 第 1431-1453 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_11-cuda12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.11"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1454-1475 / 第 1454-1475 行

````yaml
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 1476-1497 / 第 1476-1497 行

````yaml
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1498-1519 / 第 1498-1519 行

````yaml
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1520-1541 / 第 1520-1541 行

````yaml
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_11-cuda12_6
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1543-1565 / 第 1543-1565 行

````yaml
  wheel-py3_11-cuda12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_11-cuda12_6-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.11"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1566-1587 / 第 1566-1587 行

````yaml
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1588-1610 / 第 1588-1610 行

````yaml
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1611-1632 / 第 1611-1632 行

````yaml
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_11-cuda12_6
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1633-1653 / 第 1633-1653 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_11-cuda12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_11-cuda12_6-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1654-1674 / 第 1654-1674 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.11"
      build_name: wheel-py3_11-cuda12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_11-cuda13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1675-1694 / 第 1675-1694 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.11"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1695-1713 / 第 1695-1713 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1714-1736 / 第 1714-1736 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1737-1758 / 第 1737-1758 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1759-1779 / 第 1759-1779 行

````yaml
        with:
          name: wheel-py3_11-cuda13_0
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_11-cuda13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_11-cuda13_0-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1780-1799 / 第 1780-1799 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.11"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1800-1822 / 第 1800-1822 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1823-1845 / 第 1823-1845 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1846-1865 / 第 1846-1865 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_11-cuda13_0
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1866-1886 / 第 1866-1886 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_11-cuda13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_11-cuda13_0-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1887-1907 / 第 1887-1907 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.11"
      build_name: wheel-py3_11-cuda13_0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_11-cuda13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 1908-1927 / 第 1908-1927 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.11"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1928-1946 / 第 1928-1946 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1947-1969 / 第 1947-1969 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1970-1991 / 第 1970-1991 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 1992-2012 / 第 1992-2012 行

````yaml
        with:
          name: wheel-py3_11-cuda13_2
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_11-cuda13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_11-cuda13_2-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2013-2032 / 第 2013-2032 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.11"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2033-2055 / 第 2033-2055 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2056-2078 / 第 2056-2078 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2079-2098 / 第 2079-2098 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_11-cuda13_2
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2099-2119 / 第 2099-2119 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_11-cuda13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_11-cuda13_2-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2120-2140 / 第 2120-2140 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.11"
      build_name: wheel-py3_11-cuda13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_11-xpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2141-2160 / 第 2141-2160 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.11"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: intel-cmplr-lib-rt==2025.3.2 | intel-cmplr-lib-ur==2025.3.2 | intel-cmplr-lic-rt==2025.3.2 | intel-sycl-rt==2025.3.2 | oneccl-devel==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | oneccl==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | impi-rt==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | onemkl-license==2025.3.1 | onemkl-sycl-blas==2025.3.1 | onemkl-sycl-dft==2025.3.1 | onemkl-sycl-lapack==2025.3.1 | onemkl-sycl-rng==2025.3.1 | onemkl-sycl-sparse==2025.3.1 | dpcpp-cpp-rt==2025.3.2 | intel-opencl-rt==2025.3.2 | mkl==2025.3.1 | intel-openmp==2025.3.2 | tbb==2022.3.1 | tcmlib==1.4.1 | umf==1.0.3 | intel-pti==0.16.0
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2161-2179 / 第 2161-2179 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2180-2202 / 第 2180-2202 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2203-2224 / 第 2203-2224 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2225-2245 / 第 2225-2245 行

````yaml
        with:
          name: wheel-py3_11-xpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_11-xpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_11-xpu-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2246-2264 / 第 2246-2264 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.4xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.11"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2265-2287 / 第 2265-2287 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2288-2310 / 第 2288-2310 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2311-2330 / 第 2311-2330 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_11-xpu
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2331-2351 / 第 2331-2351 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_11-xpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_11-xpu-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2352-2374 / 第 2352-2374 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DESIRED_PYTHON: "3.11"
      build_name: wheel-py3_11-xpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_12-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2375-2397 / 第 2375-2397 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.12"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 2398-2420 / 第 2398-2420 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2421-2443 / 第 2421-2443 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2444-2466 / 第 2444-2466 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_12-cpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2467-2489 / 第 2467-2489 行

````yaml
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_12-cpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_12-cpu-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.4xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.12"
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2490-2508 / 第 2490-2508 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2509-2531 / 第 2509-2531 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2532-2554 / 第 2532-2554 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2555-2571 / 第 2555-2571 行

````yaml
        with:
          name: wheel-py3_12-cpu
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2572-2591 / 第 2572-2591 行

````yaml
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_12-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_12-cpu-test
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DESIRED_PYTHON: "3.12"
      build_name: wheel-py3_12-cpu
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2592-2614 / 第 2592-2614 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_12-cuda12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.12"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2615-2636 / 第 2615-2636 行

````yaml
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 2637-2658 / 第 2637-2658 行

````yaml
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2659-2680 / 第 2659-2680 行

````yaml
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2681-2702 / 第 2681-2702 行

````yaml
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_12-cuda12_6
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2704-2726 / 第 2704-2726 行

````yaml
  wheel-py3_12-cuda12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_12-cuda12_6-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.12"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2727-2748 / 第 2727-2748 行

````yaml
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2749-2771 / 第 2749-2771 行

````yaml
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2772-2793 / 第 2772-2793 行

````yaml
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_12-cuda12_6
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2794-2814 / 第 2794-2814 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_12-cuda12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_12-cuda12_6-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2815-2835 / 第 2815-2835 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.12"
      build_name: wheel-py3_12-cuda12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_12-cuda13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 2836-2855 / 第 2836-2855 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.12"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2856-2874 / 第 2856-2874 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2875-2897 / 第 2875-2897 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2898-2919 / 第 2898-2919 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2920-2940 / 第 2920-2940 行

````yaml
        with:
          name: wheel-py3_12-cuda13_0
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_12-cuda13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_12-cuda13_0-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2941-2960 / 第 2941-2960 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.12"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 2961-2983 / 第 2961-2983 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 2984-3006 / 第 2984-3006 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3007-3026 / 第 3007-3026 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_12-cuda13_0
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3027-3047 / 第 3027-3047 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_12-cuda13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_12-cuda13_0-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3048-3068 / 第 3048-3068 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.12"
      build_name: wheel-py3_12-cuda13_0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_12-cuda13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3069-3088 / 第 3069-3088 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.12"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3089-3107 / 第 3089-3107 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3108-3130 / 第 3108-3130 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3131-3152 / 第 3131-3152 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3153-3173 / 第 3153-3173 行

````yaml
        with:
          name: wheel-py3_12-cuda13_2
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_12-cuda13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_12-cuda13_2-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3174-3193 / 第 3174-3193 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.12"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3194-3216 / 第 3194-3216 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3217-3239 / 第 3217-3239 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3240-3259 / 第 3240-3259 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_12-cuda13_2
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3260-3280 / 第 3260-3280 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_12-cuda13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_12-cuda13_2-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3281-3301 / 第 3281-3301 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.12"
      build_name: wheel-py3_12-cuda13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_12-xpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3302-3321 / 第 3302-3321 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.12"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: intel-cmplr-lib-rt==2025.3.2 | intel-cmplr-lib-ur==2025.3.2 | intel-cmplr-lic-rt==2025.3.2 | intel-sycl-rt==2025.3.2 | oneccl-devel==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | oneccl==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | impi-rt==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | onemkl-license==2025.3.1 | onemkl-sycl-blas==2025.3.1 | onemkl-sycl-dft==2025.3.1 | onemkl-sycl-lapack==2025.3.1 | onemkl-sycl-rng==2025.3.1 | onemkl-sycl-sparse==2025.3.1 | dpcpp-cpp-rt==2025.3.2 | intel-opencl-rt==2025.3.2 | mkl==2025.3.1 | intel-openmp==2025.3.2 | tbb==2022.3.1 | tcmlib==1.4.1 | umf==1.0.3 | intel-pti==0.16.0
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3322-3340 / 第 3322-3340 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3341-3363 / 第 3341-3363 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3364-3385 / 第 3364-3385 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3386-3406 / 第 3386-3406 行

````yaml
        with:
          name: wheel-py3_12-xpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_12-xpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_12-xpu-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3407-3425 / 第 3407-3425 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.4xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.12"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3426-3448 / 第 3426-3448 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3449-3471 / 第 3449-3471 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3472-3491 / 第 3472-3491 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_12-xpu
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3492-3512 / 第 3492-3512 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_12-xpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_12-xpu-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3513-3535 / 第 3513-3535 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DESIRED_PYTHON: "3.12"
      build_name: wheel-py3_12-xpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_13-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3536-3558 / 第 3536-3558 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 3559-3581 / 第 3559-3581 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3582-3604 / 第 3582-3604 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3605-3627 / 第 3605-3627 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_13-cpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3628-3650 / 第 3628-3650 行

````yaml
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_13-cpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_13-cpu-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.4xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13"
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3651-3669 / 第 3651-3669 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3670-3692 / 第 3670-3692 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3693-3715 / 第 3693-3715 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3716-3732 / 第 3716-3732 行

````yaml
        with:
          name: wheel-py3_13-cpu
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3733-3752 / 第 3733-3752 行

````yaml
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_13-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_13-cpu-test
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DESIRED_PYTHON: "3.13"
      build_name: wheel-py3_13-cpu
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3753-3775 / 第 3753-3775 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_13-cuda12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3776-3797 / 第 3776-3797 行

````yaml
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 3798-3819 / 第 3798-3819 行

````yaml
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3820-3841 / 第 3820-3841 行

````yaml
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3842-3863 / 第 3842-3863 行

````yaml
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_13-cuda12_6
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3865-3887 / 第 3865-3887 行

````yaml
  wheel-py3_13-cuda12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_13-cuda12_6-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3888-3909 / 第 3888-3909 行

````yaml
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3910-3932 / 第 3910-3932 行

````yaml
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3933-3954 / 第 3933-3954 行

````yaml
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_13-cuda12_6
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 3955-3975 / 第 3955-3975 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_13-cuda12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_13-cuda12_6-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 3976-3996 / 第 3976-3996 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.13"
      build_name: wheel-py3_13-cuda12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_13-cuda13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 3997-4016 / 第 3997-4016 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4017-4035 / 第 4017-4035 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4036-4058 / 第 4036-4058 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4059-4080 / 第 4059-4080 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4081-4101 / 第 4081-4101 行

````yaml
        with:
          name: wheel-py3_13-cuda13_0
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_13-cuda13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_13-cuda13_0-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4102-4121 / 第 4102-4121 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4122-4144 / 第 4122-4144 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4145-4167 / 第 4145-4167 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4168-4187 / 第 4168-4187 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_13-cuda13_0
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4188-4208 / 第 4188-4208 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_13-cuda13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_13-cuda13_0-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4209-4229 / 第 4209-4229 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.13"
      build_name: wheel-py3_13-cuda13_0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_13-cuda13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 4230-4249 / 第 4230-4249 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4250-4268 / 第 4250-4268 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4269-4291 / 第 4269-4291 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4292-4313 / 第 4292-4313 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4314-4334 / 第 4314-4334 行

````yaml
        with:
          name: wheel-py3_13-cuda13_2
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_13-cuda13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_13-cuda13_2-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4335-4354 / 第 4335-4354 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4355-4377 / 第 4355-4377 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4378-4400 / 第 4378-4400 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4401-4420 / 第 4401-4420 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_13-cuda13_2
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4421-4441 / 第 4421-4441 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_13-cuda13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_13-cuda13_2-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4442-4462 / 第 4442-4462 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.13"
      build_name: wheel-py3_13-cuda13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_13-xpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 4463-4482 / 第 4463-4482 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: intel-cmplr-lib-rt==2025.3.2 | intel-cmplr-lib-ur==2025.3.2 | intel-cmplr-lic-rt==2025.3.2 | intel-sycl-rt==2025.3.2 | oneccl-devel==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | oneccl==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | impi-rt==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | onemkl-license==2025.3.1 | onemkl-sycl-blas==2025.3.1 | onemkl-sycl-dft==2025.3.1 | onemkl-sycl-lapack==2025.3.1 | onemkl-sycl-rng==2025.3.1 | onemkl-sycl-sparse==2025.3.1 | dpcpp-cpp-rt==2025.3.2 | intel-opencl-rt==2025.3.2 | mkl==2025.3.1 | intel-openmp==2025.3.2 | tbb==2022.3.1 | tcmlib==1.4.1 | umf==1.0.3 | intel-pti==0.16.0
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4483-4501 / 第 4483-4501 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4502-4524 / 第 4502-4524 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4525-4546 / 第 4525-4546 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4547-4567 / 第 4547-4567 行

````yaml
        with:
          name: wheel-py3_13-xpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_13-xpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_13-xpu-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4568-4586 / 第 4568-4586 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.4xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4587-4609 / 第 4587-4609 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4610-4632 / 第 4610-4632 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4633-4652 / 第 4633-4652 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_13-xpu
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4653-4673 / 第 4653-4673 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_13-xpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_13-xpu-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4674-4696 / 第 4674-4696 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DESIRED_PYTHON: "3.13"
      build_name: wheel-py3_13-xpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_13t-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4697-4719 / 第 4697-4719 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13t"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 4720-4742 / 第 4720-4742 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4743-4765 / 第 4743-4765 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4766-4788 / 第 4766-4788 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_13t-cpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4789-4811 / 第 4789-4811 行

````yaml
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_13t-cpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_13t-cpu-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.4xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13t"
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4812-4830 / 第 4812-4830 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4831-4853 / 第 4831-4853 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4854-4876 / 第 4854-4876 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4877-4893 / 第 4877-4893 行

````yaml
        with:
          name: wheel-py3_13t-cpu
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4894-4913 / 第 4894-4913 行

````yaml
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_13t-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_13t-cpu-test
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DESIRED_PYTHON: "3.13t"
      build_name: wheel-py3_13t-cpu
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4914-4936 / 第 4914-4936 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_13t-cuda12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13t"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 4937-4958 / 第 4937-4958 行

````yaml
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 4959-4980 / 第 4959-4980 行

````yaml
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 4981-5002 / 第 4981-5002 行

````yaml
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5003-5024 / 第 5003-5024 行

````yaml
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_13t-cuda12_6
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5026-5048 / 第 5026-5048 行

````yaml
  wheel-py3_13t-cuda12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_13t-cuda12_6-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13t"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5049-5070 / 第 5049-5070 行

````yaml
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5071-5093 / 第 5071-5093 行

````yaml
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5094-5115 / 第 5094-5115 行

````yaml
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_13t-cuda12_6
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5116-5136 / 第 5116-5136 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_13t-cuda12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_13t-cuda12_6-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5137-5157 / 第 5137-5157 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.13t"
      build_name: wheel-py3_13t-cuda12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_13t-cuda13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 5158-5177 / 第 5158-5177 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13t"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5178-5196 / 第 5178-5196 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5197-5219 / 第 5197-5219 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5220-5241 / 第 5220-5241 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5242-5262 / 第 5242-5262 行

````yaml
        with:
          name: wheel-py3_13t-cuda13_0
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_13t-cuda13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_13t-cuda13_0-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5263-5282 / 第 5263-5282 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13t"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5283-5305 / 第 5283-5305 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5306-5328 / 第 5306-5328 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5329-5348 / 第 5329-5348 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_13t-cuda13_0
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5349-5369 / 第 5349-5369 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_13t-cuda13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_13t-cuda13_0-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5370-5390 / 第 5370-5390 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.13t"
      build_name: wheel-py3_13t-cuda13_0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_13t-cuda13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 5391-5410 / 第 5391-5410 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13t"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5411-5429 / 第 5411-5429 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5430-5452 / 第 5430-5452 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5453-5474 / 第 5453-5474 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5475-5495 / 第 5475-5495 行

````yaml
        with:
          name: wheel-py3_13t-cuda13_2
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_13t-cuda13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_13t-cuda13_2-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5496-5515 / 第 5496-5515 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13t"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5516-5538 / 第 5516-5538 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5539-5561 / 第 5539-5561 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5562-5581 / 第 5562-5581 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_13t-cuda13_2
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5582-5602 / 第 5582-5602 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_13t-cuda13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_13t-cuda13_2-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5603-5623 / 第 5603-5623 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.13t"
      build_name: wheel-py3_13t-cuda13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_13t-xpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 5624-5643 / 第 5624-5643 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13t"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: intel-cmplr-lib-rt==2025.3.2 | intel-cmplr-lib-ur==2025.3.2 | intel-cmplr-lic-rt==2025.3.2 | intel-sycl-rt==2025.3.2 | oneccl-devel==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | oneccl==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | impi-rt==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | onemkl-license==2025.3.1 | onemkl-sycl-blas==2025.3.1 | onemkl-sycl-dft==2025.3.1 | onemkl-sycl-lapack==2025.3.1 | onemkl-sycl-rng==2025.3.1 | onemkl-sycl-sparse==2025.3.1 | dpcpp-cpp-rt==2025.3.2 | intel-opencl-rt==2025.3.2 | mkl==2025.3.1 | intel-openmp==2025.3.2 | tbb==2022.3.1 | tcmlib==1.4.1 | umf==1.0.3 | intel-pti==0.16.0
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5644-5662 / 第 5644-5662 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5663-5685 / 第 5663-5685 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5686-5707 / 第 5686-5707 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5708-5728 / 第 5708-5728 行

````yaml
        with:
          name: wheel-py3_13t-xpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_13t-xpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_13t-xpu-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5729-5747 / 第 5729-5747 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.4xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.13t"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5748-5770 / 第 5748-5770 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5771-5793 / 第 5771-5793 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5794-5813 / 第 5794-5813 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_13t-xpu
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5814-5834 / 第 5814-5834 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_13t-xpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_13t-xpu-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5835-5857 / 第 5835-5857 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DESIRED_PYTHON: "3.13t"
      build_name: wheel-py3_13t-xpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_14-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5858-5880 / 第 5858-5880 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 5881-5903 / 第 5881-5903 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5904-5926 / 第 5904-5926 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5927-5949 / 第 5927-5949 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_14-cpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5950-5972 / 第 5950-5972 行

````yaml
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_14-cpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_14-cpu-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.4xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14"
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 5973-5991 / 第 5973-5991 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 5992-6014 / 第 5992-6014 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6015-6037 / 第 6015-6037 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6038-6054 / 第 6038-6054 行

````yaml
        with:
          name: wheel-py3_14-cpu
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6055-6074 / 第 6055-6074 行

````yaml
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_14-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_14-cpu-test
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DESIRED_PYTHON: "3.14"
      build_name: wheel-py3_14-cpu
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6075-6097 / 第 6075-6097 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_14-cuda12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6098-6119 / 第 6098-6119 行

````yaml
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 6120-6141 / 第 6120-6141 行

````yaml
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6142-6163 / 第 6142-6163 行

````yaml
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6164-6185 / 第 6164-6185 行

````yaml
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_14-cuda12_6
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6187-6209 / 第 6187-6209 行

````yaml
  wheel-py3_14-cuda12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_14-cuda12_6-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6210-6231 / 第 6210-6231 行

````yaml
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6232-6254 / 第 6232-6254 行

````yaml
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6255-6276 / 第 6255-6276 行

````yaml
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_14-cuda12_6
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6277-6297 / 第 6277-6297 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_14-cuda12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_14-cuda12_6-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6298-6318 / 第 6298-6318 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.14"
      build_name: wheel-py3_14-cuda12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_14-cuda13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 6319-6338 / 第 6319-6338 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6339-6357 / 第 6339-6357 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6358-6380 / 第 6358-6380 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6381-6402 / 第 6381-6402 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6403-6423 / 第 6403-6423 行

````yaml
        with:
          name: wheel-py3_14-cuda13_0
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_14-cuda13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_14-cuda13_0-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6424-6443 / 第 6424-6443 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6444-6466 / 第 6444-6466 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6467-6489 / 第 6467-6489 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6490-6509 / 第 6490-6509 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_14-cuda13_0
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6510-6530 / 第 6510-6530 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_14-cuda13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_14-cuda13_0-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6531-6551 / 第 6531-6551 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.14"
      build_name: wheel-py3_14-cuda13_0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_14-cuda13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 6552-6571 / 第 6552-6571 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6572-6590 / 第 6572-6590 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6591-6613 / 第 6591-6613 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6614-6635 / 第 6614-6635 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6636-6656 / 第 6636-6656 行

````yaml
        with:
          name: wheel-py3_14-cuda13_2
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_14-cuda13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_14-cuda13_2-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6657-6676 / 第 6657-6676 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6677-6699 / 第 6677-6699 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6700-6722 / 第 6700-6722 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6723-6742 / 第 6723-6742 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_14-cuda13_2
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6743-6763 / 第 6743-6763 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_14-cuda13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_14-cuda13_2-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6764-6784 / 第 6764-6784 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.14"
      build_name: wheel-py3_14-cuda13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_14-xpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 6785-6804 / 第 6785-6804 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: intel-cmplr-lib-rt==2025.3.2 | intel-cmplr-lib-ur==2025.3.2 | intel-cmplr-lic-rt==2025.3.2 | intel-sycl-rt==2025.3.2 | oneccl-devel==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | oneccl==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | impi-rt==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | onemkl-license==2025.3.1 | onemkl-sycl-blas==2025.3.1 | onemkl-sycl-dft==2025.3.1 | onemkl-sycl-lapack==2025.3.1 | onemkl-sycl-rng==2025.3.1 | onemkl-sycl-sparse==2025.3.1 | dpcpp-cpp-rt==2025.3.2 | intel-opencl-rt==2025.3.2 | mkl==2025.3.1 | intel-openmp==2025.3.2 | tbb==2022.3.1 | tcmlib==1.4.1 | umf==1.0.3 | intel-pti==0.16.0
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6805-6823 / 第 6805-6823 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6824-6846 / 第 6824-6846 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6847-6868 / 第 6847-6868 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6869-6889 / 第 6869-6889 行

````yaml
        with:
          name: wheel-py3_14-xpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_14-xpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_14-xpu-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6890-6908 / 第 6890-6908 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.4xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6909-6931 / 第 6909-6931 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6932-6954 / 第 6932-6954 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6955-6974 / 第 6955-6974 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_14-xpu
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 6975-6995 / 第 6975-6995 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_14-xpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_14-xpu-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 6996-7018 / 第 6996-7018 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DESIRED_PYTHON: "3.14"
      build_name: wheel-py3_14-xpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_14t-cpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7019-7041 / 第 7019-7041 行

````yaml
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14t"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: cuda-toolkit[nvrtc,cudart,cupti,cufft,curand,cusolver,cusparse,cufile,nvjitlink,nvtx]==13.0.2; platform_system == 'Linux' | nvidia-cublas>=13.1.0.3,<=13.1.1.3; platform_system == 'Linux' | cuda-bindings>=13.0.3,<14; platform_system == 'Linux' | nvidia-cudnn-cu13==9.20.0.48; platform_system == 'Linux' | nvidia-cusparselt-cu13==0.8.1; platform_system == 'Linux' | nvidia-nccl-cu13==2.29.7; platform_system == 'Linux' | nvidia-nvshmem-cu13==3.4.5; platform_system == 'Linux'
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 7042-7064 / 第 7042-7064 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7065-7087 / 第 7065-7087 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7088-7110 / 第 7088-7110 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_14t-cpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7111-7133 / 第 7111-7133 行

````yaml
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_14t-cpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_14t-cpu-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.4xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14t"
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7134-7152 / 第 7134-7152 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7153-7175 / 第 7153-7175 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7176-7198 / 第 7176-7198 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7199-7215 / 第 7199-7215 行

````yaml
        with:
          name: wheel-py3_14t-cpu
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7216-7235 / 第 7216-7235 行

````yaml
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_14t-cpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_14t-cpu-test
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      DESIRED_PYTHON: "3.14t"
      build_name: wheel-py3_14t-cpu
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7236-7258 / 第 7236-7258 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_14t-cuda12_6-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14t"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7259-7280 / 第 7259-7280 行

````yaml
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 7281-7302 / 第 7281-7302 行

````yaml
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7303-7324 / 第 7303-7324 行

````yaml
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7325-7346 / 第 7325-7346 行

````yaml
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: wheel-py3_14t-cuda12_6
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7348-7370 / 第 7348-7370 行

````yaml
  wheel-py3_14t-cuda12_6-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_14t-cuda12_6-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14t"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7371-7392 / 第 7371-7392 行

````yaml
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7393-7415 / 第 7393-7415 行

````yaml
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7416-7437 / 第 7416-7437 行

````yaml
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_14t-cuda12_6
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7438-7458 / 第 7438-7458 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_14t-cuda12_6-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_14t-cuda12_6-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7459-7479 / 第 7459-7479 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.14t"
      build_name: wheel-py3_14t-cuda12_6
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_14t-cuda13_0-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 7480-7499 / 第 7480-7499 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14t"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7500-7518 / 第 7500-7518 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7519-7541 / 第 7519-7541 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7542-7563 / 第 7542-7563 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7564-7584 / 第 7564-7584 行

````yaml
        with:
          name: wheel-py3_14t-cuda13_0
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_14t-cuda13_0-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_14t-cuda13_0-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7585-7604 / 第 7585-7604 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14t"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7605-7627 / 第 7605-7627 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7628-7650 / 第 7628-7650 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7651-7670 / 第 7651-7670 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_14t-cuda13_0
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7671-7691 / 第 7671-7691 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_14t-cuda13_0-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_14t-cuda13_0-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7692-7712 / 第 7692-7712 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.14t"
      build_name: wheel-py3_14t-cuda13_0
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_14t-cuda13_2-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 7713-7732 / 第 7713-7732 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14t"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7733-7751 / 第 7733-7751 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7752-7774 / 第 7752-7774 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7775-7796 / 第 7775-7796 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7797-7817 / 第 7797-7817 行

````yaml
        with:
          name: wheel-py3_14t-cuda13_2
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_14t-cuda13_2-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_14t-cuda13_2-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7818-7837 / 第 7818-7837 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14t"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7838-7860 / 第 7838-7860 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7861-7883 / 第 7861-7883 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7884-7903 / 第 7884-7903 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_14t-cuda13_2
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7904-7924 / 第 7904-7924 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_14t-cuda13_2-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_14t-cuda13_2-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7925-7945 / 第 7925-7945 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      DESIRED_PYTHON: "3.14t"
      build_name: wheel-py3_14t-cuda13_2
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  wheel-py3_14t-xpu-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 7946-7965 / 第 7946-7965 行

````yaml
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14t"
      PYTORCH_EXTRA_INSTALL_REQUIREMENTS: intel-cmplr-lib-rt==2025.3.2 | intel-cmplr-lib-ur==2025.3.2 | intel-cmplr-lic-rt==2025.3.2 | intel-sycl-rt==2025.3.2 | oneccl-devel==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | oneccl==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | impi-rt==2021.17.2; platform_system == 'Linux' and platform_machine == 'x86_64' | onemkl-license==2025.3.1 | onemkl-sycl-blas==2025.3.1 | onemkl-sycl-dft==2025.3.1 | onemkl-sycl-lapack==2025.3.1 | onemkl-sycl-rng==2025.3.1 | onemkl-sycl-sparse==2025.3.1 | dpcpp-cpp-rt==2025.3.2 | intel-opencl-rt==2025.3.2 | mkl==2025.3.1 | intel-openmp==2025.3.2 | tbb==2022.3.1 | tcmlib==1.4.1 | umf==1.0.3 | intel-pti==0.16.0
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 7966-7984 / 第 7966-7984 行

````yaml
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 7985-8007 / 第 7985-8007 行

````yaml
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 8008-8029 / 第 8008-8029 行

````yaml
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 8030-8050 / 第 8030-8050 行

````yaml
        with:
          name: wheel-py3_14t-xpu
          retention-days: 14
          if-no-files-found: error
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1

  wheel-py3_14t-xpu-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_14t-xpu-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 8051-8069 / 第 8051-8069 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.4xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      SKIP_ALL_TESTS: 1
      DESIRED_PYTHON: "3.14t"
    steps:
      - name: Display EC2 information
        shell: bash
        run: |
          set -euo pipefail
          function get_ec2_metadata() {
            # Pulled from instance metadata endpoint for EC2
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 8070-8092 / 第 8070-8092 行

````yaml
            # see https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
            category=$1
            curl -H "X-aws-ec2-metadata-token: $(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 30")" -fsSL "http://169.254.169.254/latest/meta-data/${category}"
          }
          echo "ami-id: $(get_ec2_metadata ami-id)"
          echo "instance-id: $(get_ec2_metadata instance-id)"
          echo "instance-type: $(get_ec2_metadata instance-type)"
          echo "system info $(uname -a)"
      - name: "[FB EMPLOYEES] Enable SSH (Click me for login details)"
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        continue-on-error: true
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
      - name: Enable git long paths and symlinks on Windows and disable fsmonitor daemon
        shell: bash
        run: |
          git config --global core.longpaths true
          git config --global core.symlinks true

          # https://git-scm.com/docs/git-fsmonitor--daemon.  The daemon could lock
          # the directory on Windows and prevent GHA from checking out as reported
          # in https://github.com/actions/checkout/issues/1018
          git config --global core.fsmonitor false
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 8093-8115 / 第 8093-8115 行

````yaml
      # Needed for binary builds, see: https://github.com/pytorch/pytorch/issues/73339#issuecomment-1058981560
      - name: Enable long paths on Windows
        shell: powershell
        run: |
          Set-ItemProperty -Path "HKLM:\\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1
      # Since it's just a defensive command, the workflow should continue even the command fails. This step can be
      # removed once Windows Defender is removed from the AMI
      - name: Disables Windows Defender scheduled and real-time scanning for files in directories used by PyTorch
        continue-on-error: true
        shell: powershell
        run: |
          Add-MpPreference -ExclusionPath $(Get-Location).tostring(),$Env:TEMP -ErrorAction Ignore
          # Let's both exclude the path and disable Windows Defender completely just to be sure
          # that it doesn't interfere
          Set-MpPreference -DisableRealtimeMonitoring $True -ErrorAction Ignore
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          submodules: recursive
          show-progress: false
      - name: Clean PyTorch checkout
        run: |
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 8116-8135 / 第 8116-8135 行

````yaml
          # Remove any artifacts from the previous checkouts
          git clean -fxd
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
      - name: Populate binary env
        shell: bash
        run: |
          echo "BINARY_ENV_FILE=${RUNNER_TEMP}/env" >> "${GITHUB_ENV}"
          echo "PYTORCH_FINAL_PACKAGE_DIR=${RUNNER_TEMP}/artifacts" >> "${GITHUB_ENV}"
          echo "WIN_PACKAGE_WORK_DIR=${RUNNER_TEMP}"
      - uses: actions/download-artifact@v4.1.7
        name: Download Build Artifacts
        with:
          name: wheel-py3_14t-xpu
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 8136-8156 / 第 8136-8156 行

````yaml
      - name: Test PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_test.sh"
      - name: Wait until all sessions have drained
        shell: powershell
        if: always()
        timeout-minutes: 120
        run: |
          .github\scripts\wait_for_ssh_to_drain.ps1
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  wheel-py3_14t-xpu-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: wheel-py3_14t-xpu-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 8157-8178 / 第 8157-8178 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: wheel
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: xpu
      GPU_ARCH_TYPE: xpu
      DESIRED_PYTHON: "3.14t"
      build_name: wheel-py3_14t-xpu
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  libtorch-cpu-shared-with-deps-release-extract:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_10-cpu-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge"
    timeout-minutes: 60
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 8179-8193 / 第 8179-8193 行

````yaml
    env:
      DESIRED_CUDA: cpu
      LIBTORCH_VARIANT: shared-with-deps
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          sparse-checkout: .ci/libtorch/
          show-progress: false
      - uses: actions/download-artifact@v4.1.7
        name: Download Wheel Artifact
        with:
          name: wheel-py3_10-cpu
          path: "${{ runner.temp }}/wheel_artifact/"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 8194-8213 / 第 8194-8213 行

````yaml
      - name: Extract libtorch from wheel
        shell: bash
        run: |
          set -eux
          mkdir -p "${{ runner.temp }}/libtorch_output"
          python3 .ci/libtorch/extract_libtorch_from_wheel.py \
            --wheel-dir "${{ runner.temp }}/wheel_artifact" \
            --output-dir "${{ runner.temp }}/libtorch_output" \
            --platform windows \
            --desired-cuda "$DESIRED_CUDA" \
            --libtorch-variant "$LIBTORCH_VARIANT"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: libtorch-cpu-shared-with-deps-release
          retention-days: 14
          if-no-files-found: error
          path: "${{ runner.temp }}/libtorch_output/"
  libtorch-cpu-shared-with-deps-release-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 8214-8235 / 第 8214-8235 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: libtorch-cpu-shared-with-deps-release-extract
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: libtorch
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      LIBTORCH_CONFIG: release
      LIBTORCH_VARIANT: shared-with-deps
      build_name: libtorch-cpu-shared-with-deps-release
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  libtorch-cuda12_6-shared-with-deps-release-extract:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_10-cuda12_6-build
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 8236-8253 / 第 8236-8253 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge"
    timeout-minutes: 60
    env:
      DESIRED_CUDA: cu126
      LIBTORCH_VARIANT: shared-with-deps
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          sparse-checkout: .ci/libtorch/
          show-progress: false
      - uses: actions/download-artifact@v4.1.7
        name: Download Wheel Artifact
        with:
          name: wheel-py3_10-cuda12_6
          path: "${{ runner.temp }}/wheel_artifact/"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 8254-8273 / 第 8254-8273 行

````yaml
      - name: Extract libtorch from wheel
        shell: bash
        run: |
          set -eux
          mkdir -p "${{ runner.temp }}/libtorch_output"
          python3 .ci/libtorch/extract_libtorch_from_wheel.py \
            --wheel-dir "${{ runner.temp }}/wheel_artifact" \
            --output-dir "${{ runner.temp }}/libtorch_output" \
            --platform windows \
            --desired-cuda "$DESIRED_CUDA" \
            --libtorch-variant "$LIBTORCH_VARIANT"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: libtorch-cuda12_6-shared-with-deps-release
          retention-days: 14
          if-no-files-found: error
          path: "${{ runner.temp }}/libtorch_output/"
  libtorch-cuda12_6-shared-with-deps-release-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 8274-8296 / 第 8274-8296 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: libtorch-cuda12_6-shared-with-deps-release-extract
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: libtorch
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      LIBTORCH_CONFIG: release
      LIBTORCH_VARIANT: shared-with-deps
      build_name: libtorch-cuda12_6-shared-with-deps-release
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  libtorch-cuda13_0-shared-with-deps-release-extract:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_10-cuda13_0-build
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 8297-8314 / 第 8297-8314 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge"
    timeout-minutes: 60
    env:
      DESIRED_CUDA: cu130
      LIBTORCH_VARIANT: shared-with-deps
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          sparse-checkout: .ci/libtorch/
          show-progress: false
      - uses: actions/download-artifact@v4.1.7
        name: Download Wheel Artifact
        with:
          name: wheel-py3_10-cuda13_0
          path: "${{ runner.temp }}/wheel_artifact/"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 8315-8334 / 第 8315-8334 行

````yaml
      - name: Extract libtorch from wheel
        shell: bash
        run: |
          set -eux
          mkdir -p "${{ runner.temp }}/libtorch_output"
          python3 .ci/libtorch/extract_libtorch_from_wheel.py \
            --wheel-dir "${{ runner.temp }}/wheel_artifact" \
            --output-dir "${{ runner.temp }}/libtorch_output" \
            --platform windows \
            --desired-cuda "$DESIRED_CUDA" \
            --libtorch-variant "$LIBTORCH_VARIANT"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: libtorch-cuda13_0-shared-with-deps-release
          retention-days: 14
          if-no-files-found: error
          path: "${{ runner.temp }}/libtorch_output/"
  libtorch-cuda13_0-shared-with-deps-release-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 8335-8357 / 第 8335-8357 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: libtorch-cuda13_0-shared-with-deps-release-extract
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: libtorch
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      LIBTORCH_CONFIG: release
      LIBTORCH_VARIANT: shared-with-deps
      build_name: libtorch-cuda13_0-shared-with-deps-release
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  libtorch-cuda13_2-shared-with-deps-release-extract:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - wheel-py3_10-cuda13_2-build
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 8358-8375 / 第 8358-8375 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge"
    timeout-minutes: 60
    env:
      DESIRED_CUDA: cu132
      LIBTORCH_VARIANT: shared-with-deps
    steps:
      - name: Checkout PyTorch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
          sparse-checkout: .ci/libtorch/
          show-progress: false
      - uses: actions/download-artifact@v4.1.7
        name: Download Wheel Artifact
        with:
          name: wheel-py3_10-cuda13_2
          path: "${{ runner.temp }}/wheel_artifact/"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 8376-8395 / 第 8376-8395 行

````yaml
      - name: Extract libtorch from wheel
        shell: bash
        run: |
          set -eux
          mkdir -p "${{ runner.temp }}/libtorch_output"
          python3 .ci/libtorch/extract_libtorch_from_wheel.py \
            --wheel-dir "${{ runner.temp }}/wheel_artifact" \
            --output-dir "${{ runner.temp }}/libtorch_output" \
            --platform windows \
            --desired-cuda "$DESIRED_CUDA" \
            --libtorch-variant "$LIBTORCH_VARIANT"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: libtorch-cuda13_2-shared-with-deps-release
          retention-days: 14
          if-no-files-found: error
          path: "${{ runner.temp }}/libtorch_output/"
  libtorch-cuda13_2-shared-with-deps-release-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 8396-8408 / 第 8396-8408 行

````yaml
    permissions:
      id-token: write
      contents: read
    needs: libtorch-cuda13_2-shared-with-deps-release-extract
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: libtorch
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      LIBTORCH_CONFIG: release
      LIBTORCH_VARIANT: shared-with-deps
      build_name: libtorch-cuda13_2-shared-with-deps-release
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 8409-8414 / 第 8409-8414 行

````yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`.
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
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `pytorch/test-infra/.github/actions/setup-ssh@main`, `actions/checkout@v4`, `./.github/workflows/_binary-upload.yml`
- Inline commands / 内联命令: `echo`, `set`, `curl`, `git`, `Set-ItemProperty`, `Add-MpPreference`, `Set-MpPreference`, `"${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"`, ...
- Environment variables / 环境变量: `NOT`, `EDIT`, `MANUALLY`, `NOTE`, `ALPINE_IMAGE`, `AWS_DEFAULT_REGION`, `BUILD_ENVIRONMENT`, `GITHUB_TOKEN`, `PR_NUMBER`, `SHA1`, ...
- Named jobs or sections / 命名作业或章节: `on`, `push`, `branches`, `tags`, `workflow_dispatch`, `env`, `concurrency`, `jobs`, `get-label-type`, `with`, ...
