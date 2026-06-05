# cpp_wrapper_mps.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/cpp_wrapper_mps.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `CppWrapperMps`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `CppWrapperMps` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
from typing import Any

import sympy

import torch
from torch.utils._ordered_set import OrderedSet

from ..ir import GraphPartitionSignature
from ..virtualized import V
from .cpp_wrapper_cpu import CppWrapperCpu
from .cpp_wrapper_gpu import CppWrapperGpu
from .wrapper import KernelCallLine, PythonWrapperCodegen


class CppWrapperMps(CppWrapperGpu):
    """
    Generates cpp wrapper for running on MPS and calls metal kernels
    """

    def __init__(self) -> None:
````
- **EN**: Imports dependencies such as `typing`, `sympy`, `torch`, `torch.utils._ordered_set`, `..ir`, `..virtualized`, and `...+3` for the logic in this range. Introduces class `CppWrapperMps`, function `__init__`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `typing`、`sympy`、`torch`、`torch.utils._ordered_set`、`..ir`、`..virtualized`、`另有3项` 等依赖，为后续逻辑提供基础能力。这里定义了类`CppWrapperMps`、函数`__init__`。借助符号表达式来推理索引、形状或代数性质。

### Lines 21-40 / 第 21-40 行
````python
        super().__init__()
        self._used_kernel_names: OrderedSet[str] = OrderedSet()
        self._lambda_counter: int = 0

    @staticmethod
    def create(
        is_subgraph: bool,
        subgraph_name: str | None,
        parent_wrapper: PythonWrapperCodegen | None,
        partition_signatures: GraphPartitionSignature | None = None,
    ) -> "CppWrapperMps":
        return CppWrapperMps()

    def _generate_kernel_call_helper(
        self,
        kernel_name: str,
        call_args: list[str],
        *,
        device: torch.device | None = None,
        triton: bool = True,
````
- **EN**: Introduces function `create`, function `_generate_kernel_call_helper`. Applies decorators to register behavior or alter how the following definition is constructed. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`create`、函数`_generate_kernel_call_helper`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。

### Lines 41-60 / 第 41-60 行
````python
        arg_types: tuple[Any, ...] | None = None,
        raw_keys: tuple[Any, ...] | None = None,
        raw_args: tuple[Any, ...] | None = None,
        triton_meta: dict[str, Any] | None = None,
        inductor_meta: dict[str, Any] | None = None,
        graph_name: str = "",
        original_fxnode_name: str | None = None,
        current_stream_idx: int | None = None,
    ) -> None:
        """
        Generates MPS kernel call code. It should look something like:
        ```
        auto mps_lib_0_lambda = [&](AOTIMetalKernelFunctionHandle handle) {
            aoti_torch_mps_start_encoding(handle);
            aoti_torch_mps_set_arg_tensor(handle, 0, buf0);
            aoti_torch_mps_set_arg_tensor(handle, 1, arg0_1);
            aoti_torch_mps_set_arg_tensor(handle, 2, arg1_1);
            aoti_torch_mps_dispatch_single(handle, static_cast<uint64_t>(10LL));
        };

````
- **EN**: Initializes or updates values such as `arg_types`, `raw_keys`, `raw_args`, `triton_meta`, `inductor_meta`, `graph_name`, and `...+2`. This range continues the implementation of function `CppWrapperMps._generate_kernel_call_helper`.
- **CN**: 初始化或更新了 `arg_types`、`raw_keys`、`raw_args`、`triton_meta`、`inductor_meta`、`graph_name`、`另有2项` 等值。这一段延续了函数`CppWrapperMps._generate_kernel_call_helper` 的具体实现。

### Lines 61-80 / 第 61-80 行
````python
        std::function<void(AOTIMetalKernelFunctionHandle)> mps_lib_0_func_wrapper = mps_lib_0_lambda;
        aoti_torch_mps_run_command_block(get_mps_lib_0_handle(), aoti_torch_mps_shared_callback, &mps_lib_0_func_wrapper);
        ```
        """
        device = device or V.graph.get_current_device_or_throw()
        if device.type == "cpu":
            # Even in CppWrapperGpu, we may see cpp kernels
            return CppWrapperCpu._generate_kernel_call_helper(
                self,
                kernel_name,
                call_args,
                device=device,
                triton=triton,
                arg_types=arg_types,
                raw_keys=raw_keys,
                raw_args=raw_args,
                triton_meta=triton_meta,
                inductor_meta=inductor_meta,
            )

