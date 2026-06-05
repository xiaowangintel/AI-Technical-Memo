# attn_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/attn_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `AttentionCGSupportInfo`, `get_kv_cache_spec`, `init_attn_backend` for the V1 `worker/gpu` subsystem. / 为 V1 的 `worker/gpu` 子系统实现 `AttentionCGSupportInfo`, `get_kv_cache_spec`, `init_attn_backend`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from collections.abc import Sequence
from dataclasses import dataclass
from typing import Any, cast

import torch

from vllm.config import VllmConfig, get_layers_from_vllm_config
from vllm.model_executor.layers.attention_layer_base import AttentionLayerBase
from vllm.utils.torch_utils import get_dtype_size
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionCGSupport,
    CommonAttentionMetadata,
)
from vllm.v1.kv_cache_interface import (
    AttentionSpec,
    KVCacheConfig,
    KVCacheSpec,
    MambaSpec,
    UniformTypeKVCacheSpecs,
)
from vllm.v1.worker.gpu.model_states.interface import ModelSpecificAttnMetadata
from vllm.v1.worker.utils import AttentionGroup, bind_kv_cache
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.config`, `vllm.model_executor.layers.attention_layer_base`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.config`, `vllm.model_executor.layers.attention_layer_base`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `AttentionCGSupportInfo` class / `AttentionCGSupportInfo` 类
```python
@dataclass(frozen=True)
class AttentionCGSupportInfo:
    min_cg_support: AttentionCGSupport = AttentionCGSupport.ALWAYS
    min_cg_attn_backend: str | None = None
```
**EN:** Introduces the `AttentionCGSupportInfo` class. Core methods include its methods defined below.
**CN:** 这里定义 `AttentionCGSupportInfo` 类。核心方法包括 下方定义的方法。

