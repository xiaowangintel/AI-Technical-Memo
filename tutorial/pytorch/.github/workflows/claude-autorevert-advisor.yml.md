# claude-autorevert-advisor.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/claude-autorevert-advisor.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-17 / 第 1-17 行

````yaml
name: Claude Autorevert Advisor

on:
  workflow_dispatch:
    inputs:
      suspect_commit:
        description: "SHA of the commit autorevert suspects caused the failure"
        required: true
        type: string
      pr_number:
        description: "PR number of the suspect commit"
        required: true
        type: string
      signal_pattern:
        description: "JSON object with signal metadata, failed partition (with job links/log URLs), and successful partition (baseline)"
        required: true
        type: string
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 19-34 / 第 19-34 行

````yaml
jobs:
  analyze:
    if: github.repository == 'pytorch/pytorch'
    runs-on: ubuntu-latest
    timeout-minutes: 15
    environment: bedrock
    permissions:
      contents: read
      id-token: write

    steps:
      - name: Checkout pytorch/pytorch trunk
        uses: actions/checkout@v4
        with:
          ref: main
          fetch-depth: 128
````

- EN: This section lays out job topology or execution stages; reuses actions and step building blocks such as `actions/checkout@v4`; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 36-48 / 第 36-48 行

````yaml
      - name: Fetch suspect commit
        run: |
          git fetch origin ${{ inputs.suspect_commit }} --depth=32 || true

      - name: Configure AWS credentials via OIDC
        uses: aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a # v4
        with:
          role-to-assume: arn:aws:iam::308535385114:role/gha_workflow_claude_code
          aws-region: us-east-1

      - name: Run AI Advisor
        id: claude
        uses: anthropics/claude-code-action@6e2bd52842c65e914eba5c8badd17560bd26b5de # v1.0.89
````

