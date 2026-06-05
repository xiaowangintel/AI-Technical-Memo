# benchmark_fused_collective.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/flashinfer_allreduce_fusion/benchmark_fused_collective.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels flashinfer allreduce fusion fused. It primarily builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 kernels flashinfer allreduce fusion fused 相关流程。它主要用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 3-82: Imports and setup / 导入与初始化
```python
"""
Benchmark for FlashInfer fused collective operations vs standard operations.

This benchmark compares:
1. FlashInfer's trtllm_allreduce_fusion (fused allreduce + rmsnorm + optional quant)
2. Standard tensor_model_parallel_all_reduce + separate rmsnorm/quant operations

Usage with torchrun:
    torchrun --nproc_per_node=2 benchmark/kernels/flashinfer_allreduce_fusion/benchmark_fused_collective.py --no-quant --hidden-dim 1024 --seq-len 512 1024 2048 4096 --trials 100
    torchrun --nproc_per_node=2 benchmark/kernels/flashinfer_allreduce_fusion/benchmark_fused_collective.py --quant-fp8 --hidden-dim 1024 --seq-len 512 1024 2048 4096 --trials 100
    torchrun --nproc_per_node=2 benchmark/kernels/flashinfer_allreduce_fusion/benchmark_fused_collective.py --quant-fp4 --hidden-dim 1024 --seq-len 512 1024 2048 4096 --trials 100

    torchrun --nproc_per_node=2 benchmark/kernels/flashinfer_allreduce_fusion/benchmark_fused_collective.py --no-quant --hidden-dim 4096 --seq-len 512 1024 2048 4096 --trials 100
    torchrun --nproc_per_node=2 benchmark/kernels/flashinfer_allreduce_fusion/benchmark_fused_collective.py --quant-fp8 --hidden-dim 4096 --seq-len 512 1024 2048 4096 --trials 100
    torchrun --nproc_per_node=2 benchmark/kernels/flashinfer_allreduce_fusion/benchmark_fused_collective.py --quant-fp4 --hidden-dim 4096 --seq-len 512 1024 2048 4096 --trials 100
"""

import argparse
import contextlib
import itertools
import logging
import os
import time
from typing import Optional

import torch  # type: ignore
import torch.distributed as dist  # type: ignore

from sglang.srt.distributed import get_tp_group, tensor_model_parallel_all_reduce
from sglang.srt.distributed.parallel_state import (
    cleanup_dist_env_and_memory,
    graph_capture,
    init_distributed_environment,
    initialize_model_parallel,
)
from sglang.srt.layers.layernorm import RMSNorm  # noqa
from sglang.srt.layers.quantization.fp8_kernel import fp8_dtype as SGLANG_FP8_DTYPE
from sglang.srt.layers.quantization.fp8_kernel import static_quant_fp8

try:
    from sgl_kernel import fused_add_rmsnorm as SGL_FUSED_ADD_RMS_NORM
    from sgl_kernel import rmsnorm as SGL_RMS_NORM

    from sglang.jit_kernel.nvfp4 import scaled_fp4_quant as SGL_SCALED_FP4_QUANT
except Exception:  # pragma: no cover - fallback on non-supported platforms
    SGL_FUSED_ADD_RMS_NORM = None
    SGL_RMS_NORM = None
    SGL_SCALED_FP4_QUANT = None

FP8_DTYPE = SGLANG_FP8_DTYPE

logger = logging.getLogger(__name__)

# Try to import FlashInfer
try:
    import flashinfer.comm as flashinfer_comm  # type: ignore

    if not hasattr(flashinfer_comm, "trtllm_allreduce_fusion"):
        flashinfer_comm = None
        logger.warning(
            "FlashInfer comm module found but missing trtllm_allreduce_fusion"
        )
except ImportError:
    flashinfer_comm = None
    logger.warning("FlashInfer not found, only benchmarking standard operations")

# Constants
MiB = 1024 * 1024

# FlashInfer max sizes per world size
# Enable 64MB for 2, 4, 8 world sizes to verify large input sizes
# use --disable-oneshot to disable oneshot mode for very large input sizes
_FI_MAX_SIZES = {
    2: 64 * MiB,  # 64MB
    4: 64 * MiB,  # 64MB
    8: 64 * MiB,  # 64MB
}

# Global workspace tensor for FlashInfer
_FI_WORKSPACE_TENSOR = None
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

### Lines 85-119: Function `setup_flashinfer_workspace` / 函数 `setup_flashinfer_workspace`
```python
def setup_flashinfer_workspace(
    world_size: int,
    rank: int,
    hidden_dim: int,
    max_token_num: int,
    use_fp32_lamport: bool = False,
):
    """Setup FlashInfer workspace for fused allreduce operations."""
    global _FI_WORKSPACE_TENSOR

    if flashinfer_comm is None:
        return None, None

    if world_size not in _FI_MAX_SIZES:
        logger.warning("FlashInfer not supported for world size %s", world_size)
        return None, None

    try:
        # Create IPC workspace
        ipc_handles, workspace_tensor = (
            flashinfer_comm.trtllm_create_ipc_workspace_for_all_reduce_fusion(
                tp_rank=rank,
                tp_size=world_size,
                max_token_num=max_token_num,
                hidden_dim=hidden_dim,
                group=get_tp_group().device_group,
                use_fp32_lamport=use_fp32_lamport,
            )
        )

        _FI_WORKSPACE_TENSOR = workspace_tensor
        return ipc_handles, workspace_tensor
    except Exception as e:
        logger.error("Failed to setup FlashInfer workspace: %s", e)
        return None, None
```
**EN:** `setup_flashinfer_workspace` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. The docstring frames it as: Setup FlashInfer workspace for fused allreduce operations. It returns `(None, None)` to the caller. Notable calls include `logger.warning`, `flashinfer_comm.trtllm_create_ipc_workspace_for_all_reduce_fusion`, `logger.error`.
**CN:** `setup_flashinfer_workspace` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `(None, None)`。其中较关键的调用包括 `logger.warning`, `flashinfer_comm.trtllm_create_ipc_workspace_for_all_reduce_fusion`, `logger.error`。

### Lines 122-131: Function `cleanup_flashinfer_workspace` / 函数 `cleanup_flashinfer_workspace`
```python
def cleanup_flashinfer_workspace(ipc_handles):
    """Cleanup FlashInfer workspace."""
    if flashinfer_comm is None or ipc_handles is None:
        return

    try:
        group = get_tp_group().device_group
        flashinfer_comm.trtllm_destroy_ipc_workspace_for_all_reduce(ipc_handles, group)
    except Exception as e:
        logger.error("Failed to cleanup FlashInfer workspace: %s", e)
```
**EN:** `cleanup_flashinfer_workspace` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: Cleanup FlashInfer workspace. Notable calls include `flashinfer_comm.trtllm_destroy_ipc_workspace_for_all_reduce`, `get_tp_group`, `logger.error`.
**CN:** `cleanup_flashinfer_workspace` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `flashinfer_comm.trtllm_destroy_ipc_workspace_for_all_reduce`, `get_tp_group`, `logger.error`。

### Lines 134-134: Class `FlashInferFusedAllReduceParams` declaration / 类 `FlashInferFusedAllReduceParams` 声明
```python
class FlashInferFusedAllReduceParams:
```
**EN:** This block introduces class `FlashInferFusedAllReduceParams`, which prepares tensors and invokes GPU kernels.
**CN:** 该代码块引入类 `FlashInferFusedAllReduceParams`，用于准备张量并调用 GPU 内核。

### Lines 135-135: Class-level state / 类级状态
```python
    """Parameters for FlashInfer fused allreduce operations."""
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized. In practice it prepares tensors and invokes GPU kernels.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。 从实现上看，它会准备张量并调用 GPU 内核。

