# _linux-test-stable-fa3.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/_linux-test-stable-fa3.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-15 / 第 1-15 行

````yaml
# The point of this workflow is to test that a FA3 wheel that was built based off the
# stable ABI as of torch nightly 20250830 can still run on the newer torch.
#
# This workflow is very similar to the _linux-test.yml workflow, with the following
# differences:
#   1. It is simpler (there is no test matrix)
#   2. It pulls flash-attention as a secondary repository in order to access the tests.
#      Note that it does not BUILD anything from flash-attention, as we have a prebuilt
#      wheel. We pull flash-attention only to run a few tests.
#   3. It runs only FA3 tests. No PyTorch tests are run.
name: linux-test-stable-fa3

on:
  workflow_call:
    inputs:
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 16-29 / 第 16-29 行

````yaml
      build-environment:
        required: true
        type: string
        description: Top-level label for what's being built/tested.
      docker-image:
        required: true
        type: string
        description: Docker image to run in.
      timeout-minutes:
        required: false
        type: number
        default: 60
        description: |
          Set the maximum (in minutes) how long the workflow should take to finish
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-test-stable-fa3.yml`.
- CN: 该部分描述 `.github/workflows/_linux-test-stable-fa3.yml` 的仓库自动化行为。

### Lines 30-43 / 第 30-43 行

````yaml
      s3-bucket:
        description: S3 bucket to download artifact
        required: false
        type: string
        default: "gha-artifacts"
    secrets:
      HUGGING_FACE_HUB_TOKEN:
        required: false
        description: |
          HF Auth token to avoid rate limits when downloading models or datasets from hub
      VLLM_TEST_HUGGING_FACE_TOKEN:
        required: false
        description: |
          HF Auth token to test vllm
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-test-stable-fa3.yml`.
- CN: 该部分描述 `.github/workflows/_linux-test-stable-fa3.yml` 的仓库自动化行为。

### Lines 44-60 / 第 44-60 行

````yaml
      SCRIBE_GRAPHQL_ACCESS_TOKEN:
        required: false
        description: |
          FB app token to write to scribe endpoint

env:
  GIT_DEFAULT_BRANCH: ${{ github.event.repository.default_branch }}

jobs:
  test:
    # Don't run on forked repos
    if: github.repository_owner == 'pytorch'
    runs-on: linux.aws.h100
    timeout-minutes: ${{ inputs.timeout-minutes || 60 }}
    permissions:
      id-token: write
      contents: read
````

- EN: This section lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 61-76 / 第 61-76 行

````yaml
    steps:
      - name: Setup Linux
        uses: pytorch/pytorch/.github/actions/setup-linux@main

      - name: Checkout flash-attention as a secondary repository
        uses: actions/checkout@v4
        with:
          repository: Dao-AILab/flash-attention
          path: flash-attention

      - name: Login to ECR
        uses: ./.github/actions/ecr-login

      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/setup-linux@main`, `actions/checkout@v4`, `./.github/actions/ecr-login`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 77-92 / 第 77-92 行

````yaml
        with:
          docker-image-name: ${{ inputs.docker-image }}

      - name: Use following to pull public copy of the image
        id: print-ghcr-mirror
        env:
          ECR_DOCKER_IMAGE: ${{ steps.calculate-docker-image.outputs.docker-image }}
        shell: bash
        run: |
          tag=${ECR_DOCKER_IMAGE##*:}
          echo "docker pull ghcr.io/pytorch/ci-image:${tag/:/-}"

      - name: Pull docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/pull-docker-image@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 94-106 / 第 94-106 行

````yaml
      - name: Check if in a container runner
        shell: bash
        id: check_container_runner
        run: echo "IN_CONTAINER_RUNNER=$(if [ -f /.inarc ] || [ -f /.incontainer ]; then echo true ; else echo false; fi)" >> "$GITHUB_OUTPUT"

      - name: Setup GPU_FLAG for docker run
        id: setup-gpu-flag
        run: echo "GPU_FLAG=--gpus all -e NVIDIA_DRIVER_CAPABILITIES=all" >> "${GITHUB_ENV}"

      - name: Setup SCCACHE_SERVER_PORT environment for docker run when on container
        id: setup-sscache-port-flag
        run: echo "SCCACHE_SERVER_PORT_DOCKER_FLAG=-e SCCACHE_SERVER_PORT=$((RUNNER_UID + 4226))" >> "${GITHUB_ENV}"
        if: ${{ steps.check_container_runner.outputs.IN_CONTAINER_RUNNER == 'true' }}
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 108-123 / 第 108-123 行

````yaml
      - name: Get workflow job id
        id: get-job-id
        uses: ./.github/actions/get-workflow-job-id
        if: always()
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}

      - name: Download build artifacts
        uses: ./.github/actions/download-build-artifacts
        with:
          name: ${{ inputs.build-environment }}
          s3-bucket: ${{ inputs.s3-bucket }}

      - name: Parse ref
        id: parse-ref
        run: .github/scripts/parse_ref.py
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/get-workflow-job-id`, `./.github/actions/download-build-artifacts`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 125-141 / 第 125-141 行

