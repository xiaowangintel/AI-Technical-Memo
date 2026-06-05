# _rocm-test.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/_rocm-test.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-17 / 第 1-17 行

````yaml
# TODO: this looks sort of similar to _linux-test, but there are like a dozen
# places where you would have to insert an if statement. Probably it's better to
# just use a different workflow altogether

name: test

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
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 18-34 / 第 18-34 行

````yaml
      docker-image:
        required: true
        type: string
        description: Docker image to run in.
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
        default: 300
        description: |
          Set the maximum (in minutes) how long the workflow should take to finish
````

- EN: This section describes repository automation behavior for `.github/workflows/_rocm-test.yml`.
- CN: 该部分描述 `.github/workflows/_rocm-test.yml` 的仓库自动化行为。

### Lines 35-52 / 第 35-52 行

````yaml
      tests-to-include:
        required: false
        type: string
        default: ""
        description: |
          List of tests to include (empty string implies default list)
      dashboard-tag:
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
        default: true
````

- EN: This section describes repository automation behavior for `.github/workflows/_rocm-test.yml`.
- CN: 该部分描述 `.github/workflows/_rocm-test.yml` 的仓库自动化行为。

### Lines 53-69 / 第 53-69 行

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
    secrets:
      HUGGING_FACE_HUB_TOKEN:
        required: false
        description: |
          HF Auth token to avoid rate limits when downloading models or datasets from hub
````

- EN: This section describes repository automation behavior for `.github/workflows/_rocm-test.yml`.
- CN: 该部分描述 `.github/workflows/_rocm-test.yml` 的仓库自动化行为。

### Lines 70-86 / 第 70-86 行

````yaml
env:
  GIT_DEFAULT_BRANCH: ${{ github.event.repository.default_branch }}

permissions:
  id-token: write
  contents: read

jobs:
  test:
    # Don't run on forked repos or empty test matrix
    if: github.repository_owner == 'pytorch' && toJSON(fromJSON(inputs.test-matrix).include) != '[]'
    strategy:
      matrix: ${{ fromJSON(inputs.test-matrix) }}
      fail-fast: false
    runs-on: ${{ matrix.runner }}
    timeout-minutes: ${{ matrix.mem_leak_check == 'mem_leak_check' && 600 || inputs.timeout-minutes }}
    steps:
````

- EN: This section lays out job topology or execution stages; reuses actions and step building blocks such as named actions; controls token scope or environment variables for safe execution.
- CN: 该部分铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 87-103 / 第 87-103 行

````yaml
      - name: Checkout PyTorch
        uses: pytorch/pytorch/.github/actions/checkout-pytorch@main
        with:
          no-sudo: true

      - name: Setup ROCm
        uses: ./.github/actions/setup-rocm

      - name: Runner check GPU count (distributed jobs)
        if: ${{ contains(matrix.config, 'distributed') }}
        shell: bash
        run: |
          ngpu=$(rocminfo | grep -c -E 'Name:.*\sgfx')
          if [[ $ngpu -lt 2 ]]; then #We are temporarily reducing this down to 2 from 4 so that we can run tests on nodes with less gpus.
            echo "Error: only $ngpu GPU(s) detected, at least 2 GPUs are needed for distributed jobs"
            exit 1
          fi
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `./.github/actions/setup-rocm`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 105-121 / 第 105-121 行

