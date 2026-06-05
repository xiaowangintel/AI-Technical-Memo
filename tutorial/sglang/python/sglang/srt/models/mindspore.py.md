# mindspore.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/mindspore.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the mindspore model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 mindspore 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 3-18: Module imports
```python
from __future__ import annotations

import logging
from typing import Any, Iterable, List, Optional, Tuple

import torch

from sglang.srt.distributed import (
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
)
from sglang.srt.layers.logits_processor import LogitsProcessorOutput
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.models.registry import import_model_classes
from sglang.srt.utils import is_npu
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、SGLang 运行时组件。

### Lines 20-20: Top-level assign
```python
_is_npu = is_npu()
```
**EN:** Defines or updates _is_npu, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_npu，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 22-26: Top-level if
```python
if _is_npu:
    import mindspore as ms
    import numpy as np
    import torch_npu
    from mindspore import Tensor, mint, mutable
```
**EN:** Applies conditional top-level logic so the module can adapt to optional dependencies, feature flags, or environment-specific behavior.
**CN:** 应用条件化的顶层逻辑，使模块能够适配可选依赖、功能开关或特定环境行为。

### Lines 28-28: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 31-41: Function `_get_arch_from_config`
```python
def _get_arch_from_config(config):
    mindspore_models = import_model_classes("sgl_mindspore.models")
    architectures = getattr(config, "architectures", [])
    if isinstance(architectures, str):
        architectures = [architectures]
    if not architectures:
        raise ValueError("No model architectures are specified")
    for arch in architectures:
        if arch in mindspore_models:
            return mindspore_models[arch]
    raise ValueError(f"Unsupported arch {architectures}")
```
**EN:** This function implements `_get_arch_from_config(config)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个函数实现了 `_get_arch_from_config(config)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 44-51: Function `tensor_torch2ms`
```python
def tensor_torch2ms(x: torch.Tensor):
    if x is None or not isinstance(x, torch.Tensor):
        return x

    # torch tensor -> dlpack -> mindspore tensor
    pt_dlpack = torch.utils.dlpack.to_dlpack(x)
    ms_tensor = ms.utils.dlpack.from_dlpack(pt_dlpack)
    return ms_tensor
```
**EN:** This function implements `tensor_torch2ms(x: ...)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `tensor_torch2ms(x: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 54-62: Function `tensor_ms2torch`
```python
def tensor_ms2torch(x: "ms.Tensor"):
    if x is None or not isinstance(x, ms.Tensor):
        return x

    # ms tensor -> dlpack -> torch tensor
    ms_dlpack = ms.utils.dlpack.to_dlpack(x)
    torch_tensor = torch.utils.dlpack.from_dlpack(ms_dlpack)
    torch_npu.npu.synchronize()
    return torch_tensor
```
**EN:** This function implements `tensor_ms2torch(x: ...)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `tensor_ms2torch(x: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 66-73: Class `LowerTriangularMask` overview
```python
class LowerTriangularMask:
    r"""
    Provide Infer model attention mask.
    Args:
        dtype (ms dtype): The compute type of Infer model.
        max_model_len (int): The max model length of Infer model.
    """
```
**EN:** Defines `LowerTriangularMask` as a reusable runtime type derived from no explicit base class. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `LowerTriangularMask`，其继承关系为 no explicit base class。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 74-100: Method `LowerTriangularMask.__init__`
```python
    def __init__(self, dtype, max_model_len, decode_mask_coeff=-10000.0):
        self.dtype = dtype
        self.max_model_len = max_model_len
        self.cached_mask_len = 8 * 1024
        self.decode_mask_coeff = decode_mask_coeff

        prefill_mask_coeff = 1.0 if self.dtype == ms.bfloat16 else -10000.0
        self.prefill_mask = Tensor(
            np.triu(np.ones(shape=(128, 128), dtype=np.float16), k=1)
            * prefill_mask_coeff,
            dtype=self.dtype,
        )

        self.hard_mask = mint.zeros((1, 1), dtype=dtype)
        self.decode_mask = (
            Tensor(
                np.triu(
                    np.ones(
                        shape=(self.cached_mask_len, self.cached_mask_len),
                        dtype=np.int8,
                    ),
                    k=1,
                ),
                dtype=self.dtype,
            )
            * self.decode_mask_coeff
        )
