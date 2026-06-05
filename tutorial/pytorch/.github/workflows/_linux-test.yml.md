# _linux-test.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/_linux-test.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-17 / 第 1-17 行

````yaml
name: linux-test

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
      docker-image:
        required: true
        type: string
        description: Docker image to run in.
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 18-40 / 第 18-40 行

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
      tests-to-include:
        required: false
        type: string
        default: ""
        description: Space-separated tests to include (empty string implies default list)
      use-gha:
        required: false
        type: string
        default: ""
        description: If set to any value, upload to GHA. Otherwise upload to S3.
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-test.yml`.
- CN: 该部分描述 `.github/workflows/_linux-test.yml` 的仓库自动化行为。

### Lines 41-62 / 第 41-62 行

````yaml
      dashboard-tag:
        required: false
        type: string
        default: ""
      s3-bucket:
        description: S3 bucket to download artifact
        required: false
        type: string
        default: "gha-artifacts"
      aws-role-to-assume:
        description: role to assume for downloading artifacts
        required: false
        type: string
        default: ""
      disable-monitor:
        description: |
          [Experimental] Disable utilization monitoring for tests.
          Currently, by default we disable the monitor job and only look for specific tests,
          since we are investigating the behaviour of the monitor script with different tests.
        required: false
        type: boolean
        default: false
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-test.yml`.
- CN: 该部分描述 `.github/workflows/_linux-test.yml` 的仓库自动化行为。

### Lines 63-84 / 第 63-84 行

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
      use-arc:
        required: false
        type: boolean
        default: false
        description: If true, use ARC (OSDC) runner path instead of EC2.
      python-version:
        required: false
        type: string
        default: ""
        description: Python version to use for the OSDC test.
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-test.yml`.
- CN: 该部分描述 `.github/workflows/_linux-test.yml` 的仓库自动化行为。

### Lines 85-107 / 第 85-107 行

````yaml
      compiler:
        required: false
        type: string
        default: ""
        description: Compiler to use for the OSDC test.
      cuda-version:
        required: false
        type: string
        default: ""
        description: CUDA version to use for the OSDC test.
      export-profiler-trace:
        description: |
          If set to "1", export Chrome profiler traces from performance benchmarks.
        required: false
        type: string
        default: ""
      enable-torch-trace:
        description: |
          If set to "1", enable TORCH_TRACE structured logging and collect tlparse output.
        required: false
        type: string
        default: ""
    secrets:
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-test.yml`.
- CN: 该部分描述 `.github/workflows/_linux-test.yml` 的仓库自动化行为。

### Lines 108-127 / 第 108-127 行

````yaml
      HUGGING_FACE_HUB_TOKEN:
        required: false
        description: |
          HF Auth token to avoid rate limits when downloading models or datasets from hub
      VLLM_TEST_HUGGING_FACE_TOKEN:
        required: false
        description: |
          HF Auth token to test vllm
      SCRIBE_GRAPHQL_ACCESS_TOKEN:
        required: false
        description: |
          FB app token to write to scribe endpoint

env:
  GIT_DEFAULT_BRANCH: ${{ github.event.repository.default_branch }}

jobs:
  test:
    # Don't run on forked repos or empty test matrix
    if: github.repository_owner == 'pytorch' && toJSON(fromJSON(inputs.test-matrix).include) != '[]' && !inputs.use-arc
````

- EN: This section lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 128-149 / 第 128-149 行

````yaml
    strategy:
      matrix: ${{ fromJSON(inputs.test-matrix) }}
      fail-fast: false
    environment: ${{ github.ref == 'refs/heads/main' && 'scribe-protected' || startsWith(github.ref, 'refs/heads/release/') && 'scribe-protected' || contains(github.event.pull_request.labels.*.name, 'ci-scribe') && 'scribe-pr' || '' }}
    runs-on: ${{ matrix.runner }}
    timeout-minutes: ${{ matrix.mem_leak_check == 'mem_leak_check' && 600 || inputs.timeout-minutes }}
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Setup SSH (Click me for login details)
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        if: ${{ !contains(matrix.runner, 'b200') && inputs.build-environment != 'linux-s390x-binary-manywheel' }}
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
          instructions: |
            All testing is done inside the container, to start an interactive session run:
              docker exec -it $(docker container ps --format '{{.ID}}') bash

      - name: Setup Linux
        id: setup-linux
        uses: pytorch/pytorch/.github/actions/setup-linux@main
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`, `pytorch/pytorch/.github/actions/setup-linux@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 150-172 / 第 150-172 行

