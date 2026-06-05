# _registration.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/_internal/exporter/_registration.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal ONNX exporter logic for registration, covering graph capture, translation, validation, or serialization.
- 用途 (CN): 实现与 registration 相关的 ONNX 导出器内部逻辑，涵盖图捕获、转换、校验或序列化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11
```python
"""Module for handling ATen to ONNX functions registration.

https://github.com/pytorch/pytorch/blob/6aa5bb1a76dee8112f1a9e7c194c790b5cdc6462/torch/onnx/_internal/fx/registration.py
"""

# NOTE: Why do we need a different registry than the one in torchlib?
# The registry in torchlib is used to register functions that are already implemented in
# torchlib, and is designed to be a static singleton. It does not take into account custom ops or different
# opsets etc. The registry implemented for the exporter is designed to be modifiable at
# export time by users, and is designed with dispatching in mind.

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 12-18
```python
# mypy: allow-untyped-defs
from __future__ import annotations

import dataclasses
import importlib.util
import logging
import math
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `__future__`, `dataclasses`, `importlib.util`, `logging`, `math`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`__future__`, `dataclasses`, `importlib.util`, `logging`, `math`。

### Lines 19-25
```python
import operator
import types
from collections.abc import Callable
from typing import Literal, TypeAlias

import torch
import torch._ops
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch._ops`; external imports: `operator`, `types`, `collections.abc`, `typing`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch._ops`；外部导入：`operator`, `types`, `collections.abc`, `typing`。

### Lines 26-32
```python
from torch.onnx._internal._lazy_import import onnx_ir as ir, onnxscript, onnxscript_apis
from torch.onnx._internal.exporter import _constants, _schemas
from torch.onnx._internal.exporter._torchlib import _torchlib_registry


TorchOp: TypeAlias = torch._ops.OpOverload | types.BuiltinFunctionType | Callable

```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter`, `torch.onnx._internal.exporter._torchlib`; external imports: none.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter`, `torch.onnx._internal.exporter._torchlib`；外部导入：无。

### Lines 33-39
```python
logger = logging.getLogger(__name__)


