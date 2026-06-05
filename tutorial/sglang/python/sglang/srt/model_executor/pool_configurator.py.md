# pool_configurator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/model_executor/pool_configurator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the model execution and graph runners part of the SRT runtime and implements logic centered on `pool_configurator`. The module docstring frames it as: "Memory pool configurators for profiling and sizing KV cache pools." / 该模块属于 SRT 运行时的模型执行与图运行器部分，主要实现围绕 `pool_configurator` 的逻辑。 它对外提供的主要入口包括 `MemoryPoolConfig`, `MemoryPoolConfigurator`, `DefaultPoolConfigurator`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: Module imports, constants, and setup
```python
"""Memory pool configurators for profiling and sizing KV cache pools.

Each model architecture has its own configurator that computes pool sizes
from available GPU memory using a unified coeff+bias model:

    available_bytes = max_tokens * coeff + bias
    max_tokens = (available_bytes - bias) / coeff

Two entry points, same core computation:
- calculate_pool_sizes(available_bytes, page_size): profiling path
- calculate_pool_sizes_from_max_tokens(max_tokens, page_size): constraint path
"""

from __future__ import annotations

import logging
from dataclasses import dataclass
from typing import TYPE_CHECKING, Optional

import torch

from sglang.srt.configs.model_config import (
    get_nsa_index_head_dim,
    is_deepseek_nsa,
    is_deepseek_v4,
)
from sglang.srt.environ import envs
from sglang.srt.layers.dp_attention import get_attention_tp_size
from sglang.srt.mem_cache.deepseek_v4_memory_pool import get_compress_state_ring_size
from sglang.srt.mem_cache.memory_pool import NSATokenToKVPool
from sglang.srt.utils.common import is_float4_e2m1fn_x2


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; reads environment-driven configuration.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；读取环境变量驱动的配置。

### Lines 34-50: Class MemoryPoolConfig
```python
@dataclass
class MemoryPoolConfig:
    """Resolved memory pool config, shared between target and draft workers."""

    max_total_num_tokens: int
    max_running_requests: Optional[int] = None
    full_max_total_num_tokens: Optional[int] = None
    swa_max_total_num_tokens: Optional[int] = None

    # DSV4 compressed-attention pool sizes (target only; draft workers leave at 0).
    c4_max_total_num_tokens: int = 0
    c128_max_total_num_tokens: int = 0
    c4_state_pool_size: int = 0
    c128_state_pool_size: int = 0

    mem_fraction_static: Optional[float] = None

```
**EN:** This range introduces `MemoryPoolConfig` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Resolved memory pool config, shared between target and draft workers."
**CN:** 这一段引入 `MemoryPoolConfig`，并定义其后续方法依赖的结构或元数据。

### Lines 51-56: Method MemoryPoolConfig.__post_init__
```python
    def __post_init__(self):
        if self.max_total_num_tokens <= 0:
            msg = "Not enough memory. Please try to increase --mem-fraction-static."
            if self.mem_fraction_static is not None:
                msg += f" Current value: mem_fraction_static={self.mem_fraction_static}"
            raise RuntimeError(msg)
```
**EN:** This callable implements `MemoryPoolConfig.__post_init__` and mainly implements post init. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `MemoryPoolConfig.__post_init__`，主要用于实现 post init 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 57-64: Module imports, constants, and setup
```python


if TYPE_CHECKING:
    from sglang.srt.model_executor.model_runner import ModelRunner

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断。

### Lines 65-72: Class MemoryPoolConfigurator
```python
class MemoryPoolConfigurator:
    """Base class for memory pool configurators.

    Subclasses compute pool sizes for their architecture via coeff+bias model.
    Both entry points return MemoryPoolConfig (with max_running_requests=None,
    to be filled by the consumer).
    """

```
**EN:** This range introduces `MemoryPoolConfigurator` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Base class for memory pool configurators."
**CN:** 这一段引入 `MemoryPoolConfigurator`，并定义其后续方法依赖的结构或元数据。

