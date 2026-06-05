# build.sh — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `.ci/pytorch/build.sh`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Automates repository maintenance, build, test, packaging, or release tasks through shell commands. Performance measurement or benchmark orchestration is part of its intent. The opening comment frames the file as: "!/bin/bash."
- **Purpose (CN)**: 通过 shell 命令自动化执行仓库维护、构建、测试、打包或发布任务。 性能测量或基准编排是其意图的一部分。 开头注释将该文件概括为：“!/bin/bash”。

## Content Analysis / 内容分析

### Lines 1-13 / 第 1-13 行

```bash
#!/bin/bash

set -ex -o pipefail

# Required environment variable: $BUILD_ENVIRONMENT
# (This is set by default in the Docker images we build, so you don't
# need to set it yourself.

# shellcheck source=./common.sh
source "$(dirname "${BASH_SOURCE[0]}")/common.sh"
# shellcheck source=./common-build.sh
source "$(dirname "${BASH_SOURCE[0]}")/common-build.sh"

```

- **EN:** This chunk introduces sections such as !/bin/bash, Required environment variable: $BUILD_ENVIRONMENT, (This is set by default in the Docker images we build, so you don't, need to set it yourself., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 !/bin/bash、Required environment variable: $BUILD_ENVIRONMENT、(This is set by default in the Docker images we build, so you don't、need to set it yourself. 等标题组织周边说明或配置。
- **EN:** It invokes commands such as source, showing the operational steps the workflow performs.
- **CN:** 它调用了 source 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ENVIRONMENT, BASH_SOURCE communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT、BASH_SOURCE 等环境变量用于说明所需工具位置或行为开关。

### Lines 14-25 / 第 14-25 行

```bash
echo "Python version:"
python --version

echo "GCC version:"
gcc --version

echo "CMake version:"
cmake --version

echo "Environment variables:"
env

```

- **EN:** It invokes commands such as python, gcc, cmake, env, showing the operational steps the workflow performs.
- **CN:** 它调用了 python、gcc、cmake、env 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as GCC communicate required tool locations or behavioral switches.
- **CN:** GCC 等环境变量用于说明所需工具位置或行为开关。

### Lines 26-37 / 第 26-37 行

```bash
if [[ "$BUILD_ENVIRONMENT" == *cuda* ]]; then
  # Use jemalloc during compilation to mitigate https://github.com/pytorch/pytorch/issues/116289
  export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so.2
  echo "NVCC version:"
  nvcc --version
fi

if [[ "$BUILD_ENVIRONMENT" == *cuda13* ]]; then
  # Disable FBGEMM for CUDA 13 builds
  export USE_FBGEMM=0
fi

```

- **EN:** This chunk introduces sections such as Use jemalloc during compilation to mitigate https://github.com/pytorch/pytorch/issues/116289, Disable FBGEMM for CUDA 13 builds, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Use jemalloc during compilation to mitigate https://github.com/pytorch/pytorch/issues/116289、Disable FBGEMM for CUDA 13 builds 等标题组织周边说明或配置。
- **EN:** It invokes commands such as nvcc, showing the operational steps the workflow performs.
- **CN:** 它调用了 nvcc 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ENVIRONMENT, LD_PRELOAD, NVCC, FBGEMM, CUDA, USE_FBGEMM communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT、LD_PRELOAD、NVCC、FBGEMM、CUDA、USE_FBGEMM 等环境变量用于说明所需工具位置或行为开关。

### Lines 38-53 / 第 38-53 行

```bash
if [[ ${BUILD_ENVIRONMENT} == *"parallelnative"* ]]; then
  export ATEN_THREADING=NATIVE
fi


if ! which conda; then
  # In ROCm CIs, we are doing cross compilation on build machines with
  # intel cpu and later run tests on machines with amd cpu.
  # Also leave out two builds to make sure non-mkldnn builds still work.
  if [[ "$BUILD_ENVIRONMENT" != *rocm* ]]; then
    export USE_MKLDNN=1
  else
    export USE_MKLDNN=0
  fi
else
  # CMAKE_PREFIX_PATH precedences
```

- **EN:** This chunk introduces sections such as In ROCm CIs, we are doing cross compilation on build machines with, intel cpu and later run tests on machines with amd cpu., Also leave out two builds to make sure non-mkldnn builds still work., CMAKE_PREFIX_PATH precedences, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 In ROCm CIs, we are doing cross compilation on build machines with、intel cpu and later run tests on machines with amd cpu.、Also leave out two builds to make sure non-mkldnn builds still work.、CMAKE_PREFIX_PATH precedences 等标题组织周边说明或配置。
- **EN:** Environment variables such as BUILD_ENVIRONMENT, ATEN_THREADING, NATIVE, USE_MKLDNN, CMAKE_PREFIX_PATH communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT、ATEN_THREADING、NATIVE、USE_MKLDNN、CMAKE_PREFIX_PATH 等环境变量用于说明所需工具位置或行为开关。

### Lines 54-65 / 第 54-65 行

```bash
  # 1. $CONDA_PREFIX, if defined. This follows the pytorch official build instructions.
  # 2. /opt/conda/envs/py_${ANACONDA_PYTHON_VERSION}, if ANACONDA_PYTHON_VERSION defined.
  #    This is for CI, which defines ANACONDA_PYTHON_VERSION but not CONDA_PREFIX.
  # 3. $(conda info --base). The fallback value of pytorch official build
  #    instructions actually refers to this.
  #    Commonly this is /opt/conda/
  if [[ -v CONDA_PREFIX ]]; then
    export CMAKE_PREFIX_PATH=${CONDA_PREFIX}
  elif [[ -v ANACONDA_PYTHON_VERSION ]]; then
    export CMAKE_PREFIX_PATH="/opt/conda/envs/py_${ANACONDA_PYTHON_VERSION}"
  else
    # already checked by `! which conda`
```

- **EN:** This chunk introduces sections such as 1. $CONDA_PREFIX, if defined. This follows the pytorch official build instructions., 2. /opt/conda/envs/py_${ANACONDA_PYTHON_VERSION}, if ANACONDA_PYTHON_VERSION defined., This is for CI, which defines ANACONDA_PYTHON_VERSION but not CONDA_PREFIX., 3. $(conda info --base). The fallback value of pytorch official build, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 1. $CONDA_PREFIX, if defined. This follows the pytorch official build instructions.、2. /opt/conda/envs/py_${ANACONDA_PYTHON_VERSION}, if ANACONDA_PYTHON_VERSION defined.、This is for CI, which defines ANACONDA_PYTHON_VERSION but not CONDA_PREFIX.、3. $(conda info --base). The fallback value of pytorch official build 等标题组织周边说明或配置。
- **EN:** Environment variables such as CONDA_PREFIX, ANACONDA_PYTHON_VERSION, CMAKE_PREFIX_PATH communicate required tool locations or behavioral switches.
- **CN:** CONDA_PREFIX、ANACONDA_PYTHON_VERSION、CMAKE_PREFIX_PATH 等环境变量用于说明所需工具位置或行为开关。

### Lines 66-77 / 第 66-77 行

```bash
    CMAKE_PREFIX_PATH="$(conda info --base)"
    export CMAKE_PREFIX_PATH
  fi

  # Workaround required for MKL library linkage
  # https://github.com/pytorch/pytorch/issues/119557
  if [[ "$ANACONDA_PYTHON_VERSION" = "3.12" || "$ANACONDA_PYTHON_VERSION" = "3.13" ]]; then
    export CMAKE_LIBRARY_PATH="/opt/conda/envs/py_$ANACONDA_PYTHON_VERSION/lib/"
    export CMAKE_INCLUDE_PATH="/opt/conda/envs/py_$ANACONDA_PYTHON_VERSION/include/"
  fi
fi

```

- **EN:** This chunk introduces sections such as Workaround required for MKL library linkage, https://github.com/pytorch/pytorch/issues/119557, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Workaround required for MKL library linkage、https://github.com/pytorch/pytorch/issues/119557 等标题组织周边说明或配置。
- **EN:** Environment variables such as CMAKE_PREFIX_PATH, MKL, ANACONDA_PYTHON_VERSION, CMAKE_LIBRARY_PATH, CMAKE_INCLUDE_PATH communicate required tool locations or behavioral switches.
- **CN:** CMAKE_PREFIX_PATH、MKL、ANACONDA_PYTHON_VERSION、CMAKE_LIBRARY_PATH、CMAKE_INCLUDE_PATH 等环境变量用于说明所需工具位置或行为开关。

### Lines 78-92 / 第 78-92 行

```bash
if [[ "$BUILD_ENVIRONMENT" == *aarch64* ]]; then
  export USE_MKLDNN=1
  export USE_MKLDNN_ACL=1
  export ACL_ROOT_DIR=/acl
fi

if [[ "$BUILD_ENVIRONMENT" == *riscv64* ]]; then
  if [[ -f /opt/riscv-cross-env/bin/activate ]]; then
    # shellcheck disable=SC1091
    source /opt/riscv-cross-env/bin/activate
  else
    echo "Activation file not found"
    exit 1
  fi

```

- **EN:** This chunk introduces sections such as shellcheck disable=SC1091, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 shellcheck disable=SC1091 等标题组织周边说明或配置。
- **EN:** It invokes commands such as source, exit, showing the operational steps the workflow performs.
- **CN:** 它调用了 source、exit 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ENVIRONMENT, USE_MKLDNN, USE_MKLDNN_ACL, ACL_ROOT_DIR, SC1091 communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT、USE_MKLDNN、USE_MKLDNN_ACL、ACL_ROOT_DIR、SC1091 等环境变量用于说明所需工具位置或行为开关。

### Lines 93-104 / 第 93-104 行

```bash
  export CMAKE_CROSSCOMPILING=TRUE
  export CMAKE_SYSTEM_NAME=Linux
  export CMAKE_SYSTEM_PROCESSOR=riscv64

  export USE_CUDA=0
  export USE_MKLDNN=0

  export SLEEF_TARGET_EXEC_USE_QEMU=ON
  sudo chown -R jenkins /var/lib/jenkins/workspace /opt

fi

```

- **EN:** It invokes commands such as chown, showing the operational steps the workflow performs.
- **CN:** 它调用了 chown 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CMAKE_CROSSCOMPILING, TRUE, CMAKE_SYSTEM_NAME, CMAKE_SYSTEM_PROCESSOR, USE_CUDA, USE_MKLDNN communicate required tool locations or behavioral switches.
- **CN:** CMAKE_CROSSCOMPILING、TRUE、CMAKE_SYSTEM_NAME、CMAKE_SYSTEM_PROCESSOR、USE_CUDA、USE_MKLDNN 等环境变量用于说明所需工具位置或行为开关。

### Lines 105-116 / 第 105-116 行

```bash
# Use special scripts for Android builds

if [[ "$BUILD_ENVIRONMENT" == *vulkan* ]]; then
  export USE_VULKAN=1
  # shellcheck disable=SC1091
  source /var/lib/jenkins/vulkansdk/setup-env.sh
fi

if [[ "$BUILD_ENVIRONMENT" == *rocm* ]]; then
  # hcc used to run out of memory, silently exiting without stopping
  # the build process, leaving undefined symbols in the shared lib,
  # causing undefined symbol errors when later running tests.
```

- **EN:** This chunk introduces sections such as Use special scripts for Android builds, shellcheck disable=SC1091, hcc used to run out of memory, silently exiting without stopping, the build process, leaving undefined symbols in the shared lib,, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Use special scripts for Android builds、shellcheck disable=SC1091、hcc used to run out of memory, silently exiting without stopping、the build process, leaving undefined symbols in the shared lib, 等标题组织周边说明或配置。
- **EN:** It invokes commands such as source, showing the operational steps the workflow performs.
- **CN:** 它调用了 source 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ENVIRONMENT, USE_VULKAN, SC1091 communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT、USE_VULKAN、SC1091 等环境变量用于说明所需工具位置或行为开关。

### Lines 117-128 / 第 117-128 行

```bash
  # We used to set MAX_JOBS to 4 to avoid, but this is no longer an issue.
  if [ -z "$MAX_JOBS" ]; then
    export MAX_JOBS=$(($(nproc) - 1))
  fi

  if [[ -n "$CI" && -z "$PYTORCH_ROCM_ARCH" ]]; then
      # Set ROCM_ARCH to gfx906 for CI builds, if user doesn't override.
      echo "Limiting PYTORCH_ROCM_ARCH to gfx906 for CI builds"
      export PYTORCH_ROCM_ARCH="gfx906"
  fi

  # hipify sources
```

- **EN:** This chunk introduces sections such as We used to set MAX_JOBS to 4 to avoid, but this is no longer an issue., Set ROCM_ARCH to gfx906 for CI builds, if user doesn't override., hipify sources, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 We used to set MAX_JOBS to 4 to avoid, but this is no longer an issue.、Set ROCM_ARCH to gfx906 for CI builds, if user doesn't override.、hipify sources 等标题组织周边说明或配置。
- **EN:** Environment variables such as MAX_JOBS, PYTORCH_ROCM_ARCH, ROCM_ARCH communicate required tool locations or behavioral switches.
- **CN:** MAX_JOBS、PYTORCH_ROCM_ARCH、ROCM_ARCH 等环境变量用于说明所需工具位置或行为开关。

### Lines 129-141 / 第 129-141 行

```bash
  python tools/amd_build/build_amd.py
fi

if [[ "$BUILD_ENVIRONMENT" == *xpu* ]]; then
  # shellcheck disable=SC1091
  source /opt/intel/oneapi/compiler/latest/env/vars.sh
  # shellcheck disable=SC1091
  source /opt/intel/oneapi/umf/latest/env/vars.sh
  # shellcheck disable=SC1091
  source /opt/intel/oneapi/ccl/latest/env/vars.sh
  # shellcheck disable=SC1091
  source /opt/intel/oneapi/mpi/latest/env/vars.sh
  # shellcheck disable=SC1091
```

- **EN:** This chunk introduces sections such as shellcheck disable=SC1091, shellcheck disable=SC1091, shellcheck disable=SC1091, shellcheck disable=SC1091, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 shellcheck disable=SC1091、shellcheck disable=SC1091、shellcheck disable=SC1091、shellcheck disable=SC1091 等标题组织周边说明或配置。
- **EN:** It invokes commands such as python, source, showing the operational steps the workflow performs.
- **CN:** 它调用了 python、source 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ENVIRONMENT, SC1091 communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT、SC1091 等环境变量用于说明所需工具位置或行为开关。

### Lines 142-157 / 第 142-157 行

```bash
  source /opt/intel/oneapi/pti/latest/env/vars.sh
  # Enable XCCL build
  export USE_XCCL=1
  export USE_MPI=0
  export TORCH_XPU_ARCH_LIST=pvc
  export USE_STATIC_MKL=1
fi

# sccache will fail for CUDA builds if all cores are used for compiling
# gcc 7 with sccache seems to have intermittent OOM issue if all cores are used
if [ -z "$MAX_JOBS" ]; then
  if { [[ "$BUILD_ENVIRONMENT" == *cuda* ]]; } && which sccache > /dev/null; then
    export MAX_JOBS=$(($(nproc) - 1))
  fi
fi

```

- **EN:** This chunk introduces sections such as Enable XCCL build, sccache will fail for CUDA builds if all cores are used for compiling, gcc 7 with sccache seems to have intermittent OOM issue if all cores are used, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Enable XCCL build、sccache will fail for CUDA builds if all cores are used for compiling、gcc 7 with sccache seems to have intermittent OOM issue if all cores are used 等标题组织周边说明或配置。
- **EN:** It invokes commands such as source, showing the operational steps the workflow performs.
- **CN:** 它调用了 source 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as XCCL, USE_XCCL, USE_MPI, TORCH_XPU_ARCH_LIST, USE_STATIC_MKL, CUDA communicate required tool locations or behavioral switches.
- **CN:** XCCL、USE_XCCL、USE_MPI、TORCH_XPU_ARCH_LIST、USE_STATIC_MKL、CUDA 等环境变量用于说明所需工具位置或行为开关。

### Lines 158-177 / 第 158-177 行

```bash
# TORCH_CUDA_ARCH_LIST must be passed from an environment variable
if [[ "$BUILD_ENVIRONMENT" == *cuda* && -z "$TORCH_CUDA_ARCH_LIST" ]]; then
  echo "TORCH_CUDA_ARCH_LIST must be defined"
  exit 1
fi

# We only build FlashAttention files for CUDA 8.0+, and they require large amounts of
# memory to build and will OOM

if [[ "$BUILD_ENVIRONMENT" == *cuda* ]] && echo "${TORCH_CUDA_ARCH_LIST}" | tr ' ' '\n' | sed 's/$/>= 8.0/' | bc | grep -q 1; then
  J=2  # default to 2 jobs
  case "$RUNNER" in
    linux.12xlarge.memory|linux.24xlarge.memory)
      J=24
      ;;
  esac
  echo "Building FlashAttention with job limit $J"
  export BUILD_CUSTOM_STEP="ninja -C build flash_attention -j ${J}"
fi

```

- **EN:** This chunk introduces sections such as TORCH_CUDA_ARCH_LIST must be passed from an environment variable, We only build FlashAttention files for CUDA 8.0+, and they require large amounts of, memory to build and will OOM, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 TORCH_CUDA_ARCH_LIST must be passed from an environment variable、We only build FlashAttention files for CUDA 8.0+, and they require large amounts of、memory to build and will OOM 等标题组织周边说明或配置。
- **EN:** It invokes commands such as exit, linux.12xlarge.memory, showing the operational steps the workflow performs.
- **CN:** 它调用了 exit、linux.12xlarge.memory 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as TORCH_CUDA_ARCH_LIST, BUILD_ENVIRONMENT, CUDA, OOM, RUNNER, BUILD_CUSTOM_STEP communicate required tool locations or behavioral switches.
- **CN:** TORCH_CUDA_ARCH_LIST、BUILD_ENVIRONMENT、CUDA、OOM、RUNNER、BUILD_CUSTOM_STEP 等环境变量用于说明所需工具位置或行为开关。

### Lines 178-191 / 第 178-191 行

```bash
# TODO: Removeme once all the wrappers are gone
if [[ "$BUILD_ENVIRONMENT" == *clang* ]] && [[ "$BUILD_ENVIRONMENT" == *cuda* ]]; then
  sudo rm -f /opt/cache/bin/clang++
fi

if [[ "$BUILD_ENVIRONMENT" == *-clang*-asan* ]]; then
  if [[ "$BUILD_ENVIRONMENT" == *cuda* ]]; then
    export USE_CUDA=1
  fi
  export USE_ASAN=1
  export REL_WITH_DEB_INFO=1
  export UBSAN_FLAGS="-fno-sanitize-recover=all"
fi

```

- **EN:** This chunk introduces sections such as TODO: Removeme once all the wrappers are gone, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 TODO: Removeme once all the wrappers are gone 等标题组织周边说明或配置。
- **EN:** It invokes commands such as rm, showing the operational steps the workflow performs.
- **CN:** 它调用了 rm 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as TODO, BUILD_ENVIRONMENT, USE_CUDA, USE_ASAN, REL_WITH_DEB_INFO, UBSAN_FLAGS communicate required tool locations or behavioral switches.
- **CN:** TODO、BUILD_ENVIRONMENT、USE_CUDA、USE_ASAN、REL_WITH_DEB_INFO、UBSAN_FLAGS 等环境变量用于说明所需工具位置或行为开关。

### Lines 192-205 / 第 192-205 行

```bash
if [[ "${BUILD_ENVIRONMENT}" == *no-ops* ]]; then
  export USE_PER_OPERATOR_HEADERS=0
fi

if [[ "${BUILD_ENVIRONMENT}" != *cuda* ]]; then
  export BUILD_STATIC_RUNTIME_BENCHMARK=ON
fi

if [[ "$BUILD_ENVIRONMENT" == *-full-debug* ]]; then
  export CMAKE_BUILD_TYPE=Debug
elif [[ "$BUILD_ENVIRONMENT" == *-debug* ]]; then
  export CMAKE_BUILD_TYPE=RelWithAssert
fi

```

- **EN:** Environment variables such as BUILD_ENVIRONMENT, USE_PER_OPERATOR_HEADERS, BUILD_STATIC_RUNTIME_BENCHMARK, CMAKE_BUILD_TYPE communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT、USE_PER_OPERATOR_HEADERS、BUILD_STATIC_RUNTIME_BENCHMARK、CMAKE_BUILD_TYPE 等环境变量用于说明所需工具位置或行为开关。

### Lines 206-217 / 第 206-217 行

```bash
# Do not change workspace permissions for ROCm and s390x CI jobs
# as it can leave workspace with bad permissions for cancelled jobs
if [[ "$BUILD_ENVIRONMENT" != *rocm* && "$BUILD_ENVIRONMENT" != *s390x* && "$BUILD_ENVIRONMENT" != *riscv64* && -d /var/lib/jenkins/workspace ]]; then
  # Workaround for dind-rootless userid mapping (https://github.com/pytorch/ci-infra/issues/96)
  WORKSPACE_ORIGINAL_OWNER_ID=$(stat -c '%u' "/var/lib/jenkins/workspace")
  cleanup_workspace() {
    echo "sudo may print the following warning message that can be ignored. The chown command will still run."
    echo "    sudo: setrlimit(RLIMIT_STACK): Operation not permitted"
    echo "For more details refer to https://github.com/sudo-project/sudo/issues/42"
    sudo chown -R "$WORKSPACE_ORIGINAL_OWNER_ID" /var/lib/jenkins/workspace
  }
  # Disable shellcheck SC2064 as we want to parse the original owner immediately.
```

- **EN:** This chunk introduces sections such as Do not change workspace permissions for ROCm and s390x CI jobs, as it can leave workspace with bad permissions for cancelled jobs, Workaround for dind-rootless userid mapping (https://github.com/pytorch/ci-infra/issues/96), Disable shellcheck SC2064 as we want to parse the original owner immediately., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Do not change workspace permissions for ROCm and s390x CI jobs、as it can leave workspace with bad permissions for cancelled jobs、Workaround for dind-rootless userid mapping (https://github.com/pytorch/ci-infra/issues/96)、Disable shellcheck SC2064 as we want to parse the original owner immediately. 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as cleanup_workspace to structure repeated tasks.
- **CN:** 脚本定义了 cleanup_workspace 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as cleanup_workspace, chown, showing the operational steps the workflow performs.
- **CN:** 它调用了 cleanup_workspace、chown 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ENVIRONMENT, WORKSPACE_ORIGINAL_OWNER_ID, RLIMIT_STACK, SC2064 communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT、WORKSPACE_ORIGINAL_OWNER_ID、RLIMIT_STACK、SC2064 等环境变量用于说明所需工具位置或行为开关。

### Lines 218-229 / 第 218-229 行

```bash
  # shellcheck disable=SC2064
  trap_add cleanup_workspace EXIT
  sudo chown -R jenkins /var/lib/jenkins/workspace
  git config --global --add safe.directory /var/lib/jenkins/workspace
fi

# check that setup.py would fail with bad arguments
echo "The next three invocations are expected to fail with invalid command error messages."
( ! get_exit_code python setup.py bad_argument )
( ! get_exit_code python setup.py clean] )
( ! get_exit_code python setup.py clean bad_argument )

```

- **EN:** This chunk introduces sections such as shellcheck disable=SC2064, check that setup.py would fail with bad arguments, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 shellcheck disable=SC2064、check that setup.py would fail with bad arguments 等标题组织周边说明或配置。
- **EN:** It invokes commands such as trap_add, chown, git, showing the operational steps the workflow performs.
- **CN:** 它调用了 trap_add、chown、git 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as SC2064, EXIT communicate required tool locations or behavioral switches.
- **CN:** SC2064、EXIT 等环境变量用于说明所需工具位置或行为开关。

### Lines 230-242 / 第 230-242 行

```bash
if [[ "$BUILD_ENVIRONMENT" != *libtorch* ]]; then
  # rocm builds fail when WERROR=1
  # XLA test build fails when WERROR=1
  # set only when building other architectures
  # or building non-XLA tests.
  if [[ "$BUILD_ENVIRONMENT" != *rocm*  && "$BUILD_ENVIRONMENT" != *xla* && "$BUILD_ENVIRONMENT" != *riscv64* ]]; then
    # TODO: Remove me and may be just focus on numpy-2.x testing
    if [[ "$ANACONDA_PYTHON_VERSION" =~ ^3\.1[0-2]$ ]]; then
      # Install numpy-2.0.2 for builds which are backward compatible with 1.X
      # In relality it's only needed for numpy_2_x and vllm shards (where vllm depends on numpy-2)
      python -mpip install numpy==2.0.2
    fi

```

- **EN:** This chunk introduces sections such as rocm builds fail when WERROR=1, XLA test build fails when WERROR=1, set only when building other architectures, or building non-XLA tests., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 rocm builds fail when WERROR=1、XLA test build fails when WERROR=1、set only when building other architectures、or building non-XLA tests. 等标题组织周边说明或配置。
- **EN:** It invokes commands such as python, showing the operational steps the workflow performs.
- **CN:** 它调用了 python 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ENVIRONMENT, WERROR, XLA, TODO, ANACONDA_PYTHON_VERSION communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT、WERROR、XLA、TODO、ANACONDA_PYTHON_VERSION 等环境变量用于说明所需工具位置或行为开关。

### Lines 243-255 / 第 243-255 行

```bash
    WERROR=1 python setup.py clean

    WERROR=1 python -m build --wheel --no-isolation
  else
    python setup.py clean
    if [[ "$BUILD_ENVIRONMENT" == *xla* ]]; then
      source .ci/pytorch/install_cache_xla.sh
    fi
    python -m build --wheel --no-isolation
  fi
  pip_install_whl "$(echo dist/*.whl)"
  if [[ "$BUILD_ENVIRONMENT" == *full-debug* ]]; then
    # Regression test for https://github.com/pytorch/pytorch/issues/164297
```

- **EN:** This chunk introduces sections such as Regression test for https://github.com/pytorch/pytorch/issues/164297, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Regression test for https://github.com/pytorch/pytorch/issues/164297 等标题组织周边说明或配置。
- **EN:** It invokes commands such as python, source, pip_install_whl, showing the operational steps the workflow performs.
- **CN:** 它调用了 python、source、pip_install_whl 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as WERROR, BUILD_ENVIRONMENT communicate required tool locations or behavioral switches.
- **CN:** WERROR、BUILD_ENVIRONMENT 等环境变量用于说明所需工具位置或行为开关。

### Lines 256-267 / 第 256-267 行

```bash
    # Torch should be importable and that's about it
    pushd /; python -c "import torch;print(torch.__config__.show(), torch.randn(5) + 1.7)"; popd
  fi

  if [[ "${BUILD_ADDITIONAL_PACKAGES:-}" == *vision* ]]; then
    install_torchvision
  fi

  if [[ "${BUILD_ADDITIONAL_PACKAGES:-}" == *audio* ]]; then
    install_torchaudio
  fi

```

- **EN:** This chunk introduces sections such as Torch should be importable and that's about it, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Torch should be importable and that's about it 等标题组织周边说明或配置。
- **EN:** It invokes commands such as pushd, install_torchvision, install_torchaudio, showing the operational steps the workflow performs.
- **CN:** 它调用了 pushd、install_torchvision、install_torchaudio 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ADDITIONAL_PACKAGES communicate required tool locations or behavioral switches.
- **CN:** BUILD_ADDITIONAL_PACKAGES 等环境变量用于说明所需工具位置或行为开关。

### Lines 268-287 / 第 268-287 行

```bash
  if [[ "${BUILD_ADDITIONAL_PACKAGES:-}" == *torchrec* || "${BUILD_ADDITIONAL_PACKAGES:-}" == *fbgemm* ]]; then
    install_torchrec_and_fbgemm
  fi

  if [[ "${BUILD_ADDITIONAL_PACKAGES:-}" == *torchao* ]]; then
    install_torchao
  fi

  if [[ "$BUILD_ENVIRONMENT" == *xpu* ]]; then
    echo "Checking that xpu is compiled"
    pushd dist/
    if python -c 'import torch; exit(0 if torch.xpu._is_compiled() else 1)'; then
      echo "XPU support is compiled in."
    else
      echo "XPU support is NOT compiled in."
      exit 1
    fi
    popd
  fi

```

- **EN:** It invokes commands such as install_torchrec_and_fbgemm, install_torchao, pushd, exit, popd, showing the operational steps the workflow performs.
- **CN:** 它调用了 install_torchrec_and_fbgemm、install_torchao、pushd、exit、popd 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ADDITIONAL_PACKAGES, BUILD_ENVIRONMENT, XPU, NOT communicate required tool locations or behavioral switches.
- **CN:** BUILD_ADDITIONAL_PACKAGES、BUILD_ENVIRONMENT、XPU、NOT 等环境变量用于说明所需工具位置或行为开关。

### Lines 288-299 / 第 288-299 行

```bash
  # TODO: I'm not sure why, but somehow we lose verbose commands
  set -x

  assert_git_not_dirty
  # Copy ninja build logs to dist folder
  mkdir -p dist
  if [ -f build/.ninja_log ]; then
    cp build/.ninja_log dist
  fi

  if [[ "$BUILD_ENVIRONMENT" == *rocm* ]]; then
    # remove sccache wrappers post-build; runtime compilation of MIOpen kernels does not yet fully support them
```

- **EN:** This chunk introduces sections such as TODO: I'm not sure why, but somehow we lose verbose commands, Copy ninja build logs to dist folder, remove sccache wrappers post-build; runtime compilation of MIOpen kernels does not yet fully support them, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 TODO: I'm not sure why, but somehow we lose verbose commands、Copy ninja build logs to dist folder、remove sccache wrappers post-build; runtime compilation of MIOpen kernels does not yet fully support them 等标题组织周边说明或配置。
- **EN:** It invokes commands such as assert_git_not_dirty, mkdir, cp, showing the operational steps the workflow performs.
- **CN:** 它调用了 assert_git_not_dirty、mkdir、cp 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as TODO, BUILD_ENVIRONMENT communicate required tool locations or behavioral switches.
- **CN:** TODO、BUILD_ENVIRONMENT 等环境变量用于说明所需工具位置或行为开关。

### Lines 300-317 / 第 300-317 行

```bash
    sudo rm -f /opt/cache/bin/cc
    sudo rm -f /opt/cache/bin/c++
    sudo rm -f /opt/cache/bin/gcc
    sudo rm -f /opt/cache/bin/g++
    # Restore original clang compilers that were backed up during sccache wrapping.
    # Skip for theRock nightly: sccache wrapping is disabled, so no backup exists.
    # theRock also uses ${ROCM_PATH}/lib/llvm/bin instead of /opt/rocm/llvm/bin.
    if [[ -d /opt/rocm/llvm/bin ]]; then
      pushd /opt/rocm/llvm/bin
      if [[ -d original ]]; then
        sudo mv original/clang .
        sudo mv original/clang++ .
      fi
      sudo rm -rf original
      popd
    fi
  fi

```

- **EN:** This chunk introduces sections such as Restore original clang compilers that were backed up during sccache wrapping., Skip for theRock nightly: sccache wrapping is disabled, so no backup exists., theRock also uses ${ROCM_PATH}/lib/llvm/bin instead of /opt/rocm/llvm/bin., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Restore original clang compilers that were backed up during sccache wrapping.、Skip for theRock nightly: sccache wrapping is disabled, so no backup exists.、theRock also uses ${ROCM_PATH}/lib/llvm/bin instead of /opt/rocm/llvm/bin. 等标题组织周边说明或配置。
- **EN:** It invokes commands such as rm, pushd, mv, popd, showing the operational steps the workflow performs.
- **CN:** 它调用了 rm、pushd、mv、popd 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as ROCM_PATH communicate required tool locations or behavioral switches.
- **CN:** ROCM_PATH 等环境变量用于说明所需工具位置或行为开关。

### Lines 318-336 / 第 318-336 行

```bash
  CUSTOM_TEST_ARTIFACT_BUILD_DIR=${CUSTOM_TEST_ARTIFACT_BUILD_DIR:-"build/custom_test_artifacts"}
  CUSTOM_TEST_USE_ROCM=$([[ "$BUILD_ENVIRONMENT" == *rocm* ]] && echo "ON" || echo "OFF")
  CUSTOM_TEST_MODULE_PATH="${PWD}/cmake/public"
  mkdir -pv "${CUSTOM_TEST_ARTIFACT_BUILD_DIR}"

  # Build custom operator tests.
  CUSTOM_OP_BUILD="${CUSTOM_TEST_ARTIFACT_BUILD_DIR}/custom-op-build"
  CUSTOM_OP_TEST="$PWD/test/custom_operator"
  python --version
  SITE_PACKAGES="$(python -c 'import site; print(";".join([x for x in site.getsitepackages()] + [x + "/torch" for x in site.getsitepackages()]))')"

  mkdir -p "$CUSTOM_OP_BUILD"
  pushd "$CUSTOM_OP_BUILD"
  cmake "$CUSTOM_OP_TEST" -DCMAKE_PREFIX_PATH="$SITE_PACKAGES" -DPython_EXECUTABLE="$(which python)" \
        -DCMAKE_MODULE_PATH="$CUSTOM_TEST_MODULE_PATH" -DUSE_ROCM="$CUSTOM_TEST_USE_ROCM"
  make VERBOSE=1
  popd
  assert_git_not_dirty

```

- **EN:** This chunk introduces sections such as Build custom operator tests., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Build custom operator tests. 等标题组织周边说明或配置。
- **EN:** It invokes commands such as mkdir, python, pushd, cmake, -DCMAKE_MODULE_PATH, make, showing the operational steps the workflow performs.
- **CN:** 它调用了 mkdir、python、pushd、cmake、-DCMAKE_MODULE_PATH、make 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CUSTOM_TEST_ARTIFACT_BUILD_DIR, CUSTOM_TEST_USE_ROCM, BUILD_ENVIRONMENT, OFF, CUSTOM_TEST_MODULE_PATH, PWD communicate required tool locations or behavioral switches.
- **CN:** CUSTOM_TEST_ARTIFACT_BUILD_DIR、CUSTOM_TEST_USE_ROCM、BUILD_ENVIRONMENT、OFF、CUSTOM_TEST_MODULE_PATH、PWD 等环境变量用于说明所需工具位置或行为开关。

### Lines 337-349 / 第 337-349 行

```bash
  # Build jit hook tests
  JIT_HOOK_BUILD="${CUSTOM_TEST_ARTIFACT_BUILD_DIR}/jit-hook-build"
  JIT_HOOK_TEST="$PWD/test/jit_hooks"
  python --version
  SITE_PACKAGES="$(python -c 'import site; print(";".join([x for x in site.getsitepackages()] + [x + "/torch" for x in site.getsitepackages()]))')"
  mkdir -p "$JIT_HOOK_BUILD"
  pushd "$JIT_HOOK_BUILD"
  cmake "$JIT_HOOK_TEST" -DCMAKE_PREFIX_PATH="$SITE_PACKAGES" -DPython_EXECUTABLE="$(which python)" \
        -DCMAKE_MODULE_PATH="$CUSTOM_TEST_MODULE_PATH" -DUSE_ROCM="$CUSTOM_TEST_USE_ROCM"
  make VERBOSE=1
  popd
  assert_git_not_dirty

```

- **EN:** This chunk introduces sections such as Build jit hook tests, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Build jit hook tests 等标题组织周边说明或配置。
- **EN:** It invokes commands such as python, mkdir, pushd, cmake, -DCMAKE_MODULE_PATH, make, showing the operational steps the workflow performs.
- **CN:** 它调用了 python、mkdir、pushd、cmake、-DCMAKE_MODULE_PATH、make 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as JIT_HOOK_BUILD, CUSTOM_TEST_ARTIFACT_BUILD_DIR, JIT_HOOK_TEST, PWD, SITE_PACKAGES, DCMAKE_PREFIX_PATH communicate required tool locations or behavioral switches.
- **CN:** JIT_HOOK_BUILD、CUSTOM_TEST_ARTIFACT_BUILD_DIR、JIT_HOOK_TEST、PWD、SITE_PACKAGES、DCMAKE_PREFIX_PATH 等环境变量用于说明所需工具位置或行为开关。

### Lines 350-362 / 第 350-362 行

```bash
  # Build custom backend tests.
  CUSTOM_BACKEND_BUILD="${CUSTOM_TEST_ARTIFACT_BUILD_DIR}/custom-backend-build"
  CUSTOM_BACKEND_TEST="$PWD/test/custom_backend"
  python --version
  mkdir -p "$CUSTOM_BACKEND_BUILD"
  pushd "$CUSTOM_BACKEND_BUILD"
  cmake "$CUSTOM_BACKEND_TEST" -DCMAKE_PREFIX_PATH="$SITE_PACKAGES" -DPython_EXECUTABLE="$(which python)" \
        -DCMAKE_MODULE_PATH="$CUSTOM_TEST_MODULE_PATH" -DUSE_ROCM="$CUSTOM_TEST_USE_ROCM"
  make VERBOSE=1
  popd
  assert_git_not_dirty
else
  # Test no-Python build
```

- **EN:** This chunk introduces sections such as Build custom backend tests., Test no-Python build, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Build custom backend tests.、Test no-Python build 等标题组织周边说明或配置。
- **EN:** It invokes commands such as python, mkdir, pushd, cmake, -DCMAKE_MODULE_PATH, make, showing the operational steps the workflow performs.
- **CN:** 它调用了 python、mkdir、pushd、cmake、-DCMAKE_MODULE_PATH、make 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CUSTOM_BACKEND_BUILD, CUSTOM_TEST_ARTIFACT_BUILD_DIR, CUSTOM_BACKEND_TEST, PWD, DCMAKE_PREFIX_PATH, SITE_PACKAGES communicate required tool locations or behavioral switches.
- **CN:** CUSTOM_BACKEND_BUILD、CUSTOM_TEST_ARTIFACT_BUILD_DIR、CUSTOM_BACKEND_TEST、PWD、DCMAKE_PREFIX_PATH、SITE_PACKAGES 等环境变量用于说明所需工具位置或行为开关。

### Lines 363-379 / 第 363-379 行

```bash
  echo "Building libtorch"

  # This is an attempt to mitigate flaky libtorch build OOM error. By default, the build parallelization
  # is set to be the number of CPU minus 2. So, let's try a more conservative value here. A 4xlarge has
  # 16 CPUs
  MAX_JOBS=$(nproc --ignore=4)
  export MAX_JOBS

  BUILD_LIBTORCH_PY=$PWD/tools/build_libtorch.py
  # Build outside the source tree so the artifacts don't interfere with
  # the workspace. /tmp is writable on both EC2 and OSDC runners.
  mkdir -p /tmp/cpp-build/caffe2
  pushd /tmp/cpp-build/caffe2
  WERROR=1 VERBOSE=1 DEBUG=1 python "$BUILD_LIBTORCH_PY"
  popd
fi

```

- **EN:** This chunk introduces sections such as This is an attempt to mitigate flaky libtorch build OOM error. By default, the build parallelization, is set to be the number of CPU minus 2. So, let's try a more conservative value here. A 4xlarge has, 16 CPUs, Build outside the source tree so the artifacts don't interfere with, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 This is an attempt to mitigate flaky libtorch build OOM error. By default, the build parallelization、is set to be the number of CPU minus 2. So, let's try a more conservative value here. A 4xlarge has、16 CPUs、Build outside the source tree so the artifacts don't interfere with 等标题组织周边说明或配置。
- **EN:** It invokes commands such as mkdir, pushd, popd, showing the operational steps the workflow performs.
- **CN:** 它调用了 mkdir、pushd、popd 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as OOM, CPU, MAX_JOBS, BUILD_LIBTORCH_PY, PWD, EC2 communicate required tool locations or behavioral switches.
- **CN:** OOM、CPU、MAX_JOBS、BUILD_LIBTORCH_PY、PWD、EC2 等环境变量用于说明所需工具位置或行为开关。

### Lines 380-388 / 第 380-388 行

```bash
if [[ "$BUILD_ENVIRONMENT" != *libtorch* ]]; then
  # export test times so that potential sharded tests that'll branch off this build will use consistent data
  # don't do this for libtorch as libtorch is C++ only and thus won't have python tests run on its build
  PYTHONPATH=. python tools/stats/export_test_times.py
fi
# don't do this for s390x or riscv64 as they don't use sccache
if [[ "$BUILD_ENVIRONMENT" != *s390x* && "$BUILD_ENVIRONMENT" != *riscv64* ]]; then
  print_sccache_stats
fi
```

- **EN:** This chunk introduces sections such as export test times so that potential sharded tests that'll branch off this build will use consistent data, don't do this for libtorch as libtorch is C++ only and thus won't have python tests run on its build, don't do this for s390x or riscv64 as they don't use sccache, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 export test times so that potential sharded tests that'll branch off this build will use consistent data、don't do this for libtorch as libtorch is C++ only and thus won't have python tests run on its build、don't do this for s390x or riscv64 as they don't use sccache 等标题组织周边说明或配置。
- **EN:** It invokes commands such as print_sccache_stats, showing the operational steps the workflow performs.
- **CN:** 它调用了 print_sccache_stats 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ENVIRONMENT, PYTHONPATH communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT、PYTHONPATH 等环境变量用于说明所需工具位置或行为开关。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **Android integration** — 聚焦 Android 构建、打包或运行时集成细节。
- **CMake build logic** — 控制构建系统配置、特性检查或依赖发现。
- **Repository bootstrap** — 与仓库状态、子模块或源码检出流程交互。
- **Representative symbols: cleanup_workspace** — 代表性符号：cleanup_workspace

## Dependencies / 依赖关系

- `"$(dirname`
- `/opt/riscv-cross-env/bin/activate`
- `/var/lib/jenkins/vulkansdk/setup-env.sh`
- `/opt/intel/oneapi/compiler/latest/env/vars.sh`
- `/opt/intel/oneapi/umf/latest/env/vars.sh`
- `/opt/intel/oneapi/ccl/latest/env/vars.sh`
- `/opt/intel/oneapi/mpi/latest/env/vars.sh`
- `/opt/intel/oneapi/pti/latest/env/vars.sh`
- `.ci/pytorch/install_cache_xla.sh`
- `bash`
- `python`
- `cmake`
- `make`
- `ninja`
- `git`
