# build_xpu.sh — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `.ci/manywheel/build_xpu.sh`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Automates repository maintenance, build, test, packaging, or release tasks through shell commands. The opening comment frames the file as: "!/usr/bin/env bash."
- **Purpose (CN)**: 通过 shell 命令自动化执行仓库维护、构建、测试、打包或发布任务。 开头注释将该文件概括为：“!/usr/bin/env bash”。

## Content Analysis / 内容分析

### Lines 1-8 / 第 1-8 行

```bash
#!/usr/bin/env bash

set -ex

export TH_BINARY_BUILD=1
export USE_CUDA=0

# Keep an array of cmake variables to add to
```

- **EN:** This chunk introduces sections such as !/usr/bin/env bash, Keep an array of cmake variables to add to, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 !/usr/bin/env bash、Keep an array of cmake variables to add to 等标题组织周边说明或配置。
- **EN:** Environment variables such as TH_BINARY_BUILD, USE_CUDA communicate required tool locations or behavioral switches.
- **CN:** TH_BINARY_BUILD、USE_CUDA 等环境变量用于说明所需工具位置或行为开关。

### Lines 9-17 / 第 9-17 行

```bash
if [[ -z "$CMAKE_ARGS" ]]; then
    # These are passed to tools/build_pytorch_libs.sh::build()
    CMAKE_ARGS=()
fi
if [[ -z "$EXTRA_CAFFE2_CMAKE_FLAGS" ]]; then
    # These are passed to tools/build_pytorch_libs.sh::build_caffe2()
    EXTRA_CAFFE2_CMAKE_FLAGS=()
fi

```

- **EN:** This chunk introduces sections such as These are passed to tools/build_pytorch_libs.sh::build(), These are passed to tools/build_pytorch_libs.sh::build_caffe2(), which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 These are passed to tools/build_pytorch_libs.sh::build()、These are passed to tools/build_pytorch_libs.sh::build_caffe2() 等标题组织周边说明或配置。
- **EN:** Environment variables such as CMAKE_ARGS, EXTRA_CAFFE2_CMAKE_FLAGS communicate required tool locations or behavioral switches.
- **CN:** CMAKE_ARGS、EXTRA_CAFFE2_CMAKE_FLAGS 等环境变量用于说明所需工具位置或行为开关。

### Lines 18-29 / 第 18-29 行

```bash

# Refer https://www.intel.com/content/www/us/en/developer/articles/tool/pytorch-prerequisites-for-intel-gpus.html
source /opt/intel/oneapi/compiler/latest/env/vars.sh
source /opt/intel/oneapi/pti/latest/env/vars.sh
source /opt/intel/oneapi/umf/latest/env/vars.sh
source /opt/intel/oneapi/ccl/latest/env/vars.sh
source /opt/intel/oneapi/mpi/latest/env/vars.sh
export USE_STATIC_MKL=1
export USE_ONEMKL=1
export USE_XCCL=1
export USE_MPI=0

```

- **EN:** This chunk introduces sections such as Refer https://www.intel.com/content/www/us/en/developer/articles/tool/pytorch-prerequisites-for-intel-gpus.html, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Refer https://www.intel.com/content/www/us/en/developer/articles/tool/pytorch-prerequisites-for-intel-gpus.html 等标题组织周边说明或配置。
- **EN:** It invokes commands such as source, showing the operational steps the workflow performs.
- **CN:** 它调用了 source 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as USE_STATIC_MKL, USE_ONEMKL, USE_XCCL, USE_MPI communicate required tool locations or behavioral switches.
- **CN:** USE_STATIC_MKL、USE_ONEMKL、USE_XCCL、USE_MPI 等环境变量用于说明所需工具位置或行为开关。

### Lines 30-40 / 第 30-40 行

