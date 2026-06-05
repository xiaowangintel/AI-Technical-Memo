# lora_registry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/lora/lora_registry.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements core LoRA runtime logic such as adapter metadata, memory management, layer wrapping, or orchestration. It is part of SGLang's infrastructure for serving models with dynamic adapters. / 该文件实现了核心 LoRA 运行时逻辑，例如适配器元数据、内存管理、层封装或整体编排。它属于 SGLang 支持动态适配器服务能力的基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25: Module header, imports, and shared constants
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================


import asyncio
from collections import OrderedDict
from dataclasses import dataclass, field, fields
from typing import Dict, List, Optional, Union
from uuid import NAMESPACE_URL, uuid4, uuid5

from sglang.srt.utils import ConcurrentCounter
from sglang.srt.utils.aio_rwlock import RWLock
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 27-40: LoRARef class declaration
```python
class LoRARef:
    """
    Reference record for a LoRA model.

    This object guarantees a unique ``lora_id`` and may include ``lora_name``, ``lora_path``, and ``pinned``.
    The ID eliminates conflicts from reused LoRA names or paths and can be used to generate deterministic cache
    keys (e.g., radix cache).
    """

    lora_id: str = field(default_factory=lambda: uuid4().hex)
    lora_name: Optional[str] = None
    lora_path: Optional[str] = None
    pinned: Optional[bool] = None
```
**EN:** This block declares the `LoRARef` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `LoRARef` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 41-43: LoRARef.__post_init__ method
```python
    def __post_init__(self):
        if self.lora_id is None:
            raise ValueError("lora_id cannot be None")
```
**EN:** This block uses `LoRARef.__post_init__` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRARef.__post_init__` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 45-53: LoRARef.deterministic_id method
```python
    @staticmethod
    def deterministic_id(lora_name: str, lora_path: str) -> str:
        """Stable ``lora_id`` for ``--lora-paths`` adapters.

        Each node in a multi-node launch parses ``--lora-paths`` independently;
        ``uuid4`` would mint a different id per node for the same adapter,
        breaking cross-node lookups when the master broadcasts a request id.
        """
        return uuid5(NAMESPACE_URL, f"{lora_name}\0{lora_path}").hex
```
**EN:** This block uses `LoRARef.deterministic_id` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRARef.deterministic_id` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 55-61: LoRARef.__str__ method
```python
    def __str__(self) -> str:
        parts = [
            f"{f.name}={value}"
            for f in fields(self)
            if (value := getattr(self, f.name)) is not None
        ]
        return f"{self.__class__.__name__}({', '.join(parts)})"
```
**EN:** This block uses `LoRARef.__str__` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRARef.__str__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 64-72: LoRARegistry class declaration
```python
class LoRARegistry:
    """
    The central registry to keep track of available LoRA adapters and ongoing LoRA requests.

    The `LoRARegistry` resides in the tokenizer manager process and acts as the single source of truth for all
    available LoRA adapters. It supports concurrent inference and dynamic adapter updates through a two-phase
    update / eventual consistency model between the tokenizer manager process and the scheduler processes.
    """
```
**EN:** This block declares the `LoRARegistry` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `LoRARegistry` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 73-96: LoRARegistry initializer
```python
    def __init__(self, lora_paths: Optional[List[LoRARef]] = None):
        assert lora_paths is None or all(
            isinstance(lora, LoRARef) for lora in lora_paths
        ), (
            "server_args.lora_paths should have been normalized to LoRARef objects during server initialization. "
            "Please file an issue if you see this error."
        )

        # A read-write lock to ensure adapters loading / unloading operations are exclusive.
        # Please note that the counter increment/decrement operations are not synchronized through this
        # lock, as they are designed to be non-blocking and can be performed concurrently.
        self._registry_lock = RWLock()
        # An ordered dictionary to hold LoRARef objects, mapping from LoRA name to LoRARef.
        # The LoRARefs are stored in LRU order, such that LoRA adapters that have been
        # most recently used are stored at the end. Note that lookups count for accesses.
        # Ties are broken arbitrarily.
        self._registry: OrderedDict[str, LoRARef] = OrderedDict()
        # Counters for ongoing requests, mapping from LoRA ID to ConcurrentCounter.
        self._counters: Dict[str, ConcurrentCounter] = {}

        # Initialize the registry with provided LoRA paths, if present.
        if lora_paths:
            for lora_ref in lora_paths:
                self._register_adapter(lora_ref)
```
**EN:** This block initializes the `LoRARegistry` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `LoRARegistry` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 98-106: LoRARegistry.register method
```python
    async def register(self, lora_ref: LoRARef):
        """
        Register a new LoRARef object in the registry.

        Args:
            lora_ref (LoRARef): The LoRARef object to register.
        """
        async with self._registry_lock.writer_lock:
            self._register_adapter(lora_ref)
```
**EN:** This block uses `LoRARegistry.register` to register components for later lookup. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRARegistry.register` 来注册组件以供后续查找。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 108-123: LoRARegistry.unregister method
```python
    async def unregister(self, lora_name: str) -> str:
        """
        Unregister a LoRARef object from the registry and returns the removed LoRA ID.

        Args:
            lora_name (str): The name of the LoRA model to unregister.
        """
        async with self._registry_lock.writer_lock:
            lora_ref = self._registry.get(lora_name, None)
            if lora_ref is None:
                raise ValueError(
                    f"LoRA with name {lora_name} does not exist. Loaded LoRAs: {self._registry.keys()}"
                )
            del self._registry[lora_name]

        return lora_ref.lora_id
