# tp_worker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/tp_worker.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements tp worker logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 tp 工作器 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Provide supporting module logic / 提供辅助模块逻辑
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
"""A tensor parallel worker."""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 16-16: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 18-20: Import runtime dependencies / 导入运行时依赖
```python
import logging
from abc import ABC, abstractmethod
from typing import TYPE_CHECKING, List, Optional, Tuple
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 22-22: Import runtime dependencies / 导入运行时依赖
```python
import torch
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 24-53: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.distributed import get_pp_group, get_world_group
from sglang.srt.managers.io_struct import (
    DestroyWeightsUpdateGroupReqInput,
    GetWeightsByNameReqInput,
    InitWeightsSendGroupForRemoteInstanceReqInput,
    InitWeightsUpdateGroupReqInput,
    LoadLoRAAdapterFromTensorsReqInput,
    LoadLoRAAdapterReqInput,
    SendWeightsToRemoteInstanceReqInput,
    UnloadLoRAAdapterReqInput,
    UpdateWeightFromDiskReqInput,
    UpdateWeightsFromDistributedReqInput,
    UpdateWeightsFromIPCReqInput,
    UpdateWeightsFromTensorReqInput,
)
from sglang.srt.managers.schedule_batch import ScheduleBatch
from sglang.srt.managers.scheduler import GenerationBatchResult
from sglang.srt.mem_cache.allocator import BaseTokenToKVPoolAllocator
from sglang.srt.mem_cache.memory_pool import ReqToTokenPool
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, PPProxyTensors
from sglang.srt.model_executor.pool_configurator import MemoryPoolConfig
from sglang.srt.server_args import ServerArgs
from sglang.srt.utils import MultiprocessingSerializer, broadcast_pyobj, set_random_seed
from sglang.srt.utils.hf_transformers_utils import (
    get_processor,
    get_tokenizer,
    get_tokenizer_from_processor,
)
from sglang.srt.utils.patch_torch import monkey_patch_torch_reductions
from sglang.srt.weight_sync.tensor_bucket import FlattenedTensorBucket
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 55-58: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.managers.cache_controller import LayerDoneCounter
    from sglang.srt.model_executor.model_runner import ModelRunner
    from sglang.srt.model_executor.pool_configurator import MemoryPoolConfig
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 60-60: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 63-64: Provide supporting module logic / 提供辅助模块逻辑
```python
class BaseTpWorker(ABC):
    @abstractmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 65-66: Implement forward batch generation / 实现前向 批处理 generation
```python
    def forward_batch_generation(self, forward_batch: ForwardBatch):
        pass
```
**EN:** This block implements the method `forward_batch_generation(forward_batch)` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `forward_batch_generation`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `forward_batch_generation(forward_batch)`。它围绕 `forward_batch_generation` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 67-69: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
    @abstractmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 70-71: Implement model runner / 实现模型 runner
```python
    def model_runner(self) -> "ModelRunner":
        pass
```
**EN:** This block implements the method `model_runner()` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `model_runner`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `model_runner()`。它围绕 `model_runner` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 72-73: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 74-75: Implement sliding window size / 实现sliding window size
```python
    def sliding_window_size(self) -> Optional[int]:
        return self.model_runner.sliding_window_size
```
**EN:** This block implements the method `sliding_window_size()` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `sliding_window_size`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `sliding_window_size()`。它围绕 `sliding_window_size` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 76-77: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 78-79: Implement is hybrid swa / 实现is hybrid swa
```python
    def is_hybrid_swa(self) -> bool:
        return self.model_runner.is_hybrid_swa
```
**EN:** This block implements the method `is_hybrid_swa()` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `is_hybrid_swa`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `is_hybrid_swa()`。它围绕 `is_hybrid_swa` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 81-85: Implement get tokens per layer info / 实现get tokens per layer info
```python
    def get_tokens_per_layer_info(self):
        return (
            self.model_runner.full_max_total_num_tokens,
            self.model_runner.swa_max_total_num_tokens,
        )
```
**EN:** This block implements the method `get_tokens_per_layer_info()` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `get_tokens_per_layer_info`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `get_tokens_per_layer_info()`。它围绕 `get_tokens_per_layer_info` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 87-88: Implement get pad input ids func / 实现get pad input ids func
```python
    def get_pad_input_ids_func(self):
        return getattr(self.model_runner.model, "pad_input_ids", None)
