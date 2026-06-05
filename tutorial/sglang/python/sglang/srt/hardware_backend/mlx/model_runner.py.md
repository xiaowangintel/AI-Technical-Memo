# model_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/mlx/model_runner.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements MLX backend support for model execution orchestration inside the SGLang runtime. / 为 SGLang 运行时提供面向 MLX 后端的模型执行编排支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-42: Module setup and shared state / 模块设置与共享状态
```python
"""MLX model runner for Apple Silicon.

Slot allocation and radix-trie prefix matching are handled by the
scheduler (``TokenToKVPoolAllocator`` / ``RadixCache``).  This runner
reads cached KV from ``MlxKVPool``, runs the forward pass, and writes
new KV back.  Each request also keeps a ``ContiguousKVCache`` for
decode-time attention.

The module also exposes a lazy-eval (`*_start` / `*_finalize`) surface
used by the MLX overlap scheduler to pipeline CPU bookkeeping with
GPU execution.  The lazy API is a thin split of the synchronous API:
``*_start`` builds the compute graph without materialising outputs,
``*_finalize`` blocks on the lazy token(s) and commits per-request
state.
"""

import logging
import time
from dataclasses import dataclass

import mlx.core as mx
import psutil
from mlx.utils import tree_flatten
from mlx_lm import load as mlx_lm_load
from mlx_lm.utils import quantize_model as mlx_lm_quantize_model

from sglang.srt.hardware_backend.mlx.kv_cache import (
    BatchedDecodeContext,
# ... omitted for brevity ...
    set_context,
)
from sglang.srt.hardware_backend.mlx.kv_cache.kv_pool import MlxKVPool
from sglang.srt.mem_cache.memory_pool import ReqToTokenPool

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `logging`, `time`, `dataclasses`, `mlx.core`, `psutil`, `mlx.utils`. It also defines symbols such as `logger` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `logging`, `time`, `dataclasses`, `mlx.core`, `psutil`, `mlx.utils`。 同时定义了 `logger` 等符号，供后续逻辑使用。

### Lines 45-60: Class `MlxPendingPrefill` declaration / 类 `MlxPendingPrefill` 声明
```python
@dataclass
class MlxPendingPrefill:
    """Lazy prefill state, finalised after ``mx.eval``/``async_eval``.

    ``cache`` is the per-layer list of ``ContiguousKVCache`` that will
    become ``_req_caches[req_id]`` once the request is committed.  It
    may have been converted from a transient ``PoolBackedCache`` list
    already (so its ``state`` arrays are safe to hand to ``async_eval``).
    """

    lazy_token: mx.array
    cache: list  # list[ContiguousKVCache]
    req_id: str
    full_token_ids: list[int]
    req_pool_idx: int
    synced_offset: int
```
**EN:** This class establishes `MlxPendingPrefill` as a compact data container for the surrounding logic. The main stored fields include `lazy_token`, `cache`, `req_id`, `full_token_ids`, `req_pool_idx`, `synced_offset`.
**CN:** 该类将 `MlxPendingPrefill` 定义为周边逻辑的紧凑的数据容器。 其主要存储字段包括 `lazy_token`, `cache`, `req_id`, `full_token_ids`, `req_pool_idx`, `synced_offset`。

### Lines 63-76: Class `MlxPendingExtend` declaration / 类 `MlxPendingExtend` 声明
```python
@dataclass
class MlxPendingExtend:
    """Lazy chunked-prefill-continuation state for an existing request.

    Mirrors :meth:`MlxModelRunner.extend` split into launch/finalize
    halves.  ``cache`` is the request's existing per-layer cache (not a
    fresh one) so the graph writes extend onto the already-materialised
    prefix.
    """

    lazy_token: mx.array
    req_id: str
    new_token_ids: list[int]
    new_synced_offset: int
```
**EN:** This class establishes `MlxPendingExtend` as a compact data container for the surrounding logic. The main stored fields include `lazy_token`, `req_id`, `new_token_ids`, `new_synced_offset`.
**CN:** 该类将 `MlxPendingExtend` 定义为周边逻辑的紧凑的数据容器。 其主要存储字段包括 `lazy_token`, `req_id`, `new_token_ids`, `new_synced_offset`。

### Lines 79-92: Class `MlxPendingDecode` declaration / 类 `MlxPendingDecode` 声明
```python
@dataclass
class MlxPendingDecode:
    """Lazy decode state, finalised after ``mx.eval``/``async_eval``.

    ``caches`` is a per-request list of per-layer ``ContiguousKVCache``
    references (``caches[req_idx][layer_idx]``).  These are the same
    objects the attention wrapper writes into during the forward pass,
    so :meth:`decode_batch_start_chained` can launch the next step on
    top of the same caches without materialising this step first.
    """

    lazy_tokens: mx.array
    req_ids: list[str]
    caches: list  # list[list[ContiguousKVCache]]
```
**EN:** This class establishes `MlxPendingDecode` as a compact data container for the surrounding logic. The main stored fields include `lazy_tokens`, `req_ids`, `caches`.
**CN:** 该类将 `MlxPendingDecode` 定义为周边逻辑的紧凑的数据容器。 其主要存储字段包括 `lazy_tokens`, `req_ids`, `caches`。

### Lines 95-99: Constants and shared state / 常量与共享状态
```python
_MLX_QUANTIZATION_PRESETS: dict[str, tuple[int, int]] = {
    # name -> (bits, group_size). group_size=64 matches the mlx-community convention.
    "mlx_q4": (4, 64),
    "mlx_q8": (8, 64),
}
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `_MLX_QUANTIZATION_PRESETS`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `_MLX_QUANTIZATION_PRESETS`。

