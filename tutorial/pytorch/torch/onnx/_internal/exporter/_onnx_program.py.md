# _onnx_program.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/_internal/exporter/_onnx_program.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal ONNX exporter logic for onnx program, covering graph capture, translation, validation, or serialization.
- 用途 (CN): 实现与 onnx program 相关的 ONNX 导出器内部逻辑，涵盖图捕获、转换、校验或序列化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
# mypy: allow-untyped-defs
# mypy: disable-error-code="attr-defined,name-defined"
from __future__ import annotations


__all__ = ["ONNXProgram"]

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 8-14
```python
import contextlib
import copy
import gc
import logging
import os
import tempfile
import textwrap
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `contextlib`, `copy`, `gc`, `logging`, `os`, `tempfile`, `...`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`contextlib`, `copy`, `gc`, `logging`, `os`, `tempfile`, `...`。

### Lines 15-21
```python
import warnings
from collections.abc import Callable, Sequence
from typing import Any, TYPE_CHECKING

import torch
from torch.onnx._internal._lazy_import import onnx, onnx_ir as ir, onnxscript_apis
from torch.onnx._internal.exporter import _dynamic_shapes, _ir_passes
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter`; external imports: `warnings`, `collections.abc`, `typing`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter`；外部导入：`warnings`, `collections.abc`, `typing`。

### Lines 22-28
```python
from torch.utils import _pytree


# NOTE: DO NOT import module from torch.onnx._internal to this module in the global scope
# because ONNXProgram is exposed to the public API

if TYPE_CHECKING:
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流。关键符号：无明显局部符号。

### Lines 29-41
```python
    import numpy as np
    import onnxruntime as ort

_LARGE_MODEL_THRESHOLD = 1536 * 1024 * 1024  # 1536MB
_NP_UNSUPPORTED_DTYPES_8BIT = frozenset(
    {
        torch.float8_e4m3fn,
        torch.float8_e4m3fnuz,
        torch.float8_e5m2,
        torch.float8_e5m2fnuz,
    }
)

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 42-48
```python
logger = logging.getLogger(__name__)


def _ort_session_initializer(model: str | bytes) -> ort.InferenceSession:
    """Initialize an ONNX Runtime inference session with the specified model."""
    import onnxruntime as ort

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_ort_session_initializer`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_ort_session_initializer`。

### Lines 49-59
```python
    session_options = ort.SessionOptions()
    session_options.log_severity_level = 3  # 3: Error
    possible_providers = (
        "CUDAExecutionProvider",
        "CPUExecutionProvider",
    )
    available_providers = set(ort.get_available_providers())
    providers = [
        provider for provider in possible_providers if provider in available_providers
    ]
    return ort.InferenceSession(
```
- EN: This block reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 60-66
```python
        model, providers=providers, sess_options=session_options
    )


