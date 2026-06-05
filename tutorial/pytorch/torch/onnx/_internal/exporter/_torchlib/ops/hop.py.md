# hop.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/_internal/exporter/_torchlib/ops/hop.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal ONNX exporter logic for hop, covering graph capture, translation, validation, or serialization.
- 用途 (CN): 实现与 hop 相关的 ONNX 导出器内部逻辑，涵盖图捕获、转换、校验或序列化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
"""Implementation for higher-order operators."""

from __future__ import annotations

from typing import TYPE_CHECKING

import torch
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`; external imports: `__future__`, `typing`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`；外部导入：`__future__`, `typing`。

### Lines 8-14
```python
from torch.onnx._internal._lazy_import import onnx_ir as ir
from torch.onnx._internal.exporter import _core
from torch.onnx._internal.exporter._torchlib._torchlib_registry import onnx_impl


if TYPE_CHECKING:
    from collections.abc import Sequence
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter`, `torch.onnx._internal.exporter._torchlib._torchlib_registry`; external imports: `collections.abc`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter`, `torch.onnx._internal.exporter._torchlib._torchlib_registry`；外部导入：`collections.abc`。

### Lines 15-25
```python


def call_op(
    op_type: str,
    *args: ir.Value | None,
    _num_outputs: int = 1,
    _domain: str = "",
    **kwargs: int | float | str | bool | ir.Graph | ir.TensorProtocol | Sequence[int],
) -> Sequence[ir.Value]:
    """Call an operator with the given arguments and keyword arguments.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `call_op`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`call_op`。

### Lines 26-32
```python
    Arguments are always inputs, while keyword arguments are attributes.
    """
    # This is a wrapper around the IR node creation that hooks into the _builder.OpRecorder
    # tracer so that all nodes created are recorded the same way as if we were to use
    # onnxscript ops directly.
    from onnxscript.ir import convenience as ir_convenience

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 33-40
```python
    if _core.current_tracer is None:
        raise AssertionError("current_tracer must be non-None")
    tracer = _core.current_tracer

    inputs = list(args)

    # If final inputs are None, strip them from the node inputs
    for input in reversed(inputs):
```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 41-48
```python
        if input is not None:
            break
        inputs.pop()

    # Construct and filter out None attributes
    attributes = [
        attr
        for attr in ir_convenience.convert_attributes(kwargs)
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 49-61
```python
        if attr.value is not None  # type: ignore[union-attr]
    ]
    tracer.nodes.append(
        node := ir.Node(
            _domain,
            op_type,
            inputs=inputs,
            attributes=attributes,
            num_outputs=_num_outputs,
            version=tracer.opset.version,
        )
    )
    return node.outputs
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 62-75
```python


@onnx_impl(torch.ops.higher_order.cond, no_compile=True)
def higher_order_cond(
    cond: ir.Value,
    true_func: ir.Function,
    false_func: ir.Function,
    inputs: Sequence[ir.Value],
) -> Sequence[ir.Value]:
    then_node = ir.Node(
        true_func.domain, true_func.name, inputs, num_outputs=len(true_func.outputs)
    )
    else_node = ir.Node(
        false_func.domain, false_func.name, inputs, num_outputs=len(false_func.outputs)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `higher_order_cond`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`higher_order_cond`。

### Lines 76-82
```python
    )

    # ONNX Runtime complains about duplicate output names if we don't rename them.
    # But the doesn't seem to be an actual violation of SSA form without renaming.
    for func_out, out in zip(true_func.outputs, then_node.outputs):
        out.name = f"{func_out.name}_{true_func.name}"
    for func_out, out in zip(false_func.outputs, else_node.outputs):
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 83-96
```python
        out.name = f"{func_out.name}_{false_func.name}"

    return call_op(
        "If",
        cond,
        _num_outputs=len(true_func.outputs),
        then_branch=ir.Graph(
            (), then_node.outputs, nodes=[then_node], name=true_func.name
        ),
        else_branch=ir.Graph(
            (), else_node.outputs, nodes=[else_node], name=false_func.name
        ),
    )

```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 97-110
```python

@onnx_impl(torch.ops.higher_order.scan, no_compile=True)
def higher_order_scan(
    body_func: ir.Function,
    scan_inits: Sequence[ir.Value],
    scan_inputs: Sequence[ir.Value],
    additional_inputs: Sequence[ir.Value] | None,
    reverse: bool = False,
) -> Sequence[ir.Value]:
    """https://github.com/pytorch/pytorch/blob/66ac724b56e6c37a534f3e066423ef2f41d7477f/torch/_higher_order_ops/scan.py#L109"""
    subgraph_inputs = [
        *[
            ir.Value(
                name=f"{inp.name}_{body_func.name}__subgraph_in",
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `higher_order_scan`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`higher_order_scan`。

### Lines 111-124
```python
                shape=inp.shape,
                type=ir.TensorType(inp.dtype),  # type: ignore[arg-type]
            )
            for inp in scan_inits
        ],
        *[
            ir.Value(
                name=f"{inp.name}_{body_func.name}__subgraph_in",
                # The iterated element passed to the body subgraph does not have a sequence axis.
                # It will have a rank one less than the rank of the corresponding scan_input.
                shape=ir.Shape(inp.shape[1:]),  # type: ignore[index]
                type=ir.TensorType(inp.dtype),  # type: ignore[arg-type]
            )
            for inp in scan_inputs
```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 125-137
```python
        ],
    ]
    # The one and only node in the Scan subgraph that calls the body_func
    body_node = ir.Node(
        body_func.domain,
        body_func.name,
        [
            *subgraph_inputs,
            *(additional_inputs or []),
        ],
        num_outputs=len(body_func.outputs),
    )