### Lines 137-150: Method `FlashInferFusedAllReduceParams.__init__` / 方法 `FlashInferFusedAllReduceParams.__init__`
```python
    def __init__(
        self,
        rank: int,
        world_size: int,
        use_fp32_lamport: bool = False,
        max_token_num: int = 1024,
    ):
        self.rank = rank
        self.world_size = world_size
        self.use_fp32_lamport = use_fp32_lamport
        self.trigger_completion_at_end = True
        self.launch_with_pdl = True
        self.fp32_acc = True
        self.max_token_num = max_token_num
```
**EN:** `FlashInferFusedAllReduceParams.__init__` is a method that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs.
**CN:** `FlashInferFusedAllReduceParams.__init__` 是一个方法，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。

### Lines 152-159: Method `FlashInferFusedAllReduceParams.get_trtllm_fused_allreduce_kwargs` / 方法 `FlashInferFusedAllReduceParams.get_trtllm_fused_allreduce_kwargs`
```python
    def get_trtllm_fused_allreduce_kwargs(self):
        return {
            "world_rank": self.rank,
            "world_size": self.world_size,
            "launch_with_pdl": self.launch_with_pdl,
            "trigger_completion_at_end": self.trigger_completion_at_end,
            "fp32_acc": self.fp32_acc,
        }
```
**EN:** `FlashInferFusedAllReduceParams.get_trtllm_fused_allreduce_kwargs` is a method that prepares tensors and invokes GPU kernels. It returns `{'world_rank': self.rank, 'world_size': self.world_size, 'launch_with_pdl': self.launch_with_pdl,...` to the caller.
**CN:** `FlashInferFusedAllReduceParams.get_trtllm_fused_allreduce_kwargs` 是一个方法，用于准备张量并调用 GPU 内核。它会向调用方返回 `{'world_rank': self.rank, 'world_size': self.world_size, 'launch_with_pdl': self.launch_with_pdl,...`。

### Lines 162-199: Function `flashinfer_fused_allreduce_rmsnorm` / 函数 `flashinfer_fused_allreduce_rmsnorm`
```python
def flashinfer_fused_allreduce_rmsnorm(
    input_tensor: torch.Tensor,
    residual: Optional[torch.Tensor],
    rms_gamma: torch.Tensor,
    rms_eps: float,
    allreduce_params: "FlashInferFusedAllReduceParams",
    use_oneshot: bool,
    norm_out: Optional[torch.Tensor] = None,
):
    """FlashInfer fused allreduce + rmsnorm operation."""
    if flashinfer_comm is None or _FI_WORKSPACE_TENSOR is None:
        raise RuntimeError("FlashInfer not available or workspace not initialized")

    if norm_out is None:
        norm_out = input_tensor
        residual_out = residual
    else:
        residual_out = input_tensor

    flashinfer_comm.trtllm_allreduce_fusion(
        allreduce_in=input_tensor,
        token_num=input_tensor.shape[0],
        residual_in=residual,
        residual_out=residual_out,
        norm_out=norm_out,
        rms_gamma=rms_gamma,
        rms_eps=rms_eps,
        hidden_dim=input_tensor.shape[-1],
        workspace_ptrs=_FI_WORKSPACE_TENSOR,
        pattern_code=flashinfer_comm.AllReduceFusionPattern.kARResidualRMSNorm,
        allreduce_out=None,
        quant_out=None,
        scale_out=None,
        layout_code=None,
        scale_factor=None,
        use_oneshot=use_oneshot,
        **allreduce_params.get_trtllm_fused_allreduce_kwargs(),
    )
```
**EN:** `flashinfer_fused_allreduce_rmsnorm` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. The docstring frames it as: FlashInfer fused allreduce + rmsnorm operation. Notable calls include `flashinfer_comm.trtllm_allreduce_fusion`, `RuntimeError`, `allreduce_params.get_trtllm_fused_allreduce_kwargs`.
**CN:** `flashinfer_fused_allreduce_rmsnorm` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `flashinfer_comm.trtllm_allreduce_fusion`, `RuntimeError`, `allreduce_params.get_trtllm_fused_allreduce_kwargs`。

### Lines 202-241: Function `flashinfer_fused_allreduce_rmsnorm_fp8_quant` / 函数 `flashinfer_fused_allreduce_rmsnorm_fp8_quant`
```python
def flashinfer_fused_allreduce_rmsnorm_fp8_quant(
    input_tensor: torch.Tensor,
    residual: Optional[torch.Tensor],
    rms_gamma: torch.Tensor,
    rms_eps: float,
    scale_factor: torch.Tensor,
    allreduce_params: FlashInferFusedAllReduceParams,
    use_oneshot: bool = True,
    norm_out: Optional[torch.Tensor] = None,
    quant_out: Optional[torch.Tensor] = None,
):
    """FlashInfer fused allreduce + rmsnorm + FP8 quantization."""
    if flashinfer_comm is None or _FI_WORKSPACE_TENSOR is None:
        raise RuntimeError("FlashInfer not available or workspace not initialized")

    if norm_out is None:
        norm_out = input_tensor
        residual_out = residual
    else:
        residual_out = input_tensor

    flashinfer_comm.trtllm_allreduce_fusion(
        allreduce_in=input_tensor,
        token_num=input_tensor.shape[0],
        residual_in=residual,
        residual_out=residual_out,
        norm_out=norm_out,
        rms_gamma=rms_gamma,
        rms_eps=rms_eps,
        hidden_dim=input_tensor.shape[-1],
        workspace_ptrs=_FI_WORKSPACE_TENSOR,
        pattern_code=flashinfer_comm.AllReduceFusionPattern.kARResidualRMSNormFP8Quant,
        allreduce_out=None,
        quant_out=quant_out,
        scale_out=None,
        layout_code=None,
        scale_factor=scale_factor,
        use_oneshot=use_oneshot,
        **allreduce_params.get_trtllm_fused_allreduce_kwargs(),
    )
```
**EN:** `flashinfer_fused_allreduce_rmsnorm_fp8_quant` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. The docstring frames it as: FlashInfer fused allreduce + rmsnorm + FP8 quantization. Notable calls include `flashinfer_comm.trtllm_allreduce_fusion`, `RuntimeError`, `allreduce_params.get_trtllm_fused_allreduce_kwargs`.
**CN:** `flashinfer_fused_allreduce_rmsnorm_fp8_quant` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `flashinfer_comm.trtllm_allreduce_fusion`, `RuntimeError`, `allreduce_params.get_trtllm_fused_allreduce_kwargs`。

