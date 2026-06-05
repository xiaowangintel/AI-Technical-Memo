# decorators.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/compilation/decorators.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Decorators and feature gates that control how vLLM modules opt into torch.compile. / 控制 vLLM 模块如何接入 torch.compile 的装饰器与特性开关。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-33)
```python
import contextlib
import hashlib
import inspect
import os
import sys
from collections.abc import Callable, Generator
from typing import TYPE_CHECKING, Any, TypeVar, overload
from unittest.mock import patch

import torch
import torch.nn as nn
from torch._dynamo.symbolic_convert import InliningInstructionTranslator

import vllm.envs as envs
from vllm.compilation.counter import compilation_counter
from vllm.compilation.wrapper import TorchCompileWithNoGuardsWrapper
from vllm.config import (
    CompilationMode,
    VllmConfig,
    get_current_vllm_config,
    set_current_vllm_config,
)
from vllm.config.compilation import DynamicShapesType
from vllm.forward_context import get_forward_context, is_forward_context_available
from vllm.logger import init_logger
from vllm.sequence import IntermediateTensors
from vllm.utils.import_utils import resolve_obj_by_qualname
from vllm.utils.torch_utils import is_torch_equal_or_newer
...
```
**EN:** This import block assembles the PyTorch/FX, Inductor, and vLLM modules that the rest of the file relies on for graph rewriting, backend integration, caching, or runtime control in vLLM's compilation path.
**CN:** 这一组导入语句组织了 PyTorch/FX、Inductor 与 vLLM 模块，供文件后续代码在vLLM 的编译路径中完成图改写、后端集成、缓存或运行时控制。

### Constants / assignments (lines 36-50)
```python
_SUPPORTS_SHAPE_ID = is_torch_equal_or_newer("2.11.0")
logger = init_logger(__name__)
IGNORE_COMPILE_KEY = "_ignore_compile_vllm"
_T = TypeVar("_T", bound=nn.Module)
```
**EN:** These assignments establish shared module state such as loggers, global flags, dtype/op handles, or counters that later logic reuses throughout vLLM's compilation path.
**CN:** 这些赋值语句建立了共享模块状态，例如日志器、全局开关、dtype/算子句柄或计数器，供后续代码在vLLM 的编译路径中复用。

### Function `should_torch_compile_mm_encoder` (lines 53-55)
```python
def should_torch_compile_mm_encoder(vllm_config: VllmConfig) -> bool:
    """Callable to be passed to `@support_torch_compile`'s `enable_if` argument."""
    return vllm_config.compilation_config.compile_mm_encoder
```
**EN:** Function `should_torch_compile_mm_encoder` is a decision helper that answers whether a condition or optimization should apply. It fits into vLLM's compilation path. Callable to be passed to `@support_torch_compile`'s `enable_if` argument.
**CN:** 函数 `should_torch_compile_mm_encoder` 是一个判定辅助函数，用于判断某个条件或优化是否应当生效，并嵌入vLLM 的编译路径。 文档字符串说明：Callable to be passed to `@support_torch_compile`'s `enable_if` argument.

### Function `ignore_torch_compile` (lines 58-75)
```python
def ignore_torch_compile(cls: type[_T]) -> type[_T]:
    """
    A decorator to ignore support_torch_compile decorator
    on the class. This is useful when a parent class has
    a support_torch_compile decorator, but we don't want to
    compile the class `cls` that inherits the parent class.
    This only ignores compiling the forward of the class the
    decorator is applied to.

    If the parent has ignore_torch_compile but the child has
    support_torch_compile, the child will still be compiled.

    If the class has one or more submodules
    that have support_torch_compile decorator applied, compile will
    not be ignored for those submodules.
    """
...
```
**EN:** Function `ignore_torch_compile` provides a focused helper used by the surrounding module. It fits into vLLM's compilation path. A decorator to ignore support_torch_compile decorator on the class.
**CN:** 函数 `ignore_torch_compile` 提供供周边模块使用的聚焦辅助逻辑，并嵌入vLLM 的编译路径。 文档字符串说明：A decorator to ignore support_torch_compile decorator on the class.