```
**EN:** This block implements the method `get_pad_input_ids_func()` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `get_pad_input_ids_func`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `get_pad_input_ids_func()`。它围绕 `get_pad_input_ids_func` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 90-94: Implement get memory pool / 实现get memory 池
```python
    def get_memory_pool(self) -> Tuple[ReqToTokenPool, BaseTokenToKVPoolAllocator]:
        return (
            self.model_runner.req_to_token_pool,
            self.model_runner.token_to_kv_pool_allocator,
        )
```
**EN:** This block implements the method `get_memory_pool()` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `get_memory_pool`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `get_memory_pool()`。它围绕 `get_memory_pool` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 96-102: Implement update weights from disk / 实现update weights from disk
```python
    def update_weights_from_disk(self, recv_req: UpdateWeightFromDiskReqInput):
        success, message = self.model_runner.update_weights_from_disk(
            recv_req.model_path,
            recv_req.load_format,
            recapture_cuda_graph=recv_req.recapture_cuda_graph,
        )
        return success, message
```
**EN:** This block implements the method `update_weights_from_disk(recv_req)` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `update_weights_from_disk`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `update_weights_from_disk(recv_req)`。它围绕 `update_weights_from_disk` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 104-113: Implement init weights update group / 实现init weights update group
```python
    def init_weights_update_group(self, recv_req: InitWeightsUpdateGroupReqInput):
        success, message = self.model_runner.init_weights_update_group(
            recv_req.master_address,
            recv_req.master_port,
            recv_req.rank_offset,
            recv_req.world_size,
            recv_req.group_name,
            recv_req.backend,
        )
        return success, message
```
**EN:** This block implements the method `init_weights_update_group(recv_req)` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `init_weights_update_group`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `init_weights_update_group(recv_req)`。它围绕 `init_weights_update_group` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 115-119: Implement destroy weights update group / 实现destroy weights update group
```python
    def destroy_weights_update_group(self, recv_req: DestroyWeightsUpdateGroupReqInput):
        success, message = self.model_runner.destroy_weights_update_group(
            recv_req.group_name,
        )
        return success, message
```
**EN:** This block implements the method `destroy_weights_update_group(recv_req)` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `destroy_weights_update_group`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `destroy_weights_update_group(recv_req)`。它围绕 `destroy_weights_update_group` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 121-134: Implement init weights send group for remote instance / 实现init weights send group for remote instance
```python
    def init_weights_send_group_for_remote_instance(
        self, recv_req: InitWeightsSendGroupForRemoteInstanceReqInput
    ):
        success, message = (
            self.model_runner.init_weights_send_group_for_remote_instance(
                recv_req.master_address,
                recv_req.ports,
                recv_req.group_rank,
                recv_req.world_size,
                recv_req.group_name,
                recv_req.backend,
            )
        )
        return success, message
```
**EN:** This block implements the method `init_weights_send_group_for_remote_instance(recv_req)` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `init_weights_send_group_for_remote_instance`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `init_weights_send_group_for_remote_instance(recv_req)`。它围绕 `init_weights_send_group_for_remote_instance` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 136-144: Implement send weights to remote instance / 实现send weights to remote instance
```python
    def send_weights_to_remote_instance(
        self, recv_req: SendWeightsToRemoteInstanceReqInput
    ):
        success, message = self.model_runner.send_weights_to_remote_instance(
            recv_req.master_address,
            recv_req.ports,
            recv_req.group_name,
        )
        return success, message
```
**EN:** This block implements the method `send_weights_to_remote_instance(recv_req)` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `send_weights_to_remote_instance`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `send_weights_to_remote_instance(recv_req)`。它围绕 `send_weights_to_remote_instance` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 146-156: Implement update weights from distributed / 实现update weights from distributed
```python
    def update_weights_from_distributed(
        self, recv_req: UpdateWeightsFromDistributedReqInput
    ):
        success, message = self.model_runner.update_weights_from_distributed(
            recv_req.names,
            recv_req.dtypes,
            recv_req.shapes,
            recv_req.group_name,
            recv_req.load_format,
        )
        return success, message
