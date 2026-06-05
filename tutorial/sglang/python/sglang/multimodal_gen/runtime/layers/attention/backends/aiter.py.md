# aiter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/attention/backends/aiter.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `AITerBackend`, `AITerImpl`, and `_can_use_mla_prefill`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `AITerBackend`、`AITerImpl` 和 `_can_use_mla_prefill` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 5-19: module setup and imports / 模块初始化与导入
```python
import logging
import os
from typing import Optional

import aiter
import torch

from sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend import (
    AttentionBackend,
    AttentionImpl,
    AttentionMetadata,
    AttentionMetadataBuilder,
)
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
from sglang.srt.models.deepseek_common.utils import _use_aiter_gfx95
```
**EN:** This block establishes the module context and imports `logging`, `os`, `typing`, `aiter`, `torch`, and `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `logging`、`os`、`typing`、`aiter`、`torch` 和 `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`。这些依赖为后续实现提供所需符号。

### Lines 21-50: supporting statements / 辅助语句
```python
logger = logging.getLogger(__name__)

_use_fp8_attn = os.environ.get("SGLANG_DIFFUSION_AITER_FP8_ATTN", "0") == "1"
_fp8_dtype = torch.float8_e4m3fn

# ── MLA prefill ASM kernel constraints ──────────────────────────────
# The only available FP8 prefill kernel is the pre-compiled ASM binary
# mla_pfl_qh192_vh128_m32x8_n128x1_causal{0,1}.co, originally built for
# DeepSeek-style MLA.  Four hard constraints:
#
# 1. GPU arch must be gfx950 (MI350/MI355).  The ASM binary is compiled
#    exclusively for gfx950; it will crash or fail to load on other archs.
#
# 2. qk_head_dim baked at 192.  Models with smaller head dims (e.g.
#    Wan's 128) are handled by zero-padding Q/K — extra dims contribute
#    0 to dot products, preserving correctness.
#
# 3. v_head_dim baked at 128.  Models with V head dim != 128 cannot use
#    this kernel.
#
# 4. Kernel tiles over heads in groups of 8 ("m32x8" = 32 tokens x 8
#    heads per tile).  num_heads not divisible by 8 causes OOB reads.
#    E.g. Ulysses SP degree=4 with 40 heads -> 10 heads/rank -> crash.
_MLA_PREFILL_QK_HEAD_DIM = 192
_MLA_PREFILL_V_HEAD_DIM = 128
_MLA_PREFILL_HEAD_TILE = 8


if _use_fp8_attn:
    logger.info("DiT FP8 attention enabled via SGLANG_DIFFUSION_AITER_FP8_ATTN=1")
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, `_use_fp8_attn`, `_fp8_dtype`, `_MLA_PREFILL_QK_HEAD_DIM`, `_MLA_PREFILL_V_HEAD_DIM`, and `_MLA_PREFILL_HEAD_TILE`. The code collaborates with `logging.getLogger`, `os.environ.get`, and `logger.info`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger`、`_use_fp8_attn`、`_fp8_dtype`、`_MLA_PREFILL_QK_HEAD_DIM`、`_MLA_PREFILL_V_HEAD_DIM` 和 `_MLA_PREFILL_HEAD_TILE` 等名称。 代码会与 `logging.getLogger`、`os.environ.get` 和 `logger.info` 协同工作。

### Lines 53-59: `_can_use_mla_prefill` implementation / `_can_use_mla_prefill` 实现
```python
def _can_use_mla_prefill(v_head_dim: int, num_heads: int) -> bool:
    """Check if the MLA prefill ASM kernel supports the given shape and GPU."""
    return (
        _use_aiter_gfx95
        and v_head_dim == _MLA_PREFILL_V_HEAD_DIM
        and num_heads % _MLA_PREFILL_HEAD_TILE == 0
    )
```
**EN:** This block defines function `_can_use_mla_prefill`. Check if the MLA prefill ASM kernel supports the given shape and GPU. Parameters such as `v_head_dim`, and `num_heads` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_can_use_mla_prefill`。 它用于处理 can use mla prefill 相关逻辑。 本段逻辑主要由 `v_head_dim` 和 `num_heads` 等参数驱动。

### Lines 62-66: `AITerBackend` class overview / `AITerBackend` 类概览
```python
class AITerBackend(AttentionBackend):
    """
    Backend for AITemplate attention implementation.
    """
```
**EN:** This block defines class `AITerBackend`. Backend for AITemplate attention implementation. It inherits from `AttentionBackend`.
**CN:** 该代码块定义了类 `AITerBackend`。 它用于封装 aiter backend 相关行为。 它继承自 `AttentionBackend`。

