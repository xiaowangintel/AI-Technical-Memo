# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/disaggregation/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file provides shared enums, constants, and helper routines used across the disaggregation stack. It centralizes configuration-sensitive logic so multiple components can reuse it. / 该文件提供解耦栈共用的枚举、常量与辅助函数，把依赖配置的逻辑集中起来供多个组件复用。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: Imports and module setup
```python
from __future__ import annotations

import os
import random
from collections import deque
from contextlib import nullcontext
from enum import Enum
from typing import TYPE_CHECKING, List, Literal, Optional, Tuple, Type, overload

import numpy as np
import torch
import torch.distributed as dist

from sglang.srt.environ import envs
from sglang.srt.utils import is_npu

if TYPE_CHECKING:
    from sglang.srt.disaggregation.base.conn import KVArgs, StateType
    from sglang.srt.disaggregation.common.conn import (
        CommonKVBootstrapServer,
        CommonKVManager,
        CommonKVReceiver,
        CommonKVSender,
    )
    from sglang.srt.managers.schedule_batch import Req

#########################
# Constants & Enums
#########################
FAKE_BOOTSTRAP_HOST = "2.2.2.2"


```
**EN:** This block gathers the imports and module-level setup for shared disaggregation helpers and enums. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`.
**CN:** 这一段汇集了与共享解耦部署辅助函数与枚举相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`。

### Lines 33-33: Class `DisaggregationMode` declaration
```python
class DisaggregationMode(Enum):
```
**EN:** This block declares the class `DisaggregationMode` and establishes its responsibility inside shared disaggregation helpers and enums. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `DisaggregationMode`.
**CN:** 这一段声明了类 `DisaggregationMode`，并说明它在共享解耦部署辅助函数与枚举中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `DisaggregationMode`。

### Lines 34-37: Supporting state inside `DisaggregationMode`
```python
    NULL = "null"
    PREFILL = "prefill"
    DECODE = "decode"

```
**EN:** This block adds supporting state or helper logic inside `DisaggregationMode`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `DisaggregationMode` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 38-44: Method `to_engine_type`
```python
    @staticmethod
    def to_engine_type(mode: str) -> str:
        if mode == DisaggregationMode.PREFILL.value:
            return "prefill"
        elif mode == DisaggregationMode.DECODE.value:
            return "decode"
        return "unified"
```
**EN:** This block defines the method `to_engine_type` on `DisaggregationMode`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `to_engine_type`.
**CN:** 这一段定义了method `to_engine_type`（属于 `DisaggregationMode`），介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `to_engine_type`。

### Lines 45-54: Module-level constants and helper logic
```python


#########################
# Synchronization
#########################

# env var for testing failure, convert to float explicitly
FAILURE_PROB = float(os.getenv("DISAGGREGATION_TEST_FAILURE_PROB", 0))


```
**EN:** This block contains module-level constants, helpers, or documentation for shared disaggregation helpers and enums. It prepares shared state that later classes and functions build on. Notable operations include `getenv`.
**CN:** 这一段包含与共享解耦部署辅助函数与枚举相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `getenv`。

### Lines 55-68: Function `poll_and_all_reduce`
```python
def poll_and_all_reduce(pollers, gloo_group: dist.ProcessGroup):
    # at a certain prob, the poll is failed to simulate failure
    if FAILURE_PROB > 0:
        from sglang.srt.disaggregation.base import KVPoll

        polls = [
            int(KVPoll.Failed) if random.random() < FAILURE_PROB else int(poller.poll())
            for poller in pollers
        ]
    else:
        polls = [int(poller.poll()) for poller in pollers]
    tensor_to_reduce = torch.tensor(polls, dtype=torch.uint8, device="cpu")
    dist.all_reduce(tensor_to_reduce, op=dist.ReduceOp.MIN, group=gloo_group)
    return tensor_to_reduce.tolist()
```
**EN:** This block defines the function `poll_and_all_reduce`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `poll_and_all_reduce`. Notable operations include `random`, `poll`, `tensor`, `all_reduce`.
**CN:** 这一段定义了function `poll_and_all_reduce`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `poll_and_all_reduce`。 值得注意的操作包括 `random`、`poll`、`tensor`、`all_reduce`。

### Lines 71-88: Function `poll_and_all_reduce_attn_cp_tp_group`
```python
def poll_and_all_reduce_attn_cp_tp_group(
    pollers,
    attn_cp_cpu_group: dist.ProcessGroup,
    attn_tp_cpu_group: dist.ProcessGroup,
):
    # First sync across attn-tp ranks so all TP participants for a given (dp, cp)
    # shard observe the same status transitions.
    polls = poll_and_all_reduce(pollers, attn_tp_cpu_group)

    # Then sync across attn-cp ranks, so all TPxCP participants in one DP shard
    # converge to the same global status.
    tensor_to_reduce = torch.tensor(polls, dtype=torch.uint8, device="cpu")
    dist.all_reduce(
        tensor_to_reduce,
        op=dist.ReduceOp.MIN,
        group=attn_cp_cpu_group,
    )
    return tensor_to_reduce.tolist()
```
**EN:** This block defines the function `poll_and_all_reduce_attn_cp_tp_group`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `poll_and_all_reduce_attn_cp_tp_group`. Notable operations include `given`, `poll_and_all_reduce`, `tensor`, `all_reduce`.
**CN:** 这一段定义了function `poll_and_all_reduce_attn_cp_tp_group`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `poll_and_all_reduce_attn_cp_tp_group`。 值得注意的操作包括 `given`、`poll_and_all_reduce`、`tensor`、`all_reduce`。