```
**EN:** This block implements the method `update_weights_from_distributed(recv_req)` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `update_weights_from_distributed`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `update_weights_from_distributed(recv_req)`。它围绕 `update_weights_from_distributed` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 158-167: Implement update weights from tensor / 实现update weights from 张量
```python
    def update_weights_from_tensor(self, recv_req: UpdateWeightsFromTensorReqInput):

        monkey_patch_torch_reductions()
        success, message = self.model_runner.update_weights_from_tensor(
            named_tensors=MultiprocessingSerializer.deserialize(
                recv_req.serialized_named_tensors[self.tp_rank]
            ),
            load_format=recv_req.load_format,
        )
        return success, message
```
**EN:** This block implements the method `update_weights_from_tensor(recv_req)` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `update_weights_from_tensor`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `update_weights_from_tensor(recv_req)`。它围绕 `update_weights_from_tensor` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 169-172: Implement update weights from ipc / 实现update weights from ipc
```python
    def update_weights_from_ipc(self, recv_req: UpdateWeightsFromIPCReqInput):
        """Update weights from IPC for checkpoint-engine integration."""
        success, message = self.model_runner.update_weights_from_ipc(recv_req)
        return success, message
```
**EN:** This block implements the method `update_weights_from_ipc(recv_req)` on `BaseTpWorker`. It focuses on Update weights from IPC for checkpoint-engine integration., so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `update_weights_from_ipc(recv_req)`。它围绕 `update_weights_from_ipc` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 174-178: Implement get weights by name / 实现get weights by name
```python
    def get_weights_by_name(self, recv_req: GetWeightsByNameReqInput):
        parameter = self.model_runner.get_weights_by_name(
            recv_req.name, recv_req.truncate_size
        )
        return parameter
```
**EN:** This block implements the method `get_weights_by_name(recv_req)` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `get_weights_by_name`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `get_weights_by_name(recv_req)`。它围绕 `get_weights_by_name` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 180-182: Implement load lora adapter / 实现load lora adapter
```python
    def load_lora_adapter(self, recv_req: LoadLoRAAdapterReqInput):
        result = self.model_runner.load_lora_adapter(recv_req.to_ref())
        return result
```
**EN:** This block implements the method `load_lora_adapter(recv_req)` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `load_lora_adapter`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `load_lora_adapter(recv_req)`。它围绕 `load_lora_adapter` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 184-186: Implement unload lora adapter / 实现unload lora adapter
```python
    def unload_lora_adapter(self, recv_req: UnloadLoRAAdapterReqInput):
        result = self.model_runner.unload_lora_adapter(recv_req.to_ref())
        return result
```
**EN:** This block implements the method `unload_lora_adapter(recv_req)` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `unload_lora_adapter`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `unload_lora_adapter(recv_req)`。它围绕 `unload_lora_adapter` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 188-210: Implement load lora adapter from tensors / 实现load lora adapter from tensors
```python
    def load_lora_adapter_from_tensors(
        self, recv_req: LoadLoRAAdapterFromTensorsReqInput
    ):
        # The LoRA code handles TP sharding internally using slice_lora_a_weights
        # and slice_lora_b_weights methods (see lora/layers.py:46-49, mem_pool.py:437-440).
        if recv_req.load_format == "flattened_bucket":
            flattened_data = MultiprocessingSerializer.deserialize(
                recv_req.serialized_tensors
            )
            bucket = FlattenedTensorBucket(
                flattened_tensor=flattened_data["flattened_tensor"],
                metadata=flattened_data["metadata"],
            )
            tensors = dict(bucket.reconstruct_tensors())
        else:
            tensors = MultiprocessingSerializer.deserialize(recv_req.serialized_tensors)
        result = self.model_runner.load_lora_adapter_from_tensors(
            recv_req.to_ref(),
            tensors,
            recv_req.config_dict,
            recv_req.added_tokens_config,
        )
        return result
```
**EN:** This block implements the method `load_lora_adapter_from_tensors(recv_req)` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `load_lora_adapter_from_tensors`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `load_lora_adapter_from_tensors(recv_req)`。它围绕 `load_lora_adapter_from_tensors` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 212-215: Implement forward batch embedding / 实现前向 批处理 嵌入
```python
    def forward_batch_embedding(self, batch: ScheduleBatch):
        forward_batch = ForwardBatch.init_new(batch, self.model_runner)
        output = self.model_runner.forward(forward_batch).logits_output
        return output  # Returns EmbeddingPoolerOutput
