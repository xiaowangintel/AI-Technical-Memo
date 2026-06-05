# dumper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/dumper.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on shared debugging utilities. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于共享调试工具。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Import dependencies and shared types / 导入依赖与共享类型
```python
import enum
import functools
import json
import os
import random
import re
import socket
import threading
import time
import traceback
from abc import ABC, abstractmethod
from collections.abc import Callable
from contextlib import contextmanager
from copy import deepcopy
from dataclasses import asdict, dataclass, field, fields, replace
from functools import cached_property
from http.server import BaseHTTPRequestHandler, HTTPServer
from pathlib import Path
from typing import Any, List, Literal, Optional, Union, get_args, get_type_hints

import torch
import torch.distributed as dist
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 28-28: Define class `_BaseConfig` and class context / 定义类 `_BaseConfig`及类上下文
```python
class _BaseConfig(ABC):
```
**EN:** This section introduces `_BaseConfig`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_BaseConfig`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 29-30: Implement method `__post_init__` for `_BaseConfig` / 为 `_BaseConfig` 实现方法 `__post_init__`
```python
    def __post_init__(self) -> None:
        self._verify_types()
```
**EN:** Method `__post_init__` implements behavior on `_BaseConfig`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__post_init__` 为 `_BaseConfig` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 32-44: Implement method `_verify_types` for `_BaseConfig` / 为 `_BaseConfig` 实现方法 `_verify_types`
```python
    def _verify_types(self) -> None:
        hints = get_type_hints(type(self))
        cls_name = type(self).__name__
        for f in fields(self):
            value = getattr(self, f.name)
            if value is None:
                continue
            expected = self._unwrap_type(hints[f.name])
            if not isinstance(value, expected):
                raise TypeError(
                    f"{cls_name}.{f.name}: expected {expected.__name__}, "
                    f"got {type(value).__name__}"
                )
```
**EN:** Method `_verify_types` implements behavior on `_BaseConfig`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_verify_types` 为 `_BaseConfig` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 48-48: Implement method `_env_prefix` for `_BaseConfig` / 为 `_BaseConfig` 实现方法 `_env_prefix`
```python
    def _env_prefix(cls) -> str: ...
```
**EN:** Method `_env_prefix` implements behavior on `_BaseConfig`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_env_prefix` 为 `_BaseConfig` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 51-52: Implement method `_env_name` for `_BaseConfig` / 为 `_BaseConfig` 实现方法 `_env_name`
```python
    def _env_name(cls, field_name: str) -> str:
        return f"{cls._env_prefix()}{field_name.upper()}"
```
**EN:** Method `_env_name` implements behavior on `_BaseConfig`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_env_name` 为 `_BaseConfig` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 55-61: Implement method `from_env` for `_BaseConfig` / 为 `_BaseConfig` 实现方法 `from_env`
```python
    def from_env(cls) -> "_BaseConfig":
        return cls(
            **{
                f.name: cls._parse_env_field(cls._env_name(f.name), f.default)
                for f in fields(cls)
            }
        )
```
**EN:** Method `from_env` implements behavior on `_BaseConfig`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `from_env` 为 `_BaseConfig` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 63-70: Implement method `with_defaults` for `_BaseConfig` / 为 `_BaseConfig` 实现方法 `with_defaults`
```python
    def with_defaults(self, **kwargs) -> "_BaseConfig":
        cls = type(self)
        actual = {
            key: value
            for key, value in kwargs.items()
            if os.getenv(cls._env_name(key)) is None
        }
        return replace(self, **actual) if actual else self
```
**EN:** Method `with_defaults` implements behavior on `_BaseConfig`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `with_defaults` 为 `_BaseConfig` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 73-77: Implement method `_unwrap_type` for `_BaseConfig` / 为 `_BaseConfig` 实现方法 `_unwrap_type`
```python
    def _unwrap_type(hint) -> type:
        args = get_args(hint)
        if args:
            return next(a for a in args if a is not type(None))
        return hint
```
**EN:** Method `_unwrap_type` implements behavior on `_BaseConfig`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_unwrap_type` 为 `_BaseConfig` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 80-81: Implement method `_parse_env_field` for `_BaseConfig` / 为 `_BaseConfig` 实现方法 `_parse_env_field`
```python
    def _parse_env_field(cls, env_name: str, default):
        return cls._parse_env_value(os.getenv(env_name), default)
```
**EN:** Method `_parse_env_field` implements behavior on `_BaseConfig`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_parse_env_field` 为 `_BaseConfig` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 84-91: Implement method `_parse_env_value` for `_BaseConfig` / 为 `_BaseConfig` 实现方法 `_parse_env_value`
```python
    def _parse_env_value(raw, default):
        if raw is None or not raw.strip():
            return default
        if isinstance(default, bool):
            return raw.lower() in ("true", "1")
        if isinstance(default, int):
            return int(raw)
        return raw
```
**EN:** Method `_parse_env_value` implements behavior on `_BaseConfig`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_parse_env_value` 为 `_BaseConfig` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 94-95: Implement method `from_kv_pairs` for `_BaseConfig` / 为 `_BaseConfig` 实现方法 `from_kv_pairs`
```python
    def from_kv_pairs(cls, pairs: Optional[List[str]]) -> "_BaseConfig":
        return cls(**cls._kv_pairs_to_dict(pairs))
```
**EN:** Method `from_kv_pairs` implements behavior on `_BaseConfig`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `from_kv_pairs` 为 `_BaseConfig` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 98-121: Implement method `_kv_pairs_to_dict` for `_BaseConfig` / 为 `_BaseConfig` 实现方法 `_kv_pairs_to_dict`
```python
    def _kv_pairs_to_dict(cls, pairs: Optional[List[str]]) -> dict:
        if not pairs:
            return {}

        missing = object()
        defaults = {f.name: f.default for f in fields(cls)}
        result: dict = {}

        for pair in pairs:
            key, sep, value = pair.partition("=")
            if not sep:
                raise ValueError(f"Invalid config pair (missing '='): {pair!r}")
            default = defaults.get(key, missing)
            if default is missing:
                raise ValueError(
                    f"Unknown config key {key!r}. Valid keys: {sorted(defaults)}"
                )
            try:
                result[key] = cls._parse_env_value(value, default)
            except (ValueError, TypeError) as exc:
                field_type = type(default).__name__
                raise TypeError(f"{key}: expected {field_type}, got {value!r}") from exc

        return result
```
**EN:** Method `_kv_pairs_to_dict` implements behavior on `_BaseConfig`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_kv_pairs_to_dict` 为 `_BaseConfig` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 124-124: Declare module-level symbols such as `_DEFAULT_EXP_NAME_PREFIX` / 声明模块级符号，例如 `_DEFAULT_EXP_NAME_PREFIX`
```python
_DEFAULT_EXP_NAME_PREFIX = "dump_"
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 128-128: Define class `DumperConfig` and class context / 定义类 `DumperConfig`及类上下文
```python
class DumperConfig(_BaseConfig):
```
**EN:** This section introduces `DumperConfig`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `DumperConfig`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 129-157: Declare fields for `DumperConfig` such as `enable`, `filter`, `dir`, `enable_output_file`, `enable_output_console` / 为 `DumperConfig` 声明字段，例如 `enable`, `filter`, `dir`, `enable_output_file`, `enable_output_console`
```python
    enable: bool = False
    filter: Optional[str] = None
    dir: str = "/tmp/dumper"
    enable_output_file: bool = True
    enable_output_console: bool = True
    enable_value: bool = True
    enable_grad: bool = False
    enable_model_value: bool = False
    enable_model_grad: bool = False
    exp_name: Optional[str] = None
    cleanup_previous: bool = False
    collective_timeout: int = 60
    server_port: str = "-1"
    non_intrusive_mode: str = "core"
    source_patcher_config: Optional[str] = None
    grafter_enable: bool = False
    grafter_role: str = ""  # required if enabled: "baseline" or "target"
    grafter_b2t_filter: Optional[str] = None  # names flowing baseline -> target
    grafter_t2b_filter: Optional[str] = None  # names flowing target -> baseline
    grafter_master_address: str = ""  # required if enabled
    grafter_master_port: int = -1  # required if enabled (positive port)
    grafter_baseline_world_size: int = -1  # required if enabled
    grafter_target_world_size: int = -1  # required if enabled
    grafter_backend: str = "nccl"
    grafter_group_name: str = "graft"
    grafter_timeout: int = 300
    # Fully-qualified Python path "pkg.subpkg.module.fn_name"
    # None -> use the default identity-by-rank fallback in _Grafter._default_transform.
    grafter_transform_path: Optional[str] = None
```
**EN:** These lines declare the state carried by `DumperConfig`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `DumperConfig` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 160-162: Implement method `_env_prefix` for `DumperConfig` / 为 `DumperConfig` 实现方法 `_env_prefix`
```python
    def _env_prefix(cls) -> str:
        # NOTE: should not be `SGLANG_DUMPER_`, otherwise it is weird when dumping Megatron in Miles
        return "DUMPER_"
```
**EN:** Method `_env_prefix` implements behavior on `DumperConfig`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_env_prefix` 为 `DumperConfig` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 164-192: Implement method `__post_init__` for `DumperConfig` / 为 `DumperConfig` 实现方法 `__post_init__`
```python
    def __post_init__(self) -> None:
        super().__post_init__()
        if self.grafter_enable:
            assert self.grafter_role in ("baseline", "target"), (
                f"grafter_role must be 'baseline' or 'target' when grafter_enable=True, "
                f"got {self.grafter_role!r}"
            )
            assert (
                self.grafter_master_address
            ), "grafter_master_address must be set when grafter_enable=True"
            assert self.grafter_master_port > 0, (
                f"grafter_master_port must be a positive port when grafter_enable=True, "
                f"got {self.grafter_master_port}"
            )
            assert self.grafter_baseline_world_size > 0, (
                f"grafter_baseline_world_size must be > 0 when grafter_enable=True, "
                f"got {self.grafter_baseline_world_size}"
            )
            assert self.grafter_target_world_size > 0, (
                f"grafter_target_world_size must be > 0 when grafter_enable=True, "
                f"got {self.grafter_target_world_size}"
            )
            assert (
                self.grafter_b2t_filter is not None
                or self.grafter_t2b_filter is not None
            ), (
                "grafter_enable=True but neither grafter_b2t_filter nor "
                "grafter_t2b_filter is set; nothing would ever be grafted"
            )
```
**EN:** Method `__post_init__` implements behavior on `DumperConfig`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__post_init__` 为 `DumperConfig` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 195-202: Implement method `server_port_parsed` for `DumperConfig` / 为 `DumperConfig` 实现方法 `server_port_parsed`
```python
    def server_port_parsed(self) -> Optional[Union[int, Literal["reuse"]]]:
        raw = self.server_port
        if raw == "reuse":
            return "reuse"
        port = int(raw)
        if port <= 0:
            return None
        return port
