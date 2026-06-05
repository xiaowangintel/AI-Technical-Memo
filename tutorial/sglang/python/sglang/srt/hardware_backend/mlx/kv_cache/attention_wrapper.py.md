# attention_wrapper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/mlx/kv_cache/attention_wrapper.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements MLX backend support for attention execution and masking inside the SGLang runtime. / 为 SGLang 运行时提供面向 MLX 后端的注意力执行与掩码处理支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Module setup and shared state / 模块设置与共享状态
```python
"""Batched decode attention wrapper for MLX backend."""

from __future__ import annotations

import threading
from dataclasses import dataclass, field
from typing import Any, Optional

import mlx.core as mx
import mlx.nn as nn

from sglang.srt.hardware_backend.mlx.kv_cache.contiguous_cache import ContiguousKVCache

_thread_local = threading.local()
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `threading`, `dataclasses`, `typing`, `mlx.core`, `mlx.nn`. It also defines symbols such as `_thread_local` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `threading`, `dataclasses`, `typing`, `mlx.core`, `mlx.nn`。 同时定义了 `_thread_local` 等符号，供后续逻辑使用。

### Lines 18-33: Class `BatchedDecodeContext` declaration / 类 `BatchedDecodeContext` 声明
```python
@dataclass
class BatchedDecodeContext:
    """Context set before batched decode, read by attention wrappers."""

    batch_size: int
    seq_lens: list[int]  # per-request token count before the new token
    # layer_caches[layer_idx][req_idx] = ContiguousKVCache
    layer_caches: list[list[ContiguousKVCache]]

    # Derived tensors/metadata, shared across all layers in one forward pass.
    offsets: mx.array = field(init=False)
    max_len: int = field(init=False)
    valid_lens: mx.array = field(init=False)
    needs_padding: bool = field(init=False)
    pad_sizes: list[int] = field(init=False)
    positions: Optional[mx.array] = field(init=False)
```
**EN:** This class establishes `BatchedDecodeContext` as a compact data container for the surrounding logic. Its core interface includes methods such as `__post_init__`.
**CN:** 该类将 `BatchedDecodeContext` 定义为周边逻辑的紧凑的数据容器。 其核心接口包括 `__post_init__` 等方法。

### Lines 35-43: Method `BatchedDecodeContext.__post_init__` / 方法 `BatchedDecodeContext.__post_init__`
```python
    def __post_init__(self) -> None:
        seq_lens = self.seq_lens
        max_seq_len = max(seq_lens)
        self.offsets = mx.array(seq_lens, dtype=mx.int32)
        self.max_len = max_seq_len + 1
        self.valid_lens = self.offsets + 1
        self.needs_padding = min(seq_lens) < max_seq_len
        self.pad_sizes = [max_seq_len - s for s in seq_lens]
        self.positions = mx.arange(self.max_len) if self.needs_padding else None
```
**EN:** This method implements `__post_init__` on `BatchedDecodeContext`. It primarily calls `max`, `mx.array`, `min`, `mx.arange` to complete its work. State updates are written into `seq_lens`, `max_seq_len`, `self.offsets`, `self.max_len`, `self.valid_lens`, `self.needs_padding`.
**CN:** 该方法（属于 `BatchedDecodeContext`）实现了 `__post_init__`。 它主要通过调用 `max`, `mx.array`, `min`, `mx.arange` 来完成任务。 状态更新主要写入 `seq_lens`, `max_seq_len`, `self.offsets`, `self.max_len`, `self.valid_lens`, `self.needs_padding`。

### Lines 46-47: Function `set_context` / 函数 `set_context`
```python
def set_context(ctx: Optional[BatchedDecodeContext]) -> None:
    _thread_local.batched_ctx = ctx
```
**EN:** This function implements `set_context`. State updates are written into `_thread_local.batched_ctx`.
**CN:** 该函数实现了 `set_context`。 状态更新主要写入 `_thread_local.batched_ctx`。

### Lines 50-51: Function `get_context` / 函数 `get_context`
```python
def get_context() -> Optional[BatchedDecodeContext]:
    return getattr(_thread_local, "batched_ctx", None)
```
**EN:** This function implements `get_context`. It primarily calls `getattr` to complete its work.
**CN:** 该函数实现了 `get_context`。 它主要通过调用 `getattr` 来完成任务。

### Lines 54-55: Function `clear_context` / 函数 `clear_context`
```python
def clear_context() -> None:
    _thread_local.batched_ctx = None
```
**EN:** This function implements `clear_context`. State updates are written into `_thread_local.batched_ctx`.
**CN:** 该函数实现了 `clear_context`。 状态更新主要写入 `_thread_local.batched_ctx`。

### Lines 58-63: Class `MLXAttentionWrapper` declaration / 类 `MLXAttentionWrapper` 声明
```python
class MLXAttentionWrapper(nn.Module):
    """Wraps an mlx-lm Attention for batched decode (BS>1).

    When ``BatchedDecodeContext`` is set, performs per-request RoPE,
    cache writes, and batched SDPA.  Otherwise delegates to inner module.
    """
