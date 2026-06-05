# triton_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/triton_attn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: High-Performance Triton-only Attention layer. / 该模块位于 `attention/backends` 子系统，主要围绕 `TritonAttentionMetadata`, `TritonAttentionMetadataBuilder`, `TritonAttentionBackend` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""High-Performance Triton-only Attention layer."""

from dataclasses import dataclass
from typing import ClassVar

import torch

import vllm.envs as envs
from vllm._aiter_ops import rocm_aiter_ops
from vllm.config import CUDAGraphMode, VllmConfig
from vllm.config.cache import CacheDType
from vllm.logger import init_logger
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kFp8StaticTensorSym,
)
from vllm.platforms import current_platform
from vllm.platforms.interface import DeviceCapability
from vllm.utils.math_utils import next_power_of_2
from vllm.utils.torch_utils import async_tensor_h2d, is_quantized_kv_cache
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
from vllm.v1.attention.backends.utils import get_kv_cache_layout
from vllm.v1.attention.ops.triton_prefill_attention import context_attention_fwd
from vllm.v1.attention.ops.triton_reshape_and_cache_flash import (
    triton_reshape_and_cache_flash,
    triton_reshape_and_cache_flash_per_token_head_quant,
)
from vllm.v1.attention.ops.triton_unified_attention import unified_attention
from vllm.v1.kv_cache_interface import (
    AttentionSpec,
    get_kv_quant_mode,
    kv_cache_uses_per_token_head_scales,
)

logger = init_logger(__name__)


# constants
MIN_LAUNCH_GRID_SIZE_2D = 128  # Minimum launch grid size of 2D kernel
NUM_PAR_SOFTMAX_SEGMENTS = 16  # Number of parallel tiled softmax segments
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `MIN_LAUNCH_GRID_SIZE_2D`, `NUM_PAR_SOFTMAX_SEGMENTS`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `MIN_LAUNCH_GRID_SIZE_2D`, `NUM_PAR_SOFTMAX_SEGMENTS`。

### `TritonAttentionMetadata` class / `TritonAttentionMetadata` 类
```python
@dataclass
class TritonAttentionMetadata:
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

    seq_threshold_3D: int
    num_par_softmax_segments: int
    softmax_segm_output: torch.Tensor
    softmax_segm_max: torch.Tensor
    softmax_segm_expsum: torch.Tensor

    # For cascade attention.
    use_cascade: bool
    common_prefix_len: int
    cu_prefix_query_lens: torch.Tensor | None
    prefix_kv_lens: torch.Tensor | None
    suffix_kv_lens: torch.Tensor | None

    # Optional aot scheduling
    scheduler_metadata: torch.Tensor | None = None
    prefix_scheduler_metadata: torch.Tensor | None = None
    mm_prefix_range: dict[int, list[tuple[int, int]]] | None = None
    mm_prefix_range_tensor: torch.Tensor | None = None
```
**EN:** Uses `@dataclass` to package related state for `TritonAttentionMetadata`. Typical fields include `num_actual_tokens`, `max_query_len`, `query_start_loc`, `max_seq_len`, `seq_lens`, `block_table`.
**CN:** `TritonAttentionMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_actual_tokens`, `max_query_len`, `query_start_loc`, `max_seq_len`, `seq_lens`, `block_table`。