````yaml
      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-image-name: ${{ inputs.docker-image }}

      - name: Pull docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.calculate-docker-image.outputs.docker-image }}

      - name: Get workflow job id
        id: get-job-id
        uses: ./.github/actions/get-workflow-job-id
        if: always()
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/get-workflow-job-id`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 123-138 / 第 123-138 行

````yaml
      - name: Start monitoring script
        id: monitor-script
        if: ${{ !inputs.disable-monitor }}
        shell: bash
        continue-on-error: true
        env:
          JOB_ID: ${{ steps.get-job-id.outputs.job-id }}
          JOB_NAME: ${{ steps.get-job-id.outputs.job-name }}
          WORKFLOW_NAME: ${{ github.workflow }}
          WORKFLOW_RUN_ID: ${{github.run_id}}
          MONITOR_LOG_INTERVAL: ${{ inputs.monitor-log-interval }}
          MONITOR_DATA_COLLECT_INTERVAL: ${{ inputs.monitor-data-collect-interval }}
        run: |
          python3 -m pip install psutil==5.9.8 dataclasses_json==0.6.7
          python3 -m tools.stats.monitor --log-interval "$MONITOR_LOG_INTERVAL" --data-collect-interval "$MONITOR_DATA_COLLECT_INTERVAL" > usage_log.txt 2>&1 &
          echo "monitor-script-pid=${!}" >> "${GITHUB_OUTPUT}"
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 140-155 / 第 140-155 行

````yaml
      - name: Download build artifacts
        uses: ./.github/actions/download-build-artifacts
        with:
          name: ${{ inputs.build-environment }}

      - name: Download TD artifacts
        continue-on-error: true
        uses: ./.github/actions/download-td-artifacts

      - name: Parse ref
        id: parse-ref
        run: .github/scripts/parse_ref.py

      - name: Check for keep-going label and re-enabled test issues
        # This uses the filter-test-configs action because it conveniently
        # checks for labels and re-enabled test issues.  It does not actually do
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/download-build-artifacts`, `./.github/actions/download-td-artifacts`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 156-170 / 第 156-170 行

