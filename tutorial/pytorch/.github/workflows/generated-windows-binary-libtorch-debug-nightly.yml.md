# generated-windows-binary-libtorch-debug-nightly.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/generated-windows-binary-libtorch-debug-nightly.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-20 / 第 1-20 行

````yaml
# @generated DO NOT EDIT MANUALLY

# Template is at:    .github/templates/windows_binary_build_workflow.yml.j2
# Generation script: .github/scripts/generate_ci_workflows.py
name: windows-binary-libtorch-debug

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
  workflow_dispatch:

env:
````

- EN: This section declares workflow triggers or event bindings; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；控制令牌权限范围或环境变量以实现安全执行。

### Lines 21-43 / 第 21-43 行

````yaml
  # Needed for conda builds
  ALPINE_IMAGE: "308535385114.dkr.ecr.us-east-1.amazonaws.com/tool/alpine"
  AWS_DEFAULT_REGION: us-east-1
  BUILD_ENVIRONMENT: windows-binary-libtorch-debug
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  PR_NUMBER: ${{ github.event.pull_request.number }}
  SHA1: ${{ github.event.pull_request.head.sha || github.sha }}
  SKIP_ALL_TESTS: 1
  OS: windows
concurrency:
  group: windows-binary-libtorch-debug-${{ github.event.pull_request.number || github.ref_name }}-${{ github.ref_type == 'branch' && github.sha }}-${{ github.event_name == 'workflow_dispatch' }}
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

### Lines 44-65 / 第 44-65 行

````yaml
  libtorch-cpu-shared-with-deps-debug-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: libtorch
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      LIBTORCH_CONFIG: debug
      LIBTORCH_VARIANT: shared-with-deps
      # This is a dummy value for libtorch to work correctly with our batch scripts
      # without this value pip does not get installed for some reason
      DESIRED_PYTHON: "3.10"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
````

- EN: This section reuses actions and step building blocks such as named actions; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 66-88 / 第 66-88 行

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

### Lines 89-107 / 第 89-107 行

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

### Lines 108-129 / 第 108-129 行

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

### Lines 130-151 / 第 130-151 行

````yaml
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: libtorch-cpu-shared-with-deps-debug
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

### Lines 153-173 / 第 153-173 行

````yaml
  libtorch-cpu-shared-with-deps-debug-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - libtorch-cpu-shared-with-deps-debug-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.4xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: libtorch
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      SKIP_ALL_TESTS: 1
      LIBTORCH_CONFIG: debug
      LIBTORCH_VARIANT: shared-with-deps
      # This is a dummy value for libtorch to work correctly with our batch scripts
      # without this value pip does not get installed for some reason
      DESIRED_PYTHON: "3.10"
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 174-192 / 第 174-192 行

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

### Lines 193-215 / 第 193-215 行

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

### Lines 216-238 / 第 216-238 行

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

### Lines 239-255 / 第 239-255 行

````yaml
        with:
          name: libtorch-cpu-shared-with-deps-debug
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

### Lines 256-276 / 第 256-276 行

````yaml
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  libtorch-cpu-shared-with-deps-debug-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: libtorch-cpu-shared-with-deps-debug-test
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: libtorch
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cpu
      GPU_ARCH_TYPE: cpu
      LIBTORCH_CONFIG: debug
      LIBTORCH_VARIANT: shared-with-deps
      # This is a dummy value for libtorch to work correctly with our batch scripts
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 277-294 / 第 277-294 行

````yaml
      # without this value pip does not get installed for some reason
      DESIRED_PYTHON: "3.10"
      build_name: libtorch-cpu-shared-with-deps-debug
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  libtorch-cuda12_6-shared-with-deps-debug-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: libtorch
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 295-314 / 第 295-314 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      LIBTORCH_CONFIG: debug
      LIBTORCH_VARIANT: shared-with-deps
      # This is a dummy value for libtorch to work correctly with our batch scripts
      # without this value pip does not get installed for some reason
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

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 315-333 / 第 315-333 行

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

### Lines 334-356 / 第 334-356 行

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

### Lines 357-378 / 第 357-378 行

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

### Lines 379-399 / 第 379-399 行

````yaml
        with:
          name: libtorch-cuda12_6-shared-with-deps-debug
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

  libtorch-cuda12_6-shared-with-deps-debug-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - libtorch-cuda12_6-shared-with-deps-debug-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 400-422 / 第 400-422 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: libtorch
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      LIBTORCH_CONFIG: debug
      LIBTORCH_VARIANT: shared-with-deps
      # This is a dummy value for libtorch to work correctly with our batch scripts
      # without this value pip does not get installed for some reason
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

### Lines 423-444 / 第 423-444 行

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

### Lines 445-467 / 第 445-467 行

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

### Lines 468-489 / 第 468-489 行

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
          name: libtorch-cuda12_6-shared-with-deps-debug
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 490-510 / 第 490-510 行

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
  libtorch-cuda12_6-shared-with-deps-debug-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: libtorch-cuda12_6-shared-with-deps-debug-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 511-530 / 第 511-530 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: libtorch
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu126
      GPU_ARCH_VERSION: "12.6"
      GPU_ARCH_TYPE: cuda
      LIBTORCH_CONFIG: debug
      LIBTORCH_VARIANT: shared-with-deps
      # This is a dummy value for libtorch to work correctly with our batch scripts
      # without this value pip does not get installed for some reason
      DESIRED_PYTHON: "3.10"
      build_name: libtorch-cuda12_6-shared-with-deps-debug
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 531-553 / 第 531-553 行

