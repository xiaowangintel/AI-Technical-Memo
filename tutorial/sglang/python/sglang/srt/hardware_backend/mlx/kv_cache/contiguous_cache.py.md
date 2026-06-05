# contiguous_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/mlx/kv_cache/contiguous_cache.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements MLX backend support for KV-cache management inside the SGLang runtime. / 为 SGLang 运行时提供面向 MLX 后端的KV Cache 管理支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Module setup and shared state / 模块设置与共享状态
```python
"""ContiguousKVCache, PoolBackedCache and OffsetCache for MLX backend."""

from __future__ import annotations

from typing import TYPE_CHECKING

import mlx.core as mx

if TYPE_CHECKING:
    from sglang.srt.hardware_backend.mlx.kv_cache.kv_pool import MlxKVPool
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `typing`, `mlx.core`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `typing`, `mlx.core`。

### Lines 13-17: Class `OffsetCache` declaration / 类 `OffsetCache` 声明
```python
class OffsetCache:
    """Data-free shim satisfying mlx-lm's cache protocol.

    Provides ``make_mask`` and ``state`` without storing actual K/V.
    """
```
**EN:** This class establishes `OffsetCache` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `state`, `make_mask`, `update_and_fetch`.
**CN:** 该类将 `OffsetCache` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `state`, `make_mask`, `update_and_fetch` 等方法。

### Lines 19-20: Method `OffsetCache.__init__` / 方法 `OffsetCache.__init__`
```python
    def __init__(self, offset: int = 0):
        self.offset = offset
```
**EN:** This method implements `__init__` on `OffsetCache`. State updates are written into `self.offset`.
**CN:** 该方法（属于 `OffsetCache`）实现了 `__init__`。 状态更新主要写入 `self.offset`。

### Lines 22-24: Method `OffsetCache.state` / 方法 `OffsetCache.state`
```python
    @property
    def state(self):
        return ()  # Empty — safe for mx.eval unpacking
```
**EN:** This method implements `state` on `OffsetCache`.
**CN:** 该方法（属于 `OffsetCache`）实现了 `state`。

### Lines 26-27: Method `OffsetCache.make_mask` / 方法 `OffsetCache.make_mask`
```python
    def make_mask(self, N, **kwargs):
        return None if N == 1 else "causal"
```
**EN:** This method implements `make_mask` on `OffsetCache`.
**CN:** 该方法（属于 `OffsetCache`）实现了 `make_mask`。

### Lines 29-30: Method `OffsetCache.update_and_fetch` / 方法 `OffsetCache.update_and_fetch`
```python
    def update_and_fetch(self, keys, values):
        raise RuntimeError("OffsetCache should not store data")
```
**EN:** This method implements `update_and_fetch` on `OffsetCache`. It primarily calls `RuntimeError` to complete its work.
**CN:** 该方法（属于 `OffsetCache`）实现了 `update_and_fetch`。 它主要通过调用 `RuntimeError` 来完成任务。

### Lines 33-33: Constants and shared state / 常量与共享状态
```python
_DEFAULT_MAX_SEQ_LEN = 4096
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `_DEFAULT_MAX_SEQ_LEN`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `_DEFAULT_MAX_SEQ_LEN`。

### Lines 36-43: Class `ContiguousKVCache` declaration / 类 `ContiguousKVCache` 声明
```python
class ContiguousKVCache:
    """Pre-allocated KV buffer for one request × one layer.

    Shape ``(1, n_kv_heads, max_seq_len, head_dim)``.  Slice assignment
    instead of ``mx.concatenate``.  Lazy-allocated on first write.
    """

    __slots__ = ("keys", "values", "offset", "max_seq_len")
```
**EN:** This class establishes `ContiguousKVCache` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `_allocate`, `state`, `make_mask`, `_grow`, `update_and_fetch`.
**CN:** 该类将 `ContiguousKVCache` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `_allocate`, `state`, `make_mask`, `_grow`, `update_and_fetch` 等方法。

