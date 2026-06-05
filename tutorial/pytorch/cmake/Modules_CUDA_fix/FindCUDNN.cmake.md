# FindCUDNN.cmake — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `cmake/Modules_CUDA_fix/FindCUDNN.cmake`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines CMake configuration logic that shapes how the PyTorch build is configured. Dependency discovery and platform-specific probing are central themes in the file. The opening comment frames the file as: "Find the CUDNN libraries The following variables are optionally searched for defaults CUDNN_ROOT: Base directory where CUDNN is found CUDNN_INCLUDE_DIR: Directory where CUDNN header is searched for CUDNN_LIBRARY: Directory where CUDNN library is searched for CUDNN_STATIC: Are we looking for a static library? (default: no) The following are set after configuration is done: CUDNN_FOUND CUDNN_INCLUDE_PATH CUDNN_LIBRARY_PATH."
- **Purpose (CN)**: 定义影响 PyTorch 构建配置方式的 CMake 逻辑。 依赖发现与平台特定探测是该文件的核心主题。 开头注释将该文件概括为：“Find the CUDNN libraries The following variables are optionally searched for defaults CUDNN_ROOT: Base directory where CUDNN is found CUDNN_INCLUDE_DIR: Directory where CUDNN header is searched for CUDNN_LIBRARY: Directory where CUDNN library is searched for CUDNN_STATIC: Are we looking for a static library? (default: no) The following are set after configuration is done: CUDNN_FOUND CUDNN_INCLUDE_PATH CUDNN_LIBRARY_PATH”。

## Content Analysis / 内容分析

### Lines 1-5 / 第 1-5 行

```cmake
# Find the CUDNN libraries
#
# The following variables are optionally searched for defaults
#  CUDNN_ROOT: Base directory where CUDNN is found
#  CUDNN_INCLUDE_DIR: Directory where CUDNN header is searched for
```

- **EN:** This chunk introduces sections such as Find the CUDNN libraries, , The following variables are optionally searched for defaults, CUDNN_ROOT: Base directory where CUDNN is found, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Find the CUDNN libraries、、The following variables are optionally searched for defaults、CUDNN_ROOT: Base directory where CUDNN is found 等标题组织周边说明或配置。

### Lines 6-10 / 第 6-10 行

```cmake
#  CUDNN_LIBRARY: Directory where CUDNN library is searched for
#  CUDNN_STATIC: Are we looking for a static library? (default: no)
#
# The following are set after configuration is done:
#  CUDNN_FOUND
```

- **EN:** This chunk introduces sections such as CUDNN_LIBRARY: Directory where CUDNN library is searched for, CUDNN_STATIC: Are we looking for a static library? (default: no), , The following are set after configuration is done:, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 CUDNN_LIBRARY: Directory where CUDNN library is searched for、CUDNN_STATIC: Are we looking for a static library? (default: no)、、The following are set after configuration is done: 等标题组织周边说明或配置。

### Lines 11-16 / 第 11-16 行

```cmake
#  CUDNN_INCLUDE_PATH
#  CUDNN_LIBRARY_PATH
#

include(FindPackageHandleStandardArgs)

```

- **EN:** This chunk introduces sections such as CUDNN_INCLUDE_PATH, CUDNN_LIBRARY_PATH, , which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 CUDNN_INCLUDE_PATH、CUDNN_LIBRARY_PATH、 等标题组织周边说明或配置。
- **EN:** CMake commands like include drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 include 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 17-22 / 第 17-22 行

```cmake
set(CUDNN_ROOT $ENV{CUDNN_ROOT_DIR} CACHE PATH "Folder containing NVIDIA cuDNN")
if (DEFINED $ENV{CUDNN_ROOT_DIR})
  message(WARNING "CUDNN_ROOT_DIR is deprecated. Please set CUDNN_ROOT instead.")
endif()
list(APPEND CUDNN_ROOT $ENV{CUDNN_ROOT_DIR} ${CUDA_TOOLKIT_ROOT_DIR})

```

- **EN:** CMake commands like set, if, message, endif, list drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、if、message、endif、list 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 23-27 / 第 23-27 行

```cmake
# Compatible layer for CMake <3.12. CUDNN_ROOT will be accounted in for searching paths and libraries for CMake >=3.12.
list(APPEND CMAKE_PREFIX_PATH ${CUDNN_ROOT})

set(CUDNN_INCLUDE_DIR $ENV{CUDNN_INCLUDE_DIR} CACHE PATH "Folder containing NVIDIA cuDNN header files")

```

- **EN:** This chunk introduces sections such as Compatible layer for CMake <3.12. CUDNN_ROOT will be accounted in for searching paths and libraries for CMake >=3.12., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Compatible layer for CMake <3.12. CUDNN_ROOT will be accounted in for searching paths and libraries for CMake >=3.12. 等标题组织周边说明或配置。
- **EN:** CMake commands like list, set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 list、set 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 28-37 / 第 28-37 行