````yaml
        # any filtering.  All filtering is done in the build step.
        id: keep-going
        uses: ./.github/actions/filter-test-configs
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          test-matrix: ${{ inputs.test-matrix }}
          job-name: ${{ steps.get-job-id.outputs.job-name }}

      - name: Set Test step time
        id: test-timeout
        shell: bash
        env:
          JOB_TIMEOUT: ${{ matrix.mem_leak_check == 'mem_leak_check' && 600 || inputs.timeout-minutes }}
        run: |
          echo "timeout=$((JOB_TIMEOUT-30))" >> "${GITHUB_OUTPUT}"
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/filter-test-configs`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 172-189 / 第 172-189 行

````yaml
      - name: Preserve github env variables for use in docker
        shell: bash
        run: |
          env | grep '^GITHUB' >> "/tmp/github_env_${GITHUB_RUN_ID}"
          env | grep '^CI' >> "/tmp/github_env_${GITHUB_RUN_ID}"

      - name: Test
        id: test
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
````

- EN: This section declares workflow triggers or event bindings; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 190-207 / 第 190-207 行

````yaml
          JOB_NAME: ${{ steps.get-job-id.outputs.job-name }}
          BRANCH: ${{ steps.parse-ref.outputs.branch }}
          SHA1: ${{ github.event.pull_request.head.sha || github.sha }}
          BASE_SHA: ${{ github.event.pull_request.base.sha || github.sha }}
          TEST_CONFIG: ${{ matrix.config }}
          SHARD_NUMBER: ${{ matrix.shard }}
          NUM_TEST_SHARDS: ${{ matrix.num_shards }}
          REENABLED_ISSUES: ${{ steps.keep-going.outputs.reenabled-issues }}
          CONTINUE_THROUGH_ERROR: ${{ steps.keep-going.outputs.keep-going }}
          VERBOSE_TEST_LOGS: ${{ steps.keep-going.outputs.ci-verbose-test-logs }}
          TEST_SHOWLOCALS: ${{ steps.keep-going.outputs.ci-test-showlocals }}
          NO_TEST_TIMEOUT: ${{ steps.keep-going.outputs.ci-no-test-timeout }}
          NO_TD: ${{ steps.keep-going.outputs.ci-no-td }}
          DOCKER_IMAGE: ${{ inputs.docker-image }}
          PYTORCH_TEST_CUDA_MEM_LEAK_CHECK: ${{ matrix.mem_leak_check && '1' || '0' }}
          PYTORCH_TEST_RERUN_DISABLED_TESTS: ${{ matrix.rerun_disabled_tests && '1' || '0' }}
          TESTS_TO_INCLUDE: ${{ inputs.tests-to-include }}
          DASHBOARD_TAG: ${{ inputs.dashboard-tag }}
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 208-223 / 第 208-223 行

````yaml
          HUGGING_FACE_HUB_TOKEN: ${{ secrets.HUGGING_FACE_HUB_TOKEN }}
        timeout-minutes: ${{ fromJson(steps.test-timeout.outputs.timeout) }}
        run: |
          set -x

          if [[ $TEST_CONFIG == 'multigpu' ]]; then
            TEST_COMMAND=.ci/pytorch/multigpu-test.sh
          elif [[ $BUILD_ENVIRONMENT == *onnx* ]]; then
            TEST_COMMAND=.ci/caffe2/test.sh
          else
            TEST_COMMAND=.ci/pytorch/test.sh
          fi

          # detached container should get cleaned up by teardown_ec2_linux
          # TODO: Stop building test binaries as part of the build phase
          # Used for GPU_FLAG since that doesn't play nice
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 224-241 / 第 224-241 行

````yaml
          # shellcheck disable=SC2086,SC2090
          container_name=$(docker run \
            ${GPU_FLAG:-} \
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
````

- EN: This section describes repository automation behavior for `.github/workflows/_rocm-test.yml`.
- CN: 该部分描述 `.github/workflows/_rocm-test.yml` 的仓库自动化行为。

### Lines 242-259 / 第 242-259 行

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
            -e MAX_JOBS="$(nproc --ignore=2)" \
            -e PYTORCH_TEST_CUDA_MEM_LEAK_CHECK \
            -e PYTORCH_TEST_RERUN_DISABLED_TESTS \
            -e TESTS_TO_INCLUDE \
            -e HUGGING_FACE_HUB_TOKEN \
            -e DASHBOARD_TAG \
            --env-file="${RUNNER_TEMP}/github_env_${GITHUB_RUN_ID}" \
            --ulimit stack=10485760:83886080 \
````

- EN: This section describes repository automation behavior for `.github/workflows/_rocm-test.yml`.
- CN: 该部分描述 `.github/workflows/_rocm-test.yml` 的仓库自动化行为。

### Lines 260-276 / 第 260-276 行

````yaml
            --ulimit core=0 \
            --env-file="/tmp/github_env_${GITHUB_RUN_ID}" \
            --security-opt seccomp=unconfined \
            --cap-add=SYS_PTRACE \
            --shm-size="8g" \
            --tty \
            --detach \
            --name="${container_name}" \
            --user jenkins \
            -v "${GITHUB_WORKSPACE}:/var/lib/jenkins/workspace" \
            -w /var/lib/jenkins/workspace \
            "${DOCKER_IMAGE}"
          )
          # save container name for later step
          echo "CONTAINER_NAME=${container_name}" >> "$GITHUB_ENV"
          # jenkins user does not have write permission to mounted workspace; work-around by copying within container to jenkins home
          docker exec -t "${container_name}" sh -c "cd .. && cp -R workspace pytorch && cd pytorch && pip install dist/*.whl && ${TEST_COMMAND}"
````

- EN: This section describes repository automation behavior for `.github/workflows/_rocm-test.yml`.
- CN: 该部分描述 `.github/workflows/_rocm-test.yml` 的仓库自动化行为。

### Lines 278-293 / 第 278-293 行

````yaml
      - name: Save test results
        if: always()
        run: |
          # copy test results back to the mounted workspace, needed sudo, resulting permissions were correct
          docker exec -t "${{ env.CONTAINER_NAME }}" sh -c "cd ../pytorch && sudo cp -R test/test-reports ../workspace/test"

      - name: Change permissions (only needed for kubernetes runners for now)
        if: ${{ always() && steps.test.conclusion && (contains(matrix.runner, 'gfx942') || contains(matrix.runner, 'gfx950') || contains(matrix.runner, 'mi210')) }}
        run: |
          docker exec -t "${{ env.CONTAINER_NAME }}" sh -c "sudo chown -R 1001:1001 test"

      - name: Print remaining test logs
        shell: bash
        if: always() && steps.test.conclusion
        run: |
          cat test/**/*_toprint.log || true
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 295-309 / 第 295-309 行

````yaml
      - name: Stop monitoring script
        if: ${{ always() && steps.monitor-script.outputs.monitor-script-pid }}
        shell: bash
        continue-on-error: true
        env:
          MONITOR_SCRIPT_PID: ${{ steps.monitor-script.outputs.monitor-script-pid }}
        run: |
          kill "$MONITOR_SCRIPT_PID"

      - name: Upload test artifacts
        uses: ./.github/actions/upload-test-artifacts
        if: always() && steps.test.conclusion && steps.test.conclusion != 'skipped'
        with:
          use-gha: true
          file-suffix: ${{ github.job }}-${{ matrix.config }}-${{ matrix.shard }}-${{ matrix.num_shards }}-${{ matrix.runner }}_${{ steps.get-job-id.outputs.job-id }}
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/upload-test-artifacts`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 311-327 / 第 311-327 行