```
**EN:** Method `server_port_parsed` implements behavior on `DumperConfig`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `server_port_parsed` 为 `DumperConfig` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 209-209: Define class `_DumperState` and class context / 定义类 `_DumperState`及类上下文
```python
class _DumperState:
```
**EN:** This section introduces `_DumperState`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_DumperState`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 210-214: Declare fields for `_DumperState` such as `dump_index`, `step`, `global_ctx`, `captured_output_data`, `cleanup_previous_handled` / 为 `_DumperState` 声明字段，例如 `dump_index`, `step`, `global_ctx`, `captured_output_data`, `cleanup_previous_handled`
```python
    dump_index: int = 0
    step: int = 0
    global_ctx: dict = field(default_factory=dict)
    captured_output_data: Optional[dict] = None
    cleanup_previous_handled: bool = False
```
**EN:** These lines declare the state carried by `_DumperState`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_DumperState` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 217-217: Define class `_Dumper` and class context / 定义类 `_Dumper`及类上下文
```python
class _Dumper:
```
**EN:** This section introduces `_Dumper`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_Dumper`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 218-245: Document the module intent / 说明模块意图
```python
    """Utility to dump tensors, which can be useful when comparison checking models.

    Example usage:
    dumper.dump("layer_start__hidden_states", hidden_states, layer_id=self.layer_id)
    dumper.step()

    Import from non-SGLang system:
    ```
    import sys
    sys.path.append("/YOUR_PATH/sglang/python/sglang/srt/debug_utils")
    from dumper import dumper
    ```

    Then run the program:
    `DUMPER_ENABLE=1 python ...`

    Auto-cleanup old dumps before first write:
    `DUMPER_CLEANUP_PREVIOUS=1 python ...`

    Alternatively, disable at startup and configure via HTTP:
    1. `python ...`
    2. sglang mode:  `curl -X POST http://localhost:30000/dumper/configure -d '{"enable": true}'`
       standalone:   `curl -X POST http://localhost:40000/dumper/configure -d '{"enable": true}'`
    3. `curl -X POST http://localhost:30000/dumper/configure -d '{"enable": true, "filter": "layer_id=[0-3]"}'`
    4. `curl -X POST http://localhost:30000/dumper/reset`

    Related: `sglang.srt.debug_utils.dump_comparator` for dump comparison
    """
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 247-251: Implement method `__init__` for `_Dumper` / 为 `_Dumper` 实现方法 `__init__`
```python
    def __init__(self, *, config: DumperConfig):
        self._config = config
        self._state = _DumperState()
        self._non_intrusives: list["_NonIntrusiveDumper"] = []
        self._grafter = _Grafter(config=config)
```
**EN:** Method `__init__` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__init__` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 256-257: Implement method `may_enable` for `_Dumper` / 为 `_Dumper` 实现方法 `may_enable`
```python
    def may_enable(self) -> bool:
        return self._config.enable or self._config.server_port_parsed is not None
```
**EN:** Method `may_enable` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `may_enable` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 259-271: Implement method `step` for `_Dumper` / 为 `_Dumper` 实现方法 `step`
```python
    def step(self):
        """This should be called on all ranks at the end of each iteration."""

        self._http_manager  # noqa: B018

        if not self._config.enable:
            return

        # Users may want to `dump` only on some ranks, thus determine name here
        self._ensure_exp_name()

        self._state.step += 1
        _log(f"step={self._state.step}")
```
**EN:** Method `step` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `step` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 273-305: Implement method `dump` for `_Dumper` / 为 `_Dumper` 实现方法 `dump`
```python
    def dump(
        self,
        name: str,
        value,
        save: bool = True,
        dims: Optional[str] = None,
        dims_grad: Optional[str] = None,
        grafter_extras: Optional[dict] = None,
        **kwargs,
    ) -> None:
        value_meta: dict = {}
        grad_meta: dict = {}
        if dims is not None:
            value_meta["dims"] = dims
            grad_meta["dims"] = dims
        if dims_grad is not None:
            value_meta["dims_grad"] = dims_grad
            grad_meta["dims"] = dims_grad

        self._dump_inner(
            name=name,
            value=value,
            extra_kwargs=kwargs,
            save=save,
            enable_value=self._config.enable_value,
            enable_curr_grad=False,
            enable_future_grad=self._config.enable_grad,
            value_tag="Dumper.Value",
            grad_tag="Dumper.Grad",
            value_meta_only_fields=value_meta,
            grad_meta_only_fields=grad_meta,
            grafter_extras=grafter_extras,
        )
```
**EN:** Method `dump` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `dump` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 307-325: Implement method `dump_model` for `_Dumper` / 为 `_Dumper` 实现方法 `dump_model`
```python
    def dump_model(
        self,
        model: "torch.nn.Module",
        name_prefix: str = "param",
        save: bool = True,
        **kwargs,
    ) -> None:
        for param_name, param in model.named_parameters():
            self._dump_inner(
                name=f"{name_prefix}__{param_name}",
                value=param,
                extra_kwargs=kwargs,
                save=save,
                enable_value=self._config.enable_model_value,
                enable_curr_grad=self._config.enable_model_grad,
                enable_future_grad=False,
                value_tag="Dumper.ParamValue",
                grad_tag="Dumper.ParamGrad",
            )
```
**EN:** Method `dump_model` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `dump_model` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 327-330: Implement method `dump_dict` for `_Dumper` / 为 `_Dumper` 实现方法 `dump_dict`
```python
    def dump_dict(self, name_prefix, data, save: bool = True, **kwargs):
        data = _obj_to_dict(data)
        for name, value in data.items():
            self.dump(f"{name_prefix}_{name}", value, save=save, **kwargs)
```
**EN:** Method `dump_dict` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `dump_dict` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 332-343: Implement method `set_ctx` for `_Dumper` / 为 `_Dumper` 实现方法 `set_ctx`
```python
    def set_ctx(self, **kwargs):
        """
        Example:

        dumper.configure_default(filter='layer_id=[0-3]')
        dumper.set_ctx(layer_id=self.layer_id)
        ...
        dumper.set_ctx(layer_id=None)
        """
        self._state.global_ctx = {
            k: v for k, v in (self._state.global_ctx | kwargs).items() if v is not None
        }
```
**EN:** Method `set_ctx` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `set_ctx` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 345-376: Implement method `ctx` for `_Dumper` / 为 `_Dumper` 实现方法 `ctx`
```python
    def ctx(
        self,
        _extractor: Optional[Callable[..., dict]] = None,
        **static_ctx: Any,
    ) -> Callable:
        """Decorator that sets context before calling the wrapped function and clears it after.

        Two forms:
            @dumper.ctx(lambda self: dict(layer_id=self.layer_id))
            def forward(self, x): ...

            @dumper.ctx(phase="decode")
            def decode_step(self, x): ...
        """
        if _extractor is not None and static_ctx:
            raise ValueError("cannot mix lambda extractor with static kwargs")
        if _extractor is None and not static_ctx:
            raise ValueError("must provide either a lambda or static kwargs")

        def decorator(fn: Callable) -> Callable:
            @functools.wraps(fn)
            def wrapper(*args: Any, **kwargs: Any) -> Any:
                ctx_dict: dict = _extractor(args[0]) if _extractor else static_ctx
                self.set_ctx(**ctx_dict)
                try:
                    return fn(*args, **kwargs)
                finally:
                    self.set_ctx(**{k: None for k in ctx_dict})

            return wrapper

        return decorator
```
**EN:** Method `ctx` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `ctx` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 378-395: Implement method `apply_source_patches` for `_Dumper` / 为 `_Dumper` 实现方法 `apply_source_patches`
```python
    def apply_source_patches(self) -> None:
        """Apply source patches from DUMPER_SOURCE_PATCHER_CONFIG if set.

        Automatically injects ``from sglang.srt.debug_utils.dumper import dumper``
        into every replacement block so users don't need to write it in YAML.
        """
        config_path = self._config.source_patcher_config
        if not config_path:
            return

        from sglang.srt.debug_utils.source_patcher import apply_patches_from_config

        yaml_content: str = Path(config_path).read_text()
        _log(f"[source_patcher] loading config from {config_path}")
        apply_patches_from_config(
            yaml_content,
            extra_imports=["from sglang.srt.debug_utils.dumper import dumper"],
        )
```
**EN:** Method `apply_source_patches` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `apply_source_patches` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 397-407: Implement method `register_non_intrusive_dumper` for `_Dumper` / 为 `_Dumper` 实现方法 `register_non_intrusive_dumper`
```python
    def register_non_intrusive_dumper(
        self,
        model: "torch.nn.Module",
    ) -> Optional["_NonIntrusiveDumper"]:
        self._http_manager  # noqa: B018
        mode = self._config.non_intrusive_mode
        if mode == "off":
            return None
        non_intrusive = _NonIntrusiveDumper(dumper=self, model=model, mode=mode)
        self._non_intrusives.append(non_intrusive)
        return non_intrusive
```
**EN:** Method `register_non_intrusive_dumper` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `register_non_intrusive_dumper` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 411-412: Implement method `configure` for `_Dumper` / 为 `_Dumper` 实现方法 `configure`
```python
    def configure(self, **kwargs) -> None:
        self._config = replace(self._config, **kwargs)
```
**EN:** Method `configure` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `configure` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 414-415: Implement method `configure_default` for `_Dumper` / 为 `_Dumper` 实现方法 `configure_default`
```python
    def configure_default(self, **kwargs) -> None:
        self._config = self._config.with_defaults(**kwargs)
```
**EN:** Method `configure_default` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `configure_default` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 417-421: Implement method `reset` for `_Dumper` / 为 `_Dumper` 实现方法 `reset`
```python
    def reset(self) -> None:
        for non_intrusive in self._non_intrusives:
            non_intrusive.remove()
        self._non_intrusives.clear()
        self._state = _DumperState()
```
**EN:** Method `reset` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `reset` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 424-430: Implement method `capture_output` for `_Dumper` / 为 `_Dumper` 实现方法 `capture_output`
```python
    def capture_output(self):
        assert self._state.captured_output_data is None
        self._state.captured_output_data = {}
        try:
            yield self._state.captured_output_data
        finally:
            self._state.captured_output_data = None
```
**EN:** Method `capture_output` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `capture_output` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 432-437: Implement method `get_state` for `_Dumper` / 为 `_Dumper` 实现方法 `get_state`
```python
    def get_state(self) -> dict:
        return {
            "config": asdict(self._config),
            "dump_index": self._state.dump_index,
            "step": self._state.step,
        }
```
**EN:** Method `get_state` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `get_state` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 440-443: Implement method `_http_manager` for `_Dumper` / 为 `_Dumper` 实现方法 `_http_manager`
```python
    def _http_manager(self) -> Optional["_DumperHttpManager"]:
        if self._config.server_port_parsed is None:
            return None
        return _DumperHttpManager(self)
```
**EN:** Method `_http_manager` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_http_manager` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 447-515: Implement method `_dump_inner` for `_Dumper` / 为 `_Dumper` 实现方法 `_dump_inner`
```python
    def _dump_inner(
        self,
        *,
        name: str,
        value,
        extra_kwargs: dict,
        save: bool,
        enable_value: bool,
        enable_curr_grad: bool,
        enable_future_grad: bool,
        value_tag: str,
        grad_tag: str,
        value_meta_only_fields: Optional[dict] = None,
        grad_meta_only_fields: Optional[dict] = None,
        grafter_extras: Optional[dict] = None,
    ) -> None:
        self._http_manager  # noqa: B018

        if not self._config.enable:
            return

        recompute_status = _detect_recompute_status()
        tags = dict(
            name=name,
            recompute_status=recompute_status.value,
            **extra_kwargs,
            **self._state.global_ctx,
        )

        if (f := self._config.filter) is not None and not _evaluate_filter(f, tags):
            return

        if not (enable_value or enable_curr_grad or enable_future_grad):
            return

        recompute_meta = recompute_status.to_pseudo_parallel_meta()
        value = _materialize_value(value)
        self._grafter.maybe_intercept(value=value, tags=tags, extras=grafter_extras)

        if enable_value:
            self._dump_single(
                tag=value_tag,
                tags=tags,
                value=value,
                save=save,
                meta_only_fields={**(value_meta_only_fields or {}), **recompute_meta},
            )

        if (
            enable_curr_grad
            and isinstance(value, torch.Tensor)
            and (g := value.grad) is not None
        ):
            self._dump_single(
                tag=grad_tag,
                tags={**tags, "name": f"grad__{name}"},
                value=g,
                save=save,
                meta_only_fields={**(grad_meta_only_fields or {}), **recompute_meta},
            )

        if enable_future_grad:
            self._register_dump_grad_hook(
                name=name,
                tensor=value,
                extra_kwargs=extra_kwargs,
                save=save,
                meta_only_fields=grad_meta_only_fields or {},
            )
```
**EN:** Method `_dump_inner` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_dump_inner` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 517-548: Implement method `_register_dump_grad_hook` for `_Dumper` / 为 `_Dumper` 实现方法 `_register_dump_grad_hook`
```python
    def _register_dump_grad_hook(
        self,
        *,
        name: str,
        tensor,
        extra_kwargs: dict,
        save: bool,
        meta_only_fields: Optional[dict] = None,
    ) -> None:
        if not isinstance(tensor, torch.Tensor):
            return
        if not tensor.requires_grad:
            return

        captured_step = self._state.step
        captured_tags = dict(
            name=f"grad__{name}",
            **deepcopy(extra_kwargs),
        )
        captured_meta_only = meta_only_fields or {}

        def grad_hook(grad: torch.Tensor) -> None:
            self._dump_single(
                tag="Dumper.Grad",
                tags=captured_tags,
                value=grad,
                save=save,
                step=captured_step,
                meta_only_fields=captured_meta_only,
            )

        tensor.register_hook(grad_hook)
```
**EN:** Method `_register_dump_grad_hook` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_register_dump_grad_hook` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 550-609: Implement method `_dump_single` for `_Dumper` / 为 `_Dumper` 实现方法 `_dump_single`
```python
    def _dump_single(
        self,
        *,
        tag: str,
        tags: dict,
        value,
        save: bool,
        step: Optional[int] = None,
        meta_only_fields: Optional[dict] = None,
    ) -> None:
        self._ensure_exp_name()
        self._state.dump_index += 1

        rank = _get_rank()
        full_kwargs = dict(
            step=(step if step is not None else self._state.step),
            rank=rank,
            dump_index=self._state.dump_index,
            **tags,
        )
        full_filename = _format_tags(full_kwargs) + ".pt"
        path = Path(self._config.dir) / self._config.exp_name / full_filename

        if self._config.enable_output_console:
            _log(
                f"[{tag}] {path} "
                f"type={type(value)} "
                f"shape={value.shape if isinstance(value, torch.Tensor) else None} "
                f"dtype={value.dtype if isinstance(value, torch.Tensor) else None} "
                f"device={value.device if isinstance(value, torch.Tensor) else None} "
                f"id={id(value)} "
                f"sample_value={get_truncated_value(value)}"
            )

        capturing = self._state.captured_output_data is not None
        if save and (self._config.enable_output_file or capturing):
            output_data = {
                "value": value,
                "meta": dict(
                    **full_kwargs,
                    **self._static_meta,
                    **(meta_only_fields or {}),
                ),
            }

            if capturing:
                output_data["value"] = _deepcopy_or_clone(output_data["value"])
                self._state.captured_output_data[tags["name"]] = output_data
            else:
                if (
                    not self._state.cleanup_previous_handled
                    and self._config.cleanup_previous
                ):
                    self._state.cleanup_previous_handled = True
                    _cleanup_old_dumps(
                        Path(self._config.dir), exp_name=self._config.exp_name
                    )

                path.parent.mkdir(parents=True, exist_ok=True)
                _torch_save(output_data, str(path))
```
**EN:** Method `_dump_single` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_dump_single` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 614-615: Implement method `_static_meta` for `_Dumper` / 为 `_Dumper` 实现方法 `_static_meta`
```python
    def _static_meta(self) -> dict:
        return _compute_static_meta()
```
**EN:** Method `_static_meta` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_static_meta` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 617-623: Implement method `_ensure_exp_name` for `_Dumper` / 为 `_Dumper` 实现方法 `_ensure_exp_name`
```python
    def _ensure_exp_name(self):
        if self._config.exp_name is None:
            name = _get_default_exp_name(
                timeout_seconds=self._config.collective_timeout
            )
            self.configure(exp_name=name)
            _log(f"Choose exp_name={name}")
```
**EN:** Method `_ensure_exp_name` implements behavior on `_Dumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_ensure_exp_name` 为 `_Dumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 629-629: Define class `_NonIntrusiveDumper` and class context / 定义类 `_NonIntrusiveDumper`及类上下文
```python
class _NonIntrusiveDumper:
```
**EN:** This section introduces `_NonIntrusiveDumper`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_NonIntrusiveDumper`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 630-631: Declare fields for `_NonIntrusiveDumper` such as `_NAME_PREFIX`, `_LAYER_NAME_RE` / 为 `_NonIntrusiveDumper` 声明字段，例如 `_NAME_PREFIX`, `_LAYER_NAME_RE`
```python
    _NAME_PREFIX = "non_intrusive__"
    _LAYER_NAME_RE = re.compile(r"(?:.+\.)?layers\.(\d+)$")
```
**EN:** These lines declare the state carried by `_NonIntrusiveDumper`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_NonIntrusiveDumper` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 633-660: Implement method `__init__` for `_NonIntrusiveDumper` / 为 `_NonIntrusiveDumper` 实现方法 `__init__`
```python
    def __init__(
        self,
        dumper: _Dumper,
        model: "torch.nn.Module",
        mode: str,
    ):
        self._dumper = dumper
        self._mode = mode
        self._handles: list = []
        self._core_fields: frozenset[str] = frozenset().union(
            *(p.core_fields() for p in _plugins)
        )

        for module_name, module in model.named_modules():
            if ctx := self._detect_module_ctx(module_name, module):
                self._register_ctx_hooks(module, ctx=ctx)

            is_root = module_name == ""
            pre_hook = self._make_forward_pre_hook(
                module_name=module_name, is_root=is_root
            )
            hook = self._make_forward_hook(module_name=module_name, is_root=is_root)
            self._handles += _register_forward_hook_or_replace_fn(
                module,
                pre_hook=pre_hook,
                hook=hook,
                mode="replace_fn" if is_root else "hook",
            )
```
**EN:** Method `__init__` implements behavior on `_NonIntrusiveDumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__init__` 为 `_NonIntrusiveDumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 662-665: Implement method `remove` for `_NonIntrusiveDumper` / 为 `_NonIntrusiveDumper` 实现方法 `remove`
```python
    def remove(self) -> None:
        for handle in self._handles:
            handle.remove()
        self._handles.clear()
```
**EN:** Method `remove` implements behavior on `_NonIntrusiveDumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `remove` 为 `_NonIntrusiveDumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 668-678: Implement method `_detect_module_ctx` for `_NonIntrusiveDumper` / 为 `_NonIntrusiveDumper` 实现方法 `_detect_module_ctx`
```python
    def _detect_module_ctx(
        cls, module_name: str, module: "torch.nn.Module"
    ) -> Optional[dict]:
        match = cls._LAYER_NAME_RE.fullmatch(module_name)
        if match:
            for plugin in _plugins:
                layer_id = plugin.detect_layer_id(module)
                if layer_id is not None:
                    return {"layer_id": layer_id}
            return {"layer_id": int(match.group(1))}
        return None
```
**EN:** Method `_detect_module_ctx` implements behavior on `_NonIntrusiveDumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_detect_module_ctx` 为 `_NonIntrusiveDumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 680-693: Implement method `_register_ctx_hooks` for `_NonIntrusiveDumper` / 为 `_NonIntrusiveDumper` 实现方法 `_register_ctx_hooks`
```python
    def _register_ctx_hooks(self, module: "torch.nn.Module", *, ctx: dict) -> None:
        clear_ctx = {k: None for k in ctx}
        self._handles.append(
            module.register_forward_pre_hook(
                lambda _mod, _input, _ctx=ctx: self._dumper.set_ctx(**_ctx)
            )
        )
        self._handles.append(
            module.register_forward_hook(
                lambda _mod, _input, _output, _clear=clear_ctx: self._dumper.set_ctx(
                    **_clear
                )
            )
        )
```
**EN:** Method `_register_ctx_hooks` implements behavior on `_NonIntrusiveDumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_register_ctx_hooks` 为 `_NonIntrusiveDumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 695-709: Implement method `_make_forward_pre_hook` for `_NonIntrusiveDumper` / 为 `_NonIntrusiveDumper` 实现方法 `_make_forward_pre_hook`
```python
    def _make_forward_pre_hook(self, *, module_name: str, is_root: bool):
        def _hook(_module, args, kwargs):
            for i, item in enumerate(args):
                self._dump_value(
                    module_name, item, sub_name=f"inputs.{i}", is_root=is_root
                )
            for name, value in kwargs.items():
                self._dump_value(
                    module_name,
                    value,
                    sub_name=f"inputs.{name}",
                    is_root=is_root,
                )

        return _hook
```
**EN:** Method `_make_forward_pre_hook` implements behavior on `_NonIntrusiveDumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_make_forward_pre_hook` 为 `_NonIntrusiveDumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 711-716: Implement method `_make_forward_hook` for `_NonIntrusiveDumper` / 为 `_NonIntrusiveDumper` 实现方法 `_make_forward_hook`
```python
    def _make_forward_hook(self, *, module_name: str, is_root: bool):
        def _hook(_module, input, output):
            if output is not None:
                self._dump_value(module_name, output, sub_name="output", is_root=False)

        return _hook
```
**EN:** Method `_make_forward_hook` implements behavior on `_NonIntrusiveDumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_make_forward_hook` 为 `_NonIntrusiveDumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 718-729: Implement method `_dump_value` for `_NonIntrusiveDumper` / 为 `_NonIntrusiveDumper` 实现方法 `_dump_value`
```python
    def _dump_value(
        self, module_name: str, value: Any, sub_name: str, *, is_root: bool
    ) -> None:
        for key, item in self._convert_value(
            value, skip_forward_batch=(not is_root)
        ).items():
            effective_key = key or sub_name.rsplit(".", 1)[-1]
            if effective_key in self._core_fields:
                self._dumper.dump(effective_key, item)
            elif self._mode == "all":
                parts = [p for p in (module_name, sub_name, key) if p]
                self._dumper.dump(self._NAME_PREFIX + ".".join(parts), item)
```
**EN:** Method `_dump_value` implements behavior on `_NonIntrusiveDumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_dump_value` 为 `_NonIntrusiveDumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 732-747: Implement method `_convert_value` for `_NonIntrusiveDumper` / 为 `_NonIntrusiveDumper` 实现方法 `_convert_value`
```python
    def _convert_value(value, *, skip_forward_batch: bool = False) -> dict[str, Any]:
        if isinstance(value, torch.Tensor):
            return {"": value}

        if isinstance(value, (tuple, list)):
            tensors = [t for t in value if isinstance(t, torch.Tensor)]
            if len(tensors) == 1:
                return {"": tensors[0]}
            return {str(i): t for i, t in enumerate(tensors)}

        for plugin in _plugins:
            result = plugin.convert_value(value, skip_forward_batch=skip_forward_batch)
            if result is not None:
                return result

        return {}
```
**EN:** Method `_convert_value` implements behavior on `_NonIntrusiveDumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_convert_value` 为 `_NonIntrusiveDumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 750-792: Implement helper `_register_forward_hook_or_replace_fn` / 实现辅助函数 `_register_forward_hook_or_replace_fn`
```python
def _register_forward_hook_or_replace_fn(
    module: "torch.nn.Module",
    *,
    pre_hook,
    hook,
    mode: str,
) -> list:
    """Attach pre/post forward hooks to *module*.

    mode="hook"       — standard ``register_forward_pre_hook`` / ``register_forward_hook``
                        (fires only via ``__call__``).
    mode="replace_fn" — monkey-patch ``module.forward`` so hooks fire even when
                        callers invoke ``.forward()`` directly (as sglang does for the
                        root model).

    Returns a list of handle objects with a ``.remove()`` method that undoes
    the registration.
    """
    if mode == "hook":
        return [
            module.register_forward_pre_hook(pre_hook, with_kwargs=True),
            module.register_forward_hook(hook),
        ]
    elif mode == "replace_fn":
        original_forward = module.forward

        @functools.wraps(original_forward)
        def _wrapped(*args, **kwargs):
            pre_hook(module, args, kwargs)
            output = original_forward(*args, **kwargs)
            hook(module, args, output)
            return output

        module.forward = _wrapped

        class _Handle:
            def remove(self) -> None:
                assert module.forward is _wrapped
                module.forward = original_forward

        return [_Handle()]
    else:
        raise ValueError(f"Unknown mode {mode!r}")
```
**EN:** Function `_register_forward_hook_or_replace_fn` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_register_forward_hook_or_replace_fn` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 798-798: Define class `_GraftRole` and class context / 定义类 `_GraftRole`及类上下文
```python
class _GraftRole(enum.Enum):
```
**EN:** This section introduces `_GraftRole`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_GraftRole`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 799-800: Declare fields for `_GraftRole` such as `BASELINE`, `TARGET` / 为 `_GraftRole` 声明字段，例如 `BASELINE`, `TARGET`
```python
    BASELINE = "baseline"
    TARGET = "target"
```
**EN:** These lines declare the state carried by `_GraftRole`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_GraftRole` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 803-803: Define class `_GraftDirection` and class context / 定义类 `_GraftDirection`及类上下文
```python
class _GraftDirection(enum.Enum):
```
**EN:** This section introduces `_GraftDirection`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_GraftDirection`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 804-805: Declare fields for `_GraftDirection` such as `B2T`, `T2B` / 为 `_GraftDirection` 声明字段，例如 `B2T`, `T2B`
```python
    B2T = "b2t"  # name flows baseline -> target
    T2B = "t2b"  # name flows target -> baseline
```
**EN:** These lines declare the state carried by `_GraftDirection`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_GraftDirection` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 809-809: Define class `GraftTransformInput` and class context / 定义类 `GraftTransformInput`及类上下文
```python
class GraftTransformInput:
```
**EN:** This section introduces `GraftTransformInput`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `GraftTransformInput`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 810-818: Document the module intent / 说明模块意图
```python
    """Single argument passed to a user-supplied transform function.

    User transforms have signature::

        def transform(graft_input: GraftTransformInput) -> torch.Tensor: ...

    The dataclass shape lets us add fields (e.g., direction, sender ranks)
    later without breaking existing transforms.
    """
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 821-828: Declare fields for `GraftTransformInput` such as `tags`, `received_list`, `received_extras_list`, `target` / 为 `GraftTransformInput` 声明字段，例如 `tags`, `received_list`, `received_extras_list`, `target`
```python
    tags: "dict[str, Any]"
    # One tensor per sender rank, in sender-rank order.
    received_list: "list[torch.Tensor]"
    # Parallel list of per-sender `grafter_extras` (the dict passed to
    # dumper.dump on each sender; None if the sender omitted it).
    received_extras_list: "list[Optional[dict]]"
    # Recv side's local tensor that will be copy_'d into.
    target: "torch.Tensor"
```
**EN:** These lines declare the state carried by `GraftTransformInput`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `GraftTransformInput` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 831-831: Define class `_Grafter` and class context / 定义类 `_Grafter`及类上下文
```python
class _Grafter:
```
**EN:** This section introduces `_Grafter`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_Grafter`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 832-844: Document the module intent / 说明模块意图
```python
    """Cross-system tensor transplant. Triggered silently from dumper.dump.

    Both sides set the SAME grafter_b2t_filter (names that flow baseline ->
    target) and grafter_t2b_filter (names that flow target -> baseline). The
    only per-side difference is grafter_role ("baseline" | "target"), which
    determines whether a name match means send or recv on this side.

    Graft global rank layout: baseline occupies ranks 0..baseline_world-1;
    target occupies ranks baseline_world..baseline_world+target_world-1. Each
    side derives its own rank from its local default PG via dist.get_rank().

    Please refer to TestGrafterE2eExample in tests for an example.
    """
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 846-848: Implement method `__init__` for `_Grafter` / 为 `_Grafter` 实现方法 `__init__`
```python
    def __init__(self, *, config: DumperConfig):
        self._config = config
        self._pg = None
```
**EN:** Method `__init__` implements behavior on `_Grafter`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__init__` 为 `_Grafter` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 851-852: Implement method `enabled` for `_Grafter` / 为 `_Grafter` 实现方法 `enabled`
```python
    def enabled(self) -> bool:
        return self._config.grafter_enable
```
**EN:** Method `enabled` implements behavior on `_Grafter`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `enabled` 为 `_Grafter` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 854-935: Implement method `maybe_intercept` for `_Grafter` / 为 `_Grafter` 实现方法 `maybe_intercept`
```python
    def maybe_intercept(
        self, *, value: Any, tags: dict, extras: Optional[dict] = None
    ) -> None:
        """Intercept a dumper.dump call. `extras` is per-call auxiliary data
        (e.g., shard layout, dtype hint) that the sender attaches and the
        recv side's transform receives as `received_extras_list`."""
        cfg = self._config
        if not cfg.grafter_enable:
            return

        direction = self._classify_direction(tags)
        if direction is None:
            return

        if not isinstance(value, torch.Tensor):
            _log(
                f"[Grafter] tags={tags} matched grafter_{direction.value}_filter but "
                f"value is not a torch.Tensor (got type={type(value).__name__}); "
                f"skipping graft. Common cause: dumper.dump called with a non-tensor "
                f"value (dict, list, ...) on this name. Either narrow the filter or "
                f"wrap the value in a tensor."
            )
            return

        self._ensure_group()
        role = _GraftRole(cfg.grafter_role)
        is_send = self._is_sender(role=role, direction=direction)

        # all-gather over the graft world; sender ranks contribute (value,
        # extras) tuples, recv ranks contribute None (their local target is
        # private and shouldn't leak). all_gather_object is pickle-routed,
        # so tensor shapes may differ across sender ranks.
        total_world = cfg.grafter_baseline_world_size + cfg.grafter_target_world_size
        my_contribution = (value, extras) if is_send else None
        gathered: list = [None] * total_world
        dist.all_gather_object(gathered, my_contribution, group=self._pg)

        if is_send:
            _log(
                f"[Grafter] send role={role.value} dir={direction.value} "
                f"tags={tags} extras={extras} local={get_tensor_info(value)}"
            )
            return

        sender_contribs = self._sender_slice(direction=direction, gathered=gathered)
        # Pickled CUDA tensors are restored on their original-device name;
        # that may not match this process's local device, so normalize.
        sender_tensors = [
            (c[0].to(value.device) if isinstance(c[0], torch.Tensor) else c[0])
            for c in sender_contribs
# ... truncated for brevity ...
                received_extras_list=sender_extras,
                target=value,
            )
            diff = _compare_tensors_quick(value, value_to_override)
            _log(
                f"[Grafter] recv role={role.value} dir={direction.value} "
                f"tags={tags} n_senders={len(sender_tensors)} "
                f"sender_extras={sender_extras} "
                f"before_overridden={info_before_overridden} "
                f"to_override={get_tensor_info(value_to_override)} "
                f"diff_pre_vs_new={diff}"
            )
            value.copy_(value_to_override)
        except Exception as e:
            _log(
                f"[Grafter] recv role={role.value} dir={direction.value} "
                f"tags={tags} transform/copy_ raised {type(e).__name__}: {e}; "
                f"skipping graft for this call (target tensor unchanged)\n"
                f"{traceback.format_exc()}"
            )
