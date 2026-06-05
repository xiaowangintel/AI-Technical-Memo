# tilelang_kernel.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/nsa/tilelang_kernel.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module wraps low-level tilelang kernel compute routines used by the attention stack. It focuses on tensor layout, launch preparation, and accelerated execution details. / 该模块封装注意力栈使用的底层 tilelang kernel 计算例程，重点处理张量布局、启动准备和加速执行细节。
## Line-by-Line Analysis / 逐行分析
### Lines 1-9: imports
```python
from functools import lru_cache
from typing import Optional, Tuple

import tilelang
import tilelang.language as T
import torch

from sglang.srt.layers.quantization.fp8_kernel import is_fp8_fnuz
from sglang.srt.utils import is_gfx95_supported, is_hip
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 11-11: expr
```python
tilelang.set_log_level("WARNING")
```
**EN:** Implements supporting module logic that does not fit into a named function or class.
**CN:** 实现不属于命名函数或类的辅助模块逻辑。

### Lines 13-16: module constants
```python
pass_configs = {
    tilelang.PassConfigKey.TL_DISABLE_WARP_SPECIALIZED: True,
    tilelang.PassConfigKey.TL_DISABLE_TMA_LOWER: True,
}
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 18-21: conditional branch
```python
if hasattr(tilelang.PassConfigKey, "TL_DISABLE_FAST_MATH"):
    pass_configs[tilelang.PassConfigKey.TL_DISABLE_FAST_MATH] = True
elif hasattr(tilelang.PassConfigKey, "TL_ENABLE_FAST_MATH"):
    pass_configs[tilelang.PassConfigKey.TL_ENABLE_FAST_MATH] = False
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 23-29: module constants
```python
_is_hip = is_hip()
_is_gfx95_supported = is_gfx95_supported()
_is_fp8_fnuz = is_fp8_fnuz()

BF16 = "bfloat16"
FP8 = "float8_e4m3fnuz" if _is_fp8_fnuz else "float8_e4m3"
FP32 = "float32"
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 32-36: function fast_log2_ceil
```python
def fast_log2_ceil(x):
    bits_x = T.reinterpret("uint32", x)
    exp_x = (bits_x >> 23) & 0xFF
    man_bits = bits_x & ((1 << 23) - 1)
    return T.Cast("int32", exp_x - 127 + T.if_then_else(man_bits != 0, 1, 0))
```
**EN:** Implements the fast log2 ceil routine used by this attention module.
**CN:** 实现该注意力模块使用的 fast log2 ceil 例程。

### Lines 39-41: function fast_pow2
```python
def fast_pow2(x):
    bits_x = (x + 127) << 23
    return T.reinterpret("float32", bits_x)
```
**EN:** Implements the fast pow2 routine used by this attention module.
**CN:** 实现该注意力模块使用的 fast pow2 例程。

### Lines 44-45: function fast_round_scale
```python
def fast_round_scale(amax, fp8_max_inv):
    return fast_pow2(fast_log2_ceil(amax * fp8_max_inv))
```
**EN:** Implements the fast round scale routine used by this attention module.
**CN:** 实现该注意力模块使用的 fast round scale 例程。

### Lines 48-62: function _pick_inner_iter
```python
@lru_cache(maxsize=8)
def _pick_inner_iter(seq: int, ni: int, cu: int, block_per_cu: int) -> int:
    """
    Pick the largest valid inner_iter (power-of-two divisor of ni) that keeps
    enough work per CU (seq * ni / inner_iter / cu >= block_per_cu), so we avoid
    under-utilization while minimizing the number of partial groups.
    """

    max_it = int(seq * ni / (cu * block_per_cu))
    it = ni
    while it >= 2:
        if it <= max_it and ni % it == 0:
            return it
        it //= 2
    return 1
```
**EN:** Implements the pick inner iter routine used by this attention module.
**CN:** 实现该注意力模块使用的 pick inner iter 例程。