### Lines 244-284: Function `flashinfer_fused_allreduce_rmsnorm_fp4_quant` / 函数 `flashinfer_fused_allreduce_rmsnorm_fp4_quant`
```python
def flashinfer_fused_allreduce_rmsnorm_fp4_quant(
    input_tensor: torch.Tensor,
    residual: Optional[torch.Tensor],
    rms_gamma: torch.Tensor,
    rms_eps: float,
    input_global_scale: torch.Tensor,
    allreduce_params: FlashInferFusedAllReduceParams,
    quant_out: torch.Tensor,
    use_oneshot: bool,
    output_scale: torch.Tensor,
    norm_out: Optional[torch.Tensor] = None,
):
    """FlashInfer fused allreduce + rmsnorm + FP4 quantization."""
    if flashinfer_comm is None or _FI_WORKSPACE_TENSOR is None:
        raise RuntimeError("FlashInfer not available or workspace not initialized")

    if norm_out is None:
        norm_out = input_tensor
        residual_out = residual
    else:
        residual_out = input_tensor

    flashinfer_comm.trtllm_allreduce_fusion(
        allreduce_in=input_tensor,
        token_num=input_tensor.shape[0],
        residual_in=residual,
        residual_out=residual_out,
        norm_out=norm_out,
        rms_gamma=rms_gamma,
        rms_eps=rms_eps,
        hidden_dim=input_tensor.shape[-1],
        workspace_ptrs=_FI_WORKSPACE_TENSOR,
        pattern_code=flashinfer_comm.AllReduceFusionPattern.kARResidualRMSNormFP4Quant,
        allreduce_out=None,
        quant_out=quant_out,
        scale_out=output_scale,
        layout_code=None,
        scale_factor=input_global_scale,
        use_oneshot=use_oneshot,
        **allreduce_params.get_trtllm_fused_allreduce_kwargs(),
    )
```
**EN:** `flashinfer_fused_allreduce_rmsnorm_fp4_quant` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. The docstring frames it as: FlashInfer fused allreduce + rmsnorm + FP4 quantization. Notable calls include `flashinfer_comm.trtllm_allreduce_fusion`, `RuntimeError`, `allreduce_params.get_trtllm_fused_allreduce_kwargs`.
**CN:** `flashinfer_fused_allreduce_rmsnorm_fp4_quant` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `flashinfer_comm.trtllm_allreduce_fusion`, `RuntimeError`, `allreduce_params.get_trtllm_fused_allreduce_kwargs`。

### Lines 287-313: Function `standard_allreduce_rmsnorm` / 函数 `standard_allreduce_rmsnorm`
```python
def standard_allreduce_rmsnorm(
    input_tensor: torch.Tensor,
    residual: Optional[torch.Tensor],
    rms_gamma: torch.Tensor,
    rms_eps: float,
    norm_out: Optional[torch.Tensor] = None,
):
    """Standard allreduce + rmsnorm operations."""
    # All-reduce first
    allreduce_out = tensor_model_parallel_all_reduce(input_tensor)
    # Then RMS norm
    if residual is not None:
        # Fused add + RMS norm (in-place on allreduce_out)
        if SGL_FUSED_ADD_RMS_NORM is not None:
            SGL_FUSED_ADD_RMS_NORM(allreduce_out, residual, rms_gamma, rms_eps)
        else:
            rms = RMSNorm(allreduce_out.shape[-1], eps=rms_eps)
            rms.weight.data = rms_gamma
            rms.forward_native(allreduce_out, residual)
    else:
        # Just RMS norm
        if SGL_RMS_NORM is not None:
            _ = SGL_RMS_NORM(allreduce_out, rms_gamma, rms_eps)
        else:
            rms = RMSNorm(allreduce_out.shape[-1], eps=rms_eps)
            rms.weight.data = rms_gamma
            _ = rms.forward_native(allreduce_out)
```
**EN:** `standard_allreduce_rmsnorm` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: Standard allreduce + rmsnorm operations. Notable calls include `tensor_model_parallel_all_reduce`, `SGL_FUSED_ADD_RMS_NORM`, `RMSNorm`.
**CN:** `standard_allreduce_rmsnorm` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `tensor_model_parallel_all_reduce`, `SGL_FUSED_ADD_RMS_NORM`, `RMSNorm`。

### Lines 316-350: Function `standard_allreduce_rmsnorm_fp8_quant` / 函数 `standard_allreduce_rmsnorm_fp8_quant`
```python
def standard_allreduce_rmsnorm_fp8_quant(
    input_tensor: torch.Tensor,
    residual: Optional[torch.Tensor],
    rms_gamma: torch.Tensor,
    rms_eps: float,
    scale_factor: torch.Tensor,
    norm_out: Optional[torch.Tensor] = None,
    quant_out: Optional[torch.Tensor] = None,
):
    """Standard allreduce + rmsnorm + FP8 quantization."""
    # All-reduce first
    allreduce_out = tensor_model_parallel_all_reduce(input_tensor)

    # Then RMS norm + static FP8 quantization
    if residual is not None:
        if SGL_FUSED_ADD_RMS_NORM is not None:
            SGL_FUSED_ADD_RMS_NORM(allreduce_out, residual, rms_gamma, rms_eps)
            quant_out, _ = static_quant_fp8(
                allreduce_out, scale_factor, repeat_scale=False
            )
        else:
            rms = RMSNorm(allreduce_out.shape[-1], eps=rms_eps)
            rms.weight.data = rms_gamma
            normed, _ = rms.forward_native(allreduce_out, residual)
            quant_out, _ = static_quant_fp8(normed, scale_factor, repeat_scale=False)
        return quant_out, residual
    else:
        if SGL_RMS_NORM is not None:
            normed = SGL_RMS_NORM(allreduce_out, rms_gamma, rms_eps)
        else:
            rms = RMSNorm(allreduce_out.shape[-1], eps=rms_eps)
            rms.weight.data = rms_gamma
            normed = rms.forward_native(allreduce_out)
        quant_out, _ = static_quant_fp8(normed, scale_factor, repeat_scale=False)
        return quant_out
```
**EN:** `standard_allreduce_rmsnorm_fp8_quant` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: Standard allreduce + rmsnorm + FP8 quantization. It returns `(quant_out, residual)` to the caller. Notable calls include `tensor_model_parallel_all_reduce`, `static_quant_fp8`, `SGL_FUSED_ADD_RMS_NORM`.
**CN:** `standard_allreduce_rmsnorm_fp8_quant` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `(quant_out, residual)`。其中较关键的调用包括 `tensor_model_parallel_all_reduce`, `static_quant_fp8`, `SGL_FUSED_ADD_RMS_NORM`。

### Lines 353-394: Function `standard_allreduce_rmsnorm_fp4_quant` / 函数 `standard_allreduce_rmsnorm_fp4_quant`
```python
def standard_allreduce_rmsnorm_fp4_quant(
    input_tensor: torch.Tensor,
    residual: Optional[torch.Tensor],
    rms_gamma: torch.Tensor,
    rms_eps: float,
    input_global_scale: torch.Tensor,
    quant_out: torch.Tensor,
    output_scale: torch.Tensor,
    norm_out: Optional[torch.Tensor] = None,
):
    """Standard allreduce + rmsnorm + FP4 quantization."""

    # All-reduce first
    allreduce_out = tensor_model_parallel_all_reduce(input_tensor)

    # Then RMS norm
    if residual is not None:
        if SGL_FUSED_ADD_RMS_NORM is not None:
            SGL_FUSED_ADD_RMS_NORM(allreduce_out, residual, rms_gamma, rms_eps)
            quant_input = allreduce_out
        else:
            rms = RMSNorm(allreduce_out.shape[-1], eps=rms_eps)
            rms.weight.data = rms_gamma
            quant_input, _ = rms.forward_native(allreduce_out, residual)
        residual_out = residual
    else:
        if SGL_RMS_NORM is not None:
            quant_input = SGL_RMS_NORM(allreduce_out, rms_gamma, rms_eps)
        else:
            rms = RMSNorm(allreduce_out.shape[-1], eps=rms_eps)
            rms.weight.data = rms_gamma
            quant_input = rms.forward_native(allreduce_out)
        residual_out = allreduce_out

    # Finally FP4 quantization
    if SGL_SCALED_FP4_QUANT is None:
        raise RuntimeError("scaled_fp4_quant is not available on this platform")
    quant_res, output_scale_res = SGL_SCALED_FP4_QUANT(quant_input, input_global_scale)
    if residual is not None:
        return quant_res, residual_out, output_scale_res
    else:
        return quant_res, quant_input
```
**EN:** `standard_allreduce_rmsnorm_fp4_quant` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: Standard allreduce + rmsnorm + FP4 quantization. It returns `(quant_res, residual_out, output_scale_res)` to the caller. Notable calls include `tensor_model_parallel_all_reduce`, `SGL_SCALED_FP4_QUANT`, `RuntimeError`.
**CN:** `standard_allreduce_rmsnorm_fp4_quant` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `(quant_res, residual_out, output_scale_res)`。其中较关键的调用包括 `tensor_model_parallel_all_reduce`, `SGL_SCALED_FP4_QUANT`, `RuntimeError`。

