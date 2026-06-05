# memory_pool.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/memory_pool.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Copyright 2023-2024 SGLang Team Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. / 该模块实现与内存池相关的核心逻辑，并服务于 SGLang 的内存缓存子系统。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: License header and introductory comments / 许可证头与说明注释
```python
"""
Copyright 2023-2024 SGLang Team
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
"""

```
**EN:** Preserves licensing information and introductory commentary for the module.
**CN:** 保留模块的许可证信息以及开场说明文字。

### Lines 16-67: Imports and setup / 导入与初始化
```python
from __future__ import annotations

"""
Memory pool.

SGLang has two levels of memory pool.
ReqToTokenPool maps a request to its token locations.
TokenToKVPoolAllocator manages the indices to kv cache data.
KVCache actually holds the physical kv cache.
"""

import abc
# ... omitted for brevity ...
    is_npu,
    next_power_of_2,
)
from sglang.srt.utils.torch_memory_saver_adapter import TorchMemorySaverAdapter
```
**EN:** Imports `__future__`, `abc`, `dataclasses`, `logging`, `contextlib`, `typing` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `abc`, `dataclasses`, `logging`, `contextlib`, `typing` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 69-71: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.managers.cache_controller import LayerDoneCounter
    from sglang.srt.managers.schedule_batch import Req
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 74-82: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)

GB = 1024 * 1024 * 1024
_is_cuda = is_cuda()
_is_npu = is_npu()
_is_cpu = is_cpu()
_cpu_has_amx_support = cpu_has_amx_support()
_is_hip = is_hip()
_is_fp8_fnuz = is_fp8_fnuz()
```
**EN:** Defines module-level variables such as `logger`, `GB`, `_is_cuda`, `_is_npu`, `_is_cpu`.
**CN:** 定义模块级变量，例如 `logger`, `GB`, `_is_cuda`, `_is_npu`, `_is_cpu`。

### Lines 85-88: get_tensor_size_bytes implementation / get_tensor_size_bytes 实现
```python
def get_tensor_size_bytes(t: Union[torch.Tensor, List[torch.Tensor]]):
    if isinstance(t, list):
        return sum(get_tensor_size_bytes(x) for x in t)
    return np.prod(t.shape) * t.dtype.itemsize
```
**EN:** Retrieves the requested data or state from the current object. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。它会向调用方返回计算结果。

### Lines 91-125: _set_kv_buffer_impl implementation / _set_kv_buffer_impl 实现
```python
def _set_kv_buffer_impl(
    k: torch.Tensor,
    v: torch.Tensor,
    k_cache: torch.Tensor,
    v_cache: torch.Tensor,
    indices: torch.Tensor,
    row_dim: int,  # head_num * head_dim
    store_dtype: torch.dtype,
    device_module: Any,
    alt_stream: Optional[torch.cuda.Stream] = None,
    same_kv_dim: bool = True,
) -> None:
# ... omitted for brevity ...
        current_stream.wait_stream(alt_stream)
    else:  # fallback to naive implementation
        k_cache[indices] = k
        v_cache[indices] = v
```
**EN:** Implements the set KV buffer impl routine for this scope. It returns a computed result to its caller.
**CN:** 实现当前作用域中的set KV buffer impl例程。它会向调用方返回计算结果。

### Lines 128-130: ReqToTokenPool declaration / ReqToTokenPool 声明
```python
class ReqToTokenPool:
    """A memory pool that maps a request to its token locations."""

```
**EN:** A memory pool that maps a request to its token locations. Declares the `ReqToTokenPool` class.
**CN:** 声明 `ReqToTokenPool` 类。

### Lines 131-152: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        size: int,
        max_context_len: int,
        device: str,
        enable_memory_saver: bool,
    ):
        memory_saver_adapter = TorchMemorySaverAdapter.create(
            enable=enable_memory_saver
        )

        self.size = size
# ... omitted for brevity ...
            self.req_to_token = torch.zeros(
                (self._alloc_size, max_context_len), dtype=torch.int32, device=device
            )
        self.free_slots = list(range(1, self._alloc_size))
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `ReqToTokenPool`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `ReqToTokenPool`。

### Lines 154-155: write implementation / write 实现
```python
    def write(self, indices, values):
        self.req_to_token[indices] = values
```
**EN:** Implements the write routine for this scope. It belongs to `ReqToTokenPool`.
**CN:** 实现当前作用域中的write例程。 该方法属于 `ReqToTokenPool`。

### Lines 157-158: available_size implementation / available_size 实现
```python
    def available_size(self):
        return len(self.free_slots)
```
**EN:** Implements the available size routine for this scope. It belongs to `ReqToTokenPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的available size例程。 该方法属于 `ReqToTokenPool`。它会向调用方返回计算结果。

### Lines 160-184: alloc implementation / alloc 实现
```python
    def alloc(self, reqs: list[Req]) -> Optional[List[int]]:
        # Indices of reqs that already have a req_pool_idx and will reuse
        # their existing slot (e.g. chunked prefill continuing across chunks).
        reusing = [i for i, r in enumerate(reqs) if r.req_pool_idx is not None]
        # NOTE: this check is relaxed temporarily
        # https://github.com/sgl-project/sglang/pull/20476
        # if not any(r.is_dllm() for r in reqs):
        #     assert (
        #         sum(1 for i in reusing if reqs[i].is_chunked > 0) <= 1
        #     ), "only one chunked request may reuse req_pool_idx in a batch"
        assert all(
            reqs[i].is_chunked > 0 or reqs[i].kv_committed_len > 0 for i in reusing
# ... omitted for brevity ...
            if r.req_pool_idx is None:
                r.req_pool_idx = select_index[offset]
                offset += 1
        return [r.req_pool_idx for r in reqs]
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `ReqToTokenPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `ReqToTokenPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 186-189: free implementation / free 实现
```python
    def free(self, req: Req):
        assert req.req_pool_idx is not None, "request must have req_pool_idx"
        self.free_slots.append(req.req_pool_idx)
        req.req_pool_idx = None
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `ReqToTokenPool`.
**CN:** 将已分配资源释放回资源池。 该方法属于 `ReqToTokenPool`。

### Lines 191-192: clear implementation / clear 实现
```python
    def clear(self):
        self.free_slots = list(range(1, self._alloc_size))
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `ReqToTokenPool`.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `ReqToTokenPool`。

### Lines 195-195: MambaPool declaration / MambaPool 声明
```python
class MambaPool:
```
**EN:** Declares the `MambaPool` class.
**CN:** 声明 `MambaPool` 类。

### Lines 196-197: State declaration / State 声明
```python
    @dataclass(frozen=True, kw_only=True)
    class State:
```
**EN:** Declares the `State` class.
**CN:** 声明 `State` 类。

### Lines 198-199: Shared state definitions / 共享状态定义
```python
        conv: List[torch.Tensor]
        temporal: torch.Tensor
```
**EN:** Defines class-level variables such as `conv`, `temporal`.
**CN:** 定义类级变量，例如 `conv`, `temporal`。

### Lines 201-212: at_layer_idx implementation / at_layer_idx 实现
```python
        def at_layer_idx(self, layer: int):
            kwargs = {}
            # Use fields instead of vars to avoid torch.compile graph break
            for f in fields(self):
                name = f.name
                v = getattr(self, name)
                if name in ("conv", "intermediate_conv_window"):
                    kwargs[name] = [conv[layer] for conv in v]
                else:
                    kwargs[name] = v[layer]

            return type(self)(**kwargs)
```
**EN:** Implements the AT layer IDX routine for this scope. It belongs to `State`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的at layer idx例程。 该方法属于 `State`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 214-218: mem_usage_bytes implementation / mem_usage_bytes 实现
```python
        def mem_usage_bytes(self):
            return sum(
                get_tensor_size_bytes(getattr(self, f.name))
                for f in dataclasses.fields(self)
            )
```
**EN:** Implements the MEM usage bytes routine for this scope. It belongs to `State`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的mem usage bytes例程。 该方法属于 `State`。它会向调用方返回计算结果。

### Lines 220-221: SpeculativeState declaration / SpeculativeState 声明
```python
    @dataclass(frozen=True, kw_only=True)
    class SpeculativeState(State):
```
**EN:** Declares the `SpeculativeState` class and connects it to `State`.
**CN:** 声明 `SpeculativeState` 类，并将其关联到 `State`。

### Lines 222-223: Shared state definitions / 共享状态定义
```python
        intermediate_ssm: torch.Tensor
        intermediate_conv_window: List[torch.Tensor]
```
**EN:** Defines class-level variables such as `intermediate_ssm`, `intermediate_conv_window`.
**CN:** 定义类级变量，例如 `intermediate_ssm`, `intermediate_conv_window`。

### Lines 225-348: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        *,
        size: int,
        spec_state_size: int,
        cache_params: BaseLinearStateParams,
        mamba_layer_ids: List[int],
        device: str,
        enable_memory_saver: bool = False,
        speculative_num_draft_tokens: Optional[int] = None,
    ):
        conv_state_shape = cache_params.shape.conv
# ... omitted for brevity ...
                1, self.size + 1, dtype=torch.int64, device=self.device
            )
            self.mem_usage = self.mamba_cache.mem_usage_bytes() / GB
            self.num_mamba_layers = num_mamba_layers
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `MambaPool`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `MambaPool`。

### Lines 350-352: get_speculative_mamba2_params_all_layers implementation / get_speculative_mamba2_params_all_layers 实现
```python
    def get_speculative_mamba2_params_all_layers(self) -> SpeculativeState:
        assert isinstance(self.mamba_cache, self.SpeculativeState)
        return self.mamba_cache
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MambaPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MambaPool`。它会向调用方返回计算结果。

### Lines 354-355: mamba2_layer_cache implementation / mamba2_layer_cache 实现
```python
    def mamba2_layer_cache(self, layer_id: int):
        return self.mamba_cache.at_layer_idx(layer_id)
```
**EN:** Implements the mamba 2 layer cache routine for this scope. It belongs to `MambaPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的Mamba 2 layer缓存例程。 该方法属于 `MambaPool`。它会向调用方返回计算结果。

### Lines 357-358: available_size implementation / available_size 实现
```python
    def available_size(self):
        return len(self.free_slots)
```
**EN:** Implements the available size routine for this scope. It belongs to `MambaPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的available size例程。 该方法属于 `MambaPool`。它会向调用方返回计算结果。

### Lines 360-379: alloc implementation / alloc 实现
```python
    def alloc(self, need_size: int) -> Optional[torch.Tensor]:
        if need_size > len(self.free_slots):
            return None

        select_index = self.free_slots[:need_size]
        self.free_slots = self.free_slots[need_size:]
        # clear at alloc time — expand a scalar GPU zero to the right shape, no CPU-GPU sync
        for i in range(len(self.mamba_cache.conv)):
            t = self.mamba_cache.conv[i]
            z = torch.zeros(1, dtype=t.dtype, device=t.device).expand(
                t.shape[0], need_size, *t.shape[2:]
            )
