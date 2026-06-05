# scheduler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/mooncake/store/scheduler.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Scheduler-side logic for MooncakeStoreConnector / 实现基于 Mooncake 的 KV 传输连接器、存储层或运行时辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Module overview / 模块概览
```python
"""Scheduler-side logic for MooncakeStoreConnector."""
```
**EN:** The opening docstring defines the module scope and high-level contract. It highlights: Scheduler-side logic for MooncakeStoreConnector.
**CN:** 开头的文档字符串给出了模块范围与高层契约。 它重点概括了文件的职责、参与方以及主要接口。

### Imports and setup / 导入与初始化
```python
from typing import Any

from vllm.config import VllmConfig
from vllm.distributed.kv_transfer.kv_connector.v1.base import (
    KVConnectorMetadata,
)
from vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store.data import (  # noqa: E501
    LoadSpec,
    MooncakeStoreConnectorMetadata,
    ReqMeta,
    RequestTracker,
)
from vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store.worker import (  # noqa: E501
    LookupKeyClient,
)
from vllm.logger import init_logger
from vllm.v1.core.kv_cache_manager import KVCacheBlocks
from vllm.v1.core.sched.output import NewRequestData, SchedulerOutput
from vllm.v1.request import Request
```
**EN:** This block imports `typing`, `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store.data`, `vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store.worker`, `vllm.logger` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `typing`, `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store.data`, `vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store.worker`, `vllm.logger`，为后续实现准备运行时、类型与辅助 API。

### Module constants / 模块常量
```python
logger = init_logger(__name__)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`，供后续代码复用。

### Function `_new_req_prefill_tokens` / 函数 `_new_req_prefill_tokens`
```python
def _new_req_prefill_tokens(request: NewRequestData) -> list[int]:
    """Tokens this prefill will compute KV for.

    Under the v2 model runner, resumed-from-preemption requests appear in
    ``scheduled_new_reqs`` with ``prefill_token_ids`` set to the request's full
    token list (prompt + previously-generated). For all other cases this falls
    back to the original prompt.
    """
    if request.prefill_token_ids is not None:
        return request.prefill_token_ids
    assert request.prompt_token_ids is not None
    return request.prompt_token_ids
```
**EN:** `_new_req_prefill_tokens` implements a focused helper routine for this module. The docstring frames it as: Tokens this prefill will compute KV for. It primarily works with arguments like `request`.
**CN:** `_new_req_prefill_tokens` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `request` 这样的参数。

### Class `MooncakeStoreScheduler` / 类 `MooncakeStoreScheduler`
```python
class MooncakeStoreScheduler:
    """Scheduler-side component for MooncakeStoreConnector."""

    def __init__(self, vllm_config: VllmConfig):
        assert vllm_config.kv_transfer_config is not None
        self.kv_role = vllm_config.kv_transfer_config.kv_role
        self.load_async = vllm_config.kv_transfer_config.kv_connector_extra_config.get(
            "load_async", True
        )
        self.client = LookupKeyClient(vllm_config)

        self.pcp_size = vllm_config.parallel_config.prefill_context_parallel_size
        self.dcp_size = vllm_config.parallel_config.decode_context_parallel_size
        self.original_block_size = vllm_config.cache_config.block_size
        self._block_size = vllm_config.cache_config.block_size
        if self.pcp_size > 1:
            self._block_size *= self.pcp_size
        if self.dcp_size > 1:
            self._block_size *= self.dcp_size

        self._discard_partial_chunks = (
            vllm_config.kv_transfer_config.get_from_extra_config(
                "discard_partial_chunks", True
            )
        )

        # Per-request state
        self.load_specs: dict[str, LoadSpec] = {}  # to be loaded
        self._request_trackers: dict[str, RequestTracker] = {}  # scheduled new requests
        self._preempted_req_ids: set[str] = set()  # preempted requests
        self._unfinished_requests: dict[str, tuple[Request, list[int]]] = {}
        self._unfinished_request_ids: set[str] = set()

    def get_num_new_matched_tokens(
        self,
        request: Request,
        num_computed_tokens: int,
    ) -> tuple[int, bool]:
        """Check for external KV cache hit."""
        # Look up against the full prefill range, not just the prompt.
        if self._discard_partial_chunks:
            token_len = request.num_tokens // self._block_size * self._block_size
        else:
            token_len = request.num_tokens
# ... truncated for analysis ...
        if tracker.num_saved_tokens <= 0:
            return False, None
        delay_free_blocks = len(block_ids) > 0
        if delay_free_blocks:
            logger.debug(
                "Delaying free of %d blocks for request %s",
                len(block_ids),
                request.request_id,
            )
        return delay_free_blocks, None
```
**EN:** Declares `MooncakeStoreScheduler`, a class. Key methods include `__init__`, `get_num_new_matched_tokens`, `update_state_after_alloc`, `build_connector_meta`, `request_finished`. The docstring summarizes its role as: Scheduler-side component for MooncakeStoreConnector.
**CN:** 声明 `MooncakeStoreScheduler`，它是一个类。 关键方法包括 `__init__`, `get_num_new_matched_tokens`, `update_state_after_alloc`, `build_connector_meta`, `request_finished`。 文档字符串概括了它在整体流程中的职责。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `_new_req_prefill_tokens`: module-level helper or API entry / `_new_req_prefill_tokens`：模块级辅助函数或 API 入口
- `MooncakeStoreScheduler`: class interface or data carrier / `MooncakeStoreScheduler`：类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方**: None / 无
- **Internal modules / 内部模块**: `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store.data`, `vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store.worker`, `vllm.logger`, `vllm.v1.core.kv_cache_manager`, `vllm.v1.core.sched.output`, `vllm.v1.request`
