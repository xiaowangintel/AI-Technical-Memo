# stale.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/stale.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-13 / 第 1-13 行

````yaml
# A workflow that implements similar logic to actions/stale.
#
# Compared to actions/stale, it is implemented to make API requests proportional
# to the number of stale PRs, not the total number of issues in the repo. This
# is because PyTorch has a lot of issues/PRs, so the actions/stale runs into
# rate limits way too quickly.
#
# The behavior is:
# - If a PR is not labeled stale, after 60 days inactivity label the PR as stale and comment about it.
# - If a PR is labeled stale, after 30 days inactivity close the PR.
# - `high priority` and `no-stale` PRs are exempt.

name: Close stale pull requests
````

- EN: This section describes repository automation behavior for `.github/workflows/stale.yml`.
- CN: 该部分描述 `.github/workflows/stale.yml` 的仓库自动化行为。

### Lines 15-27 / 第 15-27 行

````yaml
on:
  schedule:
    # Run hourly.
    - cron: 30 * * * *
  workflow_dispatch:

jobs:
  stale:
    if: ${{ github.repository == 'pytorch/pytorch' }}
    runs-on: linux.large
    permissions:
      contents: read
      pull-requests: write
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 29-42 / 第 29-42 行

````yaml
    steps:
      - uses: actions/github-script@60a0d83039c74a4aee543508d2ffcb1c3799cdea # v7.0.1
        with:
          script: |
            // Do some dumb retries on requests.
            const retries = 7;
            const baseBackoff = 100;
            const sleep = timeout => new Promise(resolve => setTimeout(resolve, timeout));
            github.hook.wrap('request', async (request, options) => {
              for (let attempt = 1; attempt <= retries; attempt++) {
                try {
                  return await request(options);
                } catch (err) {
                  if (attempt < retries) {
````

- EN: This section reuses actions and step building blocks such as named actions; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 43-52 / 第 43-52 行

````yaml
                    core.warning(`Request getting retried. Attempt: ${attempt}`);
                    await sleep(baseBackoff * Math.pow(2, attempt));
                    continue;
                  }
                  throw err;
                }
              }
            });

            const MAX_API_REQUESTS = 100;
````

- EN: This section describes repository automation behavior for `.github/workflows/stale.yml`.
- CN: 该部分描述 `.github/workflows/stale.yml` 的仓库自动化行为。

### Lines 54-64 / 第 54-64 行

````yaml
            // If a PRs not labeled stale, label them stale after no update for 60 days.
            const STALE_LABEL_THRESHOLD_MS = 1000 * 60 * 60 * 24 * 60;
            // For PRs already labeled stale, close after not update for 30 days.
            const STALE_CLOSE_THRESHOLD_MS = 1000 * 60 * 60 * 24 * 30;

            const STALE_MESSAGE =
              "Looks like this PR hasn't been updated in a while so we're going to go ahead and mark this as `Stale`. <br>" +
              "Feel free to remove the `Stale` label if you feel this was a mistake. <br>" +
              "If you are unable to remove the `Stale` label please contact a maintainer in order to do so. <br>" +
              "If you want the bot to never mark this PR stale again, add the `no-stale` label.<br>" +
              "`Stale` pull requests will automatically be closed after 30 days of inactivity.<br>";
````

- EN: This section describes repository automation behavior for `.github/workflows/stale.yml`.
- CN: 该部分描述 `.github/workflows/stale.yml` 的仓库自动化行为。

### Lines 66-78 / 第 66-78 行

