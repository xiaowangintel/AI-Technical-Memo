# nn.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/_internal/exporter/_torchlib/ops/nn.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal ONNX exporter logic for nn, covering graph capture, translation, validation, or serialization.
- 用途 (CN): 实现与 nn 相关的 ONNX 导出器内部逻辑，涵盖图捕获、转换、校验或序列化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
"""torch.ops.aten operators under the `core` module."""
# mypy: disable-error-code="misc,arg-type,type-arg,valid-type,assignment,return-value,type-var,operator,no-untyped-def,index"
# pyrefly: ignore-errors
# ruff: noqa: TC001

from __future__ import annotations

```
- EN: This block reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 8-15
```python
from typing import Sequence, TYPE_CHECKING  # noqa: UP035

from onnxscript.onnx_opset import (  # type: ignore[attr-defined]
    opset20 as op20,
    opset21 as op21,
    opset23 as op23,
)

```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 16-22
```python
import torch
from torch.onnx._internal._lazy_import import onnx_ir as ir
from torch.onnx._internal.exporter._torchlib._tensor_typing import TFloat, TReal
from torch.onnx._internal.exporter._torchlib._torchlib_registry import onnx_impl


if TYPE_CHECKING:
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter._torchlib._tensor_typing`, `torch.onnx._internal.exporter._torchlib._torchlib_registry`; external imports: none.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter._torchlib._tensor_typing`, `torch.onnx._internal.exporter._torchlib._torchlib_registry`；外部导入：无。

### Lines 23-29
```python
    from onnxscript.values import Opset

aten = torch.ops.aten


@onnx_impl(aten.gelu.default, trace_only=True, opset_introduced=20)
def aten_gelu_opset20(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `aten_gelu_opset20`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`aten_gelu_opset20`。

### Lines 30-36
```python
    self: TReal,
    approximate: str = "none",
) -> TReal:
    """gelu(Tensor self, *, str approximate="none") -> Tensor"""
    return op20.Gelu(self, approximate=approximate)


```
- EN: This block handles tensor metadata or sample values; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 37-47
```python
@onnx_impl(aten.group_norm.default, trace_only=True, opset_introduced=21)
def aten_group_norm(
    input: TFloat,
    num_groups: int,
    weight: TFloat | None = None,
    bias: TFloat | None = None,
    eps: float = 1e-05,
    cudnn_enabled: bool = True,
) -> TFloat:
    """group_norm(Tensor input, int num_groups, Tensor? weight=None, Tensor? bias=None, float eps=1e-05, bool cudnn_enabled=True) -> Tensor"""

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `aten_group_norm`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`aten_group_norm`。

### Lines 48-56
```python
    c = op21.Shape(input, start=1, end=2)
    if weight is None:
        weight = op21.ConstantOfShape(c, value=ir.tensor([1.0], dtype=input.dtype))
    if bias is None:
        bias = op21.ConstantOfShape(c, value=ir.tensor([0.0], dtype=input.dtype))
    return op21.GroupNormalization(
        input, weight, bias, epsilon=eps, num_groups=num_groups
    )

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 57-66
```python

@onnx_impl(aten.rms_norm.default, trace_only=True, opset_introduced=23)
def aten_rms_norm(
    input: TFloat,
    normalized_shape: Sequence[int],
    weight: TFloat | None = None,
    eps: float | None = None,
) -> TFloat:
    """rms_norm(Tensor input, SymInt[] normalized_shape, Tensor? weight=None, float? eps=None) -> Tensor"""

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `aten_rms_norm`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`aten_rms_norm`。