### `get_kv_cache_spec` function / `get_kv_cache_spec` 函数
```python
def get_kv_cache_spec(vllm_config: VllmConfig) -> dict[str, KVCacheSpec]:
    kv_cache_spec: dict[str, KVCacheSpec] = {}
    layer_type = cast(type[Any], AttentionLayerBase)
    attn_layers = get_layers_from_vllm_config(vllm_config, layer_type)
    for layer_name, attn_module in attn_layers.items():
        # Skip modules that don't need KV cache (eg encoder-only attention)
        if spec := attn_module.get_kv_cache_spec(vllm_config):
            kv_cache_spec[layer_name] = spec
    return kv_cache_spec
```
**EN:** This function returns or derives a value within the module. Key calls include `cast`, `get_layers_from_vllm_config`, `items`, `get_kv_cache_spec`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `cast`, `get_layers_from_vllm_config`, `items`, `get_kv_cache_spec`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `init_attn_backend` function / `init_attn_backend` 函数
```python
def init_attn_backend(
    kv_cache_config: KVCacheConfig,
    vllm_config: VllmConfig,
    device: torch.device,
    active_layer_names: set[str] | None = None,
) -> tuple[
    dict[str, type[AttentionBackend]],
    list[list[AttentionGroup]],
    AttentionCGSupportInfo,
]:
    attn_backends: dict[str, type[AttentionBackend]] = {}
    attn_groups: list[list[AttentionGroup]] = []
    attn_backend_workspace: torch.Tensor | None = None
    # Find minimum cudagraph support across all attention backends
    min_cg_support = AttentionCGSupport.ALWAYS
    min_cg_attn_backend = None
    for kv_cache_group_id, kv_cache_group_spec in enumerate(
        kv_cache_config.kv_cache_groups
    ):
        layer_names = kv_cache_group_spec.layer_names
        if active_layer_names is not None:
            layer_names = list(active_layer_names.intersection(layer_names))

        layer_type = cast(type[Any], AttentionLayerBase)
        attn_layers = get_layers_from_vllm_config(vllm_config, layer_type, layer_names)

        group_map: dict[tuple[tuple[str, str], KVCacheSpec], AttentionGroup] = {}
        group_order: list[tuple[tuple[str, str], KVCacheSpec]] = []

        for layer_name in layer_names:
            attn_backend = attn_layers[layer_name].get_attn_backend()
            attn_backends[layer_name] = attn_backend

            layer_kv_cache_spec: KVCacheSpec = kv_cache_group_spec.kv_cache_spec
            if isinstance(layer_kv_cache_spec, UniformTypeKVCacheSpecs):
                layer_kv_cache_spec = layer_kv_cache_spec.kv_cache_specs[layer_name]

            key = (attn_backend.full_cls_name(), layer_kv_cache_spec)
            if key not in group_map:
                group_map[key] = AttentionGroup(
                    attn_backend,
                    [layer_name],
                    layer_kv_cache_spec,
                    kv_cache_group_id,
                )
                group_order.append(key)
            else:
                group_map[key].layer_names.append(layer_name)

        groups = [group_map[key] for key in group_order]
        for group in groups:
            group.create_metadata_builders(
                vllm_config=vllm_config,
                device=device,
                kernel_block_size=None,
                num_metadata_builders=1,
            )
            builder = group.get_metadata_builder(0)
            if attn_backend_workspace is None:
                if hasattr(builder, "_get_workspace_buffer"):
                    attn_backend_workspace = builder._get_workspace_buffer()
            else:
                if hasattr(builder, "set_workspace_buffer"):
                    builder.set_workspace_buffer(attn_backend_workspace)
            # Check cudagraph support for the attention backend
            cg_support = builder.get_cudagraph_support(
                vllm_config,
                cast(AttentionSpec, kv_cache_group_spec.kv_cache_spec),
            )
            if cg_support.value < min_cg_support.value:
                min_cg_support = cg_support
                min_cg_attn_backend = attn_backend.__name__
        attn_groups.append(groups)

    return (
        attn_backends,
        attn_groups,
        AttentionCGSupportInfo(
            min_cg_support=min_cg_support,
            min_cg_attn_backend=min_cg_attn_backend,
        ),
    )
```
**EN:** This function implements `init_attn_backend` within the module. Key calls include `enumerate`, `cast`, `get_layers_from_vllm_config`, `append`, `AttentionCGSupportInfo`, `list`. The control flow contains 7 branch(es) and 3 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `init_attn_backend`，其作用域位于the module。 关键调用包括 `enumerate`, `cast`, `get_layers_from_vllm_config`, `append`, `AttentionCGSupportInfo`, `list`。 控制流包含 7 个分支和 3 个循环，说明这里承担了较强的协调逻辑。