### Lines 91-112: Function `poll_and_all_reduce_with_staging`
```python
def poll_and_all_reduce_with_staging(
    decode_reqs, staging_handler, gloo_group: dist.ProcessGroup
):
    """Staging-aware polling: advance scatter, demote incomplete transfers, all_reduce."""
    from sglang.srt.disaggregation.base import KVPoll

    for decode_req in decode_reqs:
        if decode_req.kv_receiver.require_staging and not staging_handler.is_done(
            decode_req
        ):
            staging_handler.advance_scatter(decode_req)

    raw_polls = [int(dr.kv_receiver.poll()) for dr in decode_reqs]
    for i, decode_req in enumerate(decode_reqs):
        if raw_polls[i] == int(KVPoll.Success):
            if decode_req.kv_receiver.require_staging and not staging_handler.is_done(
                decode_req
            ):
                raw_polls[i] = int(KVPoll.Transferring)
    poll_tensor = torch.tensor(raw_polls, dtype=torch.uint8, device="cpu")
    dist.all_reduce(poll_tensor, op=dist.ReduceOp.MIN, group=gloo_group)
    return poll_tensor.tolist()
```
**EN:** This block defines the function `poll_and_all_reduce_with_staging`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `poll_and_all_reduce_with_staging`. Notable operations include `is_done`, `advance_scatter`, `poll`, `tensor`.
**CN:** 这一段定义了function `poll_and_all_reduce_with_staging`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `poll_and_all_reduce_with_staging`。 值得注意的操作包括 `is_done`、`advance_scatter`、`poll`、`tensor`。

### Lines 113-119: Module-level constants and helper logic
```python


#########################
# Metadata Buffers
#########################


```
**EN:** This block contains module-level constants, helpers, or documentation for shared disaggregation helpers and enums. It prepares shared state that later classes and functions build on.
**CN:** 这一段包含与共享解耦部署辅助函数与枚举相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。

### Lines 120-120: Class `ReqToMetadataIdxAllocator` declaration
```python
class ReqToMetadataIdxAllocator:
```
**EN:** This block declares the class `ReqToMetadataIdxAllocator` and establishes its responsibility inside shared disaggregation helpers and enums. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `ReqToMetadataIdxAllocator`.
**CN:** 这一段声明了类 `ReqToMetadataIdxAllocator`，并说明它在共享解耦部署辅助函数与枚举中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `ReqToMetadataIdxAllocator`。

### Lines 121-122: Supporting state inside `ReqToMetadataIdxAllocator`
```python
    """A memory pool that maps a request to its first output token location."""

```
**EN:** This block adds supporting state or helper logic inside `ReqToMetadataIdxAllocator`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `ReqToMetadataIdxAllocator` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 123-128: Method `__init__`
```python
    def __init__(
        self,
        size: int,
    ):
        self.size = size
        self.free_slots = deque(list(range(size)))
```
**EN:** This block defines the method `__init__` on `ReqToMetadataIdxAllocator`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `__init__`. Notable operations include `deque`.
**CN:** 这一段定义了method `__init__`（属于 `ReqToMetadataIdxAllocator`），介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `deque`。

### Lines 130-131: Method `available_size`
```python
    def available_size(self):
        return len(self.free_slots)
```
**EN:** This block defines the method `available_size` on `ReqToMetadataIdxAllocator`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `available_size`.
**CN:** 这一段定义了method `available_size`（属于 `ReqToMetadataIdxAllocator`），介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `available_size`。

### Lines 133-137: Method `alloc`
```python
    def alloc(self) -> Optional[int]:
        if len(self.free_slots) == 0:
            return None

        return self.free_slots.popleft()
```
**EN:** This block defines the method `alloc` on `ReqToMetadataIdxAllocator`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `alloc`. Notable operations include `popleft`.
**CN:** 这一段定义了method `alloc`（属于 `ReqToMetadataIdxAllocator`），介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `alloc`。 值得注意的操作包括 `popleft`。

### Lines 139-140: Method `free`
```python
    def free(self, free_index: int):
        self.free_slots.append(free_index)
```
**EN:** This block defines the method `free` on `ReqToMetadataIdxAllocator`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `free`. Notable operations include `append`.
**CN:** 这一段定义了method `free`（属于 `ReqToMetadataIdxAllocator`），介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `free`。 值得注意的操作包括 `append`。

### Lines 143-143: Class `MetadataBuffers` declaration
```python
class MetadataBuffers:
```
**EN:** This block declares the class `MetadataBuffers` and establishes its responsibility inside shared disaggregation helpers and enums. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `MetadataBuffers`.
**CN:** 这一段声明了类 `MetadataBuffers`，并说明它在共享解耦部署辅助函数与枚举中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `MetadataBuffers`。