```
**EN:** This block implements the method `forward_batch_embedding(batch)` on `BaseTpWorker`. It focuses on handling the tp worker responsibilities represented by `forward_batch_embedding`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `BaseTpWorker` 上的方法 `forward_batch_embedding(batch)`。它围绕 `forward_batch_embedding` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 218-219: Provide supporting module logic / 提供辅助模块逻辑
```python
class TpModelWorker(BaseTpWorker):
    """A tensor parallel model worker."""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 221-256: Initialize TpModelWorker / 初始化 TpModelWorker
```python
    def __init__(
        self,
        server_args: ServerArgs,
        gpu_id: int,
        tp_rank: int,
        moe_ep_rank: int,
        pp_rank: int,
        attn_cp_rank: int,
        moe_dp_rank: int,
        dp_rank: Optional[int],
        nccl_port: int,
        is_draft_worker: bool = False,
        req_to_token_pool: Optional[ReqToTokenPool] = None,
        token_to_kv_pool_allocator: Optional[BaseTokenToKVPoolAllocator] = None,
        memory_pool_config: Optional[MemoryPoolConfig] = None,
        is_multi_layer_eagle: bool = False,
    ):
        # Parse args
        self.server_args = server_args
        self.tp_size = server_args.tp_size
        self.ep_size = server_args.ep_size
        self.pp_size = server_args.pp_size
        self.tp_rank = tp_rank
        self.moe_ep_rank = moe_ep_rank
        self.pp_rank = pp_rank
        self.dp_rank = dp_rank
        self.gpu_id = gpu_id
        self.nccl_port = nccl_port
        self.is_draft_worker = is_draft_worker
        self.is_multi_layer_eagle = is_multi_layer_eagle
        self.req_to_token_pool = req_to_token_pool
        self.token_to_kv_pool_allocator = token_to_kv_pool_allocator
        self.attn_cp_rank = attn_cp_rank
        self.moe_dp_rank = moe_dp_rank
        # Draft worker: target's resolved MemoryPoolConfig (forwarded to ModelRunner).
        self.memory_pool_config = memory_pool_config
```
**EN:** This block implements the initializer `__init__(server_args, gpu_id, tp_rank, moe_ep_rank, pp_rank, ...)` for `TpModelWorker`. It prepares the object state and connects the instance to the surrounding tp worker workflow.
**CN:** 该代码块实现 `TpModelWorker` 的初始化方法 `__init__(server_args, gpu_id, tp_rank, moe_ep_rank, pp_rank, ...)`。它负责准备对象状态，并把实例接入 tp 工作器 相关的运行流程。

### Lines 257-292: Initialize TpModelWorker (continued) / 初始化 TpModelWorker（续）
```python

        # MTP model runners
        self.model_runner_list: List[ModelRunner] = []

        self._init_model_config()
        self._init_model_runner()

        if is_multi_layer_eagle:
            self._init_multi_layer_eagle_model_runners()

        self._init_dllm_algorithm()

        if server_args.skip_tokenizer_init:
            self.tokenizer = self.processor = None
        else:
            if self.model_config.is_multimodal:
                self.processor = get_processor(
                    server_args.tokenizer_path,
                    tokenizer_mode=server_args.tokenizer_mode,
                    trust_remote_code=server_args.trust_remote_code,
                    revision=server_args.revision,
                    tokenizer_backend=server_args.tokenizer_backend,
                )
                self.tokenizer = get_tokenizer_from_processor(self.processor)
            else:
                self.tokenizer = get_tokenizer(
                    server_args.tokenizer_path,
                    tokenizer_mode=server_args.tokenizer_mode,
                    trust_remote_code=server_args.trust_remote_code,
                    revision=server_args.revision,
                    tokenizer_backend=server_args.tokenizer_backend,
                )
        self.device = self.model_runner.device

        # Init nccl groups
        self.pp_group = get_pp_group()
```
**EN:** This block implements the initializer `__init__(server_args, gpu_id, tp_rank, moe_ep_rank, pp_rank, ...)` for `TpModelWorker`. It prepares the object state and connects the instance to the surrounding tp worker workflow.
**CN:** 该代码块实现 `TpModelWorker` 的初始化方法 `__init__(server_args, gpu_id, tp_rank, moe_ep_rank, pp_rank, ...)`。它负责准备对象状态，并把实例接入 tp 工作器 相关的运行流程。

