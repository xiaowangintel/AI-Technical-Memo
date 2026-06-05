# default.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/model_states/default.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `DefaultModelState` for the V1 `worker/gpu/model_states` subsystem. / 为 V1 的 `worker/gpu/model_states` 子系统实现 `DefaultModelState`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from typing import Any

import torch
import torch.nn as nn

from vllm.config import VllmConfig
from vllm.config.compilation import CUDAGraphMode
from vllm.tasks import GenerationTask
from vllm.v1.core.sched.output import NewRequestData
from vllm.v1.kv_cache_interface import KVCacheConfig
from vllm.v1.worker.gpu.attn_utils import build_attn_metadata
from vllm.v1.worker.gpu.input_batch import InputBatch
from vllm.v1.worker.gpu.mm.encoder_cache import EncoderCache
from vllm.v1.worker.gpu.mm.encoder_runner import EncoderRunner
from vllm.v1.worker.gpu.mm.rope import get_rope_state
from vllm.v1.worker.gpu.model_states.interface import ModelState
from vllm.v1.worker.gpu.states import RequestState
from vllm.v1.worker.utils import AttentionGroup
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.config`, `vllm.config.compilation`, `vllm.tasks`, `vllm.v1.core.sched.output`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.config`, `vllm.config.compilation`, `vllm.tasks`, `vllm.v1.core.sched.output` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `DefaultModelState` class / `DefaultModelState` 类
```python
class DefaultModelState(ModelState):
```
**EN:** Introduces the `DefaultModelState` class on top of `ModelState`. Core methods include `__init__`, `get_supported_generation_tasks`, `add_request`, `apply_staged_writes`, `get_mm_embeddings`, `prepare_inputs`.
**CN:** 这里定义 `DefaultModelState` 类，其基类包括 `ModelState`。核心方法包括 `__init__`, `get_supported_generation_tasks`, `add_request`, `apply_staged_writes`, `get_mm_embeddings`, `prepare_inputs`。

### `DefaultModelState.__init__` method / `DefaultModelState.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        model: nn.Module,
        encoder_cache: EncoderCache | None,
        device: torch.device,
    ):
        self.vllm_config = vllm_config
        self.model_config = vllm_config.model_config
        self.scheduler_config = vllm_config.scheduler_config
        self.model = model
        self.device = device

        self.supports_mm_inputs = encoder_cache is not None
        self.max_model_len = self.model_config.max_model_len
        self.max_num_reqs = self.scheduler_config.max_num_seqs
        self.max_num_tokens = self.scheduler_config.max_num_batched_tokens
        self.inputs_embeds_size = self.model_config.get_inputs_embeds_size()
        self.dtype = self.model_config.dtype

        if self.supports_mm_inputs:
            assert encoder_cache is not None
            self.encoder_cache = encoder_cache
            self.encoder_runner = EncoderRunner(
                model=self.model,
                max_num_tokens=self.max_num_tokens,
                hidden_size=self.inputs_embeds_size,
                encoder_cache=encoder_cache,
                dtype=self.dtype,
                device=self.device,
            )

        self.rope_state = get_rope_state(
            self.model_config,
            model,
            max_num_reqs=self.max_num_reqs,
            max_num_tokens=self.max_num_tokens,
            max_model_len=self.max_model_len,
            device=self.device,
        )