````yaml
        with:
          python-version: ${{ inputs.python-version }}
          compiler: ${{ inputs.compiler }}
          cuda-version: ${{ inputs.cuda-version }}
          submodules: 'false'
          github-token: ${{ secrets.GITHUB_TOKEN }}

      - name: Check TPU Availability
        id: check-tpu
        uses: ./.github/actions/check-tpu
        if: inputs.build-environment != 'linux-s390x-binary-manywheel' && !contains(matrix.runner, 'b200')

      - name: Setup TPU docker flags
        id: setup-tpu-flags
        if: steps.check-tpu.outputs.has_tpu == 'true'
        shell: bash
        run: echo "TPU_DOCKER_FLAGS=--privileged --network=host -e PJRT_DEVICE=TPU -e TPU_SKIP_MDS_QUERY -e TPU_TOPOLOGY -e TPU_WORKER_ID -e TPU_TOPOLOGY_WRAP -e TPU_CHIPS_PER_HOST_BOUNDS -e TPU_ACCELERATOR_TYPE -e TPU_RUNTIME_METRICS_PORTS -e TPU_TOPOLOGY_ALT -e HOST_BOUNDS -e TPU_HOST_BOUNDS -e VBAR_CONTROL_SERVICE_URL -e CHIPS_PER_HOST_BOUNDS -e TPU_WORKER_HOSTNAMES" >> "$GITHUB_ENV"

      - name: Login to ECR
        if: inputs.build-environment != 'linux-s390x-binary-manywheel'
        uses: ./.github/actions/ecr-login
        with:
          aws-role-to-assume: ${{ inputs.aws-role-to-assume }}
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/check-tpu`, `./.github/actions/ecr-login`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 174-195 / 第 174-195 行

````yaml
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        if: inputs.build-environment != 'linux-s390x-binary-manywheel'
        with:
          docker-image-name: ${{ inputs.docker-image }}

      - name: Use following to pull public copy of the image
        id: print-ghcr-mirror
        if: inputs.build-environment != 'linux-s390x-binary-manywheel'
        env:
          ECR_DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
        shell: bash
        run: |
          tag=${ECR_DOCKER_IMAGE##*:}
          echo "docker pull ghcr.io/pytorch/ci-image:${tag/:/-}"

      - name: Pull docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        if: inputs.build-environment != 'linux-s390x-binary-manywheel'
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 197-217 / 第 197-217 行

````yaml
      - name: Check if in a container runner
        shell: bash
        id: check_container_runner
        run: echo "IN_CONTAINER_RUNNER=$(if [ -f /.inarc ] || [ -f /.incontainer ]; then echo true ; else echo false; fi)" >> "$GITHUB_OUTPUT"

      - name: Install nvidia driver, nvidia-docker runtime, set GPU_FLAG
        id: install-nvidia-driver
        uses: pytorch/test-infra/.github/actions/setup-nvidia@main
        with:
          driver-version: '580.82.07'
        if: ${{ !contains(matrix.runner, 'b200') }}

      - name: Setup GPU_FLAG for docker run
        id: setup-gpu-flag
        run: echo "GPU_FLAG=--gpus all -e NVIDIA_DRIVER_CAPABILITIES=all" >> "${GITHUB_ENV}"
        if: ${{ steps.install-nvidia-driver.outputs.has-nvidia == 'true' || contains(matrix.runner, 'b200') }}

      - name: Setup SCCACHE_SERVER_PORT environment for docker run when on container
        id: setup-sscache-port-flag
        run: echo "SCCACHE_SERVER_PORT_DOCKER_FLAG=-e SCCACHE_SERVER_PORT=$((RUNNER_UID + 4226))" >> "${GITHUB_ENV}"
        if: ${{ steps.check_container_runner.outputs.IN_CONTAINER_RUNNER == 'true' && !contains(matrix.runner, 'b200') }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-nvidia@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 219-240 / 第 219-240 行

````yaml
      - name: Start monitoring script
        id: monitor-script
        if: ${{ !inputs.disable-monitor }}
        shell: bash
        continue-on-error: true
        env:
          JOB_ID: ${{ steps.setup-linux.outputs.job-id }}
          JOB_NAME: ${{ steps.setup-linux.outputs.job-name }}
          WORKFLOW_NAME: ${{ github.workflow }}
          WORKFLOW_RUN_ID: ${{github.run_id}}
          MONITOR_LOG_INTERVAL: ${{ inputs.monitor-log-interval }}
          MONITOR_DATA_COLLECT_INTERVAL: ${{ inputs.monitor-data-collect-interval }}
        run: |
          uv run --no-project --with psutil==5.9.8 --with dataclasses_json==0.6.7 --with nvidia-ml-py==11.525.84 python -m tools.stats.monitor --log-interval "$MONITOR_LOG_INTERVAL" --data-collect-interval "$MONITOR_DATA_COLLECT_INTERVAL" > usage_log.txt 2>&1 &
          echo "monitor-script-pid=${!}" >> "${GITHUB_OUTPUT}"

      - name: Download build artifacts
        uses: ./.github/actions/download-build-artifacts
        with:
          name: ${{ inputs.build-environment }}
          s3-bucket: ${{ inputs.s3-bucket }}
          use-gha: ${{ inputs.use-gha }}
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/download-build-artifacts`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 242-257 / 第 242-257 行

````yaml
      - name: Download TD artifacts
        continue-on-error: true
        uses: ./.github/actions/download-td-artifacts

      - name: Download Windows torch wheel for cross-compilation
        if: matrix.win_torch_wheel_artifact != ''
        uses: seemethere/download-artifact-s3@1da556a7aa0a088e3153970611f6c432d58e80e6 # v4.2.0
        with:
          name: ${{ matrix.win_torch_wheel_artifact }}
          path: win-torch-wheel

      - name: Extract Windows wheel and setup CUDA libraries
        if: matrix.win_torch_wheel_artifact != ''
        shell: bash
        run: |
          set -x
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/download-td-artifacts`, `seemethere/download-artifact-s3@1da556a7aa0a088e3153970611f6c432d58e80e6`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 259-280 / 第 259-280 行

````yaml
          # Find the wheel file
          WHEEL_FILE=$(find win-torch-wheel -name "*.whl" -type f | head -n 1)
          if [ -z "$WHEEL_FILE" ]; then
            echo "Error: No wheel file found in win-torch-wheel directory"
            exit 1
          fi
          echo "Found wheel file: $WHEEL_FILE"

          # Unzip the wheel file
          unzip -q "$WHEEL_FILE" -d win-torch-wheel-extracted
          echo "Extracted wheel contents"

          # Setup CUDA libraries (cuda.lib and cudart.lib) directory
          mkdir -p win-torch-wheel-extracted/lib/x64
          if [ -f "win-torch-wheel/cuda.lib" ]; then
            mv win-torch-wheel/cuda.lib win-torch-wheel-extracted/lib/x64/
            echo "Moved cuda.lib to win-torch-wheel-extracted/lib/x64/"
          fi
          if [ -f "win-torch-wheel/cudart.lib" ]; then
            mv win-torch-wheel/cudart.lib win-torch-wheel-extracted/lib/x64/
            echo "Moved cudart.lib to win-torch-wheel-extracted/lib/x64/"
          fi
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-test.yml`.
- CN: 该部分描述 `.github/workflows/_linux-test.yml` 的仓库自动化行为。

