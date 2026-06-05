# compressor_v2.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/dsv4/compressor_v2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module manages compressor v2 logic around KV-cache layout, indexing, or paging for SGLang attention execution. / 该模块管理与 compressor v2 相关的 KV 缓存布局、索引或分页逻辑，用于 SGLang 注意力执行。
## Line-by-Line Analysis / 逐行分析
### Lines 1-13: imports
```python
from __future__ import annotations

from typing import TYPE_CHECKING, List, Literal, Optional, TypeAlias, Union, cast

import torch

from sglang.jit_kernel.dsv4 import (
    CompressorDecodePlan,
    CompressorPrefillPlan,
    compress_forward,
    compress_norm_rope_store,
)
from sglang.srt.environ import envs
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 15-20: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.layers.attention.deepseek_v4_backend import DSV4Metadata
    from sglang.srt.layers.attention.dsv4.compressor import Compressor
    from sglang.srt.layers.layernorm import RMSNorm
    from sglang.srt.mem_cache.deepseek_v4_memory_pool import DeepSeekV4TokenToKVPool
    from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 23-25: module constants
```python
CompressMetadata: TypeAlias = Union[CompressorDecodePlan, CompressorPrefillPlan]
# NOTE: alias for backward compatibility
FusedCompressMetadata: TypeAlias = CompressMetadata
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 28-30: function _use_online_compress
```python
def _use_online_compress(compress_ratio: int) -> bool:
    """Online state-pool path is c128-only."""
    return compress_ratio == 128 and envs.SGLANG_OPT_USE_ONLINE_COMPRESS.get()
```
**EN:** Implements the use online compress routine used by this attention module.
**CN:** 实现该注意力模块使用的 use online compress 例程。

### Lines 33-33: class CompressorBackendMixin
```python
class CompressorBackendMixin:
```
**EN:** Concrete attention backend that connects compressor backend mixin to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 compressor backend mixin 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 34-36: method CompressorBackendMixin.__init__
```python
    def __init__(self):
        super().__init__()
        self.forward_metadata: DSV4Metadata
```
**EN:** Initializes the CompressorBackendMixin instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 CompressorBackendMixin 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 39-39: method CompressorBackendMixin._maybe_upgrade_forward_metadata
```python
    def _maybe_upgrade_forward_metadata(self): ...
```
**EN:** Runs the forward-path logic for maybe upgrade forward metadata, transforming tensors and dispatching the required compute steps.
**CN:** 执行 maybe upgrade forward metadata 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 41-43: method CompressorBackendMixin._get_paged_compress_metadata
```python
    def _get_paged_compress_metadata(self, compress_ratio: int) -> CompressMetadata:
        attr_name = f"c{compress_ratio}_compress_metadata"
        return getattr(self.forward_metadata, attr_name)
```
**EN:** Implements the get paged compress metadata routine used by this attention module.
**CN:** 实现该注意力模块使用的 get paged compress metadata 例程。

### Lines 45-47: method CompressorBackendMixin._get_out_loc
```python
    def _get_out_loc(self, compress_ratio: int) -> torch.Tensor:
        attr_name = f"c{compress_ratio}_out_loc"
        return getattr(self.forward_metadata.core_metadata, attr_name)
```
**EN:** Implements the get out loc routine used by this attention module.
**CN:** 实现该注意力模块使用的 get out loc 例程。

