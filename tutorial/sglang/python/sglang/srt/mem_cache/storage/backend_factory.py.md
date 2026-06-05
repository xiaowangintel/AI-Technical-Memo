# backend_factory.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/storage/backend_factory.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the backend factory logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的后端工厂相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: License header and introductory comments / 许可证头与说明注释
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to SGLang project

```
**EN:** Preserves licensing information and introductory commentary for the module.
**CN:** 保留模块的许可证信息以及开场说明文字。

### Lines 4-8: Imports and setup / 导入与初始化
```python
import importlib
import logging
from typing import TYPE_CHECKING, Any, Dict

from sglang.srt.mem_cache.hicache_storage import HiCacheStorage, HiCacheStorageConfig
```
**EN:** Imports `importlib`, `logging`, `typing`, `sglang.srt.mem_cache.hicache_storage` and other helpers used by the surrounding scope.
**CN:** 导入 `importlib`, `logging`, `typing`, `sglang.srt.mem_cache.hicache_storage` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 10-11: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    pass
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 13-13: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `logger`.
**CN:** 定义模块级变量，例如 `logger`。

### Lines 16-18: StorageBackendFactory declaration / StorageBackendFactory 声明
```python
class StorageBackendFactory:
    """Factory for creating storage backend instances with support for dynamic loading."""

```
**EN:** Factory for creating storage backend instances with support for dynamic loading. Declares the `StorageBackendFactory` class.
**CN:** 声明 `StorageBackendFactory` 类。

### Lines 19-19: Shared state definitions / 共享状态定义
```python
    _registry: Dict[str, Dict[str, Any]] = {}
```
**EN:** Defines class-level variables such as `_registry`.
**CN:** 定义类级变量，例如 `_registry`。

### Lines 21-41: _load_backend_class implementation / _load_backend_class 实现
```python
    @staticmethod
    def _load_backend_class(
        module_path: str, class_name: str, backend_name: str
    ) -> type[HiCacheStorage]:
        """Load and validate a backend class from module path."""
        try:
            module = importlib.import_module(module_path)
            backend_class = getattr(module, class_name)
            if not issubclass(backend_class, HiCacheStorage):
                raise TypeError(
                    f"Backend class {class_name} must inherit from HiCacheStorage"
                )
# ... omitted for brevity ...
        except AttributeError as e:
            raise AttributeError(
                f"Class '{class_name}' not found in module '{module_path}': {e}"
            ) from e
```
**EN:** Load and validate a backend class from module path. Implements the load backend class routine for this scope. It belongs to `StorageBackendFactory`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 实现当前作用域中的load后端类例程。 该方法属于 `StorageBackendFactory`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 43-63: register_backend implementation / register_backend 实现
```python
    @classmethod
    def register_backend(cls, name: str, module_path: str, class_name: str) -> None:
        """Register a storage backend with lazy loading.

        Args:
            name: Backend identifier
            module_path: Python module path containing the backend class
            class_name: Name of the backend class
        """
        if name in cls._registry:
            logger.warning(f"Backend '{name}' is already registered, overwriting")

# ... omitted for brevity ...
            "loader": loader,
            "module_path": module_path,
            "class_name": class_name,
        }
