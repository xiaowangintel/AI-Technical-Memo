# FindMKLDNN.cmake — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `cmake/Modules/FindMKLDNN.cmake`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements a CMake find-module that searches for an external dependency and reports whether it is usable.
- **用途 (CN)**: 实现 CMake 查找模块，用于搜索外部依赖并报告其是否可用。

## Content Analysis / 内容分析
### Lines 1-17 / 第 1-17 行

````cmake
# - Try to find MKLDNN
#
# The following variables are optionally searched for defaults
#  MKL_FOUND             : set to true if a library implementing the CBLAS interface is found
#
# The following are set after configuration is done:
#  MKLDNN_FOUND          : set to true if mkl-dnn is found.
#  MKLDNN_INCLUDE_DIR    : path to mkl-dnn include dir.
#  MKLDNN_LIBRARIES      : list of libraries for mkl-dnn
#
# The following variables are used:
#  MKLDNN_USE_NATIVE_ARCH : Whether native CPU instructions should be used in MKLDNN. This should be turned off for
#  general packaging to avoid incompatible CPU instructions. Default: OFF.

IF(NOT MKLDNN_FOUND)
  SET(MKLDNN_LIBRARIES)
  SET(MKLDNN_INCLUDE_DIR)
````

- EN: This section defines configure-time variables and search paths; checks discovery results before exposing them to the wider build.
- CN: 该部分定义配置阶段变量与搜索路径；在对外暴露结果前检查发现状态。

### Lines 19-29 / 第 19-29 行

````cmake
  SET(IDEEP_ROOT "${PROJECT_SOURCE_DIR}/third_party/ideep")
  SET(MKLDNN_ROOT "${PROJECT_SOURCE_DIR}/third_party/ideep/mkl-dnn")

  if(USE_XPU) # Build oneDNN GPU library
    if(WIN32)
      # Windows
      set(DNNL_HOST_COMPILER "DEFAULT")
      set(SYCL_CXX_DRIVER "icx")
      set(DNNL_LIB_NAME "dnnl.lib")
    elseif(LINUX)
      # Linux
````

- EN: This section defines configure-time variables and search paths; checks discovery results before exposing them to the wider build.
- CN: 该部分定义配置阶段变量与搜索路径；在对外暴露结果前检查发现状态。

### Lines 30-47 / 第 30-47 行