````yaml
  libtorch-cuda13_0-shared-with-deps-debug-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: libtorch
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      LIBTORCH_CONFIG: debug
      LIBTORCH_VARIANT: shared-with-deps
      # This is a dummy value for libtorch to work correctly with our batch scripts
      # without this value pip does not get installed for some reason
      DESIRED_PYTHON: "3.10"
    steps:
      # NOTE: These environment variables are put here so that they can be applied on every job equally
      #       They are also here because setting them at a workflow level doesn't give us access to the
      #       runner.temp variable, which we need.
````

- EN: This section reuses actions and step building blocks such as named actions; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 554-576 / 第 554-576 行

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

### Lines 577-595 / 第 577-595 行

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

### Lines 596-617 / 第 596-617 行

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

### Lines 618-639 / 第 618-639 行

````yaml
      - name: Build PyTorch binary
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_windows_build.sh"
      - uses: actions/upload-artifact@v4.4.0
        if: always()
        with:
          name: libtorch-cuda13_0-shared-with-deps-debug
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

### Lines 641-662 / 第 641-662 行

````yaml
  libtorch-cuda13_0-shared-with-deps-debug-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - libtorch-cuda13_0-shared-with-deps-debug-build
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: libtorch
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      LIBTORCH_CONFIG: debug
      LIBTORCH_VARIANT: shared-with-deps
      # This is a dummy value for libtorch to work correctly with our batch scripts
      # without this value pip does not get installed for some reason
      DESIRED_PYTHON: "3.10"
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 663-681 / 第 663-681 行

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

### Lines 682-704 / 第 682-704 行

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

### Lines 705-727 / 第 705-727 行

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

### Lines 728-744 / 第 728-744 行

````yaml
        with:
          name: libtorch-cuda13_0-shared-with-deps-debug
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

### Lines 745-766 / 第 745-766 行

````yaml
      - name: Kill active ssh sessions if still around (Useful if workflow was cancelled)
        shell: powershell
        if: always()
        run: |
          .github\scripts\kill_active_ssh_sessions.ps1
  libtorch-cuda13_0-shared-with-deps-debug-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: libtorch-cuda13_0-shared-with-deps-debug-test
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: libtorch
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu130
      GPU_ARCH_VERSION: "13.0"
      GPU_ARCH_TYPE: cuda
      LIBTORCH_CONFIG: debug
      LIBTORCH_VARIANT: shared-with-deps
      # This is a dummy value for libtorch to work correctly with our batch scripts
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 767-784 / 第 767-784 行

````yaml
      # without this value pip does not get installed for some reason
      DESIRED_PYTHON: "3.10"
      build_name: libtorch-cuda13_0-shared-with-deps-debug
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      R2_ACCOUNT_ID: ${{ secrets.R2_ACCOUNT_ID }}
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
    uses: ./.github/workflows/_binary-upload.yml
  libtorch-cuda13_2-shared-with-deps-debug-build:
    if: ${{ github.repository_owner == 'pytorch' }}
    needs: get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.12xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: libtorch
      # TODO: This is a legacy variable that we eventually want to get rid of in
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_binary-upload.yml`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 785-804 / 第 785-804 行

````yaml
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      LIBTORCH_CONFIG: debug
      LIBTORCH_VARIANT: shared-with-deps
      # This is a dummy value for libtorch to work correctly with our batch scripts
      # without this value pip does not get installed for some reason
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

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 805-823 / 第 805-823 行

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

### Lines 824-846 / 第 824-846 行

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

### Lines 847-868 / 第 847-868 行

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

### Lines 869-889 / 第 869-889 行

````yaml
        with:
          name: libtorch-cuda13_2-shared-with-deps-debug
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

  libtorch-cuda13_2-shared-with-deps-debug-test:  # Testing
    if: ${{ github.repository_owner == 'pytorch' }}
    needs:
      - libtorch-cuda13_2-shared-with-deps-debug-build
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 890-912 / 第 890-912 行

````yaml
      - get-label-type
    runs-on: "${{ needs.get-label-type.outputs.label-type }}windows.g4dn.xlarge"
    timeout-minutes: 360
    env:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: libtorch
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      SKIP_ALL_TESTS: 1
      LIBTORCH_CONFIG: debug
      LIBTORCH_VARIANT: shared-with-deps
      # This is a dummy value for libtorch to work correctly with our batch scripts
      # without this value pip does not get installed for some reason
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

### Lines 913-934 / 第 913-934 行

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

### Lines 935-957 / 第 935-957 行

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

### Lines 958-979 / 第 958-979 行

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
          name: libtorch-cuda13_2-shared-with-deps-debug
          path: "${{ env.PYTORCH_FINAL_PACKAGE_DIR }}"
      - name: Populate binary env
        shell: bash
        run: |
          "${PYTORCH_ROOT}/.ci/pytorch/binary_populate_env.sh"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 980-1000 / 第 980-1000 行

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
  libtorch-cuda13_2-shared-with-deps-debug-upload:  # Uploading
    if: ${{ github.repository_owner == 'pytorch' }}
    permissions:
      id-token: write
      contents: read
    needs: libtorch-cuda13_2-shared-with-deps-debug-test
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 1001-1014 / 第 1001-1014 行

````yaml
    with:
      PYTORCH_ROOT: ${{ github.workspace }}
      PACKAGE_TYPE: libtorch
      # TODO: This is a legacy variable that we eventually want to get rid of in
      #       favor of GPU_ARCH_VERSION
      DESIRED_CUDA: cu132
      GPU_ARCH_VERSION: "13.2"
      GPU_ARCH_TYPE: cuda
      LIBTORCH_CONFIG: debug
      LIBTORCH_VARIANT: shared-with-deps
      # This is a dummy value for libtorch to work correctly with our batch scripts
      # without this value pip does not get installed for some reason
      DESIRED_PYTHON: "3.10"
      build_name: libtorch-cuda13_2-shared-with-deps-debug
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 1015-1020 / 第 1015-1020 行

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
