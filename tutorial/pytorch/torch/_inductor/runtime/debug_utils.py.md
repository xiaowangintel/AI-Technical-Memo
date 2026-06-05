# debug_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/runtime/debug_utils.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module provides runtime helpers used after compilation. It defines classes such as `BufferMemoryTracker`. It exposes functions such as `get_mem_tracker`, `track_tensor`, `tracked_empty_strided`, `check_memory_step`, and `register_check_mem_op`.
- **用途（中文）**: 该模块提供编译完成后使用的运行期辅助逻辑。其中定义了 `BufferMemoryTracker` 等类。同时提供 `get_mem_tracker`、`track_tensor`、`tracked_empty_strided`、`check_memory_step`、`register_check_mem_op` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
import functools
import logging
import threading
import weakref

import torch
from torch.utils._ordered_set import OrderedSet


log = logging.getLogger(__name__)

local = threading.local()
local.memory_tracker = None

````
- **EN**: Imports dependencies such as `functools`, `logging`, `threading`, `weakref`, `torch`, and `torch.utils._ordered_set` for the logic in this range. Initializes or updates values such as `log`, and `local`.
- **CN**: 这里导入了 `functools`、`logging`、`threading`、`weakref`、`torch`、`torch.utils._ordered_set` 等依赖，为后续逻辑提供基础能力。初始化或更新了 `log`、`local` 等值。

### Lines 15-28 / 第 15-28 行
````python

class BufferMemoryTracker:
    """
    Tracks inductor runtime allocations and deallocations to compare against
    expected behavior.
    """

    def __init__(self) -> None:
        self.tensor_tracker: dict[str, torch.storage.UntypedStorage] = (
            weakref.WeakValueDictionary()  # type: ignore[assignment]
        )
        self.died_since_last_step: OrderedSet[str] = OrderedSet()
        self.added_since_last_step: OrderedSet[str] = OrderedSet()
        self.error = (
````
- **EN**: Introduces class `BufferMemoryTracker`, function `__init__`.
- **CN**: 这里定义了类`BufferMemoryTracker`、函数`__init__`。

### Lines 29-42 / 第 29-42 行
````python
            torch._inductor.config.test_configs.track_memory_lifecycle == "assert"
        )

    def set_tensor(self, name: str, tensor: torch.Tensor) -> None:
        storage = tensor.untyped_storage()

        self.added_since_last_step.add(name)
        self.tensor_tracker[name] = storage

        def on_tensor_death() -> None:
            self.died_since_last_step.add(name)

        weakref.finalize(storage, on_tensor_death)

````
- **EN**: Introduces function `set_tensor`, function `on_tensor_death`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `storage`.
- **CN**: 这里定义了函数`set_tensor`、函数`on_tensor_death`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `storage` 等值。

### Lines 43-56 / 第 43-56 行
````python
    def advance_step(self) -> None:
        self.died_since_last_step.clear()
        self.added_since_last_step.clear()

    def log_or_raise(self, msg: str) -> None:
        if self.error:
            raise RuntimeError(msg)
        else:
            log.info(msg)

    def check_step_delta(
        self,
        expected_allocated: list[str],
        expected_freed: list[str],
````
- **EN**: Introduces function `advance_step`, function `log_or_raise`, function `check_step_delta`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `expected_allocated`, and `expected_freed`.
- **CN**: 这里定义了函数`advance_step`、函数`log_or_raise`、函数`check_step_delta`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`expected_allocated`、`expected_freed` 等值。

### Lines 57-70 / 第 57-70 行
````python
        is_final_step: bool,
    ) -> None:
        """Check only the delta changes since last step"""

        # Check expected deaths - we dont currently distinguish between nodes which die in last step
        # and are returned as outputs, so skip if final_step.
        if not is_final_step:
            missing_deaths = OrderedSet(expected_freed) - self.died_since_last_step
            if missing_deaths:
                self.log_or_raise(
                    f"Expected tensors to die but still alive: {missing_deaths}"
                )

        # Check for unexpected deaths
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `is_final_step`, and `missing_deaths`. This range continues the implementation of function `BufferMemoryTracker.check_step_delta`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `is_final_step`、`missing_deaths` 等值。这一段延续了函数`BufferMemoryTracker.check_step_delta` 的具体实现。

