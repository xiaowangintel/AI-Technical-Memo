# hook_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/model_executor/hook_manager.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the model execution and graph runners part of the SRT runtime and implements logic centered on `hook_manager`. It exposes primary entry points such as `register_forward_hooks`, `resolve_callable`. / 该模块属于 SRT 运行时的模型执行与图运行器部分，主要实现围绕 `hook_manager` 的逻辑。 它对外提供的主要入口包括 `register_forward_hooks`, `resolve_callable`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Module imports, constants, and setup
```python
import fnmatch
import importlib
import logging
from typing import Any, Callable, List, Optional

import torch.nn as nn

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断。

### Lines 11-56: Function register_forward_hooks
```python
def register_forward_hooks(model: nn.Module, hook_specs: List[dict[str, Any]]) -> None:
    """
    hook_specs is a list of dicts from server_args.forward_hooks.
    Attaches forward hooks to the matching modules.
    """
    name_to_module = dict(model.named_modules())

    for spec in hook_specs:
        spec_name = spec.get("name", "")
        target_patterns = spec.get("target_modules", [])
        if not target_patterns:
            logger.warning(f"Hook spec '{spec_name}' has no 'target_modules', skipping")
            continue

        hook_factory_path = spec.get("hook_factory")
        if not hook_factory_path:
            logger.warning(f"Hook spec '{spec_name}' has no 'hook_factory', skipping")
            continue

        config = spec.get("config") or {}
        hook_factory = resolve_callable(hook_factory_path)

        hook = hook_factory(config) if hook_factory else None
        if hook is None:
            logger.warning(
                f"Hook factory '{hook_factory_path}' for spec '{spec_name}' "
                "returned None, not registering any hook"
            )
            continue

        # Resolve patterns like "model.layers.*.mlp"
        matched = []
        for name, module in name_to_module.items():
            if any(fnmatch.fnmatch(name, pattern) for pattern in target_patterns):
                matched.append((name, module))

        if not matched:
            logger.warning(
                f"No modules matched hook spec '{spec_name}' "
                f"patterns={target_patterns}"
            )
            continue

        for module_name, module in matched:
            _ = module.register_forward_hook(hook)
            logger.info(f"Registered forward hook '{spec_name}' " f"on {module_name}")
```
**EN:** This callable implements `register_forward_hooks`. It takes `model`, `hook_specs` and mainly registers hooks, handlers, or operators. The docstring states: "hook_specs is a list of dicts from server_args.forward_hooks." In this range it sets up imports and shared symbols; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `register_forward_hooks`。它接收 `model`, `hook_specs`，主要用于注册钩子、处理器或算子。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断。

### Lines 59-82: Function resolve_callable
```python
def resolve_callable(path: Optional[str]) -> Optional[Callable]:
    if path is None:
        return None

    if ":" in path:
        module_name, fn_name = path.split(":", 1)
    else:
        parts = path.split(".")
        if len(parts) < 2:
            raise ValueError(
                f"Invalid hook callable path '{path}'. "
                "Expected 'module.submodule:factory' or 'module.submodule.factory'."
            )
        *mod_parts, fn_name = parts
        module_name = ".".join(mod_parts)

    module = importlib.import_module(module_name)
    try:
        return getattr(module, fn_name)
    except AttributeError as e:
        raise AttributeError(
            f"Module '{module_name}' has no attribute '{fn_name}' "
            f"(from hook path '{path}')"
        ) from e
```
**EN:** This callable implements `resolve_callable`. It takes `path` and mainly implements resolve callable. In this range it sets up imports and shared symbols; performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `resolve_callable`。它接收 `path`，主要用于实现 resolve callable 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查。

## Key Concepts / 关键概念
- `register_forward_hooks`: registers hooks, handlers, or operators / 注册钩子、处理器或算子
- `resolve_callable`: implements resolve callable / 实现 resolve callable 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `fnmatch`, `importlib`, `logging`, `typing`
- **Third-party / 第三方**: `torch.nn`