### Lines 67-69: `get_enum` implementation / `get_enum` 实现
```python
    @staticmethod
    def get_enum() -> AttentionBackendEnum:
        return AttentionBackendEnum.AITER
```
**EN:** This block defines method `get_enum` on `AITerBackend`. It retrieves enum.
**CN:** 该代码块定义了 `AITerBackend` 的方法 `get_enum`。 它用于获取enum。

### Lines 71-73: `get_impl_cls` implementation / `get_impl_cls` 实现
```python
    @staticmethod
    def get_impl_cls() -> type["AITerImpl"]:
        return AITerImpl
```
**EN:** This block defines method `get_impl_cls` on `AITerBackend`. It retrieves impl cls.
**CN:** 该代码块定义了 `AITerBackend` 的方法 `get_impl_cls`。 它用于获取impl cls。

### Lines 75-78: `get_metadata_cls` implementation / `get_metadata_cls` 实现
```python
    @staticmethod
    def get_metadata_cls() -> type["AttentionMetadata"]:
        # AITer backend does not require special metadata.
        return AttentionMetadata
```
**EN:** This block defines method `get_metadata_cls` on `AITerBackend`. It retrieves metadata cls.
**CN:** 该代码块定义了 `AITerBackend` 的方法 `get_metadata_cls`。 它用于获取metadata cls。

### Lines 80-82: `get_builder_cls` implementation / `get_builder_cls` 实现
```python
    @staticmethod
    def get_builder_cls() -> type["AttentionMetadataBuilder"]:
        raise NotImplementedError("AITer backend does not have a metadata builder.")
```
**EN:** This block defines method `get_builder_cls` on `AITerBackend`. It retrieves builder cls. Key calls include `NotImplementedError`.
**CN:** 该代码块定义了 `AITerBackend` 的方法 `get_builder_cls`。 它用于获取builder cls。 关键调用包括 `NotImplementedError`。

