# compile.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/compilation/compile.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the compilation and graph lowering part of the SRT runtime and implements logic centered on `compile`. It exposes primary entry points such as `IntermediateTensors`, `_normalize_dims`, `_MaybeIntermediateTensors`. / 该模块属于 SRT 运行时的编译与图优化部分，主要实现围绕 `compile` 的逻辑。 它对外提供的主要入口包括 `IntermediateTensors`, `_normalize_dims`, `_MaybeIntermediateTensors`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Module imports, constants, and setup
```python
import inspect
import logging
import os
import sys
import types
from dataclasses import dataclass
from typing import Any, Callable, Optional, Union

import torch

from sglang.srt.compilation.compilation_config import CompilationConfig
from sglang.srt.compilation.piecewise_context_manager import is_in_piecewise_cuda_graph

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; manages graph capture or replay logic.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；管理图捕获或回放逻辑。

### Lines 17-31: Class IntermediateTensors
```python
@dataclass
class IntermediateTensors:
    """For all pipeline stages except the last, we need to return the hidden
    states and residuals to be sent to the next stage. This data structure
    contains the hidden states and residuals for a request.

    Each stage also needs to handle its own finished_sending and
    finished_recving in case of kv transfer.
    """

    tensors: dict[str, torch.Tensor]
    # [req_ids]
    finished_sending: Optional[set[str]] = None
    finished_recving: Optional[set[str]] = None

```
**EN:** This range introduces `IntermediateTensors` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "For all pipeline stages except the last, we need to return the hidden states and residuals to be sent to the next stage."
**CN:** 这一段引入 `IntermediateTensors`，并定义其后续方法依赖的结构或元数据。

### Lines 32-37: Method IntermediateTensors.__init__
```python
    def __init__(self, tensors):
        # manually define this function, so that
        # Dynamo knows `IntermediateTensors()` comes from this file.
        # Otherwise, dataclass will generate this function by evaluating
        # a string, and we will lose the information about the source file.
        self.tensors = tensors
```
**EN:** This callable implements `IntermediateTensors.__init__`. It takes `tensors` and mainly initializes instance state and defaults. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `IntermediateTensors.__init__`。它接收 `tensors`，主要用于初始化实例状态与默认值。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 39-43: Method IntermediateTensors.__getitem__
```python
    def __getitem__(self, key: Union[str, slice]):
        if isinstance(key, str):
            return self.tensors[key]
        elif isinstance(key, slice):
            return self.__class__({k: v[key] for k, v in self.tensors.items()})
```
**EN:** This callable implements `IntermediateTensors.__getitem__`. It takes `key` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `IntermediateTensors.__getitem__`。它接收 `key`，主要用于获取某个值或派生视图。

### Lines 45-46: Method IntermediateTensors.__setitem__
```python
    def __setitem__(self, key: str, value: torch.Tensor):
        self.tensors[key] = value
```
**EN:** This callable implements `IntermediateTensors.__setitem__`. It takes `key`, `value` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `IntermediateTensors.__setitem__`。它接收 `key`, `value`，主要用于将配置写入可变状态。

### Lines 48-49: Method IntermediateTensors.items
```python
    def items(self):
        return self.tensors.items()
```
**EN:** This callable implements `IntermediateTensors.items` and mainly implements items.
**CN:** 这一可调用对象实现了 `IntermediateTensors.items`，主要用于实现 items 相关逻辑。

### Lines 51-52: Method IntermediateTensors.__len__
```python
    def __len__(self):
        return len(self.tensors)
```
**EN:** This callable implements `IntermediateTensors.__len__` and mainly implements len.
**CN:** 这一可调用对象实现了 `IntermediateTensors.__len__`，主要用于实现 len 相关逻辑。

### Lines 54-55: Method IntermediateTensors.__eq__
```python
    def __eq__(self, other: object):
        return isinstance(other, self.__class__) and self
```
**EN:** This callable implements `IntermediateTensors.__eq__`. It takes `other` and mainly implements eq.
**CN:** 这一可调用对象实现了 `IntermediateTensors.__eq__`。它接收 `other`，主要用于实现 eq 相关逻辑。

### Lines 57-58: Method IntermediateTensors.__repr__
```python
    def __repr__(self) -> str:
        return f"IntermediateTensors(tensors={self.tensors})"