### Lines 293-324: Initialize TpModelWorker (continued) / 初始化 TpModelWorker（续）
```python
        self.world_group = get_world_group()

        # Profile number of tokens
        self.max_total_num_tokens = self.model_runner.max_total_num_tokens
        self.max_prefill_tokens = server_args.max_prefill_tokens
        self.max_running_requests = self.model_runner.max_running_requests
        assert self.max_running_requests > 0, "max_running_request is zero"
        self.max_queued_requests = server_args.max_queued_requests
        assert (
            self.max_queued_requests is None or self.max_queued_requests >= 1
        ), "If configured, max_queued_requests must be at least 1 for any work to be scheduled."
        self.max_req_len = min(
            self.model_config.context_len - 1,
            self.model_runner.max_token_pool_size - 1,
        )
        self.max_req_input_len = self.max_req_len - 5
        assert (
            self.max_req_len > 0 and self.max_req_input_len > 0
        ), "Memory pool size is too small"

        # Sync random seed across TP workers
        self.random_seed = broadcast_pyobj(
            [server_args.random_seed],
            self.tp_size * self.pp_rank + tp_rank,
            self.world_group.cpu_group,
            src=self.world_group.ranks[0],
        )[0]
        set_random_seed(self.random_seed)

        self.enable_overlap = not server_args.disable_overlap_schedule
        self.enable_spec = server_args.speculative_algorithm is not None
        self.hicache_layer_transfer_counter = None
```
**EN:** This block implements the initializer `__init__(server_args, gpu_id, tp_rank, moe_ep_rank, pp_rank, ...)` for `TpModelWorker`. It prepares the object state and connects the instance to the surrounding tp worker workflow.
**CN:** 该代码块实现 `TpModelWorker` 的初始化方法 `__init__(server_args, gpu_id, tp_rank, moe_ep_rank, pp_rank, ...)`。它负责准备对象状态，并把实例接入 tp 工作器 相关的运行流程。

