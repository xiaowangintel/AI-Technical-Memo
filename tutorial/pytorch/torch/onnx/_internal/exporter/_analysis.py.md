# _analysis.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/_internal/exporter/_analysis.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal ONNX exporter logic for analysis, covering graph capture, translation, validation, or serialization.
- 用途 (CN): 实现与 analysis 相关的 ONNX 导出器内部逻辑，涵盖图捕获、转换、校验或序列化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
"""Compatibility analyzer for PyTorch models."""

# mypy: allow-untyped-defs

from __future__ import annotations

import dataclasses
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `__future__`, `dataclasses`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`__future__`, `dataclasses`。

### Lines 8-14
```python
import operator
import textwrap
import traceback
from collections import defaultdict
from typing import TYPE_CHECKING

import torch
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`; external imports: `operator`, `textwrap`, `traceback`, `collections`, `typing`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`；外部导入：`operator`, `textwrap`, `traceback`, `collections`, `typing`。

### Lines 15-21
```python
import torch._export.serde.schema
from torch.export import graph_signature
from torch.onnx._internal.exporter import _dispatching, _registration


if TYPE_CHECKING:
    import torch.fx
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch._export.serde.schema`, `torch.export`, `torch.onnx._internal.exporter`, `torch.fx`; external imports: none.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch._export.serde.schema`, `torch.export`, `torch.onnx._internal.exporter`, `torch.fx`；外部导入：无。

### Lines 22-35
```python


@dataclasses.dataclass
class ModelInfo:
    """Information about the model."""

    parameter_count: defaultdict[torch.dtype, int] = dataclasses.field(
        default_factory=lambda: defaultdict(int)
    )
    buffer_count: defaultdict[torch.dtype, int] = dataclasses.field(
        default_factory=lambda: defaultdict(int)
    )
    fx_node_count: int = 0
    fx_node_op_count: defaultdict[str, int] = dataclasses.field(
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `ModelInfo`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`ModelInfo`。

### Lines 36-49
```python
        default_factory=lambda: defaultdict(int)
    )
    fx_node_target_count: defaultdict[str, int] = dataclasses.field(
        default_factory=lambda: defaultdict(int)
    )
    dispatch_failures: list[tuple[torch.fx.Node, str]] = dataclasses.field(
        default_factory=list
    )
    inputs: dict[str, torch._export.serde.schema.TensorMeta] = dataclasses.field(
        default_factory=dict
    )
    outputs: dict[str, torch._export.serde.schema.TensorMeta] = dataclasses.field(
        default_factory=dict
    )
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：无明显局部符号。

### Lines 50-56
```python


def _count_weights(
    exported_program: torch.export.ExportedProgram,
) -> tuple[defaultdict[torch.dtype, int], defaultdict[torch.dtype, int]]:
    """Count the size of the parameters in the exported program."""

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_count_weights`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_count_weights`。

### Lines 57-63
```python
    parameter_count: defaultdict[torch.dtype, int] = defaultdict(int)
    buffer_count: defaultdict[torch.dtype, int] = defaultdict(int)
    for parameter in exported_program.parameters():
        dtype = parameter.dtype
        parameter_count[dtype] += parameter.numel()

    for buffer in exported_program.buffers():
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 64-70
```python
        dtype = buffer.dtype
        buffer_count[dtype] += buffer.numel()

    return parameter_count, buffer_count


def _format_model_info(model_info: ModelInfo) -> str:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_format_model_info`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_format_model_info`。

### Lines 71-78
```python
    """Format the information about the model."""
    lines = [
        textwrap.dedent(
            f"""\
            PyTorch ONNX Conversion Analysis

            ## Model Information

```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 79-92
```python
            The model has {sum(model_info.parameter_count.values())} parameters and {sum(model_info.buffer_count.values())} buffers (non-trainable parameters).
            Number of parameters per dtype:
            ```python
            {model_info.parameter_count}
            ```
            Number of buffers per dtype:
            ```python
            {model_info.buffer_count}
            ```
            """
        ),
        "Inputs:",
        *[f"- `{name}`: `{meta}`" for name, meta in model_info.inputs.items()],
        "",
```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 93-106
```python
        "Outputs:",
        *[f"- `{name}`: `{meta}`" for name, meta in model_info.outputs.items()],
        "",
        f"The FX graph has {model_info.fx_node_count} nodes in total. Number of FX nodes per op:",
    ]
    for op, count in model_info.fx_node_op_count.items():
        lines.append(f"- `{op}`: {count}")
    lines.append("\n")
    lines.append("Of the call_function nodes, the counts of operators used are:\n")
    sorted_targets = sorted(
        model_info.fx_node_target_count.items(),
        key=operator.itemgetter(1),
        reverse=True,
    )