### Lines 73-77: Method MemoryPoolConfigurator.calculate_pool_sizes
```python
    def calculate_pool_sizes(
        self, available_bytes: int, page_size: int
    ) -> MemoryPoolConfig:
        """Profiling path: compute pool sizes from available bytes."""
        raise NotImplementedError
```
**EN:** This callable implements `MemoryPoolConfigurator.calculate_pool_sizes`. It takes `available_bytes`, `page_size` and mainly implements calculate pool sizes. The docstring states: "Profiling path: compute pool sizes from available bytes." In this range it sets up imports and shared symbols; performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `MemoryPoolConfigurator.calculate_pool_sizes`。它接收 `available_bytes`, `page_size`，主要用于实现 calculate pool sizes 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查。

### Lines 79-83: Method MemoryPoolConfigurator.calculate_pool_sizes_from_max_tokens
```python
    def calculate_pool_sizes_from_max_tokens(
        self, max_total_num_tokens: int, page_size: int
    ) -> MemoryPoolConfig:
        """Constraint path: recalculate pool sizes from a constrained max_tokens."""
        raise NotImplementedError
```
**EN:** This callable implements `MemoryPoolConfigurator.calculate_pool_sizes_from_max_tokens`. It takes `max_total_num_tokens`, `page_size` and mainly constructs data from an external representation. The docstring states: "Constraint path: recalculate pool sizes from a constrained max_tokens." In this range it sets up imports and shared symbols; performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `MemoryPoolConfigurator.calculate_pool_sizes_from_max_tokens`。它接收 `max_total_num_tokens`, `page_size`，主要用于从外部表示构造数据。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查。

### Lines 86-92: Class DefaultPoolConfigurator
```python
class DefaultPoolConfigurator(MemoryPoolConfigurator):
    """Configurator for standard models: MHA, MLA, NSA, FP4.

    coeff = cell_size (bytes per token across all layers)
    bias = 0
    """

```
**EN:** This range introduces `DefaultPoolConfigurator` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Configurator for standard models: MHA, MLA, NSA, FP4."
**CN:** 这一段引入 `DefaultPoolConfigurator`，并定义其后续方法依赖的结构或元数据。

