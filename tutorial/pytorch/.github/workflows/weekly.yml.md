# weekly.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/weekly.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-13 / 第 1-13 行

````yaml
name: weekly

on:
  schedule:
    # Mondays at 7:37am UTC = 12:27am PST
    # Choose a random time near midnight PST because it may be delayed if there are high loads
    # See https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule
    - cron: 37 7 * * 1
  workflow_dispatch:

permissions: read-all

jobs:
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 14-25 / 第 14-25 行

````yaml
  update-commit-hash:
    if: github.repository_owner == 'pytorch'
    runs-on: ubuntu-latest
    environment: update-commit-hash
    steps:
      - name: Checkout repo
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
        with:
          fetch-depth: 0
      - name: update-xla-commit-hash
        continue-on-error: true
        uses: pytorch/test-infra/.github/actions/update-commit-hash@main
````

- EN: This section reuses actions and step building blocks such as `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`, `pytorch/test-infra/.github/actions/update-commit-hash@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 26-38 / 第 26-38 行

````yaml
        with:
          repo-name: xla
          branch: master
          pin-folder: .github/ci_commit_pins
          test-infra-ref: main
          updatebot-token: ${{ secrets.UPDATEBOT_TOKEN }}
          pytorchbot-token: ${{ secrets.GH_PYTORCHBOT_TOKEN }}

  update-slow-tests:
    if: github.repository_owner == 'pytorch'
    runs-on: ubuntu-latest
    environment: update-commit-hash
    steps:
````

- EN: This section reuses actions and step building blocks such as named actions; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 39-51 / 第 39-51 行

````yaml
      - name: Checkout repo
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
        with:
          fetch-depth: 0
          token: ${{ secrets.UPDATEBOT_TOKEN }}
      - name: Setup Python
        uses: actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v5.6.0
        with:
          python-version: '3.10'
      - name: Install requirements
        shell: bash
        run: |
          pip install requests==2.32.2 clickhouse-connect==0.8.14
````

- EN: This section reuses actions and step building blocks such as `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`, `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 52-63 / 第 52-63 行

````yaml
      - name: Update slow test file
        shell: bash
        env:
          PYTORCHBOT_TOKEN: ${{ secrets.GH_PYTORCHBOT_TOKEN }}
          UPDATEBOT_TOKEN: ${{ secrets.UPDATEBOT_TOKEN }}
          CLICKHOUSE_ENDPOINT: ${{ secrets.CLICKHOUSE_ENDPOINT }}
          CLICKHOUSE_USERNAME: ${{ secrets.CLICKHOUSE_READONLY_USERNAME }}
          CLICKHOUSE_PASSWORD: ${{ secrets.CLICKHOUSE_READONLY_PASSWORD }}
        run: |
          git config --global user.name "PyTorch UpdateBot"
          git config --global user.email "pytorchupdatebot@users.noreply.github.com"
          python tools/testing/update_slow_tests.py
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
- EN: Manual workflow triggering — maintainers can run this automation on demand from GitHub Actions.
  CN: 手动工作流触发——维护者可以在 GitHub Actions 中按需执行该自动化流程。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683`, `pytorch/test-infra/.github/actions/update-commit-hash@main`, `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065`
- Inline commands / 内联命令: `pip`, `git`, `python`
- Environment variables / 环境变量: `UTC`, `PST`, `UPDATEBOT_TOKEN`, `GH_PYTORCHBOT_TOKEN`, `PYTORCHBOT_TOKEN`, `CLICKHOUSE_ENDPOINT`, `CLICKHOUSE_USERNAME`, `CLICKHOUSE_READONLY_USERNAME`, `CLICKHOUSE_PASSWORD`, `CLICKHOUSE_READONLY_PASSWORD`
- Named jobs or sections / 命名作业或章节: `on`, `schedule`, `workflow_dispatch`, `jobs`, `update-commit-hash`, `steps`, `update-slow-tests`