```
**EN:** This class establishes `MLXAttentionWrapper` as the main container/coordinator for the surrounding logic. It inherits from `nn.Module`. Its core interface includes methods such as `__init__`, `__call__`, `_batched_decode`.
**CN:** 该类将 `MLXAttentionWrapper` 定义为周边逻辑的主要封装体或协调者。 它继承自 `nn.Module`。 其核心接口包括 `__init__`, `__call__`, `_batched_decode` 等方法。

### Lines 65-68: Method `MLXAttentionWrapper.__init__` / 方法 `MLXAttentionWrapper.__init__`
```python
    def __init__(self, inner: nn.Module, layer_idx: int):
        super().__init__()
        object.__setattr__(self, "_inner", inner)
        object.__setattr__(self, "_layer_idx", layer_idx)
```
**EN:** This method implements `__init__` on `MLXAttentionWrapper`. It primarily calls `super.__init__`, `object.__setattr__`, `super` to complete its work.
**CN:** 该方法（属于 `MLXAttentionWrapper`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `object.__setattr__`, `super` 来完成任务。

### Lines 70-74: Method `MLXAttentionWrapper.__call__` / 方法 `MLXAttentionWrapper.__call__`
```python
    def __call__(self, x: mx.array, mask: Any = None, cache: Any = None) -> mx.array:
        ctx = get_context()
        if ctx is None:
            return self._inner(x, mask=mask, cache=cache)
        return self._batched_decode(x, ctx)
```
**EN:** This method implements `__call__` on `MLXAttentionWrapper`. It primarily calls `get_context`, `self._batched_decode`, `self._inner` to complete its work. State updates are written into `ctx`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MLXAttentionWrapper`）实现了 `__call__`。 它主要通过调用 `get_context`, `self._batched_decode`, `self._inner` 来完成任务。 状态更新主要写入 `ctx`。 实现中使用了条件分支。

### Lines 76-150: Method `MLXAttentionWrapper._batched_decode` / 方法 `MLXAttentionWrapper._batched_decode`
```python
    def _batched_decode(self, x: mx.array, ctx: BatchedDecodeContext) -> mx.array:
        inner = self._inner
        layer_idx = self._layer_idx
        B = ctx.batch_size

        queries = inner.q_proj(x)
        keys = inner.k_proj(x)
        values = inner.v_proj(x)

        head_dim = queries.shape[-1] // inner.n_heads
        queries = queries.reshape(B, 1, inner.n_heads, head_dim)
        keys = keys.reshape(B, 1, inner.n_kv_heads, head_dim)
        values = values.reshape(B, 1, inner.n_kv_heads, head_dim)

        if hasattr(inner, "q_norm"):
            queries = inner.q_norm(queries)
        if hasattr(inner, "k_norm"):
            keys = inner.k_norm(keys)

        queries = queries.transpose(0, 2, 1, 3)
        keys = keys.transpose(0, 2, 1, 3)
        values = values.transpose(0, 2, 1, 3)

        # Vectorized RoPE with per-batch offsets
        offsets = ctx.offsets
        queries = inner.rope(queries, offset=offsets)
        keys = inner.rope(keys, offset=offsets)

# ... omitted for brevity ...
        output = mx.fast.scaled_dot_product_attention(
            queries, keys_b, values_b, scale=inner.scale, mask=attn_mask
        )

        output = output.transpose(0, 2, 1, 3).reshape(B, 1, -1)
        return inner.o_proj(output)
```
**EN:** This method implements `_batched_decode` on `MLXAttentionWrapper`. It primarily calls `inner.q_proj`, `inner.k_proj`, `inner.v_proj`, `queries.reshape`, `keys.reshape`, `values.reshape` to complete its work. State updates are written into `inner`, `layer_idx`, `B`, `queries`, `keys`, `values`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `MLXAttentionWrapper`）实现了 `_batched_decode`。 它主要通过调用 `inner.q_proj`, `inner.k_proj`, `inner.v_proj`, `queries.reshape`, `keys.reshape`, `values.reshape` 来完成任务。 状态更新主要写入 `inner`, `layer_idx`, `B`, `queries`, `keys`, `values`。 实现中使用了条件分支、迭代逻辑。

## Key Concepts / 关键概念
- **Classes / 类**: `BatchedDecodeContext`, `MLXAttentionWrapper`
- **Functions / 函数**: `set_context`, `get_context`, `clear_context`, `__post_init__`, `__init__`, `__call__`, `_batched_decode`
- **Themes / 主题**: `attention`, `kv_cache`, `cache`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.hardware_backend.mlx.kv_cache.contiguous_cache`
- **External / 外部依赖**: `mlx.core`, `mlx.nn`
- **Standard library / 标准库**: `__future__`, `threading`, `dataclasses`, `typing`
