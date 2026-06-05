# build.sh — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `.ci/flash-attention/build.sh`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Automates repository maintenance, build, test, packaging, or release tasks through shell commands. The opening comment frames the file as: "!/bin/bash."
- **Purpose (CN)**: 通过 shell 命令自动化执行仓库维护、构建、测试、打包或发布任务。 开头注释将该文件概括为：“!/bin/bash”。

## Content Analysis / 内容分析

### Lines 1-10 / 第 1-10 行

```bash
#!/bin/bash

set -ex -o pipefail

PYTORCH_ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/../.." && pwd)"

source "${PYTORCH_ROOT}/.ci/pytorch/common_utils.sh"
FLASH_ATTENTION_DIR="${PYTORCH_ROOT}/third_party/flash-attention"
FLASH_ATTENTION_HOPPER_DIR="${FLASH_ATTENTION_DIR}/hopper"

```

- **EN:** This chunk introduces sections such as !/bin/bash, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 !/bin/bash 等标题组织周边说明或配置。
- **EN:** It invokes commands such as source, showing the operational steps the workflow performs.
- **CN:** 它调用了 source 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as PYTORCH_ROOT, BASH_SOURCE, FLASH_ATTENTION_DIR, FLASH_ATTENTION_HOPPER_DIR communicate required tool locations or behavioral switches.
- **CN:** PYTORCH_ROOT、BASH_SOURCE、FLASH_ATTENTION_DIR、FLASH_ATTENTION_HOPPER_DIR 等环境变量用于说明所需工具位置或行为开关。

### Lines 11-20 / 第 11-20 行

```bash
[[ -z "$FA_FINAL_PACKAGE_DIR" ]] && fatal "FA_FINAL_PACKAGE_DIR must be set"
[[ -z "$MANYLINUX_PLAT" ]] && fatal "MANYLINUX_PLAT must be set"
[[ -z "$CUDA_VERSION" ]] && fatal "CUDA_VERSION must be set"
[[ -z "$CUDA_SHORT" ]] && fatal "CUDA_SHORT must be set"
[[ -z "$PYTHON_VERSION" ]] && fatal "PYTHON_VERSION must be set"
[[ ! -d "$FLASH_ATTENTION_HOPPER_DIR" ]] && fatal "flash attn directory not found $FLASH_ATTENTION_HOPPER_DIR"

TORCH_MIN_VERSION="2.9.0"
PYTHON="${PYTHON_EXECUTABLE:-python}"

```

- **EN:** Environment variables such as FA_FINAL_PACKAGE_DIR, MANYLINUX_PLAT, CUDA_VERSION, CUDA_SHORT, PYTHON_VERSION, FLASH_ATTENTION_HOPPER_DIR communicate required tool locations or behavioral switches.
- **CN:** FA_FINAL_PACKAGE_DIR、MANYLINUX_PLAT、CUDA_VERSION、CUDA_SHORT、PYTHON_VERSION、FLASH_ATTENTION_HOPPER_DIR 等环境变量用于说明所需工具位置或行为开关。

### Lines 21-36 / 第 21-36 行

```bash
# for ARM builds we need GLIBC 2.29+ so we use upstream linux image
# need to install dependencies
if [[ "$(uname -m)" == "aarch64" ]]; then
    if command -v dnf &> /dev/null; then
        dnf install -y \
            wget \
            perl \
            make \
            xz \
            bzip2 \
            gcc-toolset-13-gcc \
            gcc-toolset-13-gcc-c++
        export PATH=/opt/rh/gcc-toolset-13/root/usr/bin:$PATH
        export LD_LIBRARY_PATH=/opt/rh/gcc-toolset-13/root/usr/lib64:/opt/rh/gcc-toolset-13/root/usr/lib:${LD_LIBRARY_PATH:-}
    fi

```