### Lines 102-103: Class `MlxModelRunner` declaration / 类 `MlxModelRunner` 声明
```python
class MlxModelRunner:
    """MLX model runner with radix-cache prefix sharing."""
```
**EN:** This class establishes `MlxModelRunner` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `_extract_logits`, `_acquire_cache`, `_release_cache`, `_eval_with_cache`, `_cache_state_arrays`.
**CN:** 该类将 `MlxModelRunner` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `_extract_logits`, `_acquire_cache`, `_release_cache`, `_eval_with_cache`, `_cache_state_arrays` 等方法。

### Lines 105-150: Method `MlxModelRunner.__init__` / 方法 `MlxModelRunner.__init__`
```python
    def __init__(
        self,
        model_path: str,
        trust_remote_code: bool = False,
        disable_radix_cache: bool = False,
        pool_size: int | None = None,
        mem_fraction_static: float = 0.8,
        quantization: str | None = None,
    ):
        self.model_path = model_path
        self.trust_remote_code = trust_remote_code
        self.model = None
        self.disable_radix_cache = disable_radix_cache
        self._mem_fraction_static = mem_fraction_static
        # Counter used to trigger periodic mx.clear_cache() calls.
        self._decode_step_ct: int = 0
        # On-the-fly quantization preset (e.g. "mlx_q4"). None = no on-load quantization.
        # Pre-quantized HF repos (e.g. mlx-community/Qwen3-0.6B-4bit) load correctly
        # regardless of this setting — mlx_lm.load() detects the config and instantiates
        # QuantizedLinear modules directly.
        self._quantization: str | None = quantization

        self._load_model()

        # Pin MLX allocations to prevent OS paging
        device_info = mx.device_info()
        max_wired = int(device_info.get("max_recommended_working_set_size", 0))
        if max_wired > 0:
# ... omitted for brevity ...
        self._kv_pool: MlxKVPool | None = None
        self._req_to_token_pool: ReqToTokenPool | None = None
        self._req_pool_idx: dict[str, int] = {}
        self._req_synced_offset: dict[str, int] = {}

        self._pool_size = self._compute_pool_size(pool_size)
```
**EN:** This method implements `__init__` on `MlxModelRunner`. It primarily calls `self._load_model`, `mx.device_info`, `int`, `patch_model_attention`, `get_num_layers`, `self._compute_pool_size` to complete its work. State updates are written into `self.model_path`, `self.trust_remote_code`, `self.model`, `self.disable_radix_cache`, `self._mem_fraction_static`, `self._decode_step_ct`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `__init__`。 它主要通过调用 `self._load_model`, `mx.device_info`, `int`, `patch_model_attention`, `get_num_layers`, `self._compute_pool_size` 来完成任务。 状态更新主要写入 `self.model_path`, `self.trust_remote_code`, `self.model`, `self.disable_radix_cache`, `self._mem_fraction_static`, `self._decode_step_ct`。 实现中使用了条件分支。

### Lines 152-157: Method `MlxModelRunner._extract_logits` / 方法 `MlxModelRunner._extract_logits`
```python
    @staticmethod
    def _extract_logits(model_output):
        """Extract logits from model output, handling both tuple and direct returns."""
        if isinstance(model_output, tuple):
            return model_output[0]
        return model_output
```
**EN:** This method implements `_extract_logits` on `MlxModelRunner`. It primarily calls `isinstance` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `_extract_logits`。 它主要通过调用 `isinstance` 来完成任务。 实现中使用了条件分支。

### Lines 159-169: Method `MlxModelRunner._acquire_cache` / 方法 `MlxModelRunner._acquire_cache`
```python
    def _acquire_cache(self) -> list[ContiguousKVCache]:
        """Get a reusable cache list from the pool, or create a new one."""
        if self._cache_pool:
            cache = self._cache_pool.pop()
            for c in cache:
                c.offset = 0
            return cache
        return [
            ContiguousKVCache(max_seq_len=self._max_seq_len)
            for _ in range(self._num_layers)
        ]
```
**EN:** This method implements `_acquire_cache` on `MlxModelRunner`. It primarily calls `self._cache_pool.pop`, `ContiguousKVCache`, `range` to complete its work. State updates are written into `cache`, `c.offset`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `_acquire_cache`。 它主要通过调用 `self._cache_pool.pop`, `ContiguousKVCache`, `range` 来完成任务。 状态更新主要写入 `cache`, `c.offset`。 实现中使用了条件分支、迭代逻辑。

### Lines 171-173: Method `MlxModelRunner._release_cache` / 方法 `MlxModelRunner._release_cache`
```python
    def _release_cache(self, cache: list[ContiguousKVCache]) -> None:
        """Return a cache list to the pool for reuse."""
        self._cache_pool.append(cache)
```
**EN:** This method implements `_release_cache` on `MlxModelRunner`. It primarily calls `self._cache_pool.append` to complete its work.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `_release_cache`。 它主要通过调用 `self._cache_pool.append` 来完成任务。

### Lines 175-180: Method `MlxModelRunner._eval_with_cache` / 方法 `MlxModelRunner._eval_with_cache`
```python
    @staticmethod
    def _eval_with_cache(
        token_result: mx.array, cache: list[ContiguousKVCache | PoolBackedCache]
    ) -> None:
        """Evaluate token result and all cache buffers in one mx.eval call."""
        mx.eval(token_result, *[s for c in cache for s in c.state])
```
**EN:** This method implements `_eval_with_cache` on `MlxModelRunner`. It primarily calls `mx.eval` to complete its work.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `_eval_with_cache`。 它主要通过调用 `mx.eval` 来完成任务。

