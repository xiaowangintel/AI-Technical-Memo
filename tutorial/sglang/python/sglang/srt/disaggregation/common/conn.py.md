# conn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/disaggregation/common/conn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements the common connection backend for disaggregated serving. It manages handshakes, transfer state, and KV movement for that specific transport or runtime environment. / 该文件实现了解耦式服务中的 common 连接后端，负责该特定传输/运行时环境下的握手、传输状态管理以及 KV 数据移动。

## Line-by-Line Analysis / 逐行分析
### Lines 1-48: Imports and module setup
```python
from __future__ import annotations

import asyncio
import dataclasses
import logging
import threading
import time
from collections import defaultdict
from functools import cache
from typing import Dict, List, Optional, Set, Tuple, Union

import numpy as np
import numpy.typing as npt
import requests
import torch.distributed as dist
import zmq
from aiohttp import web

from sglang.srt.disaggregation.base.conn import (
    BaseKVBootstrapServer,
    BaseKVManager,
    BaseKVReceiver,
    BaseKVSender,
    KVArgs,
    KVPoll,
    KVTransferMetric,
)
from sglang.srt.disaggregation.utils import DisaggregationMode
from sglang.srt.distributed import get_pp_group, get_world_group
from sglang.srt.environ import envs
from sglang.srt.layers.dp_attention import (
    get_attention_cp_rank,
    get_attention_cp_size,
    get_attention_dp_rank,
    get_attention_dp_size,
    get_attention_tp_rank,
    get_attention_tp_size,
)
from sglang.srt.server_args import ServerArgs
from sglang.srt.utils.network import (
    NetworkAddress,
    get_local_ip_auto,
    get_zmq_socket_on_host,
)

logger = logging.getLogger(__name__)


```
**EN:** This block gathers the imports and module-level setup for common backend connection and KV transfer management. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与common 后端连接与 KV 传输管理相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 49-51: Class `PrefillServerInfo` declaration
```python
@dataclasses.dataclass
class PrefillServerInfo:
    # Topology fields (fetched from bootstrap server)
```
**EN:** This block declares the class `PrefillServerInfo` and establishes its responsibility inside common backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `PrefillServerInfo`. Notable operations include `fields`.
**CN:** 这一段声明了类 `PrefillServerInfo`，并说明它在common 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `PrefillServerInfo`。 值得注意的操作包括 `fields`。

### Lines 52-67: Supporting state inside `PrefillServerInfo`
```python
    attn_tp_size: int
    attn_cp_size: int
    dp_size: int
    pp_size: int
    page_size: Optional[int]
    kv_cache_dtype: Optional[str]
    follow_bootstrap_room: bool

    # Pre-computed rank mapping (set by try_ensure_parallel_info on decode side)
    target_tp_rank: Optional[int] = None
    target_tp_ranks: Optional[List[int]] = None
    target_cp_ranks: Optional[List[int]] = None
    target_pp_ranks: Optional[List[int]] = None
    required_dst_info_num: Optional[int] = None
    required_prefill_response_num: Optional[int] = None

```
**EN:** This block adds supporting state or helper logic inside `PrefillServerInfo`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `mapping`.
**CN:** 这一段为 `PrefillServerInfo` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `mapping`。

### Lines 68-77: Method `__post_init__`
```python
    def __post_init__(self):
        self.attn_tp_size = int(self.attn_tp_size)
        self.attn_cp_size = int(self.attn_cp_size)
        self.dp_size = int(self.dp_size)
        self.pp_size = int(self.pp_size)
        self.page_size = int(self.page_size) if self.page_size is not None else None
        self.kv_cache_dtype = (
            str(self.kv_cache_dtype) if self.kv_cache_dtype is not None else None
        )
        self.follow_bootstrap_room = bool(self.follow_bootstrap_room)
```
**EN:** This block defines the method `__post_init__` on `PrefillServerInfo`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `__post_init__`.
**CN:** 这一段定义了method `__post_init__`（属于 `PrefillServerInfo`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `__post_init__`。

### Lines 80-81: Class `PrefillRankInfo` declaration
```python
@dataclasses.dataclass
class PrefillRankInfo:
```
**EN:** This block declares the class `PrefillRankInfo` and establishes its responsibility inside common backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `PrefillRankInfo`.
**CN:** 这一段声明了类 `PrefillRankInfo`，并说明它在common 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `PrefillRankInfo`。

### Lines 82-84: Supporting state inside `PrefillRankInfo`
```python
    rank_ip: str
    rank_port: int

```
**EN:** This block adds supporting state or helper logic inside `PrefillRankInfo`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `PrefillRankInfo` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 85-87: Method `__post_init__`
```python
    def __post_init__(self):
        self.rank_ip = str(self.rank_ip)
        self.rank_port = int(self.rank_port)
```
**EN:** This block defines the method `__post_init__` on `PrefillRankInfo`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `__post_init__`.
**CN:** 这一段定义了method `__post_init__`（属于 `PrefillRankInfo`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `__post_init__`。

### Lines 90-90: Class `CommonKVManager` declaration
```python
class CommonKVManager(BaseKVManager):
```
**EN:** This block declares the class `CommonKVManager` and establishes its responsibility inside common backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `CommonKVManager`.
**CN:** 这一段声明了类 `CommonKVManager`，并说明它在common 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `CommonKVManager`。

### Lines 91-93: Method `__init__` signature and setup
```python
    def __init__(
        self,
        args: KVArgs,
```
**EN:** This block defines the method `__init__` on `CommonKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `CommonKVManager`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 94-125: Method `__init__` logic (part 1)
```python
        disaggregation_mode: DisaggregationMode,
        server_args: ServerArgs,
        is_mla_backend: Optional[bool] = False,
    ):
        self.kv_args = args
        self.kv_item_lens_sum = sum(args.kv_item_lens)
        self.state_item_lens_sum = sum(x for comp in args.state_item_lens for x in comp)
        self.is_mla_backend = is_mla_backend
        self.disaggregation_mode = disaggregation_mode
        self.server_args = server_args
        # for p/d multi node infer
        self.bootstrap_host = server_args.host
        self.bootstrap_port = server_args.disaggregation_bootstrap_port
        self.dist_init_addr = server_args.dist_init_addr
        self.attn_tp_size = get_attention_tp_size()
        self.attn_tp_rank = get_attention_tp_rank()
        self.attn_cp_size = get_attention_cp_size()
        self.attn_cp_rank = get_attention_cp_rank()
        self.attn_dp_size = get_attention_dp_size()
        self.attn_dp_rank = get_attention_dp_rank()
        self.system_dp_size = (
            1 if server_args.enable_dp_attention else server_args.dp_size
        )
        self.system_dp_rank = (
            self.kv_args.system_dp_rank if self.kv_args.system_dp_rank else 0
        )
        self.pp_size = server_args.pp_size
        self.pp_rank = self.kv_args.pp_rank
        self.local_ip = get_local_ip_auto()
        self.enable_all_cp_ranks_for_transfer = (
            envs.SGLANG_DISAGGREGATION_ALL_CP_RANKS_TRANSFER.get()
        )
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `get_attention_tp_size`, `get_attention_tp_rank`, `get_attention_cp_size`, `get_attention_cp_rank`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `get_attention_tp_size`、`get_attention_tp_rank`、`get_attention_cp_size`、`get_attention_cp_rank`。

### Lines 126-157: Method `__init__` logic (part 2)
```python

        # bind zmq socket
        context = zmq.Context()
        self.rank_port, self.server_socket = get_zmq_socket_on_host(
            context, zmq.PULL, host=self.local_ip
        )
        logger.debug(f"kv manager bind to {self.local_ip}:{self.rank_port}")

        self.request_status: Dict[int, KVPoll] = {}
        self.failure_records: Dict[int, str] = {}
        self.failure_lock = threading.Lock()

        if self.disaggregation_mode == DisaggregationMode.PREFILL:
            # When SGLANG_DISAGGREGATION_ALL_CP_RANKS_TRANSFER is True, all CP ranks
            # participate in KV transfer; Otherwise only CP rank 0 sends.
            self.is_dummy_cp_rank = (
                not self.enable_all_cp_ranks_for_transfer
                and self.attn_cp_size > 1
                and self.attn_cp_rank != 0
            )
            # Sync the leader's bootstrap port to every rank before
            # registering: in multi-node prefill, registration targets
            # `dist_init_addr` (rank 0) but each rank's local port may
            # differ when the launcher auto-reserves a free port per host.
            self.bootstrap_port = self._sync_bootstrap_port_across_nodes(
                self.bootstrap_port
            )
            self.register_to_bootstrap()
            self.transfer_infos = {}
            self.req_to_decode_prefix_len: Dict[int, int] = {}
            self.decode_kv_args_table = {}
            self.pp_group = get_pp_group()
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `Context`, `get_zmq_socket_on_host`, `debug`, `Lock`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `Context`、`get_zmq_socket_on_host`、`debug`、`Lock`。

### Lines 158-188: Method `__init__` logic (part 3)
```python
            # If a timeout happens on the prefill side, it means prefill instances
            # fail to receive the KV indices from the decode instance of this request.
            # These timeout requests should be aborted to release the tree cache.
            self.bootstrap_timeout = envs.SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT.get()
        elif self.disaggregation_mode == DisaggregationMode.DECODE:
            self.enable_staging: bool = False
            self.connection_pool: Dict[str, Dict[str, Union[str, int]]] = {}
            self.connection_lock = threading.Lock()
            self.required_prefill_response_num_table: Dict[int, int] = {}
            self.prefill_info_table: Dict[str, PrefillServerInfo] = {}
            self.heartbeat_failures: Dict[str, int] = {}
            self.session_pool: Dict = defaultdict(requests.Session)
            self.session_pool_lock = threading.Lock()
            self.addr_to_rooms_tracker: Dict[str, Set[int]] = defaultdict(set)
            self.prefill_response_tracker: Dict[int, Set[int]] = defaultdict(set)
            # Heartbeat interval should be at least 2 seconds
            self.heartbeat_interval = max(
                envs.SGLANG_DISAGGREGATION_HEARTBEAT_INTERVAL.get(), 2.0
            )
            # Heartbeat failure should be at least 1
            self.max_failures = max(
                envs.SGLANG_DISAGGREGATION_HEARTBEAT_MAX_FAILURE.get(), 1
            )
            # If a timeout happens on the decode side, it means decode instances
            # fail to receive the KV Cache transfer done signal after bootstrapping.
            # These timeout requests should be aborted to release the tree cache.
            self.waiting_timeout = envs.SGLANG_DISAGGREGATION_WAITING_TIMEOUT.get()
        else:
            raise ValueError(
                f"Unsupported DisaggregationMode: {self.disaggregation_mode}"
            )
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `get`, `Lock`, `defaultdict`, `ValueError`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `get`、`Lock`、`defaultdict`、`ValueError`。

