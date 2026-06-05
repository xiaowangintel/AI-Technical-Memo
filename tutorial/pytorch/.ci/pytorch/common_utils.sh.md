# common_utils.sh — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `.ci/pytorch/common_utils.sh`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Automates repository maintenance, build, test, packaging, or release tasks through shell commands. The opening comment frames the file as: "!/bin/bash."
- **Purpose (CN)**: 通过 shell 命令自动化执行仓库维护、构建、测试、打包或发布任务。 开头注释将该文件概括为：“!/bin/bash”。

## Content Analysis / 内容分析

### Lines 1-15 / 第 1-15 行

```bash
#!/bin/bash

# Common util **functions** that can be sourced in other scripts.

# note: printf is used instead of echo to avoid backslash
# processing and to properly handle values that begin with a '-'.

log() { printf '%s\n' "$*"; }
error() { log "ERROR: $*" >&2; }
fatal() { error "$@"; exit 1; }

retry () {
    "$@" || (sleep 10 && "$@") || (sleep 20 && "$@") || (sleep 40 && "$@")
}

```

- **EN:** This chunk introduces sections such as !/bin/bash, Common util **functions** that can be sourced in other scripts., note: printf is used instead of echo to avoid backslash, processing and to properly handle values that begin with a '-'., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 !/bin/bash、Common util **functions** that can be sourced in other scripts.、note: printf is used instead of echo to avoid backslash、processing and to properly handle values that begin with a '-'. 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as log, error, fatal, retry to structure repeated tasks.
- **CN:** 脚本定义了 log、error、fatal、retry 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as log, error, fatal, retry, showing the operational steps the workflow performs.
- **CN:** 它调用了 log、error、fatal、retry 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as ERROR communicate required tool locations or behavioral switches.
- **CN:** ERROR 等环境变量用于说明所需工具位置或行为开关。

### Lines 16-27 / 第 16-27 行

```bash
# compositional trap taken from https://stackoverflow.com/a/7287873/23845
# appends a command to a trap
#
# - 1st arg:  code to add
# - remaining args:  names of traps to modify
#
trap_add() {
    trap_add_cmd=$1; shift || fatal "${FUNCNAME[0]} usage error"
    for trap_add_name in "$@"; do
        trap -- "$(
            # helper fn to get existing trap command from output
            # of trap -p
```

- **EN:** This chunk introduces sections such as compositional trap taken from https://stackoverflow.com/a/7287873/23845, appends a command to a trap, , - 1st arg:  code to add, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 compositional trap taken from https://stackoverflow.com/a/7287873/23845、appends a command to a trap、、- 1st arg:  code to add 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as trap_add to structure repeated tasks.
- **CN:** 脚本定义了 trap_add 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as trap_add, showing the operational steps the workflow performs.
- **CN:** 它调用了 trap_add 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as FUNCNAME communicate required tool locations or behavioral switches.
- **CN:** FUNCNAME 等环境变量用于说明所需工具位置或行为开关。

### Lines 28-39 / 第 28-39 行

```bash
            extract_trap_cmd() { printf '%s\n' "$3"; }
            # print existing trap command with newline
            eval "extract_trap_cmd $(trap -p "${trap_add_name}")"
            # print the new trap command
            printf '%s\n' "${trap_add_cmd}"
        )" "${trap_add_name}" \
            || fatal "unable to add to trap ${trap_add_name}"
    done
}
# set the trace attribute for the above function.  this is
# required to modify DEBUG or RETURN traps because functions don't
# inherit them unless the trace attribute is set
```

- **EN:** This chunk introduces sections such as print existing trap command with newline, print the new trap command, set the trace attribute for the above function.  this is, required to modify DEBUG or RETURN traps because functions don't, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 print existing trap command with newline、print the new trap command、set the trace attribute for the above function.  this is、required to modify DEBUG or RETURN traps because functions don't 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as extract_trap_cmd to structure repeated tasks.
- **CN:** 脚本定义了 extract_trap_cmd 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as extract_trap_cmd, eval, printf, showing the operational steps the workflow performs.
- **CN:** 它调用了 extract_trap_cmd、eval、printf 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as DEBUG, RETURN communicate required tool locations or behavioral switches.
- **CN:** DEBUG、RETURN 等环境变量用于说明所需工具位置或行为开关。

### Lines 40-55 / 第 40-55 行