```
**EN:** This block uses `LoRARegistry.unregister` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRARegistry.unregister` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 125-164: LoRARegistry.acquire method
```python
    async def acquire(self, lora_name: Union[str, List[str]]) -> Union[str, List[str]]:
        """
        Queries registry for LoRA IDs based on LoRA names and start tracking the usage of the corresponding LoRA adapters
        by incrementing its counter.
        """

        def _lookup(name: str) -> str:
            if name is None:
                return None

            lora_ref = self._registry.get(name, None)
            if lora_ref is None:
                raise ValueError(
                    f"The following requested LoRA adapters are not loaded: {name}\n"
                    f"Loaded adapters: {self._registry.keys()}."
                )
            self._registry.move_to_end(name)
            return lora_ref.lora_id

        if isinstance(lora_name, str):
            async with self._registry_lock.writer_lock:
                lora_id = _lookup(lora_name)

            await self._counters[lora_id].increment(notify_all=False)
            return lora_id
        elif isinstance(lora_name, list):
            async with self._registry_lock.writer_lock:
                lora_ids = [_lookup(name) for name in lora_name]

            # Increment the counters only after all IDs are looked up.
            await asyncio.gather(
                *[
                    self._counters[id].increment(notify_all=False)
                    for id in lora_ids
                    if id is not None
                ]
            )
            return lora_ids
        else:
            raise TypeError("lora_name must be either a string or a list of strings.")
```
**EN:** This block uses `LoRARegistry.acquire` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRARegistry.acquire` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 166-183: LoRARegistry.release method
```python
    async def release(self, lora_id: Union[str, List[str]]):
        """
        Decrements the usage counter for a LoRA adapter, indicating that it is no longer in use.
        """

        async with self._registry_lock.reader_lock:
            if isinstance(lora_id, str):
                await self._counters[lora_id].decrement()
            elif isinstance(lora_id, list):
                await asyncio.gather(
                    *[
                        self._counters[id].decrement()
                        for id in lora_id
                        if id is not None
                    ]
                )
            else:
                raise TypeError("lora_id must be either a string or a list of strings.")