### Lines 85-195: `_build_mla_prefill_metadata` implementation / `_build_mla_prefill_metadata` 实现
```python
def _build_mla_prefill_metadata(
    batch_size: int,
    seq_lens: torch.Tensor,
    num_heads: int,
    num_kv_heads: int,
    is_causal: bool,
    block_size: int = 1,
    tile_q: int = 256,
    tile_kv: int = 128,
    kv_seq_lens: Optional[torch.Tensor] = None,
) -> dict:
    """
    Build persistent-scheduling metadata required by mla_prefill_ps_asm_fwd.

    Args:
        batch_size: number of sequences in the batch.
        seq_lens: [batch_size] int tensor with per-sequence Q lengths (on CPU).
        num_heads: number of query heads.
        num_kv_heads: number of KV heads.
        is_causal: whether causal masking is used.
        block_size: KV page size (1 for non-paged token-level layout).
        tile_q: Q tile size used by the kernel.
        tile_kv: KV tile granularity.
        kv_seq_lens: [batch_size] int tensor with per-sequence KV lengths (on CPU).
            If None, defaults to seq_lens (self-attention).

    Returns:
        dict with all metadata tensors needed by the kernel + reduce.
    """
    if kv_seq_lens is None:
        kv_seq_lens = seq_lens

    device = "cuda"
    gqa_ratio = num_heads // num_kv_heads

    qo_indptr = torch.zeros(batch_size + 1, dtype=torch.int32)
    kv_indptr = torch.zeros(batch_size + 1, dtype=torch.int32)

    qo_indptr[1 : batch_size + 1] = torch.cumsum(seq_lens, dim=0)
    actual_blocks = (kv_seq_lens + block_size - 1) // block_size
    kv_indptr[1 : batch_size + 1] = torch.cumsum(actual_blocks, dim=0)
    num_blocks = int(kv_indptr[-1])

    kv_indices = torch.arange(num_blocks, dtype=torch.int32)

    max_qlen = seq_lens.max()

    qhead_granularity = gqa_ratio
    qlen_granularity = tile_q // qhead_granularity
    kvlen_granularity = max(tile_kv, block_size)

    (
        (work_meta_data_size, work_meta_data_type),
        (work_indptr_size, work_indptr_type),
        (work_info_size, work_info_type),
        (reduce_indptr_size, reduce_indptr_type),
        (reduce_final_map_size, reduce_final_map_type),
        (reduce_partial_map_size, reduce_partial_map_type),
    ) = aiter.get_ps_metadata_info_v1(
        batch_size=batch_size,
        num_head_k=num_kv_heads,
        max_qlen=max_qlen,
        qlen_granularity=qlen_granularity,
    )

    work_metadata_ptrs = torch.empty(
        work_meta_data_size, dtype=work_meta_data_type, device=device
    )
    work_indptr = torch.empty(work_indptr_size, dtype=work_indptr_type, device=device)
    work_info = torch.empty(work_info_size, dtype=work_info_type, device=device)
    reduce_indptr = torch.empty(
        reduce_indptr_size, dtype=reduce_indptr_type, device=device
    )
    reduce_final_map = torch.empty(
        reduce_final_map_size, dtype=reduce_final_map_type, device=device
    )
    reduce_partial_map = torch.empty(
        reduce_partial_map_size, dtype=reduce_partial_map_type, device=device
    )

    aiter.get_ps_metadata_v1(
        qo_indptr.cpu(),
        kv_indptr.cpu(),
        seq_lens.cpu().int(),
        gqa_ratio,
        num_kv_heads,
        work_metadata_ptrs,
        work_indptr,
        work_info,
        reduce_indptr,
        reduce_final_map,
        reduce_partial_map,
        qhead_granularity=qhead_granularity,
        qlen_granularity=qlen_granularity,
        kvlen_granularity=kvlen_granularity,
        block_size=block_size,
        is_causal=is_causal,
    )

    return {
        "qo_indptr": qo_indptr.to(device),
        "kv_indptr": kv_indptr.to(device),
        "kv_indices": kv_indices.to(device),
        "work_indptr": work_indptr,
        "work_info": work_info,
        "reduce_indptr": reduce_indptr,
        "reduce_final_map": reduce_final_map,
        "reduce_partial_map": reduce_partial_map,
        "max_seqlen_q": max_qlen,
        "tile_q": tile_q,
    }
```
**EN:** This block defines function `_build_mla_prefill_metadata`. Build persistent-scheduling metadata required by mla_prefill_ps_asm_fwd. Args: batch_size: number of sequences in the batch. Key calls include `torch.zeros`, `torch.cumsum`, `int`, `torch.arange`, and `seq_lens.max`. The implementation branches on conditions. Parameters such as `batch_size`, `seq_lens`, `num_heads`, `num_kv_heads`, and `is_causal` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_build_mla_prefill_metadata`。 它用于构建mla prefill metadata。 关键调用包括 `torch.zeros`、`torch.cumsum`、`int`、`torch.arange` 和 `seq_lens.max`。 实现中包含条件分支。 本段逻辑主要由 `batch_size`、`seq_lens`、`num_heads`、`num_kv_heads` 和 `is_causal` 等参数驱动。

### Lines 198-301: `_mla_prefill_ps_attention` implementation / `_mla_prefill_ps_attention` 实现
```python
@torch.compiler.disable
def _mla_prefill_ps_attention(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    softmax_scale: float,
    is_causal: bool,
    q_scale: Optional[torch.Tensor] = None,
    k_scale: Optional[torch.Tensor] = None,
    v_scale: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    """
    Run mla_prefill_ps_asm_fwd + mla_reduce_v1 on 4D batch tensors.

    Reshapes [B, S, H, D] -> varlen [B*S, H, D] with trivial indptr,
    calls the kernel, then reshapes back.

    Supports cross-attention where q has seq_len S_q and k/v have seq_len S_kv.

    The ASM kernel has qk_head_dim=192 baked in at compile time (see module-level
    comments).  If the model's head dim is smaller (e.g. 128), Q and K are
    zero-padded along the last dimension to 192 before calling the kernel.
    The padded zeros contribute nothing to the QK dot product, so attention
    scores are identical to the unpadded case.
    """
    B, S_q, H, D_q = q.shape
    S_kv = k.shape[1]
    D_v = v.shape[-1]
    device = q.device
    num_kv_heads = k.shape[2]

    # Zero-pad Q/K head dim to match the kernel's compiled qk_head_dim=192.
    # Padding with zeros preserves dot-product correctness.
    pad_qk = _MLA_PREFILL_QK_HEAD_DIM - D_q
    if pad_qk > 0:
        q = torch.nn.functional.pad(q, (0, pad_qk))
        k = torch.nn.functional.pad(k, (0, pad_qk))
    D_q_kernel = q.shape[-1]

    q_varlen = q.reshape(B * S_q, H, D_q_kernel).contiguous()
    k_varlen = k.reshape(B * S_kv, num_kv_heads, D_q_kernel).contiguous()
    v_varlen = v.reshape(B * S_kv, num_kv_heads, D_v).contiguous()

    q_seq_lens = torch.full((B,), S_q, dtype=torch.int32)
    kv_seq_lens = torch.full((B,), S_kv, dtype=torch.int32)

    meta = _build_mla_prefill_metadata(
        batch_size=B,
        seq_lens=q_seq_lens,
        kv_seq_lens=kv_seq_lens,
        num_heads=H,
        num_kv_heads=num_kv_heads,
        is_causal=is_causal,
        block_size=1,
    )

    total_s = B * S_q
    tile_q = meta["tile_q"]

    output = torch.empty((total_s, H, D_v), dtype=torch.bfloat16, device=device)
    logits = torch.empty(
        (meta["reduce_partial_map"].size(0) * tile_q, H, D_v),
        dtype=torch.float32,
        device=device,
    )
    attn_lse = torch.empty(
        (meta["reduce_partial_map"].size(0) * tile_q, H),
        dtype=torch.float32,
        device=device,
    )
    final_lse = torch.empty((total_s, H), dtype=torch.float32, device=device)

    aiter.mla_prefill_ps_asm_fwd(
        q_varlen,
        k_varlen,
        v_varlen,
        meta["qo_indptr"],
        meta["kv_indptr"],
        meta["kv_indices"],
        meta["work_indptr"],
        meta["work_info"],
        meta["max_seqlen_q"],
        softmax_scale,
        is_causal,
        logits,
        attn_lse,
        output,
        q_scale,
        k_scale,
        v_scale,
    )

    aiter.mla_reduce_v1(
        logits,
        attn_lse,
        meta["reduce_indptr"],
        meta["reduce_final_map"],
        meta["reduce_partial_map"],
        tile_q,
        output,
        final_lse,
    )

    return output.view(B, S_q, H, D_v)
```
**EN:** This block defines function `_mla_prefill_ps_attention`. Run mla_prefill_ps_asm_fwd + mla_reduce_v1 on 4D batch tensors. Reshapes [B, S, H, D] -> varlen [B*S, H, D] with trivial indptr, calls the kernel, then reshapes back. Key calls include `q.reshape.contiguous`, `k.reshape.contiguous`, `v.reshape.contiguous`, `torch.full`, and `_build_mla_prefill_metadata`. The implementation branches on conditions. Parameters such as `q`, `k`, `v`, `softmax_scale`, and `is_causal` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_mla_prefill_ps_attention`。 它用于处理 mla prefill ps attention 相关逻辑。 关键调用包括 `q.reshape.contiguous`、`k.reshape.contiguous`、`v.reshape.contiguous`、`torch.full` 和 `_build_mla_prefill_metadata`。 实现中包含条件分支。 本段逻辑主要由 `q`、`k`、`v`、`softmax_scale` 和 `is_causal` 等参数驱动。

### Lines 304-308: `AITerImpl` class overview / `AITerImpl` 类概览
```python
class AITerImpl(AttentionImpl):
    """
    Implementation of attention using AITemplate.
    """
```
**EN:** This block defines class `AITerImpl`. Implementation of attention using AITemplate. It inherits from `AttentionImpl`.
**CN:** 该代码块定义了类 `AITerImpl`。 它用于封装 aiter impl 相关行为。 它继承自 `AttentionImpl`。

### Lines 309-326: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        softmax_scale: float,
        causal: bool = False,
        num_kv_heads: int | None = None,
        prefix: str = "",
        dropout_p: float = 0.0,
        **extra_impl_args,
    ) -> None:
        if num_kv_heads is not None and num_kv_heads != num_heads:
            raise NotImplementedError(
                "AITer backend does not support Grouped Query Attention yet."
            )
        self.causal = causal
        self.dropout_p = dropout_p
        self.softmax_scale = softmax_scale