### Lines 65-113: function act_quant_kernel
```python
@tilelang.jit(pass_configs=pass_configs)
def act_quant_kernel(
    N, in_dtype=BF16, out_dtype=FP8, scale_dtype=FP32, round_scale=False
):
    M = T.symbolic("M")
    fp8_min = -224.0 if _is_fp8_fnuz else -448.0
    fp8_max = 224.0 if _is_fp8_fnuz else 448.0
    fp8_max_inv = 1 / fp8_max
    num_stages = 0 if round_scale else 2
    blk_m = 32
    group_size = 128

    @T.prim_func
    def act_quant_kernel_(
        X: T.Tensor[(M, N), in_dtype],
        Y: T.Tensor[(M, N), out_dtype],
        S: T.Tensor[(M, T.ceildiv(N, group_size)), scale_dtype],
    ):
        with T.Kernel(T.ceildiv(M, blk_m), T.ceildiv(N, group_size), threads=128) as (
            pid_m,
            pid_n,
        ):
            x_shared = T.alloc_shared((blk_m, group_size), in_dtype)
            x_local = T.alloc_fragment((blk_m, group_size), in_dtype)
            amax_local = T.alloc_fragment((blk_m,), scale_dtype)
            s_local = T.alloc_fragment((blk_m,), scale_dtype)
            y_local = T.alloc_fragment((blk_m, group_size), out_dtype)
            y_shared = T.alloc_shared((blk_m, group_size), out_dtype)

            for _ in T.Pipelined(1, num_stages=num_stages):
                T.copy(X[pid_m * blk_m, pid_n * group_size], x_shared)
                T.copy(x_shared, x_local)
# ... omitted 5 lines ...
                    else:
                        s_local[i] = amax_local[i] * fp8_max_inv
                for i, j in T.Parallel(blk_m, group_size):
                    y_local[i, j] = T.clamp(
                        x_local[i, j] / s_local[i], fp8_min, fp8_max
                    )
                for i in T.Parallel(blk_m):
                    S[pid_m * blk_m + i, pid_n] = s_local[i]
                T.copy(y_local, y_shared)
                T.copy(y_shared, Y[pid_m * blk_m, pid_n * group_size])

    return act_quant_kernel_
```
**EN:** Implements the act quant kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 act quant kernel 例程。

### Lines 116-143: function act_quant
```python
def act_quant(
    x: torch.Tensor, block_size: int = 128, scale_fmt: Optional[str] = None
) -> Tuple[torch.Tensor, torch.Tensor]:
    """
    Quantizes the input tensor `x` using block-wise quantization.

    Args:
        x (torch.Tensor): The input tensor to be quantized. Must be contiguous and its last dimension size must be divisible by `block_size`.
        block_size (int, optional): The size of the blocks to be used for quantization. Default is 128.
        scale_fmt (Optional[str], optional): The format of the scale. Default is None.
    Returns:
        Tuple[torch.Tensor, torch.Tensor]: A tuple containing:
            - The quantized tensor with dtype `torch.float8_e4m3fn`.
            - A tensor of scaling factors with dtype `torch.float32`.
    """
    assert x.is_contiguous(), "Input tensor must be contiguous"
    assert (
        x.size(-1) % block_size == 0
    ), f"Last dimension size must be divisible by block_size (block_size={block_size})"
    N = x.size(-1)
    if _is_fp8_fnuz:
        y = torch.empty_like(x, dtype=torch.float8_e4m3fnuz)
    else:
        y = torch.empty_like(x, dtype=torch.float8_e4m3fn)
    s = x.new_empty(*x.size()[:-1], N // block_size, dtype=torch.float32)
    kernel = act_quant_kernel(N, round_scale=scale_fmt is not None)
    kernel(x.view(-1, N), y.view(-1, N), s.view(-1, N // block_size))
    return y, s
```
**EN:** Implements the act quant routine used by this attention module.
**CN:** 实现该注意力模块使用的 act quant 例程。

