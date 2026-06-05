# patch_torch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/musa/utils/patch_torch.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements MUSA backend support for framework patching for compatibility inside the SGLang runtime. / 为 SGLang 运行时提供面向 MUSA 后端的兼容性补丁逻辑支持。

## Line-by-Line Analysis / 逐行分析
### Lines 14-20: Module setup and shared state / 模块设置与共享状态
```python
import re
from dataclasses import replace as _dataclass_replace

import torch
import torch.fx.graph as fx_graph

_DEVICE_REPR_RE = re.compile(r"\bdevice\(type='([^']+)'(?:,\s*index=(\d+))?\)")
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `re`, `dataclasses`, `torch`, `torch.fx.graph`. It also defines symbols such as `_DEVICE_REPR_RE` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `re`, `dataclasses`, `torch`, `torch.fx.graph`。 同时定义了 `_DEVICE_REPR_RE` 等符号，供后续逻辑使用。

### Lines 23-28: Function `_replace_device_repr` / 函数 `_replace_device_repr`
```python
def _replace_device_repr(m: re.Match) -> str:
    dev_type = m.group(1)
    dev_index = m.group(2)
    if dev_index is not None:
        return f"torch.device('{dev_type}:{dev_index}')"
    return f"torch.device('{dev_type}')"
```
**EN:** This function implements `_replace_device_repr`. It primarily calls `m.group` to complete its work. State updates are written into `dev_type`, `dev_index`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_replace_device_repr`。 它主要通过调用 `m.group` 来完成任务。 状态更新主要写入 `dev_type`, `dev_index`。 实现中使用了条件分支。

### Lines 31-63: Function `patch_fx_custom_device` / 函数 `patch_fx_custom_device`
```python
def patch_fx_custom_device() -> None:
    """
    Fix FX codegen serialization for non-standard devices (e.g. torch_musa).

    Root cause:
    torch.device is registered as a custom builtin named 'device', imported
    via 'from torch import device'. repr(torch.device('musa', 0)) produces
    "device(type='musa', index=0)", which is syntactically valid but fails
    at runtime because torch.device does not recognize 'musa' as a type when
    invoked through the standard import path.

    Fix:
    Post-process the generated src string, replacing all occurrences of
    device(type='x', index=N) with torch.device('x:N'), and ensure 'torch'
    is present in the graph globals.

    Note:
    _get_repr is a closure inside _gen_python_code and cannot be patched
    directly, so we wrap _gen_python_code and rewrite its output instead.
    """
    original = fx_graph.CodeGen._gen_python_code

    def patched(self, nodes, root_module, namespace, **kwargs):
        result = original(self, nodes, root_module, namespace, **kwargs)
        new_src = _DEVICE_REPR_RE.sub(_replace_device_repr, result.src)
        if new_src is result.src:
            return result
        result.globals.setdefault("torch", torch)
        if hasattr(result, "_replace"):
            return result._replace(src=new_src)
        return _dataclass_replace(result, src=new_src)

    fx_graph.CodeGen._gen_python_code = patched
```
**EN:** This function implements `patch_fx_custom_device`. It primarily calls `original`, `_DEVICE_REPR_RE.sub`, `result.globals.setdefault`, `hasattr`, `_dataclass_replace`, `result._replace` to complete its work. State updates are written into `original`, `fx_graph.CodeGen._gen_python_code`, `result`, `new_src`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `patch_fx_custom_device`。 它主要通过调用 `original`, `_DEVICE_REPR_RE.sub`, `result.globals.setdefault`, `hasattr`, `_dataclass_replace`, `result._replace` 来完成任务。 状态更新主要写入 `original`, `fx_graph.CodeGen._gen_python_code`, `result`, `new_src`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Functions / 函数**: `_replace_device_repr`, `patch_fx_custom_device`, `patched`
- **Themes / 主题**: `patch`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: None / 无
- **External / 外部依赖**: `torch`, `torch.fx.graph`
- **Standard library / 标准库**: `re`, `dataclasses`
