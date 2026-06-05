# _xpu_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/_xpu_ops.py`
- **Repository**: vllm-project/vllm
- **Purpose**: EN: Register Intel XPU-specific custom ops and provide Python fallbacks/helpers for attention, quantization, and cache indexing. / CN: 注册 Intel XPU 专用自定义算子，并为注意力、量化和缓存索引提供 Python 侧回退与辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Optional fake registrations for XPU ops
```python
if TYPE_CHECKING:

    def register_fake(fn):
        return lambda name: fn
else:
    try:
        from torch.library import register_fake
    except ImportError:
        from torch.library import impl_abstract as register_fake

if hasattr(torch.ops._xpu_C, "fp8_gemm"):

    @register_fake("_xpu_C::fp8_gemm")
    def _fp8_gemm_fake(
        q_input: torch.Tensor,
        q_weight: torch.Tensor,
        out_dtype: torch.dtype,
        input_scales: torch.Tensor,
        weight_scale: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        input_2d = q_input.view(-1, q_input.shape[-1])
        M = input_2d.size(0)
        N = q_weight.size(1)
        return torch.empty((M, N), dtype=out_dtype, device=q_input.device)


if hasattr(torch.ops._xpu_C, "fp8_gemm_w8a16"):

    @register_fake("_xpu_C::fp8_gemm_w8a16")
    def _fp8_gemm_w8a16_fake(
        input: torch.Tensor,
        q_weight: torch.Tensor,
        weight_scale: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        input_2d = input.view(-1, input.shape[-1])
        M = input_2d.size(0)
        N = q_weight.size(1)
        return torch.empty((M, N), dtype=input.dtype, device=input.device)


if hasattr(torch.ops._xpu_C, "int4_gemm_w4a8"):

    @register_fake("_xpu_C::int4_gemm_w4a8")
    def _int4_gemm_w4a8_fake(
        input: torch.Tensor,
        input_scales: torch.Tensor,
        input_zero_points: torch.Tensor,
        q_weight: torch.Tensor,
        weight_scale: torch.Tensor,
        weight_zp: torch.Tensor,
        group_size: int,
        g_idx: torch.Tensor | None = None,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        input_2d = input.view(-1, input.shape[-1])
        M = input_2d.size(0)
        N = q_weight.size(1)
        return torch.empty((M, N), dtype=torch.float16, device=input.device)


if hasattr(torch.ops._xpu_C, "int4_gemm_w4a16"):

    @register_fake("_xpu_C::int4_gemm_w4a16")
    def _int4_gemm_w4a16_fake(
        input: torch.Tensor,
        q_weight: torch.Tensor,
        bias: torch.Tensor | None,
        weight_scale: torch.Tensor,
        qzeros: torch.Tensor,
        group_size: int,
        group_idx: torch.Tensor | None = None,
    ) -> torch.Tensor:
        input_2d = input.view(-1, input.shape[-1])
        M = input_2d.size(0)
        N = q_weight.size(1)
        return torch.empty((M, N), dtype=input.dtype, device=input.device)
```
**EN:** The file only registers fake implementations for operators that actually exist in `torch.ops._xpu_C`, which avoids crashing on partial XPU builds. Each fake function computes the correct output shape from matrix dimensions and chooses the expected dtype, giving `torch.compile` and export flows enough metadata to reason about FP8 and INT4 GEMMs without executing device code.
**CN:** 这个文件只会为 `torch.ops._xpu_C` 中真实存在的算子注册 fake 实现，因此在功能不完整的 XPU 构建上也不会因为注册失败而崩溃。每个 fake 函数都会根据矩阵维度推导正确的输出形状，并选择预期 dtype，从而让 `torch.compile` 和导出流程在不执行设备代码的情况下也能理解 FP8 / INT4 GEMM 的结果。

