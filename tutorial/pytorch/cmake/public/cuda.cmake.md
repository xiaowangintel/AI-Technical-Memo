# cuda.cmake — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `cmake/public/cuda.cmake`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines public-facing CMake helpers that wire optional backends, toolchains, or third-party libraries into the build.
- **用途 (CN)**: 定义面向外部的 CMake 辅助逻辑，把可选后端、工具链或第三方库接入构建系统。

## Content Analysis / 内容分析
### Lines 1-22 / 第 1-22 行

````cmake
# ---[ cuda

# Poor man's include guard
if(TARGET torch::cudart)
  return()
endif()

# sccache is only supported in CMake master and not in the newest official
# release (3.11.3) yet. Hence we need our own Modules_CUDA_fix to enable sccache.
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/../Modules_CUDA_fix)

# We don't want to statically link cudart, because we rely on it's dynamic linkage in
# python (follow along torch/cuda/__init__.py and usage of cudaGetErrorName).
# Technically, we can link cudart here statically, and link libtorch_python.so
# to a dynamic libcudart.so, but that's just wasteful.
# However, on Windows, if this one gets switched off, the error "cuda: unknown error"
# will be raised when running the following code:
# >>> import torch
# >>> torch.cuda.is_available()
# >>> torch.cuda.current_device()
# More details can be found in the following links.
# https://github.com/pytorch/pytorch/issues/20635
````

- EN: This section checks discovery results before exposing them to the wider build.
- CN: 该部分在对外暴露结果前检查发现状态。

### Lines 23-38 / 第 23-38 行

````cmake
# https://github.com/pytorch/pytorch/issues/17108
if(NOT MSVC)
  set(CUDA_USE_STATIC_CUDA_RUNTIME OFF CACHE INTERNAL "")
endif()

# Find CUDA.
find_package(CUDA)
if(NOT CUDA_FOUND)
  # If user explicitly set USE_CUDA=1, error out instead of falling back
  if(_USE_CUDA_EXPLICITLY_SET AND USE_CUDA)
    message(FATAL_ERROR
      "PyTorch: CUDA was explicitly requested (USE_CUDA=1) but cannot be found. "
      "Please check your CUDA installation, ensure CUDA toolkit is installed, "
      "and that CUDA_HOME or CMAKE_CUDA_COMPILER is set correctly. "
      "If you want to build without CUDA, please set USE_CUDA=0.")
  endif()
````

- EN: This section defines configure-time variables and search paths; probes the host system for headers, libraries, or SDK components; checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分定义配置阶段变量与搜索路径；探测主机系统中的头文件、库或 SDK 组件；在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 40-60 / 第 40-60 行

````cmake
  message(WARNING
    "PyTorch: CUDA cannot be found. Depending on whether you are building "
    "PyTorch or a PyTorch dependent library, the next warning / error will "
    "give you more info.")
  set(CAFFE2_USE_CUDA OFF)
  return()
endif()

