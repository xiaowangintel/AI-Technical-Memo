# deepseek_v4_memory_pool.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/deepseek_v4_memory_pool.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the deepseek V 4 memory pool logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的DeepSeek v 4内存池相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Imports and setup / 导入与初始化
```python
from __future__ import annotations

import logging
from contextlib import nullcontext
from typing import List, Literal, NamedTuple, Optional, Tuple

import torch

from sglang.jit_kernel.deepseek_v4 import fused_k_norm_rope_flashmla, fused_store_cache
from sglang.srt.constants import GPU_MEMORY_TYPE_KV_CACHE
from sglang.srt.environ import envs
from sglang.srt.layers.attention.dsv4 import (
# ... omitted for brevity ...
from sglang.srt.mem_cache.deepseek_v4_compress_state import CompressStatePool
from sglang.srt.mem_cache.memory_pool import KVCache
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import ceil_div
```
**EN:** Imports `__future__`, `logging`, `contextlib`, `typing`, `torch`, `sglang.jit_kernel.deepseek_v4` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `logging`, `contextlib`, `typing`, `torch`, `sglang.jit_kernel.deepseek_v4` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 23-25: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)

ONLINE_C128 = envs.SGLANG_OPT_USE_ONLINE_COMPRESS.get()
```
**EN:** Defines module-level variables such as `logger`, `ONLINE_C128`.
**CN:** 定义模块级变量，例如 `logger`, `ONLINE_C128`。

### Lines 28-41: get_compress_state_ring_size implementation / get_compress_state_ring_size 实现
```python
def get_compress_state_ring_size(
    compress_ratio: int, is_speculative: bool = False
) -> int:
    assert compress_ratio in [4, 128], f"Unsupported {compress_ratio = }"
    # Online c128 keeps a single (max, sum, kv) state per index instead of a
    # 128-slot ring buffer of raw tokens, so ring_size collapses to 1. Online
    # is incompatible with speculative decode for now.
    if compress_ratio == 128 and ONLINE_C128:
        assert not is_speculative, "online c128 does not support MTP"
        return 1
    if is_speculative:
        return 16 if compress_ratio == 4 else 256
    else:
        return 8 if compress_ratio == 4 else 128
```
**EN:** Retrieves the requested data or state from the current object. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。它会向调用方返回计算结果。

### Lines 44-44: DeepSeekV4SingleKVPool declaration / DeepSeekV4SingleKVPool 声明
```python
class DeepSeekV4SingleKVPool(KVCache):
```
**EN:** Declares the `DeepSeekV4SingleKVPool` class and connects it to `KVCache`.
**CN:** 声明 `DeepSeekV4SingleKVPool` 类，并将其关联到 `KVCache`。

### Lines 45-75: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        size: int,
        page_size: int,
        dtype: torch.dtype,
        qk_nope_head_dim: int,
        qk_rope_head_dim: int,
        layer_num: int,
        device: str,
        enable_memory_saver: bool,
        start_layer: Optional[int] = None,
        end_layer: Optional[int] = None,
# ... omitted for brevity ...
        self.quantize_block_size = 64
        self.rope_storage_dtype = torch.bfloat16
        self.k_with_scale_buffer_dtype = torch.int8
        self._create_buffers()
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `DeepSeekV4SingleKVPool`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `DeepSeekV4SingleKVPool`。

### Lines 77-89: _create_buffers implementation / _create_buffers 实现
```python
    def _create_buffers(self):
        with self.memory_saver_adapter.region(GPU_MEMORY_TYPE_KV_CACHE):
            with (
                torch.cuda.use_mem_pool(self.custom_mem_pool)
                if self.custom_mem_pool
                else nullcontext()
            ):
                self.kv_buffer = [
                    self.create_buffer(
                        num_pages=(self.size + self.page_size + 1) // self.page_size,
                    )
                    for _ in range(self.layer_num)
                ]
```
**EN:** Implements the create buffers routine for this scope. It belongs to `DeepSeekV4SingleKVPool`.
**CN:** 实现当前作用域中的create buffers例程。 该方法属于 `DeepSeekV4SingleKVPool`。

### Lines 91-98: get_bytes_per_token implementation / get_bytes_per_token 实现
```python
    def get_bytes_per_token(self) -> int:
        dim_per_token = (
            self.qk_nope_head_dim
            + self.qk_rope_head_dim * self.rope_storage_dtype.itemsize
            + self.qk_nope_head_dim // self.quantize_block_size
            + self.scale_pad
        )
        return dim_per_token
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4SingleKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4SingleKVPool`。它会向调用方返回计算结果。

### Lines 100-117: create_buffer implementation / create_buffer 实现
```python
    def create_buffer(self, *, num_pages: int):
        bytes_per_token = self.get_bytes_per_token()
        self.kv_cache_total_dim = bytes_per_token
        bytes_per_page_non_padded = self.page_size * bytes_per_token
        self.bytes_per_page_padded = ceil_div(bytes_per_page_non_padded, 576) * 576

        assert bytes_per_token == 448 + 64 * 2 + 8, (
            "DSV4 KV layout: qk_nope_head_dim FP8 (448) + qk_rope_head_dim BF16 "
            "(64*2) + nope FP8 scales + scale_pad = 584 bytes/token"
        )
        assert self.store_dtype == torch.uint8

        return torch.zeros(
            num_pages,
            self.bytes_per_page_padded,
            dtype=self.store_dtype,
            device=self.device,
        )
