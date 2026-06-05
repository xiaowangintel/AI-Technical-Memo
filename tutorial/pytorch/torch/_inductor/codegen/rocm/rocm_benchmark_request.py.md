# rocm_benchmark_request.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/rocm/rocm_benchmark_request.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `ROCmBenchmarkRequest`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `ROCmBenchmarkRequest` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
# mypy: allow-untyped-defs
from __future__ import annotations

import functools
import logging
from ctypes import byref, c_int, c_size_t, c_void_p
from typing import Any, TYPE_CHECKING

import torch
from torch._inductor import config
from torch._inductor.autotune_process import (
    BenchmarkRequest,
    GPUDeviceBenchmarkMixin,
    TensorMeta,
````
- **EN**: Imports dependencies such as `__future__`, `functools`, `logging`, `ctypes`, `typing`, `torch`, and `...+2` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis.
- **CN**: 这里导入了 `__future__`、`functools`、`logging`、`ctypes`、`typing`、`torch`、`另有2项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。

### Lines 15-28 / 第 15-28 行
````python
)
from torch._inductor.codecache import DLLWrapper, ROCmCodeCache


if TYPE_CHECKING:
    from collections.abc import Callable, Iterable


log = logging.getLogger(__name__)


class ROCmBenchmarkRequest(GPUDeviceBenchmarkMixin, BenchmarkRequest):
    # Important: Instances of this class have to be serializable
    # across process boundaries. Do not put CUDA Tensors in here!
````
- **EN**: Imports dependencies such as `torch._inductor.codecache`, and `collections.abc` for the logic in this range. Introduces class `ROCmBenchmarkRequest`. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里导入了 `torch._inductor.codecache`、`collections.abc` 等依赖，为后续逻辑提供基础能力。这里定义了类`ROCmBenchmarkRequest`。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 29-42 / 第 29-42 行
````python

    def __init__(
        self,
        kernel_name: str,
        input_tensor_meta: TensorMeta | list[TensorMeta],
        output_tensor_meta: TensorMeta | list[TensorMeta],
        extra_args: Iterable[Any],
        source_code: str,
    ) -> None:
        super().__init__(kernel_name, input_tensor_meta, output_tensor_meta, extra_args)
        self.source_code = source_code
        self.workspace_size: int = 0
        self.workspace: torch.Tensor | None = None
        self.DLL: DLLWrapper | None = None
````
- **EN**: Introduces function `__init__`. Initializes or updates values such as `kernel_name`, `input_tensor_meta`, `output_tensor_meta`, `extra_args`, and `source_code`.
- **CN**: 这里定义了函数`__init__`。初始化或更新了 `kernel_name`、`input_tensor_meta`、`output_tensor_meta`、`extra_args`、`source_code` 等值。

### Lines 43-56 / 第 43-56 行
````python
        self._workspace_size_updated = False
        self.hash_key: str = ""
        self.source_file: str = ""
        self.hash_key, self.source_file = ROCmCodeCache.write(self.source_code, "so")

    def precompile(self):
        # Prepopulate code cache
        # may happen in separate Threadpool
        log.debug("Precompiling %s", self)
        ROCmCodeCache.compile(self.source_code, "so")
        if config.rocm.generate_test_runner:
            ROCmCodeCache.compile(self.source_code, "exe")
        log.debug("Done precompiling %s", self)

````
- **EN**: Introduces function `precompile`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`precompile`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。

### Lines 57-70 / 第 57-70 行
````python
    def make_run_fn(
        self, *input_tensors: torch.Tensor, out: torch.Tensor
    ) -> Callable[[], None]:
        self.ensure_dll_loaded()
        self.update_workspace_size()
        args = [c_void_p(tensor.data_ptr()) for tensor in list(input_tensors) + [out]]
        size_args = [c_int(arg) for arg in self.extra_args]
        log.debug(
            "make_run_fn: self.kernel_name=%s, self.source_file=%s, self.hash_key=%s, self.DLL=%s, args=%s, self.extra_args=%s",
            self.kernel_name,
            self.source_file,
            self.hash_key,
            self.DLL,
            args,
````
- **EN**: Introduces function `make_run_fn`. Initializes or updates values such as `args`, and `size_args`.
- **CN**: 这里定义了函数`make_run_fn`。初始化或更新了 `args`、`size_args` 等值。

### Lines 71-84 / 第 71-84 行
````python
            self.extra_args,
        )
        stream_ptr = c_void_p(torch.cuda.current_stream().cuda_stream)
        run_method = getattr(self.DLL, self.kernel_name)
        workspace_ptr = c_void_p(0)
        if self.workspace_size > 0:
            self.workspace = torch.zeros(
                (self.workspace_size + 7) // 8,
                dtype=torch.float64,
                device=out.device,
            )
            workspace_ptr = c_void_p(self.workspace.data_ptr())

        # Generate partial function.
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `stream_ptr`, `run_method`, `workspace_ptr`, `dtype`, and `device`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `stream_ptr`、`run_method`、`workspace_ptr`、`dtype`、`device` 等值。

### Lines 85-98 / 第 85-98 行
````python
        return functools.partial(
            run_method,
            *args,
            *size_args,
            None,  # null workspace size ptr
            workspace_ptr,  # set workspace ptr,
            stream_ptr,
        )

    def update_workspace_size(self) -> None:
        if self._workspace_size_updated:
            return
        self.ensure_dll_loaded()
        unique_input_count = len(
````
- **EN**: Introduces function `update_workspace_size`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `unique_input_count`.
- **CN**: 这里定义了函数`update_workspace_size`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `unique_input_count` 等值。

### Lines 99-112 / 第 99-112 行
````python
            dict.fromkeys(meta.name for meta in self.input_tensor_meta)
        )
        args = [c_void_p(None) for _ in range(unique_input_count + 1)]
        stream_ptr = c_void_p(torch.cuda.current_stream().cuda_stream)

        run_method = getattr(self.DLL, self.kernel_name)
        # Retrieve workspace_size and initialize workspace.
        c_workspace_size = c_size_t()
        size_args = [c_int(arg) for arg in self.extra_args]
        run_method(
            *args,  # input ptrs and output ptrs
            *size_args,
            byref(
                c_workspace_size
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Initializes or updates values such as `args`, `stream_ptr`, `run_method`, `c_workspace_size`, and `size_args`. This range continues the implementation of function `ROCmBenchmarkRequest.update_workspace_size`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。初始化或更新了 `args`、`stream_ptr`、`run_method`、`c_workspace_size`、`size_args` 等值。这一段延续了函数`ROCmBenchmarkRequest.update_workspace_size` 的具体实现。

### Lines 113-126 / 第 113-126 行
````python
            ),  # set workspace size ptr to retrieve workspace size
            None,  # null workspace ptr
            stream_ptr,
        )
        torch.cuda.synchronize()  # shake out any CUDA errors
        self.workspace_size = c_workspace_size.value
        log.debug(
            "update_workspace_size called: new workspace size=%d, self.kernel_name=%s, self.source_file=%s, self.hash_key=%s, self.DLL=%s, args=%s, self.extra_args=%s",
            self.workspace_size,
            self.kernel_name,
            self.source_file,
            self.hash_key,
            self.DLL,
            args,
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. This range continues the implementation of function `ROCmBenchmarkRequest.update_workspace_size`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。这一段延续了函数`ROCmBenchmarkRequest.update_workspace_size` 的具体实现。

### Lines 127-140 / 第 127-140 行
````python
            self.extra_args,
        )
        self._workspace_size_updated = True

    def ensure_dll_loaded(self):
        if self.DLL is None:
            self.DLL, self.hash_key, self.source_file = ROCmCodeCache.load(
                self.source_code, "so"
            )

    def cleanup_run_fn(self) -> None:
        if self.DLL is not None:
            self.DLL.close()
        self.workspace = None
````
- **EN**: Introduces function `ensure_dll_loaded`, function `cleanup_run_fn`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`ensure_dll_loaded`、函数`cleanup_run_fn`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。

### Lines 141-143 / 第 141-143 行
````python

    def __str__(self) -> str:
        return f"{self.kernel_name=}, {self.source_file=}, {self.hash_key=}"
````
- **EN**: Introduces function `__str__`. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`__str__`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `ROCmBenchmarkRequest`  
  **CN**: 主要类：`ROCmBenchmarkRequest`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `functools`, `logging`, `ctypes`, `typing`, `collections.abc`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor`, `torch._inductor.autotune_process`, `torch._inductor.codecache`