```
**EN:** This callable implements `IntermediateTensors.__repr__` and mainly implements repr.
**CN:** 这一可调用对象实现了 `IntermediateTensors.__repr__`，主要用于实现 repr 相关逻辑。

### Lines 61-63: Function _normalize_dims
```python
def _normalize_dims(dims, ndim: int):
    dims = [dims] if isinstance(dims, int) else list(dims)
    return [d if d >= 0 else ndim + d for d in dims]
```
**EN:** This callable implements `_normalize_dims`. It takes `dims`, `ndim` and mainly normalizes user-provided values.
**CN:** 这一可调用对象实现了 `_normalize_dims`。它接收 `dims`, `ndim`，主要用于规范化用户提供的值。

### Lines 66-68: Class _MaybeIntermediateTensors
```python
class _MaybeIntermediateTensors:
    """Duck-typed check to support your IntermediateTensors without importing."""

```
**EN:** This range introduces `_MaybeIntermediateTensors` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Duck-typed check to support your IntermediateTensors without importing."
**CN:** 这一段引入 `_MaybeIntermediateTensors`，并定义其后续方法依赖的结构或元数据。

### Lines 69-73: Method _MaybeIntermediateTensors.__init__
```python
    def __init__(self, obj):
        self.is_intermediate = hasattr(obj, "tensors") and isinstance(
            getattr(obj, "tensors"), dict
        )
        self.obj = obj
```
**EN:** This callable implements `_MaybeIntermediateTensors.__init__`. It takes `obj` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `_MaybeIntermediateTensors.__init__`。它接收 `obj`，主要用于初始化实例状态与默认值。

### Lines 76-83: Function _mark_dynamic_on_value
```python
def _mark_dynamic_on_value(val, dims):
    if isinstance(val, torch.Tensor):
        torch._dynamo.maybe_mark_dynamic(val, _normalize_dims(dims, val.ndim))
    else:
        mit = _MaybeIntermediateTensors(val)
        if mit.is_intermediate:
            for t in mit.obj.tensors.values():
                torch._dynamo.maybe_mark_dynamic(t, _normalize_dims(dims, t.ndim))
```
**EN:** This callable implements `_mark_dynamic_on_value`. It takes `val`, `dims` and mainly implements mark dynamic on value.
**CN:** 这一可调用对象实现了 `_mark_dynamic_on_value`。它接收 `val`, `dims`，主要用于实现 mark dynamic on value 相关逻辑。

### Lines 84-86: Module-level logic
```python
        # else: ignore (None or non-tensor)


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 87-108: Function _infer_dynamic_arg_dims_from_annotations
```python
def _infer_dynamic_arg_dims_from_annotations(forward_fn):
    sig = inspect.signature(forward_fn)
    dyn = {}
    for name, p in sig.parameters.items():
        ann = p.annotation
        # Accept torch.Tensor / Optional[torch.Tensor] / your IntermediateTensors types by name
        if (
            ann is torch.Tensor
            or getattr(getattr(ann, "__args__", [None])[0], "__name__", "") == "Tensor"
        ):
            dyn[name] = 0
        elif getattr(ann, "__name__", "") in ("IntermediateTensors",) or any(
            getattr(a, "__name__", "") == "IntermediateTensors"
            for a in getattr(ann, "__args__", [])
        ):
            dyn[name] = 0
        elif ann == "torch.Tensor" or ann == "Optional[torch.Tensor]":
            # For future import annotations (e.g. from __future__ import annotations), the annotation is a string
            dyn[name] = 0
    if not dyn:
        raise ValueError("No dynamic dims inferred; pass dynamic_arg_dims explicitly.")
    return dyn
```
**EN:** This callable implements `_infer_dynamic_arg_dims_from_annotations`. It takes `forward_fn` and mainly constructs data from an external representation. In this range it sets up imports and shared symbols; performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `_infer_dynamic_arg_dims_from_annotations`。它接收 `forward_fn`，主要用于从外部表示构造数据。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查。

