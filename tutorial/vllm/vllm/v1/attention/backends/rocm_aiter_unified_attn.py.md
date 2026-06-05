# rocm_aiter_unified_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/rocm_aiter_unified_attn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Attention layer with PagedAttention and Triton prefix prefill. / 该模块位于 `attention/backends` 子系统，主要围绕 `RocmAiterUnifiedAttentionBackend`, `RocmAiterUnifiedAttentionImpl` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Attention layer with PagedAttention and Triton prefix prefill."""

import torch

from vllm import _custom_ops as ops
from vllm._aiter_ops import rocm_aiter_ops
from vllm.logger import init_logger
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kFp8StaticTensorSym,
)
from vllm.utils.torch_utils import is_quantized_kv_cache
from vllm.v1.attention.backend import AttentionLayer, AttentionType, MultipleOf
from vllm.v1.attention.backends.rocm_attn import (
    RocmAttentionBackend,
    RocmAttentionImpl,
    RocmAttentionMetadata,
    RocmAttentionMetadataBuilder,
)

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `RocmAiterUnifiedAttentionBackend` class / `RocmAiterUnifiedAttentionBackend` 类
```python
class RocmAiterUnifiedAttentionBackend(RocmAttentionBackend):
```
**EN:** Introduces the `RocmAiterUnifiedAttentionBackend` class on top of `RocmAttentionBackend`. Core methods include `get_supported_kernel_block_sizes`, `get_preferred_block_size`, `supports_block_size`, `supports_head_size`, `supports_mm_prefix`, `supports_sink`.
**CN:** 这里定义 `RocmAiterUnifiedAttentionBackend` 类，其基类包括 `RocmAttentionBackend`。核心方法包括 `get_supported_kernel_block_sizes`, `get_preferred_block_size`, `supports_block_size`, `supports_head_size`, `supports_mm_prefix`, `supports_sink`。

### `RocmAiterUnifiedAttentionBackend.get_supported_kernel_block_sizes` method / `RocmAiterUnifiedAttentionBackend.get_supported_kernel_block_sizes` 方法
```python
    @staticmethod
    def get_supported_kernel_block_sizes() -> list[int | MultipleOf]:
        return [MultipleOf(16)]
```
**EN:** This method returns or derives a value within `RocmAiterUnifiedAttentionBackend`. Key calls include `MultipleOf`.
**CN:** 该方法会返回或推导一个值，其作用域位于`RocmAiterUnifiedAttentionBackend`。 关键调用包括 `MultipleOf`。

### `RocmAiterUnifiedAttentionBackend.get_preferred_block_size` method / `RocmAiterUnifiedAttentionBackend.get_preferred_block_size` 方法
```python
    @classmethod
    def get_preferred_block_size(cls, default_block_size: int) -> int:
        logger.warning_once(
            "[ROCM_AITER_UNIFIED_ATTN]: Setting kv cache block size to 64."
        )
        return 64
```
**EN:** This method returns or derives a value within `RocmAiterUnifiedAttentionBackend`. Key calls include `warning_once`.
**CN:** 该方法会返回或推导一个值，其作用域位于`RocmAiterUnifiedAttentionBackend`。 关键调用包括 `warning_once`。

### `RocmAiterUnifiedAttentionBackend.supports_block_size` method / `RocmAiterUnifiedAttentionBackend.supports_block_size` 方法
```python
    @classmethod
    def supports_block_size(cls, block_size: int | None) -> bool:
        if block_size is None:
            return True
        return block_size % 16 == 0
```
**EN:** This method implements `supports_block_size` within `RocmAiterUnifiedAttentionBackend`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `supports_block_size`，其作用域位于`RocmAiterUnifiedAttentionBackend`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RocmAiterUnifiedAttentionBackend.supports_head_size` method / `RocmAiterUnifiedAttentionBackend.supports_head_size` 方法
```python
    @classmethod
    def supports_head_size(cls, head_size: int) -> bool:
        return head_size >= 32
```
**EN:** This method implements `supports_head_size` within `RocmAiterUnifiedAttentionBackend`.
**CN:** 该方法会实现 `supports_head_size`，其作用域位于`RocmAiterUnifiedAttentionBackend`。

### `RocmAiterUnifiedAttentionBackend.supports_mm_prefix` method / `RocmAiterUnifiedAttentionBackend.supports_mm_prefix` 方法
```python
    @classmethod
    def supports_mm_prefix(cls) -> bool:
        return True
