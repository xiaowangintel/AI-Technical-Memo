# install_miopen.sh — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `.ci/docker/common/install_miopen.sh`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Automates repository maintenance, build, test, packaging, or release tasks through shell commands. The opening comment frames the file as: "!/bin/bash Script used only in CD pipeline."
- **Purpose (CN)**: 通过 shell 命令自动化执行仓库维护、构建、测试、打包或发布任务。 开头注释将该文件概括为：“!/bin/bash Script used only in CD pipeline”。

## Content Analysis / 内容分析

### Lines 1-12 / 第 1-12 行

```bash
#!/bin/bash
# Script used only in CD pipeline

set -ex

ROCM_VERSION=$1

if [[ -z $ROCM_VERSION ]]; then
    echo "missing ROCM_VERSION"
    exit 1;
fi

```

- **EN:** This chunk introduces sections such as !/bin/bash, Script used only in CD pipeline, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 !/bin/bash、Script used only in CD pipeline 等标题组织周边说明或配置。
- **EN:** It invokes commands such as exit, showing the operational steps the workflow performs.
- **CN:** 它调用了 exit 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as ROCM_VERSION communicate required tool locations or behavioral switches.
- **CN:** ROCM_VERSION 等环境变量用于说明所需工具位置或行为开关。

### Lines 13-27 / 第 13-27 行

```bash
IS_UBUNTU=0
ID=$(grep -oP '(?<=^ID=).+' /etc/os-release | tr -d '"')
case "$ID" in
  ubuntu)
    IS_UBUNTU=1
    ;;
  almalinux)
    IS_UBUNTU=0
    ;;
  *)
    echo "Unable to determine OS..."
    exit 1
    ;;
esac

```

- **EN:** It invokes commands such as ubuntu, almalinux, exit, showing the operational steps the workflow performs.
- **CN:** 它调用了 ubuntu、almalinux、exit 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as IS_UBUNTU communicate required tool locations or behavioral switches.
- **CN:** IS_UBUNTU 等环境变量用于说明所需工具位置或行为开关。

### Lines 28-43 / 第 28-43 行

```bash
# To make version comparison easier, create an integer representation.
save_IFS="$IFS"
IFS=. ROCM_VERSION_ARRAY=(${ROCM_VERSION})
IFS="$save_IFS"
if [[ ${#ROCM_VERSION_ARRAY[@]} == 2 ]]; then
    ROCM_VERSION_MAJOR=${ROCM_VERSION_ARRAY[0]}
    ROCM_VERSION_MINOR=${ROCM_VERSION_ARRAY[1]}
    ROCM_VERSION_PATCH=0
elif [[ ${#ROCM_VERSION_ARRAY[@]} == 3 ]]; then
    ROCM_VERSION_MAJOR=${ROCM_VERSION_ARRAY[0]}
    ROCM_VERSION_MINOR=${ROCM_VERSION_ARRAY[1]}
    ROCM_VERSION_PATCH=${ROCM_VERSION_ARRAY[2]}
else
    echo "Unhandled ROCM_VERSION ${ROCM_VERSION}"
    exit 1
fi
```

- **EN:** This chunk introduces sections such as To make version comparison easier, create an integer representation., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 To make version comparison easier, create an integer representation. 等标题组织周边说明或配置。
- **EN:** It invokes commands such as exit, showing the operational steps the workflow performs.
- **CN:** 它调用了 exit 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as IFS, ROCM_VERSION_ARRAY, ROCM_VERSION, ROCM_VERSION_MAJOR, ROCM_VERSION_MINOR, ROCM_VERSION_PATCH communicate required tool locations or behavioral switches.
- **CN:** IFS、ROCM_VERSION_ARRAY、ROCM_VERSION、ROCM_VERSION_MAJOR、ROCM_VERSION_MINOR、ROCM_VERSION_PATCH 等环境变量用于说明所需工具位置或行为开关。

### Lines 44-51 / 第 44-51 行

```bash
ROCM_INT=$(($ROCM_VERSION_MAJOR * 10000 + $ROCM_VERSION_MINOR * 100 + $ROCM_VERSION_PATCH))

# Function to retry functions that sometimes timeout or have flaky failures
retry () {
    $*  || (sleep 1 && $*) || (sleep 2 && $*) || (sleep 4 && $*) || (sleep 8 && $*)
}

# Build custom MIOpen to use comgr for offline compilation.
```