### Lines 93-123: Method DefaultPoolConfigurator.__init__
```python
    def __init__(self, mr: ModelRunner):
        # Determine effective number of layers for KV cache
        if mambaish := mr.mambaish_config:
            effective_layer_ids = [
                i
                for i in mambaish.full_attention_layer_ids
                if mr.start_layer <= i < mr.end_layer
            ]
            num_layers = len(effective_layer_ids)
        else:
            num_layers = mr.num_effective_layers

        self._cell_size = self._compute_cell_size(mr, num_layers)

        # DFLASH: scale cell_size to account for draft model KV cache
        if mr.spec_algorithm.is_dflash() and not mr.is_draft_worker:
            from sglang.srt.speculative.dflash_utils import (
                scale_kv_cell_size_per_token_for_dflash,
            )

            draft_num_layers = mr.dflash_draft_num_layers
            if (
                draft_num_layers is not None
                and int(draft_num_layers) > 0
                and int(num_layers) > 0
            ):
                self._cell_size = scale_kv_cell_size_per_token_for_dflash(
                    target_cell_size_per_token=self._cell_size,
                    target_num_layers=int(num_layers),
                    draft_num_layers=int(draft_num_layers),
                )
```
**EN:** This callable implements `DefaultPoolConfigurator.__init__`. It takes `mr` and mainly initializes instance state and defaults. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `DefaultPoolConfigurator.__init__`。它接收 `mr`，主要用于初始化实例状态与默认值。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 125-180: Method DefaultPoolConfigurator._compute_cell_size
```python
    def _compute_cell_size(self, mr: ModelRunner, num_layers: int) -> int:
        """Compute per-token KV cache cost in bytes. Subclasses can override."""
        # args to config cell size
        model_config = mr.model_config
        kv_cache_dtype = mr.kv_cache_dtype

        kv_size = torch._utils._element_size(kv_cache_dtype)
        tp_size = get_attention_tp_size()

        if mr.use_mla_backend:
            cell_size = (
                (model_config.kv_lora_rank + model_config.qk_rope_head_dim)
                * num_layers
                * kv_size
            )
            if is_float4_e2m1fn_x2(kv_cache_dtype):
                # kv_scale_buffer
                scale_block_size = 16
                cell_size = (cell_size // 2) + (
                    (
                        (model_config.kv_lora_rank + model_config.qk_rope_head_dim)
                        // scale_block_size
                    )
                    * num_layers
                    * kv_size
                )

            # Add indexer KV cache overhead for NSA models (DeepSeek V3.2)
            if is_deepseek_nsa(model_config.hf_config):
                index_head_dim = get_nsa_index_head_dim(model_config.hf_config)
                indexer_size_per_token = (
                    index_head_dim
                    + index_head_dim // NSATokenToKVPool.quant_block_size * 4
                )
                element_size = torch._utils._element_size(
                    NSATokenToKVPool.index_k_with_scale_buffer_dtype
                )
                cell_size += indexer_size_per_token * num_layers * element_size
        else:
            cell_size = (
                model_config.get_num_kv_heads(tp_size)
                * (model_config.head_dim + model_config.v_head_dim)
                * num_layers
                * kv_size
            )

            if is_float4_e2m1fn_x2(kv_cache_dtype):
                # kv_scale_buffer
                scale_block_size = 16
                n = model_config.get_num_kv_heads(tp_size)
                k = model_config.head_dim
                cell_size = (cell_size // 2) + (
                    (n * k * num_layers * 2 * kv_size) // scale_block_size
                )

        return cell_size
```
**EN:** This callable implements `DefaultPoolConfigurator._compute_cell_size`. It takes `mr`, `num_layers` and mainly implements compute cell size. The docstring states: "Compute per-token KV cache cost in bytes."
**CN:** 这一可调用对象实现了 `DefaultPoolConfigurator._compute_cell_size`。它接收 `mr`, `num_layers`，主要用于实现 compute cell size 相关逻辑。

### Lines 182-187: Method DefaultPoolConfigurator.calculate_pool_sizes
```python
    def calculate_pool_sizes(
        self, available_bytes: int, page_size: int
    ) -> MemoryPoolConfig:
        max_total_num_tokens = available_bytes // self._cell_size
        max_total_num_tokens = max_total_num_tokens // page_size * page_size
        return MemoryPoolConfig(max_total_num_tokens=max_total_num_tokens)
```
**EN:** This callable implements `DefaultPoolConfigurator.calculate_pool_sizes`. It takes `available_bytes`, `page_size` and mainly implements calculate pool sizes.
**CN:** 这一可调用对象实现了 `DefaultPoolConfigurator.calculate_pool_sizes`。它接收 `available_bytes`, `page_size`，主要用于实现 calculate pool sizes 相关逻辑。

### Lines 189-193: Method DefaultPoolConfigurator.calculate_pool_sizes_from_max_tokens
```python
    def calculate_pool_sizes_from_max_tokens(
        self, max_total_num_tokens: int, page_size: int
    ) -> MemoryPoolConfig:
        max_total_num_tokens = max_total_num_tokens // page_size * page_size
        return MemoryPoolConfig(max_total_num_tokens=max_total_num_tokens)
```
**EN:** This callable implements `DefaultPoolConfigurator.calculate_pool_sizes_from_max_tokens`. It takes `max_total_num_tokens`, `page_size` and mainly constructs data from an external representation.
**CN:** 这一可调用对象实现了 `DefaultPoolConfigurator.calculate_pool_sizes_from_max_tokens`。它接收 `max_total_num_tokens`, `page_size`，主要用于从外部表示构造数据。