### Lines 45-59: Method `ContiguousKVCache.__init__` / 方法 `ContiguousKVCache.__init__`
```python
    def __init__(
        self,
        n_kv_heads: int | None = None,
        head_dim: int | None = None,
        max_seq_len: int = _DEFAULT_MAX_SEQ_LEN,
        dtype: mx.Dtype | None = None,
    ):
        if n_kv_heads is not None and head_dim is not None and dtype is not None:
            self.keys = mx.zeros((1, n_kv_heads, max_seq_len, head_dim), dtype=dtype)
            self.values = mx.zeros((1, n_kv_heads, max_seq_len, head_dim), dtype=dtype)
        else:
            self.keys = None
            self.values = None
        self.offset = 0
        self.max_seq_len = max_seq_len
```
**EN:** This method implements `__init__` on `ContiguousKVCache`. It primarily calls `mx.zeros` to complete its work. State updates are written into `self.offset`, `self.max_seq_len`, `self.keys`, `self.values`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `ContiguousKVCache`）实现了 `__init__`。 它主要通过调用 `mx.zeros` 来完成任务。 状态更新主要写入 `self.offset`, `self.max_seq_len`, `self.keys`, `self.values`。 实现中使用了条件分支。

### Lines 61-69: Method `ContiguousKVCache._allocate` / 方法 `ContiguousKVCache._allocate`
```python
    def _allocate(self, keys: mx.array) -> None:
        """Allocate buffers matching the first key tensor's shape."""
        B, n_kv_heads, _, head_dim = keys.shape
        self.keys = mx.zeros(
            (B, n_kv_heads, self.max_seq_len, head_dim), dtype=keys.dtype
        )
        self.values = mx.zeros(
            (B, n_kv_heads, self.max_seq_len, head_dim), dtype=keys.dtype
        )
```
**EN:** This method implements `_allocate` on `ContiguousKVCache`. It primarily calls `mx.zeros` to complete its work. State updates are written into `self.keys`, `self.values`.
**CN:** 该方法（属于 `ContiguousKVCache`）实现了 `_allocate`。 它主要通过调用 `mx.zeros` 来完成任务。 状态更新主要写入 `self.keys`, `self.values`。

### Lines 71-76: Method `ContiguousKVCache.state` / 方法 `ContiguousKVCache.state`
```python
    @property
    def state(self):
        """Arrays for ``mx.eval`` unpacking."""
        if self.keys is None:
            return ()
        return (self.keys, self.values)
```
**EN:** This method implements `state` on `ContiguousKVCache`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `ContiguousKVCache`）实现了 `state`。 实现中使用了条件分支。

### Lines 78-79: Method `ContiguousKVCache.make_mask` / 方法 `ContiguousKVCache.make_mask`
```python
    def make_mask(self, N, **kwargs):
        return None if N == 1 else "causal"
```
**EN:** This method implements `make_mask` on `ContiguousKVCache`.
**CN:** 该方法（属于 `ContiguousKVCache`）实现了 `make_mask`。

### Lines 81-94: Method `ContiguousKVCache._grow` / 方法 `ContiguousKVCache._grow`
```python
    def _grow(self, required: int) -> None:
        """Double the buffer until it can hold *required* tokens."""
        new_max = self.max_seq_len
        while new_max < required:
            new_max *= 2
        B, n_kv_heads, _, head_dim = self.keys.shape
        new_k = mx.zeros((B, n_kv_heads, new_max, head_dim), dtype=self.keys.dtype)
        new_v = mx.zeros((B, n_kv_heads, new_max, head_dim), dtype=self.values.dtype)
        if self.offset > 0:
            new_k[:, :, : self.offset, :] = self.keys[:, :, : self.offset, :]
            new_v[:, :, : self.offset, :] = self.values[:, :, : self.offset, :]
        self.keys = new_k
        self.values = new_v
        self.max_seq_len = new_max
```
**EN:** This method implements `_grow` on `ContiguousKVCache`. It primarily calls `mx.zeros` to complete its work. State updates are written into `new_max`, `new_k`, `new_v`, `self.keys`, `self.values`, `self.max_seq_len`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `ContiguousKVCache`）实现了 `_grow`。 它主要通过调用 `mx.zeros` 来完成任务。 状态更新主要写入 `new_max`, `new_k`, `new_v`, `self.keys`, `self.values`, `self.max_seq_len`。 实现中使用了条件分支、迭代逻辑。

