# FindAtlas.cmake — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `cmake/Modules/FindAtlas.cmake`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines a CMake discovery/configuration module that locates external dependencies or build capabilities. Dependency discovery and platform-specific probing are central themes in the file. The opening comment frames the file as: "Find the Atlas (and Lapack) libraries The following variables are optionally searched for defaults Atlas_ROOT_DIR: Base directory where all Atlas components are found The following are set after configuration is done: Atlas_FOUND Atlas_INCLUDE_DIRS Atlas_LIBRARIES Atlas_LIBRARYRARY_DIRS."
- **Purpose (CN)**: 定义一个 CMake 发现/配置模块，用于定位外部依赖或构建能力。 依赖发现与平台特定探测是该文件的核心主题。 开头注释将该文件概括为：“Find the Atlas (and Lapack) libraries The following variables are optionally searched for defaults Atlas_ROOT_DIR: Base directory where all Atlas components are found The following are set after configuration is done: Atlas_FOUND Atlas_INCLUDE_DIRS Atlas_LIBRARIES Atlas_LIBRARYRARY_DIRS”。

## Content Analysis / 内容分析

### Lines 1-5 / 第 1-5 行

```cmake
# Find the Atlas (and Lapack) libraries
#
# The following variables are optionally searched for defaults
#  Atlas_ROOT_DIR:            Base directory where all Atlas components are found
#
```

- **EN:** This chunk introduces sections such as Find the Atlas (and Lapack) libraries, , The following variables are optionally searched for defaults, Atlas_ROOT_DIR:            Base directory where all Atlas components are found, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Find the Atlas (and Lapack) libraries、、The following variables are optionally searched for defaults、Atlas_ROOT_DIR:            Base directory where all Atlas components are found 等标题组织周边说明或配置。

### Lines 6-10 / 第 6-10 行

```cmake
# The following are set after configuration is done:
#  Atlas_FOUND
#  Atlas_INCLUDE_DIRS
#  Atlas_LIBRARIES
#  Atlas_LIBRARYRARY_DIRS
```

- **EN:** This chunk introduces sections such as The following are set after configuration is done:, Atlas_FOUND, Atlas_INCLUDE_DIRS, Atlas_LIBRARIES, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 The following are set after configuration is done:、Atlas_FOUND、Atlas_INCLUDE_DIRS、Atlas_LIBRARIES 等标题组织周边说明或配置。

### Lines 11-18 / 第 11-18 行

```cmake

set(Atlas_INCLUDE_SEARCH_PATHS
  /usr/include/atlas
  /usr/include/atlas-base
  $ENV{Atlas_ROOT_DIR}
  $ENV{Atlas_ROOT_DIR}/include
)

```

- **EN:** CMake commands like set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 19-25 / 第 19-25 行

```cmake
set(Atlas_LIB_SEARCH_PATHS
  /usr/lib/atlas
  /usr/lib/atlas-base
  $ENV{Atlas_ROOT_DIR}
  $ENV{Atlas_ROOT_DIR}/lib
)

```

- **EN:** CMake commands like set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 26-32 / 第 26-32 行

```cmake
find_path(Atlas_CBLAS_INCLUDE_DIR   NAMES cblas.h   PATHS ${Atlas_INCLUDE_SEARCH_PATHS})
find_path(Atlas_CLAPACK_INCLUDE_DIR NAMES clapack.h PATHS ${Atlas_INCLUDE_SEARCH_PATHS})

find_library(Atlas_CBLAS_LIBRARY NAMES  ptcblas_r ptcblas cblas_r cblas       PATHS ${Atlas_LIB_SEARCH_PATHS})
find_library(Atlas_BLAS_LIBRARY NAMES   atlas_r   atlas                       PATHS ${Atlas_LIB_SEARCH_PATHS})
find_library(Atlas_LAPACK_LIBRARY NAMES lapack alapack_r alapack lapack_atlas PATHS ${Atlas_LIB_SEARCH_PATHS})

```

- **EN:** CMake commands like find_path, find_library drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 find_path、find_library 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 33-41 / 第 33-41 行

```cmake
set(LOOKED_FOR
  Atlas_CBLAS_INCLUDE_DIR
  Atlas_CLAPACK_INCLUDE_DIR

  Atlas_CBLAS_LIBRARY
  Atlas_BLAS_LIBRARY
  Atlas_LAPACK_LIBRARY
)

```

- **EN:** CMake commands like set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 42-49 / 第 42-49 行

```cmake
include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(Atlas DEFAULT_MSG ${LOOKED_FOR})

if(ATLAS_FOUND)
  set(Atlas_INCLUDE_DIR ${Atlas_CBLAS_INCLUDE_DIR} ${Atlas_CLAPACK_INCLUDE_DIR})
  set(Atlas_LIBRARIES ${Atlas_LAPACK_LIBRARY} ${Atlas_CBLAS_LIBRARY} ${Atlas_BLAS_LIBRARY})
  mark_as_advanced(${LOOKED_FOR})

```

- **EN:** CMake commands like include, find_package_handle_standard_args, if, set, mark_as_advanced drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 include、find_package_handle_standard_args、if、set、mark_as_advanced 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 50-51 / 第 50-51 行

```cmake
  message(STATUS "Found Atlas (include: ${Atlas_CBLAS_INCLUDE_DIR}, library: ${Atlas_BLAS_LIBRARY})")
endif(ATLAS_FOUND)
```

- **EN:** CMake commands like message, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 message、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

## Key Concepts / 关键概念

- **CMake dependency module** — CMake 依赖发现模块
- **CMake build logic** — 控制构建系统配置、特性检查或依赖发现。
- **External package discovery** — 搜索 PyTorch 构建所需的外部库。
- **Representative symbols: set, find_path, find_library, include, find_package_handle_standard_args, if, mark_as_advanced, message** — 代表性符号：set、find_path、find_library、include、find_package_handle_standard_args、if、mark_as_advanced、message

## Dependencies / 依赖关系

- `FindPackageHandleStandardArgs`
- `Atlas_CBLAS_LIBRARY`
- `Atlas_BLAS_LIBRARY`
- `Atlas_LAPACK_LIBRARY`
