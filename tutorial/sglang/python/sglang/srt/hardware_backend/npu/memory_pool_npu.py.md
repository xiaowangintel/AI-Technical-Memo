# memory_pool_npu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/memory_pool_npu.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for memory pooling inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的内存池管理支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Module setup and shared state / 模块设置与共享状态
```python
from typing import TYPE_CHECKING, Optional

import torch

from sglang.srt.constants import GPU_MEMORY_TYPE_KV_CACHE
from sglang.srt.mem_cache.memory_pool import (
    MHATokenToKVPool,
    MLATokenToKVPool,
    get_tensor_size_bytes,
)
from sglang.srt.utils import get_bool_env_var
from sglang.srt.utils.common import is_npu

if TYPE_CHECKING:
    from sglang.srt.layers.radix_attention import RadixAttention

if is_npu():
    import torch_npu
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `typing`, `torch`, `sglang.srt.constants`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.utils`, `sglang.srt.utils.common`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `typing`, `torch`, `sglang.srt.constants`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.utils`, `sglang.srt.utils.common`。

### Lines 21-42: Function `_init_npu_conv_state` / 函数 `_init_npu_conv_state`
```python
def _init_npu_conv_state(
    conv_state_in, conv_state_shape, speculative_num_draft_tokens: Optional[int] = None
):
    extra_conv_len = 0
    if speculative_num_draft_tokens is not None:
        extra_conv_len = speculative_num_draft_tokens - 1

    # conv_state shape (layers, pool_size, conv_wind + draft_step, dim) for conv1d ascendc ops require dim as last dim
    conv_state = [
        torch.zeros(
            size=(
                conv_state_in.shape[0],
                conv_state_in.shape[1],
                conv_shape[1] + extra_conv_len,
                conv_shape[0],
            ),
            dtype=conv_state_in.dtype,
            device=conv_state_in.device,
        )
        for conv_shape in conv_state_shape
    ]
    return conv_state
```
**EN:** This function implements `_init_npu_conv_state`. It primarily calls `torch.zeros` to complete its work. State updates are written into `extra_conv_len`, `conv_state`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_init_npu_conv_state`。 它主要通过调用 `torch.zeros` 来完成任务。 状态更新主要写入 `extra_conv_len`, `conv_state`。 实现中使用了条件分支。

### Lines 45-45: Class `NPUMHATokenToKVPool` declaration / 类 `NPUMHATokenToKVPool` 声明
```python
class NPUMHATokenToKVPool(MHATokenToKVPool):
```
**EN:** This class establishes `NPUMHATokenToKVPool` as the main container/coordinator for the surrounding logic. It inherits from `MHATokenToKVPool`. Its core interface includes methods such as `__init__`, `_create_buffers`, `get_contiguous_buf_infos`, `set_kv_buffer`, `_chunk_copy_npu_to_cpu`, `get_cpu_copy`.
**CN:** 该类将 `NPUMHATokenToKVPool` 定义为周边逻辑的主要封装体或协调者。 它继承自 `MHATokenToKVPool`。 其核心接口包括 `__init__`, `_create_buffers`, `get_contiguous_buf_infos`, `set_kv_buffer`, `_chunk_copy_npu_to_cpu`, `get_cpu_copy` 等方法。

### Lines 47-76: Method `NPUMHATokenToKVPool.__init__` / 方法 `NPUMHATokenToKVPool.__init__`
```python
    def __init__(
        self,
        size: int,
        page_size: int,
        dtype: torch.dtype,
        head_num: int,
        head_dim: int,
        layer_num: int,
        device: str,
        enable_memory_saver: bool,
        start_layer: Optional[int] = None,
        end_layer: Optional[int] = None,
        enable_alt_stream: bool = True,
        enable_kv_cache_copy: bool = False,
    ):
        self.use_fia = get_bool_env_var("ASCEND_USE_FIA", "False")
        super().__init__(
            size=size,
            page_size=page_size,
            dtype=dtype,
            head_num=head_num,
            head_dim=head_dim,
            layer_num=layer_num,
            device=device,
            enable_memory_saver=enable_memory_saver,
            start_layer=start_layer,
            end_layer=end_layer,
            enable_alt_stream=enable_alt_stream,
            enable_kv_cache_copy=enable_kv_cache_copy,
        )
```
**EN:** This method implements `__init__` on `NPUMHATokenToKVPool`. It primarily calls `get_bool_env_var`, `super.__init__`, `super` to complete its work. State updates are written into `self.use_fia`.
**CN:** 该方法（属于 `NPUMHATokenToKVPool`）实现了 `__init__`。 它主要通过调用 `get_bool_env_var`, `super.__init__`, `super` 来完成任务。 状态更新主要写入 `self.use_fia`。