### Lines 182-195: Method `MlxModelRunner._cache_state_arrays` / 方法 `MlxModelRunner._cache_state_arrays`
```python
    @staticmethod
    def _cache_state_arrays(
        pending_caches: list[list[ContiguousKVCache | PoolBackedCache]],
    ) -> list[mx.array]:
        """Flatten pending decode cache state list into an array list.

        Safe to hand to ``mx.async_eval``.
        """
        return [
            s
            for cache_list in pending_caches
            for cache in cache_list
            for s in cache.state
        ]
```
**EN:** This method implements `_cache_state_arrays` on `MlxModelRunner`.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `_cache_state_arrays`。

### Lines 197-261: Method `MlxModelRunner._load_model` / 方法 `MlxModelRunner._load_model`
```python
    def _load_model(self):
        """Load model using mlx_lm. If ``self._quantization`` requests a preset
        (e.g. ``mlx_q4``), quantize fp16 weights in-place via
        :func:`mlx_lm.utils.quantize_model` after load.
        """
        logger.info(f"Loading MLX model: {self.model_path}")
        start_time = time.time()

        # We need the config dict to pass into quantize_model so it knows tied/embedding
        # layout. return_config=True is cheap and ignored when no quantization is requested.
        loaded = mlx_lm_load(
            self.model_path,
            tokenizer_config={"trust_remote_code": self.trust_remote_code},
            return_config=True,
        )
        self.model, _tokenizer, config = loaded

        if self._quantization in _MLX_QUANTIZATION_PRESETS:
            bits, group_size = _MLX_QUANTIZATION_PRESETS[self._quantization]
            # Skip if the model was already loaded quantized (pre-quantized HF repo);
            # mlx_lm.load detects the config and instantiates QuantizedLinear directly,
            # so applying the preset on top would be redundant.
            if "quantization" in (config or {}):
                logger.info(
                    "MLX model is already quantized by the HF repo; "
                    f"ignoring --quantization={self._quantization}"
                )
            else:
# ... omitted for brevity ...
        # Force-evaluate weights so mx.get_active_memory() reflects
        # actual usage before KV pool sizing.
        mx.eval(self.model.parameters())

        load_time = time.time() - start_time
        logger.info(f"MLX model loaded in {load_time:.2f}s")
```
**EN:** This method implements `_load_model` on `MlxModelRunner`. It primarily calls `logger.info`, `time.time`, `mlx_lm_load`, `mx.eval`, `self.model.parameters`, `sum` to complete its work. State updates are written into `start_time`, `loaded`, `load_time`, `bytes_before`, `q_start`, `bytes_after`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `_load_model`。 它主要通过调用 `logger.info`, `time.time`, `mlx_lm_load`, `mx.eval`, `self.model.parameters`, `sum` 来完成任务。 状态更新主要写入 `start_time`, `loaded`, `load_time`, `bytes_before`, `q_start`, `bytes_after`。 实现中使用了条件分支。

### Lines 263-281: Method `MlxModelRunner._get_attn_config` / 方法 `MlxModelRunner._get_attn_config`
```python
    def _get_attn_config(self) -> tuple[int, int, mx.Dtype]:
        """Return (n_kv_heads, head_dim, dtype) from the model."""
        layer_list, attn_attr = find_attention_layers(self.model)
        if not layer_list:
            raise RuntimeError("Cannot determine attention config: no layers found")
        sample_attn = getattr(layer_list[0], attn_attr)
        if isinstance(sample_attn, MLXAttentionWrapper):
            sample_attn = sample_attn._inner
        n_kv_heads = sample_attn.n_kv_heads
        if hasattr(sample_attn, "head_dim"):
            head_dim = sample_attn.head_dim
        elif hasattr(sample_attn, "k_proj") and hasattr(sample_attn.k_proj, "weight"):
            head_dim = sample_attn.k_proj.weight.shape[0] // n_kv_heads
        else:
            raise RuntimeError("Cannot determine head_dim from attention module")
        dtype = mx.float16
        if hasattr(sample_attn, "k_proj") and hasattr(sample_attn.k_proj, "weight"):
            dtype = sample_attn.k_proj.weight.dtype
        return n_kv_heads, head_dim, dtype
```
**EN:** This method implements `_get_attn_config` on `MlxModelRunner`. It primarily calls `find_attention_layers`, `getattr`, `isinstance`, `hasattr`, `RuntimeError` to complete its work. State updates are written into `sample_attn`, `n_kv_heads`, `dtype`, `head_dim`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `_get_attn_config`。 它主要通过调用 `find_attention_layers`, `getattr`, `isinstance`, `hasattr`, `RuntimeError` 来完成任务。 状态更新主要写入 `sample_attn`, `n_kv_heads`, `dtype`, `head_dim`。 实现中使用了条件分支。

