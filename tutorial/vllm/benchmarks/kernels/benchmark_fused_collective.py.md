# benchmark_fused_collective.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_fused_collective.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, centered around `RMS_NORM_OP`, `FUSED_ADD_RMS_NORM_OP`, `RMS_NORM_STATIC_FP8_QUANT_OP`, `FUSED_ADD_RMS_NORM_STATIC_FP8_QUANT_OP`. / 实现与基准测试编排相关的逻辑，核心符号包括 `RMS_NORM_OP`, `FUSED_ADD_RMS_NORM_OP`, `RMS_NORM_STATIC_FP8_QUANT_OP`, `FUSED_ADD_RMS_NORM_STATIC_FP8_QUANT_OP`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 4-17)
```python
"""
Benchmark for FlashInfer fused collective operations vs standard operations.

This benchmark compares:
1. FlashInfer's allreduce_fusion with trtllm backend
   (fused allreduce + rmsnorm + optional FP8/FP4 quant)
2. FlashInfer's allreduce_fusion with mnnvl backend
   (fused allreduce + rmsnorm only, no quantization support)
3. Standard tensor_model_parallel_all_reduce + separate rmsnorm/quant operations

Usage with torchrun:
    torchrun --nproc_per_node=2 benchmark_fused_collective.py

"""
```
**EN:** The module docstring introduces Benchmark for FlashInfer fused collective operations vs standard operations. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Benchmark for FlashInfer fused collective operations vs standard operations 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 19-42)
```python
import argparse
import itertools
import os
import time

import pandas as pd
import torch  # type: ignore
import torch.distributed as dist  # type: ignore

from vllm._custom_ops import create_fp4_output_tensors
from vllm.config.vllm import CompilationConfig, VllmConfig, set_current_vllm_config
from vllm.distributed import (
    tensor_model_parallel_all_reduce,
)
from vllm.distributed.parallel_state import (
    graph_capture,
    init_distributed_environment,
    initialize_model_parallel,
)
from vllm.logger import init_logger
from vllm.model_executor.layers.layernorm import RMSNorm  # noqa
from vllm.model_executor.layers.quantization.input_quant_fp8 import QuantFP8  # noqa
from vllm.model_executor.layers.quantization.utils.quant_utils import GroupShape  # noqa
from vllm.platforms import current_platform  # noqa
```
**EN:** This block gathers standard-library helpers such as `argparse`, `itertools`, `os`, `time`; third-party packages such as `pandas`, `torch`, `torch.distributed`; project-local modules such as `vllm._custom_ops`, `vllm.config.vllm`, `vllm.distributed`, `vllm.distributed.parallel_state`, `vllm.logger`, `vllm.model_executor.layers.layernorm`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `itertools`, `os`, `time`；第三方依赖，如 `pandas`, `torch`, `torch.distributed`；项目内部模块，如 `vllm._custom_ops`, `vllm.config.vllm`, `vllm.distributed`, `vllm.distributed.parallel_state`, `vllm.logger`, `vllm.model_executor.layers.layernorm`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 44-89)
```python
RMS_NORM_OP = torch.ops._C.rms_norm
FUSED_ADD_RMS_NORM_OP = torch.ops._C.fused_add_rms_norm
RMS_NORM_STATIC_FP8_QUANT_OP = torch.ops._C.rms_norm_static_fp8_quant
FUSED_ADD_RMS_NORM_STATIC_FP8_QUANT_OP = (
    torch.ops._C.fused_add_rms_norm_static_fp8_quant
)
SCALED_FP4_QUANT_OUT_OP = torch.ops._C.scaled_fp4_quant.out

logger = init_logger(__name__)

# Try to import FlashInfer
TorchDistBackend = None
try:
    import flashinfer.comm as flashinfer_comm  # type: ignore
    from flashinfer.comm.mnnvl import (  # type: ignore
        TorchDistBackend,
    )

    if not (
        hasattr(flashinfer_comm, "allreduce_fusion")
        and hasattr(flashinfer_comm, "create_allreduce_fusion_workspace")
    ):
        flashinfer_comm = None
        logger.warning("FlashInfer comm module found but missing allreduce_fusion API")
except ImportError:
    flashinfer_comm = None
    logger.warning("FlashInfer not found, only benchmarking standard operations")

# Constants
FP8_DTYPE = current_platform.fp8_dtype()
MiB = 1024 * 1024

# FlashInfer max sizes per world size
# Enable 64MB for 2, 4, 8 world sizes to verify large input sizes
# use --disable-oneshot to disable oneshot mode for very large input sizes
_FI_MAX_SIZES = {
    2: 64 * MiB,  # 64MB
    4: 64 * MiB,  # 64MB
    8: 64 * MiB,  # 64MB
}

# Global workspace tensors for FlashInfer (keyed by backend name)
_FI_WORKSPACES: dict = {}

# Backends to benchmark
FLASHINFER_BACKENDS = ["trtllm", "mnnvl"]
```
**EN:** This top-level block prepares shared state such as `RMS_NORM_OP`, `FUSED_ADD_RMS_NORM_OP`, `RMS_NORM_STATIC_FP8_QUANT_OP`, `FUSED_ADD_RMS_NORM_STATIC_FP8_QUANT_OP`, `SCALED_FP4_QUANT_OUT_OP`, `logger`. It uses `init_logger`, `hasattr`, `logger.warning`, `current_platform.fp8_dtype` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `RMS_NORM_OP`, `FUSED_ADD_RMS_NORM_OP`, `RMS_NORM_STATIC_FP8_QUANT_OP`, `FUSED_ADD_RMS_NORM_STATIC_FP8_QUANT_OP`, `SCALED_FP4_QUANT_OUT_OP`, `logger`。它借助 `init_logger`, `hasattr`, `logger.warning`, `current_platform.fp8_dtype` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `setup_flashinfer_workspace` (lines 92-131)
```python
def setup_flashinfer_workspace(
    backend: str,
    world_size: int,
    rank: int,
    hidden_dim: int,
    max_token_num: int,
    dtype: torch.dtype,
):
    """Setup FlashInfer workspace for fused allreduce operations."""
    global FI_WORKSPACES

    if flashinfer_comm is None:
        return None

    if world_size not in _FI_MAX_SIZES:
        logger.warning("FlashInfer not supported for world size %s", world_size)
        return None

    try:
        kwargs = {}
        if TorchDistBackend is not None:
            kwargs["comm_backend"] = TorchDistBackend(group=dist.group.WORLD)

        workspace = flashinfer_comm.create_allreduce_fusion_workspace(
            backend=backend,
            world_size=world_size,
            rank=rank,
            max_token_num=max_token_num,
            hidden_dim=hidden_dim,
            dtype=dtype,
            **kwargs,
        )

        _FI_WORKSPACES[backend] = workspace
        return workspace
    except Exception as e:
        logger.error(
            "Failed to setup FlashInfer workspace (backend=%s): %s", backend, e
        )
        return None
```
**EN:** `setup_flashinfer_workspace` Setup FlashInfer workspace for fused allreduce operations. It mainly works with `backend`, `world_size`, `rank`, `hidden_dim`, `max_token_num`, ... and relies on `logger.warning`, `TorchDistBackend`, `flashinfer_comm.create_allreduce_fusion_workspace`, `logger.error` plus branching and error handling to move data through this part of the benchmark pipeline.
**CN:** `setup_flashinfer_workspace` 的职责是：Setup FlashInfer workspace for fused allreduce operations。 它主要处理 `backend`, `world_size`, `rank`, `hidden_dim`, `max_token_num`, ...，并结合 `logger.warning`, `TorchDistBackend`, `flashinfer_comm.create_allreduce_fusion_workspace`, `logger.error` 以及 条件分支、异常处理 来完成这一段基准测试流程。