def _count_initializer_size(graph: ir.Graph) -> int:
    """Count the total size of the initializers in bytes."""
    return sum(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_count_initializer_size`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_count_initializer_size`。

### Lines 67-73
```python
        v.const_value.nbytes
        for v in graph.initializers.values()
        if v.const_value is not None
    )


@contextlib.contextmanager
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 74-87
```python
def _set_graph_outputs(
    graph: ir.Graph,
    outputs: list[ir.Value],
):
    """Temporarily set the outputs of the graph.

    Args:
        graph: The graph to set the outputs for.
        outputs: The outputs to set.
    """
    original_outputs = list(graph.outputs)
    graph.outputs.clear()
    graph.outputs.extend(outputs)
    try:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_set_graph_outputs`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_set_graph_outputs`。

### Lines 88-94
```python
        yield
    finally:
        graph.outputs.clear()
        graph.outputs.extend(original_outputs)


def _create_value_mapping(graph: ir.Graph) -> dict[str, ir.Value]:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_create_value_mapping`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_create_value_mapping`。

### Lines 95-101
```python
    """Return a dictionary mapping names to values in the graph.

    The mapping does not include values from subgraphs.

    Args:
        graph: The graph to extract the mapping from.

```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 102-108
```python
    Returns:
        A dictionary mapping names to values.
    """
    values: dict[str, ir.Value] = {}
    values.update(graph.initializers)
    # The names of the values can be None or "", which we need to exclude
    for input in graph.inputs:
```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 109-117
```python
        if not input.name:
            continue
        values[input.name] = input
    for node in graph:
        for value in node.outputs:
            if not value.name:
                continue
            values[value.name] = value
    return values
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 118-124
```python


def _to_numpy_array(input: torch.Tensor | int | float | str | bool) -> np.ndarray:
    if isinstance(input, (int, float, str, bool)):
        return ir.tensor(input).numpy()

    from torch.onnx._internal.exporter import _core
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_to_numpy_array`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_to_numpy_array`。

### Lines 125-131
```python

    return _core.TorchTensor(input).numpy()


def _from_numpy_array(array: np.ndarray) -> torch.Tensor:
    """Convert a NumPy array to a PyTorch tensor."""
    import ml_dtypes  # type: ignore[import-not-found]
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_from_numpy_array`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_from_numpy_array`。

### Lines 132-138
```python
    import numpy as np

    if array.dtype == ml_dtypes.bfloat16:
        return torch.from_numpy(array.view(np.uint16)).view(torch.bfloat16)
    if array.dtype == ml_dtypes.float8_e4m3fn:
        return torch.from_numpy(array.view(np.uint8)).view(torch.float8_e4m3fn)
    if array.dtype == ml_dtypes.float8_e4m3fnuz:
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 139-145
```python
        return torch.from_numpy(array.view(np.uint8)).view(torch.float8_e4m3fnuz)
    if array.dtype == ml_dtypes.float8_e5m2:
        return torch.from_numpy(array.view(np.uint8)).view(torch.float8_e5m2)
    if array.dtype == ml_dtypes.float8_e5m2fnuz:
        return torch.from_numpy(array.view(np.uint8)).view(torch.float8_e5m2fnuz)
    return torch.from_numpy(array)

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 146-152
```python

def _to_ort_value(input: torch.Tensor | int | float | str | bool) -> ort.OrtValue:
    """Convert a PyTorch tensor to an ONNX Runtime OrtValue."""
    import numpy as np
    import onnxruntime as ort

    from torch.onnx._internal.exporter import _core
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.onnx._internal.exporter`; external imports: `numpy`, `onnxruntime`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.onnx._internal.exporter`；外部导入：`numpy`, `onnxruntime`。

### Lines 153-162
```python

    if isinstance(input, (int, float, str, bool)):
        # Convert scalar values to OrtValue
        dtype_mapping = {
            int: np.int64,
            float: np.float32,
        }
        # pyrefly: ignore [bad-argument-type, no-matching-overload]
        dtype = dtype_mapping.get(type(input))
        return ort.OrtValue.ortvalue_from_numpy(np.array(input, dtype=dtype))
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 163-174
```python

    if input.dtype == torch.bfloat16 or input.dtype in _NP_UNSUPPORTED_DTYPES_8BIT:
        if hasattr(ort.OrtValue, "ortvalue_from_numpy_with_onnx_type"):
            # This requires ONNX Runtime 1.21 or newer
            if input.dtype == torch.bfloat16:
                uint_type = torch.uint16
            else:
                uint_type = torch.uint8
            onnx_type = _core.torch_dtype_to_onnx_dtype(input.dtype)
            # Make tensor contiguous to ensure view() works
            input = input.contiguous()
            return ort.OrtValue.ortvalue_from_numpy_with_onnx_type(
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 175-182
```python
                input.view(uint_type).numpy(force=True), onnx_element_type=onnx_type
            )
        raise RuntimeError(
            f"Failed to convert tensor of type '{input.dtype}' to OrtValue. "
            "Please ensure that ONNX Runtime is built with DLPack support or is the latest version"
        )
    # TODO(#151064): Use dlpack when ORT properly supports it
    return ort.OrtValue.ortvalue_from_numpy(input.numpy(force=True))
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; reports or normalizes error conditions; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；报告或规范化错误情况；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 183-195
```python


def _from_ort_value(value: ort.OrtValue) -> torch.Tensor:
    if value.element_type() in (
        ir.DataType.BFLOAT16,
        ir.DataType.FLOAT8E4M3FN,
        ir.DataType.FLOAT8E4M3FNUZ,
        ir.DataType.FLOAT8E5M2,
        ir.DataType.FLOAT8E5M2FNUZ,
    ):
        # This requires ONNX Runtime 1.21 or newer
        try:
            return torch.from_dlpack(value._get_c_value())
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_from_ort_value`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_from_ort_value`。

### Lines 196-202
```python
        except Exception as e:
            raise RuntimeError(
                "Failed to convert OrtValue to torch.Tensor. "
                "Please ensure that ONNX Runtime is built with DLPack support or is the latest version"
            ) from e
    return torch.from_numpy(value.numpy())

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; reports or normalizes error conditions; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；报告或规范化错误情况；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 203-211
```python

class ONNXProgram:
    """A class to represent an ONNX program that is callable with torch tensors.

    Attributes:
        model: The ONNX model as an ONNX IR model object.
        exported_program: The exported program that produced the ONNX model.
    """

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `ONNXProgram`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`ONNXProgram`。

### Lines 212-225
```python
    def __init__(
        self, model: ir.Model, exported_program: torch.export.ExportedProgram | None
    ) -> None:
        """Initialize the ONNX program with the specified model and exported program.
        Args:
            model: The ONNX model.
            exported_program: The exported program that produced the ONNX model. Optional.
        """
        self.model: ir.Model = model
        self.exported_program = exported_program
        self._inference_session: ort.InferenceSession | None = None
        self._tempdir: tempfile.TemporaryDirectory | None = None
        # Strategy used to capture the exported program
        self._capture_strategy: str | None = None
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`。

### Lines 226-237
```python

    def __repr__(self) -> str:
        return f"""\
ONNXProgram(
    model=
{textwrap.indent(str(self.model), " " * 8)}
    ,
    exported_program=
{textwrap.indent(str(self.exported_program), " " * 8)}
)
"""

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__repr__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__repr__`。

### Lines 238-244
```python
    def __call__(self, *args, **kwargs) -> Sequence[torch.Tensor]:
        """Run the ONNX model with the same arguments you would provide to the GraphModule."""
        import onnxruntime as ort

        flatten_args = _process_args(args, kwargs)

        if self._inference_session is None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__call__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__call__`。

### Lines 245-252
```python
            self.initialize_inference_session()

        if self._inference_session is None:
            raise AssertionError("_inference_session must be non-None")

        ort_input = {
            k.name: _to_ort_value(v)
            for k, v in zip(self.model.graph.inputs, flatten_args)
```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 253-261
```python
        }
        run_options = ort.RunOptions()
        run_options.log_severity_level = 3  # 3: Error
        logger.debug("Running the inference session with %s arguments.", len(ort_input))
        outputs = self._inference_session.run_with_ort_values(
            None, ort_input, run_options=run_options
        )
        logger.debug("Inference session run completed.")
        return tuple(_from_ort_value(output) for output in outputs)