### Lines 283-310: Method `MlxModelRunner._compute_pool_size` / 方法 `MlxModelRunner._compute_pool_size`
```python
    def _compute_pool_size(self, explicit_size: int | None) -> int:
        """Determine pool slot count (auto-size from available memory if needed)."""
        if explicit_size is not None:
            return explicit_size
        n_kv_heads, head_dim, dtype = self._get_attn_config()
        num_layers = self._num_layers
        sys_available = psutil.virtual_memory().available
        mlx_limit = mx.device_info().get(
            "max_recommended_working_set_size",
            mx.device_info().get("memory_size", 0),
        )
        mlx_used = mx.get_active_memory()
        mlx_usable = int(mlx_limit * self._mem_fraction_static)
        kv_budget = min(
            max(mlx_usable - mlx_used, 0),
            int(sys_available * self._mem_fraction_static),
        )
        bytes_per_slot = 2 * num_layers * n_kv_heads * head_dim * dtype.size
        pool_size = max(kv_budget // bytes_per_slot, 256)
        logger.info(
            f"Auto-sized KV pool: "
            f"sys_available={sys_available / (1024**3):.2f} GB, "
            f"mlx_limit={mlx_limit / (1024**3):.1f} GB, "
            f"mlx_used={mlx_used / (1024**3):.2f} GB, "
            f"kv_budget={kv_budget / (1024**3):.2f} GB, "
            f"bytes_per_slot={bytes_per_slot}, pool_size={pool_size}"
        )
        return pool_size
```
**EN:** This method implements `_compute_pool_size` on `MlxModelRunner`. It primarily calls `self._get_attn_config`, `mx.device_info.get`, `mx.get_active_memory`, `int`, `min`, `max` to complete its work. State updates are written into `num_layers`, `sys_available`, `mlx_limit`, `mlx_used`, `mlx_usable`, `kv_budget`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `_compute_pool_size`。 它主要通过调用 `self._get_attn_config`, `mx.device_info.get`, `mx.get_active_memory`, `int`, `min`, `max` 来完成任务。 状态更新主要写入 `num_layers`, `sys_available`, `mlx_limit`, `mlx_used`, `mlx_usable`, `kv_budget`。 实现中使用了条件分支。

### Lines 312-314: Method `MlxModelRunner.pool_size` / 方法 `MlxModelRunner.pool_size`
```python
    @property
    def pool_size(self) -> int:
        return self._pool_size
```
**EN:** This method implements `pool_size` on `MlxModelRunner`.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `pool_size`。

### Lines 316-334: Method `MlxModelRunner.init_kv_pool` / 方法 `MlxModelRunner.init_kv_pool`
```python
    def init_kv_pool(self, req_to_token_pool: ReqToTokenPool) -> None:
        """Create MlxKVPool (+1 for padding slot 0) and wire scheduler pools."""
        self._req_to_token_pool = req_to_token_pool
        if self.disable_radix_cache:
            return
        n_kv_heads, head_dim, dtype = self._get_attn_config()
        # +1 for padding slot 0
        self._kv_pool = MlxKVPool(
            pool_size=self._pool_size + 1,
            num_layers=self._num_layers,
            n_kv_heads=n_kv_heads,
            head_dim=head_dim,
            dtype=dtype,
        )
        logger.info(
            f"KV pool initialized: pool_size={self._pool_size} "
            f"(buffer size {self._pool_size + 1} incl. padding slot 0), "
            f"{self._num_layers} layers, {n_kv_heads} kv_heads, {head_dim} head_dim"
        )
```
**EN:** This method implements `init_kv_pool` on `MlxModelRunner`. It primarily calls `self._get_attn_config`, `MlxKVPool`, `logger.info` to complete its work. State updates are written into `self._req_to_token_pool`, `self._kv_pool`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `init_kv_pool`。 它主要通过调用 `self._get_attn_config`, `MlxKVPool`, `logger.info` 来完成任务。 状态更新主要写入 `self._req_to_token_pool`, `self._kv_pool`。 实现中使用了条件分支。

### Lines 336-355: Method `MlxModelRunner.prefill` / 方法 `MlxModelRunner.prefill`
```python
    def prefill(
        self,
        req_id: str,
        new_token_ids: list[int],
        full_token_ids: list[int],
        prefix_slot_ids: list[int],
        new_slot_ids: list[int],
        req_pool_idx: int,
    ) -> int:
        """Prefill a request.  Returns next_token_id."""
        pending = self.prefill_start(
            req_id=req_id,
            new_token_ids=new_token_ids,
            full_token_ids=full_token_ids,
            prefix_slot_ids=prefix_slot_ids,
            new_slot_ids=new_slot_ids,
            req_pool_idx=req_pool_idx,
        )
        self._eval_with_cache(pending.lazy_token, pending.cache)
        return self.prefill_finalize(pending)
```
**EN:** This method implements `prefill` on `MlxModelRunner`. It primarily calls `self.prefill_start`, `self._eval_with_cache`, `self.prefill_finalize` to complete its work. State updates are written into `pending`.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `prefill`。 它主要通过调用 `self.prefill_start`, `self._eval_with_cache`, `self.prefill_finalize` 来完成任务。 状态更新主要写入 `pending`。

### Lines 357-366: Method `MlxModelRunner.extend` / 方法 `MlxModelRunner.extend`
```python
    def extend(
        self,
        req_id: str,
        new_token_ids: list[int],
        new_slot_ids: list[int],
    ) -> int:
        """Continue prefill for a chunked request.  Returns next_token_id."""
        pending = self.extend_start(req_id, new_token_ids, new_slot_ids)
        self._eval_with_cache(pending.lazy_token, self._req_caches[req_id])
        return self.extend_finalize(pending)
```
**EN:** This method implements `extend` on `MlxModelRunner`. It primarily calls `self.extend_start`, `self._eval_with_cache`, `self.extend_finalize` to complete its work. State updates are written into `pending`.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `extend`。 它主要通过调用 `self.extend_start`, `self._eval_with_cache`, `self.extend_finalize` 来完成任务。 状态更新主要写入 `pending`。

