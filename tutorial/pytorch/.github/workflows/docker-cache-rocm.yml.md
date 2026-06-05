# docker-cache-rocm.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/docker-cache-rocm.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-10 / 第 1-10 行

````yaml
name: docker-cache-rocm

on:
  workflow_run:
    workflows: [docker-builds]
    branches: [main, release]
    types:
      - completed
  workflow_dispatch:
    inputs:
````

- EN: This section declares workflow triggers or event bindings; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 11-22 / 第 11-22 行

````yaml
      branch:
        type: string
        description: Branch corresponding to the docker images being cached
        required: true
      run_id:
        type: string
        description: Workflow run id to pull artifacts from
        required: true

concurrency:
  group: ${{ github.workflow }}-${{ github.event.workflow_run.head_branch || github.event.inputs.branch }}
  cancel-in-progress: true
````

- EN: This section describes repository automation behavior for `.github/workflows/docker-cache-rocm.yml`.
- CN: 该部分描述 `.github/workflows/docker-cache-rocm.yml` 的仓库自动化行为。

### Lines 24-33 / 第 24-33 行

````yaml
permissions:
  id-token: write
  contents: read
  actions: read

jobs:
  download-docker-builds-artifacts:
    if: github.repository_owner == 'pytorch'
    name: download-docker-builds-artifacts
    runs-on: ubuntu-latest
````

- EN: This section lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 34-45 / 第 34-45 行

````yaml
    outputs:
      pytorch-linux-jammy-rocm-n-py3: ${{ steps.process-artifacts.outputs.pytorch-linux-jammy-rocm-n-py3 }}
      pytorch-linux-noble-rocm-n-py3: ${{ steps.process-artifacts.outputs.pytorch-linux-noble-rocm-n-py3 }}
      pytorch-linux-jammy-rocm-n-py3-benchmarks: ${{ steps.process-artifacts.outputs.pytorch-linux-jammy-rocm-n-py3-benchmarks }}
    steps:
      - name: Download artifacts
        uses: actions/download-artifact@65a9edc5881444af0b9093a5e628f2fe47ea3b2e #4.1.7
        with:
          run-id: ${{ github.event.workflow_run.id || github.event.inputs.run_id }}
          path: ./docker-builds-artifacts
          merge-multiple: true
          github-token: ${{ secrets.GITHUB_TOKEN }}
````

- EN: This section reuses actions and step building blocks such as `actions/download-artifact@65a9edc5881444af0b9093a5e628f2fe47ea3b2e`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 47-58 / 第 47-58 行

````yaml
      - name: Process artifacts
        id: process-artifacts
        run: |
          ls -R ./docker-builds-artifacts
          cat ./docker-builds-artifacts/*txt >> "${GITHUB_OUTPUT}"
          cat "${GITHUB_OUTPUT}"

  docker-cache:
    if: github.repository_owner == 'pytorch'
    needs: download-docker-builds-artifacts
    strategy:
      fail-fast: false
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 59-71 / 第 59-71 行

````yaml
      matrix:
        runner: [linux.rocm.mi250.docker-cache, linux.rocm.mi210.docker-cache]
        docker-image: [
          "${{ needs.download-docker-builds-artifacts.outputs.pytorch-linux-jammy-rocm-n-py3 }}",
          "${{ needs.download-docker-builds-artifacts.outputs.pytorch-linux-noble-rocm-n-py3 }}"
          #"${{ needs.download-docker-builds-artifacts.outputs.pytorch-linux-jammy-rocm-n-py3-benchmarks }}"
        ]
    runs-on: "${{ matrix.runner }}"
    steps:
      - name: debug
        run: |
          JSON_STRINGIFIED="${{ toJSON(needs.download-docker-builds-artifacts.outputs) }}"
          echo "Outputs of download-docker-builds-artifacts job: ${JSON_STRINGIFIED}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 73-80 / 第 73-80 行

````yaml
      - name: Checkout PyTorch
        uses: pytorch/pytorch/.github/actions/checkout-pytorch@main
        with:
          no-sudo: true
          submodules: 'false'

      - name: Login to ECR
        uses: ./.github/actions/ecr-login
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `./.github/actions/ecr-login`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 82-92 / 第 82-92 行

````yaml
      - name: Generate ghrc.io tag
        id: ghcr-io-tag
        run: |
            ecr_image="${{ matrix.docker-image }}"
            ghcr_image="ghcr.io/pytorch/ci-image:${ecr_image##*:}"
            echo "ghcr_image=${ghcr_image}" >> "$GITHUB_OUTPUT"

      - name: Pull docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.ghcr-io-tag.outputs.ghcr_image }}
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/pull-docker-image@main`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 94-107 / 第 94-107 行

````yaml
      - name: Save as tarball
        run: |
          docker_image_tag=${{ matrix.docker-image }}
          docker_image_tag="${docker_image_tag#*:}" # Remove everything before and including first ":"
          docker_image_tag="${docker_image_tag%-*}" # Remove everything after and including last "-"
          ref_name=${WORKFLOW_RUN_HEAD_BRANCH}
          if [[ $ref_name =~ "release/" ]]; then
            ref_suffix="release"
          elif [[ $ref_name == "main" ]]; then
            ref_suffix="main"
          else
            echo "Unexpected branch in ref_name: ${ref_name}" && exit 1
          fi
          docker tag ${{ steps.ghcr-io-tag.outputs.ghcr_image }} ${{ matrix.docker-image }}
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 108-110 / 第 108-110 行

````yaml
          # mv is atomic operation, so we use intermediate tar.tmp file to prevent read-write contention
          docker save -o ~/pytorch-data/docker/${docker_image_tag}.tar.tmp ${{ matrix.docker-image }}
          mv ~/pytorch-data/docker/${docker_image_tag}.tar.tmp ~/pytorch-data/docker/${docker_image_tag}_${ref_suffix}.tar
````

- EN: This section describes repository automation behavior for `.github/workflows/docker-cache-rocm.yml`.
- CN: 该部分描述 `.github/workflows/docker-cache-rocm.yml` 的仓库自动化行为。

### Lines 112-113 / 第 112-113 行

````yaml
        env:
          WORKFLOW_RUN_HEAD_BRANCH: ${{ github.event.workflow_run.head_branch || github.event.inputs.branch }}
````

- EN: This section controls token scope or environment variables for safe execution.
- CN: 该部分控制令牌权限范围或环境变量以实现安全执行。


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
- Reusable actions / 复用 Action: `actions/download-artifact@65a9edc5881444af0b9093a5e628f2fe47ea3b2e`, `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `./.github/actions/ecr-login`, `pytorch/test-infra/.github/actions/pull-docker-image@main`
- Inline commands / 内联命令: `ls`, `cat`, `"${{`, `]`, `echo`, `docker`, `mv`
- Environment variables / 环境变量: `GITHUB_TOKEN`, `GITHUB_OUTPUT`, `JSON_STRINGIFIED`, `ECR`, `WORKFLOW_RUN_HEAD_BRANCH`
- Named jobs or sections / 命名作业或章节: `on`, `workflow_run`, `types`, `workflow_dispatch`, `inputs`, `concurrency`, `permissions`, `jobs`, `download-docker-builds-artifacts`, `outputs`, ...