### `TritonAttentionMetadata.compute_mm_prefix_range_tensor` method / `TritonAttentionMetadata.compute_mm_prefix_range_tensor` 方法
```python
    @staticmethod
    def compute_mm_prefix_range_tensor(
        mm_prefix_range: dict[int, list[tuple[int, int]]] | None,
        num_seqs: int,
        device: torch.device,
    ) -> torch.Tensor | None:
        """Convert mm_prefix_range dict to padded tensor for Triton kernel.

        Returns shape: (num_seqs, max_ranges, 2) with 0-padding for empty ranges.
        Empty ranges have start==end==0, which kernel skips via is_valid check.
        """
        if mm_prefix_range is None:
            return None

        # Collect ranges, using [(0,0)] for empty sequences to ensure uniform dims
        range_lists = [
            mm_prefix_range.get(i, [(0, 0)]) or [(0, 0)] for i in range(num_seqs)
        ]

        # Return None if all ranges are trivial (only (0,0) placeholders)
        if all(r == [(0, 0)] for r in range_lists):
            return None

        # Build on CPU first then move to GPU in a single H2D transfer
        max_ranges = max(len(r) for r in range_lists)
        # Pad all sequences to the same number of ranges
        padded = []
        for r in range_lists:
            padded_r = list(r) + [(0, 0)] * (max_ranges - len(r))
            padded.append(padded_r)
        # Build on pinned CPU memory so the H2D transfer is non-blocking.
        padded = async_tensor_h2d(padded, dtype=torch.int32, device=device)
        return padded.view(num_seqs, max_ranges, 2)
```
**EN:** This method computes derived values within `TritonAttentionMetadata`. The docstring frames it as: Convert mm_prefix_range dict to padded tensor for Triton kernel. Key calls include `all`, `max`, `async_tensor_h2d`, `view`, `append`, `get`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会计算派生值，其作用域位于`TritonAttentionMetadata`。 关键调用包括 `all`, `max`, `async_tensor_h2d`, `view`, `append`, `get`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `TritonAttentionMetadataBuilder` class / `TritonAttentionMetadataBuilder` 类
```python
class TritonAttentionMetadataBuilder(AttentionMetadataBuilder[TritonAttentionMetadata]):
    _cudagraph_support: ClassVar[AttentionCGSupport] = AttentionCGSupport.ALWAYS
```
**EN:** Introduces the `TritonAttentionMetadataBuilder` class on top of `AttentionMetadataBuilder[TritonAttentionMetadata]`. Core methods include `__init__`, `build_for_cudagraph_capture`, `build`.
**CN:** 这里定义 `TritonAttentionMetadataBuilder` 类，其基类包括 `AttentionMetadataBuilder[TritonAttentionMetadata]`。核心方法包括 `__init__`, `build_for_cudagraph_capture`, `build`。

### `TritonAttentionMetadataBuilder.__init__` method / `TritonAttentionMetadataBuilder.__init__` 方法
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

        # Check if CUDA Graphs are enabled for decode
        self.decode_cudagraph_enabled = (
            self.vllm_config.compilation_config.cudagraph_mode
            in (
                CUDAGraphMode.FULL_AND_PIECEWISE,
                CUDAGraphMode.FULL_DECODE_ONLY,
                CUDAGraphMode.FULL,
            )
        )

        # The launch grid for the 2D kernel is defined as (num_q_blocks, num_heads_kv).
        # A lower bound for num_q_blocks is the number of sequences.
        # To ensure the minimum launch grid size is achieved, the number of sequences
        # must be at least equal to the threshold below.
        # If this threshold is not reached (i.e., the batch size is not large enough),
        # the 3D kernel will be selected instead.
        self.seq_threshold_3D = MIN_LAUNCH_GRID_SIZE_2D // self.num_heads_kv

        # Modify the threshold if needed.
        if self.decode_cudagraph_enabled:
            capture_sizes = self.vllm_config.compilation_config.cudagraph_capture_sizes
            assert capture_sizes, "CUDA Graphs enabled but no capture sizes specified."

            # Select the CUDA Graph capture size closest to self.seq_threshold_3D
            # as threshold. This ensures that each captured graph covers the
            # correct execution path.
            self.seq_threshold_3D = min(
                capture_sizes,
                key=lambda x: abs(x - self.seq_threshold_3D),
            )

        self.num_par_softmax_segments = NUM_PAR_SOFTMAX_SEGMENTS
        headdim_padded = next_power_of_2(self.headdim)
        self.softmax_segm_output = torch.empty(
            (
                self.seq_threshold_3D,
                self.num_heads_q,
                self.num_par_softmax_segments,
                headdim_padded,
            ),
            dtype=torch.float32,
            device=device,
        )
        self.softmax_segm_max = torch.empty(
            (self.seq_threshold_3D, self.num_heads_q, self.num_par_softmax_segments),
            dtype=torch.float32,
            device=device,
        )
        self.softmax_segm_expsum = torch.empty(
            (self.seq_threshold_3D, self.num_heads_q, self.num_par_softmax_segments),
            dtype=torch.float32,
            device=device,
        )
