# onnx_proto_utils.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/_internal/torchscript_exporter/onnx_proto_utils.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal ONNX exporter logic for onnx proto utils, covering graph capture, translation, validation, or serialization.
- 用途 (CN): 实现与 onnx proto utils 相关的 ONNX 导出器内部逻辑，涵盖图捕获、转换、校验或序列化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
# mypy: allow-untyped-defs
"""Utilities for manipulating the onnx and onnx-script dependencies and ONNX proto."""

from __future__ import annotations

import glob
import os
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `__future__`, `glob`, `os`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`__future__`, `glob`, `os`。

### Lines 8-14
```python
import shutil
from typing import Any, TYPE_CHECKING

import torch
import torch.serialization
from torch.onnx import errors
from torch.onnx._internal.torchscript_exporter import jit_utils, registration
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.serialization`, `torch.onnx`, `torch.onnx._internal.torchscript_exporter`; external imports: `shutil`, `typing`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.serialization`, `torch.onnx`, `torch.onnx._internal.torchscript_exporter`；外部导入：`shutil`, `typing`。

### Lines 15-21
```python


if TYPE_CHECKING:
    import io
    from collections.abc import Mapping


```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `io`, `collections.abc`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`io`, `collections.abc`。

### Lines 22-29
```python
def export_as_test_case(
    model_bytes: bytes, inputs_data, outputs_data, name: str, dir: str
) -> str:
    """Export an ONNX model as a self contained ONNX test case.

    The test case contains the model and the inputs/outputs data. The directory structure
    is as follows:

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `export_as_test_case`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`export_as_test_case`。

### Lines 30-38
```python
    dir
    \u251c\u2500\u2500 test_<name>
    \u2502   \u251c\u2500\u2500 model.onnx
    \u2502   \u2514\u2500\u2500 test_data_set_0
    \u2502       \u251c\u2500\u2500 input_0.pb
    \u2502       \u251c\u2500\u2500 input_1.pb
    \u2502       \u251c\u2500\u2500 output_0.pb
    \u2502       \u2514\u2500\u2500 output_1.pb

```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 39-48
```python
    Args:
        model_bytes: The ONNX model in bytes.
        inputs_data: The inputs data, nested data structure of numpy.ndarray.
        outputs_data: The outputs data, nested data structure of numpy.ndarray.

    Returns:
        The path to the test case directory.
    """
    try:
        import onnx
```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 49-62
```python
    except ImportError as exc:
        raise ImportError(
            "Export test case to ONNX format failed: Please install ONNX."
        ) from exc

    test_case_dir = os.path.join(dir, "test_" + name)
    os.makedirs(test_case_dir, exist_ok=True)
    _export_file(
        model_bytes,
        os.path.join(test_case_dir, "model.onnx"),
        {},
    )
    data_set_dir = os.path.join(test_case_dir, "test_data_set_0")
    if os.path.exists(data_set_dir):
```
- EN: This block advances ONNX export translation; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 63-70
```python
        shutil.rmtree(data_set_dir)
    os.makedirs(data_set_dir)

    proto = onnx.load_model_from_string(model_bytes)  # type: ignore[attr-defined]

    for i, (input_proto, input) in enumerate(zip(proto.graph.input, inputs_data)):
        export_data(input, input_proto, os.path.join(data_set_dir, f"input_{i}.pb"))
    for i, (output_proto, output) in enumerate(zip(proto.graph.output, outputs_data)):
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 71-78
```python
        export_data(output, output_proto, os.path.join(data_set_dir, f"output_{i}.pb"))

    return test_case_dir


def load_test_case(dir: str) -> tuple[bytes, Any, Any]:
    """Load a self contained ONNX test case from a directory.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `load_test_case`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`load_test_case`。

### Lines 79-90
```python
    The test case must contain the model and the inputs/outputs data. The directory structure
    should be as follows:

    dir
    \u251c\u2500\u2500 test_<name>
    \u2502   \u251c\u2500\u2500 model.onnx
    \u2502   \u2514\u2500\u2500 test_data_set_0
    \u2502       \u251c\u2500\u2500 input_0.pb
    \u2502       \u251c\u2500\u2500 input_1.pb
    \u2502       \u251c\u2500\u2500 output_0.pb
    \u2502       \u2514\u2500\u2500 output_1.pb

