# example_connector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/example_connector.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements a v1 KV-transfer connector, interface, or helper. / 实现 v1 KV 传输连接器、接口或辅助模块。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
import os
from dataclasses import dataclass, field
from typing import TYPE_CHECKING, Any

import safetensors
import torch

from vllm.config import VllmConfig
from vllm.distributed.kv_transfer.kv_connector.v1.base import (
    KVConnectorBase_V1,
    KVConnectorMetadata,
    KVConnectorRole,
)
from vllm.logger import init_logger
from vllm.model_executor.layers.attention.mla_attention import MLACommonMetadata
from vllm.utils.hashing import safe_hash
from vllm.v1.attention.backend import AttentionMetadata
from vllm.v1.attention.backends.triton_attn import TritonAttentionMetadata
from vllm.v1.core.sched.output import SchedulerOutput
```
**EN:** This block imports `os`, `dataclasses`, `typing`, `safetensors`, `torch`, `vllm.config` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `os`, `dataclasses`, `typing`, `safetensors`, `torch`, `vllm.config`，为后续实现准备运行时、类型与辅助 API。

### Conditional block / 条件代码块
```python
if TYPE_CHECKING:
    from vllm.forward_context import ForwardContext
    from vllm.v1.core.kv_cache_manager import KVCacheBlocks
    from vllm.v1.kv_cache_interface import KVCacheConfig
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

### Class `ReqMeta` / 类 `ReqMeta`
```python
@dataclass
class ReqMeta:
    # Request tokens
    token_ids: torch.Tensor
    # Slot mappings, should have the same length as token_ids
    slot_mapping: torch.Tensor
    # Is store or load
    is_store: bool
    mm_hashes: list[str]

    @staticmethod
    def make_meta(
        token_ids: list[int],
        block_ids: list[int],
        block_size: int,
        is_store: bool,
        mm_hashes: list[str],
    ) -> "ReqMeta":
        valid_num_tokens = align_to_block_size(len(token_ids), block_size)
        token_ids_tensor = torch.tensor(token_ids)[:valid_num_tokens]
        block_ids_tensor = torch.tensor(block_ids)
        num_blocks = block_ids_tensor.shape[0]
        block_offsets = torch.arange(0, block_size)
        slot_mapping = (
            block_offsets.reshape((1, block_size))
            + block_ids_tensor.reshape((num_blocks, 1)) * block_size
        )
        slot_mapping = slot_mapping.flatten()[:valid_num_tokens]
        return ReqMeta(
            token_ids=token_ids_tensor,
            slot_mapping=slot_mapping,
            is_store=is_store,
            mm_hashes=mm_hashes,
        )
```
**EN:** Declares `ReqMeta`, a dataclass. It packages structured data fields such as `token_ids`, `slot_mapping`, `is_store`, `mm_hashes`.
**CN:** 声明 `ReqMeta`，它是一个数据类。 它封装了 `token_ids`, `slot_mapping`, `is_store`, `mm_hashes` 等结构化字段。

### Class `ExampleConnectorMetadata` / 类 `ExampleConnectorMetadata`
```python
@dataclass
class ExampleConnectorMetadata(KVConnectorMetadata):
    requests: list[ReqMeta] = field(default_factory=list)

    def add_request(
        self,
        token_ids: list[int],
        block_ids: list[int],
        block_size: int,
        is_store: bool,
        mm_hashes: list[str],
    ) -> None:
        self.requests.append(
            ReqMeta.make_meta(token_ids, block_ids, block_size, is_store, mm_hashes)
        )
```
**EN:** Declares `ExampleConnectorMetadata`, a dataclass derived from `KVConnectorMetadata`. It packages structured data fields such as `requests`.
**CN:** 声明 `ExampleConnectorMetadata`，它是一个数据类，继承自 `KVConnectorMetadata`。 它封装了 `requests` 等结构化字段。