```
**EN:** This method initializes the object state within `TritonAttentionMetadataBuilder`. Key calls include `__init__`, `get_num_attention_heads`, `get_num_kv_heads`, `get_head_size`, `next_power_of_2`, `empty`. It touches state such as `block_size`, `num_heads_q`, `num_heads_kv`, `headdim`, `decode_cudagraph_enabled`, `seq_threshold_3D`, `num_par_softmax_segments`, `softmax_segm_output`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`TritonAttentionMetadataBuilder`。 关键调用包括 `__init__`, `get_num_attention_heads`, `get_num_kv_heads`, `get_head_size`, `next_power_of_2`, `empty`。 它会读写 `block_size`, `num_heads_q`, `num_heads_kv`, `headdim`, `decode_cudagraph_enabled`, `seq_threshold_3D`, `num_par_softmax_segments`, `softmax_segm_output` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TritonAttentionMetadataBuilder.build_for_cudagraph_capture` method / `TritonAttentionMetadataBuilder.build_for_cudagraph_capture` 方法
```python
    def build_for_cudagraph_capture(
        self, common_attn_metadata: CommonAttentionMetadata
    ) -> TritonAttentionMetadata:
        attn_metadata = self.build(0, common_attn_metadata)
        # When doing full graph capture, setting seq_lens to
        # max_model_len will cause graph capture to be extremely
        # slow, so here we set it to 1.
        attn_metadata.seq_lens.fill_(1)
        return attn_metadata
```
**EN:** This method builds derived structures within `TritonAttentionMetadataBuilder`. Key calls include `build`, `fill_`.
**CN:** 该方法会构建派生结构，其作用域位于`TritonAttentionMetadataBuilder`。 关键调用包括 `build`, `fill_`。

### `TritonAttentionMetadataBuilder.build` method / `TritonAttentionMetadataBuilder.build` 方法
```python
    def build(
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        fast_build: bool = False,
    ) -> TritonAttentionMetadata:
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

        attn_metadata = TritonAttentionMetadata(
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
            seq_threshold_3D=self.seq_threshold_3D,
            num_par_softmax_segments=self.num_par_softmax_segments,
            softmax_segm_output=self.softmax_segm_output,
            softmax_segm_max=self.softmax_segm_max,
            softmax_segm_expsum=self.softmax_segm_expsum,
        )
        return attn_metadata
```
**EN:** This method builds derived structures within `TritonAttentionMetadataBuilder`. Key calls include `TritonAttentionMetadata`, `tensor`, `to`, `cpu`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会构建派生结构，其作用域位于`TritonAttentionMetadataBuilder`。 关键调用包括 `TritonAttentionMetadata`, `tensor`, `to`, `cpu`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TritonAttentionBackend` class / `TritonAttentionBackend` 类
```python
class TritonAttentionBackend(AttentionBackend):
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
        "int8_per_token_head",
        "fp8_per_token_head",
    ]
```
**EN:** Introduces the `TritonAttentionBackend` class on top of `AttentionBackend`. Core methods include `get_supported_kernel_block_sizes`, `supports_block_size`, `get_name`, `supports_batch_invariance`, `get_impl_cls`, `get_kv_cache_shape`.
**CN:** 这里定义 `TritonAttentionBackend` 类，其基类包括 `AttentionBackend`。核心方法包括 `get_supported_kernel_block_sizes`, `supports_block_size`, `get_name`, `supports_batch_invariance`, `get_impl_cls`, `get_kv_cache_shape`。

### `TritonAttentionBackend.get_kv_cache_shape` method / `TritonAttentionBackend.get_kv_cache_shape` 方法
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
        if kv_cache_uses_per_token_head_scales(cache_dtype_str):
            # Pad head_size by sizeof(float32)/sizeof(cache_dtype) so
            # the per-head scale fits inline.  The backend extracts
            # data[:head_size] and scale[head_size:] via typed views.
            from vllm.utils.torch_utils import (
                STR_DTYPE_TO_TORCH_DTYPE,
                get_dtype_size,
            )

            cache_dtype = STR_DTYPE_TO_TORCH_DTYPE[cache_dtype_str]
            scale_pad = get_dtype_size(torch.float32) // get_dtype_size(cache_dtype)
            return (num_blocks, 2, block_size, num_kv_heads, head_size + scale_pad)
        return (num_blocks, 2, block_size, num_kv_heads, head_size)
```
**EN:** This method returns or derives a value within `TritonAttentionBackend`. Key calls include `kv_cache_uses_per_token_head_scales`, `ValueError`, `get_dtype_size`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`TritonAttentionBackend`。 关键调用包括 `kv_cache_uses_per_token_head_scales`, `ValueError`, `get_dtype_size`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TritonAttentionBackend.get_builder_cls` method / `TritonAttentionBackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["TritonAttentionMetadataBuilder"]:
        return TritonAttentionMetadataBuilder
```
**EN:** This method returns or derives a value within `TritonAttentionBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`TritonAttentionBackend`。

### `TritonAttentionBackend.supports_compute_capability` method / `TritonAttentionBackend.supports_compute_capability` 方法
```python
    @classmethod
    def supports_compute_capability(cls, capability: DeviceCapability) -> bool:
        return True
```
**EN:** This method implements `supports_compute_capability` within `TritonAttentionBackend`.
**CN:** 该方法会实现 `supports_compute_capability`，其作用域位于`TritonAttentionBackend`。

