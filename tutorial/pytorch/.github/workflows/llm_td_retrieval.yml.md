# llm_td_retrieval.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/llm_td_retrieval.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-12 / 第 1-12 行

````yaml
name: Retrieval PyTorch Tests for Target Determination

on:
  workflow_call:

permissions:
  id-token: write
  contents: read

jobs:
  get-label-type:
    name: get-label-type
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 13-22 / 第 13-22 行

````yaml
    # Don't run on forked repos
    if: github.repository_owner == 'pytorch'
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}

  llm-retrieval:
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 23-33 / 第 23-33 行

````yaml
    # Don't run on forked repos
    if: github.repository_owner == 'pytorch'
    runs-on: "${{ needs.get-label-type.outputs.label-type }}linux.4xlarge"
    continue-on-error: true
    needs: get-label-type
    steps:
      - name: Setup Linux
        uses: pytorch/pytorch/.github/actions/setup-linux@main

      - name: Clone CodeLlama
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/setup-linux@main`, `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 34-44 / 第 34-44 行

````yaml
        with:
          repository: osalpekar/codellama
          ref: main
          path: codellama

      - name: Clone Target Determination Code
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
        with:
          repository: osalpekar/llm-target-determinator
          ref: v0.0.2
          path: llm-target-determinator
````

- EN: This section reuses actions and step building blocks such as `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 46-59 / 第 46-59 行

````yaml
      - name: Install requirements
        shell: bash
        run: |
          set -euxo pipefail
          python3 -m pip install -r llm-target-determinator/requirements.txt
          cd "${GITHUB_WORKSPACE}/codellama"
          python3 -m pip install -e .

      - name: Fetch CodeLlama Checkpoint
        shell: bash
        run: |
          set -euxo pipefail
          cd "${GITHUB_WORKSPACE}/codellama"
          mkdir "CodeLlama-7b-Python"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 60-73 / 第 60-73 行

````yaml
          aws s3 cp "s3://target-determinator-assets/CodeLlama-7b-Python" "CodeLlama-7b-Python" --recursive --no-progress

      - name: Fetch indexes
        uses: nick-fields/retry@7152eba30c6575329ac0576536151aca5a72780e # v3.0.0
        with:
          max_attempts: 3
          retry_wait_seconds: 10
          timeout_minutes: 5
          shell: bash
          command: |
            set -euxo pipefail
            python3 -m pip install awscli==1.29.40
            cd "${GITHUB_WORKSPACE}"/llm-target-determinator/assets
            aws s3 cp "s3://target-determinator-assets/indexes/latest" . --recursive
````

- EN: This section reuses actions and step building blocks such as `nick-fields/retry@7152eba30c6575329ac0576536151aca5a72780e`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 75-88 / 第 75-88 行

````yaml
            unzip -o indexer-files\*.zip
            rm indexer-files*.zip

      - name: Run Retriever
        id: run_retriever
        continue-on-error: true  # ghstack not currently supported due to problems getting git diff
        shell: bash
        run: |
          set -euxo pipefail
          cd "${GITHUB_WORKSPACE}"/llm-target-determinator
          export PATH="$HOME/.local/bin:$PATH"
          torchrun \
            --standalone \
            --nnodes=1 \
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 89-98 / 第 89-98 行

````yaml
            --nproc-per-node=1 \
            retriever.py \
            --experiment-name indexer-files \
            --pr-parse-format GITDIFF
          cd assets
          zip -r mappings.zip mappings

      - name: Upload results to s3
        uses: seemethere/upload-artifact-s3@baba72d0712b404f646cebe0730933554ebce96a # v5.1.0
        if: ${{ steps.run_retriever.outcome == 'success' }}
````

- EN: This section reuses actions and step building blocks such as `seemethere/upload-artifact-s3@baba72d0712b404f646cebe0730933554ebce96a`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 99-109 / 第 99-109 行

````yaml
        with:
          name: llm_results
          retention-days: 14
          if-no-files-found: warn
          path: llm-target-determinator/assets/mappings.zip
        env:
          AWS_ACCESS_KEY_ID: ""
          AWS_SECRET_ACCESS_KEY: ""
          AWS_SESSION_TOKEN: ""
          AWS_DEFAULT_REGION: ""
          AWS_REGION: ""
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 111-113 / 第 111-113 行

````yaml
      - name: Teardown Linux
        uses: pytorch/test-infra/.github/actions/teardown-linux@main
        if: always()
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/teardown-linux@main`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `pytorch/pytorch/.github/actions/setup-linux@main`, `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`, `nick-fields/retry@7152eba30c6575329ac0576536151aca5a72780e`, `seemethere/upload-artifact-s3@baba72d0712b404f646cebe0730933554ebce96a`, `pytorch/test-infra/.github/actions/teardown-linux@main`
- Inline commands / 内联命令: `set`, `python3`, `cd`, `mkdir`, `aws`, `unzip`, `rm`, `torchrun`, ...
- Environment variables / 环境变量: `GITHUB_WORKSPACE`, `PATH`, `HOME`, `GITDIFF`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`, `AWS_DEFAULT_REGION`, `AWS_REGION`
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `permissions`, `jobs`, `get-label-type`, `with`, `llm-retrieval`, `steps`
