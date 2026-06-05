# _linux-build.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/_linux-build.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-18 / 第 1-18 行

````yaml
name: linux-build

on:
  workflow_call:
    inputs:
      build-environment:
        required: true
        type: string
        description: Top-level label for what's being built/tested.
      docker-image-name:
        required: true
        type: string
        description: Name of the base docker image to build with.
      build-generates-artifacts:
        required: false
        type: boolean
        default: true
        description: If set, upload generated build artifacts.
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 19-36 / 第 19-36 行

````yaml
      sync-tag:
        required: false
        type: string
        default: ""
        description: |
          If this is set, our linter will use this to make sure that every other
          job with the same `sync-tag` is identical.
      cuda-arch-list:
        required: false
        type: string
        default: "7.5"
        description: |
          List of CUDA architectures CI build should target.
      runner_prefix:
        required: false
        default: ""
        type: string
        description: Prefix for runner label
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-build.yml`.
- CN: 该部分描述 `.github/workflows/_linux-build.yml` 的仓库自动化行为。

### Lines 37-56 / 第 37-56 行

````yaml
      runner:
        required: false
        type: string
        default: "linux.c7i.2xlarge"
        description: |
          Label of the runner this job should run on.
      test-matrix:
        required: false
        type: string
        description: |
          An option JSON description of what test configs to run later on. This
          is moved here from the Linux test workflow so that we can apply filter
          logic using test-config labels earlier and skip unnecessary builds
      selected-test-configs:
        description: |
          A comma-separated list of test configurations from the test matrix to keep,
          The empty list means we are going to keep every configurations by defaults
        required: false
        type: string
        default: ""
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-build.yml`.
- CN: 该部分描述 `.github/workflows/_linux-build.yml` 的仓库自动化行为。

### Lines 57-78 / 第 57-78 行

````yaml
      s3-bucket:
        description: S3 bucket to download artifact
        required: false
        type: string
        default: "gha-artifacts"
      aws-role-to-assume:
        description: Role to assume for downloading artifacts
        required: false
        type: string
        default: ""
      disable-monitor:
        description: |
          Disable utilization monitoring for build job
        required: false
        type: boolean
        default: false
      monitor-log-interval:
        description: |
          Set the interval for the monitor script to log utilization.
        required: false
        type: number
        default: 5
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-build.yml`.
- CN: 该部分描述 `.github/workflows/_linux-build.yml` 的仓库自动化行为。

### Lines 79-98 / 第 79-98 行

````yaml
      monitor-data-collect-interval:
        description: |
          Set the interval for the monitor script to collect data.
        required: false
        type: number
        default: 1
      allow-reuse-old-whl:
        description: |
          If set, the build try to pull an old wheel from s3 that was built on a
          commit with no cpp changes from this commit
        required: false
        type: boolean
        default: true
      build-additional-packages:
        description: |
          If set, the build job will also builds these packages and saves their
          wheels as artifacts
        required: false
        type: string
        default: ""
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-build.yml`.
- CN: 该部分描述 `.github/workflows/_linux-build.yml` 的仓库自动化行为。

### Lines 99-120 / 第 99-120 行

````yaml
      build-external-packages:
        description: |
          If set, the build external packages and saves their wheels as artifacts
          use command separated list of packages to build ex: 'vllm,transformers'.
        required: false
        type: string
        default: ""
      use-arc:
        required: false
        type: boolean
        default: false
        description: If true, use ARC (OSDC) runner path instead of EC2.
      python-version:
        required: false
        type: string
        default: ""
        description: Python version to use for the OSDC build.
      compiler:
        required: false
        type: string
        default: ""
        description: Compiler to use for the OSDC build.
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-build.yml`.
- CN: 该部分描述 `.github/workflows/_linux-build.yml` 的仓库自动化行为。

### Lines 121-142 / 第 121-142 行

````yaml
      cuda-version:
        required: false
        type: string
        default: ""
        description: CUDA version to use for the OSDC build.

    secrets:
      HUGGING_FACE_HUB_TOKEN:
        required: false
        description: |
          HF Auth token to avoid rate limits when downloading models or datasets from hub

    outputs:
      docker-image:
        value: ${{ jobs.build.outputs.docker-image || jobs.build-osdc.outputs.docker-image }}
        description: The docker image containing the built PyTorch.
      test-matrix:
        value: ${{ jobs.build.outputs.test-matrix || jobs.build-osdc.outputs.test-matrix }}
        description: An optional JSON description of what test configs to run later on.
      build-environment:
        value: ${{ jobs.build.outputs.build-environment || jobs.build-osdc.outputs.build-environment }}
        description: Top-level label for what's being built/tested.
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-build.yml`.
- CN: 该部分描述 `.github/workflows/_linux-build.yml` 的仓库自动化行为。