### Lines 49-95: method CompressorBackendMixin._forward_compress_all_in_one
```python
    def _forward_compress_all_in_one(
        self,
        *,
        kv_score_buffer: torch.Tensor,
        kv_score_input: torch.Tensor,
        ape: torch.Tensor,
        head_dim: int,
        norm: RMSNorm,
        freqs_cis_cache: torch.Tensor,
        kv_cache: torch.Tensor,
        is_indexer: bool,
        rotate: bool,
        compress_ratio: int,
        page_size: int,
    ) -> None:
        assert compress_ratio == 4 or compress_ratio == 128
        assert rotate == is_indexer == (head_dim == 128)

        plan = self._get_paged_compress_metadata(compress_ratio)
        is_online = _use_online_compress(compress_ratio)
        if is_online:
            kv_score_buffer = kv_score_buffer.view(-1, 1, head_dim * 3)
        else:
            coff = 2 if is_overlap_compress(compress_ratio) else 1
            last_dim = 2 * head_dim * coff
            assert kv_score_buffer.shape[-1] == last_dim
            kv_score_buffer = kv_score_buffer.view(-1, compress_ratio, last_dim)
        kv_compressed = compress_forward(
            kv_score_buffer=kv_score_buffer,
            kv_score_input=kv_score_input,
            ape=ape.view(-1, head_dim),
            plan=plan,
            compress_ratio=compress_ratio,
            head_dim=head_dim,
            is_online=is_online,
        )
        # NOTE: we use some hack here...
        compress_norm_rope_store(
            kv_compressed,
            plan,
            norm_weight=norm.weight,
            norm_eps=norm.variance_epsilon,
            freq_cis=freqs_cis_cache,
            out_loc=self._get_out_loc(compress_ratio),
            kvcache=kv_cache,
            page_size=page_size,
        )
```
**EN:** Runs the forward-path logic for forward compress all in one, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward compress all in one 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 97-130: method CompressorBackendMixin.forward_unified
```python
    def forward_unified(
        self,
        x: torch.Tensor,
        forward_batch: ForwardBatch,
        layer_id: int,
        compressor: Compressor,
    ) -> None:
        if forward_batch.forward_mode.is_idle():
            return

        self._maybe_upgrade_forward_metadata()
        token_to_kv_pool = forward_batch.token_to_kv_pool
        token_to_kv_pool = cast("DeepSeekV4TokenToKVPool", token_to_kv_pool)
        kv_score_input = compressor.compute_kv_score(x, forward_batch)
        state_pool = compressor.get_state_pool(forward_batch)
        if compressor.is_in_indexer:
            kv_cache = token_to_kv_pool.get_index_k_with_scale_buffer(layer_id)
            page_size = token_to_kv_pool.get_index_k_page_size()
        else:
            kv_cache = token_to_kv_pool.get_extra_key_buffer(layer_id)
            page_size = token_to_kv_pool.get_extra_key_page_size(layer_id)
        self._forward_compress_all_in_one(
            kv_score_buffer=state_pool.kv_score_buffer.kv_score,
            kv_score_input=kv_score_input,
            ape=compressor.ape,
            head_dim=compressor.head_dim,
            norm=compressor.norm,
            freqs_cis_cache=compressor.freqs_cis,
            kv_cache=kv_cache.view(dtype=torch.uint8),
            is_indexer=compressor.is_in_indexer,
            rotate=compressor.rotate,
            compress_ratio=compressor.ratio,
            page_size=page_size,
        )
```
**EN:** Runs the forward-path logic for forward unified, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward unified 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 133-134: CompressorBackendMixin fields
```python
    forward_indexer_compressor = forward_unified
    forward_core_compressor = forward_unified
```
**EN:** Defines class-level fields, defaults, or reusable constants consumed by later methods.
**CN:** 定义后续方法会使用的类级字段、默认值或可复用常量。

### Lines 137-138: function is_overlap_compress
```python
def is_overlap_compress(compress_ratio: int) -> bool:
    return compress_ratio == 4
```
**EN:** Checks whether is overlap compress is valid for the current configuration or execution state.
**CN:** 检查在当前配置或执行状态下 is overlap compress 是否成立。

