# whisper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/model_states/whisper.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `WhisperAttnMetadata`, `WhisperModelState` for the V1 `worker/gpu/model_states` subsystem. / 为 V1 的 `worker/gpu/model_states` 子系统实现 `WhisperAttnMetadata`, `WhisperModelState`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from dataclasses import dataclass
from typing import Any

import numpy as np
import torch
import torch.nn as nn

from vllm.config import VllmConfig
from vllm.config.compilation import CUDAGraphMode
from vllm.v1.kv_cache_interface import CrossAttentionSpec, KVCacheConfig
from vllm.v1.worker.gpu.attn_utils import build_attn_metadata
from vllm.v1.worker.gpu.input_batch import InputBatch
from vllm.v1.worker.gpu.mm.encoder_cache import EncoderCache
from vllm.v1.worker.gpu.mm.encoder_runner import EncoderRunner
from vllm.v1.worker.gpu.model_states.interface import (
    ModelSpecificAttnMetadata,
    ModelState,
)
from vllm.v1.worker.gpu.states import RequestState
from vllm.v1.worker.utils import AttentionGroup
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, and internal vLLM modules such as `vllm.config`, `vllm.config.compilation`, `vllm.v1.kv_cache_interface`, `vllm.v1.worker.gpu.attn_utils`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖，以及 `vllm.config`, `vllm.config.compilation`, `vllm.v1.kv_cache_interface`, `vllm.v1.worker.gpu.attn_utils` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `WhisperAttnMetadata` class / `WhisperAttnMetadata` 类
```python
@dataclass
class WhisperAttnMetadata(ModelSpecificAttnMetadata):
    encoder_seq_lens: dict[int, tuple[torch.Tensor, np.ndarray]]
```
**EN:** Uses `@dataclass` to package related state for `WhisperAttnMetadata`. Typical fields include `encoder_seq_lens`.
**CN:** `WhisperAttnMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `encoder_seq_lens`。

### `WhisperAttnMetadata.get_extra_common_attn_kwargs` method / `WhisperAttnMetadata.get_extra_common_attn_kwargs` 方法
```python
    def get_extra_common_attn_kwargs(
        self,
        kv_cache_group_id: int,
        num_reqs: int,
    ) -> dict[str, Any]:
        encoder_seq_lens = self.encoder_seq_lens.get(kv_cache_group_id)
        if encoder_seq_lens is None:
            return {}
        encoder_seq_lens_gpu, encoder_seq_lens_cpu = encoder_seq_lens
        return {
            "encoder_seq_lens": encoder_seq_lens_gpu[:num_reqs],
            "encoder_seq_lens_cpu": encoder_seq_lens_cpu[:num_reqs],
        }
```
**EN:** This method returns or derives a value within `WhisperAttnMetadata`. Key calls include `get`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`WhisperAttnMetadata`。 关键调用包括 `get`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `WhisperModelState` class / `WhisperModelState` 类
```python
class WhisperModelState(ModelState):
```
**EN:** Introduces the `WhisperModelState` class on top of `ModelState`. Core methods include `__init__`, `get_supported_generation_tasks`, `get_mm_embeddings`, `prepare_inputs`, `prepare_dummy_inputs`, `prepare_attn`.
**CN:** 这里定义 `WhisperModelState` 类，其基类包括 `ModelState`。核心方法包括 `__init__`, `get_supported_generation_tasks`, `get_mm_embeddings`, `prepare_inputs`, `prepare_dummy_inputs`, `prepare_attn`。

### `WhisperModelState.__init__` method / `WhisperModelState.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        model: nn.Module,
        encoder_cache: EncoderCache | None,
        device: torch.device,
    ) -> None:
        self.vllm_config = vllm_config
        self.model_config = vllm_config.model_config
        self.scheduler_config = vllm_config.scheduler_config
        self.model = model
        self.max_num_reqs = vllm_config.scheduler_config.max_num_seqs
        self.max_num_tokens = self.scheduler_config.max_num_batched_tokens
        self.max_model_len = self.model_config.max_model_len
        self.device = device

        assert encoder_cache is not None
        self.encoder_cache = encoder_cache
        self.encoder_runner = EncoderRunner(
            model=self.model,
            max_num_tokens=self.max_num_tokens,
            hidden_size=self.model_config.get_inputs_embeds_size(),
            encoder_cache=self.encoder_cache,
            dtype=self.model_config.dtype,
            device=self.device,
        )

        self.max_encoder_len = getattr(
            self.model_config.hf_config,
            "max_source_positions",
            self.max_model_len,
        )
        self.encoder_seq_lens_gpu = torch.zeros(
            self.max_num_reqs, dtype=torch.int32, device=self.device
        )

        self.encoder_outputs: list[torch.Tensor] = []
