# deepseek_v4_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/deepseek_v4_attention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: DeepseekV4 MLA Attention Layer / DeepSeek 专用压缩或注意力逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 7-86 — imports and setup
```python
from collections.abc import Callable
from dataclasses import dataclass
from typing import TYPE_CHECKING, Any, cast

import torch
import torch.nn as nn
import torch.nn.functional as F
from transformers import DeepseekV2Config, DeepseekV3Config

import vllm.envs as envs
from vllm.compilation.breakable_cudagraph import eager_break_during_capture
from vllm.model_executor.layers.linear import (
    ReplicatedLinear,
)
from vllm.model_executor.layers.sparse_attn_indexer import SparseAttnIndexer
from vllm.utils.deep_gemm import fp8_einsum
from vllm.utils.torch_utils import direct_register_custom_op
from vllm.v1.attention.ops.deepseek_v4_ops import (
    combine_topk_swa_indices,
    compute_global_topk_indices_and_lens,
    dequantize_and_gather_k_cache,
    fused_indexer_q_rope_quant,
    fused_inv_rope_fp8_quant,
    fused_q_kv_rmsnorm,
)
from vllm.v1.attention.ops.rocm_aiter_mla_sparse import rocm_inv_rope_einsum

if TYPE_CHECKING:
    from vllm.v1.attention.backends.mla.sparse_swa import (
        DeepseekSparseSWAMetadata,
    )

from vllm.config import (
    CacheConfig,
    VllmConfig,
    get_current_vllm_config,
)
from vllm.distributed import get_tensor_model_parallel_world_size
from vllm.forward_context import ForwardContext, get_forward_context
from vllm.logger import init_logger
from vllm.model_executor.custom_op import PluggableLayer
from vllm.model_executor.layers.attention_layer_base import AttentionLayerBase
from vllm.model_executor.layers.deepseek_compressor import DeepseekCompressor
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.quantization.input_quant_fp8 import (
    QuantFP8,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
)
from vllm.platforms import current_platform
from vllm.utils.multi_stream_utils import (
    execute_in_parallel,
    maybe_execute_in_parallel,
)
# ... omitted for brevity ...
# reservation in attention_impl's dummy-run branch).
PREFILL_CHUNK_SIZE = 4
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`, `PREFILL_CHUNK_SIZE`. The body uses conditional branches to cover different runtime cases.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`, `PREFILL_CHUNK_SIZE`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 89-105 — class `DeepseekV4MLAModules`
```python
@dataclass
class DeepseekV4MLAModules:
    """Modules used in DeepseekV4 MLA."""

    vllm_config: VllmConfig
    fused_wqa_wkv: torch.nn.Module
    q_norm: torch.nn.Module
    wq_b: torch.nn.Module
    kv_norm: torch.nn.Module
    wo_a: torch.nn.Module
    wo_b: torch.nn.Module
    attn_sink: torch.nn.Module
    rotary_emb: torch.nn.Module
    indexer: torch.nn.Module | None
    indexer_rotary_emb: torch.nn.Module
    topk_indices_buffer: torch.Tensor | None
    aux_stream_list: list[torch.cuda.Stream] | None = None