### Lines 368-394: Method `MlxModelRunner._sync_new_kv_to_pool` / 方法 `MlxModelRunner._sync_new_kv_to_pool`
```python
    def _sync_new_kv_to_pool(
        self,
        cache: list[ContiguousKVCache],
        cache_start: int,
        slot_ids: list[int],
    ) -> None:
        """Sync KV from contiguous cache to pool at the given slot IDs."""
        if not slot_ids or self._kv_pool is None:
            return
        num_layers = len(cache)
        end = cache_start + len(slot_ids)
        slot_ids_mx = mx.array(slot_ids, dtype=mx.int32)
        # TODO: Standardize ContiguousKVCache size to avoid transpose
        # Transpose cache (1, n_kv_heads, S, head_dim) → pool (S, n_kv_heads, head_dim)
        k_all = mx.stack(
            [
                cache[i].keys[0, :, cache_start:end, :].transpose(1, 0, 2)
                for i in range(num_layers)
            ]
        )
        v_all = mx.stack(
            [
                cache[i].values[0, :, cache_start:end, :].transpose(1, 0, 2)
                for i in range(num_layers)
            ]
        )
        self._kv_pool.set_kv_all_layers(slot_ids_mx, k_all, v_all)
```
**EN:** This method implements `_sync_new_kv_to_pool` on `MlxModelRunner`. It primarily calls `len`, `mx.array`, `mx.stack`, `self._kv_pool.set_kv_all_layers`, `cache.keys.transpose`, `cache.values.transpose` to complete its work. State updates are written into `num_layers`, `end`, `slot_ids_mx`, `k_all`, `v_all`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `_sync_new_kv_to_pool`。 它主要通过调用 `len`, `mx.array`, `mx.stack`, `self._kv_pool.set_kv_all_layers`, `cache.keys.transpose`, `cache.values.transpose` 来完成任务。 状态更新主要写入 `num_layers`, `end`, `slot_ids_mx`, `k_all`, `v_all`。 实现中使用了条件分支。

### Lines 396-419: Method `MlxModelRunner._sync_decode_kv_to_pool` / 方法 `MlxModelRunner._sync_decode_kv_to_pool`
```python
    def _sync_decode_kv_to_pool(self, req_id: str) -> None:
        """Sync un-flushed decode KV for *req_id* to the shared pool."""
        if self._kv_pool is None or self._req_to_token_pool is None:
            return
        cache = self._req_caches.get(req_id)
        if cache is None:
            return
        current_offset = cache[0].offset
        synced_offset = self._req_synced_offset.get(req_id, 0)
        if current_offset <= synced_offset:
            return
        req_pool_idx = self._req_pool_idx.get(req_id)
        if req_pool_idx is None:
            return
        # Read slot IDs from scheduler's req_to_token_pool
        slot_ids = (
            self._req_to_token_pool.req_to_token[
                req_pool_idx, synced_offset:current_offset
            ]
            .to(dtype=int)
            .tolist()
        )
        self._sync_new_kv_to_pool(cache, synced_offset, slot_ids)
        self._req_synced_offset[req_id] = current_offset
```
**EN:** This method implements `_sync_decode_kv_to_pool` on `MlxModelRunner`. It primarily calls `self._req_caches.get`, `self._req_synced_offset.get`, `self._req_pool_idx.get`, `self._req_to_token_pool.req_to_token.to.tolist`, `self._sync_new_kv_to_pool`, `self._req_to_token_pool.req_to_token.to` to complete its work. State updates are written into `cache`, `current_offset`, `synced_offset`, `req_pool_idx`, `slot_ids`, `self._req_synced_offset`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `_sync_decode_kv_to_pool`。 它主要通过调用 `self._req_caches.get`, `self._req_synced_offset.get`, `self._req_pool_idx.get`, `self._req_to_token_pool.req_to_token.to.tolist`, `self._sync_new_kv_to_pool`, `self._req_to_token_pool.req_to_token.to` 来完成任务。 状态更新主要写入 `cache`, `current_offset`, `synced_offset`, `req_pool_idx`, `slot_ids`, `self._req_synced_offset`。 实现中使用了条件分支。

### Lines 421-426: Method `MlxModelRunner.flush_all_decode_kv` / 方法 `MlxModelRunner.flush_all_decode_kv`
```python
    def flush_all_decode_kv(self) -> None:
        """Sync all active requests' un-flushed decode KV to the pool."""
        if self.disable_radix_cache or self._kv_pool is None:
            return
        for req_id in list(self._req_caches.keys()):
            self._sync_decode_kv_to_pool(req_id)
```
**EN:** This method implements `flush_all_decode_kv` on `MlxModelRunner`. It primarily calls `list`, `self._req_caches.keys`, `self._sync_decode_kv_to_pool` to complete its work. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `flush_all_decode_kv`。 它主要通过调用 `list`, `self._req_caches.keys`, `self._sync_decode_kv_to_pool` 来完成任务。 实现中使用了条件分支、迭代逻辑。

### Lines 428-439: Method `MlxModelRunner.decode_batch` / 方法 `MlxModelRunner.decode_batch`
```python
    def decode_batch(
        self,
        req_ids: list[str],
    ) -> list[int]:
        """Decode one token per request."""
        pending = self.decode_batch_start(req_ids)
        # Evaluate lazy_tokens together with every affected cache buffer so
        # the attention write-then-read ordering is materialised in one
        # kernel submission.
        cache_arrays = self._cache_state_arrays(pending.caches)
        mx.eval(pending.lazy_tokens, *cache_arrays)
        return self.decode_batch_finalize(pending)
```
**EN:** This method implements `decode_batch` on `MlxModelRunner`. It primarily calls `self.decode_batch_start`, `self._cache_state_arrays`, `mx.eval`, `self.decode_batch_finalize` to complete its work. State updates are written into `pending`, `cache_arrays`.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `decode_batch`。 它主要通过调用 `self.decode_batch_start`, `self._cache_state_arrays`, `mx.eval`, `self.decode_batch_finalize` 来完成任务。 状态更新主要写入 `pending`, `cache_arrays`。

