# Codegen.cmake — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `cmake/Codegen.cmake`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines CMake configuration logic that shapes how the PyTorch build is configured. The opening comment frames the file as: "This ill-named file does a number of things: - Installs Caffe2 header files (this has nothing to do with code generation) - Configures caffe2/core/macros.h - Creates an ATen target for its generated C++ files and adds it as a dependency - Reads build lists defined in build_variables.bzl."
- **Purpose (CN)**: 定义影响 PyTorch 构建配置方式的 CMake 逻辑。 开头注释将该文件概括为：“This ill-named file does a number of things: - Installs Caffe2 header files (this has nothing to do with code generation) - Configures caffe2/core/macros.h - Creates an ATen target for its generated C++ files and adds it as a dependency - Reads build lists defined in build_variables.bzl”。

## Content Analysis / 内容分析

### Lines 1-23 / 第 1-23 行

```cmake
# This ill-named file does a number of things:
# - Installs Caffe2 header files (this has nothing to do with code generation)
# - Configures caffe2/core/macros.h
# - Creates an ATen target for its generated C++ files and adds it
#   as a dependency
# - Reads build lists defined in build_variables.bzl

################################################################################
# Helper functions
################################################################################

function(filter_list output input)
    unset(result)
    foreach(filename ${${input}})
        foreach(pattern ${ARGN})
            if("${filename}" MATCHES "${pattern}")
                list(APPEND result "${filename}")
            endif()
        endforeach()
    endforeach()
    set(${output} ${result} PARENT_SCOPE)
endfunction()

```

- **EN:** This chunk introduces sections such as This ill-named file does a number of things:, - Installs Caffe2 header files (this has nothing to do with code generation), - Configures caffe2/core/macros.h, - Creates an ATen target for its generated C++ files and adds it, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 This ill-named file does a number of things:、- Installs Caffe2 header files (this has nothing to do with code generation)、- Configures caffe2/core/macros.h、- Creates an ATen target for its generated C++ files and adds it 等标题组织周边说明或配置。
- **EN:** CMake commands like function, unset, foreach, if, list, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 function、unset、foreach、if、list、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 24-35 / 第 24-35 行

```cmake
function(filter_list_exclude output input)
    unset(result)
    foreach(filename ${${input}})
        foreach(pattern ${ARGN})
            if(NOT "${filename}" MATCHES "${pattern}")
                list(APPEND result "${filename}")
            endif()
        endforeach()
    endforeach()
    set(${output} ${result} PARENT_SCOPE)
endfunction()

```

- **EN:** CMake commands like function, unset, foreach, if, list, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 function、unset、foreach、if、list、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 36-49 / 第 36-49 行

```cmake
################################################################################

# -- [ Determine commit hash
execute_process(
    COMMAND "${Python_EXECUTABLE}" -c "from tools.generate_torch_version import get_sha;print(get_sha('.'), end='')"
    OUTPUT_VARIABLE COMMIT_SHA
    WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/..
)

# ---[ Write the macros file
configure_file(
    ${CMAKE_CURRENT_LIST_DIR}/../caffe2/core/macros.h.in
    ${CMAKE_BINARY_DIR}/caffe2/core/macros.h)

```