```
**EN:** Constructs a new object or resource with the requested configuration. It belongs to `DeepSeekV4SingleKVPool`. It returns a computed result to its caller.
**CN:** 按给定配置构造新的对象或资源。 该方法属于 `DeepSeekV4SingleKVPool`。它会向调用方返回计算结果。

### Lines 119-130: set_key_buffer implementation / set_key_buffer 实现
```python
    def set_key_buffer(
        self,
        layer_id: int,
        loc: torch.Tensor,
        cache_nope_fp8_rope_bf16_pack: NopeFp8RopeBf16Pack,
    ):
        dsv4_index_buf_accessor.SetKAndS.execute(
            pool=self,
            buf=self.kv_buffer[layer_id],
            loc=loc,
            nope_fp8_rope_bf16_pack=cache_nope_fp8_rope_bf16_pack,
        )
```
**EN:** Updates internal state with the provided value. It belongs to `DeepSeekV4SingleKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `DeepSeekV4SingleKVPool`。

### Lines 132-144: set_key_buffer_fused implementation / set_key_buffer_fused 实现
```python
    def set_key_buffer_fused(
        self,
        layer_id: int,
        loc: torch.Tensor,
        cache_k: torch.Tensor,
    ) -> None:
        return fused_store_cache(
            input=cache_k,
            cache=self.kv_buffer[layer_id],
            indices=loc,
            page_size=self.page_size,
            type="flashmla",
        )
```
**EN:** Updates internal state with the provided value. It belongs to `DeepSeekV4SingleKVPool`. It returns a computed result to its caller.
**CN:** 使用给定值更新内部状态。 该方法属于 `DeepSeekV4SingleKVPool`。它会向调用方返回计算结果。

### Lines 146-147: get_key_buffer implementation / get_key_buffer 实现
```python
    def get_key_buffer(self, layer_id: int):
        return self.kv_buffer[layer_id]
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4SingleKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4SingleKVPool`。它会向调用方返回计算结果。

### Lines 149-150: set_kv_buffer implementation / set_kv_buffer 实现
```python
    def set_kv_buffer(self, *args, **kwargs) -> None:
        raise NotImplementedError()
```
**EN:** Updates internal state with the provided value. It belongs to `DeepSeekV4SingleKVPool`. It validates error cases explicitly.
**CN:** 使用给定值更新内部状态。 该方法属于 `DeepSeekV4SingleKVPool`。它会显式处理错误场景。

### Lines 152-153: get_value_buffer implementation / get_value_buffer 实现
```python
    def get_value_buffer(self, layer_id: int) -> torch.Tensor:
        raise NotImplementedError("Use get_key_buffer instead.")
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4SingleKVPool`. It validates error cases explicitly.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4SingleKVPool`。它会显式处理错误场景。

### Lines 155-156: get_kv_buffer implementation / get_kv_buffer 实现
```python
    def get_kv_buffer(self, layer_id: int) -> Tuple[torch.Tensor, torch.Tensor]:
        raise NotImplementedError("Use get_key_buffer instead.")
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4SingleKVPool`. It validates error cases explicitly.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4SingleKVPool`。它会显式处理错误场景。

### Lines 159-160: HiSparseC4DevicePool declaration / HiSparseC4DevicePool 声明
```python
class HiSparseC4DevicePool(DeepSeekV4SingleKVPool):

```
**EN:** Declares the `HiSparseC4DevicePool` class and connects it to `DeepSeekV4SingleKVPool`.
**CN:** 声明 `HiSparseC4DevicePool` 类，并将其关联到 `DeepSeekV4SingleKVPool`。

### Lines 161-192: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        size: int,
        page_size: int,
        dtype: torch.dtype,
        qk_nope_head_dim: int,
        qk_rope_head_dim: int,
        layer_num: int,
        device: str,
        enable_memory_saver: bool,
        start_layer: int | None = None,
        end_layer: int | None = None,
# ... omitted for brevity ...
            dtype=torch.uint64,
            device=self.device,
        )
        self.compress_ratio = 4
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `HiSparseC4DevicePool`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `HiSparseC4DevicePool`。

### Lines 194-197: register_mapping implementation / register_mapping 实现
```python
    def register_mapping(self, full_to_hisparse_device_index_mapping: torch.Tensor):
        self.full_to_hisparse_device_index_mapping = (
            full_to_hisparse_device_index_mapping
        )
```
**EN:** Registers metadata so other components can discover this object. It belongs to `HiSparseC4DevicePool`.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `HiSparseC4DevicePool`。

### Lines 199-202: translate_loc_from_full_to_compressed implementation / translate_loc_from_full_to_compressed 实现
```python
    def translate_loc_from_full_to_compressed(self, full_indices: torch.Tensor):
        mask = (full_indices + 1) % self.compress_ratio == 0
        compressed_indices = full_indices[mask] // self.compress_ratio
        return compressed_indices
```
**EN:** Implements the translate LOC from full TO compressed routine for this scope. It belongs to `HiSparseC4DevicePool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的translate loc from完整to compressed例程。 该方法属于 `HiSparseC4DevicePool`。它会向调用方返回计算结果。

### Lines 204-207: translate_loc_to_hisparse_device implementation / translate_loc_to_hisparse_device 实现
```python
    def translate_loc_to_hisparse_device(self, compressed_indices: torch.Tensor):
        return self.full_to_hisparse_device_index_mapping[compressed_indices].to(
            torch.int32
        )
