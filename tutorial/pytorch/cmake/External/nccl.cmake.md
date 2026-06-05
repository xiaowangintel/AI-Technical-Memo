# nccl.cmake — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `cmake/External/nccl.cmake`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines CMake configuration logic that shapes how the PyTorch build is configured. Dependency discovery and platform-specific probing are central themes in the file.
- **Purpose (CN)**: 定义影响 PyTorch 构建配置方式的 CMake 逻辑。 依赖发现与平台特定探测是该文件的核心主题。

## Content Analysis / 内容分析

### Lines 1-5 / 第 1-5 行

```cmake
if(NOT __NCCL_INCLUDED)
  set(__NCCL_INCLUDED TRUE)

  if(USE_SYSTEM_NCCL)
    # NCCL_ROOT, NCCL_LIB_DIR, NCCL_INCLUDE_DIR will be accounted in the following line.
```

- **EN:** This chunk introduces sections such as NCCL_ROOT, NCCL_LIB_DIR, NCCL_INCLUDE_DIR will be accounted in the following line., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 NCCL_ROOT, NCCL_LIB_DIR, NCCL_INCLUDE_DIR will be accounted in the following line. 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 6-15 / 第 6-15 行

```cmake
    find_package(NCCL REQUIRED)
    if(NCCL_FOUND)
      add_library(__caffe2_nccl INTERFACE)
      target_link_libraries(__caffe2_nccl INTERFACE ${NCCL_LIBRARIES})
      target_include_directories(__caffe2_nccl INTERFACE ${NCCL_INCLUDE_DIRS})
    endif()
  else()
    torch_cuda_get_nvcc_gencode_flag(NVCC_GENCODE)
    string(REPLACE "-gencode;" "-gencode=" NVCC_GENCODE "${NVCC_GENCODE}")
    # this second replacement is needed when there are multiple archs
```

- **EN:** This chunk introduces sections such as this second replacement is needed when there are multiple archs, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 this second replacement is needed when there are multiple archs 等标题组织周边说明或配置。
- **EN:** CMake commands like find_package, if, add_library, target_link_libraries, target_include_directories, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 find_package、if、add_library、target_link_libraries、target_include_directories、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 16-23 / 第 16-23 行

```cmake
    string(REPLACE ";-gencode" " -gencode" NVCC_GENCODE "${NVCC_GENCODE}")

    if(DEFINED ENV{MAX_JOBS})
      set(MAX_JOBS "$ENV{MAX_JOBS}")
    else()
      include(ProcessorCount)
      ProcessorCount(NUM_HARDWARE_THREADS)
      # Assume 2 hardware threads per cpu core
```

- **EN:** This chunk introduces sections such as Assume 2 hardware threads per cpu core, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Assume 2 hardware threads per cpu core 等标题组织周边说明或配置。
- **EN:** CMake commands like string, if, set, else, include, ProcessorCount drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 string、if、set、else、include、ProcessorCount 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 24-30 / 第 24-30 行

```cmake
      math(EXPR MAX_JOBS "${NUM_HARDWARE_THREADS} / 2")
      # ProcessorCount might return 0, set to a positive number
      if(MAX_JOBS LESS 2)
        set(MAX_JOBS 2)
      endif()
    endif()

```

- **EN:** This chunk introduces sections such as ProcessorCount might return 0, set to a positive number, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ProcessorCount might return 0, set to a positive number 等标题组织周边说明或配置。
- **EN:** CMake commands like math, if, set, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 math、if、set、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 31-36 / 第 31-36 行

```cmake
    if("${CMAKE_GENERATOR}" MATCHES "Make")
      # Recursive make with jobserver for parallelism, and also put a load limit
      # here to avoid flaky OOM, https://www.gnu.org/software/make/manual/html_node/Parallel.html
      set(MAKE_COMMAND "$(MAKE)" "-l${MAX_JOBS}")
    else()
      # Parallel build with CPU load limit to avoid oversubscription
```

