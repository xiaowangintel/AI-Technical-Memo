# rocm_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/rocm_attn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Attention layer with PagedAttention and Triton prefix prefill. / 该模块位于 `attention/backends` 子系统，主要围绕 `RocmAttentionMetadata`, `RocmAttentionMetadataBuilder`, `RocmAttentionBackend` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Attention layer with PagedAttention and Triton prefix prefill."""

from dataclasses import dataclass
from typing import ClassVar

import torch

from vllm._aiter_ops import rocm_aiter_ops
from vllm.config import VllmConfig
from vllm.config.cache import CacheDType
from vllm.logger import init_logger
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kFp8StaticTensorSym,
)
from vllm.platforms import current_platform
from vllm.utils.torch_utils import is_quantized_kv_cache
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionCGSupport,
    AttentionImpl,
    AttentionLayer,
    AttentionMetadataBuilder,
    AttentionType,
    CommonAttentionMetadata,
    MultipleOf,
)
from vllm.v1.attention.ops.chunked_prefill_paged_decode import (
    chunked_prefill_paged_decode,
    has_native_kv_cache_layout,
)
from vllm.v1.attention.ops.paged_attn import PagedAttention
from vllm.v1.attention.ops.triton_reshape_and_cache_flash import (
    triton_reshape_and_cache_flash,
)
from vllm.v1.kv_cache_interface import AttentionSpec

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `RocmAttentionMetadata` class / `RocmAttentionMetadata` 类
```python
@dataclass
class RocmAttentionMetadata:
    # NOTE(sang): Definition of context_len, query_len, and seq_len.
    # |---------- N-1 iteration --------|
    # |---------------- N iteration ---------------------|
    # |- tokenA -|......................|-- newTokens ---|
    # |---------- context_len ----------|
    # |-------------------- seq_len ---------------------|
    #                                   |-- query_len ---|

    num_actual_tokens: int  # Number of tokens excluding padding.
    max_query_len: int
    query_start_loc: torch.Tensor
    max_seq_len: int
    seq_lens: torch.Tensor
    block_table: torch.Tensor
    slot_mapping: torch.Tensor

    # For cascade attention.
    use_cascade: bool
    common_prefix_len: int
    cu_prefix_query_lens: torch.Tensor | None
    prefix_kv_lens: torch.Tensor | None
    suffix_kv_lens: torch.Tensor | None

    # Optional aot scheduling
    scheduler_metadata: torch.Tensor | None = None
    prefix_scheduler_metadata: torch.Tensor | None = None

    # DFlash drafting sets this to False via CommonAttentionMetadata.
    causal: bool = True
```
**EN:** Uses `@dataclass` to package related state for `RocmAttentionMetadata`. Typical fields include `num_actual_tokens`, `max_query_len`, `query_start_loc`, `max_seq_len`, `seq_lens`, `block_table`.
**CN:** `RocmAttentionMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_actual_tokens`, `max_query_len`, `query_start_loc`, `max_seq_len`, `seq_lens`, `block_table`。

### `RocmAttentionMetadataBuilder` class / `RocmAttentionMetadataBuilder` 类
```python
class RocmAttentionMetadataBuilder(AttentionMetadataBuilder[RocmAttentionMetadata]):
    _cudagraph_support: ClassVar[AttentionCGSupport] = AttentionCGSupport.ALWAYS
```
**EN:** Introduces the `RocmAttentionMetadataBuilder` class on top of `AttentionMetadataBuilder[RocmAttentionMetadata]`. Core methods include `__init__`, `build_for_cudagraph_capture`, `build`.
**CN:** 这里定义 `RocmAttentionMetadataBuilder` 类，其基类包括 `AttentionMetadataBuilder[RocmAttentionMetadata]`。核心方法包括 `__init__`, `build_for_cudagraph_capture`, `build`。

### `RocmAttentionMetadataBuilder.__init__` method / `RocmAttentionMetadataBuilder.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_spec: AttentionSpec,
        layer_names: list[str],
        vllm_config: VllmConfig,
        device: torch.device,
    ):
        super().__init__(kv_cache_spec, layer_names, vllm_config, device)

        self.block_size = kv_cache_spec.block_size

        model_config = vllm_config.model_config
        self.num_heads_q = model_config.get_num_attention_heads(
            vllm_config.parallel_config
        )
        self.num_heads_kv = model_config.get_num_kv_heads(vllm_config.parallel_config)
        self.headdim = model_config.get_head_size()