```
- EN: This block reports or normalizes error conditions; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 262-268
```python

    def call_reference(self, *args, **kwargs) -> Sequence[torch.Tensor]:
        """Run the ONNX model using the reference backend."""
        import onnx.reference

        evaluator = onnx.reference.ReferenceEvaluator(self.model_proto)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `call_reference`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`call_reference`。

### Lines 269-275
```python
        flatten_args = _process_args(args, kwargs)
        ref_input = {
            k.name: _to_numpy_array(v)
            for k, v in zip(self.model.graph.inputs, flatten_args)
        }
        outputs = evaluator.run(None, ref_input)  # type: ignore[arg-type]
        if not isinstance(outputs, Sequence):
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 276-283
```python
            raise AssertionError(f"Expected Sequence, got {type(outputs)}")
        return tuple(_from_numpy_array(output) for output in outputs)

    def compute_values(
        self, value_names: Sequence[str], args=(), kwargs=None
    ) -> Sequence[torch.Tensor]:
        """Compute the values of the specified names in the ONNX model.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `compute_values`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`compute_values`。

### Lines 284-293
```python
        This method is used to compute the values of the specified names in the ONNX model.
        The values are returned as a dictionary mapping names to tensors.

        Args:
            value_names: The names of the values to compute.

        Returns:
            A dictionary mapping names to tensors.
        """
        if kwargs is None:
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。

### Lines 294-307
```python
            kwargs = {}
        self.release()
        values = _create_value_mapping(self.model.graph)
        for name in value_names:
            if name not in values:
                raise ValueError(
                    f"Value '{name}' not found in the model. "
                    "Please provide a valid value name."
                )
        temporary_outputs = [values[name] for name in value_names]
        with _set_graph_outputs(self.model.graph, temporary_outputs):
            try:
                result = self(*args, **kwargs)
            finally:
```
- EN: This block manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 308-314
```python
                self.release()
        return result

    @property
    def model_proto(self) -> onnx.ModelProto:
        """Return the ONNX ``ModelProto`` object."""
        return ir.serde.serialize_model(self.model)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `model_proto`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`model_proto`。

### Lines 315-323
```python

    def optimize(self) -> None:
        """Optimize the ONNX model.

        This method optimizes the ONNX model by performing constant folding and
        eliminating redundancies in the graph. The optimization is done in-place.
        """
        self.model = onnxscript_apis.optimize(self.model)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `optimize`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`optimize`。