### Lines 78-110: Method `NPUMHATokenToKVPool._create_buffers` / 方法 `NPUMHATokenToKVPool._create_buffers`
```python
    def _create_buffers(self):
        with self.memory_saver_adapter.region(GPU_MEMORY_TYPE_KV_CACHE):
            # [size, head_num, head_dim] for each layer
            # The padded slot 0 is used for writing dummy outputs from padded tokens.
            # Continuous memory improves the efficiency of Ascend`s transmission backend,
            # while other backends remain unchanged.
            self.kv_buffer = torch.zeros(
                (
                    2,
                    self.layer_num,
                    self.size // self.page_size + 1,
                    self.page_size,
                    self.head_num,
                    self.head_dim,
                ),
                dtype=self.store_dtype,
                device=self.device,
            )
            self.k_buffer = self.kv_buffer[0]
            self.v_buffer = self.kv_buffer[1]

            if self.use_fia:
                self.k_buffer = []
                self.v_buffer = []
                for i in range(self.layer_num):
                    k_buffer_layer = self.kv_buffer[0][i].view(
                        -1, 1, self.head_num, self.head_dim
                    )
                    v_buffer_layer = self.kv_buffer[1][i].view(
                        -1, 1, self.head_num, self.head_dim
                    )
                    self.k_buffer.append(k_buffer_layer)
                    self.v_buffer.append(v_buffer_layer)
```
**EN:** This method implements `_create_buffers` on `NPUMHATokenToKVPool`. It primarily calls `self.memory_saver_adapter.region`, `torch.zeros`, `range`, `self.kv_buffer.view`, `self.k_buffer.append`, `self.v_buffer.append` to complete its work. State updates are written into `self.kv_buffer`, `self.k_buffer`, `self.v_buffer`, `k_buffer_layer`, `v_buffer_layer`. The implementation relies on conditional branches, iteration, context-managed resources.
**CN:** 该方法（属于 `NPUMHATokenToKVPool`）实现了 `_create_buffers`。 它主要通过调用 `self.memory_saver_adapter.region`, `torch.zeros`, `range`, `self.kv_buffer.view`, `self.k_buffer.append`, `self.v_buffer.append` 来完成任务。 状态更新主要写入 `self.kv_buffer`, `self.k_buffer`, `self.v_buffer`, `k_buffer_layer`, `v_buffer_layer`。 实现中使用了条件分支、迭代逻辑、上下文管理资源。

### Lines 113-146: Method `NPUMHATokenToKVPool.get_contiguous_buf_infos` / 方法 `NPUMHATokenToKVPool.get_contiguous_buf_infos`
```python
    def get_contiguous_buf_infos(self):
        # layer_num x [seq_len, head_num, head_dim]
        # layer_num x [page_num, page_size, head_num, head_dim]
        kv_data_ptrs = [
            self.get_key_buffer(i).data_ptr()
            for i in range(self.start_layer, self.start_layer + self.layer_num)
        ] + [
            self.get_value_buffer(i).data_ptr()
            for i in range(self.start_layer, self.start_layer + self.layer_num)
        ]
        kv_data_lens = [
            self.get_key_buffer(i).nbytes
            for i in range(self.start_layer, self.start_layer + self.layer_num)
        ] + [
            self.get_value_buffer(i).nbytes
            for i in range(self.start_layer, self.start_layer + self.layer_num)
        ]
        if self.use_fia:
            kv_item_lens = [
                self.get_key_buffer(i)[0].nbytes * self.page_size
                for i in range(self.start_layer, self.start_layer + self.layer_num)
            ] + [
                self.get_value_buffer(i)[0].nbytes * self.page_size
                for i in range(self.start_layer, self.start_layer + self.layer_num)
            ]
        else:
            kv_item_lens = [
                self.get_key_buffer(i)[0].nbytes
                for i in range(self.start_layer, self.start_layer + self.layer_num)
            ] + [
                self.get_value_buffer(i)[0].nbytes
                for i in range(self.start_layer, self.start_layer + self.layer_num)
            ]
        return kv_data_ptrs, kv_data_lens, kv_item_lens
```
**EN:** This method implements `get_contiguous_buf_infos` on `NPUMHATokenToKVPool`. It primarily calls `self.get_key_buffer.data_ptr`, `self.get_value_buffer.data_ptr`, `range`, `self.get_key_buffer`, `self.get_value_buffer` to complete its work. State updates are written into `kv_data_ptrs`, `kv_data_lens`, `kv_item_lens`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUMHATokenToKVPool`）实现了 `get_contiguous_buf_infos`。 它主要通过调用 `self.get_key_buffer.data_ptr`, `self.get_value_buffer.data_ptr`, `range`, `self.get_key_buffer`, `self.get_value_buffer` 来完成任务。 状态更新主要写入 `kv_data_ptrs`, `kv_data_lens`, `kv_item_lens`。 实现中使用了条件分支。

