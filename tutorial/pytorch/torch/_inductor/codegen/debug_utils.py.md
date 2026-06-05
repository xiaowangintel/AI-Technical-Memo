# debug_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/debug_utils.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `IntermediateValueDebuggingLevel`, and `DebugPrinterManager`. It exposes functions such as `_print_debugging_tensor_value_info`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `IntermediateValueDebuggingLevel`、`DebugPrinterManager` 等类。同时提供 `_print_debugging_tensor_value_info` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
from __future__ import annotations

import functools
import logging
import os
from enum import Enum
from typing import TYPE_CHECKING

import torch
from torch import dtype as torch_dtype

from .. import config
from ..virtualized import V
from .multi_kernel import MultiKernel


if TYPE_CHECKING:
    from collections.abc import Callable

````
- **EN**: Imports dependencies such as `__future__`, `functools`, `logging`, `os`, `enum`, `typing`, and `...+5` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `__future__`、`functools`、`logging`、`os`、`enum`、`typing`、`另有5项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。

### Lines 21-40 / 第 21-40 行
````python

log = logging.getLogger(__name__)


def _print_debugging_tensor_value_info(msg, arg):
    # helper for printing debugging stats for intermediate tensor values
    # at jit inductor level codegen
    max_numel_to_print = 64
    print(msg)
    if not isinstance(arg, torch.Tensor):
        print("Value: ", arg)
        return
    numel = arg.float().numel()
    # print the debug printing stats
    if numel <= max_numel_to_print:
        print(arg)
    print("Number of elements: ", numel)
    print("Size: ", arg.float().size())
    print("Dtype: ", arg.float().mean().item())
    print("Mean: ", arg.float().mean().item())
````
- **EN**: Introduces function `_print_debugging_tensor_value_info`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_print_debugging_tensor_value_info`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 41-60 / 第 41-60 行
````python
    print("Min: ", arg.float().min().item())
    print("Max: ", arg.float().max().item())
    print("Std: ", arg.float().std().item())


# AOTI debug printing related configs
class IntermediateValueDebuggingLevel(Enum):
    # OFF: No intermediate tensor value debug info will be printed or saved.
    OFF = "0"
    # LEVEL 1: Save all intermediate tensor values to individual `.pt` files. No debug printing will be displayed.
    SAVE_ONLY = "1"
    # LEVEL 2: Print all intermediate tensor values by default to the console. No debug saving will be performed.
    PRINT_ONLY = "2"
    # LEVEL 3: Print all kernel names to the console only. No debug saving/printing for input tensor value info will be performed.
    # This mode can be helpful in cases when you just want to pinpointing what kernel is running into a CUDA IMA issue, etc.
    PRINT_KERNEL_NAMES_ONLY = "3"


class DebugPrinterManager:
    def __init__(
````
- **EN**: Introduces class `IntermediateValueDebuggingLevel`, class `DebugPrinterManager`, function `__init__`. Handles device-specific behavior, capabilities, or dispatch decisions. Initializes or updates values such as `OFF`, `SAVE_ONLY`, `PRINT_ONLY`, and `PRINT_KERNEL_NAMES_ONLY`.
- **CN**: 这里定义了类`IntermediateValueDebuggingLevel`、类`DebugPrinterManager`、函数`__init__`。处理与设备相关的行为、能力信息或派发决策。初始化或更新了 `OFF`、`SAVE_ONLY`、`PRINT_ONLY`、`PRINT_KERNEL_NAMES_ONLY` 等值。

### Lines 61-80 / 第 61-80 行
````python
        self,
        debug_printer_level,
        use_array_ref: bool,
        writeline: Callable[..., None] | None = None,
        args_to_print_or_save: list[str] | None = None,
        kernel_name: str = "",
        kernel=None,
        arg_signatures: list[type] | None = None,
        kernel_type=None,
    ):
        self.debug_printer_level = IntermediateValueDebuggingLevel(debug_printer_level)
        self.use_array_ref = use_array_ref
        if args_to_print_or_save is None:
            args_to_print_or_save = []
        self.args_to_print_or_save = args_to_print_or_save
        self.kernel_name = kernel_name
        self.arg_signatures: list[type] | None = None
        self.kernel = kernel
        self.filtered_kernel_names_to_print = self._get_debug_filtered_kernel_names()
        self.kernel_type = None
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `use_array_ref`, `writeline`, `args_to_print_or_save`, `kernel_name`, `kernel`, `arg_signatures`, and `...+1`. This range continues the implementation of function `DebugPrinterManager.__init__`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `use_array_ref`、`writeline`、`args_to_print_or_save`、`kernel_name`、`kernel`、`arg_signatures`、`另有1项` 等值。这一段延续了函数`DebugPrinterManager.__init__` 的具体实现。