### GDN attention bridge
```python
def _gdn_attention_core_xpu_impl(
    core_attn_out: torch.Tensor,
    z: torch.Tensor,
    projected_states_qkvz: torch.Tensor,
    projected_states_ba: torch.Tensor,
    layer_name: str,
) -> None:
    """Custom op wrapping the XPU SYCL GDN kernel for torch.compile."""
    from vllm.forward_context import get_forward_context
    from vllm.v1.attention.backends.gdn_attn import GDNAttentionMetadata

    forward_context = get_forward_context()
    self = forward_context.no_compile_layers[layer_name]
    attn_metadata_raw = forward_context.attn_metadata

    if attn_metadata_raw is None:
        return

    assert isinstance(attn_metadata_raw, dict)
    attn_metadata = attn_metadata_raw[self.prefix]
    assert isinstance(attn_metadata, GDNAttentionMetadata)

    # TODO: xpu does not support speculative decoding yet
    assert attn_metadata.spec_sequence_masks is None  # type: ignore[attr-defined]

    conv_weights = self.conv1d.weight.view(
        self.conv1d.weight.size(0), self.conv1d.weight.size(2)
    )

    torch.ops._xpu_C.gdn_attention(
        core_attn_out,
        z,
        projected_states_qkvz,
        projected_states_ba,
        self.num_k_heads,
        self.num_v_heads,
        self.head_k_dim,
        self.head_v_dim,
        conv_state=self.kv_cache[0],
        ssm_state=self.kv_cache[1],
        conv_weights=conv_weights,
        conv_bias=self.conv1d.bias,
        activation=self.activation,
        A_log=self.A_log,
        dt_bias=self.dt_bias,
        num_prefills=attn_metadata.num_prefills,  # type: ignore[attr-defined]
        num_decodes=attn_metadata.num_decodes,  # type: ignore[attr-defined]
        has_initial_state=attn_metadata.has_initial_state,  # type: ignore[attr-defined]
        non_spec_query_start_loc=attn_metadata.non_spec_query_start_loc,  # type: ignore[attr-defined]
        non_spec_state_indices_tensor=attn_metadata.non_spec_state_indices_tensor,  # type: ignore[attr-defined]
        num_actual_tokens=attn_metadata.num_actual_tokens,  # type: ignore[attr-defined]
        tp_size=self.tp_size,
        reorder_input=not self.gqa_interleaved_layout,
    )
```
**EN:** `_gdn_attention_core_xpu_impl()` bridges vLLM's high-level forward context into a monolithic XPU kernel for Gated DeltaNet attention. It looks up the live layer object by name, extracts decode metadata, reshapes the conv1d weights into the 2D form the backend expects, and passes both convolution state and SSM state from the layer cache into `torch.ops._xpu_C.gdn_attention`.
**CN:** `_gdn_attention_core_xpu_impl()` 把 vLLM 的高层 forward context 桥接到一个面向 Gated DeltaNet attention 的单体 XPU 内核。它会根据层名找到运行中的层对象，提取解码元数据，把 conv1d 权重重排成后端期望的 2D 形式，并把卷积状态与 SSM 状态一起从层缓存传入 `torch.ops._xpu_C.gdn_attention`。

### RoPE and sampling wrappers
```python
def _xpu_ops_deepseek_scaling_rope_impl(
    positions: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor | None,
    offsets: torch.Tensor | None,
    cos_sin_cache: torch.Tensor | None,
    rotary_dim: int,
    is_neox_style: bool,
) -> tuple[torch.Tensor, torch.Tensor]:
    assert key is not None
    return torch.ops._xpu_C.deepseek_scaling_rope(
        positions, query, key, offsets, cos_sin_cache, rotary_dim, is_neox_style
    )


def _xpu_ops_deepseek_scaling_rope_fake(
    positions: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor | None,
    offsets: torch.Tensor | None,
    cos_sin_cache: torch.Tensor | None,
    rotary_dim: int,
    is_neox_style: bool,
) -> tuple[torch.Tensor, torch.Tensor]:
    return query, key


def _topk_topp_sample_impl(
    random_sampled: torch.Tensor,
    logits_to_return: torch.Tensor | None,
    logits: torch.Tensor,
    k: torch.Tensor | None,
    p: torch.Tensor | None,
    logprobs_mode: str,
    seeds: torch.Tensor | None,
    lambda_: float = 1.0,
) -> None:
    torch.ops._xpu_C.topk_topp_sampler(
        random_sampled, logits_to_return, logits, k, p, logprobs_mode, seeds, lambda_
    )
    return


def _topk_topp_sample_fake(
    random_sampled: torch.Tensor,
    logits_to_return: torch.Tensor | None,
    logits: torch.Tensor,
    k: torch.Tensor | None,
    p: torch.Tensor | None,
    logprobs_mode: str,
    seeds: torch.Tensor | None,
    lambda_: float = 1.0,
) -> None:
    return
```
**EN:** These wrappers are intentionally thin. The DeepSeek scaling-RoPE helper just delegates to an XPU kernel and returns updated query/key tensors, while the top-k/top-p sampler mutates preallocated output buffers in-place. Their fake implementations either pass through inputs or return `None`, which is sufficient for tracing because shape semantics are straightforward even though sampling behavior is not simulated.
**CN:** 这些包装函数刻意保持得很薄。DeepSeek scaling-RoPE 辅助函数只是简单委托给 XPU 内核并返回更新后的 query/key；top-k/top-p sampler 则原地写入预分配的输出缓冲区。对应的 fake 实现要么直接透传输入，要么返回 `None`，对于 tracing 来说已经足够，因为它只需要形状语义，而不需要真的模拟采样行为。