### Lines 324-333
```python
    def save(
        self,
        destination: str | os.PathLike,
        *,
        include_initializers: bool = True,
        keep_initializers_as_inputs: bool = False,
        external_data: bool | None = None,
    ) -> None:
        """Save the ONNX model to the specified destination.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `save`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`save`。

### Lines 334-347
```python
        When ``external_data`` is ``True`` or the model is larger than 2GB,
        the weights are saved as external data in a separate file.

        Initializer (model weights) serialization behaviors:

        * ``include_initializers=True``, ``keep_initializers_as_inputs=False`` (default):
          The initializers are included in the saved model.
        * ``include_initializers=True``, ``keep_initializers_as_inputs=True``:
          The initializers are included in the saved model and kept as model inputs.
          Choose this option if you want the ability to override the model weights
          during inference.
        * ``include_initializers=False``, ``keep_initializers_as_inputs=False``:
          The initializers are not included in the saved model and are not listed
          as model inputs. Choose this option if you want to attach the initializers
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 348-361
```python
          to the ONNX model in a separate, post-processing, step.
        * ``include_initializers=False``, ``keep_initializers_as_inputs=True``:
          The initializers are not included in the saved model but are listed as model
          inputs. Choose this option if you want to supply the initializers during
          inference and want to minimize the size of the saved model.

        Args:
            destination: The path to save the ONNX model to.
            include_initializers: Whether to include the initializers in the saved model.
            keep_initializers_as_inputs: Whether to keep the initializers as inputs in the saved model.
                If `True`, the initializers are added as inputs to the model which means they can be overwritten.
                by providing the initializers as model inputs.
            external_data: Whether to save the weights as external data in a separate file.

```
- EN: This block advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。

### Lines 362-369
```python
        Raises:
            TypeError: If ``external_data`` is ``True`` and ``destination`` is not a file path.
        """
        original_initializers = copy.copy(self.model.graph.initializers)
        original_inputs = copy.copy(self.model.graph.inputs)

        # Adjust the model based on options
        if not include_initializers:
```
- EN: This block manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 370-376
```python
            self.model.graph.initializers.clear()
        if keep_initializers_as_inputs:
            self.model.graph.inputs.extend(original_initializers.values())  # type: ignore[arg-type]

        try:
            # Save the model to disk
            if (
```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：无明显局部符号。

### Lines 377-385
```python
                external_data
                or _count_initializer_size(self.model.graph) > _LARGE_MODEL_THRESHOLD
            ):
                onnxscript_apis.save_model_with_external_data(self.model, destination)
            else:
                ir.save(self.model, destination)
        finally:
            # Revert the changes to the model
            if not include_initializers:
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流。关键符号：无明显局部符号。

### Lines 386-393
```python
                self.model.graph.initializers.update(original_initializers)
            if keep_initializers_as_inputs:
                self.model.graph.inputs.clear()
                self.model.graph.inputs.extend(original_inputs)

    def apply_weights(self, state_dict: dict[str, torch.Tensor]) -> None:
        """Apply the weights from the specified state dict to the ONNX model.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `apply_weights`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`apply_weights`。

### Lines 394-400
```python
        Use this method to replace FakeTensors or other weights.

        Args:
            state_dict: The state dict containing the weights to apply to the ONNX model.
        """
        from torch.onnx._internal.exporter import _core

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 401-412
```python
        for name, tensor in state_dict.items():
            if name in self.model.graph.initializers:
                self.model.graph.initializers[name].const_value = _core.TorchTensor(
                    tensor, name
                )
            else:
                warnings.warn(
                    f"Weight '{name}' not found in the model. Skipped applying.",
                    category=torch.onnx.errors.OnnxExporterWarning,
                    stacklevel=1,
                )

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 413-420
```python
    def initialize_inference_session(
        self,
        initializer: Callable[
            [str | bytes], ort.InferenceSession
        ] = _ort_session_initializer,
    ) -> None:
        """Initialize the ONNX Runtime inference session.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `initialize_inference_session`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`initialize_inference_session`。

### Lines 421-428
```python
        Args:
            initializer: The function to initialize the ONNX Runtime inference
                session with the specified model. By default, it uses the
                :func:`_ort_session_initializer` function.
        """
        # TODO(justinchuby): Allow different inference options
        logger.debug("Initializing the inference session.")
        if (
```
- EN: This block advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。

### Lines 429-439
```python
            byte_size := _count_initializer_size(self.model.graph)
        ) > _LARGE_MODEL_THRESHOLD:
            logger.debug("The model initializers is larger than 1.5GB (%s).", byte_size)
            # Save the model to a temporary file if too large
            self._tempdir = tempfile.TemporaryDirectory(ignore_cleanup_errors=True)
            model_path = os.path.join(self._tempdir.name, "model.onnx")
            self.save(model_path, external_data=True)
            model = model_path
        else:
            model = self.model_proto.SerializeToString()  # type: ignore[assignment]

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 440-449
```python
        self._inference_session = initializer(model)
        logger.debug("Inference session initialized.")

    def release(self) -> None:
        """Release the inference session.

        You may call this method to release the resources used by the inference session.
        """
        # Release the inference session first so that the model file can be deleted
        if self._inference_session is not None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `release`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`release`。

### Lines 450-456
```python
            self._inference_session = None
        gc.collect()
        if self._tempdir is not None:
            self._tempdir.cleanup()
            self._tempdir = None

    def rename_axes(self, rename_mapping: dict[str | ir.SymbolicDim, str]) -> None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `rename_axes`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`rename_axes`。

