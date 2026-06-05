# Dependencies.cmake — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `cmake/Dependencies.cmake`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines CMake configuration logic that shapes how the PyTorch build is configured. Dependency discovery and platform-specific probing are central themes in the file. Performance measurement or benchmark orchestration is part of its intent. The opening comment frames the file as: "RPATH stuff see https://cmake.org/Wiki/CMake_RPATH_handling."
- **Purpose (CN)**: 定义影响 PyTorch 构建配置方式的 CMake 逻辑。 依赖发现与平台特定探测是该文件的核心主题。 性能测量或基准编排是其意图的一部分。 开头注释将该文件概括为：“RPATH stuff see https://cmake.org/Wiki/CMake_RPATH_handling”。

## Content Analysis / 内容分析

### Lines 1-17 / 第 1-17 行

```cmake
# RPATH stuff
# see https://cmake.org/Wiki/CMake_RPATH_handling
if(APPLE)
  set(CMAKE_MACOSX_RPATH ON)
  set(_rpath_portable_origin "@loader_path")
else()
  set(_rpath_portable_origin $ORIGIN)
endif(APPLE)
# Use separate rpaths during build and install phases
set(CMAKE_SKIP_BUILD_RPATH  FALSE)
# Don't use the install-rpath during the build phase
set(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
set(CMAKE_INSTALL_RPATH "${_rpath_portable_origin}")
# Automatically add all linked folders that are NOT in the build directory to
# the rpath (per library?)
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

```

- **EN:** This chunk introduces sections such as RPATH stuff, see https://cmake.org/Wiki/CMake_RPATH_handling, Use separate rpaths during build and install phases, Don't use the install-rpath during the build phase, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 RPATH stuff、see https://cmake.org/Wiki/CMake_RPATH_handling、Use separate rpaths during build and install phases、Don't use the install-rpath during the build phase 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, else, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、else、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 18-35 / 第 18-35 行

```cmake
 # UBSAN triggers when compiling protobuf, so we need to disable it.
set(UBSAN_FLAG "-fsanitize=undefined")

macro(disable_ubsan)
  if(CMAKE_C_FLAGS MATCHES ${UBSAN_FLAG} OR CMAKE_CXX_FLAGS MATCHES ${UBSAN_FLAG})
    set(CAFFE2_UBSAN_ENABLED ON)
    string(REPLACE ${UBSAN_FLAG} "" CMAKE_C_FLAGS ${CMAKE_C_FLAGS})
    string(REPLACE ${UBSAN_FLAG} "" CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS})
  endif()
endmacro()

macro(enable_ubsan)
  if(CAFFE2_UBSAN_ENABLED)
    set(CMAKE_C_FLAGS "${UBSAN_FLAG} ${CMAKE_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "${UBSAN_FLAG} ${CMAKE_CXX_FLAGS}")
  endif()
endmacro()

```

- **EN:** This chunk introduces sections such as UBSAN triggers when compiling protobuf, so we need to disable it., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 UBSAN triggers when compiling protobuf, so we need to disable it. 等标题组织周边说明或配置。
- **EN:** CMake commands like set, macro, if, string, endif, endmacro drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、macro、if、string、endif、endmacro 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 36-67 / 第 36-67 行

```cmake
# ---[ CUDA
if(USE_CUDA)
  # public/*.cmake uses CAFFE2_USE_*
  set(CAFFE2_USE_CUDA ${USE_CUDA})
  set(CAFFE2_USE_CUDNN ${USE_CUDNN})
  set(CAFFE2_USE_CUSPARSELT ${USE_CUSPARSELT})
  set(CAFFE2_USE_CUFILE ${USE_CUFILE})
  set(CAFFE2_USE_NVRTC ${USE_NVRTC})
  include(${CMAKE_CURRENT_LIST_DIR}/public/cuda.cmake)
  if(CAFFE2_USE_CUDA)
    # A helper variable recording the list of Caffe2 dependent libraries
    # torch::cudart is dealt with separately, due to CUDA_ADD_LIBRARY
    # design reason (it adds CUDA_LIBRARIES itself).
    set(Caffe2_PUBLIC_CUDA_DEPENDENCY_LIBS )
    list(APPEND Caffe2_CUDA_DEPENDENCY_LIBS caffe2::curand caffe2::cufft caffe2::cublas)
    if(CAFFE2_USE_CUDNN)
      if(NOT CAFFE2_USE_NVRTC)
        message(FATAL_ERROR
          "USE_CUDNN requires USE_NVRTC (required by cudnn_frontend 1.21+). "
          "Please set -DUSE_NVRTC=ON or disable cuDNN with -DUSE_CUDNN=OFF.")
      endif()
      list(APPEND Caffe2_CUDA_DEPENDENCY_LIBS torch::cudnn)
      list(APPEND Caffe2_CUDA_DEPENDENCY_LIBS caffe2::nvrtc_runtime)
    else()
      caffe2_update_option(USE_CUDNN OFF)
    endif()
    if(NOT CAFFE2_USE_NVRTC)
      caffe2_update_option(USE_NVRTC OFF)
    endif()
    if(CAFFE2_USE_CUSPARSELT)
      list(APPEND Caffe2_CUDA_DEPENDENCY_LIBS torch::cusparselt)
    else()
```