### Lines 397-412: Function `standard_allreduce_rmsnorm_native` / 函数 `standard_allreduce_rmsnorm_native`
```python
def standard_allreduce_rmsnorm_native(
    input_tensor: torch.Tensor,
    residual: Optional[torch.Tensor],
    rmsnorm_layer: RMSNorm,
    norm_out: Optional[torch.Tensor] = None,
):
    """Standard allreduce + rmsnorm operations using native RMSNorm forward."""
    # All-reduce first
    allreduce_out = tensor_model_parallel_all_reduce(input_tensor)
    # Apply native RMSNorm
    if residual is not None:
        result = rmsnorm_layer.forward_native(allreduce_out, residual)
        return result  # Returns (norm_out, residual_out)
    else:
        result = rmsnorm_layer.forward_native(allreduce_out)
        return result  # Returns norm_out
```
**EN:** `standard_allreduce_rmsnorm_native` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: Standard allreduce + rmsnorm operations using native RMSNorm forward. It returns `result` to the caller. Notable calls include `tensor_model_parallel_all_reduce`, `rmsnorm_layer.forward_native`.
**CN:** `standard_allreduce_rmsnorm_native` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `result`。其中较关键的调用包括 `tensor_model_parallel_all_reduce`, `rmsnorm_layer.forward_native`。

### Lines 415-440: Function `standard_allreduce_rmsnorm_fp8_quant_native` / 函数 `standard_allreduce_rmsnorm_fp8_quant_native`
```python
def standard_allreduce_rmsnorm_fp8_quant_native(
    input_tensor: torch.Tensor,
    residual: Optional[torch.Tensor],
    rmsnorm_layer: RMSNorm,
    scale_factor: torch.Tensor,
    norm_out: Optional[torch.Tensor] = None,
    quant_out: Optional[torch.Tensor] = None,
):
    """Standard allreduce + rmsnorm + FP8 quantization using native implementations."""
    # All-reduce first
    allreduce_out = tensor_model_parallel_all_reduce(input_tensor)

    # Apply native RMSNorm
    if residual is not None:
        norm_out, residual_out = rmsnorm_layer.forward_native(allreduce_out, residual)
    else:
        norm_out = rmsnorm_layer.forward_native(allreduce_out)
        residual_out = allreduce_out

    # Apply native FP8 quantization
    quant_out, _ = static_quant_fp8(norm_out, scale_factor, repeat_scale=False)

    if residual is not None:
        return quant_out, residual_out
    else:
        return quant_out
```
**EN:** `standard_allreduce_rmsnorm_fp8_quant_native` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: Standard allreduce + rmsnorm + FP8 quantization using native implementations. It returns `(quant_out, residual_out)` to the caller. Notable calls include `tensor_model_parallel_all_reduce`, `static_quant_fp8`, `rmsnorm_layer.forward_native`.
**CN:** `standard_allreduce_rmsnorm_fp8_quant_native` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `(quant_out, residual_out)`。其中较关键的调用包括 `tensor_model_parallel_all_reduce`, `static_quant_fp8`, `rmsnorm_layer.forward_native`。

### Lines 443-473: Function `standard_allreduce_rmsnorm_fp4_quant_native` / 函数 `standard_allreduce_rmsnorm_fp4_quant_native`
```python
def standard_allreduce_rmsnorm_fp4_quant_native(
    input_tensor: torch.Tensor,
    residual: Optional[torch.Tensor],
    rmsnorm_layer: RMSNorm,
    input_global_scale: torch.Tensor,
    quant_out: torch.Tensor,
    output_scale: torch.Tensor,
    norm_out: Optional[torch.Tensor] = None,
):
    """Standard allreduce + rmsnorm + FP4 quantization using native RMSNorm."""
    # All-reduce first
    allreduce_out = tensor_model_parallel_all_reduce(input_tensor)

    # Apply native RMSNorm
    if residual is not None:
        norm_out, residual_out = rmsnorm_layer.forward_native(allreduce_out, residual)
        quant_input = norm_out
    else:
        norm_out = rmsnorm_layer.forward_native(allreduce_out)
        quant_input = norm_out
        residual_out = allreduce_out

    # Apply FP4 quantization (still using fused CUDA op as there's no native FP4)
    if SGL_SCALED_FP4_QUANT is None:
        raise RuntimeError("scaled_fp4_quant is not available on this platform")
    quant_res, output_scale_res = SGL_SCALED_FP4_QUANT(quant_input, input_global_scale)

    if residual is not None:
        return quant_res, residual_out, output_scale_res
    else:
        return quant_res, norm_out
```
**EN:** `standard_allreduce_rmsnorm_fp4_quant_native` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: Standard allreduce + rmsnorm + FP4 quantization using native RMSNorm. It returns `(quant_res, residual_out, output_scale_res)` to the caller. Notable calls include `tensor_model_parallel_all_reduce`, `SGL_SCALED_FP4_QUANT`, `rmsnorm_layer.forward_native`.
**CN:** `standard_allreduce_rmsnorm_fp4_quant_native` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `(quant_res, residual_out, output_scale_res)`。其中较关键的调用包括 `tensor_model_parallel_all_reduce`, `SGL_SCALED_FP4_QUANT`, `rmsnorm_layer.forward_native`。

### Lines 478-487: Function `standard_allreduce_rmsnorm_native_compiled` / 函数 `standard_allreduce_rmsnorm_native_compiled`
```python
def standard_allreduce_rmsnorm_native_compiled(
    input_tensor: torch.Tensor,
    residual: Optional[torch.Tensor],
    rmsnorm_layer: RMSNorm,
    norm_out: Optional[torch.Tensor] = None,
):
    """Compiled version of standard allreduce + rmsnorm."""
    return standard_allreduce_rmsnorm_native(
        input_tensor, residual, rmsnorm_layer, norm_out
    )
```
**EN:** `standard_allreduce_rmsnorm_native_compiled` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: Compiled version of standard allreduce + rmsnorm. It returns `standard_allreduce_rmsnorm_native(input_tensor, residual, rmsnorm_layer, norm_out)` to the caller. Notable calls include `standard_allreduce_rmsnorm_native`.
**CN:** `standard_allreduce_rmsnorm_native_compiled` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `standard_allreduce_rmsnorm_native(input_tensor, residual, rmsnorm_layer, norm_out)`。其中较关键的调用包括 `standard_allreduce_rmsnorm_native`。

