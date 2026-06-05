# _impl.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/ops/_impl.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Provides public ONNX export APIs and compatibility helpers for impl.
- 用途 (CN): 为 impl 提供公开的 ONNX 导出 API 与兼容性辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
"""Implementations of ONNX operators as native Torch ops.

NOTE: Fake implementations:
    Refer to https://docs.pytorch.org/docs/stable/library.html#torch.library.register_fake
    for more details on how to create fake kernels.
"""

```
- EN: This block advances ONNX export translation; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 8-14
```python
import math
from collections.abc import Callable
from typing import TypeVar
from typing_extensions import ParamSpec

import torch
from torch.onnx.ops import _dtype_mappings
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.onnx.ops`; external imports: `math`, `collections.abc`, `typing`, `typing_extensions`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.onnx.ops`；外部导入：`math`, `collections.abc`, `typing`, `typing_extensions`。

### Lines 15-28
```python


# Use ParamSpec for better type preservation instead of bound Callable TypeVar
_P = ParamSpec("_P")
_R = TypeVar("_R")

# ONNX to ATen decomp table
ONNX_ATEN_DECOMP_TABLE: dict[torch._ops.OpOverload, Callable] = {}
_ATTENTION_23_ALLOWED_INTERMEDIATE_PRECISIONS = frozenset(
    {
        1,  # FLOAT
        10,  # FLOAT16
        11,  # DOUBLE
        16,  # BFLOAT16
```
- EN: This block advances ONNX export translation; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 29-37
```python
    }
)


def _onnx_op(
    op_type: str, opset_version: int, fake_impl: Callable[_P, _R]
) -> Callable[[Callable[_P, _R]], Callable[_P, _R]]:
    """Decorator to register an ONNX operator with a custom implementation."""

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_onnx_op`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_onnx_op`。

### Lines 38-47
```python
    def decorator(func: Callable[_P, _R]) -> Callable[_P, _R]:
        overload = f"opset{opset_version}"
        torch_op = torch.library.custom_op(
            f"onnx::{op_type}.{overload}", mutates_args=()
        )(func)
        ONNX_ATEN_DECOMP_TABLE[getattr(getattr(torch.ops.onnx, op_type), overload)] = (
            func  # type: ignore[assignment]
        )
        torch_op.register_fake(fake_impl)
        return torch_op  # type: ignore[return-value]
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `decorator`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`decorator`。

### Lines 48-61
```python

    return decorator


def _rotary_embedding_23_fake_impl(
    x: torch.Tensor,
    cos_cache: torch.Tensor,
    sin_cache: torch.Tensor,
    position_ids: torch.Tensor | None = None,
    *,
    interleaved: bool = False,
    num_heads: int = 0,
    rotary_embedding_dim: int = 0,
) -> torch.Tensor:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_rotary_embedding_23_fake_impl`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_rotary_embedding_23_fake_impl`。

### Lines 62-75
```python
    """Fake implementation for RotaryEmbedding-23 for torch.compile purposes."""
    return x.clone()