````yaml
      - name: Set Test step time
        id: test-timeout
        shell: bash
        env:
          JOB_TIMEOUT: ${{ inputs.timeout-minutes }}
        run: |
          echo "timeout=$((JOB_TIMEOUT-30))" >> "${GITHUB_OUTPUT}"

      - name: Preserve github env variables for use in docker
        shell: bash
        run: |
          env | grep '^GITHUB' >> "/tmp/github_env_${GITHUB_RUN_ID}"
          env | grep '^CI' >> "/tmp/github_env_${GITHUB_RUN_ID}"

      - name: Test
        id: test
        timeout-minutes: ${{ fromJson(steps.test-timeout.outputs.timeout) }}
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 142-159 / 第 142-159 行

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
          JOB_ID: ${{ steps.get-job-id.outputs.job-id }}
          JOB_NAME: ${{ steps.get-job-id.outputs.job-name }}
          BRANCH: ${{ steps.parse-ref.outputs.branch }}
          SHA1: ${{ github.event.pull_request.head.sha || github.sha }}
          BASE_SHA: ${{ github.event.pull_request.base.sha || github.sha }}
          SHM_SIZE: '2g'
          DOCKER_IMAGE: ${{ inputs.docker-image }}
          VLLM_TEST_HUGGING_FACE_TOKEN: ${{ secrets.VLLM_TEST_HUGGING_FACE_TOKEN }}
          HUGGING_FACE_HUB_TOKEN: ${{ secrets.HUGGING_FACE_HUB_TOKEN }}
````

- EN: This section declares workflow triggers or event bindings; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；控制令牌权限范围或环境变量以实现安全执行。

### Lines 160-176 / 第 160-176 行

````yaml
          SCRIBE_GRAPHQL_ACCESS_TOKEN: ${{ secrets.SCRIBE_GRAPHQL_ACCESS_TOKEN }}
          ARTIFACTS_FILE_SUFFIX: ${{ github.job }}-${{ steps.get-job-id.outputs.job-id }}
        run: |
          set -x

          TEST_COMMAND=.ci/pytorch/test_fa3_abi_stable.sh

          # Leaving 1GB for the runner and other things
          TOTAL_AVAILABLE_MEMORY_IN_GB=$(awk '/MemTotal/ { printf "%.3f \n", $2/1024/1024 - 1 }' /proc/meminfo)
          # https://docs.docker.com/engine/containers/resource_constraints/#--memory-swap-details, the 3GB swap
          # comes from https://github.com/pytorch/test-infra/pull/6058
          TOTAL_MEMORY_WITH_SWAP=$(("${TOTAL_AVAILABLE_MEMORY_IN_GB%.*}" + 3))


          SHM_OPTS="--shm-size=${SHM_SIZE}"
          JENKINS_USER="--user jenkins"
          DOCKER_SHELL_CMD=
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 178-195 / 第 178-195 行