````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 81-100 / 第 81-100 行
````python
        assert device.type == "mps"

        assert arg_types is not None

        new_args = []
        for idx, (arg, arg_type) in enumerate(zip(call_args[:-2], arg_types[:-2])):
            if isinstance(arg_type, torch.dtype):
                new_args.append(f"aoti_torch_mps_set_arg_tensor(handle, {idx}, {arg});")
            elif arg_type in (int, sympy.core.symbol.Symbol):
                new_args.append(f"aoti_torch_mps_set_arg_int(handle, {idx}, {arg});")
            else:
                raise NotImplementedError(
                    f"Unsupported arg type {arg_type} for arg {arg} for kernel {kernel_name}"
                )

        threads, group_size = call_args[-2], call_args[-1]
        if threads is None:
            raise NotImplementedError("No threads or group_size provided")

        # Check if threads is a single value or an array-like structure
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 101-120 / 第 101-120 行
````python
        threads_str = str(threads)
        is_single_value = (
            threads_str.startswith("{")
            and threads_str.endswith("}")
            and threads_str.count(",") == 0
        ) or not threads_str.startswith(("{", "["))

        if is_single_value:
            # Extract single value from braces if present
            if threads_str.startswith("{") and threads_str.endswith("}"):
                single_value = threads_str[1:-1].strip()  # Remove braces
            else:
                single_value = threads_str

            if group_size is None:
                new_args.append(
                    f"aoti_torch_mps_dispatch_single(handle, {single_value});"
                )
            else:
                # Extract group size value if it's also in braces
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `threads_str`, `is_single_value`, `single_value`, and `else`. This range continues the implementation of function `CppWrapperMps._generate_kernel_call_helper`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `threads_str`、`is_single_value`、`single_value`、`else` 等值。这一段延续了函数`CppWrapperMps._generate_kernel_call_helper` 的具体实现。

### Lines 121-140 / 第 121-140 行
````python
                group_size_str = str(group_size)
                if group_size_str.startswith("{") and group_size_str.endswith("}"):
                    group_size_value = group_size_str[1:-1].strip()
                else:
                    group_size_value = group_size_str
                new_args.append(
                    f"aoti_torch_mps_dispatch_single_with_group_size(handle, {single_value}, {group_size_value});"
                )
        else:
            # Handle array case - need to convert initializer list to array
            # Use kernel name to make variable names unique
            threads_var = f"{kernel_name}_threads_array"
            group_size_var = f"{kernel_name}_group_size_array"

            # Extract array size from the initializer list string
            def get_array_size(array_str: str) -> int:
                # Remove braces and whitespace
                content = array_str.strip()
                if content.startswith("{") and content.endswith("}"):
                    content = content[1:-1].strip()
````
- **EN**: Introduces function `get_array_size`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `group_size_str`, `group_size_value`, `else`, `threads_var`, `group_size_var`, and `content`.
- **CN**: 这里定义了函数`get_array_size`。包含分支、循环或上下文管理等控制流。初始化或更新了 `group_size_str`、`group_size_value`、`else`、`threads_var`、`group_size_var`、`content` 等值。

### Lines 141-160 / 第 141-160 行
````python

                if not content:  # Empty array
                    return 0

                # Count elements by counting commas, accounting for nested structures
                depth = 0
                comma_count = 0
                for char in content:
                    if char in "({[<":
                        depth += 1
                    elif char in ")}]>":
                        depth -= 1
                    elif char == "," and depth == 0:
                        comma_count += 1

                return comma_count + 1  # Number of elements = commas + 1

            threads_size = get_array_size(threads_str)

            if group_size is None:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `depth`, `comma_count`, and `threads_size`. This range continues the implementation of function `CppWrapperMps._generate_kernel_call_helper.get_array_size`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `depth`、`comma_count`、`threads_size` 等值。这一段延续了函数`CppWrapperMps._generate_kernel_call_helper.get_array_size` 的具体实现。