```
**EN:** Method `maybe_intercept` implements behavior on `_Grafter`. It turns the class state into a focused operation that other parts of the toolchain can call. The snippet is abbreviated to keep the analysis readable.
**CN:** 方法 `maybe_intercept` 为 `_Grafter` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。 为了保持分析可读性，这里的代码片段做了节选。

### Lines 937-949: Implement method `_classify_direction` for `_Grafter` / 为 `_Grafter` 实现方法 `_classify_direction`
```python
    def _classify_direction(self, tags: dict) -> Optional["_GraftDirection"]:
        cfg = self._config
        match_b2t = self._match(cfg.grafter_b2t_filter, tags)
        match_t2b = self._match(cfg.grafter_t2b_filter, tags)
        if match_b2t and match_t2b:
            raise RuntimeError(
                f"[Grafter] tags={tags} matched BOTH grafter_b2t_filter and grafter_t2b_filter"
            )
        if match_b2t:
            return _GraftDirection.B2T
        if match_t2b:
            return _GraftDirection.T2B
        return None
```
**EN:** Method `_classify_direction` implements behavior on `_Grafter`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_classify_direction` 为 `_Grafter` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 952-954: Implement method `_is_sender` for `_Grafter` / 为 `_Grafter` 实现方法 `_is_sender`
```python
    def _is_sender(*, role: "_GraftRole", direction: "_GraftDirection") -> bool:
        # baseline is the sender for B2T names; target is the sender for T2B.
        return (role == _GraftRole.BASELINE) == (direction == _GraftDirection.B2T)
```
**EN:** Method `_is_sender` implements behavior on `_Grafter`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_is_sender` 为 `_Grafter` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 956-960: Implement method `_sender_slice` for `_Grafter` / 为 `_Grafter` 实现方法 `_sender_slice`
```python
    def _sender_slice(self, *, direction: "_GraftDirection", gathered: list) -> list:
        cfg = self._config
        if direction == _GraftDirection.B2T:
            return gathered[: cfg.grafter_baseline_world_size]
        return gathered[cfg.grafter_baseline_world_size :]