- **EN:** This chunk introduces sections such as , -- [ Determine commit hash, ---[ Write the macros file, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 、-- [ Determine commit hash、---[ Write the macros file 等标题组织周边说明或配置。
- **EN:** CMake commands like execute_process, configure_file drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 execute_process、configure_file 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 50-61 / 第 50-61 行

```cmake
# ---[ Installing the header files
install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/../caffe2
        DESTINATION include
        FILES_MATCHING PATTERN "*.h")
if(NOT INTERN_BUILD_ATEN_OPS)
  install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/../aten/src/ATen/core
          DESTINATION include/ATen
          FILES_MATCHING PATTERN "*.h")
endif()
install(FILES ${CMAKE_BINARY_DIR}/caffe2/core/macros.h
        DESTINATION include/caffe2/core)

```

- **EN:** This chunk introduces sections such as ---[ Installing the header files, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ Installing the header files 等标题组织周边说明或配置。
- **EN:** CMake commands like install, if, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 install、if、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 62-76 / 第 62-76 行

```cmake
# ---[ ATen specific
if(INTERN_BUILD_ATEN_OPS)
  if(MSVC)
    set(OPT_FLAG "/fp:strict ")
  else(MSVC)
    set(OPT_FLAG "-O3 ")
    if("${CMAKE_BUILD_TYPE}" MATCHES "Debug")
      set(OPT_FLAG " ")
    endif()
  endif(MSVC)

  if(NOT MSVC AND NOT "${CMAKE_C_COMPILER_ID}" MATCHES "Clang")
    set_source_files_properties(${CMAKE_CURRENT_LIST_DIR}/../aten/src/ATen/MapAllocator.cpp PROPERTIES COMPILE_FLAGS "-fno-openmp")
  endif()

```

- **EN:** This chunk introduces sections such as ---[ ATen specific, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ---[ ATen specific 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, else, endif, set_source_files_properties drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、else、endif、set_source_files_properties 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 77-88 / 第 77-88 行

```cmake
  file(GLOB_RECURSE all_python "${CMAKE_CURRENT_LIST_DIR}/../torchgen/*.py")

  # Handle files that may need sm89/sm90a/sm100a flags (stable/nightly
  # builds are not built for these archs).
  if(USE_CUDA)
    # The stable/nightly builds do not enable some SM architectures,
    # like 89/90a/100a.  Still, some files need to be built for these
    # architectures specifically.  This function makes it possible to
    # enable building given file for a specific such architecture, in
    # case if PyTorch is built for corresponding other architecture;
    # for example, it will enable building for SM 90a in case PyTorch
    # built for SM 90, etc.  For examples of how to use the function,
```

- **EN:** This chunk introduces sections such as Handle files that may need sm89/sm90a/sm100a flags (stable/nightly, builds are not built for these archs)., The stable/nightly builds do not enable some SM architectures,, like 89/90a/100a.  Still, some files need to be built for these, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Handle files that may need sm89/sm90a/sm100a flags (stable/nightly、builds are not built for these archs).、The stable/nightly builds do not enable some SM architectures,、like 89/90a/100a.  Still, some files need to be built for these 等标题组织周边说明或配置。
- **EN:** CMake commands like file, if drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 file、if 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 89-110 / 第 89-110 行

```cmake
    # see below the function itself.
    function(_BUILD_FOR_ADDITIONAL_ARCHS file archs)
      torch_cuda_get_nvcc_gencode_flag(_existing_arch_flags)

      set(_file_compile_flags "")
      foreach(_arch ${archs})
        if("${_arch}" STREQUAL "89")
          if(_existing_arch_flags MATCHES ".*compute_86.*")
            list(APPEND _file_compile_flags "-gencode;arch=compute_89,code=sm_89")
          endif()
        endif()
        if("${_arch}" STREQUAL "90a")
          if(_existing_arch_flags MATCHES ".*compute_90.*")
            list(APPEND _file_compile_flags "-gencode;arch=compute_90a,code=sm_90a")
          endif()
        endif()
        if("${_arch}" STREQUAL "100a")
          if(_existing_arch_flags MATCHES ".*compute_100.*")
            list(APPEND _file_compile_flags "-gencode;arch=compute_100a,code=sm_100a")
          endif()
        endif()
        # We will need to gate against CUDA version, because sm_103a is available on CUDA 12.9+
```

- **EN:** This chunk introduces sections such as see below the function itself., We will need to gate against CUDA version, because sm_103a is available on CUDA 12.9+, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 see below the function itself.、We will need to gate against CUDA version, because sm_103a is available on CUDA 12.9+ 等标题组织周边说明或配置。
- **EN:** CMake commands like function, torch_cuda_get_nvcc_gencode_flag, set, foreach, if, list drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 function、torch_cuda_get_nvcc_gencode_flag、set、foreach、if、list 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 111-127 / 第 111-127 行

```cmake
        if("${_arch}" STREQUAL "103a" AND CUDA_VERSION VERSION_GREATER_EQUAL 12.9)
          if(_existing_arch_flags MATCHES ".*compute_100.*")
            list(APPEND _file_compile_flags "-gencode;arch=compute_103a,code=sm_103a")
          endif()
        endif()
        # We will need to gate against CUDA version, because sm_110a is available on CUDA 13.0+
        if("${_arch}" STREQUAL "110a" AND CUDA_VERSION VERSION_GREATER_EQUAL 13.0)
          if(_existing_arch_flags MATCHES ".*compute_110.*")
            list(APPEND _file_compile_flags "-gencode;arch=compute_110a,code=sm_110a")
          endif()
        endif()
        if("${_arch}" STREQUAL "120a")
          if(_existing_arch_flags MATCHES ".*compute_120.*")
            list(APPEND _file_compile_flags "-gencode;arch=compute_120a,code=sm_120a")
          endif()
        endif()
        # We will need to gate against CUDA version, sm_121a was introduced in CUDA 12.9
```

- **EN:** This chunk introduces sections such as We will need to gate against CUDA version, because sm_110a is available on CUDA 13.0+, We will need to gate against CUDA version, sm_121a was introduced in CUDA 12.9, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 We will need to gate against CUDA version, because sm_110a is available on CUDA 13.0+、We will need to gate against CUDA version, sm_121a was introduced in CUDA 12.9 等标题组织周边说明或配置。
- **EN:** CMake commands like if, list, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、list、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 128-148 / 第 128-148 行

```cmake
        if("${_arch}" STREQUAL "121a" AND CUDA_VERSION VERSION_GREATER_EQUAL 12.9)
          if(_existing_arch_flags MATCHES ".*compute_120.*")
            list(APPEND _file_compile_flags "-gencode;arch=compute_121a,code=sm_121a")
          endif()
        endif()
      endforeach()
      list(JOIN _file_compile_flags " " _file_compile_flags)

      set_source_files_properties(${file} PROPERTIES COMPILE_FLAGS "${_file_compile_flags}")
    endfunction()

    _BUILD_FOR_ADDITIONAL_ARCHS(
      "${CMAKE_CURRENT_LIST_DIR}/../aten/src/ATen/native/cuda/RowwiseScaledMM.cu"
      "89;90a;100a;103a;110a;120a;121a")
    _BUILD_FOR_ADDITIONAL_ARCHS(
      "${CMAKE_CURRENT_LIST_DIR}/../aten/src/ATen/native/cuda/ScaledGroupMM.cu"
      "90a")
    _BUILD_FOR_ADDITIONAL_ARCHS(
      "${CMAKE_CURRENT_LIST_DIR}/../aten/src/ATen/native/cuda/GroupMM.cu"
      "90a;100a;103a;110a")

```

- **EN:** CMake commands like if, list, endif, endforeach, set_source_files_properties, endfunction drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、list、endif、endforeach、set_source_files_properties、endfunction 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 149-160 / 第 149-160 行

```cmake
  endif()

  set(GEN_ROCM_FLAG)
  if(USE_ROCM)
    set(GEN_ROCM_FLAG --rocm)
  endif()

  set(GEN_MPS_FLAG)
  if(USE_MPS)
    set(GEN_MPS_FLAG --mps)
  endif()

```

- **EN:** CMake commands like endif, set, if drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 endif、set、if 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 161-179 / 第 161-179 行

```cmake
  set(GEN_XPU_FLAG)
  if(USE_XPU)
    set(GEN_XPU_FLAG --xpu)
  endif()

  set(GEN_MTIA_FLAG)
  if(USE_MTIA)
    set(GEN_MTIA_FLAG --mtia)
  endif()

  set(CUSTOM_BUILD_FLAGS)
  if(INTERN_BUILD_MOBILE)
    if(USE_VULKAN)
      list(APPEND CUSTOM_BUILD_FLAGS --backend_whitelist CPU QuantizedCPU Vulkan)
    else()
      list(APPEND CUSTOM_BUILD_FLAGS --backend_whitelist CPU QuantizedCPU)
    endif()
  endif()

```

- **EN:** CMake commands like set, if, endif, list, else drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、if、endif、list、else 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 180-193 / 第 180-193 行

```cmake
  if(SELECTED_OP_LIST)
    if(TRACING_BASED)
      message(STATUS "Running tracing-based selective build given operator list: ${SELECTED_OP_LIST}")
      list(APPEND CUSTOM_BUILD_FLAGS
        --op_selection_yaml_path ${SELECTED_OP_LIST})
    elseif(NOT STATIC_DISPATCH_BACKEND)
      message(WARNING
        "You have to run tracing-based selective build with dynamic dispatch.\n"
        "Switching to STATIC_DISPATCH_BACKEND=CPU."
      )
      set(STATIC_DISPATCH_BACKEND CPU)
    endif()
  endif()

```

- **EN:** CMake commands like if, message, list, elseif, set, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、message、list、elseif、set、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 194-217 / 第 194-217 行

```cmake
  if(STATIC_DISPATCH_BACKEND)
    message(STATUS "Custom build with static dispatch backends: ${STATIC_DISPATCH_BACKEND}")
    list(LENGTH STATIC_DISPATCH_BACKEND len)
    list(APPEND CUSTOM_BUILD_FLAGS
      --static_dispatch_backend ${STATIC_DISPATCH_BACKEND})
  endif()

  # Codegen unboxing
  if(USE_LIGHTWEIGHT_DISPATCH)
    file(GLOB_RECURSE all_unboxing_script "${CMAKE_CURRENT_LIST_DIR}/../tools/jit/*.py")
    list(APPEND CUSTOM_BUILD_FLAGS --skip_dispatcher_op_registration)
    set(GEN_UNBOXING_COMMAND
        "${Python_EXECUTABLE}" -m tools.jit.gen_unboxing
        --source-path ${CMAKE_CURRENT_LIST_DIR}/../aten/src/ATen
        --install_dir ${CMAKE_BINARY_DIR}/aten/src/ATen
        )
    if(SELECTED_OP_LIST)
      list(APPEND GEN_UNBOXING_COMMAND
              --TEST_ONLY_op_registration_allowlist_yaml_path "${SELECTED_OP_LIST}")
    endif()
    set("GEN_UNBOXING_COMMAND_sources"
        ${GEN_UNBOXING_COMMAND}
        --output-dependencies ${CMAKE_BINARY_DIR}/aten/src/ATen/generated_unboxing_sources.cmake
        )
```

- **EN:** This chunk introduces sections such as Codegen unboxing, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Codegen unboxing 等标题组织周边说明或配置。
- **EN:** CMake commands like if, message, list, endif, file, set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、message、list、endif、file、set 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 218-241 / 第 218-241 行

```cmake
    message(STATUS "Generating sources for lightweight dispatch")
    execute_process(
        COMMAND ${GEN_UNBOXING_COMMAND_sources} --dry-run
        RESULT_VARIABLE RETURN_VALUE
        WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/..
    )
    if(NOT RETURN_VALUE EQUAL 0)
      message(FATAL_ERROR "Failed to get generated_unboxing_sources list")
    endif()

    include("${CMAKE_BINARY_DIR}/aten/src/ATen/generated_unboxing_sources.cmake")
    add_custom_command(
        COMMENT "Generating ATen unboxing sources"
        OUTPUT
        ${generated_unboxing_sources}
        ${CMAKE_BINARY_DIR}/aten/src/ATen/generated_unboxing_sources.cmake
        COMMAND ${GEN_UNBOXING_COMMAND_sources}
        DEPENDS ${all_unboxing_script} ${sources_templates}
        ${CMAKE_CURRENT_LIST_DIR}/../aten/src/ATen/native/native_functions.yaml
        ${CMAKE_CURRENT_LIST_DIR}/../aten/src/ATen/native/tags.yaml
        WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/..
    )
  else() # Otherwise do not generate or include sources into build.
    set(generated_unboxing_sources "")
```

- **EN:** CMake commands like message, execute_process, if, endif, include, add_custom_command drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 message、execute_process、if、endif、include、add_custom_command 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 242-260 / 第 242-260 行

```cmake
  endif()

  set(GEN_PER_OPERATOR_FLAG)
  if(USE_PER_OPERATOR_HEADERS)
    list(APPEND GEN_PER_OPERATOR_FLAG "--per-operator-headers")
  endif()

  set(GEN_COMMAND
      "${Python_EXECUTABLE}" -m torchgen.gen
      --source-path ${CMAKE_CURRENT_LIST_DIR}/../aten/src/ATen
      --install_dir ${CMAKE_BINARY_DIR}/aten/src/ATen
      ${GEN_PER_OPERATOR_FLAG}
      ${GEN_ROCM_FLAG}
      ${GEN_MPS_FLAG}
      ${GEN_XPU_FLAG}
      ${GEN_MTIA_FLAG}
      ${CUSTOM_BUILD_FLAGS}
  )

```

- **EN:** CMake commands like endif, set, if, list drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 endif、set、if、list 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 261-272 / 第 261-272 行

```cmake
  file(GLOB_RECURSE headers_templates "${CMAKE_CURRENT_LIST_DIR}/../aten/src/ATen/templates/*\.h")
  file(GLOB_RECURSE sources_templates "${CMAKE_CURRENT_LIST_DIR}/../aten/src/ATen/templates/*\.cpp")
  set(declarations_yaml_templates "")

  foreach(gen_type "headers" "sources" "declarations_yaml")
    # The codegen outputs may change dynamically as PyTorch is
    # developed, but add_custom_command only supports dynamic inputs.
    #
    # We work around this by generating a .cmake file which is
    # included below to set the list of output files. If that file
    # ever changes then cmake will be re-run automatically because it
    # was included and so we get fully dynamic outputs.
```

- **EN:** This chunk introduces sections such as The codegen outputs may change dynamically as PyTorch is, developed, but add_custom_command only supports dynamic inputs., , We work around this by generating a .cmake file which is, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 The codegen outputs may change dynamically as PyTorch is、developed, but add_custom_command only supports dynamic inputs.、、We work around this by generating a .cmake file which is 等标题组织周边说明或配置。
- **EN:** CMake commands like file, set, foreach drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 file、set、foreach 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 273-286 / 第 273-286 行

```cmake

    set("GEN_COMMAND_${gen_type}"
        ${GEN_COMMAND}
        --generate ${gen_type}
        --output-dependencies ${CMAKE_BINARY_DIR}/aten/src/ATen/generated_${gen_type}.cmake
    )

    # Dry run to bootstrap the output variables
    execute_process(
        COMMAND ${GEN_COMMAND_${gen_type}} --dry-run
        RESULT_VARIABLE RETURN_VALUE
        WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/..
    )

```

- **EN:** This chunk introduces sections such as Dry run to bootstrap the output variables, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Dry run to bootstrap the output variables 等标题组织周边说明或配置。
- **EN:** CMake commands like set, execute_process drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、execute_process 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 287-310 / 第 287-310 行

```cmake
    if(NOT RETURN_VALUE EQUAL 0)
      message(FATAL_ERROR "Failed to get generated_${gen_type} list")
    endif()

    include("${CMAKE_BINARY_DIR}/aten/src/ATen/generated_${gen_type}.cmake")
    include("${CMAKE_BINARY_DIR}/aten/src/ATen/core_generated_${gen_type}.cmake")
    include("${CMAKE_BINARY_DIR}/aten/src/ATen/cpu_vec_generated_${gen_type}.cmake")
    include("${CMAKE_BINARY_DIR}/aten/src/ATen/cuda_generated_${gen_type}.cmake")
    include("${CMAKE_BINARY_DIR}/aten/src/ATen/ops_generated_${gen_type}.cmake")
    if(USE_XPU)
        include("${CMAKE_BINARY_DIR}/aten/src/ATen/xpu_generated_${gen_type}.cmake")
    endif()
    message(STATUS "${gen_type} outputs: ${gen_outputs}")
    set(OUTPUT_LIST
      ${generated_${gen_type}}
      ${cuda_generated_${gen_type}}
      ${core_generated_${gen_type}}
      ${cpu_vec_generated_${gen_type}}
      ${ops_generated_${gen_type}}
      ${CMAKE_BINARY_DIR}/aten/src/ATen/generated_${gen_type}.cmake
      ${CMAKE_BINARY_DIR}/aten/src/ATen/ops_generated_${gen_type}.cmake
      ${CMAKE_BINARY_DIR}/aten/src/ATen/core_generated_${gen_type}.cmake
      ${CMAKE_BINARY_DIR}/aten/src/ATen/cpu_vec_generated_${gen_type}.cmake
      ${CMAKE_BINARY_DIR}/aten/src/ATen/cuda_generated_${gen_type}.cmake)
```

- **EN:** CMake commands like if, message, endif, include, set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、message、endif、include、set 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 311-328 / 第 311-328 行

```cmake
    if(USE_XPU)
      list(APPEND OUTPUT_LIST
        ${xpu_generated_${gen_type}}
        ${CMAKE_BINARY_DIR}/aten/src/ATen/xpu_generated_${gen_type}.cmake
      )
    endif()

    add_custom_command(
      COMMENT "Generating ATen ${gen_type}"
      OUTPUT ${OUTPUT_LIST}
      COMMAND ${GEN_COMMAND_${gen_type}}
      DEPENDS ${all_python} ${${gen_type}_templates}
        ${CMAKE_CURRENT_LIST_DIR}/../aten/src/ATen/native/native_functions.yaml
        ${CMAKE_CURRENT_LIST_DIR}/../aten/src/ATen/native/tags.yaml
      WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/..
    )
  endforeach()

```

- **EN:** CMake commands like if, list, endif, add_custom_command, endforeach drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、list、endif、add_custom_command、endforeach 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 329-342 / 第 329-342 行

```cmake
  # Generated headers used from a CUDA (.cu) file are
  # not tracked correctly in CMake. We make the libATen.so depend explicitly
  # on building the generated ATen files to workaround.
  add_custom_target(ATEN_CPU_FILES_GEN_TARGET DEPENDS
      ${generated_headers} ${core_generated_headers} ${cpu_vec_generated_headers} ${ops_generated_headers}
      ${generated_sources} ${core_generated_sources} ${cpu_vec_generated_sources} ${ops_generated_sources}
      ${generated_declarations_yaml} ${generated_unboxing_sources})
  add_custom_target(ATEN_CUDA_FILES_GEN_TARGET DEPENDS
      ${cuda_generated_headers} ${cuda_generated_sources})
  add_library(ATEN_CPU_FILES_GEN_LIB INTERFACE)
  add_library(ATEN_CUDA_FILES_GEN_LIB INTERFACE)
  add_dependencies(ATEN_CPU_FILES_GEN_LIB ATEN_CPU_FILES_GEN_TARGET)
  add_dependencies(ATEN_CUDA_FILES_GEN_LIB ATEN_CUDA_FILES_GEN_TARGET)

```

- **EN:** This chunk introduces sections such as Generated headers used from a CUDA (.cu) file are, not tracked correctly in CMake. We make the libATen.so depend explicitly, on building the generated ATen files to workaround., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Generated headers used from a CUDA (.cu) file are、not tracked correctly in CMake. We make the libATen.so depend explicitly、on building the generated ATen files to workaround. 等标题组织周边说明或配置。
- **EN:** CMake commands like add_custom_target, add_library, add_dependencies drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 add_custom_target、add_library、add_dependencies 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 343-358 / 第 343-358 行

```cmake
  if(USE_PER_OPERATOR_HEADERS)
    target_compile_definitions(ATEN_CPU_FILES_GEN_LIB INTERFACE AT_PER_OPERATOR_HEADERS)
    target_compile_definitions(ATEN_CUDA_FILES_GEN_LIB INTERFACE AT_PER_OPERATOR_HEADERS)
  endif()

  if(USE_XPU)
    add_custom_target(ATEN_XPU_FILES_GEN_TARGET DEPENDS
        ${xpu_generated_headers} ${xpu_generated_sources})
    add_library(ATEN_XPU_FILES_GEN_LIB INTERFACE)
    add_dependencies(ATEN_XPU_FILES_GEN_LIB ATEN_XPU_FILES_GEN_TARGET)

    if(USE_PER_OPERATOR_HEADERS)
      target_compile_definitions(ATEN_XPU_FILES_GEN_LIB INTERFACE AT_PER_OPERATOR_HEADERS)
    endif()
  endif()
  # Handle source files that need to be compiled multiple times for
```

- **EN:** This chunk introduces sections such as Handle source files that need to be compiled multiple times for, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Handle source files that need to be compiled multiple times for 等标题组织周边说明或配置。
- **EN:** CMake commands like if, target_compile_definitions, endif, add_custom_target, add_library, add_dependencies drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、target_compile_definitions、endif、add_custom_target、add_library、add_dependencies 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 359-374 / 第 359-374 行

```cmake
  # different vectorization options
  file(GLOB cpu_kernel_cpp_in "${PROJECT_SOURCE_DIR}/aten/src/ATen/native/cpu/*.cpp" "${PROJECT_SOURCE_DIR}/aten/src/ATen/native/quantized/cpu/kernels/*.cpp")

  list(APPEND CPU_CAPABILITY_NAMES "DEFAULT")
  list(APPEND CPU_CAPABILITY_FLAGS "${OPT_FLAG}")

  if(CXX_AVX512_FOUND)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DHAVE_AVX512_CPU_DEFINITION")
    list(APPEND CPU_CAPABILITY_NAMES "AVX512")
    if(MSVC)
      list(APPEND CPU_CAPABILITY_FLAGS "${OPT_FLAG}/arch:AVX512")
    else(MSVC)
      list(APPEND CPU_CAPABILITY_FLAGS "${OPT_FLAG} -mavx512f -mavx512bw -mavx512vl -mavx512dq -mfma")
    endif(MSVC)
  endif(CXX_AVX512_FOUND)

```

- **EN:** This chunk introduces sections such as different vectorization options, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 different vectorization options 等标题组织周边说明或配置。
- **EN:** CMake commands like file, list, if, set, else, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 file、list、if、set、else、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 375-386 / 第 375-386 行

```cmake
  if(CXX_AVX2_FOUND)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DHAVE_AVX2_CPU_DEFINITION")

    # Some versions of GCC pessimistically split unaligned load and store
    # instructions when using the default tuning. This is a bad choice on
    # new Intel and AMD processors so we disable it when compiling with AVX2.
    # See https://stackoverflow.com/questions/52626726/why-doesnt-gcc-resolve-mm256-loadu-pd-as-single-vmovupd#tab-top
    check_cxx_compiler_flag("-mno-avx256-split-unaligned-load -mno-avx256-split-unaligned-store" COMPILER_SUPPORTS_NO_AVX256_SPLIT)
    if(COMPILER_SUPPORTS_NO_AVX256_SPLIT)
      set(CPU_NO_AVX256_SPLIT_FLAGS "-mno-avx256-split-unaligned-load -mno-avx256-split-unaligned-store")
    endif(COMPILER_SUPPORTS_NO_AVX256_SPLIT)

```

- **EN:** This chunk introduces sections such as Some versions of GCC pessimistically split unaligned load and store, instructions when using the default tuning. This is a bad choice on, new Intel and AMD processors so we disable it when compiling with AVX2., See https://stackoverflow.com/questions/52626726/why-doesnt-gcc-resolve-mm256-loadu-pd-as-single-vmovupd#tab-top, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Some versions of GCC pessimistically split unaligned load and store、instructions when using the default tuning. This is a bad choice on、new Intel and AMD processors so we disable it when compiling with AVX2.、See https://stackoverflow.com/questions/52626726/why-doesnt-gcc-resolve-mm256-loadu-pd-as-single-vmovupd#tab-top 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, check_cxx_compiler_flag, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、check_cxx_compiler_flag、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 387-407 / 第 387-407 行

```cmake
    list(APPEND CPU_CAPABILITY_NAMES "AVX2")
    if(DEFINED ENV{ATEN_AVX512_256})
      if($ENV{ATEN_AVX512_256} MATCHES "TRUE")
        if(CXX_AVX512_FOUND)
          message("-- ATen AVX2 kernels will use 32 ymm registers")
          if(MSVC)
            list(APPEND CPU_CAPABILITY_FLAGS "${OPT_FLAG}/arch:AVX512")
          else(MSVC)
            list(APPEND CPU_CAPABILITY_FLAGS "${OPT_FLAG} -march=native ${CPU_NO_AVX256_SPLIT_FLAGS}")
          endif(MSVC)
        endif(CXX_AVX512_FOUND)
      endif()
    else()
      if(MSVC)
        list(APPEND CPU_CAPABILITY_FLAGS "${OPT_FLAG}/arch:AVX2")
      else(MSVC)
        list(APPEND CPU_CAPABILITY_FLAGS "${OPT_FLAG} -mavx2 -mfma -mf16c ${CPU_NO_AVX256_SPLIT_FLAGS}")
      endif(MSVC)
    endif()
  endif(CXX_AVX2_FOUND)

```

- **EN:** CMake commands like list, if, message, else, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 list、if、message、else、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 408-419 / 第 408-419 行

```cmake
  if(CXX_VSX_FOUND)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DHAVE_VSX_CPU_DEFINITION")
    LIST(APPEND CPU_CAPABILITY_NAMES "VSX")
    LIST(APPEND CPU_CAPABILITY_FLAGS "${OPT_FLAG}  ${CXX_VSX_FLAGS}")
  endif(CXX_VSX_FOUND)

  if(CXX_ZVECTOR_FOUND)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DHAVE_ZVECTOR_CPU_DEFINITION")
    LIST(APPEND CPU_CAPABILITY_NAMES "ZVECTOR")
    LIST(APPEND CPU_CAPABILITY_FLAGS "${OPT_FLAG}  ${CXX_ZVECTOR_FLAGS}")
  endif(CXX_ZVECTOR_FOUND)

```

- **EN:** CMake commands like if, SET, LIST, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、SET、LIST、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 420-435 / 第 420-435 行

```cmake
  if(CXX_SVE256_FOUND)
    list(APPEND CPU_CAPABILITY_NAMES "SVE256")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DHAVE_SVE_CPU_DEFINITION")
    list(APPEND CPU_CAPABILITY_FLAGS "${OPT_FLAG} -march=armv8-a+sve+bf16 -D__ARM_FEATURE_BF16 -msve-vector-bits=256")
  endif()

  list(LENGTH CPU_CAPABILITY_NAMES NUM_CPU_CAPABILITY_NAMES)
  math(EXPR NUM_CPU_CAPABILITY_NAMES "${NUM_CPU_CAPABILITY_NAMES}-1")

  # The sources list might get reordered later based on the capabilities.
  # See NOTE [ Linking AVX and non-AVX files ]
  foreach(i RANGE ${NUM_CPU_CAPABILITY_NAMES})
    function(process_vec NAME)
      list(GET CPU_CAPABILITY_NAMES ${i} CPU_CAPABILITY)
      set(NEW_IMPL ${CMAKE_BINARY_DIR}/aten/src/ATen/${NAME}.${CPU_CAPABILITY}.cpp)
      # IMPL is absolute here; make it relative to NEW_IMPL's directory so the
```

- **EN:** This chunk introduces sections such as The sources list might get reordered later based on the capabilities., See NOTE [ Linking AVX and non-AVX files ], IMPL is absolute here; make it relative to NEW_IMPL's directory so the, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 The sources list might get reordered later based on the capabilities.、See NOTE [ Linking AVX and non-AVX files ]、IMPL is absolute here; make it relative to NEW_IMPL's directory so the 等标题组织周边说明或配置。
- **EN:** CMake commands like if, list, set, endif, math, foreach drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、list、set、endif、math、foreach 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 436-448 / 第 436-448 行

```cmake
      # generated #include is worktree-independent (ccache/re-cc friendly).
      if(USE_RELATIVE_PATHS)
        file(RELATIVE_PATH IMPL "${CMAKE_BINARY_DIR}/aten/src/ATen" "${IMPL}")
      endif()
      configure_file("${PROJECT_SOURCE_DIR}/cmake/IncludeSource.cpp.in" ${NEW_IMPL})
      set(cpu_kernel_cpp ${NEW_IMPL} ${cpu_kernel_cpp} PARENT_SCOPE) # Create list of copies
      list(GET CPU_CAPABILITY_FLAGS ${i} FLAGS)
      if(MSVC)
        set(EXTRA_FLAGS "/DCPU_CAPABILITY=${CPU_CAPABILITY} /DCPU_CAPABILITY_${CPU_CAPABILITY}")
      else(MSVC)
        set(EXTRA_FLAGS "-DCPU_CAPABILITY=${CPU_CAPABILITY} -DCPU_CAPABILITY_${CPU_CAPABILITY}")
      endif(MSVC)

```

- **EN:** This chunk introduces sections such as generated #include is worktree-independent (ccache/re-cc friendly)., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 generated #include is worktree-independent (ccache/re-cc friendly). 等标题组织周边说明或配置。
- **EN:** CMake commands like if, file, endif, configure_file, set, list drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、file、endif、configure_file、set、list 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 449-461 / 第 449-461 行

```cmake
      # Only parallelize the SortingKernel for now to avoid side effects
      if(${NAME} STREQUAL "native/cpu/SortingKernel.cpp" AND NOT MSVC AND USE_OMP)
        string(APPEND EXTRA_FLAGS " -D_GLIBCXX_PARALLEL")
      endif()

      # Disable certain warnings for GCC-9.X
      if(CMAKE_COMPILER_IS_GNUCXX)
        if(("${NAME}" STREQUAL "native/cpu/GridSamplerKernel.cpp") AND ("${CPU_CAPABILITY}" STREQUAL "DEFAULT"))
          # See https://github.com/pytorch/pytorch/issues/38855
          set(EXTRA_FLAGS "${EXTRA_FLAGS} -Wno-uninitialized")
        endif()
        if("${NAME}" STREQUAL "native/quantized/cpu/kernels/QuantizedOpKernels.cpp")
          # See https://github.com/pytorch/pytorch/issues/38854
```

- **EN:** This chunk introduces sections such as Only parallelize the SortingKernel for now to avoid side effects, Disable certain warnings for GCC-9.X, See https://github.com/pytorch/pytorch/issues/38855, See https://github.com/pytorch/pytorch/issues/38854, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Only parallelize the SortingKernel for now to avoid side effects、Disable certain warnings for GCC-9.X、See https://github.com/pytorch/pytorch/issues/38855、See https://github.com/pytorch/pytorch/issues/38854 等标题组织周边说明或配置。
- **EN:** CMake commands like if, string, endif, set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、string、endif、set 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 462-478 / 第 462-478 行

```cmake
          set(EXTRA_FLAGS "${EXTRA_FLAGS} -Wno-deprecated-copy")
        endif()
      endif()
      set_source_files_properties(${NEW_IMPL} PROPERTIES COMPILE_FLAGS "${FLAGS} ${EXTRA_FLAGS}")
    endfunction()
    foreach(IMPL ${cpu_kernel_cpp_in})
      file(RELATIVE_PATH NAME "${PROJECT_SOURCE_DIR}/aten/src/ATen/" "${IMPL}")
      process_vec("${NAME}")
    endforeach()
    foreach(IMPL ${cpu_vec_generated_sources})
      file(RELATIVE_PATH NAME "${CMAKE_BINARY_DIR}/aten/src/ATen/" "${IMPL}")
      process_vec("${NAME}")
    endforeach()
  endforeach()
  list(APPEND ATen_CPU_SRCS ${cpu_kernel_cpp})
endif()

```

- **EN:** CMake commands like set, endif, set_source_files_properties, endfunction, foreach, file drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、endif、set_source_files_properties、endfunction、foreach、file 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 479-498 / 第 479-498 行

```cmake
function(append_filelist name outputvar)
  set(_rootdir "${Torch_SOURCE_DIR}/")
  # configure_file adds its input to the list of CMAKE_RERUN dependencies
  configure_file(
      ${PROJECT_SOURCE_DIR}/build_variables.bzl
      ${PROJECT_BINARY_DIR}/caffe2/build_variables.bzl)
  execute_process(
    COMMAND "${Python_EXECUTABLE}" -c
            "exec(open('${PROJECT_SOURCE_DIR}/build_variables.bzl').read());print(';'.join(['${_rootdir}' + x for x in ${name}]))"
    WORKING_DIRECTORY "${_rootdir}"
    RESULT_VARIABLE _retval
    OUTPUT_VARIABLE _tempvar)
  if(NOT _retval EQUAL 0)
    message(FATAL_ERROR "Failed to fetch filelist ${name} from build_variables.bzl")
  endif()
  string(REPLACE "\n" "" _tempvar "${_tempvar}")
  list(APPEND ${outputvar} ${_tempvar})
  set(${outputvar} "${${outputvar}}" PARENT_SCOPE)
endfunction()

```

- **EN:** This chunk introduces sections such as configure_file adds its input to the list of CMAKE_RERUN dependencies, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 configure_file adds its input to the list of CMAKE_RERUN dependencies 等标题组织周边说明或配置。
- **EN:** CMake commands like function, set, configure_file, execute_process, if, message drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 function、set、configure_file、execute_process、if、message 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 499-500 / 第 499-500 行

```cmake
set(NUM_CPU_CAPABILITY_NAMES ${NUM_CPU_CAPABILITY_NAMES} PARENT_SCOPE)
set(CPU_CAPABILITY_FLAGS ${CPU_CAPABILITY_FLAGS} PARENT_SCOPE)
```

- **EN:** CMake commands like set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set 等 CMake 命令完成依赖探测、变量设置或平台检查。

## Key Concepts / 关键概念

- **CMake build configuration** — CMake 构建配置
- **CMake build logic** — 控制构建系统配置、特性检查或依赖发现。
- **Declarative configuration** — 以声明式格式表示构建或工作流设置。
- **Representative symbols: function, unset, foreach, if, list, endif, endforeach, set** — 代表性符号：function、unset、foreach、if、list、endif、endforeach、set

## Dependencies / 依赖关系

- `"${CMAKE_BINARY_DIR}/aten/src/ATen/generated_unboxing_sources.cmake"`
- `"${CMAKE_BINARY_DIR}/aten/src/ATen/generated_${gen_type}.cmake"`
- `"${CMAKE_BINARY_DIR}/aten/src/ATen/core_generated_${gen_type}.cmake"`
- `"${CMAKE_BINARY_DIR}/aten/src/ATen/cpu_vec_generated_${gen_type}.cmake"`
- `"${CMAKE_BINARY_DIR}/aten/src/ATen/cuda_generated_${gen_type}.cmake"`
- `"${CMAKE_BINARY_DIR}/aten/src/ATen/ops_generated_${gen_type}.cmake"`
- `"${CMAKE_BINARY_DIR}/aten/src/ATen/xpu_generated_${gen_type}.cmake"`