### Lines 81-100 / 第 81-100 行
````python

    def __enter__(self):
        self._perform_debug_print_or_save_helper(
            self.args_to_print_or_save,
            self.kernel_name,
            before_launch=True,
            arg_signatures=self.arg_signatures,
        )

    def __exit__(self, args_to_print_or_save, kernel_name, arg_signatures):
        self._perform_debug_print_or_save_helper(
            args_to_print_or_save,
            kernel_name,
            before_launch=False,
            arg_signatures=arg_signatures,
        )

    def _perform_debug_print_or_save_helper(
        self,
        args_to_print_or_save,
````
- **EN**: Introduces function `__enter__`, function `__exit__`, function `_perform_debug_print_or_save_helper`. Initializes or updates values such as `before_launch`, and `arg_signatures`.
- **CN**: 这里定义了函数`__enter__`、函数`__exit__`、函数`_perform_debug_print_or_save_helper`。初始化或更新了 `before_launch`、`arg_signatures` 等值。

### Lines 101-120 / 第 101-120 行
````python
        kernel_name,
        before_launch,
        arg_signatures: list[type] | None = None,
    ):
        if self.debug_printer_level == IntermediateValueDebuggingLevel.OFF:
            return
        if self.debug_printer_level == IntermediateValueDebuggingLevel.SAVE_ONLY:
            # by default save all the tensor values before launch
            self.codegen_intermediate_tensor_value_save(
                self.args_to_print_or_save,
                self.kernel_name,
                before_launch,
                arg_signatures=self.arg_signatures,
            )
        if self.debug_printer_level == IntermediateValueDebuggingLevel.PRINT_ONLY:
            # by default print all the tensor values before launch
            self.codegen_intermediate_tensor_value_print(
                self.args_to_print_or_save,
                self.kernel_name,
                before_launch,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `arg_signatures`. This range continues the implementation of function `DebugPrinterManager._perform_debug_print_or_save_helper`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `arg_signatures` 等值。这一段延续了函数`DebugPrinterManager._perform_debug_print_or_save_helper` 的具体实现。

### Lines 121-140 / 第 121-140 行
````python
                arg_signatures=self.arg_signatures,
            )
        if (
            self.debug_printer_level
            == IntermediateValueDebuggingLevel.PRINT_KERNEL_NAMES_ONLY
        ):
            # Print all kernel names to the console only
            self.codegen_intermediate_tensor_value_print(
                [],
                self.kernel_name,
                before_launch,
            )

    @functools.lru_cache  # noqa: B019
    def _get_debug_filtered_kernel_names(self) -> list[str]:
        if config.aot_inductor.filtered_kernel_names is None:
            return []
        return [
            x.strip()
            for x in config.aot_inductor.filtered_kernel_names.lower().split(",")
````
- **EN**: Introduces function `_get_debug_filtered_kernel_names`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_get_debug_filtered_kernel_names`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 141-160 / 第 141-160 行
````python
        ]

    def set_printer_args(
        self,
        args_to_print_or_save: list[str],
        kernel_name: str,
        arg_signatures: list[type] | None,
        kernel,
        kernel_type=None,
    ):
        # Note: MultiKernel debug printing is not supported for now
        if isinstance(kernel, MultiKernel):
            log.info(
                "MultiKernel type is not supported in AOTI debug printer tool yet."
            )
            self.debug_printer_level = IntermediateValueDebuggingLevel.OFF

        self.kernel_type = kernel_type
        # Note: if the kernel type is an extern kernel (or cpp kernel), we do a special handling to
        # get the list of args_to_print_or_save
````
- **EN**: Introduces function `set_printer_args`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `args_to_print_or_save`, `kernel_name`, `arg_signatures`, and `kernel_type`.
- **CN**: 这里定义了函数`set_printer_args`。包含分支、循环或上下文管理等控制流。初始化或更新了 `args_to_print_or_save`、`kernel_name`、`arg_signatures`、`kernel_type` 等值。

### Lines 161-180 / 第 161-180 行
````python
        # TODO: Find a more reliable way to detect kernel args types to print for extern kernel calls
        if kernel_type == "extern":
            args_to_print_or_save_extern = [
                arg
                for arg in args_to_print_or_save
                if isinstance(arg, str) and arg.startswith(("buf", "arg"))
            ]
            self.args_to_print_or_save = args_to_print_or_save_extern
        elif kernel_type == "cpp":
            self.args_to_print_or_save = [
                (
                    f"copy_arrayref_tensor_to_tensor({arg})"
                    if self.use_array_ref
                    else arg
                )
                for arg in args_to_print_or_save
                if isinstance(arg, str) and arg.startswith(("buf", "arg"))
            ]
        else:
            self.args_to_print_or_save = args_to_print_or_save
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `args_to_print_or_save_extern`, and `else`. This range continues the implementation of function `DebugPrinterManager.set_printer_args`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `args_to_print_or_save_extern`、`else` 等值。这一段延续了函数`DebugPrinterManager.set_printer_args` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
        self.kernel_name = kernel_name
        self.arg_signatures = arg_signatures
        self.kernel = kernel

    def codegen_model_inputs_value_print(self, input_args_to_print: list[str]) -> None:
        if self.debug_printer_level != IntermediateValueDebuggingLevel.PRINT_ONLY:
            return
        for arg in input_args_to_print:
            if V.graph.cpp_wrapper:
                V.graph.wrapper_code.prefix.writeline(
                    f'aoti_torch_print_tensor_handle({arg}, "aoti_model_inputs - {arg}");'
                )

    def codegen_intermediate_tensor_value_save(
        self,
        args_to_save,
        kernel_name,
        before_launch=True,
        arg_signatures: list[type] | None = None,
    ) -> None:
````
- **EN**: Introduces function `codegen_model_inputs_value_print`, function `codegen_intermediate_tensor_value_save`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `before_launch`, and `arg_signatures`.
- **CN**: 这里定义了函数`codegen_model_inputs_value_print`、函数`codegen_intermediate_tensor_value_save`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `before_launch`、`arg_signatures` 等值。

### Lines 201-220 / 第 201-220 行
````python
        for i, arg in enumerate(args_to_save):
            if arg_signatures is not None and not isinstance(
                arg_signatures[i], torch_dtype
            ):
                # infer from the arg data type (has torch.dtype) to see if it is a tensor type
                continue
            launch_prefix = "before_launch" if before_launch else "after_launch"
            if V.graph.cpp_wrapper:
                V.graph.wrapper_code.writeline(
                    f'aoti_torch_save_tensor_handle({arg}, "{arg}", "{launch_prefix}", "{kernel_name}");'
                )
            else:
                cwd = os.getcwd()
                saved_dir = cwd + "/tmp/jit_inductor/"
                if not os.path.exists(saved_dir):
                    log.info(
                        "Creating directory to save inductor intermediate tensor values."
                    )
                    os.makedirs(saved_dir)
                # Save the model to the directory
````
- **EN**: Touches the filesystem to load, validate, or store compiler artifacts. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `launch_prefix`, `else`, `cwd`, and `saved_dir`.
- **CN**: 这一段会访问文件系统，用于加载、校验或保存编译产物。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `launch_prefix`、`else`、`cwd`、`saved_dir` 等值。

### Lines 221-240 / 第 221-240 行
````python
                saved_path = saved_dir + f"{launch_prefix}_{kernel_name}_{arg}.pt"
                log.info(
                    "Saved intermediate tensor %s for %s to %s",
                    arg,
                    kernel_name,
                    saved_path,
                )
                line = f"torch.save({arg}, '{saved_path}')"
                V.graph.wrapper_code.writeline(line)

    def codegen_intermediate_tensor_value_print(
        self,
        args_to_print,
        kernel_name,
        before_launch=True,
        arg_signatures: list[type] | None = None,
    ) -> None:
        launch_prefix = "before_launch" if before_launch else "after_launch"

        # if the debug printing level is PRINT_KERNEL_NAMES_ONLY
````
- **EN**: Introduces function `codegen_intermediate_tensor_value_print`. Initializes or updates values such as `saved_path`, `line`, `before_launch`, `arg_signatures`, and `launch_prefix`.
- **CN**: 这里定义了函数`codegen_intermediate_tensor_value_print`。初始化或更新了 `saved_path`、`line`、`before_launch`、`arg_signatures`、`launch_prefix` 等值。

### Lines 241-260 / 第 241-260 行
````python
        # we only print the kernel name to the console
        if (
            self.debug_printer_level
            == IntermediateValueDebuggingLevel.PRINT_KERNEL_NAMES_ONLY
        ):
            if V.graph.cpp_wrapper:
                V.graph.wrapper_code.writeline(
                    f'printf("[ {launch_prefix}: {kernel_name} ]\\n");'
                )
            return

        if self.debug_printer_level != IntermediateValueDebuggingLevel.PRINT_ONLY:
            return
        for i, arg in enumerate(args_to_print):
            # when debug printing is enabled i.e. IntermediateValueDebuggingLevel.PRINT_ONLY,
            # check if filtered kernel name list is provided
            if (
                len(self.filtered_kernel_names_to_print) > 0
                and kernel_name.lower() not in self.filtered_kernel_names_to_print
            ):
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `DebugPrinterManager.codegen_intermediate_tensor_value_print`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`DebugPrinterManager.codegen_intermediate_tensor_value_print` 的具体实现。

### Lines 261-280 / 第 261-280 行
````python
                continue
            if V.graph.cpp_wrapper:
                if arg_signatures is not None and isinstance(
                    arg_signatures[i], torch_dtype
                ):
                    # infer from the arg data type (has torch.dtype) to see if it is a tensor type
                    V.graph.wrapper_code.writeline(
                        f'aoti_torch_print_tensor_handle({arg}, "{launch_prefix} - {kernel_name} - {arg}");'
                    )
                elif arg_signatures is not None and isinstance(
                    arg_signatures[i],
                    (
                        type(torch._inductor.codegen.wrapper.SymbolicCallArg),
                        type(int),
                        type(float),
                        type(bool),
                    ),
                ):
                    V.graph.wrapper_code.writeline(
                        f'printf("[  {launch_prefix} - {kernel_name} - {arg}: %ld  ]", {arg}); printf("\\\\n");'
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `DebugPrinterManager.codegen_intermediate_tensor_value_print`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。这一段延续了函数`DebugPrinterManager.codegen_intermediate_tensor_value_print` 的具体实现。

### Lines 281-290 / 第 281-290 行
````python
                    )
                else:
                    if arg_signatures is None and self.kernel_type in ("cpp", "extern"):
                        V.graph.wrapper_code.writeline(
                            f'aoti_torch_print_tensor_handle({arg}, "{launch_prefix} - {kernel_name} - {arg}");'
                        )
            else:
                V.graph.wrapper_code.writeline(
                    f'_print_debugging_tensor_value_info("inductor: {launch_prefix} - {kernel_name} - {arg}", {arg})'
                )
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`. This range continues the implementation of function `DebugPrinterManager.codegen_intermediate_tensor_value_print`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。这一段延续了函数`DebugPrinterManager.codegen_intermediate_tensor_value_print` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `IntermediateValueDebuggingLevel`, and `DebugPrinterManager`  
  **CN**: 主要类：`IntermediateValueDebuggingLevel`、`DebugPrinterManager`
- **EN**: Primary functions: `_print_debugging_tensor_value_info`  
  **CN**: 主要函数：`_print_debugging_tensor_value_info`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `functools`, `logging`, `os`, `enum`, `typing`, `collections.abc`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `..`, `..virtualized`, `.multi_kernel`