```
**EN:** Implements the translate LOC TO hisparse device routine for this scope. It belongs to `HiSparseC4DevicePool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的translate loc to hisparse device例程。 该方法属于 `HiSparseC4DevicePool`。它会向调用方返回计算结果。

### Lines 209-210: _translate_loc_to_hisparse_device implementation / _translate_loc_to_hisparse_device 实现
```python
    def _translate_loc_to_hisparse_device(self, compressed_indices: torch.Tensor):
        return self.full_to_hisparse_device_index_mapping[compressed_indices]
```
**EN:** Implements the translate LOC TO hisparse device routine for this scope. It belongs to `HiSparseC4DevicePool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的translate loc to hisparse device例程。 该方法属于 `HiSparseC4DevicePool`。它会向调用方返回计算结果。

### Lines 212-215: translate_loc_from_full_to_hisparse_device implementation / translate_loc_from_full_to_hisparse_device 实现
```python
    def translate_loc_from_full_to_hisparse_device(self, full_indices: torch.Tensor):
        return self._translate_loc_to_hisparse_device(
            self.translate_loc_from_full_to_compressed(full_indices)
        )
```
**EN:** Implements the translate LOC from full TO hisparse device routine for this scope. It belongs to `HiSparseC4DevicePool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的translate loc from完整to hisparse device例程。 该方法属于 `HiSparseC4DevicePool`。它会向调用方返回计算结果。

### Lines 217-224: set_key_buffer implementation / set_key_buffer 实现
```python
    def set_key_buffer(
        self,
        layer_id: int,
        loc: torch.Tensor,
        cache_nope_fp8_rope_bf16_pack,
    ):
        loc = self.translate_loc_to_hisparse_device(loc)
        super().set_key_buffer(layer_id, loc, cache_nope_fp8_rope_bf16_pack)
```
**EN:** Updates internal state with the provided value. It belongs to `HiSparseC4DevicePool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `HiSparseC4DevicePool`。

### Lines 226-233: set_key_buffer_fused implementation / set_key_buffer_fused 实现
```python
    def set_key_buffer_fused(
        self,
        layer_id: int,
        loc: torch.Tensor,
        cache_k: torch.Tensor,
    ) -> None:
        loc = self.translate_loc_to_hisparse_device(loc)
        return super().set_key_buffer_fused(layer_id, loc, cache_k)
```
**EN:** Updates internal state with the provided value. It belongs to `HiSparseC4DevicePool`. It returns a computed result to its caller.
**CN:** 使用给定值更新内部状态。 该方法属于 `HiSparseC4DevicePool`。它会向调用方返回计算结果。

### Lines 235-236: get_cpu_copy implementation / get_cpu_copy 实现
```python
    def get_cpu_copy(self, indices, mamba_indices=None):
        raise NotImplementedError("HiSparseC4DevicePool does not support get_cpu_copy")
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HiSparseC4DevicePool`. It validates error cases explicitly.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HiSparseC4DevicePool`。它会显式处理错误场景。

### Lines 238-239: load_cpu_copy implementation / load_cpu_copy 实现
```python
    def load_cpu_copy(self, kv_cache_cpu, indices, mamba_indices=None):
        raise NotImplementedError("HiSparseC4DevicePool does not support load_cpu_copy")
```
**EN:** Loads state from an external or serialized representation. It belongs to `HiSparseC4DevicePool`. It validates error cases explicitly.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `HiSparseC4DevicePool`。它会显式处理错误场景。

### Lines 242-242: DeepSeekV4IndexerPool declaration / DeepSeekV4IndexerPool 声明
```python
class DeepSeekV4IndexerPool(KVCache):
```
**EN:** Declares the `DeepSeekV4IndexerPool` class and connects it to `KVCache`.
**CN:** 声明 `DeepSeekV4IndexerPool` 类，并将其关联到 `KVCache`。

### Lines 243-244: Shared state definitions / 共享状态定义
```python
    quant_block_size = 128
    index_k_with_scale_buffer_dtype = torch.uint8
```
**EN:** Defines class-level variables such as `quant_block_size`, `index_k_with_scale_buffer_dtype`.
**CN:** 定义类级变量，例如 `quant_block_size`, `index_k_with_scale_buffer_dtype`。

### Lines 246-270: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        size: int,
        page_size: int,
        dtype: torch.dtype,
        index_head_dim: int,
        layer_num: int,
        device: str,
        enable_memory_saver: bool,
        start_layer: Optional[int] = None,
        end_layer: Optional[int] = None,
    ):
# ... omitted for brevity ...
        )
        self.index_head_dim = index_head_dim

        self._create_buffer()
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `DeepSeekV4IndexerPool`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `DeepSeekV4IndexerPool`。