### Class `ExampleConnector` / 类 `ExampleConnector`
```python
class ExampleConnector(KVConnectorBase_V1):
    # NOTE: This is Simple debug implementation of the KV connector.
    # It save / load the KV cache to / from the disk.
    # It does extra work which will overwrite the existing prefix-cache in GPU
    # - to remove the overhead, need to add some "mask" in the ReqMeta class

    def __init__(
        self,
        vllm_config: "VllmConfig",
        role: KVConnectorRole,
        kv_cache_config: "KVCacheConfig",
    ):
        super().__init__(
            vllm_config=vllm_config,
            role=role,
            kv_cache_config=kv_cache_config,
        )
        self._block_size = vllm_config.cache_config.block_size
        self._requests_need_load: dict[str, Request] = {}
        self._storage_path = self._kv_transfer_config.get_from_extra_config(
            "shared_storage_path", "/tmp"
        )
        logger.info(self._kv_transfer_config)
        logger.info("Shared storage path is %s", self._storage_path)

    def start_load_kv(self, forward_context: "ForwardContext", **kwargs: Any) -> None:
        """Start loading the KV cache from the connector buffer to vLLM's
        paged KV buffer.

        Args:
            forward_context (ForwardContext): the forward context.
            **kwargs: additional arguments for the load operation

        Note:
            The number of elements in kv_caches and layer_names should be
            the same.
        """

        def inject_kv_into_layer(
            dst_kv_cache_layer: torch.Tensor,
            src_kv_cache: torch.Tensor,
            slot_mapping: torch.Tensor,
            attn_metadata: AttentionMetadata,
        ) -> None:
            """Inject the KV cache into the layer.
# ... truncated for analysis ...
        token_ids: torch.Tensor,
        mm_hashes: list[str],
    ) -> str:
        """Generate a file name based on the layer name and the hash
        of the bytes of the input ids.
        """
        foldername = self._generate_foldername_debug(
            token_ids, mm_hashes=mm_hashes, create_folder=True
        )
        return os.path.join(foldername, f"{layer_name}.safetensors")
```
**EN:** Declares `ExampleConnector`, a class derived from `KVConnectorBase_V1`. Key methods include `__init__`, `start_load_kv`, `wait_for_layer_load`, `save_kv_layer`, `wait_for_save`.
**CN:** 声明 `ExampleConnector`，它是一个类，继承自 `KVConnectorBase_V1`。 关键方法包括 `__init__`, `start_load_kv`, `wait_for_layer_load`, `save_kv_layer`, `wait_for_save`。

### Function `align_to_block_size` / 函数 `align_to_block_size`
```python
def align_to_block_size(num_tokens: int, block_size) -> int:
    """Align the number of tokens to the block size."""
    return (num_tokens - 1) // block_size * block_size
```
**EN:** `align_to_block_size` implements a focused helper routine for this module. The docstring frames it as: Align the number of tokens to the block size. It primarily works with arguments like `num_tokens`, `block_size`.
**CN:** `align_to_block_size` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `num_tokens`, `block_size` 这样的参数。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `ReqMeta`: dataclass interface or data carrier / `ReqMeta`：数据类接口或数据载体
- `ExampleConnectorMetadata`: dataclass interface or data carrier / `ExampleConnectorMetadata`：数据类接口或数据载体
- `ExampleConnector`: class interface or data carrier / `ExampleConnector`：类接口或数据载体
- `align_to_block_size`: module-level helper or API entry / `align_to_block_size`：模块级辅助函数或 API 入口

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `dataclasses`, `typing`
- **Third-party / 第三方**: `safetensors`, `torch`
- **Internal modules / 内部模块**: `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.logger`, `vllm.model_executor.layers.attention.mla_attention`, `vllm.utils.hashing`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.triton_attn`, `vllm.v1.core.sched.output`, `vllm.forward_context`, `vllm.v1.core.kv_cache_manager`, `vllm.v1.kv_cache_interface`, `vllm.v1.request`