# ... omitted for brevity ...
        )
        t[:, select_index] = z

        return select_index
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `MambaPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `MambaPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 381-384: free implementation / free 实现
```python
    def free(self, free_index: torch.Tensor):
        if free_index.numel() == 0:
            return
        self.free_slots = torch.cat((self.free_slots, free_index))
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `MambaPool`. It returns a computed result to its caller.
**CN:** 将已分配资源释放回资源池。 该方法属于 `MambaPool`。它会向调用方返回计算结果。

### Lines 386-389: clear implementation / clear 实现
```python
    def clear(self):
        self.free_slots = torch.arange(
            1, self.size + 1, dtype=torch.int64, device=self.device
        )
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `MambaPool`.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `MambaPool`。

### Lines 391-399: copy_from implementation / copy_from 实现
```python
    def copy_from(self, src_index: torch.Tensor, dst_index: torch.Tensor):
        for i in range(len(self.mamba_cache.conv)):
            self.mamba_cache.conv[i][:, dst_index] = self.mamba_cache.conv[i][
                :, src_index
            ]
        self.mamba_cache.temporal[:, dst_index] = self.mamba_cache.temporal[
            :, src_index
        ]
        return
```
**EN:** Copies data into a new location or representation. It belongs to `MambaPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 将数据复制到新的位置或表示中。 该方法属于 `MambaPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 401-406: fork_from implementation / fork_from 实现
```python
    def fork_from(self, src_index: torch.Tensor) -> Optional[torch.Tensor]:
        dst_index = self.alloc(1)
        if dst_index is None:
            return None
        self.copy_from(src_index, dst_index)
        return dst_index
```
**EN:** Implements the fork from routine for this scope. It belongs to `MambaPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的fork from例程。 该方法属于 `MambaPool`。它会向调用方返回计算结果。

### Lines 408-418: get_cpu_copy implementation / get_cpu_copy 实现
```python
    def get_cpu_copy(self, indices):
        current_platform.synchronize()
        conv_cpu = [
            conv[:, indices].to("cpu", non_blocking=True)
            for conv in self.mamba_cache.conv
        ]
        temporal_cpu = self.mamba_cache.temporal[:, indices].to(
            "cpu", non_blocking=True
        )
        current_platform.synchronize()
        return conv_cpu, temporal_cpu
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MambaPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MambaPool`。它会向调用方返回计算结果。

### Lines 420-428: load_cpu_copy implementation / load_cpu_copy 实现
```python
    def load_cpu_copy(self, mamba_cache_cpu, indices):
        conv_cpu, temporal_cpu = mamba_cache_cpu
        current_platform.synchronize()
        for i, conv in enumerate(self.mamba_cache.conv):
            conv[:, indices] = conv_cpu[i].to(conv.device, non_blocking=True)
        self.mamba_cache.temporal[:, indices] = temporal_cpu.to(
            self.mamba_cache.temporal.device, non_blocking=True
        )
        current_platform.synchronize()
```
**EN:** Loads state from an external or serialized representation. It belongs to `MambaPool`. The implementation iterates over inputs or managed entries.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `MambaPool`。实现过程中会遍历输入或受管条目。

### Lines 430-457: get_contiguous_buf_infos implementation / get_contiguous_buf_infos 实现
```python
    def get_contiguous_buf_infos(self):
        """
        Get buffer info for RDMA registration.
        Only returns conv and temporal state buffers, excluding intermediate buffers
        used for speculative decoding (intermediate_ssm, intermediate_conv_window).
        """
        state_tensors = []
        for field in vars(self.mamba_cache):
            # Skip intermediate buffers used only for speculative decoding
            # These buffers have different size (spec_state_size + 1) and should not be transferred
            if field in ("intermediate_ssm", "intermediate_conv_window"):
                continue
# ... omitted for brevity ...
            item_lens += [
                state_tensor[i][0].nbytes for i in range(self.num_mamba_layers)
            ]
        return data_ptrs, data_lens, item_lens
```
**EN:** Get buffer info for RDMA registration. Retrieves the requested data or state from the current object. It belongs to `MambaPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MambaPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 459-484: get_state_dim_per_tensor implementation / get_state_dim_per_tensor 实现
```python
    def get_state_dim_per_tensor(self):
        """Get the sliceable dimension size for each state tensor.

        For mamba state, the layout is:
        - conv_state: [num_layers, size+1, conv_dim/tp, conv_kernel-1]
        - temporal_state: [num_layers, size+1, num_heads/tp, head_dim, state_size]

        The 3rd dimension (index 2) is the one that gets sliced by TP.
        Returns the size of this dimension for each tensor (repeated for each layer).
        """
        state_tensors = []
        for field in vars(self.mamba_cache):
# ... omitted for brevity ...
            sliceable_dim = state_tensor.shape[2]
            # Repeat for each layer since we have per-layer data_ptrs
            dim_per_tensor += [sliceable_dim] * self.num_mamba_layers
        return dim_per_tensor
```
**EN:** Get the sliceable dimension size for each state tensor. Retrieves the requested data or state from the current object. It belongs to `MambaPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MambaPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 487-489: HybridReqToTokenPool declaration / HybridReqToTokenPool 声明
```python
class HybridReqToTokenPool(ReqToTokenPool):
    """A memory pool that maps a request to its token locations."""

```
**EN:** A memory pool that maps a request to its token locations. Declares the `HybridReqToTokenPool` class and connects it to `ReqToTokenPool`.
**CN:** 声明 `HybridReqToTokenPool` 类，并将其关联到 `ReqToTokenPool`。

### Lines 490-526: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        *,
        size: int,
        mamba_size: int,
        mamba_spec_state_size: int,
        max_context_len: int,
        device: str,
        enable_memory_saver: bool,
        cache_params: BaseLinearStateParams,
        mamba_layer_ids: List[int],
        enable_mamba_extra_buffer: bool,
# ... omitted for brevity ...
            device=device,
            enable_mamba_extra_buffer=enable_mamba_extra_buffer,
            speculative_num_draft_tokens=speculative_num_draft_tokens,
        )
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `HybridReqToTokenPool`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `HybridReqToTokenPool`。

### Lines 528-563: _init_mamba_pool implementation / _init_mamba_pool 实现
```python
    def _init_mamba_pool(
        self,
        mamba_size: int,
        mamba_spec_state_size: int,
        cache_params: BaseLinearStateParams,
        mamba_layer_ids: List[int],
        device: str,
        enable_mamba_extra_buffer: bool,
        speculative_num_draft_tokens: int = None,
    ):
        self.mamba_pool = MambaPool(
            size=mamba_size,
# ... omitted for brevity ...
                    dtype=torch.int32,
                    device=self.device,
                )
            )
```
**EN:** Implements the init mamba pool routine for this scope. It belongs to `HybridReqToTokenPool`.
**CN:** 实现当前作用域中的初始化Mamba池例程。 该方法属于 `HybridReqToTokenPool`。

### Lines 565-568: register_layer_transfer_counter implementation / register_layer_transfer_counter 实现
```python
    def register_layer_transfer_counter(
        self, layer_transfer_counter: "LayerDoneCounter"
    ):
        self.layer_transfer_counter = layer_transfer_counter
```
**EN:** Registers metadata so other components can discover this object. It belongs to `HybridReqToTokenPool`.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `HybridReqToTokenPool`。

### Lines 569-571: Comment block / 注释块
```python

    # For chunk prefill req, we do not need to allocate mamba cache,
    # We could use allocated mamba cache instead.
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 572-617: alloc implementation / alloc 实现
```python
    def alloc(self, reqs: List["Req"]) -> Optional[List[int]]:
        select_index = super().alloc(reqs)
        if select_index is None:
            return None

        mamba_indices: list[torch.Tensor] = []
        mamba_ping_pong_track_buffers: list[torch.Tensor] = []
        for req in reqs:
            mid = None
            if req.mamba_pool_idx is not None:  # for radix cache
                mid = req.mamba_pool_idx
            else:
# ... omitted for brevity ...
            self.req_index_to_mamba_ping_pong_track_buffer_mapping[select_index] = (
                ping_pong_tensor
            )
        return select_index
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `HybridReqToTokenPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `HybridReqToTokenPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 619-620: get_mamba_indices implementation / get_mamba_indices 实现
```python
    def get_mamba_indices(self, req_indices: torch.Tensor) -> torch.Tensor:
        return self.req_index_to_mamba_index_mapping[req_indices]
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HybridReqToTokenPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HybridReqToTokenPool`。它会向调用方返回计算结果。

### Lines 622-626: mamba2_layer_cache implementation / mamba2_layer_cache 实现
```python
    def mamba2_layer_cache(self, layer_id: int):
        assert layer_id in self.mamba_map
        if self.layer_transfer_counter is not None:
            self.layer_transfer_counter.wait_until(layer_id - self.start_layer)
        return self.mamba_pool.mamba2_layer_cache(self.mamba_map[layer_id])
```
**EN:** Implements the mamba 2 layer cache routine for this scope. It belongs to `HybridReqToTokenPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的Mamba 2 layer缓存例程。 该方法属于 `HybridReqToTokenPool`。它会向调用方返回计算结果。

### Lines 628-629: get_speculative_mamba2_params_all_layers implementation / get_speculative_mamba2_params_all_layers 实现
```python
    def get_speculative_mamba2_params_all_layers(self) -> MambaPool.SpeculativeState:
        return self.mamba_pool.get_speculative_mamba2_params_all_layers()
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HybridReqToTokenPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HybridReqToTokenPool`。它会向调用方返回计算结果。

### Lines 631-632: get_state_buf_infos implementation / get_state_buf_infos 实现
```python
    def get_state_buf_infos(self):
        return self.mamba_pool.get_contiguous_buf_infos()
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HybridReqToTokenPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HybridReqToTokenPool`。它会向调用方返回计算结果。

### Lines 634-635: get_state_dim_per_tensor implementation / get_state_dim_per_tensor 实现
```python
    def get_state_dim_per_tensor(self):
        return self.mamba_pool.get_state_dim_per_tensor()
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HybridReqToTokenPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HybridReqToTokenPool`。它会向调用方返回计算结果。

### Lines 637-641: get_mamba_ping_pong_other_idx implementation / get_mamba_ping_pong_other_idx 实现
```python
    def get_mamba_ping_pong_other_idx(self, mamba_next_track_idx: int) -> int:
        if self.mamba_ping_pong_track_buffer_size == 2:
            return 1 - mamba_next_track_idx
        else:
            return mamba_next_track_idx
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HybridReqToTokenPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HybridReqToTokenPool`。它会向调用方返回计算结果。

