# _verification.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/_internal/exporter/_verification.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal ONNX exporter logic for verification, covering graph capture, translation, validation, or serialization.
- 用途 (CN): 实现与 verification 相关的 ONNX 导出器内部逻辑，涵盖图捕获、转换、校验或序列化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8
```python
from __future__ import annotations


__all__ = [
    "VerificationInfo",
    "verify_onnx_program",
]

```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：无明显局部符号。

### Lines 9-15
```python
import dataclasses
import logging
import math
from typing import Any, TYPE_CHECKING

import torch
from torch.utils import _pytree
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.utils`; external imports: `dataclasses`, `logging`, `math`, `typing`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.utils`；外部导入：`dataclasses`, `logging`, `math`, `typing`。

### Lines 16-22
```python


if TYPE_CHECKING:
    from torch.onnx._internal._lazy_import import onnx_ir as ir
    from torch.onnx._internal.exporter import _onnx_program


```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter`; external imports: none.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter`；外部导入：无。

### Lines 23-29
```python
logger = logging.getLogger(__name__)


@dataclasses.dataclass
class VerificationInfo:
    """Verification information for a value in the ONNX program.

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `VerificationInfo`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`VerificationInfo`。

### Lines 30-36
```python
    This class contains the maximum absolute difference, maximum relative difference,
    and histograms of absolute and relative differences between the expected and actual
    values. It also includes the expected and actual data types.

    The histograms are represented as tuples of tensors, where the first tensor is the
    histogram counts and the second tensor is the bin edges.

```
- EN: Declares C++ types or containers needed in this file. Representative symbols: no dominant local symbols.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：无明显局部符号。

### Lines 37-48
```python
    Attributes:
        name: The name of the value (output or intermediate).
        max_abs_diff: The maximum absolute difference between the expected and actual values.
        max_rel_diff: The maximum relative difference between the expected and actual values.
        abs_diff_hist: A tuple of tensors representing the histogram of absolute differences.
            The first tensor is the histogram counts and the second tensor is the bin edges.
        rel_diff_hist: A tuple of tensors representing the histogram of relative differences.
            The first tensor is the histogram counts and the second tensor is the bin edges.
        expected_dtype: The data type of the expected value.
        actual_dtype: The data type of the actual value.
    """

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 49-58
```python
    name: str
    max_abs_diff: float
    max_rel_diff: float
    abs_diff_hist: tuple[torch.Tensor, torch.Tensor]
    rel_diff_hist: tuple[torch.Tensor, torch.Tensor]
    expected_dtype: torch.dtype
    actual_dtype: torch.dtype
    # NOTE: We don't need to include shape because the expected shape is already known
    # and checked by the runtime

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 59-67
```python
    @classmethod
    def from_tensors(
        cls,
        name: str,
        expected: torch.Tensor | float | int | bool,
        actual: torch.Tensor | float | int | bool,
    ) -> VerificationInfo:
        """Create a VerificationInfo object from two tensors.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `from_tensors`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`from_tensors`。

### Lines 68-76
```python
        Args:
            name: The name of the value.
            expected: The expected tensor.
            actual: The actual tensor.

        Returns:
            VerificationInfo: The VerificationInfo object.
        """
        if not isinstance(expected, torch.Tensor):
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 77-90
```python
            expected = torch.tensor(expected)
        if not isinstance(actual, torch.Tensor):
            actual = torch.tensor(actual)

        max_abs_diff, max_rel_diff, abs_diff, rel_diff = _compare_tensors(
            expected, actual
        )
        bins = torch.tensor(
            [0.0, 1e-6, 1e-5, 1e-4, 1e-3, 1e-2, 1e-1, 1.0, 10, 1000000],
            dtype=torch.float,
        )
        abs_diff_hist = torch.histogram(abs_diff.float(), bins=bins)
        rel_diff_hist = torch.histogram(rel_diff.float(), bins=bins)
        return cls(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 91-99
```python
            name=name,
            max_abs_diff=max_abs_diff,
            max_rel_diff=max_rel_diff,
            abs_diff_hist=abs_diff_hist,
            rel_diff_hist=rel_diff_hist,
            expected_dtype=expected.dtype,
            actual_dtype=actual.dtype,
        )

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 100-106
```python
    def asdict(self) -> dict[str, Any]:
        """Convert the VerificationInfo object to a dictionary.

        Returns:
            A dictionary representation of the VerificationInfo object.
        """
        return {
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `asdict`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`asdict`。

### Lines 107-120
```python
            "name": self.name,
            "max_abs_diff": self.max_abs_diff,
            "max_rel_diff": self.max_rel_diff,
            "abs_diff_hist": [
                self.abs_diff_hist[0].tolist(),
                self.abs_diff_hist[1].tolist(),
            ],
            "rel_diff_hist": [
                self.rel_diff_hist[0].tolist(),
                self.rel_diff_hist[1].tolist(),
            ],
            "expected_dtype": str(self.expected_dtype),
            "actual_dtype": str(self.actual_dtype),
        }
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 121-130
```python


def _compare_tensors(
    expected: torch.Tensor,
    actual: torch.Tensor,
) -> tuple[float, float, torch.Tensor, torch.Tensor]:
    # Move tensors to the same device
    expected = expected.detach().cpu()
    actual = actual.detach().cpu()
    if expected.numel() == 0 or actual.numel() == 0:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_compare_tensors`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_compare_tensors`。

### Lines 131-141
```python
        return math.inf, math.inf, torch.tensor(math.inf), torch.tensor(math.inf)
    if expected.dtype == torch.bool:
        expected = expected.to(torch.float32)
        actual = actual.to(torch.float32)
    if torch.is_complex(expected):
        expected = torch.view_as_real(expected)
    abs_diff = torch.abs(expected - actual)
    eps = 1e-7
    normalizer = torch.abs(expected) + eps
    rel_diff = abs_diff / normalizer

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 142-148
```python
    max_absolute_difference = abs_diff.max().item()
    max_relative_difference = rel_diff.max().item()

    return max_absolute_difference, max_relative_difference, abs_diff, rel_diff


def verify_onnx_program(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `verify_onnx_program`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`verify_onnx_program`。

### Lines 149-155
```python
    onnx_program: _onnx_program.ONNXProgram,
    args: tuple[Any, ...] | None = None,
    kwargs: dict[str, Any] | None = None,
    compare_intermediates: bool = False,
) -> list[VerificationInfo]:
    """Verify the ONNX model by comparing the values with the expected values from ExportedProgram.

```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：无明显局部符号。

### Lines 156-162
```python
    Args:
        onnx_program: The ONNX program to verify.
        args: The input arguments for the model.
        kwargs: The keyword arguments for the model.
        compare_intermediates: Whether to verify intermediate values. This is going
            to take longer time, so it is disabled by default.

```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 163-172
```python
    Returns:
        VerificationInfo objects containing the verification information for each value.
    """
    exported_program = onnx_program.exported_program
    if exported_program is None:
        raise ValueError(
            "The ONNX program does not contain an exported_program. "
            "Please provide an exported_program to verify the ONNX program."
        )
    if args is None and kwargs is None:
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 173-180
```python
        # User did not provide example inputs, use the default example inputs
        if exported_program.example_inputs is None:
            raise ValueError(
                "No example inputs provided and the exported_program does not contain example inputs. "
                "Please provide arguments to verify the ONNX program."
            )
        args, kwargs = exported_program.example_inputs
    if args is None:
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 181-187
```python
        args = ()
    if kwargs is None:
        kwargs = {}

    # Flatten args for ONNX program and the VerificationInterpreter
    flat_args, _ = exported_program._get_flat_args_with_check(args, kwargs)

