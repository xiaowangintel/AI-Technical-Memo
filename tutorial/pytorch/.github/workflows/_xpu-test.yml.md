# _xpu-test.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/_xpu-test.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-17 / 第 1-17 行

````yaml
# TODO: this looks sort of similar to _linux-test, but there are like a dozen
# places where you would have to insert an if statement. Probably it's better to
# just use a different workflow altogether

name: xpu-test

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
        default: 390
        description: |
          Set the maximum (in minutes) how long the workflow should take to finish
````

- EN: This section describes repository automation behavior for `.github/workflows/_xpu-test.yml`.
- CN: 该部分描述 `.github/workflows/_xpu-test.yml` 的仓库自动化行为。

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

- EN: This section describes repository automation behavior for `.github/workflows/_xpu-test.yml`.
- CN: 该部分描述 `.github/workflows/_xpu-test.yml` 的仓库自动化行为。

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

- EN: This section describes repository automation behavior for `.github/workflows/_xpu-test.yml`.
- CN: 该部分描述 `.github/workflows/_xpu-test.yml` 的仓库自动化行为。

### Lines 70-86 / 第 70-86 行

````yaml
permissions:
  id-token: write
  contents: read

env:
  GIT_DEFAULT_BRANCH: ${{ github.event.repository.default_branch }}

jobs:
  test:
    # Don't run on forked repos or empty test matrix
    if: github.repository_owner == 'pytorch' && toJSON(fromJSON(inputs.test-matrix).include) != '[]'
    strategy:
      matrix: ${{ fromJSON(inputs.test-matrix) }}
      fail-fast: false
    timeout-minutes: ${{ matrix.mem_leak_check == 'mem_leak_check' && 600 || inputs.timeout-minutes }}
    runs-on: ${{ matrix.runner }}
    steps:
````

- EN: This section lays out job topology or execution stages; reuses actions and step building blocks such as named actions; controls token scope or environment variables for safe execution.
- CN: 该部分铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 87-101 / 第 87-101 行

````yaml
      # [see note: pytorch repo ref]
      - name: Checkout PyTorch
        uses: pytorch/pytorch/.github/actions/checkout-pytorch@main

      - name: Setup XPU
        uses: ./.github/actions/setup-xpu

      - name: Calculate docker image
        id: calculate-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-image-name: ${{ inputs.docker-image }}

      - name: Use following to pull public copy of the image
        id: print-ghcr-mirror
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `./.github/actions/setup-xpu`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 102-117 / 第 102-117 行

````yaml
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

      - name: Get workflow job id
        id: get-job-id
        uses: ./.github/actions/get-workflow-job-id
        if: always()
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/get-workflow-job-id`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 118-135 / 第 118-135 行

````yaml
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}

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
          python3 -m pip install psutil==5.9.8 dataclasses_json==0.6.7 nvidia-ml-py==11.525.84
          python3 -m tools.stats.monitor --log-interval "$MONITOR_LOG_INTERVAL" --data-collect-interval "$MONITOR_DATA_COLLECT_INTERVAL" > usage_log.txt 2>&1 &
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 136-152 / 第 136-152 行

````yaml
          echo "monitor-script-pid=${!}" >> "${GITHUB_OUTPUT}"

      - name: Download build artifacts
        uses: ./.github/actions/download-build-artifacts
        with:
          name: ${{ inputs.build-environment }}

      - name: Parse ref
        id: parse-ref
        run: .github/scripts/parse_ref.py

      - name: Check for keep-going label and re-enabled test issues
        # This uses the filter-test-configs action because it conveniently
        # checks for labels and re-enabled test issues.  It does not actually do
        # any filtering.  All filtering is done in the build step.
        id: keep-going
        uses: ./.github/actions/filter-test-configs
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/download-build-artifacts`, `./.github/actions/filter-test-configs`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 153-167 / 第 153-167 行