### Lines 643-682: free_mamba_cache implementation / free_mamba_cache 实现
```python
    def free_mamba_cache(
        self, req: "Req", mamba_ping_pong_track_buffer_to_keep: Optional[int] = None
    ):
        mamba_index = req.mamba_pool_idx
        assert mamba_index is not None, "double free? mamba_index is None"
        self.mamba_pool.free(mamba_index.unsqueeze(0))
        req.mamba_pool_idx = None

        if self.enable_mamba_extra_buffer:
            mamba_ping_pong_track_buffer_to_free = (
                self.req_index_to_mamba_ping_pong_track_buffer_mapping[req.req_pool_idx]
            )
# ... omitted for brevity ...
                    mamba_ping_pong_track_buffer_to_free = (
                        mamba_ping_pong_track_buffer_to_free[0:0]
                    )
            self.mamba_pool.free(mamba_ping_pong_track_buffer_to_free)
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `HybridReqToTokenPool`.
**CN:** 将已分配资源释放回资源池。 该方法属于 `HybridReqToTokenPool`。

### Lines 684-690: clear implementation / clear 实现
```python
    def clear(self):
        logger.info("Reset HybridReqToTokenPool")
        super().clear()
        self.mamba_pool.clear()
        self.req_index_to_mamba_index_mapping.zero_()
        if self.enable_mamba_extra_buffer:
            self.req_index_to_mamba_ping_pong_track_buffer_mapping.zero_()
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `HybridReqToTokenPool`.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `HybridReqToTokenPool`。

### Lines 693-693: KVCache declaration / KVCache 声明
```python
class KVCache(abc.ABC):
```
**EN:** Declares the `KVCache` class and connects it to `abc.ABC`.
**CN:** 声明 `KVCache` 类，并将其关联到 `abc.ABC`。

### Lines 694-732: __init__ implementation / __init__ 实现
```python
    @abc.abstractmethod
    def __init__(
        self,
        size: int,
        page_size: int,
        dtype: torch.dtype,
        layer_num: int,
        device: str,
        enable_memory_saver: bool,
        start_layer: Optional[int] = None,
        end_layer: Optional[int] = None,
    ):
# ... omitted for brevity ...
        # for disagg with nvlink
        self.enable_custom_mem_pool, self.custom_mem_pool, _ = (
            maybe_init_custom_mem_pool(device=self.device)
        )
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `KVCache`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `KVCache`。

### Lines 734-752: _finalize_allocation_log implementation / _finalize_allocation_log 实现
```python
    def _finalize_allocation_log(self, num_tokens: int):
        """Common logging and mem_usage computation for KV cache allocation.
        Supports both tuple (K, V) size returns and single KV size returns.
        """
        kv_size_bytes = self.get_kv_size_bytes()
        if isinstance(kv_size_bytes, tuple):
            k_size, v_size = kv_size_bytes
            k_size_GB = k_size / GB
            v_size_GB = v_size / GB
            logger.info(
                f"KV Cache is allocated. #tokens: {num_tokens}, K size: {k_size_GB:.2f} GB, V size: {v_size_GB:.2f} GB"
            )
# ... omitted for brevity ...
            logger.info(
                f"KV Cache is allocated. #tokens: {num_tokens}, KV size: {kv_size_GB:.2f} GB"
            )
            self.mem_usage = kv_size_GB
```
**EN:** Common logging and mem_usage computation for KV cache allocation. Implements the finalize allocation LOG routine for this scope. It belongs to `KVCache`.
**CN:** 实现当前作用域中的finalize allocation log例程。 该方法属于 `KVCache`。

### Lines 754-756: get_key_buffer implementation / get_key_buffer 实现
```python
    @abc.abstractmethod
    def get_key_buffer(self, layer_id: int) -> torch.Tensor:
        raise NotImplementedError()
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `KVCache`. It validates error cases explicitly.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `KVCache`。它会显式处理错误场景。

### Lines 758-760: get_value_buffer implementation / get_value_buffer 实现
```python
    @abc.abstractmethod
    def get_value_buffer(self, layer_id: int) -> torch.Tensor:
        raise NotImplementedError()
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `KVCache`. It validates error cases explicitly.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `KVCache`。它会显式处理错误场景。

### Lines 762-764: get_kv_buffer implementation / get_kv_buffer 实现
```python
    @abc.abstractmethod
    def get_kv_buffer(self, layer_id: int) -> Tuple[torch.Tensor, torch.Tensor]:
        raise NotImplementedError()
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `KVCache`. It validates error cases explicitly.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `KVCache`。它会显式处理错误场景。

### Lines 766-774: set_kv_buffer implementation / set_kv_buffer 实现
```python
    @abc.abstractmethod
    def set_kv_buffer(
        self,
        layer: RadixAttention,
        loc: torch.Tensor,
        cache_k: torch.Tensor,
        cache_v: torch.Tensor,
    ) -> None:
        raise NotImplementedError()
```
**EN:** Updates internal state with the provided value. It belongs to `KVCache`. It validates error cases explicitly.
**CN:** 使用给定值更新内部状态。 该方法属于 `KVCache`。它会显式处理错误场景。

### Lines 776-777: register_layer_transfer_counter implementation / register_layer_transfer_counter 实现
```python
    def register_layer_transfer_counter(self, layer_transfer_counter: LayerDoneCounter):
        self.layer_transfer_counter = layer_transfer_counter
```
**EN:** Registers metadata so other components can discover this object. It belongs to `KVCache`.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `KVCache`。

### Lines 779-780: get_cpu_copy implementation / get_cpu_copy 实现
```python
    def get_cpu_copy(self, indices, mamba_indices=None):
        raise NotImplementedError()
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `KVCache`. It validates error cases explicitly.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `KVCache`。它会显式处理错误场景。

### Lines 782-783: load_cpu_copy implementation / load_cpu_copy 实现
```python
    def load_cpu_copy(self, kv_cache_cpu, indices, mamba_indices=None):
        raise NotImplementedError()
```
**EN:** Loads state from an external or serialized representation. It belongs to `KVCache`. It validates error cases explicitly.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `KVCache`。它会显式处理错误场景。

### Lines 785-786: maybe_get_custom_mem_pool implementation / maybe_get_custom_mem_pool 实现
```python
    def maybe_get_custom_mem_pool(self):
        return self.custom_mem_pool
```
**EN:** Implements the maybe get custom MEM pool routine for this scope. It belongs to `KVCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的maybe get custom mem池例程。 该方法属于 `KVCache`。它会向调用方返回计算结果。

### Lines 789-790: MHATokenToKVPool declaration / MHATokenToKVPool 声明
```python
class MHATokenToKVPool(KVCache):

```
**EN:** Declares the `MHATokenToKVPool` class and connects it to `KVCache`.
**CN:** 声明 `MHATokenToKVPool` 类，并将其关联到 `KVCache`。

### Lines 791-848: __init__ implementation / __init__ 实现
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
        v_head_dim: Optional[int] = None,
        swa_head_num: Optional[int] = None,
# ... omitted for brevity ...

        # for store_cache JIT kernel
        self.row_dim = self.head_num * self.head_dim
        self.same_kv_dim = self.head_dim == self.v_head_dim
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `MHATokenToKVPool`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `MHATokenToKVPool`。

### Lines 850-895: _init_kv_copy_and_warmup implementation / _init_kv_copy_and_warmup 实现
```python
    def _init_kv_copy_and_warmup(self):
        # Heuristics for KV copy tiling
        _KV_COPY_STRIDE_THRESHOLD_LARGE = 8192
        _KV_COPY_STRIDE_THRESHOLD_MEDIUM = 4096
        _KV_COPY_TILE_SIZE_LARGE = 512
        _KV_COPY_TILE_SIZE_MEDIUM = 256
        _KV_COPY_TILE_SIZE_SMALL = 128
        _KV_COPY_NUM_WARPS_LARGE_TILE = 8
        _KV_COPY_NUM_WARPS_SMALL_TILE = 4

        stride_bytes = int(self.data_strides[0].item())
        if stride_bytes >= _KV_COPY_STRIDE_THRESHOLD_LARGE:
# ... omitted for brevity ...
            BYTES_PER_TILE=self._kv_copy_config["bytes_per_tile"],
            num_warps=self._kv_copy_config["num_warps"],
            num_stages=2,
        )
```
**EN:** Implements the init KV copy AND warmup routine for this scope. It belongs to `MHATokenToKVPool`.
**CN:** 实现当前作用域中的初始化KV copy and warmup例程。 该方法属于 `MHATokenToKVPool`。

### Lines 897-940: _create_buffers implementation / _create_buffers 实现
```python
    def _create_buffers(self):
        with self.memory_saver_adapter.region(GPU_MEMORY_TYPE_KV_CACHE):
            with (
                torch.cuda.use_mem_pool(self.custom_mem_pool)
                if self.enable_custom_mem_pool
                else nullcontext()
            ):
                # [size, head_num, head_dim] for each layer
                # The padded slot 0 is used for writing dummy outputs from padded tokens.
                self.k_buffer = [
                    torch.zeros(
                        (self.size + self.page_size, self.head_num, self.head_dim),
# ... omitted for brevity ...
                for x in self.k_buffer + self.v_buffer
            ],
            device=self.device,
        )
```
**EN:** Implements the create buffers routine for this scope. It belongs to `MHATokenToKVPool`.
**CN:** 实现当前作用域中的create buffers例程。 该方法属于 `MHATokenToKVPool`。

### Lines 942-944: _clear_buffers implementation / _clear_buffers 实现
```python
    def _clear_buffers(self):
        del self.k_buffer
        del self.v_buffer
```
**EN:** Implements the clear buffers routine for this scope. It belongs to `MHATokenToKVPool`.
**CN:** 实现当前作用域中的clear buffers例程。 该方法属于 `MHATokenToKVPool`。

