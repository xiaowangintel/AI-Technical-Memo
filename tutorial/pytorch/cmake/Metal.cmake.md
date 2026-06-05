# Metal.cmake — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `cmake/Metal.cmake`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines CMake configuration logic that shapes how the PyTorch build is configured.
- **Purpose (CN)**: 定义影响 PyTorch 构建配置方式的 CMake 逻辑。

## Content Analysis / 内容分析

### Lines 1-9 / 第 1-9 行

```cmake
if(NOT APPLE)
    return()
endif()

set(METAL_CFLAGS -Wall -Wextra -fno-fast-math)
if(WERROR)
    string(APPEND METAL_CFLAGS -Werror)
endif()

```

- **EN:** CMake commands like if, return, endif, set, string drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、return、endif、set、string 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 10-17 / 第 10-17 行

```cmake
function(metal_to_air SRC TARGET FLAGS)
    add_custom_command(COMMAND xcrun metal -c ${SRC} -I ${CMAKE_SOURCE_DIR} -I ${CMAKE_SOURCE_DIR}/aten/src -o ${TARGET} ${FLAGS} ${METAL_CFLAGS}
                       DEPENDS ${SRC}
                       OUTPUT ${TARGET}
                       COMMENT "Compiling ${SRC} to ${TARGET}"
                       VERBATIM)
endfunction()

```

- **EN:** CMake commands like function, add_custom_command, endfunction drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 function、add_custom_command、endfunction 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 18-26 / 第 18-26 行

```cmake
function(air_to_metallib TARGET OBJECTS)
    set(_OBJECTS ${OBJECTS} ${ARGN})
    add_custom_command(COMMAND xcrun metallib -o ${TARGET} ${_OBJECTS}
                       DEPENDS ${_OBJECTS}
                       OUTPUT ${TARGET}
                       COMMENT "Linking ${TARGET}"
                       VERBATIM)
endfunction()

```

- **EN:** CMake commands like function, set, add_custom_command, endfunction drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 function、set、add_custom_command、endfunction 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 27-41 / 第 27-41 行

```cmake
function(metal_to_metallib_h SRC TGT)
    execute_process(COMMAND ${Python_EXECUTABLE} torch/utils/_cpp_embed_headers.py ${SRC}
                    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
                    OUTPUT_VARIABLE SHADER_CONTENT
                    RESULT_VARIABLE _exitcode)
    if(NOT _exitcode EQUAL 0)
        message(FATAL_ERROR "Failed to preprocess Metal shader ${SRC}")
        return()
    endif()
    file(WRITE ${TGT} "#include <ATen/native/mps/OperationUtils.h>\n")
    file(APPEND ${TGT} "static ::at::native::mps::MetalShaderLibrary lib(R\"SHDR(\n")
    file(APPEND ${TGT} "${SHADER_CONTENT}")
    file(APPEND ${TGT} ")SHDR\");\n")
endfunction()

```

- **EN:** CMake commands like function, execute_process, if, message, return, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 function、execute_process、if、message、return、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 42-57 / 第 42-57 行

```cmake
set(BFLOAT_METAL_CODE "
  kernel void inc(device bfloat* ptr,
                   uint idx [[thread_position_in_grid]]) {
    ptr[idx] += 1;
  }
")
set(LAMBDA_METAL_CODE "
  kernel void test(device float* ptr,
                   uint idx [[thread_position_in_grid]]) {
    auto fn = [](float x) { return x + 1.0; };
    ptr[idx] = fn(ptr[idx]);
  }
")
if(NOT CAN_COMPILE_METAL_FOUND)
    file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/bfloat_inc.metal" "${BFLOAT_METAL_CODE}")
    execute_process(COMMAND xcrun metal -std=metal3.1 bfloat_inc.metal
```

- **EN:** CMake commands like set, if, file, execute_process drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、if、file、execute_process 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 58-73 / 第 58-73 行

```cmake
                    WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
                    OUTPUT_VARIABLE XCRUN_OUTPUT
                    ERROR_VARIABLE XCRUN_OUTPUT
                    RESULT_VARIABLE XCRUN_RC)
    if(${XCRUN_RC} EQUAL 0)
        message(STATUS "Machine can compile metal shaders")
        set(CAN_COMPILE_METAL YES CACHE BOOL "Host can compile metal shaders")
    else()
        message(WARNING "Machine can not compile metal shaders, fails with ${XCRUN_OUTPUT}")
        set(CAN_COMPILE_METAL NO CACHE BOOL "Host can compile metal shaders")
    endif()
    if(CAN_COMPILE_METAL)
        file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/lambda_test.metal" "${LAMBDA_METAL_CODE}")
        execute_process(COMMAND xcrun metal -std=metal4.0 -c lambda_test.metal -o /dev/null
                        WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
                        OUTPUT_VARIABLE XCRUN_OUTPUT
```

- **EN:** CMake commands like if, message, set, else, endif, file drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、message、set、else、endif、file 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 74-88 / 第 74-88 行