```
**EN:** This method initializes the object state within `DefaultModelState`. Key calls include `get_inputs_embeds_size`, `get_rope_state`, `EncoderRunner`. It touches state such as `vllm_config`, `model_config`, `scheduler_config`, `model`, `device`, `supports_mm_inputs`, `max_model_len`, `max_num_reqs`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`DefaultModelState`。 关键调用包括 `get_inputs_embeds_size`, `get_rope_state`, `EncoderRunner`。 它会读写 `vllm_config`, `model_config`, `scheduler_config`, `model`, `device`, `supports_mm_inputs`, `max_model_len`, `max_num_reqs` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DefaultModelState.get_supported_generation_tasks` method / `DefaultModelState.get_supported_generation_tasks` 方法
```python
    def get_supported_generation_tasks(self) -> tuple[GenerationTask, ...]:
        from vllm.model_executor.models.interfaces import (
            supports_realtime,
            supports_transcription,
        )
        from vllm.model_executor.models.interfaces_base import is_text_generation_model

        supported_tasks = list[GenerationTask]()

        if is_text_generation_model(self.model):
            supported_tasks.append("generate")

        if supports_transcription(self.model):
            if self.model.supports_transcription_only:
                return ("transcription",)
            supported_tasks.append("transcription")

        if supports_realtime(self.model):
            supported_tasks.append("realtime")

        return tuple(supported_tasks)
```
**EN:** This method returns or derives a value within `DefaultModelState`. Key calls include `is_text_generation_model`, `supports_transcription`, `supports_realtime`, `tuple`, `append`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`DefaultModelState`。 关键调用包括 `is_text_generation_model`, `supports_transcription`, `supports_realtime`, `tuple`, `append`。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DefaultModelState.add_request` method / `DefaultModelState.add_request` 方法
```python
    def add_request(self, req_index: int, new_req_data: NewRequestData) -> None:
        if self.rope_state is not None:
            assert new_req_data.prefill_token_ids is not None
            self.rope_state.init_prefill_positions(
                req_index,
                self.model,
                new_req_data.prefill_token_ids,
                mm_features=new_req_data.mm_features,
            )
```
**EN:** This method implements `add_request` within `DefaultModelState`. Key calls include `init_prefill_positions`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `add_request`，其作用域位于`DefaultModelState`。 关键调用包括 `init_prefill_positions`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DefaultModelState.apply_staged_writes` method / `DefaultModelState.apply_staged_writes` 方法
```python
    def apply_staged_writes(self) -> None:
        if self.rope_state is not None:
            self.rope_state.apply_staged_writes()
```
**EN:** This method implements `apply_staged_writes` within `DefaultModelState`. Key calls include `apply_staged_writes`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `apply_staged_writes`，其作用域位于`DefaultModelState`。 关键调用包括 `apply_staged_writes`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DefaultModelState.get_mm_embeddings` method / `DefaultModelState.get_mm_embeddings` 方法
```python
    def get_mm_embeddings(
        self,
        scheduled_encoder_inputs: dict[str, list[int]],
        input_batch: InputBatch,
        req_states: RequestState,
    ) -> torch.Tensor:
        mm_hashes, mm_kwargs = self.encoder_runner.prepare_mm_inputs(
            scheduled_encoder_inputs
        )
        if mm_kwargs:
            # Execute the multimodal encoder.
            encoder_outputs = self.encoder_runner.execute_mm_encoder(mm_kwargs)
            # Cache the encoder outputs by mm_hash
            self.encoder_cache.encoder_outputs.update(zip(mm_hashes, encoder_outputs))

        mm_embeds, is_mm_embed = self.encoder_runner.gather_mm_embeddings(
            input_batch.req_ids,
            input_batch.num_tokens,
            input_batch.num_scheduled_tokens,
            input_batch.query_start_loc_np,
            req_states.prefill_len.np[input_batch.idx_mapping_np],
            req_states.num_computed_prefill_tokens[input_batch.idx_mapping_np],
        )
        # Use unpadded input_ids to match is_mm_embed size (num_tokens).
        # input_batch.input_ids may be padded for CUDA graphs.
        input_ids_unpadded = input_batch.input_ids[: input_batch.num_tokens]
        inputs_embeds = self.encoder_runner.get_inputs_embeds(
            input_ids_unpadded, mm_embeds, is_mm_embed
        )
        return inputs_embeds[: input_batch.num_tokens_after_padding]
