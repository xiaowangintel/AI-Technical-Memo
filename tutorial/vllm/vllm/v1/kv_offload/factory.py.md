# factory.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/kv_offload/factory.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `OffloadingSpecFactory` for the V1 `kv_offload` subsystem. / 为 V1 的 `kv_offload` 子系统实现 `OffloadingSpecFactory`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import importlib
from collections.abc import Callable
from typing import TYPE_CHECKING

from vllm.logger import init_logger
from vllm.v1.kv_offload.base import OffloadingSpec

if TYPE_CHECKING:
    from vllm.config import VllmConfig
    from vllm.v1.kv_cache_interface import KVCacheConfig

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `OffloadingSpecFactory` class / `OffloadingSpecFactory` 类
```python
class OffloadingSpecFactory:
    _registry: dict[str, Callable[[], type[OffloadingSpec]]] = {}
```
**EN:** Introduces the `OffloadingSpecFactory` class. Core methods include `register_spec`, `create_spec`.
**CN:** 这里定义 `OffloadingSpecFactory` 类。核心方法包括 `register_spec`, `create_spec`。

### `OffloadingSpecFactory.register_spec` method / `OffloadingSpecFactory.register_spec` 方法
```python
    @classmethod
    def register_spec(cls, name: str, module_path: str, class_name: str) -> None:
        """Register a spec with a lazy-loading module and class name."""
        if name in cls._registry:
            raise ValueError(f"Connector '{name}' is already registered.")

        def loader() -> type[OffloadingSpec]:
            module = importlib.import_module(module_path)
            return getattr(module, class_name)

        cls._registry[name] = loader
```
**EN:** This method implements `register_spec` within `OffloadingSpecFactory`. The docstring frames it as: Register a spec with a lazy-loading module and class name. Key calls include `ValueError`, `import_module`, `getattr`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `register_spec`，其作用域位于`OffloadingSpecFactory`。 关键调用包括 `ValueError`, `import_module`, `getattr`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `OffloadingSpecFactory.create_spec` method / `OffloadingSpecFactory.create_spec` 方法
```python
    @classmethod
    def create_spec(
        cls,
        config: "VllmConfig",
        kv_cache_config: "KVCacheConfig",
    ) -> OffloadingSpec:
        kv_transfer_config = config.kv_transfer_config
        assert kv_transfer_config is not None
        extra_config = kv_transfer_config.kv_connector_extra_config
        spec_name = extra_config.get("spec_name", "CPUOffloadingSpec")
        if spec_name in cls._registry:
            spec_cls = cls._registry[spec_name]()
        else:
            spec_module_path = extra_config.get("spec_module_path")
            if spec_module_path is None:
                raise ValueError(f"Unsupported spec type: {spec_name}")
            spec_module = importlib.import_module(spec_module_path)
            spec_cls = getattr(spec_module, spec_name)
        assert issubclass(spec_cls, OffloadingSpec)
        logger.info("Creating offloading spec with name: %s", spec_name)
        return spec_cls(config, kv_cache_config)
```
**EN:** This method creates a new object or plan within `OffloadingSpecFactory`. Key calls include `get`, `issubclass`, `info`, `spec_cls`, `import_module`, `getattr`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会创建新的对象或计划，其作用域位于`OffloadingSpecFactory`。 关键调用包括 `get`, `issubclass`, `info`, `spec_cls`, `import_module`, `getattr`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### Module setup / 模块初始化
```python
OffloadingSpecFactory.register_spec(
    "CPUOffloadingSpec", "vllm.v1.kv_offload.cpu.spec", "CPUOffloadingSpec"
)
OffloadingSpecFactory.register_spec(
    "TieringOffloadingSpec",
    "vllm.v1.kv_offload.tiering.spec",
    "TieringOffloadingSpec",
)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

## Key Concepts / 关键概念
- `OffloadingSpecFactory`: central class or interface in this module. / `OffloadingSpecFactory`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `importlib`, `collections`, `typing`
- Internal vLLM / 内部依赖: `vllm.logger`, `vllm.v1.kv_offload.base`, `vllm.config`, `vllm.v1.kv_cache_interface`