### Function `cleanup_flashinfer_workspaces` (lines 134-148)
```python
def cleanup_flashinfer_workspaces():
    """Cleanup all FlashInfer workspaces."""
    if flashinfer_comm is None:
        return

    for backend, workspace in _FI_WORKSPACES.items():
        try:
            workspace.destroy()
        except Exception as e:
            logger.error(
                "Failed to cleanup FlashInfer workspace (backend=%s): %s",
                backend,
                e,
            )
    _FI_WORKSPACES.clear()
```
**EN:** `cleanup_flashinfer_workspaces` Cleanup all FlashInfer workspaces. It mainly works with no explicit parameters and relies on `_FI_WORKSPACES.items`, `workspace.destroy`, `logger.error`, `_FI_WORKSPACES.clear` plus iteration, branching and error handling to move data through this part of the benchmark pipeline.
**CN:** `cleanup_flashinfer_workspaces` 的职责是：Cleanup all FlashInfer workspaces。 它主要处理 无显式参数，并结合 `_FI_WORKSPACES.items`, `workspace.destroy`, `logger.error`, `_FI_WORKSPACES.clear` 以及 循环迭代、条件分支、异常处理 来完成这一段基准测试流程。

### Class `FlashInferFusedAllReduceParams` (lines 151-166)
```python
class FlashInferFusedAllReduceParams:
    """Parameters for FlashInfer fused allreduce operations."""

    def __init__(
        self,
        max_token_num: int = 1024,
    ):
        self.launch_with_pdl = True
        self.fp32_acc = True
        self.max_token_num = max_token_num

    def get_flashinfer_fused_allreduce_kwargs(self):
        return {
            "launch_with_pdl": self.launch_with_pdl,
            "fp32_acc": self.fp32_acc,
        }
```
**EN:** Class `FlashInferFusedAllReduceParams` is the main object-oriented wrapper for this module. Its docstring says: Parameters for FlashInfer fused allreduce operations. It extends `object` and organizes behavior through `__init__`, `get_flashinfer_fused_allreduce_kwargs`.
**CN:** 类 `FlashInferFusedAllReduceParams` 是该模块中的主要面向对象封装。文档字符串指出：Parameters for FlashInfer fused allreduce operations。它继承自 `object`，并通过 `__init__`, `get_flashinfer_fused_allreduce_kwargs` 组织行为。

