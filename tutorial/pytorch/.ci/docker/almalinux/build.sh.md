# build.sh — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `.ci/docker/almalinux/build.sh`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Automates repository maintenance, build, test, packaging, or release tasks through shell commands. The opening comment frames the file as: "!/usr/bin/env bash Script used only in CD pipeline."
- **Purpose (CN)**: 通过 shell 命令自动化执行仓库维护、构建、测试、打包或发布任务。 开头注释将该文件概括为：“!/usr/bin/env bash Script used only in CD pipeline”。

## Content Analysis / 内容分析

### Lines 1-5 / 第 1-5 行

```bash
#!/usr/bin/env bash
# Script used only in CD pipeline

set -exou pipefail

```

- **EN:** This chunk introduces sections such as !/usr/bin/env bash, Script used only in CD pipeline, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 !/usr/bin/env bash、Script used only in CD pipeline 等标题组织周边说明或配置。

### Lines 6-13 / 第 6-13 行

```bash
image="$1"
shift

if [ -z "${image}" ]; then
  echo "Usage: $0 IMAGENAME:ARCHTAG"
  exit 1
fi

```

- **EN:** It invokes commands such as shift, exit, showing the operational steps the workflow performs.
- **CN:** 它调用了 shift、exit 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as IMAGENAME, ARCHTAG communicate required tool locations or behavioral switches.
- **CN:** IMAGENAME、ARCHTAG 等环境变量用于说明所需工具位置或行为开关。

### Lines 14-21 / 第 14-21 行

```bash
# Go from imagename:tag to tag
DOCKER_TAG_PREFIX=$(echo "${image}" | awk -F':' '{print $2}')

CUDA_VERSION=""
ROCM_VERSION=""
EXTRA_BUILD_ARGS=""
if [[ "${DOCKER_TAG_PREFIX}" == cuda* ]]; then
    # extract cuda version from image name and tag.  e.g. manylinux2_28-builder:cuda12.8 returns 12.8
```

- **EN:** This chunk introduces sections such as Go from imagename:tag to tag, extract cuda version from image name and tag.  e.g. manylinux2_28-builder:cuda12.8 returns 12.8, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Go from imagename:tag to tag、extract cuda version from image name and tag.  e.g. manylinux2_28-builder:cuda12.8 returns 12.8 等标题组织周边说明或配置。
- **EN:** Environment variables such as DOCKER_TAG_PREFIX, CUDA_VERSION, ROCM_VERSION, EXTRA_BUILD_ARGS communicate required tool locations or behavioral switches.
- **CN:** DOCKER_TAG_PREFIX、CUDA_VERSION、ROCM_VERSION、EXTRA_BUILD_ARGS 等环境变量用于说明所需工具位置或行为开关。

### Lines 22-29 / 第 22-29 行

```bash
    CUDA_VERSION=$(echo "${DOCKER_TAG_PREFIX}" | awk -F'cuda' '{print $2}')
    EXTRA_BUILD_ARGS="--build-arg CUDA_VERSION=${CUDA_VERSION}"
elif [[ "${DOCKER_TAG_PREFIX}" == rocm* ]]; then
    # extract rocm version from image name and tag.  e.g. manylinux2_28-builder:rocm6.2.4 returns 6.2.4
    ROCM_VERSION=$(echo "${DOCKER_TAG_PREFIX}" | awk -F'rocm' '{print $2}')
    EXTRA_BUILD_ARGS="--build-arg ROCM_IMAGE=rocm/dev-almalinux-8:${ROCM_VERSION}-complete"
fi

```

- **EN:** This chunk introduces sections such as extract rocm version from image name and tag.  e.g. manylinux2_28-builder:rocm6.2.4 returns 6.2.4, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 extract rocm version from image name and tag.  e.g. manylinux2_28-builder:rocm6.2.4 returns 6.2.4 等标题组织周边说明或配置。
- **EN:** Environment variables such as CUDA_VERSION, DOCKER_TAG_PREFIX, EXTRA_BUILD_ARGS, ROCM_VERSION, ROCM_IMAGE communicate required tool locations or behavioral switches.
- **CN:** CUDA_VERSION、DOCKER_TAG_PREFIX、EXTRA_BUILD_ARGS、ROCM_VERSION、ROCM_IMAGE 等环境变量用于说明所需工具位置或行为开关。

### Lines 30-39 / 第 30-39 行

```bash
case ${DOCKER_TAG_PREFIX} in
  cpu)
    BASE_TARGET=base
    ;;
  cuda*)
    BASE_TARGET=cuda${CUDA_VERSION}
    ;;
  rocm*)
    BASE_TARGET=rocm
    PYTORCH_ROCM_ARCH="gfx900;gfx906;gfx908;gfx90a;gfx942;gfx1030;gfx1100;gfx1101;gfx1102;gfx1103;gfx1200;gfx1201;gfx950;gfx1150;gfx1151"
```

