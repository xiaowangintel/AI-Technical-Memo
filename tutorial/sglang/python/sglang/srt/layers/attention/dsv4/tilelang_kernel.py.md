# tilelang_kernel.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/dsv4/tilelang_kernel.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module wraps low-level tilelang kernel compute routines used by the attention stack. It focuses on tensor layout, launch preparation, and accelerated execution details. / 该模块封装注意力栈使用的底层 tilelang kernel 计算例程，重点处理张量布局、启动准备和加速执行细节。
## Line-by-Line Analysis / 逐行分析
### Lines 1-8: imports
```python
import functools
from typing import Any

import tilelang
import tilelang.language as T
import torch

from sglang.srt.utils import is_hip
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 10-15: conditional branch
```python
if is_hip():
    FP8 = "float8_e5m2fnuz"
    FP8_ = torch.float8_e5m2
else:
    FP8 = "float8_e4m3"
    FP8_ = torch.float8_e4m3fn
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 16-17: module constants
```python
FP32 = "float32"
INT32 = "int32"
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 20-85: function fp8_paged_mqa_logits_kernel
```python
@functools.cache
def fp8_paged_mqa_logits_kernel(
    head_dim: int = 128,
    num_heads: int = 64,
    block_size: int = 64,
    clear_accum: bool = True,
) -> Any:
    N = T.symbolic("batch_size")
    L = T.symbolic("max_table_length")
    S = T.symbolic("max_seq_len")
    C = T.symbolic("num_blocks")
    B = block_size
    D = head_dim
    H = num_heads
    d_0, d_1 = T.dynamic("d_0, d_1")

    assert D % 4 == 0
    assert H % 4 == 0
    assert D == 128

    @tilelang.jit
    def fp8_paged_mqa_logits(
        q: T.Tensor[(N, H, D), FP8],
        kvcache: T.StridedTensor[(C, B, D), (d_0, D, 1), FP8],
        kvcache_scale: T.StridedTensor[(C, B), (d_1, 1), FP32],
        weight: T.Tensor[(N, H), FP32],
        seq_lens: T.Tensor[(N,), INT32],
        page_table: T.Tensor[(N, L), INT32],
        o: T.Tensor[(N, S), FP32],
    ) -> None:
        _ = N, L, S, C, D, H, B, d_0, d_1
        with T.Kernel(N) as bx:
# ... omitted 22 lines ...
                    clear_accum=clear_accum,
                )

                for h, j in T.Parallel(H, B):
                    logits[j, h] = T.max(logits[j, h], 0.0) * q_s_frag[h]
                logits_sum = T.alloc_fragment((B,), FP32)
                T.reduce_sum(logits, logits_sum, dim=1)
                for j in T.Parallel(B):
                    logits_sum[j] *= k_s_frag[j]
                T.copy(logits_sum, o[bx, i * B])

    return fp8_paged_mqa_logits
```
**EN:** Implements the fp8 paged mqa logits kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 fp8 paged mqa logits kernel 例程。

### Lines 88-123: function tilelang_fp8_paged_mqa_logits
```python
def tilelang_fp8_paged_mqa_logits(
    q_fp8: torch.Tensor,
    kvcache_fp8: torch.Tensor,
    weight: torch.Tensor,
    seq_lens: torch.Tensor,
    page_table: torch.Tensor,
    deep_gemm_metadata: Any,
    max_seq_len: int,
    clean_logits: bool = True,
) -> torch.Tensor:
    _ = deep_gemm_metadata
    batch_size, _, num_heads, head_dim = q_fp8.shape
    block_size = kvcache_fp8.shape[1]
    assert head_dim == 128, "TODO"
    assert block_size == 64, "TODO"
    assert q_fp8.shape == (batch_size, 1, num_heads, head_dim)
    assert kvcache_fp8.shape[1:] == (block_size, 1, head_dim + 4)
    assert weight.shape == (batch_size, num_heads)
    assert seq_lens.shape == (batch_size,)
    assert page_table.shape[0] == batch_size
    assert clean_logits == False

    logits = page_table.new_empty((batch_size, max_seq_len), dtype=torch.float32)
    kernel = fp8_paged_mqa_logits_kernel(
        head_dim=head_dim,
        num_heads=num_heads,
        block_size=block_size,
        clear_accum=clean_logits,
    )
    q_fp8 = q_fp8.view(batch_size, num_heads, head_dim)
    kvcache_fp8 = kvcache_fp8.view(-1, block_size * (head_dim + 4))
    kvcache = kvcache_fp8[..., : block_size * head_dim].view(dtype=FP8_)
    kvcache = kvcache.view(-1, block_size, head_dim)
    kvcache_scale = kvcache_fp8[..., block_size * head_dim :].view(dtype=torch.float32)
    kernel(q_fp8, kvcache, kvcache_scale, weight, seq_lens, page_table, logits)
    return logits
```
**EN:** Implements the tilelang fp8 paged mqa logits routine used by this attention module.
**CN:** 实现该注意力模块使用的 tilelang fp8 paged mqa logits 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** Paged attention layouts / **CN:** 分页注意力布局
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理

## Dependencies / 依赖关系
- `functools`
- `typing.Any`
- `tilelang`
- `tilelang.language`
- `torch`
- `sglang.srt.utils.is_hip`