```
**EN:** This method initializes the object state within `RocmAttentionMetadataBuilder`. Key calls include `__init__`, `get_num_attention_heads`, `get_num_kv_heads`, `get_head_size`, `super`. It touches state such as `block_size`, `num_heads_q`, `num_heads_kv`, `headdim`.
**CN:** 该方法会初始化对象状态，其作用域位于`RocmAttentionMetadataBuilder`。 关键调用包括 `__init__`, `get_num_attention_heads`, `get_num_kv_heads`, `get_head_size`, `super`。 它会读写 `block_size`, `num_heads_q`, `num_heads_kv`, `headdim` 等状态。

### `RocmAttentionMetadataBuilder.build_for_cudagraph_capture` method / `RocmAttentionMetadataBuilder.build_for_cudagraph_capture` 方法
```python
    def build_for_cudagraph_capture(
        self, common_attn_metadata: CommonAttentionMetadata
    ) -> RocmAttentionMetadata:
        attn_metadata = self.build(0, common_attn_metadata)
        # When doing full graph capture, setting seq_lens to
        # max_model_len will cause graph capture to be extremely
        # slow, so here we set it to 1.
        attn_metadata.seq_lens.fill_(1)

        # Here we set the query start locs to 0. This is to
        # cover up an invalid memory access in the prefix_prefil kernel
        # that we run into during graph capture (#25985)
        common_attn_metadata.query_start_loc.zero_()
        common_attn_metadata.query_start_loc_cpu.zero_()

        return attn_metadata
```
**EN:** This method builds derived structures within `RocmAttentionMetadataBuilder`. Key calls include `build`, `fill_`, `zero_`.
**CN:** 该方法会构建派生结构，其作用域位于`RocmAttentionMetadataBuilder`。 关键调用包括 `build`, `fill_`, `zero_`。

### `RocmAttentionMetadataBuilder.build` method / `RocmAttentionMetadataBuilder.build` 方法
```python
    def build(
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        fast_build: bool = False,
    ) -> RocmAttentionMetadata:
        num_actual_tokens = common_attn_metadata.num_actual_tokens
        max_query_len = common_attn_metadata.max_query_len

        max_seq_len = common_attn_metadata.max_seq_len
        query_start_loc = common_attn_metadata.query_start_loc
        seq_lens = common_attn_metadata.seq_lens
        block_table_tensor = common_attn_metadata.block_table_tensor
        slot_mapping = common_attn_metadata.slot_mapping

        use_cascade = common_prefix_len > 0

        if use_cascade:
            cu_prefix_query_lens = torch.tensor(
                [0, num_actual_tokens], dtype=torch.int32, device=self.device
            )
            prefix_kv_lens = torch.tensor(
                [common_prefix_len], dtype=torch.int32, device=self.device
            )
            suffix_kv_lens = common_attn_metadata.seq_lens.cpu() - common_prefix_len
            suffix_kv_lens = suffix_kv_lens.to(self.device)
        else:
            cu_prefix_query_lens = None
            prefix_kv_lens = None
            suffix_kv_lens = None
            prefix_scheduler_metadata = None

        attn_metadata = RocmAttentionMetadata(
            num_actual_tokens=num_actual_tokens,
            max_query_len=max_query_len,
            query_start_loc=query_start_loc,
            max_seq_len=max_seq_len,
            seq_lens=seq_lens,
            block_table=block_table_tensor,
            slot_mapping=slot_mapping,
            use_cascade=use_cascade,
            common_prefix_len=common_prefix_len,
            cu_prefix_query_lens=cu_prefix_query_lens,
            prefix_kv_lens=prefix_kv_lens,
            suffix_kv_lens=suffix_kv_lens,
            prefix_scheduler_metadata=prefix_scheduler_metadata,
            causal=common_attn_metadata.causal,
        )
        return attn_metadata