- **EN:** This chunk introduces sections such as Function to retry functions that sometimes timeout or have flaky failures, Build custom MIOpen to use comgr for offline compilation., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Function to retry functions that sometimes timeout or have flaky failures、Build custom MIOpen to use comgr for offline compilation. 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as retry to structure repeated tasks.
- **CN:** 脚本定义了 retry 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as retry, showing the operational steps the workflow performs.
- **CN:** 它调用了 retry 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as ROCM_INT, ROCM_VERSION_MAJOR, ROCM_VERSION_MINOR, ROCM_VERSION_PATCH communicate required tool locations or behavioral switches.
- **CN:** ROCM_INT、ROCM_VERSION_MAJOR、ROCM_VERSION_MINOR、ROCM_VERSION_PATCH 等环境变量用于说明所需工具位置或行为开关。

### Lines 52-62 / 第 52-62 行

```bash

## Need a sanitized ROCM_VERSION without patchlevel; patchlevel version 0 must be added to paths.
ROCM_DOTS=$(echo ${ROCM_VERSION} | tr -d -c '.' | wc -c)
if [[ ${ROCM_DOTS} == 1 ]]; then
    ROCM_VERSION_NOPATCH="${ROCM_VERSION}"
    ROCM_INSTALL_PATH="/opt/rocm-${ROCM_VERSION}.0"
else
    ROCM_VERSION_NOPATCH="${ROCM_VERSION%.*}"
    ROCM_INSTALL_PATH="/opt/rocm-${ROCM_VERSION}"
fi

```

- **EN:** This chunk introduces sections such as Need a sanitized ROCM_VERSION without patchlevel; patchlevel version 0 must be added to paths., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Need a sanitized ROCM_VERSION without patchlevel; patchlevel version 0 must be added to paths. 等标题组织周边说明或配置。
- **EN:** Environment variables such as ROCM_VERSION, ROCM_DOTS, ROCM_VERSION_NOPATCH, ROCM_INSTALL_PATH communicate required tool locations or behavioral switches.
- **CN:** ROCM_VERSION、ROCM_DOTS、ROCM_VERSION_NOPATCH、ROCM_INSTALL_PATH 等环境变量用于说明所需工具位置或行为开关。

### Lines 63-73 / 第 63-73 行

```bash
MIOPEN_CMAKE_COMMON_FLAGS="
-DMIOPEN_USE_COMGR=ON
-DMIOPEN_BUILD_DRIVER=OFF
"
if [[ $ROCM_INT -ge 60200 ]] && [[ $ROCM_INT -lt 60204 ]]; then
    MIOPEN_BRANCH="release/rocm-rel-6.2-staging"
else
    echo "ROCm ${ROCM_VERSION} does not need any patches, do not build from source"
    exit 0
fi

```

- **EN:** It invokes commands such as -DMIOPEN_USE_COMGR, -DMIOPEN_BUILD_DRIVER, exit, showing the operational steps the workflow performs.
- **CN:** 它调用了 -DMIOPEN_USE_COMGR、-DMIOPEN_BUILD_DRIVER、exit 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as MIOPEN_CMAKE_COMMON_FLAGS, DMIOPEN_USE_COMGR, DMIOPEN_BUILD_DRIVER, OFF, ROCM_INT, MIOPEN_BRANCH communicate required tool locations or behavioral switches.
- **CN:** MIOPEN_CMAKE_COMMON_FLAGS、DMIOPEN_USE_COMGR、DMIOPEN_BUILD_DRIVER、OFF、ROCM_INT、MIOPEN_BRANCH 等环境变量用于说明所需工具位置或行为开关。

### Lines 74-81 / 第 74-81 行

```bash

if [[ ${IS_UBUNTU} == 1 ]]; then
  apt-get remove -y miopen-hip
else
  # Workaround since almalinux manylinux image already has this and cget doesn't like that
  rm -rf /usr/local/lib/pkgconfig/sqlite3.pc

  # Versioned package name needs regex match
```

- **EN:** This chunk introduces sections such as Workaround since almalinux manylinux image already has this and cget doesn't like that, Versioned package name needs regex match, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Workaround since almalinux manylinux image already has this and cget doesn't like that、Versioned package name needs regex match 等标题组织周边说明或配置。
- **EN:** It invokes commands such as apt-get, rm, showing the operational steps the workflow performs.
- **CN:** 它调用了 apt-get、rm 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as IS_UBUNTU communicate required tool locations or behavioral switches.
- **CN:** IS_UBUNTU 等环境变量用于说明所需工具位置或行为开关。

### Lines 82-90 / 第 82-90 行

```bash
  # Use --noautoremove to prevent other rocm packages from being uninstalled
  yum remove -y miopen-hip* --noautoremove
fi

git clone https://github.com/ROCm/MIOpen -b ${MIOPEN_BRANCH}
pushd MIOpen
# remove .git to save disk space since CI runner was running out
rm -rf .git
# Don't build CK to save docker build time
```