### Method `FlashInferFusedAllReduceParams.__init__` (lines 154-160)
```python
    def __init__(
        self,
        max_token_num: int = 1024,
    ):
        self.launch_with_pdl = True
        self.fp32_acc = True
        self.max_token_num = max_token_num
```
**EN:** `__init__` implements a helper used by `benchmark_fused_collective.py`. It mainly works with `max_token_num` and relies on direct statements plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `benchmark_fused_collective.py` 使用的辅助逻辑。 它主要处理 `max_token_num`，并结合 direct statements 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `FlashInferFusedAllReduceParams.get_flashinfer_fused_allreduce_kwargs` (lines 162-166)
```python
    def get_flashinfer_fused_allreduce_kwargs(self):
        return {
            "launch_with_pdl": self.launch_with_pdl,
            "fp32_acc": self.fp32_acc,
        }
```
**EN:** `get_flashinfer_fused_allreduce_kwargs` implements a helper used by `benchmark_fused_collective.py`. It mainly works with no explicit parameters and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_flashinfer_fused_allreduce_kwargs` 负责实现 `benchmark_fused_collective.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Function `flashinfer_fused_allreduce_rmsnorm` (lines 169-208)
```python
def flashinfer_fused_allreduce_rmsnorm(
    input_tensor: torch.Tensor,
    residual: torch.Tensor | None,
    rms_gamma: torch.Tensor,
    rms_eps: float,
    allreduce_params: "FlashInferFusedAllReduceParams",
    workspace: object,
    use_oneshot: bool,
    norm_out: torch.Tensor | None = None,
):
    """FlashInfer fused allreduce + rmsnorm operation."""
    if flashinfer_comm is None or workspace is None:
        raise RuntimeError("FlashInfer not available or workspace not initialized")

    if norm_out is None:
        norm_out = input_tensor
        residual_out = residual
    else:
        residual_out = input_tensor

    layout_code = None
    if workspace.backend == "trtllm":
        layout_code = flashinfer_comm.QuantizationSFLayout.SWIZZLED_128x4

    flashinfer_comm.allreduce_fusion(
        input=input_tensor,
        workspace=workspace,
        pattern=flashinfer_comm.AllReduceFusionPattern.kARResidualRMSNorm,
        residual_in=residual,
        residual_out=residual_out,
        norm_out=norm_out,
        rms_gamma=rms_gamma,
        rms_eps=rms_eps,
        quant_out=None,
        scale_out=None,
        layout_code=layout_code,
        scale_factor=None,
        use_oneshot=use_oneshot,
        **allreduce_params.get_flashinfer_fused_allreduce_kwargs(),
    )
```
**EN:** `flashinfer_fused_allreduce_rmsnorm` FlashInfer fused allreduce + rmsnorm operation. It mainly works with `input_tensor`, `residual`, `rms_gamma`, `rms_eps`, `allreduce_params`, ... and relies on `RuntimeError`, `flashinfer_comm.allreduce_fusion`, `allreduce_params.get_flashinfer_fused_allreduce_kwargs` plus branching to move data through this part of the benchmark pipeline.
**CN:** `flashinfer_fused_allreduce_rmsnorm` 的职责是：FlashInfer fused allreduce + rmsnorm operation。 它主要处理 `input_tensor`, `residual`, `rms_gamma`, `rms_eps`, `allreduce_params`, ...，并结合 `RuntimeError`, `flashinfer_comm.allreduce_fusion`, `allreduce_params.get_flashinfer_fused_allreduce_kwargs` 以及 条件分支 来完成这一段基准测试流程。

### Function `flashinfer_fused_allreduce_rmsnorm_fp8_quant` (lines 211-251)
```python
def flashinfer_fused_allreduce_rmsnorm_fp8_quant(
    input_tensor: torch.Tensor,
    residual: torch.Tensor | None,
    rms_gamma: torch.Tensor,
    rms_eps: float,
    scale_factor: torch.Tensor,
    allreduce_params: FlashInferFusedAllReduceParams,
    workspace: object,
    use_oneshot: bool = True,
    norm_out: torch.Tensor | None = None,
    quant_out: torch.Tensor | None = None,
):
    """FlashInfer fused allreduce + rmsnorm + FP8 quantization.

    Note: Only supported by the trtllm backend.
    """
    if flashinfer_comm is None or workspace is None:
        raise RuntimeError("FlashInfer not available or workspace not initialized")

    if norm_out is None:
        norm_out = input_tensor
        residual_out = residual
    else:
        residual_out = input_tensor

    flashinfer_comm.allreduce_fusion(
        input=input_tensor,
        workspace=workspace,
        pattern=flashinfer_comm.AllReduceFusionPattern.kARResidualRMSNormFP8Quant,
        residual_in=residual,
        residual_out=residual_out,
        norm_out=norm_out,
        rms_gamma=rms_gamma,
        rms_eps=rms_eps,
        quant_out=quant_out,
        scale_out=None,
        layout_code=flashinfer_comm.QuantizationSFLayout.SWIZZLED_128x4,
        scale_factor=scale_factor,
        use_oneshot=use_oneshot,
        **allreduce_params.get_flashinfer_fused_allreduce_kwargs(),
    )
```
**EN:** `flashinfer_fused_allreduce_rmsnorm_fp8_quant` FlashInfer fused allreduce + rmsnorm + FP8 quantization. It mainly works with `input_tensor`, `residual`, `rms_gamma`, `rms_eps`, `scale_factor`, ... and relies on `RuntimeError`, `flashinfer_comm.allreduce_fusion`, `allreduce_params.get_flashinfer_fused_allreduce_kwargs` plus branching to move data through this part of the benchmark pipeline.
**CN:** `flashinfer_fused_allreduce_rmsnorm_fp8_quant` 的职责是：FlashInfer fused allreduce + rmsnorm + FP8 quantization。 它主要处理 `input_tensor`, `residual`, `rms_gamma`, `rms_eps`, `scale_factor`, ...，并结合 `RuntimeError`, `flashinfer_comm.allreduce_fusion`, `allreduce_params.get_flashinfer_fused_allreduce_kwargs` 以及 条件分支 来完成这一段基准测试流程。

