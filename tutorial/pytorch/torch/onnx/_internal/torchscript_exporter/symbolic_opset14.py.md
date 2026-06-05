# symbolic_opset14.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/_internal/torchscript_exporter/symbolic_opset14.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Defines ONNX symbolic translation rules for symbolic opset14, mapping PyTorch operators to exported ONNX graphs.
- 用途 (CN): 定义与 symbolic opset14 相关的 ONNX symbolic 翻译规则，把 PyTorch 算子映射到导出的 ONNX 图。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9
```python
# mypy: allow-untyped-defs
# mypy: disable-error-code=arg-type
"""This file exports ONNX ops for opset 14.

Note [ONNX operators that are added/updated in opset 14]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
New operators:
    HardSwish, Trilu

```
- EN: This block advances ONNX export translation; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 10-16
```python
Updated operators:
    Reshape
    Add, Sub, Mul, Div
    GRU, LSTM, RNN
    BatchNorm, Cumsum, Relu
"""

```
- EN: This block implements local helper logic for symbolic opset14. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 symbolic opset14 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 17-23
```python
# EDITING THIS FILE? READ THIS FIRST!
# see Note [Edit Symbolic Files] in README.md
from __future__ import annotations

import functools

import torch
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`; external imports: `__future__`, `functools`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`；外部导入：`__future__`, `functools`。

### Lines 24-31
```python
from torch.onnx import _constants
from torch.onnx._internal.torchscript_exporter import (
    _type_utils,
    jit_utils,
    registration,
    symbolic_helper,
)
from torch.onnx._internal.torchscript_exporter._globals import GLOBALS
```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 32-43
```python


__all__ = [
    "hardswish",
    "tril",
    "triu",
    "reshape",
    "batch_norm",
    "quantized_hardswish",
    "scaled_dot_product_attention",
]

```
- EN: This block implements local helper logic for symbolic opset14. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 symbolic opset14 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 44-50
```python
_onnx_symbolic = functools.partial(registration.onnx_symbolic, opset=14)


@_onnx_symbolic("aten::hardswish")
@symbolic_helper.parse_args("v")
def hardswish(g: jit_utils.GraphContext, self):
    return g.op("HardSwish", self)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `hardswish`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`hardswish`。

### Lines 51-57
```python


@_onnx_symbolic("aten::tril")
def tril(g: jit_utils.GraphContext, self, diagonal, out=None):
    return g.op("Trilu", self, diagonal, upper_i=0)


```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `tril`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`tril`。

### Lines 58-64
```python
@_onnx_symbolic("aten::triu")
def triu(g: jit_utils.GraphContext, self, diagonal, out=None):
    return g.op("Trilu", self, diagonal, upper_i=1)


@_onnx_symbolic("aten::reshape")
@symbolic_helper.quantized_args(True)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `triu`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`triu`。

### Lines 65-71
```python
@symbolic_helper.parse_args("v", "v")
def reshape(g: jit_utils.GraphContext, self, shape):
    # NOTE: Due to bug in ORT https://github.com/microsoft/onnxruntime/issues/10664
    #       Reshape export cannot utilize the new allowzero attribute introduced in opset 14.
    return symbolic_helper._reshape_helper(g, self, shape, allowzero=0)


```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reshape`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reshape`。

### Lines 72-85
```python
@_onnx_symbolic("aten::batch_norm")
@symbolic_helper.parse_args("v", "v", "v", "v", "v", "i", "f", "f", "i")
def batch_norm(
    g: jit_utils.GraphContext,
    input,
    weight,
    bias,
    running_mean,
    running_var,
    training,
    momentum,
    eps,
    cudnn_enabled,
):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `batch_norm`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`batch_norm`。

### Lines 86-93
```python
    if (
        torch.is_autocast_enabled()
        and not symbolic_helper.args_have_same_dtype(
            [input, weight, bias, running_mean, running_var]
        )
        and GLOBALS.export_onnx_opset_version < 15
    ):
        return symbolic_helper._onnx_opset_unsupported_detailed(
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 94-101
```python
            "BatchNormalization",
            14,
            15,
            "All input tensors must have the same `dtype`."
            " Turn off Autocast or export using opset version 15.",
            input,
        )

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 102-115
```python
    symbolic_helper.check_training_mode(training, "batch_norm")
    weight, bias, running_mean, running_var = symbolic_helper._batchnorm_helper(
        g, input, weight, bias, running_mean, running_var
    )
    out = g.op(
        "BatchNormalization",
        input,
        weight,
        bias,
        running_mean,
        running_var,
        epsilon_f=eps,
        momentum_f=1 - momentum,
        training_mode_i=0 if not training else 1,
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。

### Lines 116-124
```python
        outputs=1 if not training else 3,
    )
    if not training:
        return out
    else:
        res, new_running_mean, new_running_var = out
        new_running_mean.setType(running_mean.type())
        new_running_var.setType(running_var.type())
        return res
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 125-132
```python


@_onnx_symbolic("quantized::hardswish")
def quantized_hardswish(g: jit_utils.GraphContext, x, op_scale, op_zero_point):
    x, _, _, _ = symbolic_helper.dequantize_helper(g, x)

    output = hardswish(g, x)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `quantized_hardswish`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`quantized_hardswish`。

### Lines 133-140
```python
    return symbolic_helper.quantize_helper(g, output, op_scale, op_zero_point)