```bash
WHEELHOUSE_DIR="wheelhousexpu"
LIBTORCH_HOUSE_DIR="libtorch_housexpu"
if [[ -z "$PYTORCH_FINAL_PACKAGE_DIR" ]]; then
    if [[ -z "$BUILD_PYTHONLESS" ]]; then
        PYTORCH_FINAL_PACKAGE_DIR="/remote/wheelhousexpu"
    else
        PYTORCH_FINAL_PACKAGE_DIR="/remote/libtorch_housexpu"
    fi
fi
mkdir -p "$PYTORCH_FINAL_PACKAGE_DIR" || true

```

- **EN:** It invokes commands such as mkdir, showing the operational steps the workflow performs.
- **CN:** 它调用了 mkdir 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as WHEELHOUSE_DIR, LIBTORCH_HOUSE_DIR, PYTORCH_FINAL_PACKAGE_DIR, BUILD_PYTHONLESS communicate required tool locations or behavioral switches.
- **CN:** WHEELHOUSE_DIR、LIBTORCH_HOUSE_DIR、PYTORCH_FINAL_PACKAGE_DIR、BUILD_PYTHONLESS 等环境变量用于说明所需工具位置或行为开关。

### Lines 41-55 / 第 41-55 行

```bash
OS_NAME=$(awk -F= '/^NAME/{print $2}' /etc/os-release)
if [[ "$OS_NAME" == *"CentOS Linux"* ]]; then
    LIBGOMP_PATH="/usr/lib64/libgomp.so.1"
elif [[ "$OS_NAME" == *"Red Hat Enterprise Linux"* ]]; then
    LIBGOMP_PATH="/usr/lib64/libgomp.so.1"
elif [[ "$OS_NAME" == *"AlmaLinux"* ]]; then
    LIBGOMP_PATH="/usr/lib64/libgomp.so.1"
elif [[ "$OS_NAME" == *"Ubuntu"* ]]; then
    if [[ "$(uname -m)" == "s390x" ]]; then
        LIBGOMP_PATH="/usr/lib/s390x-linux-gnu/libgomp.so.1"
    else
        LIBGOMP_PATH="/usr/lib/x86_64-linux-gnu/libgomp.so.1"
    fi
fi

```

- **EN:** Environment variables such as OS_NAME, NAME, LIBGOMP_PATH communicate required tool locations or behavioral switches.
- **CN:** OS_NAME、NAME、LIBGOMP_PATH 等环境变量用于说明所需工具位置或行为开关。

### Lines 56-65 / 第 56-65 行

```bash
DEPS_LIST=(
    "$LIBGOMP_PATH"
    "/opt/intel/oneapi/compiler/latest/lib/libOpenCL.so.1"
)

DEPS_SONAME=(
    "libgomp.so.1"
    "libOpenCL.so.1"
)

```

- **EN:** Environment variables such as DEPS_LIST, LIBGOMP_PATH, DEPS_SONAME communicate required tool locations or behavioral switches.
- **CN:** DEPS_LIST、LIBGOMP_PATH、DEPS_SONAME 等环境变量用于说明所需工具位置或行为开关。

### Lines 66-81 / 第 66-81 行

```bash
if [[ -z "$PYTORCH_EXTRA_INSTALL_REQUIREMENTS" ]]; then
    echo "Bundling with xpu support package libs."
    DEPS_LIST+=(
        "/opt/intel/oneapi/compiler/latest/lib/libsycl.so.8"
        "/opt/intel/oneapi/compiler/latest/lib/libur_loader.so.0"
        "/opt/intel/oneapi/compiler/latest/lib/libur_adapter_level_zero.so.0"
        "/opt/intel/oneapi/compiler/latest/lib/libur_adapter_opencl.so.0"
        "/opt/intel/oneapi/compiler/latest/lib/libsvml.so"
        "/opt/intel/oneapi/compiler/latest/lib/libirng.so"
        "/opt/intel/oneapi/compiler/latest/lib/libimf.so"
        "/opt/intel/oneapi/compiler/latest/lib/libintlc.so.5"
        "/opt/intel/oneapi/pti/latest/lib/libpti_view.so.0.10"
        "/opt/intel/oneapi/umf/latest/lib/libumf.so.0"
        "/opt/intel/oneapi/tcm/latest/lib/libhwloc.so.15"
    )
    DEPS_SONAME+=(
```