```bash
declare -f -t trap_add

function assert_git_not_dirty() {
    # TODO: we should add an option to `build_amd.py` that reverts the repo to
    #       an unmodified state.
    if [[ "$BUILD_ENVIRONMENT" != *rocm* ]] && [[ "$BUILD_ENVIRONMENT" != *xla* ]] ; then
        git_status=$(git status --porcelain | grep -v '?? third_party' || true)
        if [[ $git_status ]]; then
            echo "Build left local git repository checkout dirty"
            echo "git status --porcelain:"
            echo "${git_status}"
            exit 1
        fi
    fi
}

```

- **EN:** This chunk introduces sections such as TODO: we should add an option to `build_amd.py` that reverts the repo to, an unmodified state., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 TODO: we should add an option to `build_amd.py` that reverts the repo to、an unmodified state. 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as assert_git_not_dirty to structure repeated tasks.
- **CN:** 脚本定义了 assert_git_not_dirty 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as declare, exit, showing the operational steps the workflow performs.
- **CN:** 它调用了 declare、exit 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as TODO, BUILD_ENVIRONMENT communicate required tool locations or behavioral switches.
- **CN:** TODO、BUILD_ENVIRONMENT 等环境变量用于说明所需工具位置或行为开关。

### Lines 56-67 / 第 56-67 行

```bash
function pip_install_whl() {
  # This is used to install PyTorch and other build artifacts wheel locally
  # without using any network connection

  # Convert the input arguments into an array
  local args=("$@")

  # Check if the first argument contains multiple paths separated by spaces
  if [[ "${args[0]}" == *" "* ]]; then
    # Split the string by spaces into an array
    IFS=' ' read -r -a paths <<< "${args[0]}"
    # Loop through each path and install individually
```

- **EN:** This chunk introduces sections such as This is used to install PyTorch and other build artifacts wheel locally, without using any network connection, Convert the input arguments into an array, Check if the first argument contains multiple paths separated by spaces, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 This is used to install PyTorch and other build artifacts wheel locally、without using any network connection、Convert the input arguments into an array、Check if the first argument contains multiple paths separated by spaces 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as pip_install_whl to structure repeated tasks.
- **CN:** 脚本定义了 pip_install_whl 等 shell 辅助函数，以组织重复任务。
- **EN:** Environment variables such as IFS communicate required tool locations or behavioral switches.
- **CN:** IFS 等环境变量用于说明所需工具位置或行为开关。

### Lines 68-80 / 第 68-80 行

```bash
    for path in "${paths[@]}"; do
      echo "Installing $path"
      python3 -mpip install --no-index --no-deps "$path"
    done
  else
    # Loop through each argument and install individually
    for path in "${args[@]}"; do
      echo "Installing $path"
      python3 -mpip install --no-index --no-deps "$path"
    done
  fi
}

```

- **EN:** This chunk introduces sections such as Loop through each argument and install individually, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Loop through each argument and install individually 等标题组织周边说明或配置。
- **EN:** It invokes commands such as python3, showing the operational steps the workflow performs.
- **CN:** 它调用了 python3 等命令，展示该工作流执行的操作步骤。

### Lines 81-93 / 第 81-93 行

```bash
function pip_build_and_install() {
  local build_target=$1
  local wheel_dir=$2

  local found_whl=0
  for file in "${wheel_dir}"/*.whl
  do
    if [[ -f "${file}" ]]; then
      found_whl=1
      break
    fi
  done

```

- **EN:** The script defines shell helpers such as pip_build_and_install to structure repeated tasks.
- **CN:** 脚本定义了 pip_build_and_install 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as break, showing the operational steps the workflow performs.
- **CN:** 它调用了 break 等命令，展示该工作流执行的操作步骤。

### Lines 94-108 / 第 94-108 行

```bash
  # Build the wheel if it doesn't exist
  if [ "${found_whl}" == "0" ]; then
    python3 -m pip wheel \
      --no-build-isolation \
      --no-deps \
      -w "${wheel_dir}" \
      "${build_target}"
  fi

  for file in "${wheel_dir}"/*.whl
  do
    pip_install_whl "${file}"
  done
}

```

- **EN:** This chunk introduces sections such as Build the wheel if it doesn't exist, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Build the wheel if it doesn't exist 等标题组织周边说明或配置。
- **EN:** It invokes commands such as python3, --no-build-isolation, --no-deps, -w, pip_install_whl, showing the operational steps the workflow performs.
- **CN:** 它调用了 python3、--no-build-isolation、--no-deps、-w、pip_install_whl 等命令，展示该工作流执行的操作步骤。