### Lines 144-166 / 第 144-166 行

````yaml
jobs:
  build:
    # Don't run on forked repos
    if: github.repository_owner == 'pytorch' && !inputs.use-arc
    runs-on: ${{ inputs.runner_prefix }}${{ inputs.runner }}
    timeout-minutes: 480
    outputs:
      docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
      test-matrix: ${{ steps.filter.outputs.test-matrix }}
      build-environment: ${{ inputs.build-environment }}
    steps:
      - name: Setup SSH (Click me for login details)
        uses: pytorch/test-infra/.github/actions/setup-ssh@main
        if: inputs.build-environment != 'linux-s390x-binary-manywheel'
        with:
          github-secret: ${{ secrets.GITHUB_TOKEN }}
          instructions: |
            Build is done inside the container, to start an interactive session run:
              docker exec -it $(docker container ps --format '{{.ID}}') bash

      - name: Setup Linux
        id: setup-linux
        uses: pytorch/pytorch/.github/actions/setup-linux@main
````

- EN: This section lays out job topology or execution stages; reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-ssh@main`, `pytorch/pytorch/.github/actions/setup-linux@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 167-188 / 第 167-188 行

````yaml
        with:
          python-version: ${{ inputs.python-version }}
          compiler: ${{ inputs.compiler }}
          cuda-version: ${{ inputs.cuda-version }}
          github-token: ${{ secrets.GITHUB_TOKEN }}

      - name: Login to ECR
        if: inputs.build-environment != 'linux-s390x-binary-manywheel'
        uses: ./.github/actions/ecr-login
        with:
          aws-role-to-assume: ${{ inputs.aws-role-to-assume }}

      - name: Check if can use old whl build
        id: use-old-whl
        uses: ./.github/actions/reuse-old-whl
        if: ${{ inputs.allow-reuse-old-whl }}
        with:
          build-environment: ${{ inputs.build-environment }}
          run-id: ${{ github.run_id }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          job-id: ${{ steps.setup-linux.outputs.job-id }}
          job-name: ${{ steps.setup-linux.outputs.job-name }}
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/ecr-login`, `./.github/actions/reuse-old-whl`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 190-211 / 第 190-211 行

````yaml
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        if: inputs.build-environment != 'linux-s390x-binary-manywheel'
        with:
          docker-image-name: ${{ inputs.docker-image-name }}

      - name: Use following to pull public copy of the image
        id: print-ghcr-mirror
        if: inputs.build-environment != 'linux-s390x-binary-manywheel' && steps.use-old-whl.outputs.reuse != 'true'
        env:
          ECR_DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
        shell: bash
        run: |
          tag=${ECR_DOCKER_IMAGE##*:}
          echo "docker pull ghcr.io/pytorch/ci-image:${tag/:/-}"

      - name: Pull docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        if: inputs.build-environment != 'linux-s390x-binary-manywheel' && steps.use-old-whl.outputs.reuse != 'true'
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 213-227 / 第 213-227 行

````yaml
      # Apply the filter logic to the build step too if the test-config label is already there
      - name: Select all requested test configurations (if the test matrix is available)
        id: filter
        uses: ./.github/actions/filter-test-configs
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          test-matrix: ${{ inputs.test-matrix }}
          selected-test-configs: ${{ inputs.selected-test-configs }}
          job-name: ${{ steps.setup-linux.outputs.job-name }}

      - name: Start monitoring script
        id: monitor-script
        if: ${{ !inputs.disable-monitor }}
        shell: bash
        continue-on-error: true
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/filter-test-configs`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 228-246 / 第 228-246 行

````yaml
        env:
          JOB_ID: ${{ steps.setup-linux.outputs.job-id }}
          JOB_NAME: ${{ steps.setup-linux.outputs.job-name }}
          WORKFLOW_NAME: ${{ github.workflow }}
          WORKFLOW_RUN_ID: ${{github.run_id}}
          MONITOR_LOG_INTERVAL: ${{ inputs.monitor-log-interval }}
          MONITOR_DATA_COLLECT_INTERVAL: ${{ inputs.monitor-data-collect-interval }}
        run: |
          mkdir -p ../../usage_logs
          python3 -m pip install psutil==5.9.8 dataclasses_json==0.6.7
          python3 -m tools.stats.monitor \
          --log-interval "$MONITOR_LOG_INTERVAL" \
          --data-collect-interval "$MONITOR_DATA_COLLECT_INTERVAL" \
          > "../../usage_logs/usage_log_build_${JOB_ID}.txt" 2>&1 &
          echo "monitor-script-pid=${!}" >> "${GITHUB_OUTPUT}"

      - name: Build
        if: (steps.filter.outputs.is-test-matrix-empty == 'False' || inputs.test-matrix == '') && steps.use-old-whl.outputs.reuse != 'true'
        id: build
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 247-269 / 第 247-269 行

````yaml
        env:
          BUILD_ENVIRONMENT: ${{ inputs.build-environment }}
          BRANCH: ${{ steps.setup-linux.outputs.branch }}
          PR_NUMBER: ${{ github.event.pull_request.number }}
          SHA1: ${{ github.event.pull_request.head.sha || github.sha }}
          # Do not set SCCACHE_S3_KEY_PREFIX to share the cache between all build jobs
          SCCACHE_BUCKET: ossci-compiler-cache-circleci-v2
          SCCACHE_REGION: us-east-1
          XLA_CLANG_CACHE_S3_BUCKET_NAME: ossci-compiler-clang-cache-circleci-xla
          PR_LABELS: ${{ toJson(github.event.pull_request.labels.*.name) }}
          TORCH_CUDA_ARCH_LIST: ${{ inputs.cuda-arch-list }}
          DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
          DOCKER_IMAGE_S390X: ${{ inputs.docker-image-name }}
          XLA_CUDA: ${{ contains(inputs.build-environment, 'xla') && '0' || '' }}
          OUR_GITHUB_JOB_ID: ${{ steps.setup-linux.outputs.job-id }}
          HUGGING_FACE_HUB_TOKEN: ${{ secrets.HUGGING_FACE_HUB_TOKEN }}
          BUILD_ADDITIONAL_PACKAGES: ${{ inputs.build-additional-packages }}
          RUNNER: ${{ inputs.runner }}
        run: |
          START_TIME=$(date +%s)
          if [[ ${BUILD_ENVIRONMENT} == *"s390x"* ]]; then
            JENKINS_USER=
            USED_IMAGE="${DOCKER_IMAGE_S390X}"
````

- EN: This section declares workflow triggers or event bindings; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 270-291 / 第 270-291 行

````yaml
            # ensure that docker container cleanly exits in 12 hours
            # if for some reason cleanup action doesn't stop container
            # when job is cancelled
            DOCKER_SHELL_CMD="sleep 12h"

            # since some steps are skipped on s390x, if they are necessary, run them here
            env | grep '^GITHUB' >> "/tmp/github_env_${GITHUB_RUN_ID}"
            env | grep '^CI' >> "/tmp/github_env_${GITHUB_RUN_ID}"
          else
            JENKINS_USER="--user jenkins"
            USED_IMAGE="${DOCKER_IMAGE}"
            DOCKER_SHELL_CMD=
          fi

          # Leaving 1GB for the runner and other things
          TOTAL_AVAILABLE_MEMORY_IN_GB=$(awk '/MemTotal/ { printf "%.3f \n", $2/1024/1024 - 1 }' /proc/meminfo)
          # https://docs.docker.com/engine/containers/resource_constraints/#--memory-swap-details, the 3GB swap
          # comes from https://github.com/pytorch/test-infra/pull/6058
          TOTAL_MEMORY_WITH_SWAP=$(("${TOTAL_AVAILABLE_MEMORY_IN_GB%.*}" + 3))

          if [[ ${BUILD_ENVIRONMENT} == *"riscv64"* ]]; then
            # EC2 specific setup for RISC-V emulation
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-build.yml`.
- CN: 该部分描述 `.github/workflows/_linux-build.yml` 的仓库自动化行为。

### Lines 292-313 / 第 292-313 行

````yaml
            # Ensure binfmt_misc is available
            echo "Mounting binfmt_misc filesystem"
            sudo mount binfmt_misc -t binfmt_misc /proc/sys/fs/binfmt_misc 2>/dev/null || true

            echo "QEMU registration: multiarch/qemu-user-static"
            docker run --rm --privileged multiarch/qemu-user-static --reset -p yes || true

            # Final verification
            echo "Checking binfmt_misc status:"
            ls -la /proc/sys/fs/binfmt_misc/ 2>/dev/null || echo "Cannot access binfmt_misc directory"

            if [ -f /proc/sys/fs/binfmt_misc/qemu-riscv64 ]; then
              echo "qemu-riscv64 registration successful"
            else
              echo "qemu-riscv64 registration failed - proceeding without emulation"
              echo "This may cause RISC-V builds to fail"
            fi

            RISCV_DOCKER_ARGS="--privileged"
          else
            RISCV_DOCKER_ARGS=
          fi
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-build.yml`.
- CN: 该部分描述 `.github/workflows/_linux-build.yml` 的仓库自动化行为。

### Lines 315-338 / 第 315-338 行

````yaml
          # detached container should get cleaned up by teardown_ec2_linux
          # Used for JENKINS_USER and DOCKER_SHELL_CMD, which can be empty
          # shellcheck disable=SC2086
          container_name=$(docker run \
            ${RISCV_DOCKER_ARGS} \
            -e BUILD_ENVIRONMENT \
            -e MAX_JOBS="$(nproc --ignore=2)" \
            -e PR_NUMBER \
            -e SHA1 \
            -e BRANCH \
            -e SCCACHE_BUCKET \
            -e SCCACHE_REGION \
            -e XLA_CUDA \
            -e XLA_CLANG_CACHE_S3_BUCKET_NAME \
            -e SKIP_SCCACHE_INITIALIZATION=1 \
            -e TORCH_CUDA_ARCH_LIST \
            -e PR_LABELS \
            -e OUR_GITHUB_JOB_ID \
            -e HUGGING_FACE_HUB_TOKEN \
            -e BUILD_ADDITIONAL_PACKAGES \
            -e RUNNER \
            --memory="${TOTAL_AVAILABLE_MEMORY_IN_GB%.*}g" \
            --memory-swap="${TOTAL_MEMORY_WITH_SWAP}g" \
            --env-file="/tmp/github_env_${GITHUB_RUN_ID}" \
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-build.yml`.
- CN: 该部分描述 `.github/workflows/_linux-build.yml` 的仓库自动化行为。

### Lines 339-359 / 第 339-359 行

````yaml
            --security-opt seccomp=unconfined \
            --cap-add=SYS_PTRACE \
            --tty \
            --detach \
            ${JENKINS_USER} \
            -v "${GITHUB_WORKSPACE}:/var/lib/jenkins/workspace" \
            -w /var/lib/jenkins/workspace \
            "${USED_IMAGE}" \
            ${DOCKER_SHELL_CMD}
          )

          if [[ ${BUILD_ENVIRONMENT} == *"s390x"* ]]; then
            # sometimes there are intermittent network errors, do a couple of retries
            docker exec -t "${container_name}" sh -c "python3 -m pip install -r requirements.txt || \
                (sleep 15 && python3 -m pip install -r requirements.txt) || \
                (sleep 30 && python3 -m pip install -r requirements.txt) || \
                (sleep 60 && python3 -m pip install -r requirements.txt) || \
                (sleep 300 && python3 -m pip install -r requirements.txt)"
          fi

          docker exec -t "${container_name}" sh -c '.ci/pytorch/build.sh'
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-build.yml`.
- CN: 该部分描述 `.github/workflows/_linux-build.yml` 的仓库自动化行为。

### Lines 361-382 / 第 361-382 行

````yaml
          END_TIME=$(date +%s)
          echo "build_time=$((END_TIME - START_TIME))" >> "$GITHUB_OUTPUT"

      - name: Build external packages
        id: build-external-packages
        if: inputs.build-external-packages != '' &&  steps.build.outcome != 'skipped'
        uses: ./.github/actions/build-external-packages
        with:
          build-targets: ${{ inputs.build-external-packages }}
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
          cuda-arch-list: ${{ inputs.cuda-arch-list }}
          output-dir: external

      - name: Move external packages to dist
        if: steps.build-external-packages.outputs.output_dir != '' && steps.build-external-packages.outcome != 'skipped'
        shell: bash
        run: |
          src="${{ steps.build-external-packages.outputs.output_dir }}"
          if [ -d "$src" ]; then
            mkdir -p "dist/$(dirname "$src")"
            mv "$src" "dist/$(dirname "$src")/"
          fi
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/build-external-packages`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 384-406 / 第 384-406 行

````yaml
      - name: Stop monitoring script
        if: ${{ always() && steps.monitor-script.outputs.monitor-script-pid }}
        shell: bash
        continue-on-error: true
        env:
          MONITOR_SCRIPT_PID: ${{ steps.monitor-script.outputs.monitor-script-pid }}
        run: |
          kill "$MONITOR_SCRIPT_PID"

      - name: Archive artifacts into zip
        if: inputs.build-generates-artifacts && steps.build.outcome != 'skipped' && steps.use-old-whl.outputs.reuse != 'true'
        run: |
          zip -1 -r artifacts.zip dist/ build/custom_test_artifacts build/lib build/bin .additional_ci_files

      - name: Store PyTorch Build Artifacts on S3
        uses: seemethere/upload-artifact-s3@baba72d0712b404f646cebe0730933554ebce96a # v5.1.0
        if: inputs.build-generates-artifacts && (steps.build.outcome != 'skipped' || steps.use-old-whl.outputs.reuse == 'true') && inputs.build-environment != 'linux-s390x-binary-manywheel'
        with:
          name: ${{ inputs.build-environment }}
          retention-days: 14
          if-no-files-found: error
          path: artifacts.zip
          s3-bucket: ${{ inputs.s3-bucket }}
````

- EN: This section reuses actions and step building blocks such as `seemethere/upload-artifact-s3@baba72d0712b404f646cebe0730933554ebce96a`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 408-428 / 第 408-428 行

````yaml
      - name: Store PyTorch Build Artifacts for s390x
        uses: actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 # v4.6.2
        if: inputs.build-generates-artifacts && (steps.build.outcome != 'skipped' || steps.use-old-whl.outputs.reuse == 'true') && inputs.build-environment == 'linux-s390x-binary-manywheel'
        with:
          name: ${{ inputs.build-environment }}
          retention-days: 14
          if-no-files-found: error
          path: artifacts.zip

      - name: Copy logs
        shell: bash
        if: ${{ always() && steps.build.outcome != 'skipped' && !inputs.disable-monitor && inputs.build-environment != 'linux-s390x-binary-manywheel'}}
        continue-on-error: true
        run: |
          rm -f ./usage_logs
          mkdir -p ./usage_logs
          cp ../../usage_logs/usage_log_build_*.txt ./usage_logs/

      - name: Upload raw usage log to s3
        if: ${{ always() && steps.build.outcome != 'skipped' && !inputs.disable-monitor && inputs.build-environment != 'linux-s390x-binary-manywheel'}}
        uses: seemethere/upload-artifact-s3@baba72d0712b404f646cebe0730933554ebce96a # v5
````

- EN: This section reuses actions and step building blocks such as `actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02`, `seemethere/upload-artifact-s3@baba72d0712b404f646cebe0730933554ebce96a`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 429-446 / 第 429-446 行

````yaml
        with:
          s3-prefix: |
            ${{ github.repository }}/${{ github.run_id }}/${{ github.run_attempt }}/artifact
          retention-days: 14
          if-no-files-found: warn
          path: usage_logs/usage_log_build_*.txt

      - name: Upload sccache stats
        if: steps.build.outcome != 'skipped' && inputs.build-environment != 'linux-s390x-binary-manywheel'
        uses: ./.github/actions/upload-sccache-stats
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          build-time: ${{ steps.build.outputs.build_time }}

      - name: Upload utilization stats
        if: ${{ always() && steps.build.outcome != 'skipped' && !inputs.disable-monitor && inputs.build-environment != 'linux-s390x-binary-manywheel' }}
        continue-on-error: true
        uses: ./.github/actions/upload-utilization-stats
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/upload-sccache-stats`, `./.github/actions/upload-utilization-stats`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 447-467 / 第 447-467 行

````yaml
        with:
          job_id: ${{ steps.setup-linux.outputs.job-id }}
          job_name: ${{ steps.setup-linux.outputs.job-name }}
          workflow_name: ${{ github.workflow }}
          workflow_run_id: ${{github.run_id}}
          workflow_attempt: ${{github.run_attempt}}
          artifact_prefix: usage_log_build_${{ steps.setup-linux.outputs.job-id }}

      - name: Teardown Linux
        uses: pytorch/test-infra/.github/actions/teardown-linux@main
        if: always() && inputs.build-environment != 'linux-s390x-binary-manywheel'

      - name: Cleanup docker
        if: always() && inputs.build-environment == 'linux-s390x-binary-manywheel'
        shell: bash
        run: |
          # on s390x stop the container for clean worker stop
          docker stop -a || true
          docker kill -a || true

  build-osdc:
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/teardown-linux@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 468-485 / 第 468-485 行

````yaml
    permissions:
      id-token: write
      contents: read
      actions: read
    # Don't run on forked repos
    if: github.repository_owner == 'pytorch' && inputs.use-arc
    runs-on: ${{ inputs.runner_prefix }}${{ startsWith(inputs.runner, 'l-') && inputs.runner || contains(inputs.runner, 'arm64') && 'l-arm64g4-16-62' || 'l-x86iavx512-8-64' }}
    container:
      image: ghcr.io/pytorch/${{ inputs.docker-image-name }}
    timeout-minutes: 480
    outputs:
      docker-image: ghcr.io/pytorch/${{ inputs.docker-image-name }}
      test-matrix: ${{ steps.map-runners.outputs.test-matrix }}
      build-environment: ${{ inputs.build-environment }}
    steps:
      - name: Setup Linux
        id: setup-linux
        uses: pytorch/pytorch/.github/actions/setup-linux@main
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/setup-linux@main`; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 486-507 / 第 486-507 行

````yaml
        with:
          use-arc: true
          python-version: ${{ inputs.python-version }}
          compiler: ${{ inputs.compiler }}
          cuda-version: ${{ inputs.cuda-version }}
          github-token: ${{ secrets.GITHUB_TOKEN }}

      - name: Check if can use old whl build
        id: use-old-whl
        uses: pytorch/pytorch/.github/actions/reuse-old-whl@main
        if: ${{ inputs.allow-reuse-old-whl }}
        with:
          build-environment: ${{ inputs.build-environment }}
          run-id: ${{ github.run_id }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          job-id: ${{ steps.setup-linux.outputs.job-id }}
          job-name: ${{ steps.setup-linux.outputs.job-name }}

      # Apply the filter logic to the build step too if the test-config label is already there
      - name: Select all requested test configurations (if the test matrix is available)
        id: filter
        uses: pytorch/pytorch/.github/actions/filter-test-configs@main
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/reuse-old-whl@main`, `pytorch/pytorch/.github/actions/filter-test-configs@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 508-529 / 第 508-529 行

````yaml
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          test-matrix: ${{ inputs.test-matrix }}
          selected-test-configs: ${{ inputs.selected-test-configs }}
          job-name: ${{ steps.setup-linux.outputs.job-name }}

      - name: Map EC2 runners to ARC runners
        id: map-runners
        env:
          FILTERED_TEST_MATRIX: ${{ steps.filter.outputs.test-matrix }}
          RUNNER_PREFIX: ${{ inputs.runner_prefix }}
        shell: bash
        run: |
          python3 .github/scripts/map_ec2_to_arc.py --prefix "${RUNNER_PREFIX}" "${FILTERED_TEST_MATRIX}"

      - name: Configure AWS credentials
        id: aws-creds
        continue-on-error: true
        uses: aws-actions/configure-aws-credentials@8df5847569e6427dd6c4fb1cf565c83acfa8afa7
        with:
          role-to-assume: arn:aws:iam::308535385114:role/arc
          aws-region: us-east-1
````

- EN: This section reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@8df5847569e6427dd6c4fb1cf565c83acfa8afa7`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 530-553 / 第 530-553 行

````yaml
          # The max duration enforced by the server side
          role-duration-seconds: 18000

      - name: Build
        if: (steps.filter.outputs.is-test-matrix-empty == 'False' || inputs.test-matrix == '') && steps.use-old-whl.outputs.reuse != 'true'
        id: build
        env:
          BUILD_ENVIRONMENT: ${{ inputs.build-environment }}
          BRANCH: ${{ steps.setup-linux.outputs.branch }}
          PR_NUMBER: ${{ github.event.pull_request.number }}
          SHA1: ${{ github.event.pull_request.head.sha || github.sha }}
          SCCACHE_BUCKET: ossci-compiler-cache-circleci-v2
          SCCACHE_REGION: us-east-1
          SCCACHE_S3_NO_CREDENTIALS: ${{ steps.aws-creds.outcome != 'success' && 'true' || 'false' }}
          XLA_CLANG_CACHE_S3_BUCKET_NAME: ossci-compiler-clang-cache-circleci-xla
          PR_LABELS: ${{ toJson(github.event.pull_request.labels.*.name) }}
          TORCH_CUDA_ARCH_LIST: ${{ inputs.cuda-arch-list }}
          XLA_CUDA: ${{ contains(inputs.build-environment, 'xla') && '0' || '' }}
          OUR_GITHUB_JOB_ID: ${{ steps.setup-linux.outputs.job-id }}
          HUGGING_FACE_HUB_TOKEN: ${{ secrets.HUGGING_FACE_HUB_TOKEN }}
          BUILD_ADDITIONAL_PACKAGES: ${{ inputs.build-additional-packages }}
          RUNNER: ${{ inputs.runner }}
          SKIP_SCCACHE_INITIALIZATION: 1
        shell: bash
````

- EN: This section declares workflow triggers or event bindings; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 554-568 / 第 554-568 行

````yaml
        run: |
          START_TIME=$(date +%s)
          .ci/pytorch/build.sh
          END_TIME=$(date +%s)
          echo "build_time=$((END_TIME - START_TIME))" >> "$GITHUB_OUTPUT"

      - name: Build external packages
        id: build-external-packages
        if: inputs.build-external-packages != '' &&  steps.build.outcome != 'skipped'
        uses: pytorch/pytorch/.github/actions/build-external-packages@main
        with:
          build-targets: ${{ inputs.build-external-packages }}
          docker-image: ghcr.io/pytorch/${{ inputs.docker-image-name }}
          cuda-arch-list: ${{ inputs.cuda-arch-list }}
          output-dir: external
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/build-external-packages@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 570-591 / 第 570-591 行

````yaml
      - name: Move external packages to dist
        if: steps.build-external-packages.outputs.output_dir != '' && steps.build-external-packages.outcome != 'skipped'
        shell: bash
        run: |
          src="${{ steps.build-external-packages.outputs.output_dir }}"
          if [ -d "$src" ]; then
            mkdir -p "dist/$(dirname "$src")"
            mv "$src" "dist/$(dirname "$src")/"
          fi

      - name: Archive artifacts into zip
        if: inputs.build-generates-artifacts && steps.build.outcome != 'skipped' && steps.use-old-whl.outputs.reuse != 'true'
        run: |
          zip -1 -r artifacts.zip dist/ build/custom_test_artifacts build/lib build/bin .additional_ci_files

      - name: Store build artifacts
        if: inputs.build-generates-artifacts && (steps.build.outcome != 'skipped' || steps.use-old-whl.outputs.reuse == 'true')
        uses: pytorch/pytorch/.github/actions/upload-build-artifacts@main
        with:
          name: ${{ inputs.build-environment }}
          s3-bucket: ${{ inputs.s3-bucket }}
          use-gha: ${{ steps.aws-creds.outcome != 'success' }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/upload-build-artifacts@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 593-595 / 第 593-595 行

````yaml
      - name: Upload sccache stats
        if: steps.build.outcome != 'skipped' && steps.aws-creds.outcome == 'success'
        uses: pytorch/pytorch/.github/actions/upload-sccache-stats@main
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/upload-sccache-stats@main`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 596-598 / 第 596-598 行

````yaml
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          build-time: ${{ steps.build.outputs.build_time }}
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
- Reusable actions / 复用 Action: `pytorch/test-infra/.github/actions/setup-ssh@main`, `pytorch/pytorch/.github/actions/setup-linux@main`, `./.github/actions/ecr-login`, `./.github/actions/reuse-old-whl`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/filter-test-configs`, `./.github/actions/build-external-packages`, ...
- Inline commands / 内联命令: `If`, `job`, `List`, `Label`, `An`, `is`, `logic`, `A`, ...
- Environment variables / 环境变量: `CUDA`, `JSON`, `ARC`, `OSDC`, `EC2`, `HUGGING_FACE_HUB_TOKEN`, `SSH`, `GITHUB_TOKEN`, `ECR`, `ECR_DOCKER_IMAGE`, ...
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `inputs`, `secrets`, `outputs`, `jobs`, `build`, `steps`, `build-osdc`, `permissions`, ...