- EN: This section reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`, `anthropics/claude-code-action@6e2bd52842c65e914eba5c8badd17560bd26b5de`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 49-63 / 第 49-63 行

````yaml
        with:
          allowed_bots: "pytorch-auto-revert[bot],pytorch-bot[bot]"
          use_bedrock: "true"
          claude_args: |
            --model global.anthropic.claude-opus-4-6-v1
            --allowedTools "Bash,Read,Glob,Grep,WebFetch"
            --json-schema '{"type":"object","required":["verdict","confidence","summary","causal_reasoning"],"properties":{"verdict":{"type":"string","enum":["revert","unsure","not_related","garbage"]},"confidence":{"type":"number","minimum":0,"maximum":1},"summary":{"type":"string","description":"1-3 sentence explanation of the verdict"},"causal_reasoning":{"type":"string","description":"Detailed explanation of the causal chain (or lack thereof) between the code change and the observed failures"}}}'
          settings: '{"alwaysThinkingEnabled": true}'
          prompt: |
            You are an AI advisor for PyTorch's autorevert system. Your job is to analyze a suspect commit and CI failures to determine whether the commit caused the failures.

            ## Context

            **Suspect commit:** ${{ inputs.suspect_commit }}
            **PR:** https://github.com/pytorch/pytorch/pull/${{ inputs.pr_number }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 65-81 / 第 65-81 行

````yaml
            **Signal pattern (partition data from autorevert):**
            ```json
            ${{ inputs.signal_pattern }}
            ```

            The signal pattern contains:
            - `signal_key`: the test or job name that defines this signal
            - `signal_source`: "test" (individual test) or "job" (whole job)
            - `workflow_name`: the CI workflow where this signal lives
            - `failed_partition`: commits where this signal FAILS (newest first, suspect commit marked with `is_suspect: true`). Each has job links and log URLs.
            - `successful_partition`: baseline commits where this signal was GREEN (most recent first). These prove the signal was passing before the suspect commit.

            The partition is ground truth from CI data. The signal transitioned from SUCCESS to FAILURE at the suspect commit. Your job is to determine whether the CODE CHANGE caused the transition, or whether it's coincidence (upstream dependency change, flaky environment, infra issue).

            ## Investigation Steps

            1. **Read the suspect commit diff.** Run `git show ${{ inputs.suspect_commit }}` to understand what code changed. Read the modified files to understand the broader context around the changes.
````

- EN: This section describes repository automation behavior for `.github/workflows/claude-autorevert-advisor.yml`.
- CN: 该部分描述 `.github/workflows/claude-autorevert-advisor.yml` 的仓库自动化行为。

### Lines 83-99 / 第 83-99 行

````yaml
            2. **Analyze the PR.** Fetch the PR description and understand the intent of the change: `gh pr view ${{ inputs.pr_number }} --repo pytorch/pytorch --json title,body,files`

            3. **Investigate the failed jobs.** For each job in the failed_partition:
               - Fetch the job logs from the log_url to find error messages, stack traces, and assertion failures
               - Identify the specific test(s) or build step(s) that failed
               - Read the relevant test source code in this checkout to understand what the test exercises

            4. **Trace the causal chain.** This is the critical step. Do NOT take shortcuts:
               - Trace imports, function calls, class hierarchies, and runtime dispatch paths from the changed code to the failing tests
               - Consider indirect dependencies: a change to a utility function may break tests through several layers of abstraction
               - Consider runtime behavior: changes to tensor operations, kernel dispatch, autograd behavior, or compilation paths can have non-obvious downstream effects
               - Check if the error message or stack trace references any of the changed code, even indirectly

            5. **Check for alternative explanations:**
               - Is this a known flaky test? (Look for skip/xfail decorators, recent similar failures on other commits)
               - Is this an infrastructure issue? (OOM, timeout, network error, GPU driver issue)
               - Did a different concurrent commit plausibly cause this? (Check the commit timestamp and other recent merges)
````

- EN: This section describes repository automation behavior for `.github/workflows/claude-autorevert-advisor.yml`.
- CN: 该部分描述 `.github/workflows/claude-autorevert-advisor.yml` 的仓库自动化行为。

### Lines 100-115 / 第 100-115 行

````yaml
               - Is this signal fundamentally unreliable? (Persistent timeouts, environment-dependent flakes, resource exhaustion not caused by any code change)

            6. **Render your verdict:**
               - `revert` — You found a plausible causal chain from the code change to the failure. Confidence should reflect how direct and clear the chain is.
               - `not_related` — After thorough investigation, there is no plausible causal chain. The failure has a clear alternative explanation.
               - `unsure` — The investigation is inconclusive. There may be a connection but you cannot confirm it with sufficient confidence.
               - `garbage` — The signal itself is unreliable and should be temporarily suppressed. Use this when the failure is clearly not caused by ANY code change: persistent infra flakes (runner timeouts, network errors, GPU driver crashes), environment issues (disk full, dependency mirror outage), or tests so flaky that the signal has no diagnostic value. This tells autorevert to ignore this signal for ~2 hours.

            When in doubt between `unsure` and `not_related`, prefer `unsure`. A false `not_related` verdict is more costly than an `unsure` (which just lets autorevert continue its normal restart-to-confirm flow).

            Use `garbage` only when the failure clearly has nothing to do with any code change — it's a statement about the signal quality, not about the suspect commit.

            IMPORTANT: Be thorough. Read actual code. Trace actual call paths. Do not guess based on file names alone.

      - name: Save verdict artifact
        if: always() && steps.claude.outputs.structured_output != ''
````

- EN: This section describes repository automation behavior for `.github/workflows/claude-autorevert-advisor.yml`.
- CN: 该部分描述 `.github/workflows/claude-autorevert-advisor.yml` 的仓库自动化行为。

### Lines 116-132 / 第 116-132 行

````yaml
        env:
          VERDICT_JSON: ${{ steps.claude.outputs.structured_output }}
        run: |
          mkdir -p /tmp/verdict
          printf '%s' "$VERDICT_JSON" > /tmp/verdict/verdict.json
          cat /tmp/verdict/verdict.json

      - name: Upload verdict artifact
        if: always() && steps.claude.outputs.structured_output != ''
        uses: actions/upload-artifact@v4
        with:
          name: autorevert-verdict
          path: /tmp/verdict/verdict.json
          retention-days: 30

      - name: Upload verdict to S3 for ClickHouse ingestion
        if: always() && steps.claude.outputs.structured_output != ''
````

- EN: This section reuses actions and step building blocks such as `actions/upload-artifact@v4`; embeds shell commands that perform the actual side effects; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 133-150 / 第 133-150 行

````yaml
        env:
          VERDICT_JSON: ${{ steps.claude.outputs.structured_output }}
          SIGNAL_PATTERN: ${{ inputs.signal_pattern }}
        run: |
          # Build enriched verdict JSON with signal metadata for CH ingestion
          jq -n \
            --arg repo "${{ github.repository }}" \
            --argjson run_id "${{ github.run_id }}" \
            --argjson run_attempt "${{ github.run_attempt }}" \
            --arg timestamp "$(date -u +%Y-%m-%dT%H:%M:%S.000)" \
            --arg suspect_commit "${{ inputs.suspect_commit }}" \
            --argjson pr_number "${{ inputs.pr_number }}" \
            --arg signal_key "$(echo "$SIGNAL_PATTERN" | jq -r '.signal_key // ""')" \
            --arg signal_source "$(echo "$SIGNAL_PATTERN" | jq -r '.signal_source // ""')" \
            --arg workflow_name "$(echo "$SIGNAL_PATTERN" | jq -r '.workflow_name // ""')" \
            --arg verdict "$(echo "$VERDICT_JSON" | jq -r '.verdict // ""')" \
            --argjson confidence "$(echo "$VERDICT_JSON" | jq -r '.confidence // 0')" \
            --arg summary "$(echo "$VERDICT_JSON" | jq -r '.summary // ""')" \
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 151-166 / 第 151-166 行

````yaml
            --arg causal_reasoning "$(echo "$VERDICT_JSON" | jq -r '.causal_reasoning // ""')" \
            '{
              repo: $repo,
              run_id: $run_id,
              run_attempt: $run_attempt,
              timestamp: $timestamp,
              suspect_commit: $suspect_commit,
              pr_number: $pr_number,
              signal_key: $signal_key,
              signal_source: $signal_source,
              workflow_name: $workflow_name,
              verdict: $verdict,
              confidence: $confidence,
              summary: $summary,
              causal_reasoning: $causal_reasoning
            }' > /tmp/advisor_verdict.json
````

- EN: This section describes repository automation behavior for `.github/workflows/claude-autorevert-advisor.yml`.
- CN: 该部分描述 `.github/workflows/claude-autorevert-advisor.yml` 的仓库自动化行为。

### Lines 168-169 / 第 168-169 行

````yaml
          aws s3 cp /tmp/advisor_verdict.json \
            "s3://ossci-raw-job-status/autorevert_advisor_verdicts/${{ github.repository }}/${{ github.run_id }}_${{ github.run_attempt }}.json"
````

- EN: This section describes repository automation behavior for `.github/workflows/claude-autorevert-advisor.yml`.
- CN: 该部分描述 `.github/workflows/claude-autorevert-advisor.yml` 的仓库自动化行为。

### Lines 171-173 / 第 171-173 行

````yaml
      - name: Upload usage metrics
        if: always()
        uses: pytorch/test-infra/.github/actions/upload-claude-usage@main
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/upload-claude-usage@main`.
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
- Reusable actions / 复用 Action: `actions/checkout@v4`, `aws-actions/configure-aws-credentials@7474bc4690e29a8392af63c5b98e7449536d5c3a`, `anthropics/claude-code-action@6e2bd52842c65e914eba5c8badd17560bd26b5de`, `actions/upload-artifact@v4`, `pytorch/test-infra/.github/actions/upload-claude-usage@main`
- Inline commands / 内联命令: `git`, `model`, `allowedTools`, `json-schema`, `You`, `**Suspect`, `**PR:**`, `**Signal`, ...
- Environment variables / 环境变量: `SHA`, `JSON`, `AWS`, `OIDC`, `FAILS`, `GREEN`, `SUCCESS`, `FAILURE`, `CODE`, `CHANGE`, ...
- Named jobs or sections / 命名作业或章节: `on`, `workflow_dispatch`, `inputs`, `jobs`, `analyze`, `permissions`, `steps`