```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 107-113
```python
    for target, count in sorted_targets:
        lines.append(f"- `{target}`: {count}")

    lines.append("")
    lines.append("## ONNX Conversion Information")
    lines.append("")

```
- EN: This block advances ONNX export translation; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 114-122
```python
    if model_info.dispatch_failures:
        lines.append(
            "The model contains operators the dispatcher could not find registered ONNX decompositions for. "
            "This may be due to missing implementations, decompositions not registered "
            "correctly, or a bug in the dispatcher."
        )
        lines.append("")
        lines.append("Errors grouped by operator:\n")

```
- EN: This block advances ONNX export translation; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 123-129
```python
        target_to_nodes = defaultdict(list)
        for node, _ in model_info.dispatch_failures:
            target_to_nodes[str(node.target)].append(node)

        target_to_messages = {}
        for node, message in model_info.dispatch_failures:
            if str(node.target) not in target_to_messages:
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 130-142
```python
                target_to_messages[str(node.target)] = message

        for target, nodes in sorted(
            target_to_nodes.items(), key=operator.itemgetter(0), reverse=True
        ):
            message = textwrap.indent(
                f"{target_to_messages[target]}. Example node: `{nodes[0].format_node()}`. All nodes: `{nodes}`",
                "    ",
            )
            lines.append(f"- `{target}`: {message}")
    else:
        lines.append("All operators in the model have registered ONNX decompositions.")

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 143-154
```python
    return "\n".join(lines)