### Lines 67-77
```python
    # Default eps value if not provided
    if eps is None:
        eps = torch.finfo(torch.float).eps  # Observed from decomp

    # Calculate axis: the first normalization dimension
    # For normalized_shape with D dimensions, normalize over last D dimensions
    # Since ONNX RMSNormalization supports negative axis values, we use -len(normalized_shape)
    # which correctly maps to the first axis of the normalized dimensions
    normalized_dims = len(normalized_shape)
    axis = -normalized_dims

```
- EN: This block advances ONNX export translation; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 78-84
```python
    # Create weight tensor if not provided
    if weight is None:
        weight = op23.ConstantOfShape(
            op23.Shape(input), value=ir.tensor([1], dtype=input.dtype)
        )

    return op23.RMSNormalization(input, weight, axis=axis, epsilon=eps)
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 85-98
```python


@onnx_impl(
    aten.scaled_dot_product_attention.default, trace_only=True, opset_introduced=23
)
def aten_scaled_dot_product_attention_23(
    query: TFloat,
    key: TFloat,
    value: TFloat,
    attn_mask: TFloat | None = None,
    dropout_p: float = 0.0,
    is_causal: bool = False,
    scale: float | None = None,
    enable_gqa: bool = False,
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `aten_scaled_dot_product_attention_23`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`aten_scaled_dot_product_attention_23`。

### Lines 99-105
```python
) -> TFloat:
    """scaled_dot_product_attention(Tensor query, Tensor key, Tensor value, Tensor? attn_mask=None, float dropout_p=0.0, bool is_causal=False, *, float? scale=None, bool enable_gqa=False) -> Tensor

    Reference:
        1. https://pytorch.org/docs/stable/generated/torch.nn.functional.scaled_dot_product_attention.html
        2. https://onnx.ai/onnx/operators/onnx__Attention.html

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 106-115
```python
    Attempts to convert SDPA to Attention onnx op and fallbacks to an onnx graph equivalent to the following PyTorch code::
        scale_factor = 1 / math.sqrt(Q.size(-1)) if scale is None else scale
        attn_mask = (
            torch.ones(L, S, dtype=torch.bool).tril(diagonal=0)
            if is_causal
            else attn_mask
        )
        attn_mask = (
            attn_mask.masked_fill(not attn_mask, -float("inf"))
            if attn_mask.dtype == torch.bool
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流。关键符号：无明显局部符号。

### Lines 116-122
```python
            else attn_mask
        )
        attn_weight = torch.softmax(
            (Q @ K.transpose(-2, -1) * scale_factor) + attn_mask, dim=-1
        )
        attn_weight = torch.dropout(attn_weight, dropout_p)
        return attn_weight @ V
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 123-129
```python

    where Q, K, V are the query, key, and value tensors, respectively.
    L is the target sequence length, S is the source sequence length, and E is the embedding size.
    """
    if is_causal and attn_mask is not None:
        raise AssertionError("is_causal and attn_mask cannot be set at the same time")
    if not (len(query.shape) == 4 and len(key.shape) == 4 and len(value.shape) == 4):
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 130-143
```python
        raise AssertionError("only 4D query, key, and value are supported")

    # Attention onnx op can only handle non-training scenarios where dropout is disabled.
    if dropout_p == 0:
        if enable_gqa:
            if not (
                query.shape[1] > key.shape[1] == value.shape[1]
                and query.shape[1] % key.shape[1] == 0
            ):
                raise AssertionError(
                    "SDPA (GQA or MQA) requires q_num_heads > kv_num_heads & "
                    "q_num_heads % kv_num_heads == 0"
                )
        else:
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 144-153
```python
            if not (query.shape[1] == key.shape[1] == value.shape[1]):
                raise AssertionError("SDPA (MHA) requires q_num_heads = kv_num_heads")

        # NOTE: num_heads attributes (q_num_heads/kv_num_heads) should not be specified for 4D.
        # They are not populated with 4D inputs because this information directly comes from input shapes:
        # `q_num_heads=query.shape[1]` and `kv_num_heads=key.shape[1]`.
        # This dimension is usually static but it could not be dynamic if also given as an attribute.
        # num_heads attributes are needed for 3D attention inputs:
        # (shape: [B, S, N*H]), 4D shape is ([B, N, S, H]).

```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 154-162
```python
        Y, _, _, _ = op23.Attention(
            query,
            key,
            value,
            attn_mask=attn_mask,
            scale=scale,
            is_causal=is_causal,
        )
        return Y
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 163-170
```python

    if scale is None:
        scale = _attention_scale(query, op23)
    scale = op23.CastLike(scale, query)

    if is_causal:
        attn_mask = _causal_attention_mask(query, key, op23)

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 171-178
```python
    if enable_gqa:
        key, value = _attention_repeat_kv_for_group_query(query, key, value, op23)

    if attn_mask is None:
        return _aten_scaled_dot_product_attention_no_mask_onnx(
            query, key, value, scale, dropout_p, op23
        )

```
- EN: This block advances ONNX export translation; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 179-188
```python
    return _aten_scaled_dot_product_attention_float_mask_onnx(
        query, key, value, attn_mask, scale, dropout_p, op23
    )


def _attention_repeat_kv_for_group_query(
    query: TFloat, key: TFloat, value: TFloat, op: Opset
) -> tuple[TFloat, TFloat]:
    """Expand key and value for group query attention.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_attention_repeat_kv_for_group_query`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_attention_repeat_kv_for_group_query`。

### Lines 189-195
```python
    repeat_interleave is applied on key and value to match the number of heads in query.

    Args:
        query: Tensor of shape [B, q_num_heads, q_S, E]
        key: Tensor of shape [B, k_num_heads, kv_S, E]
        value: Tensor of shape [B, v_num_heads, kv_S, E]

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 196-202
```python
    Returns:
        Tuple of (expanded_key, expanded_value) where:
            - expanded_key: Tensor of shape [B, q_num_heads, kv_S, E]
            - expanded_value: Tensor of shape [B, q_num_heads, kv_S, E]
    """

    if not (
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 203-210
```python
        query.shape[1] > key.shape[1] == value.shape[1]
        and query.shape[1] % key.shape[1] == 0
    ):
        raise AssertionError(
            "SDPA (GQA or MQA) requires q_num_heads > kv_num_heads & "
            "q_num_heads % kv_num_heads == 0"
        )

```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 211-219
```python
    # NOTE: QKV are expected to be 4D tensors

    batch_size = op.Shape(query, start=0, end=1)  # [B]
    q_num_heads = op.Shape(query, start=1, end=2)  # [Hq]
    kv_num_heads = op.Shape(key, start=1, end=2)  # [Hk]
    qk_head_size = op.Shape(key, start=3, end=4)  # [Dk]
    v_head_size = op.Shape(value, start=3, end=4)  # [Dv]
    new_kv_seq_len = op.Shape(key, start=2, end=3)  # [T]

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 220-233
```python
    interleave_dim = op.Div(q_num_heads, kv_num_heads)  # Hq / Hk
    two = op.Constant(value_int=2)
    k_unsqueezed = op.Unsqueeze(key, two)  # [B, Hk, 1, T, Dk]
    v_unsqueezed = op.Unsqueeze(value, two)  # [B, Hv, 1, T, Dv]

    k_expand_shape = op.Concat(
        batch_size, kv_num_heads, interleave_dim, new_kv_seq_len, qk_head_size, axis=0
    )
    k_expand = op.Expand(k_unsqueezed, k_expand_shape)
    v_expand_shape = op.Concat(
        batch_size, kv_num_heads, interleave_dim, new_kv_seq_len, v_head_size, axis=0
    )
    v_expand = op.Expand(v_unsqueezed, v_expand_shape)

```
- EN: This block implements local helper logic for nn. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 nn 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 234-240
```python
    k_attention_shape = op.Concat(
        batch_size, q_num_heads, new_kv_seq_len, qk_head_size, axis=0
    )
    v_attention_shape = op.Concat(
        batch_size, q_num_heads, new_kv_seq_len, v_head_size, axis=0
    )

```
- EN: This block implements local helper logic for nn. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 nn 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 241-247
```python
    expanded_key = op.Reshape(k_expand, k_attention_shape)
    expanded_value = op.Reshape(v_expand, v_attention_shape)

    return expanded_key, expanded_value


def _attention_scale(query: TFloat, op: Opset) -> TFloat:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_attention_scale`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_attention_scale`。

### Lines 248-261
```python
    """Calculate the scale factor for the attention result.

    Args:
        query: Tensor of shape [..., L, E]

    Returns:
        Scalar scale factor := 1 / math.sqrt(query.size(-1))
    """
    q_shape = op.Shape(query)
    q_last_dim = op.Gather(q_shape, op.Constant(value_ints=[-1]))
    embedding_size = op.CastLike(q_last_dim, query)
    one = op.Constant(value_float=1.0)
    cast_one = op.CastLike(one, query)
    scale = op.Div(cast_one, op.Sqrt(embedding_size))
```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 262-272
```python
    return scale


def _causal_attention_mask(query: TFloat, key: TFloat, op: Opset) -> TFloat:
    """Create a causal mask for the given query and key tensors.

    Equivalent to::
        mask = torch.ones(L, S, dtype=torch.bool).tril(diagonal=0)
        attn_mask = torch.zeros(L, S, dtype=torch.float)
        attn_mask = attn_mask.masked_fill(not mask, -float("inf"))

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_causal_attention_mask`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_causal_attention_mask`。

### Lines 273-282
```python
    Args:
        query: Tensor of shape [..., L, E]
        key: Tensor of shape [..., S, E]

    Returns:
        Tensor of shape [L, S]
    """
    q_shape = op.Shape(query)
    k_shape = op.Shape(key)

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 283-296
```python
    target_length = op.Slice(
        q_shape, op.Constant(value_ints=[-2]), op.Constant(value_ints=[-1])
    )
    source_length = op.Slice(
        k_shape, op.Constant(value_ints=[-2]), op.Constant(value_ints=[-1])
    )
    # attn_mask = torch.ones(L, S) := {
    size = op.Concat(target_length, source_length, axis=0)
    attn_mask = op.Expand(op.Constant(value_float=1.0), size)
    # }
    attn_mask = op.Trilu(attn_mask, upper=0)
    # The causal mask has 0s in the lower triangle and -inf in the upper triangle.
    attn_mask = op.Where(
        op.Equal(attn_mask, op.Constant(value_float=0.0)),
```
- EN: This block bridges to backend-specific execution artifacts. Key symbols: no dominant local symbols.
- CN: 该代码块桥接到特定后端的执行产物。关键符号：无明显局部符号。

### Lines 297-303
```python
        op.Constant(value_float=-float("inf")),
        op.Constant(value_float=0.0),
    )
    attn_mask = op.CastLike(attn_mask, query)
    return attn_mask


```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 304-317
```python
def _aten_scaled_dot_product_attention_no_mask_onnx(
    query: TFloat,
    key: TFloat,
    value: TFloat,
    scale: TFloat,
    dropout_p: float,
    op: Opset,
) -> TFloat:
    # Swap the last two axes of key
    key_last_dim = op.Shape(key, start=-1)
    key_second_last_dim = op.Shape(key, start=-2, end=-1)
    key_first_dims = op.Shape(key, end=-2)
    # Contract the dimensions that are not the last two so we can transpose
    # with a static permutation.
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_aten_scaled_dot_product_attention_no_mask_onnx`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_aten_scaled_dot_product_attention_no_mask_onnx`。

### Lines 318-327
```python
    key_squeezed_shape = op.Concat(
        op.Constant(value_ints=[-1]), key_second_last_dim, key_last_dim, axis=0
    )
    key_squeezed = op.Reshape(key, key_squeezed_shape)
    key_squeezed_transposed = op.Transpose(key_squeezed, perm=[0, 2, 1])
    key_transposed_shape = op.Concat(
        key_first_dims, key_last_dim, key_second_last_dim, axis=0
    )
    key_transposed = op.Reshape(key_squeezed_transposed, key_transposed_shape)

```
- EN: This block implements local helper logic for nn. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 nn 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 328-339
```python
    # https://github.com/pytorch/pytorch/blob/12da0c70378b5be9135c6fda62a9863bce4a4818/aten/src/ATen/native/transformers/attention.cpp#L653
    # Scale q, k before matmul for stability see https://tinyurl.com/sudb9s96 for math
    query_scaled = op.Mul(query, op.Sqrt(scale))
    key_transposed_scaled = op.Mul(
        key_transposed, op.CastLike(op.Sqrt(scale), key_transposed)
    )
    attn_weight = op.Softmax(
        op.MatMul(query_scaled, key_transposed_scaled),
        axis=-1,
    )
    attn_weight, _ = op.Dropout(attn_weight, dropout_p)
    return op.MatMul(attn_weight, value)
```
- EN: This block iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 340-353
```python


def _aten_scaled_dot_product_attention_float_mask_onnx(
    query: TFloat,
    key: TFloat,
    value: TFloat,
    attn_mask: TFloat,
    scale: TFloat,
    dropout_p: float,
    op: Opset,
) -> TFloat:
    # Swap the last two axes of key
    key_last_dim = op.Shape(key, start=-1)
    key_second_last_dim = op.Shape(key, start=-2, end=-1)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_aten_scaled_dot_product_attention_float_mask_onnx`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_aten_scaled_dot_product_attention_float_mask_onnx`。

### Lines 354-366
```python
    key_first_dims = op.Shape(key, end=-2)
    # Contract the dimensions that are not the last two so we can transpose
    # with a static permutation.
    key_squeezed_shape = op.Concat(
        op.Constant(value_ints=[-1]), key_second_last_dim, key_last_dim, axis=0
    )
    key_squeezed = op.Reshape(key, key_squeezed_shape)
    key_squeezed_transposed = op.Transpose(key_squeezed, perm=[0, 2, 1])
    key_transposed_shape = op.Concat(
        key_first_dims, key_last_dim, key_second_last_dim, axis=0
    )
    key_transposed = op.Reshape(key_squeezed_transposed, key_transposed_shape)

```
- EN: This block implements local helper logic for nn. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 nn 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 367-376
```python
    # https://github.com/pytorch/pytorch/blob/12da0c70378b5be9135c6fda62a9863bce4a4818/aten/src/ATen/native/transformers/attention.cpp#L653
    # Scale q, k before matmul for stability see https://tinyurl.com/sudb9s96 for math
    query_scaled = op.Mul(query, op.Sqrt(scale))
    key_transposed_scaled = op.Mul(key_transposed, op.Sqrt(scale))
    attn_weight = op.Softmax(
        op.Add(op.MatMul(query_scaled, key_transposed_scaled), attn_mask),
        axis=-1,
    )
    attn_weight, _ = op.Dropout(attn_weight, dropout_p)
    return op.MatMul(attn_weight, value)
```
- EN: This block iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Backend lowering — the code bridges higher-level programs to executable backend artifacts.
  CN: 后端降级——代码把更高层程序桥接到可执行的后端产物。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter._torchlib._tensor_typing`, `torch.onnx._internal.exporter._torchlib._torchlib_registry`
- External imports / 外部导入: `__future__`, `typing`, `onnxscript.onnx_opset`, `onnxscript.values`
- Representative symbols / 代表性符号: `aten_gelu_opset20`, `aten_group_norm`, `aten_rms_norm`, `aten_scaled_dot_product_attention_23`, `_attention_repeat_kv_for_group_query`, `_attention_scale`, `_causal_attention_mask`, `_aten_scaled_dot_product_attention_no_mask_onnx`, `_aten_scaled_dot_product_attention_float_mask_onnx`