````yaml
            let numAPIRequests = 0;
            let numProcessed = 0;

            async function processPull(pull) {
              core.info(`[${pull.number}] URL: ${pull.html_url}`);
              numProcessed += 1;
              const labels = pull.labels.map((label) => label.name);

              // Skip if certain labels are present.
              if (labels.includes("no-stale") || labels.includes("high priority")) {
                core.info(`[${pull.number}] Skipping because PR has an exempting label.`);
                return false;
              }
````

- EN: This section describes repository automation behavior for `.github/workflows/stale.yml`.
- CN: 该部分描述 `.github/workflows/stale.yml` 的仓库自动化行为。

### Lines 80-91 / 第 80-91 行

````yaml
              // Check if the PR is stale, according to our configured thresholds.
              let staleThresholdMillis;
              if (labels.includes("Stale")) {
                core.info(`[${pull.number}] PR is labeled stale, checking whether we should close it.`);
                staleThresholdMillis = STALE_CLOSE_THRESHOLD_MS;
              } else {
                core.info(`[${pull.number}] Checking whether to label PR as stale.`);
                staleThresholdMillis = STALE_LABEL_THRESHOLD_MS;
              }

              const millisSinceLastUpdated =
                new Date().getTime() - new Date(pull.updated_at).getTime();
````

- EN: This section describes repository automation behavior for `.github/workflows/stale.yml`.
- CN: 该部分描述 `.github/workflows/stale.yml` 的仓库自动化行为。

### Lines 93-106 / 第 93-106 行

````yaml
              if (millisSinceLastUpdated < staleThresholdMillis) {
                core.info(`[${pull.number}] Skipping because PR was updated recently`);
                return false;
              }

              // At this point, we know we should do something.
              // For PRs already labeled stale, close them.
              if (labels.includes("Stale")) {
                core.info(`[${pull.number}] Closing PR.`);
                numAPIRequests += 1;
                await github.rest.issues.update({
                  owner: "pytorch",
                  repo: "pytorch",
                  issue_number: pull.number,
````

- EN: This section describes repository automation behavior for `.github/workflows/stale.yml`.
- CN: 该部分描述 `.github/workflows/stale.yml` 的仓库自动化行为。

### Lines 107-119 / 第 107-119 行

````yaml
                  state: "closed",
                });
              } else {
                // For PRs not labeled stale, label them stale.
                core.info(`[${pull.number}] Labeling PR as stale.`);

                numAPIRequests += 1;
                await github.rest.issues.createComment({
                  owner: "pytorch",
                  repo: "pytorch",
                  issue_number: pull.number,
                  body: STALE_MESSAGE,
                });
````

- EN: This section describes repository automation behavior for `.github/workflows/stale.yml`.
- CN: 该部分描述 `.github/workflows/stale.yml` 的仓库自动化行为。

### Lines 121-129 / 第 121-129 行

````yaml
                numAPIRequests += 1;
                await github.rest.issues.addLabels({
                  owner: "pytorch",
                  repo: "pytorch",
                  issue_number: pull.number,
                  labels: ["Stale"],
                });
              }
            }
````

- EN: This section describes repository automation behavior for `.github/workflows/stale.yml`.
- CN: 该部分描述 `.github/workflows/stale.yml` 的仓库自动化行为。

### Lines 131-144 / 第 131-144 行

````yaml
            for await (const response of github.paginate.iterator(
              github.rest.pulls.list,
              {
                owner: "pytorch",
                repo: "pytorch",
                state: "open",
                sort: "created",
                direction: "asc",
                per_page: 100,
              }
            )) {
              numAPIRequests += 1;
              const pulls = response.data;
              // Awaiting in a loop is intentional here. We want to serialize execution so
````

- EN: This section describes repository automation behavior for `.github/workflows/stale.yml`.
- CN: 该部分描述 `.github/workflows/stale.yml` 的仓库自动化行为。

### Lines 145-156 / 第 145-156 行

````yaml
              // that log groups are printed correctl
              for (const pull of pulls) {
                if (numAPIRequests > MAX_API_REQUESTS) {
                  core.warning("Max API requests exceeded, exiting.");
                  process.exit(0);
                }
                await core.group(`Processing PR #${pull.number}`, async () => {
                  await processPull(pull);
                });
              }
            }
            core.info(`Processed ${numProcessed} PRs total.`);
````

- EN: This section describes repository automation behavior for `.github/workflows/stale.yml`.
- CN: 该部分描述 `.github/workflows/stale.yml` 的仓库自动化行为。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Manual workflow triggering — maintainers can run this automation on demand from GitHub Actions.
  CN: 手动工作流触发——维护者可以在 GitHub Actions 中按需执行该自动化流程。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: none
- Inline commands / 内联命令: `//`, `const`, `github.hook.wrap`, `try`, `return`, `core.warning`, `await`, `continue`, ...
- Environment variables / 环境变量: `API`, `MAX_API_REQUESTS`, `STALE_LABEL_THRESHOLD_MS`, `STALE_CLOSE_THRESHOLD_MS`, `STALE_MESSAGE`, `URL`
- Named jobs or sections / 命名作业或章节: `on`, `schedule`, `workflow_dispatch`, `jobs`, `stale`, `permissions`, `steps`