### Lines 282-302 / 第 282-302 行

````yaml
          # Setup CUDA runtime DLL (needed for MinGW import lib generation on CUDA 13.0+)
          mkdir -p win-torch-wheel-extracted/bin/x64
          for dll in win-torch-wheel/cudart64_*.dll; do
            if [ -f "$dll" ]; then
              mv "$dll" win-torch-wheel-extracted/bin/x64/
              echo "Moved $(basename $dll) to win-torch-wheel-extracted/bin/x64/"
            fi
          done

          # Verify CUDA libraries and DLLs are present
          echo "CUDA libraries:"
          ls -la win-torch-wheel-extracted/lib/x64/ || echo "No CUDA libraries found"
          echo "CUDA DLLs:"
          ls -la win-torch-wheel-extracted/bin/x64/ || echo "No CUDA DLLs found"

      - name: Check for keep-going label and re-enabled test issues
        # This uses the filter-test-configs action because it conveniently
        # checks for labels and re-enabled test issues.  It does not actually do
        # any filtering.  All filtering is done in the build step.
        id: keep-going
        uses: ./.github/actions/filter-test-configs
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/filter-test-configs`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 303-323 / 第 303-323 行

````yaml
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          test-matrix: ${{ inputs.test-matrix }}
          job-name: ${{ steps.setup-linux.outputs.job-name }}

      - name: Set Test step time
        id: test-timeout
        shell: bash
        env:
          JOB_TIMEOUT: ${{ matrix.mem_leak_check == 'mem_leak_check' && 600 || inputs.timeout-minutes }}
        run: |
          echo "timeout=$((JOB_TIMEOUT-30))" >> "${GITHUB_OUTPUT}"

      - name: Preserve github env variables for use in docker
        shell: bash
        run: |
          env | grep '^GITHUB' >> "/tmp/github_env_${GITHUB_RUN_ID}"
          env | grep '^CI' >> "/tmp/github_env_${GITHUB_RUN_ID}"

      # Allow the test results to be uploaded S3 right after they finish, which are
      # then used in the test insight dashboard
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 324-347 / 第 324-347 行

````yaml
      - name: Authenticate with AWS
        if: ${{ contains(matrix.runner, 'b200') || steps.check-tpu.outputs.has_tpu == 'true' }}
        uses: aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722 # v4.1.0
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_upload-benchmark-results
          # The max duration enforced by the server side
          role-duration-seconds: 18000
          aws-region: us-east-1

      - name: Test
        id: test
        timeout-minutes: ${{ fromJson(steps.test-timeout.outputs.timeout) }}
        env:
          BUILD_ENVIRONMENT: ${{ inputs.build-environment }}
          PR_NUMBER: ${{ github.event.pull_request.number }}
          GITHUB_REPOSITORY: ${{ github.repository }}
          GITHUB_WORKFLOW: ${{ github.workflow }}
          GITHUB_JOB: ${{ github.job }}
          GITHUB_RUN_ID: ${{ github.run_id }}
          GITHUB_RUN_NUMBER: ${{ github.run_number }}
          GITHUB_RUN_ATTEMPT: ${{ github.run_attempt }}
          JOB_ID: ${{ steps.setup-linux.outputs.job-id }}
          JOB_NAME: ${{ steps.setup-linux.outputs.job-name }}
          BRANCH: ${{ steps.setup-linux.outputs.branch }}
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 348-371 / 第 348-371 行

````yaml
          SHA1: ${{ github.event.pull_request.head.sha || github.sha }}
          BASE_SHA: ${{ github.event.pull_request.base.sha || github.sha }}
          TEST_CONFIG: ${{ matrix.config }}
          SHARD_NUMBER: ${{ matrix.shard }}
          NUM_TEST_SHARDS: ${{ matrix.num_shards }}
          EXTRA_FLAGS: ${{ matrix.extra_flags || '' }}
          OP_BENCHMARK_TESTS: ${{ matrix.op_benchmark_tests }}
          REENABLED_ISSUES: ${{ steps.keep-going.outputs.reenabled-issues }}
          CONTINUE_THROUGH_ERROR: ${{ steps.keep-going.outputs.keep-going }}
          VERBOSE_TEST_LOGS: ${{ steps.keep-going.outputs.ci-verbose-test-logs }}
          TEST_SHOWLOCALS: ${{ steps.keep-going.outputs.ci-test-showlocals }}
          NO_TEST_TIMEOUT: ${{ steps.keep-going.outputs.ci-no-test-timeout }}
          NO_TD: ${{ steps.keep-going.outputs.ci-no-td }}
          TD_DISTRIBUTED: ${{ steps.keep-going.outputs.ci-td-distributed }}
          # Do not set SCCACHE_S3_KEY_PREFIX to share the cache between all build jobs
          SCCACHE_BUCKET: ${{ !contains(matrix.runner, 'b200') && 'ossci-compiler-cache-circleci-v2' || '' }}
          SCCACHE_REGION: ${{ !contains(matrix.runner, 'b200') && 'us-east-1' || '' }}
          SHM_SIZE: ${{ steps.install-nvidia-driver.outputs.has-nvidia == 'true' && '2g' || '1g' }}
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
          DOCKER_IMAGE_S390X: ${{ inputs.docker-image }}
          XLA_CUDA: ${{ contains(inputs.build-environment, 'xla') && '0' || '' }}
          XLA_CLANG_CACHE_S3_BUCKET_NAME: ossci-compiler-clang-cache-circleci-xla
          PYTORCH_TEST_CUDA_MEM_LEAK_CHECK: ${{ matrix.mem_leak_check && '1' || '0' }}
          PYTORCH_TEST_RERUN_DISABLED_TESTS: ${{ matrix.rerun_disabled_tests && '1' || '0' }}
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 372-391 / 第 372-391 行

````yaml
          TESTS_TO_INCLUDE: ${{ inputs.tests-to-include }}
          DASHBOARD_TAG: ${{ inputs.dashboard-tag }}
          EXPORT_PROFILER_TRACE: ${{ inputs.export-profiler-trace }}
          ENABLE_TORCH_TRACE: ${{ inputs.enable-torch-trace }}
          VLLM_TEST_HUGGING_FACE_TOKEN: ${{ secrets.VLLM_TEST_HUGGING_FACE_TOKEN }}
          HF_CACHE: /mnt/hf_cache
          # Use offline mode by default, only enable online mode to refresh the models
          # from HF when the PR has a special ci-refresh-hf-cache label or when the job
          # is a nightly scheduled run on main. The ci-refresh-hf-cache label is
          # automatically added to the vLLM pinned commit hash update, which effectively
          # refreshes the cache daily
          TRANSFORMERS_OFFLINE: ${{ (github.event_name == 'schedule' || contains(steps.keep-going.outputs.labels, 'ci-refresh-hf-cache')) && '0' || '1' }}
          HF_DATASETS_OFFLINE: ${{ (github.event_name == 'schedule' || contains(steps.keep-going.outputs.labels, 'ci-refresh-hf-cache')) && '0' || '1' }}
          HUGGING_FACE_HUB_TOKEN: ${{ secrets.HUGGING_FACE_HUB_TOKEN }}
          SCRIBE_GRAPHQL_ACCESS_TOKEN: ${{ secrets.SCRIBE_GRAPHQL_ACCESS_TOKEN }}
          ARTIFACTS_FILE_SUFFIX: ${{ github.job }}-${{ matrix.config }}-${{ matrix.shard }}-${{ matrix.num_shards }}-${{ matrix.runner }}_${{ steps.setup-linux.outputs.job-id }}
          TORCH_TPU: ${{ steps.check-tpu.outputs.has_tpu == 'true' && '1' || '' }}
          TORCH_TPU_TEXT_FILE: /var/lib/jenkins/workspace/.github/ci_commit_pins/torch_tpu.txt
        run: |
          set -x
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 393-409 / 第 393-409 行

````yaml
          if [[ $TEST_CONFIG == 'multigpu' ]]; then
            TEST_COMMAND=.ci/pytorch/multigpu-test.sh
          else
            TEST_COMMAND=.ci/pytorch/test.sh
          fi

          # Leaving 1GB for the runner and other things
          TOTAL_AVAILABLE_MEMORY_IN_GB=$(awk '/MemTotal/ { printf "%.3f \n", $2/1024/1024 - 1 }' /proc/meminfo)
          # https://docs.docker.com/engine/containers/resource_constraints/#--memory-swap-details, the 3GB swap
          # comes from https://github.com/pytorch/test-infra/pull/6058
          TOTAL_MEMORY_WITH_SWAP=$(("${TOTAL_AVAILABLE_MEMORY_IN_GB%.*}" + 3))

          if [[ ${BUILD_ENVIRONMENT} == *"s390x"* ]]; then
            SHM_OPTS=
            JENKINS_USER=
            # ensure that docker container cleanly exits in 12 hours
            # if for some reason cleanup action doesn't stop container
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-test.yml`.
- CN: 该部分描述 `.github/workflows/_linux-test.yml` 的仓库自动化行为。