### Lines 190-191: Method `check_status`
```python
    def check_status(self, bootstrap_room: int) -> KVPoll:
        return self.request_status[bootstrap_room]
```
**EN:** This block defines the method `check_status` on `CommonKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `check_status`.
**CN:** 这一段定义了method `check_status`（属于 `CommonKVManager`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `check_status`。

### Lines 193-208: Method `update_status`
```python
    def update_status(self, bootstrap_room: int, status: KVPoll):
        if bootstrap_room not in self.request_status:
            # Do not resurrect a cleared entry with Failed: once clear() has
            # popped the room from request_status, any late update_status(Failed)
            # (e.g. from abort()) must be a no-op. Otherwise a Failed entry could
            # pollute a future request that reuses the same bootstrap_room.
            if status == KVPoll.Failed:
                return
            self.request_status[bootstrap_room] = status
        else:
            if status == KVPoll.Failed:
                self.request_status[bootstrap_room] = KVPoll.Failed
            else:
                self.request_status[bootstrap_room] = max(
                    self.request_status[bootstrap_room], status
                )
```
**EN:** This block defines the method `update_status` on `CommonKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `update_status`. Notable operations include `clear`, `update_status`, `abort`.
**CN:** 这一段定义了method `update_status`（属于 `CommonKVManager`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `update_status`。 值得注意的操作包括 `clear`、`update_status`、`abort`。

### Lines 210-212: Method `record_failure`
```python
    def record_failure(self, bootstrap_room: int, failure_reason: str):
        with self.failure_lock:
            self.failure_records[bootstrap_room] = failure_reason
```
**EN:** This block defines the method `record_failure` on `CommonKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `record_failure`.
**CN:** 这一段定义了method `record_failure`（属于 `CommonKVManager`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `record_failure`。

### Lines 214-216: Method `try_ensure_parallel_info` signature and setup
```python
    def try_ensure_parallel_info(self, bootstrap_addr: str) -> bool:
        """Single non-blocking attempt to fetch and cache prefill parallel info.
        Returns True if info is available (cached or freshly fetched)."""
```
**EN:** This block defines the method `try_ensure_parallel_info` on `CommonKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `try_ensure_parallel_info`. Notable operations include `available`.
**CN:** 这一段定义了method `try_ensure_parallel_info`（属于 `CommonKVManager`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `try_ensure_parallel_info`。 值得注意的操作包括 `available`。

### Lines 217-234: Method `try_ensure_parallel_info` logic (part 1)
```python
        if bootstrap_addr in self.prefill_info_table:
            return True

        info: PrefillServerInfo = None
        try:
            url = f"http://{bootstrap_addr}/route?prefill_dp_rank={-1}&prefill_cp_rank={-1}&target_tp_rank={-1}&target_pp_rank={-1}"
            response = requests.get(url, timeout=5)
            if response.status_code == 200:
                data = response.json()
                info = PrefillServerInfo(**data)
            else:
                logger.error(
                    f"Failed to get prefill server info: {response.status_code}, {response.text}"
                )
                return False
        except Exception as e:
            logger.error(f"Error fetching prefill server info from bootstrap: {e}")
            return False
```
**EN:** This block continues `try_ensure_parallel_info` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `get`, `json`, `PrefillServerInfo`, `error`.
**CN:** 这一段延续了 `try_ensure_parallel_info` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `get`、`json`、`PrefillServerInfo`、`error`。

### Lines 235-260: Method `try_ensure_parallel_info` logic (part 2)
```python

        # Sanity checks
        if info.page_size is not None and info.page_size != self.kv_args.page_size:
            if self.server_args.enable_hisparse:
                # HiSparse: decode host pool page_size=1, prefill device pool page_size >= 1.
                # Transfer will use send_kvcache_hisparse with per-token item_lens.
                logger.info(
                    f"HiSparse PD transfer mode: prefill page_size={info.page_size}, "
                    f"decode host page_size={self.kv_args.page_size}"
                )
            else:
                raise RuntimeError(
                    f"Page size mismatch: prefill server has page_size={info.page_size}, "
                    f"but decode server has page_size={self.kv_args.page_size}. "
                    f"Both servers must use the same --page-size value."
                )

        if (
            info.kv_cache_dtype is not None
            and info.kv_cache_dtype != self.server_args.kv_cache_dtype
        ):
            raise RuntimeError(
                f"KV cache dtype mismatch: prefill server has kv_cache_dtype={info.kv_cache_dtype}, "
                f"but decode server has kv_cache_dtype={self.server_args.kv_cache_dtype}. "
                f"Both servers must use the same --kv-cache-dtype value."
            )
```
**EN:** This block continues `try_ensure_parallel_info` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `info`, `RuntimeError`.
**CN:** 这一段延续了 `try_ensure_parallel_info` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `info`、`RuntimeError`。

### Lines 261-265: Method `try_ensure_parallel_info` logic (part 3)
```python

        self._resolve_rank_mapping(info)
        self.prefill_info_table[bootstrap_addr] = info
        logger.debug(f"Prefill parallel info for [{bootstrap_addr}]: {info}")
        return True
```
**EN:** This block continues `try_ensure_parallel_info` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `_resolve_rank_mapping`, `debug`.
**CN:** 这一段延续了 `try_ensure_parallel_info` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `_resolve_rank_mapping`、`debug`。