### Lines 272-290: _create_buffer implementation / _create_buffer 实现
```python
    def _create_buffer(self):
        num_scales_per_token = self.index_head_dim // self.quant_block_size
        page_bytes = self.page_size * self.index_head_dim
        page_bytes += self.page_size * num_scales_per_token * 4
        with self.memory_saver_adapter.region(GPU_MEMORY_TYPE_KV_CACHE):
            with (
                torch.cuda.use_mem_pool(self.custom_mem_pool)
                if self.custom_mem_pool
                else nullcontext()
            ):
                self.index_k_with_scale_buffer = [
                    torch.zeros(
# ... omitted for brevity ...
                        device=self.device,
                    )
                    for _ in range(self.layer_num)
                ]
```
**EN:** Implements the create buffer routine for this scope. It belongs to `DeepSeekV4IndexerPool`.
**CN:** 实现当前作用域中的create buffer例程。 该方法属于 `DeepSeekV4IndexerPool`。

### Lines 292-293: get_kv_buffer implementation / get_kv_buffer 实现
```python
    def get_kv_buffer(self, layer_id: int) -> Tuple[torch.Tensor, torch.Tensor]:
        raise NotImplementedError()
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4IndexerPool`. It validates error cases explicitly.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4IndexerPool`。它会显式处理错误场景。

### Lines 295-296: get_key_buffer implementation / get_key_buffer 实现
```python
    def get_key_buffer(self, layer_id: int) -> torch.Tensor:
        raise NotImplementedError()
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4IndexerPool`. It validates error cases explicitly.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4IndexerPool`。它会显式处理错误场景。

### Lines 298-299: get_value_buffer implementation / get_value_buffer 实现
```python
    def get_value_buffer(self, layer_id: int) -> torch.Tensor:
        raise NotImplementedError()
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4IndexerPool`. It validates error cases explicitly.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4IndexerPool`。它会显式处理错误场景。

### Lines 301-302: set_kv_buffer implementation / set_kv_buffer 实现
```python
    def set_kv_buffer(self, *args, **kwargs) -> None:
        raise NotImplementedError()