### Lines 441-525: Method `MlxModelRunner.prefill_start` / 方法 `MlxModelRunner.prefill_start`
```python
    def prefill_start(
        self,
        req_id: str,
        new_token_ids: list[int],
        full_token_ids: list[int],
        prefix_slot_ids: list[int],
        new_slot_ids: list[int],
        req_pool_idx: int,
    ) -> MlxPendingPrefill:
        """Queue a prefill forward pass without evaluating.

        Returns an :class:`MlxPendingPrefill` containing the lazy
        next-token ``mx.array`` plus everything needed to commit the
        request in :meth:`prefill_finalize`.  The caller drives the GPU
        by handing ``lazy_token`` (and cache state) to ``mx.async_eval``.
        """
        num_layers = self._num_layers
        prefix_len = len(prefix_slot_ids)

        if self.disable_radix_cache:
            cache = self._acquire_cache()
            input_ids = mx.array([new_token_ids], dtype=mx.int32)
            model_output = self.model(input_ids, cache=cache)
            logits = self._extract_logits(model_output)
            lazy_token = mx.argmax(logits[:, -1, :], axis=-1)
            return MlxPendingPrefill(
                lazy_token=lazy_token,
                cache=cache,
# ... omitted for brevity ...
            cache=cache,
            req_id=req_id,
            full_token_ids=list(full_token_ids),
            req_pool_idx=req_pool_idx,
            synced_offset=prefix_len + len(new_slot_ids),
        )
```
**EN:** This method implements `prefill_start` on `MlxModelRunner`. It primarily calls `len`, `mx.array`, `self.model`, `self._extract_logits`, `mx.argmax`, `MlxPendingPrefill` to complete its work. State updates are written into `num_layers`, `prefix_len`, `new_token_count`, `input_ids`, `model_output`, `logits`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `prefill_start`。 它主要通过调用 `len`, `mx.array`, `self.model`, `self._extract_logits`, `mx.argmax`, `MlxPendingPrefill` 来完成任务。 状态更新主要写入 `num_layers`, `prefix_len`, `new_token_count`, `input_ids`, `model_output`, `logits`。 实现中使用了条件分支、迭代逻辑。

### Lines 527-541: Method `MlxModelRunner.prefill_finalize` / 方法 `MlxModelRunner.prefill_finalize`
```python
    def prefill_finalize(self, pending: MlxPendingPrefill) -> int:
        """Materialise a pending prefill and commit per-request state.

        Must be called *after* ``pending.lazy_token`` has been handed to
        ``mx.async_eval`` / ``mx.eval``.  ``.item()`` here is blocking on
        that specific lazy scalar.
        """
        next_token = int(pending.lazy_token.item())
        self._req_token_ids[pending.req_id] = list(pending.full_token_ids) + [
            next_token
        ]
        self._req_caches[pending.req_id] = pending.cache
        self._req_pool_idx[pending.req_id] = pending.req_pool_idx
        self._req_synced_offset[pending.req_id] = pending.synced_offset
        return next_token
```
**EN:** This method implements `prefill_finalize` on `MlxModelRunner`. It primarily calls `int`, `pending.lazy_token.item`, `list` to complete its work. State updates are written into `next_token`, `self._req_token_ids`, `self._req_caches`, `self._req_pool_idx`, `self._req_synced_offset`.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `prefill_finalize`。 它主要通过调用 `int`, `pending.lazy_token.item`, `list` 来完成任务。 状态更新主要写入 `next_token`, `self._req_token_ids`, `self._req_caches`, `self._req_pool_idx`, `self._req_synced_offset`。

### Lines 543-573: Method `MlxModelRunner.extend_start` / 方法 `MlxModelRunner.extend_start`
```python
    def extend_start(
        self,
        req_id: str,
        new_token_ids: list[int],
        new_slot_ids: list[int],
    ) -> MlxPendingExtend:
        """Queue chunked-prefill continuation without evaluating."""
        assert (
            req_id in self._req_caches
        ), f"extend_start called for unknown request {req_id}"

        cache = self._req_caches[req_id]

        input_ids = mx.array([new_token_ids], dtype=mx.int32)
        model_output = self.model(input_ids, cache=cache)
        logits = self._extract_logits(model_output)
        lazy_token = mx.argmax(logits[:, -1, :], axis=-1)

        if not self.disable_radix_cache and new_slot_ids:
            synced = self._req_synced_offset[req_id]
            self._sync_new_kv_to_pool(cache, synced, new_slot_ids)
            new_synced_offset = synced + len(new_slot_ids)
        else:
            new_synced_offset = self._req_synced_offset.get(req_id, 0)

        return MlxPendingExtend(
            lazy_token=lazy_token,
            req_id=req_id,
            new_token_ids=list(new_token_ids),
            new_synced_offset=new_synced_offset,
        )
```
**EN:** This method implements `extend_start` on `MlxModelRunner`. It primarily calls `mx.array`, `self.model`, `self._extract_logits`, `mx.argmax`, `MlxPendingExtend`, `self._sync_new_kv_to_pool` to complete its work. State updates are written into `cache`, `input_ids`, `model_output`, `logits`, `lazy_token`, `synced`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `extend_start`。 它主要通过调用 `mx.array`, `self.model`, `self._extract_logits`, `mx.argmax`, `MlxPendingExtend`, `self._sync_new_kv_to_pool` 来完成任务。 状态更新主要写入 `cache`, `input_ids`, `model_output`, `logits`, `lazy_token`, `synced`。 实现中使用了条件分支。