- **EN:** This chunk introduces sections such as for ARM builds we need GLIBC 2.29+ so we use upstream linux image, need to install dependencies, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 for ARM builds we need GLIBC 2.29+ so we use upstream linux image、need to install dependencies 等标题组织周边说明或配置。
- **EN:** It invokes commands such as dnf, wget, perl, make, xz, bzip2, showing the operational steps the workflow performs.
- **CN:** 它调用了 dnf、wget、perl、make、xz、bzip2 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as ARM, GLIBC, PATH, LD_LIBRARY_PATH communicate required tool locations or behavioral switches.
- **CN:** ARM、GLIBC、PATH、LD_LIBRARY_PATH 等环境变量用于说明所需工具位置或行为开关。

### Lines 37-47 / 第 37-47 行

```bash
    source "${PYTORCH_ROOT}/.ci/docker/common/install_cuda.sh"
    [[ -z "$CUDA_INSTALLER_NAME" ]] && fatal "CUDA_INSTALLER_NAME must be set for aarch64 builds"
    install_cuda "$CUDA_VERSION" "$CUDA_INSTALLER_NAME"

    export CUDA_HOME=/usr/local/cuda
    export PATH=/usr/local/cuda/bin:$PATH

    echo "installed CUDA version:"
    nvcc --version
fi

```

- **EN:** It invokes commands such as source, install_cuda, nvcc, showing the operational steps the workflow performs.
- **CN:** 它调用了 source、install_cuda、nvcc 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as PYTORCH_ROOT, CUDA_INSTALLER_NAME, CUDA_VERSION, CUDA_HOME, PATH, CUDA communicate required tool locations or behavioral switches.
- **CN:** PYTORCH_ROOT、CUDA_INSTALLER_NAME、CUDA_VERSION、CUDA_HOME、PATH、CUDA 等环境变量用于说明所需工具位置或行为开关。

### Lines 48-63 / 第 48-63 行

```bash
echo "installing dependencies"
"$PYTHON" -m pip install einops packaging ninja numpy wheel setuptools

export PATH="$(dirname "$PYTHON"):$PATH"

export FLASH_ATTENTION_FORCE_BUILD="${FLASH_ATTENTION_FORCE_BUILD:-TRUE}"

export FLASH_ATTENTION_DISABLE_SPLIT="${FLASH_ATTENTION_DISABLE_SPLIT:-FALSE}"
export FLASH_ATTENTION_DISABLE_PAGEDKV="${FLASH_ATTENTION_DISABLE_PAGEDKV:-FALSE}"
export FLASH_ATTENTION_DISABLE_APPENDKV="${FLASH_ATTENTION_DISABLE_APPENDKV:-FALSE}"
export FLASH_ATTENTION_DISABLE_LOCAL="${FLASH_ATTENTION_DISABLE_LOCAL:-FALSE}"
export FLASH_ATTENTION_DISABLE_SOFTCAP="${FLASH_ATTENTION_DISABLE_SOFTCAP:-FALSE}"
export FLASH_ATTENTION_DISABLE_PACKGQA="${FLASH_ATTENTION_DISABLE_PACKGQA:-FALSE}"
export FLASH_ATTENTION_DISABLE_FP16="${FLASH_ATTENTION_DISABLE_FP16:-FALSE}"
export FLASH_ATTENTION_DISABLE_FP8="${FLASH_ATTENTION_DISABLE_FP8:-FALSE}"
export FLASH_ATTENTION_DISABLE_VARLEN="${FLASH_ATTENTION_DISABLE_VARLEN:-FALSE}"
```

- **EN:** Environment variables such as PYTHON, PATH, FLASH_ATTENTION_FORCE_BUILD, TRUE, FLASH_ATTENTION_DISABLE_SPLIT, FALSE communicate required tool locations or behavioral switches.
- **CN:** PYTHON、PATH、FLASH_ATTENTION_FORCE_BUILD、TRUE、FLASH_ATTENTION_DISABLE_SPLIT、FALSE 等环境变量用于说明所需工具位置或行为开关。