### Lines 111-170: Function install_torch_compiled (part 1/2)
```python
def install_torch_compiled(
    module: torch.nn.Module,
    *,
    dynamic_arg_dims: dict[str, Union[int, list[int]]] | None = None,
    backend_factory: Optional[Callable[[torch.fx.GraphModule, list], Callable]] = None,
    compile_config: CompilationConfig = None,
    fullgraph: bool = True,
    graph_pool: Any = None,
):
    unbound_fwd = module.__class__.forward
    if not callable(unbound_fwd):
        raise TypeError("module.__class__.forward must be callable")
    original_code = unbound_fwd.__code__

    dyn_map = dynamic_arg_dims or _infer_dynamic_arg_dims_from_annotations(unbound_fwd)

    if backend_factory is None:
        from sglang.srt.compilation.backend import SGLangBackend

        backend_factory = lambda gm, ex: SGLangBackend(compile_config, graph_pool)(
            gm, ex
        )

    compiled_codes: list[type(original_code)] = []
    state = {"compiled": False, "compiled_callable": None}

    def bytecode_hook(old_code, new_code):
        if old_code is not original_code:
            return
        frame = sys._getframe()
        while frame and frame.f_back:
            frame = frame.f_back
            if (
                frame.f_code.co_name == "_compile"
                and os.path.basename(frame.f_code.co_filename) == "convert_frame.py"
            ):
                break
        try:
            dynamo_frame = frame.f_locals["frame"]
        except Exception:
            return
        if dynamo_frame.f_code is not old_code:
            return
        if dynamo_frame.f_locals.get("self") is not module:
            return
        compiled_codes.append(new_code)

    torch._dynamo.convert_frame.register_bytecode_hook(bytecode_hook)

    def _ensure_compiled(self, *args, **kwargs):
        """Compile on first use (with flag ON)."""
        if state["compiled"]:
            return
        # Mark dynamic dims only when we are about to compile
        sig = inspect.signature(unbound_fwd)
        ba = sig.bind(self, *args, **kwargs)
        ba.apply_defaults()
        for name, dims in (dyn_map or {}).items():
            if name in ba.arguments:
                val = ba.arguments[name]
```
**EN:** This callable implements `install_torch_compiled`. It takes `module` and mainly converts data into another representation. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `install_torch_compiled`。它接收 `module`，主要用于将数据转换为另一种表示。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；处理与编译相关的行为。

### Lines 171-201: Function install_torch_compiled (part 2/2)
```python
                if val is not None:
                    _mark_dynamic_on_value(val, dims)

        # Avoid cross-instance cache reuse
        torch._dynamo.eval_frame.remove_from_cache(unbound_fwd.__code__)

        bound = types.MethodType(unbound_fwd, self)
        compiled_callable = torch.compile(
            bound, fullgraph=fullgraph, backend=backend_factory
        )

        # Trigger Dynamo so bytecode hook can capture
        compiled_callable(*args, **kwargs)

        state["compiled"] = True
        state["compiled_callable"] = compiled_callable

    def trampoline(self, *args, **kwargs):
        use_compiled = is_in_piecewise_cuda_graph()
        if use_compiled:
            if not state["compiled"]:
                _ensure_compiled(self, *args, **kwargs)

            compiled_callable = state["compiled_callable"]
            return compiled_callable(*args, **kwargs)
        else:
            # Explicitly run the original uncompiled forward
            return unbound_fwd(self, *args, **kwargs)

    module.forward = types.MethodType(trampoline, module)
    return module
```
**EN:** This callable implements `install_torch_compiled`. It takes `module` and mainly converts data into another representation. This chunk is part 2 of 2 for the same logical block. In this range it prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `install_torch_compiled`。它接收 `module`，主要用于将数据转换为另一种表示。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会处理与编译相关的行为；管理图捕获或回放逻辑。

## Key Concepts / 关键概念
- `IntermediateTensors`: core class or state container / 核心类或状态容器
- `_normalize_dims`: normalizes user-provided values / 规范化用户提供的值
- `_MaybeIntermediateTensors`: core class or state container / 核心类或状态容器
- `_mark_dynamic_on_value`: implements mark dynamic on value / 实现 mark dynamic on value 相关逻辑
- `_infer_dynamic_arg_dims_from_annotations`: constructs data from an external representation / 从外部表示构造数据
- `install_torch_compiled`: converts data into another representation / 将数据转换为另一种表示

## Dependencies / 依赖关系
- **Standard library / 标准库**: `inspect`, `logging`, `os`, `sys`, `types`, `dataclasses`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.compilation.compilation_config`, `sglang.srt.compilation.piecewise_context_manager`, `sglang.srt.compilation.backend`