### Lines 196-202: Class HybridSWAPoolConfigurator
```python
class HybridSWAPoolConfigurator(MemoryPoolConfigurator):
    """Configurator for hybrid sliding window attention models (Gemma2, Command-R, MiMo).

    Splits available memory between full attention and SWA pools.
    Does NOT inherit DefaultPoolConfigurator — different coeff model.
    """

```
**EN:** This range introduces `HybridSWAPoolConfigurator` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Configurator for hybrid sliding window attention models (Gemma2, Command-R, MiMo)."
**CN:** 这一段引入 `HybridSWAPoolConfigurator`，并定义其后续方法依赖的结构或元数据。

### Lines 203-248: Method HybridSWAPoolConfigurator.__init__
```python
    def __init__(self, mr: ModelRunner):
        model_config = mr.model_config
        kv_cache_dtype = mr.kv_cache_dtype
        kv_size = torch._utils._element_size(kv_cache_dtype)
        tp_size = get_attention_tp_size()

        self._full_layers_num = len(model_config.full_attention_layer_ids)
        self._swa_layers_num = len(model_config.swa_attention_layer_ids)
        assert (
            self._swa_layers_num > 0
        ), "Hybrid SWA model must have at least one SWA layer"

        self._swa_full_tokens_ratio = mr.server_args.swa_full_tokens_ratio

        # Full layer per-token memory (bytes)
        self._full_per_token = (
            model_config.get_num_kv_heads(tp_size)
            * (model_config.head_dim + model_config.v_head_dim)
            * kv_size
        )

        # SWA layer per-token memory (bytes)
        self._swa_per_token = (
            model_config.get_swa_num_kv_heads(tp_size)
            * (model_config.swa_head_dim + model_config.swa_v_head_dim)
            * kv_size
        )

        # Bytes per token of max_total_num_tokens.
        #
        # Hybrid (full_layers > 0): max_total = full_tokens, so cell_size accounts
        # for both pools: F*nf + r*S*ns (where swa_tokens = full_tokens * r).
        #
        # All-SWA (full_layers == 0): max_total = swa_tokens directly. The ratio
        # is meaningless here -- there is no full pool to relate to, and every
        # token beyond the sliding window can be evicted. So cell_size = S*ns,
        # with no ratio factor applied.
        if self._full_layers_num == 0:
            self._cell_size = self._swa_per_token * self._swa_layers_num
        else:
            self._cell_size = (
                self._full_per_token * self._full_layers_num
                + self._swa_full_tokens_ratio
                * self._swa_per_token
                * self._swa_layers_num
            )
```
**EN:** This callable implements `HybridSWAPoolConfigurator.__init__`. It takes `mr` and mainly initializes instance state and defaults. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `HybridSWAPoolConfigurator.__init__`。它接收 `mr`，主要用于初始化实例状态与默认值。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 250-285: Method HybridSWAPoolConfigurator._solve_pool_sizes
```python
    def _solve_pool_sizes(
        self, max_total_num_tokens: int, page_size: int
    ) -> MemoryPoolConfig:
        """Core computation: split max_total_num_tokens into full/swa pool sizes."""

        def align_page_size(x: int) -> int:
            return (x // page_size) * page_size

        if self._full_layers_num == 0:
            # All-SWA: no full pool, max_total = actual SWA pool size.
            # Ratio is not applied -- see __init__ comment.
            swa_tokens = align_page_size(max_total_num_tokens)
            logger.info(
                f"Use sliding window memory pool (all SWA). "
                f"swa_layer_tokens={swa_tokens}"
            )
            return MemoryPoolConfig(
                max_total_num_tokens=swa_tokens,
                full_max_total_num_tokens=0,
                swa_max_total_num_tokens=swa_tokens,
            )

        # Hybrid: full_tokens = max_total_num_tokens, swa_tokens = full_tokens * ratio
        full_tokens = align_page_size(max_total_num_tokens)
        swa_tokens = align_page_size(int(full_tokens * self._swa_full_tokens_ratio))

        logger.info(
            f"Use sliding window memory pool. "
            f"full_layer_tokens={full_tokens}, swa_layer_tokens={swa_tokens}"
        )

        return MemoryPoolConfig(
            max_total_num_tokens=full_tokens,
            full_max_total_num_tokens=full_tokens,
            swa_max_total_num_tokens=swa_tokens,
        )
```
**EN:** This callable implements `HybridSWAPoolConfigurator._solve_pool_sizes`. It takes `max_total_num_tokens`, `page_size` and mainly implements solve pool sizes. The docstring states: "Core computation: split max_total_num_tokens into full/swa pool sizes." In this range it emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `HybridSWAPoolConfigurator._solve_pool_sizes`。它接收 `max_total_num_tokens`, `page_size`，主要用于实现 solve pool sizes 相关逻辑。 在这一范围内，它会输出日志以便诊断。