### Function `flashinfer_fused_allreduce_rmsnorm_fp4_quant` (lines 254-295)
```python
def flashinfer_fused_allreduce_rmsnorm_fp4_quant(
    input_tensor: torch.Tensor,
    residual: torch.Tensor | None,
    rms_gamma: torch.Tensor,
    rms_eps: float,
    input_global_scale: torch.Tensor,
    allreduce_params: FlashInferFusedAllReduceParams,
    workspace: object,
    quant_out: torch.Tensor,
    use_oneshot: bool,
    output_scale: torch.Tensor,
    norm_out: torch.Tensor | None = None,
):
    """FlashInfer fused allreduce + rmsnorm + FP4 quantization.

    Note: Only supported by the trtllm backend.
    """
    if flashinfer_comm is None or workspace is None:
        raise RuntimeError("FlashInfer not available or workspace not initialized")

    if norm_out is None:
        norm_out = input_tensor
        residual_out = residual
    else:
        residual_out = input_tensor

    flashinfer_comm.allreduce_fusion(
        input=input_tensor,
        workspace=workspace,
        pattern=flashinfer_comm.AllReduceFusionPattern.kARResidualRMSNormFP4Quant,
        residual_in=residual,
        residual_out=residual_out,
        norm_out=norm_out,
        rms_gamma=rms_gamma,
        rms_eps=rms_eps,
        quant_out=quant_out,
        scale_out=output_scale,
        layout_code=flashinfer_comm.QuantizationSFLayout.SWIZZLED_128x4,
        scale_factor=input_global_scale,
        use_oneshot=use_oneshot,
        **allreduce_params.get_flashinfer_fused_allreduce_kwargs(),
    )
```
**EN:** `flashinfer_fused_allreduce_rmsnorm_fp4_quant` FlashInfer fused allreduce + rmsnorm + FP4 quantization. It mainly works with `input_tensor`, `residual`, `rms_gamma`, `rms_eps`, `input_global_scale`, ... and relies on `RuntimeError`, `flashinfer_comm.allreduce_fusion`, `allreduce_params.get_flashinfer_fused_allreduce_kwargs` plus branching to move data through this part of the benchmark pipeline.
**CN:** `flashinfer_fused_allreduce_rmsnorm_fp4_quant` 的职责是：FlashInfer fused allreduce + rmsnorm + FP4 quantization。 它主要处理 `input_tensor`, `residual`, `rms_gamma`, `rms_eps`, `input_global_scale`, ...，并结合 `RuntimeError`, `flashinfer_comm.allreduce_fusion`, `allreduce_params.get_flashinfer_fused_allreduce_kwargs` 以及 条件分支 来完成这一段基准测试流程。

### Class `VllmFusedAllreduce` (lines 298-355)
```python
class VllmFusedAllreduce:
    def __init__(self, hidden_dim, dtype):
        self.rms_eps = 1e-6
        self.rms_norm = RMSNorm(hidden_dim, eps=self.rms_eps, dtype=dtype)
        self.fp8_quant = QuantFP8(
            static=True,
            group_shape=GroupShape.PER_TENSOR,
        )

    def allreduce_rmsnorm(
        self, input_tensor: torch.Tensor, residual: torch.Tensor | None
    ):
        allreduce_out = tensor_model_parallel_all_reduce(input_tensor)
        return self.rms_norm(allreduce_out, residual)

    def allreduce_rmsnorm_fp8_quant(
        self,
        input_tensor: torch.Tensor,
        residual: torch.Tensor | None,
        scale_factor: torch.Tensor,
    ):
        allreduce_out = tensor_model_parallel_all_reduce(input_tensor)
        rms_out = self.rms_norm(allreduce_out, residual)
        if residual is None:
            quant_out = self.fp8_quant(rms_out, scale_factor)
            return quant_out
        else:
            rms_out, residual_out = rms_out
            quant_out = self.fp8_quant(rms_out, scale_factor)
            return quant_out, residual_out

    def allreduce_rmsnorm_fp4_quant(
        self,
        input_tensor: torch.Tensor,
    # ... omitted for brevity ...
            output=quant_out,
            output_scale=output_scale,
        )

        if residual is None:
            return quant_out, output_scale
        else:
            return quant_out, residual_out, output_scale
```
**EN:** Class `VllmFusedAllreduce` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as `__init__`, `allreduce_rmsnorm`, `allreduce_rmsnorm_fp8_quant`, `allreduce_rmsnorm_fp4_quant`.
**CN:** 类 `VllmFusedAllreduce` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 `__init__`, `allreduce_rmsnorm`, `allreduce_rmsnorm_fp8_quant`, `allreduce_rmsnorm_fp4_quant` 等方法。