### Lines 410-432 / 第 410-432 行

````yaml
            # when job is cancelled
            DOCKER_SHELL_CMD="sleep 12h"
            USED_IMAGE="${DOCKER_IMAGE_S390X}"
          else
            SHM_OPTS="--shm-size=${SHM_SIZE}"
            JENKINS_USER="--user jenkins"
            DOCKER_SHELL_CMD=
            USED_IMAGE="${DOCKER_IMAGE}"
          fi

          # Just create an empty HF_CACHE dir if it doesn't exist. This dir is not
          # used for anything besides vLLM jobs
          if [[ ! -d "${HF_CACHE}" ]]; then
            export HF_CACHE="${RUNNER_TEMP}/hf_cache"
            mkdir -p "${HF_CACHE}"

            # When there is no cache directory, e.g. benchmark, the job has no
            # way but to reach out to HF if needed
            export TRANSFORMERS_OFFLINE=0
            export HF_DATASETS_OFFLINE=0
          fi

          # detached container should get cleaned up by teardown_ec2_linux
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-test.yml`.
- CN: 该部分描述 `.github/workflows/_linux-test.yml` 的仓库自动化行为。

### Lines 433-456 / 第 433-456 行

````yaml
          # TODO: Stop building test binaries as part of the build phase
          # Used for GPU_FLAG, SHM_OPTS, JENKINS_USER and DOCKER_SHELL_CMD since that doesn't play nice
          # shellcheck disable=SC2086,SC2090
          container_name=$(docker run \
            ${GPU_FLAG:-} \
            ${SCCACHE_SERVER_PORT_DOCKER_FLAG:-} \
            -e BUILD_ENVIRONMENT \
            -e PR_NUMBER \
            -e GITHUB_ACTIONS \
            -e GITHUB_REPOSITORY \
            -e GITHUB_WORKFLOW \
            -e GITHUB_JOB \
            -e GITHUB_RUN_ID \
            -e GITHUB_RUN_NUMBER \
            -e GITHUB_RUN_ATTEMPT \
            -e JOB_ID \
            -e JOB_NAME \
            -e BASE_SHA \
            -e BRANCH \
            -e SHA1 \
            -e AWS_DEFAULT_REGION \
            -e AWS_ACCESS_KEY_ID \
            -e AWS_SECRET_ACCESS_KEY \
            -e AWS_SESSION_TOKEN \
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-test.yml`.
- CN: 该部分描述 `.github/workflows/_linux-test.yml` 的仓库自动化行为。