### Lines 64-74 / 第 64-74 行

```bash
export FLASH_ATTENTION_DISABLE_CLUSTER="${FLASH_ATTENTION_DISABLE_CLUSTER:-FALSE}"
export FLASH_ATTENTION_DISABLE_HDIM64="${FLASH_ATTENTION_DISABLE_HDIM64:-FALSE}"
export FLASH_ATTENTION_DISABLE_HDIM96="${FLASH_ATTENTION_DISABLE_HDIM96:-FALSE}"
export FLASH_ATTENTION_DISABLE_HDIM128="${FLASH_ATTENTION_DISABLE_HDIM128:-FALSE}"
export FLASH_ATTENTION_DISABLE_HDIM192="${FLASH_ATTENTION_DISABLE_HDIM192:-FALSE}"
export FLASH_ATTENTION_DISABLE_HDIM256="${FLASH_ATTENTION_DISABLE_HDIM256:-FALSE}"
export FLASH_ATTENTION_DISABLE_SM80="${FLASH_ATTENTION_DISABLE_SM80:-FALSE}"
export FLASH_ATTENTION_ENABLE_VCOLMAJOR="${FLASH_ATTENTION_ENABLE_VCOLMAJOR:-FALSE}"
export FLASH_ATTENTION_DISABLE_HDIMDIFF64="${FLASH_ATTENTION_DISABLE_HDIMDIFF64:-FALSE}"
export FLASH_ATTENTION_DISABLE_HDIMDIFF192="${FLASH_ATTENTION_DISABLE_HDIMDIFF192:-FALSE}"

```

- **EN:** Environment variables such as FLASH_ATTENTION_DISABLE_CLUSTER, FALSE, FLASH_ATTENTION_DISABLE_HDIM64, FLASH_ATTENTION_DISABLE_HDIM96, FLASH_ATTENTION_DISABLE_HDIM128, FLASH_ATTENTION_DISABLE_HDIM192 communicate required tool locations or behavioral switches.
- **CN:** FLASH_ATTENTION_DISABLE_CLUSTER、FALSE、FLASH_ATTENTION_DISABLE_HDIM64、FLASH_ATTENTION_DISABLE_HDIM96、FLASH_ATTENTION_DISABLE_HDIM128、FLASH_ATTENTION_DISABLE_HDIM192 等环境变量用于说明所需工具位置或行为开关。

### Lines 75-82 / 第 75-82 行

```bash
export NVCC_THREADS="${NVCC_THREADS:-8}"
export MAX_JOBS="${MAX_JOBS:-$(nproc)}"

echo "NVCC_THREADS=${NVCC_THREADS}"
echo "MAX_JOBS=${MAX_JOBS}"

pushd "$FLASH_ATTENTION_HOPPER_DIR"

```

- **EN:** It invokes commands such as pushd, showing the operational steps the workflow performs.
- **CN:** 它调用了 pushd 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as NVCC_THREADS, MAX_JOBS, FLASH_ATTENTION_HOPPER_DIR communicate required tool locations or behavioral switches.
- **CN:** NVCC_THREADS、MAX_JOBS、FLASH_ATTENTION_HOPPER_DIR 等环境变量用于说明所需工具位置或行为开关。

### Lines 83-93 / 第 83-93 行

```bash
git config --global --add safe.directory '*'
git submodule update --init ../csrc/cutlass

if [[ "${CUDA_VERSION}" == 13.* ]]; then
    CCCL_INCLUDE="/usr/local/cuda/include/cccl"
    [[ ! -d "${CCCL_INCLUDE}" ]] && fatal "CCCL include directory not found at ${CCCL_INCLUDE}"
    echo "Adding CCCL include path: ${CCCL_INCLUDE}"
    export CPLUS_INCLUDE_PATH="${CCCL_INCLUDE}${CPLUS_INCLUDE_PATH:+:$CPLUS_INCLUDE_PATH}"
    export C_INCLUDE_PATH="${CCCL_INCLUDE}${C_INCLUDE_PATH:+:$C_INCLUDE_PATH}"
fi

```