### Lines 148-200: Method `NPUMHATokenToKVPool.set_kv_buffer` / 方法 `NPUMHATokenToKVPool.set_kv_buffer`
```python
    def set_kv_buffer(
        self,
        layer: "RadixAttention",
        loc: torch.Tensor,
        cache_k: torch.Tensor,
        cache_v: torch.Tensor,
        k_scale: Optional[float] = None,
        v_scale: Optional[float] = None,
        layer_id_override: Optional[int] = None,
    ):
        if layer_id_override is not None:
            layer_id = layer_id_override
        else:
            layer_id = layer.layer_id
        if cache_k.dtype != self.dtype:
            if k_scale is not None:
                cache_k.div_(k_scale)
            if v_scale is not None:
                cache_v.div_(v_scale)
            cache_k = cache_k.to(self.dtype)
            cache_v = cache_v.to(self.dtype)

        if self.store_dtype != self.dtype:
            cache_k = cache_k.view(self.store_dtype)
            cache_v = cache_v.view(self.store_dtype)

        if self.use_fia:
            k_buffer_layer = self.k_buffer[layer_id - self.start_layer]
# ... omitted for brevity ...
                ),
                value_cache=self.v_buffer[layer_id - self.start_layer].view(
                    -1, self.page_size, self.head_num, self.head_dim
                ),
                slot_indices=loc,
            )
```
**EN:** This method implements `set_kv_buffer` on `NPUMHATokenToKVPool`. It primarily calls `cache_k.to`, `cache_v.to`, `cache_k.view`, `cache_v.view`, `torch_npu.npu_scatter_nd_update_`, `loc.to` to complete its work. State updates are written into `layer_id`, `cache_k`, `cache_v`, `k_buffer_layer`, `v_buffer_layer`, `loc`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUMHATokenToKVPool`）实现了 `set_kv_buffer`。 它主要通过调用 `cache_k.to`, `cache_v.to`, `cache_k.view`, `cache_v.view`, `torch_npu.npu_scatter_nd_update_`, `loc.to` 来完成任务。 状态更新主要写入 `layer_id`, `cache_k`, `cache_v`, `k_buffer_layer`, `v_buffer_layer`, `loc`。 实现中使用了条件分支。

### Lines 202-217: Method `NPUMHATokenToKVPool._chunk_copy_npu_to_cpu` / 方法 `NPUMHATokenToKVPool._chunk_copy_npu_to_cpu`
```python
    def _chunk_copy_npu_to_cpu(self, buf_of_layers, indices):
        chunk_size = self.cpu_offloading_chunk_size
        out = []
        for tensors_per_layer in buf_of_layers:  # [k_buf, v_buf]
            layer_chunks = []
            for i in range(0, len(indices), chunk_size):
                ci = indices[i : i + chunk_size]
                layer_chunks.append(
                    [
                        t[ci].to("cpu", non_blocking=True)
                        for t in tensors_per_layer
                        if t is not None
                    ]
                )
            out.append(layer_chunks)
        return out
```
**EN:** This method implements `_chunk_copy_npu_to_cpu` on `NPUMHATokenToKVPool`. It primarily calls `range`, `out.append`, `len`, `layer_chunks.append`, `t.to` to complete its work. State updates are written into `chunk_size`, `out`, `layer_chunks`, `ci`. The implementation relies on iteration.
**CN:** 该方法（属于 `NPUMHATokenToKVPool`）实现了 `_chunk_copy_npu_to_cpu`。 它主要通过调用 `range`, `out.append`, `len`, `layer_chunks.append`, `t.to` 来完成任务。 状态更新主要写入 `chunk_size`, `out`, `layer_chunks`, `ci`。 实现中使用了迭代逻辑。

### Lines 224-237: Method `NPUMHATokenToKVPool.get_cpu_copy` / 方法 `NPUMHATokenToKVPool.get_cpu_copy`
```python
    def get_cpu_copy(self, indices):
        torch.npu.synchronize()
        buf_of_layers = []
        for local_layer_id in range(self.layer_num):
            k_layer = self.k_buffer[local_layer_id].view(
                -1, self.head_num, self.head_dim
            )
            v_layer = self.v_buffer[local_layer_id].view(
                -1, self.head_num, self.head_dim
            )
            buf_of_layers.append([k_layer, v_layer])
        kv_cache_cpu = self._chunk_copy_npu_to_cpu(buf_of_layers, indices)
        torch.npu.synchronize()
        return kv_cache_cpu