```
**EN:** This class defines `DeepseekV4MLAModules`. Modules used in DeepseekV4 MLA. It writes or updates `vllm_config`, `fused_wqa_wkv`, `q_norm`, `wq_b`, `kv_norm`, `wo_a`.
**CN:** 该类定义了 `DeepseekV4MLAModules`。 该类的文档字符串说明了它在当前模块中的职责。 它会写入或更新 `vllm_config`, `fused_wqa_wkv`, `q_norm`, `wq_b`, `kv_norm`, `wo_a`。

### Lines 109-554 — class `DeepseekV4MultiHeadLatentAttentionWrapper`
```python
@PluggableLayer.register("deepseek_v4_multi_head_latent_attention")
class DeepseekV4MultiHeadLatentAttentionWrapper(PluggableLayer):
    """Pluggable MLA layer which allows OOT backends to add
    custom implementations of the outer MLA layer (including rope & o_proj).
    Note that currently oot platforms can still use CustomOp.register_oot to
    replace MLA layer entirely, although we use PluggableLayer to register
    this layer now.

    This class takes positions and hidden_states as input.
    The input tensors can either contain prefill tokens or decode tokens.
    The class does the following:

    1. MLA Preprocess.
    2. Perform multi-head attention to prefill tokens and
# ... omitted for brevity ...
            swa_metadata.block_size,
        )
```
**EN:** This class defines `DeepseekV4MultiHeadLatentAttentionWrapper`. It inherits from `PluggableLayer`. Pluggable MLA layer which allows OOT backends to add custom implementations of the outer MLA layer (including rope & o_proj). Important methods include `__init__`, `forward`, `attn_gemm_parallel_execute`, `attention_impl`. Key calls include `PluggableLayer.register`, `super.__init__`, `get_tensor_model_parallel_world_size`, `QuantFP8`, `current_platform.get_device_capability`, `RMSNorm`. It writes or updates `hidden_size`, `n_local_heads`, `head_dim`, `scale`, `q_lora_rank`, `kv_lora_rank`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `DeepseekV4MultiHeadLatentAttentionWrapper`。 它继承自 `PluggableLayer`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `forward`, `attn_gemm_parallel_execute`, `attention_impl`。 关键调用包括 `PluggableLayer.register`, `super.__init__`, `get_tensor_model_parallel_world_size`, `QuantFP8`, `current_platform.get_device_capability`, `RMSNorm`。 它会写入或更新 `hidden_size`, `n_local_heads`, `head_dim`, `scale`, `q_lora_rank`, `kv_lora_rank`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 129-282 — method `DeepseekV4MultiHeadLatentAttentionWrapper.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        head_dim: int,
        scale: float,
        qk_nope_head_dim: int,
        qk_rope_head_dim: int,
        v_head_dim: int,
        q_lora_rank: int | None,
        kv_lora_rank: int,
        o_lora_rank: int | None,
        mla_modules: DeepseekV4MLAModules,
        window_size: int,
        compress_ratio: int | None,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        self.n_local_heads = num_heads
        self.head_dim = head_dim
        self.scale = scale

        # FlashMLA sparse kernel only supports 64 or 128 heads; pad up to the
        # next supported size. Must match DeepseekV4MLAAttention.padded_heads.
        if num_heads <= 64:
            self.padded_heads = 64
        elif num_heads <= 128:
            self.padded_heads = 128
        else:
            raise ValueError(
                f"DeepseekV4 attention does not support {num_heads} heads "
                "(must be <= 128)."
            )

        self.q_lora_rank = q_lora_rank
        self.kv_lora_rank = kv_lora_rank
        self.window_size = window_size
        self.compress_ratio = compress_ratio if compress_ratio is not None else 1
        self.prefix = prefix

        # Extract config from vllm_config
# ... omitted for brevity ...
                k_cache_prefix=self.mla_attn.prefix,
            )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `hidden_size`, `num_heads`, `head_dim`, `scale`, `qk_nope_head_dim`, `qk_rope_head_dim`. Key calls include `super.__init__`, `get_tensor_model_parallel_world_size`, `QuantFP8`, `current_platform.get_device_capability`, `RMSNorm`, `DeepseekV4SWACache`. It writes or updates `hidden_size`, `n_local_heads`, `head_dim`, `scale`, `q_lora_rank`, `kv_lora_rank`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `hidden_size`, `num_heads`, `head_dim`, `scale`, `qk_nope_head_dim`, `qk_rope_head_dim`。 关键调用包括 `super.__init__`, `get_tensor_model_parallel_world_size`, `QuantFP8`, `current_platform.get_device_capability`, `RMSNorm`, `DeepseekV4SWACache`。 它会写入或更新 `hidden_size`, `n_local_heads`, `head_dim`, `scale`, `q_lora_rank`, `kv_lora_rank`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 284-351 — method `DeepseekV4MultiHeadLatentAttentionWrapper.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        llama_4_scaling: torch.Tensor | None = None,
    ) -> torch.Tensor:
        # Pre-allocate attention output with FlashMLA-padded head count.
        # The op writes into `o_padded`; we slice to n_local_heads after.
        num_tokens = hidden_states.shape[0]
        o_padded = torch.empty(
            (num_tokens, self.padded_heads, self.head_dim),
            dtype=hidden_states.dtype,
            device=hidden_states.device,
        )

        # Attention (inside custom op for torch.compile boundary)
        torch.ops.vllm.deepseek_v4_attention(
            hidden_states,
            positions,
            o_padded,
            self.layer_name,
        )
        o = o_padded[:, : self.n_local_heads, :]

        # Keep ROCm on the BF16 reference wo_a path util kernel ready.
        if current_platform.is_rocm():
            z = rocm_inv_rope_einsum(
                self.rotary_emb,
                o,
                positions,
                self.rope_head_dim,
                self.n_local_groups,
                self.o_lora_rank,
                self.wo_a,
            )
            return self.wo_b(z.flatten(1))

        # O projection: inverse RoPE + FP8 quant + einsum + wo_b
        o_fp8, o_scale = fused_inv_rope_fp8_quant(
            o,
            positions,
            self.rotary_emb.cos_sin_cache,
            n_groups=self.n_local_groups,
            heads_per_group=self.n_local_heads // self.n_local_groups,
# ... omitted for brevity ...

        return self.wo_b(z.flatten(1))
```
**EN:** This method defines `forward`. It executes the main forward/runtime path for this component. The main inputs are `positions`, `hidden_states`, `llama_4_scaling`. Key calls include `torch.empty`, `torch.ops.vllm.deepseek_v4_attention`, `current_platform.is_rocm`, `fused_inv_rope_fp8_quant`, `torch.ops.vllm.deepseek_v4_fp8_einsum`, `self.wo_b`. It writes or updates `num_tokens`, `o_padded`, `o`, `o_fp8`, `o_scale`, `wo_a_fp8`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `positions`, `hidden_states`, `llama_4_scaling`。 关键调用包括 `torch.empty`, `torch.ops.vllm.deepseek_v4_attention`, `current_platform.is_rocm`, `fused_inv_rope_fp8_quant`, `torch.ops.vllm.deepseek_v4_fp8_einsum`, `self.wo_b`。 它会写入或更新 `num_tokens`, `o_padded`, `o`, `o_fp8`, `o_scale`, `wo_a_fp8`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 353-411 — method `DeepseekV4MultiHeadLatentAttentionWrapper.attn_gemm_parallel_execute`
```python
    def attn_gemm_parallel_execute(self, hidden_states) -> tuple[Any, ...]:
        aux_streams = self.aux_stream_list
        if aux_streams is not None:
            assert len(aux_streams) >= 3
            aux_streams = aux_streams[:3]

        # fused_wqa_wkv (heaviest) on default; the three lighter input GEMMs
        # on aux streams 0..2 when their owning module exists. ln_events[0]
        # is the fan-out start event; ln_events[1..3] are per-aux done events.
        # On ROCm, aux_streams is None and execute_in_parallel runs serially.
        aux_fns: list[Callable[[], Any] | None] = [None, None, None]

        if self.compressor is not None:
            # Local ref so the closure keeps a non-None type for mypy.
            compressor = self.compressor

            def compressor_kv_score() -> torch.Tensor:
                return torch.mm(
                    hidden_states,
                    compressor.fused_wkv_wgate.weight.T,
                    out_dtype=torch.float32,
                )

            aux_fns[0] = compressor_kv_score

        if self.indexer is not None:
            indexer = self.indexer

            def indexer_weights_proj() -> torch.Tensor:
                # ReplicatedLinear returns (output, bias); bias is None.
                weights, _ = indexer.weights_proj(hidden_states)
                return weights

            def indexer_compressor_kv_score() -> torch.Tensor:
                return torch.mm(
                    hidden_states,
                    indexer.compressor.fused_wkv_wgate.weight.T,
                    out_dtype=torch.float32,
                )

            aux_fns[1] = indexer_weights_proj
            aux_fns[2] = indexer_compressor_kv_score

        def fused_wqa_wkv() -> torch.Tensor:
# ... omitted for brevity ...

        return qr_kv, kv_score, indexer_kv_score, indexer_weights
```
**EN:** This method defines `attn_gemm_parallel_execute`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `hidden_states`. Key calls include `execute_in_parallel`, `self.fused_wqa_wkv`, `len`, `torch.mm`, `indexer.weights_proj`. It writes or updates `aux_streams`, `aux_fns`, `qr_kv`, `kv_score`, `indexer_weights`, `indexer_kv_score`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `attn_gemm_parallel_execute`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `hidden_states`。 关键调用包括 `execute_in_parallel`, `self.fused_wqa_wkv`, `len`, `torch.mm`, `indexer.weights_proj`。 它会写入或更新 `aux_streams`, `aux_fns`, `qr_kv`, `kv_score`, `indexer_weights`, `indexer_kv_score`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 413-518 — method `DeepseekV4MultiHeadLatentAttentionWrapper.attention_impl`
```python
    def attention_impl(
        self,
        hidden_states: torch.Tensor,
        positions: torch.Tensor,
        out: torch.Tensor,  # [num_tokens, padded_heads, head_dim], written in place
    ) -> None:
        forward_context = get_forward_context()
        attn_metadata = forward_context.attn_metadata

        qr_kv, kv_score, indexer_kv_score, indexer_weights = (
            self.attn_gemm_parallel_execute(hidden_states)
        )

        qr, kv = qr_kv.split([self.q_lora_rank, self.head_dim], dim=-1)
        qr, kv = fused_q_kv_rmsnorm(
            qr,
            kv,
            self.q_norm.weight.data,
            self.kv_norm.weight.data,
            self.eps,
        )

        # wq_b + kv_insert (+ MLA compressor when an indexer is present) ride
        # on the default stream so q stays on its consumer stream (mla_attn
        # downstream reads q on default). Indexer/compressor go on aux for
        # overlap with default's GEMM + cache write.
        if self.indexer is not None:
            aux_stream = (
                self.aux_stream_list[0] if self.aux_stream_list is not None else None
            )
            indexer = self.indexer
            # Local ref so the closure keeps a non-None type for mypy.
            assert self.compressor is not None
            compressor = self.compressor

            def wq_b_kv_insert_and_compress() -> torch.Tensor:
                q = self.wq_b(qr).view(-1, self.n_local_heads, self.head_dim)
                self._fused_qnorm_rope_kv_insert(q, kv, positions, attn_metadata)
                compressor(kv_score, positions, self.rotary_emb)
                return q

            q, _ = maybe_execute_in_parallel(
                wq_b_kv_insert_and_compress,
                lambda: indexer(
# ... omitted for brevity ...
        # ([num_tokens, padded_heads, head_dim]).
        self.mla_attn(q, kv, positions, output=out)
```
**EN:** This method defines `attention_impl`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `hidden_states`, `positions`, `out`. Key calls include `get_forward_context`, `self.attn_gemm_parallel_execute`, `qr_kv.split`, `fused_q_kv_rmsnorm`, `self.mla_attn`, `maybe_execute_in_parallel`. It writes or updates `forward_context`, `attn_metadata`, `qr_kv`, `kv_score`, `indexer_kv_score`, `indexer_weights`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `attention_impl`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `hidden_states`, `positions`, `out`。 关键调用包括 `get_forward_context`, `self.attn_gemm_parallel_execute`, `qr_kv.split`, `fused_q_kv_rmsnorm`, `self.mla_attn`, `maybe_execute_in_parallel`。 它会写入或更新 `forward_context`, `attn_metadata`, `qr_kv`, `kv_score`, `indexer_kv_score`, `indexer_weights`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 557-566 — function `deepseek_v4_attention`
```python
@eager_break_during_capture
def deepseek_v4_attention(
    hidden_states: torch.Tensor,
    positions: torch.Tensor,
    out: torch.Tensor,
    layer_name: str,
) -> None:
    forward_context: ForwardContext = get_forward_context()
    self = forward_context.no_compile_layers[layer_name]
    self.attention_impl(hidden_states, positions, out)
```
**EN:** This function defines `deepseek_v4_attention`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `hidden_states`, `positions`, `out`, `layer_name`. Key calls include `get_forward_context`, `self.attention_impl`. It writes or updates `forward_context`, `self`.
**CN:** 该函数定义 `deepseek_v4_attention`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `hidden_states`, `positions`, `out`, `layer_name`。 关键调用包括 `get_forward_context`, `self.attention_impl`。 它会写入或更新 `forward_context`, `self`。

### Lines 569-575 — function `deepseek_v4_attention_fake`
```python
def deepseek_v4_attention_fake(
    hidden_states: torch.Tensor,
    positions: torch.Tensor,
    out: torch.Tensor,
    layer_name: str,
) -> None:
    return None
```
**EN:** This function defines `deepseek_v4_attention_fake`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `hidden_states`, `positions`, `out`, `layer_name`.
**CN:** 该函数定义 `deepseek_v4_attention_fake`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `hidden_states`, `positions`, `out`, `layer_name`。

### Lines 586-595 — function `deepseek_v4_fp8_einsum`
```python
def deepseek_v4_fp8_einsum(
    a: torch.Tensor,
    a_scale: torch.Tensor,
    b: torch.Tensor,
    b_scale: torch.Tensor,
    out: torch.Tensor,
    equation: str,
    recipe: list[int],
) -> None:
    fp8_einsum(equation, (a, a_scale), (b, b_scale), out, recipe=tuple(recipe))
```
**EN:** This function defines `deepseek_v4_fp8_einsum`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `a`, `a_scale`, `b`, `b_scale`, `out`, `equation`. Key calls include `fp8_einsum`, `tuple`.
**CN:** 该函数定义 `deepseek_v4_fp8_einsum`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `a`, `a_scale`, `b`, `b_scale`, `out`, `equation`。 关键调用包括 `fp8_einsum`, `tuple`。

### Lines 598-607 — function `deepseek_v4_fp8_einsum_fake`
```python
def deepseek_v4_fp8_einsum_fake(
    a: torch.Tensor,
    a_scale: torch.Tensor,
    b: torch.Tensor,
    b_scale: torch.Tensor,
    out: torch.Tensor,
    equation: str,
    recipe: list[int],
) -> None:
    return None
```
**EN:** This function defines `deepseek_v4_fp8_einsum_fake`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `a`, `a_scale`, `b`, `b_scale`, `out`, `equation`.
**CN:** 该函数定义 `deepseek_v4_fp8_einsum_fake`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `a`, `a_scale`, `b`, `b_scale`, `out`, `equation`。

### Lines 618-1025 — class `DeepseekV4MLAAttention`
```python
class DeepseekV4MLAAttention(nn.Module, AttentionLayerBase):
    # FlashMLA FP8 sparse only supports 64 or 128 heads
    SUPPORTED_HEAD_COUNTS = (64, 128)

    def __init__(
        self,
        num_heads: int,
        head_dim: int,
        scale: float,
        qk_nope_head_dim: int,
        qk_rope_head_dim: int,
        q_lora_rank: int | None,
        kv_lora_rank: int,
        compress_ratio: int,
# ... omitted for brevity ...
                out=output[query_start:query_end],
            )
```
**EN:** This class defines `DeepseekV4MLAAttention`. It inherits from `nn.Module`, `AttentionLayerBase`. It handles attention-specific state, metadata, or tensor transforms. Important methods include `__init__`, `forward`, `get_attn_backend`, `get_kv_cache_spec`. Key calls include `super.__init__`, `get_current_vllm_config`, `kv_cache_dtype.startswith`, `issubclass`, `torch.tensor`, `current_platform.is_rocm`. It writes or updates `SUPPORTED_HEAD_COUNTS`, `num_heads`, `num_kv_heads`, `head_dim`, `scale`, `window_size`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该类定义了 `DeepseekV4MLAAttention`。 它继承自 `nn.Module`, `AttentionLayerBase`。 它处理注意力相关的状态、元数据或张量变换。 重要方法包括 `__init__`, `forward`, `get_attn_backend`, `get_kv_cache_spec`。 关键调用包括 `super.__init__`, `get_current_vllm_config`, `kv_cache_dtype.startswith`, `issubclass`, `torch.tensor`, `current_platform.is_rocm`。 它会写入或更新 `SUPPORTED_HEAD_COUNTS`, `num_heads`, `num_kv_heads`, `head_dim`, `scale`, `window_size`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 622-723 — method `DeepseekV4MLAAttention.__init__`
```python
    def __init__(
        self,
        num_heads: int,
        head_dim: int,
        scale: float,
        qk_nope_head_dim: int,
        qk_rope_head_dim: int,
        q_lora_rank: int | None,
        kv_lora_rank: int,
        compress_ratio: int,
        window_size: int,
        head_bytes: int,
        swa_cache_layer: DeepseekV4SWACache,
        attn_sink: torch.Tensor,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
        # Sparse MLA Args
        indexer: object | None = None,
        topk_indices_buffer: torch.Tensor | None = None,
        aux_stream: torch.cuda.Stream | None = None,
        **extra_impl_args,
    ) -> None:
        super().__init__()
        self.num_heads = num_heads
        self.num_kv_heads = 1
        self.head_dim = head_dim
        self.scale = scale
        self.window_size = window_size
        self.head_bytes = head_bytes
        self.compress_ratio = compress_ratio
        self.q_lora_rank = q_lora_rank
        self.kv_lora_rank = kv_lora_rank
        self.nope_head_dim = qk_nope_head_dim
        self.rope_head_dim = qk_rope_head_dim
        self.indexer = indexer
        self.topk_indices_buffer = topk_indices_buffer

        self.prefix = prefix  # Alias for compatibility with compressor

        self.aux_stream = aux_stream
        self.ln_events = [torch.cuda.Event(), torch.cuda.Event()]

        # Determine padded head count for FlashMLA
# ... omitted for brevity ...

        self.kv_cache = torch.tensor([])
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `num_heads`, `head_dim`, `scale`, `qk_nope_head_dim`, `qk_rope_head_dim`, `q_lora_rank`. Key calls include `super.__init__`, `get_current_vllm_config`, `kv_cache_dtype.startswith`, `issubclass`, `torch.tensor`, `torch.cuda.Event`. It writes or updates `num_heads`, `num_kv_heads`, `head_dim`, `scale`, `window_size`, `head_bytes`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `num_heads`, `head_dim`, `scale`, `qk_nope_head_dim`, `qk_rope_head_dim`, `q_lora_rank`。 关键调用包括 `super.__init__`, `get_current_vllm_config`, `kv_cache_dtype.startswith`, `issubclass`, `torch.tensor`, `torch.cuda.Event`。 它会写入或更新 `num_heads`, `num_kv_heads`, `head_dim`, `scale`, `window_size`, `head_bytes`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 750-814 — method `DeepseekV4MLAAttention.forward`
```python
    def forward(
        self,
        q: torch.Tensor,
        kv: torch.Tensor,
        positions: torch.Tensor,
        output: torch.Tensor,
    ) -> None:
        assert output.shape == q.shape, (
            f"output buffer shape {output.shape} must match q shape {q.shape}"
        )
        assert output.dtype == q.dtype, (
            f"output buffer dtype {output.dtype} must match q dtype {q.dtype}"
        )

        if current_platform.is_rocm():
            from vllm.v1.attention.backends.mla.rocm_aiter_mla_sparse_dsv4 import (
                DeepseekV4ROCMAiterMLASparseImpl,
            )

            DeepseekV4ROCMAiterMLASparseImpl.forward(self, q, kv, positions, output)
            return

        # Get SWA and indexer metadata from forward context
        forward_context = get_forward_context()
        attn_metadata = forward_context.attn_metadata
        assert isinstance(attn_metadata, dict)
        flashmla_metadata = cast(
            FlashMLASparseMetadata | None, attn_metadata.get(self.prefix)
        )
        swa_metadata = cast(
            "DeepseekSparseSWAMetadata | None",
            attn_metadata.get(self.swa_cache_layer.prefix),
        )
        assert swa_metadata is not None

        swa_only = self.compress_ratio <= 1
        # SWA-only layers (compress_ratio <= 1) don't have their own KV cache
        # allocation, so self.kv_cache may be empty after profiling cleanup.
        self_kv_cache = self.kv_cache if not swa_only else None
        swa_kv_cache = self.swa_cache_layer.kv_cache

        # Split prefill and decode
        num_decodes = swa_metadata.num_decodes
        num_prefills = swa_metadata.num_prefills
# ... omitted for brevity ...
                output=output[:num_decode_tokens],
            )
```
**EN:** This method defines `forward`. It executes the main forward/runtime path for this component. The main inputs are `q`, `kv`, `positions`, `output`. Key calls include `current_platform.is_rocm`, `get_forward_context`, `isinstance`, `cast`, `DeepseekV4ROCMAiterMLASparseImpl.forward`, `attn_metadata.get`. It writes or updates `forward_context`, `attn_metadata`, `flashmla_metadata`, `swa_metadata`, `swa_only`, `self_kv_cache`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `q`, `kv`, `positions`, `output`。 关键调用包括 `current_platform.is_rocm`, `get_forward_context`, `isinstance`, `cast`, `DeepseekV4ROCMAiterMLASparseImpl.forward`, `attn_metadata.get`。 它会写入或更新 `forward_context`, `attn_metadata`, `flashmla_metadata`, `swa_metadata`, `swa_only`, `self_kv_cache`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Deepseek-specific compression or attention logic / [CN] DeepSeek 专用压缩或注意力逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `DeepseekV4MLAModules`, `DeepseekV4MultiHeadLatentAttentionWrapper`, `deepseek_v4_attention`, `deepseek_v4_attention_fake` / [CN] 核心符号：`DeepseekV4MLAModules`, `DeepseekV4MultiHeadLatentAttentionWrapper`, `deepseek_v4_attention`, `deepseek_v4_attention_fake`

## Dependencies / 依赖关系
- **External**: `collections.abc`, `dataclasses`, `typing`, `torch`, `torch.nn`, `torch.nn.functional`, `transformers` / **外部依赖**: `collections.abc`, `dataclasses`, `typing`, `torch`, `torch.nn`, `torch.nn.functional`, `transformers`
- **Internal**: `vllm.envs`, `vllm.compilation.breakable_cudagraph`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.sparse_attn_indexer`, `vllm.utils.deep_gemm`, `vllm.utils.torch_utils`, `vllm.v1.attention.ops.deepseek_v4_ops`, `vllm.v1.attention.ops.rocm_aiter_mla_sparse`, `vllm.v1.attention.backends.mla.sparse_swa`, `vllm.config` / **内部依赖**: `vllm.envs`, `vllm.compilation.breakable_cudagraph`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.sparse_attn_indexer`, `vllm.utils.deep_gemm`, `vllm.utils.torch_utils`, `vllm.v1.attention.ops.deepseek_v4_ops`, `vllm.v1.attention.ops.rocm_aiter_mla_sparse`, `vllm.v1.attention.backends.mla.sparse_swa`, `vllm.config`
- **Runtime traits**: platform-aware dispatch, custom C++/CUDA ops, distributed collectives / **运行时特征**: platform-aware dispatch, custom C++/CUDA ops, distributed collectives