### Lines 457-480 / 第 457-480 行

````yaml
            -e IN_WHEEL_TEST \
            -e SHARD_NUMBER \
            -e TEST_CONFIG \
            -e NUM_TEST_SHARDS \
            -e REENABLED_ISSUES \
            -e CONTINUE_THROUGH_ERROR \
            -e VERBOSE_TEST_LOGS \
            -e TEST_SHOWLOCALS \
            -e NO_TEST_TIMEOUT \
            -e NO_TD \
            -e TD_DISTRIBUTED \
            -e PR_LABELS \
            -e MAX_JOBS="$(nproc --ignore=2)" \
            -e SCCACHE_BUCKET \
            -e SCCACHE_REGION \
            -e XLA_CUDA \
            -e XLA_CLANG_CACHE_S3_BUCKET_NAME \
            -e PYTORCH_TEST_CUDA_MEM_LEAK_CHECK \
            -e PYTORCH_TEST_RERUN_DISABLED_TESTS \
            -e TESTS_TO_INCLUDE \
            -e SKIP_SCCACHE_INITIALIZATION=1 \
            -e HUGGING_FACE_HUB_TOKEN \
            -e VLLM_TEST_HUGGING_FACE_TOKEN \
            -e HF_CACHE \
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-test.yml`.
- CN: 该部分描述 `.github/workflows/_linux-test.yml` 的仓库自动化行为。

### Lines 481-504 / 第 481-504 行

````yaml
            -e TRANSFORMERS_OFFLINE \
            -e HF_DATASETS_OFFLINE \
            -e SCRIBE_GRAPHQL_ACCESS_TOKEN \
            -e DASHBOARD_TAG \
            -e EXPORT_PROFILER_TRACE \
            -e ENABLE_TORCH_TRACE \
            -e ARTIFACTS_FILE_SUFFIX \
            -e TORCH_TPU \
            -e TORCH_TPU_TEXT_FILE \
            ${TPU_DOCKER_FLAGS:-} \
            --memory="${TOTAL_AVAILABLE_MEMORY_IN_GB%.*}g" \
            --memory-swap="${TOTAL_MEMORY_WITH_SWAP}g" \
            --env-file="/tmp/github_env_${GITHUB_RUN_ID}" \
            --security-opt seccomp=unconfined \
            --cap-add=SYS_PTRACE \
            --ipc=host \
            ${SHM_OPTS} \
            --tty \
            --detach \
            --name="${container_name}" \
            ${JENKINS_USER} \
            -v "${GITHUB_WORKSPACE}:/var/lib/jenkins/workspace" \
            -v "${HF_CACHE}:${HF_CACHE}" \
            -w /var/lib/jenkins/workspace \
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-test.yml`.
- CN: 该部分描述 `.github/workflows/_linux-test.yml` 的仓库自动化行为。

### Lines 505-524 / 第 505-524 行

````yaml
            "${USED_IMAGE}" \
            ${DOCKER_SHELL_CMD}
          )
          echo "DOCKER_CONTAINER_ID=${container_name}" >> "${GITHUB_ENV}"

          if [[ ${BUILD_ENVIRONMENT} == *"s390x"* ]]; then
            # sometimes there are intermittent network errors, do a couple of retries
            docker exec -t "${container_name}" sh -c "python3 -m pip install -r .ci/docker/requirements-ci.txt || \
              (sleep 15 && python3 -m pip install -r .ci/docker/requirements-ci.txt) || \
              (sleep 30 && python3 -m pip install -r .ci/docker/requirements-ci.txt) || \
              (sleep 60 && python3 -m pip install -r .ci/docker/requirements-ci.txt) || \
              (sleep 300 && python3 -m pip install -r .ci/docker/requirements-ci.txt)"
          fi

          docker exec -t "${container_name}" sh -c "python3 -m pip install $(echo dist/*.whl)[opt-einsum] && (if [ -n \"\$TORCH_TPU\" ] && [ -f .ci/docker/common/install_torch_tpu.sh ]; then bash .ci/docker/common/install_torch_tpu.sh; fi) && ${TEST_COMMAND}"

      - name: Upload pytest cache if tests failed
        uses: ./.github/actions/pytest-cache-upload
        continue-on-error: true
        if: failure() && steps.test.conclusion && steps.test.conclusion == 'failure' && inputs.build-environment != 'linux-s390x-binary-manywheel'
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/pytest-cache-upload`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 525-543 / 第 525-543 行

````yaml
        with:
          cache_dir: .pytest_cache
          shard: ${{ matrix.shard }}
          sha: ${{ github.event.pull_request.head.sha || github.sha }}
          test_config: ${{ matrix.config }}
          job_identifier: ${{ github.workflow }}_${{ inputs.build-environment }}

      - name: Authenticate with AWS
        if: ${{ always() && contains(matrix.runner, 'b200') }}
        uses: aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722 # v4.1.0
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_upload-benchmark-results
          # The max duration enforced by the server side
          role-duration-seconds: 18000
          aws-region: us-east-1

      - name: Upload the benchmark results
        uses: pytorch/test-infra/.github/actions/upload-benchmark-results@main
        if: inputs.build-environment != 'linux-s390x-binary-manywheel' && steps.check-tpu.outputs.has_tpu != 'true'
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`, `pytorch/test-infra/.github/actions/upload-benchmark-results@main`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 544-563 / 第 544-563 行

````yaml
        with:
          benchmark-results-dir: test/test-reports
          dry-run: false
          schema-version: v3
          github-token: ${{ secrets.GITHUB_TOKEN }}

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

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 565-586 / 第 565-586 行