```
**EN:** This method implements `get_cpu_copy` on `NPUMHATokenToKVPool`. It primarily calls `torch.npu.synchronize`, `range`, `self._chunk_copy_npu_to_cpu`, `self.k_buffer.view`, `self.v_buffer.view`, `buf_of_layers.append` to complete its work. State updates are written into `buf_of_layers`, `kv_cache_cpu`, `k_layer`, `v_layer`. The implementation relies on iteration.
**CN:** 该方法（属于 `NPUMHATokenToKVPool`）实现了 `get_cpu_copy`。 它主要通过调用 `torch.npu.synchronize`, `range`, `self._chunk_copy_npu_to_cpu`, `self.k_buffer.view`, `self.v_buffer.view`, `buf_of_layers.append` 来完成任务。 状态更新主要写入 `buf_of_layers`, `kv_cache_cpu`, `k_layer`, `v_layer`。 实现中使用了迭代逻辑。

### Lines 239-258: Method `NPUMHATokenToKVPool.load_cpu_copy` / 方法 `NPUMHATokenToKVPool.load_cpu_copy`
```python
    def load_cpu_copy(self, kv_cache_cpu, indices):
        torch.npu.synchronize()
        chunk_size = self.cpu_offloading_chunk_size
        for local_layer_id in range(self.layer_num):
            k_layer = self.k_buffer[local_layer_id].view(
                -1, self.head_num, self.head_dim
            )
            v_layer = self.v_buffer[local_layer_id].view(
                -1, self.head_num, self.head_dim
            )
            for i in range(0, len(indices), chunk_size):
                chunk_indices = indices[i : i + chunk_size]
                k_cpu, v_cpu = (
                    kv_cache_cpu[local_layer_id][i // chunk_size][0],
                    kv_cache_cpu[local_layer_id][i // chunk_size][1],
                )
                assert k_cpu.shape[0] == v_cpu.shape[0] == len(chunk_indices)
                k_layer[chunk_indices] = k_cpu.to(k_layer.device, non_blocking=True)
                v_layer[chunk_indices] = v_cpu.to(v_layer.device, non_blocking=True)
        torch.npu.synchronize()
```
**EN:** This method implements `load_cpu_copy` on `NPUMHATokenToKVPool`. It primarily calls `torch.npu.synchronize`, `range`, `self.k_buffer.view`, `self.v_buffer.view`, `len`, `k_cpu.to` to complete its work. State updates are written into `chunk_size`, `k_layer`, `v_layer`, `chunk_indices`. The implementation relies on iteration.
**CN:** 该方法（属于 `NPUMHATokenToKVPool`）实现了 `load_cpu_copy`。 它主要通过调用 `torch.npu.synchronize`, `range`, `self.k_buffer.view`, `self.v_buffer.view`, `len`, `k_cpu.to` 来完成任务。 状态更新主要写入 `chunk_size`, `k_layer`, `v_layer`, `chunk_indices`。 实现中使用了迭代逻辑。

### Lines 261-261: Class `NPUMLATokenToKVPool` declaration / 类 `NPUMLATokenToKVPool` 声明
```python
class NPUMLATokenToKVPool(MLATokenToKVPool):
```
**EN:** This class establishes `NPUMLATokenToKVPool` as the main container/coordinator for the surrounding logic. It inherits from `MLATokenToKVPool`. Its core interface includes methods such as `__init__`, `get_kv_size_bytes`, `get_kv_buffer`, `get_state_buf_infos`, `get_key_buffer`, `get_value_buffer`.
**CN:** 该类将 `NPUMLATokenToKVPool` 定义为周边逻辑的主要封装体或协调者。 它继承自 `MLATokenToKVPool`。 其核心接口包括 `__init__`, `get_kv_size_bytes`, `get_kv_buffer`, `get_state_buf_infos`, `get_key_buffer`, `get_value_buffer` 等方法。

### Lines 263-332: Method `NPUMLATokenToKVPool.__init__` / 方法 `NPUMLATokenToKVPool.__init__`
```python
    def __init__(
        self,
        size: int,
        page_size: int,
        dtype: torch.dtype,
        kv_lora_rank: int,
        qk_rope_head_dim: int,
        index_head_dim: Optional[int],
        layer_num: int,
        device: str,
        enable_memory_saver: bool,
        start_layer: Optional[int] = None,
        end_layer: Optional[int] = None,
    ):
        super(MLATokenToKVPool, self).__init__(
            size=size,
            page_size=page_size,
            dtype=dtype,
            layer_num=layer_num,
            device=device,
            enable_memory_saver=enable_memory_saver,
            start_layer=start_layer,
            end_layer=end_layer,
        )

        self.kv_lora_rank = kv_lora_rank
        self.qk_rope_head_dim = qk_rope_head_dim
        self.index_head_dim = index_head_dim
# ... omitted for brevity ...
                    ),
                    dtype=self.store_dtype,
                    device=self.device,
                )

        self._finalize_allocation_log(size)