- **EN:** It invokes commands such as git, showing the operational steps the workflow performs.
- **CN:** 它调用了 git 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CUDA_VERSION, CCCL_INCLUDE, CCCL, CPLUS_INCLUDE_PATH, C_INCLUDE_PATH communicate required tool locations or behavioral switches.
- **CN:** CUDA_VERSION、CCCL_INCLUDE、CCCL、CPLUS_INCLUDE_PATH、C_INCLUDE_PATH 等环境变量用于说明所需工具位置或行为开关。

### Lines 94-103 / 第 94-103 行

```bash
if [[ "${FA_TEST_BUILD}" == "true" ]]; then
    BUILD_DATE=$(date +%Y%m%d)
    export FLASH_ATTN_LOCAL_VERSION="${BUILD_DATE}.cu${CUDA_SHORT}"
fi

# stable ABI wheel requires torch>=2.9.0
# since Python 3.9 support was dropped in torch 2.9.0, we need to use Python 3.10+
sed -i "s/python_requires=\">=3.8\"/python_requires=\">=${PYTHON_VERSION}\"/" setup.py
sed -i "s/\"torch\",/\"torch>=${TORCH_MIN_VERSION}\",/" setup.py

```

- **EN:** This chunk introduces sections such as stable ABI wheel requires torch>=2.9.0, since Python 3.9 support was dropped in torch 2.9.0, we need to use Python 3.10+, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 stable ABI wheel requires torch>=2.9.0、since Python 3.9 support was dropped in torch 2.9.0, we need to use Python 3.10+ 等标题组织周边说明或配置。
- **EN:** It invokes commands such as sed, showing the operational steps the workflow performs.
- **CN:** 它调用了 sed 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as FA_TEST_BUILD, BUILD_DATE, FLASH_ATTN_LOCAL_VERSION, CUDA_SHORT, ABI, PYTHON_VERSION communicate required tool locations or behavioral switches.
- **CN:** FA_TEST_BUILD、BUILD_DATE、FLASH_ATTN_LOCAL_VERSION、CUDA_SHORT、ABI、PYTHON_VERSION 等环境变量用于说明所需工具位置或行为开关。

### Lines 104-111 / 第 104-111 行

```bash
"$PYTHON" setup.py bdist_wheel \
    -d "$FA_FINAL_PACKAGE_DIR" \
    -k \
    --plat-name "${MANYLINUX_PLAT}"

echo "wheel built: "
find "$FA_FINAL_PACKAGE_DIR" -name '*.whl' -exec ls -la {} \;

```

- **EN:** It invokes commands such as -d, -k, --plat-name, find, showing the operational steps the workflow performs.
- **CN:** 它调用了 -d、-k、--plat-name、find 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as PYTHON, FA_FINAL_PACKAGE_DIR, MANYLINUX_PLAT communicate required tool locations or behavioral switches.
- **CN:** PYTHON、FA_FINAL_PACKAGE_DIR、MANYLINUX_PLAT 等环境变量用于说明所需工具位置或行为开关。

### Lines 112-112 / 第 112-112 行

```bash
popd
```

- **EN:** It invokes commands such as popd, showing the operational steps the workflow performs.
- **CN:** 它调用了 popd 等命令，展示该工作流执行的操作步骤。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **Repository bootstrap** — 与仓库状态、子模块或源码检出流程交互。

## Dependencies / 依赖关系

- `"${PYTORCH_ROOT}/.ci/pytorch/common_utils.sh"`
- `"${PYTORCH_ROOT}/.ci/docker/common/install_cuda.sh"`
- `bash`
- `python`
- `wget`
- `perl`
- `make`
- `ninja`
- `git`