### `TritonAttentionImpl` class / `TritonAttentionImpl` 类
```python
class TritonAttentionImpl(AttentionImpl):
    # Per-token-head quant: scale views carved from inline head padding.
    _k_scale_cache: torch.Tensor | None = None
    _v_scale_cache: torch.Tensor | None = None
```
**EN:** Introduces the `TritonAttentionImpl` class on top of `AttentionImpl`. Core methods include `_ensure_scale_caches`, `fused_output_quant_supported`, `__init__`, `forward`, `_forward_encoder_attention`, `do_kv_cache_update`.
**CN:** 这里定义 `TritonAttentionImpl` 类，其基类包括 `AttentionImpl`。核心方法包括 `_ensure_scale_caches`, `fused_output_quant_supported`, `__init__`, `forward`, `_forward_encoder_attention`, `do_kv_cache_update`。

### `TritonAttentionImpl.__init__` method / `TritonAttentionImpl.__init__` 方法
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
        use_alibi_sqrt: bool = False,
        chunk_lookback: int = -1,
    ) -> None:
        self.num_heads = num_heads
        self.head_size = head_size
        self.scale = float(scale)
        self.num_kv_heads = num_kv_heads
        if alibi_slopes is not None:
            alibi_slopes = torch.tensor(alibi_slopes, dtype=torch.float32)
        self.alibi_slopes = alibi_slopes
        if sliding_window is None:
            self.sliding_window = (-1, -1)
        elif attn_type in (AttentionType.ENCODER, AttentionType.ENCODER_ONLY):
            self.sliding_window = (sliding_window - 1, sliding_window - 1)
        else:
            self.sliding_window = (sliding_window - 1, 0)
        self.kv_cache_dtype = kv_cache_dtype
        if logits_soft_cap is None:
            # In flash-attn, setting logits_soft_cap as 0 means no soft cap.
            logits_soft_cap = 0
        self.logits_soft_cap = logits_soft_cap
        self.kv_sharing_target_layer_name = kv_sharing_target_layer_name

        self.num_queries_per_kv = self.num_heads // self.num_kv_heads

        self.attn_type = attn_type
        self.fp8_dtype = current_platform.fp8_dtype()

        self.sinks = sinks
        if sinks is not None:
            assert sinks.shape[0] == num_heads, (
                "Sinks must have the same number of heads as the number of "
                f"heads in the layer. Sinks shape: {sinks.shape}, "
                f"num_heads: {num_heads}."
            )
        self.use_alibi_sqrt = use_alibi_sqrt
        self.chunk_lookback = chunk_lookback
        self.supports_quant_query_input = current_platform.is_cuda()

        self._kv_quant_mode = get_kv_quant_mode(kv_cache_dtype)
        self._is_per_token_head_quant = self._kv_quant_mode.is_per_token_head

        # Enable tensor descriptors for Q/K/V load/store on platforms that
        # benefit from HW 2D block reads (Intel Xe2/Xe3).  The dead branch
        # is eliminated at Triton compile time, so other platforms see
        # zero cost when TD is off.
        #
        # ``VLLM_TRITON_ATTN_USE_TD`` is tri-state:
        #   - unset (None): auto-select (TD on for XPU, off elsewhere),
        #   - ``1``: force TD on regardless of platform,
        #   - ``0``: force TD off regardless of platform (useful for A/B).
        td_override = envs.VLLM_TRITON_ATTN_USE_TD
        if td_override is None:
            self.use_td = current_platform.is_xpu()
        else:
            self.use_td = td_override