```
- EN: This block manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构。关键符号：无明显局部符号。

### Lines 138-144
```python
    # ONNX Runtime complains about duplicate output names if we don't rename them.
    # But the doesn't seem to be an actual violation of SSA form without renaming.
    for func_out, out in zip(body_func.outputs, body_node.outputs):
        out.name = f"{func_out.name}_{body_func.name}"

    n_outputs = len(body_func.outputs) - len(scan_inits)
    return call_op(
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 145-158
```python
        "Scan",
        *scan_inits,
        *scan_inputs,
        _num_outputs=len(body_func.outputs),
        body=ir.Graph(
            subgraph_inputs,
            body_node.outputs,
            nodes=[body_node],
            name=body_func.name,
        ),
        num_scan_inputs=len(scan_inputs),
        scan_input_directions=[(1 if reverse else 0) for _ in scan_inputs],
        scan_output_directions=[(1 if reverse else 0) for _ in range(n_outputs)],
    )
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 159-169
```python


@onnx_impl(torch.ops.higher_order.while_loop, no_compile=True)
def higher_order_while_loop(
    cond_func: ir.Function,
    body_func: ir.Function,
    carried_inputs: Sequence[ir.Value | int | float],
    additional_inputs: Sequence[ir.Value],
) -> Sequence[ir.Value]:
    """Implementation of while_loop using ONNX Loop operator.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `higher_order_while_loop`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`higher_order_while_loop`。

### Lines 170-177
```python
    The ONNX Loop operator implements a generic looping construct with the signature:
    Loop(M, cond, v_initial) -> (v_final_and_scan_outputs)

    For while_loop, we use:
    - M: None (no trip count limit)
    - cond: initial condition value
    - v_initial: carried_inputs (loop-carried dependencies)

```
- EN: This block advances ONNX export translation; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 178-188
```python
    The body subgraph takes:
    - iteration_num (int): current iteration number
    - condition_in (bool): loop continuation condition from previous iteration
    - loop_carried_dependencies: the carried values
    - additional_inputs: any additional inputs (constants/parameters)

    The body subgraph returns:
    - condition_out (bool): whether to continue looping
    - loop_carried_dependencies: updated carried values
    """

```
- EN: This block manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构。关键符号：无明显局部符号。

### Lines 189-195
```python
    # Create subgraph inputs for the Loop body
    # ONNX Loop body signature: (iter_num, cond_in, loop_carried_deps..., additional_inputs...)

    # Start subgraph construction
    subgraph_carried_inputs = []

    for i, inp in enumerate(carried_inputs):
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 196-209
```python
        if isinstance(inp, ir.Value):
            subgraph_carried_inputs.append(
                ir.Value(
                    name=f"{inp.name}_{body_func.name}__subgraph_in",
                    shape=inp.shape,
                    type=ir.TensorType(inp.dtype),  # type: ignore[arg-type]
                )
            )
        elif isinstance(inp, int):
            subgraph_carried_inputs.append(
                ir.Value(
                    name=f"carried_input_{i}_{body_func.name}__subgraph_in",
                    shape=ir.Shape([]),
                    type=ir.TensorType(ir.DataType.INT64),
```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；处理条件控制流。关键符号：无明显局部符号。

### Lines 210-223
```python
                )
            )
        elif isinstance(inp, float):
            subgraph_carried_inputs.append(
                ir.Value(
                    name=f"carried_input_{i}_{body_func.name}__subgraph_in",
                    shape=ir.Shape([]),
                    type=ir.TensorType(ir.DataType.FLOAT),
                )
            )
        else:
            raise NotImplementedError(
                f"Unsupported type for carried input: {type(inp)} ({inp}). "
                "Expected ir.Value, int, or float."
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；操作图状程序结构；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 224-237
```python
            )

    subgraph_inputs = [
        # Iteration number (int scalar, unused)
        ir.Value(
            name=f"iter_num_{body_func.name}",
            shape=ir.Shape([]),
            type=ir.TensorType(ir.DataType.INT64),
        ),
        # Condition input (bool scalar, unused)
        ir.Value(
            name=f"cond_in_{body_func.name}",
            shape=ir.Shape([]),
            type=ir.TensorType(ir.DataType.BOOL),
```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构。关键符号：无明显局部符号。

### Lines 238-251
```python
        ),
        # Loop-carried dependencies
        *subgraph_carried_inputs,
    ]

    # Create the combined body function that handles both condition and body logic
    # First, call the body function with the same inputs
    body_node = ir.Node(
        body_func.domain,
        body_func.name,
        [
            *subgraph_carried_inputs,  # carried inputs
            *additional_inputs,
        ],
```
- EN: This block manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构。关键符号：无明显局部符号。

### Lines 252-265
```python
        num_outputs=len(body_func.outputs),  # carried inputs
    )

    # Then call the condition function with carried inputs + additional inputs
    cond_node = ir.Node(
        cond_func.domain,
        cond_func.name,
        [
            *body_node.outputs,  # updated carried inputs from body
            *additional_inputs,
        ],
        num_outputs=len(cond_func.outputs),
    )