```cmake
                        ERROR_VARIABLE XCRUN_OUTPUT
                        RESULT_VARIABLE XCRUN_RC)
        if(${XCRUN_RC} EQUAL 0)
            message(STATUS "Metal toolchain supports Metal 4.0")
            set(CAN_COMPILE_METAL_40 YES CACHE BOOL "Host can compile Metal 4.0 shaders" FORCE)
        else()
            message(STATUS "Metal toolchain does not support Metal 4.0")
            set(CAN_COMPILE_METAL_40 NO CACHE BOOL "Host can compile Metal 4.0 shaders" FORCE)
        endif()
    else()
        set(CAN_COMPILE_METAL_40 NO CACHE BOOL "Host can compile Metal 4.0 shaders" FORCE)
    endif()
    set(CAN_COMPILE_METAL_FOUND YES CACHE INTERNAL "Run check for shader compiler")
endif()

```

- **EN:** CMake commands like if, message, set, else, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、message、set、else、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 89-96 / 第 89-96 行

```cmake
if(NOT USE_PYTORCH_METAL)
    return()
endif()

if(IOS OR INTERN_BUILD_MOBILE)
    return()
endif()

```

- **EN:** CMake commands like if, return, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、return、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 97-111 / 第 97-111 行

```cmake
set(OSX_PLATFORM "MacOSX.platform")
exec_program(/usr/bin/xcode-select ARGS -print-path OUTPUT_VARIABLE CMAKE_XCODE_DEVELOPER_DIR)
set(XCODE_POST_43_ROOT "${CMAKE_XCODE_DEVELOPER_DIR}/Platforms/${OSX_PLATFORM}/Developer")
set(XCODE_PRE_43_ROOT "/Developer/Platforms/${OSX_PLATFORM}/Developer")
if(NOT DEFINED CMAKE_OSX_DEVELOPER_ROOT)
    if(EXISTS ${XCODE_POST_43_ROOT})
        set(CMAKE_OSX_DEVELOPER_ROOT ${XCODE_POST_43_ROOT})
    elseif(EXISTS ${XCODE_PRE_43_ROOT})
        set(CMAKE_OSX_DEVELOPER_ROOT ${XCODE_PRE_43_ROOT})
    elseif(EXISTS ${CMAKE_XCODE_DEVELOPER_DIR} AND ${CMAKE_XCODE_DEVELOPER_DIR} STREQUAL "/Library/Developer/CommandLineTools")
            set(CMAKE_OSX_DEVELOPER_ROOT ${CMAKE_XCODE_DEVELOPER_DIR})
    endif()
endif(NOT DEFINED CMAKE_OSX_DEVELOPER_ROOT)
set(CMAKE_OSX_DEVELOPER_ROOT ${CMAKE_OSX_DEVELOPER_ROOT} CACHE PATH "Location of OSX SDKs root directory")

```

- **EN:** CMake commands like set, exec_program, if, elseif, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、exec_program、if、elseif、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 112-127 / 第 112-127 行

```cmake
if(NOT DEFINED CMAKE_OSX_SDK_ROOT)
    file(GLOB _CMAKE_OSX_SDKS "${CMAKE_OSX_DEVELOPER_ROOT}/SDKs/*")
    if(_CMAKE_OSX_SDKS)
        list(SORT _CMAKE_OSX_SDKS)
        list(REVERSE _CMAKE_OSX_SDKS)
        list(GET _CMAKE_OSX_SDKS 0 CMAKE_OSX_SDK_ROOT)
        message(STATUS "_CMAKE_OSX_SDKS: ${_CMAKE_OSX_SDKS}")
    else(_CMAKE_OSX_SDKS)
        message(FATAL_ERROR "No OSX SDK's found in default search path ${CMAKE_OSX_DEVELOPER_ROOT}.")
    endif(_CMAKE_OSX_SDKS)
    message(STATUS "Toolchain using default OSX SDK: ${CMAKE_OSX_SDK_ROOT}")
endif(NOT DEFINED CMAKE_OSX_SDK_ROOT)
set(CMAKE_OSX_SDK_ROOT ${CMAKE_OSX_SDK_ROOT} CACHE PATH "Location of the selected OSX SDK")
set(CMAKE_FRAMEWORK_PATH
    ${CMAKE_OSX_SDK_ROOT}/System/Library/Frameworks
    ${CMAKE_OSX_SDK_ROOT}/System/Library/PrivateFrameworks
```

- **EN:** CMake commands like if, file, list, message, else, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、file、list、message、else、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 128-131 / 第 128-131 行

```cmake
    ${CMAKE_OSX_SDK_ROOT}/Developer/Library/Frameworks
)
message(STATUS "CMAKE_FRAMEWORK_PATH: ${CMAKE_FRAMEWORK_PATH}")
set(CMAKE_FIND_FRAMEWORK FIRST)
```

- **EN:** CMake commands like message, set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 message、set 等 CMake 命令完成依赖探测、变量设置或平台检查。

## Key Concepts / 关键概念

- **CMake build configuration** — CMake 构建配置
- **Device coverage** — 突出 CPU/CUDA/移动端等后端相关执行路径。
- **CMake build logic** — 控制构建系统配置、特性检查或依赖发现。
- **Representative symbols: if, return, endif, set, string, function, add_custom_command, endfunction** — 代表性符号：if、return、endif、set、string、function、add_custom_command、endfunction

## Dependencies / 依赖关系

- No prominent dependency reference detected. / 未检测到明显的依赖引用。