### Lines 144-146: Method `__init__` signature and setup
```python
    def __init__(
        self,
        size: int,
```
**EN:** This block defines the method `__init__` on `MetadataBuffers`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `MetadataBuffers`），介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 147-170: Method `__init__` logic (part 1)
```python
        hidden_size: int,
        hidden_states_dtype: torch.dtype,
        max_top_logprobs_num: int = 128,
        custom_mem_pool: torch.cuda.MemPool = None,
    ):
        self.custom_mem_pool = custom_mem_pool
        bootstrap_room_dtype = torch.uint64
        device = "cpu"
        if is_npu():
            # For ascend backend, output tokens are placed in the NPU and will be transferred by D2D channel.
            device = "npu"
            # TODO: Fix me when npu backend supports torch.uint64
            bootstrap_room_dtype = torch.int64
        elif self.custom_mem_pool:
            # TODO(shangming): Fix me (use 'cuda') when nvlink_transport of Mooncake is bug-free
            device = "cpu"
        elif envs.SGLANG_MOONCAKE_CUSTOM_MEM_POOL.get() == "INTRA_NODE_NVLINK":
            device = "cpu"
        with (
            torch.cuda.use_mem_pool(self.custom_mem_pool)
            if self.custom_mem_pool
            else nullcontext()
        ):
            # TODO: abort top_logprobs_num > 128 in PD
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `is_npu`, `TODO`, `me`, `get`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `is_npu`、`TODO`、`me`、`get`。

### Lines 171-194: Method `__init__` logic (part 2)
```python

            # We transfer the metadata of first output token to decode
            # The minimal size for RDMA is 64Bytes, so we pad it to > 64Bytes
            self.output_ids = torch.zeros((size, 16), dtype=torch.int32, device=device)
            self.cached_tokens = torch.zeros(
                (size, 16), dtype=torch.int32, device=device
            )
            self.output_token_logprobs_val = torch.zeros(
                (size, 16), dtype=torch.float32, device=device
            )
            self.output_token_logprobs_idx = torch.zeros(
                (size, 16), dtype=torch.int32, device=device
            )
            self.output_top_logprobs_val = torch.zeros(
                (size, max_top_logprobs_num), dtype=torch.float32, device=device
            )
            self.output_top_logprobs_idx = torch.zeros(
                (size, max_top_logprobs_num), dtype=torch.int32, device=device
            )
            # For PD + spec decode
            self.output_topk_p = torch.zeros(
                (size, 16), dtype=torch.float32, device=device
            )
            self.output_topk_index = torch.zeros(
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `zeros`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `zeros`。

### Lines 195-203: Method `__init__` logic (part 3)
```python
                (size, 16), dtype=torch.int64, device=device
            )
            self.output_hidden_states = torch.zeros(
                (size, hidden_size), dtype=hidden_states_dtype, device=device
            )
            # Request validation: store bootstrap_room to detect metadata corruption
            self.bootstrap_room = torch.zeros(
                (size, 8), dtype=bootstrap_room_dtype, device=device
            )
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `zeros`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `zeros`。

### Lines 205-207: Method `get_buf_infos` signature and setup
```python
    def get_buf_infos(self):
        ptrs = [
            self.output_ids.data_ptr(),
```
**EN:** This block defines the method `get_buf_infos` on `MetadataBuffers`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `get_buf_infos`. Notable operations include `data_ptr`.
**CN:** 这一段定义了method `get_buf_infos`（属于 `MetadataBuffers`），介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `get_buf_infos`。 值得注意的操作包括 `data_ptr`。

### Lines 208-231: Method `get_buf_infos` logic (part 1)
```python
            self.cached_tokens.data_ptr(),
            self.output_token_logprobs_val.data_ptr(),
            self.output_token_logprobs_idx.data_ptr(),
            self.output_top_logprobs_val.data_ptr(),
            self.output_top_logprobs_idx.data_ptr(),
            self.output_topk_p.data_ptr(),
            self.output_topk_index.data_ptr(),
            self.output_hidden_states.data_ptr(),
            self.bootstrap_room.data_ptr(),
        ]
        data_lens = [
            self.output_ids.nbytes,
            self.cached_tokens.nbytes,
            self.output_token_logprobs_val.nbytes,
            self.output_token_logprobs_idx.nbytes,
            self.output_top_logprobs_val.nbytes,
            self.output_top_logprobs_idx.nbytes,
            self.output_topk_p.nbytes,
            self.output_topk_index.nbytes,
            self.output_hidden_states.nbytes,
            self.bootstrap_room.nbytes,
        ]
        item_lens = [
            self.output_ids[0].nbytes,
```
**EN:** This block continues `get_buf_infos` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `data_ptr`.
**CN:** 这一段延续了 `get_buf_infos` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `data_ptr`。

### Lines 232-242: Method `get_buf_infos` logic (part 2)
```python
            self.cached_tokens[0].nbytes,
            self.output_token_logprobs_val[0].nbytes,
            self.output_token_logprobs_idx[0].nbytes,
            self.output_top_logprobs_val[0].nbytes,
            self.output_top_logprobs_idx[0].nbytes,
            self.output_topk_p[0].nbytes,
            self.output_topk_index[0].nbytes,
            self.output_hidden_states[0].nbytes,
            self.bootstrap_room[0].nbytes,
        ]
        return ptrs, data_lens, item_lens
```
**EN:** This block continues `get_buf_infos` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow.
**CN:** 这一段延续了 `get_buf_infos` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。

### Lines 244-256: Method `get_buf`
```python
    def get_buf(self, idx: int):
        return (
            self.output_ids[idx].clone(),
            self.cached_tokens[idx].clone(),
            self.output_token_logprobs_val[idx].clone(),
            self.output_token_logprobs_idx[idx].clone(),
            self.output_top_logprobs_val[idx].clone(),
            self.output_top_logprobs_idx[idx].clone(),
            self.output_topk_p[idx].clone(),
            self.output_topk_index[idx].clone(),
            self.output_hidden_states[idx].clone(),
            self.bootstrap_room[idx].clone(),
        )
```
**EN:** This block defines the method `get_buf` on `MetadataBuffers`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `get_buf`. Notable operations include `clone`.
**CN:** 这一段定义了method `get_buf`（属于 `MetadataBuffers`），介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `get_buf`。 值得注意的操作包括 `clone`。

### Lines 258-260: Method `set_buf` signature and setup
```python
    def set_buf(self, req: Req):

        self.output_ids[req.metadata_buffer_index][0] = req.output_ids[0]
```
**EN:** This block defines the method `set_buf` on `MetadataBuffers`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `set_buf`.
**CN:** 这一段定义了method `set_buf`（属于 `MetadataBuffers`），介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `set_buf`。

### Lines 261-290: Method `set_buf` logic (part 1)
```python
        self.cached_tokens[req.metadata_buffer_index][0] = req.cached_tokens
        self.cached_tokens[req.metadata_buffer_index][1] = req.cached_tokens_device
        self.cached_tokens[req.metadata_buffer_index][2] = req.cached_tokens_host
        self.cached_tokens[req.metadata_buffer_index][3] = req.cached_tokens_storage
        if req.return_logprob:
            if req.output_token_logprobs_val:  # not none or empty list
                self.output_token_logprobs_val[req.metadata_buffer_index][0] = (
                    req.output_token_logprobs_val[0]
                )
            if req.output_token_logprobs_idx:  # not none or empty list
                self.output_token_logprobs_idx[req.metadata_buffer_index][0] = (
                    req.output_token_logprobs_idx[0]
                )

            if req.output_top_logprobs_val:  # not none or empty list
                self.output_top_logprobs_val[req.metadata_buffer_index][
                    : len(req.output_top_logprobs_val[0])
                ] = torch.tensor(
                    req.output_top_logprobs_val[0], dtype=torch.float32, device="cpu"
                )
            if req.output_top_logprobs_idx:  # not none or empty list
                self.output_top_logprobs_idx[req.metadata_buffer_index][
                    : len(req.output_top_logprobs_idx[0])
                ] = torch.tensor(
                    req.output_top_logprobs_idx[0], dtype=torch.int32, device="cpu"
                )
        # For PD + spec decode
        if req.hidden_states_tensor is not None:
            # speculative_eagle_topk should not be greater than 16 currently
            topk = req.output_topk_p.size(0)
```
**EN:** This block continues `set_buf` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `tensor`, `size`.
**CN:** 这一段延续了 `set_buf` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `tensor`、`size`。

### Lines 291-304: Method `set_buf` logic (part 2)
```python

            self.output_topk_p[req.metadata_buffer_index, :topk].copy_(
                req.output_topk_p
            )
            self.output_topk_index[req.metadata_buffer_index, :topk].copy_(
                req.output_topk_index
            )
            self.output_hidden_states[req.metadata_buffer_index].copy_(
                req.hidden_states_tensor
            )
        # Store bootstrap_room for validation on decode side
        self.bootstrap_room[req.metadata_buffer_index, 0] = (
            req.bootstrap_room if req.bootstrap_room is not None else 0
        )
```
**EN:** This block continues `set_buf` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `copy_`.
**CN:** 这一段延续了 `set_buf` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `copy_`。

### Lines 305-311: Module-level constants and helper logic
```python


#########################
# Transfer Backend
#########################


```
**EN:** This block contains module-level constants, helpers, or documentation for shared disaggregation helpers and enums. It prepares shared state that later classes and functions build on.
**CN:** 这一段包含与共享解耦部署辅助函数与枚举相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。

### Lines 312-312: Class `TransferBackend` declaration
```python
class TransferBackend(Enum):
```
**EN:** This block declares the class `TransferBackend` and establishes its responsibility inside shared disaggregation helpers and enums. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `TransferBackend`.
**CN:** 这一段声明了类 `TransferBackend`，并说明它在共享解耦部署辅助函数与枚举中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `TransferBackend`。

### Lines 313-317: Supporting state inside `TransferBackend`
```python
    MOONCAKE = "mooncake"
    MORI = "mori"
    NIXL = "nixl"
    ASCEND = "ascend"
    FAKE = "fake"
```
**EN:** This block adds supporting state or helper logic inside `TransferBackend`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `TransferBackend` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 320-320: Class `KVClassType` declaration
```python
class KVClassType(Enum):
```
**EN:** This block declares the class `KVClassType` and establishes its responsibility inside shared disaggregation helpers and enums. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `KVClassType`.
**CN:** 这一段声明了类 `KVClassType`，并说明它在共享解耦部署辅助函数与枚举中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `KVClassType`。

### Lines 321-325: Supporting state inside `KVClassType`
```python
    KVARGS = "kvargs"
    MANAGER = "manager"
    SENDER = "sender"
    RECEIVER = "receiver"
    BOOTSTRAP_SERVER = "bootstrap_server"
```
**EN:** This block adds supporting state or helper logic inside `KVClassType`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `KVClassType` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 328-331: Function `get_kv_class`
```python
@overload
def get_kv_class(
    transfer_backend: TransferBackend, class_type: Literal[KVClassType.KVARGS]
) -> Type[KVArgs]: ...
```
**EN:** This block defines the function `get_kv_class`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `get_kv_class`.
**CN:** 这一段定义了function `get_kv_class`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `get_kv_class`。

### Lines 332-335: Function `get_kv_class`
```python
@overload
def get_kv_class(
    transfer_backend: TransferBackend, class_type: Literal[KVClassType.MANAGER]
) -> Type[CommonKVManager]: ...
```
**EN:** This block defines the function `get_kv_class`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `get_kv_class`.
**CN:** 这一段定义了function `get_kv_class`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `get_kv_class`。

### Lines 336-339: Function `get_kv_class`
```python
@overload
def get_kv_class(
    transfer_backend: TransferBackend, class_type: Literal[KVClassType.SENDER]
) -> Type[CommonKVSender]: ...
```
**EN:** This block defines the function `get_kv_class`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `get_kv_class`.
**CN:** 这一段定义了function `get_kv_class`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `get_kv_class`。

### Lines 340-343: Function `get_kv_class`
```python
@overload
def get_kv_class(
    transfer_backend: TransferBackend, class_type: Literal[KVClassType.RECEIVER]
) -> Type[CommonKVReceiver]: ...
```
**EN:** This block defines the function `get_kv_class`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `get_kv_class`.
**CN:** 这一段定义了function `get_kv_class`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `get_kv_class`。

### Lines 344-347: Function `get_kv_class`
```python
@overload
def get_kv_class(
    transfer_backend: TransferBackend, class_type: Literal[KVClassType.BOOTSTRAP_SERVER]
) -> Type[CommonKVBootstrapServer]: ...
```
**EN:** This block defines the function `get_kv_class`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `get_kv_class`.
**CN:** 这一段定义了function `get_kv_class`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `get_kv_class`。

### Lines 350-352: Function `get_kv_class` signature and setup
```python
def get_kv_class(
    transfer_backend: TransferBackend, class_type: KVClassType
) -> Optional[Type]:
```
**EN:** This block defines the function `get_kv_class`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `get_kv_class`.
**CN:** 这一段定义了function `get_kv_class`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `get_kv_class`。

### Lines 353-379: Function `get_kv_class` logic (part 1)
```python
    from sglang.srt.disaggregation.fake import FakeKVReceiver, FakeKVSender

    if transfer_backend == TransferBackend.MOONCAKE:
        from sglang.srt.disaggregation.base import KVArgs
        from sglang.srt.disaggregation.mooncake import (
            MooncakeKVBootstrapServer,
            MooncakeKVManager,
            MooncakeKVReceiver,
            MooncakeKVSender,
        )

        class_mapping = {
            KVClassType.KVARGS: KVArgs,
            KVClassType.MANAGER: MooncakeKVManager,
            KVClassType.SENDER: MooncakeKVSender,
            KVClassType.RECEIVER: (MooncakeKVReceiver),
            KVClassType.BOOTSTRAP_SERVER: MooncakeKVBootstrapServer,
        }
        return class_mapping.get(class_type)
    elif transfer_backend == TransferBackend.MORI:
        from sglang.srt.disaggregation.base import KVArgs
        from sglang.srt.disaggregation.mori import (
            MoriKVBootstrapServer,
            MoriKVManager,
            MoriKVReceiver,
            MoriKVSender,
        )
```
**EN:** This block continues `get_kv_class` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `import`, `get`.
**CN:** 这一段延续了 `get_kv_class` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `import`、`get`。

### Lines 380-413: Function `get_kv_class` logic (part 2)
```python

        class_mapping = {
            KVClassType.KVARGS: KVArgs,
            KVClassType.MANAGER: MoriKVManager,
            KVClassType.SENDER: MoriKVSender,
            KVClassType.RECEIVER: (MoriKVReceiver),
            KVClassType.BOOTSTRAP_SERVER: MoriKVBootstrapServer,
        }
        return class_mapping.get(class_type)
    elif transfer_backend == TransferBackend.ASCEND:
        from sglang.srt.disaggregation.ascend import (
            AscendKVBootstrapServer,
            AscendKVManager,
            AscendKVReceiver,
            AscendKVSender,
        )
        from sglang.srt.disaggregation.base import KVArgs

        class_mapping = {
            KVClassType.KVARGS: KVArgs,
            KVClassType.MANAGER: AscendKVManager,
            KVClassType.SENDER: AscendKVSender,
            KVClassType.RECEIVER: (AscendKVReceiver),
            KVClassType.BOOTSTRAP_SERVER: AscendKVBootstrapServer,
        }
        return class_mapping.get(class_type)
    elif transfer_backend == TransferBackend.NIXL:
        from sglang.srt.disaggregation.base import KVArgs
        from sglang.srt.disaggregation.nixl import (
            NixlKVBootstrapServer,
            NixlKVManager,
            NixlKVReceiver,
            NixlKVSender,
        )
```
**EN:** This block continues `get_kv_class` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `get`, `import`.
**CN:** 这一段延续了 `get_kv_class` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `get`、`import`。

### Lines 414-439: Function `get_kv_class` logic (part 3)
```python

        class_mapping = {
            KVClassType.KVARGS: KVArgs,
            KVClassType.MANAGER: NixlKVManager,
            KVClassType.SENDER: NixlKVSender,
            KVClassType.RECEIVER: (NixlKVReceiver),
            KVClassType.BOOTSTRAP_SERVER: NixlKVBootstrapServer,
        }
        return class_mapping.get(class_type)
    elif transfer_backend == TransferBackend.FAKE:
        from sglang.srt.disaggregation.base import KVArgs
        from sglang.srt.disaggregation.fake import (
            FakeKVManager,
            FakeKVReceiver,
            FakeKVSender,
        )

        class_mapping = {
            KVClassType.KVARGS: KVArgs,
            KVClassType.MANAGER: FakeKVManager,
            KVClassType.SENDER: FakeKVSender,
            KVClassType.RECEIVER: (FakeKVReceiver),
        }
        return class_mapping.get(class_type)

    raise ValueError(f"Unsupported transfer backend: {transfer_backend}")
```
**EN:** This block continues `get_kv_class` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `get`, `import`, `ValueError`.
**CN:** 这一段延续了 `get_kv_class` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `get`、`import`、`ValueError`。

### Lines 442-461: Function `page_indices_to_cp_rank_page_indices` signature and setup
```python
def page_indices_to_cp_rank_page_indices(
    page_indices: np.ndarray,
    total_pages: int,
    cp_rank: int,
    cp_size: int,
) -> np.ndarray:
    """
    Filter page_indices (which are *global* page ids in the KV pool) to those
    belonging to the given CP rank for this request.

    For a single request, its pages occupy a contiguous global range
    [first_page, first_page + total_pages). We first compute the local
    split [0, total_pages) across cp_size ranks, then shift that local
    range by first_page back into the global page id space and take
    the intersection with page_indices.

    Returns:
        Subset of page_indices that fall in this rank's global
        [start_page, end_page) slice for the given CP rank.
    """
```
**EN:** This block defines the function `page_indices_to_cp_rank_page_indices`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `page_indices_to_cp_rank_page_indices`. Notable operations include `page_indices`.
**CN:** 这一段定义了function `page_indices_to_cp_rank_page_indices`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `page_indices_to_cp_rank_page_indices`。 值得注意的操作包括 `page_indices`。

### Lines 462-485: Function `page_indices_to_cp_rank_page_indices` logic (part 1)
```python
    if cp_size <= 1:
        return page_indices

    if page_indices.size == 0:
        return np.asarray(page_indices)

    first_page = int(page_indices.min())
    base = total_pages // cp_size
    rem = total_pages % cp_size

    if rem == 0:
        local_start = cp_rank * base
        local_end = local_start + base
    else:
        local_start = cp_rank * base + min(cp_rank, rem)
        n_pages = base + (1 if cp_rank < rem else 0)
        local_end = local_start + n_pages

    # Map back to global page ids.
    start_page = first_page + local_start
    end_page = first_page + local_end

    mask = (page_indices >= start_page) & (page_indices < end_page)
    return np.asarray(page_indices)[mask]
```
**EN:** This block continues `page_indices_to_cp_rank_page_indices` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `asarray`.
**CN:** 这一段延续了 `page_indices_to_cp_rank_page_indices` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `asarray`。

### Lines 488-491: Function `filter_kv_indices_for_cp_rank` signature and setup
```python
def filter_kv_indices_for_cp_rank(
    kv_mgr: CommonKVManager, kv_indices: np.ndarray, index_slice: slice
) -> Tuple[np.ndarray, slice]:
    """Filters kv_indices and index_slice for the current CP rank."""
```
**EN:** This block defines the function `filter_kv_indices_for_cp_rank`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `filter_kv_indices_for_cp_rank`.
**CN:** 这一段定义了function `filter_kv_indices_for_cp_rank`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `filter_kv_indices_for_cp_rank`。

### Lines 492-513: Function `filter_kv_indices_for_cp_rank` logic (part 1)
```python
    total_pages = len(kv_indices)
    cp_rank = kv_mgr.attn_cp_rank
    cp_size = kv_mgr.attn_cp_size

    rank_page_indices = page_indices_to_cp_rank_page_indices(
        page_indices=kv_indices,
        total_pages=total_pages,
        cp_rank=cp_rank,
        cp_size=cp_size,
    )

    if rank_page_indices.size == 0:
        new_kv_indices = kv_indices[:0]
        new_index_slice = slice(index_slice.start, index_slice.start)
    else:
        mask = np.isin(kv_indices, rank_page_indices)
        if not mask.any():
            new_kv_indices = kv_indices[:0]
            new_index_slice = slice(index_slice.start, index_slice.start)
        else:
            first_pos = int(mask.argmax())
            last_pos = len(mask) - int(mask[::-1].argmax())
```
**EN:** This block continues `filter_kv_indices_for_cp_rank` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `page_indices_to_cp_rank_page_indices`, `slice`, `isin`, `argmax`.
**CN:** 这一段延续了 `filter_kv_indices_for_cp_rank` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `page_indices_to_cp_rank_page_indices`、`slice`、`isin`、`argmax`。

### Lines 514-520: Function `filter_kv_indices_for_cp_rank` logic (part 2)
```python

            new_kv_indices = kv_indices[first_pos:last_pos]
            new_index_slice = slice(
                index_slice.start + first_pos,
                index_slice.start + last_pos,
            )
    return new_kv_indices, new_index_slice
```
**EN:** This block continues `filter_kv_indices_for_cp_rank` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `slice`.
**CN:** 这一段延续了 `filter_kv_indices_for_cp_rank` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `slice`。

### Lines 521-527: Module-level constants and helper logic
```python


#########################
# Misc
#########################


```
**EN:** This block contains module-level constants, helpers, or documentation for shared disaggregation helpers and enums. It prepares shared state that later classes and functions build on.
**CN:** 这一段包含与共享解耦部署辅助函数与枚举相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。

### Lines 528-532: Function `is_mla_backend`
```python
def is_mla_backend(target_kv_pool) -> bool:
    from sglang.srt.mem_cache.deepseek_v4_memory_pool import DeepSeekV4TokenToKVPool
    from sglang.srt.mem_cache.memory_pool import MLATokenToKVPool

    return isinstance(target_kv_pool, (MLATokenToKVPool, DeepSeekV4TokenToKVPool))
```
**EN:** This block defines the function `is_mla_backend`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `is_mla_backend`.
**CN:** 这一段定义了function `is_mla_backend`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `is_mla_backend`。

### Lines 535-549: Function `append_state_component`
```python
def append_state_component(
    kv_args: KVArgs,
    state_type: StateType,
    data_ptrs: List[int],
    data_lens: List[int],
    item_lens: List[int],
    dim_per_tensor: Optional[List[int]] = None,
) -> None:
    """Append one state component. Caller orders state_types consistently
    on prefill and decode sides."""
    kv_args.state_types.append(state_type)
    kv_args.state_data_ptrs.append(data_ptrs)
    kv_args.state_data_lens.append(data_lens)
    kv_args.state_item_lens.append(item_lens)
    kv_args.state_dim_per_tensor.append(dim_per_tensor or [])
```
**EN:** This block defines the function `append_state_component`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `append_state_component`. Notable operations include `append`.
**CN:** 这一段定义了function `append_state_component`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `append_state_component`。 值得注意的操作包括 `append`。

### Lines 552-562: Function `setup_state_kv_args` signature and setup
```python
def setup_state_kv_args(
    kv_args: KVArgs,
    token_to_kv_pool,
    draft_token_to_kv_pool=None,
    total_kv_layers: int = None,
    req_to_token_pool=None,
) -> None:
    """Populate ``kv_args`` state-buffer fields from the given pool.
    Shared by prefill and decode bootstrap paths so the state_type dispatch
    lives in one place.
    """
```
**EN:** This block defines the function `setup_state_kv_args`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `setup_state_kv_args`.
**CN:** 这一段定义了function `setup_state_kv_args`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `setup_state_kv_args`。

### Lines 563-586: Function `setup_state_kv_args` logic (part 1)
```python
    from sglang.srt.disaggregation.base.conn import StateType
    from sglang.srt.hardware_backend.npu.memory_pool_npu import NPUMLATokenToKVPool
    from sglang.srt.mem_cache.base_swa_memory_pool import BaseSWAKVPool
    from sglang.srt.mem_cache.memory_pool import HybridLinearKVPool, NSATokenToKVPool

    kv_args.state_types = []
    kv_args.state_data_ptrs = []
    kv_args.state_data_lens = []
    kv_args.state_item_lens = []
    kv_args.state_dim_per_tensor = []

    if hasattr(token_to_kv_pool, "get_state_buf_infos"):
        data_ptrs, data_lens, item_lens = token_to_kv_pool.get_state_buf_infos()

        # DeepSeekV4TokenToKVPool inherits BaseSWAKVPool; its heterogeneous
        # state list is described per-entry via get_state_buf_infos.
        if isinstance(token_to_kv_pool, BaseSWAKVPool):
            append_state_component(
                kv_args, StateType.SWA, data_ptrs, data_lens, item_lens
            )
        elif isinstance(token_to_kv_pool, HybridLinearKVPool):
            dim = (
                token_to_kv_pool.get_state_dim_per_tensor()
                if hasattr(token_to_kv_pool, "get_state_dim_per_tensor")
```
**EN:** This block continues `setup_state_kv_args` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `get_state_buf_infos`, `append_state_component`, `get_state_dim_per_tensor`.
**CN:** 这一段延续了 `setup_state_kv_args` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `get_state_buf_infos`、`append_state_component`、`get_state_dim_per_tensor`。

### Lines 587-612: Function `setup_state_kv_args` logic (part 2)
```python
                else None
            )
            append_state_component(
                kv_args, StateType.MAMBA, data_ptrs, data_lens, item_lens, dim
            )
        elif isinstance(token_to_kv_pool, (NSATokenToKVPool, NPUMLATokenToKVPool)):
            if draft_token_to_kv_pool is not None and isinstance(
                draft_token_to_kv_pool, NSATokenToKVPool
            ):
                (
                    draft_data_ptrs,
                    draft_data_lens,
                    draft_item_lens,
                ) = draft_token_to_kv_pool.get_state_buf_infos()
                data_ptrs = data_ptrs + draft_data_ptrs
                data_lens = data_lens + draft_data_lens
                item_lens = item_lens + draft_item_lens
            if isinstance(token_to_kv_pool, NPUMLATokenToKVPool):
                kv_args.kv_buf_groups = (
                    len(kv_args.kv_data_ptrs) // token_to_kv_pool.layer_num
                )
                kv_args.total_kv_layers = total_kv_layers
            else:
                append_state_component(
                    kv_args, StateType.NSA, data_ptrs, data_lens, item_lens
                )
```
**EN:** This block continues `setup_state_kv_args` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `append_state_component`, `get_state_buf_infos`.
**CN:** 这一段延续了 `setup_state_kv_args` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `append_state_component`、`get_state_buf_infos`。

### Lines 613-628: Function `setup_state_kv_args` logic (part 3)
```python

    if (
        StateType.MAMBA not in kv_args.state_types
        and req_to_token_pool is not None
        and hasattr(req_to_token_pool, "get_state_buf_infos")
    ):
        data_ptrs, data_lens, item_lens = req_to_token_pool.get_state_buf_infos()
        if data_ptrs:
            dim = (
                req_to_token_pool.get_state_dim_per_tensor()
                if hasattr(req_to_token_pool, "get_state_dim_per_tensor")
                else None
            )
            append_state_component(
                kv_args, StateType.MAMBA, data_ptrs, data_lens, item_lens, dim
            )
```
**EN:** This block continues `setup_state_kv_args` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `get_state_buf_infos`, `get_state_dim_per_tensor`, `append_state_component`.
**CN:** 这一段延续了 `setup_state_kv_args` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `get_state_buf_infos`、`get_state_dim_per_tensor`、`append_state_component`。

### Lines 631-643: Function `prepare_abort`
```python
def prepare_abort(req: Req, error_message: str, status_code=None):
    from sglang.srt.managers.schedule_batch import FINISH_ABORT

    # populate finish metadata and stream output
    req.finished_reason = FINISH_ABORT(error_message, status_code)

    if req.return_logprob:
        req.input_token_logprobs_val = []
        req.input_token_logprobs_idx = []
        req.input_top_logprobs_val = []
        req.input_top_logprobs_idx = []
        req.input_token_ids_logprobs_val = []
        req.input_token_ids_logprobs_idx = []
```
**EN:** This block defines the function `prepare_abort`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `prepare_abort`. Notable operations include `FINISH_ABORT`.
**CN:** 这一段定义了function `prepare_abort`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `prepare_abort`。 值得注意的操作包括 `FINISH_ABORT`。

## Key Concepts / 关键概念
- `DisaggregationMode`: Class that encapsulates disaggregation mode behavior in this module. / `DisaggregationMode`：封装与“解耦部署mode”相关行为的类。
- `poll_and_all_reduce`: Function that performs poll and all reduce for the surrounding workflow. / `poll_and_all_reduce`：在周边工作流中执行“pollandallreduce”相关任务的函数。
- `poll_and_all_reduce_attn_cp_tp_group`: Function that performs poll and all reduce attn cp tp group for the surrounding workflow. / `poll_and_all_reduce_attn_cp_tp_group`：在周边工作流中执行“pollandallreduceattncptpgroup”相关任务的函数。
- `poll_and_all_reduce_with_staging`: Function that performs poll and all reduce with staging for the surrounding workflow. / `poll_and_all_reduce_with_staging`：在周边工作流中执行“pollandallreducewith暂存”相关任务的函数。
- `ReqToMetadataIdxAllocator`: Class that encapsulates req to metadata idx allocator behavior in this module. / `ReqToMetadataIdxAllocator`：封装与“reqto元数据idxallocator”相关行为的类。
- `MetadataBuffers`: Class that encapsulates metadata buffers behavior in this module. / `MetadataBuffers`：封装与“元数据buffers”相关行为的类。
- `TransferBackend`: Class that encapsulates transfer backend behavior in this module. / `TransferBackend`：封装与“传输后端”相关行为的类。
- `KVClassType`: Class that encapsulates kvclass type behavior in this module. / `KVClassType`：封装与“kvclass类型”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `os`, `random`, `collections`, `contextlib`, `enum`, `typing`
- **External packages / 外部依赖**: `numpy`, `torch`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.srt.disaggregation.base.conn`, `sglang.srt.disaggregation.common.conn`, `sglang.srt.managers.schedule_batch`, `sglang.srt.disaggregation.base`, `sglang.srt.disaggregation.fake`, `sglang.srt.mem_cache.deepseek_v4_memory_pool`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.hardware_backend.npu.memory_pool_npu`, `sglang.srt.mem_cache.base_swa_memory_pool`, `sglang.srt.disaggregation.mooncake`, `sglang.srt.disaggregation.mori`, `sglang.srt.disaggregation.ascend`, `sglang.srt.disaggregation.nixl`
