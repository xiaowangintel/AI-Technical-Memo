# jit_utils.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/_internal/torchscript_exporter/jit_utils.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal ONNX exporter logic for jit utils, covering graph capture, translation, validation, or serialization.
- 用途 (CN): 实现与 jit utils 相关的 ONNX 导出器内部逻辑，涵盖图捕获、转换、校验或序列化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
# mypy: allow-untyped-defs
"""Utilities for manipulating the torch.Graph object and the torchscript."""

from __future__ import annotations

import dataclasses
import re
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `__future__`, `dataclasses`, `re`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`__future__`, `dataclasses`, `re`。

### Lines 8-14
```python
import typing
from collections.abc import Iterable, Sequence
from typing import Any

import torch
from torch import _C
from torch.onnx._internal.torchscript_exporter import registration
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.onnx._internal.torchscript_exporter`; external imports: `typing`, `collections.abc`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.onnx._internal.torchscript_exporter`；外部导入：`typing`, `collections.abc`。

### Lines 15-21
```python
from torch.onnx._internal.torchscript_exporter._globals import GLOBALS


_ATTR_PATTERN = re.compile("^(.+)_(([ifstgz])|(ty))$")
_SKIP_NODE_ATTRIBUTES = {"inplace", "aten"}


```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.onnx._internal.torchscript_exporter._globals`; external imports: none.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.onnx._internal.torchscript_exporter._globals`；外部导入：无。

### Lines 22-28
```python
@dataclasses.dataclass
class GraphContext:
    """Extra context for symbolic functions with all methods from torch.Graph.

    NOTE: This class is not meant for external consumption. Please do not depend on
    it outside of torch.onnx as the interface may evolve.

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `GraphContext`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`GraphContext`。

### Lines 29-40
```python
    Attributes:
        graph: The _C.Graph being constructed.
        block: The current _C.Block being constructed.
        opset: The opset version.
        original_node: Current node that is being converted from.
        params_dict: Mapping from graph initializer name to IValue.
        env: Mapping from Torch domain graph Value to ONNX domain graph Value.
        values_in_env: Set of all values in env, for constant-time lookups.
        new_nodes: List that tracks all new nodes that are added (used to make
            sure metadata is propagated to all new nodes).
    """

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 41-49
```python
    graph: _C.Graph
    block: _C.Block
    opset: int
    original_node: _C.Node
    params_dict: dict[str, _C.IValue]
    env: dict[_C.Value, _C.Value]
    values_in_env: set[_C.Value]
    new_nodes: list[_C.Node] = dataclasses.field(default_factory=list)

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 50-63
```python
    # Relay methods from _C.Graph for compatibility with symbolic functions that expect
    # a _C.Graph
    def __getattr__(self, name: str) -> Any:
        return getattr(self.graph, name)

    def op(
        self,
        opname: str,
        *raw_args: torch.Tensor | _C.Value,
        outputs: int = 1,
        **kwargs,
    ):
        """Creates an ONNX operator "opname", taking "raw_args" as inputs and "kwargs" as attributes.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__getattr__`, `op`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__getattr__`, `op`。

### Lines 64-77
```python
        The set of operators and the inputs/attributes they take
        is documented at https://github.com/onnx/onnx/blob/master/docs/Operators.md

        Args:
            opname: The ONNX operator name, e.g., `Abs` or `Add`, or an operator qualified
                with a namespace, e.g., `aten::add`.
            raw_args: The inputs to the operator; usually provided
                as arguments to the `symbolic` definition.
            outputs: The number of outputs this operator returns.
                By default an operator is assumed to return a single output.
                If `outputs` is greater than one, this functions returns a tuple
                of output `Value`, representing each output of the ONNX operator
                in order.
            kwargs: The attributes of the ONNX operator, whose keys are named
```
- EN: This block advances ONNX export translation; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 78-84
```python
                according to the following convention: `alpha_f` indicates
                the `alpha` attribute with type `f`.  The valid type specifiers are
                `f` (float), `i` (int), `s` (string) or `t` (Tensor).  An attribute
                specified with type float accepts either a single float, or a
                list of floats (e.g., you would say `dims_i` for a `dims` attribute
                that takes a list of integers).

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 85-91
```python
        Returns:
            The value representing the single output of this operator (see the `outputs`
            keyword argument for multi-return nodes).
        """
        return _add_op(self, opname, *raw_args, outputs=outputs, **kwargs)

    def aten_op(self, operator: str, *args, overload_name: str = "", **kwargs):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `aten_op`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`aten_op`。