```
**EN:** Register a storage backend with lazy loading. Registers metadata so other components can discover this object. It belongs to `StorageBackendFactory`. It returns a computed result to its caller.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `StorageBackendFactory`。它会向调用方返回计算结果。

### Lines 65-111: create_backend implementation / create_backend 实现
```python
    @classmethod
    def create_backend(
        cls,
        backend_name: str,
        storage_config: HiCacheStorageConfig,
        mem_pool_host: Any,
        **kwargs,
    ) -> HiCacheStorage:
        """Create a storage backend instance.
        Args:
            backend_name: Name of the backend to create
            storage_config: Storage configuration
# ... omitted for brevity ...
        raise ValueError(
            f"Unknown storage backend '{backend_name}'. "
            f"Registered backends: {available_backends}. "
        )
```
**EN:** Create a storage backend instance. Constructs a new object or resource with the requested configuration. It belongs to `StorageBackendFactory`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 按给定配置构造新的对象或资源。 该方法属于 `StorageBackendFactory`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 113-150: _create_dynamic_backend implementation / _create_dynamic_backend 实现
```python
    @classmethod
    def _create_dynamic_backend(
        cls,
        backend_config: Dict[str, Any],
        storage_config: HiCacheStorageConfig,
        mem_pool_host: Any,
        **kwargs,
    ) -> HiCacheStorage:
        """Create a backend dynamically from configuration."""
        required_fields = ["backend_name", "module_path", "class_name"]
        for field in required_fields:
            if field not in backend_config:
# ... omitted for brevity ...
            logger.error(
                f"Failed to create dynamic storage backend '{backend_name}': {e}"
            )
            raise
```
**EN:** Create a backend dynamically from configuration. Implements the create dynamic backend routine for this scope. It belongs to `StorageBackendFactory`. It validates error cases explicitly. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的create动态后端例程。 该方法属于 `StorageBackendFactory`。它会显式处理错误场景。实现过程中会遍历输入或受管条目。

### Lines 152-189: _create_builtin_backend implementation / _create_builtin_backend 实现
```python
    @classmethod
    def _create_builtin_backend(
        cls,
        backend_name: str,
        backend_class: type[HiCacheStorage],
        storage_config: HiCacheStorageConfig,
        mem_pool_host: Any,
    ) -> HiCacheStorage:
        """Create built-in backend with original initialization logic."""
        if backend_name == "file":
            return backend_class(storage_config)
        elif backend_name == "nixl":
# ... omitted for brevity ...
        elif backend_name == "simm":
            return backend_class(storage_config, mem_pool_host)
        else:
            raise ValueError(f"Unknown built-in backend: {backend_name}")
```
**EN:** Create built-in backend with original initialization logic. Implements the create builtin backend routine for this scope. It belongs to `StorageBackendFactory`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 实现当前作用域中的create builtin后端例程。 该方法属于 `StorageBackendFactory`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 190-192: Comment block / 注释块
```python


# Register built-in storage backends
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 193-195: Control flow block / 控制流代码块
```python
StorageBackendFactory.register_backend(
    "file", "sglang.srt.mem_cache.hicache_storage", "HiCacheFile"
)
```
**EN:** Implements top-level control flow needed for setup or execution.
**CN:** 实现顶层初始化或执行所需的控制流。

### Lines 197-201: Control flow block / 控制流代码块
```python
StorageBackendFactory.register_backend(
    "nixl",
    "sglang.srt.mem_cache.storage.nixl.hicache_nixl",
    "HiCacheNixl",
)
```
**EN:** Implements top-level control flow needed for setup or execution.
**CN:** 实现顶层初始化或执行所需的控制流。

### Lines 203-207: Control flow block / 控制流代码块
```python
StorageBackendFactory.register_backend(
    "mooncake",
    "sglang.srt.mem_cache.storage.mooncake_store.mooncake_store",
    "MooncakeStore",
)
```
**EN:** Implements top-level control flow needed for setup or execution.
**CN:** 实现顶层初始化或执行所需的控制流。

### Lines 209-213: Control flow block / 控制流代码块
```python
StorageBackendFactory.register_backend(
    "hf3fs",
    "sglang.srt.mem_cache.storage.hf3fs.storage_hf3fs",
    "HiCacheHF3FS",
)
```
**EN:** Implements top-level control flow needed for setup or execution.
**CN:** 实现顶层初始化或执行所需的控制流。

### Lines 215-219: Control flow block / 控制流代码块
```python
StorageBackendFactory.register_backend(
    "aibrix",
    "sglang.srt.mem_cache.storage.aibrix_kvcache.aibrix_kvcache_storage",
    "AibrixKVCacheStorage",
)
```
**EN:** Implements top-level control flow needed for setup or execution.
**CN:** 实现顶层初始化或执行所需的控制流。

### Lines 221-225: Control flow block / 控制流代码块
```python
StorageBackendFactory.register_backend(
    "eic",
    "sglang.srt.mem_cache.storage.eic.eic_storage",
    "EICStorage",
)
```
**EN:** Implements top-level control flow needed for setup or execution.
**CN:** 实现顶层初始化或执行所需的控制流。

### Lines 227-231: Control flow block / 控制流代码块
```python
StorageBackendFactory.register_backend(
    "simm",
    "sglang.srt.mem_cache.storage.simm.hicache_simm",
    "HiCacheSiMM",
)
```
**EN:** Implements top-level control flow needed for setup or execution.
**CN:** 实现顶层初始化或执行所需的控制流。

## Key Concepts / 关键概念
- **`StorageBackendFactory`**: Defines the `StorageBackendFactory` type and its core responsibilities. / 定义 `StorageBackendFactory` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `importlib`, `logging`, `typing`
- **Internal / 内部**: `sglang.srt.mem_cache.hicache_storage`