```
**EN:** This method builds derived structures within `RocmAttentionMetadataBuilder`. Key calls include `RocmAttentionMetadata`, `tensor`, `to`, `cpu`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会构建派生结构，其作用域位于`RocmAttentionMetadataBuilder`。 关键调用包括 `RocmAttentionMetadata`, `tensor`, `to`, `cpu`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RocmAttentionBackend` class / `RocmAttentionBackend` 类
```python
class RocmAttentionBackend(AttentionBackend):
    supported_dtypes: ClassVar[list[torch.dtype]] = [
        torch.float16,
        torch.bfloat16,
        torch.float32,
    ]
    supported_kv_cache_dtypes: ClassVar[list[CacheDType]] = [
        "auto",
        "float16",
        "bfloat16",
        "fp8",
        "fp8_e4m3",
        "fp8_e5m2",
    ]
```
**EN:** Introduces the `RocmAttentionBackend` class on top of `AttentionBackend`. Core methods include `get_supported_kernel_block_sizes`, `get_supported_head_sizes`, `supports_mm_prefix`, `supports_sink`, `supports_non_causal`, `get_name`.
**CN:** 这里定义 `RocmAttentionBackend` 类，其基类包括 `AttentionBackend`。核心方法包括 `get_supported_kernel_block_sizes`, `get_supported_head_sizes`, `supports_mm_prefix`, `supports_sink`, `supports_non_causal`, `get_name`。

### `RocmAttentionBackend.get_supported_kernel_block_sizes` method / `RocmAttentionBackend.get_supported_kernel_block_sizes` 方法
```python
    @staticmethod
    def get_supported_kernel_block_sizes() -> list[int | MultipleOf]:
        # ROCM paged attention native C++ kernel only supports block sizes 16 and 32
        # due to shared memory (LDS) constraints on AMD GPUs.
        # See csrc/rocm/attention.cu CALL_CUSTOM_LAUNCHER_BLK macro.
        # However, vLLM allows support for any multiple of 16 via the Triton path.
        # As addressed in PR: https://github.com/vllm-project/vllm/pull/31380,
        # non-standard models (like qwen3-next with block_size 544, or qwen3_5
        # with 784 and 1056) are dynamically routed to our optimized Triton kernel
        # in `do_kv_cache_update`.
        return [MultipleOf(16)]
```
**EN:** This method returns or derives a value within `RocmAttentionBackend`. Key calls include `MultipleOf`.
**CN:** 该方法会返回或推导一个值，其作用域位于`RocmAttentionBackend`。 关键调用包括 `MultipleOf`。

### `RocmAttentionBackend.supports_attn_type` method / `RocmAttentionBackend.supports_attn_type` 方法
```python
    @classmethod
    def supports_attn_type(cls, attn_type: str) -> bool:
        """ENCODER_DECODER is not supported because
        chunked_prefill_paged_decode's prefill kernel (context_attention_fwd)
        assumes self-attention semantics: it treats passed K/V as new tokens
        to mix with cached K/V. For cross-attention layers the encoder K/V
        are already fully cached, so mixing them again produces incorrect
        results when max_query_len > 1 (e.g. beam search).
        """
        return attn_type in (
            AttentionType.DECODER,
            AttentionType.ENCODER,
            AttentionType.ENCODER_ONLY,
        )
```
**EN:** This method implements `supports_attn_type` within `RocmAttentionBackend`. The docstring frames it as: ENCODER_DECODER is not supported because chunked_prefill_paged_decode's prefill kernel (context_attention_fwd) assumes self-attention semantics: it treats passed K/V as new tokens to mix with cached K/V.
**CN:** 该方法会实现 `supports_attn_type`，其作用域位于`RocmAttentionBackend`。

### `RocmAttentionBackend.get_builder_cls` method / `RocmAttentionBackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["RocmAttentionMetadataBuilder"]:
        return RocmAttentionMetadataBuilder
```
**EN:** This method returns or derives a value within `RocmAttentionBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`RocmAttentionBackend`。

### `RocmAttentionImpl` class / `RocmAttentionImpl` 类
```python
class RocmAttentionImpl(AttentionImpl):
```
**EN:** Introduces the `RocmAttentionImpl` class on top of `AttentionImpl`. Core methods include `fused_output_quant_supported`, `__init__`, `_forward_encoder_attention`, `forward`, `do_kv_cache_update`, `fused_rope_kvcache_supported`.
**CN:** 这里定义 `RocmAttentionImpl` 类，其基类包括 `AttentionImpl`。核心方法包括 `fused_output_quant_supported`, `__init__`, `_forward_encoder_attention`, `forward`, `do_kv_cache_update`, `fused_rope_kvcache_supported`。