```
**EN:** This method initializes the object state within `TritonAttentionImpl`. Key calls include `float`, `fp8_dtype`, `is_cuda`, `get_kv_quant_mode`, `tensor`, `is_xpu`. It touches state such as `num_heads`, `head_size`, `scale`, `num_kv_heads`, `alibi_slopes`, `kv_cache_dtype`, `logits_soft_cap`, `kv_sharing_target_layer_name`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`TritonAttentionImpl`。 关键调用包括 `float`, `fp8_dtype`, `is_cuda`, `get_kv_quant_mode`, `tensor`, `is_xpu`。 它会读写 `num_heads`, `head_size`, `scale`, `num_kv_heads`, `alibi_slopes`, `kv_cache_dtype`, `logits_soft_cap`, `kv_sharing_target_layer_name` 等状态。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TritonAttentionImpl.forward` method / `TritonAttentionImpl.forward` 方法
```python
    def forward(
        self,
        layer: torch.nn.Module,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        kv_cache: torch.Tensor,
        attn_metadata: TritonAttentionMetadata,
        output: torch.Tensor,
        output_scale: torch.Tensor | None = None,
        output_block_scale: torch.Tensor | None = None,
    ) -> torch.Tensor:
        """Forward pass with Paged Attention impl. in Triton.

        Args:
            query: shape = [num_tokens, num_heads, head_size]
            key: shape = [num_tokens, num_kv_heads, head_size]
            value: shape = [num_tokens, num_kv_heads, head_size]
            kv_cache: shape =
                [num_blocks, 2, block_size, num_kv_heads, head_size]
            attn_metadata: Metadata for attention.
        Returns:
            shape = [num_tokens, num_heads * head_size]
        """
        if output_block_scale is not None:
            raise NotImplementedError(
                "fused block_scale output quantization is not yet supported"
                " for TritonAttentionImpl"
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
            causal=True,
            alibi_slopes=self.alibi_slopes,
            use_alibi_sqrt=self.use_alibi_sqrt,
            window_size=self.sliding_window,
            block_table=block_table,
            softcap=self.logits_soft_cap,
            q_descale=None,  # Not supported
            k_descale=k_descale,
            v_descale=v_descale,
            seq_threshold_3D=seq_threshold_3D,
            num_par_softmax_segments=num_par_softmax_segments,
            softmax_segm_output=softmax_segm_output,
            softmax_segm_max=softmax_segm_max,
            softmax_segm_expsum=softmax_segm_expsum,
            sinks=self.sinks,
            output_scale=output_scale,
            mm_prefix_range=mm_prefix_range_tensor,
            kv_quant_mode=self._kv_quant_mode,
            k_scale_cache=k_scale_cache,
            v_scale_cache=v_scale_cache,
            chunk_lookback=self.chunk_lookback,
            use_td=self.use_td,
        )

        return output
```
**EN:** This method drives the forward-pass computation within `TritonAttentionImpl`. The docstring frames it as: Forward pass with Paged Attention impl. Key calls include `unified_attention`, `NotImplementedError`, `fill_`, `_forward_encoder_attention`, `_ensure_scale_caches`, `unbind`. The control flow contains 7 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会驱动前向计算流程，其作用域位于`TritonAttentionImpl`。 关键调用包括 `unified_attention`, `NotImplementedError`, `fill_`, `_forward_encoder_attention`, `_ensure_scale_caches`, `unbind`。 控制流包含 7 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `TritonAttentionImpl.do_kv_cache_update` method / `TritonAttentionImpl.do_kv_cache_update` 方法
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
        # Reshape the input keys and values and store them in the cache.
        if self._is_per_token_head_quant:
            self._ensure_scale_caches(kv_cache)
            key_cache, value_cache = kv_cache.unbind(1)
            if key_cache.dtype == torch.uint8:
                key_cache = key_cache.view(self.fp8_dtype)
                value_cache = value_cache.view(self.fp8_dtype)
            triton_reshape_and_cache_flash_per_token_head_quant(
                key,
                value,
                key_cache,
                value_cache,
                self._k_scale_cache,
                self._v_scale_cache,
                slot_mapping,
            )
            return
        # For decoder and cross-attention, use KV cache as before.
        key_cache, value_cache = kv_cache.unbind(1)
        if is_quantized_kv_cache(self.kv_cache_dtype):
            key_cache = key_cache.view(self.fp8_dtype)
            value_cache = value_cache.view(self.fp8_dtype)
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
**EN:** This method implements `do_kv_cache_update` within `TritonAttentionImpl`. Key calls include `unbind`, `is_quantized_kv_cache`, `triton_reshape_and_cache_flash`, `_ensure_scale_caches`, `triton_reshape_and_cache_flash_per_token_head_quant`, `view`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `do_kv_cache_update`，其作用域位于`TritonAttentionImpl`。 关键调用包括 `unbind`, `is_quantized_kv_cache`, `triton_reshape_and_cache_flash`, `_ensure_scale_caches`, `triton_reshape_and_cache_flash_per_token_head_quant`, `view`。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `TritonAttentionMetadata`: central class or interface in this module. / `TritonAttentionMetadata`：本模块中的核心类或接口。
- `TritonAttentionMetadataBuilder`: central class or interface in this module. / `TritonAttentionMetadataBuilder`：本模块中的核心类或接口。
- `TritonAttentionBackend`: central class or interface in this module. / `TritonAttentionBackend`：本模块中的核心类或接口。
- `TritonAttentionImpl`: central class or interface in this module. / `TritonAttentionImpl`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm._aiter_ops`, `vllm.config`, `vllm.config.cache`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.platforms.interface`, `vllm.utils.math_utils`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.utils`