```
**EN:** This method implements `__init__` on `NPUMLATokenToKVPool`. It primarily calls `super.__init__`, `self._finalize_allocation_log`, `self.memory_saver_adapter.region`, `torch.zeros`, `super` to complete its work. State updates are written into `self.kv_lora_rank`, `self.qk_rope_head_dim`, `self.index_head_dim`, `self.custom_mem_pool`, `self.k_buffer`, `self.v_buffer`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该方法（属于 `NPUMLATokenToKVPool`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `self._finalize_allocation_log`, `self.memory_saver_adapter.region`, `torch.zeros`, `super` 来完成任务。 状态更新主要写入 `self.kv_lora_rank`, `self.qk_rope_head_dim`, `self.index_head_dim`, `self.custom_mem_pool`, `self.k_buffer`, `self.v_buffer`。 实现中使用了条件分支、上下文管理资源。

### Lines 334-346: Method `NPUMLATokenToKVPool.get_kv_size_bytes` / 方法 `NPUMLATokenToKVPool.get_kv_size_bytes`
```python
    def get_kv_size_bytes(self):
        assert hasattr(self, "k_buffer")
        assert hasattr(self, "v_buffer")
        kv_size_bytes = 0
        for k_cache in self.k_buffer:
            kv_size_bytes += get_tensor_size_bytes(k_cache)
        for v_cache in self.v_buffer:
            kv_size_bytes += get_tensor_size_bytes(v_cache)
        if self.index_head_dim is not None:
            assert hasattr(self, "index_k_buffer")
            for index_k_cache in self.index_k_buffer:
                kv_size_bytes += get_tensor_size_bytes(index_k_cache)
        return kv_size_bytes
```
**EN:** This method implements `get_kv_size_bytes` on `NPUMLATokenToKVPool`. It primarily calls `hasattr`, `get_tensor_size_bytes` to complete its work. State updates are written into `kv_size_bytes`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `NPUMLATokenToKVPool`）实现了 `get_kv_size_bytes`。 它主要通过调用 `hasattr`, `get_tensor_size_bytes` 来完成任务。 状态更新主要写入 `kv_size_bytes`。 实现中使用了条件分支、迭代逻辑。

### Lines 348-354: Method `NPUMLATokenToKVPool.get_kv_buffer` / 方法 `NPUMLATokenToKVPool.get_kv_buffer`
```python
    def get_kv_buffer(self, layer_id: int):
        if self.layer_transfer_counter is not None:
            self.layer_transfer_counter.wait_until(layer_id - self.start_layer)
        return (
            self.k_buffer[layer_id - self.start_layer],
            self.v_buffer[layer_id - self.start_layer],
        )
```
**EN:** This method implements `get_kv_buffer` on `NPUMLATokenToKVPool`. It primarily calls `self.layer_transfer_counter.wait_until` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUMLATokenToKVPool`）实现了 `get_kv_buffer`。 它主要通过调用 `self.layer_transfer_counter.wait_until` 来完成任务。 实现中使用了条件分支。

### Lines 356-360: Method `NPUMLATokenToKVPool.get_state_buf_infos` / 方法 `NPUMLATokenToKVPool.get_state_buf_infos`
```python
    def get_state_buf_infos(self):
        data_ptrs = [self.index_k_buffer[i].data_ptr() for i in range(self.layer_num)]
        data_lens = [self.index_k_buffer[i].nbytes for i in range(self.layer_num)]
        item_lens = [self.index_k_buffer[i][0].nbytes for i in range(self.layer_num)]
        return data_ptrs, data_lens, item_lens
```
**EN:** This method implements `get_state_buf_infos` on `NPUMLATokenToKVPool`. It primarily calls `self.index_k_buffer.data_ptr`, `range` to complete its work. State updates are written into `data_ptrs`, `data_lens`, `item_lens`.
**CN:** 该方法（属于 `NPUMLATokenToKVPool`）实现了 `get_state_buf_infos`。 它主要通过调用 `self.index_k_buffer.data_ptr`, `range` 来完成任务。 状态更新主要写入 `data_ptrs`, `data_lens`, `item_lens`。

