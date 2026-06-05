# encoder_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/mm/encoder_runner.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `EncoderRunner` for the V1 `worker/gpu/mm` subsystem. / 为 V1 的 `worker/gpu/mm` 子系统实现 `EncoderRunner`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import numpy as np
import torch

from vllm.model_executor.models.interfaces import SupportsMultiModal
from vllm.multimodal.inputs import MultiModalKwargsItem
from vllm.multimodal.utils import group_and_batch_mm_kwargs
from vllm.v1.worker.gpu.mm.encoder_cache import EncoderCache
from vllm.v1.worker.utils import sanity_check_mm_encoder_outputs
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, and internal vLLM modules such as `vllm.model_executor.models.interfaces`, `vllm.multimodal.inputs`, `vllm.multimodal.utils`, `vllm.v1.worker.gpu.mm.encoder_cache`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖，以及 `vllm.model_executor.models.interfaces`, `vllm.multimodal.inputs`, `vllm.multimodal.utils`, `vllm.v1.worker.gpu.mm.encoder_cache` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `EncoderRunner` class / `EncoderRunner` 类
```python
class EncoderRunner:
```
**EN:** Introduces the `EncoderRunner` class. Core methods include `__init__`, `prepare_mm_inputs`, `execute_mm_encoder`, `gather_mm_embeddings`, `get_inputs_embeds`.
**CN:** 这里定义 `EncoderRunner` 类。核心方法包括 `__init__`, `prepare_mm_inputs`, `execute_mm_encoder`, `gather_mm_embeddings`, `get_inputs_embeds`。

### `EncoderRunner.__init__` method / `EncoderRunner.__init__` 方法
```python
    def __init__(
        self,
        model: SupportsMultiModal,
        max_num_tokens: int,
        hidden_size: int,
        encoder_cache: EncoderCache,
        dtype: torch.dtype,
        device: torch.device,
    ):
        self.model = model
        self.max_num_tokens = max_num_tokens
        self.hidden_size = hidden_size
        self.encoder_cache = encoder_cache
        self.dtype = dtype
        self.device = device

        self.inputs_embeds = torch.zeros(
            max_num_tokens, hidden_size, dtype=dtype, device=device
        )
```
**EN:** This method initializes the object state within `EncoderRunner`. Key calls include `zeros`. It touches state such as `model`, `max_num_tokens`, `hidden_size`, `encoder_cache`, `dtype`, `device`, `inputs_embeds`.
**CN:** 该方法会初始化对象状态，其作用域位于`EncoderRunner`。 关键调用包括 `zeros`。 它会读写 `model`, `max_num_tokens`, `hidden_size`, `encoder_cache`, `dtype`, `device`, `inputs_embeds` 等状态。