### Lines 109-121 / 第 109-121 行

```bash
function pip_install() {
  # retry 3 times
  pip_install_pkg="python3 -m pip install --progress-bar off"
  ${pip_install_pkg} "$@" || \
    ${pip_install_pkg} "$@" || \
    ${pip_install_pkg} "$@"
}

function pip_uninstall() {
  # uninstall 2 times
  pip3 uninstall -y "$@" || pip3 uninstall -y "$@"
}

```

- **EN:** This chunk introduces sections such as retry 3 times, uninstall 2 times, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 retry 3 times、uninstall 2 times 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as pip_install, pip_uninstall to structure repeated tasks.
- **CN:** 脚本定义了 pip_install、pip_uninstall 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as pip3, showing the operational steps the workflow performs.
- **CN:** 它调用了 pip3 等命令，展示该工作流执行的操作步骤。

### Lines 122-134 / 第 122-134 行

```bash
function get_exit_code() {
  set +e
  "$@"
  retcode=$?
  set -e
  return $retcode
}

function install_monkeytype {
  # Install MonkeyType
  pip_install MonkeyType
}

```

- **EN:** This chunk introduces sections such as Install MonkeyType, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Install MonkeyType 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as get_exit_code to structure repeated tasks.
- **CN:** 脚本定义了 get_exit_code 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as return, pip_install, showing the operational steps the workflow performs.
- **CN:** 它调用了 return、pip_install 等命令，展示该工作流执行的操作步骤。

### Lines 135-146 / 第 135-146 行

```bash

function get_pinned_commit() {
  cat .github/ci_commit_pins/"${1}".txt
}

function detect_cuda_arch() {
  if [[ "${BUILD_ENVIRONMENT}" == *cuda* ]]; then
    if command -v nvidia-smi; then
      TORCH_CUDA_ARCH_LIST=$(nvidia-smi --query-gpu=compute_cap --format=csv | tail -n 1)
    elif [[ "${TEST_CONFIG}" == *nogpu* ]]; then
      # There won't be nvidia-smi in nogpu tests, so just set TORCH_CUDA_ARCH_LIST to the default
      # minimum supported value here
```

- **EN:** This chunk introduces sections such as There won't be nvidia-smi in nogpu tests, so just set TORCH_CUDA_ARCH_LIST to the default, minimum supported value here, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 There won't be nvidia-smi in nogpu tests, so just set TORCH_CUDA_ARCH_LIST to the default、minimum supported value here 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as get_pinned_commit, detect_cuda_arch to structure repeated tasks.
- **CN:** 脚本定义了 get_pinned_commit、detect_cuda_arch 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as cat, showing the operational steps the workflow performs.
- **CN:** 它调用了 cat 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ENVIRONMENT, TORCH_CUDA_ARCH_LIST, TEST_CONFIG communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT、TORCH_CUDA_ARCH_LIST、TEST_CONFIG 等环境变量用于说明所需工具位置或行为开关。

### Lines 147-158 / 第 147-158 行

```bash
      TORCH_CUDA_ARCH_LIST=8.0
    fi
    export TORCH_CUDA_ARCH_LIST
  fi
}

function install_torchaudio() {
  local commit
  commit=$(get_pinned_commit audio)
  pip_build_and_install "git+https://github.com/pytorch/audio.git@${commit}" dist/audio
}

```

- **EN:** The script defines shell helpers such as install_torchaudio to structure repeated tasks.
- **CN:** 脚本定义了 install_torchaudio 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as pip_build_and_install, showing the operational steps the workflow performs.
- **CN:** 它调用了 pip_build_and_install 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as TORCH_CUDA_ARCH_LIST communicate required tool locations or behavioral switches.
- **CN:** TORCH_CUDA_ARCH_LIST 等环境变量用于说明所需工具位置或行为开关。

### Lines 159-174 / 第 159-174 行

