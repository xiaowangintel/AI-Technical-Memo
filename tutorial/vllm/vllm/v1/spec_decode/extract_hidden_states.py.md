# extract_hidden_states.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/spec_decode/extract_hidden_states.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `ExtractHiddenStatesProposer` for the V1 `spec_decode` subsystem. / 为 V1 的 `spec_decode` 子系统实现 `ExtractHiddenStatesProposer`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from __future__ import annotations

from typing import TYPE_CHECKING

import torch
import torch.nn as nn

from vllm.config import CUDAGraphMode, VllmConfig, get_layers_from_vllm_config
from vllm.forward_context import set_forward_context
from vllm.model_executor.layers.attention_layer_base import AttentionLayerBase
from vllm.model_executor.model_loader import get_model
from vllm.v1.attention.backend import AttentionMetadataBuilder, CommonAttentionMetadata
from vllm.v1.cudagraph_dispatcher import CudagraphDispatcher
from vllm.v1.worker.dp_utils import coordinate_batch_across_dp
from vllm.v1.worker.gpu_input_batch import CachedRequestState, InputBatch

if TYPE_CHECKING:
    from vllm.v1.kv_cache_interface import KVCacheConfig

PADDING_SLOT_ID = -1
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `PADDING_SLOT_ID`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `PADDING_SLOT_ID`。

### `ExtractHiddenStatesProposer` class / `ExtractHiddenStatesProposer` 类
```python
class ExtractHiddenStatesProposer:
```
**EN:** Introduces the `ExtractHiddenStatesProposer` class. Core methods include `__init__`, `propose`, `_get_slot_mapping`, `_determine_batch_execution_and_padding`, `initialize_cudagraph_keys`, `dummy_run`.
**CN:** 这里定义 `ExtractHiddenStatesProposer` 类。核心方法包括 `__init__`, `propose`, `_get_slot_mapping`, `_determine_batch_execution_and_padding`, `initialize_cudagraph_keys`, `dummy_run`。

