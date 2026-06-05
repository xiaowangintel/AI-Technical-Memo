# build-manywheel-images-s390x.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/build-manywheel-images-s390x.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-9 / 第 1-9 行

````yaml
name: Build manywheel docker images for s390x

on:
  workflow_dispatch:
  push:
    tags:
      - ciflow/s390/*
    paths:
      - .github/workflows/build-manywheel-images-s390x.yml
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 12-21 / 第 12-21 行

````yaml
env:
  DOCKER_REGISTRY: "docker.io"
  DOCKER_BUILDKIT: 1
  WITH_PUSH: ${{ github.event_name == 'push' && (github.ref == 'refs/heads/main' || startsWith(github.ref, 'refs/heads/release') || startsWith(github.ref, 'refs/tags/v')) }}

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.sha }}-${{ github.event_name == 'workflow_dispatch' }}
  cancel-in-progress: true

jobs:
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；控制令牌权限范围或环境变量以实现安全执行。

### Lines 22-31 / 第 22-31 行

````yaml
  build-docker-cpu-s390x:
    if: github.repository_owner == 'pytorch'
    environment: ${{ (github.event_name == 'push' && (github.ref == 'refs/heads/main' || startsWith(github.ref, 'refs/heads/release') || startsWith(github.ref, 'refs/tags/v')) && 'docker-build') || '' }}
    runs-on: linux.s390x
    steps:
      - name: Checkout PyTorch
        uses: pytorch/pytorch/.github/actions/checkout-pytorch@main
        with:
          submodules: false
          no-sudo: true
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/checkout-pytorch@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 33-46 / 第 33-46 行

````yaml
      - name: Build Docker Image
        run: |
          .ci/docker/manywheel/build.sh manylinuxs390x-builder:cpu-s390x -t manylinuxs390x-builder:cpu-s390x

      - name: Tag and (if WITH_PUSH) push docker image to docker.io
        env:
          DOCKER_TOKEN: ${{ secrets.DOCKER_TOKEN }}
          DOCKER_ID: ${{ secrets.DOCKER_ID }}
          CREATED_FULL_DOCKER_IMAGE_NAME: manylinuxs390x-builder:cpu-s390x
        shell: bash
        run: |
          set -euox pipefail
          GITHUB_REF="${GITHUB_REF:-$(git symbolic-ref -q HEAD || git describe --tags --exact-match)}"
          GIT_BRANCH_NAME="${GITHUB_REF##*/}"
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 47-56 / 第 47-56 行

````yaml
          GIT_COMMIT_SHA="${GITHUB_SHA:-$(git rev-parse HEAD)}"
          CI_FOLDER_SHA="$(git rev-parse HEAD:.ci/docker)"

          DOCKER_IMAGE_NAME_PREFIX="docker.io/pytorch/${CREATED_FULL_DOCKER_IMAGE_NAME}"

          docker tag "${CREATED_FULL_DOCKER_IMAGE_NAME}" "${DOCKER_IMAGE_NAME_PREFIX}-${GIT_BRANCH_NAME}"
          docker tag "${CREATED_FULL_DOCKER_IMAGE_NAME}" "${DOCKER_IMAGE_NAME_PREFIX}-${GIT_COMMIT_SHA}"
          docker tag "${CREATED_FULL_DOCKER_IMAGE_NAME}" "${DOCKER_IMAGE_NAME_PREFIX}-${CI_FOLDER_SHA}"

          # Pretty sure Github will mask tokens and I'm not sure if it will even be
````

- EN: This section describes repository automation behavior for `.github/workflows/build-manywheel-images-s390x.yml`.
- CN: 该部分描述 `.github/workflows/build-manywheel-images-s390x.yml` 的仓库自动化行为。

### Lines 57-69 / 第 57-69 行

````yaml
          # printed due to pipe, but just in case
          set +x
          if [[ "${WITH_PUSH:-false}" == "true" ]]; then
            echo "${DOCKER_TOKEN}" | docker login -u "${DOCKER_ID}" --password-stdin
            docker push "${DOCKER_IMAGE_NAME_PREFIX}-${GIT_BRANCH_NAME}"
            docker push "${DOCKER_IMAGE_NAME_PREFIX}-${GIT_COMMIT_SHA}"
            docker push "${DOCKER_IMAGE_NAME_PREFIX}-${CI_FOLDER_SHA}"
          fi

      - name: Cleanup docker
        if: cancelled()
        shell: bash
        run: |
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 70-74 / 第 70-74 行

````yaml
          # If podman build command is interrupted,
          # it can leave a couple of processes still running.
          # Order them to stop for clean shutdown.
          # It looks like sometimes some processes remain
          # after first cleanup.
````

- EN: This section describes repository automation behavior for `.github/workflows/build-manywheel-images-s390x.yml`.
- CN: 该部分描述 `.github/workflows/build-manywheel-images-s390x.yml` 的仓库自动化行为。

### Lines 75-78 / 第 75-78 行

````yaml
          # Wait a bit and do cleanup again. It looks like it helps.
          docker system prune --build -f || true
          sleep 60
          docker system prune --build -f || true
````

- EN: This section describes repository automation behavior for `.github/workflows/build-manywheel-images-s390x.yml`.
- CN: 该部分描述 `.github/workflows/build-manywheel-images-s390x.yml` 的仓库自动化行为。


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
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/actions/checkout-pytorch@main`
- Inline commands / 内联命令: `.ci/docker/manywheel/build.sh`, `set`, `docker`, `echo`, `sleep`
- Environment variables / 环境变量: `DOCKER_REGISTRY`, `DOCKER_BUILDKIT`, `WITH_PUSH`, `DOCKER_TOKEN`, `DOCKER_ID`, `CREATED_FULL_DOCKER_IMAGE_NAME`, `GITHUB_REF`, `HEAD`, `GIT_BRANCH_NAME`, `GIT_COMMIT_SHA`, ...
- Named jobs or sections / 命名作业或章节: `on`, `workflow_dispatch`, `push`, `tags`, `paths`, `env`, `concurrency`, `jobs`, `build-docker-cpu-s390x`, `steps`
