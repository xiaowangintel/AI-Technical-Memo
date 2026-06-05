# compressor.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/dsv4/compressor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module manages compressor logic around KV-cache layout, indexing, or paging for SGLang attention execution. / 该模块管理与 compressor 相关的 KV 缓存布局、索引或分页逻辑，用于 SGLang 注意力执行。
## Line-by-Line Analysis / 逐行分析
### Lines 1-29: imports
```python
from __future__ import annotations

from typing import TYPE_CHECKING, List, Literal, NamedTuple, Optional, Union

import torch
import torch.nn as nn

from sglang.jit_kernel.deepseek_v4 import (
    CompressorDecodePlan,
    CompressorPrefillPlan,
    compress_forward,
    compress_fused_norm_rope_inplace,
    linear_bf16_fp32,
    triton_create_paged_compress_data,
)
from sglang.srt.configs.deepseek_v4 import DeepSeekV4Config
from sglang.srt.environ import envs
from sglang.srt.layers.attention.dsv4.quant_k_cache import (
    quant_to_nope_fp8_rope_bf16_pack_triton,
)
from sglang.srt.layers.attention.nsa.triton_kernel import act_quant
from sglang.srt.layers.attention.nsa.utils import nsa_use_prefill_cp
from sglang.srt.layers.dp_attention import get_attention_cp_size
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.linear import ReplicatedLinear
from sglang.srt.layers.utils.cp_utils import cp_all_gather_rerange_output
from sglang.srt.mem_cache.deepseek_v4_compress_state import CompressStatePool
from sglang.srt.mem_cache.deepseek_v4_memory_pool import DeepSeekV4TokenToKVPool
from sglang.srt.utils import add_prefix
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 31-33: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.layers.attention.deepseek_v4_backend import DeepseekV4AttnBackend
    from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 36-40: class FusedCompressMetadata
```python
class FusedCompressMetadata(NamedTuple):
    write_loc: torch.Tensor
    extra_data: Optional[torch.Tensor]
    plan: Union[CompressorDecodePlan, CompressorPrefillPlan]
```
**EN:** Named tuple that packages immutable metadata related to fused compress metadata.
**CN:** 该具名元组用于封装与 fused compress metadata 相关的不可变元数据。

### Lines 41-46: method FusedCompressMetadata.copy_
```python
    def copy_(self, other: FusedCompressMetadata) -> None:
        from .metadata import maybe_copy_inplace

        self.write_loc.copy_(other.write_loc)
        maybe_copy_inplace(self.extra_data, src=other.extra_data)
        self.plan.copy_(other.plan)
```
**EN:** Implements the copy routine used by this attention module.
**CN:** 实现该注意力模块使用的 copy 例程。

### Lines 49-49: class CompressorBackendMixin
```python
class CompressorBackendMixin:
```
**EN:** Concrete attention backend that connects compressor backend mixin to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 compressor backend mixin 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 50-54: method CompressorBackendMixin.get_paged_compress_metadata
```python
    def get_paged_compress_metadata(self, compress_ratio: int) -> FusedCompressMetadata:
        attr_name = f"c{compress_ratio}_compress_metadata"
        metadata = getattr(self.forward_metadata, attr_name)
        assert isinstance(metadata, FusedCompressMetadata)
        return metadata
```
**EN:** Computes and returns get paged compress metadata from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get paged compress metadata。

### Lines 56-107: method CompressorBackendMixin.forward_compress
```python
    def forward_compress(
        self,
        *,
        kv_score_buffer: torch.Tensor,
        kv_score_input: torch.Tensor,
        ape: torch.Tensor,
        head_dim: int,
        norm: RMSNorm,
        freqs_cis_cache: torch.Tensor,
        rotate: bool,
        forward_batch: ForwardBatch,
        compress_ratio: int,
        is_paged: bool = False,
    ) -> torch.Tensor:
        from sglang.srt.layers.attention.nsa.nsa_indexer import rotate_activation

        assert compress_ratio in (
            4,
            128,
        ), f"DSV4 supports CSA(4x) and HCA(128x) only, got {compress_ratio=}"
        if is_paged:
            metadata = self.get_paged_compress_metadata(compress_ratio)
            coff = 2 if is_overlap_compress(compress_ratio) else 1
            if compress_ratio == 128 and envs.SGLANG_OPT_USE_ONLINE_COMPRESS.get():
                kv_score_buffer = kv_score_buffer.view(-1, 1, head_dim * 3)
            else:
                last_dim = 2 * head_dim * coff
                assert kv_score_buffer.shape[-1] == last_dim
                kv_score_buffer = kv_score_buffer.view(-1, compress_ratio, last_dim)
        else:
            plan = make_compressor_plan(compress_ratio, forward_batch)
            metadata = (forward_batch.req_pool_indices.to(torch.int32), None, plan)