### Lines 141-214: function create_paged_compressor_data
```python
def create_paged_compressor_data(
    compress_ratio: Literal[4, 128],
    *,
    is_prefill: bool,
    token_to_kv_pool: DeepSeekV4TokenToKVPool,
    req_to_token: torch.Tensor,
    req_pool_indices: torch.Tensor,
    seq_lens: torch.Tensor,
    extend_lens: Optional[torch.Tensor] = None,
    seq_lens_cpu: Optional[List[int]] = None,
    extend_lens_cpu: Optional[List[int]] = None,
    use_prefill_cuda_graph: bool = False,
    num_q_tokens: Optional[int] = None,
) -> CompressMetadata:
    """Build the paged compress metadata (= the plan).

    State-pool slot translation is done inside the C++ planner; the
    Python side just hands the relevant tensors over.
    """
    if _use_online_compress(compress_ratio):
        return _create_online_paged_compressor_data(
            is_prefill=is_prefill,
            token_to_kv_pool=token_to_kv_pool,
            req_to_token=req_to_token,
            req_pool_indices=req_pool_indices,
            seq_lens=seq_lens,
            extend_lens=extend_lens,
            seq_lens_cpu=seq_lens_cpu,
            extend_lens_cpu=extend_lens_cpu,
            use_prefill_cuda_graph=use_prefill_cuda_graph,
            num_q_tokens=num_q_tokens,
        )
# ... omitted 30 lines ...
            use_cuda_graph=use_prefill_cuda_graph,
        )
    else:
        return CompressorDecodePlan.generate(
            compress_ratio=compress_ratio,
            req_pool_indices=req_pool_indices_i64,
            req_to_token=req_to_token,
            full_to_swa=full_to_swa,
            seq_lens=seq_lens.to(torch.int64),
            swa_page_size=swa_page_size,
            ring_size=ring_size,
        )
```
**EN:** Factory helper that constructs create paged compressor data and validates the prerequisites needed for this execution path.
**CN:** 该工厂辅助函数构建 create paged compressor data，并校验此执行路径所需的前置条件。

### Lines 217-267: function _create_online_paged_compressor_data
```python
def _create_online_paged_compressor_data(
    *,
    is_prefill: bool,
    token_to_kv_pool: DeepSeekV4TokenToKVPool,
    req_to_token: torch.Tensor,
    req_pool_indices: torch.Tensor,
    seq_lens: torch.Tensor,
    extend_lens: Optional[torch.Tensor],
    seq_lens_cpu: Optional[List[int]],
    extend_lens_cpu: Optional[List[int]],
    use_prefill_cuda_graph: bool,
    num_q_tokens: Optional[int],
) -> CompressMetadata:
    assert not use_prefill_cuda_graph, "online c128 doesn't support cuda graph"

    swa_page_size = int(token_to_kv_pool.swa_page_size)
    full_to_swa = token_to_kv_pool.full_to_swa_index_mapping.detach()
    req_pool_indices = req_pool_indices.to(torch.int64)

    if is_prefill:
        # Sync-on-entry: catch IMA from a prior layer / kernel BEFORE we touch
        # anything in this builder, so blame doesn't land on us spuriously.
        assert extend_lens is not None
        if seq_lens_cpu is not None:
            assert extend_lens_cpu is not None
            seq_lens_planner = torch.tensor(seq_lens_cpu, dtype=torch.int64)
            extend_lens_planner = torch.tensor(extend_lens_cpu, dtype=torch.int64)
            num_q_tokens_planner = sum(extend_lens_cpu)
        else:
            assert num_q_tokens is not None
            seq_lens_planner = seq_lens.to(torch.int64)
            extend_lens_planner = extend_lens.to(torch.int64)
# ... omitted 7 lines ...
            full_to_swa=full_to_swa,
            num_q_tokens=int(num_q_tokens_planner),
            swa_page_size=swa_page_size,
        )
    else:
        return CompressorDecodePlan.generate_online(
            seq_lens=seq_lens.to(torch.int64),
            req_pool_indices=req_pool_indices,
            req_to_token=req_to_token,
            full_to_swa=full_to_swa,
            swa_page_size=swa_page_size,
        )
```
**EN:** Implements the create online paged compressor data routine used by this attention module.
**CN:** 实现该注意力模块使用的 create online paged compressor data 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Paged attention layouts / **CN:** 分页注意力布局
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理

## Dependencies / 依赖关系
- `__future__.annotations`
- `typing.TYPE_CHECKING`
- `typing.List`
- `typing.Literal`
- `typing.Optional`
- `typing.TypeAlias`
- `typing.Union`
- `typing.cast`
- `torch`
- `sglang.jit_kernel.dsv4.CompressorDecodePlan`
- `sglang.jit_kernel.dsv4.CompressorPrefillPlan`
- `sglang.jit_kernel.dsv4.compress_forward`
- `sglang.jit_kernel.dsv4.compress_norm_rope_store`
- `sglang.srt.environ.envs`
- `sglang.srt.layers.attention.deepseek_v4_backend.DSV4Metadata`
- `sglang.srt.layers.attention.dsv4.compressor.Compressor`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.mem_cache.deepseek_v4_memory_pool.DeepSeekV4TokenToKVPool`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