````yaml
      - name: Upload test artifacts
        uses: ./.github/actions/upload-test-artifacts
        if: always() && steps.test.conclusion && steps.test.conclusion != 'skipped'
        with:
          file-suffix: ${{ github.job }}-${{ matrix.config }}-${{ matrix.shard }}-${{ matrix.num_shards }}-${{ matrix.runner }}_${{ steps.setup-linux.outputs.job-id }}
          use-gha: ${{ inputs.use-gha }}
          s3-bucket: ${{ inputs.s3-bucket }}

      - name: Collect backtraces from coredumps (if any)
        if: always()
        run: |
          # shellcheck disable=SC2156
          find . -iname "core.[1-9]*" -exec docker exec "${DOCKER_CONTAINER_ID}" sh -c "gdb python {} -ex 'bt' -ex 'q'" \;

      - name: Store Core dumps on S3
        uses: seemethere/upload-artifact-s3@baba72d0712b404f646cebe0730933554ebce96a # v5.1.0
        if: failure()
        with:
          name: coredumps-${{ matrix.config }}-${{ matrix.shard }}-${{ matrix.num_shards }}-${{ matrix.runner }}
          retention-days: 14
          if-no-files-found: ignore
          path: ./**/core.[1-9]*
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/upload-test-artifacts`, `seemethere/upload-artifact-s3@baba72d0712b404f646cebe0730933554ebce96a`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 588-609 / 第 588-609 行

````yaml
      - name: Upload utilization stats
        if: ${{ always() && steps.test.conclusion && steps.test.conclusion != 'skipped' && !inputs.disable-monitor && inputs.build-environment != 'linux-s390x-binary-manywheel' }}
        continue-on-error: true
        uses: ./.github/actions/upload-utilization-stats
        with:
          job_id: ${{ steps.setup-linux.outputs.job-id }}
          job_name: ${{ steps.setup-linux.outputs.job-name }}
          workflow_name: ${{ github.workflow }}
          workflow_run_id: ${{github.run_id}}
          workflow_attempt: ${{github.run_attempt}}

      - name: Teardown Linux
        uses: pytorch/test-infra/.github/actions/teardown-linux@main
        if: always() && steps.check_container_runner.outputs.IN_CONTAINER_RUNNER == 'false'

      - name: Cleanup docker
        if: always() && inputs.build-environment == 'linux-s390x-binary-manywheel'
        shell: bash
        run: |
          # on s390x stop the container for clean worker stop
          docker stop -a || true
          docker kill -a || true
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/upload-utilization-stats`, `pytorch/test-infra/.github/actions/teardown-linux@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 611-630 / 第 611-630 行

````yaml
  test-osdc:
    # Don't run on forked repos or empty test matrix
    if: github.repository_owner == 'pytorch' && toJSON(fromJSON(inputs.test-matrix).include) != '[]' && inputs.use-arc
    strategy:
      matrix: ${{ fromJSON(inputs.test-matrix) }}
      fail-fast: false
    environment: ${{ github.ref == 'refs/heads/main' && 'scribe-protected' || startsWith(github.ref, 'refs/heads/release/') && 'scribe-protected' || contains(github.event.pull_request.labels.*.name, 'ci-scribe') && 'scribe-pr' || '' }}
    runs-on: ${{ matrix.runner }}
    container:
      image: ${{ inputs.docker-image }}
      options: "--gpus all"
    timeout-minutes: ${{ matrix.mem_leak_check == 'mem_leak_check' && 600 || inputs.timeout-minutes }}
    permissions:
      id-token: write
      contents: read
      actions: read
    steps:
      - name: Setup Linux
        id: setup-linux
        uses: pytorch/pytorch/.github/actions/setup-linux@main
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/setup-linux@main`; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 631-650 / 第 631-650 行

````yaml
        with:
          use-arc: true
          python-version: ${{ inputs.python-version }}
          compiler: ${{ inputs.compiler }}
          cuda-version: ${{ inputs.cuda-version }}
          submodules: 'false'
          github-token: ${{ secrets.GITHUB_TOKEN }}

      - name: Configure AWS credentials
        id: aws-creds
        continue-on-error: true
        uses: aws-actions/configure-aws-credentials@8df5847569e6427dd6c4fb1cf565c83acfa8afa7
        with:
          role-to-assume: arn:aws:iam::308535385114:role/arc
          aws-region: us-east-1
          # The max duration enforced by the server side
          role-duration-seconds: 18000

      - name: Download build artifacts
        uses: pytorch/pytorch/.github/actions/download-build-artifacts@main
````

- EN: This section reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@8df5847569e6427dd6c4fb1cf565c83acfa8afa7`, `pytorch/pytorch/.github/actions/download-build-artifacts@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 651-671 / 第 651-671 行