```
**EN:** This method implements `__init__(dtype, max_model_len, decode_mask_coeff=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(dtype, max_model_len, decode_mask_coeff=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 102-146: Method `LowerTriangularMask.create_mask`
```python
    def create_mask(self, query_lens_np, seq_lens_np):
        """
        when query_lens_np = [3], seq_lens_np = [6], decode_mask_coeff = 1
        init attention mask
        0 0 0 0 0 0
        0 0 0 0 0 0
        0 0 0 0 0 0
        """
        max_seq_len = seq_lens_np.max().item()
        total_q_len = query_lens_np.sum().item()
        attention_mask = mint.zeros((total_q_len, max_seq_len), dtype=self.dtype)

        req_num = query_lens_np.shape[0]
        current_row = 0
        for i in range(req_num):
            q_len = query_lens_np[i].item()
            current_row += q_len
            # skip row when q_len <= 1, to decrease execute time
            if q_len <= 1:
                continue
            seq_len = seq_lens_np[i].item()
            context_len = seq_len - q_len
            """
            set the right half to 1
            0 0 0 1 1 1
            0 0 0 1 1 1
            0 0 0 1 1 1
            """
            attention_mask[current_row - q_len : current_row, context_len:] = (
                self.decode_mask_coeff
            )
            """
            set the lower triangle of the right half to 0
            0 0 0 0 1 1
            0 0 0 0 0 1
            0 0 0 0 0 0
            """
            right_tensor = attention_mask[
                current_row - q_len : current_row, context_len:seq_len
            ]
# ... truncated for brevity ...
```
**EN:** This method implements `create_mask(query_lens_np, seq_lens_np)` and when query_lens_np = [3], seq_lens_np = [6], decode_mask_coeff = 1.
**CN:** 这个方法实现了 `create_mask(query_lens_np, seq_lens_np)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 148-166: Method `LowerTriangularMask.gen_attention_mask`
```python
    def gen_attention_mask(
        self,
        is_prefill: bool,
        position_ids: "ms.Tensor",
        query_lens_np: np.ndarray,
        seq_lens_np: np.ndarray,
    ):
        max_query_len = query_lens_np.max()
        max_seq_len = seq_lens_np.max()
        if is_prefill:
            attention_mask = self.prefill_mask
        elif max_query_len > 1:
            if max_seq_len <= self.cached_mask_len:
                attention_mask = mint.index_select(self.decode_mask, 0, position_ids)
            else:
                attention_mask = self.create_mask(query_lens_np, seq_lens_np)
        else:
            attention_mask = self.hard_mask
        return attention_mask
```
**EN:** This method implements `gen_attention_mask(is_prefill: ..., position_ids: ..., query_lens_np: ..., seq_lens_np: ...)` and implements attention-related tensor preparation or execution.
**CN:** 这个方法实现了 `gen_attention_mask(is_prefill: ..., position_ids: ..., query_lens_np: ..., seq_lens_np: ...)`，其作用是实现与注意力相关的张量准备或执行逻辑。

### Lines 169-169: Class `MindSporeForCausalLM` overview
```python
class MindSporeForCausalLM(torch.nn.Module):
```
**EN:** Defines `MindSporeForCausalLM` as a reusable runtime type derived from torch.nn.Module. The class groups 15 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MindSporeForCausalLM`，其继承关系为 torch.nn.Module。这个类组织了 15 个方法，用于实现模型相关行为。

### Lines 170-198: Method `MindSporeForCausalLM.__init__`
```python
    def __init__(
        self,
        config: Any,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config

        ms.set_context(graph_kernel_flags="--disable_pass=gather_pre_rms_norm_fusion")
        ms.set_kernel_launch_capture(False)

        logger.info(
            "MindSporeForCausalLM tp size %d tp rank %d",
            get_tensor_model_parallel_world_size(),
            get_tensor_model_parallel_rank(),
        )
        if get_tensor_model_parallel_world_size() not in (1, 2, 4, 8):
            # MatMulAllReduce only support tp size in (1, 2, 4, 8)
            ms.set_context(graph_kernel_flags="--disable_pass=MatMulAllReduce")

        arch = self.get_arch(self.config)
        self.model = arch(config=config, quant_config=quant_config)

        self.causal_mask = LowerTriangularMask(
            self.config.param_dtype, self.config.max_position_embeddings
        )
        self.key_cache = []
        self.value_cache = []
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 200-204: Method `MindSporeForCausalLM.hot_token_id`
```python
    @property
    def hot_token_id(self):
        if hasattr(self.model, "hot_token_id"):
            return tensor_ms2torch(self.model.hot_token_id)
        return None
```
**EN:** This method implements `hot_token_id()` and implements a focused helper that supports the surrounding runtime flow inside `MindSporeForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `hot_token_id()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MindSporeForCausalLM` 内部调用 装饰器：property。

### Lines 206-207: Method `MindSporeForCausalLM.get_arch`
```python
    def get_arch(self, config):
        return _get_arch_from_config(config)
```
**EN:** This method implements `get_arch(config)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_arch(config)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 209-211: Method `MindSporeForCausalLM.use_mla`
```python
    @property
    def use_mla(self):
        return self.config.architectures[0] in ("DeepseekV3ForCausalLM")
```
**EN:** This method implements `use_mla()` and implements a focused helper that supports the surrounding runtime flow inside `MindSporeForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `use_mla()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MindSporeForCausalLM` 内部调用 装饰器：property。

### Lines 213-218: Method `MindSporeForCausalLM.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        self.model.load_weights(weights)
        for _, cell in self.model.cells_and_names():
            quant_method = getattr(cell, "quant_method", None)
            if quant_method is not None:
                quant_method.process_weights_after_loading(cell)
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 220-243: Method `MindSporeForCausalLM.get_kvcache`
```python
    def get_kvcache(self, forward_batch: ForwardBatch):
        def prepare_cache(cache_list, is_key_cache):
            for i in range(self.config.num_hidden_layers):
                if is_key_cache:
                    cache = forward_batch.token_to_kv_pool.get_key_buffer(i)
                else:
                    cache = forward_batch.token_to_kv_pool.get_value_buffer(i)
                cache_ms = tensor_torch2ms(cache)
                if self.use_mla and cache_ms.ndim == 3:
                    cache_ms = mint.unsqueeze(cache_ms, 2)
                cache_list.append(cache_ms)

        if self.use_mla:
            if not self.key_cache:
                prepare_cache(self.key_cache, is_key_cache=True)
            return mutable(self.key_cache)

        if self.key_cache and self.value_cache:
            return mutable(self.key_cache), mutable(self.value_cache)

        prepare_cache(self.key_cache, is_key_cache=True)
        prepare_cache(self.value_cache, is_key_cache=False)

        return mutable(self.key_cache), mutable(self.value_cache)
```
**EN:** This method implements `get_kvcache(forward_batch: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_kvcache(forward_batch: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 245-259: Method `MindSporeForCausalLM._is_prefill`
```python
    def _is_prefill(self, forward_batch: ForwardBatch):
        # Different processing for the mindspore attention operator
        # Without any prefix cache => Use FlashAttentionScore
        # With cache => Use PagedAttention, no matter the query length is 1 or not
        is_prefill = (
            forward_batch.forward_mode.is_extend()
            and not forward_batch.forward_mode.is_draft_extend_v2()
            and not forward_batch.forward_mode.is_draft_extend()
            and not forward_batch.forward_mode.is_target_verify()
        )
        if forward_batch.extend_prefix_lens is not None:
            is_prefill = (
                is_prefill and forward_batch.extend_prefix_lens.sum().item() == 0
            )
        return is_prefill
```
**EN:** This method implements `_is_prefill(forward_batch: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MindSporeForCausalLM`.
**CN:** 这个方法实现了 `_is_prefill(forward_batch: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MindSporeForCausalLM` 内部调用。

### Lines 261-308: Method `MindSporeForCausalLM.prepare_inputs`
```python
    def prepare_inputs(self, input_ids, positions, forward_batch):
        if self.use_mla:
            key_cache = self.get_kvcache(forward_batch)
        else:
            key_cache, value_cache = self.get_kvcache(forward_batch)

        is_prefill = self._is_prefill(forward_batch)
        batch_valid_length = forward_batch.seq_lens.cpu().numpy()
        if forward_batch.forward_mode.is_target_verify():
            batch_valid_length += forward_batch.spec_info.num_tokens_per_req
        if forward_batch.extend_seq_lens is not None:
            q_seq_lens = forward_batch.extend_seq_lens.cpu().numpy()
        else:
            q_seq_lens = np.ones([forward_batch.batch_size], dtype=np.int32)
            if forward_batch.forward_mode.is_target_verify():
                q_seq_lens = q_seq_lens * forward_batch.spec_info.num_tokens_per_req

        page_size = forward_batch.token_to_kv_pool.page_size
        block_tables = tensor_torch2ms(
            (
                forward_batch.req_to_token_pool.req_to_token[
                    forward_batch.req_pool_indices, : batch_valid_length.max()
                ][:, ::page_size]
                // page_size
            )
        ).to(ms.int32)

        model_inputs = {}
        model_inputs["input_ids"] = tensor_torch2ms(input_ids).to(ms.int32)
        model_inputs["batch_valid_length"] = ms.Tensor(
            batch_valid_length, dtype=ms.int32
        )
        model_inputs["position_ids"] = tensor_torch2ms(positions)
        model_inputs["q_seq_lens"] = ms.Tensor(q_seq_lens, dtype=ms.int32)
        model_inputs["attention_mask"] = self.causal_mask.gen_attention_mask(
            is_prefill, model_inputs["position_ids"], q_seq_lens, batch_valid_length
        ).contiguous()
        model_inputs["out_cache_loc"] = tensor_torch2ms(forward_batch.out_cache_loc).to(
            ms.int32
        )
# ... truncated for brevity ...
```
**EN:** This method implements `prepare_inputs(input_ids, positions, forward_batch)` and prepares intermediate tensors, masks, or metadata before the main compute path.
**CN:** 这个方法实现了 `prepare_inputs(input_ids, positions, forward_batch)`，其作用是在主计算路径前准备中间张量、掩码或元数据。

### Lines 310-332: Method `MindSporeForCausalLM.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> "ms.Tensor":
        # prepare base inputs
        model_inputs = self.prepare_inputs(input_ids, positions, forward_batch)
        # prepare model inputs
        model_inputs = self.model.prepare_inputs(forward_batch, model_inputs)

        # Used by speculative decoding (EAGLE)
        if self.model.capture_aux_hidden_states:
            logits, hidden_states = self.model(**model_inputs)
        else:
            logits = self.model(**model_inputs)
            hidden_states = None

        logits_result = LogitsProcessorOutput(
            next_token_logits=tensor_ms2torch(logits),
            hidden_states=tensor_ms2torch(hidden_states),
        )
        return logits_result
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 334-343: Method `MindSporeForCausalLM.get_model_config_for_expert_location`
```python
    @classmethod
    def get_model_config_for_expert_location(cls, config):
        try:
            arch_cls = _get_arch_from_config(config)
            method = getattr(arch_cls, "get_model_config_for_expert_location", None)
            if method is None:
                return None
            return method(config)
        except Exception:
            return None
```
**EN:** This method implements `get_model_config_for_expert_location(config)` and returns a cached or derived object used elsewhere in the pipeline Decorators: classmethod.
**CN:** 这个方法实现了 `get_model_config_for_expert_location(config)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果 装饰器：classmethod。

### Lines 346-348: Method `MindSporeForCausalLM.get_embed_and_head`
```python
    def get_embed_and_head(self):
        embed, head = self.model.get_embed_and_head()
        return tensor_ms2torch(embed), tensor_ms2torch(head)
```
**EN:** This method implements `get_embed_and_head()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embed_and_head()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 350-351: Method `MindSporeForCausalLM.set_embed_and_head`
```python
    def set_embed_and_head(self, embed, head):
        self.model.set_embed_and_head(tensor_torch2ms(embed), tensor_torch2ms(head))
```
**EN:** This method implements `set_embed_and_head(embed, head)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `set_embed_and_head(embed, head)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 353-354: Method `MindSporeForCausalLM.get_embed`
```python
    def get_embed(self):
        return tensor_ms2torch(self.model.get_embed())
```
**EN:** This method implements `get_embed()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embed()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 356-357: Method `MindSporeForCausalLM.set_embed`
```python
    def set_embed(self, embed):
        self.model.set_embed(tensor_torch2ms(embed))
```
**EN:** This method implements `set_embed(embed)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `set_embed(embed)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 359-360: Method `MindSporeForCausalLM.set_eagle3_layers_to_capture`
```python
    def set_eagle3_layers_to_capture(self, layer_ids: Optional[List[int]] = None):
        self.model.set_eagle3_layers_to_capture(layer_ids)
```
**EN:** This method implements `set_eagle3_layers_to_capture(layer_ids: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `MindSporeForCausalLM`.
**CN:** 这个方法实现了 `set_eagle3_layers_to_capture(layer_ids: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MindSporeForCausalLM` 内部调用。

### Lines 363-363: Top-level assign
```python
EntryClass = [MindSporeForCausalLM]
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `__future__.annotations`
- `logging`
- `typing.Any`
- `typing.Iterable`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `sglang.srt.distributed.get_tensor_model_parallel_rank`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.layers.logits_processor.LogitsProcessorOutput`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.models.registry.import_model_classes`
- `sglang.srt.utils.is_npu`
- `mindspore`
- `numpy`
- `torch_npu`
- `mindspore.Tensor`
- `mindspore.mint`
- `mindspore.mutable`