### `ExtractHiddenStatesProposer.__init__` method / `ExtractHiddenStatesProposer.__init__` 方法
```python
    def __init__(self, vllm_config: VllmConfig, device):
        assert vllm_config.speculative_config is not None

        assert vllm_config.speculative_config.num_speculative_tokens == 1
        if vllm_config.speculative_config.disable_padded_drafter_batch:
            raise ValueError(
                "disable_padded_drafter_batch is not supported with "
                "extract_hidden_states method"
            )
        self.vllm_config = vllm_config
        self.device = device
        self.dtype = vllm_config.model_config.dtype
        self.dp_rank = vllm_config.parallel_config.data_parallel_rank

        # Model and attention layer tracking (initialized in load_model)
        self.model: nn.Module | None = None
        self.attn_layer_names: list[str] = []
        self.attn_metadata_builder: AttentionMetadataBuilder | None = None
        self.kv_cache_gid: int = -1

        # Maximum number of tokens for buffers
        max_batch_size = vllm_config.scheduler_config.max_num_seqs
        self.max_num_tokens = (
            vllm_config.scheduler_config.max_num_batched_tokens + max_batch_size
        )

        self.hf_config = vllm_config.speculative_config.draft_model_config.hf_config
        layer_ids = getattr(self.hf_config, "eagle_aux_hidden_state_layer_ids", None)
        if not layer_ids:
            raise ValueError(
                "eagle_aux_hidden_state_layer_ids must be set in the draft "
                "model config for extract_hidden_states method"
            )
        self.num_hidden_states = len(layer_ids)
        self.hidden_size = vllm_config.model_config.get_hidden_size()
        self.hidden_states = torch.zeros(
            (self.max_num_tokens, self.num_hidden_states, self.hidden_size),
            dtype=self.dtype,
            device=device,
        )
        self.cudagraph_dispatcher = CudagraphDispatcher(self.vllm_config)

        self._slot_mapping_buffer = torch.zeros(
            self.max_num_tokens, dtype=torch.int64, device=device
        )
```
**EN:** This method initializes the object state within `ExtractHiddenStatesProposer`. Key calls include `getattr`, `len`, `get_hidden_size`, `zeros`, `CudagraphDispatcher`, `ValueError`. It touches state such as `vllm_config`, `device`, `dtype`, `dp_rank`, `model`, `attn_layer_names`, `attn_metadata_builder`, `kv_cache_gid`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`ExtractHiddenStatesProposer`。 关键调用包括 `getattr`, `len`, `get_hidden_size`, `zeros`, `CudagraphDispatcher`, `ValueError`。 它会读写 `vllm_config`, `device`, `dtype`, `dp_rank`, `model`, `attn_layer_names`, `attn_metadata_builder`, `kv_cache_gid` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `ExtractHiddenStatesProposer.propose` method / `ExtractHiddenStatesProposer.propose` 方法
```python
    def propose(
        self,
        sampled_token_ids: torch.Tensor,
        target_hidden_states: list[torch.Tensor],
        common_attn_metadata: CommonAttentionMetadata,
        slot_mappings: dict[str, torch.Tensor]
        | list[dict[str, torch.Tensor]]
        | None = None,
    ) -> torch.Tensor:
        """Propose draft tokens by calling the ExtractHiddenStatesModel model.

        The ExtractHiddenStatesModel caches the hidden states in the KV cache
        without performing actual attention computation. This allows us to
        extract and store hidden states for later use (e.g., KV transfer).

        This proposer doesn't actually perform speculation - it returns the
        sampled tokens as "draft" tokens, ensuring they always verify (match).
        The main purpose is to cache hidden states, not to speculate.

        Args:
            sampled_token_ids: Sampled token IDs from the target model
            target_hidden_states: List of hidden state tensors from target model
                                (one per aux hidden state layer)
            common_attn_metadata: Attention metadata
            slot_mappings: Slot mappings for KV cache (unused, provided for
                          interface compatibility)

        Returns:
            Tuple of:
                - Draft tokens matching sampled tokens, shape [batch_size, 1]
                - KV connector output (if KV transfer is active), else None
        """
        assert self.model is not None and isinstance(target_hidden_states, list)

        # target_hidden_states is a list of tensors (one per layer)
        # Each tensor has shape [num_tokens, hidden_size]
        # Stack to shape: [num_tokens, num_hidden_states, hidden_size]
        stacked_hidden_states = torch.stack(target_hidden_states, dim=1)
        num_tokens = stacked_hidden_states.shape[0]

        # Copy hidden states to buffer
        self.hidden_states[:num_tokens] = stacked_hidden_states

        assert self.attn_metadata_builder is not None
        attn_metadata = self.attn_metadata_builder.build_for_drafting(
            common_attn_metadata=common_attn_metadata, draft_index=0
        )

        # We assume all cache-only layers belong to the same KV cache group,
        # thus using the same attention metadata.
        per_layer_attn_metadata = {}
        for layer_name in self.attn_layer_names:
            per_layer_attn_metadata[layer_name] = attn_metadata

        cudagraph_runtime_mode, num_input_tokens, num_tokens_across_dp = (
            self._determine_batch_execution_and_padding(num_tokens)
        )
        if num_tokens_across_dp is not None:
            num_tokens_across_dp[self.dp_rank] = num_input_tokens

        with set_forward_context(
            per_layer_attn_metadata,
            self.vllm_config,
            num_tokens=num_input_tokens,
            num_tokens_across_dp=num_tokens_across_dp,
            cudagraph_runtime_mode=cudagraph_runtime_mode,
            slot_mapping=self._get_slot_mapping(
                num_input_tokens, common_attn_metadata.slot_mapping
            ),
        ):
            self.model(
                hidden_states=self.hidden_states[:num_input_tokens],
            )

        # Return the sampled tokens as "draft" tokens
        # Shape: [batch_size, 1] to match num_speculative_tokens=1
        # On decode steps with spec tokens, sampled_token_ids may have
        # shape [batch_size, 2] (target + spec verification); slice to
        # return only the target-sampled column.
        return sampled_token_ids[:, :1]
```
**EN:** This method implements `propose` within `ExtractHiddenStatesProposer`. The docstring frames it as: Propose draft tokens by calling the ExtractHiddenStatesModel model. Key calls include `stack`, `build_for_drafting`, `_determine_batch_execution_and_padding`, `isinstance`, `set_forward_context`, `model`. It touches state such as `hidden_states`, `dp_rank`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `propose`，其作用域位于`ExtractHiddenStatesProposer`。 关键调用包括 `stack`, `build_for_drafting`, `_determine_batch_execution_and_padding`, `isinstance`, `set_forward_context`, `model`。 它会读写 `hidden_states`, `dp_rank` 等状态。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `ExtractHiddenStatesProposer._determine_batch_execution_and_padding` method / `ExtractHiddenStatesProposer._determine_batch_execution_and_padding` 方法
```python
    def _determine_batch_execution_and_padding(
        self,
        num_tokens: int,
        use_cudagraphs: bool = True,
    ) -> tuple[CUDAGraphMode, int, torch.Tensor | None]:
        cudagraph_mode, batch_desc = self.cudagraph_dispatcher.dispatch(
            num_tokens,
            valid_modes=({CUDAGraphMode.NONE} if not use_cudagraphs else None),
        )
        num_tokens_padded = batch_desc.num_tokens

        # Extra coordination when running data-parallel since we need to
        # coordinate across ranks
        # TODO(Flechman): support DBO ubatching
        should_ubatch, num_tokens_across_dp = False, None
        if self.vllm_config.parallel_config.data_parallel_size > 1:
            should_ubatch, num_tokens_across_dp, synced_cudagraph_mode = (
                coordinate_batch_across_dp(
                    num_tokens_unpadded=num_tokens,
                    parallel_config=self.vllm_config.parallel_config,
                    allow_microbatching=False,
                    num_tokens_padded=num_tokens_padded,
                    cudagraph_mode=cudagraph_mode.value,
                )
            )
            assert not should_ubatch, (
                "DBO ubatching not implemented for extract_hidden_states"
            )

            # Extract DP-synced values
            if num_tokens_across_dp is not None:
                dp_rank = self.dp_rank
                num_tokens_padded = int(num_tokens_across_dp[dp_rank].item())
                # Re-dispatch with DP padding so we have the correct
                # batch_descriptor
                cudagraph_mode, batch_desc = self.cudagraph_dispatcher.dispatch(
                    num_tokens_padded,
                    valid_modes={CUDAGraphMode(synced_cudagraph_mode)},
                )
                # Assert to make sure the agreed upon token count is correct
                # otherwise num_tokens_across_dp will no-longer be valid
                assert batch_desc.num_tokens == num_tokens_padded
                num_tokens_across_dp[dp_rank] = num_tokens_padded

        return cudagraph_mode, num_tokens_padded, num_tokens_across_dp
```
**EN:** This method implements `_determine_batch_execution_and_padding` within `ExtractHiddenStatesProposer`. Key calls include `dispatch`, `coordinate_batch_across_dp`, `int`, `item`, `CUDAGraphMode`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_determine_batch_execution_and_padding`，其作用域位于`ExtractHiddenStatesProposer`。 关键调用包括 `dispatch`, `coordinate_batch_across_dp`, `int`, `item`, `CUDAGraphMode`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `ExtractHiddenStatesProposer.dummy_run` method / `ExtractHiddenStatesProposer.dummy_run` 方法
```python
    @torch.inference_mode()
    def dummy_run(
        self,
        num_tokens: int,
        use_cudagraphs: bool = True,
        is_graph_capturing: bool = False,
        slot_mappings: dict[str, torch.Tensor] | None = None,
    ) -> None:
        assert self.model is not None, "Model must be initialized before dummy_run"
        cudagraph_runtime_mode, num_input_tokens, num_tokens_across_dp = (
            self._determine_batch_execution_and_padding(
                num_tokens, use_cudagraphs=use_cudagraphs
            )
        )

        if num_tokens_across_dp is not None:
            num_tokens_across_dp[self.dp_rank] = num_input_tokens

        # Use our own slot mapping buffer during cudagraph capture.
        if (
            self.attn_layer_names
            and slot_mappings is not None
            and self.attn_layer_names[0] in slot_mappings
        ):
            slot_mapping_dict = self._get_slot_mapping(num_input_tokens)
        else:
            slot_mapping_dict = slot_mappings or {}

        with set_forward_context(
            None,
            self.vllm_config,
            num_tokens=num_input_tokens,
            num_tokens_across_dp=num_tokens_across_dp,
            cudagraph_runtime_mode=cudagraph_runtime_mode,
            slot_mapping=slot_mapping_dict,
        ):
            self.model(
                hidden_states=self.hidden_states[:num_input_tokens],
            )