````yaml
        with:
          name: ${{ inputs.build-environment }}
          s3-bucket: ${{ inputs.s3-bucket }}
          use-gha: ${{ steps.aws-creds.outcome != 'success' || inputs.use-gha }}

      - name: Download TD artifacts
        continue-on-error: true
        uses: pytorch/pytorch/.github/actions/download-td-artifacts@main

      - name: Download Windows torch wheel for cross-compilation
        if: matrix.win_torch_wheel_artifact != ''
        uses: seemethere/download-artifact-s3@1da556a7aa0a088e3153970611f6c432d58e80e6 # v4.2.0
        with:
          name: ${{ matrix.win_torch_wheel_artifact }}
          path: win-torch-wheel

      - name: Extract Windows wheel and setup CUDA libraries
        if: matrix.win_torch_wheel_artifact != ''
        shell: bash
        run: |
          set -x
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/download-td-artifacts@main`, `seemethere/download-artifact-s3@1da556a7aa0a088e3153970611f6c432d58e80e6`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 673-694 / 第 673-694 行

````yaml
          # Find the wheel file
          WHEEL_FILE=$(find win-torch-wheel -name "*.whl" -type f | head -n 1)
          if [ -z "$WHEEL_FILE" ]; then
            echo "Error: No wheel file found in win-torch-wheel directory"
            exit 1
          fi
          echo "Found wheel file: $WHEEL_FILE"

          # Unzip the wheel file
          unzip -q "$WHEEL_FILE" -d win-torch-wheel-extracted
          echo "Extracted wheel contents"

          # Setup CUDA libraries (cuda.lib and cudart.lib) directory
          mkdir -p win-torch-wheel-extracted/lib/x64
          if [ -f "win-torch-wheel/cuda.lib" ]; then
            mv win-torch-wheel/cuda.lib win-torch-wheel-extracted/lib/x64/
            echo "Moved cuda.lib to win-torch-wheel-extracted/lib/x64/"
          fi
          if [ -f "win-torch-wheel/cudart.lib" ]; then
            mv win-torch-wheel/cudart.lib win-torch-wheel-extracted/lib/x64/
            echo "Moved cudart.lib to win-torch-wheel-extracted/lib/x64/"
          fi
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-test.yml`.
- CN: 该部分描述 `.github/workflows/_linux-test.yml` 的仓库自动化行为。

### Lines 696-716 / 第 696-716 行

````yaml
          # Setup CUDA runtime DLL (needed for MinGW import lib generation on CUDA 13.0+)
          mkdir -p win-torch-wheel-extracted/bin/x64
          for dll in win-torch-wheel/cudart64_*.dll; do
            if [ -f "$dll" ]; then
              mv "$dll" win-torch-wheel-extracted/bin/x64/
              echo "Moved $(basename $dll) to win-torch-wheel-extracted/bin/x64/"
            fi
          done

          # Verify CUDA libraries and DLLs are present
          echo "CUDA libraries:"
          ls -la win-torch-wheel-extracted/lib/x64/ || echo "No CUDA libraries found"
          echo "CUDA DLLs:"
          ls -la win-torch-wheel-extracted/bin/x64/ || echo "No CUDA DLLs found"

      - name: Check for keep-going label and re-enabled test issues
        # This uses the filter-test-configs action because it conveniently
        # checks for labels and re-enabled test issues. It does not actually do
        # any filtering.  All filtering is done in the build step.
        id: keep-going
        uses: pytorch/pytorch/.github/actions/filter-test-configs@main
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/filter-test-configs@main`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 717-732 / 第 717-732 行

````yaml
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          test-matrix: ${{ inputs.test-matrix }}
          job-name: ${{ steps.setup-linux.outputs.job-name }}

      - name: Set test step time
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

### Lines 733-756 / 第 733-756 行

````yaml
        env:
          BUILD_ENVIRONMENT: ${{ inputs.build-environment }}
          PR_NUMBER: ${{ github.event.pull_request.number }}
          GITHUB_REPOSITORY: ${{ github.repository }}
          GITHUB_WORKFLOW: ${{ github.workflow }}
          GITHUB_JOB: ${{ github.job }}
          GITHUB_RUN_ID: ${{ github.run_id }}
          GITHUB_RUN_NUMBER: ${{ github.run_number }}
          GITHUB_RUN_ATTEMPT: ${{ github.run_attempt }}
          JOB_ID: ${{ steps.setup-linux.outputs.job-id }}
          JOB_NAME: ${{ steps.setup-linux.outputs.job-name }}
          BRANCH: ${{ steps.setup-linux.outputs.branch }}
          SHA1: ${{ github.event.pull_request.head.sha || github.sha }}
          BASE_SHA: ${{ github.event.pull_request.base.sha || github.sha }}
          TEST_CONFIG: ${{ matrix.config }}
          SHARD_NUMBER: ${{ matrix.shard }}
          NUM_TEST_SHARDS: ${{ matrix.num_shards }}
          EXTRA_FLAGS: ${{ matrix.extra_flags || '' }}
          OP_BENCHMARK_TESTS: ${{ matrix.op_benchmark_tests }}
          REENABLED_ISSUES: ${{ steps.keep-going.outputs.reenabled-issues }}
          CONTINUE_THROUGH_ERROR: ${{ steps.keep-going.outputs.keep-going }}
          VERBOSE_TEST_LOGS: ${{ steps.keep-going.outputs.ci-verbose-test-logs }}
          TEST_SHOWLOCALS: ${{ steps.keep-going.outputs.ci-test-showlocals }}
          NO_TEST_TIMEOUT: ${{ steps.keep-going.outputs.ci-no-test-timeout }}
````

- EN: This section declares workflow triggers or event bindings; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；控制令牌权限范围或环境变量以实现安全执行。

### Lines 757-779 / 第 757-779 行

````yaml
          NO_TD: ${{ steps.keep-going.outputs.ci-no-td }}
          TD_DISTRIBUTED: ${{ steps.keep-going.outputs.ci-td-distributed }}
          SCCACHE_BUCKET: ossci-compiler-cache-circleci-v2
          SCCACHE_REGION: us-east-1
          SCCACHE_S3_NO_CREDENTIALS: ${{ steps.aws-creds.outcome != 'success' && 'true' || 'false' }}
          XLA_CUDA: ${{ contains(inputs.build-environment, 'xla') && '0' || '' }}
          XLA_CLANG_CACHE_S3_BUCKET_NAME: ossci-compiler-clang-cache-circleci-xla
          PYTORCH_TEST_CUDA_MEM_LEAK_CHECK: ${{ matrix.mem_leak_check && '1' || '0' }}
          PYTORCH_TEST_RERUN_DISABLED_TESTS: ${{ matrix.rerun_disabled_tests && '1' || '0' }}
          TESTS_TO_INCLUDE: ${{ inputs.tests-to-include }}
          DASHBOARD_TAG: ${{ inputs.dashboard-tag }}
          VLLM_TEST_HUGGING_FACE_TOKEN: ${{ secrets.VLLM_TEST_HUGGING_FACE_TOKEN }}
          HF_CACHE: /mnt/hf_cache
          TRANSFORMERS_OFFLINE: ${{ (github.event_name == 'schedule' || contains(steps.keep-going.outputs.labels, 'ci-refresh-hf-cache')) && '0' || '1' }}
          HF_DATASETS_OFFLINE: ${{ (github.event_name == 'schedule' || contains(steps.keep-going.outputs.labels, 'ci-refresh-hf-cache')) && '0' || '1' }}
          HUGGING_FACE_HUB_TOKEN: ${{ secrets.HUGGING_FACE_HUB_TOKEN }}
          SCRIBE_GRAPHQL_ACCESS_TOKEN: ${{ secrets.SCRIBE_GRAPHQL_ACCESS_TOKEN }}
          ARTIFACTS_FILE_SUFFIX: ${{ github.job }}-${{ matrix.config }}-${{ matrix.shard }}-${{ matrix.num_shards }}-${{ matrix.runner }}_${{ steps.setup-linux.outputs.job-id }}
          TORCH_TPU_TEXT_FILE: /var/lib/jenkins/workspace/.github/ci_commit_pins/torch_tpu.txt
          USE_ARC: "1"
        shell: bash
        run: |
          set -x
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 781-801 / 第 781-801 行

````yaml
          if [[ "${TEST_CONFIG}" == 'multigpu' ]]; then
            TEST_COMMAND=.ci/pytorch/multigpu-test.sh
          else
            TEST_COMMAND=.ci/pytorch/test.sh
          fi

          # Just create an empty HF_CACHE dir if it doesn't exist. This dir is not
          # used for anything besides vLLM jobs
          if [[ ! -d "${HF_CACHE}" ]]; then
            export HF_CACHE="${RUNNER_TEMP}/hf_cache"
            mkdir -p "${HF_CACHE}"

            # When there is no cache directory, e.g. benchmark, the job has no
            # way but to reach out to HF if needed
            export TRANSFORMERS_OFFLINE=0
            export HF_DATASETS_OFFLINE=0
          fi

          # shellcheck disable=SC2046
          python3 -m pip install $(echo dist/*.whl)[opt-einsum]
          ${TEST_COMMAND}
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-test.yml`.
- CN: 该部分描述 `.github/workflows/_linux-test.yml` 的仓库自动化行为。