```
**EN:** Updates internal state with the provided value. It belongs to `DeepSeekV4IndexerPool`. It validates error cases explicitly.
**CN:** 使用给定值更新内部状态。 该方法属于 `DeepSeekV4IndexerPool`。它会显式处理错误场景。

### Lines 304-305: get_index_k_with_scale_buffer implementation / get_index_k_with_scale_buffer 实现
```python
    def get_index_k_with_scale_buffer(self, layer_id: int) -> torch.Tensor:
        return self.index_k_with_scale_buffer[layer_id]
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4IndexerPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4IndexerPool`。它会向调用方返回计算结果。

### Lines 307-316: get_index_k_scale_buffer implementation / get_index_k_scale_buffer 实现
```python
    def get_index_k_scale_buffer(
        self,
        layer_id: int,
        seq_len: int,
        page_indices: torch.Tensor,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        buf = self.index_k_with_scale_buffer[layer_id]
        return index_buf_accessor.GetKAndS.execute(
            self, buf, seq_len=seq_len, page_indices=page_indices
        )
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4IndexerPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4IndexerPool`。它会向调用方返回计算结果。

### Lines 318-328: set_index_k_scale_buffer implementation / set_index_k_scale_buffer 实现
```python
    def set_index_k_scale_buffer(
        self,
        layer_id: int,
        loc: torch.Tensor,
        index_k: torch.Tensor,
        index_k_scale: torch.Tensor,
    ) -> None:
        buf = self.index_k_with_scale_buffer[layer_id - self.start_layer]
        index_buf_accessor.SetKAndS.execute(
            pool=self, buf=buf, loc=loc, index_k=index_k, index_k_scale=index_k_scale
        )
```
**EN:** Updates internal state with the provided value. It belongs to `DeepSeekV4IndexerPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `DeepSeekV4IndexerPool`。

### Lines 330-342: set_index_fused implementation / set_index_fused 实现
```python
    def set_index_fused(
        self,
        layer_id: int,
        loc: torch.Tensor,
        cache_k: torch.Tensor,
    ) -> None:
        return fused_store_cache(
            input=cache_k,
            cache=self.index_k_with_scale_buffer[layer_id - self.start_layer],
            indices=loc,
            page_size=self.page_size,
            type="indexer",
        )
```
**EN:** Updates internal state with the provided value. It belongs to `DeepSeekV4IndexerPool`. It returns a computed result to its caller.
**CN:** 使用给定值更新内部状态。 该方法属于 `DeepSeekV4IndexerPool`。它会向调用方返回计算结果。

### Lines 345-345: DeepSeekV4LayerItem declaration / DeepSeekV4LayerItem 声明
```python
class DeepSeekV4LayerItem(NamedTuple):
```
**EN:** Declares the `DeepSeekV4LayerItem` class and connects it to `NamedTuple`.
**CN:** 声明 `DeepSeekV4LayerItem` 类，并将其关联到 `NamedTuple`。

### Lines 346-348: Shared state definitions / 共享状态定义
```python
    compress_ratio: Literal[0, 4, 128]
    compress_layer_id: int
    compress_kv_pool: Optional[DeepSeekV4SingleKVPool] = None
```
**EN:** Defines class-level variables such as `compress_ratio`, `compress_layer_id`, `compress_kv_pool`.
**CN:** 定义类级变量，例如 `compress_ratio`, `compress_layer_id`, `compress_kv_pool`。

### Lines 351-352: DeepSeekV4TokenToKVPool declaration / DeepSeekV4TokenToKVPool 声明
```python
class DeepSeekV4TokenToKVPool(BaseSWAKVPool):

```
**EN:** Declares the `DeepSeekV4TokenToKVPool` class and connects it to `BaseSWAKVPool`.
**CN:** 声明 `DeepSeekV4TokenToKVPool` 类，并将其关联到 `BaseSWAKVPool`。

### Lines 353-483: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        max_num_reqs: int,
        swa_size: int,
        c4_size: int,
        c128_size: int,
        c4_state_pool_size: int,
        c128_state_pool_size: int,
        page_size: int,
        swa_page_size: int,
        dtype: torch.dtype,
        state_dtype: torch.dtype,
# ... omitted for brevity ...
        self._init_paged_compress_states(enable_memory_saver)

        self._should_cache_swa = envs.SGLANG_OPT_CACHE_SWA_TRANSLATION.get()
        self.cached_loc = None
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `DeepSeekV4TokenToKVPool`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `DeepSeekV4TokenToKVPool`。

### Lines 485-486: register_mapping implementation / register_mapping 实现
```python
    def register_mapping(self, full_to_swa_index_mapping: torch.Tensor):
        self.full_to_swa_index_mapping = full_to_swa_index_mapping
```
**EN:** Registers metadata so other components can discover this object. It belongs to `DeepSeekV4TokenToKVPool`.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `DeepSeekV4TokenToKVPool`。

### Lines 488-491: get_ring_size implementation / get_ring_size 实现
```python
    def get_ring_size(self, compress_ratio: int) -> int:
        server_args = get_global_server_args()
        is_speculative = server_args.speculative_algorithm is not None
        return get_compress_state_ring_size(compress_ratio, is_speculative)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4TokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会向调用方返回计算结果。

### Lines 493-496: translate_loc_from_full_to_swa implementation / translate_loc_from_full_to_swa 实现
```python
    def translate_loc_from_full_to_swa(self, kv_indices: torch.Tensor):
        assert self.full_to_swa_index_mapping is not None

        return self.full_to_swa_index_mapping[kv_indices].to(torch.int32)
```
**EN:** Implements the translate LOC from full TO SWA routine for this scope. It belongs to `DeepSeekV4TokenToKVPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的translate loc from完整to SWA例程。 该方法属于 `DeepSeekV4TokenToKVPool`。它会向调用方返回计算结果。

### Lines 498-503: set_swa_loc implementation / set_swa_loc 实现
```python
    def set_swa_loc(self, loc: torch.Tensor) -> None:
        # No-op: SWAKVPool's set_swa_loc precomputes SWA-translated loc once per
        # forward batch for set_kv_buffer to read via self.swa_loc. DSV4 has its
        # own equivalent cache via `_should_cache_swa + cached_loc` (in
        # set_swa_key_buffer_radix_fused), so we ignore main's precomputed loc.
        pass
```
**EN:** Updates internal state with the provided value. It belongs to `DeepSeekV4TokenToKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `DeepSeekV4TokenToKVPool`。

### Lines 505-521: get_contiguous_buf_infos implementation / get_contiguous_buf_infos 实现
```python
    def get_contiguous_buf_infos(self) -> Tuple[List[int], List[int], List[int]]:
        data_ptrs: List[int] = []
        data_lens: List[int] = []
        item_lens: List[int] = []

        for bufs in [
            self.c4_kv_pool.kv_buffer,
            self.c4_indexer_kv_pool.index_k_with_scale_buffer,
            self.c128_kv_pool.kv_buffer,
        ]:
            for buf in bufs:
                assert buf.ndim == 2, f"expected 2D buffer, got {buf.ndim}D"
                data_ptrs.append(buf.data_ptr())
                data_lens.append(buf.nbytes)
                item_lens.append(buf[0].nbytes)

        return data_ptrs, data_lens, item_lens
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4TokenToKVPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4TokenToKVPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 523-547: get_state_buf_infos implementation / get_state_buf_infos 实现
```python
    def get_state_buf_infos(self) -> Tuple[List[int], List[int], List[int]]:
        data_ptrs: List[int] = []
        data_lens: List[int] = []
        item_lens: List[int] = []

        for buf in self.swa_kv_pool.kv_buffer:
            assert buf.ndim == 2, f"expected 2D buffer, got {buf.ndim}D"
            data_ptrs.append(buf.data_ptr())
            data_lens.append(buf.nbytes)
            item_lens.append(buf[0].nbytes)

        for pools in [
# ... omitted for brevity ...
                data_lens.append(t.nbytes)
                item_lens.append(t[0].nbytes * pool.ring_size)

        return data_ptrs, data_lens, item_lens
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4TokenToKVPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4TokenToKVPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 549-588: _init_paged_compress_states implementation / _init_paged_compress_states 实现
```python
    def _init_paged_compress_states(self, enable_memory_saver: bool):
        c4_state_pool_size = self.c4_state_pool_size
        c128_state_pool_size = self.c128_state_pool_size
        total_L = len(self.compression_ratios)
        self.compress_state_pools: List[Optional[CompressStatePool]] = [None] * total_L
        self.indexer_compress_state_pools: List[Optional[CompressStatePool]] = [
            None
        ] * total_L

        for idx in range(self._stage_start, self._stage_end):
            ratio = self.compression_ratios[idx]
            if ratio == 0:
# ... omitted for brevity ...
                    dtype=self.state_dtype,
                    enable_memory_saver=enable_memory_saver,
                    ratio=ratio,
                )
