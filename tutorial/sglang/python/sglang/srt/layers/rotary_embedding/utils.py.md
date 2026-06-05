# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/rotary_embedding/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements rotary position embedding logic with embedding lookup and preprocessing for the SGLang SRT runtime. It exposes symbols such as `rotate_neox`, `rotate_gptj`, `apply_rotary_emb`, and `rotate_half` and connects them to backend-specific paths such as `NPU` and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了旋转位置编码逻辑，并结合嵌入查找与预处理。它提供了 `rotate_neox`、`rotate_gptj`、`apply_rotary_emb` 以及 `rotate_half` 等符号，并把这些符号连接到 `NPU` 和 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Imports, conditional backend setup, and runtime guards
```python
"""Primitive rotary embedding ops: _rotate_neox, _rotate_gptj, _apply_rotary_emb,
apply_rotary_pos_emb variants."""

from __future__ import annotations

from typing import Tuple

import torch

from sglang.srt.utils import cpu_has_amx_support, get_compiler_backend, is_cpu, is_npu

_is_npu = is_npu()
_is_cpu = is_cpu()
_is_cpu_amx_available = cpu_has_amx_support()

if _is_npu:
    import torch_npu

    NPU_ROTARY_MUL_MAX_NUM_HEADS = 1000
    NPU_ROTARY_MUL_MAX_HEAD_SIZE = 896
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `typing.Tuple`, `torch`, `sglang.srt.utils.cpu_has_amx_support`, `sglang.srt.utils.get_compiler_backend`, and `sglang.srt.utils.is_cpu`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_npu`, `_is_cpu`, `_is_cpu_amx_available`, `NPU_ROTARY_MUL_MAX_NUM_HEADS`, and `NPU_ROTARY_MUL_MAX_HEAD_SIZE` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`typing.Tuple`、`torch`、`sglang.srt.utils.cpu_has_amx_support`、`sglang.srt.utils.get_compiler_backend` 以及 `sglang.srt.utils.is_cpu`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_npu`、`_is_cpu`、`_is_cpu_amx_available`、`NPU_ROTARY_MUL_MAX_NUM_HEADS` 以及 `NPU_ROTARY_MUL_MAX_HEAD_SIZE` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 23-28: Function `rotate_neox` and its core logic
```python
def rotate_neox(x: torch.Tensor) -> torch.Tensor:
    x1 = x[..., : x.shape[-1] // 2]
    x2 = x[..., x.shape[-1] // 2 :]
    return torch.cat((-x2, x1), dim=-1)
```
**EN:** This block defines `rotate_neox` and contains the main logic for this step. It mainly invokes `torch.cat`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `x1` and `x2` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `rotate_neox`，并承载这一阶段的核心逻辑。 它主要调用 `torch.cat`，说明该流程会编排底层辅助函数或计算内核。 像 `x1` 和 `x2` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 29-35: Function `rotate_gptj` and its core logic
```python
def rotate_gptj(x: torch.Tensor) -> torch.Tensor:
    x1 = x[..., ::2]
    x2 = x[..., 1::2]
    x = torch.stack((-x2, x1), dim=-1)
    return x.flatten(-2)
```
**EN:** This block defines `rotate_gptj` and contains the main logic for this step. It mainly invokes `torch.stack` and `x.flatten`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `x1`, `x2`, and `x` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `rotate_gptj`，并承载这一阶段的核心逻辑。 它主要调用 `torch.stack` 和 `x.flatten`，说明该流程会编排底层辅助函数或计算内核。 像 `x1`、`x2` 以及 `x` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 36-65: `apply_rotary_emb` apply step for rotary embedding emb
```python
def apply_rotary_emb(
    x: torch.Tensor,
    cos: torch.Tensor,
    sin: torch.Tensor,
    is_neox_style: bool,
) -> torch.Tensor:
    """
    Args:
        x: [num_tokens, num_heads, head_size]
        cos: [num_tokens, head_size // 2]
        sin: [num_tokens, head_size // 2]
        is_neox_style: Whether to use the Neox-style or GPT-J-style rotary
            positional embeddings.
    """
    cos = cos.unsqueeze(-2).to(x.dtype)
    sin = sin.unsqueeze(-2).to(x.dtype)
    if is_neox_style:
        x1, x2 = torch.chunk(x, 2, dim=-1)
    else:
        x1 = x[..., ::2]
        x2 = x[..., 1::2]
    o1 = x1 * cos - x2 * sin
    o2 = x2 * cos + x1 * sin
    if is_neox_style:
        return torch.cat((o1, o2), dim=-1)
    else:
        return torch.stack((o1, o2), dim=-1).flatten(-2)


# Copied from transformers
```
**EN:** This block defines `apply_rotary_emb` and contains the main logic for this step. It mainly invokes `cos.unsqueeze.to`, `sin.unsqueeze.to`, `torch.chunk`, `torch.cat`, and `torch.stack.flatten`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `cos`, `sin`, `o1`, `o2`, and `x1` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `apply_rotary_emb`，并承载这一阶段的核心逻辑。 它主要调用 `cos.unsqueeze.to`、`sin.unsqueeze.to`、`torch.chunk`、`torch.cat` 以及 `torch.stack.flatten`，说明该流程会编排底层辅助函数或计算内核。 像 `cos`、`sin`、`o1`、`o2` 以及 `x1` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 66-72: Function `rotate_half` and its core logic
```python
def rotate_half(x):
    """Rotates half the hidden dims of the input."""
    x1 = x[..., : x.shape[-1] // 2]
    x2 = x[..., x.shape[-1] // 2 :]
    return torch.cat((-x2, x1), dim=-1)
```
**EN:** This block defines `rotate_half` and contains the main logic for this step. It mainly invokes `torch.cat`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `x1` and `x2` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `rotate_half`，并承载这一阶段的核心逻辑。 它主要调用 `torch.cat`，说明该流程会编排底层辅助函数或计算内核。 像 `x1` 和 `x2` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 73-96: `apply_rotary_pos_emb_native` apply step for rotary embedding pos emb native
```python
@torch.compile(dynamic=True, backend=get_compiler_backend())
def apply_rotary_pos_emb_native(
    q: torch.Tensor,
    k: torch.Tensor,
    cos: torch.Tensor,
    sin: torch.Tensor,
    unsqueeze_dim=1,
) -> Tuple[torch.Tensor, torch.Tensor]:
    orig_q_dtype = q.dtype
    orig_k_dtype = k.dtype
    q, k = q.float(), k.float()

    # embedding is performed in float
    cos = cos.unsqueeze(unsqueeze_dim).float()
    sin = sin.unsqueeze(unsqueeze_dim).float()
    q_embed = (q * cos) + (rotate_half(q) * sin)
    k_embed = (k * cos) + (rotate_half(k) * sin)

    q_embed = q_embed.to(orig_q_dtype)
    k_embed = k_embed.to(orig_k_dtype)

    return q_embed, k_embed
```
**EN:** This block defines `apply_rotary_pos_emb_native` and contains the main logic for this step. Decorators like `torch.compile` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `torch.compile`, `cos.unsqueeze.float`, `sin.unsqueeze.float`, `q_embed.to`, and `k_embed.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `orig_q_dtype`, `orig_k_dtype`, `q`, `k`, and `cos` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `apply_rotary_pos_emb_native`，并承载这一阶段的核心逻辑。 像 `torch.compile` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `torch.compile`、`cos.unsqueeze.float`、`sin.unsqueeze.float`、`q_embed.to` 以及 `k_embed.to`，说明该流程会编排底层辅助函数或计算内核。 像 `orig_q_dtype`、`orig_k_dtype`、`q`、`k` 以及 `cos` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 97-130: `apply_rotary_pos_emb_npu` apply step for rotary embedding pos emb NPU
```python
def apply_rotary_pos_emb_npu(
    q: torch.Tensor,
    k: torch.Tensor,
    cos: torch.Tensor,
    sin: torch.Tensor,
    unsqueeze_dim=1,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """Ascend implementation equivalent to apply_rotary_pos_emb_native.

    Args:
        q: [num_tokens, num_heads, head_size]
        k: [num_tokens, num_kv_heads, head_size]
        cos: [num_tokens, head_size]
        sin: [num_tokens, head_size]
    """
    if (
        cos.dim() != 2
        or q.dim() != 3
        or q.shape[1] >= NPU_ROTARY_MUL_MAX_NUM_HEADS
        or q.shape[2] >= NPU_ROTARY_MUL_MAX_HEAD_SIZE
    ):
        # Note: num_heads and head_size of q must be less than 1000 and 896, respectively
        return apply_rotary_pos_emb_native(q, k, cos, sin, unsqueeze_dim)
    cos = cos.unsqueeze(unsqueeze_dim).unsqueeze(0)
    sin = sin.unsqueeze(unsqueeze_dim).unsqueeze(0)
    q = q.unsqueeze(0)
    k = k.unsqueeze(0)
    q_embed = torch_npu.npu_rotary_mul(q, cos, sin)
    k_embed = torch_npu.npu_rotary_mul(k, cos, sin)
    q_embed = q_embed.squeeze(0)
    k_embed = k_embed.squeeze(0)
    return q_embed, k_embed
```
**EN:** This block defines `apply_rotary_pos_emb_npu` and contains the main logic for this step. It mainly invokes `cos.unsqueeze.unsqueeze`, `sin.unsqueeze.unsqueeze`, `q.unsqueeze`, `k.unsqueeze`, and `torch_npu.npu_rotary_mul`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `cos`, `sin`, `q`, `k`, and `q_embed` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `apply_rotary_pos_emb_npu`，并承载这一阶段的核心逻辑。 它主要调用 `cos.unsqueeze.unsqueeze`、`sin.unsqueeze.unsqueeze`、`q.unsqueeze`、`k.unsqueeze` 以及 `torch_npu.npu_rotary_mul`，说明该流程会编排底层辅助函数或计算内核。 像 `cos`、`sin`、`q`、`k` 以及 `q_embed` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 131-136: Module-level helper logic
```python
if _is_npu:
    apply_rotary_pos_emb = apply_rotary_pos_emb_npu
elif _is_cpu and _is_cpu_amx_available:
    apply_rotary_pos_emb = torch.ops.sgl_kernel.apply_rotary_pos_emb_cpu
else:
    apply_rotary_pos_emb = apply_rotary_pos_emb_native
```
**EN:** This section prepares the module namespace. Shared names such as `apply_rotary_pos_emb` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 像 `apply_rotary_pos_emb` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `rotate_neox`, `rotate_gptj`, `apply_rotary_emb`, `rotate_half`, and `apply_rotary_pos_emb_native`. / **主要符号**：核心入口包括 `rotate_neox`、`rotate_gptj`、`apply_rotary_emb`、`rotate_half` 以及 `apply_rotary_pos_emb_native`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。
- **Embedding pipeline**: Describes how IDs or features are mapped into model-space tensors. / **嵌入流水线**：说明 ID 或特征如何映射到模型空间张量。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations` and `typing.Tuple` / **标准库**：`__future__.annotations` 和 `typing.Tuple`
- **Third-party**: `torch` and `torch_npu` / **第三方依赖**：`torch` 和 `torch_npu`
- **Internal SGLang modules**: `sglang.srt.utils.cpu_has_amx_support`, `sglang.srt.utils.get_compiler_backend`, `sglang.srt.utils.is_cpu`, and `sglang.srt.utils.is_npu` / **SGLang 内部模块**：`sglang.srt.utils.cpu_has_amx_support`、`sglang.srt.utils.get_compiler_backend`、`sglang.srt.utils.is_cpu` 以及 `sglang.srt.utils.is_npu`