### MXFP8 quantization helper
```python
def _xpu_mxfp8_quantize_impl(
    x: torch.Tensor, dtype: torch.dtype | None = None
) -> tuple[torch.Tensor, torch.Tensor]:
    MXFP8_BLOCK_SIZE = 32
    assert x.shape[-1] % MXFP8_BLOCK_SIZE == 0
    if dtype is not None:
        assert dtype in (torch.float8_e4m3fn, torch.float8_e5m2), (
            f"Unsupported dtype for xpu_mxfp8_quantize: {dtype}. "
            f"Expected torch.float8_e4m3fn or torch.float8_e5m2."
        )
    else:
        dtype = current_platform.fp8_dtype()

    finfo = torch.finfo(dtype)
    fp8_min = finfo.min
    fp8_max = finfo.max
    eps = 1e-10

    x_q = torch.empty_like(x, device=x.device, dtype=dtype)
    shape = x.shape[:-1] + (x.shape[-1] // MXFP8_BLOCK_SIZE,)
    x_s = torch.empty(shape, device=x.device, dtype=torch.float32)
    torch.ops._C.per_token_group_fp8_quant(
        x, x_q, x_s, MXFP8_BLOCK_SIZE, eps, fp8_min, fp8_max, True
    )
    x_s = x_s.to(torch.float8_e8m0fnu)
    return x_q, x_s


def _xpu_mxfp8_quantize_fake(
    x: torch.Tensor, dtype: torch.dtype | None = None
) -> tuple[torch.Tensor, torch.Tensor]:
    if dtype is None:
        dtype = current_platform.fp8_dtype()

    MXFP8_BLOCK_SIZE = 32

    shape = x.shape[:-1] + (x.shape[-1] // MXFP8_BLOCK_SIZE,)
    x_s = torch.zeros(shape, device=x.device, dtype=torch.float32)

    return x.to(dtype), x_s.to(torch.float8_e8m0fnu)
```
**EN:** MXFP8 quantization is built on a more generic core op: `torch.ops._C.per_token_group_fp8_quant`. The helper enforces 32-element groups, validates or picks the FP8 dtype, allocates per-group float32 scales, and finally converts those scales to `float8_e8m0fnu` so they match MX-style storage conventions used by downstream kernels.
**CN:** MXFP8 量化建立在更通用的核心算子 `torch.ops._C.per_token_group_fp8_quant` 之上。该辅助函数强制使用 32 元素分组，校验或选择 FP8 dtype，分配逐组 float32 scale，并在最后把这些 scale 转成 `float8_e8m0fnu`，以匹配下游内核采用的 MX 风格存储约定。