- **EN:** This chunk introduces sections such as Recursive make with jobserver for parallelism, and also put a load limit, here to avoid flaky OOM, https://www.gnu.org/software/make/manual/html_node/Parallel.html, Parallel build with CPU load limit to avoid oversubscription, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Recursive make with jobserver for parallelism, and also put a load limit、here to avoid flaky OOM, https://www.gnu.org/software/make/manual/html_node/Parallel.html、Parallel build with CPU load limit to avoid oversubscription 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, else drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、else 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 37-46 / 第 37-46 行

```cmake
      set(MAKE_COMMAND "make" "-j${MAX_JOBS}" "-l${MAX_JOBS}")
    endif()

    set(__NCCL_BUILD_DIR "${CMAKE_CURRENT_BINARY_DIR}/nccl")
    ExternalProject_Add(nccl_external
      SOURCE_DIR ${PROJECT_SOURCE_DIR}/third_party/nccl
      BUILD_IN_SOURCE 1
      CONFIGURE_COMMAND ""
      BUILD_COMMAND
        ${MAKE_COMMAND}
```

- **EN:** CMake commands like set, endif, ExternalProject_Add drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、endif、ExternalProject_Add 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 47-56 / 第 47-56 行

```cmake
        "CXX=${CMAKE_CXX_COMPILER}"
        "CUDA_HOME=${CUDA_TOOLKIT_ROOT_DIR}"
        "NVCC=${CUDA_NVCC_EXECUTABLE}"
        "NVCC_GENCODE=${NVCC_GENCODE}"
        "BUILDDIR=${__NCCL_BUILD_DIR}"
        "VERBOSE=0"
        "DEBUG=0"
      BUILD_BYPRODUCTS "${__NCCL_BUILD_DIR}/lib/libnccl_static.a"
      INSTALL_COMMAND ""
      )
```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 57-63 / 第 57-63 行

```cmake

    set(__NCCL_LIBRARY_DEP nccl_external)
    set(NCCL_LIBRARIES ${__NCCL_BUILD_DIR}/lib/libnccl_static.a)

    set(NCCL_FOUND TRUE)
    add_library(__caffe2_nccl INTERFACE)
    # The following old-style variables are set so that other libs, such as Gloo,
```

- **EN:** This chunk introduces sections such as The following old-style variables are set so that other libs, such as Gloo,, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 The following old-style variables are set so that other libs, such as Gloo, 等标题组织周边说明或配置。
- **EN:** CMake commands like set, add_library drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、add_library 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 64-69 / 第 64-69 行

```cmake
    # can still use it.
    set(NCCL_INCLUDE_DIRS ${__NCCL_BUILD_DIR}/include)
    add_dependencies(__caffe2_nccl ${__NCCL_LIBRARY_DEP})
    target_link_libraries(__caffe2_nccl INTERFACE ${NCCL_LIBRARIES})
    target_include_directories(__caffe2_nccl INTERFACE ${NCCL_INCLUDE_DIRS})
    # nccl includes calls to shm_open/shm_close and therefore must depend on librt on Linux
```

- **EN:** This chunk introduces sections such as can still use it., nccl includes calls to shm_open/shm_close and therefore must depend on librt on Linux, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 can still use it.、nccl includes calls to shm_open/shm_close and therefore must depend on librt on Linux 等标题组织周边说明或配置。
- **EN:** CMake commands like set, add_dependencies, target_link_libraries, target_include_directories drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、add_dependencies、target_link_libraries、target_include_directories 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 70-74 / 第 70-74 行

```cmake
    if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
      target_link_libraries(__caffe2_nccl INTERFACE rt)
    endif()
  endif()
endif()
```

- **EN:** CMake commands like if, target_link_libraries, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、target_link_libraries、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

## Key Concepts / 关键概念

- **CMake build configuration** — CMake 构建配置
- **CMake build logic** — 控制构建系统配置、特性检查或依赖发现。
- **External package discovery** — 搜索 PyTorch 构建所需的外部库。
- **Representative symbols: if, set, find_package, add_library, target_link_libraries, target_include_directories, endif, else** — 代表性符号：if、set、find_package、add_library、target_link_libraries、target_include_directories、endif、else

## Dependencies / 依赖关系

- `ProcessorCount`
- `NCCL`