### Lines 96-109: Method `ContiguousKVCache.update_and_fetch` / 方法 `ContiguousKVCache.update_and_fetch`
```python
    def update_and_fetch(
        self, keys: mx.array, values: mx.array
    ) -> tuple[mx.array, mx.array]:
        """Append K/V and return all valid K/V up to current offset."""
        if self.keys is None:
            self._allocate(keys)
        S = keys.shape[2]
        end = self.offset + S
        if end > self.max_seq_len:
            self._grow(end)
        self.keys[:, :, self.offset : end, :] = keys
        self.values[:, :, self.offset : end, :] = values
        self.offset = end
        return self.keys[:, :, :end, :], self.values[:, :, :end, :]
```
**EN:** This method implements `update_and_fetch` on `ContiguousKVCache`. It primarily calls `self._allocate`, `self._grow` to complete its work. State updates are written into `S`, `end`, `self.keys`, `self.values`, `self.offset`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `ContiguousKVCache`）实现了 `update_and_fetch`。 它主要通过调用 `self._allocate`, `self._grow` 来完成任务。 状态更新主要写入 `S`, `end`, `self.keys`, `self.values`, `self.offset`。 实现中使用了条件分支。

### Lines 111-115: Method `ContiguousKVCache.write_token` / 方法 `ContiguousKVCache.write_token`
```python
    def write_token(self, k: mx.array, v: mx.array) -> None:
        """Write one token. k, v shape: (1, n_kv_heads, 1, head_dim)."""
        self.keys[:, :, self.offset : self.offset + 1, :] = k
        self.values[:, :, self.offset : self.offset + 1, :] = v
        self.offset += 1
```
**EN:** This method implements `write_token` on `ContiguousKVCache`. State updates are written into `self.keys`, `self.values`, `self.offset`.
**CN:** 该方法（属于 `ContiguousKVCache`）实现了 `write_token`。 状态更新主要写入 `self.keys`, `self.values`, `self.offset`。

### Lines 117-119: Method `ContiguousKVCache.get_kv` / 方法 `ContiguousKVCache.get_kv`
```python
    def get_kv(self) -> tuple[mx.array, mx.array]:
        """Return valid K/V: (1, n_kv_heads, offset, head_dim)."""
        return self.keys[:, :, : self.offset, :], self.values[:, :, : self.offset, :]
```
**EN:** This method implements `get_kv` on `ContiguousKVCache`.
**CN:** 该方法（属于 `ContiguousKVCache`）实现了 `get_kv`。

### Lines 122-139: Class `PoolBackedCache` declaration / 类 `PoolBackedCache` 声明
```python
class PoolBackedCache:
    """Lazily gathers cached KV from the shared pool during forward pass.

    Each ``update_and_fetch`` gathers this layer's prefix from the pool
    on demand, keeping operations in the lazy compute graph.  Convert to
    ``ContiguousKVCache`` via ``to_contiguous`` after the forward pass.
    """

    __slots__ = (
        "_pool",
        "_layer_idx",
        "_slots",
        "offset",
        "_full_keys",
        "_full_values",
        "_new_keys",
        "_new_values",
    )
```
**EN:** This class establishes `PoolBackedCache` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `keys`, `values`, `state`, `make_mask`, `update_and_fetch`.
**CN:** 该类将 `PoolBackedCache` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `keys`, `values`, `state`, `make_mask`, `update_and_fetch` 等方法。

### Lines 141-155: Method `PoolBackedCache.__init__` / 方法 `PoolBackedCache.__init__`
```python
    def __init__(
        self,
        pool: MlxKVPool,
        layer_idx: int,
        slots: mx.array,
        prefix_len: int,
    ):
        self._pool = pool
        self._layer_idx = layer_idx
        self._slots = slots
        self.offset = prefix_len
        self._full_keys: mx.array | None = None
        self._full_values: mx.array | None = None
        self._new_keys: mx.array | None = None
        self._new_values: mx.array | None = None
```
**EN:** This method implements `__init__` on `PoolBackedCache`. State updates are written into `self._pool`, `self._layer_idx`, `self._slots`, `self.offset`, `self._full_keys`, `self._full_values`.
**CN:** 该方法（属于 `PoolBackedCache`）实现了 `__init__`。 状态更新主要写入 `self._pool`, `self._layer_idx`, `self._slots`, `self.offset`, `self._full_keys`, `self._full_values`。

### Lines 157-159: Method `PoolBackedCache.keys` / 方法 `PoolBackedCache.keys`
```python
    @property
    def keys(self) -> mx.array | None:
        return self._full_keys
```
**EN:** This method implements `keys` on `PoolBackedCache`.
**CN:** 该方法（属于 `PoolBackedCache`）实现了 `keys`。