### Lines 71-84 / 第 71-84 行
````python
        unexpected_deaths = self.died_since_last_step - OrderedSet(expected_freed)
        if unexpected_deaths:
            self.log_or_raise(f"Unexpected tensor deaths: {unexpected_deaths}")

        # Check newly alive tensors - separate messages like deaths
        actual_allocated = self.added_since_last_step
        expected_allocated_set = OrderedSet(expected_allocated)

        extra_alive = actual_allocated - expected_allocated_set
        if extra_alive:
            self.log_or_raise(f"Unexpected allocated tensors: {extra_alive}")

        missing_alive = expected_allocated_set - actual_allocated
        if missing_alive:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `unexpected_deaths`, `actual_allocated`, `expected_allocated_set`, `extra_alive`, and `missing_alive`. This range continues the implementation of function `BufferMemoryTracker.check_step_delta`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `unexpected_deaths`、`actual_allocated`、`expected_allocated_set`、`extra_alive`、`missing_alive` 等值。这一段延续了函数`BufferMemoryTracker.check_step_delta` 的具体实现。

### Lines 85-98 / 第 85-98 行
````python
            self.log_or_raise(
                f"Expected allocated tensors but missing: {missing_alive}"
            )

        # Reset for next step
        self.advance_step()

        if is_final_step:
            local.memory_tracker = None


def get_mem_tracker() -> BufferMemoryTracker:
    if local.memory_tracker is None:
        local.memory_tracker = BufferMemoryTracker()
````
- **EN**: Introduces function `get_mem_tracker`. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_mem_tracker`。包含分支、循环或上下文管理等控制流。

### Lines 99-112 / 第 99-112 行
````python
    return local.memory_tracker


def track_tensor(tensor: torch.Tensor, name: str) -> None:
    get_mem_tracker().set_tensor(name, tensor)


def tracked_empty_strided(
    size: list[int],
    stride: list[int],
    *,
    dtype: torch.dtype,
    device: torch.device,
    name: str,
````
- **EN**: Introduces function `track_tensor`, function `tracked_empty_strided`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`track_tensor`、函数`tracked_empty_strided`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 113-126 / 第 113-126 行
````python
) -> torch.Tensor:
    o = torch.empty_strided(size, stride, dtype=dtype, device=device)
    track_tensor(o, name)
    return o


def check_memory_step(
    allocated: list[str], freed: list[str], is_final_step: bool = False
) -> None:
    tracker = get_mem_tracker()
    tracker.check_step_delta(allocated, freed, is_final_step)


@functools.lru_cache(None)
````
- **EN**: Introduces function `check_memory_step`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`check_memory_step`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。

### Lines 127-138 / 第 127-138 行
````python
def register_check_mem_op() -> None:
    lib = torch.library.Library("_inductor_debug", "FRAGMENT")  # noqa: TOR901
    lib.define(
        "check_memory_step(str[] allocated, str[] freed, bool is_final_step) -> ()"
    )
    lib.impl("check_memory_step", check_memory_step, "BackendSelect")
    from torch._higher_order_ops.effects import _EffectType, _register_effectful_op

    _register_effectful_op(
        torch.ops._inductor_debug.check_memory_step.default,
        _EffectType.ORDERED,
    )
````
- **EN**: Imports dependencies such as `torch._higher_order_ops.effects` for the logic in this range. Introduces function `register_check_mem_op`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `lib`.
- **CN**: 这里导入了 `torch._higher_order_ops.effects` 等依赖，为后续逻辑提供基础能力。这里定义了函数`register_check_mem_op`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `lib` 等值。

## Key Concepts / 关键概念
- **EN**: Provides runtime helpers used after compilation  
  **CN**: 提供编译完成后使用的运行期辅助逻辑
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `BufferMemoryTracker`  
  **CN**: 主要类：`BufferMemoryTracker`
- **EN**: Primary functions: `get_mem_tracker`, `track_tensor`, `tracked_empty_strided`, `check_memory_step`, and `register_check_mem_op`  
  **CN**: 主要函数：`get_mem_tracker`、`track_tensor`、`tracked_empty_strided`、`check_memory_step`、`register_check_mem_op`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `logging`, `threading`, `weakref`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.utils._ordered_set`, `torch._higher_order_ops.effects`