### Lines 491-507: Function `standard_allreduce_rmsnorm_fp8_quant_native_compiled` / 函数 `standard_allreduce_rmsnorm_fp8_quant_native_compiled`
```python
def standard_allreduce_rmsnorm_fp8_quant_native_compiled(
    input_tensor: torch.Tensor,
    residual: Optional[torch.Tensor],
    rmsnorm_layer: RMSNorm,
    scale_factor: torch.Tensor,
    norm_out: Optional[torch.Tensor] = None,
    quant_out: Optional[torch.Tensor] = None,
):
    """Compiled version of standard allreduce + rmsnorm + FP8 quantization."""
    return standard_allreduce_rmsnorm_fp8_quant_native(
        input_tensor,
        residual,
        rmsnorm_layer,
        scale_factor,
        norm_out,
        quant_out,
    )
```
**EN:** `standard_allreduce_rmsnorm_fp8_quant_native_compiled` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: Compiled version of standard allreduce + rmsnorm + FP8 quantization. It returns `standard_allreduce_rmsnorm_fp8_quant_native(input_tensor, residual, rmsnorm_layer, scale_factor,...` to the caller. Notable calls include `standard_allreduce_rmsnorm_fp8_quant_native`.
**CN:** `standard_allreduce_rmsnorm_fp8_quant_native_compiled` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `standard_allreduce_rmsnorm_fp8_quant_native(input_tensor, residual, rmsnorm_layer, scale_factor,...`。其中较关键的调用包括 `standard_allreduce_rmsnorm_fp8_quant_native`。

### Lines 511-529: Function `standard_allreduce_rmsnorm_fp4_quant_native_compiled` / 函数 `standard_allreduce_rmsnorm_fp4_quant_native_compiled`
```python
def standard_allreduce_rmsnorm_fp4_quant_native_compiled(
    input_tensor: torch.Tensor,
    residual: Optional[torch.Tensor],
    rmsnorm_layer: RMSNorm,
    input_global_scale: torch.Tensor,
    quant_out: torch.Tensor,
    output_scale: torch.Tensor,
    norm_out: Optional[torch.Tensor] = None,
):
    """Compiled version of standard allreduce + rmsnorm + FP4 quantization."""
    return standard_allreduce_rmsnorm_fp4_quant_native(
        input_tensor,
        residual,
        rmsnorm_layer,
        input_global_scale,
        quant_out,
        output_scale,
        norm_out,
    )
```
**EN:** `standard_allreduce_rmsnorm_fp4_quant_native_compiled` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: Compiled version of standard allreduce + rmsnorm + FP4 quantization. It returns `standard_allreduce_rmsnorm_fp4_quant_native(input_tensor, residual, rmsnorm_layer, input_global_s...` to the caller. Notable calls include `standard_allreduce_rmsnorm_fp4_quant_native`.
**CN:** `standard_allreduce_rmsnorm_fp4_quant_native_compiled` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `standard_allreduce_rmsnorm_fp4_quant_native(input_tensor, residual, rmsnorm_layer, input_global_s...`。其中较关键的调用包括 `standard_allreduce_rmsnorm_fp4_quant_native`。