```
**EN:** This method implements `dummy_run` within `ExtractHiddenStatesProposer`. Key calls include `inference_mode`, `_determine_batch_execution_and_padding`, `_get_slot_mapping`, `set_forward_context`, `model`. It touches state such as `dp_rank`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `dummy_run`，其作用域位于`ExtractHiddenStatesProposer`。 关键调用包括 `inference_mode`, `_determine_batch_execution_and_padding`, `_get_slot_mapping`, `set_forward_context`, `model`。 它会读写 `dp_rank` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `ExtractHiddenStatesProposer._build_attn_metadata_builder` method / `ExtractHiddenStatesProposer._build_attn_metadata_builder` 方法
```python
    def _build_attn_metadata_builder(
        self, draft_attn_layers: dict[str, AttentionLayerBase]
    ) -> AttentionMetadataBuilder:
        """Build the attention metadata builder from draft attention layers."""
        if not draft_attn_layers:
            raise ValueError("No attention layers found for ExtractHiddenStatesModel")
        layer = next(iter(draft_attn_layers.values()))
        attn_backend = layer.get_attn_backend()
        return attn_backend.get_builder_cls()(
            layer.get_kv_cache_spec(self.vllm_config),
            self.attn_layer_names,
            self.vllm_config,
            self.device,
        )