### MXFP4 quantization helper
```python
def _xpu_mxfp4_quantize_impl(
    x: torch.Tensor,
) -> tuple[torch.Tensor, torch.Tensor]:
    MXFP4_BLOCK_SIZE = 32
    eps = 1e-10
    assert x.ndim == 2, "input must be 2-D"
    assert x.shape[-1] % MXFP4_BLOCK_SIZE == 0, (
        f"last dimension {x.shape[-1]} must be divisible by group_size "
        f"{MXFP4_BLOCK_SIZE}"
    )
    assert x.is_contiguous(), "input groups must be contiguous"

    M, N = x.shape

    # Packed FP4 output: two nibbles per byte
    x_q = torch.empty(M, N // 2, device=x.device, dtype=torch.uint8)
    x_s = torch.empty(M, N // MXFP4_BLOCK_SIZE, device=x.device, dtype=torch.float32)

    torch.ops._C.per_token_group_quant_mxfp4(x, x_q, x_s, MXFP4_BLOCK_SIZE, eps)

    x_q = x_q.view(torch.float4_e2m1fn_x2)
    x_s = x_s.to(dtype=torch.float8_e8m0fnu, memory_format=torch.preserve_format)
    return x_q, x_s


def _xpu_mxfp4_quantize_fake(
    x: torch.Tensor,
) -> tuple[torch.Tensor, torch.Tensor]:
    MXFP4_BLOCK_SIZE = 32
    M, N = x.shape

    # Packed FP4 output: two nibbles per byte
    x_q = torch.empty(M, N // 2, device=x.device, dtype=torch.uint8)
    x_s = torch.empty(M, N // MXFP4_BLOCK_SIZE, device=x.device, dtype=torch.float32)

    x_q = x_q.view(torch.float4_e2m1fn_x2)
    x_s = x_s.to(dtype=torch.float8_e8m0fnu, memory_format=torch.preserve_format)
    return x_q, x_s
```
**EN:** The MXFP4 path follows the same 32-element grouping scheme but packs two 4-bit values into each byte. After the backend fills a `uint8` payload plus float32 scales, the code reinterprets the payload as `torch.float4_e2m1fn_x2` and converts the scales to E8M0 format, which is the representation expected by MXFP4 kernels.
**CN:** MXFP4 路径沿用了相同的 32 元素分组策略，但会把两个 4-bit 数打包到一个字节里。后端填充完 `uint8` 载荷和 float32 scale 之后，代码会把载荷重解释为 `torch.float4_e2m1fn_x2`，并把 scale 转成 E8M0 格式，这正是 MXFP4 内核期望的表示方式。