```bash
function install_torchtext() {
  local data_commit
  local text_commit
  data_commit=$(get_pinned_commit data)
  text_commit=$(get_pinned_commit text)
  pip_build_and_install "git+https://github.com/pytorch/data.git@${data_commit}" dist/data
  pip_build_and_install "git+https://github.com/pytorch/text.git@${text_commit}" dist/text
}

function install_torchvision() {
  local orig_preload
  local commit
  commit=$(get_pinned_commit vision)
  orig_preload=${LD_PRELOAD}
  if [ -n "${LD_PRELOAD}" ]; then
    # Silence dlerror to work-around glibc ASAN bug, see https://sourceware.org/bugzilla/show_bug.cgi?id=27653#c9
```

- **EN:** This chunk introduces sections such as Silence dlerror to work-around glibc ASAN bug, see https://sourceware.org/bugzilla/show_bug.cgi?id=27653#c9, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Silence dlerror to work-around glibc ASAN bug, see https://sourceware.org/bugzilla/show_bug.cgi?id=27653#c9 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as install_torchtext, install_torchvision to structure repeated tasks.
- **CN:** 脚本定义了 install_torchtext、install_torchvision 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as pip_build_and_install, showing the operational steps the workflow performs.
- **CN:** 它调用了 pip_build_and_install 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as LD_PRELOAD, ASAN communicate required tool locations or behavioral switches.
- **CN:** LD_PRELOAD、ASAN 等环境变量用于说明所需工具位置或行为开关。

### Lines 175-190 / 第 175-190 行

```bash
    echo 'char* dlerror(void) { return "";}'|gcc -fpic -shared -o "${HOME}/dlerror.so" -x c -
    LD_PRELOAD=${orig_preload}:${HOME}/dlerror.so
  fi

  if [[ "${BUILD_ENVIRONMENT}" == *cuda* ]]; then
    # Not sure if both are needed, but why not
    export FORCE_CUDA=1
    export WITH_CUDA=1
  fi
  pip_build_and_install "git+https://github.com/pytorch/vision.git@${commit}" dist/vision

  if [ -n "${LD_PRELOAD}" ]; then
    LD_PRELOAD=${orig_preload}
  fi
}

```

- **EN:** This chunk introduces sections such as Not sure if both are needed, but why not, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Not sure if both are needed, but why not 等标题组织周边说明或配置。
- **EN:** It invokes commands such as pip_build_and_install, showing the operational steps the workflow performs.
- **CN:** 它调用了 pip_build_and_install 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as HOME, LD_PRELOAD, BUILD_ENVIRONMENT, FORCE_CUDA, WITH_CUDA communicate required tool locations or behavioral switches.
- **CN:** HOME、LD_PRELOAD、BUILD_ENVIRONMENT、FORCE_CUDA、WITH_CUDA 等环境变量用于说明所需工具位置或行为开关。

### Lines 191-210 / 第 191-210 行

```bash
function install_fbgemm() {
  local build_variant=$1

  local fbgemm_commit
  fbgemm_commit=$(get_pinned_commit fbgemm)
  if [[ "$BUILD_ENVIRONMENT" == *rocm* ]] ; then
    fbgemm_commit=$(get_pinned_commit fbgemm_rocm)
  fi

  # Check if the wheel has been already been built
  local wheel_dir=dist/fbgemm_gpu
  local found_whl=0
  for file in "${wheel_dir}"/*.whl
  do
    if [[ -f "${file}" ]]; then
      found_whl=1
      break
    fi
  done

```

- **EN:** This chunk introduces sections such as Check if the wheel has been already been built, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Check if the wheel has been already been built 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as install_fbgemm to structure repeated tasks.
- **CN:** 脚本定义了 install_fbgemm 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as break, showing the operational steps the workflow performs.
- **CN:** 它调用了 break 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ENVIRONMENT communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT 等环境变量用于说明所需工具位置或行为开关。

### Lines 211-222 / 第 211-222 行

```bash
  pip_install tabulate==0.9.0 tensordict==0.10.0  # needed for newer fbgemm
  pip_install patchelf  # needed for rocm fbgemm

  # Build the wheel if it doesn't exist
  if [ "${found_whl}" == "0" ]; then
    git clone --recursive https://github.com/pytorch/fbgemm
    pushd fbgemm/fbgemm_gpu
    git checkout "${fbgemm_commit}" --recurse-submodules
    python setup.py bdist_wheel --build-target=default --build-variant="${build_variant}"
    popd

    # Save the wheel before cleaning up
```