```
**EN:** This method implements `supports_mm_prefix` within `RocmAiterUnifiedAttentionBackend`.
**CN:** 该方法会实现 `supports_mm_prefix`，其作用域位于`RocmAiterUnifiedAttentionBackend`。

### `RocmAiterUnifiedAttentionBackend.get_kv_cache_shape` method / `RocmAiterUnifiedAttentionBackend.get_kv_cache_shape` 方法
```python
    @staticmethod
    def get_kv_cache_shape(
        num_blocks: int,
        block_size: int,
        num_kv_heads: int,
        head_size: int,
        cache_dtype_str: str = "auto",
    ) -> tuple[int, ...]:
        if block_size % 16 != 0:
            raise ValueError("Block size must be a multiple of 16.")
        return (2, num_blocks, block_size, num_kv_heads, head_size)
```
**EN:** This method returns or derives a value within `RocmAiterUnifiedAttentionBackend`. Key calls include `ValueError`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`RocmAiterUnifiedAttentionBackend`。 关键调用包括 `ValueError`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RocmAiterUnifiedAttentionBackend.get_builder_cls` method / `RocmAiterUnifiedAttentionBackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["RocmAttentionMetadataBuilder"]:
        return RocmAttentionMetadataBuilder
```
**EN:** This method returns or derives a value within `RocmAiterUnifiedAttentionBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`RocmAiterUnifiedAttentionBackend`。

### `RocmAiterUnifiedAttentionBackend.supports_attn_type` method / `RocmAiterUnifiedAttentionBackend.supports_attn_type` 方法
```python
    @classmethod
    def supports_attn_type(cls, attn_type: str) -> bool:
        """RocmAiterUnifiedAttention supports all attention types."""
        return attn_type in (
            AttentionType.DECODER,
            AttentionType.ENCODER,
            AttentionType.ENCODER_ONLY,
            AttentionType.ENCODER_DECODER,
        )
```
**EN:** This method implements `supports_attn_type` within `RocmAiterUnifiedAttentionBackend`. The docstring frames it as: RocmAiterUnifiedAttention supports all attention types.
**CN:** 该方法会实现 `supports_attn_type`，其作用域位于`RocmAiterUnifiedAttentionBackend`。

### `RocmAiterUnifiedAttentionImpl` class / `RocmAiterUnifiedAttentionImpl` 类
```python
class RocmAiterUnifiedAttentionImpl(RocmAttentionImpl):
```
**EN:** Introduces the `RocmAiterUnifiedAttentionImpl` class on top of `RocmAttentionImpl`. Core methods include `fused_output_quant_supported`, `__init__`, `forward`, `do_kv_cache_update`, `fused_rope_kvcache_supported`, `do_rope_and_kv_cache_update`.
**CN:** 这里定义 `RocmAiterUnifiedAttentionImpl` 类，其基类包括 `RocmAttentionImpl`。核心方法包括 `fused_output_quant_supported`, `__init__`, `forward`, `do_kv_cache_update`, `fused_rope_kvcache_supported`, `do_rope_and_kv_cache_update`。

### `RocmAiterUnifiedAttentionImpl.fused_output_quant_supported` method / `RocmAiterUnifiedAttentionImpl.fused_output_quant_supported` 方法
```python
    def fused_output_quant_supported(self, quant_key: QuantKey):
        return quant_key == kFp8StaticTensorSym
```
**EN:** This method implements `fused_output_quant_supported` within `RocmAiterUnifiedAttentionImpl`.
**CN:** 该方法会实现 `fused_output_quant_supported`，其作用域位于`RocmAiterUnifiedAttentionImpl`。

### `RocmAiterUnifiedAttentionImpl.__init__` method / `RocmAiterUnifiedAttentionImpl.__init__` 方法
```python
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        scale: float,
        num_kv_heads: int,
        alibi_slopes: list[float] | None,
        sliding_window: int | None,
        kv_cache_dtype: str,
        logits_soft_cap: float | None = None,
        attn_type: AttentionType = AttentionType.DECODER,
        kv_sharing_target_layer_name: int | None = None,
        sinks: torch.Tensor | None = None,
    ) -> None:
        super().__init__(
            num_heads,
            head_size,
            scale,
            num_kv_heads,
            alibi_slopes,
            sliding_window,
            kv_cache_dtype,
            logits_soft_cap,
            attn_type,
            kv_sharing_target_layer_name,
            sinks,
        )
        logger.info_once(
            "Using aiter unified attention for RocmAiterUnifiedAttentionImpl"
        )
        from aiter.ops.triton.unified_attention import unified_attention

        self.unified_attention = unified_attention
        self.supports_quant_query_input = True