- **EN:** This chunk introduces sections such as ---[ CUDA, public/*.cmake uses CAFFE2_USE_*, A helper variable recording the list of Caffe2 dependent libraries, torch::cudart is dealt with separately, due to CUDA_ADD_LIBRARY, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ CUDA、public/*.cmake uses CAFFE2_USE_*、A helper variable recording the list of Caffe2 dependent libraries、torch::cudart is dealt with separately, due to CUDA_ADD_LIBRARY 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, include, list, message, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、include、list、message、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 68-96 / 第 68-96 行

```cmake
      caffe2_update_option(USE_CUSPARSELT OFF)
    endif()
    if(CAFFE2_USE_CUFILE)
      list(APPEND Caffe2_CUDA_DEPENDENCY_LIBS torch::cufile)
    endif()
    find_program(SCCACHE_EXECUTABLE sccache)
    if(SCCACHE_EXECUTABLE)
      # Using RSP/--options-file renders output noncacheable by sccache
      # as they fall under `multiple input files` non-cacheable rule
      set(CMAKE_CUDA_USE_RESPONSE_FILE_FOR_INCLUDES 0)
      set(CMAKE_CUDA_USE_RESPONSE_FILE_FOR_LIBRARIES 0)
      set(CMAKE_CUDA_USE_RESPONSE_FILE_FOR_OBJECTS 0)
    endif()
  else()
    message(WARNING
      "Not compiling with CUDA. Suppress this warning with "
      "-DUSE_CUDA=OFF.")
    caffe2_update_option(USE_CUDA OFF)
    caffe2_update_option(USE_CUDNN OFF)
    caffe2_update_option(USE_CUSPARSELT OFF)
    caffe2_update_option(USE_NVRTC OFF)
    set(CAFFE2_USE_CUDA OFF)
    set(CAFFE2_USE_CUDNN OFF)
    set(CAFFE2_USE_CUSPARSELT OFF)
    set(CAFFE2_USE_CUFILE OFF)
    set(CAFFE2_USE_NVRTC OFF)
  endif()
endif()

```

- **EN:** This chunk introduces sections such as Using RSP/--options-file renders output noncacheable by sccache, as they fall under `multiple input files` non-cacheable rule, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Using RSP/--options-file renders output noncacheable by sccache、as they fall under `multiple input files` non-cacheable rule 等标题组织周边说明或配置。
- **EN:** CMake commands like caffe2_update_option, endif, if, list, find_program, set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 caffe2_update_option、endif、if、list、find_program、set 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 97-116 / 第 97-116 行

```cmake
# ---[ XPU
if(USE_XPU)
  include(${CMAKE_CURRENT_LIST_DIR}/public/xpu.cmake)
  if(NOT PYTORCH_FOUND_XPU)
    message(WARNING "Not compiling with XPU. Could NOT find SYCL. "
    "Suppress this warning with -DUSE_XPU=OFF.")
    caffe2_update_option(USE_XPU OFF)
  endif()
  foreach(flag ${XPU_HOST_CXX_FLAGS})
    add_definitions(${flag})
  endforeach()
endif()

# ---[ Custom Protobuf
if(CAFFE2_CMAKE_BUILDING_WITH_MAIN_REPO AND NOT INTERN_BUILD_MOBILE)
  disable_ubsan()
  include(${CMAKE_CURRENT_LIST_DIR}/ProtoBuf.cmake)
  enable_ubsan()
endif()

```

- **EN:** This chunk introduces sections such as ---[ XPU, ---[ Custom Protobuf, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ XPU、---[ Custom Protobuf 等标题组织周边说明或配置。
- **EN:** CMake commands like if, include, message, caffe2_update_option, endif, foreach drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、include、message、caffe2_update_option、endif、foreach 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 117-147 / 第 117-147 行

```cmake
if(USE_ASAN OR USE_LSAN OR USE_TSAN)
  find_package(Sanitizer REQUIRED)
  if(USE_ASAN)
    if(TARGET Sanitizer::address)
      list(APPEND Caffe2_DEPENDENCY_LIBS Sanitizer::address)
    else()
      message(WARNING "ASAN not found. Suppress this warning with -DUSE_ASAN=OFF.")
      caffe2_update_option(USE_ASAN OFF)
    endif()
    if(TARGET Sanitizer::undefined)
      list(APPEND Caffe2_DEPENDENCY_LIBS Sanitizer::undefined)
    endif()
  endif()
  if(USE_LSAN)
    if(TARGET Sanitizer::leak)
      list(APPEND Caffe2_DEPENDENCY_LIBS Sanitizer::leak)
    else()
      message(WARNING "LSAN not found. Suppress this warning with -DUSE_LSAN=OFF.")
      caffe2_update_option(USE_LSAN OFF)
    endif()
  endif()
  if(USE_TSAN)
    if(TARGET Sanitizer::thread)
      list(APPEND Caffe2_DEPENDENCY_LIBS Sanitizer::thread)
    else()
      message(WARNING "TSAN not found. Suppress this warning with -DUSE_TSAN=OFF.")
      caffe2_update_option(USE_TSAN OFF)
    endif()
  endif()
endif()

```

- **EN:** CMake commands like if, find_package, list, else, message, caffe2_update_option drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、find_package、list、else、message、caffe2_update_option 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 148-163 / 第 148-163 行

```cmake
# ---[ Threads
find_package(Threads REQUIRED)
if(TARGET Threads::Threads)
  list(APPEND Caffe2_DEPENDENCY_LIBS Threads::Threads)
else()
  message(FATAL_ERROR
      "Cannot find threading library. PyTorch requires Threads to compile.")
endif()

# ---[ protobuf
if(CAFFE2_CMAKE_BUILDING_WITH_MAIN_REPO)
  if(USE_LITE_PROTO)
    set(CAFFE2_USE_LITE_PROTO 1)
  endif()
endif()

```

- **EN:** This chunk introduces sections such as ---[ Threads, ---[ protobuf, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ Threads、---[ protobuf 等标题组织周边说明或配置。
- **EN:** CMake commands like find_package, if, list, else, message, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 find_package、if、list、else、message、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 164-182 / 第 164-182 行

```cmake
# ---[ BLAS

set(AT_MKLDNN_ACL_ENABLED 0)
set(AT_MKLDNN_ENABLED 0)
set(AT_MKL_ENABLED 0)
set(AT_KLEIDIAI_ENABLED 0)
set(AT_USE_EIGEN_SPARSE 0)
# setting default preferred BLAS options if not already present.
if(NOT INTERN_BUILD_MOBILE)
  set(BLAS "MKL" CACHE STRING "Selected BLAS library")
else()
  set(BLAS "Eigen" CACHE STRING "Selected BLAS library")
  set(AT_MKLDNN_ENABLED 0)
  set(AT_MKL_ENABLED 0)
endif()
set_property(CACHE BLAS PROPERTY STRINGS "ATLAS;BLIS;Eigen;FLAME;Generic;MKL;OpenBLAS;vecLib;APL")
message(STATUS "Trying to find preferred BLAS backend of choice: " ${BLAS})
set(BLAS_CHECK_F2C 0)

```

- **EN:** This chunk introduces sections such as ---[ BLAS, setting default preferred BLAS options if not already present., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ BLAS、setting default preferred BLAS options if not already present. 等标题组织周边说明或配置。
- **EN:** CMake commands like set, if, else, endif, set_property, message drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、if、else、endif、set_property、message 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 183-214 / 第 183-214 行

```cmake
if(BLAS STREQUAL "Eigen")
  # Eigen is header-only and we do not have any dependent libraries
  set(CAFFE2_USE_EIGEN_FOR_BLAS ON)
elseif(BLAS STREQUAL "ATLAS")
  find_package(Atlas REQUIRED)
  include_directories(SYSTEM ${ATLAS_INCLUDE_DIRS})
  list(APPEND Caffe2_DEPENDENCY_LIBS ${ATLAS_LIBRARIES})
  list(APPEND Caffe2_DEPENDENCY_LIBS cblas)
  set(BLAS_INFO "atlas")
  set(BLAS_FOUND 1)
  set(BLAS_LIBRARIES ${ATLAS_LIBRARIES} cblas)
  set(BLAS_CHECK_F2C 1)
elseif(BLAS STREQUAL "OpenBLAS")
  find_package(OpenBLAS REQUIRED)
  include_directories(SYSTEM ${OpenBLAS_INCLUDE_DIR})
  list(APPEND Caffe2_DEPENDENCY_LIBS ${OpenBLAS_LIB})
  set(BLAS_INFO "open")
  set(BLAS_FOUND 1)
  set(BLAS_LIBRARIES ${OpenBLAS_LIB})
  set(BLAS_CHECK_F2C 1)
elseif(BLAS STREQUAL "BLIS")
  find_package(BLIS REQUIRED)
  include_directories(SYSTEM ${BLIS_INCLUDE_DIR})
  list(APPEND Caffe2_DEPENDENCY_LIBS ${BLIS_LIB})
  set(BLAS_CHECK_F2C 1)
elseif(BLAS STREQUAL "MKL")
  if(BLAS_SET_BY_USER)
    find_package(MKL REQUIRED)
  else()
    find_package(MKL QUIET)
  endif()
  include(${CMAKE_CURRENT_LIST_DIR}/public/mkl.cmake)
```

- **EN:** This chunk introduces sections such as Eigen is header-only and we do not have any dependent libraries, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Eigen is header-only and we do not have any dependent libraries 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, elseif, find_package, include_directories, list drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、elseif、find_package、include_directories、list 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 215-246 / 第 215-246 行

```cmake
  if(MKL_FOUND)
    message(STATUS "MKL libraries: ${MKL_LIBRARIES}")
    message(STATUS "MKL include directory: ${MKL_INCLUDE_DIR}")
    message(STATUS "MKL OpenMP type: ${MKL_OPENMP_TYPE}")
    message(STATUS "MKL OpenMP library: ${MKL_OPENMP_LIBRARY}")
    include_directories(AFTER SYSTEM ${MKL_INCLUDE_DIR})
    list(APPEND Caffe2_PUBLIC_DEPENDENCY_LIBS caffe2::mkl)
    set(CAFFE2_USE_MKL ON)
    set(BLAS_INFO "mkl")
    set(BLAS_FOUND 1)
    set(BLAS_LIBRARIES ${MKL_LIBRARIES})
  else()
    message(WARNING "MKL could not be found. Defaulting to Eigen")
    set(CAFFE2_USE_EIGEN_FOR_BLAS ON)
    set(CAFFE2_USE_MKL OFF)
  endif()
elseif(BLAS STREQUAL "NVPL")
  find_package(NVPL_BLAS REQUIRED)
  list(APPEND Caffe2_DEPENDENCY_LIBS nvpl::blas_lp64_omp)
  set(BLAS_INFO "nvpl")
  set(BLAS_FOUND 1)
  set(BLAS_USE_CBLAS_DOT TRUE)
  set(BLAS_CHECK_F2C 1)
elseif(BLAS STREQUAL "vecLib")
  find_package(vecLib REQUIRED)
  include_directories(SYSTEM ${vecLib_INCLUDE_DIR})
  list(APPEND Caffe2_DEPENDENCY_LIBS ${vecLib_LINKER_LIBS})
  set(BLAS_INFO "veclib")
  set(BLAS_FOUND 1)
  set(BLAS_LIBRARIES ${vecLib_LINKER_LIBS})
elseif(BLAS STREQUAL "FlexiBLAS")
  find_package(FlexiBLAS REQUIRED)
```

- **EN:** CMake commands like if, message, include_directories, list, set, else drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、message、include_directories、list、set、else 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 247-274 / 第 247-274 行

```cmake
  include_directories(SYSTEM ${FlexiBLAS_INCLUDE_DIR})
  list(APPEND Caffe2_DEPENDENCY_LIBS ${FlexiBLAS_LIB})
  set(BLAS_CHECK_F2C 1)
elseif(BLAS STREQUAL "APL")
  find_package(APL REQUIRED)
  include_directories(SYSTEM ${APL_INCLUDE_DIR})
  set(BLAS_INFO "apl")
  set(BLAS_FOUND 1)
  set(BLAS_LIBRARIES ${APL_LIBRARIES})
  set(BLAS_CHECK_F2C 1)
elseif(BLAS STREQUAL "Generic")
  # On Debian family, the CBLAS ABIs have been merged into libblas.so
  if(ENV{GENERIC_BLAS_LIBRARIES} STREQUAL "")
    set(GENERIC_BLAS "blas")
  else()
    set(GENERIC_BLAS $ENV{GENERIC_BLAS_LIBRARIES})
  endif()
  find_library(BLAS_LIBRARIES NAMES ${GENERIC_BLAS})
  message("-- Using BLAS: ${BLAS_LIBRARIES}")
  list(APPEND Caffe2_DEPENDENCY_LIBS ${BLAS_LIBRARIES})
  set(GENERIC_BLAS_FOUND TRUE)
  set(BLAS_INFO "generic")
  set(BLAS_FOUND 1)
  set(BLAS_CHECK_F2C 1)
else()
  message(FATAL_ERROR "Unrecognized BLAS option: " ${BLAS})
endif()

```

- **EN:** This chunk introduces sections such as On Debian family, the CBLAS ABIs have been merged into libblas.so, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 On Debian family, the CBLAS ABIs have been merged into libblas.so 等标题组织周边说明或配置。
- **EN:** CMake commands like include_directories, list, set, elseif, find_package, if drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 include_directories、list、set、elseif、find_package、if 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 275-299 / 第 275-299 行

```cmake
# Determine if blas was compiled with the f2c conventions
if(BLAS_LIBRARIES AND BLAS_CHECK_F2C)
  include(cmake/BLAS_ABI.cmake)
endif()

if(USE_EIGEN_SPARSE AND BLAS_INFO STREQUAL "mkl")
  message(WARNING "Disabling USE_EIGEN_SPARSE because MKL is enabled")
  set(USE_EIGEN_SPARSE OFF)
endif()

if(USE_EIGEN_SPARSE)
  set(AT_USE_EIGEN_SPARSE 1)
endif()

if(NOT INTERN_BUILD_MOBILE)
  set(AT_MKL_SEQUENTIAL 0)
  set(USE_BLAS 1)
  if(NOT (ATLAS_FOUND OR BLIS_FOUND OR GENERIC_BLAS_FOUND OR MKL_FOUND OR OpenBLAS_FOUND OR VECLIB_FOUND OR FlexiBLAS_FOUND OR NVPL_BLAS_FOUND OR APL_FOUND))
    message(WARNING "Preferred BLAS (" ${BLAS} ") cannot be found, now searching for a general BLAS library")
    find_package(BLAS)
    if(NOT BLAS_FOUND)
      set(USE_BLAS 0)
    endif()
  endif()

```

- **EN:** This chunk introduces sections such as Determine if blas was compiled with the f2c conventions, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Determine if blas was compiled with the f2c conventions 等标题组织周边说明或配置。
- **EN:** CMake commands like if, include, endif, message, set, find_package drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、include、endif、message、set、find_package 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 300-322 / 第 300-322 行

```cmake
  if(MKL_FOUND)
    if("${MKL_THREADING}" STREQUAL "SEQ")
      set(AT_MKL_SEQUENTIAL 1)
    endif()
    set(AT_MKL_ENABLED 1)
  endif()
elseif(INTERN_USE_EIGEN_BLAS)
  # Eigen BLAS for Mobile
  set(USE_BLAS 1)
  include(${CMAKE_CURRENT_LIST_DIR}/External/EigenBLAS.cmake)
  list(APPEND Caffe2_DEPENDENCY_LIBS eigen_blas)
endif()

# --- [ PocketFFT
set(AT_POCKETFFT_ENABLED 0)
if(NOT AT_MKL_ENABLED)
  set(POCKETFFT_INCLUDE_DIR "${Torch_SOURCE_DIR}/third_party/pocketfft/")
  if(NOT EXISTS "${POCKETFFT_INCLUDE_DIR}")
    message(FATAL_ERROR "pocketfft directory not found, expected ${POCKETFFT_INCLUDE_DIR}")
  elseif(NOT EXISTS "${POCKETFFT_INCLUDE_DIR}/pocketfft_hdronly.h")
    message(FATAL_ERROR "pocketfft headers not found in ${POCKETFFT_INCLUDE_DIR}")
  endif()

```

- **EN:** This chunk introduces sections such as Eigen BLAS for Mobile, --- [ PocketFFT, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Eigen BLAS for Mobile、--- [ PocketFFT 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, endif, elseif, include, list drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、endif、elseif、include、list 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 323-338 / 第 323-338 行

```cmake
  set(AT_POCKETFFT_ENABLED 1)
  message(STATUS "Using pocketfft in directory: ${POCKETFFT_INCLUDE_DIR}")
endif()

# ---[ Dependencies
# NNPACK and family (QNNPACK, PYTORCH_QNNPACK, and XNNPACK) can download and
# compile their dependencies in isolation as part of their build.  These dependencies
# are then linked statically with PyTorch.  To avoid the possibility of a version
# mismatch between these shared dependencies, explicitly declare our intent to these
# libraries that we are interested in using the exact same source dependencies for all.

if(USE_NNPACK OR USE_PYTORCH_QNNPACK OR USE_XNNPACK)
  set(DISABLE_NNPACK_AND_FAMILY OFF)

  # Sanity checks - Can we actually build NNPACK and family given the configuration provided?
  # Disable them and warn the user if not.
```

- **EN:** This chunk introduces sections such as ---[ Dependencies, NNPACK and family (QNNPACK, PYTORCH_QNNPACK, and XNNPACK) can download and, compile their dependencies in isolation as part of their build.  These dependencies, are then linked statically with PyTorch.  To avoid the possibility of a version, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ Dependencies、NNPACK and family (QNNPACK, PYTORCH_QNNPACK, and XNNPACK) can download and、compile their dependencies in isolation as part of their build.  These dependencies、are then linked statically with PyTorch.  To avoid the possibility of a version 等标题组织周边说明或配置。
- **EN:** CMake commands like set, message, endif, if drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、message、endif、if 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 339-370 / 第 339-370 行

```cmake

  if(IOS)
    list(LENGTH IOS_ARCH IOS_ARCH_COUNT)
    if(IOS_ARCH_COUNT GREATER 1)
      message(WARNING
        "Multi-architecture (${IOS_ARCH}) builds are not supported in {Q/X}NNPACK. "
        "Specify a single architecture in IOS_ARCH and re-configure, or "
        "turn this warning off by USE_{Q/X}NNPACK=OFF.")
      set(DISABLE_NNPACK_AND_FAMILY ON)
    endif()
    if(NOT IOS_ARCH MATCHES "^(i386|x86_64|armv7.*|arm64.*)$")
      message(WARNING
        "Target architecture \"${IOS_ARCH}\" is not supported in {Q/X}NNPACK. "
        "Supported architectures are x86, x86-64, ARM, and ARM64. "
        "Turn this warning off by USE_{Q/X}NNPACK=OFF.")
      set(DISABLE_NNPACK_AND_FAMILY ON)
    endif()
  else()
    if(NOT IOS AND NOT (CMAKE_SYSTEM_NAME MATCHES "^(Android|Linux|Darwin|Windows)$"))
      message(WARNING
        "Target platform \"${CMAKE_SYSTEM_NAME}\" is not supported in {Q/X}NNPACK. "
        "Supported platforms are Android, iOS, Linux, and macOS. "
        "Turn this warning off by USE_{Q/X}NNPACK=OFF.")
      set(DISABLE_NNPACK_AND_FAMILY ON)
    endif()
    if(NOT IOS AND NOT (CMAKE_SYSTEM_PROCESSOR MATCHES "^(i686|AMD64|x86_64|armv[0-9].*|arm64|aarch64)$"))
      message(WARNING
        "Target architecture \"${CMAKE_SYSTEM_PROCESSOR}\" is not supported in {Q/X}NNPACK. "
        "Supported architectures are x86, x86-64, ARM, and ARM64. "
        "Turn this warning off by USE_{Q/X}NNPACK=OFF.")
      set(DISABLE_NNPACK_AND_FAMILY ON)
    endif()
```

- **EN:** CMake commands like if, list, message, set, endif, else drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、list、message、set、endif、else 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 371-402 / 第 371-402 行

```cmake
  endif()

  if(DISABLE_NNPACK_AND_FAMILY)
    caffe2_update_option(USE_NNPACK OFF)
    caffe2_update_option(USE_PYTORCH_QNNPACK OFF)
    caffe2_update_option(USE_XNNPACK OFF)
  else()
    # Disable unsupported NNPack combinations with MSVC
    if(MSVC)
      caffe2_update_option(USE_NNPACK OFF)
      caffe2_update_option(USE_PYTORCH_QNNPACK OFF)
    endif()

    set(CAFFE2_THIRD_PARTY_ROOT "${PROJECT_SOURCE_DIR}/third_party")

    if(NOT DEFINED CPUINFO_SOURCE_DIR)
      set(CPUINFO_SOURCE_DIR "${CAFFE2_THIRD_PARTY_ROOT}/cpuinfo" CACHE STRING "cpuinfo source directory")
    endif()
    if(NOT DEFINED FP16_SOURCE_DIR)
      set(FP16_SOURCE_DIR "${CAFFE2_THIRD_PARTY_ROOT}/FP16" CACHE STRING "FP16 source directory")
    endif()
    if(NOT DEFINED FXDIV_SOURCE_DIR)
      set(FXDIV_SOURCE_DIR "${CAFFE2_THIRD_PARTY_ROOT}/FXdiv" CACHE STRING "FXdiv source directory")
    endif()
    if(NOT DEFINED PSIMD_SOURCE_DIR)
      set(PSIMD_SOURCE_DIR "${CAFFE2_THIRD_PARTY_ROOT}/psimd" CACHE STRING "PSimd source directory")
    endif()
    if(NOT DEFINED PTHREADPOOL_SOURCE_DIR)
      set(PTHREADPOOL_SOURCE_DIR "${CAFFE2_THIRD_PARTY_ROOT}/pthreadpool" CACHE STRING "pthreadpool source directory")
    endif()
  endif()
else()
```

- **EN:** This chunk introduces sections such as Disable unsupported NNPack combinations with MSVC, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Disable unsupported NNPack combinations with MSVC 等标题组织周边说明或配置。
- **EN:** CMake commands like endif, if, caffe2_update_option, else, set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 endif、if、caffe2_update_option、else、set 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 403-432 / 第 403-432 行

```cmake
  set(DISABLE_NNPACK_AND_FAMILY ON)
endif()

set(CONFU_DEPENDENCIES_SOURCE_DIR ${PROJECT_BINARY_DIR}/confu-srcs
  CACHE PATH "Confu-style dependencies source directory")
set(CONFU_DEPENDENCIES_BINARY_DIR ${PROJECT_BINARY_DIR}/confu-deps
  CACHE PATH "Confu-style dependencies binary directory")

# ---[ pthreadpool
# Only add a dependency on pthreadpool if we are on a mobile build
# or are building any of the libraries in the {Q/X}NNPACK family.
if(INTERN_BUILD_MOBILE OR NOT DISABLE_NNPACK_AND_FAMILY)
  set(USE_PTHREADPOOL ON CACHE BOOL "" FORCE)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DUSE_PTHREADPOOL")

  if(NOT TARGET pthreadpool)
    if(USE_SYSTEM_PTHREADPOOL)
      add_library(pthreadpool SHARED IMPORTED)
      find_library(PTHREADPOOL_LIBRARY pthreadpool)
      set_property(TARGET pthreadpool PROPERTY IMPORTED_LOCATION "${PTHREADPOOL_LIBRARY}")
      if(NOT PTHREADPOOL_LIBRARY)
        message(FATAL_ERROR "Cannot find pthreadpool")
      endif()
      message("-- Found pthreadpool: ${PTHREADPOOL_LIBRARY}")
    else()
      if(NOT DEFINED PTHREADPOOL_SOURCE_DIR)
        set(CAFFE2_THIRD_PARTY_ROOT "${PROJECT_SOURCE_DIR}/third_party")
        set(PTHREADPOOL_SOURCE_DIR "${CAFFE2_THIRD_PARTY_ROOT}/pthreadpool" CACHE STRING "pthreadpool source directory")
      endif()

```

- **EN:** This chunk introduces sections such as ---[ pthreadpool, Only add a dependency on pthreadpool if we are on a mobile build, or are building any of the libraries in the {Q/X}NNPACK family., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ pthreadpool、Only add a dependency on pthreadpool if we are on a mobile build、or are building any of the libraries in the {Q/X}NNPACK family. 等标题组织周边说明或配置。
- **EN:** CMake commands like set, endif, if, add_library, find_library, set_property drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、endif、if、add_library、find_library、set_property 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 433-448 / 第 433-448 行

```cmake
      set(PTHREADPOOL_BUILD_TESTS OFF CACHE BOOL "")
      set(PTHREADPOOL_BUILD_BENCHMARKS OFF CACHE BOOL "")
      set(PTHREADPOOL_LIBRARY_TYPE "static" CACHE STRING "")
      set(PTHREADPOOL_ALLOW_DEPRECATED_API ON CACHE BOOL "")
      add_subdirectory(
        "${PTHREADPOOL_SOURCE_DIR}"
        "${CONFU_DEPENDENCIES_BINARY_DIR}/pthreadpool")
      set_property(TARGET pthreadpool PROPERTY POSITION_INDEPENDENT_CODE ON)
    endif()

    list(APPEND Caffe2_DEPENDENCY_LIBS pthreadpool)
  endif()
else()
  set(USE_PTHREADPOOL OFF CACHE BOOL "" FORCE)
endif()

```

- **EN:** CMake commands like set, add_subdirectory, set_property, endif, list, else drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、add_subdirectory、set_property、endif、list、else 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 449-464 / 第 449-464 行

```cmake
if(NOT CMAKE_SYSTEM_PROCESSOR MATCHES "^(s390x|ppc64le)$")
  # ---[ Caffe2 uses cpuinfo library in the thread pool
  # ---[ But it doesn't support s390x/powerpc and thus not used on s390x/powerpc
  if(NOT TARGET cpuinfo AND USE_SYSTEM_CPUINFO)
    add_library(cpuinfo SHARED IMPORTED)
    find_library(CPUINFO_LIBRARY cpuinfo)
    if(NOT CPUINFO_LIBRARY)
      message(FATAL_ERROR "Cannot find cpuinfo")
    endif()
    message("Found cpuinfo: ${CPUINFO_LIBRARY}")
    set_target_properties(cpuinfo PROPERTIES IMPORTED_LOCATION "${CPUINFO_LIBRARY}")
  elseif(NOT TARGET cpuinfo)
    if(NOT DEFINED CPUINFO_SOURCE_DIR)
      set(CPUINFO_SOURCE_DIR "${CMAKE_CURRENT_LIST_DIR}/../third_party/cpuinfo" CACHE STRING "cpuinfo source directory")
    endif()

```

- **EN:** This chunk introduces sections such as ---[ Caffe2 uses cpuinfo library in the thread pool, ---[ But it doesn't support s390x/powerpc and thus not used on s390x/powerpc, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ Caffe2 uses cpuinfo library in the thread pool、---[ But it doesn't support s390x/powerpc and thus not used on s390x/powerpc 等标题组织周边说明或配置。
- **EN:** CMake commands like if, add_library, find_library, message, endif, set_target_properties drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、add_library、find_library、message、endif、set_target_properties 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 465-481 / 第 465-481 行

```cmake
    set(CPUINFO_BUILD_TOOLS OFF CACHE BOOL "")
    set(CPUINFO_BUILD_UNIT_TESTS OFF CACHE BOOL "")
    set(CPUINFO_BUILD_MOCK_TESTS OFF CACHE BOOL "")
    set(CPUINFO_BUILD_BENCHMARKS OFF CACHE BOOL "")
    set(CPUINFO_LIBRARY_TYPE "static" CACHE STRING "")
    set(CPUINFO_LOG_LEVEL "error" CACHE STRING "")
    if(MSVC)
      if(CAFFE2_USE_MSVC_STATIC_RUNTIME)
        set(CPUINFO_RUNTIME_TYPE "static" CACHE STRING "")
      else()
        set(CPUINFO_RUNTIME_TYPE "shared" CACHE STRING "")
      endif()
    endif()
    add_subdirectory(
      "${CPUINFO_SOURCE_DIR}"
      "${CONFU_DEPENDENCIES_BINARY_DIR}/cpuinfo")
    # We build static version of cpuinfo but link
```

- **EN:** This chunk introduces sections such as We build static version of cpuinfo but link, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 We build static version of cpuinfo but link 等标题组织周边说明或配置。
- **EN:** CMake commands like set, if, else, endif, add_subdirectory drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、if、else、endif、add_subdirectory 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 482-503 / 第 482-503 行

```cmake
    # them into a shared library for Caffe2, so they need PIC.
    set_property(TARGET cpuinfo PROPERTY POSITION_INDEPENDENT_CODE ON)
  endif()
  list(APPEND Caffe2_DEPENDENCY_LIBS cpuinfo)
endif()


# ---[ PYTORCH_QNNPACK
set(CAFFE2_THIRD_PARTY_ROOT "${PROJECT_SOURCE_DIR}/third_party")
if(USE_PYTORCH_QNNPACK)
    if(NOT DEFINED PYTORCH_QNNPACK_SOURCE_DIR)
      set(PYTORCH_QNNPACK_SOURCE_DIR "${PROJECT_SOURCE_DIR}/aten/src/ATen/native/quantized/cpu/qnnpack" CACHE STRING "QNNPACK source directory")
    endif()

    if(NOT TARGET pytorch_qnnpack)
      set(PYTORCH_QNNPACK_BUILD_TESTS OFF CACHE BOOL "")
      set(PYTORCH_QNNPACK_BUILD_BENCHMARKS OFF CACHE BOOL "")
      set(PYTORCH_QNNPACK_LIBRARY_TYPE "static" CACHE STRING "")
      add_subdirectory(
        "${PYTORCH_QNNPACK_SOURCE_DIR}"
        "${CONFU_DEPENDENCIES_BINARY_DIR}/pytorch_qnnpack")
      # We build static versions of QNNPACK and pthreadpool but link
```

- **EN:** This chunk introduces sections such as them into a shared library for Caffe2, so they need PIC., ---[ PYTORCH_QNNPACK, We build static versions of QNNPACK and pthreadpool but link, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 them into a shared library for Caffe2, so they need PIC.、---[ PYTORCH_QNNPACK、We build static versions of QNNPACK and pthreadpool but link 等标题组织周边说明或配置。
- **EN:** CMake commands like set_property, endif, list, set, if, add_subdirectory drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set_property、endif、list、set、if、add_subdirectory 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 504-519 / 第 504-519 行

```cmake
      # them into a shared library for Caffe2, so they need PIC.
      set_property(TARGET pytorch_qnnpack PROPERTY POSITION_INDEPENDENT_CODE ON)
      set_property(TARGET cpuinfo PROPERTY POSITION_INDEPENDENT_CODE ON)
      # QNNPACK depends on gemmlowp headers
      target_include_directories(pytorch_qnnpack PRIVATE "${CAFFE2_THIRD_PARTY_ROOT}/gemmlowp")
    endif()

    list(APPEND Caffe2_DEPENDENCY_LIBS pytorch_qnnpack)
endif()

# ---[ NNPACK
if(USE_NNPACK)
  include(${CMAKE_CURRENT_LIST_DIR}/External/nnpack.cmake)
  if(NNPACK_FOUND)
    if(TARGET nnpack)
      # ---[ NNPACK is being built together with Caffe2: explicitly specify dependency
```

- **EN:** This chunk introduces sections such as them into a shared library for Caffe2, so they need PIC., QNNPACK depends on gemmlowp headers, ---[ NNPACK, ---[ NNPACK is being built together with Caffe2: explicitly specify dependency, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 them into a shared library for Caffe2, so they need PIC.、QNNPACK depends on gemmlowp headers、---[ NNPACK、---[ NNPACK is being built together with Caffe2: explicitly specify dependency 等标题组织周边说明或配置。
- **EN:** CMake commands like set_property, target_include_directories, endif, list, if, include drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set_property、target_include_directories、endif、list、if、include 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 520-536 / 第 520-536 行

```cmake
      list(APPEND Caffe2_DEPENDENCY_LIBS nnpack)
    else()
      include_directories(SYSTEM ${NNPACK_INCLUDE_DIRS})
      list(APPEND Caffe2_DEPENDENCY_LIBS ${NNPACK_LIBRARIES})
    endif()
  else()
    message(WARNING "Not compiling with NNPACK. Suppress this warning with -DUSE_NNPACK=OFF")
    caffe2_update_option(USE_NNPACK OFF)
  endif()
endif()

# ---[ XNNPACK
if(USE_XNNPACK AND NOT USE_SYSTEM_XNNPACK)
  if(NOT DEFINED XNNPACK_SOURCE_DIR)
    set(XNNPACK_SOURCE_DIR "${CAFFE2_THIRD_PARTY_ROOT}/XNNPACK" CACHE STRING "XNNPACK source directory")
  endif()

```

- **EN:** This chunk introduces sections such as ---[ XNNPACK, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ XNNPACK 等标题组织周边说明或配置。
- **EN:** CMake commands like list, else, include_directories, endif, message, caffe2_update_option drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 list、else、include_directories、endif、message、caffe2_update_option 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 537-553 / 第 537-553 行

```cmake
  if(NOT DEFINED XNNPACK_INCLUDE_DIR)
    set(XNNPACK_INCLUDE_DIR "${XNNPACK_SOURCE_DIR}/include" CACHE STRING "XNNPACK include directory")
  endif()

  if(NOT TARGET XNNPACK OR NOT TARGET microkernels-prod)
    set(XNNPACK_LIBRARY_TYPE "static" CACHE STRING "")
    set(XNNPACK_BUILD_BENCHMARKS OFF CACHE BOOL "")
    set(XNNPACK_BUILD_TESTS OFF CACHE BOOL "")

    # Disable ARM BF16 and FP16 vector for now; unused and causes build failures because
    # these new ISA features may not be supported on older compilers
    set(XNNPACK_ENABLE_ARM_BF16 OFF CACHE BOOL "")

    # Disable AVXVNNI for now, older clang versions seem not to support it
    # (clang 12 is where avx-vnni support is added)
    set(XNNPACK_ENABLE_AVXVNNI OFF CACHE BOOL "")

```

- **EN:** This chunk introduces sections such as Disable ARM BF16 and FP16 vector for now; unused and causes build failures because, these new ISA features may not be supported on older compilers, Disable AVXVNNI for now, older clang versions seem not to support it, (clang 12 is where avx-vnni support is added), which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Disable ARM BF16 and FP16 vector for now; unused and causes build failures because、these new ISA features may not be supported on older compilers、Disable AVXVNNI for now, older clang versions seem not to support it、(clang 12 is where avx-vnni support is added) 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 554-573 / 第 554-573 行

```cmake
    # Disable I8MM For CI since clang 9 does not support neon i8mm.
    set(XNNPACK_ENABLE_ARM_I8MM OFF CACHE BOOL "")

    # Disable avxvnni int8
    set(XNNPACK_ENABLE_AVXVNNIINT8 OFF CACHE BOOL "")

    # Older MSVC versions don't support AVX512FP. TODO Minimum version support?
    IF(CMAKE_C_COMPILER_ID STREQUAL "MSVC")
      set(XNNPACK_ENABLE_AVX512FP16  OFF CACHE BOOL "")
    ENDIF()

    # Conditionally disable AVX512AMX, as it requires Clang 11 or later. Note that
    # XNNPACK does conditionally compile this based on GCC version. Once it also does
    # so based on Clang version, this logic can be removed.
    IF(CMAKE_C_COMPILER_ID STREQUAL "Clang")
      IF(CMAKE_C_COMPILER_VERSION VERSION_LESS "11")
        set(XNNPACK_ENABLE_AVX512AMX OFF CACHE BOOL "")
      ENDIF()
    ENDIF()

```

- **EN:** This chunk introduces sections such as Disable I8MM For CI since clang 9 does not support neon i8mm., Disable avxvnni int8, Older MSVC versions don't support AVX512FP. TODO Minimum version support?, Conditionally disable AVX512AMX, as it requires Clang 11 or later. Note that, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Disable I8MM For CI since clang 9 does not support neon i8mm.、Disable avxvnni int8、Older MSVC versions don't support AVX512FP. TODO Minimum version support?、Conditionally disable AVX512AMX, as it requires Clang 11 or later. Note that 等标题组织周边说明或配置。
- **EN:** CMake commands like set, IF, ENDIF drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、IF、ENDIF 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 574-589 / 第 574-589 行

```cmake
    # Setting this global PIC flag for all XNNPACK targets.
    # This is needed for Object libraries within XNNPACK which must
    # be PIC to successfully link this static libXNNPACK with pytorch
    set(__caffe2_CMAKE_POSITION_INDEPENDENT_CODE_FLAG ${CMAKE_POSITION_INDEPENDENT_CODE})
    set(CMAKE_POSITION_INDEPENDENT_CODE ON)

    if(WIN32)
      # Disable libm dependency explicitly to avoid symbol conflict for XNNPACK as
      # Windows runtime has provided the math functions - #134989
      set(XNNPACK_BUILD_WITH_LIBM OFF CACHE BOOL "")
    endif()

    add_subdirectory(
      "${XNNPACK_SOURCE_DIR}"
      "${CONFU_DEPENDENCIES_BINARY_DIR}/XNNPACK")

```

- **EN:** This chunk introduces sections such as Setting this global PIC flag for all XNNPACK targets., This is needed for Object libraries within XNNPACK which must, be PIC to successfully link this static libXNNPACK with pytorch, Disable libm dependency explicitly to avoid symbol conflict for XNNPACK as, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Setting this global PIC flag for all XNNPACK targets.、This is needed for Object libraries within XNNPACK which must、be PIC to successfully link this static libXNNPACK with pytorch、Disable libm dependency explicitly to avoid symbol conflict for XNNPACK as 等标题组织周边说明或配置。
- **EN:** CMake commands like set, if, endif, add_subdirectory drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、if、endif、add_subdirectory 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 590-615 / 第 590-615 行

```cmake
    if(CMAKE_C_COMPILER_ID STREQUAL "GNU" AND CMAKE_C_COMPILER_VERSION VERSION_GREATER_EQUAL "14")
      foreach(xnn_tgt IN ITEMS XNNPACK microkernels-prod microkernels-all)
          target_compile_options(${xnn_tgt} PRIVATE -Wno-error=incompatible-pointer-types)
      endforeach()
    endif()

    # Revert to whatever it was before
    set(CMAKE_POSITION_INDEPENDENT_CODE ${__caffe2_CMAKE_POSITION_INDEPENDENT_CODE_FLAG})
  endif()

  include_directories(SYSTEM ${XNNPACK_INCLUDE_DIR})
  list(APPEND Caffe2_DEPENDENCY_LIBS XNNPACK microkernels-prod)
elseif(NOT TARGET XNNPACK AND USE_SYSTEM_XNNPACK)
  add_library(XNNPACK SHARED IMPORTED)
  add_library(microkernels-prod SHARED IMPORTED)
  find_library(XNNPACK_LIBRARY XNNPACK)
  find_library(microkernels-prod_LIBRARY microkernels-prod)
  set_property(TARGET XNNPACK PROPERTY IMPORTED_LOCATION "${XNNPACK_LIBRARY}")
  set_property(TARGET microkernels-prod PROPERTY IMPORTED_LOCATION "${microkernels-prod_LIBRARY}")
  if(NOT XNNPACK_LIBRARY OR NOT microkernels-prod_LIBRARY)
    message(FATAL_ERROR "Cannot find XNNPACK")
  endif()
  message("-- Found XNNPACK: ${XNNPACK_LIBRARY}")
  list(APPEND Caffe2_DEPENDENCY_LIBS XNNPACK microkernels-prod)
endif()

```

- **EN:** This chunk introduces sections such as Revert to whatever it was before, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Revert to whatever it was before 等标题组织周边说明或配置。
- **EN:** CMake commands like if, foreach, target_compile_options, endforeach, endif, set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、foreach、target_compile_options、endforeach、endif、set 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 616-638 / 第 616-638 行

```cmake
# ---[ Vulkan deps
if(USE_VULKAN)
  set(Vulkan_DEFINES)
  set(Vulkan_INCLUDES)
  set(Vulkan_LIBS)
  include(${CMAKE_CURRENT_LIST_DIR}/VulkanDependencies.cmake)
  string(APPEND CMAKE_CXX_FLAGS ${Vulkan_DEFINES})
  include_directories(SYSTEM ${Vulkan_INCLUDES})
  list(APPEND Caffe2_DEPENDENCY_LIBS ${Vulkan_LIBS})
endif()

# ---[ gflags
if(USE_GFLAGS)
  include(${CMAKE_CURRENT_LIST_DIR}/public/gflags.cmake)
  if(NOT TARGET gflags)
    message(WARNING
        "gflags is not found. Caffe2 will build without gflags support but "
        "it is strongly recommended that you install gflags. Suppress this "
        "warning with -DUSE_GFLAGS=OFF")
    caffe2_update_option(USE_GFLAGS OFF)
  endif()
endif()

```

- **EN:** This chunk introduces sections such as ---[ Vulkan deps, ---[ gflags, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ Vulkan deps、---[ gflags 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, include, string, include_directories, list drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、include、string、include_directories、list 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 639-654 / 第 639-654 行

```cmake
# ---[ Google-glog
if(USE_GLOG)
  include(${CMAKE_CURRENT_LIST_DIR}/public/glog.cmake)
  if(TARGET glog::glog)
    set(CAFFE2_USE_GOOGLE_GLOG 1)
  else()
    message(WARNING
        "glog is not found. Caffe2 will build without glog support but it is "
        "strongly recommended that you install glog. Suppress this warning "
        "with -DUSE_GLOG=OFF")
    caffe2_update_option(USE_GLOG OFF)
  endif()
endif()


# ---[ Googletest and benchmark
```

- **EN:** This chunk introduces sections such as ---[ Google-glog, ---[ Googletest and benchmark, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ Google-glog、---[ Googletest and benchmark 等标题组织周边说明或配置。
- **EN:** CMake commands like if, include, set, else, message, caffe2_update_option drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、include、set、else、message、caffe2_update_option 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 655-670 / 第 655-670 行

```cmake
if(BUILD_TEST OR BUILD_MOBILE_BENCHMARK OR BUILD_MOBILE_TEST)
  # Preserve build options.
  set(TEMP_BUILD_SHARED_LIBS ${BUILD_SHARED_LIBS})

  # We will build gtest as static libs and embed it directly into the binary.
  set(BUILD_SHARED_LIBS OFF CACHE BOOL "Build shared libs" FORCE)

  # For gtest, we will simply embed it into our test binaries, so we won't
  # need to install it.
  set(INSTALL_GTEST OFF CACHE BOOL "Install gtest." FORCE)
  set(BUILD_GMOCK ON CACHE BOOL "Build gmock." FORCE)

  add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/../third_party/googletest)
  include_directories(BEFORE SYSTEM ${CMAKE_CURRENT_LIST_DIR}/../third_party/googletest/googletest/include)
  include_directories(BEFORE SYSTEM ${CMAKE_CURRENT_LIST_DIR}/../third_party/googletest/googlemock/include)

```

- **EN:** This chunk introduces sections such as Preserve build options., We will build gtest as static libs and embed it directly into the binary., For gtest, we will simply embed it into our test binaries, so we won't, need to install it., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Preserve build options.、We will build gtest as static libs and embed it directly into the binary.、For gtest, we will simply embed it into our test binaries, so we won't、need to install it. 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, add_subdirectory, include_directories drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、add_subdirectory、include_directories 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 671-686 / 第 671-686 行

```cmake
  # We will not need to test benchmark lib itself.
  set(BENCHMARK_ENABLE_TESTING OFF CACHE BOOL "Disable benchmark testing as we don't need it.")
  # We will not need to install benchmark since we link it statically.
  set(BENCHMARK_ENABLE_INSTALL OFF CACHE BOOL "Disable benchmark install to avoid overwriting vendor install.")
  if(NOT USE_SYSTEM_BENCHMARK)
    add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/../third_party/benchmark)
  else()
    add_library(benchmark SHARED IMPORTED)
    find_library(BENCHMARK_LIBRARY benchmark)
    if(NOT BENCHMARK_LIBRARY)
      message(FATAL_ERROR "Cannot find google benchmark library")
    endif()
    message("-- Found benchmark: ${BENCHMARK_LIBRARY}")
    set_property(TARGET benchmark PROPERTY IMPORTED_LOCATION ${BENCHMARK_LIBRARY})
  endif()

```

- **EN:** This chunk introduces sections such as We will not need to test benchmark lib itself., We will not need to install benchmark since we link it statically., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 We will not need to test benchmark lib itself.、We will not need to install benchmark since we link it statically. 等标题组织周边说明或配置。
- **EN:** CMake commands like set, if, add_subdirectory, else, add_library, find_library drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、if、add_subdirectory、else、add_library、find_library 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 687-702 / 第 687-702 行

```cmake
  # Recover build options.
  set(BUILD_SHARED_LIBS ${TEMP_BUILD_SHARED_LIBS} CACHE BOOL "Build shared libs" FORCE)
endif()

# ---[ FBGEMM
if(USE_FBGEMM)
  set(CAFFE2_THIRD_PARTY_ROOT "${PROJECT_SOURCE_DIR}/third_party")
  if(NOT DEFINED FBGEMM_SOURCE_DIR)
    set(FBGEMM_SOURCE_DIR "${CAFFE2_THIRD_PARTY_ROOT}/fbgemm" CACHE STRING "FBGEMM source directory")
  endif()
  if(USE_FBGEMM AND NOT TARGET fbgemm)
    set(FBGEMM_BUILD_TESTS OFF CACHE BOOL "")
    set(FBGEMM_BUILD_BENCHMARKS OFF CACHE BOOL "")
    set(FBGEMM_LIBRARY_TYPE "static" CACHE STRING "")
    add_subdirectory("${FBGEMM_SOURCE_DIR}")

```

- **EN:** This chunk introduces sections such as Recover build options., ---[ FBGEMM, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Recover build options.、---[ FBGEMM 等标题组织周边说明或配置。
- **EN:** CMake commands like set, endif, if, add_subdirectory drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、endif、if、add_subdirectory 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 703-720 / 第 703-720 行

```cmake
    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
      target_compile_options_if_supported(asmjit -Wno-extra-semi)
      target_compile_options_if_supported(fbgemm -Wno-extra-semi)
    endif()
    target_compile_options_if_supported(asmjit -Wno-unused-but-set-variable)
    target_compile_options_if_supported(asmjit -Wno-unused-variable)
  endif()
  if(USE_FBGEMM)
    list(APPEND Caffe2_DEPENDENCY_LIBS fbgemm)
  endif()
endif()

if(USE_FBGEMM)
  caffe2_update_option(USE_FBGEMM ON)
else()
  caffe2_update_option(USE_FBGEMM OFF)
endif()

```

- **EN:** CMake commands like if, target_compile_options_if_supported, endif, list, caffe2_update_option, else drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、target_compile_options_if_supported、endif、list、caffe2_update_option、else 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 721-741 / 第 721-741 行

```cmake
if(USE_OPENCL)
  message(INFO "USING OPENCL")
  find_package(OpenCL REQUIRED)
  include_directories(SYSTEM ${OpenCL_INCLUDE_DIRS})
  list(APPEND Caffe2_DEPENDENCY_LIBS ${OpenCL_LIBRARIES})
endif()

# ---[ NUMA
if(USE_NUMA)
  if(LINUX)
    find_package(Numa)
    if(NOT NUMA_FOUND)
      message(WARNING "Not compiling with NUMA. Suppress this warning with -DUSE_NUMA=OFF")
      caffe2_update_option(USE_NUMA OFF)
    endif()
  else()
    message(WARNING "NUMA is currently only supported under Linux.")
    caffe2_update_option(USE_NUMA OFF)
  endif()
endif()

```

- **EN:** This chunk introduces sections such as ---[ NUMA, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ NUMA 等标题组织周边说明或配置。
- **EN:** CMake commands like if, message, find_package, include_directories, list, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、message、find_package、include_directories、list、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 742-758 / 第 742-758 行

```cmake
if(USE_ITT)
  find_package(ITT)
  if(ITT_FOUND)
    include_directories(SYSTEM ${ITT_INCLUDE_DIR})
    list(APPEND Caffe2_DEPENDENCY_LIBS ${ITT_LIBRARIES})
    list(APPEND TORCH_PYTHON_LINK_LIBRARIES ${ITT_LIBRARIES})
  else()
    message(WARNING "Not compiling with ITT. Suppress this warning with -DUSE_ITT=OFF")
    set(USE_ITT OFF CACHE BOOL "" FORCE)
    caffe2_update_option(USE_ITT OFF)
  endif()
endif()

# ---[ Caffe2 depends on FP16 library for half-precision conversions
if(NOT TARGET fp16 AND NOT USE_SYSTEM_FP16)
  set(CAFFE2_THIRD_PARTY_ROOT "${PROJECT_SOURCE_DIR}/third_party")
  # PSIMD is required by FP16
```

- **EN:** This chunk introduces sections such as ---[ Caffe2 depends on FP16 library for half-precision conversions, PSIMD is required by FP16, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ Caffe2 depends on FP16 library for half-precision conversions、PSIMD is required by FP16 等标题组织周边说明或配置。
- **EN:** CMake commands like if, find_package, include_directories, list, else, message drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、find_package、include_directories、list、else、message 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 759-785 / 第 759-785 行

```cmake
  if(NOT DEFINED PSIMD_SOURCE_DIR)
    set(PSIMD_SOURCE_DIR "${CAFFE2_THIRD_PARTY_ROOT}/psimd" CACHE STRING "PSimd source directory")
  endif()
  if(NOT DEFINED FP16_SOURCE_DIR)
    set(FP16_SOURCE_DIR "${CAFFE2_THIRD_PARTY_ROOT}/FP16" CACHE STRING "FP16 source directory")
  endif()

  set(FP16_BUILD_TESTS OFF CACHE BOOL "")
  set(FP16_BUILD_BENCHMARKS OFF CACHE BOOL "")
  if(CMAKE_VERSION VERSION_GREATER_EQUAL "4.0.0")
    message(WARNING "FP16 is only cmake-2.8 compatible")
    set(CMAKE_POLICY_VERSION_MINIMUM 3.5)
    add_subdirectory(
      "${FP16_SOURCE_DIR}"
      "${CONFU_DEPENDENCIES_BINARY_DIR}/FP16")
    unset(CMAKE_POLICY_VERSION_MINIMUM)
  else()
    add_subdirectory(
      "${FP16_SOURCE_DIR}"
      "${CONFU_DEPENDENCIES_BINARY_DIR}/FP16")
  endif()
elseif(NOT TARGET fp16 AND USE_SYSTEM_FP16)
  add_library(fp16 STATIC "/usr/include/fp16.h")
  set_target_properties(fp16 PROPERTIES LINKER_LANGUAGE C)
endif()
list(APPEND Caffe2_DEPENDENCY_LIBS fp16)

```

- **EN:** CMake commands like if, set, endif, message, add_subdirectory, unset drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、endif、message、add_subdirectory、unset 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 786-801 / 第 786-801 行

```cmake
# ---[ Python Interpreter
# If not given a Python installation, then use the current active Python
if(NOT Python_EXECUTABLE)
  execute_process(
    COMMAND "which" "python3" RESULT_VARIABLE _exitcode OUTPUT_VARIABLE _py_exe)
  if(${_exitcode} EQUAL 0)
    if(NOT MSVC)
      string(STRIP ${_py_exe} Python_EXECUTABLE)
    endif()
    message(STATUS "Setting Python to ${Python_EXECUTABLE}")
  endif()
endif()


# ---[ EIGEN
# Due to license considerations, we will only use the MPL2 parts of Eigen.
```

- **EN:** This chunk introduces sections such as ---[ Python Interpreter, If not given a Python installation, then use the current active Python, ---[ EIGEN, Due to license considerations, we will only use the MPL2 parts of Eigen., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ Python Interpreter、If not given a Python installation, then use the current active Python、---[ EIGEN、Due to license considerations, we will only use the MPL2 parts of Eigen. 等标题组织周边说明或配置。
- **EN:** CMake commands like if, execute_process, string, endif, message drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、execute_process、string、endif、message 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 802-820 / 第 802-820 行

```cmake
set(EIGEN_MPL2_ONLY 1)
if(USE_SYSTEM_EIGEN_INSTALL)
  find_package(Eigen3)
  if(EIGEN3_FOUND)
    message(STATUS "Found system Eigen at " ${EIGEN3_INCLUDE_DIR})
  else()
    message(STATUS "Did not find system Eigen. Using third party subdirectory.")
    execute_process(COMMAND ${Python_EXECUTABLE} ../tools/optional_modules.py checkout_eigen
                    WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR})

    set(EIGEN3_INCLUDE_DIR ${CMAKE_CURRENT_LIST_DIR}/../third_party/eigen)
    caffe2_update_option(USE_SYSTEM_EIGEN_INSTALL OFF)
  endif()
else()
  message(STATUS "Using third party subdirectory Eigen.")
  set(EIGEN3_INCLUDE_DIR ${CMAKE_CURRENT_LIST_DIR}/../third_party/eigen)
endif()
include_directories(SYSTEM ${EIGEN3_INCLUDE_DIR})

```

- **EN:** CMake commands like set, if, find_package, message, else, execute_process drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、if、find_package、message、else、execute_process 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 821-840 / 第 821-840 行

```cmake

if(BUILD_PYTHON)
  set(PYTHON_COMPONENTS Development.Module)
  if(USE_NUMPY)
    list(APPEND PYTHON_COMPONENTS NumPy)
  endif()
  find_package(Python COMPONENTS Interpreter OPTIONAL_COMPONENTS ${PYTHON_COMPONENTS})
else()
  find_package(Python COMPONENTS Interpreter)
endif()

if(NOT Python_Interpreter_FOUND)
  message(FATAL_ERROR "Python3 could not be found.")
endif()

if(${Python_VERSION} VERSION_LESS 3.10)
  message(FATAL_ERROR
    "Found Python libraries version ${Python_VERSION}. Python < 3.10 is no longer supported by PyTorch.")
endif()

```

- **EN:** CMake commands like if, set, list, endif, find_package, else drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、list、endif、find_package、else 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 841-859 / 第 841-859 行

```cmake
# ---[ Python + Numpy
if(BUILD_PYTHON)
  if(Python_Development.Module_FOUND)
    if(USE_NUMPY)
      if(NOT Python_NumPy_FOUND)
        message(WARNING "NumPy could not be found. Not building with NumPy. Suppress this warning with -DUSE_NUMPY=OFF")
        caffe2_update_option(USE_NUMPY OFF)
      else()
        caffe2_update_option(USE_NUMPY ON)
      endif()
    endif()
    # Observers are required in the python build
    caffe2_update_option(USE_OBSERVERS ON)
  else()
    message(WARNING "Python dependencies not met. Not compiling with python. Suppress this warning with -DBUILD_PYTHON=OFF")
    caffe2_update_option(BUILD_PYTHON OFF)
  endif()
endif()

```

- **EN:** This chunk introduces sections such as ---[ Python + Numpy, Observers are required in the python build, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ Python + Numpy、Observers are required in the python build 等标题组织周边说明或配置。
- **EN:** CMake commands like if, message, caffe2_update_option, else, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、message、caffe2_update_option、else、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 860-880 / 第 860-880 行

```cmake
# ---[ pybind11
if(USE_SYSTEM_PYBIND11)
  find_package(pybind11 CONFIG)
  if(NOT pybind11_FOUND)
    find_package(pybind11)
  endif()
  if(NOT pybind11_FOUND)
    message(FATAL "Cannot find system pybind11")
  endif()
else()
    message(STATUS "Using third_party/pybind11.")
    set(pybind11_INCLUDE_DIRS ${CMAKE_CURRENT_LIST_DIR}/../third_party/pybind11/include)
    install(DIRECTORY ${pybind11_INCLUDE_DIRS}
            DESTINATION ${CMAKE_INSTALL_PREFIX}
            FILES_MATCHING PATTERN "*.h")
endif()
message(STATUS "pybind11 include dirs: " "${pybind11_INCLUDE_DIRS}")
add_library(pybind::pybind11 INTERFACE IMPORTED)
target_include_directories(pybind::pybind11 SYSTEM INTERFACE ${pybind11_INCLUDE_DIRS})
target_link_libraries(pybind::pybind11 INTERFACE Python::Module)

```

- **EN:** This chunk introduces sections such as ---[ pybind11, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ pybind11 等标题组织周边说明或配置。
- **EN:** CMake commands like if, find_package, endif, message, else, set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、find_package、endif、message、else、set 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 881-908 / 第 881-908 行

```cmake
# ---[ MPI
if(USE_MPI)
  find_package(MPI)
  if(MPI_CXX_FOUND)
    message(STATUS "MPI support found")
    message(STATUS "MPI compile flags: " ${MPI_CXX_COMPILE_FLAGS})
    message(STATUS "MPI include path: " ${MPI_CXX_INCLUDE_PATH})
    message(STATUS "MPI LINK flags path: " ${MPI_CXX_LINK_FLAGS})
    message(STATUS "MPI libraries: " ${MPI_CXX_LIBRARIES})
    find_program(OMPI_INFO
      NAMES ompi_info
      HINTS ${MPI_CXX_LIBRARIES}/../bin)
    if(OMPI_INFO)
      execute_process(COMMAND ${OMPI_INFO}
                      OUTPUT_VARIABLE _output)
      if(_output MATCHES "smcuda")
        message(STATUS "Found OpenMPI with CUDA support built.")
      else()
        message(WARNING "OpenMPI found, but it is not built with CUDA support.")
        set(CAFFE2_FORCE_FALLBACK_CUDA_MPI 1)
      endif()
    endif()
  else()
    message(WARNING "Not compiling with MPI. Suppress this warning with -DUSE_MPI=OFF")
    caffe2_update_option(USE_MPI OFF)
  endif()
endif()

```

- **EN:** This chunk introduces sections such as ---[ MPI, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ MPI 等标题组织周边说明或配置。
- **EN:** CMake commands like if, find_package, message, find_program, execute_process, else drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、find_package、message、find_program、execute_process、else 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 909-932 / 第 909-932 行

```cmake
# ---[ OpenMP
if(USE_OPENMP AND NOT TARGET caffe2::openmp)
  include(${CMAKE_CURRENT_LIST_DIR}/Modules/FindOpenMP.cmake)
  if(OPENMP_FOUND)
    message(STATUS "Adding OpenMP CXX_FLAGS: " ${OpenMP_CXX_FLAGS})
    if(APPLE AND USE_MPS)
      string(APPEND CMAKE_OBJCXX_FLAGS " ${OpenMP_CXX_FLAGS}")
    endif()
    if(OpenMP_CXX_LIBRARIES)
      message(STATUS "Will link against OpenMP libraries: ${OpenMP_CXX_LIBRARIES}")
    endif()
    add_library(caffe2::openmp INTERFACE IMPORTED)
    target_link_libraries(caffe2::openmp INTERFACE OpenMP::OpenMP_CXX)
    list(APPEND Caffe2_DEPENDENCY_LIBS caffe2::openmp)
    if(MSVC AND OpenMP_CXX_LIBRARIES MATCHES ".*libiomp5md\\.lib.*")
      target_compile_definitions(caffe2::openmp INTERFACE _OPENMP_NOFORCE_MANIFEST)
      target_link_options(caffe2::openmp INTERFACE "/NODEFAULTLIB:vcomp")
    endif()
  else()
    message(WARNING "Not compiling with OpenMP. Suppress this warning with -DUSE_OPENMP=OFF")
    caffe2_update_option(USE_OPENMP OFF)
  endif()
endif()

```

- **EN:** This chunk introduces sections such as ---[ OpenMP, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ OpenMP 等标题组织周边说明或配置。
- **EN:** CMake commands like if, include, message, string, endif, add_library drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、include、message、string、endif、add_library 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 933-948 / 第 933-948 行

```cmake


# ---[ Android specific ones
if(ANDROID)
  list(APPEND Caffe2_DEPENDENCY_LIBS log)
endif()

# ---[ LLVM
if(USE_LLVM)
  message(STATUS "Looking for LLVM in ${USE_LLVM}")
  find_package(LLVM PATHS ${USE_LLVM} NO_DEFAULT_PATH)

  if(LLVM_FOUND)
    message(STATUS "Found LLVM ${LLVM_PACKAGE_VERSION}")
    message(STATUS "Using LLVMConfig.cmake in: ${LLVM_DIR}")

```

- **EN:** This chunk introduces sections such as ---[ Android specific ones, ---[ LLVM, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ Android specific ones、---[ LLVM 等标题组织周边说明或配置。
- **EN:** CMake commands like if, list, endif, message, find_package drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、list、endif、message、find_package 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 949-979 / 第 949-979 行

```cmake
    include_directories(${LLVM_INCLUDE_DIRS})
    add_definitions(-DTORCH_ENABLE_LLVM)
  endif(LLVM_FOUND)
endif(USE_LLVM)

# ---[ cuDNN
if(USE_CUDNN)
  if(CUDNN_VERSION VERSION_LESS 8.5)
    message(FATAL_ERROR "PyTorch needs CuDNN-8.5 or above, but found ${CUDNN_VERSION}. Builds are still possible with `USE_CUDNN=0`")
  endif()
  set(CUDNN_FRONTEND_INCLUDE_DIR ${CMAKE_CURRENT_LIST_DIR}/../third_party/cudnn_frontend/include)
  target_include_directories(torch::cudnn INTERFACE ${CUDNN_FRONTEND_INCLUDE_DIR})
endif()

# ---[ nvtx
if(USE_SYSTEM_NVTX)
  find_path(nvtx3_dir NAMES nvtx3 PATHS ${CUDA_INCLUDE_DIRS})
else()
  find_path(nvtx3_dir NAMES nvtx3 PATHS "${PROJECT_SOURCE_DIR}/third_party/NVTX/c/include" NO_DEFAULT_PATH)
endif()
find_package_handle_standard_args(nvtx3 DEFAULT_MSG nvtx3_dir)
if(nvtx3_FOUND)
  add_library(torch::nvtx3 INTERFACE IMPORTED)
  target_include_directories(torch::nvtx3 INTERFACE "${nvtx3_dir}")
  target_compile_definitions(torch::nvtx3 INTERFACE TORCH_CUDA_USE_NVTX3)
else()
  message(WARNING "Cannot find NVTX3, find old NVTX instead")
  add_library(torch::nvtoolsext INTERFACE IMPORTED)
  set_property(TARGET torch::nvtoolsext PROPERTY INTERFACE_LINK_LIBRARIES CUDA::nvToolsExt)
endif()

```

- **EN:** This chunk introduces sections such as ---[ cuDNN, ---[ nvtx, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ cuDNN、---[ nvtx 等标题组织周边说明或配置。
- **EN:** CMake commands like include_directories, add_definitions, endif, if, message, set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 include_directories、add_definitions、endif、if、message、set 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 980-1011 / 第 980-1011 行

```cmake

# ---[ HIP
if(USE_ROCM)
  include(${CMAKE_CURRENT_LIST_DIR}/public/LoadHIP.cmake)
  if(PYTORCH_FOUND_HIP)
    message(INFO "Compiling with HIP for AMD.")
    caffe2_update_option(USE_ROCM ON)

    if(USE_NCCL AND NOT USE_SYSTEM_NCCL)
      message(INFO "Forcing USE_SYSTEM_NCCL to ON since it's required by using RCCL")
      caffe2_update_option(USE_SYSTEM_NCCL ON)
    endif()

    if(WIN32)
      if(${CAFFE2_USE_MSVC_STATIC_RUNTIME})
        if(CMAKE_BUILD_TYPE MATCHES Debug)
          list(APPEND HIP_CXX_FLAGS -fms-runtime-lib=static_dbg)
        else()
          list(APPEND HIP_CXX_FLAGS -fms-runtime-lib=static)
        endif()
      else()
        if(CMAKE_BUILD_TYPE MATCHES Debug)
          list(APPEND HIP_CXX_FLAGS -fms-runtime-lib=dll_dbg)
        else()
          list(APPEND HIP_CXX_FLAGS -fms-runtime-lib=dll)
        endif()
      endif()
    else()
      list(APPEND HIP_CXX_FLAGS -fPIC)
    endif()
    list(APPEND HIP_CXX_FLAGS -D__HIP_PLATFORM_AMD__=1)
    list(APPEND HIP_CXX_FLAGS -DCUDA_HAS_FP16=1)
```

- **EN:** This chunk introduces sections such as ---[ HIP, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ HIP 等标题组织周边说明或配置。
- **EN:** CMake commands like if, include, message, caffe2_update_option, endif, list drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、include、message、caffe2_update_option、endif、list 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1012-1033 / 第 1012-1033 行

```cmake
    list(APPEND HIP_CXX_FLAGS -DUSE_ROCM)
    list(APPEND HIP_CXX_FLAGS -D__HIP_NO_HALF_OPERATORS__=1)
    list(APPEND HIP_CXX_FLAGS -D__HIP_NO_HALF_CONVERSIONS__=1)
    list(APPEND HIP_CXX_FLAGS -DTORCH_HIP_VERSION=${TORCH_HIP_VERSION})
    list(APPEND HIP_CXX_FLAGS -Wno-shift-count-negative)
    list(APPEND HIP_CXX_FLAGS -Wno-shift-count-overflow)
    list(APPEND HIP_CXX_FLAGS -DCAFFE2_USE_MIOPEN)
    list(APPEND HIP_CXX_FLAGS -DTHRUST_DEVICE_SYSTEM=THRUST_DEVICE_SYSTEM_HIP)
    list(APPEND HIP_CXX_FLAGS -DHIPBLAS_V2)
    list(APPEND HIP_CXX_FLAGS -DHIP_ENABLE_WARP_SYNC_BUILTINS)
    if(HIPBLASLT_OUTER_VEC)
      list(APPEND HIP_CXX_FLAGS -DHIPBLASLT_OUTER_VEC)
    endif()
    if(HIPBLASLT_VEC_EXT)
      list(APPEND HIP_CXX_FLAGS -DHIPBLASLT_VEC_EXT)
    endif()
    if(USE_ROCM_CK_GEMM)
      list(APPEND HIP_CXX_FLAGS -DUSE_ROCM_CK_GEMM)
    endif()
    list(APPEND HIP_HIPCC_FLAGS --offload-compress)
    list(APPEND HIP_HIPCC_FLAGS -std=c++20)
    # Pass device library path for theRock nightly builds
```

- **EN:** This chunk introduces sections such as Pass device library path for theRock nightly builds, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Pass device library path for theRock nightly builds 等标题组织周边说明或配置。
- **EN:** CMake commands like list, if, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 list、if、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1034-1050 / 第 1034-1050 行

```cmake
    if(DEFINED ENV{HIP_DEVICE_LIB_PATH})
      file(TO_CMAKE_PATH "$ENV{HIP_DEVICE_LIB_PATH}" _hip_device_lib_path)
      list(APPEND HIP_HIPCC_FLAGS --rocm-device-lib-path=${_hip_device_lib_path})
    elseif(EXISTS "${ROCM_PATH}/lib/llvm/amdgcn/bitcode")
      file(TO_CMAKE_PATH "${ROCM_PATH}/lib/llvm/amdgcn/bitcode" _rocm_device_lib_path)
      list(APPEND HIP_HIPCC_FLAGS --rocm-device-lib-path=${_rocm_device_lib_path})
    endif()
    if(WIN32)
      add_definitions(-DROCM_ON_WINDOWS)
      list(APPEND HIP_CXX_FLAGS -fms-extensions)
      # Suppress warnings about dllexport.
      list(APPEND HIP_CXX_FLAGS -Wno-ignored-attributes)
    endif()
    add_definitions(-DROCM_VERSION=${ROCM_VERSION_DEV_INT})
    add_definitions(-DTORCH_HIP_VERSION=${TORCH_HIP_VERSION})
    message("TORCH_HIP_VERSION=${TORCH_HIP_VERSION} is added as a compiler defines")

```

- **EN:** This chunk introduces sections such as Suppress warnings about dllexport., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Suppress warnings about dllexport. 等标题组织周边说明或配置。
- **EN:** CMake commands like if, file, list, elseif, endif, add_definitions drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、file、list、elseif、endif、add_definitions 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1051-1067 / 第 1051-1067 行

```cmake
    if(CMAKE_BUILD_TYPE MATCHES Debug)
       list(APPEND HIP_CXX_FLAGS -g2)
       list(APPEND HIP_CXX_FLAGS -O0)
       list(APPEND HIP_HIPCC_FLAGS -fdebug-info-for-profiling)
    endif(CMAKE_BUILD_TYPE MATCHES Debug)

    # Get EnVar 'USE_LAYERNORM_FAST_RECIPROCAL' (or default to on).
    if(DEFINED ENV{USE_LAYERNORM_FAST_RECIPROCAL})
      set(USE_LAYERNORM_FAST_RECIPROCAL $ENV{USE_LAYERNORM_FAST_RECIPROCAL})
    else()
      set(USE_LAYERNORM_FAST_RECIPROCAL ON)
    endif()

    if(USE_LAYERNORM_FAST_RECIPROCAL)
      add_definitions(-DUSE_LAYERNORM_FAST_RECIPROCAL)
    endif()

```

- **EN:** This chunk introduces sections such as Get EnVar 'USE_LAYERNORM_FAST_RECIPROCAL' (or default to on)., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Get EnVar 'USE_LAYERNORM_FAST_RECIPROCAL' (or default to on). 等标题组织周边说明或配置。
- **EN:** CMake commands like if, list, endif, set, else, add_definitions drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、list、endif、set、else、add_definitions 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1068-1084 / 第 1068-1084 行

```cmake
    # needed for compat with newer versions of hip-clang that introduced C++20 mangling rules
    list(APPEND HIP_HIPCC_FLAGS -fclang-abi-compat=17)

    set(HIP_CLANG_FLAGS ${HIP_CXX_FLAGS})
    string(JOIN " " HIP_HIPCC_FLAGS_STR ${HIP_HIPCC_FLAGS})
    set(HIP_HIPCC_FLAGS ${HIP_HIPCC_FLAGS_STR})
    set(CMAKE_HIP_FLAGS ${HIP_HIPCC_FLAGS})
    # Ask hcc to generate device code during compilation so we can use
    # host linker to link.
    list(APPEND HIP_CLANG_FLAGS -fno-gpu-rdc)
    foreach(pytorch_rocm_arch ${PYTORCH_ROCM_ARCH})
      list(APPEND HIP_CLANG_FLAGS --offload-arch=${pytorch_rocm_arch})
    endforeach()

    set(Caffe2_HIP_INCLUDE
       $<INSTALL_INTERFACE:include> ${Caffe2_HIP_INCLUDE})
    # This is needed for library added by hip_add_library (same for hip_add_executable)
```

- **EN:** This chunk introduces sections such as needed for compat with newer versions of hip-clang that introduced C++20 mangling rules, Ask hcc to generate device code during compilation so we can use, host linker to link., This is needed for library added by hip_add_library (same for hip_add_executable), which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 needed for compat with newer versions of hip-clang that introduced C++20 mangling rules、Ask hcc to generate device code during compilation so we can use、host linker to link.、This is needed for library added by hip_add_library (same for hip_add_executable) 等标题组织周边说明或配置。
- **EN:** CMake commands like list, set, string, foreach, endforeach drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 list、set、string、foreach、endforeach 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1085-1102 / 第 1085-1102 行

```cmake
    hip_include_directories(${Caffe2_HIP_INCLUDE})

    set(Caffe2_PUBLIC_HIP_DEPENDENCY_LIBS
      hip::amdhip64 MIOpen hiprtc::hiprtc) # libroctx will be linked in with MIOpen

    # Math libraries
    list(APPEND Caffe2_PUBLIC_HIP_DEPENDENCY_LIBS
      roc::hipblas roc::rocblas hip::hipfft hip::hiprand roc::hipsparse roc::hipsolver roc::hipblaslt roc::rocsolver)
    # hipsparselt is an optional component that will eventually be enabled by default.
    if(hipsparselt_FOUND)
      list(APPEND Caffe2_PUBLIC_HIP_DEPENDENCY_LIBS
        roc::hipsparselt
      )
      if(ROCM_VERSION_DEV VERSION_GREATER_EQUAL "7.12.0")
          set(CAFFE2_USE_HIPSPARSELT ON)
      endif()
    endif()

```

- **EN:** This chunk introduces sections such as Math libraries, hipsparselt is an optional component that will eventually be enabled by default., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Math libraries、hipsparselt is an optional component that will eventually be enabled by default. 等标题组织周边说明或配置。
- **EN:** CMake commands like hip_include_directories, set, list, if, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 hip_include_directories、set、list、if、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1103-1118 / 第 1103-1118 行

```cmake
    # ROCM-SMI needed to support symmetric memory
    if(USE_DISTRIBUTED AND UNIX)
      list(APPEND Caffe2_PUBLIC_HIP_DEPENDENCY_LIBS
        rocm_smi64
      )
    endif()

    # ---[ Kernel asserts
    # Kernel asserts is disabled for ROCm by default.
    # It can be turned on by turning on the env USE_ROCM_KERNEL_ASSERT to the build system.
    if(USE_ROCM_KERNEL_ASSERT)
      message(STATUS "Enabling Kernel Assert for ROCm")
    else()
      message(STATUS "Disabling Kernel Assert for ROCm")
    endif()

```

- **EN:** This chunk introduces sections such as ROCM-SMI needed to support symmetric memory, ---[ Kernel asserts, Kernel asserts is disabled for ROCm by default., It can be turned on by turning on the env USE_ROCM_KERNEL_ASSERT to the build system., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ROCM-SMI needed to support symmetric memory、---[ Kernel asserts、Kernel asserts is disabled for ROCm by default.、It can be turned on by turning on the env USE_ROCM_KERNEL_ASSERT to the build system. 等标题组织周边说明或配置。
- **EN:** CMake commands like if, list, endif, message, else drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、list、endif、message、else 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1119-1148 / 第 1119-1148 行

```cmake
  else()
    caffe2_update_option(USE_ROCM OFF)
  endif()

  # Add ROCm includes as SYSTEM includes (lower priority than regular includes).
  # This ensures third_party vendored headers take precedence over ROCm headers.
  if(USE_ROCM AND ROCM_INCLUDE_DIRS)
    include_directories(SYSTEM ${ROCM_INCLUDE_DIRS})
  endif()
endif()

# ---[ NCCL
if(USE_NCCL)
  if(NOT (USE_CUDA OR USE_ROCM))
    message(WARNING
        "Not using CUDA/ROCM, so disabling USE_NCCL. Suppress this warning with "
        "-DUSE_NCCL=OFF.")
    caffe2_update_option(USE_NCCL OFF)
  elseif(NOT CMAKE_SYSTEM_NAME STREQUAL "Linux")
    message(WARNING "NCCL is currently only supported under Linux.")
    caffe2_update_option(USE_NCCL OFF)
  elseif(USE_CUDA)
    include(${CMAKE_CURRENT_LIST_DIR}/External/nccl.cmake)
    list(APPEND Caffe2_CUDA_DEPENDENCY_LIBS __caffe2_nccl)
  elseif(USE_ROCM)
    include(${CMAKE_CURRENT_LIST_DIR}/External/rccl.cmake)
    list(APPEND Caffe2_CUDA_DEPENDENCY_LIBS __caffe2_nccl)
  endif()
endif()

```

- **EN:** This chunk introduces sections such as Add ROCm includes as SYSTEM includes (lower priority than regular includes)., This ensures third_party vendored headers take precedence over ROCm headers., ---[ NCCL, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Add ROCm includes as SYSTEM includes (lower priority than regular includes).、This ensures third_party vendored headers take precedence over ROCm headers.、---[ NCCL 等标题组织周边说明或配置。
- **EN:** CMake commands like else, caffe2_update_option, endif, if, include_directories, message drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 else、caffe2_update_option、endif、if、include_directories、message 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1149-1166 / 第 1149-1166 行

```cmake
# ---[ XCCL
if(USE_XCCL)
  if(NOT USE_XPU)
    message(WARNING "Not using XPU, so disabling USE_XCCL. Suppress this warning with -DUSE_XCCL=OFF.")
    caffe2_update_option(USE_XCCL OFF)
  endif()
endif()

# ---[ UCC
if(USE_UCC)
  if(NOT CMAKE_SYSTEM_NAME STREQUAL "Linux")
    message(WARNING "UCC is currently only supported under Linux.")
    caffe2_update_option(USE_UCC OFF)
  else()
    include(${CMAKE_CURRENT_LIST_DIR}/External/ucc.cmake)
  endif()
endif()

```

- **EN:** This chunk introduces sections such as ---[ XCCL, ---[ UCC, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ XCCL、---[ UCC 等标题组织周边说明或配置。
- **EN:** CMake commands like if, message, caffe2_update_option, endif, else, include drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、message、caffe2_update_option、endif、else、include 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1167-1188 / 第 1167-1188 行

```cmake
# ---[ CUB
if(USE_CUDA AND CUDA_VERSION VERSION_LESS 13.0)
  find_package(CUB)
  if(NOT CUB_FOUND)
    message(FATAL_ERROR "Cannot find CUB.")
  endif()
  include_directories(SYSTEM ${CUB_INCLUDE_DIRS})
endif()

if(USE_DISTRIBUTED AND USE_TENSORPIPE)
  if(MSVC)
    message(WARNING "Tensorpipe cannot be used on Windows.")
  else()
    if(USE_CUDA)
      set(TP_USE_CUDA ON CACHE BOOL "" FORCE)
      set(TP_ENABLE_CUDA_IPC ON CACHE BOOL "" FORCE)
    endif()
    set(TP_BUILD_LIBUV ON CACHE BOOL "" FORCE)
    add_compile_options(-DTORCH_USE_LIBUV)
    include_directories(BEFORE SYSTEM ${CMAKE_CURRENT_LIST_DIR}/../third_party/tensorpipe/third_party/libuv/include)
    set(TP_STATIC_OR_SHARED STATIC CACHE STRING "" FORCE)

```

- **EN:** This chunk introduces sections such as ---[ CUB, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ CUB 等标题组织周边说明或配置。
- **EN:** CMake commands like if, find_package, message, endif, include_directories, else drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、find_package、message、endif、include_directories、else 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1189-1209 / 第 1189-1209 行

```cmake
    # Tensorpipe uses cuda_add_library
    torch_update_find_cuda_flags()
    add_subdirectory(${PROJECT_SOURCE_DIR}/third_party/tensorpipe)
    # Suppress warning to unblock libnop compilation by clang-17
    # See https://github.com/pytorch/pytorch/issues/151316
    target_compile_options_if_supported(tensorpipe -Wno-missing-template-arg-list-after-template-kw)
    # Workaround for relocation truncated to fit: R_AARCH64_CALL26 against symbol __aarch64_swp4_relax'
    # When compiling for ARMv8.0, build uv with embedded atomics, which are slightly slower
    # But are used only once during shutdown
    if(CMAKE_SYSTEM_PROCESSOR STREQUAL "aarch64")
      target_compile_options_if_supported(tensorpipe_uv -mno-outline-atomics)
    endif()

    list(APPEND Caffe2_DEPENDENCY_LIBS tensorpipe)
    list(APPEND Caffe2_DEPENDENCY_LIBS nlohmann)
    list(APPEND Caffe2_DEPENDENCY_LIBS moodycamel)
    if(USE_CUDA)
      list(APPEND Caffe2_CUDA_DEPENDENCY_LIBS tensorpipe_cuda)
    elseif(USE_ROCM)
      message(WARNING "TensorPipe doesn't yet support ROCm")
      # Not yet...
```

- **EN:** This chunk introduces sections such as Tensorpipe uses cuda_add_library, Suppress warning to unblock libnop compilation by clang-17, See https://github.com/pytorch/pytorch/issues/151316, Workaround for relocation truncated to fit: R_AARCH64_CALL26 against symbol __aarch64_swp4_relax', which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Tensorpipe uses cuda_add_library、Suppress warning to unblock libnop compilation by clang-17、See https://github.com/pytorch/pytorch/issues/151316、Workaround for relocation truncated to fit: R_AARCH64_CALL26 against symbol __aarch64_swp4_relax' 等标题组织周边说明或配置。
- **EN:** CMake commands like torch_update_find_cuda_flags, add_subdirectory, target_compile_options_if_supported, if, endif, list drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 torch_update_find_cuda_flags、add_subdirectory、target_compile_options_if_supported、if、endif、list 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1210-1227 / 第 1210-1227 行

```cmake
      # list(APPEND Caffe2_HIP_DEPENDENCY_LIBS tensorpipe_hip)
    endif()
  endif()
endif()

if(USE_GLOO)
  if(NOT CMAKE_SIZEOF_VOID_P EQUAL 8)
    message(WARNING "Gloo can only be used on 64-bit systems.")
    caffe2_update_option(USE_GLOO OFF)
  else()
    # Don't install gloo
    set(GLOO_INSTALL OFF CACHE BOOL "" FORCE)
    set(GLOO_STATIC_OR_SHARED STATIC CACHE STRING "" FORCE)

    if(USE_GLOO_IBVERBS)
      set(USE_IBVERBS ON)
    endif()

```

- **EN:** This chunk introduces sections such as list(APPEND Caffe2_HIP_DEPENDENCY_LIBS tensorpipe_hip), Don't install gloo, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 list(APPEND Caffe2_HIP_DEPENDENCY_LIBS tensorpipe_hip)、Don't install gloo 等标题组织周边说明或配置。
- **EN:** CMake commands like endif, if, message, caffe2_update_option, else, set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 endif、if、message、caffe2_update_option、else、set 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1228-1246 / 第 1228-1246 行

```cmake
    # Build BFloat16 cuda kernels
    set(GLOO_USE_TORCH_DTYPES 1)
    set(GLOO_TORCH_DIR ${PROJECT_SOURCE_DIR} ${CMAKE_BINARY_DIR})

    # Temporarily override variables to avoid building Gloo tests/benchmarks
    set(__BUILD_TEST ${BUILD_TEST})
    set(__BUILD_BENCHMARK ${BUILD_BENCHMARK})
    set(BUILD_TEST OFF)
    set(BUILD_BENCHMARK OFF)
    if(USE_ROCM)
      set(ENV{GLOO_ROCM_ARCH} "${PYTORCH_ROCM_ARCH}")
    endif()
    if(NOT USE_SYSTEM_GLOO)
      if(USE_DISTRIBUED AND USE_TENSORPIPE)
        get_target_property(_include_dirs uv_a INCLUDE_DIRECTORIES)
        set_target_properties(uv_a PROPERTIES INTERFACE_INCLUDE_DIRECTORIES "${_include_dirs}")
      endif()
      set(GLOO_USE_CUDA_TOOLKIT ON CACHE BOOL "" FORCE)

```

- **EN:** This chunk introduces sections such as Build BFloat16 cuda kernels, Temporarily override variables to avoid building Gloo tests/benchmarks, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Build BFloat16 cuda kernels、Temporarily override variables to avoid building Gloo tests/benchmarks 等标题组织周边说明或配置。
- **EN:** CMake commands like set, if, endif, get_target_property, set_target_properties drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、if、endif、get_target_property、set_target_properties 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1247-1277 / 第 1247-1277 行

```cmake
      # Disable NCCL/RCCL since we don't use Gloo+NCCL, make sure to re-enable it!
      set(USE_NCCL_SAVED ${USE_NCCL})
      set(USE_RCCL_SAVED ${USE_RCCL})
      set(USE_NCCL OFF)
      set(USE_RCCL OFF)
      add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/../third_party/gloo)
      set(USE_NCCL ${USE_NCCL_SAVED})
      set(USE_RCCL ${USE_RCCL_SAVED})

      # Here is a little bit hacky. We have to put PROJECT_BINARY_DIR in front
      # of PROJECT_SOURCE_DIR with/without conda system. The reason is that
      # gloo generates a new config.h in the binary directory.
      include_directories(BEFORE SYSTEM ${CMAKE_CURRENT_LIST_DIR}/../third_party/gloo)
      include_directories(BEFORE SYSTEM ${PROJECT_BINARY_DIR}/third_party/gloo)
    else()
      find_package(Gloo)
      if(NOT Gloo_FOUND)
        message(FATAL_ERROR "Cannot find gloo")
      endif()
      message("Found gloo: ${Gloo_NATIVE_LIBRARY}, cuda lib: ${Gloo_CUDA_LIBRARY}, hip lib: ${Gloo_HIP_LIBRARY}")
      message("Found gloo include directories: ${Gloo_INCLUDE_DIRS}")
      add_library(gloo SHARED IMPORTED)
      set_target_properties(gloo PROPERTIES IMPORTED_LOCATION ${Gloo_NATIVE_LIBRARY})
      if(USE_CUDA)
        add_library(gloo_cuda SHARED IMPORTED)
        set_target_properties(gloo_cuda PROPERTIES IMPORTED_LOCATION ${Gloo_CUDA_LIBRARY})
      elseif(USE_ROCM)
        add_library(gloo_hip SHARED IMPORTED)
        set_target_properties(gloo_hip PROPERTIES IMPORTED_LOCATION ${Gloo_HIP_LIBRARY})
      endif()
      # need to use Gloo_INCLUDE_DIRS over third_party/gloo to find Gloo's auto-generated config.h
```

- **EN:** This chunk introduces sections such as Disable NCCL/RCCL since we don't use Gloo+NCCL, make sure to re-enable it!, Here is a little bit hacky. We have to put PROJECT_BINARY_DIR in front, of PROJECT_SOURCE_DIR with/without conda system. The reason is that, gloo generates a new config.h in the binary directory., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Disable NCCL/RCCL since we don't use Gloo+NCCL, make sure to re-enable it!、Here is a little bit hacky. We have to put PROJECT_BINARY_DIR in front、of PROJECT_SOURCE_DIR with/without conda system. The reason is that、gloo generates a new config.h in the binary directory. 等标题组织周边说明或配置。
- **EN:** CMake commands like set, add_subdirectory, include_directories, else, find_package, if drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、add_subdirectory、include_directories、else、find_package、if 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1278-1299 / 第 1278-1299 行

```cmake
      include_directories(BEFORE SYSTEM ${Gloo_INCLUDE_DIRS})
    endif()
    set(BUILD_TEST ${__BUILD_TEST})
    set(BUILD_BENCHMARK ${__BUILD_BENCHMARK})

    # Add explicit dependency since NCCL is built from third_party.
    # Without dependency, make -jN with N>1 can fail if the NCCL build
    # hasn't finished when CUDA targets are linked.
    if(NOT USE_SYSTEM_NCCL AND USE_NCCL AND NOT USE_ROCM)
      add_dependencies(gloo_cuda nccl_external)
    endif()
    # Pick the right dependency depending on USE_CUDA
    list(APPEND Caffe2_DEPENDENCY_LIBS gloo)
    if(USE_CUDA)
      list(APPEND Caffe2_CUDA_DEPENDENCY_LIBS gloo_cuda)
    elseif(USE_ROCM)
      list(APPEND Caffe2_HIP_DEPENDENCY_LIBS gloo_hip)
    endif()
    add_compile_options(-DCAFFE2_USE_GLOO)
  endif()
endif()

```

- **EN:** This chunk introduces sections such as Add explicit dependency since NCCL is built from third_party., Without dependency, make -jN with N>1 can fail if the NCCL build, hasn't finished when CUDA targets are linked., Pick the right dependency depending on USE_CUDA, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Add explicit dependency since NCCL is built from third_party.、Without dependency, make -jN with N>1 can fail if the NCCL build、hasn't finished when CUDA targets are linked.、Pick the right dependency depending on USE_CUDA 等标题组织周边说明或配置。
- **EN:** CMake commands like include_directories, endif, set, if, add_dependencies, list drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 include_directories、endif、set、if、add_dependencies、list 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1300-1322 / 第 1300-1322 行

```cmake
# ---[ profiling
if(USE_PROF)
  find_package(htrace)
  if(htrace_FOUND)
    set(USE_PROF_HTRACE ON)
  else()
    message(WARNING "htrace not found. Caffe2 will build without htrace prof")
  endif()
endif()

if(USE_SNPE AND ANDROID)
  if(SNPE_LOCATION AND SNPE_HEADERS)
    message(STATUS "Using SNPE location specified by -DSNPE_LOCATION: " ${SNPE_LOCATION})
    message(STATUS "Using SNPE headers specified by -DSNPE_HEADERS: " ${SNPE_HEADERS})
    include_directories(SYSTEM ${SNPE_HEADERS})
    add_library(snpe SHARED IMPORTED)
    set_property(TARGET snpe PROPERTY IMPORTED_LOCATION ${SNPE_LOCATION})
    list(APPEND Caffe2_DEPENDENCY_LIBS snpe)
  else()
    caffe2_update_option(USE_SNPE OFF)
  endif()
endif()

```

- **EN:** This chunk introduces sections such as ---[ profiling, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ profiling 等标题组织周边说明或配置。
- **EN:** CMake commands like if, find_package, set, else, message, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、find_package、set、else、message、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1323-1338 / 第 1323-1338 行

```cmake
if(USE_NNAPI AND NOT ANDROID)
  message(WARNING "NNApi is only used in android builds.")
  caffe2_update_option(USE_NNAPI OFF)
endif()

# ---[ Onnx
if(CAFFE2_CMAKE_BUILDING_WITH_MAIN_REPO AND NOT INTERN_DISABLE_ONNX)
  if(EXISTS "${CAFFE2_CUSTOM_PROTOC_EXECUTABLE}")
    set(ONNX_CUSTOM_PROTOC_EXECUTABLE ${CAFFE2_CUSTOM_PROTOC_EXECUTABLE})
  endif()
  set(TEMP_BUILD_SHARED_LIBS ${BUILD_SHARED_LIBS})
  set(BUILD_SHARED_LIBS OFF)
  set(ONNX_USE_MSVC_STATIC_RUNTIME ${CAFFE2_USE_MSVC_STATIC_RUNTIME})
  set(ONNX_USE_LITE_PROTO ${CAFFE2_USE_LITE_PROTO})
  # If linking local protobuf, make sure ONNX has the same protobuf
  # patches as Caffe2 and Caffe proto. This forces some functions to
```

- **EN:** This chunk introduces sections such as ---[ Onnx, If linking local protobuf, make sure ONNX has the same protobuf, patches as Caffe2 and Caffe proto. This forces some functions to, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ Onnx、If linking local protobuf, make sure ONNX has the same protobuf、patches as Caffe2 and Caffe proto. This forces some functions to 等标题组织周边说明或配置。
- **EN:** CMake commands like if, message, caffe2_update_option, endif, set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、message、caffe2_update_option、endif、set 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1339-1354 / 第 1339-1354 行

```cmake
  # not be inline and instead route back to the statically-linked protobuf.
  if(CAFFE2_LINK_LOCAL_PROTOBUF)
    set(ONNX_PROTO_POST_BUILD_SCRIPT ${PROJECT_SOURCE_DIR}/cmake/ProtoBufPatch.cmake)
  endif()
  if(ONNX_ML)
    add_definitions(-DONNX_ML=1)
  endif()
  add_definitions(-DONNXIFI_ENABLE_EXT=1)
  set(Python3_EXECUTABLE "${Python_EXECUTABLE}")
  if(NOT USE_SYSTEM_ONNX)
    add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/../third_party/onnx EXCLUDE_FROM_ALL)
  endif()

  add_definitions(-DONNX_NAMESPACE=${ONNX_NAMESPACE})
  if(NOT USE_SYSTEM_ONNX)
    # In mobile build we care about code size, and so we need drop
```

- **EN:** This chunk introduces sections such as not be inline and instead route back to the statically-linked protobuf., In mobile build we care about code size, and so we need drop, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 not be inline and instead route back to the statically-linked protobuf.、In mobile build we care about code size, and so we need drop 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, endif, add_definitions, add_subdirectory drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、endif、add_definitions、add_subdirectory 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1355-1378 / 第 1355-1378 行

```cmake
    # everything (e.g. checker) in onnx but the pb definition.
    if(ANDROID OR IOS)
      caffe2_interface_library(onnx_proto onnx_library)
    else()
      caffe2_interface_library(onnx onnx_library)
    endif()
    list(APPEND Caffe2_DEPENDENCY_WHOLE_LINK_LIBS onnx_library)
  else()
    add_library(onnx SHARED IMPORTED)
    find_library(ONNX_LIBRARY onnx)
    if(NOT ONNX_LIBRARY)
      message(FATAL_ERROR "Cannot find onnx")
    endif()
    set_property(TARGET onnx PROPERTY IMPORTED_LOCATION ${ONNX_LIBRARY})
    add_library(onnx_proto SHARED IMPORTED)
    find_library(ONNX_PROTO_LIBRARY onnx_proto)
    if(NOT ONNX_PROTO_LIBRARY)
      message(FATAL_ERROR "Cannot find onnx")
    endif()
    set_property(TARGET onnx_proto PROPERTY IMPORTED_LOCATION ${ONNX_PROTO_LIBRARY})
    message("-- Found onnx: ${ONNX_LIBRARY} ${ONNX_PROTO_LIBRARY}")
    list(APPEND Caffe2_DEPENDENCY_LIBS onnx_proto onnx)
  endif()
  # Recover the build shared libs option.
```

- **EN:** This chunk introduces sections such as everything (e.g. checker) in onnx but the pb definition., Recover the build shared libs option., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 everything (e.g. checker) in onnx but the pb definition.、Recover the build shared libs option. 等标题组织周边说明或配置。
- **EN:** CMake commands like if, caffe2_interface_library, else, endif, list, add_library drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、caffe2_interface_library、else、endif、list、add_library 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1379-1394 / 第 1379-1394 行

```cmake
  set(BUILD_SHARED_LIBS ${TEMP_BUILD_SHARED_LIBS})
endif()

# --[ ATen checks
set(USE_LAPACK 0)

# we need to build all targets to be linked with PIC
if(USE_KINETO AND INTERN_BUILD_MOBILE AND USE_LITE_INTERPRETER_PROFILER)
  set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)
endif()

if(NOT INTERN_BUILD_MOBILE)
  set(TORCH_CUDA_ARCH_LIST $ENV{TORCH_CUDA_ARCH_LIST})
  string(APPEND CMAKE_CUDA_FLAGS " $ENV{TORCH_NVCC_FLAGS}")
  set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)

```

- **EN:** This chunk introduces sections such as --[ ATen checks, we need to build all targets to be linked with PIC, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 --[ ATen checks、we need to build all targets to be linked with PIC 等标题组织周边说明或配置。
- **EN:** CMake commands like set, endif, if, string drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、endif、if、string 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1395-1425 / 第 1395-1425 行

```cmake
  # Top-level build config
  ############################################
  # Flags
  # When using MSVC
  # Detect CUDA architecture and get best NVCC flags
  # finding cuda must be first because other things depend on the result
  #
  # NB: We MUST NOT run this find_package if NOT USE_CUDA is set, because upstream
  # FindCUDA has a bug where it will still attempt to make use of NOTFOUND
  # compiler variables to run various probe tests.  We could try to fix
  # this, but since FindCUDA upstream is subsumed by first-class support
  # for CUDA language, it seemed not worth fixing.

  if(MSVC)
    # we want to respect the standard, and we are bored of those **** .
    add_definitions(-D_CRT_SECURE_NO_DEPRECATE=1)
    string(APPEND CMAKE_CUDA_FLAGS " -Xcompiler=/wd4819,/wd4503,/wd4190,/wd4244,/wd4251,/wd4275,/wd4522")
  else()
    if(WERROR)
      if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" AND ${CMAKE_CXX_COMPILER_VERSION} VERSION_GREATER_EQUAL 13)
        string(APPEND CMAKE_CUDA_FLAGS " -Xcompiler -Wno-dangling-reference ")
      endif()
      if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        string(APPEND CMAKE_CUDA_FLAGS " -Xcompiler -Wno-extra-semi ")
      endif()
      if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR (CMAKE_CXX_COMPILER_ID STREQUAL "Clang" AND ${CMAKE_CXX_COMPILER_VERSION} VERSION_GREATER_EQUAL 13))
        string(APPEND CMAKE_CUDA_FLAGS " -Xcompiler -Werror -Xcompiler -Wno-error=sign-compare ")
      endif()
    endif()
  endif()

```

- **EN:** This chunk introduces sections such as Top-level build config, , Flags, When using MSVC, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Top-level build config、、Flags、When using MSVC 等标题组织周边说明或配置。
- **EN:** CMake commands like if, add_definitions, string, else, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、add_definitions、string、else、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1426-1441 / 第 1426-1441 行

```cmake
  string(APPEND CMAKE_CUDA_FLAGS " -Wno-deprecated-gpu-targets --expt-extended-lambda")

  # use cub in a safe manner, see:
  # https://github.com/pytorch/pytorch/pull/55292
  string(APPEND CMAKE_CUDA_FLAGS " -DCUB_WRAPPED_NAMESPACE=at_cuda_detail")

  # Suppress cusparse warnings
  string(APPEND CMAKE_CUDA_FLAGS " -DDISABLE_CUSPARSE_DEPRECATED")

  message(STATUS "Found CUDA with FP16 support, compiling with torch.cuda.HalfTensor")
  string(APPEND CMAKE_CUDA_FLAGS " -DCUDA_HAS_FP16=1"
                                 " -D__CUDA_NO_HALF_OPERATORS__"
                                 " -D__CUDA_NO_HALF_CONVERSIONS__"
                                 " -D__CUDA_NO_HALF2_OPERATORS__"
                                 " -D__CUDA_NO_BFLOAT16_CONVERSIONS__")

```

- **EN:** This chunk introduces sections such as use cub in a safe manner, see:, https://github.com/pytorch/pytorch/pull/55292, Suppress cusparse warnings, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 use cub in a safe manner, see:、https://github.com/pytorch/pytorch/pull/55292、Suppress cusparse warnings 等标题组织周边说明或配置。
- **EN:** CMake commands like string, message drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 string、message 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 1442-1457 / 第 1442-1457 行

```cmake
  string(APPEND CMAKE_C_FLAGS_RELEASE " -DNDEBUG")
  string(APPEND CMAKE_CXX_FLAGS_RELEASE " -DNDEBUG")
  if(NOT GENERATOR_IS_MULTI_CONFIG)
    if(${CMAKE_BUILD_TYPE} STREQUAL "Release")
      message(STATUS "Adding -DNDEBUG to compile flags")
      string(APPEND CMAKE_C_FLAGS " -DNDEBUG")
      string(APPEND CMAKE_CXX_FLAGS " -DNDEBUG")
    else()
      message(STATUS "Removing -DNDEBUG from compile flags")
      string(REGEX REPLACE "[-/]DNDEBUG" "" CMAKE_C_FLAGS "" ${CMAKE_C_FLAGS})
      string(REGEX REPLACE "[-/]DNDEBUG" "" CMAKE_CXX_FLAGS "" ${CMAKE_CXX_FLAGS})
    endif()
  endif()
  string(REGEX REPLACE "[-/]DNDEBUG" "" CMAKE_C_FLAGS_DEBUG "" ${CMAKE_C_FLAGS_DEBUG})
  string(REGEX REPLACE "[-/]DNDEBUG" "" CMAKE_CXX_FLAGS_DEBUG "" ${CMAKE_CXX_FLAGS_DEBUG})

```

- **EN:** CMake commands like string, if, message, else, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 string、if、message、else、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1458-1479 / 第 1458-1479 行

```cmake
  set(CUDA_ATTACH_VS_BUILD_RULE_TO_CUDA_FILE OFF)

  if(USE_CUDA OR USE_ROCM)
    if(USE_MAGMA)
      find_package(MAGMA)
      if(MAGMA_FOUND)
        message(STATUS "Compiling with MAGMA support")
        message(STATUS "MAGMA INCLUDE DIRECTORIES: ${MAGMA_INCLUDE_DIR}")
        message(STATUS "MAGMA LIBRARIES: ${MAGMA_LIBRARIES}")
        message(STATUS "MAGMA V2 check: ${MAGMA_V2}")
      else()
        message(STATUS "MAGMA not found. Compiling without MAGMA support")
        caffe2_update_option(USE_MAGMA OFF)
      endif()
    endif()
  elseif(USE_MAGMA)
    message(WARNING
      "Not compiling with MAGMA. Suppress this warning with "
      "-DUSE_MAGMA=OFF.")
    caffe2_update_option(USE_MAGMA OFF)
  endif()

```

- **EN:** CMake commands like set, if, find_package, message, else, caffe2_update_option drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、if、find_package、message、else、caffe2_update_option 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1480-1495 / 第 1480-1495 行

```cmake
  # ARM specific flags
  find_package(ARM)

  find_package(LAPACK)
  if(LAPACK_FOUND)
    set(USE_LAPACK 1)
    list(APPEND Caffe2_PRIVATE_DEPENDENCY_LIBS ${LAPACK_LIBRARIES})
  endif()

  if(NOT USE_CUDA)
    message("disabling CUDA because NOT USE_CUDA is set")
    set(AT_CUDA_ENABLED 0)
  else()
    set(AT_CUDA_ENABLED 1)
  endif()

```

- **EN:** This chunk introduces sections such as ARM specific flags, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ARM specific flags 等标题组织周边说明或配置。
- **EN:** CMake commands like find_package, if, set, list, endif, message drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 find_package、if、set、list、endif、message 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1496-1527 / 第 1496-1527 行

```cmake
  if(NOT USE_ROCM)
    message("disabling ROCM because NOT USE_ROCM is set")
    message(STATUS "MIOpen not found. Compiling without MIOpen support")
    set(AT_ROCM_ENABLED 0)
  else()
    include_directories(BEFORE ${MIOPEN_INCLUDE_DIRS})
    set(AT_ROCM_ENABLED 1)
  endif()

  if(USE_MKLDNN)
    if(NOT CMAKE_SIZEOF_VOID_P EQUAL 8)
      message(WARNING
        "x64 operating system is required for MKLDNN. "
        "Not compiling with MKLDNN. "
        "Turn this warning off by USE_MKLDNN=OFF.")
      set(USE_MKLDNN OFF)
    endif()
    if(USE_MKLDNN_ACL)
      set(AT_MKLDNN_ACL_ENABLED 1)
    endif()
  endif()
  if(USE_MKLDNN)
    include(${CMAKE_CURRENT_LIST_DIR}/public/mkldnn.cmake)
    if(MKLDNN_FOUND)
      set(AT_MKLDNN_ENABLED 1)
      include_directories(AFTER SYSTEM ${MKLDNN_INCLUDE_DIR})
    else()
      message(WARNING "MKLDNN could not be found.")
      caffe2_update_option(USE_MKLDNN OFF)
    endif()
  else()
    message("disabling MKLDNN because USE_MKLDNN is not set")
```

- **EN:** CMake commands like if, message, set, else, include_directories, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、message、set、else、include_directories、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1528-1544 / 第 1528-1544 行

```cmake
  endif()

  if(USE_KLEIDIAI)
    set(TEMP_BUILD_SHARED_LIBS ${BUILD_SHARED_LIBS})
    set(BUILD_SHARED_LIBS OFF CACHE BOOL "Build shared libs" FORCE)
    set(AT_KLEIDIAI_ENABLED 1)
    set(KLEIDIAI_BUILD_TESTS OFF) # Disable building KLEIDIAI tests
    set(KLEIDIAI_SRC "${PROJECT_SOURCE_DIR}/third_party/kleidiai")
    add_subdirectory(${KLEIDIAI_SRC})
    list(APPEND Caffe2_DEPENDENCY_LIBS kleidiai)
    # Recover build options.
    set(BUILD_SHARED_LIBS ${TEMP_BUILD_SHARED_LIBS} CACHE BOOL "Build shared libs" FORCE)
  endif()

  if(UNIX AND NOT APPLE)
     include(CheckLibraryExists)
     # https://github.com/libgit2/libgit2/issues/2128#issuecomment-35649830
```

- **EN:** This chunk introduces sections such as Recover build options., https://github.com/libgit2/libgit2/issues/2128#issuecomment-35649830, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Recover build options.、https://github.com/libgit2/libgit2/issues/2128#issuecomment-35649830 等标题组织周边说明或配置。
- **EN:** CMake commands like endif, if, set, add_subdirectory, list, include drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 endif、if、set、add_subdirectory、list、include 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1545-1576 / 第 1545-1576 行

```cmake
     CHECK_LIBRARY_EXISTS(rt clock_gettime "time.h" NEED_LIBRT)
     if(NEED_LIBRT)
       list(APPEND Caffe2_DEPENDENCY_LIBS rt)
       set(CMAKE_REQUIRED_LIBRARIES ${CMAKE_REQUIRED_LIBRARIES} rt)
     endif(NEED_LIBRT)
  endif(UNIX AND NOT APPLE)

  if(UNIX)
    set(CMAKE_EXTRA_INCLUDE_FILES "sys/mman.h")
    CHECK_FUNCTION_EXISTS(mmap HAVE_MMAP)
    if(HAVE_MMAP)
      add_definitions(-DHAVE_MMAP=1)
    endif(HAVE_MMAP)
    # done for lseek: https://www.gnu.org/software/libc/manual/html_node/File-Position-Primitive.html
    add_definitions(-D_FILE_OFFSET_BITS=64)
    CHECK_FUNCTION_EXISTS(shm_open HAVE_SHM_OPEN)
    if(HAVE_SHM_OPEN)
      add_definitions(-DHAVE_SHM_OPEN=1)
    endif(HAVE_SHM_OPEN)
    CHECK_FUNCTION_EXISTS(shm_unlink HAVE_SHM_UNLINK)
    if(HAVE_SHM_UNLINK)
      add_definitions(-DHAVE_SHM_UNLINK=1)
    endif(HAVE_SHM_UNLINK)
    CHECK_FUNCTION_EXISTS(malloc_usable_size HAVE_MALLOC_USABLE_SIZE)
    if(HAVE_MALLOC_USABLE_SIZE)
      add_definitions(-DHAVE_MALLOC_USABLE_SIZE=1)
    endif(HAVE_MALLOC_USABLE_SIZE)
    set(CMAKE_EXTRA_INCLUDE_FILES "fcntl.h")
    CHECK_FUNCTION_EXISTS(posix_fallocate HAVE_POSIX_FALLOCATE)
    if(HAVE_POSIX_FALLOCATE)
      add_definitions(-DHAVE_POSIX_FALLOCATE=1)
    endif(HAVE_POSIX_FALLOCATE)
```

- **EN:** This chunk introduces sections such as done for lseek: https://www.gnu.org/software/libc/manual/html_node/File-Position-Primitive.html, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 done for lseek: https://www.gnu.org/software/libc/manual/html_node/File-Position-Primitive.html 等标题组织周边说明或配置。
- **EN:** CMake commands like CHECK_LIBRARY_EXISTS, if, list, set, endif, CHECK_FUNCTION_EXISTS drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 CHECK_LIBRARY_EXISTS、if、list、set、endif、CHECK_FUNCTION_EXISTS 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1577-1592 / 第 1577-1592 行

```cmake
  endif(UNIX)

  add_definitions(-DUSE_EXTERNAL_MZCRC)
  add_definitions(-DMINIZ_DISABLE_ZIP_READER_CRC32_CHECKS)

  find_package(ZVECTOR) # s390x simd support
endif()

#
# End ATen checks
#

# Install `fmtlib` header.
# This was the default behavior before version 12.0.0.
# Since PyTorch C API depends on it, make it available for projects that
# depend on PyTorch.
```

- **EN:** This chunk introduces sections such as , End ATen checks, , Install `fmtlib` header., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 、End ATen checks、、Install `fmtlib` header. 等标题组织周边说明或配置。
- **EN:** CMake commands like endif, add_definitions, find_package drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 endif、add_definitions、find_package 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1593-1609 / 第 1593-1609 行

```cmake
set(FMT_INSTALL ON)
set(TEMP_BUILD_SHARED_LIBS ${BUILD_SHARED_LIBS})
set(BUILD_SHARED_LIBS OFF CACHE BOOL "Build shared libs" FORCE)
add_subdirectory(${PROJECT_SOURCE_DIR}/third_party/fmt)

# Disable compiler feature checks for `fmt`.
#
# CMake compiles a little program to check compiler features. Some of our build
# configurations (notably the mobile build analyzer) will populate
# CMAKE_CXX_FLAGS in ways that break feature checks. Since we already know
# `fmt` is compatible with a superset of the compilers that PyTorch is, it
# shouldn't be too bad to just disable the checks.
set_target_properties(fmt-header-only PROPERTIES INTERFACE_COMPILE_FEATURES "")

list(APPEND Caffe2_DEPENDENCY_LIBS fmt::fmt-header-only)
set(BUILD_SHARED_LIBS ${TEMP_BUILD_SHARED_LIBS} CACHE BOOL "Build shared libs" FORCE)

```

- **EN:** This chunk introduces sections such as Disable compiler feature checks for `fmt`., , CMake compiles a little program to check compiler features. Some of our build, configurations (notably the mobile build analyzer) will populate, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Disable compiler feature checks for `fmt`.、、CMake compiles a little program to check compiler features. Some of our build、configurations (notably the mobile build analyzer) will populate 等标题组织周边说明或配置。
- **EN:** CMake commands like set, add_subdirectory, set_target_properties, list drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、add_subdirectory、set_target_properties、list 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 1610-1629 / 第 1610-1629 行

```cmake
# ---[ Kineto
# edge profiler depends on KinetoProfiler but it only does cpu
# profiling. Thus we dont need USE_CUDA/USE_ROCM
if(USE_KINETO AND INTERN_BUILD_MOBILE AND NOT (BUILD_LITE_INTERPRETER AND USE_LITE_INTERPRETER_PROFILER))
  message(STATUS "Not using libkineto in a mobile build.")
  set(USE_KINETO OFF)
endif()

if(USE_KINETO AND INTERN_BUILD_MOBILE AND USE_LITE_INTERPRETER_PROFILER AND (USE_CUDA OR USE_ROCM))
  message(FATAL_ERROR "Mobile build with profiler does not support CUDA or ROCM")
endif()

if(USE_KINETO)
  if(NOT USE_CUDA)
    set(LIBKINETO_NOCUPTI ON CACHE STRING "" FORCE)
  else()
    set(LIBKINETO_NOCUPTI OFF CACHE STRING "")
    message(STATUS "Using Kineto with CUPTI support")
  endif()

```

- **EN:** This chunk introduces sections such as ---[ Kineto, edge profiler depends on KinetoProfiler but it only does cpu, profiling. Thus we dont need USE_CUDA/USE_ROCM, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ Kineto、edge profiler depends on KinetoProfiler but it only does cpu、profiling. Thus we dont need USE_CUDA/USE_ROCM 等标题组织周边说明或配置。
- **EN:** CMake commands like if, message, set, endif, else drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、message、set、endif、else 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1630-1647 / 第 1630-1647 行

```cmake
  if(NOT USE_ROCM)
    set(LIBKINETO_NOROCTRACER ON CACHE STRING "" FORCE)
  else()
    set(LIBKINETO_NOROCTRACER OFF CACHE STRING "")
    message(STATUS "Using Kineto with Roctracer support")
  endif()

  if((NOT USE_XPU) OR (NOT XPU_ENABLE_KINETO))
    set(LIBKINETO_NOXPUPTI ON CACHE STRING "" FORCE)
  else()
    set(LIBKINETO_NOXPUPTI OFF CACHE STRING "")
    message(STATUS "Using Kineto with XPUPTI support")
  endif()

  if(LIBKINETO_NOCUPTI AND LIBKINETO_NOROCTRACER AND LIBKINETO_NOXPUPTI)
    message(STATUS "Using CPU-only version of Kineto")
  endif()

```

- **EN:** CMake commands like if, set, else, message, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、else、message、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1648-1663 / 第 1648-1663 行

```cmake
  set(CAFFE2_THIRD_PARTY_ROOT "${PROJECT_SOURCE_DIR}/third_party" CACHE STRING "")
  set(KINETO_SOURCE_DIR "${CAFFE2_THIRD_PARTY_ROOT}/kineto/libkineto" CACHE STRING "")
  set(KINETO_BUILD_TESTS OFF CACHE BOOL "")
  set(KINETO_LIBRARY_TYPE "static" CACHE STRING "")

  message(STATUS "Configuring Kineto dependency:")
  message(STATUS "  KINETO_SOURCE_DIR = ${KINETO_SOURCE_DIR}")
  message(STATUS "  KINETO_BUILD_TESTS = ${KINETO_BUILD_TESTS}")
  message(STATUS "  KINETO_LIBRARY_TYPE = ${KINETO_LIBRARY_TYPE}")

  if(NOT LIBKINETO_NOROCTRACER)
    if("$ENV{ROCM_SOURCE_DIR}" STREQUAL "")
      set(ENV{ROCM_SOURCE_DIR} "${ROCM_PATH}")
    endif()
  endif()

```

- **EN:** CMake commands like set, message, if, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、message、if、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1664-1687 / 第 1664-1687 行

```cmake
  if(NOT TARGET kineto)
    add_subdirectory("${KINETO_SOURCE_DIR}")
    set_property(TARGET kineto PROPERTY POSITION_INDEPENDENT_CODE ON)
  endif()
  list(APPEND Caffe2_DEPENDENCY_LIBS kineto)
  string(APPEND CMAKE_CXX_FLAGS " -DUSE_KINETO")
  if(LIBKINETO_NOCUPTI)
    string(APPEND CMAKE_CXX_FLAGS " -DLIBKINETO_NOCUPTI")
  endif()
  if(LIBKINETO_NOROCTRACER)
    string(APPEND CMAKE_CXX_FLAGS " -DLIBKINETO_NOROCTRACER")
  endif()
  if(LIBKINETO_NOXPUPTI)
    string(APPEND CMAKE_CXX_FLAGS " -DLIBKINETO_NOXPUPTI=ON")
  else()
    string(APPEND CMAKE_CXX_FLAGS " -DLIBKINETO_NOXPUPTI=OFF")
  endif()
  if(LIBKINETO_NOCUPTI AND LIBKINETO_NOROCTRACER AND LIBKINETO_NOXPUPTI)
    message(STATUS "Configured Kineto (CPU)")
  else()
    message(STATUS "Configured Kineto")
  endif()
endif()

```

- **EN:** CMake commands like if, add_subdirectory, set_property, endif, list, string drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、add_subdirectory、set_property、endif、list、string 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1688-1701 / 第 1688-1701 行

```cmake
# Include google/FlatBuffers
include(${CMAKE_CURRENT_LIST_DIR}/FlatBuffers.cmake)

# Include cpp-httplib
add_library(httplib INTERFACE IMPORTED)
target_include_directories(httplib SYSTEM INTERFACE ${PROJECT_SOURCE_DIR}/third_party/cpp-httplib)

# Include nlohmann-json
add_library(nlohmann INTERFACE IMPORTED)
include_directories(nlohmann SYSTEM INTERFACE ${PROJECT_SOURCE_DIR}/third_party/nlohmann/include)

# Include moodycamel
add_library(moodycamel INTERFACE IMPORTED)
include_directories(moodycamel SYSTEM INTERFACE ${PROJECT_SOURCE_DIR}/third_party/concurrentqueue)
```

- **EN:** This chunk introduces sections such as Include google/FlatBuffers, Include cpp-httplib, Include nlohmann-json, Include moodycamel, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Include google/FlatBuffers、Include cpp-httplib、Include nlohmann-json、Include moodycamel 等标题组织周边说明或配置。
- **EN:** CMake commands like include, add_library, target_include_directories, include_directories drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 include、add_library、target_include_directories、include_directories 等 CMake 命令完成依赖探测、变量设置或平台检查。

## Key Concepts / 关键概念

- **CMake build configuration** — CMake 构建配置
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **Device coverage** — 突出 CPU/CUDA/移动端等后端相关执行路径。
- **Android integration** — 聚焦 Android 构建、打包或运行时集成细节。
- **Benchmark workflow** — 记录或实现性能测量场景。
- **CMake build logic** — 控制构建系统配置、特性检查或依赖发现。
- **External package discovery** — 搜索 PyTorch 构建所需的外部库。
- **Structured metadata** — 存放机器可读的设置或清单式元数据。
- **Representative symbols: if, set, else, endif, macro, string, endmacro, include** — 代表性符号：if、set、else、endif、macro、string、endmacro、include

## Dependencies / 依赖关系

- `${CMAKE_CURRENT_LIST_DIR}/public/cuda.cmake`
- `${CMAKE_CURRENT_LIST_DIR}/public/xpu.cmake`
- `${CMAKE_CURRENT_LIST_DIR}/ProtoBuf.cmake`
- `${CMAKE_CURRENT_LIST_DIR}/public/mkl.cmake`
- `cmake/BLAS_ABI.cmake`
- `${CMAKE_CURRENT_LIST_DIR}/External/EigenBLAS.cmake`
- `${CMAKE_CURRENT_LIST_DIR}/External/nnpack.cmake`
- `${CMAKE_CURRENT_LIST_DIR}/VulkanDependencies.cmake`
- `${CMAKE_CURRENT_LIST_DIR}/public/gflags.cmake`
- `${CMAKE_CURRENT_LIST_DIR}/public/glog.cmake`
- `${CMAKE_CURRENT_LIST_DIR}/Modules/FindOpenMP.cmake`
- `${CMAKE_CURRENT_LIST_DIR}/public/LoadHIP.cmake`
- `${CMAKE_CURRENT_LIST_DIR}/External/nccl.cmake`
- `${CMAKE_CURRENT_LIST_DIR}/External/rccl.cmake`
- `${CMAKE_CURRENT_LIST_DIR}/External/ucc.cmake`
- `${CMAKE_CURRENT_LIST_DIR}/public/mkldnn.cmake`
- `CheckLibraryExists`
- `${CMAKE_CURRENT_LIST_DIR}/FlatBuffers.cmake`
- `Sanitizer`
- `Threads`
- `Atlas`
- `OpenBLAS`
- `BLIS`
- `MKL`
