# driver.py — Code Analysis / 代码分析

## Source / 来源

- **Path:** `./python/triton/runtime/driver.py`
- **EN:** This source file at `./python/triton/runtime/driver.py` defines the main symbols `DriverConfig`, `_create_driver` for the surrounding Triton subsystem.
- **CN:** 位于 `./python/triton/runtime/driver.py` 的这个源文件为周边 Triton 子系统定义了主要符号 `DriverConfig`, `_create_driver`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1
```python
from __future__ import annotations
```
**EN:** At module scope, this block imports annotations from `__future__` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `__future__` 导入 annotations，把当前文件与周边 API 和辅助工具连接起来。

### Lines 3-3
```python
import os
```
**EN:** At module scope, this block imports os so later definitions can reuse those modules or symbols.
**CN:** 在模块级作用域中，这段代码导入了 os，供后续定义复用这些模块或符号。

### Lines 5-5
```python
from ..backends import backends, DriverBase
```
**EN:** At module scope, this block imports backends, DriverBase from `..backends` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `..backends` 导入 backends, DriverBase，把当前文件与周边 API 和辅助工具连接起来。

### Lines 8-8
```python
def _create_driver() -> DriverBase:
```
**EN:** At module scope, this header declares the function `_create_driver()`, which is responsible for create driver.
**CN:** 在模块级作用域中，这段头部声明了函数 `_create_driver()`，它负责处理 create driver 相关逻辑。

### Lines 9-9
```python
    selected = os.environ.get("TRITON_DEFAULT_BACKEND", None)
```
**EN:** Inside function `_create_driver`, this assignment updates `selected` with `os.environ.get('TRITON_DEFAULT_BACKEND', None)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `_create_driver` 内部，这段赋值把 `os.environ.get('TRITON_DEFAULT_BACKEND', None)` 写入 `selected`，为后续逻辑建立状态、别名或配置。

### Lines 10-21
```python
    if selected:
        if selected not in backends:
            raise RuntimeError(f"Unknown backend device '{selected}'. Available backends: {list(backends.keys())}")
        driver = backends[selected].driver
        if not driver.is_active():
            raise RuntimeError(f"Backend device '{selected}' is not active.")
        return driver()
    else:
        active_drivers = [x.driver for x in backends.values() if x.driver.is_active()]
        if len(active_drivers) != 1:
            raise RuntimeError(f"{len(active_drivers)} active drivers ({active_drivers}). There should only be one.")
        return active_drivers[0]()
```
**EN:** Inside function `_create_driver`, this conditional checks `selected` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `_create_driver` 内部，这段条件语句检查 `selected`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 24-25
```python
class DriverConfig:
```
**EN:** At module scope, this header defines class `DriverConfig`, a container for driver config related behavior.
**CN:** 在模块级作用域中，这段头部定义了类 `DriverConfig`，用于封装 driver config 相关行为。

### Lines 26-26
```python
    def __init__(self) -> None:
```
**EN:** Inside class `DriverConfig`, this header declares the function `__init__(self)`, which is responsible for object initialization.
**CN:** 在类 `DriverConfig` 内部，这段头部声明了函数 `__init__(self)`，它负责处理 对象初始化 相关逻辑。

### Lines 27-27
```python
        self._default: DriverBase | None = None
```
**EN:** Inside class `DriverConfig` and function `__init__`, this assignment updates `self._default` with `None`, establishing state, aliases, or configuration used later.
**CN:** 在类 `DriverConfig`、函数 `__init__` 内部，这段赋值把 `None` 写入 `self._default`，为后续逻辑建立状态、别名或配置。

### Lines 28-28
```python
        self._active: DriverBase | None = None
```
**EN:** Inside class `DriverConfig` and function `__init__`, this assignment updates `self._active` with `None`, establishing state, aliases, or configuration used later.
**CN:** 在类 `DriverConfig`、函数 `__init__` 内部，这段赋值把 `None` 写入 `self._active`，为后续逻辑建立状态、别名或配置。

### Lines 30-31
```python
    @property
    def default(self) -> DriverBase:
```
**EN:** Inside class `DriverConfig`, this header declares the function `default(self)`, which is responsible for default. Decorators: property.
**CN:** 在类 `DriverConfig` 内部，这段头部声明了函数 `default(self)`，它负责处理 default 相关逻辑。 装饰器包括：property。

### Lines 32-33
```python
        if self._default is None:
            self._default = _create_driver()
