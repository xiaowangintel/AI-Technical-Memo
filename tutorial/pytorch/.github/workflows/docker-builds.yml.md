# docker-builds.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/docker-builds.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-17 / 第 1-17 行

````yaml
name: docker-builds

on:
  workflow_dispatch:
  pull_request:
    paths:
      - .ci/docker/**
      - .github/workflows/docker-builds.yml
      - .lintrunner.toml
  push:
    branches:
      - main
      - release/*
      - landchecks/*
    paths:
      - .ci/docker/**
      - .github/workflows/docker-builds.yml
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 18-32 / 第 18-32 行

````yaml
      - .lintrunner.toml
  schedule:
    - cron: 1 3 * * 3

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.sha }}-${{ github.event_name == 'workflow_dispatch' }}
  cancel-in-progress: true

env:
  ALPINE_IMAGE: 308535385114.dkr.ecr.us-east-1.amazonaws.com/tool/alpine
  AWS_DEFAULT_REGION: us-east-1

permissions: read-all

jobs:
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 33-48 / 第 33-48 行

````yaml
  get-label-type:
    if: github.repository_owner == 'pytorch'
    name: get-label-type
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}

  docker-build:
    environment: ${{ (github.ref == 'refs/heads/main' || startsWith(github.event.ref, 'refs/tags/v')) && 'docker-build' || '' }}
    needs: get-label-type
    timeout-minutes: 240
    strategy:
      fail-fast: false
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 49-66 / 第 49-66 行

````yaml
      matrix:
        runner: [linux.12xlarge]
        docker-image-name: [
          pytorch-linux-jammy-cuda12.8-cudnn9-py3-gcc11,
          pytorch-linux-jammy-cuda13.0-cudnn9-py3-gcc11,
          pytorch-linux-jammy-cuda13.0-cudnn9-py3.12-gcc11-vllm,
          pytorch-linux-jammy-cuda13.0-cudnn9-py3-gcc11-inductor-benchmarks,
          pytorch-linux-jammy-py3.10-clang18,
          pytorch-linux-jammy-py3.11-clang18,
          pytorch-linux-jammy-py3.12-clang18,
          pytorch-linux-jammy-py3.13-clang18,
          pytorch-linux-jammy-py3.14-clang18,
          pytorch-linux-jammy-py3.14t-clang18,
          pytorch-linux-jammy-rocm-n-py3,
          pytorch-linux-noble-rocm-n-py3,
          pytorch-linux-noble-rocm-nightly-py3,
          pytorch-linux-jammy-rocm-n-py3-benchmarks,
          pytorch-linux-jammy-cuda12.8-cudnn9-py3.10-clang18,
````

- EN: This section describes repository automation behavior for `.github/workflows/docker-builds.yml`.
- CN: 该部分描述 `.github/workflows/docker-builds.yml` 的仓库自动化行为。

### Lines 67-83 / 第 67-83 行

````yaml
          pytorch-linux-jammy-py3-gcc11-inductor-benchmarks,
          pytorch-linux-jammy-py3.12-halide,
          pytorch-linux-jammy-py3.12-pallas,
          pytorch-linux-jammy-cuda12.8-py3.12-pallas,
          pytorch-linux-jammy-tpu-py3.12-pallas,
          pytorch-linux-jammy-xpu-n-1-py3,
          pytorch-linux-noble-xpu-n-py3,
          pytorch-linux-noble-xpu-n-py3-client,
          pytorch-linux-noble-xpu-n-py3-inductor-benchmarks,
          pytorch-linux-jammy-linter,
          pytorch-linux-jammy-cuda12.8-cudnn9-py3.10-linter,
          # TODO: Re-enable me when docker pin update happens
          # pytorch-linux-jammy-py3-clang18-executorch,
          pytorch-linux-jammy-py3.12-triton-cpu,
          pytorch-linux-noble-riscv64-py3.12-gcc14
        ]
        include:
````

- EN: This section describes repository automation behavior for `.github/workflows/docker-builds.yml`.
- CN: 该部分描述 `.github/workflows/docker-builds.yml` 的仓库自动化行为。

### Lines 84-98 / 第 84-98 行

````yaml
          - docker-image-name: pytorch-linux-jammy-aarch64-py3.10-gcc13
            runner: linux.arm64.m7g.4xlarge
          - docker-image-name: pytorch-linux-jammy-aarch64-py3.10-gcc13-inductor-benchmarks
            runner: linux.arm64.m7g.4xlarge
            timeout-minutes: 600
    # Docker uploads fail from LF runners, see https://github.com/pytorch/pytorch/pull/137358
    # runs-on: "${{ needs.get-label-type.outputs.label-type }}${{ matrix.runner }}"
    runs-on: "${{ matrix.runner }}"
    steps:
      - name: Clean workspace
        shell: bash
        run: |
          echo "${GITHUB_WORKSPACE}"
          sudo rm -rf "${GITHUB_WORKSPACE}"
          mkdir "${GITHUB_WORKSPACE}"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 100-114 / 第 100-114 行

````yaml
      - name: Setup Linux
        uses: pytorch/pytorch/.github/actions/setup-linux@main
        with:
          submodules: 'false'

      - name: Login to ECR
        uses: ./.github/actions/ecr-login

      - name: Build docker image
        id: build-docker-image
        uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
        with:
          docker-image-name: ci-image:${{ matrix.docker-image-name }}
          always-rebuild: true
          push: true
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/setup-linux@main`, `./.github/actions/ecr-login`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 116-131 / 第 116-131 行

````yaml
      - name: Pull docker image
        uses: pytorch/test-infra/.github/actions/pull-docker-image@main
        with:
          docker-image: ${{ steps.build-docker-image.outputs.docker-image }}

      - name: Generate output
        if: contains(matrix.docker-image-name, 'rocm')
        id: generate_output
        run: |
          docker_image_name="${{ matrix.docker-image-name }}"
          docker_image_tag="${{ steps.build-docker-image.outputs.docker-image }}"
          echo "${docker_image_name}=${docker_image_tag}" >> docker-builds-output-${docker_image_name}.txt

      - name: Upload artifacts
        uses: actions/upload-artifact@v4.4.0
        if: contains(matrix.docker-image-name, 'rocm')
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/pull-docker-image@main`, `actions/upload-artifact@v4.4.0`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 132-143 / 第 132-143 行

````yaml
        with:
          name: docker-builds-artifacts-${{ matrix.docker-image-name }}
          retention-days: 14
          path: ./docker-builds-output-${{ matrix.docker-image-name }}.txt

      - uses: nick-fields/retry@7152eba30c6575329ac0576536151aca5a72780e # v3.0.0
        name: Push to https://ghcr.io/
        id: push-to-ghcr-io
        if: ${{ github.event_name == 'push' }}
        env:
          ECR_DOCKER_IMAGE: ${{ steps.build-docker-image.outputs.docker-image }}
          GHCR_PAT: ${{ secrets.GHCR_PAT }}
````

- EN: This section threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 144-158 / 第 144-158 行

````yaml
        with:
          shell: bash
          timeout_minutes: 60
          max_attempts: 5
          retry_wait_seconds: 90
          command: |
            ghcr_image="ghcr.io/pytorch/ci-image"
            tag=${ECR_DOCKER_IMAGE##*:}
            # Push docker image to the ghcr.io
            echo $GHCR_PAT | docker login ghcr.io -u pytorch --password-stdin
            docker tag "${ECR_DOCKER_IMAGE}" "${ghcr_image}:${tag}"
            docker push "${ghcr_image}:${tag}"
            # Also push a tag without the hash for easier reference
            docker tag "${ECR_DOCKER_IMAGE}" "${ghcr_image}:${{ matrix.docker-image-name }}"
            docker push "${ghcr_image}:${{ matrix.docker-image-name }}"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 160-164 / 第 160-164 行

````yaml
      - name: Chown workspace
        uses: ./.github/actions/chown-workspace
        with:
          ALPINE_IMAGE: 308535385114.dkr.ecr.us-east-1.amazonaws.com/${{ contains(matrix.runner, 'arm64') && 'arm64v8' || 'tool' }}/alpine
        if: always()
````

- EN: This section reuses actions and step building blocks such as `./.github/actions/chown-workspace`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 166-168 / 第 166-168 行

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
- EN: Configures native build inputs, optional backends, and dependency discovery at configure time.
  CN: 在配置阶段设置原生构建输入、可选后端以及依赖发现逻辑。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `pytorch/pytorch/.github/actions/setup-linux@main`, `./.github/actions/ecr-login`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`, `pytorch/test-infra/.github/actions/pull-docker-image@main`, `actions/upload-artifact@v4.4.0`, `./.github/actions/chown-workspace`, `pytorch/test-infra/.github/actions/teardown-linux@main`
- Inline commands / 内联命令: `pytorch-linux-jammy-cuda12.8-cudnn9-py3-gcc11,`, `pytorch-linux-jammy-cuda13.0-cudnn9-py3-gcc11,`, `pytorch-linux-jammy-cuda13.0-cudnn9-py3.12-gcc11-vllm,`, `pytorch-linux-jammy-cuda13.0-cudnn9-py3-gcc11-inductor-benchmarks,`, `pytorch-linux-jammy-py3.10-clang18,`, `pytorch-linux-jammy-py3.11-clang18,`, `pytorch-linux-jammy-py3.12-clang18,`, `pytorch-linux-jammy-py3.13-clang18,`, ...
- Environment variables / 环境变量: `ALPINE_IMAGE`, `AWS_DEFAULT_REGION`, `TODO`, `GITHUB_WORKSPACE`, `ECR`, `ECR_DOCKER_IMAGE`, `GHCR_PAT`
- Named jobs or sections / 命名作业或章节: `on`, `workflow_dispatch`, `pull_request`, `paths`, `push`, `branches`, `schedule`, `concurrency`, `env`, `jobs`, ...