@dataclasses.dataclass
class OnnxDecompMeta:
    """A wrapper of onnx-script function with additional metadata.

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `OnnxDecompMeta`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`OnnxDecompMeta`。

### Lines 40-51
```python
    onnx_function: The onnx-script function from torchlib.
    fx_target: The PyTorch node callable target.
    signature: The ONNX signature of the function. When None, the signature is inferred.
    is_custom: Whether the function is a custom function.
    is_complex: Whether the function is a function that handles complex valued inputs.
    opset_introduced:
        The ONNX opset version in which the function was introduced.
        Its specifies the minimum ONNX opset version required to use the function.
    device: The device the function is registered to. If None, it is registered to all devices.
    skip_signature_inference: Whether to skip signature inference for the function.
    """

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 52-60
```python
    onnx_function: Callable
    fx_target: TorchOp
    signature: ir.schemas.OpSignature | None
    is_custom: bool = False
    is_complex: bool = False
    opset_introduced: int = 18
    device: Literal["cuda", "cpu"] | str | None = None  # noqa: PYI051
    skip_signature_inference: bool = False

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：无明显局部符号。

### Lines 61-74
```python
    def __post_init__(self) -> None:
        if self.signature is None and not self.skip_signature_inference:
            try:
                if isinstance(self.onnx_function, onnxscript.OnnxFunction):
                    signature = _schemas.op_signature_from_function(
                        self.onnx_function,
                        self.onnx_function.function_ir.domain,
                        self.onnx_function.name,
                        since_version=self.onnx_function.opset.version,
                    )
                else:
                    signature = _schemas.op_signature_from_function(
                        self.onnx_function, "__traced", self.onnx_function.__name__
                    )
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__post_init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__post_init__`。

### Lines 75-88
```python
            except Exception as e:
                # Log an warning if the op is custom. Raise exception for builtin ops.
                if not self.is_custom:
                    raise
                else:
                    # When the function is targeting an HOP, for example, it will accept
                    # functions as arguments and fail to generate an ONNX signature.
                    # In this case we set signature to None and dispatch to this function always.
                    logger.warning(
                        "Failed to infer the signature for function '%s' because '%s'"
                        "All nodes targeting `%s` will be dispatched to this function",
                        self.onnx_function,
                        e,
                        self.fx_target,
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 89-95
```python
                    )
            else:
                self.signature = signature
                self.onnx_function._pt_onnx_signature = signature  # type: ignore[attr-defined]


def _get_overload(qualified_name: str) -> torch._ops.OpOverload | None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_get_overload`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_get_overload`。

### Lines 96-102
```python
    """Obtain the torch op from <namespace>::<op_name>[.<overload>]"""
    # TODO(justinchuby): Handle arbitrary custom ops
    namespace, opname_overload = qualified_name.split("::")
    op_name, *maybe_overload = opname_overload.split(".", 1)
    if namespace == "_operator":
        # Builtin functions
        return getattr(operator, op_name)
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 103-111
```python
    if namespace == "math":
        return getattr(math, op_name)
    if namespace == "torchvision":
        if importlib.util.find_spec("torchvision") is None:
            logger.warning("torchvision is not installed. Skipping %s", qualified_name)
            return None
    try:
        op_packet = getattr(getattr(torch.ops, namespace), op_name)
        if maybe_overload:
```
- EN: This block reports or normalizes error conditions; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 112-122
```python
            overload = maybe_overload[0]
        elif "default" in op_packet._overload_names or "" in op_packet._overload_names:
            # Has a default overload
            overload = "default"
        else:
            logger.warning(
                "'%s' does not have a 'default' overload. This could be an error in specifying the op name. Ignoring.",
                qualified_name,
                stacklevel=1,
            )
            return None
```
- EN: This block reports or normalizes error conditions; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 123-129
```python

        return getattr(op_packet, overload)  # type: ignore[call-overload]
    except AttributeError:
        if qualified_name.endswith("getitem"):
            # This is a special case where we registered the function incorrectly,
            # but for BC reasons (pt<=2.4) we need to keep it.
            return None
```
- EN: This block reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 130-136
```python
        logger.info("'%s' is not found in this version of PyTorch.", qualified_name)
        return None
    except Exception:
        logger.exception("Failed to find torch op '%s'", qualified_name)
        return None


```
- EN: This block reports or normalizes error conditions; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 137-143
```python
class ONNXRegistry:
    """Registry for ONNX functions.

    The registry maintains a mapping from qualified names to symbolic functions under a
    fixed opset version. It supports registering custom onnx-script functions and for
    dispatcher to dispatch calls to the appropriate function.

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `ONNXRegistry`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`ONNXRegistry`。

### Lines 144-150
```python
    """

    def __init__(self) -> None:
        """Initializes the registry"""
        self._opset_version = _constants.TORCHLIB_OPSET
        self.functions: dict[TorchOp | str, list[OnnxDecompMeta]] = {}

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`。

### Lines 151-157
```python
    @property
    def opset_version(self) -> int:
        """The ONNX opset version the exporter should target."""
        return self._opset_version

    @classmethod
    def from_torchlib(cls, opset_version=_constants.TORCHLIB_OPSET) -> ONNXRegistry:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `opset_version`, `from_torchlib`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`opset_version`, `from_torchlib`。

### Lines 158-165
```python
        """Populates the registry with ATen functions from torchlib.

        Args:
            torchlib_registry: The torchlib registry to use for populating the registry.
        """
        registry = cls()
        registry._opset_version = opset_version
        for meta in _torchlib_registry.get_torchlib_ops():
```
- EN: This block advances ONNX export translation; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 166-178
```python
            registry._register(meta.fx_target, meta)

        # TODO(justinchuby): Remove this once torchlib is migrated to PyTorch
        torchlib_ops = onnxscript_apis.get_torchlib_ops()

        for torchlib_meta in torchlib_ops:
            qualified_name = torchlib_meta.qualified_name
            overload_func = torchlib_meta.function
            try:
                # NOTE: This is heavily guarded with try-except because we don't want
                # to fail the entire registry population if one function fails.
                target = _get_overload(qualified_name)
                if target is None:
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；遍历集合或执行单元；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 179-192
```python
                    continue

                meta = OnnxDecompMeta(
                    onnx_function=overload_func,
                    fx_target=target,
                    signature=None,
                    is_custom=False,
                    is_complex=torchlib_meta.is_complex,
                )
                registry._register(target, meta)
            except Exception:
                logger.exception("Failed to register '%s'. Skipped", qualified_name)
                continue

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 193-202
```python
        registry._cleanup_registry_based_on_opset_version()
        return registry

    def _register(
        self,
        target: TorchOp,
        onnx_decomposition: OnnxDecompMeta,
    ) -> None:
        """Registers a OnnxDecompMeta to an operator.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_register`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_register`。

### Lines 203-214
```python
        Args:
            target: The PyTorch node callable target.
            onnx_decomposition: The OnnxDecompMeta to register.
        """
        target_or_name: str | TorchOp
        if isinstance(target, torch._ops.OpOverload):
            # Get the qualified name of the aten op because torch._ops.OpOverload lookup in
            # a dictionary is unreliable for some reason.
            target_or_name = target.name()
        else:
            target_or_name = target
        if onnx_decomposition.is_custom:
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 215-226
```python
            self.functions.setdefault(target_or_name, []).insert(0, onnx_decomposition)
        else:
            self.functions.setdefault(target_or_name, []).append(onnx_decomposition)

    def register_op(
        self,
        target: TorchOp,
        function: Callable,
        is_complex: bool = False,
    ) -> None:
        """Registers a custom operator: torch.ops.<namespace>.<op_name>.<overload>.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `register_op`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`register_op`。

### Lines 227-238
```python
        Args:
            target: The PyTorch node callable target.
            function: The onnx-script function to register.
            is_complex: Whether the function is a function that handles complex valued inputs.
        """
        if isinstance(target, torch._ops.OpOverloadPacket):
            raise TypeError(
                f"Target '{target}' should be provided as an OpOverload instead of an "
                "OpOverloadPacket. You can get the default overload with "
                "<op>.default"
            )

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 239-249
```python
        self._register(
            target,
            OnnxDecompMeta(
                onnx_function=function,
                fx_target=target,
                signature=None,
                is_custom=True,
                is_complex=is_complex,
            ),
        )

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：无明显局部符号。

### Lines 250-263
```python
    def get_decomps(self, target: TorchOp) -> list[OnnxDecompMeta]:
        """Returns a list of OnnxDecompMeta for the given op: torch.ops.<namespace>.<op_name>.<overload>.

        The list is ordered by the time of registration. The custom operators should come
        first in the list.

        Args:
            target: The PyTorch node callable target.
        Returns:
            A list of OnnxDecompMeta corresponding to the given name, or None if
            the name is not in the registry.
        """
        target_or_name: str | TorchOp
        if isinstance(target, torch._ops.OpOverload):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_decomps`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_decomps`。

### Lines 264-270
```python
            # Get the qualified name of the aten op because torch._ops.OpOverload lookup in
            # a dictionary is unreliable for some reason.
            target_or_name = target.name()
        else:
            target_or_name = target
        decomps = self.functions.get(target_or_name, [])
        return sorted(decomps, key=lambda x: x.is_custom, reverse=True)
```
- EN: This block handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 271-277
```python

    def is_registered(self, target: TorchOp) -> bool:
        """Returns whether the given op is registered: torch.ops.<namespace>.<op_name>.<overload>.

        Args:
            target: The PyTorch node callable target.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `is_registered`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`is_registered`。

### Lines 278-286
```python
        Returns:
            True if the given op is registered, otherwise False.
        """
        return bool(self.get_decomps(target))

    def _cleanup_registry_based_on_opset_version(self) -> None:
        """Pick the implementation with the highest opset version valid until the current opset version."""
        cleaned_functions = {}
        for target_or_name, decomps in self.functions.items():
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_cleanup_registry_based_on_opset_version`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_cleanup_registry_based_on_opset_version`。

### Lines 287-294
```python
            # Filter decompositions to only include those with opset_introduced <= opset_version
            decomps = [d for d in decomps if d.opset_introduced <= self.opset_version]

            # Keep only the decomposition with the highest opset_introduced
            if decomps:
                # Find the maximum opset_introduced
                max_opset = max(d.opset_introduced for d in decomps)

```
- EN: This block advances ONNX export translation; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 295-301
```python
                # Keep all decompositions with the maximum opset_introduced
                cleaned_functions[target_or_name] = [
                    d for d in decomps if d.opset_introduced == max_opset
                ]

        self.functions = cleaned_functions

```
- EN: This block advances ONNX export translation; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 302-303
```python
    def __repr__(self) -> str:
        return f"{self.__class__.__name__}(functions={self.functions})"
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__repr__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__repr__`。


## Key Concepts / 关键概念
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
- EN: Interop boundary — the implementation connects PyTorch semantics with ONNX model representation requirements.
  CN: 互操作边界——实现负责连接 PyTorch 语义与 ONNX 模型表示需求。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch._ops`, `torch.onnx._internal._lazy_import`, `torch.onnx._internal.exporter`, `torch.onnx._internal.exporter._torchlib`
- External imports / 外部导入: `__future__`, `dataclasses`, `importlib.util`, `logging`, `math`, `operator`, `types`, `collections.abc`, `typing`
- Representative symbols / 代表性符号: `OnnxDecompMeta`, `_get_overload`, `ONNXRegistry`
