# symbolic_opset16.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/_internal/torchscript_exporter/symbolic_opset16.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Defines ONNX symbolic translation rules for symbolic opset16, mapping PyTorch operators to exported ONNX graphs.
- 用途 (CN): 定义与 symbolic opset16 相关的 ONNX symbolic 翻译规则，把 PyTorch 算子映射到导出的 ONNX 图。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```python
# mypy: allow-untyped-defs
"""This file exports ONNX ops for opset 16.

Note [ONNX Operators that are added/updated in opset 16]

```
- EN: This block advances ONNX export translation; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 6-10
```python
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
https://github.com/onnx/onnx/blob/main/docs/Changelog.md#version-16-of-the-default-onnx-operator-set
New operators:
    GridSample https://github.com/onnx/onnx/pull/3557

```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 11-20
```python
Updated operators:
    Identity
    If
    LeakyRelu
    Loop
    PRelu
    RoiAlign
    Scan
    ScatterElements
    ScatterND
```
- EN: This block implements local helper logic for symbolic opset16. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 symbolic opset16 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 21-25
```python
    Where
    GreaterOrEqual
    LessOrEqual
"""

```
- EN: This block implements local helper logic for symbolic opset16. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 symbolic opset16 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 26-30
```python
# EDITING THIS FILE? READ THIS FIRST!
# see Note [Edit Symbolic Files] in README.md

import functools

```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `functools`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`functools`。

### Lines 31-36
```python
import torch
from torch.nn.functional import (
    GRID_SAMPLE_INTERPOLATION_MODES,
    GRID_SAMPLE_PADDING_MODES,
)
from torch.onnx import errors
```
- EN: This block advances ONNX export translation; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 37-44
```python
from torch.onnx._internal.torchscript_exporter import (
    _type_utils,
    jit_utils,
    registration,
    symbolic_helper,
    utils,
)

```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 45-51
```python

_onnx_symbolic = functools.partial(registration.onnx_symbolic, opset=16)


# note (mkozuki): Why `grid_sampler` instead of `grid_sample`?
# Because `torch.nn.functional.grid_sample` calls `torch.grid_sampler`.
@_onnx_symbolic("aten::grid_sampler")
```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 52-61
```python
@symbolic_helper.parse_args("v", "v", "i", "i", "b")
def grid_sampler(
    g: jit_utils.GraphContext,
    input,
    grid,
    mode_enum,
    padding_mode_enum,
    align_corners,
):
    # Check the input and grid tensor rank beforehand.
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `grid_sampler`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`grid_sampler`。

### Lines 62-68
```python
    if symbolic_helper._get_tensor_rank(input) == 5:
        return symbolic_helper._onnx_unsupported("GridSample with 5D volumetric input")
    mode_s = {v: k for k, v in GRID_SAMPLE_INTERPOLATION_MODES.items()}[mode_enum]  # type: ignore[call-arg]
    padding_mode_s = {v: k for k, v in GRID_SAMPLE_PADDING_MODES.items()}[  # type: ignore[call-arg]
        padding_mode_enum
    ]
    return g.op(
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 69-76
```python
        "GridSample",
        input,
        grid,
        align_corners_i=int(align_corners),
        mode_s=mode_s,
        padding_mode_s=padding_mode_s,
    )

```
- EN: This block implements local helper logic for symbolic opset16. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 symbolic opset16 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 77-86
```python

@_onnx_symbolic("aten::scatter_add")
@symbolic_helper.parse_args("v", "i", "v", "v")
def scatter_add(g: jit_utils.GraphContext, self, dim, index, src):
    src_type = _type_utils.JitScalarType.from_value(
        src, _type_utils.JitScalarType.UNDEFINED
    )
    src_sizes = symbolic_helper._get_tensor_sizes(src)
    index_sizes = symbolic_helper._get_tensor_sizes(index)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `scatter_add`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`scatter_add`。

### Lines 87-92
```python
    if len(src_sizes) != len(index_sizes):
        return symbolic_helper._unimplemented(
            "scatter_add",
            f"`index` ({index_sizes}) should have the same dimensionality as `src` ({src_sizes})",
        )

```
- EN: This block advances ONNX export translation; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 93-97
```python
    # PyTorch only allows index shape <= src shape, so we can only consider
    # taking index as subset size to src, like PyTorch does. When sizes for src
    # and index are not matched or there are dynamic axes, we take index shape to
    # slice src to accommodate.
    if src_sizes != index_sizes or None in index_sizes:
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 98-103
```python
        adjusted_shape = g.op("Shape", index)
        starts = g.op("Constant", value_t=torch.tensor([0] * len(index_sizes)))
        src = g.op("Slice", src, starts, adjusted_shape)

    src = symbolic_helper._maybe_get_scalar(src)
    if symbolic_helper._is_value(src):
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。

### Lines 104-108
```python
        return g.op("ScatterElements", self, index, src, axis_i=dim, reduction_s="add")
    else:
        # Check if scalar "src" has same type as self (PyTorch allows different
        # type for scalar src (but not when src is tensor)). If not, insert Cast node.
        if _type_utils.JitScalarType.from_value(self) != src_type:
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；操作图状程序结构；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 109-114
```python
            src = g.op(
                "Cast",
                src,
                to_i=_type_utils.JitScalarType.from_value(self).onnx_type(),
            )

```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 115-123
```python
        return g.op(
            "ScatterElements",
            self,
            index,
            src,
            axis_i=dim,
            reduction_s="add",
        )

```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 124-133
```python

@_onnx_symbolic("aten::scatter_reduce")
@symbolic_helper.parse_args("v", "i", "v", "v", "s", "b")
def scatter_reduce(
    g: jit_utils.GraphContext,
    self: torch._C.Value,
    dim: int,
    index: torch._C.Value,
    src: torch._C.Value,
    reduce: str,
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `scatter_reduce`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`scatter_reduce`。

### Lines 134-140
```python
    include_self: bool,
):
    if reduce == "mean":
        raise errors.OnnxExporterError(
            "ONNX does not support mean reduction for scatter_reduce"
        )
    if not include_self:
```
- EN: This block advances ONNX export translation; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 141-150
```python
        raise errors.OnnxExporterError(
            "ONNX does not support include_self=False for scatter_reduce"
        )

    reduce_mode = {  # convert torch string name to onnx string name
        "mean": "none",  # 'mean' doesn't support in ONNX 1.14 definition
        "sum": "add",
        "prod": "mul",
        "amin": "min",
        "amax": "max",
```
- EN: This block advances ONNX export translation; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 151-155
```python
    }
    onnx_reduce = reduce_mode[reduce]

    self_rank = g.op("Size", g.op("Shape", self))

```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 156-164
```python
    # if self_rank == 0:  # assert (index_rank == 0 and rank_src == 0)
    self_rank_is_zero = g.op(
        "Equal", self_rank, g.op("Constant", value_t=torch.tensor(0, dtype=torch.int64))
    )
    if_op, (if_context, else_context), _ = jit_utils.add_op_with_blocks(
        g, "If", self_rank_is_zero, n_blocks=2, outputs=3
    )
    neg_1 = if_context.op("Constant", value_t=torch.tensor([-1], dtype=torch.int64))

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 165-171
```python
    self_reshape = if_context.op("Reshape", self, neg_1)
    utils._add_output_to_block(if_context.block, self_reshape)
    index_reshape = if_context.op("Reshape", index, neg_1)
    utils._add_output_to_block(if_context.block, index_reshape)
    src_reshape = if_context.op("Reshape", src, neg_1)
    utils._add_output_to_block(if_context.block, src_reshape)

```
- EN: This block protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 172-178
```python
    self_identity = else_context.op("Identity", self)
    utils._add_output_to_block(else_context.block, self_identity)
    index_identitye = else_context.op("Identity", index)
    utils._add_output_to_block(else_context.block, index_identitye)
    src_identity = else_context.op("Identity", src)
    utils._add_output_to_block(else_context.block, src_identity)

```
- EN: This block protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 179-188
```python
    result = g.op("ScatterElements", *if_op, axis_i=dim, reduction_s=onnx_reduce)

    # if self_rank == 0:
    if_op, (if_context, else_context), _ = jit_utils.add_op_with_blocks(
        g, "If", self_rank_is_zero, n_blocks=2, outputs=1
    )
    result_squeezed = if_context.op("Squeeze", result)
    utils._add_output_to_block(if_context.block, result_squeezed)
    result_identity = else_context.op("Identity", result)
    utils._add_output_to_block(else_context.block, result_identity)
```
- EN: This block advances ONNX export translation; handles conditional control flow; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 189-191
```python
    result_final = if_op.node().output()

    return result_final
```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Concurrency support — the implementation manages coordination, ownership, or thread safety.
  CN: 并发支持——实现会管理协作、所有权或线程安全。
- EN: Interop boundary — the implementation connects PyTorch semantics with ONNX model representation requirements.
  CN: 互操作边界——实现负责连接 PyTorch 语义与 ONNX 模型表示需求。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.nn.functional`, `torch.onnx`, `torch.onnx._internal.torchscript_exporter`
- External imports / 外部导入: `functools`
- Representative symbols / 代表性符号: `grid_sampler`, `scatter_add`, `scatter_reduce`