# ... omitted 8 lines ...
            compress_ratio=compress_ratio,
            head_dim=head_dim,
            extra_data=extra_data,
        )
        compress_fused_norm_rope_inplace(
            kv_compressed,
            norm.weight,
            norm.variance_epsilon,
            freqs_cis_cache,
            plan,
        )
        return rotate_activation(kv_compressed) if rotate else kv_compressed
```
**EN:** Runs the forward-path logic for forward compress, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward compress 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 109-142: method CompressorBackendMixin.forward_core_compressor
```python
    def forward_core_compressor(
        self,
        x: torch.Tensor,
        forward_batch: ForwardBatch,
        layer_id: int,
        compressor: Compressor,
    ) -> None:
        if forward_batch.forward_mode.is_idle():
            return
        # PREP_IN_CG lazy upgrade: the concrete backend (DeepseekV4AttnBackend)
        # owns this helper. MQALayer._forward_prepare calls us before
        # attn_backend.forward(), so Raw -> DSV4Metadata must happen here too
        # (e.g. 1.6T layer 0 has compress_ratio=128 and needs cX_compress_metadata).
        self._maybe_upgrade_forward_metadata()
        token_to_kv_pool = forward_batch.token_to_kv_pool
        if TYPE_CHECKING:
            assert isinstance(token_to_kv_pool, DeepSeekV4TokenToKVPool)

        new_compressed_kv = compressor(x, forward_batch)
        core_metadata = self.forward_metadata.core_metadata
        out_loc = (
            core_metadata.c4_out_loc
            if compressor.ratio == 4
            else core_metadata.c128_out_loc
        )
        if envs.SGLANG_OPT_USE_FUSED_STORE_CACHE.get():
            token_to_kv_pool.set_extra_key_buffer_fused(
                layer_id=layer_id,
                loc=out_loc,
                cache_k=new_compressed_kv,
            )
        else:
            pack = quant_to_nope_fp8_rope_bf16_pack_triton(new_compressed_kv.bfloat16())
            token_to_kv_pool.set_extra_key_buffer(layer_id, out_loc, pack)
```
**EN:** Runs the forward-path logic for forward core compressor, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward core compressor 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 144-174: method CompressorBackendMixin.forward_indexer_compressor
```python
    def forward_indexer_compressor(
        self,
        x: torch.Tensor,
        forward_batch: ForwardBatch,
        layer_id: int,
        compressor: Compressor,
    ) -> None:
        assert is_overlap_compress(compressor.ratio)
        # PREP_IN_CG lazy upgrade (see forward_core_compressor for rationale).
        self._maybe_upgrade_forward_metadata()
        token_to_kv_pool = forward_batch.token_to_kv_pool
        if TYPE_CHECKING:
            assert isinstance(token_to_kv_pool, DeepSeekV4TokenToKVPool)

        new_compressed_kv = compressor(x, forward_batch)
        if envs.SGLANG_OPT_USE_FUSED_STORE_CACHE.get():
            token_to_kv_pool.set_index_k_fused(
                layer_id=layer_id,
                loc=self.forward_metadata.core_metadata.c4_out_loc,
                cache_k=new_compressed_kv,
            )
        else:
            new_compressed_kv_fp8, new_compressed_kv_scale = act_quant(
                new_compressed_kv
            )
            token_to_kv_pool.set_index_k_scale_buffer(
                layer_id=layer_id,
                loc=self.forward_metadata.core_metadata.c4_out_loc,
                index_k=new_compressed_kv_fp8,
                index_k_scale=new_compressed_kv_scale,
            )
```
**EN:** Runs the forward-path logic for forward indexer compressor, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward indexer compressor 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 177-178: function is_overlap_compress
```python
def is_overlap_compress(compress_ratio: int) -> bool:
    return compress_ratio == 4