### Lines 803-824 / 第 803-824 行

````yaml
      - name: Configure AWS credentials
        id: aws-creds-benchmark
        continue-on-error: true
        uses: aws-actions/configure-aws-credentials@8df5847569e6427dd6c4fb1cf565c83acfa8afa7
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_upload-benchmark-results
          aws-region: us-east-1

      - name: Upload pytest cache if tests failed
        uses: pytorch/pytorch/.github/actions/pytest-cache-upload@main
        continue-on-error: true
        if: failure() && steps.test.conclusion && steps.test.conclusion == 'failure' && steps.aws-creds-benchmark.outcome == 'success'
        with:
          cache_dir: .pytest_cache
          shard: ${{ matrix.shard }}
          sha: ${{ github.event.pull_request.head.sha || github.sha }}
          test_config: ${{ matrix.config }}
          job_identifier: ${{ github.workflow }}_${{ inputs.build-environment }}

      - name: Upload the benchmark results
        if: steps.aws-creds-benchmark.outcome == 'success'
        uses: pytorch/test-infra/.github/actions/upload-benchmark-results@main
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@8df5847569e6427dd6c4fb1cf565c83acfa8afa7`, `pytorch/pytorch/.github/actions/pytest-cache-upload@main`, `pytorch/test-infra/.github/actions/upload-benchmark-results@main`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 825-839 / 第 825-839 行

````yaml
        with:
          benchmark-results-dir: test/test-reports
          dry-run: false
          schema-version: v3
          github-token: ${{ secrets.GITHUB_TOKEN }}

      - name: Print remaining test logs
        shell: bash
        if: always() && steps.test.conclusion
        run: |
          cat test/**/*_toprint.log || true

      - name: Upload test artifacts
        uses: pytorch/pytorch/.github/actions/upload-test-artifacts@main
        if: always() && steps.test.conclusion && steps.test.conclusion != 'skipped'
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/upload-test-artifacts@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 840-843 / 第 840-843 行

````yaml
        with:
          file-suffix: ${{ github.job }}-${{ matrix.config }}-${{ matrix.shard }}-${{ matrix.num_shards }}-${{ matrix.runner }}_${{ steps.setup-linux.outputs.job-id }}
          use-gha: ${{ inputs.use-gha }}
          s3-bucket: ${{ inputs.s3-bucket }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。


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
- Reusable actions / 复用 Action: `pytorch/test-infra/.github/actions/setup-ssh@main`, `pytorch/pytorch/.github/actions/setup-linux@main`, `./.github/actions/check-tpu`, `./.github/actions/ecr-login`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`, `pytorch/test-infra/.github/actions/setup-nvidia@main`, `./.github/actions/download-build-artifacts`, ...
- Inline commands / 内联命令: `If`, `job`, `Set`, `[Experimental]`, `Currently,`, `since`, `HF`, `FB`, ...
- Environment variables / 环境变量: `JSON`, `GHA`, `ARC`, `OSDC`, `EC2`, `CUDA`, `TORCH_TRACE`, `HUGGING_FACE_HUB_TOKEN`, `VLLM_TEST_HUGGING_FACE_TOKEN`, `SCRIBE_GRAPHQL_ACCESS_TOKEN`, ...
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `inputs`, `secrets`, `env`, `jobs`, `test`, `strategy`, `permissions`, `steps`, ...