### Lines 267-269: Method `_resolve_rank_mapping` signature and setup
```python
    def _resolve_rank_mapping(self, info: PrefillServerInfo) -> None:
        """Compute TP/CP/PP rank mapping and store on the PrefillServerInfo object.
        Deterministic for a given (bootstrap_addr, decode engine) pair."""
```
**EN:** This block defines the method `_resolve_rank_mapping` on `CommonKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_resolve_rank_mapping`. Notable operations include `given`.
**CN:** 这一段定义了method `_resolve_rank_mapping`（属于 `CommonKVManager`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_resolve_rank_mapping`。 值得注意的操作包括 `given`。

### Lines 270-293: Method `_resolve_rank_mapping` logic (part 1)
```python
        # TP rank mapping
        if self.attn_tp_size == info.attn_tp_size:
            target_tp_rank = self.kv_args.engine_rank % self.attn_tp_size
            required_dst_info_num = 1
            required_prefill_response_num = 1
            target_tp_ranks = [target_tp_rank]
        elif self.attn_tp_size > info.attn_tp_size:
            if not self.is_mla_backend:
                logger.warning_once(
                    "Performance is NOT guaranteed when using different TP sizes for non-MLA models. "
                )
            target_tp_rank = (self.kv_args.engine_rank % self.attn_tp_size) // (
                self.attn_tp_size // info.attn_tp_size
            )
            required_dst_info_num = self.attn_tp_size // info.attn_tp_size
            required_prefill_response_num = 1
            target_tp_ranks = [target_tp_rank]
        else:
            if not self.is_mla_backend:
                logger.warning_once(
                    "Performance is NOT guaranteed when using different TP sizes for non-MLA models. "
                )
            # For non-MLA models, one decode rank needs to retrieve KVCache from multiple prefill ranks
            target_tp_ranks = list(
```
**EN:** This block continues `_resolve_rank_mapping` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `warning_once`.
**CN:** 这一段延续了 `_resolve_rank_mapping` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `warning_once`。

### Lines 294-309: Method `_resolve_rank_mapping` logic (part 2)
```python
                range(
                    (self.kv_args.engine_rank % self.attn_tp_size)
                    * (info.attn_tp_size // self.attn_tp_size),
                    (self.kv_args.engine_rank % self.attn_tp_size + 1)
                    * (info.attn_tp_size // self.attn_tp_size),
                )
            )
            # For MLA models, we can retrieve KVCache from only one prefill rank, but we still need to maintain
            # multiple connections in the connection pool and have to send dummy requests to other prefill ranks,
            # or the KVPoll will never be set correctly
            target_tp_rank = target_tp_ranks[0]
            required_dst_info_num = 1
            if self.is_mla_backend:
                required_prefill_response_num = 1
            else:
                required_prefill_response_num = info.attn_tp_size // self.attn_tp_size
```
**EN:** This block continues `_resolve_rank_mapping` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow.
**CN:** 这一段延续了 `_resolve_rank_mapping` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。

### Lines 310-327: Method `_resolve_rank_mapping` logic (part 3)
```python

        # CP rank mapping — decode cp size should be equal to 1
        assert self.attn_cp_size == 1, (
            f"Decode cp size ({self.attn_cp_size}) should be equal to 1",
        )
        if self.attn_cp_size == info.attn_cp_size:
            assert info.attn_cp_size == 1, (
                f"When prefill cp size is 1, attn cp size should be 1, but got {self.attn_cp_size}",
            )
            target_cp_ranks = [self.attn_cp_rank]
        else:
            target_cp_ranks = list(range(info.attn_cp_size))
            if not self.enable_all_cp_ranks_for_transfer:
                # Only retrieve from prefill CP rank 0 when not using all ranks
                target_cp_ranks = target_cp_ranks[:1]
                required_prefill_response_num *= 1
            else:
                required_prefill_response_num *= info.attn_cp_size // self.attn_cp_size
```
**EN:** This block continues `_resolve_rank_mapping` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `size`.
**CN:** 这一段延续了 `_resolve_rank_mapping` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `size`。

### Lines 328-344: Method `_resolve_rank_mapping` logic (part 4)
```python

        # PP rank mapping — decode pp size should be equal to prefill pp size or 1
        assert self.pp_size == info.pp_size or self.pp_size == 1, (
            f"Decode pp size ({self.pp_size}) should be equal to prefill pp size ({info.pp_size}) or 1",
        )
        if info.pp_size == self.pp_size:
            target_pp_ranks = [self.pp_rank]
        else:
            target_pp_ranks = list(range(info.pp_size))
            required_prefill_response_num *= info.pp_size // self.pp_size

        info.target_tp_rank = target_tp_rank
        info.target_tp_ranks = target_tp_ranks
        info.target_cp_ranks = target_cp_ranks
        info.target_pp_ranks = target_pp_ranks
        info.required_dst_info_num = required_dst_info_num
        info.required_prefill_response_num = required_prefill_response_num
```
**EN:** This block continues `_resolve_rank_mapping` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `size`.
**CN:** 这一段延续了 `_resolve_rank_mapping` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `size`。

### Lines 346-355: Method `_sync_bootstrap_port_across_nodes` signature and setup
```python
    def _sync_bootstrap_port_across_nodes(self, local_port: int) -> int:
        """Broadcast world-rank-0's bootstrap port to all prefill ranks.

        Required for multi-node prefill when the launcher auto-reserves a
        free port per host (e.g. Dynamo's
        `_reserve_disaggregation_bootstrap_port`): without sync, non-leader
        ranks register to `<leader_ip>:<their_local_port>`, hit
        `Connection refused`, and the leader's `prefill_port_table` ends
        up missing rows.
        """
```
**EN:** This block defines the method `_sync_bootstrap_port_across_nodes` on `CommonKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_sync_bootstrap_port_across_nodes`. Notable operations include `host`.
**CN:** 这一段定义了method `_sync_bootstrap_port_across_nodes`（属于 `CommonKVManager`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_sync_bootstrap_port_across_nodes`。 值得注意的操作包括 `host`。

### Lines 356-374: Method `_sync_bootstrap_port_across_nodes` logic (part 1)
```python
        if not self.dist_init_addr or self.server_args.nnodes == 1:
            return local_port

        if not (dist.is_available() and dist.is_initialized()):
            raise RuntimeError(
                "torch.distributed must be initialised before "
                "CommonKVManager registers to the bootstrap server in "
                "multi-node prefill mode."
            )

        world_group = get_world_group()
        synced_port = world_group.broadcast_object(local_port, src=0)
        if synced_port != local_port:
            logger.info(
                f"Synced disaggregation bootstrap port from leader: "
                f"local={local_port} -> leader={synced_port} "
                f"(world_rank={world_group.rank_in_group})"
            )
        return synced_port
```
**EN:** This block continues `_sync_bootstrap_port_across_nodes` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `not`, `is_available`, `is_initialized`, `RuntimeError`.
**CN:** 这一段延续了 `_sync_bootstrap_port_across_nodes` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `not`、`is_available`、`is_initialized`、`RuntimeError`。

### Lines 376-377: Method `register_to_bootstrap` signature and setup
```python
    def register_to_bootstrap(self):
        """Register prefill server info to bootstrap server via HTTP PUT."""
```
**EN:** This block defines the method `register_to_bootstrap` on `CommonKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `register_to_bootstrap`.
**CN:** 这一段定义了method `register_to_bootstrap`（属于 `CommonKVManager`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `register_to_bootstrap`。

### Lines 378-403: Method `register_to_bootstrap` logic (part 1)
```python
        if self.dist_init_addr:
            # Multi-node case: bootstrap server's host is dist_init_addr
            host = NetworkAddress.parse(self.dist_init_addr).resolved().host
        else:
            # Single-node case: bootstrap server's host is the same as http server's host
            host = self.bootstrap_host

        bootstrap_na = NetworkAddress(host, self.bootstrap_port)
        url = f"{bootstrap_na.to_url()}/route"
        payload = {
            "attn_tp_size": self.attn_tp_size,
            "attn_tp_rank": self.attn_tp_rank,
            "attn_cp_size": self.attn_cp_size,
            "attn_cp_rank": self.attn_cp_rank,
            "attn_dp_size": self.attn_dp_size,
            "attn_dp_rank": self.attn_dp_rank,
            "pp_size": self.pp_size,
            "pp_rank": self.pp_rank,
            "system_dp_size": self.system_dp_size,
            "system_dp_rank": self.system_dp_rank,
            "rank_ip": self.local_ip,
            "rank_port": self.rank_port,
            "page_size": self.kv_args.page_size,
            "kv_cache_dtype": self.server_args.kv_cache_dtype,
            "load_balance_method": self.server_args.load_balance_method,
        }
```
**EN:** This block continues `register_to_bootstrap` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `parse`, `resolved`, `NetworkAddress`, `to_url`.
**CN:** 这一段延续了 `register_to_bootstrap` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `parse`、`resolved`、`NetworkAddress`、`to_url`。

### Lines 404-427: Method `register_to_bootstrap` logic (part 2)
```python

        max_retries, initial_delay, max_delay = 5, 1.0, 30.0
        for attempt in range(max_retries):
            try:
                response = requests.put(url, json=payload, timeout=5)
                if response.status_code == 200:
                    logger.debug("Prefill successfully registered to bootstrap server.")
                    return
                logger.warning(
                    f"Prefill register attempt {attempt + 1}/{max_retries} failed: status {response.status_code}"
                )
            except Exception as e:
                # Walk to root cause to skip misleading urllib3 wrapper messages
                cause = e
                while cause.__cause__ is not None:
                    cause = cause.__cause__
                logger.warning(
                    f"Prefill register attempt {attempt + 1}/{max_retries} failed: {cause}"
                )
            if attempt == max_retries - 1:
                break
            delay = min(initial_delay * (2**attempt), max_delay) * (
                0.75 + 0.25 * (time.monotonic() % 1)
            )
```
**EN:** This block continues `register_to_bootstrap` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `put`, `debug`, `warning`, `monotonic`.
**CN:** 这一段延续了 `register_to_bootstrap` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `put`、`debug`、`warning`、`monotonic`。

### Lines 428-431: Method `register_to_bootstrap` logic (part 3)
```python
            time.sleep(delay)
        logger.error(
            f"Prefill instance failed to register to bootstrap server after {max_retries} retries"
        )
```
**EN:** This block continues `register_to_bootstrap` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `sleep`, `error`.
**CN:** 这一段延续了 `register_to_bootstrap` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `sleep`、`error`。

### Lines 433-439: Method `_connect`
```python
    @cache
    def _connect(self, endpoint: str, is_ipv6: bool = False):
        socket = zmq.Context().socket(zmq.PUSH)
        if is_ipv6:
            socket.setsockopt(zmq.IPV6, 1)
        socket.connect(endpoint)
        return socket
```
**EN:** This block defines the method `_connect` on `CommonKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_connect`. Notable operations include `Context`, `socket`, `setsockopt`, `connect`.
**CN:** 这一段定义了method `_connect`（属于 `CommonKVManager`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_connect`。 值得注意的操作包括 `Context`、`socket`、`setsockopt`、`connect`。

### Lines 441-443: Method `get_mha_kv_ptrs_with_pp` signature and setup
```python
    def get_mha_kv_ptrs_with_pp(
        self, src_kv_ptrs: List[int], dst_kv_ptrs: List[int]
    ) -> Tuple[List[int], List[int], List[int], List[int], int]:
```
**EN:** This block defines the method `get_mha_kv_ptrs_with_pp` on `CommonKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `get_mha_kv_ptrs_with_pp`.
**CN:** 这一段定义了method `get_mha_kv_ptrs_with_pp`（属于 `CommonKVManager`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `get_mha_kv_ptrs_with_pp`。

### Lines 444-467: Method `get_mha_kv_ptrs_with_pp` logic (part 1)
```python
        start_layer = self.kv_args.prefill_start_layer
        num_kv_layers = len(src_kv_ptrs) // 2
        end_layer = start_layer + num_kv_layers
        dst_num_total_layers = len(dst_kv_ptrs) // 2
        src_k_ptrs = src_kv_ptrs[:num_kv_layers]
        src_v_ptrs = src_kv_ptrs[num_kv_layers:]
        if num_kv_layers == dst_num_total_layers:
            dst_k_ptrs = dst_kv_ptrs[:dst_num_total_layers]
            dst_v_ptrs = dst_kv_ptrs[dst_num_total_layers:]
        elif (
            num_kv_layers < dst_num_total_layers
            and dst_num_total_layers % num_kv_layers != 0
        ):
            # Case: Decode has draft model KV while Prefill is deployed without speculative decoding
            # dst_kv_ptrs layout: [K_main..., V_main..., draft_K..., draft_V...]
            multiplier_ratio = dst_num_total_layers // num_kv_layers
            dst_k_ptrs = dst_kv_ptrs[start_layer:end_layer]
            v_ptr_offset = num_kv_layers * multiplier_ratio
            dst_v_ptrs = dst_kv_ptrs[
                v_ptr_offset + start_layer : v_ptr_offset + end_layer
            ]
        else:
            # Decode pp size should be equal to prefill pp size or 1
            dst_k_ptrs = dst_kv_ptrs[start_layer:end_layer]
```
**EN:** This block continues `get_mha_kv_ptrs_with_pp` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `elif`.
**CN:** 这一段延续了 `get_mha_kv_ptrs_with_pp` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `elif`。

### Lines 468-472: Method `get_mha_kv_ptrs_with_pp` logic (part 2)
```python
            dst_v_ptrs = dst_kv_ptrs[
                dst_num_total_layers + start_layer : dst_num_total_layers + end_layer
            ]
        layers_current_pp_stage = len(src_k_ptrs)
        return src_k_ptrs, src_v_ptrs, dst_k_ptrs, dst_v_ptrs, layers_current_pp_stage
```
**EN:** This block continues `get_mha_kv_ptrs_with_pp` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow.
**CN:** 这一段延续了 `get_mha_kv_ptrs_with_pp` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。

### Lines 474-476: Method `get_mla_kv_ptrs_with_pp` signature and setup
```python
    def get_mla_kv_ptrs_with_pp(
        self, src_kv_ptrs: List[int], dst_kv_ptrs: List[int]
    ) -> Tuple[List[int], List[int], int]:
```
**EN:** This block defines the method `get_mla_kv_ptrs_with_pp` on `CommonKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `get_mla_kv_ptrs_with_pp`.
**CN:** 这一段定义了method `get_mla_kv_ptrs_with_pp`（属于 `CommonKVManager`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `get_mla_kv_ptrs_with_pp`。

### Lines 477-494: Method `get_mla_kv_ptrs_with_pp` logic (part 1)
```python
        # Fast path: both sides use exactly the same PP layout
        if len(src_kv_ptrs) == len(dst_kv_ptrs):
            return src_kv_ptrs, dst_kv_ptrs, len(src_kv_ptrs)

        mla_ratios = getattr(self.kv_args, "mla_compression_ratios", None)
        if mla_ratios:
            # Compressed-MLA (e.g. DeepSeek V4): the flat list is organized
            # by buffer type (compression-ratio bucket) rather than by
            # layer, so we locate the sub-range for this PP stage inside each
            # section of the dst flat list.
            sliced_src_kv_ptrs, sliced_dst_kv_ptrs = self._mla_slice_ptrs_for_pp(
                src_kv_ptrs, dst_kv_ptrs, mla_ratios
            )
            return (
                sliced_src_kv_ptrs,
                sliced_dst_kv_ptrs,
                len(sliced_src_kv_ptrs),
            )
```
**EN:** This block continues `get_mla_kv_ptrs_with_pp` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `MLA`, `_mla_slice_ptrs_for_pp`.
**CN:** 这一段延续了 `get_mla_kv_ptrs_with_pp` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `MLA`、`_mla_slice_ptrs_for_pp`。

### Lines 495-501: Method `get_mla_kv_ptrs_with_pp` logic (part 2)
```python

        # Regular MLA PP slicing
        start_layer = self.kv_args.prefill_start_layer
        end_layer = start_layer + len(src_kv_ptrs)
        # Decode pp size should be equal to prefill pp size or 1
        sliced_dst_kv_ptrs = dst_kv_ptrs[start_layer:end_layer]
        return src_kv_ptrs, sliced_dst_kv_ptrs, len(src_kv_ptrs)
```
**EN:** This block continues `get_mla_kv_ptrs_with_pp` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow.
**CN:** 这一段延续了 `get_mla_kv_ptrs_with_pp` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。

### Lines 503-536: Method `_mla_slice_ptrs_for_pp` signature and setup
```python
    def _mla_slice_ptrs_for_pp(
        self,
        src_kv_ptrs: List[int],
        dst_kv_ptrs: List[int],
        mla_ratios: List[int],
    ) -> Tuple[List[int], List[int]]:
        """Produce aligned (src, dst) pointer lists for compressed-MLA
        pools (e.g. DeepSeek V4) under PP.

        The pool produces two possible flat-list layouts (selected via dst
        length):

        - kv_data layout, length = 2 * c4_L + c128_L:
            [c4_layer_{0..c4_L-1},
             c4_indexer_layer_{0..c4_L-1},
             c128_layer_{0..c128_L-1}]
          Each section is indexed by compressed-layer id within that
          compression bucket.

        - state_data layout, length = swa_L + 2 * c4_L + c128_L:
            [swa_layer_{0..swa_L-1},
             compress_state_{non-None, c4_L + c128_L},
             indexer_compress_state_{non-None, c4_L}]
          ``swa_L`` is the SWA pool's actual buffer count
          (``num_effective_layers``), which can be smaller than
          ``len(mla_ratios)`` when the HF config's ``compress_ratios``
          list contains entries for layers not materialized into the SWA
          pool (e.g. an MTP/nextn slot at the tail).

        src is already PP-filtered on the prefill side. dst is the
        decode-side full-model list (when decode is PP=1). We slice dst to
        match src's PP stage. If src itself is also full-model, it is
        returned unchanged.
        """
```
**EN:** This block defines the method `_mla_slice_ptrs_for_pp` on `CommonKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_mla_slice_ptrs_for_pp`. Notable operations include `aligned`, `pools`, `layouts`, `count`.
**CN:** 这一段定义了method `_mla_slice_ptrs_for_pp`（属于 `CommonKVManager`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_mla_slice_ptrs_for_pp`。 值得注意的操作包括 `aligned`、`pools`、`layouts`、`count`。

### Lines 537-559: Method `_mla_slice_ptrs_for_pp` logic (part 1)
```python
        start_layer = self.kv_args.prefill_start_layer
        end_layer = getattr(self.kv_args, "prefill_end_layer", None)
        assert end_layer is not None, (
            "KVArgs.prefill_end_layer must be set when using "
            "compressed-MLA PD with PP"
        )

        c4_full = sum(1 for r in mla_ratios if r == 4)
        c128_full = sum(1 for r in mla_ratios if r == 128)
        kv_layout_len = 2 * c4_full + c128_full

        c4_off_s = sum(1 for r in mla_ratios[:start_layer] if r == 4)
        c4_off_e = sum(1 for r in mla_ratios[:end_layer] if r == 4)
        c128_off_s = sum(1 for r in mla_ratios[:start_layer] if r == 128)
        c128_off_e = sum(1 for r in mla_ratios[:end_layer] if r == 128)

        if len(dst_kv_ptrs) == kv_layout_len:
            sliced_dst = (
                list(dst_kv_ptrs[c4_off_s:c4_off_e])
                + list(dst_kv_ptrs[c4_full + c4_off_s : c4_full + c4_off_e])
                + list(dst_kv_ptrs[2 * c4_full + c128_off_s : 2 * c4_full + c128_off_e])
            )
            return src_kv_ptrs, sliced_dst
```
**EN:** This block continues `_mla_slice_ptrs_for_pp` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow.
**CN:** 这一段延续了 `_mla_slice_ptrs_for_pp` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。

### Lines 560-591: Method `_mla_slice_ptrs_for_pp` logic (part 2)
```python

        # State-data layout. ``swa_L`` is derived from the actual dst
        # length so we tolerate cases where the SWA pool has fewer
        # buffers than ``len(mla_ratios)`` (e.g. nextn padding).
        swa_L = len(dst_kv_ptrs) - 2 * c4_full - c128_full
        if swa_L < 0 or swa_L > len(mla_ratios):
            raise ValueError(
                f"Unexpected compressed-MLA dst_kv_ptrs length "
                f"{len(dst_kv_ptrs)}; expected either {kv_layout_len} "
                f"(kv_data) or swa_L + {2 * c4_full + c128_full} "
                f"(state_data) given compression_ratios "
                f"(c4={c4_full}, c128={c128_full}, "
                f"total={len(mla_ratios)})."
            )
        # Guard against asking the prefill side to read past the SWA
        # pool boundary.
        assert end_layer <= swa_L, (
            f"prefill_end_layer ({end_layer}) exceeds dst SWA pool "
            f"buffer count ({swa_L}); compression_ratios may include "
            f"layers (e.g. nextn) that the SWA pool does not cover."
        )

        # compress_state non-None count up to L = count(r != 0).
        c_non_zero_s = sum(1 for r in mla_ratios[:start_layer] if r != 0)
        c_non_zero_e = sum(1 for r in mla_ratios[:end_layer] if r != 0)
        compress_section_start = swa_L
        indexer_section_start = swa_L + (c4_full + c128_full)
        sliced_dst = (
            list(dst_kv_ptrs[start_layer:end_layer])
            + list(
                dst_kv_ptrs[
                    compress_section_start
```
**EN:** This block continues `_mla_slice_ptrs_for_pp` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `ValueError`, `prefill_end_layer`, `count`, `layers`.
**CN:** 这一段延续了 `_mla_slice_ptrs_for_pp` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `ValueError`、`prefill_end_layer`、`count`、`layers`。

### Lines 592-603: Method `_mla_slice_ptrs_for_pp` logic (part 3)
```python
                    + c_non_zero_s : compress_section_start
                    + c_non_zero_e
                ]
            )
            + list(
                dst_kv_ptrs[
                    indexer_section_start + c4_off_s : indexer_section_start + c4_off_e
                ]
            )
        )

        return src_kv_ptrs, sliced_dst
```
**EN:** This block continues `_mla_slice_ptrs_for_pp` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow.
**CN:** 这一段延续了 `_mla_slice_ptrs_for_pp` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。

### Lines 606-606: Class `CommonKVSender` declaration
```python
class CommonKVSender(BaseKVSender):
```
**EN:** This block declares the class `CommonKVSender` and establishes its responsibility inside common backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `CommonKVSender`.
**CN:** 这一段声明了类 `CommonKVSender`，并说明它在common 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `CommonKVSender`。

### Lines 607-609: Method `__init__` signature and setup
```python
    def __init__(
        self,
        mgr: CommonKVManager,
```
**EN:** This block defines the method `__init__` on `CommonKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `CommonKVSender`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 610-628: Method `__init__` logic (part 1)
```python
        bootstrap_addr: str,
        bootstrap_room: int,
        dest_tp_ranks: List[int],
        pp_rank: int,
    ):
        self.kv_mgr = mgr
        self.bootstrap_room = bootstrap_room
        self.aux_index = None
        self.bootstrap_server_url = bootstrap_addr
        self.conclude_state: Optional[KVPoll] = None
        self._transfer_metric = KVTransferMetric()
        self._transfer_num_kv_indices = 0
        self._transfer_num_state_indices = 0
        # inner state
        self.curr_idx = 0
        if self.kv_mgr.is_dummy_cp_rank:
            # Non-authoritative CP ranks are dummy participants.
            self.kv_mgr.update_status(self.bootstrap_room, KVPoll.WaitingForInput)
            return
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `KVTransferMetric`, `update_status`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `KVTransferMetric`、`update_status`。

### Lines 629-652: Method `__init__` logic (part 2)
```python

        self.kv_mgr.update_status(self.bootstrap_room, KVPoll.Bootstrapping)
        if self.kv_mgr.server_args.dp_size > 1:
            if self.kv_mgr.server_args.load_balance_method != "follow_bootstrap_room":
                self._register_prefill_dp_rank()
            elif (
                self.kv_mgr.attn_dp_rank
                != self.bootstrap_room % self.kv_mgr.server_args.dp_size
            ):
                # follow_bootstrap_room was overridden by external routed_dp_rank
                if envs.SGLANG_DISAGGREGATION_FORCE_QUERY_PREFILL_DP_RANK.get():
                    self._register_prefill_dp_rank()
                else:
                    self.kv_mgr.record_failure(
                        self.bootstrap_room,
                        f"follow_bootstrap_room conflict: dispatched to dp_rank "
                        f"{self.kv_mgr.attn_dp_rank} but bootstrap_room "
                        f"{self.bootstrap_room} implies dp_rank "
                        f"{self.bootstrap_room % self.kv_mgr.server_args.dp_size}. "
                        f"Set SGLANG_DISAGGREGATION_FORCE_QUERY_PREFILL_DP_RANK=1 "
                        f"to allow mixed routing.",
                    )
                    self.kv_mgr.update_status(self.bootstrap_room, KVPoll.Failed)
                    return
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `update_status`, `_register_prefill_dp_rank`, `elif`, `get`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `update_status`、`_register_prefill_dp_rank`、`elif`、`get`。

### Lines 654-668: Method `_register_prefill_dp_rank`
```python
    def _register_prefill_dp_rank(self):
        """Register this request's prefill dp_rank to the bootstrap server."""
        url = f"http://{self.bootstrap_server_url}/register_dp_rank"
        payload = {
            "bootstrap_room": self.bootstrap_room,
            "dp_rank": self.kv_mgr.attn_dp_rank,
        }
        try:
            response = requests.post(url, json=payload, timeout=5)
            if response.status_code != 200:
                logger.error(
                    f"Failed to register prefill dp_rank: {response.status_code}, {response.text}"
                )
        except Exception as e:
            logger.error(f"Failed to register prefill dp_rank: {e}")
```
**EN:** This block defines the method `_register_prefill_dp_rank` on `CommonKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_register_prefill_dp_rank`. Notable operations include `post`, `error`.
**CN:** 这一段定义了method `_register_prefill_dp_rank`（属于 `CommonKVSender`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_register_prefill_dp_rank`。 值得注意的操作包括 `post`、`error`。

### Lines 670-675: Method `init`
```python
    def init(self, num_kv_indices: int, aux_index: Optional[int] = None):
        self.num_kv_indices = num_kv_indices
        self.aux_index = aux_index
        logger.debug(
            f"CommonKVSender init with num_kv_indices: {num_kv_indices} and aux_index: {aux_index}"
        )
```
**EN:** This block defines the method `init` on `CommonKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `init`. Notable operations include `debug`.
**CN:** 这一段定义了method `init`（属于 `CommonKVSender`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `init`。 值得注意的操作包括 `debug`。

### Lines 677-678: Method `pop_decode_prefix_len`
```python
    def pop_decode_prefix_len(self) -> int:
        return 0
```
**EN:** This block defines the method `pop_decode_prefix_len` on `CommonKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `pop_decode_prefix_len`.
**CN:** 这一段定义了method `pop_decode_prefix_len`（属于 `CommonKVSender`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `pop_decode_prefix_len`。

### Lines 680-681: Method `should_send_kv_chunk`
```python
    def should_send_kv_chunk(self, num_pages: int, last_chunk: bool) -> bool:
        return num_pages > 0
```
**EN:** This block defines the method `should_send_kv_chunk` on `CommonKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `should_send_kv_chunk`.
**CN:** 这一段定义了method `should_send_kv_chunk`（属于 `CommonKVSender`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `should_send_kv_chunk`。

### Lines 683-689: Method `get_transfer_metric`
```python
    def get_transfer_metric(self) -> KVTransferMetric:
        total_bytes = self._transfer_num_kv_indices * self.kv_mgr.kv_item_lens_sum
        total_bytes += (
            self._transfer_num_state_indices * self.kv_mgr.state_item_lens_sum
        )
        self._transfer_metric.transfer_total_bytes = total_bytes
        return self._transfer_metric
```
**EN:** This block defines the method `get_transfer_metric` on `CommonKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `get_transfer_metric`.
**CN:** 这一段定义了method `get_transfer_metric`（属于 `CommonKVSender`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `get_transfer_metric`。

### Lines 691-700: Method `_record_transfer_indices`
```python
    def _record_transfer_indices(
        self,
        kv_indices: npt.NDArray[np.int32],
        state_indices: Optional[List],
    ):
        self._transfer_num_kv_indices += len(kv_indices)
        if state_indices:
            for component_indices in state_indices:
                if component_indices is not None:
                    self._transfer_num_state_indices += len(component_indices)
```
**EN:** This block defines the method `_record_transfer_indices` on `CommonKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_record_transfer_indices`.
**CN:** 这一段定义了method `_record_transfer_indices`（属于 `CommonKVSender`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_record_transfer_indices`。

### Lines 702-707: Method `send`
```python
    def send(
        self,
        kv_indices: npt.NDArray[np.int32],
        state_indices: Optional[List] = None,
    ):
        pass
```
**EN:** This block defines the method `send` on `CommonKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `send`.
**CN:** 这一段定义了method `send`（属于 `CommonKVSender`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `send`。

### Lines 709-710: Method `poll`
```python
    def poll(self) -> KVPoll:
        pass
```
**EN:** This block defines the method `poll` on `CommonKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `poll`.
**CN:** 这一段定义了method `poll`（属于 `CommonKVSender`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `poll`。

### Lines 712-713: Method `failure_exception`
```python
    def failure_exception(self):
        raise Exception("Fake KVReceiver Exception")
```
**EN:** This block defines the method `failure_exception` on `CommonKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `failure_exception`. Notable operations include `Exception`.
**CN:** 这一段定义了method `failure_exception`（属于 `CommonKVSender`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `failure_exception`。 值得注意的操作包括 `Exception`。

### Lines 715-720: Method `clear`
```python
    def clear(self) -> None:
        self.kv_mgr.request_status.pop(self.bootstrap_room, None)
        if hasattr(self.kv_mgr, "req_to_decode_prefix_len"):
            self.kv_mgr.req_to_decode_prefix_len.pop(self.bootstrap_room, None)
        if hasattr(self.kv_mgr, "transfer_infos"):
            self.kv_mgr.transfer_infos.pop(self.bootstrap_room, None)
```
**EN:** This block defines the method `clear` on `CommonKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `clear`. Notable operations include `pop`.
**CN:** 这一段定义了method `clear`（属于 `CommonKVSender`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `clear`。 值得注意的操作包括 `pop`。

### Lines 722-728: Method `abort`
```python
    def abort(self):
        self.kv_mgr.record_failure(
            self.bootstrap_room,
            "Aborted by AbortReq.",
        )
        self.kv_mgr.update_status(self.bootstrap_room, KVPoll.Failed)
        self.conclude_state = KVPoll.Failed
```
**EN:** This block defines the method `abort` on `CommonKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `abort`. Notable operations include `record_failure`, `update_status`.
**CN:** 这一段定义了method `abort`（属于 `CommonKVSender`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `abort`。 值得注意的操作包括 `record_failure`、`update_status`。

### Lines 731-731: Class `CommonKVReceiver` declaration
```python
class CommonKVReceiver(BaseKVReceiver):
```
**EN:** This block declares the class `CommonKVReceiver` and establishes its responsibility inside common backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `CommonKVReceiver`.
**CN:** 这一段声明了类 `CommonKVReceiver`，并说明它在common 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `CommonKVReceiver`。

### Lines 732-736: Supporting state inside `CommonKVReceiver`
```python
    _ctx = zmq.Context()
    _socket_cache = {}
    _socket_locks = {}
    _global_lock = threading.Lock()

```
**EN:** This block adds supporting state or helper logic inside `CommonKVReceiver`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `Context`, `Lock`.
**CN:** 这一段为 `CommonKVReceiver` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `Context`、`Lock`。

### Lines 737-749: Method `__init__`
```python
    def __init__(
        self,
        mgr: CommonKVManager,
        bootstrap_addr: str,
        bootstrap_room: Optional[int] = None,
    ):
        self.bootstrap_room = bootstrap_room
        self.bootstrap_addr = bootstrap_addr
        self.kv_mgr = mgr
        self.conclude_state: Optional[KVPoll] = None
        self.require_staging: bool = False
        self.kv_mgr.addr_to_rooms_tracker[self.bootstrap_addr].add(self.bootstrap_room)
        self.kv_mgr.update_status(self.bootstrap_room, KVPoll.Bootstrapping)
```
**EN:** This block defines the method `__init__` on `CommonKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `__init__`. Notable operations include `add`, `update_status`.
**CN:** 这一段定义了method `__init__`（属于 `CommonKVReceiver`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `add`、`update_status`。

### Lines 751-753: Method `init` signature and setup
```python
    def init(self, prefill_dp_rank: int):
        if self.bootstrap_addr not in self.kv_mgr.prefill_info_table:
            self.kv_mgr.record_failure(
```
**EN:** This block defines the method `init` on `CommonKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `init`. Notable operations include `record_failure`.
**CN:** 这一段定义了method `init`（属于 `CommonKVReceiver`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `init`。 值得注意的操作包括 `record_failure`。

### Lines 754-774: Method `init` logic (part 1)
```python
                self.bootstrap_room,
                f"Prefill server with bootstrap_addr: {self.bootstrap_addr} is healthy before, but now it is down. Request (bootstrap_room: {self.bootstrap_room}) has been marked as failed.",
            )
            self.conclude_state = KVPoll.Failed
            self.kv_mgr.update_status(self.bootstrap_room, KVPoll.Failed)
            return

        # Read pre-computed rank mapping from prefill_info (computed in try_ensure_parallel_info)
        self.prefill_info = self.kv_mgr.prefill_info_table[self.bootstrap_addr]
        self.target_tp_rank = self.prefill_info.target_tp_rank
        self.target_tp_ranks = self.prefill_info.target_tp_ranks
        self.target_cp_ranks = self.prefill_info.target_cp_ranks
        self.target_pp_ranks = self.prefill_info.target_pp_ranks
        self.required_dst_info_num = self.prefill_info.required_dst_info_num
        self.required_prefill_response_num = (
            self.prefill_info.required_prefill_response_num
        )

        self.kv_mgr.required_prefill_response_num_table[self.bootstrap_room] = (
            self.required_prefill_response_num
        )
```
**EN:** This block continues `init` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `Request`, `update_status`, `prefill_info`.
**CN:** 这一段延续了 `init` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `Request`、`update_status`、`prefill_info`。

### Lines 775-786: Method `init` logic (part 2)
```python

        if self.kv_mgr.enable_staging:
            self.require_staging = (
                self.prefill_info.attn_tp_size != 0
                and self.prefill_info.attn_tp_size != self.kv_mgr.attn_tp_size
            )

        self.prefill_dp_rank = prefill_dp_rank
        self._setup_bootstrap_infos()
        if self.conclude_state == KVPoll.Failed:
            return
        self.kv_mgr.update_status(self.bootstrap_room, KVPoll.WaitingForInput)
```
**EN:** This block continues `init` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `_setup_bootstrap_infos`, `update_status`.
**CN:** 这一段延续了 `init` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `_setup_bootstrap_infos`、`update_status`。

### Lines 788-790: Method `_setup_bootstrap_infos` signature and setup
```python
    def _setup_bootstrap_infos(self):
        all_bootstrap_infos = []
        # NOTE: key distinguished by bootstrap_addr, prefill_dp_rank, prefill_cp_rank, and target_tp_rank
```
**EN:** This block defines the method `_setup_bootstrap_infos` on `CommonKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_setup_bootstrap_infos`.
**CN:** 这一段定义了method `_setup_bootstrap_infos`（属于 `CommonKVReceiver`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_setup_bootstrap_infos`。

### Lines 791-814: Method `_setup_bootstrap_infos` logic (part 1)
```python
        for target_cp_rank in self.target_cp_ranks:
            bootstrap_key = f"{self.bootstrap_addr}_{self.prefill_dp_rank}_{target_cp_rank}_{self.target_tp_rank}"

            if bootstrap_key not in self.kv_mgr.connection_pool:
                bootstrap_infos = []
                for target_tp_rank in self.target_tp_ranks:
                    # Enable higher PP ranks to be bootstrapped earlier to make PP PD requests bootstrap more robust
                    for target_pp_rank in reversed(self.target_pp_ranks):
                        bootstrap_info = self._get_bootstrap_info_from_server(
                            self.prefill_dp_rank,
                            target_cp_rank,
                            target_tp_rank,
                            target_pp_rank,
                        )
                        if bootstrap_info is not None:
                            if self.kv_mgr.is_mla_backend:
                                # For MLA: target_tp_rank is the selected real rank, others are dummy ranks
                                bootstrap_info["is_dummy"] = not bool(
                                    target_tp_rank == self.target_tp_rank
                                    or self.target_tp_rank is None
                                )
                            else:
                                # For non-MLA: all target_tp_ranks are selected real ranks
                                bootstrap_info["is_dummy"] = False
```
**EN:** This block continues `_setup_bootstrap_infos` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `reversed`, `_get_bootstrap_info_from_server`.
**CN:** 这一段延续了 `_setup_bootstrap_infos` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `reversed`、`_get_bootstrap_info_from_server`。

### Lines 815-837: Method `_setup_bootstrap_infos` logic (part 2)
```python
                            logger.debug(
                                f"Fetched bootstrap info: {bootstrap_info} for DP {self.prefill_dp_rank} CP {target_cp_rank} TP {target_tp_rank} PP {target_pp_rank}"
                            )
                            bootstrap_infos.append(bootstrap_info)
                        else:
                            self.kv_mgr.record_failure(
                                self.bootstrap_room,
                                f"Could not fetch bootstrap info for: prefill_dp_rank: {self.prefill_dp_rank} prefill_cp_rank: {target_cp_rank} target_tp_rank: {target_tp_rank} and target_pp_rank {target_pp_rank}",
                            )
                            self.conclude_state = KVPoll.Failed
                            self.kv_mgr.update_status(
                                self.bootstrap_room, KVPoll.Failed
                            )
                            self.bootstrap_infos = None
                            return

                self.bootstrap_infos = bootstrap_infos
                self.kv_mgr.connection_pool[bootstrap_key] = self.bootstrap_infos

                # Register kv_args only once to prefill KVManager according to the info fetched from the bootstrap server
                self._register_kv_args()
            else:
                self.bootstrap_infos = self.kv_mgr.connection_pool[bootstrap_key]
```
**EN:** This block continues `_setup_bootstrap_infos` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `debug`, `append`, `record_failure`, `update_status`.
**CN:** 这一段延续了 `_setup_bootstrap_infos` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `debug`、`append`、`record_failure`、`update_status`。

### Lines 838-842: Method `_setup_bootstrap_infos` logic (part 3)
```python

            assert len(self.bootstrap_infos) > 0
            all_bootstrap_infos.extend(self.bootstrap_infos)

        self.bootstrap_infos = all_bootstrap_infos
```
**EN:** This block continues `_setup_bootstrap_infos` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `extend`.
**CN:** 这一段延续了 `_setup_bootstrap_infos` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `extend`。

### Lines 844-861: Method `_get_bootstrap_info_from_server`
```python
    def _get_bootstrap_info_from_server(
        self, prefill_dp_rank, prefill_cp_rank, target_tp_rank, target_pp_rank
    ):
        """Fetch the bootstrap info from the bootstrap server."""
        try:
            url = f"http://{self.bootstrap_addr}/route?prefill_dp_rank={prefill_dp_rank}&prefill_cp_rank={prefill_cp_rank}&target_tp_rank={target_tp_rank}&target_pp_rank={target_pp_rank}"
            response = requests.get(url, timeout=5)
            if response.status_code == 200:
                bootstrap_info = response.json()
                return bootstrap_info
            else:
                logger.error(
                    f"Failed to get prefill server info: {response.status_code}, {response.text}"
                )
                return None
        except Exception as e:
            logger.error(f"Error fetching prefill info from bootstrap: {e}")
            return None
```
**EN:** This block defines the method `_get_bootstrap_info_from_server` on `CommonKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_get_bootstrap_info_from_server`. Notable operations include `get`, `json`, `error`.
**CN:** 这一段定义了method `_get_bootstrap_info_from_server`（属于 `CommonKVReceiver`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_get_bootstrap_info_from_server`。 值得注意的操作包括 `get`、`json`、`error`。

### Lines 863-884: Method `query_prefill_dp_ranks`
```python
    @staticmethod
    def query_prefill_dp_ranks(
        bootstrap_addr: str, bootstrap_rooms: List[int]
    ) -> Dict[str, int]:
        """Batch query prefill dp_ranks for given bootstrap_rooms."""
        try:
            url = f"http://{bootstrap_addr}/query_dp_ranks"
            response = requests.post(
                url,
                json={"bootstrap_rooms": bootstrap_rooms},
                timeout=5,
            )
            if response.status_code == 200:
                return response.json()
            else:
                logger.error(
                    f"Failed to query dp_ranks: {response.status_code}, {response.text}"
                )
                return {}
        except Exception as e:
            logger.error(f"Error querying dp_ranks from bootstrap: {e}")
            return {}
```
**EN:** This block defines the method `query_prefill_dp_ranks` on `CommonKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `query_prefill_dp_ranks`. Notable operations include `post`, `json`, `error`.
**CN:** 这一段定义了method `query_prefill_dp_ranks`（属于 `CommonKVReceiver`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `query_prefill_dp_ranks`。 值得注意的操作包括 `post`、`json`、`error`。

### Lines 886-896: Method `_connect`
```python
    @classmethod
    def _connect(cls, endpoint: str, is_ipv6: bool = False):
        with cls._global_lock:
            if endpoint not in cls._socket_cache:
                sock = cls._ctx.socket(zmq.PUSH)
                if is_ipv6:
                    sock.setsockopt(zmq.IPV6, 1)
                sock.connect(endpoint)
                cls._socket_cache[endpoint] = sock
                cls._socket_locks[endpoint] = threading.Lock()
            return cls._socket_cache[endpoint], cls._socket_locks[endpoint]
```
**EN:** This block defines the method `_connect` on `CommonKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_connect`. Notable operations include `socket`, `setsockopt`, `connect`, `Lock`.
**CN:** 这一段定义了method `_connect`（属于 `CommonKVReceiver`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_connect`。 值得注意的操作包括 `socket`、`setsockopt`、`connect`、`Lock`。

### Lines 898-904: Method `_connect_to_bootstrap_server`
```python
    @classmethod
    def _connect_to_bootstrap_server(cls, bootstrap_info: dict):
        ip_address = bootstrap_info["rank_ip"]
        port = bootstrap_info["rank_port"]
        na = NetworkAddress(ip_address, port)
        sock, lock = cls._connect(na.to_tcp(), is_ipv6=na.is_ipv6)
        return sock, lock
```
**EN:** This block defines the method `_connect_to_bootstrap_server` on `CommonKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_connect_to_bootstrap_server`. Notable operations include `NetworkAddress`, `_connect`, `to_tcp`.
**CN:** 这一段定义了method `_connect_to_bootstrap_server`（属于 `CommonKVReceiver`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_connect_to_bootstrap_server`。 值得注意的操作包括 `NetworkAddress`、`_connect`、`to_tcp`。

### Lines 906-907: Method `_register_kv_args`
```python
    def _register_kv_args(self):
        pass
```
**EN:** This block defines the method `_register_kv_args` on `CommonKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_register_kv_args`.
**CN:** 这一段定义了method `_register_kv_args`（属于 `CommonKVReceiver`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_register_kv_args`。

### Lines 909-915: Method `send_metadata`
```python
    def send_metadata(
        self,
        kv_indices: npt.NDArray[np.int32],
        aux_index: Optional[int] = None,
        state_indices: Optional[List[int]] = None,
    ):
        raise NotImplementedError
```
**EN:** This block defines the method `send_metadata` on `CommonKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `send_metadata`.
**CN:** 这一段定义了method `send_metadata`（属于 `CommonKVReceiver`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `send_metadata`。

### Lines 917-918: Method `failure_exception`
```python
    def failure_exception(self):
        raise Exception("Fake KVReceiver Exception")
```
**EN:** This block defines the method `failure_exception` on `CommonKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `failure_exception`. Notable operations include `Exception`.
**CN:** 这一段定义了method `failure_exception`（属于 `CommonKVReceiver`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `failure_exception`。 值得注意的操作包括 `Exception`。

### Lines 920-923: Method `clear`
```python
    def clear(self) -> None:
        self.kv_mgr.request_status.pop(self.bootstrap_room, None)
        self.kv_mgr.required_prefill_response_num_table.pop(self.bootstrap_room, None)
        self.kv_mgr.prefill_response_tracker.pop(self.bootstrap_room, None)
```
**EN:** This block defines the method `clear` on `CommonKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `clear`. Notable operations include `pop`.
**CN:** 这一段定义了method `clear`（属于 `CommonKVReceiver`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `clear`。 值得注意的操作包括 `pop`。

### Lines 925-931: Method `abort`
```python
    def abort(self):
        self.kv_mgr.record_failure(
            self.bootstrap_room,
            "Aborted by AbortReq.",
        )
        self.kv_mgr.update_status(self.bootstrap_room, KVPoll.Failed)
        self.conclude_state = KVPoll.Failed
```
**EN:** This block defines the method `abort` on `CommonKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `abort`. Notable operations include `record_failure`, `update_status`.
**CN:** 这一段定义了method `abort`（属于 `CommonKVReceiver`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `abort`。 值得注意的操作包括 `record_failure`、`update_status`。

### Lines 934-934: Class `CommonKVBootstrapServer` declaration
```python
class CommonKVBootstrapServer(BaseKVBootstrapServer):
```
**EN:** This block declares the class `CommonKVBootstrapServer` and establishes its responsibility inside common backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `CommonKVBootstrapServer`.
**CN:** 这一段声明了类 `CommonKVBootstrapServer`，并说明它在common 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `CommonKVBootstrapServer`。

### Lines 935-937: Method `__init__` signature and setup
```python
    def __init__(self, host: str, port: int):
        self.host = host
        self.port = port
```
**EN:** This block defines the method `__init__` on `CommonKVBootstrapServer`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `CommonKVBootstrapServer`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 938-960: Method `__init__` logic (part 1)
```python
        self.app = web.Application()
        self.store = dict()
        self.lock = asyncio.Lock()
        self._setup_routes()
        self.pp_size = None
        self.attn_tp_size = None
        self.attn_cp_size = None
        self.dp_size = None
        self.page_size = None
        self.kv_cache_dtype: Optional[str] = None
        self.follow_bootstrap_room: Optional[bool] = None
        self.prefill_port_table: Dict[
            int, Dict[int, Dict[int, Dict[int, PrefillRankInfo]]]
        ] = {}
        self.room_to_dp_rank: Dict[int, Dict[str, Union[int, float]]] = {}
        self._registered_count = 0
        self.entry_cleanup_interval = (
            envs.SGLANG_DISAGGREGATION_BOOTSTRAP_ENTRY_CLEANUP_INTERVAL.get()
        )

        # Start bootstrap server
        self.thread = threading.Thread(target=self._run_server, daemon=True)
        self.run()
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `Application`, `Lock`, `_setup_routes`, `get`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `Application`、`Lock`、`_setup_routes`、`get`。

### Lines 962-963: Method `run`
```python
    def run(self):
        self.thread.start()
```
**EN:** This block defines the method `run` on `CommonKVBootstrapServer`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `run`. Notable operations include `start`.
**CN:** 这一段定义了method `run`（属于 `CommonKVBootstrapServer`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `run`。 值得注意的操作包括 `start`。

### Lines 965-977: Method `_is_ready`
```python
    def _is_ready(self) -> bool:
        if (
            self.attn_tp_size is None
            or self.attn_cp_size is None
            or self.pp_size is None
            or self.dp_size is None
        ):
            return False
        expected = self.dp_size * self.attn_cp_size * self.attn_tp_size * self.pp_size
        logger.debug(
            f"Expected {expected} prefill servers to be registered, {self._registered_count} registered so far"
        )
        return self._registered_count >= expected
```
**EN:** This block defines the method `_is_ready` on `CommonKVBootstrapServer`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_is_ready`. Notable operations include `debug`.
**CN:** 这一段定义了method `_is_ready`（属于 `CommonKVBootstrapServer`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_is_ready`。 值得注意的操作包括 `debug`。

### Lines 979-983: Method `_setup_routes`
```python
    def _setup_routes(self):
        self.app.router.add_route("*", "/route", self._handle_route)
        self.app.router.add_post("/register_dp_rank", self._handle_register_dp_rank)
        self.app.router.add_post("/query_dp_ranks", self._handle_query_dp_ranks)
        self.app.router.add_get("/health", self._handle_health_check)
```
**EN:** This block defines the method `_setup_routes` on `CommonKVBootstrapServer`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_setup_routes`. Notable operations include `add_route`, `add_post`, `add_get`.
**CN:** 这一段定义了method `_setup_routes`（属于 `CommonKVBootstrapServer`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_setup_routes`。 值得注意的操作包括 `add_route`、`add_post`、`add_get`。

### Lines 985-986: Method `_handle_health_check`
```python
    async def _handle_health_check(self, request):
        return web.Response(text="OK", status=200)
```
**EN:** This block defines the method `_handle_health_check` on `CommonKVBootstrapServer`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_handle_health_check`. Notable operations include `Response`.
**CN:** 这一段定义了method `_handle_health_check`（属于 `CommonKVBootstrapServer`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_handle_health_check`。 值得注意的操作包括 `Response`。

### Lines 988-997: Method `_handle_route`
```python
    async def _handle_route(self, request: web.Request):
        method = request.method
        if method == "PUT":
            return await self._handle_route_put(request)
        elif method == "GET":
            return await self._handle_route_get(request)
        else:
            return web.Response(
                text="Method not allowed", status=405, content_type="application/json"
            )
```
**EN:** This block defines the method `_handle_route` on `CommonKVBootstrapServer`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_handle_route`. Notable operations include `_handle_route_put`, `_handle_route_get`, `Response`.
**CN:** 这一段定义了method `_handle_route`（属于 `CommonKVBootstrapServer`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_handle_route`。 值得注意的操作包括 `_handle_route_put`、`_handle_route_get`、`Response`。

### Lines 999-1001: Method `_handle_route_put` signature and setup
```python
    async def _handle_route_put(self, request: web.Request):
        data = await request.json()
        attn_tp_size = data["attn_tp_size"]
```
**EN:** This block defines the method `_handle_route_put` on `CommonKVBootstrapServer`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_handle_route_put`. Notable operations include `json`.
**CN:** 这一段定义了method `_handle_route_put`（属于 `CommonKVBootstrapServer`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_handle_route_put`。 值得注意的操作包括 `json`。

### Lines 1002-1023: Method `_handle_route_put` logic (part 1)
```python
        attn_tp_rank = data["attn_tp_rank"]
        attn_cp_size = data["attn_cp_size"]
        attn_cp_rank = data["attn_cp_rank"]
        attn_dp_size = data["attn_dp_size"]
        attn_dp_rank = data["attn_dp_rank"]
        pp_size = data["pp_size"]
        pp_rank = data["pp_rank"]
        system_dp_size = data["system_dp_size"]
        system_dp_rank = data["system_dp_rank"]
        rank_ip = data["rank_ip"]
        rank_port = int(data["rank_port"])
        page_size = int(data["page_size"])
        kv_cache_dtype = data["kv_cache_dtype"]

        if self.attn_tp_size is None:
            self.attn_tp_size = attn_tp_size

        if self.attn_cp_size is None:
            self.attn_cp_size = attn_cp_size

        if self.dp_size is None:
            self.dp_size = attn_dp_size if system_dp_size == 1 else system_dp_size
```
**EN:** This block continues `_handle_route_put` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow.
**CN:** 这一段延续了 `_handle_route_put` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。

### Lines 1024-1043: Method `_handle_route_put` logic (part 2)
```python

        if self.pp_size is None:
            self.pp_size = pp_size

        if self.page_size is None and page_size is not None:
            self.page_size = page_size

        if self.kv_cache_dtype is None and kv_cache_dtype is not None:
            self.kv_cache_dtype = kv_cache_dtype

        if self.follow_bootstrap_room is None:
            load_balance_method = data.get(
                "load_balance_method", "follow_bootstrap_room"
            )
            self.follow_bootstrap_room = load_balance_method == "follow_bootstrap_room"

        if system_dp_size == 1:
            dp_group = attn_dp_rank
        else:
            dp_group = system_dp_rank
```
**EN:** This block continues `_handle_route_put` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `get`.
**CN:** 这一段延续了 `_handle_route_put` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `get`。

### Lines 1044-1064: Method `_handle_route_put` logic (part 3)
```python

        # Add lock to make sure thread-safe
        async with self.lock:
            dp_group_table = self.prefill_port_table.setdefault(dp_group, {})
            cp_group_table = dp_group_table.setdefault(attn_cp_rank, {})
            tp_group_table = cp_group_table.setdefault(attn_tp_rank, {})

            tp_group_table[pp_rank] = PrefillRankInfo(
                rank_ip=rank_ip,
                rank_port=rank_port,
            )

            self._registered_count += 1

        expected = self.dp_size * self.attn_cp_size * self.attn_tp_size * self.pp_size
        logger.debug(
            f"Register prefill bootstrap: DP{dp_group} CP{attn_cp_rank} TP{attn_tp_rank} PP{pp_rank} with rank_ip: {rank_ip} and rank_port: {rank_port}"
            f" ({self._registered_count}/{expected} registered)"
        )

        return web.Response(text="OK", status=200)
```
**EN:** This block continues `_handle_route_put` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `setdefault`, `PrefillRankInfo`, `debug`, `Response`.
**CN:** 这一段延续了 `_handle_route_put` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `setdefault`、`PrefillRankInfo`、`debug`、`Response`。

### Lines 1066-1068: Method `_handle_route_get` signature and setup
```python
    async def _handle_route_get(self, request: web.Request):
        prefill_dp_rank = request.query.get("prefill_dp_rank")
        prefill_cp_rank = request.query.get("prefill_cp_rank")
```
**EN:** This block defines the method `_handle_route_get` on `CommonKVBootstrapServer`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_handle_route_get`. Notable operations include `get`.
**CN:** 这一段定义了method `_handle_route_get`（属于 `CommonKVBootstrapServer`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_handle_route_get`。 值得注意的操作包括 `get`。

### Lines 1069-1092: Method `_handle_route_get` logic (part 1)
```python
        target_tp_rank = request.query.get("target_tp_rank")
        target_pp_rank = request.query.get("target_pp_rank")
        if (
            not prefill_dp_rank
            or not prefill_cp_rank
            or not target_tp_rank
            or not target_pp_rank
        ):
            return web.Response(text="Missing inputs for bootstrap server.", status=400)

        if (
            int(prefill_dp_rank) == -1
            and int(prefill_cp_rank) == -1
            and int(target_tp_rank) == -1
            and int(target_pp_rank) == -1
        ):
            if not self._is_ready():
                return web.Response(
                    text=f"Prefill server not fully registered yet"
                    f" ({self._registered_count} workers registered).",
                    status=503,
                )
            info = PrefillServerInfo(
                attn_tp_size=self.attn_tp_size,
```
**EN:** This block continues `_handle_route_get` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `get`, `Response`, `_is_ready`, `PrefillServerInfo`.
**CN:** 这一段延续了 `_handle_route_get` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `get`、`Response`、`_is_ready`、`PrefillServerInfo`。

### Lines 1093-1111: Method `_handle_route_get` logic (part 2)
```python
                attn_cp_size=self.attn_cp_size,
                dp_size=self.dp_size,
                pp_size=self.pp_size,
                page_size=self.page_size,
                kv_cache_dtype=self.kv_cache_dtype,
                follow_bootstrap_room=(
                    self.follow_bootstrap_room
                    if self.follow_bootstrap_room is not None
                    else True
                ),
            )
            return web.json_response(dataclasses.asdict(info), status=200)

        if not self._is_ready():
            return web.Response(
                text=f"Prefill server not fully registered yet"
                f" ({self._registered_count} workers registered).",
                status=503,
            )
```
**EN:** This block continues `_handle_route_get` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `json_response`, `asdict`, `_is_ready`, `Response`.
**CN:** 这一段延续了 `_handle_route_get` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `json_response`、`asdict`、`_is_ready`、`Response`。

### Lines 1112-1126: Method `_handle_route_get` logic (part 3)
```python

        # Find corresponding prefill info
        try:
            async with self.lock:
                bootstrap_info = self.prefill_port_table[int(prefill_dp_rank)][
                    int(prefill_cp_rank)
                ][int(target_tp_rank)][int(target_pp_rank)]
        except KeyError:
            return web.Response(
                text=f"Bootstrap info not found for dp_rank={prefill_dp_rank} cp_rank={prefill_cp_rank} "
                f"tp_rank={target_tp_rank} pp_rank={target_pp_rank}",
                status=404,
            )

        return web.json_response(dataclasses.asdict(bootstrap_info), status=200)
```
**EN:** This block continues `_handle_route_get` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `Response`, `json_response`, `asdict`.
**CN:** 这一段延续了 `_handle_route_get` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `Response`、`json_response`、`asdict`。

### Lines 1128-1138: Method `_handle_register_dp_rank`
```python
    async def _handle_register_dp_rank(self, request: web.Request):
        data = await request.json()
        bootstrap_room = int(data["bootstrap_room"])
        dp_rank = int(data["dp_rank"])
        async with self.lock:
            self.room_to_dp_rank[bootstrap_room] = {
                "dp_rank": dp_rank,
                "timestamp": time.time(),
            }
        logger.debug(f"Registered dp_rank={dp_rank} for {bootstrap_room=}")
        return web.Response(text="OK", status=200)
```
**EN:** This block defines the method `_handle_register_dp_rank` on `CommonKVBootstrapServer`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_handle_register_dp_rank`. Notable operations include `json`, `time`, `debug`, `Response`.
**CN:** 这一段定义了method `_handle_register_dp_rank`（属于 `CommonKVBootstrapServer`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_handle_register_dp_rank`。 值得注意的操作包括 `json`、`time`、`debug`、`Response`。

### Lines 1140-1149: Method `_handle_query_dp_ranks`
```python
    async def _handle_query_dp_ranks(self, request: web.Request):
        data = await request.json()
        bootstrap_rooms = data["bootstrap_rooms"]
        result = {}
        async with self.lock:
            for room in bootstrap_rooms:
                room_int = int(room)
                if room_int in self.room_to_dp_rank:
                    result[str(room_int)] = self.room_to_dp_rank[room_int]["dp_rank"]
        return web.json_response(result, status=200)
```
**EN:** This block defines the method `_handle_query_dp_ranks` on `CommonKVBootstrapServer`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_handle_query_dp_ranks`. Notable operations include `json`, `json_response`.
**CN:** 这一段定义了method `_handle_query_dp_ranks`（属于 `CommonKVBootstrapServer`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_handle_query_dp_ranks`。 值得注意的操作包括 `json`、`json_response`。

### Lines 1151-1167: Method `_cleanup_expired_entries`
```python
    async def _cleanup_expired_entries(self):
        """Remove entries older than cleanup interval from room_to_dp_rank."""
        while True:
            await asyncio.sleep(self.entry_cleanup_interval)
            current_time = time.time()
            async with self.lock:
                expired_keys = [
                    key
                    for key, value in self.room_to_dp_rank.items()
                    if current_time - value["timestamp"] > self.entry_cleanup_interval
                ]
                for key in expired_keys:
                    del self.room_to_dp_rank[key]
            if expired_keys:
                logger.debug(
                    f"Cleaned up {len(expired_keys)} expired entries from room_to_dp_rank"
                )
```
**EN:** This block defines the method `_cleanup_expired_entries` on `CommonKVBootstrapServer`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_cleanup_expired_entries`. Notable operations include `sleep`, `time`, `items`, `debug`.
**CN:** 这一段定义了method `_cleanup_expired_entries`（属于 `CommonKVBootstrapServer`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_cleanup_expired_entries`。 值得注意的操作包括 `sleep`、`time`、`items`、`debug`。

### Lines 1169-1171: Method `_run_server` signature and setup
```python
    def _run_server(self):
        try:
            # Event Loop
```
**EN:** This block defines the method `_run_server` on `CommonKVBootstrapServer`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `_run_server`.
**CN:** 这一段定义了method `_run_server`（属于 `CommonKVBootstrapServer`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `_run_server`。

### Lines 1172-1195: Method `_run_server` logic (part 1)
```python
            self._loop = asyncio.new_event_loop()
            asyncio.set_event_loop(self._loop)

            self._loop.create_task(self._cleanup_expired_entries())

            access_log = None
            if logging.getLogger(__name__).getEffectiveLevel() <= logging.DEBUG:
                access_log = self.app.logger

            self._runner = web.AppRunner(self.app, access_log=access_log)
            self._loop.run_until_complete(self._runner.setup())

            site = web.TCPSite(self._runner, host=self.host, port=self.port)
            self._loop.run_until_complete(site.start())
            logger.info(
                f"CommonKVBootstrapServer started successfully on {self.host}:{self.port}"
            )
            self._loop.run_forever()
        except Exception as e:
            logger.error(f"Server error: {str(e)}", exc_info=True)
        finally:
            # Cleanup
            self._loop.run_until_complete(self._runner.cleanup())
            self._loop.close()
```
**EN:** This block continues `_run_server` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding common backend connection and KV transfer management workflow. Notable operations include `new_event_loop`, `set_event_loop`, `create_task`, `_cleanup_expired_entries`.
**CN:** 这一段延续了 `_run_server` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的common 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `new_event_loop`、`set_event_loop`、`create_task`、`_cleanup_expired_entries`。

### Lines 1197-1205: Method `close`
```python
    def close(self):
        """Shutdown"""
        if self._loop is not None and self._loop.is_running():
            self._loop.call_soon_threadsafe(self._loop.stop)
            logger.info("Stopping server loop...")

        if self.thread.is_alive():
            self.thread.join(timeout=2)
            logger.info("Server thread stopped")
```
**EN:** This block defines the method `close` on `CommonKVBootstrapServer`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `close`. Notable operations include `is_running`, `call_soon_threadsafe`, `info`, `is_alive`.
**CN:** 这一段定义了method `close`（属于 `CommonKVBootstrapServer`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `close`。 值得注意的操作包括 `is_running`、`call_soon_threadsafe`、`info`、`is_alive`。

### Lines 1207-1207: Method `poll`
```python
    def poll(self) -> KVPoll: ...
```
**EN:** This block defines the method `poll` on `CommonKVBootstrapServer`. It introduces the parameters, setup steps, and the main entry point for this piece of common backend connection and KV transfer management. Definitions introduced here include `poll`.
**CN:** 这一段定义了method `poll`（属于 `CommonKVBootstrapServer`），介绍了参数、初始化步骤，以及这部分common 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `poll`。

## Key Concepts / 关键概念
- `PrefillServerInfo`: Class that encapsulates prefill server info behavior in this module. / `PrefillServerInfo`：封装与“预填充服务端info”相关行为的类。
- `PrefillRankInfo`: Class that encapsulates prefill rank info behavior in this module. / `PrefillRankInfo`：封装与“预填充rankinfo”相关行为的类。
- `CommonKVManager`: Class that encapsulates common kvmanager behavior in this module. / `CommonKVManager`：封装与“通用kvmanager”相关行为的类。
- `CommonKVSender`: Class that encapsulates common kvsender behavior in this module. / `CommonKVSender`：封装与“通用kvsender”相关行为的类。
- `CommonKVReceiver`: Class that encapsulates common kvreceiver behavior in this module. / `CommonKVReceiver`：封装与“通用kvreceiver”相关行为的类。
- `CommonKVBootstrapServer`: Class that encapsulates common kvbootstrap server behavior in this module. / `CommonKVBootstrapServer`：封装与“通用kvbootstrap服务端”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `asyncio`, `dataclasses`, `logging`, `threading`, `time`, `collections`, `functools`, `typing`
- **External packages / 外部依赖**: `numpy`, `requests`, `torch`, `zmq`, `aiohttp`
- **Internal modules / 内部模块**: `sglang.srt.disaggregation.base.conn`, `sglang.srt.disaggregation.utils`, `sglang.srt.distributed`, `sglang.srt.environ`, `sglang.srt.layers.dp_attention`, `sglang.srt.server_args`, `sglang.srt.utils.network`