```
**EN:** This method initializes the object state within `WhisperModelState`. Key calls include `EncoderRunner`, `getattr`, `zeros`, `get_inputs_embeds_size`. It touches state such as `vllm_config`, `model_config`, `scheduler_config`, `model`, `max_num_reqs`, `max_num_tokens`, `max_model_len`, `device`.
**CN:** 该方法会初始化对象状态，其作用域位于`WhisperModelState`。 关键调用包括 `EncoderRunner`, `getattr`, `zeros`, `get_inputs_embeds_size`。 它会读写 `vllm_config`, `model_config`, `scheduler_config`, `model`, `max_num_reqs`, `max_num_tokens`, `max_model_len`, `device` 等状态。

### `WhisperModelState.get_supported_generation_tasks` method / `WhisperModelState.get_supported_generation_tasks` 方法
```python
    def get_supported_generation_tasks(self):
        return ("transcription",)
```
**EN:** This method returns or derives a value within `WhisperModelState`.
**CN:** 该方法会返回或推导一个值，其作用域位于`WhisperModelState`。

### `WhisperModelState.get_mm_embeddings` method / `WhisperModelState.get_mm_embeddings` 方法
```python
    def get_mm_embeddings(
        self,
        scheduled_encoder_inputs: dict[str, list[int]],
        input_batch: InputBatch,
        req_states: RequestState,
    ) -> None:
        # Ensure encoder inputs are ordered consistently with input_batch.req_ids.
        encoder_inputs: dict[str, list[int]] = {}
        for req_id in input_batch.req_ids:
            req_encoder_inputs = scheduled_encoder_inputs.get(req_id, [])
            if req_encoder_inputs:
                encoder_inputs[req_id] = req_encoder_inputs
        _, mm_kwargs = self.encoder_runner.prepare_mm_inputs(encoder_inputs)
        if mm_kwargs:
            # Whisper consumes encoder outputs through `encoder_outputs`, not
            # `inputs_embeds`. Single modality (audio) so execute_mm_encoder
            # preserves request order; use its return value directly.
            # No need to store in encoder_cache: cross-attention K/V are written
            # to the KV cache on the first step; decode steps use the cache.
            self.encoder_outputs = self.encoder_runner.execute_mm_encoder(mm_kwargs)
        else:
            # Decode steps: encoder K/V are in cross-attention KV cache.
            self.encoder_outputs = []
        return None
```
**EN:** This method returns or derives a value within `WhisperModelState`. Key calls include `prepare_mm_inputs`, `get`, `execute_mm_encoder`. It touches state such as `encoder_outputs`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`WhisperModelState`。 关键调用包括 `prepare_mm_inputs`, `get`, `execute_mm_encoder`。 它会读写 `encoder_outputs` 等状态。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `WhisperModelState.prepare_inputs` method / `WhisperModelState.prepare_inputs` 方法
```python
    def prepare_inputs(
        self, input_batch: InputBatch, req_states: RequestState
    ) -> dict[str, Any]:
        model_inputs = {"encoder_outputs": self.encoder_outputs}
        self.encoder_outputs = []
        return model_inputs
```
**EN:** This method prepares inputs and state within `WhisperModelState`. It touches state such as `encoder_outputs`.
**CN:** 该方法会准备输入与状态，其作用域位于`WhisperModelState`。 它会读写 `encoder_outputs` 等状态。

### `WhisperModelState.prepare_dummy_inputs` method / `WhisperModelState.prepare_dummy_inputs` 方法
```python
    def prepare_dummy_inputs(self, num_reqs: int, num_tokens: int) -> dict[str, Any]:
        return {"encoder_outputs": []}