```
**EN:** Checks whether is overlap compress is valid for the current configuration or execution state.
**CN:** 检查在当前配置或执行状态下 is overlap compress 是否成立。

### Lines 181-203: function make_compressor_plan
```python
def make_compressor_plan(
    compress_ratio: Literal[4, 128],
    forward_batch: ForwardBatch,
) -> Union[CompressorDecodePlan, CompressorPrefillPlan]:
    if forward_batch.forward_mode.is_decode():
        seq_lens_32 = forward_batch.seq_lens.to(torch.int32)
        return CompressorDecodePlan(compress_ratio, seq_lens_32)
    if forward_batch.forward_mode.is_prefill():
        assert not forward_batch.forward_mode.is_target_verify()
        extend_lens_list = forward_batch.extend_seq_lens_cpu
        seq_lens_cpu = forward_batch.seq_lens_cpu
        assert extend_lens_list is not None and seq_lens_cpu is not None
        return CompressorPrefillPlan.generate(
            compress_ratio=compress_ratio,
            num_q_tokens=sum(extend_lens_list),
            seq_lens=seq_lens_cpu,
            extend_lens=torch.tensor(extend_lens_list),
            device=forward_batch.seq_lens.device,
        )
    elif forward_batch.forward_mode.is_target_verify():
        raise NotImplementedError("target verify mode to be implemented")
    else:
        raise NotImplementedError(f"unsupported mode {forward_batch.forward_mode=}")
```
**EN:** Factory helper that constructs make compressor plan and validates the prerequisites needed for this execution path.
**CN:** 该工厂辅助函数构建 make compressor plan，并校验此执行路径所需的前置条件。

### Lines 206-282: function create_paged_compressor_data
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
) -> FusedCompressMetadata:
    swa_page_size = token_to_kv_pool.swa_page_size
    ring_size = token_to_kv_pool.get_ring_size(compress_ratio=compress_ratio)
    # assert ring_size % compress_ratio == 0

    def clip_down(positions: torch.Tensor) -> torch.Tensor:
        return positions // compress_ratio * compress_ratio

    def get_raw_loc(positions: torch.Tensor) -> torch.Tensor:
        positions = positions.masked_fill(positions < 0, 0)
        loc = req_to_token[req_pool_indices, positions]
        swa_loc = token_to_kv_pool.translate_loc_from_full_to_swa(loc)
        swa_pages = swa_loc // swa_page_size
        state_loc = swa_pages * ring_size + swa_loc % ring_size
        return (state_loc // compress_ratio).to(torch.int32)

    is_overlap = is_overlap_compress(compress_ratio)

    if is_prefill:
# ... omitted 33 lines ...
        )
    else:
        write_positions = clip_down(seq_lens - 1)
        write_loc = get_raw_loc(write_positions)
        if is_overlap:
            write_overlap_loc = get_raw_loc(write_positions - compress_ratio)
            extra_data = write_overlap_loc.view(-1, 1)
        else:
            extra_data = None
        plan = CompressorDecodePlan(compress_ratio, seq_lens.to(torch.int32))

    return FusedCompressMetadata(write_loc=write_loc, extra_data=extra_data, plan=plan)
```
**EN:** Factory helper that constructs create paged compressor data and validates the prerequisites needed for this execution path.
**CN:** 该工厂辅助函数构建 create paged compressor data，并校验此执行路径所需的前置条件。

### Lines 285-285: class Compressor
```python
class Compressor(nn.Module):
```
**EN:** Defines the compressor type and the state it exposes to the rest of the attention stack.
**CN:** 定义 compressor 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 286-326: method Compressor.__init__
```python
    def __init__(
        self,
        config: DeepSeekV4Config,
        layer_id: int,
        is_in_indexer: bool,
        freqs_cis: torch.Tensor,
        compress_ratio: Literal[0, 4, 128],
        head_dim: int,
        rotate: bool = False,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.layer_id = layer_id
        self.is_in_indexer = is_in_indexer
        self.dim = config.hidden_size
        self.head_dim = head_dim
        self.rope_head_dim = getattr(config, "qk_rope_head_dim", 64)
        assert compress_ratio != 0, "compress_ratio should not be 0"
        self.ratio = compress_ratio
        self.overlap = self.ratio == 4
        self.rotate = rotate
        coff = 1 + self.overlap

        self.ape = nn.Parameter(
            torch.empty(self.ratio, coff * self.head_dim, dtype=torch.float32)
        )
        wkv_gate_dtype = torch.bfloat16
        self.wkv_gate = ReplicatedLinear(
            self.dim,
            2 * coff * self.head_dim,
            bias=False,
            quant_config=None,
            prefix=add_prefix("wkv_gate", prefix),
            params_dtype=wkv_gate_dtype,
        )
        self.norm = RMSNorm(
            self.head_dim, eps=config.rms_norm_eps, weight_dtype=torch.float32
        )
        self.freqs_cis = freqs_cis

        self.ape_converted = False
```
**EN:** Initializes the Compressor instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 Compressor 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 328-335: method Compressor.apply_ape_hotfix
```python
    def apply_ape_hotfix(self):
        assert not self.ape_converted
        self.ape_converted = True

        if self.overlap:
            ape = torch.chunk(self.ape.data, 2, dim=-1)
            ape = torch.cat([ape[0], ape[1]], dim=0)
            self.ape.data.copy_(ape.view(self.ratio, -1))
```
**EN:** Implements the apply ape hotfix routine used by this attention module.
**CN:** 实现该注意力模块使用的 apply ape hotfix 例程。