```cmake
find_path(CUDNN_INCLUDE_PATH cudnn.h
  HINTS ${CUDNN_INCLUDE_DIR}
  PATH_SUFFIXES cuda/include cuda include)

option(CUDNN_STATIC "Look for static CUDNN" OFF)
if (CUDNN_STATIC)
  set(CUDNN_LIBNAME "libcudnn_static.a")
else()
  set(CUDNN_LIBNAME "cudnn")
endif()
```

- **EN:** CMake commands like find_path, option, if, set, else, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 find_path、option、if、set、else、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 38-43 / 第 38-43 行

```cmake

set(CUDNN_LIBRARY $ENV{CUDNN_LIBRARY} CACHE PATH "Path to the cudnn library file (e.g., libcudnn.so)")
if (CUDNN_LIBRARY MATCHES ".*cudnn_static.a" AND NOT CUDNN_STATIC)
  message(WARNING "CUDNN_LIBRARY points to a static library (${CUDNN_LIBRARY}) but CUDNN_STATIC is OFF.")
endif()

```

- **EN:** CMake commands like set, if, message, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、if、message、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 44-49 / 第 44-49 行

```cmake
find_library(CUDNN_LIBRARY_PATH ${CUDNN_LIBNAME}
  PATHS ${CUDNN_LIBRARY}
  PATH_SUFFIXES lib lib64 cuda/lib cuda/lib64 lib/x64)

find_package_handle_standard_args(CUDNN DEFAULT_MSG CUDNN_LIBRARY_PATH CUDNN_INCLUDE_PATH)

```

- **EN:** CMake commands like find_library, find_package_handle_standard_args drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 find_library、find_package_handle_standard_args 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 50-59 / 第 50-59 行

```cmake
if(CUDNN_FOUND)
  # Get cuDNN version
  if(EXISTS ${CUDNN_INCLUDE_PATH}/cudnn_version.h)
    file(READ ${CUDNN_INCLUDE_PATH}/cudnn_version.h CUDNN_HEADER_CONTENTS)
  else()
    file(READ ${CUDNN_INCLUDE_PATH}/cudnn.h CUDNN_HEADER_CONTENTS)
  endif()
  string(REGEX MATCH "define CUDNN_MAJOR * +([0-9]+)"
               CUDNN_VERSION_MAJOR "${CUDNN_HEADER_CONTENTS}")
  string(REGEX REPLACE "define CUDNN_MAJOR * +([0-9]+)" "\\1"
```

- **EN:** This chunk introduces sections such as Get cuDNN version, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Get cuDNN version 等标题组织周边说明或配置。
- **EN:** CMake commands like if, file, else, endif, string drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、file、else、endif、string 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 60-69 / 第 60-69 行

```cmake
               CUDNN_VERSION_MAJOR "${CUDNN_VERSION_MAJOR}")
  string(REGEX MATCH "define CUDNN_MINOR * +([0-9]+)"
               CUDNN_VERSION_MINOR "${CUDNN_HEADER_CONTENTS}")
  string(REGEX REPLACE "define CUDNN_MINOR * +([0-9]+)" "\\1"
               CUDNN_VERSION_MINOR "${CUDNN_VERSION_MINOR}")
  string(REGEX MATCH "define CUDNN_PATCHLEVEL * +([0-9]+)"
               CUDNN_VERSION_PATCH "${CUDNN_HEADER_CONTENTS}")
  string(REGEX REPLACE "define CUDNN_PATCHLEVEL * +([0-9]+)" "\\1"
               CUDNN_VERSION_PATCH "${CUDNN_VERSION_PATCH}")
  # Assemble cuDNN version
```

- **EN:** This chunk introduces sections such as Assemble cuDNN version, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Assemble cuDNN version 等标题组织周边说明或配置。
- **EN:** CMake commands like string drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 string 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 70-77 / 第 70-77 行

```cmake
  if(NOT CUDNN_VERSION_MAJOR)
    set(CUDNN_VERSION "?")
  else()
    set(CUDNN_VERSION
        "${CUDNN_VERSION_MAJOR}.${CUDNN_VERSION_MINOR}.${CUDNN_VERSION_PATCH}")
  endif()
endif()

```

- **EN:** CMake commands like if, set, else, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、else、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 78-78 / 第 78-78 行

```cmake
mark_as_advanced(CUDNN_ROOT CUDNN_INCLUDE_DIR CUDNN_LIBRARY CUDNN_VERSION)
```

- **EN:** CMake commands like mark_as_advanced drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 mark_as_advanced 等 CMake 命令完成依赖探测、变量设置或平台检查。

## Key Concepts / 关键概念

- **CMake build configuration** — CMake 构建配置
- **CMake build logic** — 控制构建系统配置、特性检查或依赖发现。
- **External package discovery** — 搜索 PyTorch 构建所需的外部库。
- **Representative symbols: include, set, if, message, endif, list, find_path, option** — 代表性符号：include、set、if、message、endif、list、find_path、option

## Dependencies / 依赖关系

- `FindPackageHandleStandardArgs`
- `CUDNN_LIBRARY_PATH`
