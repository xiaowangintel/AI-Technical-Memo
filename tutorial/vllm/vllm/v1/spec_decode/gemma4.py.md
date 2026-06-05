# gemma4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/spec_decode/gemma4.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Gemma4 MTP (Multi-Token Prediction) proposer for speculative decoding. / 该模块位于 `spec_decode` 子系统，主要围绕 `Gemma4Proposer` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Gemma4 MTP (Multi-Token Prediction) proposer for speculative decoding.

The Gemma4 assistant model runs all decoder layers per draft step
(producing one token), and all its attention layers share KV cache
with the target model via cross-model KV sharing.
"""

from collections import defaultdict
from copy import copy

import torch
import torch.nn as nn

from vllm.config import VllmConfig, get_layers_from_vllm_config, replace
from vllm.logger import init_logger
from vllm.model_executor.layers.attention_layer_base import AttentionLayerBase
from vllm.v1.attention.backend import CommonAttentionMetadata
from vllm.v1.kv_cache_interface import (
    KVCacheConfig,
    KVCacheSpec,
    UniformTypeKVCacheSpecs,
)
from vllm.v1.spec_decode.llm_base_proposer import SpecDecodeBaseProposer
from vllm.v1.worker.utils import AttentionGroup

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `Gemma4Proposer` class / `Gemma4Proposer` 类
```python
class Gemma4Proposer(SpecDecodeBaseProposer):
```
**EN:** Introduces the `Gemma4Proposer` class on top of `SpecDecodeBaseProposer`. Core methods include `__init__`, `set_per_group_block_table`, `model_returns_tuple`, `build_per_group_and_layer_attn_metadata`, `_greedy_sample`, `_setup_centroids_cuda_graphs`.
**CN:** 这里定义 `Gemma4Proposer` 类，其基类包括 `SpecDecodeBaseProposer`。核心方法包括 `__init__`, `set_per_group_block_table`, `model_returns_tuple`, `build_per_group_and_layer_attn_metadata`, `_greedy_sample`, `_setup_centroids_cuda_graphs`。

### `Gemma4Proposer.__init__` method / `Gemma4Proposer.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        device: torch.device,
        runner=None,
    ):
        super().__init__(
            vllm_config,
            device,
            pass_hidden_states_to_model=True,
            runner=runner,
        )
        # All draft steps predict from the same position (the last
        # target-model position), so positions and seq_lens must not
        # advance between steps.
        self.constant_draft_positions = True

        # Per-group block tables for multi-group KV cache models.
        # Populated by gpu_model_runner during _prepare_inputs.
        self._per_group_block_tables: dict[int, torch.Tensor] = {}

        # Centroids CUDA graphs — populated in load_model if centroids
        # masking is active. _centroids_sizes is pre-sorted for fast
        # lookup in _greedy_sample.
        self._centroids_sizes: list[int] = []
        self._centroids_graphs: dict[int, torch.cuda.CUDAGraph] = {}
        self._centroids_inputs: dict[int, torch.Tensor] = {}
        self._centroids_outputs: dict[int, torch.Tensor] = {}
```
**EN:** This method initializes the object state within `Gemma4Proposer`. Key calls include `__init__`, `super`. It touches state such as `constant_draft_positions`, `_per_group_block_tables`, `_centroids_sizes`, `_centroids_graphs`, `_centroids_inputs`, `_centroids_outputs`.
**CN:** 该方法会初始化对象状态，其作用域位于`Gemma4Proposer`。 关键调用包括 `__init__`, `super`。 它会读写 `constant_draft_positions`, `_per_group_block_tables`, `_centroids_sizes`, `_centroids_graphs`, `_centroids_inputs`, `_centroids_outputs` 等状态。

### `Gemma4Proposer.build_per_group_and_layer_attn_metadata` method / `Gemma4Proposer.build_per_group_and_layer_attn_metadata` 方法
```python
    def build_per_group_and_layer_attn_metadata(
        self,
        common_attn_metadata: CommonAttentionMetadata,
        draft_index: int = 0,
    ) -> tuple[list[object], dict[str, object]]:
        """Build attention metadata using the correct block table per group.

        Gemma4 has multiple KV cache groups (sliding vs full attention)
        with different block tables.  The base class receives a single
        common_attn_metadata whose block_table belongs to one group.
        We swap in the correct block table for each draft attention group.
        """
        per_group_attn_metadata: list[object] = []
        per_layer_attn_metadata: dict[str, object] = {}
        for attn_group in self.draft_attn_groups:
            gid = attn_group.kv_cache_group_id
            if gid in self._per_group_block_tables:
                cm = copy(common_attn_metadata)
                cm.block_table_tensor = self._per_group_block_tables[gid]
            else:
                cm = common_attn_metadata
            attn_metadata = attn_group.get_metadata_builder().build_for_drafting(
                common_attn_metadata=cm, draft_index=draft_index
            )
            per_group_attn_metadata.append(attn_metadata)
            for layer_name in attn_group.layer_names:
                per_layer_attn_metadata[layer_name] = attn_metadata
        return per_group_attn_metadata, per_layer_attn_metadata
```
**EN:** This method builds derived structures within `Gemma4Proposer`. The docstring frames it as: Build attention metadata using the correct block table per group. Key calls include `build_for_drafting`, `append`, `copy`, `get_metadata_builder`. The control flow contains 1 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会构建派生结构，其作用域位于`Gemma4Proposer`。 关键调用包括 `build_for_drafting`, `append`, `copy`, `get_metadata_builder`。 控制流包含 1 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `Gemma4Proposer._greedy_sample` method / `Gemma4Proposer._greedy_sample` 方法
```python
    def _greedy_sample(self, hidden_states: torch.Tensor) -> torch.Tensor:
        if self._centroids_sizes:
            T = hidden_states.shape[0]
            for size in self._centroids_sizes:
                if size >= T:
                    self._centroids_inputs[size][:T].copy_(hidden_states)
                    self._centroids_graphs[size].replay()
                    return self._centroids_outputs[size][:T].clone()
            return self.model.get_top_tokens(hidden_states)
        return super()._greedy_sample(hidden_states)
```
**EN:** This method implements `_greedy_sample` within `Gemma4Proposer`. Key calls include `_greedy_sample`, `get_top_tokens`, `super`, `copy_`, `replay`, `clone`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_greedy_sample`，其作用域位于`Gemma4Proposer`。 关键调用包括 `_greedy_sample`, `get_top_tokens`, `super`, `copy_`, `replay`, `clone`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `Gemma4Proposer._create_draft_vllm_config` method / `Gemma4Proposer._create_draft_vllm_config` 方法
```python
    def _create_draft_vllm_config(self) -> VllmConfig:
        """Preserve the target's forced TRITON_ATTN backend for draft layers.

        Gemma4 forces TRITON_ATTN due to heterogeneous head dimensions
        (head_dim=256 sliding, global_head_dim=512 full). The base class
        resets attention_config.backend to None for draft models, causing
        sliding layers to fall back to FLASH_ATTN which cannot handle
        KV-shared cache. Override to carry the target's backend through.
        """
        base = super()._create_draft_vllm_config()
        target_backend = self.vllm_config.attention_config.backend
        if target_backend is not None:
            base = replace(
                base,
                attention_config=replace(
                    base.attention_config,
                    backend=target_backend,
                ),
            )
        return base
```
**EN:** This method implements `_create_draft_vllm_config` within `Gemma4Proposer`. The docstring frames it as: Preserve the target's forced TRITON_ATTN backend for draft layers. Key calls include `_create_draft_vllm_config`, `replace`, `super`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_create_draft_vllm_config`，其作用域位于`Gemma4Proposer`。 关键调用包括 `_create_draft_vllm_config`, `replace`, `super`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `Gemma4Proposer.load_model` method / `Gemma4Proposer.load_model` 方法
```python
    def load_model(self, target_model: nn.Module) -> None:
        target_attn_layer_names = set(
            get_layers_from_vllm_config(
                self.vllm_config,
                AttentionLayerBase,  # type: ignore[type-abstract]
            ).keys()
        )

        super().load_model(target_model)

        self._setup_gemma4_kv_sharing(target_attn_layer_names)

        if getattr(self.model, "masked_embedding", None) is not None:
            self._setup_centroids_cuda_graphs()
```
**EN:** This method loads external or cached state within `Gemma4Proposer`. Key calls include `set`, `load_model`, `_setup_gemma4_kv_sharing`, `keys`, `getattr`, `_setup_centroids_cuda_graphs`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会加载外部或缓存状态，其作用域位于`Gemma4Proposer`。 关键调用包括 `set`, `load_model`, `_setup_gemma4_kv_sharing`, `keys`, `getattr`, `_setup_centroids_cuda_graphs`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `Gemma4Proposer.validate_same_kv_cache_group` method / `Gemma4Proposer.validate_same_kv_cache_group` 方法
```python
    def validate_same_kv_cache_group(self, kv_cache_config: KVCacheConfig) -> None:
        """Draft layers span multiple KV cache groups (sliding + full
        attention with different head dimensions), so skip the base
        class single-group assertion."""
```
**EN:** This method validates assumptions or constraints within `Gemma4Proposer`. The docstring frames it as: Draft layers span multiple KV cache groups (sliding + full attention with different head dimensions), so skip the base class single-group assertion.
**CN:** 该方法会校验前提与约束，其作用域位于`Gemma4Proposer`。

### `Gemma4Proposer.initialize_attn_backend` method / `Gemma4Proposer.initialize_attn_backend` 方法
```python
    def initialize_attn_backend(
        self,
        kv_cache_config: KVCacheConfig,
        kernel_block_sizes: list[int] | None = None,
    ) -> None:
        """Create separate AttentionGroup objects per KV cache spec
        so that each head-dim variant gets its own metadata builder."""
        all_attn_layers = get_layers_from_vllm_config(
            self.vllm_config,
            AttentionLayerBase,  # type: ignore[type-abstract]
        )

        layer_to_gid: dict[str, int] = {}
        layer_to_spec: dict[str, KVCacheSpec] = {}
        for gid, group in enumerate(kv_cache_config.kv_cache_groups):
            group_spec = group.kv_cache_spec
            for ln in group.layer_names:
                layer_to_gid[ln] = gid
                if isinstance(group_spec, UniformTypeKVCacheSpecs):
                    if ln in group_spec.kv_cache_specs:
                        layer_to_spec[ln] = group_spec.kv_cache_specs[ln]
                    else:
                        tgt = getattr(
                            all_attn_layers.get(ln),
                            "kv_sharing_target_layer_name",
                            None,
                        )
                        if tgt and tgt in group_spec.kv_cache_specs:
                            layer_to_spec[ln] = group_spec.kv_cache_specs[tgt]
                        else:
                            layer_to_spec[ln] = group_spec
                else:
                    layer_to_spec[ln] = group_spec

        attention_groups: dict[tuple[tuple[str, str], KVCacheSpec], AttentionGroup] = {}
        for layer_name in self._draft_attn_layer_names:
            if layer_name not in layer_to_spec:
                continue
            attn_layer = all_attn_layers[layer_name]
            attn_backend = attn_layer.get_attn_backend()
            spec = layer_to_spec[layer_name]
            gid = layer_to_gid[layer_name]
            group_key = (attn_backend.full_cls_name(), spec)

            if group_key not in attention_groups:
                kernel_block_size = (
                    kernel_block_sizes[gid]
                    if kernel_block_sizes is not None and gid < len(kernel_block_sizes)
                    else None
                )
                attn_group = AttentionGroup(
                    backend=attn_backend,
                    layer_names=[layer_name],
                    kv_cache_spec=spec,
                    kv_cache_group_id=gid,
                )
                attn_group.create_metadata_builders(
                    self.vllm_config,
                    self.device,
                    kernel_block_size=kernel_block_size,
                )
                attention_groups[group_key] = attn_group
            else:
                attention_groups[group_key].layer_names.append(layer_name)

        self.draft_attn_groups = list(attention_groups.values())
        if self.draft_attn_groups:
            self.kv_cache_gid = self.draft_attn_groups[0].kv_cache_group_id
            self.block_size = (
                self.draft_attn_groups[0]
                .get_metadata_builder()
                .kv_cache_spec.block_size
            )
        else:
            self.kv_cache_gid = 0
            self.block_size = kv_cache_config.kv_cache_groups[
                0
            ].kv_cache_spec.block_size
        logger.debug("Using block size %d for drafting layers", self.block_size)
```
**EN:** This method implements `initialize_attn_backend` within `Gemma4Proposer`. The docstring frames it as: Create separate AttentionGroup objects per KV cache spec so that each head-dim variant gets its own metadata builder. Key calls include `get_layers_from_vllm_config`, `enumerate`, `list`, `debug`, `get_attn_backend`, `values`. It touches state such as `draft_attn_groups`, `kv_cache_gid`, `block_size`. The control flow contains 7 branch(es) and 3 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `initialize_attn_backend`，其作用域位于`Gemma4Proposer`。 关键调用包括 `get_layers_from_vllm_config`, `enumerate`, `list`, `debug`, `get_attn_backend`, `values`。 它会读写 `draft_attn_groups`, `kv_cache_gid`, `block_size` 等状态。 控制流包含 7 个分支和 3 个循环，说明这里承担了较强的协调逻辑。

### `Gemma4Proposer._setup_gemma4_kv_sharing` method / `Gemma4Proposer._setup_gemma4_kv_sharing` 方法
```python
    def _setup_gemma4_kv_sharing(
        self,
        target_attn_layer_names: set[str],
    ) -> None:
        """Wire draft layers to share KV with the target model.

        Each draft decoder layer is mapped to the last non-KV-shared
        target layer of the same attention type (sliding or full).
        """
        draft_config = self.speculative_config.draft_model_config.hf_config
        draft_text_config = draft_config.get_text_config()
        target_config = self.vllm_config.model_config.hf_config
        target_text_config = target_config.get_text_config()
        target_layer_types = getattr(target_text_config, "layer_types", [])

        if not (hasattr(self.model, "model") and hasattr(self.model.model, "layers")):
            return

        target_num_kv_shared = getattr(target_text_config, "num_kv_shared_layers", 0)
        num_non_shared = len(target_layer_types) - target_num_kv_shared
        type_to_target_indices: dict[str, list[int]] = defaultdict(list)
        for idx, lt in enumerate(target_layer_types[:num_non_shared]):
            type_to_target_indices[lt].append(idx)

        target_prefix = "model.layers"
        for name in target_attn_layer_names:
            if ".layers." in name:
                target_prefix = name.split(".layers.")[0] + ".layers"
                break

        draft_layer_types = getattr(draft_text_config, "layer_types", [])
        for draft_idx, layer in enumerate(self.model.model.layers):
            if not hasattr(layer, "self_attn"):
                continue
            attn = getattr(layer.self_attn, "attn", None)
            if attn is None:
                continue

            draft_layer_type = (
                draft_layer_types[draft_idx]
                if draft_idx < len(draft_layer_types)
                else "full_attention"
            )
            candidates = type_to_target_indices.get(draft_layer_type, [])
            if not candidates:
                logger.warning(
                    "No target layer of type '%s' for draft layer %d",
                    draft_layer_type,
                    draft_idx,
                )
                continue

            target_idx = candidates[-1]
            target_layer_name = f"{target_prefix}.{target_idx}.self_attn.attn"
            attn.kv_sharing_target_layer_name = target_layer_name
            logger.info(
                "Gemma4 MTP: draft layer %d (%s) -> %s",
                draft_idx,
                draft_layer_type,
                target_layer_name,
            )
```
**EN:** This method implements `_setup_gemma4_kv_sharing` within `Gemma4Proposer`. The docstring frames it as: Wire draft layers to share KV with the target model. Key calls include `get_text_config`, `getattr`, `defaultdict`, `enumerate`, `len`, `append`. The control flow contains 6 branch(es) and 3 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_setup_gemma4_kv_sharing`，其作用域位于`Gemma4Proposer`。 关键调用包括 `get_text_config`, `getattr`, `defaultdict`, `enumerate`, `len`, `append`。 控制流包含 6 个分支和 3 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `Gemma4Proposer`: central class or interface in this module. / `Gemma4Proposer`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `collections`, `copy`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.logger`, `vllm.model_executor.layers.attention_layer_base`, `vllm.v1.attention.backend`, `vllm.v1.kv_cache_interface`, `vllm.v1.spec_decode.llm_base_proposer`, `vllm.v1.worker.utils`