````yaml
      - name: Collect backtraces from coredumps (if any)
        if: always()
        run: |
          # shellcheck disable=SC2156
          find . -iname "core.[1-9]*" -exec docker exec "${CONTAINER_NAME}" sh -c "gdb python {} -ex 'bt' -ex 'q'" \;

      - name: Store Core dumps on GitHub
        uses: actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 # v4.6.2
        if: failure()
        with:
          name: coredumps-${{ matrix.config }}-${{ matrix.shard }}-${{ matrix.num_shards }}-${{ matrix.runner }}
          retention-days: 14
          if-no-files-found: ignore
          path: ./**/core.[1-9]*

      - name: Authenticate with AWS
        uses: aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722 # v4.1.0
````

- EN: This section reuses actions and step building blocks such as `actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02`, `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 328-340 / 第 328-340 行

````yaml
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_upload-benchmark-results
          # The max duration enforced by the server side
          role-duration-seconds: 18000
          aws-region: us-east-1

      - name: Upload the benchmark results
        uses: pytorch/test-infra/.github/actions/upload-benchmark-results@main
        with:
          benchmark-results-dir: test/test-reports
          dry-run: false
          schema-version: v3
          github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/upload-benchmark-results@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 342-351 / 第 342-351 行

````yaml
      - name: Upload utilization stats
        if: ${{ always() && steps.test.conclusion && steps.test.conclusion != 'skipped' && !inputs.disable-monitor }}
        continue-on-error: true
        uses: ./.github/actions/upload-utilization-stats
        with:
          job_id: ${{ steps.get-job-id.outputs.job-id }}
          job_name: ${{ steps.get-job-id.outputs.job-name }}
          workflow_name: ${{ github.workflow }}
          workflow_run_id: ${{github.run_id}}
          workflow_attempt: ${{github.run_attempt}}
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/upload-utilization-stats`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 353-354 / 第 353-354 行

````yaml
      - name: Teardown ROCm
        uses: ./.github/actions/teardown-rocm
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/teardown-rocm`.
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
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `./.github/actions/setup-rocm`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/get-workflow-job-id`, `./.github/actions/download-build-artifacts`, `./.github/actions/download-td-artifacts`, `./.github/actions/filter-test-configs`, ...
- Inline commands / 内联命令: `If`, `job`, `Set`, `List`, `[Experimental]`, `Currently,`, `since`, `HF`, ...
- Environment variables / 环境变量: `TODO`, `JSON`, `HUGGING_FACE_HUB_TOKEN`, `GIT_DEFAULT_BRANCH`, `GPU`, `GITHUB_TOKEN`, `JOB_ID`, `JOB_NAME`, `WORKFLOW_NAME`, `WORKFLOW_RUN_ID`, ...
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `inputs`, `secrets`, `env`, `permissions`, `jobs`, `test`, `strategy`, `steps`