### Lines 946-955: get_kv_size_bytes implementation / get_kv_size_bytes 实现
```python
    def get_kv_size_bytes(self):
        assert hasattr(self, "k_buffer")
        assert hasattr(self, "v_buffer")
        k_size_bytes = 0
        for k_cache in self.k_buffer:
            k_size_bytes += get_tensor_size_bytes(k_cache)
        v_size_bytes = 0
        for v_cache in self.v_buffer:
            v_size_bytes += get_tensor_size_bytes(v_cache)
        return k_size_bytes, v_size_bytes
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MHATokenToKVPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MHATokenToKVPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 956-957: Comment block / 注释块
```python

    # for disagg
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 958-982: get_contiguous_buf_infos implementation / get_contiguous_buf_infos 实现
```python
    def get_contiguous_buf_infos(self):
        # layer_num x [seq_len, head_num, head_dim]
        # layer_num x [page_num, page_size, head_num, head_dim]
        kv_data_ptrs = [
            self._get_key_buffer(i).data_ptr()
            for i in range(self.start_layer, self.start_layer + self.layer_num)
        ] + [
            self._get_value_buffer(i).data_ptr()
            for i in range(self.start_layer, self.start_layer + self.layer_num)
        ]
        kv_data_lens = [
            self._get_key_buffer(i).nbytes
# ... omitted for brevity ...
            self._get_value_buffer(i)[0].nbytes * self.page_size
            for i in range(self.start_layer, self.start_layer + self.layer_num)
        ]
        return kv_data_ptrs, kv_data_lens, kv_item_lens
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MHATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MHATokenToKVPool`。它会向调用方返回计算结果。

### Lines 984-1000: get_cpu_copy implementation / get_cpu_copy 实现
```python
    def get_cpu_copy(self, indices, mamba_indices=None):
        current_platform.synchronize()
        kv_cache_cpu = []
        chunk_size = self.cpu_offloading_chunk_size
        for layer_id in range(self.layer_num):
            kv_cache_cpu.append([])
            for i in range(0, len(indices), chunk_size):
                chunk_indices = indices[i : i + chunk_size]
                k_cpu = self.k_buffer[layer_id][chunk_indices].to(
                    "cpu", non_blocking=True
                )
                v_cpu = self.v_buffer[layer_id][chunk_indices].to(
                    "cpu", non_blocking=True
                )
                kv_cache_cpu[-1].append([k_cpu, v_cpu])
        current_platform.synchronize()
        return kv_cache_cpu
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MHATokenToKVPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MHATokenToKVPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1002-1017: load_cpu_copy implementation / load_cpu_copy 实现
```python
    def load_cpu_copy(self, kv_cache_cpu, indices, mamba_indices=None):
        current_platform.synchronize()
        chunk_size = self.cpu_offloading_chunk_size
        for layer_id in range(self.layer_num):
            for i in range(0, len(indices), chunk_size):
                chunk_indices = indices[i : i + chunk_size]
                k_cpu, v_cpu = (
                    kv_cache_cpu[layer_id][i // chunk_size][0],
                    kv_cache_cpu[layer_id][i // chunk_size][1],
                )
                assert k_cpu.shape[0] == v_cpu.shape[0] == len(chunk_indices)
                k_chunk = k_cpu.to(self.k_buffer[0].device, non_blocking=True)
                v_chunk = v_cpu.to(self.v_buffer[0].device, non_blocking=True)
                self.k_buffer[layer_id][chunk_indices] = k_chunk
                self.v_buffer[layer_id][chunk_indices] = v_chunk
        current_platform.synchronize()
```
**EN:** Loads state from an external or serialized representation. It belongs to `MHATokenToKVPool`. The implementation iterates over inputs or managed entries.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `MHATokenToKVPool`。实现过程中会遍历输入或受管条目。

### Lines 1019-1023: _get_key_buffer implementation / _get_key_buffer 实现
```python
    def _get_key_buffer(self, layer_id: int):
        # for internal use of referencing
        if self.store_dtype != self.dtype:
            return self.k_buffer[layer_id - self.start_layer].view(self.dtype)
        return self.k_buffer[layer_id - self.start_layer]
```
**EN:** Implements the get KEY buffer routine for this scope. It belongs to `MHATokenToKVPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get key buffer例程。 该方法属于 `MHATokenToKVPool`。它会向调用方返回计算结果。

### Lines 1025-1031: get_key_buffer implementation / get_key_buffer 实现
```python
    def get_key_buffer(self, layer_id: int):
        # note: get_key_buffer is hooked with synchronization for layer-wise KV cache loading
        # it is supposed to be used only by attention backend not for information purpose
        # same applies to get_value_buffer and get_kv_buffer
        if self.layer_transfer_counter is not None:
            self.layer_transfer_counter.wait_until(layer_id - self.start_layer)
        return self._get_key_buffer(layer_id)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MHATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MHATokenToKVPool`。它会向调用方返回计算结果。

### Lines 1033-1037: _get_value_buffer implementation / _get_value_buffer 实现
```python
    def _get_value_buffer(self, layer_id: int):
        # for internal use of referencing
        if self.store_dtype != self.dtype:
            return self.v_buffer[layer_id - self.start_layer].view(self.dtype)
        return self.v_buffer[layer_id - self.start_layer]
```
**EN:** Implements the get value buffer routine for this scope. It belongs to `MHATokenToKVPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get value buffer例程。 该方法属于 `MHATokenToKVPool`。它会向调用方返回计算结果。

### Lines 1039-1042: get_value_buffer implementation / get_value_buffer 实现
```python
    def get_value_buffer(self, layer_id: int):
        if self.layer_transfer_counter is not None:
            self.layer_transfer_counter.wait_until(layer_id - self.start_layer)
        return self._get_value_buffer(layer_id)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MHATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MHATokenToKVPool`。它会向调用方返回计算结果。

### Lines 1044-1045: get_kv_buffer implementation / get_kv_buffer 实现
```python
    def get_kv_buffer(self, layer_id: int):
        return self.get_key_buffer(layer_id), self.get_value_buffer(layer_id)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MHATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MHATokenToKVPool`。它会向调用方返回计算结果。

### Lines 1047-1084: set_kv_buffer implementation / set_kv_buffer 实现
```python
    def set_kv_buffer(
        self,
        layer: RadixAttention,
        loc: torch.Tensor,
        cache_k: torch.Tensor,
        cache_v: torch.Tensor,
        k_scale: Optional[float] = None,
        v_scale: Optional[float] = None,
        layer_id_override: Optional[int] = None,
    ):
        if layer_id_override is not None:
            layer_id = layer_id_override
# ... omitted for brevity ...
            device_module=self.device_module,
            alt_stream=self.alt_stream,
            same_kv_dim=self.same_kv_dim,
        )
```
**EN:** Updates internal state with the provided value. It belongs to `MHATokenToKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `MHATokenToKVPool`。

### Lines 1086-1133: move_kv_cache implementation / move_kv_cache 实现
```python
    def move_kv_cache(self, tgt_loc: torch.Tensor, src_loc: torch.Tensor):
        if envs.SGLANG_NATIVE_MOVE_KV_CACHE.get():
            move_kv_cache_native(self.k_buffer, self.v_buffer, tgt_loc, src_loc)
            return

        N = tgt_loc.numel()
        if N == 0:
            return

        assert (
            self._kv_copy_config is not None
        ), "KV copy not initialized. Set enable_kv_cache_copy=True in __init__"
# ... omitted for brevity ...
                BYTES_PER_TILE=cfg["bytes_per_tile"],
                num_warps=cfg["num_warps"],
                num_stages=2,
            )
```
**EN:** Implements the move KV cache routine for this scope. It belongs to `MHATokenToKVPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的move KV缓存例程。 该方法属于 `MHATokenToKVPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1136-1153: NoOpMHATokenToKVPool declaration / NoOpMHATokenToKVPool 声明
```python
class NoOpMHATokenToKVPool(MHATokenToKVPool):
    """KV cache pool that skips physical K/V buffer allocation.

    Used in embedding-mode prefill-only workloads with the FA
    fa_skip_kv_cache path, where no layer reads or writes KV cache because
    attention uses raw K/V via flash_attn_varlen_func. Other prefill-only paths
    such as scoring/MIS may benefit from the same idea later, but some still
    stage K/V through paged cache today.

    This class keeps the scheduler's view of pool capacity (self.size is
    honored for admission) but allocates only (page_size, head_num, head_dim)
    placeholder tensors per layer to satisfy any code paths that dereference
    the buffers.

    Callers MUST ensure no real set_kv_buffer/get_*_buffer calls happen against
    this pool; those paths raise loudly so misuse is visible.
    """

```
**EN:** KV cache pool that skips physical K/V buffer allocation. Declares the `NoOpMHATokenToKVPool` class and connects it to `MHATokenToKVPool`.
**CN:** 声明 `NoOpMHATokenToKVPool` 类，并将其关联到 `MHATokenToKVPool`。

### Lines 1154-1198: _create_buffers implementation / _create_buffers 实现
```python
    def _create_buffers(self):
        # Allocate minimal placeholder buffers. They exist purely so that code
        # paths holding `k_buffer` / `v_buffer` references (pointer tables,
        # layer-transfer counters, stride arithmetic) keep working without
        # None-guards scattered across the codebase. Shape is
        # [page_size, head_num, head_dim] per layer so that the unconditional
        # `key_cache.view(-1, page_size, head_num, head_dim)` in the FA backend
        # at the top of forward_extend succeeds regardless of --page-size.
        # Total footprint is still on the order of KB vs GBs for a real pool.
        with self.memory_saver_adapter.region(GPU_MEMORY_TYPE_KV_CACHE):
            self.k_buffer = [
                torch.zeros(
# ... omitted for brevity ...
                for x in self.k_buffer + self.v_buffer
            ],
            device=self.device,
        )
```
**EN:** Implements the create buffers routine for this scope. It belongs to `NoOpMHATokenToKVPool`.
**CN:** 实现当前作用域中的create buffers例程。 该方法属于 `NoOpMHATokenToKVPool`。

### Lines 1200-1213: _finalize_allocation_log implementation / _finalize_allocation_log 实现
```python
    def _finalize_allocation_log(self, num_tokens: int):
        self.mem_usage = 0.0
        placeholder_bytes = (
            2
            * self.layer_num
            * self.page_size
            * self.head_num
            * max(self.head_dim, self.v_head_dim)
            * self.store_dtype.itemsize
        )
        logger.info(
            f"KV Cache skipped (no-op pool). Logical #tokens: {num_tokens}, "
            f"physical K/V size: ~{placeholder_bytes / 1024:.1f} KB placeholder"
        )
```
**EN:** Implements the finalize allocation LOG routine for this scope. It belongs to `NoOpMHATokenToKVPool`.
**CN:** 实现当前作用域中的finalize allocation log例程。 该方法属于 `NoOpMHATokenToKVPool`。

### Lines 1215-1217: get_kv_size_bytes implementation / get_kv_size_bytes 实现
```python
    def get_kv_size_bytes(self):
        # Report zero so downstream memory accounting matches reality.
        return (0, 0)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `NoOpMHATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `NoOpMHATokenToKVPool`。它会向调用方返回计算结果。

### Lines 1219-1227: set_kv_buffer implementation / set_kv_buffer 实现
```python
    def set_kv_buffer(self, *args, **kwargs):
        raise RuntimeError(
            "NoOpMHATokenToKVPool.set_kv_buffer was called. This pool is only "
            "valid in prefill-only modes (e.g. --is-embedding, scoring) with "
            "the FA backend's fa_skip_kv_cache path active; the attention "
            "backend must never write to it. Check that the workload truly "
            "performs no decode and that the FA backend's fa_skip_kv_cache "
            "preconditions are met."
        )
```
**EN:** Updates internal state with the provided value. It belongs to `NoOpMHATokenToKVPool`. It validates error cases explicitly.
**CN:** 使用给定值更新内部状态。 该方法属于 `NoOpMHATokenToKVPool`。它会显式处理错误场景。

### Lines 1229-1233: get_key_buffer implementation / get_key_buffer 实现
```python
    def get_key_buffer(self, layer_id: int):
        # Return the placeholder. The FA backend reads this before taking the
        # fa_skip_kv_cache branch (which does not use it); the placeholder shape
        # is (page_size, head_num, head_dim) so downstream .view() calls succeed.
        return self.k_buffer[layer_id - self.start_layer]
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `NoOpMHATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `NoOpMHATokenToKVPool`。它会向调用方返回计算结果。

### Lines 1235-1236: get_value_buffer implementation / get_value_buffer 实现
```python
    def get_value_buffer(self, layer_id: int):
        return self.v_buffer[layer_id - self.start_layer]
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `NoOpMHATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `NoOpMHATokenToKVPool`。它会向调用方返回计算结果。

### Lines 1238-1239: get_kv_buffer implementation / get_kv_buffer 实现
```python
    def get_kv_buffer(self, layer_id: int):
        return self.get_key_buffer(layer_id), self.get_value_buffer(layer_id)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `NoOpMHATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `NoOpMHATokenToKVPool`。它会向调用方返回计算结果。

### Lines 1241-1243: move_kv_cache implementation / move_kv_cache 实现
```python
    def move_kv_cache(self, tgt_loc: torch.Tensor, src_loc: torch.Tensor):
        # no-op; embedding mode has no KV cache to move
        return
```
**EN:** Implements the move KV cache routine for this scope. It belongs to `NoOpMHATokenToKVPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的move KV缓存例程。 该方法属于 `NoOpMHATokenToKVPool`。它会向调用方返回计算结果。

### Lines 1246-1247: MHATokenToKVPoolFP4 declaration / MHATokenToKVPoolFP4 声明
```python
class MHATokenToKVPoolFP4(MHATokenToKVPool):

```
**EN:** Declares the `MHATokenToKVPoolFP4` class and connects it to `MHATokenToKVPool`.
**CN:** 声明 `MHATokenToKVPoolFP4` 类，并将其关联到 `MHATokenToKVPool`。

### Lines 1248-1295: _create_buffers implementation / _create_buffers 实现
```python
    def _create_buffers(self):
        with self.memory_saver_adapter.region(GPU_MEMORY_TYPE_KV_CACHE):
            with (
                torch.cuda.use_mem_pool(self.custom_mem_pool)
                if self.enable_custom_mem_pool
                else nullcontext()
            ):
                # [size, head_num, head_dim] for each layer
                # The padded slot 0 is used for writing dummy outputs from padded tokens.
                m = self.size + self.page_size
                n = self.head_num
                k = self.head_dim
# ... omitted for brevity ...
                        device=self.device,
                    )
                    for _ in range(self.layer_num)
                ]