```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 188-195
```python
    if not compare_intermediates:
        # Compare the output values
        torch_outputs, _ = _pytree.tree_flatten(
            exported_program.module()(*args, **kwargs)
        )
        onnx_outputs = onnx_program(*flat_args)
        results = []
        for torch_output, onnx_output, output_val in zip(
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 196-205
```python
            torch_outputs, onnx_outputs, onnx_program.model.graph.outputs
        ):
            results.append(
                VerificationInfo.from_tensors(
                    name=str(output_val.name),
                    expected=torch_output,
                    actual=onnx_output,
                )
            )
        return results
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; advances ONNX export translation; manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；推进 ONNX 导出翻译流程；操作图状程序结构；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 206-212
```python

    # Use the _VerificationInterpreter to get the intermediate values
    # By design the output values are included too
    interpreter = _VerificationInterpreter(onnx_program)
    interpreter.run(*flat_args)

    return interpreter.verification_infos
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 213-219
```python


def _create_value_mapping(graph: ir.Graph) -> dict[str, ir.Value]:
    """Return a dictionary mapping names to values in the graph.

    The mapping does not include values from subgraphs.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_create_value_mapping`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_create_value_mapping`。

### Lines 220-229
```python
    Args:
        graph: The graph to extract the mapping from.

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

### Lines 230-238
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

### Lines 239-248
```python


class _VerificationInterpreter(torch.fx.Interpreter):
    """Interpreter for verifying converted ONNX model accuracy by comparing intermediate values.

    To compare models, first initialize the interpreter with an ONNX program.
    Then, call the :meth:`run` method with the input arguments to execute the model.
    The :meth:`run` method will execute the model and populate the
    :attr:`verification_infos` attribute with the verification information for each value.

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `_VerificationInterpreter`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`_VerificationInterpreter`。

### Lines 249-256
```python
    ::
        onnx_program = torch.onnx.export(model, args, dynamo=True)
        interpreter = _VerificationInterpreter(onnx_program)
        interpreter.run(*args)
        verification_infos = interpreter.verification_infos
        for info in verification_infos:
            print("value name:", info.name, info)

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 257-265
```python
    The verification information includes the maximum absolute difference, maximum relative
    difference, and histograms of absolute and relative differences between the expected
    and actual values. See :class:`VerificationInfo` for more details.

    Attributes:
        verification_infos: A list of verification information for each value.
            It is populated when the `run` method is called.
    """

```
- EN: Declares C++ types or containers needed in this file. Representative symbols: no dominant local symbols.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：无明显局部符号。

### Lines 266-272
```python
    def __init__(self, onnx_program: torch.onnx.ONNXProgram) -> None:
        """Initialize the _VerificationInterpreter with an ONNX program.

        Args:
            onnx_program: The ONNX program to verify.
        """
        if onnx_program.exported_program is None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`。

### Lines 273-282
```python
            raise ValueError(
                "The ONNX program does not contain an exported_program. "
                "Please provide an exported_program to verify the ONNX program."
            )
        super().__init__(onnx_program.exported_program.module())
        self._onnx_program = onnx_program
        self._onnx_values = _create_value_mapping(onnx_program.model.graph)
        self._args: tuple[Any, ...] = ()
        self.verification_infos: list[VerificationInfo] = []

```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 283-290
```python
    def run(
        self,
        *args: Any,
        initial_env: dict[torch.fx.Node, Any] | None = None,
        enable_io_processing: bool = True,
    ) -> Any:
        """Run the interpreter with the given input arguments.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `run`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`run`。

### Lines 291-298
```python
        This method executes the model and populates the :attr:`verification_infos` attribute
        with the verification information for each value.

        Args:
            args: The input arguments for the model.
            initial_env: The initial environment for the interpreter.
            enable_io_processing: Whether to enable IO processing.

```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 299-309
```python
        Returns:
            Any: The result of executing the model.
        """
        self.verification_infos = []
        self._args = args
        return super().run(
            *args,
            initial_env=initial_env,
            enable_io_processing=enable_io_processing,
        )

```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 310-316
```python
    def run_node(self, n: torch.fx.Node) -> Any:
        result = super().run_node(n)
        if n.op != "call_function":
            return result
        node_name = n.name
        if node_name not in self._onnx_values:
            return result
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `run_node`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`run_node`。

### Lines 317-323
```python
        try:
            (onnx_result,) = self._onnx_program.compute_values([node_name], self._args)
        except Exception:
            logger.warning(
                "Failed to compute value for node %s", node_name, exc_info=True
            )
            return result
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 324-330
```python
        info = VerificationInfo.from_tensors(
            name=node_name,
            expected=result,
            actual=onnx_result,
        )
        self.verification_infos.append(info)
        if info.max_abs_diff > 0.01 or info.max_rel_diff > 0.1:
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流。关键符号：无明显局部符号。

### Lines 331-344
```python
            logger.warning(
                "Verification info for node %s: max_abs_diff: %s, max_rel_diff: %s",
                node_name,
                info.max_abs_diff,
                info.max_rel_diff,
            )
        else:
            logger.info(
                "Verification info for node %s: max_abs_diff: %s, max_rel_diff: %s",
                node_name,
                info.max_abs_diff,
                info.max_rel_diff,
            )
        return result
```
- EN: This block manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
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
- EN: Module integration — the implementation coordinates neighboring Python modules and utility layers.
  CN: 模块集成——该实现会协调相邻的 Python 模块与工具层。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.utils`, `torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter`
- External imports / 外部导入: `__future__`, `dataclasses`, `logging`, `math`, `typing`
- Representative symbols / 代表性符号: `VerificationInfo`, `_compare_tensors`, `verify_onnx_program`, `_create_value_mapping`, `_VerificationInterpreter`
