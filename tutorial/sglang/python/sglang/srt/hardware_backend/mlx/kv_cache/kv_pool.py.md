# kv_pool.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/mlx/kv_cache/kv_pool.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements MLX backend support for KV-cache management inside the SGLang runtime. / 为 SGLang 运行时提供面向 MLX 后端的KV Cache 管理支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Module setup and shared state / 模块设置与共享状态
```python
"""Flat KV pool with per-layer buffers of shape (pool_size, n_kv_heads, head_dim).

Slot 0 is reserved as padding (1-based indexing).
"""

import logging

import mlx.core as mx

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `logging`, `mlx.core`. It also defines symbols such as `logger` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `logging`, `mlx.core`。 同时定义了 `logger` 等符号，供后续逻辑使用。

### Lines 13-14: Class `MlxKVPool` declaration / 类 `MlxKVPool` 声明
```python
class MlxKVPool:
    """Pre-allocated KV pool indexed by integer slot IDs."""
```
**EN:** This class establishes `MlxKVPool` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `set_kv`, `get_kv`, `get_kv_all_layers`, `set_kv_all_layers`, `all_buffers`.
**CN:** 该类将 `MlxKVPool` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `set_kv`, `get_kv`, `get_kv_all_layers`, `set_kv_all_layers`, `all_buffers` 等方法。

### Lines 16-47: Method `MlxKVPool.__init__` / 方法 `MlxKVPool.__init__`
```python
    def __init__(
        self,
        pool_size: int,
        num_layers: int,
        n_kv_heads: int,
        head_dim: int,
        dtype: mx.Dtype = mx.float16,
    ):
        self.pool_size = pool_size
        self.num_layers = num_layers
        self.n_kv_heads = n_kv_heads
        self.head_dim = head_dim
        self.dtype = dtype

        # Per-layer buffers: (pool_size, n_kv_heads, head_dim)
        self.k_buffer: list[mx.array] = [
            mx.zeros((pool_size, n_kv_heads, head_dim), dtype=dtype)
            for _ in range(num_layers)
        ]
        self.v_buffer: list[mx.array] = [
            mx.zeros((pool_size, n_kv_heads, head_dim), dtype=dtype)
            for _ in range(num_layers)
        ]

        mem_mb = (pool_size * n_kv_heads * head_dim * 2 * num_layers * dtype.size) / (
            1024 * 1024
        )
        logger.info(
            f"MlxKVPool: {pool_size} slots × {num_layers} layers "
            f"× {n_kv_heads} heads × {head_dim} dim, "
            f"dtype={dtype}, ~{mem_mb:.1f} MB"
        )
```
**EN:** This method implements `__init__` on `MlxKVPool`. It primarily calls `logger.info`, `mx.zeros`, `range` to complete its work. State updates are written into `self.pool_size`, `self.num_layers`, `self.n_kv_heads`, `self.head_dim`, `self.dtype`, `self.k_buffer`.
**CN:** 该方法（属于 `MlxKVPool`）实现了 `__init__`。 它主要通过调用 `logger.info`, `mx.zeros`, `range` 来完成任务。 状态更新主要写入 `self.pool_size`, `self.num_layers`, `self.n_kv_heads`, `self.head_dim`, `self.dtype`, `self.k_buffer`。

### Lines 49-52: Method `MlxKVPool.set_kv` / 方法 `MlxKVPool.set_kv`
```python
    def set_kv(self, layer_id: int, slots: mx.array, k: mx.array, v: mx.array) -> None:
        """Scatter K/V into *slots* for one layer."""
        self.k_buffer[layer_id][slots] = k
        self.v_buffer[layer_id][slots] = v
```
**EN:** This method implements `set_kv` on `MlxKVPool`. State updates are written into `self.k_buffer`, `self.v_buffer`.
**CN:** 该方法（属于 `MlxKVPool`）实现了 `set_kv`。 状态更新主要写入 `self.k_buffer`, `self.v_buffer`。

### Lines 54-56: Method `MlxKVPool.get_kv` / 方法 `MlxKVPool.get_kv`
```python
    def get_kv(self, layer_id: int, slots: mx.array) -> tuple[mx.array, mx.array]:
        """Gather K/V from *slots* for one layer."""
        return self.k_buffer[layer_id][slots], self.v_buffer[layer_id][slots]
