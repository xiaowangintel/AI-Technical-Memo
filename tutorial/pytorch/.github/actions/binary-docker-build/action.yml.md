# action.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/actions/binary-docker-build/action.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a reusable composite GitHub Action, including declared inputs, step composition, and shell execution.
- **用途 (CN)**: 定义可复用的复合型 GitHub Action，包括输入声明、步骤组合以及 shell 执行逻辑。

## Content Analysis / 内容分析
### Lines 1-11 / 第 1-11 行

````yaml
name: Binary docker build

description: Build docker image for binary builds

inputs:
  docker-image-name:
    description: Docker image name for PR builds
    required: true
  docker-build-dir:
    description: Location of the build.sh relative to .ci/docker
    required: true
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 12-24 / 第 12-24 行

````yaml
  custom-tag-prefix:
    description: Custom tag prefix for the docker image
    required: false
  DOCKER_TOKEN:
    description: Docker token for authentication
    required: true
  DOCKER_ID:
    description: Docker ID for authentication
    required: true

runs:
  using: composite
  steps:
````

- EN: This section reuses actions and step building blocks such as named actions.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 25-37 / 第 25-37 行

````yaml
    - name: Checkout PyTorch
      uses: pytorch/pytorch/.github/actions/checkout-pytorch@main

    - name: Calculate docker image
      id: calculate-docker-image
      uses: pytorch/test-infra/.github/actions/calculate-docker-image@main
      with:
        docker-image-name: ${{ inputs.docker-image-name }}
        docker-build-dir: .ci/docker
        custom-tag-prefix: ${{ inputs.custom-tag-prefix }}
        docker-build-script: ${{ inputs.docker-build-dir }}/build.sh
        always-rebuild: true
        push: true
````

- EN: This section reuses actions and step building blocks such as `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 39-52 / 第 39-52 行

````yaml
    - name: Tag and (if WITH_PUSH) push docker image to docker.io
      env:
        DOCKER_TOKEN: ${{ inputs.DOCKER_TOKEN }}
        DOCKER_ID: ${{ inputs.DOCKER_ID }}
        DOCKER_IMAGE_NAME: ${{ inputs.docker-image-name }}
        DOCKER_IMAGE_PREFIX: ${{ inputs.custom-tag-prefix }}
        CREATED_FULL_DOCKER_IMAGE_NAME: ${{ steps.calculate-docker-image.outputs.docker-image }}
      shell: bash
      run: |
        set -euox pipefail
        GITHUB_REF=${GITHUB_REF:-$(git symbolic-ref -q HEAD || git describe --tags --exact-match)}
        GIT_BRANCH_NAME=${GITHUB_REF##*/}
        GIT_COMMIT_SHA=${GITHUB_SHA:-$(git rev-parse HEAD)}
        CI_FOLDER_SHA=$(git rev-parse HEAD:.ci/docker)
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 54-67 / 第 54-67 行

````yaml
        DOCKER_IMAGE_NAME_PREFIX=docker.io/pytorch/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_PREFIX}

        docker tag ${CREATED_FULL_DOCKER_IMAGE_NAME} ${DOCKER_IMAGE_NAME_PREFIX}
        docker tag ${CREATED_FULL_DOCKER_IMAGE_NAME} ${DOCKER_IMAGE_NAME_PREFIX}-${GIT_BRANCH_NAME}
        docker tag ${CREATED_FULL_DOCKER_IMAGE_NAME} ${DOCKER_IMAGE_NAME_PREFIX}-${GIT_COMMIT_SHA}
        docker tag ${CREATED_FULL_DOCKER_IMAGE_NAME} ${DOCKER_IMAGE_NAME_PREFIX}-${CI_FOLDER_SHA}

        # Pretty sure Github will mask tokens and I'm not sure if it will even be
        # printed due to pipe, but just in case
        set +x
        if [[ ${WITH_PUSH:-false} == "true" ]]; then
          echo "${DOCKER_TOKEN}" | docker login -u "${DOCKER_ID}" --password-stdin
          docker push ${DOCKER_IMAGE_NAME_PREFIX}
          docker push ${DOCKER_IMAGE_NAME_PREFIX}-${GIT_BRANCH_NAME}
````

- EN: This section describes repository automation behavior for `.github/actions/binary-docker-build/action.yml`.
- CN: 该部分描述 `.github/actions/binary-docker-build/action.yml` 的仓库自动化行为。

### Lines 68-70 / 第 68-70 行

````yaml
          docker push ${DOCKER_IMAGE_NAME_PREFIX}-${GIT_COMMIT_SHA}
          docker push ${DOCKER_IMAGE_NAME_PREFIX}-${CI_FOLDER_SHA}
        fi
````

- EN: This section describes repository automation behavior for `.github/actions/binary-docker-build/action.yml`.
- CN: 该部分描述 `.github/actions/binary-docker-build/action.yml` 的仓库自动化行为。


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
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/actions/checkout-pytorch@main`, `pytorch/test-infra/.github/actions/calculate-docker-image@main`
- Inline commands / 内联命令: `set`, `docker`, `echo`
- Environment variables / 环境变量: `DOCKER_TOKEN`, `DOCKER_ID`, `WITH_PUSH`, `DOCKER_IMAGE_NAME`, `DOCKER_IMAGE_PREFIX`, `CREATED_FULL_DOCKER_IMAGE_NAME`, `GITHUB_REF`, `HEAD`, `GIT_BRANCH_NAME`, `GIT_COMMIT_SHA`, ...
- Named jobs or sections / 命名作业或章节: `inputs`, `docker-image-name`, `docker-build-dir`, `custom-tag-prefix`, `DOCKER_TOKEN`, `DOCKER_ID`, `runs`, `steps`