### Reference quantization and flash-attention adapter
```python
class xpu_ops:
    @staticmethod
    @torch.compile
    def dynamic_per_token_int8_quant_ref(
        input: torch.Tensor, use_sym_quant: bool, bits: int
    ):
        original_sizes = input.size()
        # view is not safe in torch.compile if input is not contiguous
        input = input.reshape(
            -1, original_sizes[-1]
        )  # Flatten except for the last dimension
        qmin = -(2 ** (bits - 1)) if use_sym_quant else 0
        qmax = 2 ** (bits - 1) - 1 if use_sym_quant else 2**bits - 1
        min_val = torch.min(input, dim=-1)[0].to(dtype=torch.float32).unsqueeze(-1)
        max_val = torch.max(input, dim=-1)[0].to(dtype=torch.float32).unsqueeze(-1)
        if use_sym_quant:
            scale = (
                torch.maximum(torch.abs(min_val), torch.abs(max_val)) / qmax
            ).clamp(min=1e-5)
            zero_point = torch.zeros_like(scale).to(dtype=torch.int32)
        else:
            scale = ((max_val - min_val) / qmax).clamp(min=1e-5)
            zero_point = -1 * torch.round(min_val / scale).to(dtype=torch.int32)
        scale = scale.to(dtype=input.dtype)
        quantized = torch.clamp(
            torch.round(input / scale.to(dtype=torch.float32) + zero_point),
            qmin,
            qmax,
        ).to(dtype=torch.int8 if use_sym_quant else torch.uint8)
        return (
            quantized.view(original_sizes),
            scale.view(original_sizes[:-1] + (1,)),
            zero_point.view(original_sizes[:-1] + (1,)),
        )

    @staticmethod
    def flash_attn_varlen_func(
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        cu_seqlens_q: torch.Tensor,
        max_seqlen_q: int,
        max_seqlen_k: int,
        softmax_scale: float | None = None,
        causal: bool = False,
        out: torch.Tensor | None = None,
        block_table: torch.Tensor | None = None,
        alibi_slopes: torch.Tensor | None = None,
        window_size: list[int] | None = None,
        softcap: float | None = 0.0,
        seqused_k: torch.Tensor | None = None,
        cu_seqlens_k: torch.Tensor | None = None,
        # passed in qwen vl
        dropout_p: float = 0.0,
        # The following parameters are not used in xpu kernel currently,
        # we keep API compatible to CUDA's.
        scheduler_metadata=None,
        fa_version: int = 2,
        q_descale=None,
        k_descale=None,
        v_descale=None,
        num_splits=0,
        return_softmax_lse: bool | None = False,
        s_aux: torch.Tensor | None = None,
        return_attn_probs: bool | None = False,
    ):
        assert cu_seqlens_k is not None or seqused_k is not None, (
            "cu_seqlens_k or seqused_k must be provided"
        )
        assert cu_seqlens_k is None or seqused_k is None, (
            "cu_seqlens_k and seqused_k cannot be provided at the same time"
        )
        assert block_table is None or seqused_k is not None, (
            "when enable block_table, seqused_k is needed"
        )
        assert block_table is not None or cu_seqlens_k is not None, (
            "when block_table is disabled, cu_seqlens_k is needed"
        )
        if out is None:
            out = torch.empty(q.shape, dtype=q.dtype, device=q.device)
        real_window_size: tuple[int, int]
        if window_size is None:
            real_window_size = (-1, -1)
        else:
            assert len(window_size) == 2
            real_window_size = (window_size[0], window_size[1])  # noqa: F841

        return flash_attn_varlen_func(
            out=out,
            q=q,
            k=k,
            v=v,
            cu_seqlens_q=cu_seqlens_q,
            cu_seqlens_k=cu_seqlens_k,
            seqused_k=seqused_k,
            max_seqlen_q=max_seqlen_q,
            max_seqlen_k=max_seqlen_k,
            softmax_scale=softmax_scale,
            causal=causal,
            block_table=block_table,
            s_aux=s_aux,
            window_size=real_window_size,
            # alibi_slopes = alibi_slopes,
            # softcap=softcap,
            return_softmax_lse=return_softmax_lse,
            q_descale=q_descale,
            k_descale=k_descale,
            v_descale=v_descale,
        )
```
**EN:** The `xpu_ops` class mixes true backend bridges with Python reference paths. `dynamic_per_token_int8_quant_ref()` is a compiled pure-PyTorch reference that computes min/max, scales, zero-points, and quantized outputs for each token row; `flash_attn_varlen_func()` reshapes the XPU FlashAttention interface to match the CUDA-style API, validates mutually exclusive sequence-length inputs, normalizes `window_size`, and auto-allocates the output tensor when the caller does not supply one.
**CN:** `xpu_ops` 这个类把真实后端桥接与 Python 参考路径放在了一起。`dynamic_per_token_int8_quant_ref()` 是一个经过编译的纯 PyTorch 参考实现，会逐 token 行计算 min/max、scale、zero-point 和量化结果；`flash_attn_varlen_func()` 则把 XPU 的 FlashAttention 接口适配成与 CUDA 风格一致的 API，校验互斥的序列长度输入，规范化 `window_size`，并在调用方未提供输出时自动分配输出张量。