```
**EN:** Implements the create buffers routine for this scope. It belongs to `MHATokenToKVPoolFP4`.
**CN:** 实现当前作用域中的create buffers例程。 该方法属于 `MHATokenToKVPoolFP4`。

### Lines 1297-1301: _clear_buffers implementation / _clear_buffers 实现
```python
    def _clear_buffers(self):
        del self.k_buffer
        del self.v_buffer
        del self.k_scale_buffer
        del self.v_scale_buffer
```
**EN:** Implements the clear buffers routine for this scope. It belongs to `MHATokenToKVPoolFP4`.
**CN:** 实现当前作用域中的clear buffers例程。 该方法属于 `MHATokenToKVPoolFP4`。

### Lines 1303-1317: _get_key_buffer implementation / _get_key_buffer 实现
```python
    def _get_key_buffer(self, layer_id: int):
        # for internal use of referencing
        if self.store_dtype != self.dtype:
            cache_k_nope_fp4 = self.k_buffer[layer_id - self.start_layer].view(
                torch.uint8
            )
            cache_k_nope_fp4_sf = self.k_scale_buffer[layer_id - self.start_layer]

            from sglang.srt.layers.quantization.kvfp4_tensor import KVFP4QuantizeUtil

            cache_k_nope_fp4_dequant = KVFP4QuantizeUtil.batched_dequantize(
                cache_k_nope_fp4, cache_k_nope_fp4_sf
            )
            return cache_k_nope_fp4_dequant
        return self.k_buffer[layer_id - self.start_layer]
```
**EN:** Implements the get KEY buffer routine for this scope. It belongs to `MHATokenToKVPoolFP4`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get key buffer例程。 该方法属于 `MHATokenToKVPoolFP4`。它会向调用方返回计算结果。

### Lines 1319-1333: _get_value_buffer implementation / _get_value_buffer 实现
```python
    def _get_value_buffer(self, layer_id: int):
        # for internal use of referencing
        if self.store_dtype != self.dtype:
            cache_v_nope_fp4 = self.v_buffer[layer_id - self.start_layer].view(
                torch.uint8
            )
            cache_v_nope_fp4_sf = self.v_scale_buffer[layer_id - self.start_layer]

            from sglang.srt.layers.quantization.kvfp4_tensor import KVFP4QuantizeUtil

            cache_v_nope_fp4_dequant = KVFP4QuantizeUtil.batched_dequantize(
                cache_v_nope_fp4, cache_v_nope_fp4_sf
            )
            return cache_v_nope_fp4_dequant
        return self.v_buffer[layer_id - self.start_layer]
```
**EN:** Implements the get value buffer routine for this scope. It belongs to `MHATokenToKVPoolFP4`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get value buffer例程。 该方法属于 `MHATokenToKVPoolFP4`。它会向调用方返回计算结果。

### Lines 1335-1386: set_kv_buffer implementation / set_kv_buffer 实现
```python
    def set_kv_buffer(
        self,
        layer: RadixAttention,
        loc: torch.Tensor,
        cache_k: torch.Tensor,
        cache_v: torch.Tensor,
        k_scale: Optional[float] = None,
        v_scale: Optional[float] = None,
        layer_id_override: Optional[int] = None,
    ):
        from sglang.srt.model_executor.cuda_graph_runner import get_is_capture_mode

# ... omitted for brevity ...
            self.v_buffer[layer_id - self.start_layer][loc] = cache_v

            self.k_scale_buffer[layer_id - self.start_layer][loc] = cache_k_fp4_sf
            self.v_scale_buffer[layer_id - self.start_layer][loc] = cache_v_fp4_sf
```
**EN:** Updates internal state with the provided value. It belongs to `MHATokenToKVPoolFP4`.
**CN:** 使用给定值更新内部状态。 该方法属于 `MHATokenToKVPoolFP4`。

### Lines 1389-1391: HybridLinearKVPool declaration / HybridLinearKVPool 声明
```python
class HybridLinearKVPool(KVCache):
    """KV cache with separate pools for full and linear attention layers."""

```
**EN:** KV cache with separate pools for full and linear attention layers. Declares the `HybridLinearKVPool` class and connects it to `KVCache`.
**CN:** 声明 `HybridLinearKVPool` 类，并将其关联到 `KVCache`。

### Lines 1392-1476: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        size: int,
        dtype: torch.dtype,
        page_size: int,
        head_num: int,
        head_dim: int,
        full_attention_layer_ids: List[int],
        enable_kvcache_transpose: bool,
        device: str,
        mamba_pool: MambaPool,
        enable_memory_saver: bool = False,
# ... omitted for brevity ...
            self.mem_usage = self.get_kv_size_bytes() / GB
        else:
            k_size, v_size = self.get_kv_size_bytes()
            self.mem_usage = (k_size + v_size) / GB
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `HybridLinearKVPool`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `HybridLinearKVPool`。

### Lines 1478-1479: get_kv_size_bytes implementation / get_kv_size_bytes 实现
```python
    def get_kv_size_bytes(self):
        return self.full_kv_pool.get_kv_size_bytes()
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HybridLinearKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HybridLinearKVPool`。它会向调用方返回计算结果。

### Lines 1481-1482: get_contiguous_buf_infos implementation / get_contiguous_buf_infos 实现
```python
    def get_contiguous_buf_infos(self):
        return self.full_kv_pool.get_contiguous_buf_infos()
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HybridLinearKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HybridLinearKVPool`。它会向调用方返回计算结果。

### Lines 1484-1488: get_state_buf_infos implementation / get_state_buf_infos 实现
```python
    def get_state_buf_infos(self):
        mamba_data_ptrs, mamba_data_lens, mamba_item_lens = (
            self.mamba_pool.get_contiguous_buf_infos()
        )
        return mamba_data_ptrs, mamba_data_lens, mamba_item_lens
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HybridLinearKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HybridLinearKVPool`。它会向调用方返回计算结果。

### Lines 1490-1492: get_state_dim_per_tensor implementation / get_state_dim_per_tensor 实现
```python
    def get_state_dim_per_tensor(self):
        """Get the sliceable dimension size for each mamba state tensor."""
        return self.mamba_pool.get_state_dim_per_tensor()
```
**EN:** Get the sliceable dimension size for each mamba state tensor. Retrieves the requested data or state from the current object. It belongs to `HybridLinearKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HybridLinearKVPool`。它会向调用方返回计算结果。

### Lines 1494-1495: maybe_get_custom_mem_pool implementation / maybe_get_custom_mem_pool 实现
```python
    def maybe_get_custom_mem_pool(self):
        return self.full_kv_pool.maybe_get_custom_mem_pool()
```
**EN:** Implements the maybe get custom MEM pool routine for this scope. It belongs to `HybridLinearKVPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的maybe get custom mem池例程。 该方法属于 `HybridLinearKVPool`。它会向调用方返回计算结果。