- **EN:** It invokes commands such as cpu, cuda, rocm, showing the operational steps the workflow performs.
- **CN:** 它调用了 cpu、cuda、rocm 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as DOCKER_TAG_PREFIX, BASE_TARGET, CUDA_VERSION, PYTORCH_ROCM_ARCH communicate required tool locations or behavioral switches.
- **CN:** DOCKER_TAG_PREFIX、BASE_TARGET、CUDA_VERSION、PYTORCH_ROCM_ARCH 等环境变量用于说明所需工具位置或行为开关。

### Lines 40-47 / 第 40-47 行

```bash
    EXTRA_BUILD_ARGS="${EXTRA_BUILD_ARGS} --build-arg PYTORCH_ROCM_ARCH=${PYTORCH_ROCM_ARCH}"
    ;;
  *)
    echo "ERROR: Unknown docker tag ${DOCKER_TAG_PREFIX}"
    exit 1
    ;;
esac

```

- **EN:** It invokes commands such as exit, showing the operational steps the workflow performs.
- **CN:** 它调用了 exit 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as EXTRA_BUILD_ARGS, PYTORCH_ROCM_ARCH, ERROR, DOCKER_TAG_PREFIX communicate required tool locations or behavioral switches.
- **CN:** EXTRA_BUILD_ARGS、PYTORCH_ROCM_ARCH、ERROR、DOCKER_TAG_PREFIX 等环境变量用于说明所需工具位置或行为开关。

### Lines 48-53 / 第 48-53 行

```bash
# TODO: Remove LimitNOFILE=1048576 patch once https://github.com/pytorch/test-infra/issues/5712
# is resolved. This patch is required in order to fix timing out of Docker build on Amazon Linux 2023.
sudo sed -i s/LimitNOFILE=infinity/LimitNOFILE=1048576/ /usr/lib/systemd/system/docker.service
sudo systemctl daemon-reload
sudo systemctl restart docker

```

- **EN:** This chunk introduces sections such as TODO: Remove LimitNOFILE=1048576 patch once https://github.com/pytorch/test-infra/issues/5712, is resolved. This patch is required in order to fix timing out of Docker build on Amazon Linux 2023., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 TODO: Remove LimitNOFILE=1048576 patch once https://github.com/pytorch/test-infra/issues/5712、is resolved. This patch is required in order to fix timing out of Docker build on Amazon Linux 2023. 等标题组织周边说明或配置。
- **EN:** It invokes commands such as sed, systemctl, showing the operational steps the workflow performs.
- **CN:** 它调用了 sed、systemctl 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as TODO communicate required tool locations or behavioral switches.
- **CN:** TODO 等环境变量用于说明所需工具位置或行为开关。

### Lines 54-63 / 第 54-63 行

```bash
export DOCKER_BUILDKIT=1
TOPDIR=$(git rev-parse --show-toplevel)
tmp_tag=$(basename "$(mktemp -u)" | tr '[:upper:]' '[:lower:]')

docker build \
  --target final \
  --progress plain \
  --build-arg "BASE_TARGET=${BASE_TARGET}" \
  --build-arg "DEVTOOLSET_VERSION=13" \
  ${EXTRA_BUILD_ARGS} \
```

- **EN:** It invokes commands such as docker, --target, --progress, --build-arg, showing the operational steps the workflow performs.
- **CN:** 它调用了 docker、--target、--progress、--build-arg 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as DOCKER_BUILDKIT, TOPDIR, BASE_TARGET, DEVTOOLSET_VERSION, EXTRA_BUILD_ARGS communicate required tool locations or behavioral switches.
- **CN:** DOCKER_BUILDKIT、TOPDIR、BASE_TARGET、DEVTOOLSET_VERSION、EXTRA_BUILD_ARGS 等环境变量用于说明所需工具位置或行为开关。

### Lines 64-68 / 第 64-68 行

```bash
  -t ${tmp_tag} \
  $@ \
  -f "${TOPDIR}/.ci/docker/almalinux/Dockerfile" \
  ${TOPDIR}/.ci/docker/

```

- **EN:** It invokes commands such as -t, -f, showing the operational steps the workflow performs.
- **CN:** 它调用了 -t、-f 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as TOPDIR communicate required tool locations or behavioral switches.
- **CN:** TOPDIR 等环境变量用于说明所需工具位置或行为开关。

### Lines 69-72 / 第 69-72 行

```bash
if [ -n "${CUDA_VERSION}" ]; then
  # Test that we're using the right CUDA compiler
  docker run --rm "${tmp_tag}" nvcc --version | grep "cuda_${CUDA_VERSION}"
fi
```

- **EN:** This chunk introduces sections such as Test that we're using the right CUDA compiler, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Test that we're using the right CUDA compiler 等标题组织周边说明或配置。
- **EN:** It invokes commands such as docker, showing the operational steps the workflow performs.
- **CN:** 它调用了 docker 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CUDA_VERSION, CUDA communicate required tool locations or behavioral switches.
- **CN:** CUDA_VERSION、CUDA 等环境变量用于说明所需工具位置或行为开关。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **Repository bootstrap** — 与仓库状态、子模块或源码检出流程交互。

## Dependencies / 依赖关系

- `bash`
- `git`