### Method `VllmFusedAllreduce.__init__` (lines 299-305)
```python
    def __init__(self, hidden_dim, dtype):
        self.rms_eps = 1e-6
        self.rms_norm = RMSNorm(hidden_dim, eps=self.rms_eps, dtype=dtype)
        self.fp8_quant = QuantFP8(
            static=True,
            group_shape=GroupShape.PER_TENSOR,
        )
```
**EN:** `__init__` implements a helper used by `benchmark_fused_collective.py`. It mainly works with `hidden_dim`, `dtype` and relies on `RMSNorm`, `QuantFP8` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `benchmark_fused_collective.py` 使用的辅助逻辑。 它主要处理 `hidden_dim`, `dtype`，并结合 `RMSNorm`, `QuantFP8` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `VllmFusedAllreduce.allreduce_rmsnorm` (lines 307-311)
```python
    def allreduce_rmsnorm(
        self, input_tensor: torch.Tensor, residual: torch.Tensor | None
    ):
        allreduce_out = tensor_model_parallel_all_reduce(input_tensor)
        return self.rms_norm(allreduce_out, residual)
```
**EN:** `allreduce_rmsnorm` coordinates or measures RMSNorm kernel benchmarks. It mainly works with `input_tensor`, `residual` and relies on `tensor_model_parallel_all_reduce`, `self.rms_norm` plus value production to move data through this part of the benchmark pipeline.
**CN:** `allreduce_rmsnorm` 负责协调或测量RMSNorm 内核基准。 它主要处理 `input_tensor`, `residual`，并结合 `tensor_model_parallel_all_reduce`, `self.rms_norm` 以及 结果返回 来完成这一段基准测试流程。

### Method `VllmFusedAllreduce.allreduce_rmsnorm_fp8_quant` (lines 313-327)
```python
    def allreduce_rmsnorm_fp8_quant(
        self,
        input_tensor: torch.Tensor,
        residual: torch.Tensor | None,
        scale_factor: torch.Tensor,
    ):
        allreduce_out = tensor_model_parallel_all_reduce(input_tensor)
        rms_out = self.rms_norm(allreduce_out, residual)
        if residual is None:
            quant_out = self.fp8_quant(rms_out, scale_factor)
            return quant_out
        else:
            rms_out, residual_out = rms_out
            quant_out = self.fp8_quant(rms_out, scale_factor)
            return quant_out, residual_out
```
**EN:** `allreduce_rmsnorm_fp8_quant` coordinates or measures FP8 kernel experiments. It mainly works with `input_tensor`, `residual`, `scale_factor` and relies on `tensor_model_parallel_all_reduce`, `self.rms_norm`, `self.fp8_quant` plus branching to move data through this part of the benchmark pipeline.
**CN:** `allreduce_rmsnorm_fp8_quant` 负责协调或测量FP8 内核实验。 它主要处理 `input_tensor`, `residual`, `scale_factor`，并结合 `tensor_model_parallel_all_reduce`, `self.rms_norm`, `self.fp8_quant` 以及 条件分支 来完成这一段基准测试流程。

### Method `VllmFusedAllreduce.allreduce_rmsnorm_fp4_quant` (lines 329-355)
```python
    def allreduce_rmsnorm_fp4_quant(
        self,
        input_tensor: torch.Tensor,
        residual: torch.Tensor | None,
        input_global_scale: torch.Tensor,
        quant_out: torch.Tensor,
        output_scale: torch.Tensor,
    ):
        allreduce_out = tensor_model_parallel_all_reduce(input_tensor)
        rms_output = self.rms_norm(allreduce_out, residual)
        if residual is None:
            rms_out = rms_output
        else:
            rms_out, residual_out = rms_output

        SCALED_FP4_QUANT_OUT_OP(
            rms_out,
            input_global_scale,
            True,
            output=quant_out,
            output_scale=output_scale,
        )

        if residual is None:
            return quant_out, output_scale
        else:
            return quant_out, residual_out, output_scale
```
**EN:** `allreduce_rmsnorm_fp4_quant` coordinates or measures quantization benchmarks. It mainly works with `input_tensor`, `residual`, `input_global_scale`, `quant_out`, `output_scale` and relies on `tensor_model_parallel_all_reduce`, `self.rms_norm`, `SCALED_FP4_QUANT_OUT_OP` plus branching to move data through this part of the benchmark pipeline.
**CN:** `allreduce_rmsnorm_fp4_quant` 负责协调或测量量化基准。 它主要处理 `input_tensor`, `residual`, `input_global_scale`, `quant_out`, `output_scale`，并结合 `tensor_model_parallel_all_reduce`, `self.rms_norm`, `SCALED_FP4_QUANT_OUT_OP` 以及 条件分支 来完成这一段基准测试流程。