```
**EN:** This method implements `_build_attn_metadata_builder` within `ExtractHiddenStatesProposer`. The docstring frames it as: Build the attention metadata builder from draft attention layers. Key calls include `next`, `get_attn_backend`, `ValueError`, `iter`, `get_builder_cls`, `get_kv_cache_spec`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_build_attn_metadata_builder`，其作用域位于`ExtractHiddenStatesProposer`。 关键调用包括 `next`, `get_attn_backend`, `ValueError`, `iter`, `get_builder_cls`, `get_kv_cache_spec`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `ExtractHiddenStatesProposer.prepare_next_token_ids_padded` method / `ExtractHiddenStatesProposer.prepare_next_token_ids_padded` 方法
```python
    def prepare_next_token_ids_padded(
        self,
        sampled_token_ids: torch.Tensor,
        requests: dict[str, CachedRequestState],
        gpu_input_batch: InputBatch,
        discard_request_mask: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """
        Prepare next token IDs for speculative decoding.

        Since num_speculative_tokens == 1, sampled_token_ids has shape
        (batch_size, 1). For each request we either use the sampled token
        (if valid and not discarded) or a backup token from the request state.
        """
        num_reqs = gpu_input_batch.num_reqs
        device = sampled_token_ids.device

        # Compute backup tokens for discarded / invalid requests
        seq_lens_list = (gpu_input_batch.num_tokens_no_spec[:num_reqs] - 1).tolist()
        backup_tokens_gpu = torch.tensor(
            [
                requests[gpu_input_batch.req_ids[i]].get_token_id(seq_lens_list[i])
                for i in range(num_reqs)
            ],
            dtype=torch.int32,
            device=device,
        )

        assert discard_request_mask.dtype == torch.bool

        # With num_speculative_tokens == 1, there is exactly one token
        sampled = sampled_token_ids[:, 0]
        is_valid = (sampled >= 0) & (sampled < gpu_input_batch.vocab_size)
        valid_sampled_tokens_count = is_valid.to(torch.int32)

        use_sampled = is_valid & ~discard_request_mask[:num_reqs]
        next_token_ids = torch.where(
            use_sampled, sampled.to(torch.int32), backup_tokens_gpu
        )

        return next_token_ids, valid_sampled_tokens_count
```
**EN:** This method prepares inputs and state within `ExtractHiddenStatesProposer`. The docstring frames it as: Prepare next token IDs for speculative decoding. Key calls include `tolist`, `tensor`, `to`, `where`, `get_token_id`, `range`.
**CN:** 该方法会准备输入与状态，其作用域位于`ExtractHiddenStatesProposer`。 关键调用包括 `tolist`, `tensor`, `to`, `where`, `get_token_id`, `range`。