### Lines 92-103
```python
        """Generates an ONNX ATen op node.

        This function is for backward compatibility with the old symbolic functions.
        """
        return self.op(
            "aten::ATen",
            *args,
            operator_s=operator,
            overload_name_s=overload_name,
            **kwargs,
        )

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 104-116
```python
    # NOTE: For backward compatibility with the old symbolic functions.
    # We are probably going to remove this only after the fx exporter is established.
    at = aten_op

    def onnxscript_op(
        self,
        onnx_fn,
        *raw_args: torch.Tensor | _C.Value,
        outputs: int = 1,
        **kwargs,
    ):
        """Creates an ONNX operator from onnx-script function, taking "raw_args" as inputs and "kwargs" as attributes.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `onnxscript_op`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`onnxscript_op`。

### Lines 117-130
```python
        onnx-script repository: https://github.com/microsoft/onnx-script

        Args:
            onnx_fn: ONNXFunction from onnx-script; An example can be found at
                https://github.com/microsoft/onnx-script#example
            raw_args: The inputs to the operator; usually provided
                as arguments to the `symbolic` definition.
            outputs: The number of outputs this operator returns.
                By default an operator is assumed to return a single output.
                If `outputs` is greater than one, this functions returns a tuple
                of output `Value`, representing each output of the ONNX operator
                in order.
            kwargs: The attributes of the ONNX operator, whose keys are named
                according to the following convention: `alpha_f` indicates
```
- EN: This block advances ONNX export translation; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 131-144
```python
                the `alpha` attribute with type `f`.  The valid type specifiers are
                `f` (float), `i` (int), `s` (string) or `t` (Tensor).  An attribute
                specified with type float accepts either a single float, or a
                list of floats (e.g., you would say `dims_i` for a `dims` attribute
                that takes a list of integers).

        Returns:
            The value representing the single output of this operator (see the `outputs`
            keyword argument for multi-return nodes).
        """
        # NOTE(titaiwang): This is using class attributes, and it needs to be updated
        # if onnx-script makes any change on these.
        symbolic_name = f"{onnx_fn.opset.domain}::{onnx_fn.name}"
        opset_version = onnx_fn.opset.version
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: no dominant local symbols.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：无明显局部符号。

### Lines 145-151
```python

        registration.custom_onnx_symbolic(symbolic_name, opset_version)(onnx_fn)

        return _add_op(self, symbolic_name, *raw_args, outputs=outputs, **kwargs)


def add_op_with_blocks(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `add_op_with_blocks`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`add_op_with_blocks`。

### Lines 152-160
```python
    graph_context: GraphContext,
    opname: str,
    *inputs: _C.Value,
    outputs: int = 1,
    n_blocks: int = 1,
    **attributes,
) -> tuple[Any, tuple[GraphContext, ...], _C.Node]:
    """Creates an ONNX operator "opname", taking inputs and attributes.

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 161-173
```python
    Args:
        graph_context: The context for the current graph.
        opname: The ONNX operator name, e.g., `Abs` or `Add`, or an operator qualified
            with a namespace, e.g., `aten::add`.
        inputs: The inputs to the operator.
        outputs: The number of outputs this operator returns.
            By default an operator is assumed to return a single output.
            If `outputs` is greater than one, this functions returns a tuple
            of output `Value`, representing each output of the ONNX operator
            in order.
        n_blocks: The number of sub-blocks to create in the node.
        attributes: The attributes of the ONNX operator.

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 174-181
```python
    Returns:
        A tuple of (output_values, new_contexts, node) where:
            output_values: One or more output value of this operator
                (see the `outputs` keyword argument for multi-return nodes).
            new_contexts: A tuple of new graph contexts for each sub-block.
            node: The node representing the operator.
    """