### Function `create_test_tensors` (lines 358-390)
```python
def create_test_tensors(
    num_tokens: int, hidden_dim: int, dtype: torch.dtype, use_residual: bool = True
):
    """Create test tensors for benchmarking."""
    input_tensor = torch.randn(num_tokens, hidden_dim, dtype=dtype)
    residual = (
        torch.randn_like(input_tensor)
        if use_residual
        else torch.zeros_like(input_tensor)
    )
    rms_gamma = torch.ones(hidden_dim, dtype=dtype)
    norm_out = None if use_residual else torch.empty_like(input_tensor)

    # Quantization scales
    scale_fp8 = torch.tensor(1.0, dtype=torch.float32)
    scale_fp4 = torch.tensor(1.0, dtype=torch.float32)
    quant_out_fp8 = torch.empty_like(input_tensor, dtype=FP8_DTYPE)
    # Pre-allocate FP4 output tensors (to avoid allocation overhead in benchmarks)
    fp4_quant_out, fp4_output_scale = create_fp4_output_tensors(
        num_tokens, hidden_dim, input_tensor.device, True
    )

    return (
        input_tensor,
        norm_out,
        residual,
        rms_gamma,
        scale_fp8,
        quant_out_fp8,
        scale_fp4,
        fp4_quant_out,
        fp4_output_scale,
    )
```
**EN:** `create_test_tensors` Create test tensors for benchmarking. It mainly works with `num_tokens`, `hidden_dim`, `dtype`, `use_residual` and relies on `torch.randn`, `torch.randn_like`, `torch.zeros_like`, `torch.ones`, `torch.empty_like`, `torch.tensor` plus value production to move data through this part of the benchmark pipeline.
**CN:** `create_test_tensors` 的职责是：Create test tensors for benchmarking。 它主要处理 `num_tokens`, `hidden_dim`, `dtype`, `use_residual`，并结合 `torch.randn`, `torch.randn_like`, `torch.zeros_like`, `torch.ones`, `torch.empty_like`, `torch.tensor` 以及 结果返回 来完成这一段基准测试流程。

### Function `benchmark_operation` (lines 393-429)
```python
def benchmark_operation(
    operation_func, *args, warmup: int = 5, trials: int = 20, **kwargs
):
    """Benchmark a single operation using CUDA graphs."""
    # Warmup before graph capture
    for _ in range(warmup):
        operation_func(*args, **kwargs)
    torch.accelerator.synchronize()

    # Create CUDA graph
    graph = torch.cuda.CUDAGraph()
    num_op_per_cudagraph = 10

    # Use vLLM's graph_capture to make tensor_model_parallel_all_reduce graph-safe
    device = torch.device(f"cuda:{torch.accelerator.current_device_index()}")
    with graph_capture(device=device), torch.cuda.graph(graph):
        for _ in range(num_op_per_cudagraph):
            operation_func(*args, **kwargs)

    # Graph warmup
    torch.accelerator.synchronize()
    for _ in range(warmup):
        graph.replay()

    # Benchmark with CUDA graph
    torch.accelerator.synchronize()
    start_time = time.perf_counter()

    for _ in range(trials // num_op_per_cudagraph):
        # operation_func(*args, **kwargs)
        graph.replay()

    torch.accelerator.synchronize()
    end_time = time.perf_counter()

    avg_time_ms = ((end_time - start_time) / trials) * 1000
    return avg_time_ms
```
**EN:** `benchmark_operation` Benchmark a single operation using CUDA graphs. It mainly works with `operation_func`, `warmup`, `trials`, `*args`, `**kwargs` and relies on `range`, `operation_func`, `torch.accelerator.synchronize`, `torch.cuda.CUDAGraph`, `torch.device`, `torch.accelerator.current_device_index` plus iteration and context management to move data through this part of the benchmark pipeline.
**CN:** `benchmark_operation` 的职责是：Benchmark a single operation using CUDA graphs。 它主要处理 `operation_func`, `warmup`, `trials`, `*args`, `**kwargs`，并结合 `range`, `operation_func`, `torch.accelerator.synchronize`, `torch.cuda.CUDAGraph`, `torch.device`, `torch.accelerator.current_device_index` 以及 循环迭代、上下文管理 来完成这一段基准测试流程。

### Function `run_benchmarks` (lines 432-714)
```python
def run_benchmarks(
    num_tokens: int,
    hidden_dim: int,
    dtype: torch.dtype,
    use_residual: bool,
    allreduce_params: FlashInferFusedAllReduceParams | None,
    workspaces: dict,
    quant_modes: set[str],
    no_oneshot: bool,
):
    """Run all benchmarks for given configuration.

    Args:
        allreduce_params: Shared parameters for FlashInfer fused allreduce.
        workspaces: Dict mapping backend name ("trtllm", "mnnvl") to workspace.
        quant_modes: Set of quantization modes: "none", "fp8", "fp4".
    """
    (
        input_tensor,
        norm_out,
        residual,
        rms_gamma,
        scale_fp8,
        quant_out_fp8,
        scale_fp4,
        fp4_quant_out,
        fp4_output_scale,
    ) = create_test_tensors(num_tokens, hidden_dim, dtype, use_residual)

    rms_eps = 1e-6
    results = {}
    use_oneshot_options = [False] if no_oneshot else [True, False]

    if "none" in quant_modes:
    # ... omitted for brevity ...
                except Exception as e:
                    logger.error(
                        "FlashInfer (trtllm) Fused AllReduce+RMSNorm+FP4 failed: %s",
                        e,
                    )
                    results[key] = float("inf")

    return results
```
**EN:** `run_benchmarks` Run all benchmarks for given configuration. It mainly works with `num_tokens`, `hidden_dim`, `dtype`, `use_residual`, `allreduce_params`, ... and relies on `create_test_tensors`, `set_current_vllm_config`, `VllmConfig`, `CompilationConfig`, `VllmFusedAllreduce`, `benchmark_operation` plus iteration, branching, error handling and context management to move data through this part of the benchmark pipeline.
**CN:** `run_benchmarks` 的职责是：Run all benchmarks for given configuration。 它主要处理 `num_tokens`, `hidden_dim`, `dtype`, `use_residual`, `allreduce_params`, ...，并结合 `create_test_tensors`, `set_current_vllm_config`, `VllmConfig`, `CompilationConfig`, `VllmFusedAllreduce`, `benchmark_operation` 以及 循环迭代、条件分支、异常处理、上下文管理 来完成这一段基准测试流程。