```
- EN: This block manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构。关键符号：无明显局部符号。

### Lines 266-272
```python
    if len(cond_func.outputs) != 1:
        raise AssertionError("Condition function must return a single boolean value.")

    # ONNX Runtime complains about duplicate output names if we don't rename them
    for func_out, out in zip(body_func.outputs, body_node.outputs):
        out.name = f"{func_out.name}_{body_func.name}"
    for func_out, out in zip(cond_func.outputs, cond_node.outputs):
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 273-281
```python
        out.name = f"{func_out.name}_{cond_func.name}"

    # The Loop body must return: (cond_out, loop_carried_deps...)
    # We use the condition output and the body outputs
    loop_body_outputs = [
        cond_node.outputs[0],  # condition output (bool)
        *body_node.outputs,  # updated carried inputs
    ]

```
- EN: This block manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构。关键符号：无明显局部符号。

### Lines 282-288
```python
    body_graph = ir.Graph(
        subgraph_inputs,
        loop_body_outputs,
        nodes=[body_node, cond_node],
        name=f"{body_func.name}_loop_body",
    )

```
- EN: This block manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构。关键符号：无明显局部符号。

### Lines 289-302
```python
    # End subgraph construction

    carried_inputs_values: list[ir.Value] = []
    for inp in carried_inputs:
        if isinstance(inp, ir.Value):
            carried_inputs_values.append(inp)
        elif isinstance(inp, int):
            const = call_op("Constant", value=ir.tensor(inp))[0]
            carried_inputs_values.append(const)
        elif isinstance(inp, float):
            const = call_op("Constant", value=ir.tensor(inp))[0]
            carried_inputs_values.append(const)
        else:
            raise NotImplementedError(
```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 303-315
```python
                f"Unsupported type for carried input: {type(inp)} ({inp}). "
                "Expected ir.Value, int, or float."
            )

    # Get initial condition by calling cond_func with initial inputs
    initial_outputs = call_op(
        cond_func.name,
        *carried_inputs_values,
        *additional_inputs,
        _num_outputs=len(cond_func.outputs),
        _domain=cond_func.domain,
    )

```
- EN: This block checks invariants or expected outcomes; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 316-329
```python
    if len(initial_outputs) != 1:
        raise AssertionError("Condition function must return a single boolean value.")

    # Create the Loop operator call
    # Loop(M, cond, v_initial) where M is empty (no trip count limit)
    loop_outputs = call_op(
        "Loop",
        # M (trip count) - empty string means no limit
        None,
        # cond - initial condition
        initial_outputs[0],
        # v_initial - carried inputs (loop-carried dependencies)
        *carried_inputs_values,
        _num_outputs=len(carried_inputs_values),
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 330-336
```python
        body=body_graph,
    )

    return loop_outputs