```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 91-100
```python
    Args:
        dir: The directory containing the test case.

    Returns:
        model_bytes: The ONNX model in bytes.
        inputs: the inputs data, mapping from input name to numpy.ndarray.
        outputs: the outputs data, mapping from output name to numpy.ndarray.
    """
    try:
        import onnx
```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 101-109
```python
        from onnx import numpy_helper  # type: ignore[attr-defined]
    except ImportError as exc:
        raise ImportError(
            "Load test case from ONNX format failed: Please install ONNX."
        ) from exc

    with open(os.path.join(dir, "model.onnx"), "rb") as f:
        model_bytes = f.read()

```
- EN: This block advances ONNX export translation; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 110-119
```python
    test_data_dir = os.path.join(dir, "test_data_set_0")

    inputs = {}
    input_files = glob.glob(os.path.join(test_data_dir, "input_*.pb"))
    for input_file in input_files:
        tensor = onnx.load_tensor(input_file)  # type: ignore[attr-defined]
        inputs[tensor.name] = numpy_helper.to_array(tensor)
    outputs = {}
    output_files = glob.glob(os.path.join(test_data_dir, "output_*.pb"))
    for output_file in output_files:
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 120-126
```python
        tensor = onnx.load_tensor(output_file)  # type: ignore[attr-defined]
        outputs[tensor.name] = numpy_helper.to_array(tensor)

    return model_bytes, inputs, outputs


def export_data(data, value_info_proto, f: str) -> None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `export_data`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`export_data`。

### Lines 127-136
```python
    """Export data to ONNX protobuf format.

    Args:
        data: The data to export, nested data structure of numpy.ndarray.
        value_info_proto: The ValueInfoProto of the data. The type of the ValueInfoProto
            determines how the data is stored.
        f: The file to write the data to.
    """
    try:
        from onnx import numpy_helper  # type: ignore[attr-defined]
```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 137-143
```python
    except ImportError as exc:
        raise ImportError(
            "Export data to ONNX format failed: Please install ONNX."
        ) from exc

    with open(f, "wb") as opened_file:
        if value_info_proto.type.HasField("map_type"):
```
- EN: This block advances ONNX export translation; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 144-157
```python
            opened_file.write(
                numpy_helper.from_dict(data, value_info_proto.name).SerializeToString()
            )
        elif value_info_proto.type.HasField("sequence_type"):
            opened_file.write(
                numpy_helper.from_list(data, value_info_proto.name).SerializeToString()
            )
        elif value_info_proto.type.HasField("optional_type"):
            opened_file.write(
                numpy_helper.from_optional(
                    data, value_info_proto.name
                ).SerializeToString()
            )
        else:
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 158-164
```python
            if not value_info_proto.type.HasField("tensor_type"):
                raise AssertionError("Expected tensor_type field to be set")
            opened_file.write(
                numpy_helper.from_array(data, value_info_proto.name).SerializeToString()
            )


```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 165-171
```python
def _export_file(
    model_bytes: bytes,
    f: io.BytesIO | str,
    export_map: Mapping[str, bytes],
) -> None:
    """export/write model bytes into directory/protobuf/zip"""
    if len(export_map) != 0:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_export_file`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_export_file`。

### Lines 172-183
```python
        raise AssertionError(f"export_map must be empty, got {len(export_map)} items")
    with torch.serialization._open_file_like(f, "wb") as opened_file:
        opened_file.write(model_bytes)


def _add_onnxscript_fn(
    model_bytes: bytes,
    custom_opsets: Mapping[str, int],
) -> bytes:
    """Insert model-included custom onnx-script function into ModelProto"""
    try:
        import onnx
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_add_onnxscript_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_add_onnxscript_fn`。

### Lines 184-193
```python
    except ImportError as e:
        raise errors.OnnxExporterError("Module onnx is not installed!") from e

    # For > 2GB model, onnx.load_fromstring would fail. However, because
    # in _export_onnx, the tensors should be saved separately if the proto
    # size > 2GB, and if it for some reason did not, the model would fail on
    # serialization anyway in terms of the protobuf limitation. So we don't
    # need to worry about > 2GB model getting here.
    model_proto = onnx.load_model_from_string(model_bytes)  # type: ignore[attr-defined]

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 194-202
```python
    # Iterate graph nodes to insert only the included custom
    # function_proto into model_proto
    onnx_function_list = []  # type: ignore[var-annotated]
    included_node_func: set[str] = set()
    # onnx_function_list and included_node_func are expanded in-place
    _find_onnxscript_op(
        model_proto.graph, included_node_func, custom_opsets, onnx_function_list
    )

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：无明显局部符号。

### Lines 203-209
```python
    if onnx_function_list:
        model_proto.functions.extend(onnx_function_list)
        model_bytes = model_proto.SerializeToString()
    return model_bytes


def _find_onnxscript_op(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_find_onnxscript_op`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_find_onnxscript_op`。

### Lines 210-216
```python
    graph_proto,
    included_node_func: set[str],
    custom_opsets: Mapping[str, int],
    onnx_function_list: list,
):
    """Recursively iterate ModelProto to find ONNXFunction op as it may contain control flow Op."""
    for node in graph_proto.node:
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 217-227
```python
        node_kind = node.domain + "::" + node.op_type
        # Recursive needed for control flow nodes: IF/Loop which has inner graph_proto
        for attr in node.attribute:
            if attr.g is not None:
                _find_onnxscript_op(
                    attr.g, included_node_func, custom_opsets, onnx_function_list
                )
        # Only custom Op with ONNX function and aten with symbolic_fn should be found in registry
        onnx_function_group = registration.registry.get_function_group(node_kind)
        # Ruled out corner cases: onnx/prim in registry
        if (
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 228-237
```python
            node.domain
            and not jit_utils.is_aten(node.domain)
            and not jit_utils.is_prim(node.domain)
            and not jit_utils.is_onnx(node.domain)
            and onnx_function_group is not None
            and node_kind not in included_node_func
        ):
            specified_version = custom_opsets.get(node.domain, 1)
            onnx_fn = onnx_function_group.get(specified_version)
            if onnx_fn is not None:
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流。关键符号：无明显局部符号。

### Lines 238-248
```python
                if hasattr(onnx_fn, "to_function_proto"):
                    onnx_function_proto = onnx_fn.to_function_proto()  # type: ignore[attr-defined]
                    onnx_function_list.append(onnx_function_proto)
                    included_node_func.add(node_kind)
                continue

            raise errors.UnsupportedOperatorError(
                node_kind,
                specified_version,
                onnx_function_group.get_min_supported()
                if onnx_function_group
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 249-251
```python
                else None,
            )
    return onnx_function_list, included_node_func
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；向调用方或后续阶段返回结果。关键符号：无明显局部符号。


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
- EN: Interop boundary — the implementation connects PyTorch semantics with ONNX model representation requirements.
  CN: 互操作边界——实现负责连接 PyTorch 语义与 ONNX 模型表示需求。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.serialization`, `torch.onnx`, `torch.onnx._internal.torchscript_exporter`
- External imports / 外部导入: `__future__`, `glob`, `os`, `shutil`, `typing`, `io`, `collections.abc`, `onnx`
- Representative symbols / 代表性符号: `export_as_test_case`, `load_test_case`, `export_data`, `_export_file`, `_add_onnxscript_fn`, `_find_onnxscript_op`