@_onnx_op("RotaryEmbedding", 23, _rotary_embedding_23_fake_impl)
def rotary_embedding_23(
    x: torch.Tensor,
    cos_cache: torch.Tensor,
    sin_cache: torch.Tensor,
    position_ids: torch.Tensor | None = None,
    *,
    interleaved: bool = False,
    num_heads: int = 0,
    rotary_embedding_dim: int = 0,
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `rotary_embedding_23`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`rotary_embedding_23`。

### Lines 76-84
```python
) -> torch.Tensor:
    """RotaryEmbedding-23 https://onnx.ai/onnx/operators/onnx__RotaryEmbedding.html#rotaryembedding-23"""
    # x has shape (batch_size, num_heads, sequence_length, head_size)
    # or (batch_size, sequence_length, hidden_size)
    input_shape = x.shape
    input_rank = len(input_shape)
    batch_size = input_shape[0]
    sequence_length = input_shape[-2]

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 85-98
```python
    # Validate position_ids and caches match x
    if position_ids is not None:
        torch._check(
            position_ids.dim() == 2,
            lambda: f"position_ids must be 2D when provided. Received shape {position_ids.shape}",
        )
        torch._check(
            position_ids.shape[0] == batch_size,
            lambda: f"position_ids first dim (batch) must match x.shape[0] ({batch_size}). Received {position_ids.shape[0]}",
        )
        torch._check(
            position_ids.shape[1] == sequence_length,
            lambda: f"position_ids second dim (sequence) must match x.shape[-2] ({sequence_length}). Received {position_ids.shape[1]}",
        )
```
- EN: This block checks invariants or expected outcomes; reuses computed state to reduce repeated work; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；复用已计算状态以减少重复工作；处理条件控制流。关键符号：无明显局部符号。

### Lines 99-110
```python
        torch._check(
            cos_cache.dim() == 2 and sin_cache.dim() == 2,
            lambda: "cos_cache/sin_cache must be 2D when position_ids is provided. "
            f"Received cos_cache shape {cos_cache.shape}, sin_cache shape {sin_cache.shape}",
        )
    else:
        torch._check(
            cos_cache.dim() == 3 and sin_cache.dim() == 3,
            lambda: "cos_cache/sin_cache must be 3D when position_ids is not provided. "
            f"Received cos_cache shape {cos_cache.shape}, sin_cache shape {sin_cache.shape}",
        )

```
- EN: This block checks invariants or expected outcomes; reuses computed state to reduce repeated work; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；复用已计算状态以减少重复工作；处理条件控制流。关键符号：无明显局部符号。

### Lines 111-124
```python
    # First ensure x has shape [batch_size, num_heads, seq_len, head_size]
    # So that the rotation logic can be shared with reshaped 3D inputs
    if input_rank == 4:
        # Reshape from (batch_size, num_heads, seq_len, head_size)
        # to [batch_size, seq_len, num_heads, head_size]
        x = torch.permute(x, (0, 2, 1, 3))
    elif input_rank == 3:
        torch._check(
            num_heads != 0,
            lambda: f"num_heads must be provided for 3D inputs. Received input tensor with shape {input_shape}",
        )
        hidden_size = input_shape[2]
        head_size = hidden_size // num_heads
        new_shape = [batch_size, sequence_length, num_heads, head_size]
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 125-131
```python
        x = torch.reshape(x, new_shape)

    torch._check(len(x.shape) == 4, lambda: "x should be a 4D tensor by now")
    head_size = x.shape[3]

    # Fully or partially perform rotation on x based on rotary_embedding_dim attribute
    if rotary_embedding_dim == 0:
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 132-139
```python
        # If rotary_embedding_dim not provided, perform full rotation by using head_size
        rotary_embedding_dim = head_size
    x_rotate = x[:, :, :, :rotary_embedding_dim]
    x_not_rotate = x[:, :, :, rotary_embedding_dim:]
    rotary_embedding_dim_half = rotary_embedding_dim // 2

    # Retrieve sin and cos caches using position ids
    if position_ids is not None:
```
- EN: This block reuses computed state to reduce repeated work; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块复用已计算状态以减少重复工作；处理条件控制流。关键符号：无明显局部符号。

### Lines 140-149
```python
        cos = cos_cache[
            position_ids
        ]  # Shape: [batch_size, sequence_length, head_size/2]
        sin = sin_cache[
            position_ids
        ]  # Shape: [batch_size, sequence_length, head_size/2]
    else:
        cos = cos_cache  # Shape: [batch_size, sequence_length, rotary_embedding_dim/2]
        sin = sin_cache  # Shape: [batch_size, sequence_length, rotary_embedding_dim/2]

```
- EN: This block reuses computed state to reduce repeated work; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块复用已计算状态以减少重复工作；处理条件控制流。关键符号：无明显局部符号。

### Lines 150-163
```python
    torch._check(
        cos.shape[0] == batch_size and cos.shape[1] == sequence_length,
        lambda: f"cos has shape {cos.shape} but expected (batch={batch_size}, seq={sequence_length}, ...)",
    )
    torch._check(
        sin.shape[0] == batch_size and sin.shape[1] == sequence_length,
        lambda: f"sin has shape {sin.shape} but expected (batch={batch_size}, seq={sequence_length}, ...)",
    )
    torch._check(
        cos.shape[-1] == rotary_embedding_dim_half,
        lambda: f"Last dimension of cos cache ({cos.shape[-1]}) should match rotary_embedding_dim/2 ({rotary_embedding_dim_half}).",
    )
    torch._check(
        sin.shape[-1] == rotary_embedding_dim_half,
```
- EN: This block checks invariants or expected outcomes; reuses computed state to reduce repeated work. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；复用已计算状态以减少重复工作。关键符号：无明显局部符号。

### Lines 164-172
```python
        lambda: f"Last dimension of sin cache ({sin.shape[-1]}) should match rotary_embedding_dim/2 ({rotary_embedding_dim_half}).",
    )
    cos = torch.unsqueeze(
        cos, 2
    )  # Shape: [batch_size, sequence_length, 1, rotary_embedding_dim/2]
    sin = torch.unsqueeze(
        sin, 2
    )  # Shape: [batch_size, sequence_length, 1, rotary_embedding_dim/2]

```
- EN: This block reuses computed state to reduce repeated work. Key symbols: no dominant local symbols.
- CN: 该代码块复用已计算状态以减少重复工作。关键符号：无明显局部符号。

### Lines 173-179
```python
    # Either divide the x in halves or interleave (based on interleaved attribute)
    if interleaved:
        x1 = x_rotate[:, :, :, 0::2]
        x2 = x_rotate[:, :, :, 1::2]
    else:
        x1, x2 = torch.chunk(x_rotate, 2, dim=-1)

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 180-193
```python
    # Calculate real and imaginary values
    real = cos * x1 - sin * x2
    imag = sin * x1 + cos * x2

    # Inserted rotated embeddings back to the original x
    if interleaved:
        # x_rotate[:, :, :, 0::2] = real
        # x_rotate[:, :, :, 1::2] = imag
        real = torch.unsqueeze(real, -1)
        imag = torch.unsqueeze(imag, -1)
        x_rotate_concat = torch.cat((real, imag), dim=-1)
        x_rotate = torch.reshape(x_rotate_concat, x_rotate.shape)
    else:
        x_rotate = torch.cat((real, imag), dim=-1)
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 194-200
```python
    output = torch.cat((x_rotate, x_not_rotate), dim=-1)
    if input_rank == 3:
        return torch.reshape(output, input_shape)

    # Return the dimensions to the original order
    return torch.permute(output, (0, 2, 1, 3))

```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 201-207
```python

def _get_scale_factor(scale: float | None, head_size: int) -> float:
    """Get the scale factor for attention computation."""
    return scale if scale is not None else (1.0 / math.sqrt(head_size))


def _reshape_3d_to_4d(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_get_scale_factor`, `_reshape_3d_to_4d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_get_scale_factor`, `_reshape_3d_to_4d`。

### Lines 208-218
```python
    tensor: torch.Tensor, batch_size: int, num_heads: int
) -> torch.Tensor:
    """Reshape 3D tensor to 4D for multi-head attention."""
    sequence_length, hidden_size = tensor.shape[1], tensor.shape[2]
    head_size = hidden_size // num_heads
    return (
        tensor.view(batch_size, sequence_length, num_heads, head_size)
        .transpose(1, 2)
        .contiguous()
    )

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 219-229
```python

def _get_qk_output_for_aten_spda(
    Q: torch.Tensor,
    K: torch.Tensor,
    current_q_num_heads: int,
    current_kv_num_heads: int,
    scale: float | None,
    qk_matmul_output_mode: int,
) -> torch.Tensor:
    """Get QK output tensor based on the specified mode."""
    if qk_matmul_output_mode == 0:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_get_qk_output_for_aten_spda`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_get_qk_output_for_aten_spda`。

### Lines 230-236
```python
        return _compute_qk_output_for_mode_0(
            Q, K, current_q_num_heads, current_kv_num_heads, scale
        )
    else:
        # For other modes, return a zero tensor with correct shape
        return torch.zeros_like(torch.matmul(Q, K.transpose(-2, -1)))

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 237-246
```python

def _validate_gqa_configuration(
    current_q_num_heads: int, current_kv_num_heads: int
) -> None:
    """Validate Group Query Attention configuration."""
    torch._check(
        current_q_num_heads % current_kv_num_heads == 0,
        lambda: f"q_num_heads ({current_q_num_heads}) must be divisible by kv_num_heads ({current_kv_num_heads}) for GQA",
    )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_validate_gqa_configuration`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_validate_gqa_configuration`。

### Lines 247-258
```python

def _compute_qk_output_for_mode_0(
    Q: torch.Tensor,
    K: torch.Tensor,
    current_q_num_heads: int,
    current_kv_num_heads: int,
    scale: float | None,
) -> torch.Tensor:
    """Helper function to compute QK output for qk_matmul_output_mode == 0."""
    # Handle GQA manually for QK output
    K_for_qk = K
    if current_q_num_heads != current_kv_num_heads:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_compute_qk_output_for_mode_0`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_compute_qk_output_for_mode_0`。

### Lines 259-267
```python
        repeat_factor = current_q_num_heads // current_kv_num_heads
        K_for_qk = K.repeat_interleave(repeat_factor, dim=1)

    scale_factor = _get_scale_factor(scale, Q.shape[3])
    # Scale both Q and K by sqrt(scale_factor) for numerical stability
    sqrt_scale = math.sqrt(scale_factor)
    Q_scaled = Q * sqrt_scale
    K_scaled = K_for_qk * sqrt_scale
    return torch.matmul(Q_scaled, K_scaled.transpose(-2, -1))
```
- EN: This block iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 268-281
```python


def _attention_23_fake_impl(
    Q: torch.Tensor,
    K: torch.Tensor,
    V: torch.Tensor,
    attn_mask: torch.Tensor | None = None,
    past_key: torch.Tensor | None = None,
    past_value: torch.Tensor | None = None,
    *,
    is_causal: bool = False,
    kv_num_heads: int = 0,
    q_num_heads: int = 0,
    qk_matmul_output_mode: int = 0,
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_attention_23_fake_impl`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_attention_23_fake_impl`。

### Lines 282-288
```python
    scale: float | None = None,
    softcap: float = 0.0,
    softmax_precision: int | None = None,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    """Fake implementation for Attention-23 for torch.compile purposes."""
    batch_size = Q.shape[0]

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 289-296
```python
    # Handle 3D vs 4D input shapes
    if len(Q.shape) == 3:
        # 3D input: (batch_size, sequence_length, hidden_size)
        q_sequence_length = Q.shape[1]
        output_shape = Q.shape  # Same shape as Q for 3D output

        # For present_key and present_value, we need 4D shapes
        if past_key is not None:
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 297-310
```python
            present_key_shape = (
                batch_size,
                kv_num_heads,
                past_key.shape[2] + K.shape[1],  # Combined sequence length
                K.shape[2] // kv_num_heads,  # head_size
            )
        else:
            present_key_shape = (
                batch_size,
                kv_num_heads,
                K.shape[1],  # sequence_length
                K.shape[2] // kv_num_heads,  # head_size
            )
        present_value_shape = present_key_shape  # Same shape as present_key
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 311-324
```python

        # QK output shape for 3D input (reshaped to 4D internally)
        qk_output_shape = (
            batch_size,
            q_num_heads,
            q_sequence_length,
            present_key_shape[2],  # kv_sequence_length
        )
    else:
        # 4D input: (batch_size, num_heads, sequence_length, head_size)
        q_sequence_length = Q.shape[2]
        # Same shape as Q for 4D output
        output_shape = Q.shape  # type: ignore[assignment]

```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 325-336
```python
        # Handle past key/value concatenation
        if past_key is not None:
            present_key_shape = (
                K.shape[0],  # batch_size
                K.shape[1],  # num_heads
                past_key.shape[2] + K.shape[2],  # Combined sequence length
                K.shape[3],  # head_size
            )
        else:
            present_key_shape = K.shape  # type: ignore[assignment]
        present_value_shape = present_key_shape  # Same shape as present_key

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 337-344
```python
        # QK output shape
        qk_output_shape = (
            Q.shape[0],  # batch_size
            Q.shape[1],  # q_num_heads
            Q.shape[2],  # q_sequence_length
            present_key_shape[2],  # kv_sequence_length
        )

```
- EN: This block implements local helper logic for impl. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 impl 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 345-351
```python
    # Create fake tensors with correct shapes and dtypes
    output = torch.empty(output_shape, dtype=Q.dtype, device=Q.device)
    present_key = torch.empty(present_key_shape, dtype=K.dtype, device=K.device)
    present_value = torch.empty(present_value_shape, dtype=V.dtype, device=V.device)
    qk_output = torch.empty(qk_output_shape, dtype=Q.dtype, device=Q.device)

    return output, present_key, present_value, qk_output
```
- EN: This block handles tensor metadata or sample values; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 352-365
```python


@_onnx_op("Attention", 23, _attention_23_fake_impl)
def attention_23(
    Q: torch.Tensor,
    K: torch.Tensor,
    V: torch.Tensor,
    attn_mask: torch.Tensor | None = None,
    past_key: torch.Tensor | None = None,
    past_value: torch.Tensor | None = None,
    *,
    is_causal: bool = False,
    kv_num_heads: int = 0,
    q_num_heads: int = 0,
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `attention_23`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`attention_23`。

### Lines 366-372
```python
    qk_matmul_output_mode: int = 0,
    scale: float | None = None,
    softcap: float = 0.0,
    softmax_precision: int | None = None,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    """Attention-23 https://onnx.ai/onnx/operators/onnx__Attention.html#attention-23"""

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 373-380
```python
    num_head_dim, sequence_dim, head_dim = 1, 2, 3

    # Store original input shape to determine output shape
    input_shape_len = len(Q.shape)
    batch_size = Q.shape[0]

    # Reshape 3D inputs to 4D format
    if len(Q.shape) == 3:
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 381-389
```python
        torch._check(
            q_num_heads != 0 and kv_num_heads != 0,
            lambda: "q_num_heads and kv_num_heads must be provided for 3D inputs",
        )
        q_sequence_length = Q.shape[1]
        Q = _reshape_3d_to_4d(Q, batch_size, q_num_heads)
        K = _reshape_3d_to_4d(K, batch_size, kv_num_heads)
        V = _reshape_3d_to_4d(V, batch_size, kv_num_heads)

```
- EN: This block checks invariants or expected outcomes; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 390-398
```python
    torch._check(
        len(Q.shape) == 4 and len(K.shape) == 4 and len(V.shape) == 4,
        lambda: "Q, K, and V should be 4D tensors by now",
    )

    # Calculate scale factor if not provided
    q_head_size = Q.shape[head_dim]
    scale = _get_scale_factor(scale, q_head_size)

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 399-407
```python
    # Handle past key/value caches
    present_key = (
        torch.cat([past_key, K], dim=sequence_dim)
        if past_key is not None
        else K.clone()
    )
    present_value = (
        torch.cat([past_value, V], dim=sequence_dim)
        if past_value is not None
```
- EN: This block reuses computed state to reduce repeated work; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块复用已计算状态以减少重复工作；处理条件控制流。关键符号：无明显局部符号。

### Lines 408-419
```python
        else V.clone()
    )

    # Update K and V to include past states
    K, V = present_key, present_value

    # Get current dimensions
    current_q_num_heads = Q.shape[num_head_dim]
    current_kv_num_heads = K.shape[num_head_dim]
    q_sequence_length = Q.shape[sequence_dim]
    kv_sequence_length = K.shape[sequence_dim]

```
- EN: This block implements local helper logic for impl. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 impl 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 420-427
```python
    # Check if we can use the optimized scaled_dot_product_attention (most optimized)
    can_use_sdpa = (
        softcap == 0.0  # No softcap
        and qk_matmul_output_mode == 0  # Default QK output mode
        and softmax_precision is None  # No custom softmax precision
        and (attn_mask is None or attn_mask.dtype == torch.bool)
    )

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 428-441
```python
    _validate_gqa_configuration(current_q_num_heads, current_kv_num_heads)

    if can_use_sdpa:
        # Use PyTorch's optimized scaled_dot_product_attention
        output = torch.nn.functional.scaled_dot_product_attention(
            Q,
            K,
            V,
            attn_mask=attn_mask,
            dropout_p=0.0,
            is_causal=is_causal,
            scale=scale,
            enable_gqa=bool(
                current_q_num_heads != current_kv_num_heads
```
- EN: This block checks invariants or expected outcomes; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理条件控制流。关键符号：无明显局部符号。

### Lines 442-455
```python
            ),  # Ensure enable_gqa is not SymBool
        )

        qk_output = _get_qk_output_for_aten_spda(
            Q,
            K,
            current_q_num_heads,
            current_kv_num_heads,
            scale,
            qk_matmul_output_mode,
        )
    else:
        # Fallback to manual implementation for complex cases

```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 456-466
```python
        # Handle Group Query Attention (GQA) and Multi-Query Attention (MQA)
        if current_q_num_heads != current_kv_num_heads:
            repeat_factor = current_q_num_heads // current_kv_num_heads
            K = K.repeat_interleave(repeat_factor, dim=num_head_dim)
            V = V.repeat_interleave(repeat_factor, dim=num_head_dim)

        # Create attention bias
        attn_bias = torch.zeros(
            q_sequence_length, kv_sequence_length, dtype=Q.dtype, device=Q.device
        )

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 467-480
```python
        # Apply causal masking
        if is_causal:
            torch._check(
                attn_mask is None, lambda: "Cannot use both is_causal and attn_mask"
            )
            causal_mask = torch.tril(
                torch.ones(
                    q_sequence_length,
                    kv_sequence_length,
                    dtype=torch.bool,
                    device=Q.device,
                )
            )
            attn_bias = attn_bias.masked_fill(~causal_mask, float("-inf"))
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 481-490
```python

        # Apply attention mask
        if attn_mask is not None:
            if attn_mask.dtype == torch.bool:
                # Boolean mask: True means participate in attention
                attn_bias = attn_bias.masked_fill(~attn_mask, float("-inf"))
            else:
                # Float mask: added to attention scores
                attn_bias = attn_bias + attn_mask

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 491-498
```python
        # Apply scaling factor
        scale_factor = _get_scale_factor(scale, Q.shape[3])

        # Scale both Q and K by sqrt(scale_factor) for numerical stability
        sqrt_scale = math.sqrt(scale_factor)
        Q_scaled = Q * sqrt_scale
        K_scaled = K * sqrt_scale

```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 499-507
```python
        # Compute Q @ K^T
        qk_matmul_output = torch.matmul(Q_scaled, K_scaled.transpose(-2, -1))

        # Initialize QK output based on mode
        qk_output = qk_matmul_output  # Default case for mode 0

        # Add attention bias
        qk_with_bias = qk_matmul_output + attn_bias

```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 508-514
```python
        if qk_matmul_output_mode == 1:
            qk_output = qk_with_bias

        # Apply softcap if provided
        if softcap > 0.0:
            qk_with_bias = softcap * torch.tanh(qk_with_bias / softcap)

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 515-521
```python
        if qk_matmul_output_mode == 2:
            qk_output = qk_with_bias

        # Apply softmax with optional precision casting
        if softmax_precision is not None:
            # Map ONNX data type to torch dtype
            if softmax_precision in _ATTENTION_23_ALLOWED_INTERMEDIATE_PRECISIONS:
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。

### Lines 522-532
```python
                original_dtype = qk_with_bias.dtype
                qk_with_bias = qk_with_bias.to(
                    _dtype_mappings.ONNX_DTYPE_TO_TORCH_DTYPE[softmax_precision]
                )
                qk_softmax = torch.softmax(qk_with_bias, dim=-1)
                qk_softmax = qk_softmax.to(original_dtype)
            else:
                qk_softmax = torch.softmax(qk_with_bias, dim=-1)
        else:
            qk_softmax = torch.softmax(qk_with_bias, dim=-1)

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。

### Lines 533-540
```python
        if qk_matmul_output_mode == 3:
            qk_output = qk_softmax

        # Compute attention output
        output = torch.matmul(qk_softmax, V)

    # Reshape output back to 3D if input was 3D
    if input_shape_len == 3:
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 541-546
```python
        # output: (batch_size, q_num_heads, q_sequence_length, v_head_size) -> (batch_size, q_sequence_length, hidden_size)
        output = (
            output.transpose(1, 2).contiguous().view(batch_size, q_sequence_length, -1)
        )

    return output, present_key, present_value, qk_output
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Interop boundary — the implementation connects PyTorch semantics with ONNX model representation requirements.
  CN: 互操作边界——实现负责连接 PyTorch 语义与 ONNX 模型表示需求。
- EN: Module integration — the implementation coordinates neighboring Python modules and utility layers.
  CN: 模块集成——该实现会协调相邻的 Python 模块与工具层。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.onnx.ops`
- External imports / 外部导入: `math`, `collections.abc`, `typing`, `typing_extensions`
- Representative symbols / 代表性符号: `_P`, `_R`, `_ATTENTION_23_ALLOWED_INTERMEDIATE_PRECISIONS`, `_onnx_op`, `_rotary_embedding_23_fake_impl`, `rotary_embedding_23`, `_get_scale_factor`, `_reshape_3d_to_4d`, `_get_qk_output_for_aten_spda`, `_validate_gqa_configuration`, `...`