### Lines 287-291: Method HybridSWAPoolConfigurator.calculate_pool_sizes
```python
    def calculate_pool_sizes(
        self, available_bytes: int, page_size: int
    ) -> MemoryPoolConfig:
        max_total_num_tokens = int(available_bytes // self._cell_size)
        return self._solve_pool_sizes(max_total_num_tokens, page_size)
```
**EN:** This callable implements `HybridSWAPoolConfigurator.calculate_pool_sizes`. It takes `available_bytes`, `page_size` and mainly implements calculate pool sizes.
**CN:** 这一可调用对象实现了 `HybridSWAPoolConfigurator.calculate_pool_sizes`。它接收 `available_bytes`, `page_size`，主要用于实现 calculate pool sizes 相关逻辑。

### Lines 293-296: Method HybridSWAPoolConfigurator.calculate_pool_sizes_from_max_tokens
```python
    def calculate_pool_sizes_from_max_tokens(
        self, max_total_num_tokens: int, page_size: int
    ) -> MemoryPoolConfig:
        return self._solve_pool_sizes(max_total_num_tokens, page_size)
```
**EN:** This callable implements `HybridSWAPoolConfigurator.calculate_pool_sizes_from_max_tokens`. It takes `max_total_num_tokens`, `page_size` and mainly constructs data from an external representation.
**CN:** 这一可调用对象实现了 `HybridSWAPoolConfigurator.calculate_pool_sizes_from_max_tokens`。它接收 `max_total_num_tokens`, `page_size`，主要用于从外部表示构造数据。

### Lines 299-306: Class _DSV4PoolSizes
```python
@dataclass
class _DSV4PoolSizes:
    full_max_total_num_tokens: int
    swa_max_total_num_tokens: int
    c4_max_total_num_tokens: int
    c128_max_total_num_tokens: int
    c4_state_pool_size: int
    c128_state_pool_size: int
```
**EN:** This range introduces `_DSV4PoolSizes` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_DSV4PoolSizes`，并定义其后续方法依赖的结构或元数据。

### Lines 309-316: Class DSV4PoolConfigurator
```python
class DSV4PoolConfigurator(MemoryPoolConfigurator):
    """Configurator for DSV4 compressed-attention models.

    Splits available memory across full / swa / c4 / c128 + c4_state / c128_state
    pools. coeff is bytes_per_full_token (inflated by (T+D)/T when speculative
    decode reserves a draft worker, mirroring dflash's cell_size scaling); bias = 0.
    """