### Lines 362-368: Method `NPUMLATokenToKVPool.get_key_buffer` / 方法 `NPUMLATokenToKVPool.get_key_buffer`
```python
    def get_key_buffer(self, layer_id: int):
        if self.layer_transfer_counter is not None:
            self.layer_transfer_counter.wait_until(layer_id - self.start_layer)

        if self.store_dtype != self.dtype:
            return self.k_buffer[layer_id - self.start_layer].view(self.dtype)
        return self.k_buffer[layer_id - self.start_layer]
```
**EN:** This method implements `get_key_buffer` on `NPUMLATokenToKVPool`. It primarily calls `self.layer_transfer_counter.wait_until`, `self.k_buffer.view` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUMLATokenToKVPool`）实现了 `get_key_buffer`。 它主要通过调用 `self.layer_transfer_counter.wait_until`, `self.k_buffer.view` 来完成任务。 实现中使用了条件分支。

### Lines 370-376: Method `NPUMLATokenToKVPool.get_value_buffer` / 方法 `NPUMLATokenToKVPool.get_value_buffer`
```python
    def get_value_buffer(self, layer_id: int):
        if self.layer_transfer_counter is not None:
            self.layer_transfer_counter.wait_until(layer_id - self.start_layer)

        if self.store_dtype != self.dtype:
            return self.v_buffer[layer_id - self.start_layer].view(self.dtype)
        return self.v_buffer[layer_id - self.start_layer]
```
**EN:** This method implements `get_value_buffer` on `NPUMLATokenToKVPool`. It primarily calls `self.layer_transfer_counter.wait_until`, `self.v_buffer.view` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUMLATokenToKVPool`）实现了 `get_value_buffer`。 它主要通过调用 `self.layer_transfer_counter.wait_until`, `self.v_buffer.view` 来完成任务。 实现中使用了条件分支。

### Lines 378-384: Method `NPUMLATokenToKVPool.get_index_k_buffer` / 方法 `NPUMLATokenToKVPool.get_index_k_buffer`
```python
    def get_index_k_buffer(self, layer_id: int):
        if self.layer_transfer_counter is not None:
            self.layer_transfer_counter.wait_until(layer_id - self.start_layer)

        if self.store_dtype != self.dtype:
            return self.index_k_buffer[layer_id - self.start_layer].view(self.dtype)
        return self.index_k_buffer[layer_id - self.start_layer]
```
**EN:** This method implements `get_index_k_buffer` on `NPUMLATokenToKVPool`. It primarily calls `self.layer_transfer_counter.wait_until`, `self.index_k_buffer.view` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUMLATokenToKVPool`）实现了 `get_index_k_buffer`。 它主要通过调用 `self.layer_transfer_counter.wait_until`, `self.index_k_buffer.view` 来完成任务。 实现中使用了条件分支。

### Lines 387-408: Method `NPUMLATokenToKVPool.get_contiguous_buf_infos` / 方法 `NPUMLATokenToKVPool.get_contiguous_buf_infos`
```python
    def get_contiguous_buf_infos(self):
        # MLA has only one kv_buffer, so only the information of this buffer needs to be returned.
        kv_data_ptrs = [self.k_buffer[i].data_ptr() for i in range(self.layer_num)] + [
            self.v_buffer[i].data_ptr() for i in range(self.layer_num)
        ]
        kv_data_lens = [self.k_buffer[i].nbytes for i in range(self.layer_num)] + [
            self.v_buffer[i].nbytes for i in range(self.layer_num)
        ]
        kv_item_lens = [self.k_buffer[i][0].nbytes for i in range(self.layer_num)] + [
            self.v_buffer[i][0].nbytes for i in range(self.layer_num)
        ]
        if self.index_head_dim is not None:
            kv_data_ptrs += [
                self.index_k_buffer[i].data_ptr() for i in range(self.layer_num)
            ]
            kv_data_lens += [
                self.index_k_buffer[i].nbytes for i in range(self.layer_num)
            ]
            kv_item_lens += [
                self.index_k_buffer[i][0].nbytes for i in range(self.layer_num)
            ]
        return kv_data_ptrs, kv_data_lens, kv_item_lens
```
**EN:** This method implements `get_contiguous_buf_infos` on `NPUMLATokenToKVPool`. It primarily calls `self.k_buffer.data_ptr`, `self.v_buffer.data_ptr`, `self.index_k_buffer.data_ptr`, `range` to complete its work. State updates are written into `kv_data_ptrs`, `kv_data_lens`, `kv_item_lens`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUMLATokenToKVPool`）实现了 `get_contiguous_buf_infos`。 它主要通过调用 `self.k_buffer.data_ptr`, `self.v_buffer.data_ptr`, `self.index_k_buffer.data_ptr`, `range` 来完成任务。 状态更新主要写入 `kv_data_ptrs`, `kv_data_lens`, `kv_item_lens`。 实现中使用了条件分支。