### Function `_should_ignore_torch_compile` (lines 78-82)
```python
def _should_ignore_torch_compile(cls: type[_T]) -> bool:
    """
    Check if the class should be ignored for torch.compile.
    """
    return getattr(cls, IGNORE_COMPILE_KEY, False)
```
**EN:** Function `_should_ignore_torch_compile` provides a focused helper used by the surrounding module. It fits into vLLM's compilation path. Check if the class should be ignored for torch.compile.
**CN:** 函数 `_should_ignore_torch_compile` 提供供周边模块使用的聚焦辅助逻辑，并嵌入vLLM 的编译路径。 文档字符串说明：Check if the class should be ignored for torch.compile.

### Function overloads `support_torch_compile` (lines 86-252)
```python
def support_torch_compile(
    *,
    enable_if: Callable[[VllmConfig], bool] | None = None,
) -> Callable[[type[_T]], type[_T]]: ...


@overload
def support_torch_compile(
    *,
    dynamic_arg_dims: dict[str, int | list[int] | dict[int, str]] | None,
) -> Callable[[type[_T]], type[_T]]: ...


@overload
def support_torch_compile(
    *,
    mark_unbacked_dims: dict[str, int | list[int]] | None,
) -> Callable[[type[_T]], type[_T]]: ...


@overload
def support_torch_compile(
...
```
**EN:** These overloads define the typing-friendly call shapes for `support_torch_compile` and end with the concrete runtime implementation used inside vLLM's compilation path. A decorator to add support for compiling the forward method of a class.
**CN:** 这一组重载定义了 `support_torch_compile` 的类型友好调用形式，并以实际运行时实现收束，服务于vLLM 的编译路径。 文档字符串说明：A decorator to add support for compiling the forward method of a class.

### Function `_model_hash_key` (lines 255-262)
```python
def _model_hash_key(fn: Callable[..., Any]) -> str:
    import vllm

    sha256_hash = hashlib.sha256()
    sha256_hash.update(vllm.__version__.encode())
    sha256_hash.update(fn.__qualname__.encode())
    sha256_hash.update(str(fn.__code__.co_firstlineno).encode())
    return sha256_hash.hexdigest()
```
**EN:** Function `_model_hash_key` computes a digest or summary used for cache identity and reproducibility. It fits into vLLM's compilation path.
**CN:** 函数 `_model_hash_key` 计算用于缓存身份与可复现性的摘要或哈希，并嵌入vLLM 的编译路径。

### Function `_verify_source_unchanged` (lines 265-281)
```python
def _verify_source_unchanged(
    source_info: "SourceInfo", vllm_config: VllmConfig
) -> None:
    from .caching import _compute_code_hash, _compute_code_hash_with_content

    file_contents = {}
    for source in source_info.inlined_sources:
        module = sys.modules[source.module]
        file = inspect.getfile(module)
        vllm_config.compilation_config.traced_files.add(file)
        file_contents[file] = source.content
    expected_checksum = _compute_code_hash_with_content(file_contents)
    actual_checksum = _compute_code_hash(set(file_contents.keys()))
    if expected_checksum != actual_checksum:
        raise RuntimeError(
            "Source code has changed since the last compilation. Recompiling the model."
...
```
**EN:** Function `_verify_source_unchanged` provides a focused helper used by the surrounding module. It fits into vLLM's compilation path.
**CN:** 函数 `_verify_source_unchanged` 提供供周边模块使用的聚焦辅助逻辑，并嵌入vLLM 的编译路径。