```
**EN:** This range introduces `DSV4PoolConfigurator` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Configurator for DSV4 compressed-attention models."
**CN:** 这一段引入 `DSV4PoolConfigurator`，并定义其后续方法依赖的结构或元数据。

### Lines 317-363: Method DSV4PoolConfigurator.__init__
```python
    def __init__(self, mr: ModelRunner):
        cfg = mr.model_config
        self.qk_nope_head_dim = cfg.qk_nope_head_dim
        self.qk_rope_head_dim = cfg.qk_rope_head_dim
        self.indexer_head_dim = cfg.index_head_dim
        self.compression_ratios = cfg.compress_ratios
        self.swa_page_size = cfg.window_size
        self.swa_ratio = mr.server_args.swa_full_tokens_ratio
        self.is_speculative = mr.server_args.speculative_algorithm is not None
        if mr.enable_hisparse:
            from sglang.srt.mem_cache.sparsity import parse_hisparse_config

            self.c4_shrink_factor = parse_hisparse_config(
                mr.server_args
            ).host_to_device_ratio
        else:
            self.c4_shrink_factor = 1
        assert self.c4_shrink_factor >= 1
        if self.c4_shrink_factor > 1:
            logger.info(f"HiSparse c4 host-to-device ratio = {self.c4_shrink_factor}")

        self.c4_ring_size = get_compress_state_ring_size(4, self.is_speculative)
        self.c128_ring_size = get_compress_state_ring_size(128, self.is_speculative)

        self.num_layers_total = len(self.compression_ratios)
        self.num_layers_ca4 = sum(1 for r in self.compression_ratios if r == 4)
        self.num_layers_ca128 = sum(1 for r in self.compression_ratios if r == 128)

        self.bytes_per_full_token = self._get_bytes_per_full_token()
        if self.is_speculative:
            # Reserve memory for the speculative draft worker by inflating
            # per-token bytes by (target+draft)/target. Equivalent to dflash's
            # scale_kv_cell_size_per_token_for_dflash but applied to
            # bytes_per_full_token: tokens = avail / (bpft * (T+D)/T).
            draft_layers = 1
            target_layers = self.num_layers_total
            self.bytes_per_full_token *= (target_layers + draft_layers) / target_layers

        # Online c128 keeps a single in-progress (max, sum, kv) state per index
        # and assumes a strict forward-only schedule. Speculative decode (MTP)
        # would need rollback / replay across draft and verify, which the
        # online path doesn't support yet.
        if envs.SGLANG_OPT_USE_ONLINE_COMPRESS.get():
            assert (
                mr.spec_algorithm.is_none()
            ), "SGLANG_OPT_USE_ONLINE_COMPRESS does not support speculative decode (MTP) yet"
            logger.info("DSV4 compressed attention: online c128 enabled (ring_size=1)")
```
**EN:** This callable implements `DSV4PoolConfigurator.__init__`. It takes `mr` and mainly initializes instance state and defaults. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `DSV4PoolConfigurator.__init__`。它接收 `mr`，主要用于初始化实例状态与默认值。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；输出日志以便诊断。

### Lines 365-403: Method DSV4PoolConfigurator._get_bytes_per_full_token
```python
    def _get_bytes_per_full_token(self) -> float:
        kv_bytes = self.qk_nope_head_dim + self.qk_rope_head_dim * 2 + 8

        quant_block_size = 128
        indexer_bytes = (
            self.indexer_head_dim + self.indexer_head_dim // quant_block_size * 4
        )

        attn_head_dim = self.qk_nope_head_dim + self.qk_rope_head_dim
        state_dtype_size = 4
        c4_state_bytes = 2 * 2 * attn_head_dim * state_dtype_size
        # Online c128 stores (max, sum, kv) per slot (3*head_dim) instead of
        # raw (kv, score) (2*head_dim). Combined with ring_size=1 this still
        # nets a large reduction (~3/256x) but the per-slot bytes go up.
        c128_online = envs.SGLANG_OPT_USE_ONLINE_COMPRESS.get()
        c128_state_bytes = (
            (3 if c128_online else 2 * 1) * attn_head_dim * state_dtype_size
        )
        c4_indexer_state_bytes = 2 * 2 * self.indexer_head_dim * state_dtype_size

        c4_state_ratio = self.c4_ring_size / self.swa_page_size
        c128_state_ratio = self.c128_ring_size / self.swa_page_size

        c4_frac = 1 / (4 * self.c4_shrink_factor)
        return (
            self.swa_ratio * kv_bytes * self.num_layers_total
            + c4_frac * kv_bytes * self.num_layers_ca4
            + 1 / 128 * kv_bytes * self.num_layers_ca128
            + 1 / 4 * indexer_bytes * self.num_layers_ca4
            + self.swa_ratio * c4_state_ratio * c4_state_bytes * self.num_layers_ca4
            + self.swa_ratio
            * c128_state_ratio
            * c128_state_bytes
            * self.num_layers_ca128
            + self.swa_ratio
            * c4_state_ratio
            * c4_indexer_state_bytes
            * self.num_layers_ca4
        )