### Function `prepare_results_with_speedups` (lines 717-813)
```python
def prepare_results_with_speedups(results_dict):
    """Prepare results with speedup calculations based on dynamic baseline selection."""
    prepared_results = []

    # Determine the fastest baseline for each operation type
    def get_fastest_baseline(op_name, results_dict):
        """Get the fastest baseline between standard and native_compiled versions."""
        if "fp8_quant" in op_name:
            candidates = [
                "standard_allreduce_rmsnorm_fp8_quant",
                "standard_allreduce_rmsnorm_fp8_quant_native_compiled",
            ]
        elif "fp4_quant" in op_name:
            candidates = [
                "standard_allreduce_rmsnorm_fp4_quant",
                "standard_allreduce_rmsnorm_fp4_quant_native_compiled",
            ]
        else:
            candidates = [
                "standard_allreduce_rmsnorm",
                "standard_allreduce_rmsnorm_native_compiled",
            ]

        # Find the fastest among available candidates
        fastest_time = float("inf")
        fastest_baseline = None

        for candidate in candidates:
            if (
                candidate in results_dict
                and results_dict[candidate] != float("inf")
                and results_dict[candidate] < fastest_time
            ):
                fastest_time = results_dict[candidate]
    # ... omitted for brevity ...
                "operation": op_name,
                "time_ms": time_ms,
                "time_str": time_str,
                "speedup_str": speedup_str,
            }
        )

    return prepared_results
```
**EN:** `prepare_results_with_speedups` Prepare results with speedup calculations based on dynamic baseline selection. It mainly works with `results_dict` and relies on `float`, `op_name.startswith`, `op_name.endswith`, `get_fastest_baseline`, `results_dict.items`, `dynamic_baseline_mapping.get` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `prepare_results_with_speedups` 的职责是：Prepare results with speedup calculations based on dynamic baseline selection。 它主要处理 `results_dict`，并结合 `float`, `op_name.startswith`, `op_name.endswith`, `get_fastest_baseline`, `results_dict.items`, `dynamic_baseline_mapping.get` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `print_results` (lines 816-850)
```python
def print_results(
    results_dict,
    num_tokens,
    hidden_dim,
    dtype,
    use_residual,
    quant_modes,
    input_size_mb,
):
    """Print benchmark results in a formatted table."""
    print(f"\n{'=' * 80}")
    print(
        f"Results: num_tokens={num_tokens}, hidden_dim={hidden_dim} "
        f"(input size: {input_size_mb:.2f} MB)"
    )
    print(
        f"dtype={dtype}, residual={'yes' if use_residual else 'no'}, "
        f"quant_modes={','.join(sorted(list(quant_modes)))}"
    )
    print(f"{'=' * 80}")
    print(f"{'Operation':<50} {'Time (ms)':<12} {'Speedup':<10}")
    print(f"{'-' * 80}")

    # Prepare results with speedup calculations
    prepared_results = prepare_results_with_speedups(results_dict)

    for result in prepared_results:
        if result["time_ms"] == float("inf"):
            time_display = result["time_str"]
        else:
            time_display = f"{result['time_ms']:.3f}"

        print(
            f"{result['operation']:<50} {time_display:<12} {result['speedup_str']:<10}"
        )
```
**EN:** `print_results` Print benchmark results in a formatted table. It mainly works with `results_dict`, `num_tokens`, `hidden_dim`, `dtype`, `use_residual`, ... and relies on `print`, `join`, `sorted`, `list`, `prepare_results_with_speedups`, `float` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `print_results` 的职责是：Print benchmark results in a formatted table。 它主要处理 `results_dict`, `num_tokens`, `hidden_dim`, `dtype`, `use_residual`, ...，并结合 `print`, `join`, `sorted`, `list`, `prepare_results_with_speedups`, `float` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `format_results_markdown` (lines 853-901)
```python
def format_results_markdown(
    all_results: list[dict], world_size: int, args: argparse.Namespace
) -> str:
    """Format all benchmark results as markdown."""
    lines: list[str] = []
    lines.append("# FlashInfer Fused Collective Operations Benchmark Results")
    lines.append("")
    lines.append(f"**World Size:** {world_size}  ")
    lines.append(f"**Hidden Dimension:** {args.hidden_dim}  ")
    lines.append(f"**Warmup Iterations:** {args.warmup}  ")
    lines.append(f"**Benchmark Trials:** {args.trials}  ")
    modes = ",".join(all_results[0]["quant_modes"]) if all_results else "N/A"
    lines.append(f"**Quantization Modes:** {modes}  ")
    lines.append("")
    lines.append("---")
    lines.append("")

    for entry in all_results:
        num_tokens = entry["num_tokens"]
        dtype = entry["dtype"]
        use_residual = entry["use_residual"]
        results_dict = entry["results"]
        input_size_mb = entry["input_size_mb"]
        residual_str = "with residual" if use_residual else "no residual"

        lines.append(
            f"## Configuration: num_tokens={num_tokens}, dtype={dtype}, {residual_str}"
        )
        lines.append(f"**Input Size:** {input_size_mb:.2f} MB")
        lines.append("")

        prepared = prepare_results_with_speedups(results_dict)
        # Build DataFrame for markdown export
        rows = [
    # ... omitted for brevity ...
        df = pd.DataFrame(rows)
        if df.empty:
            lines.append("No results.")
        else:
            lines.append(df.to_markdown(index=False))
        lines.append("")

    return "\n".join(lines)
```
**EN:** `format_results_markdown` Format all benchmark results as markdown. It mainly works with `all_results`, `world_size`, `args` and relies on `lines.append`, `join`, `prepare_results_with_speedups`, `r.replace.title`, `r.replace`, `pd.DataFrame` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `format_results_markdown` 的职责是：Format all benchmark results as markdown。 它主要处理 `all_results`, `world_size`, `args`，并结合 `lines.append`, `join`, `prepare_results_with_speedups`, `r.replace.title`, `r.replace`, `pd.DataFrame` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `save_results_to_file` (lines 904-924)
```python
def save_results_to_file(
    all_results: list[dict], world_size: int, args: argparse.Namespace, rank: int
):
    """Save benchmark results to markdown file (only on rank 0)."""
    if rank != 0:
        return

    if not all_results:
        logger.warning("No results to save")
        return

    output_path = args.output_file

    try:
        markdown_content = format_results_markdown(all_results, world_size, args)

        with open(output_path, "a") as f:
            f.write(markdown_content)

    except Exception as e:
        logger.error("Failed to save results to file: %s", e)
```
**EN:** `save_results_to_file` Save benchmark results to markdown file (only on rank 0). It mainly works with `all_results`, `world_size`, `args`, `rank` and relies on `logger.warning`, `format_results_markdown`, `open`, `f.write`, `logger.error` plus branching, error handling and context management to move data through this part of the benchmark pipeline.
**CN:** `save_results_to_file` 的职责是：Save benchmark results to markdown file (only on rank 0)。 它主要处理 `all_results`, `world_size`, `args`, `rank`，并结合 `logger.warning`, `format_results_markdown`, `open`, `f.write`, `logger.error` 以及 条件分支、异常处理、上下文管理 来完成这一段基准测试流程。