### Function `_try_load_aot_compiled_fn` (lines 284-328)
```python
def _try_load_aot_compiled_fn(
    model: Any,
    aot_compilation_path: str,
) -> Any | None:
    """Try to load an AOT-compiled function from disk.

    Returns the loaded callable on success, or None on failure.
    Re-raises on failure when ``VLLM_FORCE_AOT_LOAD`` is set.
    """
    try:
        with monitor_torch_compile(model.vllm_config, is_encoder=model._is_encoder):
            with (
                set_current_vllm_config(model.vllm_config),
                open(aot_compilation_path, "rb") as f,
            ):
                loaded_fn = torch.compiler.load_compiled_function(
...
```
**EN:** Function `_try_load_aot_compiled_fn` provides a focused helper used by the surrounding module. It fits into vLLM's compilation path. Try to load an AOT-compiled function from disk.
**CN:** 函数 `_try_load_aot_compiled_fn` 提供供周边模块使用的聚焦辅助逻辑，并嵌入vLLM 的编译路径。 文档字符串说明：Try to load an AOT-compiled function from disk.

### Function `_support_torch_compile` (lines 331-721)
```python
def _support_torch_compile(
    cls: type[_T],
    dynamic_arg_dims: dict[str, int | list[int] | dict[int, str]],
    mark_unbacked_dims: dict[str, int | list[int]] | None = None,
    enable_if: Callable[[VllmConfig], bool] | None = None,
    is_encoder: bool = False,
) -> type[_T]:
    """Internal implementation of support_torch_compile decorator."""

    if TorchCompileWithNoGuardsWrapper in cls.__bases__:
        # support decorating multiple times
        return cls

    # take care of method resolution order
    # make sure super().__init__ is called on the base class
    #  other than TorchCompileWithNoGuardsWrapper
...
```
**EN:** Function `_support_torch_compile` provides a focused helper used by the surrounding module. It fits into vLLM's compilation path. Internal implementation of support_torch_compile decorator.
**CN:** 函数 `_support_torch_compile` 提供供周边模块使用的聚焦辅助逻辑，并嵌入vLLM 的编译路径。 文档字符串说明：Internal implementation of support_torch_compile decorator.

### Function `maybe_use_cudagraph_partition_wrapper` (lines 725-780)
```python
def maybe_use_cudagraph_partition_wrapper(
    vllm_config: VllmConfig,
) -> Generator[None, None, None]:
    """
    Context manager to set/unset customized cudagraph partition wrappers.

    If we're using Inductor-based graph partitioning, we currently have the
    whole `fx.Graph` before Inductor lowering and the piecewise
    splitting happens after all graph passes and fusions. Here, we add
    a custom hook for Inductor to wrap each partition with our static
    graph wrapper class to maintain more control over static graph
    capture and replay.
    """
    from vllm.config import CUDAGraphMode

    compilation_config = vllm_config.compilation_config
...
```
**EN:** Function `maybe_use_cudagraph_partition_wrapper` provides a focused helper used by the surrounding module. It fits into vLLM's compilation path. Context manager to set/unset customized cudagraph partition wrappers.
**CN:** 函数 `maybe_use_cudagraph_partition_wrapper` 提供供周边模块使用的聚焦辅助逻辑，并嵌入vLLM 的编译路径。 文档字符串说明：Context manager to set/unset customized cudagraph partition wrappers.

## Key Concepts / 关键概念
- **Compilation pipeline / 编译流水线**
  - **EN:** The file participates in turning FX graphs into optimized executable code or runtime wrappers.
  - **CN:** 该文件参与将 FX 图转换为优化后的可执行代码或运行时封装。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `import vllm.envs as envs`, `from vllm.compilation.counter import compilation_counter`, `from vllm.compilation.wrapper import TorchCompileWithNoGuardsWrapper`, `from vllm.config import CompilationMode, VllmConfig, get_current_vllm_config, set_current_vllm_config`, `from vllm.config.compilation import DynamicShapesType`, `from vllm.forward_context import get_forward_context, is_forward_context_available`, `from vllm.logger import init_logger`, `from vllm.sequence import IntermediateTensors`, ...
- **PyTorch / Torch 栈**: `import torch`, `import torch.nn as nn`, `from torch._dynamo.symbolic_convert import InliningInstructionTranslator`
- **Stdlib / 标准库**: `import contextlib`, `import hashlib`, `import inspect`, `import os`, `import sys`, `from collections.abc import Callable, Generator`, `from typing import TYPE_CHECKING, Any, TypeVar, overload`, `from unittest.mock import patch`