### Pure-PyTorch K-cache quantize-and-store fallback
```python
    @staticmethod
    def indexer_k_quant_and_cache(
        k: torch.Tensor,
        kv_cache: torch.Tensor,
        slot_mapping: torch.Tensor,
        quant_block_size: int,
        scale_fmt: str | None,
    ) -> None:
        head_dim = k.shape[-1]
        k = k.view(-1, head_dim)  # [total_tokens, head_dim]

        def group_quant_torch(
            x: torch.Tensor,
            group_size: int,
            eps: float = 1e-10,
            dtype: torch.dtype | None = None,
            column_major_scales: bool = False,
            out_q: torch.Tensor | None = None,
            use_ue8m0: bool | None = None,
        ) -> tuple[torch.Tensor, torch.Tensor]:
            if use_ue8m0 is None:
                # Default fallback - could import is_deep_gemm_e8m0_used if needed
                use_ue8m0 = False

            if dtype is None:
                dtype = current_platform.fp8_dtype()

            # Validate inputs
            assert x.shape[-1] % group_size == 0, (
                f"Last dimension {x.shape[-1]} must be divisible by "
                f"group_size {group_size}"
            )
            assert x.stride(-1) == 1, "Input tensor groups must be contiguous"

            # Prepare output tensor
            if out_q is None:
                x_q = torch.empty_like(x, dtype=dtype)
            else:
                assert out_q.shape == x.shape
                x_q = out_q

            # Reshape input for group processing
            # Original shape: (..., last_dim)
            # Target shape: (..., num_groups, group_size)
            original_shape = x.shape
            num_groups = original_shape[-1] // group_size

            # Reshape to separate groups
            group_shape = original_shape[:-1] + (num_groups, group_size)
            x_grouped = x.view(group_shape)

            # Compute per-group absolute maximum values
            # Shape: (..., num_groups)
            abs_max = torch.amax(torch.abs(x_grouped), dim=-1, keepdim=False)
            abs_max = torch.maximum(
                abs_max, torch.tensor(eps, device=x.device, dtype=x.dtype)
            )

            # Compute scales
            FP8_MAX = torch.finfo(dtype).max
            FP8_MIN = torch.finfo(dtype).min
            scale_raw = abs_max / FP8_MAX

            if use_ue8m0:
                # For UE8M0 format, scales must be powers of 2
                scales = torch.pow(2.0, torch.ceil(torch.log2(scale_raw)))
            else:
                scales = scale_raw

            # Expand scales for broadcasting with grouped data
            # Shape: (..., num_groups, 1)
            scales_expanded = scales.unsqueeze(-1)

            # Quantize the grouped data
            x_scaled = x_grouped / scales_expanded
            x_clamped = torch.clamp(x_scaled, FP8_MIN, FP8_MAX)
            x_quantized = x_clamped.to(dtype)

            # Reshape back to original shape
            x_q.copy_(x_quantized.view(original_shape))

            # Prepare scales tensor in requested format
            if column_major_scales:
                # Column-major: (num_groups,) + batch_dims
                # Transpose the scales to put group dimension first
                scales_shape = (num_groups,) + original_shape[:-1]
                x_s = scales.permute(-1, *range(len(original_shape) - 1))
                x_s = x_s.contiguous().view(scales_shape)
            else:
                # Row-major: batch_dims + (num_groups,)
                x_s = scales.contiguous()

            # Ensure scales are float32
            return x_q, x_s.float()

        k_fp8, k_scale = group_quant_torch(
            k,
            group_size=quant_block_size,
            column_major_scales=False,
            use_ue8m0=(scale_fmt == "ue8m0"),
        )

        k_fp8_bytes = k_fp8.view(-1, head_dim).view(torch.uint8)
        scale_bytes = k_scale.view(torch.uint8).view(-1, 4)
        k = torch.cat(
            [k_fp8_bytes, scale_bytes], dim=-1
        )  # [total_tokens, head_dim + 4]

        slot_mapping = slot_mapping.flatten()
        # kv_cache: [num_block, block_size, head_dim + 4]
        kv_cache.view(-1, kv_cache.shape[-1]).index_copy_(0, slot_mapping, k)

    @staticmethod
    def cp_gather_indexer_k_quant_cache(
        kv_cache: torch.Tensor,
        dst_k: torch.Tensor,
        dst_scale: torch.Tensor,
        block_table: torch.Tensor,
        cu_seq_lens: torch.Tensor,
    ) -> None:
        """
        Args:
            kv_cache: [num_blocks, block_size, cache_stride] - quantized KV cache
                    Layout per block: [k_values, scale_values]
                    - k_values: [block_size * head_dim]
                    - scale_values: [block_size * head_dim * 4 / quant_block_size]
            dst_k: [num_tokens, head_dim] - output tensor for K values
            dst_scale: [num_tokens, head_dim / quant_block_size * 4]
                - output tensor for scale values
            block_table: [batch_size, num_blocks] - block table for indexing
            cu_seq_lens: [batch_size + 1] - cumulative sequence lengths
        """
        batch_size = block_table.size(0)
        num_tokens = dst_k.size(0)
        head_dim = dst_k.size(1)
        cache_block_size = kv_cache.size(1)
        quant_block_size = head_dim * 4 // dst_scale.size(1)

        # For each token, find which batch it belongs to using searchsorted
        token_indices = torch.arange(num_tokens, device=dst_k.device) + 1
        # cu_seq_lens is [batch_size + 1], we need to find which interval each
        # token belongs to
        batch_indices = torch.searchsorted(cu_seq_lens, token_indices) - 1
        batch_indices = torch.clamp(batch_indices, 0, batch_size - 1)

        # Calculate the in-batch sequence index for each token
        inbatch_seq_indices = token_indices - cu_seq_lens[batch_indices]

        # Find which block each token belongs to
        block_indices_in_table = inbatch_seq_indices // cache_block_size
        physical_block_indices = block_table[batch_indices, block_indices_in_table]

        # Calculate the offset within each block
        inblock_offsets = (inbatch_seq_indices - 1) % cache_block_size

        # Calculate strides
        block_stride = kv_cache.stride(0)  # stride for each block

        # Flatten kv_cache for easier indexing
        kv_cache_flat = kv_cache.view(-1)

        # Calculate source offset for K values for all tokens (vectorized)
        src_block_offsets = physical_block_indices * block_stride
        src_k_offsets = src_block_offsets + inblock_offsets * head_dim

        # Gather K values using advanced indexing
        # Create indices for all elements we need to gather
        k_indices = src_k_offsets.unsqueeze(1) + torch.arange(
            head_dim, device=dst_k.device
        )
        dst_k[:] = kv_cache_flat[k_indices]

        # Calculate source offset for scale values (vectorized)
        # Scales are stored after all K values for each block
        scale_size = head_dim * 4 // quant_block_size
        src_scale_offsets = src_block_offsets + head_dim + inblock_offsets * scale_size

        # Gather scale values
        scale_indices = src_scale_offsets.unsqueeze(1) + torch.arange(
            scale_size, device=dst_scale.device
        )
        dst_scale[:] = kv_cache_flat[scale_indices]
```
**EN:** `indexer_k_quant_and_cache()` is a fully vectorized Python fallback for a specialized KV-cache layout. It quantizes K values in groups, optionally rounds scales up to powers of two for UE8M0, packs raw FP8 bytes together with scale bytes into a single per-token record, and writes those records into the physical cache according to `slot_mapping`. The paired gather routine reverses the packing via vectorized indexing over block tables and cumulative sequence lengths.
**CN:** `indexer_k_quant_and_cache()` 是一种针对特殊 KV Cache 布局的完全向量化 Python 回退实现。它按组量化 K 值，并可选地把 scale 向上取整为 2 的幂以适配 UE8M0，然后把原始 FP8 字节与 scale 字节打包到单条逐 token 记录里，再依据 `slot_mapping` 写入物理缓存。配套的 gather 例程则通过 block table 和累计序列长度做向量化索引，把这种打包布局反向还原出来。