### Lines 146-200: function fp8_index_kernel
```python
@tilelang.jit(out_idx=[4], pass_configs=pass_configs)
def fp8_index_kernel(h: int, d: int, clear_accum=True):
    b = T.symbolic("b")
    m = T.symbolic("m")
    n = T.symbolic("n")

    blk_n1 = 512
    blk_n2 = 128

    @T.prim_func
    def fp8_index_kernel_(
        q: T.Tensor[(b, m, h, d), FP8],
        q_s: T.Tensor[(b, m, h), FP32],
        k: T.Tensor[(b, n, d), FP8],
        k_s: T.Tensor[(b, n), FP32],
        o: T.Tensor[(b, m, n), FP32],
    ) -> None:
        with T.Kernel(b, m, T.ceildiv(n, blk_n1)) as (i_b, i_m, i1_n):
            q_smem = T.alloc_shared((h, d), FP8)
            T.copy(q[i_b, i_m, 0, 0], q_smem)

            q_s_frag = T.alloc_fragment(h, FP32)
            T.copy(q_s[i_b, i_m, 0], q_s_frag)

            for i2_n in T.Pipelined(blk_n1 // blk_n2, num_stages=2):
                k_smem = T.alloc_shared((blk_n2, d), FP8)
                T.copy(k[i_b, i1_n * blk_n1 + i2_n * blk_n2, 0], k_smem)

                k_s_frag = T.alloc_fragment(blk_n2, FP32)
                T.copy(k_s[i_b, i1_n * blk_n1 + i2_n * blk_n2], k_s_frag)

                logits = T.alloc_fragment((blk_n2, h), FP32)
# ... omitted 11 lines ...
                for i_h, i3_n in T.Parallel(h, blk_n2):
                    logits[i3_n, i_h] = T.max(logits[i3_n, i_h], 0) * q_s_frag[i_h]

                logits_sum = T.alloc_fragment(blk_n2, FP32)
                T.reduce_sum(logits, logits_sum, dim=1)

                for i3_n in T.Parallel(blk_n2):
                    logits_sum[i3_n] *= k_s_frag[i3_n]

                T.copy(logits_sum, o[i_b, i_m, i1_n * blk_n1 + i2_n * blk_n2])

    return fp8_index_kernel_
```
**EN:** Implements the fp8 index kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 fp8 index kernel 例程。

### Lines 203-226: function fp8_index
```python
def fp8_index(
    q: torch.Tensor,
    q_s: torch.Tensor,
    k: torch.Tensor,
    k_s: torch.Tensor,
) -> torch.Tensor:
    """
    Perform index score using FP8 precision.

    Args:
        q (torch.Tensor): The Q tensor, must be contiguous.
        q_s (torch.Tensor): The scaling factor for Q (float), must be contiguous.
        k (torch.Tensor): The K tensor, must be contiguous.
        k_s (torch.Tensor): The scaling factor for K (e8m0 here), must be contiguous.

        fp8 q @ fp8 k -> fp32 logits
        relu(fp32 logits) * q_s (weights) -> fp32 logits
        fp32 logits -> fp32 logits_sum
        fp32 logits_sum * k_s (e8m0) -> fp32 index_score
    """
    if _is_hip:
        return fp8_index_kernel(q.shape[2], q.shape[3], False)(q, q_s, k, k_s)
    else:
        return fp8_index_kernel(q.shape[2], q.shape[3])(q, q_s, k, k_s)
```
**EN:** Implements the fp8 index routine used by this attention module.
**CN:** 实现该注意力模块使用的 fp8 index 例程。