### Lines 575-586: Method `MlxModelRunner.extend_finalize` / 方法 `MlxModelRunner.extend_finalize`
```python
    def extend_finalize(self, pending: MlxPendingExtend) -> int:
        """Materialise a pending extend and commit per-request state."""
        next_token = int(pending.lazy_token.item())

        prev_tokens = self._req_token_ids[pending.req_id]
        if prev_tokens:
            prev_tokens.pop()  # remove stale intermediate token
        prev_tokens.extend(pending.new_token_ids)
        prev_tokens.append(next_token)

        self._req_synced_offset[pending.req_id] = pending.new_synced_offset
        return next_token
```
**EN:** This method implements `extend_finalize` on `MlxModelRunner`. It primarily calls `int`, `prev_tokens.extend`, `prev_tokens.append`, `pending.lazy_token.item`, `prev_tokens.pop` to complete its work. State updates are written into `next_token`, `prev_tokens`, `self._req_synced_offset`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `extend_finalize`。 它主要通过调用 `int`, `prev_tokens.extend`, `prev_tokens.append`, `pending.lazy_token.item`, `prev_tokens.pop` 来完成任务。 状态更新主要写入 `next_token`, `prev_tokens`, `self._req_synced_offset`。 实现中使用了条件分支。

### Lines 588-639: Method `MlxModelRunner.decode_batch_start` / 方法 `MlxModelRunner.decode_batch_start`
```python
    def decode_batch_start(self, req_ids: list[str]) -> MlxPendingDecode:
        """Queue a decode forward pass without evaluating.

        The caller is responsible for calling ``mx.async_eval`` on the
        returned ``lazy_tokens`` (and optionally per-cache state arrays)
        to kick off GPU work before :meth:`decode_batch_finalize`.
        """
        batch_size = len(req_ids)
        num_layers = self._num_layers

        caches = [self._req_caches[rid] for rid in req_ids]

        if batch_size == 1:
            cache = caches[0]
            last_token = self._req_token_ids[req_ids[0]][-1]
            input_ids = mx.array([[last_token]], dtype=mx.int32)
            model_output = self.model(input_ids, cache=cache)
            logits = self._extract_logits(model_output)
            lazy_tokens = mx.argmax(logits[:, -1, :], axis=-1)
            return MlxPendingDecode(
                lazy_tokens=lazy_tokens,
                req_ids=list(req_ids),
                caches=caches,
            )

        seq_lens = [caches[i][0].offset for i in range(batch_size)]
        layer_caches = [
            [caches[i][layer_idx] for i in range(batch_size)]
# ... omitted for brevity ...

        return MlxPendingDecode(
            lazy_tokens=lazy_tokens,
            req_ids=list(req_ids),
            caches=caches,
        )
```
**EN:** This method implements `decode_batch_start` on `MlxModelRunner`. It primarily calls `len`, `BatchedDecodeContext`, `set_context`, `MlxPendingDecode`, `mx.array`, `self.model` to complete its work. State updates are written into `batch_size`, `num_layers`, `caches`, `seq_lens`, `layer_caches`, `ctx`. The implementation relies on conditional branches, error handling.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `decode_batch_start`。 它主要通过调用 `len`, `BatchedDecodeContext`, `set_context`, `MlxPendingDecode`, `mx.array`, `self.model` 来完成任务。 状态更新主要写入 `batch_size`, `num_layers`, `caches`, `seq_lens`, `layer_caches`, `ctx`。 实现中使用了条件分支、错误处理。

### Lines 641-713: Method `MlxModelRunner.decode_batch_start_chained` / 方法 `MlxModelRunner.decode_batch_start_chained`
```python
    def decode_batch_start_chained(
        self,
        prev: MlxPendingDecode,
    ) -> MlxPendingDecode:
        """Build the next decode step on top of a still-lazy previous decode.

        Feeds ``prev.lazy_tokens`` (an unevaluated ``mx.array`` of shape
        ``(B,)``) as the next step's input ids, reusing
        ``prev.caches`` in-place so that the per-layer ``ContiguousKVCache``
        writes from step N and step N+1 land in the same buffers.  MLX
        tracks the full dependency graph, so once ``mx.async_eval`` is
        called the GPU executes N+1 immediately after N with no gap.

        Caller contract:

        * ``prev`` MUST refer to the same set of requests (same order) as
          the batch the caller intends to run next.  Composition changes
          (finished reqs, new prefills) must break the chain instead.
        * After calling this, finalise ``prev`` BEFORE finalising the
          returned pending: state bookkeeping for step N has to happen
          before step N+1's bookkeeping.
        """
        batch_size = len(prev.req_ids)
        num_layers = self._num_layers
        caches = prev.caches

        # TODO (changminbark): Need to fix ContiguousKVCache.write_token
        # to accommodate dynamic growing like ContiguousKVCache.update_and_fetch.
# ... omitted for brevity ...

        return MlxPendingDecode(
            lazy_tokens=lazy_tokens,
            req_ids=prev.req_ids,
            caches=caches,
        )
```
**EN:** This method implements `decode_batch_start_chained` on `MlxModelRunner`. It primarily calls `len`, `BatchedDecodeContext`, `set_context`, `MlxPendingDecode`, `self.model`, `self._extract_logits` to complete its work. State updates are written into `batch_size`, `num_layers`, `caches`, `seq_lens`, `layer_caches`, `ctx`. The implementation relies on conditional branches, error handling.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `decode_batch_start_chained`。 它主要通过调用 `len`, `BatchedDecodeContext`, `set_context`, `MlxPendingDecode`, `self.model`, `self._extract_logits` 来完成任务。 状态更新主要写入 `batch_size`, `num_layers`, `caches`, `seq_lens`, `layer_caches`, `ctx`。 实现中使用了条件分支、错误处理。