### `RocmAttentionImpl.forward` method / `RocmAttentionImpl.forward` 方法
```python
    def forward(
        self,
        layer: torch.nn.Module,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        kv_cache: torch.Tensor,
        attn_metadata: RocmAttentionMetadata,
        output: torch.Tensor,
        output_scale: torch.Tensor | None = None,
        output_block_scale: torch.Tensor | None = None,
    ) -> torch.Tensor:
        """Forward pass with FlashAttention.

        Args:
            query: shape = [num_tokens, num_heads, head_size]
            key: shape = [num_tokens, num_kv_heads, head_size]
            value: shape = [num_tokens, num_kv_heads, head_size]
            kv_cache: shape =
                [2, num_blocks, block_size, num_kv_heads, head_size]
            attn_metadata: Metadata for attention.
        Returns:
            shape = [num_tokens, num_heads * head_size]
        """
        if output_block_scale is not None:
            raise NotImplementedError(
                "fused block_scale output quantization is not yet supported"
                " for RocmAttentionImpl"
            )

        if attn_metadata is None:
            # Profiling run.
            return output.fill_(0)

        assert attn_metadata.use_cascade is False

        # IMPORTANT!
        # NOTE(woosuk): With piece-wise CUDA graphs, this method is executed in
        # eager-mode PyTorch. Thus, we need to be careful about any CPU overhead
        # in this method. For example, `view` and `slice` (or `[:n]`) operations
        # are surprisingly slow even in the case they do not invoke any GPU ops.
        # Minimize the PyTorch ops in this method as much as possible.
        # Whenever making a change in this method, please benchmark the
        # performance to make sure it does not introduce any overhead.

        num_actual_tokens = attn_metadata.num_actual_tokens

        if self.attn_type in (AttentionType.ENCODER_ONLY, AttentionType.ENCODER):
            return self._forward_encoder_attention(
                query[:num_actual_tokens],
    # ... omitted for brevity ...
        # Compute attention and update output up to `num_actual_tokens`.
        chunked_prefill_paged_decode(
            query=query[:num_actual_tokens],
            key=key[:num_actual_tokens] if key is not None else None,
            value=value[:num_actual_tokens] if value is not None else None,
            output=output[:num_actual_tokens],
            kv_cache_dtype=self.kv_cache_dtype,
            key_cache=key_cache,
            value_cache=value_cache,
            block_table=block_table,
            query_start_loc=cu_seqlens_q,
            seq_lens=seqused_k,
            max_seq_len=max_seqlen_k,
            max_query_len=max_seqlen_q,
            k_scale=layer._k_scale,
            v_scale=layer._v_scale,
            alibi_slopes=self.alibi_slopes,
            sliding_window=self.sliding_window[0],
            sm_scale=self.scale,
            output_scale=output_scale,
            sinks=self.sinks,
            causal=attn_metadata.causal,
        )

        return output
```
**EN:** This method drives the forward-pass computation within `RocmAttentionImpl`. The docstring frames it as: Forward pass with FlashAttention. Key calls include `split_kv_cache`, `is_quantized_kv_cache`, `chunked_prefill_paged_decode`, `NotImplementedError`, `fill_`, `_forward_encoder_attention`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会驱动前向计算流程，其作用域位于`RocmAttentionImpl`。 关键调用包括 `split_kv_cache`, `is_quantized_kv_cache`, `chunked_prefill_paged_decode`, `NotImplementedError`, `fill_`, `_forward_encoder_attention`。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `RocmAttentionImpl.do_kv_cache_update` method / `RocmAttentionImpl.do_kv_cache_update` 方法
```python
    def do_kv_cache_update(
        self,
        layer: AttentionLayer,
        key: torch.Tensor,
        value: torch.Tensor,
        kv_cache: torch.Tensor,
        slot_mapping: torch.Tensor,
    ):
        if self.attn_type in (AttentionType.ENCODER_ONLY, AttentionType.ENCODER):
            return
        key_cache, value_cache = PagedAttention.split_kv_cache(
            kv_cache, self.num_kv_heads, self.head_size
        )

        # Reshape the input keys and values and store them in the cache.
        # Get the actual block_size from value_cache
        # value_cache shape: [num_blocks, num_heads, head_size, block_size]
        block_size = value_cache.shape[3]
        has_native_layout = has_native_kv_cache_layout(key_cache, value_cache)

        if block_size in (16, 32) and has_native_layout:
            # Normal 16, 32 with contiguous blocks: use vLLM native HIP C++ logic.
            PagedAttention.write_to_paged_cache(
                key,
                value,
                key_cache,
                value_cache,
                slot_mapping,
                self.kv_cache_dtype,
                layer._k_scale,
                layer._v_scale,
            )
        else:
            # Non-standard blocks and hybrid attention/Mamba layouts need the
            # stride-aware Triton writer. The native reshape_and_cache kernel
            # assumes contiguous block storage and writes to the wrong hybrid
            # cache blocks.
            triton_reshape_and_cache_flash(
                key,
                value,
                key_cache,
                value_cache,
                slot_mapping,
                self.kv_cache_dtype,
                layer._k_scale,
                layer._v_scale,
            )
```
**EN:** This method implements `do_kv_cache_update` within `RocmAttentionImpl`. Key calls include `split_kv_cache`, `has_native_kv_cache_layout`, `write_to_paged_cache`, `triton_reshape_and_cache_flash`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `do_kv_cache_update`，其作用域位于`RocmAttentionImpl`。 关键调用包括 `split_kv_cache`, `has_native_kv_cache_layout`, `write_to_paged_cache`, `triton_reshape_and_cache_flash`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RocmAttentionImpl.do_rope_and_kv_cache_update` method / `RocmAttentionImpl.do_rope_and_kv_cache_update` 方法
```python
    def do_rope_and_kv_cache_update(
        self,
        layer: AttentionLayer,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        positions: torch.Tensor,
        cos_sin_cache: torch.Tensor,
        is_neox: bool,
        kv_cache: torch.Tensor,
        layer_slot_mapping: torch.Tensor,
    ):
        if self.attn_type in (AttentionType.ENCODER_ONLY, AttentionType.ENCODER):
            return
        key_cache, value_cache = PagedAttention.split_kv_cache(
            kv_cache,
            layer.num_kv_heads,  # type: ignore[attr-defined]
            layer.head_size,  # type: ignore[attr-defined]
        )
        flash_layout = False

        is_fp8_kv_cache = is_quantized_kv_cache(self.kv_cache_dtype)
        if is_fp8_kv_cache:
            key_cache = key_cache.view(self.fp8_dtype)
            value_cache = value_cache.view(self.fp8_dtype)

        rocm_aiter_ops.triton_rope_and_cache(
            query,
            key,
            value,
            positions,
            cos_sin_cache,
            is_neox,
            key_cache,
            value_cache,
            layer_slot_mapping,
            layer._k_scale,
            layer._v_scale,
            flash_layout,
            is_fp8_kv_cache,
        )
```
**EN:** This method implements `do_rope_and_kv_cache_update` within `RocmAttentionImpl`. Key calls include `split_kv_cache`, `is_quantized_kv_cache`, `triton_rope_and_cache`, `view`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `do_rope_and_kv_cache_update`，其作用域位于`RocmAttentionImpl`。 关键调用包括 `split_kv_cache`, `is_quantized_kv_cache`, `triton_rope_and_cache`, `view`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `RocmAttentionMetadata`: central class or interface in this module. / `RocmAttentionMetadata`：本模块中的核心类或接口。
- `RocmAttentionMetadataBuilder`: central class or interface in this module. / `RocmAttentionMetadataBuilder`：本模块中的核心类或接口。
- `RocmAttentionBackend`: central class or interface in this module. / `RocmAttentionBackend`：本模块中的核心类或接口。
- `RocmAttentionImpl`: central class or interface in this module. / `RocmAttentionImpl`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm._aiter_ops`, `vllm.config`, `vllm.config.cache`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.ops.chunked_prefill_paged_decode`, `vllm.v1.attention.ops.paged_attn`, `vllm.v1.attention.ops.triton_reshape_and_cache_flash`, `vllm.v1.kv_cache_interface`