### Lines 161-163: Method `PoolBackedCache.values` / 方法 `PoolBackedCache.values`
```python
    @property
    def values(self) -> mx.array | None:
        return self._full_values
```
**EN:** This method implements `values` on `PoolBackedCache`.
**CN:** 该方法（属于 `PoolBackedCache`）实现了 `values`。

### Lines 165-169: Method `PoolBackedCache.state` / 方法 `PoolBackedCache.state`
```python
    @property
    def state(self):
        if self._full_keys is not None:
            return (self._full_keys, self._full_values)
        return ()
```
**EN:** This method implements `state` on `PoolBackedCache`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `PoolBackedCache`）实现了 `state`。 实现中使用了条件分支。

### Lines 171-172: Method `PoolBackedCache.make_mask` / 方法 `PoolBackedCache.make_mask`
```python
    def make_mask(self, N, **kwargs):
        return None if N == 1 else "causal"
```
**EN:** This method implements `make_mask` on `PoolBackedCache`.
**CN:** 该方法（属于 `PoolBackedCache`）实现了 `make_mask`。

### Lines 174-198: Method `PoolBackedCache.update_and_fetch` / 方法 `PoolBackedCache.update_and_fetch`
```python
    def update_and_fetch(
        self, keys: mx.array, values: mx.array
    ) -> tuple[mx.array, mx.array]:
        """Gather cached prefix from pool, concatenate with new K/V."""
        S = keys.shape[2]

        if self.offset > 0:
            k_cached, v_cached = self._pool.get_kv(
                self._layer_idx, self._slots[: self.offset]
            )
            # Pool layout (S, n_kv_heads, head_dim) → cache (1, n_kv_heads, S, head_dim)
            k_cached = k_cached.transpose(1, 0, 2)[None]
            v_cached = v_cached.transpose(1, 0, 2)[None]
            k_all = mx.concatenate([k_cached, keys], axis=2)
            v_all = mx.concatenate([v_cached, values], axis=2)
        else:
            k_all = keys
            v_all = values

        self.offset += S
        self._full_keys = k_all
        self._full_values = v_all
        self._new_keys = keys
        self._new_values = values
        return k_all, v_all
```
**EN:** This method implements `update_and_fetch` on `PoolBackedCache`. It primarily calls `self._pool.get_kv`, `mx.concatenate`, `k_cached.transpose`, `v_cached.transpose` to complete its work. State updates are written into `S`, `self.offset`, `self._full_keys`, `self._full_values`, `self._new_keys`, `self._new_values`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `PoolBackedCache`）实现了 `update_and_fetch`。 它主要通过调用 `self._pool.get_kv`, `mx.concatenate`, `k_cached.transpose`, `v_cached.transpose` 来完成任务。 状态更新主要写入 `S`, `self.offset`, `self._full_keys`, `self._full_values`, `self._new_keys`, `self._new_values`。 实现中使用了条件分支。

### Lines 200-205: Method `PoolBackedCache.to_contiguous` / 方法 `PoolBackedCache.to_contiguous`
```python
    def to_contiguous(self, max_seq_len: int = 4096) -> ContiguousKVCache:
        """Convert to ContiguousKVCache reusing forward-pass arrays."""
        cache = ContiguousKVCache(max_seq_len=max_seq_len)
        if self._full_keys is not None:
            cache.update_and_fetch(self._full_keys, self._full_values)
        return cache
```
**EN:** This method implements `to_contiguous` on `PoolBackedCache`. It primarily calls `ContiguousKVCache`, `cache.update_and_fetch` to complete its work. State updates are written into `cache`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `PoolBackedCache`）实现了 `to_contiguous`。 它主要通过调用 `ContiguousKVCache`, `cache.update_and_fetch` 来完成任务。 状态更新主要写入 `cache`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Classes / 类**: `OffsetCache`, `ContiguousKVCache`, `PoolBackedCache`
- **Functions / 函数**: `__init__`, `state`, `make_mask`, `update_and_fetch`, `__init__`, `_allocate`, `state`, `make_mask`
- **Themes / 主题**: `kv_cache`, `cache`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.hardware_backend.mlx.kv_cache.kv_pool`
- **External / 外部依赖**: `mlx.core`
- **Standard library / 标准库**: `__future__`, `typing`