### Lines 229-395: function sparse_attention_fwd_kernel_v1
```python
@tilelang.jit(
    out_idx=[-1],
    pass_configs={
        tilelang.PassConfigKey.TL_DISABLE_TMA_LOWER: True,
        tilelang.PassConfigKey.TL_DISABLE_WARP_SPECIALIZED: True,
    },
)
def sparse_attention_fwd_kernel_v1(
    num_heads,
    dim,
    tail_dim,
    topk,
    *,
    kv_group=1,
    sm_scale=None,
    is_causal=True,
    block_I=64,
    num_stages=2,
    threads=256,
):
    assert dim == tilelang.math.next_power_of_2(
        dim
    ), f"haven't check padding correctness yet, dim={dim}"
    assert tail_dim == tilelang.math.next_power_of_2(
        tail_dim
    ), f"haven't check padding correctness yet, dim={tail_dim}"
    assert is_causal == True, "non-casual is not supported"
    assert (
        topk % block_I == 0
    ), "otherwise will load some index=0 thus causing wrong kv to be loaded"
    if sm_scale is None:
        sm_scale = (1.0 / (dim + tail_dim)) ** 0.5 * 1.44269504  # log2(e)
# ... omitted 123 lines ...
                T.gemm(S_shared, KV_shared, acc_o, policy=T.GemmWarpPolicy.FullCol)

            # Rescale
            for h_i, d_i in T.Parallel(H_per_block, D):
                acc_o[h_i, d_i] /= sumexp[h_i]
            for h_i in T.Parallel(H_per_block):
                sumexp[h_i] = T.log2(sumexp[h_i]) + m_i[h_i] * sm_scale

            T.copy(acc_o, O_shared)
            T.copy(acc_o, Output[b_i, s_i, H0:H1, :])

    return main
```
**EN:** Implements the sparse attention fwd kernel v1 routine used by this attention module.
**CN:** 实现该注意力模块使用的 sparse attention fwd kernel v1 例程。

### Lines 398-791: function sparse_attention_fwd_kernel_v2
```python
@tilelang.jit(
    out_idx=[-1],
    compile_flags=[
        "-O3",
        "-Wno-deprecated-declarations",
        "-U__CUDA_NO_HALF_OPERATORS__",
        "-U__CUDA_NO_HALF_CONVERSIONS__",
        "-U__CUDA_NO_HALF2_OPERATORS__",
        "-U__CUDA_NO_BFLOAT16_CONVERSIONS__",
        "--expt-relaxed-constexpr",
        "--expt-extended-lambda",
        "--ptxas-options=-v,--register-usage-level=10",
        "-DNDEBUG",
    ],
)  # type: ignore
def sparse_attention_fwd_kernel_v2(
    num_heads: int,
    dim: int,
    tail_dim: int,
    topk: int,
    *,
    kv_group: int = 1,
    sm_scale: Optional[float] = None,
    block_I: int = 64,
):
    assert dim == tilelang.math.next_power_of_2(
        dim
    ), f"haven't check padding correctness yet, dim={dim}"
    assert tail_dim == tilelang.math.next_power_of_2(
        tail_dim
    ), f"haven't check padding correctness yet, dim={tail_dim}"
    assert (
# ... omitted 350 lines ...
                                K_tail_shared_1[
                                    r * 16 + (tx - 256) // 8, (tx - 256) % 8 * 8 + v
                                ] = KV[
                                    b_i,
                                    indices_local[0],
                                    g_i,
                                    D + (tx - 256) % 8 * 8 + v,
                                ]

                    T.cp_async_barrier_noinc(bar_k_1_ready[0])

    return main
```
**EN:** Registers sparse attention fwd kernel v2 in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 sparse attention fwd kernel v2 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 794-969: function sparse_mla_fwd_decode_partial
```python
@tilelang.jit(
    out_idx=[-2, -1],
    pass_configs={
        tilelang.PassConfigKey.TL_DISABLE_TMA_LOWER: True,
        tilelang.PassConfigKey.TL_DISABLE_WARP_SPECIALIZED: True,
    },
)
def sparse_mla_fwd_decode_partial(
    heads,
    dim,
    tail_dim,
    topk,
    *,
    kv_group=1,
    sm_scale=None,
    is_causal=True,
    block_I=64,
    inner_iter=1,
    num_stages=1,
    threads=256,
):
    """
    grid: (seq_len * REPLICATE_H, top_k / block_I / inner_iter)
    Each GPU block processes `inner_iter` consecutive KV tiles and writes one (partial_o, partial_lse) entry.
    """

    assert is_causal == True, "non-causal is not supported"
    assert kv_group == 1
    assert topk % block_I == 0
    assert topk % (block_I * inner_iter) == 0, (
        f"topk ({topk}) must be divisible by block_I * inner_iter = "
        f"{block_I} * {inner_iter}"
# ... omitted 132 lines ...
            # sumexp==0 (all masked), use large negative so combine ignores this split
            for h_i in T.Parallel(H_per_block):
                sumexp[h_i] = T.if_then_else(
                    sumexp[h_i] == 0.0,
                    -(2**30),
                    T.log2(sumexp[h_i]) + m_i[h_i] * sm_scale,
                )

            T.copy(acc_o, Partial_O[b_i, s_i, group_i, H0:H1, :])
            T.copy(sumexp, Partial_Lse[b_i, s_i, group_i, H0:H1])

    return main
```
**EN:** Implements the sparse mla fwd decode partial routine used by this attention module.
**CN:** 实现该注意力模块使用的 sparse mla fwd decode partial 例程。