# Ported from
# https://github.com/microsoft/onnxscript/blob/6b1b81700b4523f31d8c6d3321e5d8ef5d42b764/onnxscript/function_libs/torch_aten/ops/nn.py#L1504
# aten_scaled_dot_product_attention
# NOTE: Need op.Trilu
@_onnx_symbolic("aten::scaled_dot_product_attention")
```
- EN: This block advances ONNX export translation; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 141-153
```python
@symbolic_helper.parse_args("v", "v", "v", "v", "f", "b", "v", "b")
def scaled_dot_product_attention(
    g: jit_utils.GraphContext,
    query: torch._C.Value,
    key: torch._C.Value,
    value: torch._C.Value,
    attn_mask: torch._C.Value | None = None,
    dropout_p: float = 0.0,
    is_causal: bool = False,
    scale: torch._C.Value | None = None,
    enable_gqa: bool = False,
):
    if is_causal and not symbolic_helper._is_none(attn_mask):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `scaled_dot_product_attention`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`scaled_dot_product_attention`。

### Lines 154-160
```python
        raise AssertionError("is_causal and attn_mask cannot be set at the same time")
    if enable_gqa:
        raise AssertionError(
            "conversion of scaled_dot_product_attention not implemented if enable_gqa is True"
        )

    if symbolic_helper._is_none(scale):
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 161-174
```python
        scale = _attention_scale(g, query)

    if is_causal:
        attn_mask = _causal_attention_mask(g, query, key)

    # Swap the last two axes of key
    # NOTE: onnx-script has different logic here, because the attribute perms in
    # transpose needs list of ints
    key_shape_builtin = symbolic_helper._get_tensor_rank(key)
    # pyrefly: ignore [bad-argument-type, no-matching-overload]
    key_transposed_axes = list(range(key_shape_builtin))
    key_transposed_axes[-1], key_transposed_axes[-2] = (
        key_transposed_axes[-2],
        key_transposed_axes[-1],
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。

### Lines 175-185
```python
    )
    key_transposed = g.op("Transpose", key, perm_i=key_transposed_axes)

    # https://github.com/pytorch/pytorch/blob/12da0c70378b5be9135c6fda62a9863bce4a4818/aten/src/ATen/native/transformers/attention.cpp#L653
    # Scale q, k before matmul for stability see https://tinyurl.com/sudb9s96 for math
    # pyrefly: ignore [bad-argument-type]
    query_scaled = g.op("Mul", query, g.op("Sqrt", scale))
    # pyrefly: ignore [bad-argument-type]
    key_transposed_scaled = g.op("Mul", key_transposed, g.op("Sqrt", scale))
    mul_qk = g.op("MatMul", query_scaled, key_transposed_scaled)

```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 186-199
```python
    if symbolic_helper._is_none(attn_mask):
        mul_qk_add = mul_qk
        attn_weight = g.op("Softmax", mul_qk_add, axis_i=-1)
    elif (
        _type_utils.JitScalarType.from_value(attn_mask)
        == _type_utils.JitScalarType.BOOL
    ):
        # Turn the Boolean mask to float: attn_mask.masked_fill(not attn_mask, -float('inf'))
        const_zero = g.op("Constant", value_t=torch.tensor([0.0]))
        const_neg_inf = g.op("Constant", value_t=torch.tensor([-float("inf")]))
        # pyrefly: ignore [bad-argument-type]
        attn_mask = g.op("Where", attn_mask, const_zero, const_neg_inf)
        mul_qk_add = g.op("Add", mul_qk, attn_mask)
        attn_weight = g.op("Softmax", mul_qk_add, axis_i=-1)
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。

### Lines 200-213
```python
        # When using scaled dot product attention with a boolean mask, the softmax operation might return NaN values
        # due to the presence of -inf in an entire row (padding tokens), resulting in 0/0 (NaN) in the softmax output.
        # This is because there's no safe softmax imp in ONNX, so we need to handle NaN values explicitly to match
        # the behavior of PyTorch with boolean masks.
        attn_weight = g.op("Where", g.op("IsNaN", attn_weight), const_zero, attn_weight)
    elif _type_utils.JitScalarType.from_value(attn_mask) in (
        _type_utils.JitScalarType.FLOAT,
        _type_utils.JitScalarType.HALF,
        _type_utils.JitScalarType.BFLOAT16,
    ):
        # pyrefly: ignore [bad-argument-type]
        mul_qk_add = g.op("Add", mul_qk, attn_mask)
        attn_weight = g.op("Softmax", mul_qk_add, axis_i=-1)
    else:
```
- EN: This block advances ONNX export translation; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 214-224
```python
        raise ValueError(
            f"Unsupported type for attn_mask: {_type_utils.JitScalarType.from_value(attn_mask)}"
        )

    if dropout_p != 0:
        attn_weight = g.op(
            "Dropout",
            attn_weight,
            g.op("Constant", value_t=torch.tensor(dropout_p, dtype=torch.float)),
        )

```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 225-232
```python
    return g.op("MatMul", attn_weight, value)


def _attention_scale(
    g: jit_utils.GraphContext, query: torch._C.Value
) -> torch._C.Value:
    """Calculate the scale factor for the attention result.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_attention_scale`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_attention_scale`。

### Lines 233-246
```python
    Args:
        query: Tensor of shape [..., L, E]

    Returns:
        Scalar scale factor := 1 / math.sqrt(query.size(-1))
    """
    query_shape = g.op("Shape", query)
    query_shape_last = g.op(
        "Slice",
        query_shape,
        g.op("Constant", value_t=torch.tensor([-1], dtype=torch.int64)),
        g.op(
            "Constant", value_t=torch.tensor([_constants.INT64_MAX], dtype=torch.int64)
        ),
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 247-260
```python
    )
    embedding_size = g.op(
        "Cast",
        query_shape_last,
        to_i=_type_utils.JitScalarType.from_value(query).onnx_type(),
    )
    const_one = g.op("Constant", value_t=torch.tensor([1.0], dtype=torch.float))
    scale = g.op("Div", const_one, g.op("Sqrt", embedding_size))
    # Add a Cast to convert the scale back to original type
    scale = g.op(
        "Cast",
        scale,
        to_i=_type_utils.JitScalarType.from_value(query).onnx_type(),
    )
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 261-268
```python
    return scale


def _causal_attention_mask(
    g: jit_utils.GraphContext, query: torch._C.Value, key: torch._C.Value
) -> torch._C.Value:
    """Create a causal mask for the given query and key tensors.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_causal_attention_mask`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_causal_attention_mask`。

### Lines 269-277
```python
    Equivalent to::
        mask = torch.ones(L, S, dtype=torch.bool).tril(diagonal=0)
        attn_mask = torch.zeros(L, S, dtype=torch.float)
        attn_mask = attn_mask.masked_fill(not mask, -float("inf"))

    Args:
        query: Tensor of shape [..., L, E]
        key: Tensor of shape [..., S, E]

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 278-284
```python
    Returns:
        Tensor of shape [L, S]
    """

    query_shape = g.op("Shape", query)
    key_shape = g.op("Shape", key)

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 285-298
```python
    last_idx = g.op("Constant", value_t=torch.tensor([-1], dtype=torch.int64))
    second_last_idx = g.op("Constant", value_t=torch.tensor([-2], dtype=torch.int64))
    target_length = g.op("Slice", query_shape, second_last_idx, last_idx)
    source_length = g.op("Slice", key_shape, second_last_idx, last_idx)
    # attn_mask = torch.ones(L, S) := {
    size = g.op("Concat", target_length, source_length, axis_i=0)
    const_one = g.op("Constant", value_t=torch.tensor([1.0]))
    attn_mask = g.op("Expand", const_one, size)
    # }
    attn_mask = g.op("Trilu", attn_mask, upper_i=0)
    # The causal mask has 0s in the lower triangle and -inf in the upper triangle.
    const_zero = g.op("Constant", value_t=torch.tensor([0.0]))
    const_neg_inf = g.op("Constant", value_t=torch.tensor([-float("inf")]))
    attn_mask = g.op(
```
- EN: This block handles tensor metadata or sample values; bridges to backend-specific execution artifacts. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；桥接到特定后端的执行产物。关键符号：无明显局部符号。

### Lines 299-301
```python
        "Where", g.op("Equal", attn_mask, const_zero), const_neg_inf, const_zero
    )
    return attn_mask
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Backend lowering — the code bridges higher-level programs to executable backend artifacts.
  CN: 后端降级——代码把更高层程序桥接到可执行的后端产物。
- EN: Interop boundary — the implementation connects PyTorch semantics with ONNX model representation requirements.
  CN: 互操作边界——实现负责连接 PyTorch 语义与 ONNX 模型表示需求。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.onnx`, `torch.onnx._internal.torchscript_exporter`, `torch.onnx._internal.torchscript_exporter._globals`
- External imports / 外部导入: `__future__`, `functools`
- Representative symbols / 代表性符号: `hardswish`, `tril`, `triu`, `reshape`, `batch_norm`, `quantized_hardswish`, `scaled_dot_product_attention`, `_attention_scale`, `_causal_attention_mask`