```
**EN:** Implements the init paged compress states routine for this scope. It belongs to `DeepSeekV4TokenToKVPool`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的初始化paged压缩states例程。 该方法属于 `DeepSeekV4TokenToKVPool`。实现过程中会遍历输入或受管条目。

### Lines 590-618: _init_compressed_layer_mapping implementation / _init_compressed_layer_mapping 实现
```python
    def _init_compressed_layer_mapping(self):
        c1_cnt = c4_cnt = c128_cnt = 0
        total_L = len(self.compression_ratios)
        self.layer_mapping: List[Optional[DeepSeekV4LayerItem]] = [None] * total_L

        for idx in range(self._stage_start, self._stage_end):
            ratio = self.compression_ratios[idx]
            if ratio == 0:
                self.layer_mapping[idx] = DeepSeekV4LayerItem(
                    compress_ratio=0,
                    compress_layer_id=c1_cnt,
                )
# ... omitted for brevity ...
                )
                c128_cnt += 1
            else:
                raise ValueError(f"Unsupported compression ratio: {ratio}")
```
**EN:** Implements the init compressed layer mapping routine for this scope. It belongs to `DeepSeekV4TokenToKVPool`. It validates error cases explicitly. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的初始化compressed layer mapping例程。 该方法属于 `DeepSeekV4TokenToKVPool`。它会显式处理错误场景。实现过程中会遍历输入或受管条目。

### Lines 620-622: wait_layer_transfer implementation / wait_layer_transfer 实现
```python
    def wait_layer_transfer(self, layer_id: int) -> None:
        if self.layer_transfer_counter is not None:
            self.layer_transfer_counter.wait_until(layer_id - self.start_layer)
```
**EN:** Implements the wait layer transfer routine for this scope. It belongs to `DeepSeekV4TokenToKVPool`.
**CN:** 实现当前作用域中的wait layer transfer例程。 该方法属于 `DeepSeekV4TokenToKVPool`。

### Lines 624-630: get_attention_compress_states implementation / get_attention_compress_states 实现
```python
    def get_attention_compress_states(self, layer_id: int) -> CompressStatePool:
        self.wait_layer_transfer(layer_id)
        compress_state_pool = self.compress_state_pools[layer_id]
        assert (
            compress_state_pool is not None
        ), "Only c4/c128 layers have attention states."
        return compress_state_pool
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4TokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会向调用方返回计算结果。

### Lines 632-638: get_indexer_compress_states implementation / get_indexer_compress_states 实现
```python
    def get_indexer_compress_states(self, layer_id: int) -> CompressStatePool:
        self.wait_layer_transfer(layer_id)
        indexer_compress_state_pool = self.indexer_compress_state_pools[layer_id]
        assert (
            indexer_compress_state_pool is not None
        ), "Only c4 layers have indexer states."
        return indexer_compress_state_pool
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4TokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会向调用方返回计算结果。

### Lines 640-642: _swa_local_layer_id implementation / _swa_local_layer_id 实现
```python
    def _swa_local_layer_id(self, layer_id: int) -> int:
        """Convert absolute model layer_id to SWA-pool-local (PP-stage-local) index."""
        return layer_id - self._stage_start
```
**EN:** Convert absolute model layer_id to SWA-pool-local (PP-stage-local) index. Implements the SWA local layer ID routine for this scope. It belongs to `DeepSeekV4TokenToKVPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的SWA local layer ID例程。 该方法属于 `DeepSeekV4TokenToKVPool`。它会向调用方返回计算结果。

### Lines 644-646: get_swa_key_buffer implementation / get_swa_key_buffer 实现
```python
    def get_swa_key_buffer(self, layer_id: int) -> torch.Tensor:
        self.wait_layer_transfer(layer_id)
        return self.swa_kv_pool.get_key_buffer(self._swa_local_layer_id(layer_id))
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4TokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会向调用方返回计算结果。

### Lines 648-656: set_swa_key_buffer implementation / set_swa_key_buffer 实现
```python
    def set_swa_key_buffer(
        self,
        layer_id: int,
        loc: torch.Tensor,
        cache_nope_fp8_rope_bf16_pack: NopeFp8RopeBf16Pack,
    ) -> None:
        self.swa_kv_pool.set_key_buffer(
            self._swa_local_layer_id(layer_id), loc, cache_nope_fp8_rope_bf16_pack
        )
```
**EN:** Updates internal state with the provided value. It belongs to `DeepSeekV4TokenToKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `DeepSeekV4TokenToKVPool`。

### Lines 658-661: get_extra_key_page_size implementation / get_extra_key_page_size 实现
```python
    def get_extra_key_page_size(self, layer_id: int) -> int:
        _, _, compress_kv_pool = self.layer_mapping[layer_id]
        assert compress_kv_pool is not None
        return compress_kv_pool.page_size
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4TokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会向调用方返回计算结果。

### Lines 663-667: get_extra_key_buffer implementation / get_extra_key_buffer 实现
```python
    def get_extra_key_buffer(self, layer_id: int) -> torch.Tensor | None:
        self.wait_layer_transfer(layer_id)
        _, compress_layer_id, compress_kv_pool = self.layer_mapping[layer_id]
        assert compress_kv_pool is not None
        return compress_kv_pool.get_key_buffer(compress_layer_id)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4TokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会向调用方返回计算结果。