### Lines 972-1055: function sparse_mla_fwd_decode_combine
```python
@tilelang.jit(
    out_idx=[-1],
    pass_configs={
        tilelang.PassConfigKey.TL_DISABLE_TMA_LOWER: True,
        tilelang.PassConfigKey.TL_DISABLE_WARP_SPECIALIZED: True,
    },
)
def sparse_mla_fwd_decode_combine(
    heads,
    dim,
    topk,
    head_per_block,
    *,
    block_I=64,
    threads=256,
):
    """
    grid: (seq_len * REPLICATE_H). batch=1, kv_group=1.
    Each block does one tile of heads (e.g. 4 or 8 for decode).
    """

    assert heads % head_per_block == 0, f"head_per_block must divide heads"

    batch = 1
    seq_len = T.dynamic("seq_len")

    NI = topk // block_I
    H_per_block = head_per_block
    REPLICATE_H = heads // H_per_block

    partial_o_shape = [batch, seq_len, NI, heads, dim]
    partial_lse_shape = [batch, seq_len, NI, heads]
# ... omitted 40 lines ...
                    )

            T.fill(acc_o, 0)
            for k in T.serial(NI):
                for h_i, d_i in T.Parallel(H_per_block, dim):
                    acc_o[h_i, d_i] = acc_o[h_i, d_i] + scale[h_i, k] * Partial_O[
                        b_i, s_i, k, H0 + h_i, d_i
                    ].astype(accum_dtype)

            T.copy(acc_o, Output[b_i, s_i, H0:H1, :])

    return main
```
**EN:** Implements the sparse mla fwd decode combine routine used by this attention module.
**CN:** 实现该注意力模块使用的 sparse mla fwd decode combine 例程。