```
**EN:** This method prepares inputs and state within `WhisperModelState`.
**CN:** 该方法会准备输入与状态，其作用域位于`WhisperModelState`。

### `WhisperModelState.prepare_attn` method / `WhisperModelState.prepare_attn` 方法
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
            num_reqs = input_batch.num_reqs_after_padding
            num_tokens = input_batch.num_tokens_after_padding
        else:
            num_reqs = input_batch.num_reqs
            num_tokens = input_batch.num_tokens
        whisper_attn_metadata = WhisperAttnMetadata(
            self._get_encoder_seq_lens(input_batch.req_ids, attn_groups, for_capture)
        )

        query_start_loc_cpu = torch.from_numpy(input_batch.query_start_loc_np)
        max_query_len = input_batch.num_scheduled_tokens.max().item()
        seq_lens_cpu_upper_bound = input_batch.seq_lens_cpu_upper_bound
        if for_capture:
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
            model_specific_attn_metadata=whisper_attn_metadata,
            for_cudagraph_capture=for_capture,
        )
        return attn_metadata
```
**EN:** This method prepares inputs and state within `WhisperModelState`. Key calls include `WhisperAttnMetadata`, `from_numpy`, `item`, `build_attn_metadata`, `_get_encoder_seq_lens`, `int`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会准备输入与状态，其作用域位于`WhisperModelState`。 关键调用包括 `WhisperAttnMetadata`, `from_numpy`, `item`, `build_attn_metadata`, `_get_encoder_seq_lens`, `int`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `WhisperModelState._get_encoder_seq_lens` method / `WhisperModelState._get_encoder_seq_lens` 方法
```python
    def _get_encoder_seq_lens(
        self,
        req_ids: list[str],
        attn_groups: list[list[AttentionGroup]],
        for_capture: bool,
    ) -> dict[int, tuple[torch.Tensor, np.ndarray]]:
        num_reqs = len(req_ids)
        encoder_seq_lens_np = np.zeros(num_reqs, dtype=np.int32)
        if not for_capture:
            # During normal execution, use actual encoder lengths.
            for i, req_id in enumerate(req_ids):
                mm_features = self.encoder_cache.mm_features.get(req_id, [])
                encoder_seq_lens_np[i] = sum(
                    feature.mm_position.get_num_embeds() for feature in mm_features
                )
        else:
            # During CUDA graph capture, use max encoder length so max_seqlen_k
            # is captured with the correct value for cross-attention.
            encoder_seq_lens_np[:] = self.max_encoder_len

        self.encoder_seq_lens_gpu[:num_reqs].copy_(
            torch.from_numpy(encoder_seq_lens_np), non_blocking=True
        )
        self.encoder_seq_lens_gpu[num_reqs:].fill_(0)
        encoder_seq_lens_gpu = self.encoder_seq_lens_gpu[:num_reqs]

        seq_lens_by_group: dict[int, tuple[torch.Tensor, np.ndarray]] = {}
        for kv_cache_group_idx, groups in enumerate(attn_groups):
            has_cross_attn = any(
                isinstance(attn_group.kv_cache_spec, CrossAttentionSpec)
                for attn_group in groups
            )
            if has_cross_attn:
                seq_lens_by_group[kv_cache_group_idx] = (
                    encoder_seq_lens_gpu,
                    encoder_seq_lens_np,
                )
        return seq_lens_by_group
```
**EN:** This method implements `_get_encoder_seq_lens` within `WhisperModelState`. Key calls include `len`, `zeros`, `copy_`, `fill_`, `enumerate`, `from_numpy`. The control flow contains 2 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_get_encoder_seq_lens`，其作用域位于`WhisperModelState`。 关键调用包括 `len`, `zeros`, `copy_`, `fill_`, `enumerate`, `from_numpy`。 控制流包含 2 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `WhisperAttnMetadata`: central class or interface in this module. / `WhisperAttnMetadata`：本模块中的核心类或接口。
- `WhisperModelState`: central class or interface in this module. / `WhisperModelState`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `typing`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.config.compilation`, `vllm.v1.kv_cache_interface`, `vllm.v1.worker.gpu.attn_utils`, `vllm.v1.worker.gpu.input_batch`, `vllm.v1.worker.gpu.mm.encoder_cache`, `vllm.v1.worker.gpu.mm.encoder_runner`, `vllm.v1.worker.gpu.model_states.interface`, `vllm.v1.worker.gpu.states`, `vllm.v1.worker.utils`