### `_allocate_kv_cache` function / `_allocate_kv_cache` 函数
```python
def _allocate_kv_cache(kv_cache_config: KVCacheConfig, device: torch.device):
    kv_cache_raw_tensors: dict[str, torch.Tensor] = {}
    for kv_cache_tensor in kv_cache_config.kv_cache_tensors:
        tensor = torch.zeros(kv_cache_tensor.size, dtype=torch.int8, device=device)
        for layer_name in kv_cache_tensor.shared_by:
            kv_cache_raw_tensors[layer_name] = tensor

    layer_names = set()
    for group in kv_cache_config.kv_cache_groups:
        for layer_name in group.layer_names:
            layer_names.add(layer_name)
    assert layer_names == set(kv_cache_raw_tensors.keys()), (
        "Some layers are not correctly initialized"
    )
    return kv_cache_raw_tensors
```
**EN:** This function implements `_allocate_kv_cache` within the module. Key calls include `set`, `zeros`, `add`, `keys`. The control flow contains 0 branch(es) and 4 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_allocate_kv_cache`，其作用域位于the module。 关键调用包括 `set`, `zeros`, `add`, `keys`。 控制流包含 0 个分支和 4 个循环，说明这里承担了较强的协调逻辑。

### `_reshape_kv_cache` function / `_reshape_kv_cache` 函数
```python
def _reshape_kv_cache(
    kv_cache_config: KVCacheConfig,
    kv_cache_raw_tensors: dict[str, torch.Tensor],
    attn_backends: dict[str, type[AttentionBackend]],
    cache_dtype: str,
) -> dict[str, Any]:
    kv_caches: dict[str, Any] = {}
    has_attn, has_mamba = False, False
    for kv_cache_group_spec in kv_cache_config.kv_cache_groups:
        for layer_name in kv_cache_group_spec.layer_names:
            kv_cache_spec = kv_cache_group_spec.kv_cache_spec
            if isinstance(kv_cache_spec, UniformTypeKVCacheSpecs):
                kv_cache_spec = kv_cache_spec.kv_cache_specs[layer_name]

            kv_raw_tensor = kv_cache_raw_tensors[layer_name]
            assert kv_raw_tensor.numel() % kv_cache_spec.page_size_bytes == 0
            num_blocks = kv_raw_tensor.numel() // kv_cache_spec.page_size_bytes

            if isinstance(kv_cache_spec, AttentionSpec):
                has_attn = True
                attn_backend = attn_backends[layer_name]
                kv_cache_shape = attn_backend.get_kv_cache_shape(
                    num_blocks,
                    kv_cache_spec.storage_block_size,
                    kv_cache_spec.num_kv_heads,
                    kv_cache_spec.head_size,
                    cache_dtype_str=cache_dtype,
                )

                # FIXME(woosuk): Add kv_cache_stride_order to all attention backends.
                try:
                    kv_cache_stride_order = attn_backend.get_kv_cache_stride_order()
                    assert len(kv_cache_stride_order) == len(kv_cache_shape)
                except (AttributeError, NotImplementedError):
                    kv_cache_stride_order = tuple(range(len(kv_cache_shape)))

                kv_cache_shape = tuple(kv_cache_shape[i] for i in kv_cache_stride_order)
                inv_order = [
                    kv_cache_stride_order.index(i)
                    for i in range(len(kv_cache_stride_order))
                ]

                dtype = kv_cache_spec.dtype
                kv_tensor = kv_raw_tensor.view(dtype)
                if kv_cache_spec.page_size_padded is not None:
                    # Use strided view to handle page_size_bytes that
                    # include padding. This follows the same pattern as
                    # MambaSpec handling in gpu_model_runner.py.
                    # NOTE: This assumes kv_cache_shape[0] == num_blocks
                    # (i.e. the first physical dimension is the block
# ... omitted for brevity ...
                for shape, dtype in zip(kv_cache_spec.shapes, kv_cache_spec.dtypes):
                    dtype_size = get_dtype_size(dtype)
                    num_element_per_page = kv_cache_spec.page_size_bytes // dtype_size
                    target_shape = (num_blocks, *shape)
                    stride = torch.empty(target_shape).stride()
                    target_stride = (num_element_per_page, *stride[1:])
                    assert storage_offset_bytes % dtype_size == 0
                    tensor = torch.as_strided(
                        kv_raw_tensor.view(dtype),
                        size=target_shape,
                        stride=target_stride,
                        storage_offset=storage_offset_bytes // dtype_size,
                    )
                    state_tensors.append(tensor)
                    storage_offset_bytes += stride[0] * dtype_size
                kv_caches[layer_name] = state_tensors
            else:
                raise NotImplementedError(
                    f"Unsupported KV cache spec type: {type(kv_cache_spec)}"
                )

    if has_attn and has_mamba:
        _update_hybrid_attention_layout(kv_caches, kv_cache_config)

    return kv_caches