````yaml
          # detached container should get cleaned up by teardown_ec2_linux
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
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-test-stable-fa3.yml`.
- CN: 该部分描述 `.github/workflows/_linux-test-stable-fa3.yml` 的仓库自动化行为。

### Lines 196-213 / 第 196-213 行

````yaml
            -e BASE_SHA \
            -e BRANCH \
            -e SHA1 \
            -e MAX_JOBS="$(nproc --ignore=2)" \
            -e HUGGING_FACE_HUB_TOKEN \
            -e VLLM_TEST_HUGGING_FACE_TOKEN \
            -e SCRIBE_GRAPHQL_ACCESS_TOKEN \
            -e ARTIFACTS_FILE_SUFFIX \
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
````

- EN: This section describes repository automation behavior for `.github/workflows/_linux-test-stable-fa3.yml`.
- CN: 该部分描述 `.github/workflows/_linux-test-stable-fa3.yml` 的仓库自动化行为。

### Lines 214-229 / 第 214-229 行

````yaml
            ${JENKINS_USER} \
            -v "${GITHUB_WORKSPACE}:/var/lib/jenkins/workspace" \
            -w /var/lib/jenkins/workspace \
            "${DOCKER_IMAGE}" \
            ${DOCKER_SHELL_CMD}
          )

          echo "DOCKER_CONTAINER_ID=${container_name}" >> "${GITHUB_ENV}"

          docker exec -t "${container_name}" sh -c "python3 -m pip install $(echo dist/*.whl)[opt-einsum] && ${TEST_COMMAND}"

      - name: Collect backtraces from coredumps (if any)
        if: always()
        run: |
          # shellcheck disable=SC2156
          find . -iname "core.[1-9]*" -exec docker exec "${DOCKER_CONTAINER_ID}" sh -c "gdb python {} -ex 'bt' -ex 'q'" \;
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 231-243 / 第 231-243 行

````yaml
      - name: Store Core dumps on S3
        uses: seemethere/upload-artifact-s3@baba72d0712b404f646cebe0730933554ebce96a # v5.1.0
        if: failure()
        with:
          name: coredumps-fa3-stable-abi-smoke-tests
          retention-days: 14
          if-no-files-found: ignore
          path: ./**/core.[1-9]*

      - name: Upload utilization stats
        if: ${{ always() && steps.test.conclusion && steps.test.conclusion != 'skipped' }}
        continue-on-error: true
        uses: ./.github/actions/upload-utilization-stats
````

- EN: This section reuses actions and step building blocks such as `seemethere/upload-artifact-s3@baba72d0712b404f646cebe0730933554ebce96a`, `./.github/actions/upload-utilization-stats`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 244-249 / 第 244-249 行

````yaml
        with:
          job_id: ${{ steps.get-job-id.outputs.job-id }}
          job_name: ${{ steps.get-job-id.outputs.job-name }}
          workflow_name: ${{ github.workflow }}
          workflow_run_id: ${{github.run_id}}
          workflow_attempt: ${{github.run_attempt}}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 251-253 / 第 251-253 行

````yaml
      - name: Teardown Linux
        uses: pytorch/test-infra/.github/actions/teardown-linux@main
        if: always() && steps.check_container_runner.outputs.IN_CONTAINER_RUNNER == 'false'
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/teardown-linux@main`.
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
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/actions/setup-linux@main`, `actions/checkout@v4`, `./.github/actions/ecr-login`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/get-workflow-job-id`, `./.github/actions/download-build-artifacts`, `seemethere/upload-artifact-s3@baba72d0712b404f646cebe0730933554ebce96a`, ...
- Inline commands / 内联命令: `Set`, `HF`, `FB`, `echo`, `env`, `set`, `e`, `memory="${TOTAL_AVAILABLE_MEMORY_IN_GB%.*}g"`, ...
- Environment variables / 环境变量: `FA3`, `ABI`, `BUILD`, `HUGGING_FACE_HUB_TOKEN`, `VLLM_TEST_HUGGING_FACE_TOKEN`, `SCRIBE_GRAPHQL_ACCESS_TOKEN`, `GIT_DEFAULT_BRANCH`, `ECR`, `ECR_DOCKER_IMAGE`, `IN_CONTAINER_RUNNER`, ...
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `inputs`, `secrets`, `env`, `jobs`, `test`, `permissions`, `steps`