````cmake
      # g++ is soft linked to /usr/bin/cxx, oneDNN would not treat it as an absolute path
      set(DNNL_HOST_COMPILER "g++")
      set(SYCL_CXX_DRIVER "icpx")
      set(DNNL_LIB_NAME "libdnnl.a")
    else()
      MESSAGE(FATAL_ERROR "OneDNN for Intel GPU in PyTorch currently supports only Windows and Linux.
                           Detected system '${CMAKE_SYSTEM_NAME}' is not supported.")
    endif()

    set(DNNL_MAKE_COMMAND "cmake" "--build" ".")
    include(ProcessorCount)
    ProcessorCount(proc_cnt)
    if((DEFINED ENV{MAX_JOBS}) AND ("$ENV{MAX_JOBS}" LESS_EQUAL ${proc_cnt}))
      list(APPEND DNNL_MAKE_COMMAND "-j" "$ENV{MAX_JOBS}")
      if(CMAKE_GENERATOR MATCHES "Make|Ninja")
        list(APPEND DNNL_MAKE_COMMAND "--" "-l" "$ENV{MAX_JOBS}")
      endif()
    endif()
````

- EN: This section defines configure-time variables and search paths; checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分定义配置阶段变量与搜索路径；在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 48-65 / 第 48-65 行

````cmake
    ExternalProject_Add(xpu_mkldnn_proj
      GIT_REPOSITORY https://github.com/uxlfoundation/oneDNN
      GIT_TAG main
      PREFIX ${XPU_MKLDNN_DIR_PREFIX}
      BUILD_IN_SOURCE 0
      CMAKE_ARGS  -DCMAKE_C_COMPILER=icx
      -DCMAKE_CXX_COMPILER=${SYCL_CXX_DRIVER}
      -DDNNL_GPU_RUNTIME=SYCL
      -DDNNL_CPU_RUNTIME=THREADPOOL
      -DDNNL_BUILD_TESTS=OFF
      -DDNNL_BUILD_EXAMPLES=OFF
      -DONEDNN_EXPERIMENTAL_GROUPED_MEMORY=ON
      -DONEDNN_BUILD_GRAPH=ON
      -DDNNL_LIBRARY_TYPE=STATIC
      -DDNNL_DPCPP_HOST_COMPILER=${DNNL_HOST_COMPILER} # Use global cxx compiler as host compiler
      -G ${CMAKE_GENERATOR} # Align Generator to Torch
      BUILD_COMMAND ${DNNL_MAKE_COMMAND}
      BUILD_BYPRODUCTS "xpu_mkldnn_proj-prefix/src/xpu_mkldnn_proj-build/src/${DNNL_LIB_NAME}"
````

- EN: This section records project build configuration details.
- CN: 该部分记录项目构建配置细节。

### Lines 66-79 / 第 66-79 行

````cmake
      INSTALL_COMMAND ""
    )

    ExternalProject_Get_Property(xpu_mkldnn_proj SOURCE_DIR BINARY_DIR)
    set(XPU_MKLDNN_LIBRARIES ${BINARY_DIR}/src/${DNNL_LIB_NAME})
    set(XPU_MKLDNN_INCLUDE ${SOURCE_DIR}/include ${BINARY_DIR}/include)
    # This target would be further linked to libtorch_xpu.so.
    # The libtorch_xpu.so would contain Conv&GEMM operators that depend on
    # oneDNN primitive implementations inside libdnnl.a.
    add_library(xpu_mkldnn INTERFACE)
    add_dependencies(xpu_mkldnn xpu_mkldnn_proj)
    target_link_libraries(xpu_mkldnn INTERFACE ${XPU_MKLDNN_LIBRARIES})
    target_include_directories(xpu_mkldnn INTERFACE ${XPU_MKLDNN_INCLUDE})
  endif()
````

- EN: This section defines configure-time variables and search paths.
- CN: 该部分定义配置阶段变量与搜索路径。

### Lines 81-95 / 第 81-95 行

````cmake
  IF(NOT APPLE AND NOT WIN32 AND NOT BUILD_LITE_INTERPRETER)
    MESSAGE("-- Will build oneDNN Graph")
    SET(LLGA_ROOT "${PROJECT_SOURCE_DIR}/third_party/ideep/mkl-dnn")
    SET(BUILD_ONEDNN_GRAPH ON)
    SET(ONEDNN_BUILD_GRAPH ON CACHE BOOL "" FORCE)
  ENDIF(NOT APPLE AND NOT WIN32 AND NOT BUILD_LITE_INTERPRETER)

  IF(EXISTS "${MKLDNN_ROOT}/include/oneapi/dnnl/dnnl_ukernel.hpp")
    IF(CPU_POWER OR CPU_RISCV)
      SET(DNNL_EXPERIMENTAL_UKERNEL OFF CACHE BOOL "" FORCE)
    ELSE()
      MESSAGE("-- Will build oneDNN UKERNEL")
      SET(DNNL_EXPERIMENTAL_UKERNEL ON CACHE BOOL "" FORCE)
    ENDIF()
  ENDIF(EXISTS "${MKLDNN_ROOT}/include/oneapi/dnnl/dnnl_ukernel.hpp")
````

- EN: This section defines configure-time variables and search paths; checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分定义配置阶段变量与搜索路径；在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 97-107 / 第 97-107 行

````cmake
  FIND_PACKAGE(BLAS)
  FIND_PATH(IDEEP_INCLUDE_DIR ideep.hpp PATHS ${IDEEP_ROOT} PATH_SUFFIXES include)
  FIND_PATH(MKLDNN_INCLUDE_DIR dnnl.hpp dnnl.h dnnl_ukernel.hpp dnnl_ukernel.h PATHS ${MKLDNN_ROOT} PATH_SUFFIXES include/oneapi/dnnl)
  IF(NOT MKLDNN_INCLUDE_DIR)
    MESSAGE("MKLDNN_INCLUDE_DIR not found")
    EXECUTE_PROCESS(COMMAND git${CMAKE_EXECUTABLE_SUFFIX} submodule update --init mkl-dnn WORKING_DIRECTORY ${IDEEP_ROOT})
    FIND_PATH(MKLDNN_INCLUDE_DIR dnnl.hpp dnnl.h dnnl_ukernel.hpp dnnl_ukernel.h PATHS ${MKLDNN_ROOT} PATH_SUFFIXES include)
  ENDIF(NOT MKLDNN_INCLUDE_DIR)
  IF(BUILD_ONEDNN_GRAPH)
    FIND_PATH(LLGA_INCLUDE_DIR dnnl_graph.hpp PATHS ${LLGA_ROOT} PATH_SUFFIXES include/oneapi/dnnl)
  ENDIF(BUILD_ONEDNN_GRAPH)
````

- EN: This section probes the host system for headers, libraries, or SDK components; checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分探测主机系统中的头文件、库或 SDK 组件；在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 109-124 / 第 109-124 行

````cmake
  IF(NOT IDEEP_INCLUDE_DIR OR NOT MKLDNN_INCLUDE_DIR)
    MESSAGE(STATUS "MKLDNN source files not found!")
    RETURN()
  ENDIF(NOT IDEEP_INCLUDE_DIR OR NOT MKLDNN_INCLUDE_DIR)
  LIST(APPEND MKLDNN_INCLUDE_DIR ${IDEEP_INCLUDE_DIR})
  IF(BUILD_ONEDNN_GRAPH)
    LIST(APPEND MKLDNN_INCLUDE_DIR ${LLGA_INCLUDE_DIR})
  ENDIF(BUILD_ONEDNN_GRAPH)
  IF(MKL_FOUND)
    ADD_DEFINITIONS(-DIDEEP_USE_MKL)
    # Append to mkldnn dependencies
    LIST(APPEND MKLDNN_LIBRARIES ${MKL_LIBRARIES})
    LIST(APPEND MKLDNN_INCLUDE_DIR ${MKL_INCLUDE_DIR})
  ELSE(MKL_FOUND)
    SET(MKLDNN_USE_MKL "NONE" CACHE STRING "" FORCE)
  ENDIF(MKL_FOUND)
````

- EN: This section defines configure-time variables and search paths; checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分定义配置阶段变量与搜索路径；在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 126-143 / 第 126-143 行

````cmake
  SET(MKL_cmake_included TRUE)
  IF(NOT MKLDNN_CPU_RUNTIME)
    SET(MKLDNN_CPU_RUNTIME "OMP" CACHE STRING "")
  ELSEIF(MKLDNN_CPU_RUNTIME STREQUAL "TBB")
    IF(TARGET TBB::tbb)
      MESSAGE(STATUS "MKL-DNN is using TBB")

      SET(TBB_cmake_included TRUE)
      SET(Threading_cmake_included TRUE)

      SET(DNNL_CPU_THREADING_RUNTIME ${MKLDNN_CPU_RUNTIME})
      INCLUDE_DIRECTORIES(${TBB_INCLUDE_DIR})
      LIST(APPEND EXTRA_SHARED_LIBS TBB::tbb)
    ELSE()
      MESSAGE(FATAL_ERROR "MKLDNN_CPU_RUNTIME is set to TBB but TBB is not used")
    ENDIF()
  ENDIF()
  MESSAGE(STATUS "MKLDNN_CPU_RUNTIME = ${MKLDNN_CPU_RUNTIME}")
````

- EN: This section defines configure-time variables and search paths; checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分定义配置阶段变量与搜索路径；在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 145-159 / 第 145-159 行

````cmake
  SET(MKLDNN_CPU_RUNTIME ${MKLDNN_CPU_RUNTIME} CACHE STRING "" FORCE)
  SET(DNNL_BUILD_TESTS FALSE CACHE BOOL "" FORCE)
  SET(DNNL_BUILD_EXAMPLES FALSE CACHE BOOL "" FORCE)
  SET(DNNL_LIBRARY_TYPE STATIC CACHE STRING "" FORCE)
  SET(DNNL_ENABLE_PRIMITIVE_CACHE TRUE CACHE BOOL "" FORCE)
  SET(DNNL_GRAPH_CPU_RUNTIME ${MKLDNN_CPU_RUNTIME} CACHE STRING "" FORCE)

  IF(BUILD_ONEDNN_GRAPH)
    SET(DNNL_GRAPH_LIBRARY_TYPE STATIC CACHE STRING "" FORCE)
  ENDIF(BUILD_ONEDNN_GRAPH)
  IF(MKLDNN_USE_NATIVE_ARCH)  # Disable HostOpts in MKLDNN unless MKLDNN_USE_NATIVE_ARCH is set.
    SET(DNNL_ARCH_OPT_FLAGS "HostOpts" CACHE STRING "" FORCE)
  ELSE()
    IF(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
      IF(CPU_INTEL)
````

- EN: This section defines configure-time variables and search paths; checks discovery results before exposing them to the wider build.
- CN: 该部分定义配置阶段变量与搜索路径；在对外暴露结果前检查发现状态。

### Lines 160-175 / 第 160-175 行

````cmake
        # Do not specify arch in oneDNN build option, for the portability in older systems
        SET(DNNL_ARCH_OPT_FLAGS "" CACHE STRING "" FORCE)
      ELSEIF(CPU_AARCH64)
        SET(DNNL_ARCH_OPT_FLAGS "-mcpu=generic" CACHE STRING "" FORCE)
      ENDIF()
    ELSE()
      SET(DNNL_ARCH_OPT_FLAGS "" CACHE STRING "" FORCE)
    ENDIF()
  ENDIF()

  ADD_SUBDIRECTORY(${MKLDNN_ROOT})

  IF(NOT TARGET dnnl)
    MESSAGE("Failed to include MKL-DNN target")
    RETURN()
  ENDIF(NOT TARGET dnnl)
````

- EN: This section defines configure-time variables and search paths; checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分定义配置阶段变量与搜索路径；在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 177-186 / 第 177-186 行

````cmake
  IF(NOT APPLE AND CMAKE_COMPILER_IS_GNUCC)
    TARGET_COMPILE_OPTIONS(dnnl PRIVATE -Wno-maybe-uninitialized)
    TARGET_COMPILE_OPTIONS(dnnl PRIVATE -Wno-strict-overflow)
    TARGET_COMPILE_OPTIONS(dnnl PRIVATE -Wno-error=strict-overflow)
  ENDIF(NOT APPLE AND CMAKE_COMPILER_IS_GNUCC)
  LIST(APPEND MKLDNN_LIBRARIES ${MKL_OPENMP_LIBRARY})
  LIST(APPEND MKLDNN_LIBRARIES dnnl)

  SET(MKLDNN_FOUND TRUE)
  MESSAGE(STATUS "Found MKL-DNN: TRUE")
````

- EN: This section defines configure-time variables and search paths; checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分定义配置阶段变量与搜索路径；在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 188-188 / 第 188-188 行

````cmake
ENDIF(NOT MKLDNN_FOUND)
````

- EN: This section records project build configuration details.
- CN: 该部分记录项目构建配置细节。


## Key Concepts / 关键概念
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Configures native build inputs, optional backends, and dependency discovery at configure time.
  CN: 在配置阶段设置原生构建输入、可选后端以及依赖发现逻辑。
- EN: Discovers external headers, libraries, or SDKs and exposes the resulting variables to later build logic.
  CN: 发现外部头文件、库或 SDK，并将结果变量暴露给后续构建逻辑。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。

## Dependencies / 依赖关系
- Find targets / 查找目标: `BLAS`, `IDEEP_INCLUDE_DIR`, `MKLDNN_INCLUDE_DIR`, `LLGA_INCLUDE_DIR`
- Exported variables / 导出变量: `MKLDNN`, `MKL_FOUND`, `CBLAS`, `MKLDNN_FOUND`, `MKLDNN_INCLUDE_DIR`, `MKLDNN_LIBRARIES`, `MKLDNN_USE_NATIVE_ARCH`, `CPU`, `OFF`, `IF`, ...
- Mentioned paths / 提及路径: `/third_party/ideep`, `/third_party/ideep/mkl-dnn`, `/usr/bin/cxx`, `//github.com/uxlfoundation/oneDNN`, `xpu_mkldnn_proj-prefix/src/xpu_mkldnn_proj-build/src/`, `/src/`, `/include`, `/include/oneapi/dnnl/dnnl_ukernel.hpp`, `include/oneapi/dnnl`
