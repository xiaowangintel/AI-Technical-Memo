# upload-test-stats.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/upload-test-stats.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-13 / 第 1-13 行

````yaml
name: Upload test stats

on:
  workflow_run:
    workflows:
      - pull
      - trunk
      - trunk-rocm-sandbox
      - periodic
      - periodic-rocm-mi200
      - periodic-rocm-mi300
      - periodic-rocm-mi355
      - inductor
````

- EN: This section declares workflow triggers or event bindings; embeds shell commands that perform the actual side effects.
- CN: 该部分声明工作流触发条件或事件绑定；内嵌执行实际副作用的 shell 命令。

### Lines 14-26 / 第 14-26 行

````yaml
      - unstable
      - slow
      - slow-rocm-mi200
      - unstable-periodic
      - inductor-periodic
      - rocm-mi200
      - rocm-mi300
      - rocm-mi355
      - rocm-navi31
      - rocm-nightly
      - inductor-micro-benchmark
      - inductor-micro-benchmark-x86
      - inductor-cu124
````

- EN: This section describes repository automation behavior for `.github/workflows/upload-test-stats.yml`.
- CN: 该部分描述 `.github/workflows/upload-test-stats.yml` 的仓库自动化行为。

### Lines 27-38 / 第 27-38 行

````yaml
      - inductor-rocm-mi200
      - inductor-rocm-mi300
      - inductor-rocm-mi355
      - inductor-perf-nightly-rocm-mi300
      - inductor-perf-nightly-rocm-mi355
      - mac-mps
    types:
      - completed

jobs:
  # the conclusion field in the github context is sometimes null
  # solution adapted from https://github.com/community/community/discussions/21090#discussioncomment-3226271
````

- EN: This section lays out job topology or execution stages.
- CN: 该部分铺排作业拓扑或执行阶段。

### Lines 39-50 / 第 39-50 行

````yaml
  get_workflow_conclusion:
    if: github.repository_owner == 'pytorch'
    runs-on: ubuntu-latest
    outputs:
      conclusion: ${{ fromJson(steps.get_conclusion.outputs.data).conclusion }}
    steps:
      - name: Get workflow run conclusion
        # TODO (huydhn): Pin this once https://github.com/octokit/request-action/issues/315 is resolved
        uses: octokit/request-action@05a2312de9f8207044c4c9e41fe19703986acc13 # v2.x
        id: get_conclusion
        with:
          route: GET /repos/${{ github.repository }}/actions/runs/${{ github.event.workflow_run.id }}/attempts/${{ github.event.workflow_run.run_attempt }}
````

- EN: This section reuses actions and step building blocks such as `octokit/request-action@05a2312de9f8207044c4c9e41fe19703986acc13`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 51-63 / 第 51-63 行

````yaml
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  upload-test-stats:
    needs: get_workflow_conclusion
    if: github.repository_owner == 'pytorch'
    runs-on: ubuntu-22.04
    environment: upload-stats
    permissions:
      id-token: write
    name: Upload test stats for ${{ github.event.workflow_run.id }}, attempt ${{ github.event.workflow_run.run_attempt }}
    steps:
      - name: Print workflow information
````

- EN: This section reuses actions and step building blocks such as named actions; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 64-76 / 第 64-76 行

````yaml
        env:
          TRIGGERING_WORKFLOW: ${{ toJSON(github.event.workflow_run) }}
        run: echo "${TRIGGERING_WORKFLOW}"

      - name: Checkout PyTorch
        uses: pytorch/pytorch/.github/actions/checkout-pytorch@main

      - name: Configure aws credentials
        uses: aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722 # v4.1.0
        continue-on-error: true
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_upload-torch-test-stats
          aws-region: us-east-1
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 78-87 / 第 78-87 行

````yaml
      - uses: actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5.6.0
        with:
          python-version: '3.11'
          cache: pip

      - run: |
          pip3 install requests==2.32.2 boto3==1.35.42

      - name: Upload test artifacts
        id: upload-s3
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 88-99 / 第 88-99 行

````yaml
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          WORKFLOW_ARTIFACTS_URL: ${{ github.event.workflow_run.artifacts_url }}
          WORKFLOW_RUN_ID: ${{ github.event.workflow_run.id }}
          WORKFLOW_RUN_ATTEMPT: ${{ github.event.workflow_run.run_attempt }}
          REPO_FULLNAME: ${{ github.event.workflow_run.repository.full_name }}
        run: |
          echo "${WORKFLOW_ARTIFACTS_URL}"

          # Note that in the case of Linux and Windows, their artifacts have already been uploaded to S3, so there simply won't be
          # anything on GitHub to upload. The command should return right away
          python3 -m tools.stats.upload_artifacts --workflow-run-id "${WORKFLOW_RUN_ID}" --workflow-run-attempt "${WORKFLOW_RUN_ATTEMPT}" --repo "${REPO_FULLNAME}"
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 101-112 / 第 101-112 行