```
**EN:** This block uses `LoRARegistry.release` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRARegistry.release` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 185-202: LoRARegistry.wait_for_unload method
```python
    async def wait_for_unload(self, lora_id: str):
        """
        Waits until the usage counter for a LoRA adapter reaches zero, indicating that it is no longer in use.
        This is useful for ensuring that a LoRA adapter can be safely unloaded.

        This method itself is not synchronized, which is safe because it should only be called during LoRA unloading,
        which itself is guaranteed to be sequential.
        """
        assert (
            lora_id not in self._registry
        ), "wait_for_unload should only be called after the LoRA adapter has been unregistered. "
        assert (
            lora_id in self._counters
        ), "The LoRA ID should still have a counter if it has been registered before."

        # Wait until no requests are using this LoRA adapter.
        await self._counters[lora_id].wait_for_zero()
        del self._counters[lora_id]
```
**EN:** This block uses `LoRARegistry.wait_for_unload` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRARegistry.wait_for_unload` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 204-218: LoRARegistry.get_unregistered_loras method
```python
    async def get_unregistered_loras(self, lora_name: set[str]):
        """
        Returns all LoRA adapters in lora_name that are not found in self._registry.
        """
        async with self._registry_lock.writer_lock:
            unregistered_loras = []

            for name in lora_name:
                if name in self._registry:
                    # This counts as a lookup, so we want to update the cache
                    self._registry.move_to_end(name)
                else:
                    unregistered_loras.append(name)

            return unregistered_loras
```
**EN:** This block uses `LoRARegistry.get_unregistered_loras` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRARegistry.get_unregistered_loras` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 220-233: LoRARegistry.lru_lora_name method
```python
    async def lru_lora_name(self, exclude_pinned=False):
        """
        Returns the least recently used LoRA adapter.
        If exclude_pinned is True, then return the LRU LoRA adapter that isn't pinned.
        """
        async with self._registry_lock.reader_lock:
            if not exclude_pinned:
                return next(iter(self._registry), None)

            for lora_name, lora_ref in self._registry.items():
                if not lora_ref.pinned:
                    return lora_name
            else:
                return None
```
**EN:** This block uses `LoRARegistry.lru_lora_name` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRARegistry.lru_lora_name` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 235-246: LoRARegistry._register_adapter method
```python
    def _register_adapter(self, lora_ref: LoRARef):
        """
        Internal helper method to register a LoRA adapter.
        """

        if lora_ref.lora_name in self._registry:
            raise ValueError(
                f"LoRA with name {lora_ref.lora_name} already exists. Loaded LoRAs: {self._registry.keys()}"
            )
        self._registry[lora_ref.lora_name] = lora_ref
        self._counters[lora_ref.lora_id] = ConcurrentCounter()
        return lora_ref
```
**EN:** This block uses `LoRARegistry._register_adapter` to register components for later lookup. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRARegistry._register_adapter` 来注册组件以供后续查找。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 248-253: LoRARegistry.num_registered_loras method
```python
    @property
    def num_registered_loras(self) -> int:
        """
        Returns the total number of LoRA adapters currently registered.
        """
        return len(self._registry)
```
**EN:** This block uses `LoRARegistry.num_registered_loras` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRARegistry.num_registered_loras` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 255-262: LoRARegistry.get_all_adapters method
```python
    def get_all_adapters(self) -> Dict[str, LoRARef]:
        """
        Returns a dictionary of all registered LoRA adapters.

        Returns:
            Dict[str, LoRARef]: A dictionary mapping LoRA names to LoRARef objects.
        """
        return dict(self._registry)
```
**EN:** This block uses `LoRARegistry.get_all_adapters` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRARegistry.get_all_adapters` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- LoRA adapter execution / LoRA 适配器执行
- Registration and lookup / 注册与查找

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.utils`
- `sglang.srt.utils.aio_rwlock`
### External / 外部
- `asyncio` (stdlib)
- `collections` (stdlib)
- `dataclasses` (stdlib)
- `typing` (stdlib)
- `uuid` (stdlib)
