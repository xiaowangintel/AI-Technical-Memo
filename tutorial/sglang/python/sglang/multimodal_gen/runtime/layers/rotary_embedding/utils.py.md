# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/rotary_embedding/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `_apply_rotary_emb`, `apply_flashinfer_rope_qk_inplace`, and `_warn_about_missing_flashinfer`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Primitive RoPE ops: rotate helpers and apply_rotary_emb utilities. / 该文件属于运行时算子层。它围绕 `_apply_rotary_emb`、`apply_flashinfer_rope_qk_inplace` 和 `_warn_about_missing_flashinfer` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: module setup and imports / 模块初始化与导入
```python
"""Primitive RoPE ops: rotate helpers and apply_rotary_emb utilities."""

from typing import Optional, Tuple

import torch

from sglang.jit_kernel.diffusion.triton.rotary import apply_rotary_embedding
from sglang.kernel_api_logging import debug_kernel_api
from sglang.multimodal_gen.runtime.platforms import current_platform
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.srt.utils.custom_op import register_custom_op_from_extern
```
**EN:** This block establishes the module context and imports `typing`, `torch`, `sglang.jit_kernel.diffusion.triton.rotary`, `sglang.kernel_api_logging`, `sglang.multimodal_gen.runtime.platforms`, and `sglang.multimodal_gen.runtime.utils.logging_utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `typing`、`torch`、`sglang.jit_kernel.diffusion.triton.rotary`、`sglang.kernel_api_logging`、`sglang.multimodal_gen.runtime.platforms` 和 `sglang.multimodal_gen.runtime.utils.logging_utils`。这些依赖为后续实现提供所需符号。

### Lines 13-33: supporting statements / 辅助语句
```python
logger = init_logger(__name__)

_is_cuda = current_platform.is_cuda()
if _is_cuda:
    try:
        from flashinfer.rope import (
            apply_rope_with_cos_sin_cache_inplace as _flashinfer_apply_rope_inplace,
        )
    except Exception:
        _flashinfer_apply_rope_inplace = None
else:
    _flashinfer_apply_rope_inplace = None

if _flashinfer_apply_rope_inplace is not None:
    flashinfer_apply_rope_inplace = register_custom_op_from_extern(
        _flashinfer_apply_rope_inplace,
        op_name="flashinfer_apply_rope_with_cos_sin_cache_inplace",
        mutates_args=["query", "key"],
    )
else:
    flashinfer_apply_rope_inplace = None
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, and `_is_cuda`. The code collaborates with `init_logger`, `current_platform.is_cuda`, and `register_custom_op_from_extern`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 和 `_is_cuda` 等名称。 代码会与 `init_logger`、`current_platform.is_cuda` 和 `register_custom_op_from_extern` 协同工作。