````yaml
      - name: Upload test stats
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          WORKFLOW_RUN_ID: ${{ github.event.workflow_run.id }}
          WORKFLOW_RUN_ATTEMPT: ${{ github.event.workflow_run.run_attempt }}
          WORKFLOW_URL: ${{ github.event.workflow_run.html_url }}
          HEAD_REPOSITORY: ${{ github.event.workflow_run.head_repository.full_name }}
          HEAD_BRANCH: ${{ github.event.workflow_run.head_branch }}
        run: |
          echo "${WORKFLOW_URL}"
          python3 -m tools.stats.upload_test_stats --workflow-run-id "${WORKFLOW_RUN_ID}" --workflow-run-attempt "${WORKFLOW_RUN_ATTEMPT}" --head-branch "${HEAD_BRANCH}" --head-repository "${HEAD_REPOSITORY}"
          python3 -m tools.stats.upload_sccache_stats --workflow-run-id "${WORKFLOW_RUN_ID}" --workflow-run-attempt "${WORKFLOW_RUN_ATTEMPT}"
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 114-126 / 第 114-126 行

````yaml
      - name: Analyze disabled tests rerun
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          WORKFLOW_ARTIFACTS_URL: ${{ github.event.workflow_run.artifacts_url }}
          WORKFLOW_RUN_ID: ${{ github.event.workflow_run.id }}
          WORKFLOW_RUN_ATTEMPT: ${{ github.event.workflow_run.run_attempt }}
          REPO_FULLNAME: ${{ github.event.workflow_run.repository.full_name }}
        run: |
          # Analyze the results from disable tests rerun and upload them to S3
          python3 -m tools.stats.check_disabled_tests --workflow-run-id "${WORKFLOW_RUN_ID}" --workflow-run-attempt "${WORKFLOW_RUN_ATTEMPT}" --repo "${REPO_FULLNAME}"

      - name: Upload gpt-fast benchmark results to s3
        if: steps.upload-s3.outcome && steps.upload-s3.outcome == 'success' && contains(github.event.workflow_run.name, 'inductor-micro-benchmark')
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 127-139 / 第 127-139 行

````yaml
        env:
          WORKFLOW_RUN_ID: ${{ github.event.workflow_run.id }}
          WORKFLOW_RUN_ATTEMPT: ${{ github.event.workflow_run.run_attempt }}
          REPO_FULLNAME: ${{ github.event.workflow_run.repository.full_name }}
          HEAD_BRANCH: ${{ github.event.workflow_run.head_branch }}
        run: |
          python3 -m tools.stats.upload_dynamo_perf_stats --workflow-run-id "${WORKFLOW_RUN_ID}" --workflow-run-attempt "${WORKFLOW_RUN_ATTEMPT}" --repo "${REPO_FULLNAME}" --head-branch "${HEAD_BRANCH}" --dynamodb-table torchci-oss-ci-benchmark --match-filename "^gpt_fast_benchmark"

  check-api-rate:
    if: ${{ always() && github.repository_owner == 'pytorch' }}
    runs-on: ubuntu-latest
    continue-on-error: true
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 140-140 / 第 140-140 行

````yaml
      - name: Get our GITHUB_TOKEN API limit usage
````

- EN: This section describes repository automation behavior for `.github/workflows/upload-test-stats.yml`.
- CN: 该部分描述 `.github/workflows/upload-test-stats.yml` 的仓库自动化行为。

### Lines 141-144 / 第 141-144 行

````yaml
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          curl -H "Accept: application/vnd.github.v3+json" -H "Authorization: token $GITHUB_TOKEN" https://api.github.com/rate_limit
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。


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
- Reusable actions / 复用 Action: `octokit/request-action@05a2312de9f8207044c4c9e41fe19703986acc13`, `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`
- Inline commands / 内联命令: `pip3`, `echo`, `python3`, `curl`
- Environment variables / 环境变量: `TODO`, `GET`, `GITHUB_TOKEN`, `TRIGGERING_WORKFLOW`, `WORKFLOW_ARTIFACTS_URL`, `WORKFLOW_RUN_ID`, `WORKFLOW_RUN_ATTEMPT`, `REPO_FULLNAME`, `WORKFLOW_URL`, `HEAD_REPOSITORY`, ...
- Named jobs or sections / 命名作业或章节: `on`, `workflow_run`, `workflows`, `types`, `jobs`, `get_workflow_conclusion`, `outputs`, `steps`, `upload-test-stats`, `permissions`, ...