### Lines 1497-1502: _transfer_full_attention_id implementation / _transfer_full_attention_id 实现
```python
    def _transfer_full_attention_id(self, layer_id: int):
        if layer_id not in self.full_attention_layer_id_mapping:
            raise ValueError(
                f"{layer_id=} not in full attention layers: {self.full_attention_layer_id_mapping.keys()}"
            )
        return self.full_attention_layer_id_mapping[layer_id]
```
**EN:** Implements the transfer full attention ID routine for this scope. It belongs to `HybridLinearKVPool`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 实现当前作用域中的transfer完整attention ID例程。 该方法属于 `HybridLinearKVPool`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 1504-1510: register_layer_transfer_counter implementation / register_layer_transfer_counter 实现
```python
    def register_layer_transfer_counter(
        self, layer_transfer_counter: "LayerDoneCounter"
    ):
        self.layer_transfer_counter = layer_transfer_counter
        # The layer-wise wait logic is executed at the Hybrid LinearPool level;
        # no additional wait is needed in the full_kv_pool
        self.full_kv_pool.register_layer_transfer_counter(None)
```
**EN:** Registers metadata so other components can discover this object. It belongs to `HybridLinearKVPool`.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `HybridLinearKVPool`。

### Lines 1512-1514: _wait_for_layer implementation / _wait_for_layer 实现
```python
    def _wait_for_layer(self, layer_id: int):
        if self.layer_transfer_counter is not None:
            self.layer_transfer_counter.wait_until(layer_id - self.start_layer)
```
**EN:** Implements the wait for layer routine for this scope. It belongs to `HybridLinearKVPool`.
**CN:** 实现当前作用域中的wait for layer例程。 该方法属于 `HybridLinearKVPool`。

### Lines 1516-1519: get_key_buffer implementation / get_key_buffer 实现
```python
    def get_key_buffer(self, layer_id: int):
        self._wait_for_layer(layer_id)
        layer_id = self._transfer_full_attention_id(layer_id)
        return self.full_kv_pool.get_key_buffer(layer_id)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HybridLinearKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HybridLinearKVPool`。它会向调用方返回计算结果。

### Lines 1521-1524: get_value_buffer implementation / get_value_buffer 实现
```python
    def get_value_buffer(self, layer_id: int):
        self._wait_for_layer(layer_id)
        layer_id = self._transfer_full_attention_id(layer_id)
        return self.full_kv_pool.get_value_buffer(layer_id)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HybridLinearKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HybridLinearKVPool`。它会向调用方返回计算结果。

### Lines 1526-1529: get_kv_buffer implementation / get_kv_buffer 实现
```python
    def get_kv_buffer(self, layer_id: int):
        self._wait_for_layer(layer_id)
        layer_id = self._transfer_full_attention_id(layer_id)
        return self.full_kv_pool.get_kv_buffer(layer_id)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HybridLinearKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HybridLinearKVPool`。它会向调用方返回计算结果。

### Lines 1531-1544: _transfer_id_context implementation / _transfer_id_context 实现
```python
    @contextmanager
    def _transfer_id_context(self, layer: RadixAttention):

        @contextmanager
        def _patch_layer_id(layer):
            original_layer_id = layer.layer_id
            layer.layer_id = self._transfer_full_attention_id(layer.layer_id)
            try:
                yield
            finally:
                layer.layer_id = original_layer_id

        with _patch_layer_id(layer):
            yield
```
**EN:** Implements the transfer ID context routine for this scope. It belongs to `HybridLinearKVPool`.
**CN:** 实现当前作用域中的transfer ID context例程。 该方法属于 `HybridLinearKVPool`。

### Lines 1546-1573: set_kv_buffer implementation / set_kv_buffer 实现
```python
    def set_kv_buffer(
        self,
        layer: RadixAttention,
        loc: torch.Tensor,
        cache_k: torch.Tensor,
        cache_v: torch.Tensor,
        k_scale: float = 1.0,
        v_scale: float = 1.0,
    ):
        layer_id = self._transfer_full_attention_id(layer.layer_id)
        if not self.use_mla:
            self.full_kv_pool.set_kv_buffer(
# ... omitted for brevity ...
                    loc,
                    cache_k,
                    cache_v,
                )
```
**EN:** Updates internal state with the provided value. It belongs to `HybridLinearKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `HybridLinearKVPool`。

### Lines 1575-1576: move_kv_cache implementation / move_kv_cache 实现
```python
    def move_kv_cache(self, tgt_loc: torch.Tensor, src_loc: torch.Tensor):
        self.full_kv_pool.move_kv_cache(tgt_loc, src_loc)
```
**EN:** Implements the move KV cache routine for this scope. It belongs to `HybridLinearKVPool`.
**CN:** 实现当前作用域中的move KV缓存例程。 该方法属于 `HybridLinearKVPool`。

### Lines 1578-1585: get_cpu_copy implementation / get_cpu_copy 实现
```python
    def get_cpu_copy(self, indices, mamba_indices=None):
        kv_cpu = self.full_kv_pool.get_cpu_copy(indices)
        mamba_cpu = (
            self.mamba_pool.get_cpu_copy(mamba_indices)
            if mamba_indices is not None
            else None
        )
        return kv_cpu, mamba_cpu
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HybridLinearKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HybridLinearKVPool`。它会向调用方返回计算结果。

### Lines 1587-1591: load_cpu_copy implementation / load_cpu_copy 实现
```python
    def load_cpu_copy(self, cache_cpu, indices, mamba_indices=None):
        kv_cpu, mamba_cpu = cache_cpu
        self.full_kv_pool.load_cpu_copy(kv_cpu, indices)
        if mamba_cpu is not None and mamba_indices is not None:
            self.mamba_pool.load_cpu_copy(mamba_cpu, mamba_indices)
```
**EN:** Loads state from an external or serialized representation. It belongs to `HybridLinearKVPool`.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `HybridLinearKVPool`。

### Lines 1593-1594: get_v_head_dim implementation / get_v_head_dim 实现
```python
    def get_v_head_dim(self):
        return self.full_kv_pool.get_value_buffer(0).shape[-1]
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HybridLinearKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HybridLinearKVPool`。它会向调用方返回计算结果。

### Lines 1596-1605: set_mla_kv_buffer implementation / set_mla_kv_buffer 实现
```python
    def set_mla_kv_buffer(
        self,
        layer: RadixAttention,
        loc: torch.Tensor,
        cache_k_nope: torch.Tensor,
        cache_k_rope: torch.Tensor,
    ):
        assert self.use_mla, "set_mla_kv_buffer called when use_mla is False"
        with self._transfer_id_context(layer):
            self.full_kv_pool.set_mla_kv_buffer(layer, loc, cache_k_nope, cache_k_rope)
```
**EN:** Updates internal state with the provided value. It belongs to `HybridLinearKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `HybridLinearKVPool`。

### Lines 1607-1615: get_mla_kv_buffer implementation / get_mla_kv_buffer 实现
```python
    def get_mla_kv_buffer(
        self,
        layer: RadixAttention,
        loc: torch.Tensor,
        dst_dtype: Optional[torch.dtype] = None,
    ):
        assert self.use_mla, "get_mla_kv_buffer called when use_mla is False"
        with self._transfer_id_context(layer):
            return self.full_kv_pool.get_mla_kv_buffer(layer, loc, dst_dtype)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HybridLinearKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HybridLinearKVPool`。它会向调用方返回计算结果。

### Lines 1618-1618: MLATokenToKVPool declaration / MLATokenToKVPool 声明
```python
class MLATokenToKVPool(KVCache):
```
**EN:** Declares the `MLATokenToKVPool` class and connects it to `KVCache`.
**CN:** 声明 `MLATokenToKVPool` 类，并将其关联到 `KVCache`。

### Lines 1619-1670: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        size: int,
        page_size: int,
        dtype: torch.dtype,
        kv_lora_rank: int,
        qk_rope_head_dim: int,
        layer_num: int,
        device: str,
        enable_memory_saver: bool,
        start_layer: Optional[int] = None,
        end_layer: Optional[int] = None,
# ... omitted for brevity ...
        )
        if not use_nsa:
            # NSA will allocate indexer KV cache later and then log the total size
            self._finalize_allocation_log(size)
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `MLATokenToKVPool`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `MLATokenToKVPool`。

### Lines 1672-1687: _create_buffers implementation / _create_buffers 实现
```python
    def _create_buffers(self):
        with self.memory_saver_adapter.region(GPU_MEMORY_TYPE_KV_CACHE):
            with (
                torch.cuda.use_mem_pool(self.custom_mem_pool)
                if self.custom_mem_pool
                else nullcontext()
            ):
                # The padded slot 0 is used for writing dummy outputs from padded tokens.
                self.kv_buffer = [
                    torch.zeros(
                        (self.size + self.page_size, 1, self.kv_cache_dim),
                        dtype=self.store_dtype,
                        device=self.device,
                    )
                    for _ in range(self.layer_num)
                ]
```
**EN:** Implements the create buffers routine for this scope. It belongs to `MLATokenToKVPool`.
**CN:** 实现当前作用域中的create buffers例程。 该方法属于 `MLATokenToKVPool`。

### Lines 1689-1690: _clear_buffers implementation / _clear_buffers 实现
```python
    def _clear_buffers(self):
        del self.kv_buffer
```
**EN:** Implements the clear buffers routine for this scope. It belongs to `MLATokenToKVPool`.
**CN:** 实现当前作用域中的clear buffers例程。 该方法属于 `MLATokenToKVPool`。

### Lines 1692-1697: get_kv_size_bytes implementation / get_kv_size_bytes 实现
```python
    def get_kv_size_bytes(self):
        assert hasattr(self, "kv_buffer")
        kv_size_bytes = 0
        for kv_cache in self.kv_buffer:
            kv_size_bytes += get_tensor_size_bytes(kv_cache)
        return kv_size_bytes
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MLATokenToKVPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MLATokenToKVPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1698-1699: Comment block / 注释块
```python

    # for disagg
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 1700-1707: get_contiguous_buf_infos implementation / get_contiguous_buf_infos 实现
```python
    def get_contiguous_buf_infos(self):
        # MLA has only one kv_buffer, so only the information of this buffer needs to be returned.
        kv_data_ptrs = [self.kv_buffer[i].data_ptr() for i in range(self.layer_num)]
        kv_data_lens = [self.kv_buffer[i].nbytes for i in range(self.layer_num)]
        kv_item_lens = [
            self.kv_buffer[i][0].nbytes * self.page_size for i in range(self.layer_num)
        ]
        return kv_data_ptrs, kv_data_lens, kv_item_lens
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MLATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MLATokenToKVPool`。它会向调用方返回计算结果。

### Lines 1709-1716: get_key_buffer implementation / get_key_buffer 实现
```python
    def get_key_buffer(self, layer_id: int):
        if self.layer_transfer_counter is not None:
            self.layer_transfer_counter.wait_until(layer_id - self.start_layer)

        if self.store_dtype != self.dtype:
            return self.kv_buffer[layer_id - self.start_layer].view(self.dtype)

        return self.kv_buffer[layer_id - self.start_layer]
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MLATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MLATokenToKVPool`。它会向调用方返回计算结果。