### Top-k utilities and op registration
```python
    @staticmethod
    def top_k_per_row_prefill(
        logits: torch.Tensor,
        cu_seqlen_ks: torch.Tensor,
        cu_seqlen_ke: torch.Tensor,
        raw_topk_indices: torch.Tensor,
        num_rows: int,
        stride0: int,
        strdide1: int,
        topk_tokens: int,
    ) -> torch.Tensor:
        real_topk = min(topk_tokens, logits.shape[-1])
        topk_indices = logits.topk(real_topk, dim=-1)[1].to(torch.int32)
        topk_indices -= cu_seqlen_ks[:, None]
        mask_lo = topk_indices >= 0
        mask_hi = topk_indices - (cu_seqlen_ke - cu_seqlen_ks)[:, None] < 0
        mask = torch.full_like(
            topk_indices, False, dtype=torch.bool, device=topk_indices.device
        )
        mask = mask_lo & mask_hi
        topk_indices.masked_fill_(~mask, -1)
        raw_topk_indices[: topk_indices.shape[0], : topk_indices.shape[1]] = (
            topk_indices
        )

    @staticmethod
    def top_k_per_row_decode(
        logits: torch.Tensor,
        next_n: int,
        seq_lens: torch.Tensor,
        raw_topk_indices: torch.Tensor,
        num_rows: int,
        stride0: int,
        stride1: int,
        topk_tokens: int,
    ) -> torch.Tensor:
        device = logits.device
        batch_size = seq_lens.size(0)
        # padded query len
        padded_num_tokens = batch_size * next_n
        positions = (
            torch.arange(logits.shape[-1], device=device)
            .unsqueeze(0)
            .expand(batch_size * next_n, -1)
        )
        row_indices = torch.arange(padded_num_tokens, device=device) // next_n
        next_n_offset = torch.arange(padded_num_tokens, device=device) % next_n
        index_end_pos = (seq_lens[row_indices] - next_n + next_n_offset).unsqueeze(1)
        # index_end_pos: [B * N, 1]
        mask = positions <= index_end_pos
        # mask: [B * N, L]
        logits = logits.masked_fill(~mask, float("-inf"))
        real_topk = min(topk_tokens, logits.shape[-1])
        topk_indices = logits.topk(real_topk, dim=-1)[1].to(torch.int32)  # [B * N, K]
        # ensure we don't set indices for the top k
        # that is out of range(masked already)
        # this will happen if context length is shorter than K
        topk_indices[topk_indices > index_end_pos] = -1
        raw_topk_indices[: topk_indices.shape[0], : topk_indices.shape[1]] = (
            topk_indices
        )

    @staticmethod
    def register_ops_once() -> None:
        global _OPS_REGISTERED
        if not _OPS_REGISTERED:
            # register all the custom ops here
            direct_register_custom_op(
                op_name="xpu_ops_deepseek_scaling_rope",
                op_func=_xpu_ops_deepseek_scaling_rope_impl,
                mutates_args=[],
                fake_impl=_xpu_ops_deepseek_scaling_rope_fake,
                dispatch_key=current_platform.dispatch_key,
            )

            direct_register_custom_op(
                op_name="xpu_mxfp8_quantize",
                op_func=_xpu_mxfp8_quantize_impl,
                fake_impl=_xpu_mxfp8_quantize_fake,
            )

            direct_register_custom_op(
                op_name="xpu_mxfp4_quantize",
                op_func=_xpu_mxfp4_quantize_impl,
                fake_impl=_xpu_mxfp4_quantize_fake,
            )

            direct_register_custom_op(
                op_name="gdn_attention_core_xpu",
                op_func=_gdn_attention_core_xpu_impl,
                mutates_args=["core_attn_out", "z"],
                fake_impl=_gdn_attention_core_xpu_fake,
            )

            direct_register_custom_op(
                op_name="xpu_topk_topp_sampler",
                op_func=_topk_topp_sample_impl,
                fake_impl=_topk_topp_sample_fake,
            )

            _OPS_REGISTERED = True


xpu_ops.register_ops_once()
```
**EN:** The last block provides helper top-k selection code for prefill and decode workflows, carefully masking out invalid positions before copying indices into preallocated buffers. `register_ops_once()` then bridges the Python implementations into Torch custom-op symbols exactly once, attaching fake implementations where needed so the rest of vLLM can call XPU ops through the regular `torch.ops.vllm.*` path.
**CN:** 最后这一段先提供了 prefill / decode 工作流的 top-k 辅助逻辑，会在把索引拷贝到预分配缓冲区之前仔细屏蔽掉无效位置。随后 `register_ops_once()` 把这些 Python 实现一次性桥接到 Torch 自定义算子符号上，并在需要时挂上 fake 实现，使 vLLM 其它部分能够通过常规的 `torch.ops.vllm.*` 路径调用 XPU 算子。