````yaml
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

      - name: Test
        id: test
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 168-185 / 第 168-185 行

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
          PYTORCH_RETRY_TEST_CASES: 1
          PYTORCH_OVERRIDE_FLAKY_SIGNAL: 1
          CONTINUE_THROUGH_ERROR: ${{ steps.keep-going.outputs.keep-going }}
          TEST_SHOWLOCALS: ${{ steps.keep-going.outputs.ci-test-showlocals }}
          VERBOSE_TEST_LOGS: ${{ steps.keep-going.outputs.ci-verbose-test-logs }}
````

- EN: This section declares workflow triggers or event bindings; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；控制令牌权限范围或环境变量以实现安全执行。

### Lines 186-200 / 第 186-200 行

````yaml
          NO_TEST_TIMEOUT: ${{ steps.keep-going.outputs.ci-no-test-timeout }}
          NO_TD: ${{ steps.keep-going.outputs.ci-no-td }}
          TEST_CONFIG: ${{ matrix.config }}
          SHARD_NUMBER: ${{ matrix.shard }}
          NUM_TEST_SHARDS: ${{ matrix.num_shards }}
          REENABLED_ISSUES: ${{ steps.keep-going.outputs.reenabled-issues }}
          DOCKER_IMAGE: ${{ inputs.docker-image }}
          XLA_CLANG_CACHE_S3_BUCKET_NAME: ossci-compiler-clang-cache-circleci-xla
          PYTORCH_TEST_CUDA_MEM_LEAK_CHECK: ${{ matrix.mem_leak_check && '1' || '0' }}
          PYTORCH_TEST_RERUN_DISABLED_TESTS: ${{ matrix.rerun_disabled_tests && '1' || '0' }}
          TESTS_TO_INCLUDE: ${{ inputs.tests-to-include }}
          DASHBOARD_TAG: ${{ inputs.dashboard-tag }}
          HUGGING_FACE_HUB_TOKEN: ${{ secrets.HUGGING_FACE_HUB_TOKEN }}
        timeout-minutes: ${{ fromJson(steps.test-timeout.outputs.timeout) }}
        run: |
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 201-218 / 第 201-218 行

````yaml
          # Fetch aws credential from IMDs
          eval "$(python3 .github/scripts/get_aws_session_tokens.py)"
          set -x

          TEST_COMMAND=.ci/pytorch/test.sh

          # detached container should get cleaned up by teardown_ec2_linux
          # Used for GPU_FLAG since that doesn't play nice
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
````

- EN: This section describes repository automation behavior for `.github/workflows/_xpu-test.yml`.
- CN: 该部分描述 `.github/workflows/_xpu-test.yml` 的仓库自动化行为。

### Lines 219-236 / 第 219-236 行

````yaml
            -e GITHUB_RUN_NUMBER \
            -e GITHUB_RUN_ATTEMPT \
            -e JOB_ID \
            -e BRANCH \
            -e SHA1 \
            -e AWS_DEFAULT_REGION \
            -e AWS_ACCESS_KEY_ID \
            -e AWS_SECRET_ACCESS_KEY \
            -e AWS_SESSION_TOKEN \
            -e IN_WHEEL_TEST \
            -e SHARD_NUMBER \
            -e TEST_CONFIG \
            -e NUM_TEST_SHARDS \
            -e REENABLED_ISSUES \
            -e PYTORCH_RETRY_TEST_CASES \
            -e PYTORCH_OVERRIDE_FLAKY_SIGNAL \
            -e CONTINUE_THROUGH_ERROR \
            -e VERBOSE_TEST_LOGS \
````

- EN: This section describes repository automation behavior for `.github/workflows/_xpu-test.yml`.
- CN: 该部分描述 `.github/workflows/_xpu-test.yml` 的仓库自动化行为。

### Lines 237-254 / 第 237-254 行

````yaml
            -e TEST_SHOWLOCALS \
            -e NO_TEST_TIMEOUT \
            -e NO_TD \
            -e MAX_JOBS="$(nproc --ignore=2)" \
            -e SCCACHE_BUCKET \
            -e SCCACHE_REGION \
            -e SCCACHE_S3_KEY_PREFIX \
            -e XLA_CLANG_CACHE_S3_BUCKET_NAME \
            -e PYTORCH_TEST_CUDA_MEM_LEAK_CHECK \
            -e PYTORCH_TEST_RERUN_DISABLED_TESTS \
            -e TESTS_TO_INCLUDE \
            -e ZE_AFFINITY_MASK \
            -e HUGGING_FACE_HUB_TOKEN \
            -e DASHBOARD_TAG \
            --env-file="${RUNNER_TEMP}/github_env_${GITHUB_RUN_ID}" \
            --ulimit stack=10485760:83886080 \
            --ulimit core=0 \
            --security-opt seccomp=unconfined \