### Lines 410-442: Method `NPUMLATokenToKVPool.set_kv_buffer` / 方法 `NPUMLATokenToKVPool.set_kv_buffer`
```python
    def set_kv_buffer(
        self,
        layer: "RadixAttention",
        loc: torch.Tensor,
        cache_k: torch.Tensor,
        cache_v: torch.Tensor,
    ):
        layer_id = layer.layer_id
        if cache_k.dtype != self.dtype:
            cache_k = cache_k.to(self.dtype)
            cache_v = cache_v.to(self.dtype)

        if self.store_dtype != self.dtype:
            cache_k = cache_k.view(self.store_dtype)
            cache_v = cache_v.view(self.store_dtype)

        if cache_v is None:
            cache_k, cache_v = cache_k.split(
                [self.kv_lora_rank, self.qk_rope_head_dim], dim=-1
            )

        torch_npu.npu_scatter_nd_update_(
            self.k_buffer[layer_id - self.start_layer].view(-1, 1, self.kv_lora_rank),
            loc.view(-1, 1),
            cache_k.view(-1, 1, self.kv_lora_rank),
        )
        torch_npu.npu_scatter_nd_update_(
            self.v_buffer[layer_id - self.start_layer].view(
                -1, 1, self.qk_rope_head_dim
            ),
            loc.view(-1, 1),
            cache_v.view(-1, 1, self.qk_rope_head_dim),
        )
```
**EN:** This method implements `set_kv_buffer` on `NPUMLATokenToKVPool`. It primarily calls `torch_npu.npu_scatter_nd_update_`, `cache_k.to`, `cache_v.to`, `cache_k.view`, `cache_v.view`, `cache_k.split` to complete its work. State updates are written into `layer_id`, `cache_k`, `cache_v`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUMLATokenToKVPool`）实现了 `set_kv_buffer`。 它主要通过调用 `torch_npu.npu_scatter_nd_update_`, `cache_k.to`, `cache_v.to`, `cache_k.view`, `cache_v.view`, `cache_k.split` 来完成任务。 状态更新主要写入 `layer_id`, `cache_k`, `cache_v`。 实现中使用了条件分支。

### Lines 444-462: Method `NPUMLATokenToKVPool.set_index_k_buffer` / 方法 `NPUMLATokenToKVPool.set_index_k_buffer`
```python
    def set_index_k_buffer(
        self,
        layer_id: int,
        loc: torch.Tensor,
        index_k: torch.Tensor,
    ):
        if index_k.dtype != self.dtype:
            index_k = index_k.to(self.dtype)

        if self.store_dtype != self.dtype:
            index_k = index_k.view(self.store_dtype)

        torch_npu.npu_scatter_nd_update_(
            self.index_k_buffer[layer_id - self.start_layer].view(
                -1, 1, self.index_head_dim
            ),
            loc.view(-1, 1),
            index_k.view(-1, 1, self.index_head_dim),
        )
```
**EN:** This method implements `set_index_k_buffer` on `NPUMLATokenToKVPool`. It primarily calls `torch_npu.npu_scatter_nd_update_`, `index_k.to`, `index_k.view`, `self.index_k_buffer.view`, `loc.view` to complete its work. State updates are written into `index_k`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUMLATokenToKVPool`）实现了 `set_index_k_buffer`。 它主要通过调用 `torch_npu.npu_scatter_nd_update_`, `index_k.to`, `index_k.view`, `self.index_k_buffer.view`, `loc.view` 来完成任务。 状态更新主要写入 `index_k`。 实现中使用了条件分支。

### Lines 464-479: Method `NPUMLATokenToKVPool._chunk_copy_npu_to_cpu` / 方法 `NPUMLATokenToKVPool._chunk_copy_npu_to_cpu`
```python
    def _chunk_copy_npu_to_cpu(self, buf_of_layers, indices):
        chunk_size = self.cpu_offloading_chunk_size
        out = []
        for tensors_per_layer in buf_of_layers:  # [k_buf, v_buf, ik_buf/None]
            layer_chunks = []
            for i in range(0, len(indices), chunk_size):
                ci = indices[i : i + chunk_size]
                layer_chunks.append(
                    [
                        t[ci].to("cpu", non_blocking=True)
                        for t in tensors_per_layer
                        if t is not None
                    ]
                )
            out.append(layer_chunks)
        return out
```
**EN:** This method implements `_chunk_copy_npu_to_cpu` on `NPUMLATokenToKVPool`. It primarily calls `range`, `out.append`, `len`, `layer_chunks.append`, `t.to` to complete its work. State updates are written into `chunk_size`, `out`, `layer_chunks`, `ci`. The implementation relies on iteration.
**CN:** 该方法（属于 `NPUMLATokenToKVPool`）实现了 `_chunk_copy_npu_to_cpu`。 它主要通过调用 `range`, `out.append`, `len`, `layer_chunks.append`, `t.to` 来完成任务。 状态更新主要写入 `chunk_size`, `out`, `layer_chunks`, `ci`。 实现中使用了迭代逻辑。

