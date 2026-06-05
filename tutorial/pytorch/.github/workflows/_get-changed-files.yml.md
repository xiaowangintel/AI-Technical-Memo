# _get-changed-files.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/_get-changed-files.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-11 / 第 1-11 行

````yaml
name: Get Changed Files

on:
  workflow_call:
    inputs:
      all_files:
        description: "Whether to return all files instead of just changed files"
        required: false
        type: boolean
        default: false
    outputs:
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 12-24 / 第 12-24 行

````yaml
      changed-files:
        description: "List of changed files (space-separated) or '*' if not in a PR"
        value: ${{ jobs.get-changed-files.outputs.changed-files }}

jobs:
  get-changed-files:
    runs-on: ubuntu-latest
    outputs:
      changed-files: ${{ steps.get-files.outputs.changed-files }}

    steps:
      - name: Get changed files
        id: get-files
````

- EN: This section lays out job topology or execution stages; reuses actions and step building blocks such as named actions.
- CN: 该部分铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程。

### Lines 25-34 / 第 25-34 行

````yaml
        env:
          GH_TOKEN: ${{ github.token }}
        run: |
          set -eo pipefail
          # Check if we're in a pull request context
          if [ "${{ github.event_name }}" = "pull_request" ] || [ "${{ github.event_name }}" = "pull_request_target" ]; then
            echo "Running in PR context"

            # Get the PR number from the github context
            PR_NUMBER="${{ github.event.number }}"
````

- EN: This section declares workflow triggers or event bindings; embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 36-49 / 第 36-49 行

````yaml
            # Check if all_files is requested
            if [ "${{ inputs.all_files }}" = "true" ]; then
              echo "all_files input is true, returning all files"
              echo "changed-files=*" >> "$GITHUB_OUTPUT"
            else
              # Use gh CLI to get changed files in the PR with explicit repo.
              # pipefail ensures that if gh fails (e.g. 403 rate limit), the
              # error propagates through the pipe instead of being silently
              # captured as the file list and injected into downstream lint jobs.
              if ! CHANGED_FILES=$(gh api repos/${{ github.repository }}/pulls/$PR_NUMBER/files --paginate --jq '.[] | select(.status != "removed") | .filename' | tr '\n' ' ' | sed 's/ $//'); then
                echo "Failed to get changed files from GitHub API, falling back to all files"
                echo "changed-files=*" >> "$GITHUB_OUTPUT"
                exit 0
              fi
````

- EN: This section describes repository automation behavior for `.github/workflows/_get-changed-files.yml`.
- CN: 该部分描述 `.github/workflows/_get-changed-files.yml` 的仓库自动化行为。

### Lines 51-63 / 第 51-63 行

````yaml
              # See https://github.com/pytorch/pytorch/pull/134215#issuecomment-2332128790
              PYI_FILES_TO_ADD=""
              for file in ${CHANGED_FILES}; do
                if [[ "${file}" == *".pyi.in" ]]; then
                  PYI_FILES_TO_ADD="${PYI_FILES_TO_ADD} ${file//.in/}"
                fi
              done
              CHANGED_FILES="${CHANGED_FILES}${PYI_FILES_TO_ADD}"

              if [ -z "$CHANGED_FILES" ]; then
                echo "No changed files found, setting to '*'"
                CHANGED_FILES="*"
              fi
````

- EN: This section describes repository automation behavior for `.github/workflows/_get-changed-files.yml`.
- CN: 该部分描述 `.github/workflows/_get-changed-files.yml` 的仓库自动化行为。

### Lines 65-67 / 第 65-67 行

````yaml
              echo "Changed files: $CHANGED_FILES"
              echo "changed-files=$CHANGED_FILES" >> "$GITHUB_OUTPUT"
            fi
````

- EN: This section describes repository automation behavior for `.github/workflows/_get-changed-files.yml`.
- CN: 该部分描述 `.github/workflows/_get-changed-files.yml` 的仓库自动化行为。

### Lines 69-72 / 第 69-72 行

````yaml
          else
            echo "Not in PR context, setting changed files to '*'"
            echo "changed-files=*" >> "$GITHUB_OUTPUT"
          fi
````

- EN: This section describes repository automation behavior for `.github/workflows/_get-changed-files.yml`.
- CN: 该部分描述 `.github/workflows/_get-changed-files.yml` 的仓库自动化行为。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: none
- Inline commands / 内联命令: `set`, `echo`, `exit`
- Environment variables / 环境变量: `GH_TOKEN`, `PR_NUMBER`, `GITHUB_OUTPUT`, `CLI`, `CHANGED_FILES`, `API`, `PYI_FILES_TO_ADD`
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `inputs`, `outputs`, `jobs`, `get-changed-files`, `steps`