````

- EN: This section describes repository automation behavior for `.github/workflows/_xpu-test.yml`.
- CN: 该部分描述 `.github/workflows/_xpu-test.yml` 的仓库自动化行为。

### Lines 255-269 / 第 255-269 行

````yaml
            --cap-add=SYS_PTRACE \
            --shm-size="8g" \
            --tty \
            --detach \
            --name="${container_name}" \
            --user jenkins \
            --privileged \
            -v "${GITHUB_WORKSPACE}:/var/lib/jenkins/workspace" \
            -w /var/lib/jenkins/workspace \
            "${DOCKER_IMAGE}"
          )
          # save container name for later step
          echo "CONTAINER_NAME=${container_name}" >> "$GITHUB_ENV"
          # jenkins user does not have write permission to mounted workspace; work-around by copying within container to jenkins home
          docker exec -t "${container_name}" sh -c "cd .. && cp -R workspace pytorch && cd pytorch && pip install dist/*.whl && ${TEST_COMMAND}"
````

- EN: This section describes repository automation behavior for `.github/workflows/_xpu-test.yml`.
- CN: 该部分描述 `.github/workflows/_xpu-test.yml` 的仓库自动化行为。

### Lines 271-286 / 第 271-286 行

````yaml
      - name: Save test results
        if: always()
        run: |
          # copy test results back to the mounted workspace, needed sudo, resulting permissions were correct
          docker exec -t "${{ env.CONTAINER_NAME }}" sh -c "cd ../pytorch && sudo cp -R test/test-reports ../workspace/test"

      - name: Change permissions
        if: ${{ always() && steps.test.conclusion }}
        run: |
          docker exec -t "${{ env.CONTAINER_NAME }}" sh -c "sudo chown -R 1000:1000 test"

      - name: Print remaining test logs
        shell: bash
        if: always() && steps.test.conclusion
        run: |
          cat test/**/*_toprint.log || true
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 288-302 / 第 288-302 行

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

### Lines 304-319 / 第 304-319 行

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

      - name: Collect backtraces from coredumps (if any)
        if: always()
        run: |
          # shellcheck disable=SC2156
          find . -iname "core.[1-9]*" -exec docker exec "${CONTAINER_NAME}" sh -c "gdb python {} -ex 'bt' -ex 'q'" \;
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/upload-utilization-stats`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 321-337 / 第 321-337 行

````yaml
      - name: Stop container before exit
        if: always()
        run: |
          # Workaround for multiple runners on same IDC node
          docker stop "${{ env.CONTAINER_NAME }}"

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

### Lines 338-350 / 第 338-350 行

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

### Lines 352-353 / 第 352-353 行

````yaml
      - name: Teardown XPU
        uses: ./.github/actions/teardown-xpu
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/teardown-xpu`.
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
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `./.github/actions/setup-xpu`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`, `./.github/actions/get-workflow-job-id`, `./.github/actions/download-build-artifacts`, `./.github/actions/filter-test-configs`, `./.github/actions/upload-test-artifacts`, ...
- Inline commands / 内联命令: `If`, `job`, `Set`, `List`, `[Experimental]`, `Currently,`, `since`, `HF`, ...
- Environment variables / 环境变量: `TODO`, `JSON`, `HUGGING_FACE_HUB_TOKEN`, `GIT_DEFAULT_BRANCH`, `XPU`, `ECR_DOCKER_IMAGE`, `GITHUB_TOKEN`, `JOB_ID`, `JOB_NAME`, `WORKFLOW_NAME`, ...
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `inputs`, `secrets`, `permissions`, `env`, `jobs`, `test`, `strategy`, `steps`