```
**EN:** Method `_sender_slice` implements behavior on `_Grafter`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_sender_slice` 为 `_Grafter` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 963-966: Implement method `_match` for `_Grafter` / 为 `_Grafter` 实现方法 `_match`
```python
    def _match(expr: Optional[str], tags: dict) -> bool:
        if expr is None:
            return False
        return _evaluate_filter(expr, tags)
```
**EN:** Method `_match` implements behavior on `_Grafter`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_match` 为 `_Grafter` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 968-1010: Implement method `_ensure_group` for `_Grafter` / 为 `_Grafter` 实现方法 `_ensure_group`
```python
    def _ensure_group(self) -> None:
        if self._pg is not None:
            return

        cfg = self._config
        assert (
            dist.is_initialized()
        ), "[Grafter] default torch.distributed must be initialized"
        role = _GraftRole(cfg.grafter_role)
        local_world = dist.get_world_size()
        local_rank = dist.get_rank()
        if role == _GraftRole.BASELINE:
            assert local_world == cfg.grafter_baseline_world_size, (
                f"[Grafter] grafter_baseline_world_size={cfg.grafter_baseline_world_size} "
                f"but dist.get_world_size()={local_world}; they must match on the baseline side"
            )
            global_rank = local_rank
        else:
            assert local_world == cfg.grafter_target_world_size, (
                f"[Grafter] grafter_target_world_size={cfg.grafter_target_world_size} "
                f"but dist.get_world_size()={local_world}; they must match on the target side"
            )
            global_rank = cfg.grafter_baseline_world_size + local_rank
        total_world = cfg.grafter_baseline_world_size + cfg.grafter_target_world_size
        init_method = f"tcp://{cfg.grafter_master_address}:{cfg.grafter_master_port}"
        _log(
            f"[Grafter] init group: role={role.value} "
            f"baseline_world={cfg.grafter_baseline_world_size} "
            f"target_world={cfg.grafter_target_world_size} "
            f"rank={global_rank} init_method={init_method} "
            f"backend={cfg.grafter_backend} name={cfg.grafter_group_name}"
        )
        self._pg = _collective_with_timeout(
            lambda: _init_custom_process_group(
                backend=cfg.grafter_backend,
                init_method=init_method,
                world_size=total_world,
                rank=global_rank,
                group_name=cfg.grafter_group_name,
            ),
            operation_name="_init_custom_process_group in _Grafter",
            timeout_seconds=cfg.grafter_timeout,
        )
