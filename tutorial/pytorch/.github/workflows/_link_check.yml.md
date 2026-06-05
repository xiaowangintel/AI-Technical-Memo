# _link_check.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/_link_check.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-9 / 第 1-9 行

````yaml
on:
  workflow_call:
    inputs:
      runner:
        type: string
        required: true
      ref:
        type: string
        required: true
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 11-20 / 第 11-20 行

````yaml
jobs:
  lint-urls:
    if: ${{ github.event_name != 'pull_request' || !contains(github.event.pull_request.labels.*.name, 'skip-url-lint') }}
    uses: ./.github/workflows/_lint.yml
    with:
      runner: mt-l-x86iamx-8-16
      docker-image: ghcr.io/pytorch/test-infra:cpu-x86_64-810d48d
      script: |
        ./scripts/lint_urls.sh $(
          if [ "${{ github.event_name }}" = "pull_request" ]; then
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; reuses actions and step building blocks such as `./.github/workflows/_lint.yml`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 21-30 / 第 21-30 行

````yaml
            echo "${{ github.event.pull_request.base.sha }}" "${{ github.event.pull_request.head.sha }}"
          else
            echo "${{ github.event.before }}" "${{ github.sha }}"
          fi
        ) || {
          echo
          echo "URL lint failed."
          echo "If this is a transient outage, you can bypass it by adding the \`skip-url-lint\` label to your PR."
          echo "Or add \`@lint-ignore\` somewhere on the same line as the URL you want to skip checking."
          exit 1
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 31-40 / 第 31-40 行

````yaml
        }

  lint-xrefs:
    if: ${{ github.event_name != 'pull_request' || !contains(github.event.pull_request.labels.*.name, 'skip-xref-lint') }}
    uses: ./.github/workflows/_lint.yml
    with:
      runner: mt-l-x86iamx-8-16
      docker-image: ghcr.io/pytorch/test-infra:cpu-x86_64-810d48d
      script: |
        ./scripts/lint_xrefs.sh $(
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `./.github/workflows/_lint.yml`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 41-50 / 第 41-50 行

````yaml
          if [ "${{ github.event_name }}" = "pull_request" ]; then
            echo "${{ github.event.pull_request.base.sha }}" "${{ github.event.pull_request.head.sha }}"
          else
            echo "${{ github.event.before }}" "${{ github.sha }}"
          fi
        ) || {
          echo
          echo "Xref lint failed."
          echo "If this is a transient outage, you can bypass it by adding the \`skip-xref-lint\` label to your PR."
          echo "Or add \`@lint-ignore\` somewhere on the same line as the reference you want to skip checking."
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 51-52 / 第 51-52 行

````yaml
          exit 1
        }
````

- EN: This section describes repository automation behavior for `.github/workflows/_link_check.yml`.
- CN: 该部分描述 `.github/workflows/_link_check.yml` 的仓库自动化行为。


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

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: `./.github/workflows/_lint.yml`
- Inline commands / 内联命令: `./scripts/lint_urls.sh`, `echo`, `exit`, `./scripts/lint_xrefs.sh`
- Environment variables / 环境变量: `URL`
- Named jobs or sections / 命名作业或章节: `on`, `workflow_call`, `inputs`, `jobs`, `lint-urls`, `with`, `lint-xrefs`
