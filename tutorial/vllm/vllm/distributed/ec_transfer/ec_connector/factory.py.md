# factory.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/ec_transfer/ec_connector/factory.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements EC-transfer connectors and runtime state. / 实现 EC 传输连接器与运行时状态。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
import importlib
from collections.abc import Callable
from typing import TYPE_CHECKING

from vllm.distributed.ec_transfer.ec_connector.base import (
    ECConnectorBase,
    ECConnectorRole,
)
from vllm.logger import init_logger
```
**EN:** This block imports `importlib`, `collections.abc`, `typing`, `vllm.distributed.ec_transfer.ec_connector.base`, `vllm.logger` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `importlib`, `collections.abc`, `typing`, `vllm.distributed.ec_transfer.ec_connector.base`, `vllm.logger`，为后续实现准备运行时、类型与辅助 API。

### Conditional block / 条件代码块
```python
if TYPE_CHECKING:
    from vllm.config import ECTransferConfig, VllmConfig
```
**EN:** This conditional keeps imports and declarations that are only needed for static type checking out of the runtime path.
**CN:** 该条件块将仅用于静态类型检查的导入与声明隔离在运行时路径之外。

### Module constants / 模块常量
```python
logger = init_logger(__name__)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`，供后续代码复用。

### Class `ECConnectorFactory` / 类 `ECConnectorFactory`
```python
class ECConnectorFactory:
    _registry: dict[str, Callable[[], type[ECConnectorBase]]] = {}

    @classmethod
    def register_connector(cls, name: str, module_path: str, class_name: str) -> None:
        """Register a connector with a lazy-loading module and class name."""
        if name in cls._registry:
            raise ValueError(f"Connector '{name}' is already registered.")

        def loader() -> type[ECConnectorBase]:
            module = importlib.import_module(module_path)
            return getattr(module, class_name)

        cls._registry[name] = loader

    @classmethod
    def create_connector(
        cls,
        config: "VllmConfig",
        role: ECConnectorRole,
    ) -> ECConnectorBase:
        ec_transfer_config = config.ec_transfer_config
        if ec_transfer_config is None:
            raise ValueError("ec_transfer_config must be set to create a connector")
        connector_cls = cls.get_connector_class(ec_transfer_config)
        logger.info(
            "Creating connector with name: %s and engine_id: %s",
            connector_cls.__name__,
            ec_transfer_config.engine_id,
        )
        # Connector is explicitly separated into two roles.
        # Scheduler connector:
        # - Co-locate with scheduler process
        # - Should only be used inside the Scheduler class
        # Worker connector:
        # - Co-locate with worker process
        return connector_cls(config, role)

    @classmethod
    def get_connector_class(
        cls, ec_transfer_config: "ECTransferConfig"
    ) -> type[ECConnectorBase]:
        """Get the connector class by name."""
        connector_name = ec_transfer_config.ec_connector
        if connector_name is None:
            raise ValueError("EC connect must not be None")
        elif connector_name in cls._registry:
            connector_cls = cls._registry[connector_name]()
        else:
            connector_module_path = ec_transfer_config.ec_connector_module_path
            if connector_module_path is None:
                raise ValueError(f"Unsupported connector type: {connector_name}")
            connector_module = importlib.import_module(connector_module_path)
            connector_cls = getattr(connector_module, connector_name)
        return connector_cls
```
**EN:** Declares `ECConnectorFactory`, a class. Key methods include `register_connector`, `create_connector`, `get_connector_class`.
**CN:** 声明 `ECConnectorFactory`，它是一个类。 关键方法包括 `register_connector`, `create_connector`, `get_connector_class`。

### Expr block / Expr 代码块
```python
ECConnectorFactory.register_connector(
    "ECExampleConnector",
    "vllm.distributed.ec_transfer.ec_connector.example_connector",
    "ECExampleConnector",
)
```
**EN:** This top-level `Expr` block contributes supporting module logic or declarations.
**CN:** 该顶层 `Expr` 代码块为模块补充辅助逻辑或声明。

## Key Concepts / 关键概念
- `ECConnectorFactory`: class interface or data carrier / `ECConnectorFactory`：类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `importlib`, `collections.abc`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal modules / 内部模块**: `vllm.distributed.ec_transfer.ec_connector.base`, `vllm.logger`, `vllm.config`