### Lines 36-65: `_apply_rotary_emb` implementation / `_apply_rotary_emb` 实现
```python
def _apply_rotary_emb(
    x: torch.Tensor,
    cos: torch.Tensor,
    sin: torch.Tensor,
    is_neox_style: bool,
    interleaved: bool = False,
) -> torch.Tensor:
    """
    Args:
        x: [num_tokens, num_heads, head_size] or [num_tokens, head_size]
        cos: [num_tokens, head_size // 2]
        sin: [num_tokens, head_size // 2]
        is_neox_style: Whether to use the Neox-style or GPT-J-style rotary
            positional embeddings.
    """
    # cos = cos.unsqueeze(-2).to(x.dtype)
    # sin = sin.unsqueeze(-2).to(x.dtype)
    if is_neox_style:
        cos = cos.unsqueeze(-2)
        sin = sin.unsqueeze(-2)
        if is_neox_style:
            x1, x2 = torch.chunk(x, 2, dim=-1)
        else:
            x1 = x[..., ::2]
            x2 = x[..., 1::2]
        o1 = (x1.float() * cos - x2.float() * sin).type_as(x)
        o2 = (x2.float() * cos + x1.float() * sin).type_as(x)
        return torch.cat((o1, o2), dim=-1)
    else:
        return apply_rotary_embedding(x, cos, sin, interleaved)
```
**EN:** This block defines function `_apply_rotary_emb`. Args: x: [num_tokens, num_heads, head_size] or [num_tokens, head_size] cos: [num_tokens, head_size // 2] sin: [num_tokens, head_size // 2] is_neox_style: Whether to use the Neox-style or GPT-J-style rotary positional embeddings. Key calls include `cos.unsqueeze`, `sin.unsqueeze`, `type_as`, `torch.cat`, and `apply_rotary_embedding`. The implementation branches on conditions. Parameters such as `x`, `cos`, `sin`, `is_neox_style`, and `interleaved` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_apply_rotary_emb`。 它用于应用rotary emb。 关键调用包括 `cos.unsqueeze`、`sin.unsqueeze`、`type_as`、`torch.cat` 和 `apply_rotary_embedding`。 实现中包含条件分支。 本段逻辑主要由 `x`、`cos`、`sin`、`is_neox_style` 和 `interleaved` 等参数驱动。

### Lines 68-143: `apply_flashinfer_rope_qk_inplace` implementation / `apply_flashinfer_rope_qk_inplace` 实现
```python
@debug_kernel_api
def apply_flashinfer_rope_qk_inplace(
    q: torch.Tensor,
    k: torch.Tensor,
    cos_sin_cache: torch.Tensor,
    *,
    head_size: Optional[int] = None,
    is_neox: bool = False,
    positions: Optional[torch.Tensor] = None,
) -> Tuple[torch.Tensor, torch.Tensor]:
    if q.dim() != 4 or k.dim() != 4:
        raise ValueError(
            f"Expected q/k to be 4D [bsz, seqlen, nheads, head_size], "
            f"got q:{tuple(q.shape)} k:{tuple(k.shape)}"
        )
    if q.shape != k.shape:
        raise ValueError(
            f"q and k must have the same shape, got {q.shape} vs {k.shape}"
        )

    if not (isinstance(cos_sin_cache, torch.Tensor) and cos_sin_cache.dim() == 2):
        raise ValueError("cos_sin_cache must be a 2D torch.Tensor")

    bsz, seqlen, nheads, d = q.shape
    if head_size is None:
        head_size = d
    if head_size != d:
        raise ValueError(f"head_size mismatch: inferred {d}, but head_size={head_size}")

    if flashinfer_apply_rope_inplace is None:
        # Triton fallback for AMD/ROCm where FlashInfer is not available

        _warn_about_missing_flashinfer()

        half_size = cos_sin_cache.shape[-1] // 2
        if positions is None:
            cos = cos_sin_cache[:seqlen, :half_size].to(q.dtype)
            sin = cos_sin_cache[:seqlen, half_size:].to(q.dtype)
            cos = cos.unsqueeze(0).expand(bsz, -1, -1).reshape(bsz * seqlen, -1)
            sin = sin.unsqueeze(0).expand(bsz, -1, -1).reshape(bsz * seqlen, -1)
        else:
            positions = positions.to(cos_sin_cache.device).view(-1)
            cos = cos_sin_cache[positions, :half_size].to(q.dtype)
            sin = cos_sin_cache[positions, half_size:].to(q.dtype)
        q_flat = q.reshape(bsz * seqlen, nheads, d)
        k_flat = k.reshape(bsz * seqlen, nheads, d)
        q_rot = apply_rotary_embedding(q_flat, cos, sin, interleaved=not is_neox)
        k_rot = apply_rotary_embedding(k_flat, cos, sin, interleaved=not is_neox)
        return q_rot.view(bsz, seqlen, nheads, d), k_rot.view(bsz, seqlen, nheads, d)

    if positions is None:
        pos_1d = torch.arange(seqlen, device=q.device, dtype=torch.long)
        positions = pos_1d if bsz == 1 else pos_1d.repeat(bsz)
    else:
        if not (
            isinstance(positions, torch.Tensor)
            and positions.dtype == torch.long
            and positions.dim() == 1
        ):
            raise ValueError("positions must be a 1D torch.long Tensor")
        if positions.numel() != bsz * seqlen:
            raise ValueError(
                f"positions length must be bsz*seqlen={bsz*seqlen}, got {positions.numel()}"
            )

    q_flat = q.reshape(bsz * seqlen, nheads * d).contiguous()
    k_flat = k.reshape(bsz * seqlen, nheads * d).contiguous()
    flashinfer_apply_rope_inplace(
        positions=positions,
        query=q_flat,
        key=k_flat,
        head_size=d,
        cos_sin_cache=cos_sin_cache,
        is_neox=is_neox,
    )
    return q_flat.view(bsz, seqlen, nheads, d), k_flat.view(bsz, seqlen, nheads, d)
```
**EN:** This block defines function `apply_flashinfer_rope_qk_inplace`. It applies flashinfer rope qk inplace. Key calls include `q.reshape.contiguous`, `k.reshape.contiguous`, `flashinfer_apply_rope_inplace`, `ValueError`, and `_warn_about_missing_flashinfer`. The implementation branches on conditions. Parameters such as `q`, `k`, and `cos_sin_cache` drive the behavior in this section.
**CN:** 该代码块定义了函数 `apply_flashinfer_rope_qk_inplace`。 它用于应用flashinfer rope qk inplace。 关键调用包括 `q.reshape.contiguous`、`k.reshape.contiguous`、`flashinfer_apply_rope_inplace`、`ValueError` 和 `_warn_about_missing_flashinfer`。 实现中包含条件分支。 本段逻辑主要由 `q`、`k` 和 `cos_sin_cache` 等参数驱动。

### Lines 146-154: `_warn_about_missing_flashinfer` implementation / `_warn_about_missing_flashinfer` 实现
```python
@torch.compiler.assume_constant_result
def _warn_about_missing_flashinfer():
    """
    Function to warn about the missing FlashInfer.
    Exists to not cause a graph break during the compilation.
    """
    logger.warning_once(
        "FlashInfer not available, using Triton fallback for RoPE",
    )
```
**EN:** This block defines function `_warn_about_missing_flashinfer`. Function to warn about the missing FlashInfer. Exists to not cause a graph break during the compilation. Key calls include `logger.warning_once`.
**CN:** 该代码块定义了函数 `_warn_about_missing_flashinfer`。 它用于处理 warn about missing flashinfer 相关逻辑。 关键调用包括 `logger.warning_once`。

## Key Concepts / 关键概念
- `_apply_rotary_emb`: Args: x: [num_tokens, num_heads, head_size] or [num_tokens, head_size] cos: [num_tokens, head_size // 2] sin: [num_tokens, head_size // 2] is_neox_style: Whether to use the Neox-style or GPT-J-style rotary positional embeddings. / 顶层函数，用于应用rotary emb。
- `apply_flashinfer_rope_qk_inplace`: Top-level function that applies flashinfer rope qk inplace. / 顶层函数，用于应用flashinfer rope qk inplace。
- `_warn_about_missing_flashinfer`: Function to warn about the missing FlashInfer. / 顶层函数，用于处理 warn about missing flashinfer 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方依赖**: `torch`, `flashinfer.rope`
- **Internal modules / 内部模块**: `sglang.jit_kernel.diffusion.triton.rotary`, `sglang.kernel_api_logging`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.srt.utils.custom_op`

- **Total lines / 总行数**: 154