```
**EN:** This function implements `_reshape_kv_cache` within the module. Key calls include `_update_hybrid_attention_layout`, `isinstance`, `numel`, `get_kv_cache_shape`, `tuple`, `view`. The control flow contains 6 branch(es) and 3 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `_reshape_kv_cache`，其作用域位于the module。 关键调用包括 `_update_hybrid_attention_layout`, `isinstance`, `numel`, `get_kv_cache_shape`, `tuple`, `view`。 控制流包含 6 个分支和 3 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `_update_hybrid_attention_layout` function / `_update_hybrid_attention_layout` 函数
```python
def _update_hybrid_attention_layout(
    kv_caches: dict[str, Any],
    kv_cache_config: KVCacheConfig,
) -> None:
    for kv_cache_group_spec in kv_cache_config.kv_cache_groups:
        for layer_name in kv_cache_group_spec.layer_names:
            kv_cache_spec = kv_cache_group_spec.kv_cache_spec
            if isinstance(kv_cache_spec, UniformTypeKVCacheSpecs):
                kv_cache_spec = kv_cache_spec.kv_cache_specs[layer_name]
            if not isinstance(kv_cache_spec, AttentionSpec):
                continue
            kv_cache = kv_caches[layer_name]
            if kv_cache.shape[0] == 2:
                assert kv_cache.shape[1] != 2, (
                    f"Cannot determine layout for tensor of shape {kv_cache.shape}"
                )
                hidden_size = kv_cache.shape[2:].numel()
                kv_cache.as_strided_(
                    size=kv_cache.shape,
                    stride=(
                        hidden_size,
                        2 * hidden_size,
                        *kv_cache.stride()[2:],
                    ),
                )
```
**EN:** This function implements `_update_hybrid_attention_layout` within the module. Key calls include `isinstance`, `numel`, `as_strided_`, `stride`. The control flow contains 3 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_update_hybrid_attention_layout`，其作用域位于the module。 关键调用包括 `isinstance`, `numel`, `as_strided_`, `stride`。 控制流包含 3 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `init_kv_cache` function / `init_kv_cache` 函数
```python
def init_kv_cache(
    runner_kv_caches: list[torch.Tensor],
    forward_context: dict[str, Any],
    kv_cache_config: KVCacheConfig,
    attn_backends: dict[str, type[AttentionBackend]],
    device: torch.device,
    cache_dtype: str,
) -> dict[str, Any]:
    kv_cache_raw_tensors = _allocate_kv_cache(kv_cache_config, device)
    kv_caches = _reshape_kv_cache(
        kv_cache_config, kv_cache_raw_tensors, attn_backends, cache_dtype
    )
    bind_kv_cache(kv_caches, forward_context, runner_kv_caches)
    return kv_caches
```
**EN:** This function implements `init_kv_cache` within the module. Key calls include `_allocate_kv_cache`, `_reshape_kv_cache`, `bind_kv_cache`.
**CN:** 该函数会实现 `init_kv_cache`，其作用域位于the module。 关键调用包括 `_allocate_kv_cache`, `_reshape_kv_cache`, `bind_kv_cache`。

