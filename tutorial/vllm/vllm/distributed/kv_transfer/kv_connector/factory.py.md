# factory.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/factory.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements KV-transfer connector interfaces and factories. / 实现 KV 传输连接器接口与工厂逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
import importlib
from collections.abc import Callable
from typing import TYPE_CHECKING, cast

from vllm.distributed.kv_transfer.kv_connector.base import (
    KVConnectorBase,
    KVConnectorBaseType,
)
from vllm.distributed.kv_transfer.kv_connector.v1 import (
    KVConnectorRole,
    supports_hma,
)
from vllm.logger import init_logger
from vllm.utils.func_utils import supports_kw
```
**EN:** This block imports `importlib`, `collections.abc`, `typing`, `vllm.distributed.kv_transfer.kv_connector.base`, `vllm.distributed.kv_transfer.kv_connector.v1`, `vllm.logger` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `importlib`, `collections.abc`, `typing`, `vllm.distributed.kv_transfer.kv_connector.base`, `vllm.distributed.kv_transfer.kv_connector.v1`, `vllm.logger`，为后续实现准备运行时、类型与辅助 API。

### Conditional block / 条件代码块
```python
if TYPE_CHECKING:
    from vllm.config import VllmConfig
    from vllm.config.kv_transfer import KVTransferConfig
    from vllm.v1.kv_cache_interface import KVCacheConfig
```
**EN:** This conditional keeps imports and declarations that are only needed for static type checking out of the runtime path.
**CN:** 该条件块将仅用于静态类型检查的导入与声明隔离在运行时路径之外。

### Module constants / 模块常量
```python
logger = init_logger(__name__)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`，供后续代码复用。

### Class `KVConnectorFactory` / 类 `KVConnectorFactory`
```python
class KVConnectorFactory:
    _registry: dict[str, Callable[[], type[KVConnectorBase]]] = {}

    @classmethod
    def register_connector(cls, name: str, module_path: str, class_name: str) -> None:
        """Register a connector with a lazy-loading module and class name."""
        if name in cls._registry:
            raise ValueError(f"Connector '{name}' is already registered.")

        def loader() -> type[KVConnectorBase]:
            module = importlib.import_module(module_path)
            return getattr(module, class_name)

        cls._registry[name] = loader

    @classmethod
    def create_connector(
        cls,
        config: "VllmConfig",
        role: KVConnectorRole,
        kv_cache_config: "KVCacheConfig",
    ) -> KVConnectorBase:
        kv_transfer_config = config.kv_transfer_config
        if kv_transfer_config is None:
            raise ValueError("kv_transfer_config must be set to create a connector")
        connector_cls = cls.get_connector_class(kv_transfer_config)

        # check if the connector supports HMA
        hma_enabled = not config.scheduler_config.disable_hybrid_kv_cache_manager
        if hma_enabled and not supports_hma(connector_cls):
            raise ValueError(
                f"Connector {connector_cls.__name__} does not support HMA but "
                f"HMA is enabled. Please set `--disable-hybrid-kv-cache-manager`."
            )

        logger.info(
            "Creating v1 connector with name: %s and engine_id: %s",
            connector_cls.__name__,
            kv_transfer_config.engine_id,
        )
        # NOTE(Kuntai): v1 connector is explicitly separated into two roles.
        # Scheduler connector:
        # - Co-locate with scheduler process
        # - Should only be used inside the Scheduler class
        # Worker connector:
# ... truncated for analysis ...
                    "connectors must accept kv_cache_config as the third "
                    "constructor argument and pass it to super().__init__()."
                )
                logger.error(msg)
                raise ValueError(msg)
        elif connector_name in cls._registry:
            connector_cls = cls._registry[connector_name]()
        else:
            raise ValueError(f"Unsupported connector type: {connector_name}")
        return connector_cls