- **EN:** It invokes commands such as DEPS_LIST+, DEPS_SONAME+, showing the operational steps the workflow performs.
- **CN:** 它调用了 DEPS_LIST+、DEPS_SONAME+ 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as PYTORCH_EXTRA_INSTALL_REQUIREMENTS, DEPS_LIST, DEPS_SONAME communicate required tool locations or behavioral switches.
- **CN:** PYTORCH_EXTRA_INSTALL_REQUIREMENTS、DEPS_LIST、DEPS_SONAME 等环境变量用于说明所需工具位置或行为开关。

### Lines 82-97 / 第 82-97 行

```bash
        "libsycl.so.8"
        "libur_loader.so.0"
        "libur_adapter_level_zero.so.0"
        "libur_adapter_opencl.so.0"
        "libsvml.so"
        "libirng.so"
        "libimf.so"
        "libintlc.so.5"
        "libpti_view.so.0.10"
        "libumf.so.0"
        "libhwloc.so.15"
    )
else
    echo "Using xpu runtime libs from pypi."
    XPU_RPATHS=(
        '$ORIGIN/../../../..'
```

- **EN:** Environment variables such as XPU_RPATHS, ORIGIN communicate required tool locations or behavioral switches.
- **CN:** XPU_RPATHS、ORIGIN 等环境变量用于说明所需工具位置或行为开关。

### Lines 98-106 / 第 98-106 行

```bash
    )
    XPU_RPATHS=$(IFS=: ; echo "${XPU_RPATHS[*]}")
    export C_SO_RPATH=$XPU_RPATHS':$ORIGIN:$ORIGIN/lib'
    export LIB_SO_RPATH=$XPU_RPATHS':$ORIGIN'
    export FORCE_RPATH="--force-rpath"
fi

rm -rf /usr/local/cuda*

```

- **EN:** It invokes commands such as rm, showing the operational steps the workflow performs.
- **CN:** 它调用了 rm 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as XPU_RPATHS, IFS, C_SO_RPATH, ORIGIN, LIB_SO_RPATH, FORCE_RPATH communicate required tool locations or behavioral switches.
- **CN:** XPU_RPATHS、IFS、C_SO_RPATH、ORIGIN、LIB_SO_RPATH、FORCE_RPATH 等环境变量用于说明所需工具位置或行为开关。

### Lines 107-113 / 第 107-113 行

```bash
SOURCE_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null && pwd )"
if [[ -z "$BUILD_PYTHONLESS" ]]; then
    BUILD_SCRIPT=build_common.sh
else
    BUILD_SCRIPT=build_libtorch.sh
fi
source ${SOURCE_DIR}/${BUILD_SCRIPT}
```

- **EN:** It invokes commands such as source, showing the operational steps the workflow performs.
- **CN:** 它调用了 source 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as SOURCE_DIR, BASH_SOURCE, BUILD_PYTHONLESS, BUILD_SCRIPT communicate required tool locations or behavioral switches.
- **CN:** SOURCE_DIR、BASH_SOURCE、BUILD_PYTHONLESS、BUILD_SCRIPT 等环境变量用于说明所需工具位置或行为开关。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **CMake build logic** — 控制构建系统配置、特性检查或依赖发现。

## Dependencies / 依赖关系

- `/opt/intel/oneapi/compiler/latest/env/vars.sh`
- `/opt/intel/oneapi/pti/latest/env/vars.sh`
- `/opt/intel/oneapi/umf/latest/env/vars.sh`
- `/opt/intel/oneapi/ccl/latest/env/vars.sh`
- `/opt/intel/oneapi/mpi/latest/env/vars.sh`
- `${SOURCE_DIR}/${BUILD_SCRIPT}`
- `bash`
- `cmake`