def _get_io_specs(exported_program: torch.export.ExportedProgram) -> tuple[dict, dict]:
    """Get the input and output specs of the exported program."""

    nodes: dict[str, torch.fx.Node] = {
        node.name: node for node in exported_program.graph.nodes
    }
    user_inputs = [
        spec
        for spec in exported_program.graph_signature.input_specs
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_get_io_specs`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_get_io_specs`。

### Lines 155-164
```python
        if spec.kind == graph_signature.InputKind.USER_INPUT
    ]
    user_outputs = [
        spec
        for spec in exported_program.graph_signature.output_specs
        if spec.kind == graph_signature.OutputKind.USER_OUTPUT
    ]
    inputs: dict[str, torch._export.serde.schema.TensorMeta | str] = {}
    outputs: dict[str, torch._export.serde.schema.TensorMeta | str] = {}
    for spec in user_inputs:
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 165-171
```python
        inputs = _log_spec_into_io_specs(spec, nodes, inputs)
    for spec in user_outputs:
        outputs = _log_spec_into_io_specs(spec, nodes, outputs)
    return inputs, outputs


def _log_spec_into_io_specs(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_log_spec_into_io_specs`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_log_spec_into_io_specs`。

### Lines 172-178
```python
    spec: graph_signature.InputSpec,
    nodes: dict[str, torch.fx.Node],
    inputs_or_outputs: dict[str, torch._export.serde.schema.TensorMeta | str],
) -> dict[str, torch._export.serde.schema.TensorMeta | str]:
    # If dynamic is set to a constant input, it becomes a
    # symbolic argument, which is not a tensor.
    if isinstance(spec.arg, graph_signature.ConstantArgument):
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流。关键符号：无明显局部符号。

### Lines 179-192
```python
        # Constant input does not have tensor_meta.
        return inputs_or_outputs
    # Symbolic arguments are not tensors, so it does not have tensor_meta,
    # but we need to provide a string representation for them to inform users.
    name = spec.arg.name
    if isinstance(
        spec.arg,
        (
            graph_signature.SymIntArgument,
            graph_signature.SymFloatArgument,
            graph_signature.SymBoolArgument,
        ),
    ):
        argument_to_str: dict[type[graph_signature.ArgumentSpec], str] = {
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 193-201
```python
            graph_signature.SymIntArgument: "SymInt",
            graph_signature.SymFloatArgument: "SymFloat",
            graph_signature.SymBoolArgument: "SymBool",
        }
        inputs_or_outputs[name] = argument_to_str[type(spec.arg)]
        return inputs_or_outputs
    # FIXME: tensor_meta is None sometimes when the exported program still knows the shape/type
    inputs_or_outputs[name] = nodes[name].meta["tensor_meta"]
    return inputs_or_outputs
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；操作图状程序结构；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 202-209
```python


def _count_fx_targets(
    exported_program: torch.export.ExportedProgram,
) -> defaultdict[str, int]:
    """Count the number of targets for each node in the exported program."""
    fx_node_target_count: defaultdict[str, int] = defaultdict(int)
    for node in exported_program.graph.nodes:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_count_fx_targets`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_count_fx_targets`。

### Lines 210-223
```python
        if node.op == "call_function":
            fx_node_target_count[str(node.target)] += 1
    return fx_node_target_count


def analyze(
    exported_program: torch.export.ExportedProgram,
    registry: _registration.ONNXRegistry | None = None,
    file=None,
) -> None:
    """Analyze the compatibility of the exported program."""
    # Get basic information about the model
    model_info = ModelInfo()
    model_info.parameter_count, model_info.buffer_count = _count_weights(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `analyze`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`analyze`。

### Lines 224-231
```python
        exported_program
    )
    model_info.fx_node_count = len(exported_program.graph.nodes)
    model_info.fx_node_target_count = _count_fx_targets(exported_program)
    inputs, outputs = _get_io_specs(exported_program)
    model_info.inputs = inputs
    model_info.outputs = outputs

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：无明显局部符号。

### Lines 232-238
```python
    if registry is None:
        registry = _registration.ONNXRegistry.from_torchlib()

    # Try to find ops for every node in the graph
    for node in exported_program.graph.nodes:
        model_info.fx_node_op_count[node.op] += 1
        if node.op == "call_function":
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 239-248
```python
            try:
                onnx_function, message = _dispatching.dispatch(node, registry)
            except Exception as e:
                message = "Critical Error in dispatcher:\n"
                formatted_exception = "\n".join(
                    traceback.format_exception(type(e), e, e.__traceback__)
                )
                message += f"```pytb\n{formatted_exception}\n```\n"
                onnx_function = None
            if onnx_function is None:
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 249-255
```python
                model_info.dispatch_failures.append((node, message))

    # Print the results
    report = _format_model_info(model_info)
    print(report, file=file, flush=True)


```
- EN: This block manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构。关键符号：无明显局部符号。

### Lines 256-264
```python
def compare_ops(
    program_a: torch.export.ExportedProgram, program_b: torch.export.ExportedProgram
) -> tuple[set[str], set[str]]:
    """Compare and get unique ops in two exported programs.

    Args:
        program_a: The first exported program.
        program_b: The second exported program.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `compare_ops`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`compare_ops`。

### Lines 265-271
```python
    Returns:
        A tuple of two sets, where the first set contains the unique ops in the first program
        and the second set contains the unique ops in the second program.
    """
    program_a_ops = set(_count_fx_targets(program_a))
    program_b_ops = set(_count_fx_targets(program_b))
    return program_a_ops - program_b_ops, program_b_ops - program_a_ops
```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
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
- Internal imports / 内部导入: `torch`, `torch._export.serde.schema`, `torch.export`, `torch.onnx._internal.exporter`, `torch.fx`
- External imports / 外部导入: `__future__`, `dataclasses`, `operator`, `textwrap`, `traceback`, `collections`, `typing`
- Representative symbols / 代表性符号: `ModelInfo`, `_count_weights`, `_format_model_info`, `_get_io_specs`, `_log_spec_into_io_specs`, `_count_fx_targets`, `analyze`, `compare_ops`