- **EN:** This chunk introduces sections such as Build the wheel if it doesn't exist, Save the wheel before cleaning up, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Build the wheel if it doesn't exist、Save the wheel before cleaning up 等标题组织周边说明或配置。
- **EN:** It invokes commands such as pip_install, git, pushd, python, popd, showing the operational steps the workflow performs.
- **CN:** 它调用了 pip_install、git、pushd、python、popd 等命令，展示该工作流执行的操作步骤。

### Lines 223-236 / 第 223-236 行

```bash
    mkdir -p dist/fbgemm_gpu
    cp fbgemm/fbgemm_gpu/dist/*.whl dist/fbgemm_gpu
  fi

  # Install fbgemm wheel
  for file in "${wheel_dir}"/*.whl
  do
    pip_install_whl "${file}"
  done

  # Clean up
  rm -rf fbgemm
}

```

- **EN:** This chunk introduces sections such as Install fbgemm wheel, Clean up, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Install fbgemm wheel、Clean up 等标题组织周边说明或配置。
- **EN:** It invokes commands such as mkdir, cp, pip_install_whl, rm, showing the operational steps the workflow performs.
- **CN:** 它调用了 mkdir、cp、pip_install_whl、rm 等命令，展示该工作流执行的操作步骤。

### Lines 237-249 / 第 237-249 行

```bash
function install_torchrec_and_fbgemm() {
  local torchrec_commit
  torchrec_commit=$(get_pinned_commit torchrec)

  pip_uninstall torchrec-nightly
  pip_uninstall fbgemm-gpu-nightly
  pip_install setuptools-git-versioning scikit-build pyre-extensions

  if [[ "$BUILD_ENVIRONMENT" == *rocm* ]] ; then
    # install torchrec first because it installs fbgemm nightly on top of rocm fbgemm
    pip_build_and_install "git+https://github.com/pytorch/torchrec.git@${torchrec_commit}" dist/torchrec
    pip_uninstall fbgemm-gpu-nightly

```

- **EN:** This chunk introduces sections such as install torchrec first because it installs fbgemm nightly on top of rocm fbgemm, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 install torchrec first because it installs fbgemm nightly on top of rocm fbgemm 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as install_torchrec_and_fbgemm to structure repeated tasks.
- **CN:** 脚本定义了 install_torchrec_and_fbgemm 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as pip_uninstall, pip_install, pip_build_and_install, showing the operational steps the workflow performs.
- **CN:** 它调用了 pip_uninstall、pip_install、pip_build_and_install 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ENVIRONMENT communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT 等环境变量用于说明所需工具位置或行为开关。

### Lines 250-261 / 第 250-261 行

```bash
    # Find rocm_version.h header file for ROCm version extract
    rocm_version_h="${ROCM_HOME}/include/rocm-core/rocm_version.h"
    if [ ! -f "$rocm_version_h" ]; then
        rocm_version_h="${ROCM_HOME}/include/rocm_version.h"
    fi

    # Error out if rocm_version.h not found
    if [ ! -f "$rocm_version_h" ]; then
        echo "Error: rocm_version.h not found in expected locations." >&2
        exit 1
    fi

```

- **EN:** This chunk introduces sections such as Find rocm_version.h header file for ROCm version extract, Error out if rocm_version.h not found, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Find rocm_version.h header file for ROCm version extract、Error out if rocm_version.h not found 等标题组织周边说明或配置。
- **EN:** It invokes commands such as exit, showing the operational steps the workflow performs.
- **CN:** 它调用了 exit 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as ROCM_HOME communicate required tool locations or behavioral switches.
- **CN:** ROCM_HOME 等环境变量用于说明所需工具位置或行为开关。

### Lines 262-273 / 第 262-273 行

```bash
    # Extract major, minor and patch ROCm version numbers
    MAJOR_VERSION=$(grep 'ROCM_VERSION_MAJOR' "$rocm_version_h" | awk '{print $3}')
    MINOR_VERSION=$(grep 'ROCM_VERSION_MINOR' "$rocm_version_h" | awk '{print $3}')
    PATCH_VERSION=$(grep 'ROCM_VERSION_PATCH' "$rocm_version_h" | awk '{print $3}')
    ROCM_INT=$((MAJOR_VERSION * 10000 + MINOR_VERSION * 100 + PATCH_VERSION))
    echo "ROCm version: $ROCM_INT"
    export BUILD_ROCM_VERSION="$MAJOR_VERSION.$MINOR_VERSION"

    install_fbgemm "rocm"
  else
    pip_build_and_install "git+https://github.com/pytorch/torchrec.git@${torchrec_commit}" dist/torchrec
    # Skip fbgemm for CUDA 13 as it's not compatible yet
```