### Lines 1718-1726: get_value_buffer implementation / get_value_buffer 实现
```python
    def get_value_buffer(self, layer_id: int):
        if self.layer_transfer_counter is not None:
            self.layer_transfer_counter.wait_until(layer_id - self.start_layer)

        if self.store_dtype != self.dtype:
            return self.kv_buffer[layer_id - self.start_layer][
                ..., : self.kv_lora_rank
            ].view(self.dtype)
        return self.kv_buffer[layer_id - self.start_layer][..., : self.kv_lora_rank]
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MLATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MLATokenToKVPool`。它会向调用方返回计算结果。

### Lines 1728-1729: get_kv_buffer implementation / get_kv_buffer 实现
```python
    def get_kv_buffer(self, layer_id: int):
        return self.get_key_buffer(layer_id), self.get_value_buffer(layer_id)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MLATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MLATokenToKVPool`。它会向调用方返回计算结果。

### Lines 1731-1748: set_kv_buffer implementation / set_kv_buffer 实现
```python
    def set_kv_buffer(
        self,
        layer: RadixAttention,
        loc: torch.Tensor,
        cache_k: torch.Tensor,
        cache_v: torch.Tensor,
    ):
        layer_id = layer.layer_id
        assert not self.nsa_kv_cache_store_fp8
        if cache_k.dtype != self.dtype:
            cache_k = cache_k.to(self.dtype)

        if self.store_dtype != self.dtype:
            self.kv_buffer[layer_id - self.start_layer][loc] = cache_k.view(
                self.store_dtype
            )
        else:
            self.kv_buffer[layer_id - self.start_layer][loc] = cache_k
```
**EN:** Updates internal state with the provided value. It belongs to `MLATokenToKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `MLATokenToKVPool`。

### Lines 1750-1799: set_mla_kv_buffer implementation / set_mla_kv_buffer 实现
```python
    def set_mla_kv_buffer(
        self,
        layer: RadixAttention,
        loc: torch.Tensor,
        cache_k_nope: torch.Tensor,
        cache_k_rope: torch.Tensor,
    ):
        layer_id = layer.layer_id

        if _is_hip and self.use_nsa and self.dtype == fp8_dtype:
            # HIP FP8 path uses raw MLA KV layout (nope + rope) without per-block scales.
            # Fuse BF16/FP16 -> FP8 cast with paged KV write.
# ... omitted for brevity ...
                loc,
                cache_k_nope,
                cache_k_rope,
            )
```
**EN:** Updates internal state with the provided value. It belongs to `MLATokenToKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `MLATokenToKVPool`。

### Lines 1801-1822: get_mla_kv_buffer implementation / get_mla_kv_buffer 实现
```python
    def get_mla_kv_buffer(
        self,
        layer: RadixAttention,
        loc: torch.Tensor,
        dst_dtype: Optional[torch.dtype] = None,
    ):
        # get k nope and k rope from the kv buffer, and optionally cast them to dst_dtype.
        layer_id = layer.layer_id
        kv_buffer = self.get_key_buffer(layer_id)
        dst_dtype = dst_dtype or self.dtype
        cache_k_nope = torch.empty(
            (loc.shape[0], 1, self.kv_lora_rank),
# ... omitted for brevity ...
            device=kv_buffer.device,
        )
        get_mla_kv_buffer_triton(kv_buffer, loc, cache_k_nope, cache_k_rope)
        return cache_k_nope, cache_k_rope
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MLATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MLATokenToKVPool`。它会向调用方返回计算结果。

### Lines 1824-1837: get_cpu_copy implementation / get_cpu_copy 实现
```python
    def get_cpu_copy(self, indices, mamba_indices=None):
        current_platform.synchronize()
        kv_cache_cpu = []
        chunk_size = self.cpu_offloading_chunk_size
        for layer_id in range(self.layer_num):
            kv_cache_cpu.append([])
            for i in range(0, len(indices), chunk_size):
                chunk_indices = indices[i : i + chunk_size]
                kv_cpu = self.kv_buffer[layer_id][chunk_indices].to(
                    "cpu", non_blocking=True
                )
                kv_cache_cpu[-1].append(kv_cpu)
        current_platform.synchronize()
        return kv_cache_cpu
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MLATokenToKVPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MLATokenToKVPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1839-1849: load_cpu_copy implementation / load_cpu_copy 实现
```python
    def load_cpu_copy(self, kv_cache_cpu, indices, mamba_indices=None):
        current_platform.synchronize()
        chunk_size = self.cpu_offloading_chunk_size
        for layer_id in range(self.layer_num):
            for i in range(0, len(indices), chunk_size):
                chunk_indices = indices[i : i + chunk_size]
                kv_cpu = kv_cache_cpu[layer_id][i // chunk_size]
                assert kv_cpu.shape[0] == len(chunk_indices)
                kv_chunk = kv_cpu.to(self.kv_buffer[0].device, non_blocking=True)
                self.kv_buffer[layer_id][chunk_indices] = kv_chunk
        current_platform.synchronize()
```
**EN:** Loads state from an external or serialized representation. It belongs to `MLATokenToKVPool`. The implementation iterates over inputs or managed entries.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `MLATokenToKVPool`。实现过程中会遍历输入或受管条目。

### Lines 1852-1853: MLATokenToKVPoolFP4 declaration / MLATokenToKVPoolFP4 声明
```python
class MLATokenToKVPoolFP4(MLATokenToKVPool):

```
**EN:** Declares the `MLATokenToKVPoolFP4` class and connects it to `MLATokenToKVPool`.
**CN:** 声明 `MLATokenToKVPoolFP4` 类，并将其关联到 `MLATokenToKVPool`。

### Lines 1854-1885: _create_buffers implementation / _create_buffers 实现
```python
    def _create_buffers(self):
        with self.memory_saver_adapter.region(GPU_MEMORY_TYPE_KV_CACHE):
            with (
                torch.cuda.use_mem_pool(self.custom_mem_pool)
                if self.custom_mem_pool
                else nullcontext()
            ):
                # The padded slot 0 is used for writing dummy outputs from padded tokens.
                m = self.size + self.page_size
                n = 1  # head_num
                k = self.kv_cache_dim  # head_dim

# ... omitted for brevity ...
                        device=self.device,
                    )
                    for _ in range(self.layer_num)
                ]
```
**EN:** Implements the create buffers routine for this scope. It belongs to `MLATokenToKVPoolFP4`.
**CN:** 实现当前作用域中的create buffers例程。 该方法属于 `MLATokenToKVPoolFP4`。

### Lines 1887-1889: _clear_buffers implementation / _clear_buffers 实现
```python
    def _clear_buffers(self):
        del self.kv_buffer
        del self.kv_scale_buffer
```
**EN:** Implements the clear buffers routine for this scope. It belongs to `MLATokenToKVPoolFP4`.
**CN:** 实现当前作用域中的clear buffers例程。 该方法属于 `MLATokenToKVPoolFP4`。

### Lines 1891-1908: get_key_buffer implementation / get_key_buffer 实现
```python
    def get_key_buffer(self, layer_id: int):
        if self.layer_transfer_counter is not None:
            self.layer_transfer_counter.wait_until(layer_id - self.start_layer)

        if self.store_dtype != self.dtype:
            cache_k_nope_fp4 = self.kv_buffer[layer_id - self.start_layer].view(
                torch.uint8
            )
            cache_k_nope_fp4_sf = self.kv_scale_buffer[layer_id - self.start_layer]

            from sglang.srt.layers.quantization.kvfp4_tensor import KVFP4QuantizeUtil

            cache_k_nope_fp4_dequant = KVFP4QuantizeUtil.batched_dequantize(
                cache_k_nope_fp4, cache_k_nope_fp4_sf
            )
            return cache_k_nope_fp4_dequant

        return self.kv_buffer[layer_id - self.start_layer]
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MLATokenToKVPoolFP4`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MLATokenToKVPoolFP4`。它会向调用方返回计算结果。

### Lines 1910-1932: set_kv_buffer implementation / set_kv_buffer 实现
```python
    def set_kv_buffer(
        self,
        layer: RadixAttention,
        loc: torch.Tensor,
        cache_k: torch.Tensor,
        cache_v: torch.Tensor,
    ):
        layer_id = layer.layer_id
        assert not self.nsa_kv_cache_store_fp8
        if cache_k.dtype != self.dtype:
            from sglang.srt.layers.quantization.kvfp4_tensor import KVFP4QuantizeUtil

# ... omitted for brevity ...
                cache_k_fp4_sf.view(self.store_dtype)
            )
        else:
            self.kv_buffer[layer_id - self.start_layer][loc] = cache_k
```
**EN:** Updates internal state with the provided value. It belongs to `MLATokenToKVPoolFP4`.
**CN:** 使用给定值更新内部状态。 该方法属于 `MLATokenToKVPoolFP4`。

### Lines 1934-1978: set_mla_kv_buffer implementation / set_mla_kv_buffer 实现
```python
    def set_mla_kv_buffer(
        self,
        layer: RadixAttention,
        loc: torch.Tensor,
        cache_k_nope: torch.Tensor,
        cache_k_rope: torch.Tensor,
    ):
        layer_id = layer.layer_id

        if self.nsa_kv_cache_store_fp8:
            # original cache_k: (num_tokens, num_heads 1, hidden 576); we unsqueeze the page_size=1 dim here
            # TODO no need to cat
# ... omitted for brevity ...
                loc,
                cache_k_nope_fp4_sf,
                cache_k_rope_fp4_sf,
            )
```
**EN:** Updates internal state with the provided value. It belongs to `MLATokenToKVPoolFP4`.
**CN:** 使用给定值更新内部状态。 该方法属于 `MLATokenToKVPoolFP4`。

### Lines 1981-1981: NSATokenToKVPool declaration / NSATokenToKVPool 声明
```python
class NSATokenToKVPool(MLATokenToKVPool):
```
**EN:** Declares the `NSATokenToKVPool` class and connects it to `MLATokenToKVPool`.
**CN:** 声明 `NSATokenToKVPool` 类，并将其关联到 `MLATokenToKVPool`。

### Lines 1982-1984: Shared state definitions / 共享状态定义
```python
    quant_block_size = 128
    index_k_with_scale_buffer_dtype = torch.uint8
    rope_storage_dtype = torch.bfloat16  # rope is always stored in bf16
```
**EN:** Defines class-level variables such as `quant_block_size`, `index_k_with_scale_buffer_dtype`, `rope_storage_dtype`.
**CN:** 定义类级变量，例如 `quant_block_size`, `index_k_with_scale_buffer_dtype`, `rope_storage_dtype`。