### `build_slot_mappings_by_layer` function / `build_slot_mappings_by_layer` 函数
```python
def build_slot_mappings_by_layer(
    slot_mappings: torch.Tensor, kv_cache_config: KVCacheConfig
) -> dict[str, torch.Tensor]:
    slot_mappings_by_layer: dict[str, torch.Tensor] = {}
    kv_cache_groups = kv_cache_config.kv_cache_groups
    for slot_mapping, kv_cache_group in zip(slot_mappings, kv_cache_groups):
        for layer_name in kv_cache_group.layer_names:
            slot_mappings_by_layer[layer_name] = slot_mapping
    return slot_mappings_by_layer
```
**EN:** This function builds derived structures within the module. Key calls include `zip`. The control flow contains 0 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会构建派生结构，其作用域位于the module。 关键调用包括 `zip`。 控制流包含 0 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `build_attn_metadata` function / `build_attn_metadata` 函数
```python
def build_attn_metadata(
    attn_groups: list[list[AttentionGroup]],
    num_reqs: int,
    num_tokens: int,
    query_start_loc_gpu: torch.Tensor,
    query_start_loc_cpu: torch.Tensor,
    max_query_len: int,
    seq_lens: torch.Tensor,
    max_seq_len: int,
    block_tables: Sequence[torch.Tensor],
    slot_mappings: torch.Tensor,
    kv_cache_config: KVCacheConfig,
    seq_lens_cpu_upper_bound: torch.Tensor | None = None,
    dcp_local_seq_lens: torch.Tensor | None = None,
    positions: torch.Tensor | None = None,
    model_specific_attn_metadata: ModelSpecificAttnMetadata | None = None,
    for_cudagraph_capture: bool = False,
) -> dict[str, Any]:
    seq_lens = seq_lens[:num_reqs]
    if dcp_local_seq_lens is not None:
        dcp_local_seq_lens = dcp_local_seq_lens[:num_reqs]
    if seq_lens_cpu_upper_bound is not None:
        seq_lens_cpu_upper_bound = seq_lens_cpu_upper_bound[:num_reqs]

    attn_metadata: dict[str, Any] = {}
    num_kv_cache_groups = len(kv_cache_config.kv_cache_groups)
    for i in range(num_kv_cache_groups):
        block_table = block_tables[i]
        slot_mapping = slot_mappings[i]

        common_attn_metadata_extra_kwargs = (
            model_specific_attn_metadata.get_extra_common_attn_kwargs(i, num_reqs)
            if model_specific_attn_metadata is not None
            else {}
        )
        common_attn_metadata = CommonAttentionMetadata(
            query_start_loc=query_start_loc_gpu,
            query_start_loc_cpu=query_start_loc_cpu,
            seq_lens=seq_lens,
            seq_lens_cpu_upper_bound=seq_lens_cpu_upper_bound,
            max_seq_len=max_seq_len,
            num_reqs=num_reqs,
            num_actual_tokens=num_tokens,
            max_query_len=max_query_len,
            block_table_tensor=block_table,
            slot_mapping=slot_mapping,
            causal=True,
            dcp_local_seq_lens=dcp_local_seq_lens,
            positions=positions,
            **common_attn_metadata_extra_kwargs,
        )

        for attn_group in attn_groups[i]:
            attn_metadata_builder = attn_group.get_metadata_builder(0)
            if for_cudagraph_capture:
                metadata = attn_metadata_builder.build_for_cudagraph_capture(
                    common_attn_metadata
                )
            else:
                attn_metadata_extra_kwargs = (
                    model_specific_attn_metadata.get_extra_attn_kwargs(
                        attn_metadata_builder,
                        num_reqs,
                    )
                    if model_specific_attn_metadata is not None
                    else {}
                )
                metadata = attn_metadata_builder.build(
                    common_prefix_len=0,
                    common_attn_metadata=common_attn_metadata,
                    **attn_metadata_extra_kwargs,
                )
            for layer_name in attn_group.layer_names:
                attn_metadata[layer_name] = metadata
    return attn_metadata
```
**EN:** This function builds derived structures within the module. Key calls include `len`, `range`, `CommonAttentionMetadata`, `get_extra_common_attn_kwargs`, `get_metadata_builder`, `build_for_cudagraph_capture`. The control flow contains 5 branch(es) and 3 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会构建派生结构，其作用域位于the module。 关键调用包括 `len`, `range`, `CommonAttentionMetadata`, `get_extra_common_attn_kwargs`, `get_metadata_builder`, `build_for_cudagraph_capture`。 控制流包含 5 个分支和 3 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `AttentionCGSupportInfo`: central class or interface in this module. / `AttentionCGSupportInfo`：本模块中的核心类或接口。
- `get_kv_cache_spec`: top-level helper or orchestration entry point. / `get_kv_cache_spec`：顶层辅助函数或编排入口。
- `init_attn_backend`: top-level helper or orchestration entry point. / `init_attn_backend`：顶层辅助函数或编排入口。
- `_allocate_kv_cache`: top-level helper or orchestration entry point. / `_allocate_kv_cache`：顶层辅助函数或编排入口。
- `_reshape_kv_cache`: top-level helper or orchestration entry point. / `_reshape_kv_cache`：顶层辅助函数或编排入口。
- `_update_hybrid_attention_layout`: top-level helper or orchestration entry point. / `_update_hybrid_attention_layout`：顶层辅助函数或编排入口。
- `init_kv_cache`: top-level helper or orchestration entry point. / `init_kv_cache`：顶层辅助函数或编排入口。
- `build_slot_mappings_by_layer`: top-level helper or orchestration entry point. / `build_slot_mappings_by_layer`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `collections`, `dataclasses`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.model_executor.layers.attention_layer_base`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend`, `vllm.v1.kv_cache_interface`, `vllm.v1.worker.gpu.model_states.interface`, `vllm.v1.worker.utils`