### Lines 481-497: Method `NPUMLATokenToKVPool.get_cpu_copy` / 方法 `NPUMLATokenToKVPool.get_cpu_copy`
```python
    def get_cpu_copy(self, indices):
        torch.npu.synchronize()
        buf_of_layers = []
        has_ik = self.index_head_dim is not None
        for local_layer_id in range(self.layer_num):
            k_layer = self.k_buffer[local_layer_id].view(-1, 1, self.kv_lora_rank)
            v_layer = self.v_buffer[local_layer_id].view(-1, 1, self.qk_rope_head_dim)
            ik_layer = (
                self.index_k_buffer[local_layer_id].view(-1, 1, self.index_head_dim)
                if has_ik
                else None
            )
            buf_of_layers.append([k_layer, v_layer, ik_layer])

        kv_cache_cpu = self._chunk_copy_npu_to_cpu(buf_of_layers, indices)
        torch.npu.synchronize()
        return kv_cache_cpu
```
**EN:** This method implements `get_cpu_copy` on `NPUMLATokenToKVPool`. It primarily calls `torch.npu.synchronize`, `range`, `self._chunk_copy_npu_to_cpu`, `self.k_buffer.view`, `self.v_buffer.view`, `buf_of_layers.append` to complete its work. State updates are written into `buf_of_layers`, `has_ik`, `kv_cache_cpu`, `k_layer`, `v_layer`, `ik_layer`. The implementation relies on iteration.
**CN:** 该方法（属于 `NPUMLATokenToKVPool`）实现了 `get_cpu_copy`。 它主要通过调用 `torch.npu.synchronize`, `range`, `self._chunk_copy_npu_to_cpu`, `self.k_buffer.view`, `self.v_buffer.view`, `buf_of_layers.append` 来完成任务。 状态更新主要写入 `buf_of_layers`, `has_ik`, `kv_cache_cpu`, `k_layer`, `v_layer`, `ik_layer`。 实现中使用了迭代逻辑。

### Lines 499-523: Method `NPUMLATokenToKVPool.load_cpu_copy` / 方法 `NPUMLATokenToKVPool.load_cpu_copy`
```python
    def load_cpu_copy(self, kv_cache_cpu, indices):
        torch.npu.synchronize()
        chunk_size = self.cpu_offloading_chunk_size
        has_ik = self.index_head_dim is not None
        for local_layer_id in range(self.layer_num):
            k_layer = self.k_buffer[local_layer_id].view(-1, 1, self.kv_lora_rank)
            v_layer = self.v_buffer[local_layer_id].view(-1, 1, self.qk_rope_head_dim)
            ik_layer = (
                self.index_k_buffer[local_layer_id].view(-1, 1, self.index_head_dim)
                if has_ik
                else None
            )
            for i in range(0, len(indices), chunk_size):
                chunk_indices = indices[i : i + chunk_size]
                chunk = kv_cache_cpu[local_layer_id][i // chunk_size]
                k_cpu, v_cpu = chunk[0], chunk[1]
                assert k_cpu.shape[0] == len(chunk_indices)
                k_layer[chunk_indices] = k_cpu.to(k_layer.device, non_blocking=True)
                v_layer[chunk_indices] = v_cpu.to(v_layer.device, non_blocking=True)
                if has_ik:
                    ik_cpu = chunk[2]
                    ik_layer[chunk_indices] = ik_cpu.to(
                        ik_layer.device, non_blocking=True
                    )
        torch.npu.synchronize()
```
**EN:** This method implements `load_cpu_copy` on `NPUMLATokenToKVPool`. It primarily calls `torch.npu.synchronize`, `range`, `self.k_buffer.view`, `self.v_buffer.view`, `self.index_k_buffer.view`, `len` to complete its work. State updates are written into `chunk_size`, `has_ik`, `k_layer`, `v_layer`, `ik_layer`, `chunk_indices`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `NPUMLATokenToKVPool`）实现了 `load_cpu_copy`。 它主要通过调用 `torch.npu.synchronize`, `range`, `self.k_buffer.view`, `self.v_buffer.view`, `self.index_k_buffer.view`, `len` 来完成任务。 状态更新主要写入 `chunk_size`, `has_ik`, `k_layer`, `v_layer`, `ik_layer`, `chunk_indices`。 实现中使用了条件分支、迭代逻辑。

## Key Concepts / 关键概念
- **Classes / 类**: `NPUMHATokenToKVPool`, `NPUMLATokenToKVPool`
- **Functions / 函数**: `_init_npu_conv_state`, `__init__`, `_create_buffers`, `get_contiguous_buf_infos`, `set_kv_buffer`, `_chunk_copy_npu_to_cpu`, `get_cpu_copy`, `load_cpu_copy`
- **Themes / 主题**: `pool`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.constants`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.utils`, `sglang.srt.utils.common`, `sglang.srt.layers.radix_attention`
- **External / 外部依赖**: `torch`, `torch_npu`
- **Standard library / 标准库**: `typing`
