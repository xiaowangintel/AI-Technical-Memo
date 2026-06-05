# nightly.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/nightly.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-13 / 第 1-13 行

````yaml
name: nightly

on:
  schedule:
    - cron: 0 0 * * *
  push:
    tags:
      # NOTE: Doc build pipelines should only get triggered on:
      # Major or minor release candidates builds
      - v[0-9]+.[0-9]+.0+-rc[0-9]+
      # Final RC for major, minor and patch releases
      - v[0-9]+.[0-9]+.[0-9]+
      - ciflow/nightly/*
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 14-26 / 第 14-26 行

````yaml
  workflow_dispatch:


concurrency:
  group: ${{ github.workflow }}--${{ github.event.pull_request.number || github.sha }}-${{ github.event_name == 'workflow_dispatch' }}
  cancel-in-progress: true

permissions:
  id-token: write
  contents: read
  actions: read

jobs:
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 27-35 / 第 27-35 行

````yaml
  get-label-type:
    name: get-label-type
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
    if: ${{ (github.event_name != 'schedule' || github.repository == 'pytorch/pytorch') && github.repository_owner == 'pytorch' }}
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 37-49 / 第 37-49 行

````yaml
  link-check:
    name: Link checks
    needs: get-label-type
    uses: ./.github/workflows/_link_check.yml
    with:
      runner: ${{ needs.get-label-type.outputs.label-type }}
      ref:    ${{ github.sha }}
    secrets: inherit

  docs-build:
    name: docs build
    uses: ./.github/workflows/_linux-build.yml
    needs: get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_link_check.yml`, `./.github/workflows/_linux-build.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 50-61 / 第 50-61 行

````yaml
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-py3.10-gcc11
      docker-image-name: ci-image:pytorch-linux-jammy-py3.10-clang18
    secrets: inherit

  docs-push:
    name: docs push
    uses: ./.github/workflows/_docs.yml
    needs:
      - docs-build
      - get-label-type
````

- EN: This section reuses actions and step building blocks such as `./.github/workflows/_docs.yml`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 62-74 / 第 62-74 行

````yaml
    with:
      runner_prefix: "${{ needs.get-label-type.outputs.label-type }}"
      build-environment: linux-jammy-py3.10-gcc11
      docker-image: ${{ needs.docs-build.outputs.docker-image }}
      push: ${{ github.event_name == 'schedule' || github.event_name == 'workflow_dispatch' || startsWith(github.event.ref, 'refs/tags/v') }}
      run-doxygen: true
    secrets:
      GH_PYTORCHBOT_TOKEN: ${{ secrets.GH_PYTORCHBOT_TOKEN }}

  update-commit-hashes:
    runs-on: ubuntu-latest
    environment: update-commit-hash
    strategy:
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中。

### Lines 75-87 / 第 75-87 行

````yaml
      matrix:
        include:
          - repo-name: vision
            repo-owner: pytorch
            branch: main
            pin-folder: .github/ci_commit_pins
          - repo-name: audio
            repo-owner: pytorch
            branch: main
            pin-folder: .github/ci_commit_pins
          # executorch jobs are disabled since it needs some manual work for the hash update
          # - repo-name: executorch
          #   repo-owner: pytorch
````

- EN: This section describes repository automation behavior for `.github/workflows/nightly.yml`.
- CN: 该部分描述 `.github/workflows/nightly.yml` 的仓库自动化行为。

### Lines 88-97 / 第 88-97 行

````yaml
          #   branch: main
          #   pin-folder: .ci/docker/ci_commit_pins
          - repo-name: triton
            repo-owner: triton-lang
            branch: main
            pin-folder: .ci/docker/ci_commit_pins
          - repo-name: vllm
            repo-owner: vllm-project
            branch: main
            pin-folder: .github/ci_commit_pins
````

- EN: This section describes repository automation behavior for `.github/workflows/nightly.yml`.
- CN: 该部分描述 `.github/workflows/nightly.yml` 的仓库自动化行为。

### Lines 98-106 / 第 98-106 行

````yaml
          - repo-name: torchtitan
            repo-owner: pytorch
            branch: main
            pin-folder: .github/ci_commit_pins
    # Allow this to be triggered on either a schedule or on workflow_dispatch to allow for easier testing
    if: github.repository_owner == 'pytorch' && (github.event_name == 'schedule' || github.event_name == 'workflow_dispatch')
    steps:
      - name: "${{ matrix.repo-owner }}/${{ matrix.repo-name }} update-commit-hash"
        uses: pytorch/test-infra/.github/actions/update-commit-hash@main
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/update-commit-hash@main`.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程。

### Lines 107-113 / 第 107-113 行

````yaml
        with:
          repo-owner: ${{ matrix.repo-owner }}
          repo-name: ${{ matrix.repo-name }}
          branch: ${{ matrix.branch }}
          pin-folder: ${{ matrix.pin-folder}}
          updatebot-token: ${{ secrets.UPDATEBOT_TOKEN }}
          pytorchbot-token: ${{ secrets.GH_PYTORCHBOT_TOKEN }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。


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
- EN: Manual workflow triggering — maintainers can run this automation on demand from GitHub Actions.
  CN: 手动工作流触发——维护者可以在 GitHub Actions 中按需执行该自动化流程。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `./.github/workflows/_link_check.yml`, `./.github/workflows/_linux-build.yml`, `./.github/workflows/_docs.yml`, `pytorch/test-infra/.github/actions/update-commit-hash@main`
- Inline commands / 内联命令: none
- Environment variables / 环境变量: `NOTE`, `GH_PYTORCHBOT_TOKEN`, `UPDATEBOT_TOKEN`
- Named jobs or sections / 命名作业或章节: `on`, `schedule`, `push`, `tags`, `workflow_dispatch`, `concurrency`, `permissions`, `jobs`, `get-label-type`, `with`, ...