### Lines 457-468
```python
        """Rename axes in a model according to the specified rename mapping.

        Example::

            batch = onnx_program.model.graph.inputs[0].shape[0]
            seq_len = onnx_program.model.graph.inputs[0].shape[2]
            rename_mapping = {
                batch: "batch",
                seq_len: "seq_len",
            }
            onnx_program.rename_axes(rename_mapping)

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：无明显局部符号。

### Lines 469-476
```python
        Args:
            rename_mapping: A dictionary mapping old axes to new axis names.
                Keys can be either:

                * String axis names (e.g., "s1", "s2")
                * SymbolicDim objects obtained from the model
                  (e.g., onnx_program.model.graph.inputs[0].shape[0])

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：无明显局部符号。

### Lines 477-490
```python
                Values must be strings representing the new axis names.
        """
        _ir_passes.rename_axis(self.model, rename_mapping)

    def _rename_dynamic_axes(
        self,
        dynamic_shapes: dict[str, Any] | tuple[Any, ...] | list[Any],
    ) -> None:
        """Rename dynamic axes in a model according to the specified dynamic_axes names."""
        rename_mapping = _dynamic_shapes.create_rename_mapping(
            self.model.graph.inputs, dynamic_shapes
        )
        _ir_passes.rename_axis(self.model, rename_mapping)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_rename_dynamic_axes`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_rename_dynamic_axes`。

### Lines 491-497
```python

def _process_args(args, kwargs) -> tuple[torch.Tensor, ...]:
    """Process input arguments for the ONNX model."""
    args = _flatten_inputs(args, kwargs)
    args = _remove_none_from_inputs(args)
    args = _convert_complex_to_real_representation(args)
    return args
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_process_args`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_process_args`。

### Lines 498-504
```python


def _flatten_inputs(model_args, model_kwargs):
    flattened_args, _ = _pytree.tree_flatten((model_args, model_kwargs))
    return flattened_args


```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_flatten_inputs`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_flatten_inputs`。

### Lines 505-511
```python
def _remove_none_from_inputs(model_args):
    return tuple(arg for arg in model_args if arg is not None)


def _convert_complex_to_real_representation(model_args):
    """Convert complex dtype tensors to real representation tensors.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_remove_none_from_inputs`, `_convert_complex_to_real_representation`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_remove_none_from_inputs`, `_convert_complex_to_real_representation`。

### Lines 512-518
```python
    ONNX does not support complex dtype tensors. Thus, we convert complex dtype tensors
    to real representation tensors (i.e., float dtype tensors with an extra dimension
    representing the real and imaginary parts of the complex number).
    """
    return tuple(
        torch.view_as_real(arg.resolve_conj())
        if isinstance(arg, torch.Tensor) and arg.is_complex()
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 519-521
```python
        else arg
        for arg in model_args
    )
```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Backend coverage — the file contains backend-specific behavior or validation paths.
  CN: 后端覆盖——该文件包含特定后端的行为或校验路径。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter`, `torch.utils`
- External imports / 外部导入: `__future__`, `contextlib`, `copy`, `gc`, `logging`, `os`, `tempfile`, `textwrap`, `warnings`, `collections.abc`, `...`
- Representative symbols / 代表性符号: `_LARGE_MODEL_THRESHOLD`, `_NP_UNSUPPORTED_DTYPES_8BIT`, `_ort_session_initializer`, `_count_initializer_size`, `_set_graph_outputs`, `_create_value_mapping`, `_to_numpy_array`, `_from_numpy_array`, `_to_ort_value`, `_from_ort_value`, `...`