### `ExtractHiddenStatesProposer.load_model` method / `ExtractHiddenStatesProposer.load_model` 方法
```python
    def load_model(self, target_model: nn.Module) -> None:
        """Load the ExtractHiddenStatesModel model.

        This method instantiates the ExtractHiddenStatesModel model which is used
        to cache hidden states during speculative decoding. The model uses
        cache-only attention (no computation, just caching KV states).

        Args:
            target_model: The target model (passed for compatibility with
                         EagleProposer interface, but not used here)
        """
        # Get the target model's attention layers before loading draft model
        target_attn_layer_names = set(
            get_layers_from_vllm_config(self.vllm_config, AttentionLayerBase).keys()  # type: ignore[type-abstract]
        )

        assert self.vllm_config.speculative_config is not None
        draft_model_config = self.vllm_config.speculative_config.draft_model_config
        from vllm.compilation.backends import set_model_tag

        with set_model_tag("extract_hidden_states"):
            self.model = get_model(
                vllm_config=self.vllm_config, model_config=draft_model_config
            )

        # Identify draft model's attention layers (difference from target)
        all_attn_layers = get_layers_from_vllm_config(
            self.vllm_config,
            AttentionLayerBase,  # type: ignore[type-abstract]
        )
        draft_attn_layers = {
            name: layer
            for name, layer in all_attn_layers.items()
            if name not in target_attn_layer_names
        }
        self.attn_layer_names = list(draft_attn_layers.keys())
        assert len(draft_attn_layers) == 1, (
            "ExtractHiddenStatesModel should have exactly one "
            f"attention layer, found {len(draft_attn_layers)}"
        )
        self.attn_metadata_builder = self._build_attn_metadata_builder(
            draft_attn_layers
        )
```
**EN:** This method loads external or cached state within `ExtractHiddenStatesProposer`. The docstring frames it as: Load the ExtractHiddenStatesModel model. Key calls include `set`, `get_layers_from_vllm_config`, `list`, `_build_attn_metadata_builder`, `keys`, `set_model_tag`. It touches state such as `attn_layer_names`, `attn_metadata_builder`, `model`.
**CN:** 该方法会加载外部或缓存状态，其作用域位于`ExtractHiddenStatesProposer`。 关键调用包括 `set`, `get_layers_from_vllm_config`, `list`, `_build_attn_metadata_builder`, `keys`, `set_model_tag`。 它会读写 `attn_layer_names`, `attn_metadata_builder`, `model` 等状态。

### `ExtractHiddenStatesProposer.validate_same_kv_cache_group` method / `ExtractHiddenStatesProposer.validate_same_kv_cache_group` 方法
```python
    def validate_same_kv_cache_group(self, kv_cache_config: KVCacheConfig) -> None:
        """Validate all drafting layers belong to the same KV cache group
        and record the group index for common_attn_metadata selection."""
        assert len(self.attn_layer_names) == 1
        layer = self.attn_layer_names[0]
        for gid, group in enumerate(kv_cache_config.kv_cache_groups):
            if layer in group.layer_names:
                self.kv_cache_gid = gid
                return
        raise ValueError(f"Cache-only layer {layer!r} not in any KV cache group")
```
**EN:** This method validates assumptions or constraints within `ExtractHiddenStatesProposer`. The docstring frames it as: Validate all drafting layers belong to the same KV cache group and record the group index for common_attn_metadata selection. Key calls include `enumerate`, `ValueError`, `len`. It touches state such as `kv_cache_gid`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会校验前提与约束，其作用域位于`ExtractHiddenStatesProposer`。 关键调用包括 `enumerate`, `ValueError`, `len`。 它会读写 `kv_cache_gid` 等状态。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `ExtractHiddenStatesProposer`: central class or interface in this module. / `ExtractHiddenStatesProposer`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `__future__`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.forward_context`, `vllm.model_executor.layers.attention_layer_base`, `vllm.model_executor.model_loader`, `vllm.v1.attention.backend`, `vllm.v1.cudagraph_dispatcher`, `vllm.v1.worker.dp_utils`, `vllm.v1.worker.gpu_input_batch`, `vllm.v1.kv_cache_interface`, `vllm.compilation.backends`