### Lines 669-679: set_extra_key_buffer implementation / set_extra_key_buffer 实现
```python
    def set_extra_key_buffer(
        self,
        layer_id: int,
        loc: torch.Tensor,
        cache_nope_fp8_rope_bf16_pack: NopeFp8RopeBf16Pack,
    ) -> None:
        _, compress_layer_id, compress_kv_pool = self.layer_mapping[layer_id]
        assert compress_kv_pool is not None
        compress_kv_pool.set_key_buffer(
            compress_layer_id, loc, cache_nope_fp8_rope_bf16_pack
        )
```
**EN:** Updates internal state with the provided value. It belongs to `DeepSeekV4TokenToKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `DeepSeekV4TokenToKVPool`。

### Lines 681-682: get_index_k_page_size implementation / get_index_k_page_size 实现
```python
    def get_index_k_page_size(self) -> int:
        return self.c4_indexer_kv_pool.page_size
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4TokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会向调用方返回计算结果。

### Lines 684-688: get_index_k_with_scale_buffer implementation / get_index_k_with_scale_buffer 实现
```python
    def get_index_k_with_scale_buffer(self, layer_id: int) -> torch.Tensor:
        self.wait_layer_transfer(layer_id)
        compress_ratio, compress_layer_id, _ = self.layer_mapping[layer_id]
        assert compress_ratio == 4, f"only c4 has indexer, got {compress_ratio = }"
        return self.c4_indexer_kv_pool.get_index_k_with_scale_buffer(compress_layer_id)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4TokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会向调用方返回计算结果。

### Lines 690-701: get_index_k_scale_buffer implementation / get_index_k_scale_buffer 实现
```python
    def get_index_k_scale_buffer(
        self,
        layer_id: int,
        seq_len: int,
        page_indices: torch.Tensor,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        self.wait_layer_transfer(layer_id)
        compress_ratio, compress_layer_id, _ = self.layer_mapping[layer_id]
        assert compress_ratio == 4, f"only c4 has indexer, got {compress_ratio = }"
        return self.c4_indexer_kv_pool.get_index_k_scale_buffer(
            compress_layer_id, seq_len, page_indices
        )
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4TokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会向调用方返回计算结果。

### Lines 703-714: set_index_k_scale_buffer implementation / set_index_k_scale_buffer 实现
```python
    def set_index_k_scale_buffer(
        self,
        layer_id: int,
        loc: torch.Tensor,
        index_k: torch.Tensor,
        index_k_scale: torch.Tensor,
    ) -> None:
        compress_ratio, compress_layer_id, _ = self.layer_mapping[layer_id]
        assert compress_ratio == 4, f"only c4 has indexer, got {compress_ratio = }"
        self.c4_indexer_kv_pool.set_index_k_scale_buffer(
            compress_layer_id, loc, index_k, index_k_scale
        )
```
**EN:** Updates internal state with the provided value. It belongs to `DeepSeekV4TokenToKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `DeepSeekV4TokenToKVPool`。

### Lines 716-717: get_key_buffer implementation / get_key_buffer 实现
```python
    def get_key_buffer(self, layer_id: int) -> torch.Tensor:
        raise NotImplementedError()
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4TokenToKVPool`. It validates error cases explicitly.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会显式处理错误场景。

### Lines 719-720: get_value_buffer implementation / get_value_buffer 实现
```python
    def get_value_buffer(self, layer_id: int) -> torch.Tensor:
        raise NotImplementedError()
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4TokenToKVPool`. It validates error cases explicitly.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会显式处理错误场景。

### Lines 722-723: get_kv_buffer implementation / get_kv_buffer 实现
```python
    def get_kv_buffer(self, layer_id: int) -> Tuple[torch.Tensor, torch.Tensor]:
        raise NotImplementedError()
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4TokenToKVPool`. It validates error cases explicitly.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会显式处理错误场景。

### Lines 725-726: set_kv_buffer implementation / set_kv_buffer 实现
```python
    def set_kv_buffer(self, *args, **kwargs) -> None:
        raise NotImplementedError()
```
**EN:** Updates internal state with the provided value. It belongs to `DeepSeekV4TokenToKVPool`. It validates error cases explicitly.
**CN:** 使用给定值更新内部状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会显式处理错误场景。

### Lines 728-737: set_swa_key_buffer_radix implementation / set_swa_key_buffer_radix 实现
```python
    def set_swa_key_buffer_radix(
        self,
        layer_id: int,
        raw_loc: torch.Tensor,
        cache_nope_fp8_rope_bf16_pack: NopeFp8RopeBf16Pack,
    ) -> None:
        swa_loc = self.translate_loc_from_full_to_swa(raw_loc)
        self.swa_kv_pool.set_key_buffer(
            self._swa_local_layer_id(layer_id), swa_loc, cache_nope_fp8_rope_bf16_pack
        )
```
**EN:** Updates internal state with the provided value. It belongs to `DeepSeekV4TokenToKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `DeepSeekV4TokenToKVPool`。

### Lines 739-741: get_swa_key_buffer_radix implementation / get_swa_key_buffer_radix 实现
```python
    def get_swa_key_buffer_radix(self, layer_id: int) -> torch.Tensor:
        self.wait_layer_transfer(layer_id)
        return self.swa_kv_pool.get_key_buffer(self._swa_local_layer_id(layer_id))
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4TokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会向调用方返回计算结果。

### Lines 743-757: set_swa_key_buffer_radix_fused implementation / set_swa_key_buffer_radix_fused 实现
```python
    def set_swa_key_buffer_radix_fused(
        self,
        layer_id: int,
        raw_loc: torch.Tensor,
        cache_k: torch.Tensor,
    ) -> None:
        if self._should_cache_swa:
            if layer_id == self.start_layer or self.cached_loc is None:
                self.cached_loc = self.translate_loc_from_full_to_swa(raw_loc)
            swa_loc = self.cached_loc
        else:
            swa_loc = self.translate_loc_from_full_to_swa(raw_loc)
        return self.swa_kv_pool.set_key_buffer_fused(
            self._swa_local_layer_id(layer_id), swa_loc, cache_k
        )
```
**EN:** Updates internal state with the provided value. It belongs to `DeepSeekV4TokenToKVPool`. It returns a computed result to its caller.
**CN:** 使用给定值更新内部状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会向调用方返回计算结果。

### Lines 759-784: set_swa_key_buffer_radix_fused_norm_rope implementation / set_swa_key_buffer_radix_fused_norm_rope 实现
```python
    def set_swa_key_buffer_radix_fused_norm_rope(
        self,
        layer_id: int,
        raw_loc: torch.Tensor,
        kv: torch.Tensor,
        kv_weight: torch.Tensor,
        eps: float,
        freqs_cis: torch.Tensor,
        positions: torch.Tensor,
    ) -> None:
        if self._should_cache_swa:
            if layer_id == self.start_layer or self.cached_loc is None:
# ... omitted for brevity ...
            out_loc=swa_loc,
            kvcache=self.swa_kv_pool.kv_buffer[self._swa_local_layer_id(layer_id)],
            page_size=self.swa_kv_pool.page_size,
        )
