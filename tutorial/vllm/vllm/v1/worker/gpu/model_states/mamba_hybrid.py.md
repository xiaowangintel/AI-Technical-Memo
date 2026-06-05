# mamba_hybrid.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/model_states/mamba_hybrid.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `MambaHybridAttnMetadata`, `MambaHybridModelState` for the V1 `worker/gpu/model_states` subsystem. / 为 V1 的 `worker/gpu/model_states` 子系统实现 `MambaHybridAttnMetadata`, `MambaHybridModelState`。

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
from vllm.v1.attention.backends.gdn_attn import GDNAttentionMetadataBuilder
from vllm.v1.attention.backends.mamba2_attn import Mamba2AttentionMetadataBuilder
from vllm.v1.kv_cache_interface import KVCacheConfig
from vllm.v1.worker.gpu.attn_utils import build_attn_metadata
from vllm.v1.worker.gpu.input_batch import InputBatch
from vllm.v1.worker.gpu.mm.encoder_cache import EncoderCache
from vllm.v1.worker.gpu.model_states.default import DefaultModelState
from vllm.v1.worker.gpu.model_states.interface import ModelSpecificAttnMetadata
from vllm.v1.worker.utils import AttentionGroup
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, and internal vLLM modules such as `vllm.config`, `vllm.config.compilation`, `vllm.v1.attention.backends.gdn_attn`, `vllm.v1.attention.backends.mamba2_attn`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖，以及 `vllm.config`, `vllm.config.compilation`, `vllm.v1.attention.backends.gdn_attn`, `vllm.v1.attention.backends.mamba2_attn` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `MambaHybridAttnMetadata` class / `MambaHybridAttnMetadata` 类
```python
@dataclass
class MambaHybridAttnMetadata(ModelSpecificAttnMetadata):
    is_prefilling: torch.Tensor
    num_accepted_tokens: torch.Tensor | None = None
    num_decode_draft_tokens_cpu: torch.Tensor | None = None
```
**EN:** Uses `@dataclass` to package related state for `MambaHybridAttnMetadata`. Typical fields include `is_prefilling`, `num_accepted_tokens`, `num_decode_draft_tokens_cpu`.
**CN:** `MambaHybridAttnMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `is_prefilling`, `num_accepted_tokens`, `num_decode_draft_tokens_cpu`。

### `MambaHybridAttnMetadata.get_extra_common_attn_kwargs` method / `MambaHybridAttnMetadata.get_extra_common_attn_kwargs` 方法
```python
    def get_extra_common_attn_kwargs(
        self,
        kv_cache_group_id: int,
        num_reqs: int,
    ) -> dict[str, Any]:
        return {"is_prefilling": self.is_prefilling[:num_reqs]}
```
**EN:** This method returns or derives a value within `MambaHybridAttnMetadata`.
**CN:** 该方法会返回或推导一个值，其作用域位于`MambaHybridAttnMetadata`。

### `MambaHybridAttnMetadata.get_extra_attn_kwargs` method / `MambaHybridAttnMetadata.get_extra_attn_kwargs` 方法
```python
    def get_extra_attn_kwargs(
        self,
        attn_metadata_builder: Any,
        num_reqs: int,
    ) -> dict[str, Any]:
        if not isinstance(
            attn_metadata_builder,
            (Mamba2AttentionMetadataBuilder, GDNAttentionMetadataBuilder),
        ):
            return {}
        return {
            "num_accepted_tokens": None
            if self.num_accepted_tokens is None
            else self.num_accepted_tokens[:num_reqs],
            "num_decode_draft_tokens_cpu": None
            if self.num_decode_draft_tokens_cpu is None
            else self.num_decode_draft_tokens_cpu[:num_reqs],
        }
```
**EN:** This method returns or derives a value within `MambaHybridAttnMetadata`. Key calls include `isinstance`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`MambaHybridAttnMetadata`。 关键调用包括 `isinstance`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MambaHybridModelState` class / `MambaHybridModelState` 类
```python
class MambaHybridModelState(DefaultModelState):
    """Model state for hybrid attention + Mamba / linear-attention models."""
```
**EN:** Introduces the `MambaHybridModelState` class on top of `DefaultModelState`. Core methods include `__init__`, `prepare_attn`, `postprocess_state`. Docstring signal: Model state for hybrid attention + Mamba / linear-attention models.
**CN:** 这里定义 `MambaHybridModelState` 类，其基类包括 `DefaultModelState`。核心方法包括 `__init__`, `prepare_attn`, `postprocess_state`。

### `MambaHybridModelState.__init__` method / `MambaHybridModelState.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        model: nn.Module,
        encoder_cache: EncoderCache | None,
        device: torch.device,
    ) -> None:
        super().__init__(vllm_config, model, encoder_cache, device)
        self.num_accepted_tokens_gpu = torch.ones(
            self.max_num_reqs, dtype=torch.int32, device=self.device
        )