### Lines 1058-1304: function sparse_mla_fwd_decode_partial_fp8
```python
@tilelang.jit(out_idx=[-2, -1], pass_configs=pass_configs)
def sparse_mla_fwd_decode_partial_fp8(
    num_heads: int,
    d_v: int,
    d_tail: int,
    topk: int,
    *,
    sm_scale=None,
    block_I=64,
    inner_iter=1,
    threads=256,
):
    assert d_v == 512, f"only support d_v=512"
    assert (
        topk % block_I == 0
    ), "otherwise will load some index=0 thus causing wrong kv to be loaded"

    # Softmax scores are in [0, 1]. We scale by fp8_max_val before FP8 cast
    # to better utilize FP8 dynamic range, then apply the inverse scale after GEMM.
    # This is numerically safe because softmax output is bounded by 1.
    fp8_dtype = "float8_e4m3fnuz" if _is_fp8_fnuz else "float8_e4m3fn"
    fp8_max_val = 240.0 if _is_fp8_fnuz else 448.0
    s_inv_scale_const = fp8_max_val
    s_scale_const = 1.0 / fp8_max_val

    BI = block_I
    group_size = 128
    dim_quant_fp8 = d_v + d_tail
    rope_offset_fp8 = d_v
    n_groups = topk // (BI * inner_iter)

    if sm_scale is None:
# ... omitted 203 lines ...
            T.copy(
                acc_o_tile2,
                partial_o[b_i, s_i, group_i, H0:H1, 2 * group_size : 3 * group_size],
            )
            T.copy(
                acc_o_tile3,
                partial_o[b_i, s_i, group_i, H0:H1, 3 * group_size : 4 * group_size],
            )

            T.copy(sumexp, partial_lse[b_i, s_i, group_i, H0:H1])

    return main
```
**EN:** Implements the sparse mla fwd decode partial fp8 routine used by this attention module.
**CN:** 实现该注意力模块使用的 sparse mla fwd decode partial fp8 例程。

### Lines 1307-1377: function tilelang_sparse_fwd
```python
def tilelang_sparse_fwd(
    q: torch.Tensor,
    kv: torch.Tensor,
    indices: torch.Tensor,
    sm_scale: float,
    d_v: int = 512,
) -> torch.Tensor:
    assert q.dim() == 3 and kv.dim() == 3 and indices.dim() == 3
    num_heads = q.shape[1]
    dim = q.shape[2]
    tail_dim = dim - d_v
    topk = indices.shape[-1]
    assert topk == 2048

    if _is_hip:
        is_fp8_kv = kv.dtype in (torch.float8_e4m3fn, torch.float8_e4m3fnuz)
        if is_fp8_kv:
            if q.dtype != kv.dtype:
                q = q.to(kv.dtype)
            if _is_gfx95_supported:
                block_I, threads, block_per_cu, cu = 64, 256, 2, 256
            else:
                block_I, threads, block_per_cu, cu = 64, 256, 1, 304
            ni = topk // block_I
            inner_iter = _pick_inner_iter(q.shape[0], ni, cu, block_per_cu)
            kernel_partial = sparse_mla_fwd_decode_partial_fp8(
                num_heads,
                d_v,
                tail_dim,
                topk,
                sm_scale=sm_scale,
                block_I=block_I,
# ... omitted 27 lines ...
            n_groups * block_I,
            head_per_block=4,
            block_I=block_I,
            threads=threads,
        )
        out = kernel_combine(partial_o_batched, partial_lse_batched)
    else:
        kernel = sparse_attention_fwd_kernel_v2(
            num_heads, d_v, tail_dim, topk, sm_scale=sm_scale
        )
        out = kernel(q.unsqueeze(0), kv.unsqueeze(0), indices.unsqueeze(0))  # type: ignore
    return out
```
**EN:** Implements the tilelang sparse fwd routine used by this attention module.
**CN:** 实现该注意力模块使用的 tilelang sparse fwd 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Paged attention layouts / **CN:** 分页注意力布局
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理
- **EN:** Speculative decoding support / **CN:** 投机解码支持
- **EN:** Position encoding handling / **CN:** 位置编码处理

## Dependencies / 依赖关系
- `functools.lru_cache`
- `typing.Optional`
- `typing.Tuple`
- `tilelang`
- `tilelang.language`
- `torch`
- `sglang.srt.layers.quantization.fp8_kernel.is_fp8_fnuz`
- `sglang.srt.utils.is_gfx95_supported`
- `sglang.srt.utils.is_hip`