### Lines 532-563: Function `create_test_tensors` / 函数 `create_test_tensors`
```python
def create_test_tensors(
    seq_len: int, hidden_dim: int, dtype: torch.dtype, use_residual: bool = True
):
    """Create test tensors for benchmarking."""
    input_tensor = torch.randn(seq_len, hidden_dim, dtype=dtype)
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
    fp4_quant_out = torch.empty((seq_len, hidden_dim // 2), dtype=torch.uint8)
    fp4_output_scale = torch.empty((128, 4), dtype=torch.int32)

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
**EN:** `create_test_tensors` is a function that measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels. The docstring frames it as: Create test tensors for benchmarking. It returns `(input_tensor, norm_out, residual, rms_gamma, scale_fp8, quant_out_fp8, scale_fp4, fp4_quant_out,...` to the caller. Notable calls include `torch.randn`, `torch.ones`, `torch.tensor`.
**CN:** `create_test_tensors` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `(input_tensor, norm_out, residual, rms_gamma, scale_fp8, quant_out_fp8, scale_fp4, fp4_quant_out,...`。其中较关键的调用包括 `torch.randn`, `torch.ones`, `torch.tensor`。

### Lines 566-602: Function `benchmark_operation` / 函数 `benchmark_operation`
```python
def benchmark_operation(
    operation_func, *args, warmup: int = 5, trials: int = 20, **kwargs
):
    """Benchmark a single operation using CUDA graphs."""
    # Warmup before graph capture
    for _ in range(warmup):
        operation_func(*args, **kwargs)
    torch.cuda.synchronize()

    # Create CUDA graph
    graph = torch.cuda.CUDAGraph()
    num_op_per_cudagraph = 10

    # Use sglang's graph_capture to make tensor_model_parallel_all_reduce graph-safe
    with graph_capture() as graph_capture_context:
        with torch.cuda.graph(graph, stream=graph_capture_context.stream):
            for _ in range(num_op_per_cudagraph):
                operation_func(*args, **kwargs)

    # Graph warmup
    torch.cuda.synchronize()
    for _ in range(warmup):
        graph.replay()

    # Benchmark with CUDA graph
    torch.cuda.synchronize()
    start_time = time.perf_counter()

    for _ in range(trials // num_op_per_cudagraph):
        # operation_func(*args, **kwargs)
        graph.replay()

    torch.cuda.synchronize()
    end_time = time.perf_counter()

    avg_time_ms = ((end_time - start_time) / trials) * 1000
    return avg_time_ms
```
**EN:** `benchmark_operation` is a function that measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels. The docstring frames it as: Benchmark a single operation using CUDA graphs. It returns `avg_time_ms` to the caller. Notable calls include `range`, `torch.cuda.synchronize`, `torch.cuda.CUDAGraph`.
**CN:** `benchmark_operation` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `avg_time_ms`。其中较关键的调用包括 `range`, `torch.cuda.synchronize`, `torch.cuda.CUDAGraph`。

### Lines 605-891: Function `run_benchmarks` / 函数 `run_benchmarks`
```python
def run_benchmarks(
    seq_len: int,
    hidden_dim: int,
    dtype: torch.dtype,
    use_residual: bool,
    allreduce_params: Optional[FlashInferFusedAllReduceParams],
    quant_mode: str = "all",
    disable_oneshot: bool = False,
):
    """Run all benchmarks for given configuration.

    Args:
        quant_mode: "none", "fp8_only", "fp4_only", or "all"
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
    ) = create_test_tensors(seq_len, hidden_dim, dtype, use_residual)

    rms_eps = 1e-6
    results = {}

    # Create RMSNorm once for native benchmarks
    rmsnorm_layer = RMSNorm(hidden_dim, eps=rms_eps)
    rmsnorm_layer.weight.data = rms_gamma

    if quant_mode in ["all", "none"]:
        # Standard AllReduce + RMSNorm
        try:
            time_ms = benchmark_operation(
                standard_allreduce_rmsnorm,
                input_tensor,
                norm_out=norm_out,
                residual=residual,
                rms_gamma=rms_gamma,
                rms_eps=rms_eps,
            )
            results["standard_allreduce_rmsnorm"] = time_ms
        except Exception as e:
            logger.error("Standard AllReduce+RMSNorm failed: %s", e)
            results["standard_allreduce_rmsnorm"] = float("inf")

        # Standard AllReduce + RMSNorm Native Compiled
        try:
            time_ms = benchmark_operation(
                standard_allreduce_rmsnorm_native_compiled,
                input_tensor,
                residual=residual,
                rmsnorm_layer=rmsnorm_layer,
                norm_out=norm_out,
            )
            results["standard_allreduce_rmsnorm_native_compiled"] = time_ms
        except Exception as e:
            logger.error("Standard AllReduce+RMSNorm Native Compiled failed: %s", e)
            results["standard_allreduce_rmsnorm_native_compiled"] = float("inf")

        # FlashInfer Fused AllReduce + RMSNorm Oneshot
        if flashinfer_comm is not None and allreduce_params is not None:
            try:
                if not disable_oneshot:
                    time_ms = benchmark_operation(
                        flashinfer_fused_allreduce_rmsnorm,
                        input_tensor,
                        residual=residual,
                        norm_out=norm_out,
                        rms_gamma=rms_gamma,
                        rms_eps=rms_eps,
                        allreduce_params=allreduce_params,
                        use_oneshot=True,
                    )
                    results["flashinfer_fused_allreduce_rmsnorm_oneshot"] = time_ms
            except Exception as e:
                logger.error("FlashInfer Fused AllReduce+RMSNorm Oneshot failed: %s", e)
                results["flashinfer_fused_allreduce_rmsnorm_oneshot"] = float("inf")

            # FlashInfer Fused AllReduce + RMSNorm Two-shot
            try:
                time_ms = benchmark_operation(
                    flashinfer_fused_allreduce_rmsnorm,
                    input_tensor,
                    residual=residual,
                    norm_out=norm_out,
                    rms_gamma=rms_gamma,
                    rms_eps=rms_eps,
                    allreduce_params=allreduce_params,
                    use_oneshot=False,
                )
                results["flashinfer_fused_allreduce_rmsnorm_twoshot"] = time_ms
            except Exception as e:
                logger.error(
                    "FlashInfer Fused AllReduce+RMSNorm Two-shot failed: %s", e
                )
                results["flashinfer_fused_allreduce_rmsnorm_twoshot"] = float("inf")

    if quant_mode in ["all", "fp8_only"]:
        # Standard AllReduce + RMSNorm + FP8 Quant
        try:
            time_ms = benchmark_operation(
                standard_allreduce_rmsnorm_fp8_quant,
                input_tensor,
                norm_out=norm_out,
                residual=residual,
                rms_gamma=rms_gamma,
                rms_eps=rms_eps,
                scale_factor=scale_fp8,
                quant_out=quant_out_fp8,
            )
            results["standard_allreduce_rmsnorm_fp8_quant"] = time_ms
        except Exception as e:
            logger.error("Standard AllReduce+RMSNorm+FP8 failed: %s", e)
            results["standard_allreduce_rmsnorm_fp8_quant"] = float("inf")

        # Standard AllReduce + RMSNorm + FP8 Quant Native Compiled
        try:
            time_ms = benchmark_operation(
                standard_allreduce_rmsnorm_fp8_quant_native_compiled,
                input_tensor,
                residual=residual,
                rmsnorm_layer=rmsnorm_layer,
                # quant_fp8_layer removed in sglang version; static_quant_fp8 is used within the function
                scale_factor=scale_fp8,
                norm_out=norm_out,
                quant_out=quant_out_fp8,
            )
            results["standard_allreduce_rmsnorm_fp8_quant_native_compiled"] = time_ms
        except Exception as e:
            logger.error("Standard AllReduce+RMSNorm+FP8 Native Compiled failed: %s", e)
            results["standard_allreduce_rmsnorm_fp8_quant_native_compiled"] = float(
                "inf"
            )

        # FlashInfer Fused AllReduce + RMSNorm + FP8 Quant Oneshot
        if flashinfer_comm is not None and allreduce_params is not None:
            try:
                if not disable_oneshot:
                    time_ms = benchmark_operation(
                        flashinfer_fused_allreduce_rmsnorm_fp8_quant,
                        input_tensor,
                        norm_out=norm_out,
                        residual=residual,
                        rms_gamma=rms_gamma,
                        rms_eps=rms_eps,
                        scale_factor=scale_fp8,
                        quant_out=quant_out_fp8,
                        allreduce_params=allreduce_params,
                        use_oneshot=True,
                    )
                    results["flashinfer_fused_allreduce_rmsnorm_fp8_quant_oneshot"] = (
                        time_ms
                    )
            except Exception as e:
                logger.error(
                    "FlashInfer Fused AllReduce+RMSNorm+FP8 Oneshot failed: %s",
                    e,
                )
                results["flashinfer_fused_allreduce_rmsnorm_fp8_quant_oneshot"] = float(
                    "inf"
                )
            # FlashInfer Fused AllReduce + RMSNorm + FP8 Quant Two-shot
            try:
                time_ms = benchmark_operation(
                    flashinfer_fused_allreduce_rmsnorm_fp8_quant,
                    input_tensor,
                    norm_out=norm_out,
                    residual=residual,
                    rms_gamma=rms_gamma,
                    rms_eps=rms_eps,
                    scale_factor=scale_fp8,
                    quant_out=quant_out_fp8,
                    allreduce_params=allreduce_params,
                    use_oneshot=False,
                )
                results["flashinfer_fused_allreduce_rmsnorm_fp8_quant_twoshot"] = (
                    time_ms
                )
            except Exception as e:
                logger.error(
                    "FlashInfer Fused AllReduce+RMSNorm+FP8 Two-shot failed: %s",
                    e,
                )
                results["flashinfer_fused_allreduce_rmsnorm_fp8_quant_twoshot"] = float(
                    "inf"
                )

    if quant_mode in ["all", "fp4_only"]:
        # Standard AllReduce + RMSNorm + FP4 Quant
        try:
            time_ms = benchmark_operation(
                standard_allreduce_rmsnorm_fp4_quant,
                input_tensor,
                norm_out=norm_out,
                residual=residual,
                rms_gamma=rms_gamma,
                rms_eps=rms_eps,
                input_global_scale=scale_fp4,
                quant_out=fp4_quant_out,
                output_scale=fp4_output_scale,
            )
            results["standard_allreduce_rmsnorm_fp4_quant"] = time_ms
        except Exception as e:
            logger.error("Standard AllReduce+RMSNorm+FP4 failed: %s", e)
            results["standard_allreduce_rmsnorm_fp4_quant"] = float("inf")

        # Standard AllReduce + RMSNorm + FP4 Quant Native Compiled
        try:
            time_ms = benchmark_operation(
                standard_allreduce_rmsnorm_fp4_quant_native_compiled,
                input_tensor,
                residual=residual,
                rmsnorm_layer=rmsnorm_layer,
                input_global_scale=scale_fp4,
                quant_out=fp4_quant_out,
                output_scale=fp4_output_scale,
                norm_out=norm_out,
            )
            results["standard_allreduce_rmsnorm_fp4_quant_native_compiled"] = time_ms
        except Exception as e:
            logger.error("Standard AllReduce+RMSNorm+FP4 Native Compiled failed: %s", e)
            results["standard_allreduce_rmsnorm_fp4_quant_native_compiled"] = float(
                "inf"
            )

        # FlashInfer Fused AllReduce + RMSNorm + FP4 Quant Oneshot
        if flashinfer_comm is not None and allreduce_params is not None:
            try:
                if not disable_oneshot:
                    time_ms = benchmark_operation(
                        flashinfer_fused_allreduce_rmsnorm_fp4_quant,
                        input_tensor,
                        residual=residual,
                        norm_out=norm_out,
                        rms_gamma=rms_gamma,
                        rms_eps=rms_eps,
                        input_global_scale=scale_fp4,
                        allreduce_params=allreduce_params,
                        quant_out=fp4_quant_out,
                        output_scale=fp4_output_scale,
                        use_oneshot=True,
                    )
                    results["flashinfer_fused_allreduce_rmsnorm_fp4_quant_oneshot"] = (
                        time_ms
                    )
            except Exception as e:
                logger.error(
                    "FlashInfer Fused AllReduce+RMSNorm+FP4 Oneshot failed: %s",
                    e,
                )
                results["flashinfer_fused_allreduce_rmsnorm_fp4_quant_oneshot"] = float(
                    "inf"
                )

        # FlashInfer Fused AllReduce + RMSNorm + FP4 Quant Two-shot
        if flashinfer_comm is not None and allreduce_params is not None:
            try:
                time_ms = benchmark_operation(
                    flashinfer_fused_allreduce_rmsnorm_fp4_quant,
                    input_tensor,
                    residual=residual,
                    norm_out=norm_out,
                    rms_gamma=rms_gamma,
                    rms_eps=rms_eps,
                    input_global_scale=scale_fp4,
                    allreduce_params=allreduce_params,
                    quant_out=fp4_quant_out,
                    output_scale=fp4_output_scale,
                    use_oneshot=False,
                )
                results["flashinfer_fused_allreduce_rmsnorm_fp4_quant_twoshot"] = (
                    time_ms
                )
            except Exception as e:
                logger.error(
                    "FlashInfer Fused AllReduce+RMSNorm+FP4 Two-shot failed: %s",
                    e,
                )
                results["flashinfer_fused_allreduce_rmsnorm_fp4_quant_twoshot"] = float(
                    "inf"
                )

    return results
```
**EN:** `run_benchmarks` is a function that measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels. The docstring frames it as: Run all benchmarks for given configuration. It returns `results` to the caller. Notable calls include `create_test_tensors`, `RMSNorm`, `benchmark_operation`.
**CN:** `run_benchmarks` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `results`。其中较关键的调用包括 `create_test_tensors`, `RMSNorm`, `benchmark_operation`。

### Lines 894-990: Function `prepare_results_with_speedups` / 函数 `prepare_results_with_speedups`
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
                fastest_baseline = candidate

        return fastest_baseline

    # Create dynamic baseline mapping
    dynamic_baseline_mapping = {}
    for op_name in results_dict:
        if (
            op_name.startswith("flashinfer_")
            or op_name.startswith("standard_")
            and not op_name.endswith("_native_compiled")
        ):
            dynamic_baseline_mapping[op_name] = get_fastest_baseline(
                op_name, results_dict
            )

    for op_name, time_ms in results_dict.items():
        if time_ms == float("inf"):
            speedup_str = "FAILED"
            time_str = "FAILED"
        else:
            time_str = f"{time_ms:.3f}"
            # Find the appropriate baseline for this operation
            baseline_op = dynamic_baseline_mapping.get(op_name)
            if baseline_op and baseline_op in results_dict:
                baseline_time = results_dict[baseline_op]
                if baseline_time != float("inf") and baseline_time > 0:
                    speedup = baseline_time / time_ms
                    speedup_str = f"{speedup:.2f}x"
                else:
                    speedup_str = "N/A"
            else:
                # For baseline operations, determine if this is the fastest baseline
                if op_name.endswith("_native_compiled") or (
                    op_name.startswith("standard_")
                    and not op_name.endswith("_native_compiled")
                ):
                    fastest_baseline = get_fastest_baseline(op_name, results_dict)
                    if fastest_baseline == op_name:
                        speedup_str = "baseline"
                    else:
                        if fastest_baseline and fastest_baseline in results_dict:
                            baseline_time = results_dict[fastest_baseline]
                            if baseline_time != float("inf") and baseline_time > 0:
                                speedup = baseline_time / time_ms
                                speedup_str = f"{speedup:.2f}x"
                            else:
                                speedup_str = "N/A"
                        else:
                            speedup_str = "N/A"
                else:
                    speedup_str = "N/A"

        prepared_results.append(
            {
                "operation": op_name,
                "time_ms": time_ms,
                "time_str": time_str,
                "speedup_str": speedup_str,
            }
        )

    return prepared_results
```
**EN:** `prepare_results_with_speedups` is a function that prepares tensors and invokes GPU kernels. The docstring frames it as: Prepare results with speedup calculations based on dynamic baseline selection. It returns `prepared_results` to the caller. Notable calls include `results_dict.items`, `float`, `prepared_results.append`.
**CN:** `prepare_results_with_speedups` 是一个函数，用于准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `prepared_results`。其中较关键的调用包括 `results_dict.items`, `float`, `prepared_results.append`。

### Lines 993-1016: Function `print_results` / 函数 `print_results`
```python
def print_results(results_dict, seq_len, hidden_dim, dtype, use_residual, quant_mode):
    """Print benchmark results in a formatted table."""
    print(f"\n{'=' * 80}")
    print(f"Results: seq_len={seq_len}, hidden_dim={hidden_dim}")
    print(
        f"dtype={dtype}, residual={'yes' if use_residual else 'no'}, "
        f"quant_mode={quant_mode}"
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
**EN:** `print_results` is a function that measures runtime latency, throughput, or other benchmark metrics. The docstring frames it as: Print benchmark results in a formatted table. Notable calls include `print`, `prepare_results_with_speedups`, `float`.
**CN:** `print_results` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `print`, `prepare_results_with_speedups`, `float`。

### Lines 1019-1061: Function `format_results_markdown` / 函数 `format_results_markdown`
```python
def format_results_markdown(
    all_results: list[dict], world_size: int, args: argparse.Namespace
) -> str:
    """Format all benchmark results as markdown."""
    markdown = f"""# FlashInfer Fused Collective Operations Benchmark Results

**World Size:** {world_size}
**Hidden Dimension:** {args.hidden_dim}
**Warmup Iterations:** {args.warmup}
**Benchmark Trials:** {args.trials}
**Quantization Mode:** {all_results[0]["quant_mode"] if all_results else "N/A"}

---

"""

    for result in all_results:
        seq_len = result["seq_len"]
        dtype = result["dtype"]
        use_residual = result["use_residual"]
        results_dict = result["results"]

        residual_str = "with residual" if use_residual else "no residual"

        markdown += f"""
## Configuration: seq_len={seq_len}, dtype={dtype}, {residual_str}

| Operation | Time (ms) | Speedup |
|-----------|-----------|---------|
"""

        # Prepare results with speedup calculations
        prepared_results = prepare_results_with_speedups(results_dict)

        for result in prepared_results:
            # Format operation name for better readability
            formatted_op_name = result["operation"].replace("_", " ").title()
            markdown += f"| {formatted_op_name} | {result['time_str']} |"
            markdown += f"{result['speedup_str']} |\n"

        markdown += "\n"

    return markdown
```
**EN:** `format_results_markdown` is a function that builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. The docstring frames it as: Format all benchmark results as markdown. It returns `markdown` to the caller. Notable calls include `prepare_results_with_speedups`, `result['operation'].replace('_', ' ').title`, `result['operation'].replace`.
**CN:** `format_results_markdown` 是一个函数，用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `markdown`。其中较关键的调用包括 `prepare_results_with_speedups`, `result['operation'].replace('_', ' ').title`, `result['operation'].replace`。

### Lines 1064-1084: Function `save_results_to_file` / 函数 `save_results_to_file`
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

        with open(output_path, "w") as f:
            f.write(markdown_content)

    except Exception as e:
        logger.error("Failed to save results to file: %s", e)
```
**EN:** `save_results_to_file` is a function that builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and reads or writes local files and intermediate artifacts. The docstring frames it as: Save benchmark results to markdown file (only on rank 0). Notable calls include `logger.warning`, `format_results_markdown`, `open`.
**CN:** `save_results_to_file` 是一个函数，用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、读写本地文件及中间产物。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `logger.warning`, `format_results_markdown`, `open`。

### Lines 1087-1301: Function `main` / 函数 `main`
```python
def main():
    parser = argparse.ArgumentParser(
        description="Benchmark fused collective operations"
    )
    parser.add_argument(
        "--seq-lens",
        type=int,
        nargs="+",
        default=[128, 512, 1024, 2048],
        help="Sequence lengths to test",
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

    # Quantization mode options (mutually exclusive with --no-quant)
    quant_group = parser.add_mutually_exclusive_group()
    quant_group.add_argument(
        "--no-quant", action="store_true", help="Skip all quantization tests"
    )
    quant_group.add_argument(
        "--quant-fp8", action="store_true", help="Only run FP8 quantization tests"
    )
    quant_group.add_argument(
        "--quant-fp4", action="store_true", help="Only run FP4 quantization tests"
    )
    quant_group.add_argument(
        "--quant-all",
        action="store_true",
        help="Run all quantization tests (default)",
    )

    parser.add_argument(
        "--disable-oneshot",
        action="store_true",
        help="Disable oneshot mode for FlashInfer operations",
    )
    parser.add_argument(
        "--warmup", type=int, default=5, help="Number of warmup iterations"
    )
    parser.add_argument(
        "--trials", type=int, default=20, help="Number of benchmark trials"
    )
    parser.add_argument(
        "--output-file",
        type=str,
        help="""Output file path for markdown results
                (default: benchmark_results_<timestamp>.md)
        """,
    )

    args = parser.parse_args()

    # Check if running with torchrun (required for collective operations)
    if "RANK" not in os.environ or "WORLD_SIZE" not in os.environ:
        raise RuntimeError(
            "Must run with torchrun for distributed benchmarking. "
            "Example: torchrun --nproc_per_node=2 benchmark_fused_collective.py"
        )

    # Initialize distributed environment
    rank = int(os.environ["RANK"])
    world_size = int(os.environ["WORLD_SIZE"])

    device = torch.device(f"cuda:{rank}")
    torch.cuda.set_device(device)
    torch.set_default_device(device)

    init_distributed_environment(
        world_size=world_size,
        rank=rank,
        local_rank=rank,
        backend="nccl",
    )
    initialize_model_parallel(tensor_model_parallel_size=world_size)

    # Validate world size (must be > 1 for collective operations)
    if world_size <= 1:
        raise ValueError(
            "World size must be > 1 for collective operations benchmarking. "
            f"Current world size: {world_size}. Use torchrun with --nproc_per_node > 1."
        )

    # Determine quantization mode
    if args.no_quant:
        quant_mode = "none"
    elif args.quant_fp8:
        quant_mode = "fp8_only"
    elif args.quant_fp4:
        quant_mode = "fp4_only"
    else:  # args.quant_all or default
        quant_mode = "all"

    if rank == 0:
        logger.info("Running benchmark with world_size=%s, rank=%s", world_size, rank)
        logger.info("Quantization mode: %s", quant_mode)
        if flashinfer_comm is not None:
            oneshot_status = "enabled" if not args.disable_oneshot else "disabled"
            logger.info(
                "FlashInfer available - will benchmark fused operations (oneshot: %s)",
                oneshot_status,
            )
        else:
            logger.info(
                "FlashInfer not available - only benchmarking standard operations"
            )

    # Convert dtype strings to torch dtypes
    dtype_map = {
        "float16": torch.float16,
        "bfloat16": torch.bfloat16,
        "float32": torch.float32,
    }
    dtypes = [dtype_map[dt] for dt in args.dtypes]

    # Test configurations
    residual_options = [True] if not args.no_residual else [False]
    if not args.no_residual:
        residual_options.append(False)

    configs = list(itertools.product(args.seq_lens, dtypes, residual_options))

    # Setup FlashInfer workspace if available
    ipc_handles = None
    allreduce_params = None

    if flashinfer_comm is not None:
        # Use the largest hidden dimension for workspace setup
        max_num_token = _FI_MAX_SIZES.get(world_size) // (
            args.hidden_dim * world_size * 2
        )

        ipc_handles, workspace_tensor = setup_flashinfer_workspace(
            world_size, rank, args.hidden_dim, max_num_token
        )

        if workspace_tensor is not None:
            allreduce_params = FlashInferFusedAllReduceParams(
                rank=rank,
                world_size=world_size,
                max_token_num=max_num_token,
            )

    # Collect all results for markdown export
    all_results = []

    try:
        # Run benchmarks
        for seq_len, dtype, use_residual in configs:
            if rank == 0:
                logger.info(
                    "\nTesting:  seq_len=%s, hidden_dim=%s, dtype=%s, residual=%s",
                    seq_len,
                    args.hidden_dim,
                    dtype,
                    use_residual,
                )

            results = run_benchmarks(
                seq_len,
                args.hidden_dim,
                dtype,
                use_residual,
                allreduce_params,
                quant_mode=quant_mode,
                disable_oneshot=args.disable_oneshot,
            )

            # Store results for markdown export
            if rank == 0:
                all_results.append(
                    {
                        "seq_len": seq_len,
                        "hidden_dim": args.hidden_dim,
                        "dtype": str(dtype).replace("torch.", ""),
                        "use_residual": use_residual,
                        "quant_mode": quant_mode,
                        "results": results,
                    }
                )

                print_results(
                    results,
                    seq_len,
                    args.hidden_dim,
                    dtype,
                    use_residual,
                    quant_mode,
                )

        # Save results to markdown file
        if args.output_file and rank == 0:
            save_results_to_file(all_results, world_size, args, rank)

    finally:
        # Cleanup
        if ipc_handles is not None:
            cleanup_flashinfer_workspace(ipc_handles)

        with contextlib.suppress(Exception):
            dist.barrier()
        cleanup_dist_env_and_memory(shutdown_ray=False)
```
**EN:** `main` is a function that builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. Notable calls include `argparse.ArgumentParser`, `parser.add_argument`, `parser.add_mutually_exclusive_group`.
**CN:** `main` 是一个函数，用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。其中较关键的调用包括 `argparse.ArgumentParser`, `parser.add_argument`, `parser.add_mutually_exclusive_group`。

### Lines 1304-1305: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    main()
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `contextlib`, `itertools`, `logging`, `os`, `time`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`, `sgl_kernel`, `flashinfer.comm`
- **Internal / 项目内部依赖**: `sglang.srt.distributed`, `sglang.srt.distributed.parallel_state`, `sglang.srt.layers.layernorm`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.jit_kernel.nvfp4`