### Lines 326-342: Implement init model config / 实现init 模型 配置
```python
    def _init_model_config(self):
        from sglang.srt.configs.model_config import ModelConfig

        self.model_config = ModelConfig.from_server_args(
            self.server_args,
            model_path=(
                self.server_args.model_path
                if not self.is_draft_worker
                else self.server_args.speculative_draft_model_path
            ),
            model_revision=(
                self.server_args.revision
                if not self.is_draft_worker
                else self.server_args.speculative_draft_model_revision
            ),
            is_draft_model=self.is_draft_worker,
        )
```
**EN:** This block implements the method `_init_model_config()` on `TpModelWorker`. It focuses on handling the tp worker responsibilities represented by `_init_model_config`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `TpModelWorker` 上的方法 `_init_model_config()`。它围绕 `_init_model_config` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 344-365: Implement init model runner / 实现init 模型 runner
```python
    def _init_model_runner(self):
        from sglang.srt.model_executor.model_runner import ModelRunner

        self._model_runner = ModelRunner(
            model_config=self.model_config,
            mem_fraction_static=self.server_args.mem_fraction_static,
            gpu_id=self.gpu_id,
            tp_rank=self.tp_rank,
            tp_size=self.tp_size,
            moe_ep_rank=self.moe_ep_rank,
            moe_ep_size=self.ep_size,
            pp_rank=self.pp_rank,
            pp_size=self.pp_size,
            nccl_port=self.nccl_port,
            dp_rank=self.dp_rank,
            server_args=self.server_args,
            is_draft_worker=self.is_draft_worker,
            req_to_token_pool=self.req_to_token_pool,
            token_to_kv_pool_allocator=self.token_to_kv_pool_allocator,
            memory_pool_config=self.memory_pool_config,
            draft_model_idx=0 if self.is_multi_layer_eagle else None,
        )
```
**EN:** This block implements the method `_init_model_runner()` on `TpModelWorker`. It focuses on handling the tp worker responsibilities represented by `_init_model_runner`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `TpModelWorker` 上的方法 `_init_model_runner()`。它围绕 `_init_model_runner` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 367-392: Implement init multi layer eagle model runners / 实现init multi layer eagle 模型 runners
```python
    def _init_multi_layer_eagle_model_runners(self):
        from sglang.srt.model_executor.model_runner import ModelRunner

        self.model_runner_list.append(self.model_runner)
        for i in range(1, self.server_args.speculative_num_steps):
            self.model_runner_list.append(
                ModelRunner(
                    model_config=self.model_config,
                    mem_fraction_static=self.server_args.mem_fraction_static,
                    gpu_id=self.gpu_id,
                    tp_rank=self.tp_rank,
                    tp_size=self.tp_size,
                    moe_ep_rank=self.moe_ep_rank,
                    moe_ep_size=self.ep_size,
                    pp_rank=self.pp_rank,
                    pp_size=self.pp_size,
                    nccl_port=self.nccl_port,
                    dp_rank=self.dp_rank,
                    server_args=self.server_args,
                    is_draft_worker=self.is_draft_worker,
                    req_to_token_pool=self.req_to_token_pool,
                    token_to_kv_pool_allocator=self.token_to_kv_pool_allocator,
                    memory_pool_config=self.memory_pool_config,
                    draft_model_idx=i,
                )
            )
```
**EN:** This block implements the method `_init_multi_layer_eagle_model_runners()` on `TpModelWorker`. It focuses on handling the tp worker responsibilities represented by `_init_multi_layer_eagle_model_runners`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `TpModelWorker` 上的方法 `_init_multi_layer_eagle_model_runners()`。它围绕 `_init_multi_layer_eagle_model_runners` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 394-400: Implement init dllm algorithm / 实现init dllm algorithm
```python
    def _init_dllm_algorithm(self):
        from sglang.srt.dllm.algorithm.base import DllmAlgorithm

        if self.server_args.dllm_algorithm is not None:
            self.dllm_algorithm = DllmAlgorithm.from_server_args(self.server_args)
        else:
            self.dllm_algorithm = None
```
**EN:** This block implements the method `_init_dllm_algorithm()` on `TpModelWorker`. It focuses on handling the tp worker responsibilities represented by `_init_dllm_algorithm`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `TpModelWorker` 上的方法 `_init_dllm_algorithm()`。它围绕 `_init_dllm_algorithm` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 401-402: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 403-404: Implement model runner / 实现模型 runner
```python
    def model_runner(self) -> "ModelRunner":
        return self._model_runner
```
**EN:** This block implements the method `model_runner()` on `TpModelWorker`. It focuses on handling the tp worker responsibilities represented by `model_runner`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `TpModelWorker` 上的方法 `model_runner()`。它围绕 `model_runner` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 406-407: Implement register hicache layer transfer counter / 实现register hicache layer transfer counter
```python
    def register_hicache_layer_transfer_counter(self, counter: LayerDoneCounter):
        self.hicache_layer_transfer_counter = counter
```
**EN:** This block implements the method `register_hicache_layer_transfer_counter(counter)` on `TpModelWorker`. It focuses on handling the tp worker responsibilities represented by `register_hicache_layer_transfer_counter`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `TpModelWorker` 上的方法 `register_hicache_layer_transfer_counter(counter)`。它围绕 `register_hicache_layer_transfer_counter` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 409-411: Implement set hicache consumer / 实现set hicache consumer
```python
    def set_hicache_consumer(self, consumer_index: int):
        if self.hicache_layer_transfer_counter is not None:
            self.hicache_layer_transfer_counter.set_consumer(consumer_index)
```
**EN:** This block implements the method `set_hicache_consumer(consumer_index)` on `TpModelWorker`. It focuses on handling the tp worker responsibilities represented by `set_hicache_consumer`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `TpModelWorker` 上的方法 `set_hicache_consumer(consumer_index)`。它围绕 `set_hicache_consumer` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 413-414: Implement register hisparse coordinator / 实现register hisparse 协调器
```python
    def register_hisparse_coordinator(self, coordinator):
        self.model_runner.hisparse_coordinator = coordinator
```
**EN:** This block implements the method `register_hisparse_coordinator(coordinator)` on `TpModelWorker`. It focuses on handling the tp worker responsibilities represented by `register_hisparse_coordinator`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `TpModelWorker` 上的方法 `register_hisparse_coordinator(coordinator)`。它围绕 `register_hisparse_coordinator` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 416-430: Implement get worker info / 实现get 工作器 info
```python
    def get_worker_info(self):
        return (
            self.max_total_num_tokens,
            self.max_prefill_tokens,
            self.max_running_requests,
            self.max_queued_requests,
            self.max_req_len,
            self.max_req_input_len,
            self.random_seed,
            self.device,
            self.model_runner.forward_stream,
            self.model_runner.req_to_token_pool.size,
            self.model_runner.req_to_token_pool.max_context_len,
            self.model_runner.token_to_kv_pool.size,
        )
```
**EN:** This block implements the method `get_worker_info()` on `TpModelWorker`. It focuses on handling the tp worker responsibilities represented by `get_worker_info`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `TpModelWorker` 上的方法 `get_worker_info()`。它围绕 `get_worker_info` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 432-433: Implement is dllm / 实现is dllm
```python
    def is_dllm(self):
        return self.dllm_algorithm is not None
```
**EN:** This block implements the method `is_dllm()` on `TpModelWorker`. It focuses on handling the tp worker responsibilities represented by `is_dllm`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `TpModelWorker` 上的方法 `is_dllm()`。它围绕 `is_dllm` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 435-445: Implement forward batch generation dllm / 实现前向 批处理 generation dllm
```python
    def _forward_batch_generation_dllm(
        self, forward_batch: ForwardBatch
    ) -> GenerationBatchResult:
        logits_output, next_token_ids, can_run_cuda_graph = self.dllm_algorithm.run(
            self.model_runner, forward_batch
        )
        return GenerationBatchResult(
            logits_output=logits_output,
            next_token_ids=next_token_ids,
            can_run_cuda_graph=can_run_cuda_graph,
        )
```
**EN:** This block implements the method `_forward_batch_generation_dllm(forward_batch)` on `TpModelWorker`. It focuses on handling the tp worker responsibilities represented by `_forward_batch_generation_dllm`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `TpModelWorker` 上的方法 `_forward_batch_generation_dllm(forward_batch)`。它围绕 `_forward_batch_generation_dllm` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 447-482: Implement forward batch generation / 实现前向 批处理 generation
```python
    def forward_batch_generation(
        self,
        batch: Optional[ScheduleBatch],
        forward_batch: Optional[ForwardBatch] = None,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
        is_verify: bool = False,
        skip_attn_backend_init=False,
    ) -> GenerationBatchResult:
        # FIXME(lsyin): maybe remove skip_attn_backend_init in forward_batch_generation,
        #               which requires preparing replay to always be in this function

        # Get forward batch from schedule batch
        if batch is not None:
            # update the consumer index of hicache to the running batch
            self.set_hicache_consumer(batch.hicache_consumer_index)

            forward_batch = ForwardBatch.init_new(batch, self.model_runner)
        else:
            # FIXME(lsyin): unify the interface of forward_batch
            assert forward_batch is not None

        if self.is_dllm():
            return self._forward_batch_generation_dllm(forward_batch)

        if self.pp_group.is_last_rank:
            out = self.model_runner.forward(
                forward_batch,
                pp_proxy_tensors=pp_proxy_tensors,
                skip_attn_backend_init=skip_attn_backend_init,
            )
            logits_output, can_run_cuda_graph = out.logits_output, out.can_run_graph
            batch_result = GenerationBatchResult(
                logits_output=logits_output,
                can_run_cuda_graph=can_run_cuda_graph,
                expert_distribution_metrics=out.expert_distribution_metrics,
                routed_experts_output=out.routed_experts_output,
```
**EN:** This block implements the method `forward_batch_generation(batch, forward_batch, pp_proxy_tensors, is_verify, skip_attn_backend_init)` on `TpModelWorker`. It focuses on handling the tp worker responsibilities represented by `forward_batch_generation`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `TpModelWorker` 上的方法 `forward_batch_generation(batch, forward_batch, pp_proxy_tensors, is_verify, skip_attn_backend_init)`。它围绕 `forward_batch_generation` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 483-518: Continue forward batch generation / 继续说明前向 批处理 generation
```python
                indexer_topk_output=out.indexer_topk_output,
            )

            if is_verify:
                # Skip sampling and return logits for target forward
                return batch_result

            if (
                self.enable_overlap
                and not self.enable_spec
                and forward_batch.sampling_info.grammars is not None
            ):

                def sample_batch_func():
                    batch_result.next_token_ids = self.model_runner.sample(
                        logits_output, forward_batch
                    )
                    return batch_result

                batch_result.delay_sample_func = sample_batch_func
                return batch_result

            if not forward_batch.is_prefill_only:
                # For normal requests, sample the next token ids.
                batch_result.next_token_ids = self.model_runner.sample(
                    logits_output, forward_batch
                )
            else:
                # For prefill-only requests, create dummy token IDs on CPU
                # The size should match the batch size (number of sequences), not total tokens
                batch_result.next_token_ids = torch.zeros(
                    len(forward_batch.seq_lens),
                    dtype=torch.long,
                    device=forward_batch.input_ids.device,
                )
                if (
```
**EN:** This block implements the method `forward_batch_generation(batch, forward_batch, pp_proxy_tensors, is_verify, skip_attn_backend_init)` on `TpModelWorker`. It focuses on handling the tp worker responsibilities represented by `forward_batch_generation`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `TpModelWorker` 上的方法 `forward_batch_generation(batch, forward_batch, pp_proxy_tensors, is_verify, skip_attn_backend_init)`。它围绕 `forward_batch_generation` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 519-539: Continue forward batch generation / 继续说明前向 批处理 generation
```python
                    forward_batch.return_logprob
                    and logits_output.next_token_logits is not None
                ):
                    # NOTE: Compute logprobs without full sampling
                    self.model_runner.compute_logprobs_only(
                        logits_output, forward_batch
                    )

            return batch_result
        else:
            out = self.model_runner.forward(
                forward_batch,
                pp_proxy_tensors=pp_proxy_tensors,
                skip_attn_backend_init=skip_attn_backend_init,
            )
            pp_proxy_tensors, can_run_cuda_graph = out.logits_output, out.can_run_graph
            return GenerationBatchResult(
                pp_hidden_states_proxy_tensors=pp_proxy_tensors,
                can_run_cuda_graph=can_run_cuda_graph,
                expert_distribution_metrics=out.expert_distribution_metrics,
            )
```
**EN:** This block implements the method `forward_batch_generation(batch, forward_batch, pp_proxy_tensors, is_verify, skip_attn_backend_init)` on `TpModelWorker`. It focuses on handling the tp worker responsibilities represented by `forward_batch_generation`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `TpModelWorker` 上的方法 `forward_batch_generation(batch, forward_batch, pp_proxy_tensors, is_verify, skip_attn_backend_init)`。它围绕 `forward_batch_generation` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