### Lines 338-348: method Compressor.get_state_pool
```python
    def get_state_pool(self, forward_batch: ForwardBatch) -> CompressStatePool:
        token_to_kv_pool = forward_batch.token_to_kv_pool
        assert isinstance(token_to_kv_pool, DeepSeekV4TokenToKVPool)
        if self.is_in_indexer:
            ret = token_to_kv_pool.get_indexer_compress_states(self.layer_id)
        else:
            ret = token_to_kv_pool.get_attention_compress_states(self.layer_id)

        assert isinstance(ret, CompressStatePool)

        return ret
```
**EN:** Computes and returns get state pool from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get state pool。

### Lines 351-360: method Compressor.compute_kv_score
```python
    def compute_kv_score(self, x: torch.Tensor, forward_batch: ForwardBatch):
        kv_score = linear_bf16_fp32(x, self.wkv_gate.weight)
        if nsa_use_prefill_cp(forward_batch):
            kv_score = cp_all_gather_rerange_output(
                kv_score,
                get_attention_cp_size(),
                forward_batch,
                torch.cuda.current_stream(),
            )
        return kv_score
```
**EN:** Implements the compute kv score routine used by this attention module.
**CN:** 实现该注意力模块使用的 compute kv score 例程。

### Lines 362-385: method Compressor.forward
```python
    def forward(self, x: torch.Tensor, forward_batch: ForwardBatch) -> torch.Tensor:
        if forward_batch.forward_mode.is_idle():
            assert x.shape[0] == 0
            return x.new_empty(0, self.head_dim)

        kv_score = self.compute_kv_score(x, forward_batch)

        backend = forward_batch.attn_backend
        if TYPE_CHECKING:
            assert isinstance(backend, DeepseekV4AttnBackend)
        kv_score_buffer = self.get_state_pool(forward_batch)
        kv_score_buffer = kv_score_buffer.kv_score_buffer.kv_score
        return backend.forward_compress(
            kv_score_buffer=kv_score_buffer,
            kv_score_input=kv_score,
            ape=self.ape.view(-1, self.head_dim),
            head_dim=self.head_dim,
            norm=self.norm,
            freqs_cis_cache=self.freqs_cis,
            rotate=self.rotate,
            compress_ratio=self.ratio,
            forward_batch=forward_batch,
            is_paged=True,
        )
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Paged attention layouts / **CN:** 分页注意力布局

## Dependencies / 依赖关系
- `__future__.annotations`
- `typing.TYPE_CHECKING`
- `typing.List`
- `typing.Literal`
- `typing.NamedTuple`
- `typing.Optional`
- `typing.Union`
- `torch`
- `torch.nn`
- `sglang.jit_kernel.deepseek_v4.CompressorDecodePlan`
- `sglang.jit_kernel.deepseek_v4.CompressorPrefillPlan`
- `sglang.jit_kernel.deepseek_v4.compress_forward`
- `sglang.jit_kernel.deepseek_v4.compress_fused_norm_rope_inplace`
- `sglang.jit_kernel.deepseek_v4.linear_bf16_fp32`
- `sglang.jit_kernel.deepseek_v4.triton_create_paged_compress_data`
- `sglang.srt.configs.deepseek_v4.DeepSeekV4Config`
- `sglang.srt.environ.envs`
- `sglang.srt.layers.attention.dsv4.quant_k_cache.quant_to_nope_fp8_rope_bf16_pack_triton`
- `sglang.srt.layers.attention.nsa.triton_kernel.act_quant`
- `sglang.srt.layers.attention.nsa.utils.nsa_use_prefill_cp`
- `sglang.srt.layers.dp_attention.get_attention_cp_size`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.linear.ReplicatedLinear`
- `sglang.srt.layers.utils.cp_utils.cp_all_gather_rerange_output`
- `sglang.srt.mem_cache.deepseek_v4_compress_state.CompressStatePool`
- `sglang.srt.mem_cache.deepseek_v4_memory_pool.DeepSeekV4TokenToKVPool`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.layers.attention.deepseek_v4_backend.DeepseekV4AttnBackend`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `.metadata.maybe_copy_inplace`
- `sglang.srt.layers.attention.nsa.nsa_indexer.rotate_activation`