```
**EN:** Updates internal state with the provided value. It belongs to `DeepSeekV4TokenToKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `DeepSeekV4TokenToKVPool`。

### Lines 786-794: set_extra_key_buffer_fused implementation / set_extra_key_buffer_fused 实现
```python
    def set_extra_key_buffer_fused(
        self,
        layer_id: int,
        loc: torch.Tensor,
        cache_k: torch.Tensor,
    ) -> None:
        _, compress_layer_id, compress_kv_pool = self.layer_mapping[layer_id]
        assert compress_kv_pool is not None
        return compress_kv_pool.set_key_buffer_fused(compress_layer_id, loc, cache_k)
```
**EN:** Updates internal state with the provided value. It belongs to `DeepSeekV4TokenToKVPool`. It returns a computed result to its caller.
**CN:** 使用给定值更新内部状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会向调用方返回计算结果。

### Lines 796-804: set_index_k_fused implementation / set_index_k_fused 实现
```python
    def set_index_k_fused(
        self,
        layer_id: int,
        loc: torch.Tensor,
        cache_k: torch.Tensor,
    ) -> None:
        compress_ratio, compress_layer_id, _ = self.layer_mapping[layer_id]
        assert compress_ratio == 4, f"only c4 has indexer, got {compress_ratio = }"
        return self.c4_indexer_kv_pool.set_index_fused(compress_layer_id, loc, cache_k)
```
**EN:** Updates internal state with the provided value. It belongs to `DeepSeekV4TokenToKVPool`. It returns a computed result to its caller.
**CN:** 使用给定值更新内部状态。 该方法属于 `DeepSeekV4TokenToKVPool`。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`get_compress_state_ring_size`**: Provides the `get_compress_state_ring_size` entry point for module-level behavior. / 提供模块级行为的 `get_compress_state_ring_size` 入口。
- **`DeepSeekV4SingleKVPool`**: Defines the `DeepSeekV4SingleKVPool` type and its core responsibilities. / 定义 `DeepSeekV4SingleKVPool` 类型及其核心职责。
- **`HiSparseC4DevicePool`**: Defines the `HiSparseC4DevicePool` type and its core responsibilities. / 定义 `HiSparseC4DevicePool` 类型及其核心职责。
- **`DeepSeekV4IndexerPool`**: Defines the `DeepSeekV4IndexerPool` type and its core responsibilities. / 定义 `DeepSeekV4IndexerPool` 类型及其核心职责。
- **`DeepSeekV4LayerItem`**: Defines the `DeepSeekV4LayerItem` type and its core responsibilities. / 定义 `DeepSeekV4LayerItem` 类型及其核心职责。
- **`DeepSeekV4TokenToKVPool`**: Defines the `DeepSeekV4TokenToKVPool` type and its core responsibilities. / 定义 `DeepSeekV4TokenToKVPool` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `contextlib`, `typing`, `torch`
- **Internal / 内部**: `sglang.jit_kernel.deepseek_v4`, `sglang.srt.constants`, `sglang.srt.environ`, `sglang.srt.layers.attention.dsv4`, `sglang.srt.layers.attention.dsv4.index_buf_accessor`, `sglang.srt.layers.attention.nsa`, `sglang.srt.mem_cache.base_swa_memory_pool`, `sglang.srt.mem_cache.deepseek_v4_compress_state`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.server_args`, `sglang.srt.utils`