### Function `main` (lines 927-1142)
```python
def main():
    parser = argparse.ArgumentParser(
        description="Benchmark fused collective operations"
    )
    parser.add_argument(
        "--num-tokens",
        type=int,
        nargs="+",
        default=[128, 512, 1024, 2048],
        help="Numbers of tokens to test",
    )
    parser.add_argument(
        "--hidden-dim", type=int, default=8192, help="Hidden dimension size"
    )
    parser.add_argument(
        "--dtypes",
        type=str,
        nargs="+",
        default=["bfloat16"],
        choices=["float16", "bfloat16", "float32"],
        help="Data types to test",
    )
    parser.add_argument(
        "--no-residual",
        action="store_true",
        help="Skip residual connection tests",
    )

    parser.add_argument(
        "--quant-modes",
        type=str,
        default="none,fp8,fp4",
        help=(
            "Comma-separated quantization modes to run: none, fp8, fp4. "
    # ... omitted for brevity ...
        if args.output_file and rank == 0:
            save_results_to_file(all_results, world_size, args, rank)

    finally:
        # Cleanup
        cleanup_flashinfer_workspaces()

        dist.barrier()
```
**EN:** `main` serves as the main entry point for this script. It mainly works with no explicit parameters and relies on `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `RuntimeError`, `int`, `torch.device` plus iteration, branching and error handling to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 无显式参数，并结合 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `RuntimeError`, `int`, `torch.device` 以及 循环迭代、条件分支、异常处理 来完成这一段基准测试流程。

### Entry point (lines 1145-1149)
```python
if __name__ == "__main__":
    from vllm.config import VllmConfig, set_current_vllm_config

    with set_current_vllm_config(VllmConfig()):
        main()
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `set_current_vllm_config`, `VllmConfig`, `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `set_current_vllm_config`, `VllmConfig`, `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Distributed execution: the code coordinates work across processes, devices, or Ray workers.
- **CN:** 分布式执行：代码会在多个进程、设备或 Ray worker 之间协调工作。
- **EN:** Result presentation: the module aggregates measurements and renders tables or plots.
- **CN:** 结果呈现：该模块会汇总测量结果并生成表格或图形。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `itertools`, `os`, `time`.
- **CN:** 标准库依赖：`argparse`, `itertools`, `os`, `time`。
- **EN:** Third-party packages: `pandas`, `torch`, `torch.distributed`.
- **CN:** 第三方依赖：`pandas`, `torch`, `torch.distributed`。
- **EN:** Internal modules: `vllm._custom_ops`, `vllm.config.vllm`, `vllm.distributed`, `vllm.distributed.parallel_state`, `vllm.logger`, `vllm.model_executor.layers.layernorm`.
- **CN:** 内部模块：`vllm._custom_ops`, `vllm.config.vllm`, `vllm.distributed`, `vllm.distributed.parallel_state`, `vllm.logger`, `vllm.model_executor.layers.layernorm`。