### Lines 1986-2065: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        size: int,
        page_size: int,
        kv_lora_rank: int,
        dtype: torch.dtype,
        qk_rope_head_dim: int,
        layer_num: int,
        device: str,
        index_head_dim: int,
        enable_memory_saver: bool,
        kv_cache_dim: int,
# ... omitted for brevity ...
                )
                for _ in range(layer_num)
            ]
        self._finalize_allocation_log(size)
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `NSATokenToKVPool`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `NSATokenToKVPool`。

### Lines 2067-2069: _clear_buffers implementation / _clear_buffers 实现
```python
    def _clear_buffers(self):
        del self.kv_buffer
        del self.index_k_with_scale_buffer
```
**EN:** Implements the clear buffers routine for this scope. It belongs to `NSATokenToKVPool`.
**CN:** 实现当前作用域中的clear buffers例程。 该方法属于 `NSATokenToKVPool`。

### Lines 2071-2074: get_index_k_with_scale_buffer implementation / get_index_k_with_scale_buffer 实现
```python
    def get_index_k_with_scale_buffer(self, layer_id: int) -> torch.Tensor:
        if self.layer_transfer_counter is not None:
            self.layer_transfer_counter.wait_until(layer_id - self.start_layer)
        return self.index_k_with_scale_buffer[layer_id - self.start_layer]
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `NSATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `NSATokenToKVPool`。它会向调用方返回计算结果。

### Lines 2076-2087: get_index_k_continuous implementation / get_index_k_continuous 实现
```python
    def get_index_k_continuous(
        self,
        layer_id: int,
        seq_len: int,
        page_indices: torch.Tensor,
    ):
        if self.layer_transfer_counter is not None:
            self.layer_transfer_counter.wait_until(layer_id - self.start_layer)
        buf = self.index_k_with_scale_buffer[layer_id - self.start_layer]
        return index_buf_accessor.GetK.execute(
            self, buf, seq_len=seq_len, page_indices=page_indices
        )
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `NSATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `NSATokenToKVPool`。它会向调用方返回计算结果。

### Lines 2089-2100: get_index_k_scale_continuous implementation / get_index_k_scale_continuous 实现
```python
    def get_index_k_scale_continuous(
        self,
        layer_id: int,
        seq_len: int,
        page_indices: torch.Tensor,
    ):
        if self.layer_transfer_counter is not None:
            self.layer_transfer_counter.wait_until(layer_id - self.start_layer)
        buf = self.index_k_with_scale_buffer[layer_id - self.start_layer]
        return index_buf_accessor.GetS.execute(
            self, buf, seq_len=seq_len, page_indices=page_indices
        )
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `NSATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `NSATokenToKVPool`。它会向调用方返回计算结果。

### Lines 2102-2131: get_index_k_scale_buffer implementation / get_index_k_scale_buffer 实现
```python
    def get_index_k_scale_buffer(
        self,
        layer_id: int,
        seq_len_tensor: torch.Tensor,
        page_indices: torch.Tensor,
        seq_len_sum: int,
        max_seq_len: int,
    ):
        """
        Fused method to get both index K and scale data in a single call using Triton.
        More efficient than calling get_index_k_continuous and get_index_k_scale_continuous separately.

# ... omitted for brevity ...
            seq_len_tensor=seq_len_tensor,
            seq_len_sum=seq_len_sum,
            max_seq_len=max_seq_len,
        )
```
**EN:** Fused method to get both index K and scale data in a single call using Triton. Retrieves the requested data or state from the current object. It belongs to `NSATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `NSATokenToKVPool`。它会向调用方返回计算结果。

### Lines 2133-2143: set_index_k_scale_buffer implementation / set_index_k_scale_buffer 实现
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
**EN:** Updates internal state with the provided value. It belongs to `NSATokenToKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `NSATokenToKVPool`。

### Lines 2145-2168: get_cpu_copy implementation / get_cpu_copy 实现
```python
    def get_cpu_copy(self, indices):
        # NSA keeps a page-indexed index_k_with_scale_buffer alongside kv_buffer.
        # Retract frees the slots/pages and they get reused by other reqs'
        # set_index_k_scale_buffer, so we must offload it here too -- otherwise
        # resume restores kv_buffer but leaves foreign index/scale in place and
        # NSA attention reads garbage at those token positions.
        kv_cache_cpu = super().get_cpu_copy(indices)

        page_indices = indices[:: self.page_size] // self.page_size
        torch.cuda.synchronize()
        index_k_cpu = []
        chunk_size = self.cpu_offloading_chunk_size
# ... omitted for brevity ...
                index_k_cpu[-1].append(idx_cpu)
        torch.cuda.synchronize()

        return {"kv": kv_cache_cpu, "index_k": index_k_cpu}
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `NSATokenToKVPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `NSATokenToKVPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 2170-2187: load_cpu_copy implementation / load_cpu_copy 实现
```python
    def load_cpu_copy(self, kv_cache_cpu_dict, indices):
        super().load_cpu_copy(kv_cache_cpu_dict["kv"], indices)

        page_indices = indices[:: self.page_size] // self.page_size
        index_k_cpu = kv_cache_cpu_dict["index_k"]
        torch.cuda.synchronize()
        chunk_size = self.cpu_offloading_chunk_size
        page_chunk_size = max(1, chunk_size // self.page_size)
        for layer_id in range(self.layer_num):
            for i in range(0, len(page_indices), page_chunk_size):
                chunk_page_indices = page_indices[i : i + page_chunk_size]
                idx_cpu = index_k_cpu[layer_id][i // page_chunk_size]
                assert idx_cpu.shape[0] == len(chunk_page_indices)
                idx_chunk = idx_cpu.to(
                    self.index_k_with_scale_buffer[0].device, non_blocking=True
                )
                self.index_k_with_scale_buffer[layer_id][chunk_page_indices] = idx_chunk
        torch.cuda.synchronize()
```
**EN:** Loads state from an external or serialized representation. It belongs to `NSATokenToKVPool`. The implementation iterates over inputs or managed entries.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `NSATokenToKVPool`。实现过程中会遍历输入或受管条目。

### Lines 2189-2199: get_state_buf_infos implementation / get_state_buf_infos 实现
```python
    def get_state_buf_infos(self):
        data_ptrs = [
            self.index_k_with_scale_buffer[i].data_ptr() for i in range(self.layer_num)
        ]
        data_lens = [
            self.index_k_with_scale_buffer[i].nbytes for i in range(self.layer_num)
        ]
        item_lens = [
            self.index_k_with_scale_buffer[i][0].nbytes for i in range(self.layer_num)
        ]
        return data_ptrs, data_lens, item_lens
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `NSATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `NSATokenToKVPool`。它会向调用方返回计算结果。

### Lines 2201-2205: get_kv_size_bytes implementation / get_kv_size_bytes 实现
```python
    def get_kv_size_bytes(self):
        kv_size_bytes = super().get_kv_size_bytes()
        for index_k_cache in self.index_k_with_scale_buffer:
            kv_size_bytes += get_tensor_size_bytes(index_k_cache)
        return kv_size_bytes
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `NSATokenToKVPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `NSATokenToKVPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 2208-2221: move_kv_cache_native implementation / move_kv_cache_native 实现
```python
def move_kv_cache_native(
    k_buffer: List[torch.Tensor],
    v_buffer: List[torch.Tensor],
    tgt_loc: torch.Tensor,
    src_loc: torch.Tensor,
):
    if tgt_loc.numel() == 0:
        return

    tgt_loc_flat = tgt_loc.view(-1).long()
    src_loc_flat = src_loc.view(-1).long()
    for k_cache, v_cache in zip(k_buffer, v_buffer):
        k_cache[tgt_loc_flat] = k_cache[src_loc_flat]
        v_cache[tgt_loc_flat] = v_cache[src_loc_flat]
```
**EN:** Implements the move KV cache native routine for this scope. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的move KV缓存native例程。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 2224-2257: copy_all_layer_kv_cache_tiled implementation / copy_all_layer_kv_cache_tiled 实现
```python
@triton.jit
def copy_all_layer_kv_cache_tiled(
    data_ptrs,
    strides,
    tgt_loc_ptr,
    src_loc_ptr,
    num_locs,
    num_locs_upper: tl.constexpr,
    BYTES_PER_TILE: tl.constexpr,
):
    """2D tiled kernel. Safe for in-place copy."""
    bid = tl.program_id(0)
# ... omitted for brevity ...

    mask = mask_loc[:, None] & mask_byte[None, :]
    vals = tl.load(src_ptr, mask=mask)
    tl.store(tgt_ptr, vals, mask=mask)
```
**EN:** 2D tiled kernel. Copies data into a new location or representation.
**CN:** 将数据复制到新的位置或表示中。

## Key Concepts / 关键概念
- **`get_tensor_size_bytes`**: Provides the `get_tensor_size_bytes` entry point for module-level behavior. / 提供模块级行为的 `get_tensor_size_bytes` 入口。
- **`_set_kv_buffer_impl`**: Provides the `_set_kv_buffer_impl` entry point for module-level behavior. / 提供模块级行为的 `_set_kv_buffer_impl` 入口。
- **`ReqToTokenPool`**: Defines the `ReqToTokenPool` type and its core responsibilities. / 定义 `ReqToTokenPool` 类型及其核心职责。
- **`MambaPool`**: Defines the `MambaPool` type and its core responsibilities. / 定义 `MambaPool` 类型及其核心职责。
- **`HybridReqToTokenPool`**: Defines the `HybridReqToTokenPool` type and its core responsibilities. / 定义 `HybridReqToTokenPool` 类型及其核心职责。
- **`KVCache`**: Defines the `KVCache` type and its core responsibilities. / 定义 `KVCache` 类型及其核心职责。
- **`MHATokenToKVPool`**: Defines the `MHATokenToKVPool` type and its core responsibilities. / 定义 `MHATokenToKVPool` 类型及其核心职责。
- **`NoOpMHATokenToKVPool`**: Defines the `NoOpMHATokenToKVPool` type and its core responsibilities. / 定义 `NoOpMHATokenToKVPool` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `abc`, `dataclasses`, `logging`, `contextlib`, `typing`, `numpy`, `torch`, `triton`, `triton.language`
- **Internal / 内部**: `sglang.jit_kernel.kvcache`, `sglang.srt.configs.mamba_utils`, `sglang.srt.constants`, `sglang.srt.environ`, `sglang.srt.layers.attention.nsa`, `sglang.srt.layers.attention.nsa.quant_k_cache`, `sglang.srt.layers.attention.nsa.utils`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.srt.layers.radix_attention`, `sglang.srt.mem_cache.utils`, `sglang.srt.platforms`, `sglang.srt.utils`