```
**EN:** This block defines method `__init__` on `AITerImpl`. It initializes the instance state. Key calls include `NotImplementedError`. The implementation branches on conditions. Parameters such as `num_heads`, `head_size`, `softmax_scale`, `causal`, and `num_kv_heads` drive the behavior in this section.
**CN:** 该代码块定义了 `AITerImpl` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `NotImplementedError`。 实现中包含条件分支。 本段逻辑主要由 `num_heads`、`head_size`、`softmax_scale`、`causal` 和 `num_kv_heads` 等参数驱动。

### Lines 328-392: `forward` implementation / `forward` 实现
```python
    @torch.compiler.disable
    def forward(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        attn_metadata: AttentionMetadata | None = None,
    ) -> torch.Tensor:
        """
        Performs attention using one of:
          - _mla_prefill_ps_attention (FP8, SGLANG_DIFFUSION_AITER_FP8_ATTN=1)
          - flash_attn_func (BF16, default or FP8 fallback for unsupported shapes)

        Args:
            query: Query tensor of shape [batch_size, seq_len, num_heads, head_dim]
            key: Key tensor of shape [batch_size, seq_len, num_heads, head_dim]
            value: Value tensor of shape [batch_size, seq_len, num_heads, head_dim]
            attn_metadata: Metadata for the attention operation (unused).

        Returns:
            Output tensor of shape [batch_size, seq_len, num_heads, head_dim]
        """
        if _use_fp8_attn:
            if query.dtype != _fp8_dtype:
                q_fp8, q_scale = aiter.per_tensor_quant(query, quant_dtype=_fp8_dtype)
                k_fp8, k_scale = aiter.per_tensor_quant(key, quant_dtype=_fp8_dtype)
                v_fp8, v_scale = aiter.per_tensor_quant(value, quant_dtype=_fp8_dtype)
            else:
                q_fp8, k_fp8, v_fp8 = query, key, value
                one = torch.tensor(1.0, dtype=torch.float32, device=query.device)
                q_scale = k_scale = v_scale = one

            if _can_use_mla_prefill(v_fp8.shape[-1], q_fp8.shape[2]):
                return _mla_prefill_ps_attention(
                    q_fp8,
                    k_fp8,
                    v_fp8,
                    softmax_scale=self.softmax_scale,
                    is_causal=self.causal,
                    q_scale=q_scale,
                    k_scale=k_scale,
                    v_scale=v_scale,
                )

            logger.warning_once(
                "FP8 MLA prefill kernel unsupported "
                "(need gfx950, v_head_dim=%d, num_heads divisible by %d; "
                "got v_head_dim=%d, num_heads=%d). Falling back to BF16.",
                _MLA_PREFILL_V_HEAD_DIM,
                _MLA_PREFILL_HEAD_TILE,
                v_fp8.shape[-1],
                q_fp8.shape[2],
            )

        # BF16 path
        output, _ = aiter.flash_attn_func(
            query,
            key,
            value,
            dropout_p=self.dropout_p,
            causal=self.causal,
            return_attn_probs=False,
            return_lse=True,
        )
        return output