### `EncoderRunner.prepare_mm_inputs` method / `EncoderRunner.prepare_mm_inputs` 方法
```python
    def prepare_mm_inputs(
        self, scheduled_encoder_inputs: dict[str, list[int]]
    ) -> tuple[list[str], list[tuple[str, MultiModalKwargsItem]]]:
        mm_hashes: list[str] = []
        mm_kwargs: list[tuple[str, MultiModalKwargsItem]] = []
        for req_id, encoder_input_ids in scheduled_encoder_inputs.items():
            mm_features = self.encoder_cache.mm_features[req_id]
            for mm_input_id in encoder_input_ids:
                mm_feature = mm_features[mm_input_id]
                if mm_feature.data is None:
                    continue
                mm_hashes.append(mm_feature.identifier)
                mm_kwargs.append((mm_feature.modality, mm_feature.data))

        return mm_hashes, mm_kwargs
```
**EN:** This method prepares inputs and state within `EncoderRunner`. Key calls include `items`, `append`. The control flow contains 1 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会准备输入与状态，其作用域位于`EncoderRunner`。 关键调用包括 `items`, `append`。 控制流包含 1 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `EncoderRunner.execute_mm_encoder` method / `EncoderRunner.execute_mm_encoder` 方法
```python
    @torch.inference_mode()
    def execute_mm_encoder(
        self,
        mm_kwargs: list[tuple[str, MultiModalKwargsItem]],
    ) -> list[torch.Tensor]:
        encoder_outputs: list[torch.Tensor] = []
        for modality, num_items, mm_kwargs_batch in group_and_batch_mm_kwargs(
            mm_kwargs, device=self.device, pin_memory=False
        ):
            batch_outputs = self.model.embed_multimodal(**mm_kwargs_batch)
            sanity_check_mm_encoder_outputs(batch_outputs, expected_num_items=num_items)
            encoder_outputs.extend(batch_outputs)
        return encoder_outputs
```
**EN:** This method executes the main operation within `EncoderRunner`. Key calls include `inference_mode`, `group_and_batch_mm_kwargs`, `embed_multimodal`, `sanity_check_mm_encoder_outputs`, `extend`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会执行主要操作，其作用域位于`EncoderRunner`。 关键调用包括 `inference_mode`, `group_and_batch_mm_kwargs`, `embed_multimodal`, `sanity_check_mm_encoder_outputs`, `extend`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `EncoderRunner.gather_mm_embeddings` method / `EncoderRunner.gather_mm_embeddings` 方法
```python
    def gather_mm_embeddings(
        self,
        req_ids: list[str],
        total_num_scheduled_tokens: int,
        num_scheduled_tokens: np.ndarray,
        query_start_loc: np.ndarray,
        prefill_lens: np.ndarray,
        computed_prefill_lens: np.ndarray,
    ) -> tuple[list[torch.Tensor], torch.Tensor]:
        is_prefilling = (computed_prefill_lens < prefill_lens).tolist()
        all_decode = not any(is_prefilling)
        if all_decode:
            # All decode requests, so no need to gather any embeddings.
            return [], torch.zeros(
                total_num_scheduled_tokens, dtype=torch.bool, device=self.device
            )

        query_start = computed_prefill_lens.tolist()
        query_end = (computed_prefill_lens + num_scheduled_tokens).tolist()

        mm_embeds: list[torch.Tensor] = []
        is_mm_embed = torch.zeros(
            total_num_scheduled_tokens, dtype=torch.bool, device="cpu"
        )
        for i, req_id in enumerate(req_ids):
            if not is_prefilling[i]:
                # OPTIMIZATION: Skip decode requests.
                continue

            mm_features = self.encoder_cache.mm_features[req_id]
            for mm_feature in mm_features:
                pos_info = mm_feature.mm_position
                start_pos = pos_info.offset
                num_encoder_tokens = pos_info.length

                if start_pos >= query_end[i]:
                    # The encoder output is not needed in this step.
                    break
                if start_pos + num_encoder_tokens <= query_start[i]:
                    # The encoder output is already processed and stored
                    # in the decoder's KV cache.
                    continue

                start_idx = max(query_start[i] - start_pos, 0)
                end_idx = min(query_end[i] - start_pos, num_encoder_tokens)
                assert start_idx < end_idx
                curr_embeds_start, curr_embeds_end = (
                    pos_info.get_embeds_indices_in_range(start_idx, end_idx)
                )
                # If there are no embeddings in the current range, we skip
                # gathering the embeddings.
                if curr_embeds_start == curr_embeds_end:
                    continue

                mm_hash = mm_feature.identifier
                encoder_output = self.encoder_cache.encoder_outputs.get(mm_hash, None)
                assert encoder_output is not None, f"Encoder cache miss for {mm_hash}."

                if (is_embed := pos_info.is_embed) is not None:
                    is_embed = is_embed[start_idx:end_idx]
                    mm_embeds_item = encoder_output[curr_embeds_start:curr_embeds_end]
                else:
                    mm_embeds_item = encoder_output[start_idx:end_idx]

                req_start_pos = query_start_loc[i] + start_pos - query_start[i]
                is_mm_embed[req_start_pos + start_idx : req_start_pos + end_idx] = (
                    True if is_embed is None else is_embed
                )
                mm_embeds.append(mm_embeds_item)

        return mm_embeds, is_mm_embed
```
**EN:** This method implements `gather_mm_embeddings` within `EncoderRunner`. Key calls include `tolist`, `zeros`, `enumerate`, `any`, `max`, `min`. The control flow contains 7 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `gather_mm_embeddings`，其作用域位于`EncoderRunner`。 关键调用包括 `tolist`, `zeros`, `enumerate`, `any`, `max`, `min`。 控制流包含 7 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `EncoderRunner.get_inputs_embeds` method / `EncoderRunner.get_inputs_embeds` 方法
```python
    @torch.inference_mode()
    def get_inputs_embeds(
        self,
        input_ids: torch.Tensor,
        mm_embeds: list[torch.Tensor],
        is_mm_embed: torch.Tensor,
    ) -> torch.Tensor:
        x = self.model.embed_input_ids(
            input_ids, multimodal_embeddings=mm_embeds, is_multimodal=is_mm_embed
        )
        # Copy to the pre-allocated buffer for CUDA graphs.
        self.inputs_embeds[: x.shape[0]] = x
        return self.inputs_embeds
```
**EN:** This method returns or derives a value within `EncoderRunner`. Key calls include `inference_mode`, `embed_input_ids`. It touches state such as `inputs_embeds`.
**CN:** 该方法会返回或推导一个值，其作用域位于`EncoderRunner`。 关键调用包括 `inference_mode`, `embed_input_ids`。 它会读写 `inputs_embeds` 等状态。

## Key Concepts / 关键概念
- `EncoderRunner`: central class or interface in this module. / `EncoderRunner`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.model_executor.models.interfaces`, `vllm.multimodal.inputs`, `vllm.multimodal.utils`, `vllm.v1.worker.gpu.mm.encoder_cache`, `vllm.v1.worker.utils`