### Lines 715-741: Method `MlxModelRunner.decode_batch_finalize` / 方法 `MlxModelRunner.decode_batch_finalize`
```python
    def decode_batch_finalize(
        self,
        pending: MlxPendingDecode,
    ) -> list[int]:
        """Materialise a pending decode and update per-request token lists.

        ``pending.lazy_tokens.tolist()`` implicitly blocks until that
        specific lazy array (and its graph ancestors, including the
        per-request cache writes for this step) is evaluated.  The
        caller should have previously handed this pending's lazy_tokens
        to ``mx.async_eval`` (or to a subsequent chained step that will
        be async_eval'd).
        """
        raw = pending.lazy_tokens.tolist()
        if not isinstance(raw, list):
            raw = [raw]
        next_tokens = [int(t) for t in raw]

        for i, rid in enumerate(pending.req_ids):
            self._req_token_ids[rid].append(next_tokens[i])

        self._decode_step_ct += 1
        # TODO (changminbark): allow for flag configuration for clearing mx cache
        if self._decode_step_ct % 256 == 0:
            mx.clear_cache()

        return next_tokens
```
**EN:** This method implements `decode_batch_finalize` on `MlxModelRunner`. It primarily calls `pending.lazy_tokens.tolist`, `enumerate`, `isinstance`, `int`, `self._req_token_ids.append`, `mx.clear_cache` to complete its work. State updates are written into `raw`, `next_tokens`, `self._decode_step_ct`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `decode_batch_finalize`。 它主要通过调用 `pending.lazy_tokens.tolist`, `enumerate`, `isinstance`, `int`, `self._req_token_ids.append`, `mx.clear_cache` 来完成任务。 状态更新主要写入 `raw`, `next_tokens`, `self._decode_step_ct`。 实现中使用了条件分支、迭代逻辑。

### Lines 743-745: Method `MlxModelRunner.has_request` / 方法 `MlxModelRunner.has_request`
```python
    def has_request(self, req_id: str) -> bool:
        """Check if a request has active state."""
        return req_id in self._req_caches
```
**EN:** This method implements `has_request` on `MlxModelRunner`.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `has_request`。

### Lines 747-757: Method `MlxModelRunner.remove_request` / 方法 `MlxModelRunner.remove_request`
```python
    def remove_request(self, req_id: str):
        """Sync remaining decode KV to pool, then release request state."""
        if not self.disable_radix_cache:
            self._sync_decode_kv_to_pool(req_id)

        self._req_token_ids.pop(req_id, None)
        cache = self._req_caches.pop(req_id, None)
        if cache is not None:
            self._release_cache(cache)
        self._req_pool_idx.pop(req_id, None)
        self._req_synced_offset.pop(req_id, None)
```
**EN:** This method implements `remove_request` on `MlxModelRunner`. It primarily calls `self._req_token_ids.pop`, `self._req_caches.pop`, `self._req_pool_idx.pop`, `self._req_synced_offset.pop`, `self._sync_decode_kv_to_pool`, `self._release_cache` to complete its work. State updates are written into `cache`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `remove_request`。 它主要通过调用 `self._req_token_ids.pop`, `self._req_caches.pop`, `self._req_pool_idx.pop`, `self._req_synced_offset.pop`, `self._sync_decode_kv_to_pool`, `self._release_cache` 来完成任务。 状态更新主要写入 `cache`。 实现中使用了条件分支。

### Lines 759-768: Method `MlxModelRunner.clear` / 方法 `MlxModelRunner.clear`
```python
    def clear(self):
        """Clear all request states."""
        self._req_token_ids.clear()
        for cache in self._req_caches.values():
            self._cache_pool.append(cache)
        self._req_caches.clear()
        self._req_pool_idx.clear()
        self._req_synced_offset.clear()
        if self._kv_pool is not None:
            self._kv_pool.clear()
```
**EN:** This method implements `clear` on `MlxModelRunner`. It primarily calls `self._req_token_ids.clear`, `self._req_caches.values`, `self._req_caches.clear`, `self._req_pool_idx.clear`, `self._req_synced_offset.clear`, `self._cache_pool.append` to complete its work. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `MlxModelRunner`）实现了 `clear`。 它主要通过调用 `self._req_token_ids.clear`, `self._req_caches.values`, `self._req_caches.clear`, `self._req_pool_idx.clear`, `self._req_synced_offset.clear`, `self._cache_pool.append` 来完成任务。 实现中使用了条件分支、迭代逻辑。

## Key Concepts / 关键概念
- **Classes / 类**: `MlxPendingPrefill`, `MlxPendingExtend`, `MlxPendingDecode`, `MlxModelRunner`
- **Functions / 函数**: `__init__`, `_extract_logits`, `_acquire_cache`, `_release_cache`, `_eval_with_cache`, `_cache_state_arrays`, `_load_model`, `_get_attn_config`
- **Themes / 主题**: `runner`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.hardware_backend.mlx.kv_cache`, `sglang.srt.hardware_backend.mlx.kv_cache.kv_pool`, `sglang.srt.mem_cache.memory_pool`
- **External / 外部依赖**: `mlx.core`, `psutil`, `mlx.utils`, `mlx_lm`, `mlx_lm.utils`
- **Standard library / 标准库**: `logging`, `time`, `dataclasses`