```
**EN:** Method `_ensure_group` implements behavior on `_Grafter`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_ensure_group` 为 `_Grafter` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1012-1030: Implement method `_apply_transform` for `_Grafter` / 为 `_Grafter` 实现方法 `_apply_transform`
```python
    def _apply_transform(
        self,
        *,
        tags: dict,
        received_list: list,
        received_extras_list: list,
        target: torch.Tensor,
    ) -> torch.Tensor:
        # TODO: integrate with dump_comparator unsharder annotations once
        # full inverse (sharded -> global -> sharded) transforms exist.
        graft_input = GraftTransformInput(
            tags=tags,
            received_list=received_list,
            received_extras_list=received_extras_list,
            target=target,
        )
        path = self._config.grafter_transform_path
        fn = self._default_transform if path is None else _load_function(path)
        return fn(graft_input)
```
**EN:** Method `_apply_transform` implements behavior on `_Grafter`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_apply_transform` 为 `_Grafter` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1033-1057: Implement method `_default_transform` for `_Grafter` / 为 `_Grafter` 实现方法 `_default_transform`
```python
    def _default_transform(graft_input: GraftTransformInput) -> torch.Tensor:
        """Identity-by-rank fallback. Requires #senders == #recvs and
        shape(received_list[my_recv_rank]) == shape(target). Otherwise raises
        and asks the user for a transform."""
        received_list = graft_input.received_list
        target = graft_input.target
        my_recv_rank = dist.get_rank()
        recv_world_size = dist.get_world_size()
        if len(received_list) != recv_world_size:
            raise RuntimeError(
                _Grafter._default_transform_error(
                    f"requires #senders == #recvs but got "
                    f"#senders={len(received_list)} vs #recvs={recv_world_size}"
                )
            )
        candidate = received_list[my_recv_rank]
        if candidate.shape != target.shape:
            raise RuntimeError(
                _Grafter._default_transform_error(
                    f"requires matching shapes but "
                    f"received_list[{my_recv_rank}].shape={tuple(candidate.shape)} "
                    f"!= target.shape={tuple(target.shape)}"
                )
            )
        return candidate
```
**EN:** Method `_default_transform` implements behavior on `_Grafter`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_default_transform` 为 `_Grafter` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1060-1066: Implement method `_default_transform_error` for `_Grafter` / 为 `_Grafter` 实现方法 `_default_transform_error`
```python
    def _default_transform_error(detail: str) -> str:
        return (
            f"[Grafter] no grafter_transform_path set; default identity-by-rank "
            f"{detail}. Provide a transform via "
            f"DUMPER_GRAFTER_TRANSFORM_PATH=pkg.module.symbol defining "
            f"`transform(graft_input: GraftTransformInput) -> Tensor`."
        )
```
**EN:** Method `_default_transform_error` implements behavior on `_Grafter`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_default_transform_error` 为 `_Grafter` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1072-1085: Implement helper `_torch_save` / 实现辅助函数 `_torch_save`
```python
def _torch_save(value, path: str):
    value = _clone_if_view(value)
    try:
        try:
            return torch.save(value, path)
        except RuntimeError as e:
            if "not pickleable" in str(e):
                stripped = _strip_parameter(value)
                if stripped is not value:
                    _log(f"Observe error={e} and try pickling .data")
                    return _torch_save(stripped, path)
            raise
    except Exception as e:
        _log(f"Observe error={e} when saving data, skip the tensor")
```
**EN:** Function `_torch_save` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_torch_save` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1088-1093: Implement helper `_map_tensor` / 实现辅助函数 `_map_tensor`
```python
def _map_tensor(value, fn: Callable[[torch.Tensor], torch.Tensor]):
    if isinstance(value, dict):
        return {k: _map_tensor(v, fn) for k, v in value.items()}
    if isinstance(value, torch.Tensor):
        return fn(value)
    return value
```
**EN:** Function `_map_tensor` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_map_tensor` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1096-1102: Implement helper `_clone_if_view` / 实现辅助函数 `_clone_if_view`
```python
def _clone_if_view(value):
    def _fn(t: torch.Tensor) -> torch.Tensor:
        if t.untyped_storage().nbytes() > t.nelement() * t.element_size():
            return t.clone()
        return t

    return _map_tensor(value, _fn)
```
**EN:** Function `_clone_if_view` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_clone_if_view` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1105-1111: Implement helper `_strip_parameter` / 实现辅助函数 `_strip_parameter`
```python
def _strip_parameter(value):
    def _fn(t: torch.Tensor) -> torch.Tensor:
        if isinstance(t, torch.nn.Parameter):
            return t.data
        return t

    return _map_tensor(value, _fn)
```
**EN:** Function `_strip_parameter` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_strip_parameter` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1114-1130: Implement helper `_collective_with_timeout` / 实现辅助函数 `_collective_with_timeout`
```python
def _collective_with_timeout(fn, operation_name: str, timeout_seconds: int = 60):
    completed = threading.Event()

    def watchdog():
        if not completed.wait(timeout=timeout_seconds):
            _log(
                f"WARNING: '{operation_name}' has not completed after "
                f"{timeout_seconds}s. This usually means not all ranks are "
                f"participating in this collective operation."
            )

    thread = threading.Thread(target=watchdog, daemon=True)
    thread.start()
    try:
        return fn()
    finally:
        completed.set()
```
**EN:** Function `_collective_with_timeout` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_collective_with_timeout` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1133-1157: Implement helper `_get_default_exp_name` / 实现辅助函数 `_get_default_exp_name`
```python
def _get_default_exp_name(timeout_seconds: int = 60):
    rank = _get_rank()
    now = time.time()
    ms = int((now % 1) * 1000)
    rand_suffix = random.randint(0, 999)
    object_list = [
        (
            (
                f"{_DEFAULT_EXP_NAME_PREFIX}"
                f"{time.strftime('%Y%m%d_%H%M%S', time.gmtime(now))}"
                f"_{ms:03d}{rand_suffix:03d}"
            )
            if rank == 0
            else None
        )
    ]

    if dist.is_initialized():
        _collective_with_timeout(
            lambda: dist.broadcast_object_list(object_list, device="cuda"),
            operation_name="broadcast_object_list in _get_default_exp_name",
            timeout_seconds=timeout_seconds,
        )

    return object_list[0]
```
**EN:** Function `_get_default_exp_name` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_get_default_exp_name` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1160-1177: Implement helper `_cleanup_old_dumps` / 实现辅助函数 `_cleanup_old_dumps`
```python
def _cleanup_old_dumps(base_dir: Path, exp_name: Optional[str] = None) -> None:
    import shutil

    if _get_rank() == 0:
        targets = {entry for entry in base_dir.glob(f"{_DEFAULT_EXP_NAME_PREFIX}*")}
        if exp_name:
            targets.add(base_dir / exp_name)
        targets = {d for d in targets if d.is_dir()}

        for entry in targets:
            shutil.rmtree(entry)
            _log(f"Cleaned up {entry}")

    if dist.is_initialized():
        _collective_with_timeout(
            dist.barrier,
            operation_name="barrier in _cleanup_old_dumps",
        )
```
**EN:** Function `_cleanup_old_dumps` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_cleanup_old_dumps` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1180-1184: Implement helper `_get_rank` / 实现辅助函数 `_get_rank`
```python
def _get_rank():
    if dist.is_initialized():
        return dist.get_rank()
    else:
        return 0
```
**EN:** Function `_get_rank` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_get_rank` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1187-1191: Implement helper `_get_world_size` / 实现辅助函数 `_get_world_size`
```python
def _get_world_size():
    if dist.is_initialized():
        return dist.get_world_size()
    else:
        return 1
```
**EN:** Function `_get_world_size` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_get_world_size` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1194-1196: Implement helper `_log` / 实现辅助函数 `_log`
```python
def _log(msg: str) -> None:
    """Print a log line tagged with the current rank and wall-clock time."""
    print(f"[Dumper, rank={_get_rank()}, t={time.time():.3f}] {msg}", flush=True)
```
**EN:** Function `_log` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_log` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1199-1216: Implement helper `_compare_tensors_quick` / 实现辅助函数 `_compare_tensors_quick`
```python
def _compare_tensors_quick(a: "torch.Tensor", b: "torch.Tensor") -> str:
    """One-line summary of how close two tensors are. Inspired by
    sglang.srt.debug_utils.dump_comparator._compute_and_print_diff;
    intentionally inlined here to keep dumper.py free of cross-file imports.

    Different dtypes are fine — we unify by casting both to fp32, which is
    enough for the order-of-magnitude diff summary we log."""
    if a.shape != b.shape:
        return f"shape mismatch (a={tuple(a.shape)} vs b={tuple(b.shape)})"
    if a.numel() == 0:
        return "empty"
    a_float = a.detach().to(torch.float32)
    b_float = b.detach().to(torch.float32)
    raw_abs = (a_float - b_float).abs()
    max_abs = raw_abs.max().item()
    mean_abs = raw_abs.mean().item()
    rel_diff = _calc_rel_diff(a_float, b_float).item()
    return f"rel_diff={rel_diff:.6g} max_abs={max_abs:.6g} mean_abs={mean_abs:.6g}"
```
**EN:** Function `_compare_tensors_quick` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_compare_tensors_quick` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1221-1225: Implement helper `_calc_rel_diff` / 实现辅助函数 `_calc_rel_diff`
```python
def _calc_rel_diff(x: "torch.Tensor", y: "torch.Tensor"):
    x, y = x.double(), y.double()
    denominator = (x * x + y * y).sum()
    sim = 2 * (x * y).sum() / denominator
    return 1 - sim
```
**EN:** Function `_calc_rel_diff` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_calc_rel_diff` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1228-1242: Implement helper `_obj_to_dict` / 实现辅助函数 `_obj_to_dict`
```python
def _obj_to_dict(obj):
    if isinstance(obj, dict):
        return obj
    ret = {}
    for k in dir(obj):
        if k.startswith("__") and k.endswith("__"):
            continue
        try:
            v = getattr(obj, k)
            if not callable(v):
                ret[k] = v
        except Exception:
            # Skip attributes that raise an exception on access
            continue
    return ret
```
**EN:** Function `_obj_to_dict` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_obj_to_dict` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1245-1248: Implement helper `_materialize_value` / 实现辅助函数 `_materialize_value`
```python
def _materialize_value(value):
    if callable(value):
        value = value()
    return value
```
**EN:** Function `_materialize_value` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_materialize_value` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1251-1252: Implement helper `_format_tags` / 实现辅助函数 `_format_tags`
```python
def _format_tags(kwargs: dict) -> str:
    return "___".join(f"{k}={v}" for k, v in kwargs.items())