### Lines 161-180 / 第 161-180 行
````python
                new_args.append("{")
                new_args.append(f"    uint64_t {threads_var}[] = {threads};")
                new_args.append(
                    f"    aoti_torch_mps_dispatch_array(handle, {threads_var}, {threads_size});"
                )
                new_args.append("}")
            else:
                group_size_str = str(group_size)
                group_size_size = get_array_size(group_size_str)
                new_args.append("{")
                new_args.append(f"    uint64_t {threads_var}[] = {threads};")
                new_args.append(f"    uint64_t {group_size_var}[] = {group_size};")
                dispatch_args = f"handle, {threads_var}, {threads_size}, {group_size_var}, {group_size_size}"
                new_args.append(
                    f"    aoti_torch_mps_dispatch_array_with_group_size({dispatch_args});"
                )
                new_args.append("}")

        # debug printer related logic for cpp kernel type.
        debug_printer_manager = V.graph.wrapper_code.debug_printer
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `group_size_str`, `group_size_size`, `dispatch_args`, and `debug_printer_manager`. This range continues the implementation of function `CppWrapperMps._generate_kernel_call_helper`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`group_size_str`、`group_size_size`、`dispatch_args`、`debug_printer_manager` 等值。这一段延续了函数`CppWrapperMps._generate_kernel_call_helper` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
        debug_printer_manager.set_printer_args(
            call_args[:-2],
            kernel_name,
            None,
            None,
            "cpp",
        )
        with debug_printer_manager:
            self.write_mps_kernel_call(kernel_name, new_args)

    def write_mps_kernel_call(self, name: str, call_args: list[str]) -> None:
        # Generate unique variable names to avoid duplicate declarations
        # when the same MPS lib is used multiple times
        unique_suffix = self._lambda_counter
        self._lambda_counter += 1

        lambda_name = f"{name}_lambda_{unique_suffix}"
        wrapper_name = f"{name}_func_wrapper_{unique_suffix}"

        # Generate the function call code (in current location)
````
- **EN**: Introduces function `write_mps_kernel_call`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `unique_suffix`, `lambda_name`, and `wrapper_name`.
- **CN**: 这里定义了函数`write_mps_kernel_call`。包含分支、循环或上下文管理等控制流。初始化或更新了 `unique_suffix`、`lambda_name`、`wrapper_name` 等值。

### Lines 201-220 / 第 201-220 行
````python
        # Create lambda that captures by reference and pass its pointer through void*
        self.writeline(
            f"auto {lambda_name} = [&](AOTIMetalKernelFunctionHandle handle) {{"
        )
        self.writeline("    aoti_torch_mps_start_encoding(handle);")

        # Output call args directly since we're capturing by reference
        for call_arg in call_args:
            self.writeline(f"    {call_arg}")
        self.writeline("};")
        self.writeline("")

        # Pass lambda pointer through void*
        self.writeline(
            f"std::function<void(AOTIMetalKernelFunctionHandle)> {wrapper_name} = {lambda_name};"
        )
        self.writeline(
            f"aoti_torch_mps_run_command_block(get_{name}_handle(), aoti_torch_mps_shared_callback, &{wrapper_name});"
        )

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `CppWrapperMps.write_mps_kernel_call`.
- **CN**: 包含分支、循环或上下文管理等控制流。这一段延续了函数`CppWrapperMps.write_mps_kernel_call` 的具体实现。