```
**EN:** This method implements `get_kv` on `MlxKVPool`.
**CN:** 该方法（属于 `MlxKVPool`）实现了 `get_kv`。

### Lines 58-62: Method `MlxKVPool.get_kv_all_layers` / 方法 `MlxKVPool.get_kv_all_layers`
```python
    def get_kv_all_layers(self, slots: mx.array) -> tuple[mx.array, mx.array]:
        """Gather K/V from *slots* across all layers."""
        k_all = mx.stack([self.k_buffer[i][slots] for i in range(self.num_layers)])
        v_all = mx.stack([self.v_buffer[i][slots] for i in range(self.num_layers)])
        return k_all, v_all
```
**EN:** This method implements `get_kv_all_layers` on `MlxKVPool`. It primarily calls `mx.stack`, `range` to complete its work. State updates are written into `k_all`, `v_all`.
**CN:** 该方法（属于 `MlxKVPool`）实现了 `get_kv_all_layers`。 它主要通过调用 `mx.stack`, `range` 来完成任务。 状态更新主要写入 `k_all`, `v_all`。

### Lines 64-69: Method `MlxKVPool.set_kv_all_layers` / 方法 `MlxKVPool.set_kv_all_layers`
```python
    def set_kv_all_layers(
        self, slots: mx.array, k_all: mx.array, v_all: mx.array
    ) -> None:
        """Scatter K/V into *slots* across all layers."""
        for i in range(self.num_layers):
            self.set_kv(i, slots, k_all[i], v_all[i])
```
**EN:** This method implements `set_kv_all_layers` on `MlxKVPool`. It primarily calls `range`, `self.set_kv` to complete its work. The implementation relies on iteration.
**CN:** 该方法（属于 `MlxKVPool`）实现了 `set_kv_all_layers`。 它主要通过调用 `range`, `self.set_kv` 来完成任务。 实现中使用了迭代逻辑。

### Lines 71-73: Method `MlxKVPool.all_buffers` / 方法 `MlxKVPool.all_buffers`
```python
    def all_buffers(self) -> list[mx.array]:
        """Return all buffer arrays (for ``mx.eval``)."""
        return self.k_buffer + self.v_buffer
```
**EN:** This method implements `all_buffers` on `MlxKVPool`.
**CN:** 该方法（属于 `MlxKVPool`）实现了 `all_buffers`。

### Lines 75-80: Method `MlxKVPool.clear` / 方法 `MlxKVPool.clear`
```python
    def clear(self) -> None:
        """Zero all buffers."""
        shape = (self.pool_size, self.n_kv_heads, self.head_dim)
        for i in range(self.num_layers):
            self.k_buffer[i] = mx.zeros(shape, dtype=self.dtype)
            self.v_buffer[i] = mx.zeros(shape, dtype=self.dtype)
```
**EN:** This method implements `clear` on `MlxKVPool`. It primarily calls `range`, `mx.zeros` to complete its work. State updates are written into `shape`, `self.k_buffer`, `self.v_buffer`. The implementation relies on iteration.
**CN:** 该方法（属于 `MlxKVPool`）实现了 `clear`。 它主要通过调用 `range`, `mx.zeros` 来完成任务。 状态更新主要写入 `shape`, `self.k_buffer`, `self.v_buffer`。 实现中使用了迭代逻辑。

## Key Concepts / 关键概念
- **Classes / 类**: `MlxKVPool`
- **Functions / 函数**: `__init__`, `set_kv`, `get_kv`, `get_kv_all_layers`, `set_kv_all_layers`, `all_buffers`, `clear`
- **Themes / 主题**: `kv_cache`, `pool`, `cache`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: None / 无
- **External / 外部依赖**: `mlx.core`
- **Standard library / 标准库**: `logging`