```
**EN:** Function `_format_tags` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_tags` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1255-1255: Define class `_DefaultNoneDict` and class context / 定义类 `_DefaultNoneDict`及类上下文
```python
class _DefaultNoneDict(dict):
```
**EN:** This section introduces `_DefaultNoneDict`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_DefaultNoneDict`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 1256-1256: Document the module intent / 说明模块意图
```python
    """dict subclass that returns None for missing keys, for filter expression eval."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 1258-1259: Implement method `__missing__` for `_DefaultNoneDict` / 为 `_DefaultNoneDict` 实现方法 `__missing__`
```python
    def __missing__(self, key: str):
        return None
```
**EN:** Method `__missing__` implements behavior on `_DefaultNoneDict`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__missing__` 为 `_DefaultNoneDict` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1262-1262: Declare module-level symbols such as `_FILTER_BUILTINS` / 声明模块级符号，例如 `_FILTER_BUILTINS`
```python
_FILTER_BUILTINS: dict[str, Any] = {"search": re.search, "match": re.match}
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 1265-1273: Implement helper `_evaluate_filter` / 实现辅助函数 `_evaluate_filter`
```python
def _evaluate_filter(filter_expr: str, tags: dict[str, Any]) -> bool:
    """Evaluate a Python filter expression against the tags dict.

    Unknown tag keys resolve to None, so `layer_id is None` works when layer_id is absent.
    `re.search` and `re.match` are available as `search()` and `match()`.
    """
    namespace = _DefaultNoneDict(tags)
    namespace.update(_FILTER_BUILTINS)
    return bool(eval(filter_expr, {"__builtins__": {}}, namespace))
```
**EN:** Function `_evaluate_filter` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_evaluate_filter` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1276-1279: Implement helper `_deepcopy_or_clone` / 实现辅助函数 `_deepcopy_or_clone`
```python
def _deepcopy_or_clone(x):
    if isinstance(x, torch.Tensor):
        return x.clone()
    return deepcopy(x)
```
**EN:** Function `_deepcopy_or_clone` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_deepcopy_or_clone` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1285-1301: Implement helper `_compute_static_meta` / 实现辅助函数 `_compute_static_meta`
```python
def _compute_static_meta():
    result = {
        "world_rank": _get_rank(),
        "world_size": _get_world_size(),
    }

    for plugin in _plugins:
        if info := plugin.collect_parallel_info():
            result[f"{plugin.name}_parallel_info"] = info

    for plugin in _plugins:
        tokenizer_path: Optional[str] = plugin.get_tokenizer_path()
        if tokenizer_path is not None:
            result["tokenizer_path"] = tokenizer_path
            break

    return result
```
**EN:** Function `_compute_static_meta` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_compute_static_meta` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1307-1307: Define class `_DumperHttpManager` and class context / 定义类 `_DumperHttpManager`及类上下文
```python
class _DumperHttpManager:
```
**EN:** This section introduces `_DumperHttpManager`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_DumperHttpManager`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 1308-1325: Implement method `__init__` for `_DumperHttpManager` / 为 `_DumperHttpManager` 实现方法 `__init__`
```python
    def __init__(self, dumper: "_Dumper"):
        self._dumper = dumper
        http_port = self._dumper._config.server_port_parsed

        rpc_broadcast = _create_zmq_rpc_broadcast(
            self,
            timeout_seconds=self._dumper._config.collective_timeout,
        )

        if _get_rank() == 0:
            assert rpc_broadcast is not None
            self._rpc_broadcast = rpc_broadcast

            if http_port == "reuse":
                _log("Standalone HTTP server disabled, reusing existing ports")
            else:
                _start_http_server(prefix="/dumper/", target=self, http_port=http_port)
                _log(f"HTTP server started on port {http_port}")
```
**EN:** Method `__init__` implements behavior on `_DumperHttpManager`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__init__` 为 `_DumperHttpManager` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1329-1330: Implement method `handle_request` for `_DumperHttpManager` / 为 `_DumperHttpManager` 实现方法 `handle_request`
```python
    def handle_request(self, *, method: str, body: dict[str, Any]) -> list[dict]:
        return self._rpc_broadcast._handle_request_inner(method=method, body=body)
```
**EN:** Method `handle_request` implements behavior on `_DumperHttpManager`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `handle_request` 为 `_DumperHttpManager` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1334-1344: Implement method `_handle_request_inner` for `_DumperHttpManager` / 为 `_DumperHttpManager` 实现方法 `_handle_request_inner`
```python
    def _handle_request_inner(self, *, method: str, body: dict[str, Any]) -> dict:
        if method == "get_state":
            return self._dumper.get_state()
        elif method == "configure":
            self._dumper.configure(**body)
            return {}
        elif method == "reset":
            self._dumper.reset()
            return {}
        else:
            raise ValueError(f"Unknown dumper control method: {method!r}")
```
**EN:** Method `_handle_request_inner` implements behavior on `_DumperHttpManager`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_handle_request_inner` 为 `_DumperHttpManager` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1350-1354: Implement helper `_start_http_server` / 实现辅助函数 `_start_http_server`
```python
def _start_http_server(*, prefix: str, target: object, http_port: int):
    handler_class = _make_http_handler(prefix=prefix, target=target)
    server = HTTPServer(("0.0.0.0", http_port), handler_class)
    thread = threading.Thread(target=server.serve_forever, daemon=True)
    thread.start()
```
**EN:** Function `_start_http_server` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_start_http_server` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1357-1383: Implement helper `_make_http_handler` / 实现辅助函数 `_make_http_handler`
```python
def _make_http_handler(*, prefix: str, target):
    class _HTTPHandler(BaseHTTPRequestHandler):
        def do_POST(self):
            if not self.path.startswith(prefix):
                self.send_error(404)
                return
            method = self.path[len(prefix) :]
            try:
                req_body = self._get_request_body()
                _log(f"HTTP {self.path} {req_body=}")
                result = target.handle_request(method=method, body=req_body)
                resp_body = json.dumps(result).encode()
                self.send_response(200)
                self.send_header("Content-Type", "application/json")
                self.send_header("Content-Length", str(len(resp_body)))
                self.end_headers()
                self.wfile.write(resp_body)
            except Exception as e:
                self.send_error(400, str(e))

        def _get_request_body(self) -> dict:
            content_length = int(self.headers.get("Content-Length", 0))
            if content_length == 0:
                return {}
            return json.loads(self.rfile.read(content_length))

    return _HTTPHandler
```
**EN:** Function `_make_http_handler` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_make_http_handler` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1389-1438: Implement helper `_create_zmq_rpc_broadcast` / 实现辅助函数 `_create_zmq_rpc_broadcast`
```python
def _create_zmq_rpc_broadcast(
    handler, timeout_seconds: int = 60
) -> Optional["_ZmqRpcBroadcast"]:
    """A general-purpose minimal RPC to support broadcasting executions to multi processes"""
    import zmq

    rank = _get_rank()
    world_size = dist.get_world_size() if dist.is_initialized() else 1

    ctx = zmq.Context()
    sock = ctx.socket(zmq.REP)
    sock.bind("tcp://*:0")
    bound_port = int(sock.getsockopt_string(zmq.LAST_ENDPOINT).rsplit(":", 1)[1])
    local_addr = f"tcp://{_get_local_ip_by_remote()}:{bound_port}"

    def serve_loop():
        while True:
            try:
                req = sock.recv_pyobj()
                result = getattr(handler, req["method"])(*req["args"], **req["kwargs"])
                resp = {"result": result, "error": None}
            except Exception as e:
                _log(f"[ZmqRpc] error inside handler: {e}")
                resp = {"result": None, "error": str(e)}
            sock.send_pyobj(resp)

    thread = threading.Thread(target=serve_loop, daemon=True)
    thread.start()
    _log(f"[ZmqRpc] server started at {local_addr}")

    if dist.is_initialized():
        all_addresses = [None] * world_size
        _collective_with_timeout(
            lambda: dist.all_gather_object(all_addresses, local_addr),
            operation_name="all_gather_object in _create_zmq_rpc_broadcast",
            timeout_seconds=timeout_seconds,
        )
    else:
        all_addresses = [local_addr]
    _log(f"[ZmqRpc] all_addresses={all_addresses}")

    if rank == 0:
        handles = []
        for i, addr in enumerate(all_addresses):
            req_socket = ctx.socket(zmq.REQ)
            req_socket.connect(addr)
            handles.append(_ZmqRpcHandle(req_socket, debug_name=f"rank-{i}"))
        return _ZmqRpcBroadcast(handles)
    else:
        return None
```
**EN:** Function `_create_zmq_rpc_broadcast` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_create_zmq_rpc_broadcast` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1441-1441: Define class `_ZmqRpcHandle` and class context / 定义类 `_ZmqRpcHandle`及类上下文
```python
class _ZmqRpcHandle:
```
**EN:** This section introduces `_ZmqRpcHandle`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_ZmqRpcHandle`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 1442-1442: Document the module intent / 说明模块意图
```python
    """Proxy object to call remote handler methods via ZMQ."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 1444-1446: Implement method `__init__` for `_ZmqRpcHandle` / 为 `_ZmqRpcHandle` 实现方法 `__init__`
```python
    def __init__(self, socket, debug_name: str):
        self._socket = socket
        self._debug_name = debug_name
```
**EN:** Method `__init__` implements behavior on `_ZmqRpcHandle`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__init__` 为 `_ZmqRpcHandle` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1448-1464: Implement method `__getattr__` for `_ZmqRpcHandle` / 为 `_ZmqRpcHandle` 实现方法 `__getattr__`
```python
    def __getattr__(self, method_name: str):
        def call(*args, **kwargs):
            self._socket.send_pyobj(
                {
                    "method": method_name,
                    "args": args,
                    "kwargs": kwargs,
                }
            )
            response = self._socket.recv_pyobj()
            if response["error"]:
                raise RuntimeError(
                    f"RPC error on {self._debug_name}: {response['error']}"
                )
            return response["result"]

        return call
