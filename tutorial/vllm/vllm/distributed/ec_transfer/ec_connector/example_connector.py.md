# example_connector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/ec_transfer/ec_connector/example_connector.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements EC-transfer connectors and runtime state. / 实现 EC 传输连接器与运行时状态。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
import os
from dataclasses import dataclass
from typing import TYPE_CHECKING

import safetensors

from vllm.config import VllmConfig
from vllm.distributed.ec_transfer.ec_connector.base import (
    ECConnectorBase,
    ECConnectorMetadata,
    ECConnectorRole,
)
from vllm.logger import init_logger
from vllm.v1.core.sched.output import SchedulerOutput
```
**EN:** This block imports `os`, `dataclasses`, `typing`, `safetensors`, `vllm.config`, `vllm.distributed.ec_transfer.ec_connector.base` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `os`, `dataclasses`, `typing`, `safetensors`, `vllm.config`, `vllm.distributed.ec_transfer.ec_connector.base`，为后续实现准备运行时、类型与辅助 API。

### Conditional block / 条件代码块
```python
if TYPE_CHECKING:
    from vllm.v1.request import Request
```
**EN:** This conditional keeps imports and declarations that are only needed for static type checking out of the runtime path.
**CN:** 该条件块将仅用于静态类型检查的导入与声明隔离在运行时路径之外。

### Module constants / 模块常量
```python
logger = init_logger(__name__)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`，供后续代码复用。

### Class `MMMeta` / 类 `MMMeta`
```python
@dataclass
class MMMeta:
    mm_hash: str
    num_token: int

    @staticmethod
    def make_meta(mm_hash, num_token) -> "MMMeta":
        return MMMeta(mm_hash=mm_hash, num_token=num_token)
```
**EN:** Declares `MMMeta`, a dataclass. It packages structured data fields such as `mm_hash`, `num_token`.
**CN:** 声明 `MMMeta`，它是一个数据类。 它封装了 `mm_hash`, `num_token` 等结构化字段。

### Class `ECExampleConnectorMetadata` / 类 `ECExampleConnectorMetadata`
```python
@dataclass
class ECExampleConnectorMetadata(ECConnectorMetadata):
    mm_datas: list[MMMeta]

    def __init__(self):
        self.mm_datas = []

    def add_mm_data(self, mm_data: MMMeta):
        self.mm_datas.append(mm_data)
```
**EN:** Declares `ECExampleConnectorMetadata`, a dataclass derived from `ECConnectorMetadata`. It packages structured data fields such as `mm_datas`.
**CN:** 声明 `ECExampleConnectorMetadata`，它是一个数据类，继承自 `ECConnectorMetadata`。 它封装了 `mm_datas` 等结构化字段。

### Class `ECExampleConnector` / 类 `ECExampleConnector`
```python
class ECExampleConnector(ECConnectorBase):
    # NOTE: This is Simple debug implementation of the EC connector.
    # It save / load the EC cache to / from the disk.

    def __init__(self, vllm_config: "VllmConfig", role: ECConnectorRole):
        super().__init__(vllm_config=vllm_config, role=role)
        # req_id -> index
        self._mm_datas_need_loads: dict[str, int] = {}
        transfer_config = vllm_config.ec_transfer_config
        if transfer_config is not None:
            self._storage_path = transfer_config.get_from_extra_config(
                "shared_storage_path", "/tmp"
            )
            logger.debug(transfer_config)
            logger.debug("Shared storage path is %s", self._storage_path)
        else:
            raise ValueError("ec_transfer_config must be set for ECConnectorBase")

    def start_load_caches(self, encoder_cache, **kwargs) -> None:
        """
        Start loading the cache from the connector into vLLM's encoder cache.

        This method loads the encoder cache based on metadata provided by the scheduler.
        It is called before `_gather_mm_embeddings` for the EC Connector. For EC,
        the `encoder_cache` and `mm_hash` are stored in `kwargs`.

        Args:
            encoder_cache (dict[str, torch.Tensor]): A dictionary mapping multimodal
                data hashes (`mm_hash`) to encoder cache tensors.
            kwargs (dict): Additional keyword arguments for the connector.
        """
        from vllm.platforms import current_platform

        # Get the metadata
        metadata: ECConnectorMetadata = self._get_connector_metadata()
        assert isinstance(metadata, ECExampleConnectorMetadata)
        assert encoder_cache is not None
        if metadata is None:
            logger.warning(
                "In connector.start_load_caches, but the connector metadata is None"
            )
            return
        # Load the EC for each mm data
        for mm_data in metadata.mm_datas:
            if mm_data.mm_hash in encoder_cache:
# ... truncated for analysis ...

    def _generate_filename_debug(self, mm_hash: str) -> str:
        """
        Return the full path of the safetensors file for this mm_hash.
        Ensures the parent directory exists because
        `_generate_foldername_debug` is called with its default
        (`create_folder=True`).
        """
        foldername = self._generate_foldername_debug(mm_hash)  # <- folder auto-created
        return os.path.join(foldername, "encoder_cache.safetensors")
```
**EN:** Declares `ECExampleConnector`, a class derived from `ECConnectorBase`. Key methods include `__init__`, `start_load_caches`, `save_caches`, `has_cache_item`, `update_state_after_alloc`.
**CN:** 声明 `ECExampleConnector`，它是一个类，继承自 `ECConnectorBase`。 关键方法包括 `__init__`, `start_load_caches`, `save_caches`, `has_cache_item`, `update_state_after_alloc`。

## Key Concepts / 关键概念
- `MMMeta`: dataclass interface or data carrier / `MMMeta`：数据类接口或数据载体
- `ECExampleConnectorMetadata`: dataclass interface or data carrier / `ECExampleConnectorMetadata`：数据类接口或数据载体
- `ECExampleConnector`: class interface or data carrier / `ECExampleConnector`：类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `dataclasses`, `typing`
- **Third-party / 第三方**: `safetensors`
- **Internal modules / 内部模块**: `vllm.config`, `vllm.distributed.ec_transfer.ec_connector.base`, `vllm.logger`, `vllm.v1.core.sched.output`, `vllm.v1.request`, `vllm.platforms`