```
**EN:** This callable implements `DSV4PoolConfigurator._get_bytes_per_full_token` and mainly converts data into another representation. In this range it reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `DSV4PoolConfigurator._get_bytes_per_full_token`，主要用于将数据转换为另一种表示。 在这一范围内，它会读取环境变量驱动的配置。

### Lines 405-415: Method DSV4PoolConfigurator._compute_dsv4_sizes
```python
    def _compute_dsv4_sizes(self, full_token: int, page_size: int) -> _DSV4PoolSizes:
        full_token = full_token // page_size * page_size
        swa_tokens = int(full_token * self.swa_ratio) // page_size * page_size
        return _DSV4PoolSizes(
            full_max_total_num_tokens=full_token,
            swa_max_total_num_tokens=swa_tokens,
            c4_max_total_num_tokens=full_token // (4 * self.c4_shrink_factor),
            c128_max_total_num_tokens=full_token // 128,
            c4_state_pool_size=swa_tokens // self.swa_page_size * self.c4_ring_size,
            c128_state_pool_size=swa_tokens // self.swa_page_size * self.c128_ring_size,
        )
```
**EN:** This callable implements `DSV4PoolConfigurator._compute_dsv4_sizes`. It takes `full_token`, `page_size` and mainly implements compute dsv4 sizes.
**CN:** 这一可调用对象实现了 `DSV4PoolConfigurator._compute_dsv4_sizes`。它接收 `full_token`, `page_size`，主要用于实现 compute dsv4 sizes 相关逻辑。

### Lines 417-435: Method DSV4PoolConfigurator._to_config
```python
    def _to_config(self, sizes: _DSV4PoolSizes) -> MemoryPoolConfig:
        full = sizes.full_max_total_num_tokens
        swa = sizes.swa_max_total_num_tokens
        logger.info(
            f"DSV4 pool sizes: full={full}, swa={swa}, "
            f"c4={sizes.c4_max_total_num_tokens}, "
            f"c128={sizes.c128_max_total_num_tokens}, "
            f"c4_state={sizes.c4_state_pool_size}, "
            f"c128_state={sizes.c128_state_pool_size}"
        )
        return MemoryPoolConfig(
            max_total_num_tokens=full,
            full_max_total_num_tokens=full,
            swa_max_total_num_tokens=swa,
            c4_max_total_num_tokens=sizes.c4_max_total_num_tokens,
            c128_max_total_num_tokens=sizes.c128_max_total_num_tokens,
            c4_state_pool_size=sizes.c4_state_pool_size,
            c128_state_pool_size=sizes.c128_state_pool_size,
        )