```
**EN:** Method `__getattr__` implements behavior on `_ZmqRpcHandle`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__getattr__` 为 `_ZmqRpcHandle` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1467-1467: Define class `_RpcBroadcastBase` and class context / 定义类 `_RpcBroadcastBase`及类上下文
```python
class _RpcBroadcastBase:
```
**EN:** This section introduces `_RpcBroadcastBase`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_RpcBroadcastBase`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 1468-1468: Document the module intent / 说明模块意图
```python
    """Base for broadcasting method calls to dumper instance(s)."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 1470-1471: Implement method `__getattr__` for `_RpcBroadcastBase` / 为 `_RpcBroadcastBase` 实现方法 `__getattr__`
```python
    def __getattr__(self, method_name: str):
        raise NotImplementedError
```
**EN:** Method `__getattr__` implements behavior on `_RpcBroadcastBase`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__getattr__` 为 `_RpcBroadcastBase` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1473-1474: Implement method `__init__` for `_RpcBroadcastBase` / 为 `_RpcBroadcastBase` 实现方法 `__init__`
```python
    def __init__(self, handles: List[_ZmqRpcHandle]):
        self._handles = handles
```
**EN:** Method `__init__` implements behavior on `_RpcBroadcastBase`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__init__` 为 `_RpcBroadcastBase` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1477-1477: Define class `_ZmqRpcBroadcast` and class context / 定义类 `_ZmqRpcBroadcast`及类上下文
```python
class _ZmqRpcBroadcast(_RpcBroadcastBase):
```
**EN:** This section introduces `_ZmqRpcBroadcast`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_ZmqRpcBroadcast`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 1478-1481: Document the module intent / 说明模块意图
```python
    """Broadcasts method calls to all ZMQ RPC handles.

    Returns a list of results, one per rank (ordered by rank).
    """
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 1483-1484: Implement method `__init__` for `_ZmqRpcBroadcast` / 为 `_ZmqRpcBroadcast` 实现方法 `__init__`
```python
    def __init__(self, handles: List[_ZmqRpcHandle]):
        self._handles = handles
```
**EN:** Method `__init__` implements behavior on `_ZmqRpcBroadcast`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__init__` 为 `_ZmqRpcBroadcast` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1486-1493: Implement method `__getattr__` for `_ZmqRpcBroadcast` / 为 `_ZmqRpcBroadcast` 实现方法 `__getattr__`
```python
    def __getattr__(self, method_name: str):
        def call(*args, **kwargs):
            return [
                getattr(handle, method_name)(*args, **kwargs)
                for handle in self._handles
            ]

        return call
```
**EN:** Method `__getattr__` implements behavior on `_ZmqRpcBroadcast`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__getattr__` 为 `_ZmqRpcBroadcast` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1499-1525: Implement helper `_get_local_ip_by_remote` / 实现辅助函数 `_get_local_ip_by_remote`
```python
def _get_local_ip_by_remote() -> Optional[str]:
    # try ipv4
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    try:
        s.connect(("8.8.8.8", 80))  # Doesn't need to be reachable
        return s.getsockname()[0]
    except Exception:
        pass

    try:
        hostname = socket.gethostname()
        ip = socket.gethostbyname(hostname)
        if ip and ip != "127.0.0.1" and ip != "0.0.0.0":
            return ip
    except Exception:
        pass

    # try ipv6
    try:
        s = socket.socket(socket.AF_INET6, socket.SOCK_DGRAM)
        # Google's public DNS server, see
        # https://developers.google.com/speed/public-dns/docs/using#addresses
        s.connect(("2001:4860:4860::8888", 80))  # Doesn't need to be reachable
        return s.getsockname()[0]
    except Exception:
        _log("Can not get local ip by remote")
    return None
```
**EN:** Function `_get_local_ip_by_remote` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_get_local_ip_by_remote` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1528-1544: Implement helper `_load_function` / 实现辅助函数 `_load_function`
```python
def _load_function(path: str) -> Callable:
    """Resolve a fully-qualified Python path 'pkg.module.symbol' to its object.

    Copied (verbatim, minus the function-registry branch) from
    miles.utils.misc.load_function — kept inline so dumper.py has no
    cross-package dependency.
    """
    import importlib

    module_path, _, attr = path.rpartition(".")
    if not module_path:
        raise ValueError(
            f"_load_function expects 'pkg.module.symbol', got {path!r} "
            f"(missing dotted prefix)"
        )
    module = importlib.import_module(module_path)
    return getattr(module, attr)
```
**EN:** Function `_load_function` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_load_function` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1547-1599: Implement helper `_init_custom_process_group` / 实现辅助函数 `_init_custom_process_group`
```python
def _init_custom_process_group(
    *,
    backend: str,
    init_method: str,
    world_size: int,
    rank: int,
    group_name: str,
    timeout=None,
):
    """Build a fresh torch.distributed process group, separate from the default
    one and any other custom groups (e.g. RLHF weight-update groups). Used by
    the grafter to bridge baseline and target systems.

    Adapted from sglang.srt.utils.common.init_custom_process_group; inlined
    here to keep dumper.py free of cross-file imports.
    """
    from torch.distributed.distributed_c10d import (
        Backend,
        PrefixStore,
        _new_process_group_helper,
        _world,
        default_pg_timeout,
        rendezvous,
    )

    if timeout is None:
        timeout = default_pg_timeout

    rendezvous_iterator = rendezvous(init_method, rank, world_size, timeout=timeout)
    store, rank, world_size = next(rendezvous_iterator)
    store.set_timeout(timeout)
    store = PrefixStore(group_name, store)

    backend_obj = Backend(backend)
    # PyTorch 2.6 renamed `pg_options` to `backend_options`.
    torch_major_minor = tuple(
        int(x) for x in torch.__version__.split("+")[0].split(".")[:2]
    )
    pg_options_param_name = (
        "backend_options" if torch_major_minor >= (2, 6) else "pg_options"
    )
    pg, _ = _new_process_group_helper(
        world_size,
        rank,
        [],
        backend_obj,
        store,
        group_name=group_name,
        **{pg_options_param_name: None},
        timeout=timeout,
    )
    _world.pg_group_ranks[pg] = {i: i for i in range(world_size)}
    return pg
```
**EN:** Function `_init_custom_process_group` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_init_custom_process_group` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1605-1605: Define class `_RecomputeStatus` and class context / 定义类 `_RecomputeStatus`及类上下文
```python
class _RecomputeStatus(enum.Enum):
```
**EN:** This section introduces `_RecomputeStatus`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_RecomputeStatus`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 1606-1608: Declare fields for `_RecomputeStatus` such as `DISABLED`, `ORIGINAL`, `RECOMPUTE` / 为 `_RecomputeStatus` 声明字段，例如 `DISABLED`, `ORIGINAL`, `RECOMPUTE`
```python
    DISABLED = "disabled"
    ORIGINAL = "original"  # inside checkpoint, original forward
    RECOMPUTE = "recompute"  # inside checkpoint, recompute forward
```
**EN:** These lines declare the state carried by `_RecomputeStatus`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_RecomputeStatus` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 1610-1616: Implement method `to_pseudo_parallel_meta` for `_RecomputeStatus` / 为 `_RecomputeStatus` 实现方法 `to_pseudo_parallel_meta`
```python
    def to_pseudo_parallel_meta(self) -> dict[str, Any]:
        if self == _RecomputeStatus.DISABLED:
            return {}
        return {
            "recompute_pseudo_rank": 1 if self == _RecomputeStatus.RECOMPUTE else 0,
            "recompute_pseudo_size": 2,
        }
```
**EN:** Method `to_pseudo_parallel_meta` implements behavior on `_RecomputeStatus`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `to_pseudo_parallel_meta` 为 `_RecomputeStatus` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1619-1621: Define class `_FrameworkPlugin` and class context / 定义类 `_FrameworkPlugin`及类上下文
```python
class _FrameworkPlugin(ABC):
    @property
    @abstractmethod
```
**EN:** This section introduces `_FrameworkPlugin`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_FrameworkPlugin`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 1622-1622: Implement method `name` for `_FrameworkPlugin` / 为 `_FrameworkPlugin` 实现方法 `name`
```python
    def name(self) -> str: ...
```
**EN:** Method `name` implements behavior on `_FrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `name` 为 `_FrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1625-1625: Implement method `collect_parallel_info` for `_FrameworkPlugin` / 为 `_FrameworkPlugin` 实现方法 `collect_parallel_info`
```python
    def collect_parallel_info(self) -> dict: ...
```
**EN:** Method `collect_parallel_info` implements behavior on `_FrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `collect_parallel_info` 为 `_FrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1628-1632: Implement method `convert_value` for `_FrameworkPlugin` / 为 `_FrameworkPlugin` 实现方法 `convert_value`
```python
    def convert_value(
        self, value: Any, *, skip_forward_batch: bool
    ) -> Optional[dict[str, Any]]:
        """Return converted dict, or None if this plugin doesn't handle the value."""
        ...
```
**EN:** Method `convert_value` implements behavior on `_FrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `convert_value` 为 `_FrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1635-1637: Implement method `detect_layer_id` for `_FrameworkPlugin` / 为 `_FrameworkPlugin` 实现方法 `detect_layer_id`
```python
    def detect_layer_id(self, module: "torch.nn.Module") -> Optional[int]:
        """Return 0-indexed layer_id, or None if not detectable."""
        ...
```
**EN:** Method `detect_layer_id` implements behavior on `_FrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `detect_layer_id` 为 `_FrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1639-1640: Implement method `core_fields` for `_FrameworkPlugin` / 为 `_FrameworkPlugin` 实现方法 `core_fields`
```python
    def core_fields(self) -> frozenset[str]:
        return frozenset()
```
**EN:** Method `core_fields` implements behavior on `_FrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `core_fields` 为 `_FrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1642-1643: Implement method `get_tokenizer_path` for `_FrameworkPlugin` / 为 `_FrameworkPlugin` 实现方法 `get_tokenizer_path`
```python
    def get_tokenizer_path(self) -> Optional[str]:
        return None
```
**EN:** Method `get_tokenizer_path` implements behavior on `_FrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `get_tokenizer_path` 为 `_FrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1645-1646: Implement method `detect_recompute_status` for `_FrameworkPlugin` / 为 `_FrameworkPlugin` 实现方法 `detect_recompute_status`
```python
    def detect_recompute_status(self) -> _RecomputeStatus:
        return _RecomputeStatus.DISABLED
```
**EN:** Method `detect_recompute_status` implements behavior on `_FrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `detect_recompute_status` 为 `_FrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1649-1649: Define class `_SGLangPlugin` and class context / 定义类 `_SGLangPlugin`及类上下文
```python
class _SGLangPlugin(_FrameworkPlugin):
```
**EN:** This section introduces `_SGLangPlugin`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_SGLangPlugin`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 1650-1650: Declare fields for `_SGLangPlugin` such as `_available` / 为 `_SGLangPlugin` 声明字段，例如 `_available`
```python
    _available = True
```
**EN:** These lines declare the state carried by `_SGLangPlugin`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_SGLangPlugin` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 1651-1660: Implement supporting module logic / 实现辅助模块逻辑
```python
    try:
        from sglang.srt import distributed as _dist
        from sglang.srt.layers import dp_attention as _dp_attn
        from sglang.srt.layers.logits_processor import LogitsProcessorOutput
        from sglang.srt.model_executor.forward_batch_info import (
            ForwardBatch,
            PPProxyTensors,
        )
    except ImportError:
        _available = False
```
**EN:** This source block contains supporting logic that helps the rest of the file operate correctly.
**CN:** 这个源码片段包含辅助逻辑，用于保证文件其余部分能够正确运行。

### Lines 1663-1664: Implement method `name` for `_SGLangPlugin` / 为 `_SGLangPlugin` 实现方法 `name`
```python
    def name(self) -> str:
        return "sglang"
```
**EN:** Method `name` implements behavior on `_SGLangPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `name` 为 `_SGLangPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1666-1699: Implement method `collect_parallel_info` for `_SGLangPlugin` / 为 `_SGLangPlugin` 实现方法 `collect_parallel_info`
```python
    def collect_parallel_info(self) -> dict:
        if not self._available:
            return {}

        info = {}

        try:
            info["tp_rank"] = self._dist.get_tensor_model_parallel_rank()
            info["tp_size"] = self._dist.get_tensor_model_parallel_world_size()
            info["pp_rank"] = self._dist.get_pipeline_model_parallel_rank()
            info["pp_size"] = self._dist.get_pipeline_model_parallel_world_size()
            info["moe_ep_rank"] = self._dist.get_moe_expert_parallel_rank()
            info["moe_ep_size"] = self._dist.get_moe_expert_parallel_world_size()
            info["moe_tp_rank"] = self._dist.get_moe_tensor_parallel_rank()
            info["moe_tp_size"] = self._dist.get_moe_tensor_parallel_world_size()
            info["moe_dp_rank"] = self._dist.get_moe_data_parallel_rank()
            info["moe_dp_size"] = self._dist.get_moe_data_parallel_world_size()
        except (AttributeError, AssertionError):
            info["distributed_error"] = True

        try:
            info["enable_dp_attention"] = self._dp_attn.is_dp_attention_enabled()
            info["attn_tp_rank"] = self._dp_attn.get_attention_tp_rank()
            info["attn_tp_size"] = self._dp_attn.get_attention_tp_size()
            info["attn_dp_rank"] = self._dp_attn.get_attention_dp_rank()
            info["attn_dp_size"] = self._dp_attn.get_attention_dp_size()
            info["local_attn_dp_rank"] = self._dp_attn.get_local_attention_dp_rank()
            info["local_attn_dp_size"] = self._dp_attn.get_local_attention_dp_size()
            info["attn_cp_rank"] = self._dp_attn.get_attention_cp_rank()
            info["attn_cp_size"] = self._dp_attn.get_attention_cp_size()
        except (AttributeError, AssertionError):
            info["dp_attention_error"] = True

        return info
```
**EN:** Method `collect_parallel_info` implements behavior on `_SGLangPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `collect_parallel_info` 为 `_SGLangPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1701-1724: Implement method `convert_value` for `_SGLangPlugin` / 为 `_SGLangPlugin` 实现方法 `convert_value`
```python
    def convert_value(
        self, value: Any, *, skip_forward_batch: bool
    ) -> Optional[dict[str, Any]]:
        if not self._available:
            return None

        if isinstance(value, self.LogitsProcessorOutput):
            return {"next_token_logits": value.next_token_logits}
        if isinstance(value, self.ForwardBatch):
            if skip_forward_batch:
                return {}
            result = {
                "input_ids": value.input_ids,
                "seq_lens": value.seq_lens,
                "positions": value.positions,
                "req_pool_indices": value.req_pool_indices,
            }
            if value.rids is not None:
                result["rids"] = value.rids
            return result
        if isinstance(value, self.PPProxyTensors):
            return {k: v for k, v in value.tensors.items()}

        return None
