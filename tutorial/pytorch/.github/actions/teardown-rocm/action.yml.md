# action.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/actions/teardown-rocm/action.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a reusable composite GitHub Action, including declared inputs, step composition, and shell execution.
- **用途 (CN)**: 定义可复用的复合型 GitHub Action，包括输入声明、步骤组合以及 shell 执行逻辑。

## Content Analysis / 内容分析
### Lines 1-7 / 第 1-7 行

````yaml
name: Teardown ROCm host

description: Tear down ROCm host for CI

runs:
  using: composite
  steps:
````

- EN: This section reuses actions and step building blocks such as named actions.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 8-16 / 第 8-16 行

````yaml
    - name: Teardown ROCm
      if: always()
      shell: bash
      run: |
        # ignore expansion of "docker ps -q" since it could be empty
        # shellcheck disable=SC2046
        docker stop $(docker ps -q) || true
        # Prune all stopped containers.
        docker container prune -f
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 17-19 / 第 17-19 行

````yaml
    - name: Runner diskspace health check
      uses: ./.github/actions/diskspace-cleanup
      if: always()
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/diskspace-cleanup`.
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

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: `./.github/actions/diskspace-cleanup`
- Inline commands / 内联命令: `docker`
- Environment variables / 环境变量: `SC2046`
- Named jobs or sections / 命名作业或章节: `runs`, `steps`