- **EN:** This chunk introduces sections such as Extract major, minor and patch ROCm version numbers, Skip fbgemm for CUDA 13 as it's not compatible yet, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Extract major, minor and patch ROCm version numbers、Skip fbgemm for CUDA 13 as it's not compatible yet 等标题组织周边说明或配置。
- **EN:** It invokes commands such as install_fbgemm, pip_build_and_install, showing the operational steps the workflow performs.
- **CN:** 它调用了 install_fbgemm、pip_build_and_install 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as MAJOR_VERSION, ROCM_VERSION_MAJOR, MINOR_VERSION, ROCM_VERSION_MINOR, PATCH_VERSION, ROCM_VERSION_PATCH communicate required tool locations or behavioral switches.
- **CN:** MAJOR_VERSION、ROCM_VERSION_MAJOR、MINOR_VERSION、ROCM_VERSION_MINOR、PATCH_VERSION、ROCM_VERSION_PATCH 等环境变量用于说明所需工具位置或行为开关。

### Lines 274-291 / 第 274-291 行

```bash
    if [[ "$BUILD_ENVIRONMENT" != *cuda13* ]]; then
      install_fbgemm "cuda"
    fi
  fi
}

function clone_pytorch_xla() {
  if [[ ! -d ./xla ]]; then
    git clone --recursive --quiet https://github.com/pytorch/xla.git
    pushd xla
    # pin the xla hash so that we don't get broken by changes to xla
    git checkout "$(cat ../.github/ci_commit_pins/xla.txt)"
    git submodule sync
    git submodule update --init --recursive
    popd
  fi
}

```

- **EN:** This chunk introduces sections such as pin the xla hash so that we don't get broken by changes to xla, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 pin the xla hash so that we don't get broken by changes to xla 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as clone_pytorch_xla to structure repeated tasks.
- **CN:** 脚本定义了 clone_pytorch_xla 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as install_fbgemm, git, pushd, popd, showing the operational steps the workflow performs.
- **CN:** 它调用了 install_fbgemm、git、pushd、popd 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ENVIRONMENT communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT 等环境变量用于说明所需工具位置或行为开关。

### Lines 292-303 / 第 292-303 行

```bash
function install_torchao() {
  local commit
  commit=$(get_pinned_commit torchao)
  pip_build_and_install "git+https://github.com/pytorch/ao.git@${commit}" dist/ao
}

function install_flash_attn_cute() {
  echo "Installing FlashAttention 4 from PyPI..."
  pip_install flash-attn-4==4.0.0b5
  echo "FlashAttention 4 installation complete."
}

```

- **EN:** The script defines shell helpers such as install_torchao, install_flash_attn_cute to structure repeated tasks.
- **CN:** 脚本定义了 install_torchao、install_flash_attn_cute 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as pip_build_and_install, pip_install, showing the operational steps the workflow performs.
- **CN:** 它调用了 pip_build_and_install、pip_install 等命令，展示该工作流执行的操作步骤。

### Lines 304-317 / 第 304-317 行

```bash
function install_cutlass_dsl() {
  # cutlass-dsl requires Python >= 3.12
  local py_version
  py_version=$(python3 -c "import sys; print(f'{sys.version_info.major}.{sys.version_info.minor}')")
  if [[ "$(echo -e "3.12\n$py_version" | sort -V | head -n1)" != "3.12" ]]; then
    echo "Skipping CUTLASS DSL install: requires Python >= 3.12, have $py_version"
    return 0
  fi

  echo "Installing NVIDIA CUTLASS DSL from PyPI..."
  pip_install nvidia-cutlass-dsl
  echo "NVIDIA CUTLASS DSL installation complete."
}

```

- **EN:** This chunk introduces sections such as cutlass-dsl requires Python >= 3.12, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 cutlass-dsl requires Python >= 3.12 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as install_cutlass_dsl to structure repeated tasks.
- **CN:** 脚本定义了 install_cutlass_dsl 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as return, pip_install, showing the operational steps the workflow performs.
- **CN:** 它调用了 return、pip_install 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CUTLASS, DSL, NVIDIA communicate required tool locations or behavioral switches.
- **CN:** CUTLASS、DSL、NVIDIA 等环境变量用于说明所需工具位置或行为开关。

### Lines 318-329 / 第 318-329 行