### Lines 221-240 / 第 221-240 行
````python
    @staticmethod
    def get_device_include_path(device: str) -> str:
        assert V.graph.aot_mode
        return (
            "#include <torch/csrc/inductor/aoti_include/mps.h>\n"
            "#include <torch/csrc/inductor/aoti_torch/c/shim_mps.h>"
        )

    def codegen_additional_funcs(self) -> None:
        """
        Generate thread-safe lazy singleton pattern for MPS shader libraries with RAII cleanup.

        The generated code will look like:
        ```
        AOTIMetalKernelFunctionHandle get_mps_lib_0_handle() {
            static auto kernel_handle = []() {
                AOTIMetalShaderLibraryHandle lib_handle = nullptr;
                AOTIMetalKernelFunctionHandle kern_handle = nullptr;

                aoti_torch_mps_create_shader_library(mps_lib_0_source, &lib_handle);
````
- **EN**: Introduces function `get_device_include_path`, function `codegen_additional_funcs`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_device_include_path`、函数`codegen_additional_funcs`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 241-260 / 第 241-260 行
````python
                aoti_torch_mps_get_kernel_function(lib_handle, "generated_kernel", &kern_handle);

                // RAII wrapper with custom deleter
                auto lib_deleter = [](AOTIMetalShaderLibraryHandle h) {
                    if (h) aoti_torch_mps_delete_shader_library(h);
                };

                using LibDeleter = decltype(lib_deleter);
                using LibPtr = std::unique_ptr<AOTIMetalShaderLibraryOpaque, LibDeleter>;

                // Return pair of kernel handle and library smart pointer for cleanup
                return std::make_pair(kern_handle, LibPtr(lib_handle, lib_deleter));
            }();
            return kernel_handle.first;
        }
        ```
        """

        # Add shimified handles and functions
        shader_libraries: OrderedSet[str] = OrderedSet()
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `shader_libraries`. This range continues the implementation of function `CppWrapperMps.codegen_additional_funcs`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `shader_libraries` 等值。这一段延续了函数`CppWrapperMps.codegen_additional_funcs` 的具体实现。

### Lines 261-280 / 第 261-280 行
````python
        for line in self.lines:
            if not isinstance(line, KernelCallLine):
                continue
            if line.device.type != "mps":
                continue

            # Extract library name from kernel name (e.g., "mps_lib_0" from kernel calls)
            if line.kernel_name not in self._used_kernel_names:
                self._used_kernel_names.add(line.kernel_name)
                shader_libraries.add(line.kernel_name)

        # NOTE: For shimified version, we expect the shader source constant to be generated
        # by the existing MPS shader generation process, but instead of instantiating the
        # DynamicMetalShaderLibrary directly, we'll use our shim functions.
        # The existing codegen should produce something like:
        # const char* mps_lib_0_source = R"MTL(...shader_source...)MTL";
        # instead of:
        # at::native::mps::DynamicMetalShaderLibrary mps_lib_0(R"MTL(...shader_source...)MTL");

        # Generate thread-safe lazy singleton with RAII for each library
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `CppWrapperMps.codegen_additional_funcs`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。这一段延续了函数`CppWrapperMps.codegen_additional_funcs` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python
        for lib_name in shader_libraries:
            self.prefix.splice(f"""
AOTIMetalKernelFunctionHandle get_{lib_name}_handle() {{
    static auto kernel_handle = []() {{
        AOTIMetalShaderLibraryHandle lib_handle = nullptr;
        AOTIMetalKernelFunctionHandle kern_handle = nullptr;

        aoti_torch_mps_create_shader_library({lib_name}_source, &lib_handle);
        aoti_torch_mps_get_kernel_function(lib_handle, "generated_kernel", &kern_handle);

        // RAII wrapper with custom deleter
        auto lib_deleter = [](AOTIMetalShaderLibraryHandle h) {{
            if (h) aoti_torch_mps_delete_shader_library(h);
        }};

        using LibDeleter = decltype(lib_deleter);
        using LibPtr = std::unique_ptr<AOTIMetalShaderLibraryOpaque, LibDeleter>;

        // Return pair of kernel handle and library smart pointer for cleanup
        return std::make_pair(kern_handle, LibPtr(lib_handle, lib_deleter));
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `CppWrapperMps.codegen_additional_funcs`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`CppWrapperMps.codegen_additional_funcs` 的具体实现。

### Lines 301-304 / 第 301-304 行
````python
    }}();
    return kernel_handle.first;
}}
""")
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of function `CppWrapperMps.codegen_additional_funcs`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`CppWrapperMps.codegen_additional_funcs` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `CppWrapperMps`  
  **CN**: 主要类：`CppWrapperMps`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.utils._ordered_set`, `..ir`, `..virtualized`, `.cpp_wrapper_cpu`, `.cpp_wrapper_gpu`, `.wrapper`