```
**EN:** This method initializes the object state within `MambaHybridModelState`. Key calls include `__init__`, `ones`, `super`. It touches state such as `num_accepted_tokens_gpu`.
**CN:** 该方法会初始化对象状态，其作用域位于`MambaHybridModelState`。 关键调用包括 `__init__`, `ones`, `super`。 它会读写 `num_accepted_tokens_gpu` 等状态。

### `MambaHybridModelState.prepare_attn` method / `MambaHybridModelState.prepare_attn` 方法
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
        query_start_loc_cpu = torch.from_numpy(input_batch.query_start_loc_np)
        max_query_len = input_batch.num_scheduled_tokens.max().item()

        is_prefilling = torch.zeros(num_reqs, dtype=torch.bool, device="cpu")
        is_prefilling[: input_batch.num_reqs] = torch.from_numpy(
            input_batch.is_prefilling_np
        )
        # During CUDAGraph capture, num_decode_draft_tokens_cpu and num_accepted_tokens
        # are created by attn_metadata_builder.build_for_cudagraph_capture, so we only
        # compute them during actual (non-capture) forward execution.
        num_accepted_tokens = None
        num_decode_draft_tokens_cpu = None
        if not for_capture:
            num_accepted_tokens = self.num_accepted_tokens_gpu.new_ones(num_reqs)
            num_accepted_tokens[: input_batch.num_reqs] = self.num_accepted_tokens_gpu[
                input_batch.idx_mapping
            ]

            # GDN uses >= 0 to select spec-decode rows, so non-decode rows
            # need the -1 sentinel rather than a raw zero draft count.
            num_decode_draft_tokens_np = np.full(num_reqs, -1, dtype=np.int32)
            if input_batch.num_draft_tokens_per_req is not None:
                spec_decode_mask = (
                    input_batch.num_draft_tokens_per_req > 0
                ) & ~input_batch.is_prefilling_np
                num_decode_draft_tokens_np[: input_batch.num_reqs] = np.where(
                    spec_decode_mask,
                    input_batch.num_draft_tokens_per_req,
                    -1,
                )
            num_decode_draft_tokens_cpu = torch.from_numpy(num_decode_draft_tokens_np)

        mamba_attn_metadata = MambaHybridAttnMetadata(
            is_prefilling=is_prefilling,
            num_accepted_tokens=num_accepted_tokens,
            num_decode_draft_tokens_cpu=num_decode_draft_tokens_cpu,
        )
        return build_attn_metadata(
            attn_groups=attn_groups,
            num_reqs=num_reqs,
            num_tokens=num_tokens,
            query_start_loc_gpu=input_batch.query_start_loc,
            query_start_loc_cpu=query_start_loc_cpu,
            max_query_len=max_query_len,
            seq_lens=input_batch.seq_lens,
            max_seq_len=self.max_model_len,
            block_tables=block_tables,
            slot_mappings=slot_mappings,
            kv_cache_config=kv_cache_config,
            dcp_local_seq_lens=input_batch.dcp_local_seq_lens,
            model_specific_attn_metadata=mamba_attn_metadata,
            for_cudagraph_capture=for_capture,
        )
```
**EN:** This method prepares inputs and state within `MambaHybridModelState`. Key calls include `from_numpy`, `item`, `zeros`, `MambaHybridAttnMetadata`, `build_attn_metadata`, `new_ones`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会准备输入与状态，其作用域位于`MambaHybridModelState`。 关键调用包括 `from_numpy`, `item`, `zeros`, `MambaHybridAttnMetadata`, `build_attn_metadata`, `new_ones`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MambaHybridModelState.postprocess_state` method / `MambaHybridModelState.postprocess_state` 方法
```python
    def postprocess_state(
        self,
        input_batch: InputBatch,
        num_sampled: torch.Tensor,
    ) -> None:
        # Chunked prefill does not sample a token, so num_sampled can be 0.
        # Mamba treats num_accepted_tokens=1 as the neutral non-spec value.
        self.num_accepted_tokens_gpu[input_batch.idx_mapping] = torch.clamp(
            num_sampled, min=1
        )
```
**EN:** This method implements `postprocess_state` within `MambaHybridModelState`. Key calls include `clamp`. It touches state such as `num_accepted_tokens_gpu`.
**CN:** 该方法会实现 `postprocess_state`，其作用域位于`MambaHybridModelState`。 关键调用包括 `clamp`。 它会读写 `num_accepted_tokens_gpu` 等状态。

## Key Concepts / 关键概念
- `MambaHybridAttnMetadata`: central class or interface in this module. / `MambaHybridAttnMetadata`：本模块中的核心类或接口。
- `MambaHybridModelState`: central class or interface in this module. / `MambaHybridModelState`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `typing`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.config.compilation`, `vllm.v1.attention.backends.gdn_attn`, `vllm.v1.attention.backends.mamba2_attn`, `vllm.v1.kv_cache_interface`, `vllm.v1.worker.gpu.attn_utils`, `vllm.v1.worker.gpu.input_batch`, `vllm.v1.worker.gpu.mm.encoder_cache`, `vllm.v1.worker.gpu.model_states.default`, `vllm.v1.worker.gpu.model_states.interface`, `vllm.v1.worker.utils`