```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units; returns results to callers or downstream stages; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元；向调用方或后续阶段返回结果；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 182-189
```python
    output_values = graph_context.op(opname, *inputs, outputs=outputs, **attributes)
    if isinstance(output_values, Sequence):
        node = output_values[0].node()
    else:
        node = output_values.node()

    new_contexts = []
    for _ in range(n_blocks):
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 190-196
```python
        new_block = node.addBlock()
        # Create shallow copy of the graph context and update the block
        new_context = dataclasses.replace(graph_context, block=new_block)
        new_contexts.append(new_context)

    return output_values, tuple(new_contexts), node

```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 197-206
```python

def _add_op(
    graph_context: GraphContext,
    opname: str,
    *args: torch.Tensor | _C.Value,
    outputs: int = 1,
    **kwargs,
):
    """Creates an ONNX operator "opname", taking "args" as inputs and attributes "kwargs".

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_add_op`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_add_op`。

### Lines 207-220
```python
    The set of operators and the inputs/attributes they take
    is documented at https://github.com/onnx/onnx/blob/master/docs/Operators.md

    Args:
        graph_context: The Torch Graph or Block.
        opname: The ONNX operator name, e.g., `Abs` or `Add`, or an operator qualified
            with a namespace, e.g., `aten::add`.
        args: The inputs to the operator; usually provided
            as arguments to the `symbolic` definition.
        outputs: The number of outputs this operator returns.
            By default an operator is assumed to return a single output.
            If `outputs` is greater than one, this functions returns a tuple
            of output `Value`, representing each output of the ONNX operator
            in order.
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 221-228
```python
        kwargs: The attributes of the ONNX operator, whose keys are named
            according to the following convention: `alpha_f` indicates
            the `alpha` attribute with type `f`.  The valid type specifiers are
            `f` (float), `i` (int), `s` (string) or `t` (Tensor).  An attribute
            specified with type float accepts either a single float, or a
            list of floats (e.g., you would say `dims_i` for a `dims` attribute
            that takes a list of integers).

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 229-238
```python
    Returns:
        (Union[_C.Value, Tuple[_C.Value, ...]])
        The value representing the single output of this operator (see the `outputs`
        keyword argument for multi-return nodes).
    """
    inputs = [_const_if_tensor(graph_context, arg) for arg in args]
    # Filter out None attributes, this can be convenient client side because
    # now they can pass through None attributes, and have them not show up
    attributes = {k: v for k, v in kwargs.items() if v is not None}

```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 239-252
```python
    if "::" not in opname:
        opname = "onnx::" + opname

    node = _create_node(
        graph_context.block,
        opname,
        inputs,
        attributes,
        params_dict=graph_context.params_dict,
        opset_version=graph_context.opset,
        n_outputs=outputs,
        shape_inference=GLOBALS.onnx_shape_inference,
    )
    graph_context.new_nodes.append(node)
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 253-259
```python

    if outputs == 1:
        return node.output()
    return tuple(node.outputs())


def _const_if_tensor(graph_context: GraphContext, arg):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_const_if_tensor`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_const_if_tensor`。

### Lines 260-266
```python
    if arg is None:
        return arg
    if isinstance(arg, _C.Value):
        return arg

    return _add_op(graph_context, "onnx::Constant", value_z=arg)

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 267-279
```python

def _create_node(
    graph_or_block: _C.Graph | _C.Block,
    domain_op: str,
    inputs: Sequence,
    attributes: dict,
    params_dict: dict,
    opset_version: int,
    n_outputs: int,
    shape_inference: bool = True,
) -> _C.Node:
    """Creates an node 'domain_op', taking inputs and attributes."""
    if isinstance(graph_or_block, _C.Graph):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_create_node`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_create_node`。

### Lines 280-286
```python
        graph = graph_or_block
        node = graph.create(domain_op, inputs, n_outputs)
        node = graph.insertNode(node)
    elif isinstance(graph_or_block, _C.Block):
        block = graph_or_block
        node = block.addNode(domain_op, inputs)

```
- EN: This block manipulates graph-like program structures; handles conditional control flow; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；处理条件控制流；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 287-293
```python
        # Block does not have create defined, so we need to add outputs manually
        if n_outputs > 1:
            for _ in range(1, n_outputs):
                node.addOutput()

    node_outputs = tuple(node.outputs())  # type: ignore[possibly-undefined]
    if len(node_outputs) != n_outputs:
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 294-301
```python
        raise AssertionError(
            f"len(node_outputs)={len(node_outputs)} != n_outputs={n_outputs}"
        )

    aten = domain_op.startswith("aten::")

    # Add all attributes
    for key, value in sorted(attributes.items()):
```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 302-310
```python
        if key in _SKIP_NODE_ATTRIBUTES:
            continue
        # pyrefly: ignore [unbound-name]
        _add_attribute(node, key, value, aten=aten)
    if shape_inference:
        # pyrefly: ignore [unbound-name]
        _C._jit_pass_onnx_node_shape_type_inference(node, params_dict, opset_version)
    # pyrefly: ignore [unbound-name]
    return node
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 311-317
```python


def _is_onnx_list(value):
    return isinstance(value, Iterable) and not isinstance(
        value, (str, bytes, torch.Tensor)
    )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_is_onnx_list`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_is_onnx_list`。

### Lines 318-324
```python

def _scalar(x: torch.Tensor):
    """Convert a scalar tensor into a Python value."""
    if x.numel() != 1:
        raise AssertionError(f"Expected numel() == 1, got {x.numel()}")
    return x[0]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_scalar`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_scalar`。

### Lines 325-335
```python

def _add_attribute(node: _C.Node, key: str, value: Any, aten: bool):
    r"""Initializes the right attribute based on type of value."""
    m = _ATTR_PATTERN.match(key)
    if m is None:
        raise ValueError(
            f"Invalid attribute specifier '{key}' names "
            "must be suffixed with type, e.g. 'dim_i' or 'dims_i'"
        )
    name, kind = m.group(1), m.group(2)
    if _is_onnx_list(value):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_add_attribute`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_add_attribute`。

### Lines 336-342
```python
        kind += "s"

    return getattr(node, f"{kind}_")(name, value)


def _is_tensor(x: _C.Value) -> bool:
    return x.type().isSubtypeOf(_C.TensorType.get())
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_is_tensor`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_is_tensor`。

### Lines 343-349
```python


def get_device_from_value(value: _C.Value) -> torch.device | None:
    if not _is_tensor(value):
        return None
    tensor_type = typing.cast(_C.TensorType, value.type())
    return tensor_type.device()
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_device_from_value`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_device_from_value`。

### Lines 350-357
```python


def parse_node_kind(kind: str) -> tuple[str, str]:
    """Parse node kind into domain and Op name."""
    if "::" not in kind:
        raise ValueError(f"Node kind: {kind} is invalid. '::' is not in node kind.")
    domain, opname = kind.split("::", 1)
    if "::" in opname:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `parse_node_kind`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`parse_node_kind`。

### Lines 358-364
```python
        raise ValueError(f"Node kind: {kind} is invalid. '::' should only appear once.")
    return domain, opname


def is_aten(domain: str) -> bool:
    """Check if the domain is official."""
    return domain == "aten"
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `is_aten`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`is_aten`。

### Lines 365-371
```python


def is_prim(domain: str) -> bool:
    """Check if the domain is official."""
    return domain == "prim"


```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `is_prim`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`is_prim`。

### Lines 372-374
```python
def is_onnx(domain: str) -> bool:
    """Check if the domain is official."""
    return domain == "onnx"
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `is_onnx`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`is_onnx`。


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
- Internal imports / 内部导入: `torch`, `torch.onnx._internal.torchscript_exporter`, `torch.onnx._internal.torchscript_exporter._globals`
- External imports / 外部导入: `__future__`, `dataclasses`, `re`, `typing`, `collections.abc`
- Representative symbols / 代表性符号: `_ATTR_PATTERN`, `_SKIP_NODE_ATTRIBUTES`, `GraphContext`, `add_op_with_blocks`, `_add_op`, `_const_if_tensor`, `_create_node`, `_is_onnx_list`, `_scalar`, `_add_attribute`, `...`