- **EN:** This chunk introduces sections such as Use --noautoremove to prevent other rocm packages from being uninstalled, remove .git to save disk space since CI runner was running out, Don't build CK to save docker build time, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Use --noautoremove to prevent other rocm packages from being uninstalled、remove .git to save disk space since CI runner was running out、Don't build CK to save docker build time 等标题组织周边说明或配置。
- **EN:** It invokes commands such as yum, git, pushd, rm, showing the operational steps the workflow performs.
- **CN:** 它调用了 yum、git、pushd、rm 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as MIOPEN_BRANCH communicate required tool locations or behavioral switches.
- **CN:** MIOPEN_BRANCH 等环境变量用于说明所需工具位置或行为开关。

### Lines 91-106 / 第 91-106 行

```bash
sed -i '/composable_kernel/d' requirements.txt
## MIOpen minimum requirements
cmake -P install_deps.cmake --minimum

# clean up since CI runner was running out of disk space
rm -rf /tmp/*
if [[ ${IS_UBUNTU} == 1 ]]; then
  apt-get autoclean && apt-get clean
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
else
  yum clean all
  rm -rf /var/cache/yum
  rm -rf /var/lib/yum/yumdb
  rm -rf /var/lib/yum/history
fi

```

- **EN:** This chunk introduces sections such as MIOpen minimum requirements, clean up since CI runner was running out of disk space, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 MIOpen minimum requirements、clean up since CI runner was running out of disk space 等标题组织周边说明或配置。
- **EN:** It invokes commands such as sed, cmake, rm, apt-get, yum, showing the operational steps the workflow performs.
- **CN:** 它调用了 sed、cmake、rm、apt-get、yum 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as IS_UBUNTU communicate required tool locations or behavioral switches.
- **CN:** IS_UBUNTU 等环境变量用于说明所需工具位置或行为开关。

### Lines 107-115 / 第 107-115 行

```bash
## Build MIOpen
mkdir -p build
cd build
PKG_CONFIG_PATH=/usr/local/lib/pkgconfig CXX=${ROCM_INSTALL_PATH}/llvm/bin/clang++ cmake .. \
    ${MIOPEN_CMAKE_COMMON_FLAGS} \
    ${MIOPEN_CMAKE_DB_FLAGS} \
    -DCMAKE_PREFIX_PATH="${ROCM_INSTALL_PATH}"
make MIOpen -j $(nproc)

```

- **EN:** This chunk introduces sections such as Build MIOpen, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Build MIOpen 等标题组织周边说明或配置。
- **EN:** It invokes commands such as mkdir, cd, -DCMAKE_PREFIX_PATH, make, showing the operational steps the workflow performs.
- **CN:** 它调用了 mkdir、cd、-DCMAKE_PREFIX_PATH、make 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as PKG_CONFIG_PATH, CXX, ROCM_INSTALL_PATH, MIOPEN_CMAKE_COMMON_FLAGS, MIOPEN_CMAKE_DB_FLAGS, DCMAKE_PREFIX_PATH communicate required tool locations or behavioral switches.
- **CN:** PKG_CONFIG_PATH、CXX、ROCM_INSTALL_PATH、MIOPEN_CMAKE_COMMON_FLAGS、MIOPEN_CMAKE_DB_FLAGS、DCMAKE_PREFIX_PATH 等环境变量用于说明所需工具位置或行为开关。

### Lines 116-127 / 第 116-127 行

```bash
# Build MIOpen package
make -j $(nproc) package

# clean up since CI runner was running out of disk space
rm -rf /usr/local/cget

if [[ ${IS_UBUNTU} == 1 ]]; then
  sudo dpkg -i miopen-hip*.deb
else
  yum install -y miopen-*.rpm
fi

```

- **EN:** This chunk introduces sections such as Build MIOpen package, clean up since CI runner was running out of disk space, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Build MIOpen package、clean up since CI runner was running out of disk space 等标题组织周边说明或配置。
- **EN:** It invokes commands such as make, rm, dpkg, yum, showing the operational steps the workflow performs.
- **CN:** 它调用了 make、rm、dpkg、yum 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as IS_UBUNTU communicate required tool locations or behavioral switches.
- **CN:** IS_UBUNTU 等环境变量用于说明所需工具位置或行为开关。

### Lines 128-129 / 第 128-129 行

```bash
popd
rm -rf MIOpen
```

- **EN:** It invokes commands such as popd, rm, showing the operational steps the workflow performs.
- **CN:** 它调用了 popd、rm 等命令，展示该工作流执行的操作步骤。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **CMake build logic** — 控制构建系统配置、特性检查或依赖发现。
- **Repository bootstrap** — 与仓库状态、子模块或源码检出流程交互。
- **Representative symbols: retry** — 代表性符号：retry

## Dependencies / 依赖关系

- `bash`
- `make`
- `git`
- `cmake`