### Lines 541-562: Implement forward batch split prefill / 实现前向 批处理 split prefill
```python
    def forward_batch_split_prefill(self, batch: ScheduleBatch):
        if batch.split_index == 0:
            forward_batch = ForwardBatch.init_new(batch, self.model_runner)
            batch.split_forward_batch = forward_batch

        out = self.model_runner.forward(
            batch.split_forward_batch, split_forward_count=batch.split_forward_count
        )
        logits_output, can_run_cuda_graph = out.logits_output, out.can_run_graph
        if logits_output:
            next_token_ids = self.model_runner.sample(
                logits_output, batch.split_forward_batch
            )
        else:
            next_token_ids = None
        batch_result = GenerationBatchResult(
            logits_output=logits_output,
            can_run_cuda_graph=can_run_cuda_graph,
            expert_distribution_metrics=out.expert_distribution_metrics,
        )
        batch_result.next_token_ids = next_token_ids
        return batch_result
```
**EN:** This block implements the method `forward_batch_split_prefill(batch)` on `TpModelWorker`. It focuses on handling the tp worker responsibilities represented by `forward_batch_split_prefill`, so the class can advance the tp worker workflow in a self-contained way.
**CN:** 该代码块实现 `TpModelWorker` 上的方法 `forward_batch_split_prefill(batch)`。它围绕 `forward_batch_split_prefill` 所承担的 tp 工作器 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: BaseTpWorker, TpModelWorker
- **Domain focus / 领域焦点**: tp worker / tp 工作器
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: logging, typing
- **Third-party / 第三方库**: __future__, abc, torch
- **Local Modules / 本地模块**: sglang.srt.configs.model_config, sglang.srt.distributed, sglang.srt.dllm.algorithm.base, sglang.srt.managers.cache_controller, sglang.srt.managers.io_struct, sglang.srt.managers.schedule_batch, sglang.srt.managers.scheduler, sglang.srt.mem_cache.allocator, sglang.srt.mem_cache.memory_pool, sglang.srt.model_executor.forward_batch_info, sglang.srt.model_executor.model_runner, sglang.srt.model_executor.pool_configurator