```
**EN:** This callable implements `DSV4PoolConfigurator._to_config`. It takes `sizes` and mainly converts data into another representation. In this range it emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `DSV4PoolConfigurator._to_config`。它接收 `sizes`，主要用于将数据转换为另一种表示。 在这一范围内，它会输出日志以便诊断。

### Lines 437-452: Method DSV4PoolConfigurator.calculate_pool_sizes
```python
    def calculate_pool_sizes(
        self, available_bytes: int, page_size: int
    ) -> MemoryPoolConfig:
        assert (
            page_size % 128 == 0
        ), "page_size must be multiple of 128 for compressed attention"

        full_token = int(available_bytes / self.bytes_per_full_token)
        sizes = self._compute_dsv4_sizes(full_token, page_size)
        logger.info(
            f"DSV4 memory calculation: "
            f"bytes_per_full_token={self.bytes_per_full_token:.2f}, "
            f"available_bytes={available_bytes / (1 << 30):.2f} GB, "
            f"full_token={sizes.full_max_total_num_tokens}"
        )
        return self._to_config(sizes)
```
**EN:** This callable implements `DSV4PoolConfigurator.calculate_pool_sizes`. It takes `available_bytes`, `page_size` and mainly implements calculate pool sizes. In this range it performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `DSV4PoolConfigurator.calculate_pool_sizes`。它接收 `available_bytes`, `page_size`，主要用于实现 calculate pool sizes 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；输出日志以便诊断。

### Lines 454-461: Method DSV4PoolConfigurator.calculate_pool_sizes_from_max_tokens
```python
    def calculate_pool_sizes_from_max_tokens(
        self, max_total_num_tokens: int, page_size: int
    ) -> MemoryPoolConfig:
        assert (
            page_size % 128 == 0
        ), "page_size must be multiple of 128 for compressed attention"
        sizes = self._compute_dsv4_sizes(max_total_num_tokens, page_size)
        return self._to_config(sizes)
```
**EN:** This callable implements `DSV4PoolConfigurator.calculate_pool_sizes_from_max_tokens`. It takes `max_total_num_tokens`, `page_size` and mainly constructs data from an external representation. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `DSV4PoolConfigurator.calculate_pool_sizes_from_max_tokens`。它接收 `max_total_num_tokens`, `page_size`，主要用于从外部表示构造数据。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 464-473: Function create_memory_pool_configurator
```python
def create_memory_pool_configurator(
    mr: ModelRunner,
) -> MemoryPoolConfigurator:
    """Factory: select the right configurator for the model architecture."""
    if is_deepseek_v4(mr.model_config.hf_config) and mr.is_hybrid_swa:
        return DSV4PoolConfigurator(mr)
    if mr.is_hybrid_swa:
        return HybridSWAPoolConfigurator(mr)
    # Future: MambaPoolConfigurator
    return DefaultPoolConfigurator(mr)
```
**EN:** This callable implements `create_memory_pool_configurator`. It takes `mr` and mainly converts data into another representation. The docstring states: "Factory: select the right configurator for the model architecture."
**CN:** 这一可调用对象实现了 `create_memory_pool_configurator`。它接收 `mr`，主要用于将数据转换为另一种表示。

## Key Concepts / 关键概念
- `MemoryPoolConfig`: core class or state container / 核心类或状态容器
- `MemoryPoolConfigurator`: core class or state container / 核心类或状态容器
- `DefaultPoolConfigurator`: core class or state container / 核心类或状态容器
- `HybridSWAPoolConfigurator`: core class or state container / 核心类或状态容器
- `_DSV4PoolSizes`: core class or state container / 核心类或状态容器
- `DSV4PoolConfigurator`: core class or state container / 核心类或状态容器
- `create_memory_pool_configurator`: converts data into another representation / 将数据转换为另一种表示

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `dataclasses`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.configs.model_config`, `sglang.srt.environ`, `sglang.srt.layers.dp_attention`, `sglang.srt.mem_cache.deepseek_v4_memory_pool`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.utils.common`, `sglang.srt.model_executor.model_runner`, `sglang.srt.speculative.dflash_utils`, `sglang.srt.mem_cache.sparsity`