```bash
function install_cutlass_api() {
  # cutlass-api requires Python >= 3.12
  local py_version
  py_version=$(python3 -c "import sys; print(f'{sys.version_info.major}.{sys.version_info.minor}')")
  if [[ "$(echo -e "3.12\n$py_version" | sort -V | head -n1)" != "3.12" ]]; then
    echo "Skipping CUTLASS API install: requires Python >= 3.12, have $py_version"
    return 0
  fi

  echo "Installing CUTLASS API from Github..."

  # Install CuTeDSL dependency first
```

- **EN:** This chunk introduces sections such as cutlass-api requires Python >= 3.12, Install CuTeDSL dependency first, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 cutlass-api requires Python >= 3.12、Install CuTeDSL dependency first 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as install_cutlass_api to structure repeated tasks.
- **CN:** 脚本定义了 install_cutlass_api 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as return, showing the operational steps the workflow performs.
- **CN:** 它调用了 return 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CUTLASS, API communicate required tool locations or behavioral switches.
- **CN:** CUTLASS、API 等环境变量用于说明所需工具位置或行为开关。

### Lines 330-341 / 第 330-341 行

```bash
  install_cutlass_dsl

  # Grab latest til we have a pinned commit
  local cutlass_commit
  cutlass_commit=$(git ls-remote https://github.com/NVIDIA/cutlass.git refs/heads/cutlass_api | cut -f1)

  rm -rf cutlass-build
  git clone --depth 1 -b cutlass_api https://github.com/NVIDIA/cutlass.git cutlass-build

  pushd cutlass-build
  git checkout "${cutlass_commit}"

```

- **EN:** This chunk introduces sections such as Grab latest til we have a pinned commit, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Grab latest til we have a pinned commit 等标题组织周边说明或配置。
- **EN:** It invokes commands such as install_cutlass_dsl, rm, git, pushd, showing the operational steps the workflow performs.
- **CN:** 它调用了 install_cutlass_dsl、rm、git、pushd 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as NVIDIA communicate required tool locations or behavioral switches.
- **CN:** NVIDIA 等环境变量用于说明所需工具位置或行为开关。

### Lines 342-353 / 第 342-353 行

```bash
  # Install cutlass_api with torch extras
  pip_install "python/cutlass_api[torch]"
  popd

  rm -rf cutlass-build
  echo "CUTLASS API installation complete."
}

function print_sccache_stats() {
  echo 'PyTorch Build Statistics'
  sccache --show-stats

```

- **EN:** This chunk introduces sections such as Install cutlass_api with torch extras, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Install cutlass_api with torch extras 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as print_sccache_stats to structure repeated tasks.
- **CN:** 脚本定义了 print_sccache_stats 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as pip_install, popd, rm, sccache, showing the operational steps the workflow performs.
- **CN:** 它调用了 pip_install、popd、rm、sccache 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CUTLASS, API communicate required tool locations or behavioral switches.
- **CN:** CUTLASS、API 等环境变量用于说明所需工具位置或行为开关。

### Lines 354-360 / 第 354-360 行

```bash
  if [[ -n "${OUR_GITHUB_JOB_ID}" ]]; then
    sccache --show-stats --stats-format json | jq .stats \
      > "sccache-stats-${BUILD_ENVIRONMENT}-${OUR_GITHUB_JOB_ID}.json"
  else
    echo "env var OUR_GITHUB_JOB_ID not set, will not write sccache stats to json"
  fi
}
```

- **EN:** It invokes commands such as sccache, showing the operational steps the workflow performs.
- **CN:** 它调用了 sccache 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as OUR_GITHUB_JOB_ID, BUILD_ENVIRONMENT communicate required tool locations or behavioral switches.
- **CN:** OUR_GITHUB_JOB_ID、BUILD_ENVIRONMENT 等环境变量用于说明所需工具位置或行为开关。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **Repository bootstrap** — 与仓库状态、子模块或源码检出流程交互。
- **Structured metadata** — 存放机器可读的设置或清单式元数据。
- **Representative symbols: log, error, fatal, retry, trap_add, extract_trap_cmd, assert_git_not_dirty, pip_install_whl** — 代表性符号：log、error、fatal、retry、trap_add、extract_trap_cmd、assert_git_not_dirty、pip_install_whl

## Dependencies / 依赖关系

- `bash`
- `git`
- `python3`
- `python`