```
**EN:** Method `convert_value` implements behavior on `_SGLangPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `convert_value` 为 `_SGLangPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1726-1729: Implement method `detect_layer_id` for `_SGLangPlugin` / 为 `_SGLangPlugin` 实现方法 `detect_layer_id`
```python
    def detect_layer_id(self, module: "torch.nn.Module") -> Optional[int]:
        if hasattr(module, "layer_id"):
            return module.layer_id
        return None
```
**EN:** Method `detect_layer_id` implements behavior on `_SGLangPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `detect_layer_id` 为 `_SGLangPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1731-1734: Implement method `core_fields` for `_SGLangPlugin` / 为 `_SGLangPlugin` 实现方法 `core_fields`
```python
    def core_fields(self) -> frozenset[str]:
        return frozenset(
            {"input_ids", "positions", "seq_lens", "req_pool_indices", "rids"}
        )
```
**EN:** Method `core_fields` implements behavior on `_SGLangPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `core_fields` 为 `_SGLangPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1736-1749: Implement method `get_tokenizer_path` for `_SGLangPlugin` / 为 `_SGLangPlugin` 实现方法 `get_tokenizer_path`
```python
    def get_tokenizer_path(self) -> Optional[str]:
        if not self._available:
            return None

        try:
            from sglang.srt.server_args import get_global_server_args

            args = get_global_server_args()
            if args is None:
                return None

            return args.tokenizer_path
        except Exception:
            return None
```
**EN:** Method `get_tokenizer_path` implements behavior on `_SGLangPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `get_tokenizer_path` 为 `_SGLangPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1752-1752: Define class `_MegatronPlugin` and class context / 定义类 `_MegatronPlugin`及类上下文
```python
class _MegatronPlugin(_FrameworkPlugin):
```
**EN:** This section introduces `_MegatronPlugin`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_MegatronPlugin`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 1753-1753: Declare fields for `_MegatronPlugin` such as `_available` / 为 `_MegatronPlugin` 声明字段，例如 `_available`
```python
    _available = True
```
**EN:** These lines declare the state carried by `_MegatronPlugin`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_MegatronPlugin` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 1754-1758: Implement supporting module logic / 实现辅助模块逻辑
```python
    try:
        from megatron.core import parallel_state as _mpu
        from megatron.core.packed_seq_params import PackedSeqParams
    except ImportError:
        _available = False
```
**EN:** This source block contains supporting logic that helps the rest of the file operate correctly.
**CN:** 这个源码片段包含辅助逻辑，用于保证文件其余部分能够正确运行。

### Lines 1761-1762: Implement method `name` for `_MegatronPlugin` / 为 `_MegatronPlugin` 实现方法 `name`
```python
    def name(self) -> str:
        return "megatron"
```
**EN:** Method `name` implements behavior on `_MegatronPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `name` 为 `_MegatronPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1764-1812: Implement method `collect_parallel_info` for `_MegatronPlugin` / 为 `_MegatronPlugin` 实现方法 `collect_parallel_info`
```python
    def collect_parallel_info(self) -> dict:
        if not self._available:
            return {}

        info = {}
        try:
            info["tp_rank"] = self._mpu.get_tensor_model_parallel_rank()
            info["tp_size"] = self._mpu.get_tensor_model_parallel_world_size()
            info["pp_rank"] = self._mpu.get_pipeline_model_parallel_rank()
            info["pp_size"] = self._mpu.get_pipeline_model_parallel_world_size()
            info["dp_rank"] = self._mpu.get_data_parallel_rank()
            info["dp_size"] = self._mpu.get_data_parallel_world_size()
            info["cp_rank"] = self._mpu.get_context_parallel_rank()
            info["cp_size"] = self._mpu.get_context_parallel_world_size()
            info["vpp_rank"] = self._mpu.get_virtual_pipeline_model_parallel_rank()
            info["vpp_size"] = (
                self._mpu.get_virtual_pipeline_model_parallel_world_size()
            )
            info["ep_rank"] = self._mpu.get_expert_model_parallel_rank()
            info["ep_size"] = self._mpu.get_expert_model_parallel_world_size()
            info["etp_rank"] = self._mpu.get_expert_tensor_parallel_rank()
            info["etp_size"] = self._mpu.get_expert_tensor_parallel_world_size()
            info["edp_rank"] = self._mpu.get_expert_data_parallel_rank()
            info["edp_size"] = self._mpu.get_expert_data_parallel_world_size()
            info["tcp_rank"] = self._mpu.get_tensor_and_context_parallel_rank()
            info["tcp_size"] = self._mpu.get_tensor_and_context_parallel_world_size()
            info["etmp_rank"] = self._mpu.get_expert_tensor_and_model_parallel_rank()
            info["etmp_size"] = (
                self._mpu.get_expert_tensor_and_model_parallel_world_size()
            )
            info["tp_src_rank"] = self._mpu.get_tensor_model_parallel_src_rank()
            info["mp_src_rank"] = self._mpu.get_model_parallel_src_rank()
            info["dp_src_rank"] = self._mpu.get_data_parallel_src_rank()
        except (AttributeError, AssertionError):
            info["megatron_error"] = True

        # Megatron sequence parallel reuses the TP group (no dedicated parallel state API).
        # When sequence_parallel=True, inject sp_rank/sp_size for the comparator unsharder.
        try:
            from megatron.training.global_vars import get_args

            args = get_args()
            if getattr(args, "sequence_parallel", False) and "tp_rank" in info:
                info["sp_rank"] = info["tp_rank"]
                info["sp_size"] = info["tp_size"]
        except (ImportError, AssertionError, AttributeError):
            pass

        return info
```
**EN:** Method `collect_parallel_info` implements behavior on `_MegatronPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `collect_parallel_info` 为 `_MegatronPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1814-1825: Implement method `convert_value` for `_MegatronPlugin` / 为 `_MegatronPlugin` 实现方法 `convert_value`
```python
    def convert_value(
        self, value: Any, *, skip_forward_batch: bool
    ) -> Optional[dict[str, Any]]:
        if not self._available:
            return None
        if isinstance(value, self.PackedSeqParams):
            return {
                "cu_seqlens_q": value.cu_seqlens_q,
                "cu_seqlens_kv": value.cu_seqlens_kv,
                "qkv_format": value.qkv_format,
            }
        return None
```
**EN:** Method `convert_value` implements behavior on `_MegatronPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `convert_value` 为 `_MegatronPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1827-1830: Implement method `detect_layer_id` for `_MegatronPlugin` / 为 `_MegatronPlugin` 实现方法 `detect_layer_id`
```python
    def detect_layer_id(self, module: "torch.nn.Module") -> Optional[int]:
        if hasattr(module, "layer_number"):
            return module.layer_number - 1
        return None
```
**EN:** Method `detect_layer_id` implements behavior on `_MegatronPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `detect_layer_id` 为 `_MegatronPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1832-1835: Implement method `core_fields` for `_MegatronPlugin` / 为 `_MegatronPlugin` 实现方法 `core_fields`
```python
    def core_fields(self) -> frozenset[str]:
        return frozenset(
            {"input_ids", "position_ids", "cu_seqlens_q", "cu_seqlens_kv", "qkv_format"}
        )
```
**EN:** Method `core_fields` implements behavior on `_MegatronPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `core_fields` 为 `_MegatronPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1837-1849: Implement method `detect_recompute_status` for `_MegatronPlugin` / 为 `_MegatronPlugin` 实现方法 `detect_recompute_status`
```python
    def detect_recompute_status(self) -> _RecomputeStatus:
        if not self._available:
            return _RecomputeStatus.DISABLED
        try:
            from megatron.core.tensor_parallel.random import is_checkpointing

            if not is_checkpointing():
                return _RecomputeStatus.DISABLED
            if torch.is_grad_enabled():
                return _RecomputeStatus.RECOMPUTE
            return _RecomputeStatus.ORIGINAL
        except (ImportError, AttributeError):
            return _RecomputeStatus.DISABLED
```
**EN:** Method `detect_recompute_status` implements behavior on `_MegatronPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `detect_recompute_status` 为 `_MegatronPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 1852-1852: Declare module-level symbols such as `_plugins` / 声明模块级符号，例如 `_plugins`
```python
_plugins: list[_FrameworkPlugin] = [_SGLangPlugin(), _MegatronPlugin()]
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 1855-1860: Implement helper `_detect_recompute_status` / 实现辅助函数 `_detect_recompute_status`
```python
def _detect_recompute_status() -> _RecomputeStatus:
    for plugin in _plugins:
        info = plugin.detect_recompute_status()
        if info != _RecomputeStatus.DISABLED:
            return info
    return _RecomputeStatus.DISABLED
```
**EN:** Function `_detect_recompute_status` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_detect_recompute_status` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1866-1866: Declare module-level symbols such as `dumper` / 声明模块级符号，例如 `dumper`
```python
dumper = _Dumper(config=DumperConfig.from_env())
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 1872-1886: Implement function `get_truncated_value` / 实现函数 `get_truncated_value`
```python
def get_truncated_value(value):
    if value is None:
        return None

    if isinstance(value, tuple):
        return [get_truncated_value(x) for x in value]

    if not isinstance(value, torch.Tensor):
        return value

    if value.numel() < 200:
        return value

    slices = [slice(0, 5) if dim_size > 50 else slice(None) for dim_size in value.shape]
    return value[tuple(slices)]
```
**EN:** Function `get_truncated_value` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `get_truncated_value` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 1889-1914: Implement function `get_tensor_info` / 实现函数 `get_tensor_info`
```python
def get_tensor_info(x):
    """
    from sglang.srt.debug_utils.dumper import get_tensor_info
    """
    if not isinstance(x, torch.Tensor):
        return f"type={type(x)} value={x}"
    min = x.float().min() if x.numel() > 0 else None
    max = x.float().max() if x.numel() > 0 else None
    mean = x.float().mean() if x.numel() > 0 else None
    torch.set_printoptions(precision=10)
    x_sample_head = str(x.flatten()[:5])
    x_sample_tail = str(x.flatten()[-5:])
    torch.set_printoptions(precision=4)
    return (
        f"type={type(x)} "
        f"shape={x.shape} "
        f"dtype={x.dtype} "
        f"device={x.device} "
        f"stride={x.stride()} "
        f"req_grad={x.requires_grad} "
        f"min={min} "
        f"max={max} "
        f"mean={mean} "
        f"x_sample_head={x_sample_head} "
        f"x_sample_tail={x_sample_tail}"
    )
```
**EN:** Function `get_tensor_info` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `get_tensor_info` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `_BaseConfig`, `DumperConfig`, `_DumperState`, `_Dumper`, `_NonIntrusiveDumper`, `_GraftRole`, `_GraftDirection`, `GraftTransformInput`, `_Grafter`, `_DefaultNoneDict`, `_DumperHttpManager`, `_ZmqRpcHandle`
- **Module role / 模块角色**: Shared debugging utilities / 共享调试工具
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `enum`, `functools`, `json`, `os`, `random`, `re`, `socket`, `threading`, `time`, `traceback`, `abc`, `collections`, `contextlib`, `copy`, `dataclasses`, `http`, `pathlib`, `typing`, `shutil`, `importlib`
- **Third-party / 第三方**: `torch`, `zmq`, `megatron`
- **Internal / 内部**: `sglang.srt.debug_utils.source_patcher`, `sglang.srt`, `sglang.srt.layers`, `sglang.srt.layers.logits_processor`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.server_args`