```
**EN:** This method initializes the object state within `RocmAiterUnifiedAttentionImpl`. Key calls include `__init__`, `info_once`, `super`. It touches state such as `unified_attention`, `supports_quant_query_input`.
**CN:** 该方法会初始化对象状态，其作用域位于`RocmAiterUnifiedAttentionImpl`。 关键调用包括 `__init__`, `info_once`, `super`。 它会读写 `unified_attention`, `supports_quant_query_input` 等状态。

### `RocmAiterUnifiedAttentionImpl.forward` method / `RocmAiterUnifiedAttentionImpl.forward` 方法
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

        # Handle encoder attention differently - no KV cache needed
        if self.attn_type in (AttentionType.ENCODER_ONLY, AttentionType.ENCODER):
            # For encoder attention,
    # ... omitted for brevity ...
        block_table = attn_metadata.block_table

        self.unified_attention(
            q=query[:num_actual_tokens],
            k=key_cache,
            v=value_cache,
            out=output[:num_actual_tokens],
            cu_seqlens_q=cu_seqlens_q,
            max_seqlen_q=max_seqlen_q,
            seqused_k=seqused_k,
            max_seqlen_k=max_seqlen_k,
            softmax_scale=softmax_scale,
            causal=True,
            alibi_slopes=self.alibi_slopes,
            window_size=self.sliding_window,
            block_table=block_table,
            softcap=self.logits_soft_cap,
            q_descale=layer._q_scale if query.dtype == self.fp8_dtype else None,
            k_descale=layer._k_scale,
            v_descale=layer._v_scale,
            sinks=self.sinks,
            output_scale=output_scale,
        )

        return output
```
**EN:** This method drives the forward-pass computation within `RocmAiterUnifiedAttentionImpl`. The docstring frames it as: Forward pass with FlashAttention. Key calls include `unbind`, `is_quantized_kv_cache`, `unified_attention`, `NotImplementedError`, `fill_`, `_forward_encoder_attention`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会驱动前向计算流程，其作用域位于`RocmAiterUnifiedAttentionImpl`。 关键调用包括 `unbind`, `is_quantized_kv_cache`, `unified_attention`, `NotImplementedError`, `fill_`, `_forward_encoder_attention`。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `RocmAiterUnifiedAttentionImpl.do_kv_cache_update` method / `RocmAiterUnifiedAttentionImpl.do_kv_cache_update` 方法
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
            # For encoder attention,
            # we use direct Q, K, V tensors without caching
            return
        key_cache, value_cache = kv_cache.unbind(0)

        # Reshape the input keys and values and store them in the cache.
        ops.reshape_and_cache_flash(
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
**EN:** This method implements `do_kv_cache_update` within `RocmAiterUnifiedAttentionImpl`. Key calls include `unbind`, `reshape_and_cache_flash`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `do_kv_cache_update`，其作用域位于`RocmAiterUnifiedAttentionImpl`。 关键调用包括 `unbind`, `reshape_and_cache_flash`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RocmAiterUnifiedAttentionImpl.fused_rope_kvcache_supported` method / `RocmAiterUnifiedAttentionImpl.fused_rope_kvcache_supported` 方法
```python
    def fused_rope_kvcache_supported(self):
        return rocm_aiter_ops.is_enabled()
```
**EN:** This method implements `fused_rope_kvcache_supported` within `RocmAiterUnifiedAttentionImpl`. Key calls include `is_enabled`.
**CN:** 该方法会实现 `fused_rope_kvcache_supported`，其作用域位于`RocmAiterUnifiedAttentionImpl`。 关键调用包括 `is_enabled`。

### `RocmAiterUnifiedAttentionImpl.do_rope_and_kv_cache_update` method / `RocmAiterUnifiedAttentionImpl.do_rope_and_kv_cache_update` 方法
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
            # For encoder attention,
            # we use direct Q, K, V tensors without caching
            return
        key_cache, value_cache = kv_cache.unbind(0)
        flash_layout = True

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
**EN:** This method implements `do_rope_and_kv_cache_update` within `RocmAiterUnifiedAttentionImpl`. Key calls include `unbind`, `is_quantized_kv_cache`, `triton_rope_and_cache`, `view`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `do_rope_and_kv_cache_update`，其作用域位于`RocmAiterUnifiedAttentionImpl`。 关键调用包括 `unbind`, `is_quantized_kv_cache`, `triton_rope_and_cache`, `view`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `RocmAiterUnifiedAttentionBackend`: central class or interface in this module. / `RocmAiterUnifiedAttentionBackend`：本模块中的核心类或接口。
- `RocmAiterUnifiedAttentionImpl`: central class or interface in this module. / `RocmAiterUnifiedAttentionImpl`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`, `aiter`
- Internal vLLM / 内部依赖: `vllm`, `vllm._aiter_ops`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.rocm_attn`