# Enable CUDA language support
set(CUDAToolkit_ROOT "${CUDA_TOOLKIT_ROOT_DIR}")
# Pass clang as host compiler, which according to the docs
# Must be done before CUDA language is enabled, see
# https://cmake.org/cmake/help/v3.15/variable/CMAKE_CUDA_HOST_COMPILER.html
if("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
  set(CMAKE_CUDA_HOST_COMPILER "${CMAKE_CXX_COMPILER}")
endif()
enable_language(CUDA)
if("X${CMAKE_CUDA_STANDARD}" STREQUAL "X" )
  set(CMAKE_CUDA_STANDARD ${CMAKE_CXX_STANDARD})
endif()
set(CMAKE_CUDA_STANDARD_REQUIRED ON)
````

- EN: This section defines configure-time variables and search paths; checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分定义配置阶段变量与搜索路径；在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 62-83 / 第 62-83 行

````cmake
# CMP0074 - find_package will respect <PackageName>_ROOT variables
cmake_policy(PUSH)
if(CMAKE_VERSION VERSION_GREATER_EQUAL 3.12.0)
  cmake_policy(SET CMP0074 NEW)
endif()

find_package(CUDAToolkit REQUIRED)

cmake_policy(POP)

if(NOT CMAKE_CUDA_COMPILER_VERSION VERSION_EQUAL CUDAToolkit_VERSION)
  message(FATAL_ERROR "Found two conflicting CUDA versions:\n"
                      "V${CMAKE_CUDA_COMPILER_VERSION} in '${CUDA_INCLUDE_DIRS}' and\n"
                      "V${CUDAToolkit_VERSION} in '${CUDAToolkit_INCLUDE_DIRS}'")
endif()

message(STATUS "PyTorch: CUDA detected: " ${CUDA_VERSION})
message(STATUS "PyTorch: CUDA nvcc is: " ${CUDA_NVCC_EXECUTABLE})
message(STATUS "PyTorch: CUDA toolkit directory: " ${CUDA_TOOLKIT_ROOT_DIR})
if(CUDA_VERSION VERSION_LESS 12.1)
  message(FATAL_ERROR "PyTorch requires CUDA 12.1 or above.")
endif()
````

- EN: This section probes the host system for headers, libraries, or SDK components; checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分探测主机系统中的头文件、库或 SDK 组件；在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 85-108 / 第 85-108 行

````cmake
if(CUDA_FOUND)
  # Sometimes, we may mismatch nvcc with the CUDA headers we are
  # compiling with, e.g., if a ccache nvcc is fed to us by CUDA_NVCC_EXECUTABLE
  # but the PATH is not consistent with CUDA_HOME.  It's better safe
  # than sorry: make sure everything is consistent.
  if(MSVC AND CMAKE_GENERATOR MATCHES "Visual Studio")
    # When using Visual Studio, it attempts to lock the whole binary dir when
    # `try_run` is called, which will cause the build to fail.
    string(RANDOM BUILD_SUFFIX)
    set(PROJECT_RANDOM_BINARY_DIR "${PROJECT_BINARY_DIR}/${BUILD_SUFFIX}")
  else()
    set(PROJECT_RANDOM_BINARY_DIR "${PROJECT_BINARY_DIR}")
  endif()
  set(file "${PROJECT_BINARY_DIR}/detect_cuda_version.cc")
  file(WRITE ${file} ""
    "#include <cuda.h>\n"
    "#include <cstdio>\n"
    "int main() {\n"
    "  printf(\"%d.%d\", CUDA_VERSION / 1000, (CUDA_VERSION / 10) % 100);\n"
    "  return 0;\n"
    "}\n"
    )
  if(NOT CMAKE_CROSSCOMPILING)
    try_run(run_result compile_result ${PROJECT_RANDOM_BINARY_DIR} ${file}
````

- EN: This section defines configure-time variables and search paths; checks discovery results before exposing them to the wider build.
- CN: 该部分定义配置阶段变量与搜索路径；在对外暴露结果前检查发现状态。

### Lines 109-125 / 第 109-125 行

````cmake
      CMAKE_FLAGS "-DINCLUDE_DIRECTORIES=${CUDA_INCLUDE_DIRS}"
      LINK_LIBRARIES ${CUDA_LIBRARIES}
      RUN_OUTPUT_VARIABLE cuda_version_from_header
      COMPILE_OUTPUT_VARIABLE output_var
      )
    if(NOT compile_result)
      message(FATAL_ERROR "PyTorch: Couldn't determine version from header: " ${output_var})
    endif()
    message(STATUS "PyTorch: Header version is: " ${cuda_version_from_header})
    if(NOT cuda_version_from_header STREQUAL ${CUDA_VERSION_STRING})
      # Force CUDA to be processed for again next time
      # TODO: I'm not sure if this counts as an implementation detail of
      # FindCUDA
      set(cuda_version_from_findcuda ${CUDA_VERSION_STRING})
      unset(CUDA_TOOLKIT_ROOT_DIR_INTERNAL CACHE)
      # Not strictly necessary, but for good luck.
      unset(CUDA_VERSION CACHE)
````

- EN: This section defines configure-time variables and search paths; checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分定义配置阶段变量与搜索路径；在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 126-149 / 第 126-149 行

````cmake
      # Error out
      message(FATAL_ERROR "FindCUDA says CUDA version is ${cuda_version_from_findcuda} (usually determined by nvcc), "
        "but the CUDA headers say the version is ${cuda_version_from_header}.  This often occurs "
        "when you set both CUDA_HOME and CUDA_NVCC_EXECUTABLE to "
        "non-standard locations, without also setting PATH to point to the correct nvcc.  "
        "Perhaps, try re-running this command again with PATH=${CUDA_TOOLKIT_ROOT_DIR}/bin:$PATH.  "
        "See above log messages for more diagnostics, and see https://github.com/pytorch/pytorch/issues/8092 for more details.")
    endif()
  endif()
endif()

# ---[ CUDA libraries wrapper

# find lbnvrtc.so
set(CUDA_NVRTC_LIB "${CUDA_nvrtc_LIBRARY}" CACHE FILEPATH "")
if(CUDA_NVRTC_LIB AND NOT CUDA_NVRTC_SHORTHASH)
  find_package(Python COMPONENTS Interpreter)
  execute_process(
    COMMAND "${Python_EXECUTABLE}" -c
    "import hashlib;hash=hashlib.sha256();hash.update(open('${CUDA_NVRTC_LIB}','rb').read());print(hash.hexdigest()[:8])"
    RESULT_VARIABLE _retval
    OUTPUT_VARIABLE CUDA_NVRTC_SHORTHASH)
  if(NOT _retval EQUAL 0)
    message(WARNING "Failed to compute shorthash for libnvrtc.so")
````

- EN: This section defines configure-time variables and search paths; probes the host system for headers, libraries, or SDK components; checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分定义配置阶段变量与搜索路径；探测主机系统中的头文件、库或 SDK 组件；在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 150-167 / 第 150-167 行

````cmake
    set(CUDA_NVRTC_SHORTHASH "XXXXXXXX")
  else()
    string(STRIP "${CUDA_NVRTC_SHORTHASH}" CUDA_NVRTC_SHORTHASH)
    message(STATUS "${CUDA_NVRTC_LIB} shorthash is ${CUDA_NVRTC_SHORTHASH}")
  endif()
endif()

# Create new style imported libraries.
# Several of these libraries have a hardcoded path if CAFFE2_STATIC_LINK_CUDA
# is set. This path is where sane CUDA installations have their static
# libraries installed. This flag should only be used for binary builds, so
# end-users should never have this flag set.

# cuda
add_library(caffe2::cuda INTERFACE IMPORTED)
set_property(
    TARGET caffe2::cuda PROPERTY INTERFACE_LINK_LIBRARIES
    CUDA::cuda_driver)
````

- EN: This section defines configure-time variables and search paths; emits status or fatal diagnostics for build users.
- CN: 该部分定义配置阶段变量与搜索路径；向构建用户输出状态或致命诊断信息。

### Lines 169-186 / 第 169-186 行

````cmake
# cudart
add_library(torch::cudart INTERFACE IMPORTED)
if(CAFFE2_STATIC_LINK_CUDA)
    set_property(
        TARGET torch::cudart PROPERTY INTERFACE_LINK_LIBRARIES
        CUDA::cudart_static)
else()
    set_property(
        TARGET torch::cudart PROPERTY INTERFACE_LINK_LIBRARIES
        CUDA::cudart)
endif()


# cublas
add_library(caffe2::cublas INTERFACE IMPORTED)
if(CAFFE2_STATIC_LINK_CUDA AND NOT WIN32)
    set_property(
        TARGET caffe2::cublas PROPERTY INTERFACE_LINK_LIBRARIES
````

- EN: This section checks discovery results before exposing them to the wider build.
- CN: 该部分在对外暴露结果前检查发现状态。

### Lines 187-207 / 第 187-207 行

````cmake
        # NOTE: cublas is always linked dynamically
        CUDA::cublas CUDA::cublasLt)
    set_property(
        TARGET caffe2::cublas APPEND PROPERTY INTERFACE_LINK_LIBRARIES
        CUDA::cudart_static rt)
else()
    set_property(
        TARGET caffe2::cublas PROPERTY INTERFACE_LINK_LIBRARIES
        CUDA::cublas CUDA::cublasLt)
endif()

# cudnn interface
# static linking is handled by USE_STATIC_CUDNN environment variable
if(CAFFE2_USE_CUDNN)
  if(USE_STATIC_CUDNN)
    set(CUDNN_STATIC ON CACHE BOOL "")
  else()
    set(CUDNN_STATIC OFF CACHE BOOL "")
  endif()

  find_package(CUDNN)
````

- EN: This section defines configure-time variables and search paths; probes the host system for headers, libraries, or SDK components; checks discovery results before exposing them to the wider build.
- CN: 该部分定义配置阶段变量与搜索路径；探测主机系统中的头文件、库或 SDK 组件；在对外暴露结果前检查发现状态。

### Lines 209-229 / 第 209-229 行

````cmake
  if(NOT CUDNN_FOUND)
    message(WARNING
      "Cannot find cuDNN library. Turning the option off")
    set(CAFFE2_USE_CUDNN OFF)
  else()
    if(CUDNN_VERSION VERSION_LESS "8.1.0")
      message(FATAL_ERROR "PyTorch requires cuDNN 8.1 and above.")
    endif()
  endif()

  add_library(torch::cudnn INTERFACE IMPORTED)
  target_include_directories(torch::cudnn INTERFACE ${CUDNN_INCLUDE_PATH})
  if(CUDNN_STATIC AND NOT WIN32)
    target_link_options(torch::cudnn INTERFACE
        "-Wl,--exclude-libs,libcudnn_static.a")
  else()
    target_link_libraries(torch::cudnn INTERFACE ${CUDNN_LIBRARY_PATH})
  endif()
else()
  message(STATUS "USE_CUDNN is set to 0. Compiling without cuDNN support")
endif()
````

- EN: This section defines configure-time variables and search paths; checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分定义配置阶段变量与搜索路径；在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 231-248 / 第 231-248 行

````cmake
if(CAFFE2_USE_CUSPARSELT)
  find_package(CUSPARSELT)

  if(NOT CUSPARSELT_FOUND)
    message(WARNING
      "Cannot find cuSPARSELt library. Turning the option off")
    set(CAFFE2_USE_CUSPARSELT OFF)
  else()
    add_library(torch::cusparselt INTERFACE IMPORTED)
    target_include_directories(torch::cusparselt INTERFACE ${CUSPARSELT_INCLUDE_PATH})
    target_link_libraries(torch::cusparselt INTERFACE ${CUSPARSELT_LIBRARY_PATH})
  endif()
else()
  message(STATUS "USE_CUSPARSELT is set to 0. Compiling without cuSPARSELt support")
endif()

if(USE_CUDSS)
  find_package(CUDSS)
````

- EN: This section defines configure-time variables and search paths; probes the host system for headers, libraries, or SDK components; checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分定义配置阶段变量与搜索路径；探测主机系统中的头文件、库或 SDK 组件；在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 250-273 / 第 250-273 行

````cmake
  if(NOT CUDSS_FOUND)
    message(WARNING
      "Cannot find CUDSS library. Turning the option off")
    set(USE_CUDSS OFF)
  else()
    add_library(torch::cudss INTERFACE IMPORTED)
    target_include_directories(torch::cudss INTERFACE ${CUDSS_INCLUDE_PATH})
    target_link_libraries(torch::cudss INTERFACE ${CUDSS_LIBRARY_PATH})
  endif()
else()
  message(STATUS "USE_CUDSS is set to 0. Compiling without cuDSS support")
endif()

# cufile
if(CAFFE2_USE_CUFILE)
  add_library(torch::cufile INTERFACE IMPORTED)
  if(CAFFE2_STATIC_LINK_CUDA AND NOT WIN32)
      set_property(
          TARGET torch::cufile PROPERTY INTERFACE_LINK_LIBRARIES
          CUDA::cuFile_static)
  else()
      set_property(
          TARGET torch::cufile PROPERTY INTERFACE_LINK_LIBRARIES
          CUDA::cuFile)
````

- EN: This section defines configure-time variables and search paths; checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分定义配置阶段变量与搜索路径；在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 274-289 / 第 274-289 行

````cmake
  endif()
else()
  message(STATUS "USE_CUFILE is set to 0. Compiling without cuFile support")
endif()

# curand
add_library(caffe2::curand INTERFACE IMPORTED)
if(CAFFE2_STATIC_LINK_CUDA AND NOT WIN32)
    set_property(
        TARGET caffe2::curand PROPERTY INTERFACE_LINK_LIBRARIES
        CUDA::curand_static)
else()
    set_property(
        TARGET caffe2::curand PROPERTY INTERFACE_LINK_LIBRARIES
        CUDA::curand)
endif()
````

- EN: This section checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 291-311 / 第 291-311 行

````cmake
# cufft
add_library(caffe2::cufft INTERFACE IMPORTED)
if(CAFFE2_STATIC_LINK_CUDA AND NOT WIN32)
    if(CUDA_VERSION VERSION_LESS_EQUAL 12.9)
      set_property(
          TARGET caffe2::cufft PROPERTY INTERFACE_LINK_LIBRARIES
          CUDA::cufft_static_nocallback)
    else()
      set_property(
          TARGET caffe2::cufft PROPERTY INTERFACE_LINK_LIBRARIES
          CUDA::cufft_static)
    endif()
else()
    set_property(
        TARGET caffe2::cufft PROPERTY INTERFACE_LINK_LIBRARIES
        CUDA::cufft)
endif()

# nvrtc
# cuDNN frontend needs libnvrtc symbols, but linking through CUDA::nvrtc pulls
# CUDA::cuda_driver transitively. Keep a driver-free target for cuDNN users and
````

- EN: This section checks discovery results before exposing them to the wider build.
- CN: 该部分在对外暴露结果前检查发现状态。

### Lines 312-334 / 第 312-334 行

````cmake
# reserve caffe2::nvrtc for the stub library that actually needs the driver API.
add_library(caffe2::nvrtc_runtime INTERFACE IMPORTED)
set_property(
    TARGET caffe2::nvrtc_runtime PROPERTY INTERFACE_LINK_LIBRARIES
    "${CUDA_NVRTC_LIB}")

add_library(caffe2::nvrtc INTERFACE IMPORTED)
set_property(
    TARGET caffe2::nvrtc PROPERTY INTERFACE_LINK_LIBRARIES
    CUDA::nvrtc caffe2::cuda)

# Add onnx namespace definition to nvcc
if(ONNX_NAMESPACE)
  list(APPEND CUDA_NVCC_FLAGS "-DONNX_NAMESPACE=${ONNX_NAMESPACE}")
else()
  list(APPEND CUDA_NVCC_FLAGS "-DONNX_NAMESPACE=onnx_c2")
endif()

# Don't activate VC env again for Ninja generators with MSVC on Windows if CUDAHOSTCXX is not defined
# by adding --use-local-env.
if(MSVC AND CMAKE_GENERATOR STREQUAL "Ninja" AND NOT DEFINED ENV{CUDAHOSTCXX})
  list(APPEND CUDA_NVCC_FLAGS "--use-local-env")
endif()
````

- EN: This section checks discovery results before exposing them to the wider build.
- CN: 该部分在对外暴露结果前检查发现状态。

### Lines 336-359 / 第 336-359 行

````cmake
# setting nvcc arch flags
torch_cuda_get_nvcc_gencode_flag(NVCC_FLAGS_EXTRA)
# CMake 3.18 adds integrated support for architecture selection, but we can't rely on it
if(DEFINED CMAKE_CUDA_ARCHITECTURES)
  message(WARNING
          "pytorch is not compatible with `CMAKE_CUDA_ARCHITECTURES` and will ignore its value. "
          "Please configure `TORCH_CUDA_ARCH_LIST` instead.")
  set(CMAKE_CUDA_ARCHITECTURES OFF)
endif()

list(APPEND CUDA_NVCC_FLAGS ${NVCC_FLAGS_EXTRA})
message(STATUS "Added CUDA NVCC flags for: ${NVCC_FLAGS_EXTRA}")

# disable some nvcc diagnostic that appears in boost, glog, glags, opencv, etc.
foreach(diag cc_clobber_ignored
             field_without_dll_interface
             base_class_has_different_dll_interface
             dll_interface_conflict_none_assumed
             dll_interface_conflict_dllexport_assumed
             bad_friend_decl)
  list(APPEND SUPPRESS_WARNING_FLAGS --diag_suppress=${diag})
endforeach()
string(REPLACE ";" "," SUPPRESS_WARNING_FLAGS "${SUPPRESS_WARNING_FLAGS}")
list(APPEND CUDA_NVCC_FLAGS -Xcudafe ${SUPPRESS_WARNING_FLAGS})
````

- EN: This section defines configure-time variables and search paths; checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分定义配置阶段变量与搜索路径；在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。

### Lines 361-384 / 第 361-384 行

````cmake
set(CUDA_PROPAGATE_HOST_FLAGS_BLOCKLIST "-Werror")
if(MSVC)
  list(APPEND CUDA_NVCC_FLAGS "--Werror" "cross-execution-space-call")
  list(APPEND CUDA_NVCC_FLAGS "--no-host-device-move-forward")
endif()

# Debug and Release symbol support
if(MSVC)
  if(${CAFFE2_USE_MSVC_STATIC_RUNTIME})
    string(APPEND CMAKE_CUDA_FLAGS_DEBUG " -Xcompiler /MTd")
    string(APPEND CMAKE_CUDA_FLAGS_MINSIZEREL " -Xcompiler /MT")
    string(APPEND CMAKE_CUDA_FLAGS_RELEASE " -Xcompiler /MT")
    string(APPEND CMAKE_CUDA_FLAGS_RELWITHDEBINFO " -Xcompiler /MT")
  else()
    string(APPEND CMAKE_CUDA_FLAGS_DEBUG " -Xcompiler /MDd")
    string(APPEND CMAKE_CUDA_FLAGS_MINSIZEREL " -Xcompiler /MD")
    string(APPEND CMAKE_CUDA_FLAGS_RELEASE " -Xcompiler /MD")
    string(APPEND CMAKE_CUDA_FLAGS_RELWITHDEBINFO " -Xcompiler /MD")
  endif()
  if(CUDA_NVCC_FLAGS MATCHES "Zi")
    list(APPEND CUDA_NVCC_FLAGS "-Xcompiler" "-FS")
  endif()
elseif(CUDA_DEVICE_DEBUG)
  list(APPEND CUDA_NVCC_FLAGS "-g" "-G")  # -G enables device code debugging symbols
````

- EN: This section defines configure-time variables and search paths; checks discovery results before exposing them to the wider build.
- CN: 该部分定义配置阶段变量与搜索路径；在对外暴露结果前检查发现状态。

### Lines 385-396 / 第 385-396 行

````cmake
endif()

# needed for compat with newer versions of clang that use C++20 mangling rules
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang" AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 18)
  list(APPEND CUDA_NVCC_FLAGS "-Xcompiler=-fclang-abi-compat=17")
endif()

# Set expt-relaxed-constexpr to suppress Eigen warnings
list(APPEND CUDA_NVCC_FLAGS "--expt-relaxed-constexpr")

# Set expt-extended-lambda to support lambda on device
list(APPEND CUDA_NVCC_FLAGS "--expt-extended-lambda")
````

- EN: This section checks discovery results before exposing them to the wider build.
- CN: 该部分在对外暴露结果前检查发现状态。

### Lines 398-404 / 第 398-404 行

````cmake
foreach(FLAG ${CUDA_NVCC_FLAGS})
  string(FIND "${FLAG}" " " flag_space_position)
  if(NOT flag_space_position EQUAL -1)
    message(FATAL_ERROR "Found spaces in CUDA_NVCC_FLAGS entry '${FLAG}'")
  endif()
  string(APPEND CMAKE_CUDA_FLAGS " ${FLAG}")
endforeach()
````

- EN: This section checks discovery results before exposing them to the wider build; emits status or fatal diagnostics for build users.
- CN: 该部分在对外暴露结果前检查发现状态；向构建用户输出状态或致命诊断信息。


## Key Concepts / 关键概念
- EN: Configures native build inputs, optional backends, and dependency discovery at configure time.
  CN: 在配置阶段设置原生构建输入、可选后端以及依赖发现逻辑。
- EN: Discovers external headers, libraries, or SDKs and exposes the resulting variables to later build logic.
  CN: 发现外部头文件、库或 SDK，并将结果变量暴露给后续构建逻辑。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: CMake variable flow — the file publishes configure-time variables that later targets or summaries consume.
  CN: CMake 变量流——该文件发布配置阶段变量，供后续目标或摘要逻辑使用。

## Dependencies / 依赖关系
- Find targets / 查找目标: `CUDA`, `CUDAToolkit`, `Python`, `CUDNN`, `CUSPARSELT`, `CUDSS`
- Exported variables / 导出变量: `TARGET`, `APPEND`, `CMAKE_MODULE_PATH`, `CMAKE_CURRENT_LIST_DIR`, `NOT`, `MSVC`, `CUDA_USE_STATIC_CUDA_RUNTIME`, `OFF`, `CACHE`, `INTERNAL`, ...
- Mentioned paths / 提及路径: `/../Modules_CUDA_fix`, `torch/cuda/__init__.py`, `//github.com/pytorch/pytorch/issues/20635`, `//github.com/pytorch/pytorch/issues/17108`, `//cmake.org/cmake/help/v3.15/variable/CMAKE_CUDA_HOST_COMPILER.html`, `/detect_cuda_version.cc`, `/bin`, `//github.com/pytorch/pytorch/issues/8092`, `/MTd`, `/MT`, ...