```
**EN:** This method returns or derives a value within `DefaultModelState`. Key calls include `prepare_mm_inputs`, `gather_mm_embeddings`, `get_inputs_embeds`, `execute_mm_encoder`, `update`, `zip`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`DefaultModelState`。 关键调用包括 `prepare_mm_inputs`, `gather_mm_embeddings`, `get_inputs_embeds`, `execute_mm_encoder`, `update`, `zip`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DefaultModelState.prepare_inputs` method / `DefaultModelState.prepare_inputs` 方法
```python
    def prepare_inputs(
        self, input_batch: InputBatch, req_states: RequestState
    ) -> dict[str, torch.Tensor | None]:
        if self.rope_state is None:
            return {}  # Common case (1D positions).

        self.rope_state.prepare_positions(
            input_batch.idx_mapping,
            input_batch.query_start_loc,
            req_states.prefill_len.gpu,
            req_states.num_computed_tokens.gpu,
        )
        positions = self.rope_state.get_positions(input_batch.num_tokens_after_padding)
        return {"positions": positions}
```
**EN:** This method prepares inputs and state within `DefaultModelState`. Key calls include `prepare_positions`, `get_positions`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会准备输入与状态，其作用域位于`DefaultModelState`。 关键调用包括 `prepare_positions`, `get_positions`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DefaultModelState.prepare_dummy_inputs` method / `DefaultModelState.prepare_dummy_inputs` 方法
```python
    def prepare_dummy_inputs(self, num_reqs: int, num_tokens: int) -> dict[str, Any]:
        model_inputs = {}
        if self.supports_mm_inputs:
            inputs_embeds = self.encoder_runner.inputs_embeds[:num_tokens]
            model_inputs["inputs_embeds"] = inputs_embeds
        if self.rope_state is not None:
            model_inputs["positions"] = self.rope_state.get_positions(num_tokens)
        return model_inputs
```
**EN:** This method prepares inputs and state within `DefaultModelState`. Key calls include `get_positions`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会准备输入与状态，其作用域位于`DefaultModelState`。 关键调用包括 `get_positions`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DefaultModelState.prepare_attn` method / `DefaultModelState.prepare_attn` 方法
```python
    def prepare_attn(
        self,
        input_batch: InputBatch,
        cudagraph_mode: CUDAGraphMode,
        block_tables: tuple[torch.Tensor, ...],
        slot_mappings: torch.Tensor,
        attn_groups: list[list[AttentionGroup]],
        kv_cache_config: KVCacheConfig,
        for_capture: bool = False,
    ) -> dict[str, Any]:
        if cudagraph_mode == CUDAGraphMode.FULL:
            # Use padded sizes - padding is handled by model_runner.prepare_attn.
            num_reqs = input_batch.num_reqs_after_padding
            num_tokens = input_batch.num_tokens_after_padding
        else:
            # For piecewise cudagraphs and eager, use unpadded sizes.
            num_reqs = input_batch.num_reqs
            num_tokens = input_batch.num_tokens
        query_start_loc_cpu = torch.from_numpy(input_batch.query_start_loc_np)
        max_query_len = input_batch.num_scheduled_tokens.max().item()
        seq_lens_cpu_upper_bound = input_batch.seq_lens_cpu_upper_bound
        if for_capture:
            # Capture with worst-case max_seq_len so the graph is valid at any replay.
            max_seq_len = self.max_model_len
        else:
            max_seq_len = int(seq_lens_cpu_upper_bound[:num_reqs].max().item())
        attn_metadata = build_attn_metadata(
            attn_groups=attn_groups,
            num_reqs=num_reqs,
            num_tokens=num_tokens,
            query_start_loc_gpu=input_batch.query_start_loc,
            query_start_loc_cpu=query_start_loc_cpu,
            max_query_len=max_query_len,
            seq_lens=input_batch.seq_lens,
            max_seq_len=max_seq_len,
            block_tables=block_tables,
            slot_mappings=slot_mappings,
            kv_cache_config=kv_cache_config,
            seq_lens_cpu_upper_bound=seq_lens_cpu_upper_bound,
            dcp_local_seq_lens=input_batch.dcp_local_seq_lens,
            positions=input_batch.positions,
            for_cudagraph_capture=for_capture,
        )
        return attn_metadata
```
**EN:** This method prepares inputs and state within `DefaultModelState`. Key calls include `from_numpy`, `item`, `build_attn_metadata`, `int`, `max`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会准备输入与状态，其作用域位于`DefaultModelState`。 关键调用包括 `from_numpy`, `item`, `build_attn_metadata`, `int`, `max`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `DefaultModelState`: central class or interface in this module. / `DefaultModelState`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.config.compilation`, `vllm.tasks`, `vllm.v1.core.sched.output`, `vllm.v1.kv_cache_interface`, `vllm.v1.worker.gpu.attn_utils`, `vllm.v1.worker.gpu.input_batch`, `vllm.v1.worker.gpu.mm.encoder_cache`, `vllm.v1.worker.gpu.mm.encoder_runner`, `vllm.v1.worker.gpu.mm.rope`, `vllm.v1.worker.gpu.model_states.interface`, `vllm.v1.worker.gpu.states`