```
**EN:** Declares `KVConnectorFactory`, a class. Key methods include `register_connector`, `create_connector`, `get_connector_class_by_name`, `get_connector_class`.
**CN:** 声明 `KVConnectorFactory`，它是一个类。 关键方法包括 `register_connector`, `create_connector`, `get_connector_class_by_name`, `get_connector_class`。

### Expr block / Expr 代码块
```python
KVConnectorFactory.register_connector(
    "ExampleConnector",
    "vllm.distributed.kv_transfer.kv_connector.v1.example_connector",
    "ExampleConnector",
)
```
**EN:** This top-level `Expr` block contributes supporting module logic or declarations.
**CN:** 该顶层 `Expr` 代码块为模块补充辅助逻辑或声明。

### Expr block / Expr 代码块
```python
KVConnectorFactory.register_connector(
    "ExampleHiddenStatesConnector",
    "vllm.distributed.kv_transfer.kv_connector.v1.example_hidden_states_connector",
    "ExampleHiddenStatesConnector",
)
```
**EN:** This top-level `Expr` block contributes supporting module logic or declarations.
**CN:** 该顶层 `Expr` 代码块为模块补充辅助逻辑或声明。

### Expr block / Expr 代码块
```python
KVConnectorFactory.register_connector(
    "P2pNcclConnector",
    "vllm.distributed.kv_transfer.kv_connector.v1.p2p.p2p_nccl_connector",
    "P2pNcclConnector",
)
```
**EN:** This top-level `Expr` block contributes supporting module logic or declarations.
**CN:** 该顶层 `Expr` 代码块为模块补充辅助逻辑或声明。

### Expr block / Expr 代码块
```python
KVConnectorFactory.register_connector(
    "LMCacheConnectorV1",
    "vllm.distributed.kv_transfer.kv_connector.v1.lmcache_connector",
    "LMCacheConnectorV1",
)
```
**EN:** This top-level `Expr` block contributes supporting module logic or declarations.
**CN:** 该顶层 `Expr` 代码块为模块补充辅助逻辑或声明。

### Expr block / Expr 代码块
```python
KVConnectorFactory.register_connector(
    "LMCacheMPConnector",
    "vllm.distributed.kv_transfer.kv_connector.v1.lmcache_mp_connector",
    "LMCacheMPConnector",
)
```
**EN:** This top-level `Expr` block contributes supporting module logic or declarations.
**CN:** 该顶层 `Expr` 代码块为模块补充辅助逻辑或声明。

### Expr block / Expr 代码块
```python
KVConnectorFactory.register_connector(
    "NixlConnector",
    "vllm.distributed.kv_transfer.kv_connector.v1.nixl",
    "NixlConnector",
)
```
**EN:** This top-level `Expr` block contributes supporting module logic or declarations.
**CN:** 该顶层 `Expr` 代码块为模块补充辅助逻辑或声明。

### Expr block / Expr 代码块
```python
KVConnectorFactory.register_connector(
    "MultiConnector",
    "vllm.distributed.kv_transfer.kv_connector.v1.multi_connector",
    "MultiConnector",
)
```
**EN:** This top-level `Expr` block contributes supporting module logic or declarations.
**CN:** 该顶层 `Expr` 代码块为模块补充辅助逻辑或声明。

### Expr block / Expr 代码块
```python
KVConnectorFactory.register_connector(
    "MoRIIOConnector",
    "vllm.distributed.kv_transfer.kv_connector.v1.moriio.moriio_connector",
    "MoRIIOConnector",
)
```
**EN:** This top-level `Expr` block contributes supporting module logic or declarations.
**CN:** 该顶层 `Expr` 代码块为模块补充辅助逻辑或声明。

### Expr block / Expr 代码块
```python
KVConnectorFactory.register_connector(
    "OffloadingConnector",
    "vllm.distributed.kv_transfer.kv_connector.v1.offloading_connector",
    "OffloadingConnector",
)
```
**EN:** This top-level `Expr` block contributes supporting module logic or declarations.
**CN:** 该顶层 `Expr` 代码块为模块补充辅助逻辑或声明。

### Expr block / Expr 代码块
```python
KVConnectorFactory.register_connector(
    "DecodeBenchConnector",
    "vllm.distributed.kv_transfer.kv_connector.v1.decode_bench_connector",
    "DecodeBenchConnector",
)
```
**EN:** This top-level `Expr` block contributes supporting module logic or declarations.
**CN:** 该顶层 `Expr` 代码块为模块补充辅助逻辑或声明。

### Expr block / Expr 代码块
```python
KVConnectorFactory.register_connector(
    "MooncakeConnector",
    "vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_connector",
    "MooncakeConnector",
)
```
**EN:** This top-level `Expr` block contributes supporting module logic or declarations.
**CN:** 该顶层 `Expr` 代码块为模块补充辅助逻辑或声明。

### Expr block / Expr 代码块
```python
KVConnectorFactory.register_connector(
    "MooncakeStoreConnector",
    "vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store.connector",
    "MooncakeStoreConnector",
)
```
**EN:** This top-level `Expr` block contributes supporting module logic or declarations.
**CN:** 该顶层 `Expr` 代码块为模块补充辅助逻辑或声明。

### Expr block / Expr 代码块
```python
KVConnectorFactory.register_connector(
    "FlexKVConnectorV1",
    "vllm.distributed.kv_transfer.kv_connector.v1.flexkv_connector",
    "FlexKVConnectorV1",
)
```
**EN:** This top-level `Expr` block contributes supporting module logic or declarations.
**CN:** 该顶层 `Expr` 代码块为模块补充辅助逻辑或声明。

### Expr block / Expr 代码块
```python
KVConnectorFactory.register_connector(
    "SimpleCPUOffloadConnector",
    "vllm.distributed.kv_transfer.kv_connector.v1.simple_cpu_offload_connector",
    "SimpleCPUOffloadConnector",
)
```
**EN:** This top-level `Expr` block contributes supporting module logic or declarations.
**CN:** 该顶层 `Expr` 代码块为模块补充辅助逻辑或声明。

### Expr block / Expr 代码块
```python
KVConnectorFactory.register_connector(
    "HF3FSKVConnector",
    "vllm.distributed.kv_transfer.kv_connector.v1.hf3fs.hf3fs_connector",
    "HF3FSKVConnector",
)
```
**EN:** This top-level `Expr` block contributes supporting module logic or declarations.
**CN:** 该顶层 `Expr` 代码块为模块补充辅助逻辑或声明。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `KVConnectorFactory`: class interface or data carrier / `KVConnectorFactory`：类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `importlib`, `collections.abc`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal modules / 内部模块**: `vllm.distributed.kv_transfer.kv_connector.base`, `vllm.distributed.kv_transfer.kv_connector.v1`, `vllm.logger`, `vllm.utils.func_utils`, `vllm.config`, `vllm.config.kv_transfer`, `vllm.v1.kv_cache_interface`