@onnx_impl(torch.ops.higher_order.invoke_subgraph, no_compile=True)
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 337-343
```python
def higher_order_invoke_subgraph(
    subgraph: ir.Function,
    identifier: str | None,
    *operands: ir.Value,
) -> Sequence[ir.Value]:
    """Export invoke_subgraph HOP by creating a direct function call.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `higher_order_invoke_subgraph`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`higher_order_invoke_subgraph`。

### Lines 344-355
```python
    This preserves the function as a separate entity in the ONNX graph
    instead of inlining it, which is the purpose of invoke_subgraph.

    Note: The onnxscript optimizer should be configured to not inline functions
    created by invoke_subgraph to preserve the intended structure.

    Args:
        subgraph: The function to invoke
        identifier: Optional identifier for the subgraph (used for caching in PyTorch,
            not needed for ONNX export as the function reference provides all necessary information)
        *operands: Input values to pass to the function

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 356-363
```python
    Returns:
        Sequence of output values from the function call
    """
    # This key can be used by downstream to avoid inlining
    subgraph.metadata_props["pkg.torch.ops.higher_order.invoke_subgraph.identifier"] = (
        str(identifier)
    )

```
- EN: This block manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构。关键符号：无明显局部符号。

### Lines 364-370
```python
    # Create the function call node
    return call_op(
        subgraph.name,
        *operands,
        _num_outputs=len(subgraph.outputs),
        _domain=subgraph.domain,
    )
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
- EN: Interop boundary — the implementation connects PyTorch semantics with ONNX model representation requirements.
  CN: 互操作边界——实现负责连接 PyTorch 语义与 ONNX 模型表示需求。
- EN: Module integration — the implementation coordinates neighboring Python modules and utility layers.
  CN: 模块集成——该实现会协调相邻的 Python 模块与工具层。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter`, `torch.onnx._internal.exporter._torchlib._torchlib_registry`
- External imports / 外部导入: `__future__`, `typing`, `collections.abc`, `onnxscript.ir`
- Representative symbols / 代表性符号: `call_op`, `higher_order_cond`, `higher_order_scan`, `higher_order_while_loop`, `higher_order_invoke_subgraph`