```
**EN:** This block defines method `forward` on `AITerImpl`. Performs attention using one of: - _mla_prefill_ps_attention (FP8, SGLANG_DIFFUSION_AITER_FP8_ATTN=1) - flash_attn_func (BF16, default or FP8 fallback for unsupported shapes) Args: query: Query tensor of shape [batch_size, seq_len, num_heads, head_dim] key: Key tensor of shape [batch_size, seq_len, num_heads, head_dim] value: Value tensor of shape [batch_size, seq_len, num_heads, head_dim] attn_metadata: Metadata for the attention operation (unused). Returns: Output tensor of shape [batch_size, seq_len, num_heads, head_dim] Key calls include `aiter.flash_attn_func`, `_can_use_mla_prefill`, `logger.warning_once`, `aiter.per_tensor_quant`, and `torch.tensor`. The implementation branches on conditions. Parameters such as `query`, `key`, `value`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `AITerImpl` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `aiter.flash_attn_func`、`_can_use_mla_prefill`、`logger.warning_once`、`aiter.per_tensor_quant` 和 `torch.tensor`。 实现中包含条件分支。 本段逻辑主要由 `query`、`key`、`value` 和 `attn_metadata` 等参数驱动。

## Key Concepts / 关键概念
- `_can_use_mla_prefill`: Check if the MLA prefill ASM kernel supports the given shape and GPU. / 顶层函数，用于处理 can use mla prefill 相关逻辑。
- `AITerBackend`: Backend for AITemplate attention implementation. / 核心类，用于封装 aiter backend 相关行为。
- `_build_mla_prefill_metadata`: Build persistent-scheduling metadata required by mla_prefill_ps_asm_fwd. / 顶层函数，用于构建mla prefill metadata。
- `_mla_prefill_ps_attention`: Run mla_prefill_ps_asm_fwd + mla_reduce_v1 on 4D batch tensors. / 顶层函数，用于处理 mla prefill ps attention 相关逻辑。
- `AITerImpl`: Implementation of attention using AITemplate. / 核心类，用于封装 aiter impl 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `os`, `typing`
- **Third-party / 第三方依赖**: `aiter`, `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`, `sglang.multimodal_gen.runtime.platforms`, `sglang.srt.models.deepseek_common.utils`

- **Total lines / 总行数**: 392