```
**EN:** Inside class `DriverConfig` and function `default`, this conditional checks `self._default is None` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在类 `DriverConfig`、函数 `default` 内部，这段条件语句检查 `self._default is None`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 34-34
```python
        return self._default
```
**EN:** Inside class `DriverConfig` and function `default`, this return statement sends `self._default` back to the caller as the result of the current routine.
**CN:** 在类 `DriverConfig`、函数 `default` 内部，这条返回语句把 `self._default` 作为当前过程的结果返回给调用方。

### Lines 36-37
```python
    @property
    def active(self) -> DriverBase:
```
**EN:** Inside class `DriverConfig`, this header declares the function `active(self)`, which is responsible for active. Decorators: property.
**CN:** 在类 `DriverConfig` 内部，这段头部声明了函数 `active(self)`，它负责处理 active 相关逻辑。 装饰器包括：property。

### Lines 38-39
```python
        if self._active is None:
            self._active = self.default
```
**EN:** Inside class `DriverConfig` and function `active`, this conditional checks `self._active is None` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在类 `DriverConfig`、函数 `active` 内部，这段条件语句检查 `self._active is None`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 40-40
```python
        return self._active
```
**EN:** Inside class `DriverConfig` and function `active`, this return statement sends `self._active` back to the caller as the result of the current routine.
**CN:** 在类 `DriverConfig`、函数 `active` 内部，这条返回语句把 `self._active` 作为当前过程的结果返回给调用方。

### Lines 42-42
```python
    def set_active(self, driver: DriverBase) -> None:
```
**EN:** Inside class `DriverConfig`, this header declares the function `set_active(self, driver)`, which is responsible for set active.
**CN:** 在类 `DriverConfig` 内部，这段头部声明了函数 `set_active(self, driver)`，它负责处理 set active 相关逻辑。

### Lines 43-43
```python
        self._active = driver
```
**EN:** Inside class `DriverConfig` and function `set_active`, this assignment updates `self._active` with `driver`, establishing state, aliases, or configuration used later.
**CN:** 在类 `DriverConfig`、函数 `set_active` 内部，这段赋值把 `driver` 写入 `self._active`，为后续逻辑建立状态、别名或配置。

### Lines 45-45
```python
    def reset_active(self) -> None:
```
**EN:** Inside class `DriverConfig`, this header declares the function `reset_active(self)`, which is responsible for reset active.
**CN:** 在类 `DriverConfig` 内部，这段头部声明了函数 `reset_active(self)`，它负责处理 reset active 相关逻辑。

### Lines 46-46
```python
        self._active = self.default
```
**EN:** Inside class `DriverConfig` and function `reset_active`, this assignment updates `self._active` with `self.default`, establishing state, aliases, or configuration used later.
**CN:** 在类 `DriverConfig`、函数 `reset_active` 内部，这段赋值把 `self.default` 写入 `self._active`，为后续逻辑建立状态、别名或配置。

### Lines 49-49
```python
driver = DriverConfig()
```
**EN:** At module scope, this assignment updates `driver` with `DriverConfig()`, establishing state, aliases, or configuration used later.
**CN:** 在模块级作用域中，这段赋值把 `DriverConfig()` 写入 `driver`，为后续逻辑建立状态、别名或配置。

## Key Concepts / 关键概念

- **EN:** Path theme: `python/triton/runtime` places this module in Triton's triton / runtime area.
  **CN:** 路径主题：`python/triton/runtime` 表明该模块位于 Triton 的 triton / runtime 领域。
- **EN:** Primary classes: `DriverConfig`.
  **CN:** 主要类：`DriverConfig`。
- **EN:** Primary functions: `_create_driver`.
  **CN:** 主要函数：`_create_driver`。
- **EN:** Runtime behavior: this file affects execution-time services such as launching, caching, or device management.
  **CN:** 运行时行为：该文件影响启动、缓存、设备管理等执行期服务。

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: __future__, os.
  **CN:** 标准库依赖：__future__, os。
- **EN:** Internal Triton modules: ..backends.
  **CN:** Triton 内部模块：..backends。