## Key Concepts / 关键概念
- EN: This module combines true XPU custom-op registration with Python fallbacks for features that do not yet have dedicated kernels.
- CN: 该模块把真正的 XPU 自定义算子注册与尚无专用内核时的 Python 回退实现结合在了一起。
- EN: Fake implementations are guarded by `hasattr(torch.ops._xpu_C, ...)` so partial XPU builds remain importable.
- CN: fake 实现会通过 `hasattr(torch.ops._xpu_C, ...)` 做保护，因此即使 XPU 构建不完整，模块仍可导入。
- EN: The cache-related helpers show how quantized keys are physically packed in XPU KV-cache layouts, including adjacent scale bytes.
- CN: 与缓存相关的辅助函数展示了量化后的 key 在 XPU KV Cache 布局中的物理打包方式，包括紧邻存放的 scale 字节。

## Dependencies / 依赖关系
- EN: Depends on `torch`, XPU extension namespaces such as `torch.ops._xpu_C`, and helper registration utilities like `direct_register_custom_op`.
- CN: 依赖 `torch`、`torch.ops._xpu_C` 等 XPU 扩展命名空间，以及 `direct_register_custom_op` 之类的注册辅助工具。
- EN: Uses `vllm_xpu_kernels.flash_attn_interface.flash_attn_varlen_func` plus vLLM runtime metadata from `get_forward_context()`.
- CN: 依赖 `vllm_xpu_kernels.flash_attn_interface.flash_attn_varlen_func`，并会使用来自 `get_forward_context()` 的 vLLM 运行时元数据。
