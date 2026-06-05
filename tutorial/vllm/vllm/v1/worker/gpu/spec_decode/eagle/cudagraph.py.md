# cudagraph.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/spec_decode/eagle/cudagraph.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `EagleCudaGraphManagerBase`, `PrefillEagleCudaGraphManager`, `DecodeEagleCudaGraphManager` for the V1 `worker/gpu/spec_decode/eagle` subsystem. / 为 V1 的 `worker/gpu/spec_decode/eagle` 子系统实现 `EagleCudaGraphManagerBase`, `PrefillEagleCudaGraphManager`, `DecodeEagleCudaGraphManager`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from collections.abc import Callable

import torch

from vllm.config import VllmConfig
from vllm.config.compilation import CUDAGraphMode
from vllm.v1.kv_cache_interface import KVCacheConfig
from vllm.v1.worker.gpu.block_table import BlockTables
from vllm.v1.worker.gpu.cudagraph_utils import (
    BatchExecutionDescriptor,
    CapturedAttentionState,
    CudaGraphManager,
    prepare_inputs_to_capture,
)
from vllm.v1.worker.gpu.input_batch import InputBuffers
from vllm.v1.worker.gpu.model_states.interface import ModelState
from vllm.v1.worker.utils import AttentionGroup
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.config`, `vllm.config.compilation`, `vllm.v1.kv_cache_interface`, `vllm.v1.worker.gpu.block_table`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.config`, `vllm.config.compilation`, `vllm.v1.kv_cache_interface`, `vllm.v1.worker.gpu.block_table` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `EagleCudaGraphManagerBase` class / `EagleCudaGraphManagerBase` 类
```python
class EagleCudaGraphManagerBase(CudaGraphManager):
    """Base CudaGraphManager for Eagle with a dedicated graph pool."""
```
**EN:** Introduces the `EagleCudaGraphManagerBase` class on top of `CudaGraphManager`. Core methods include `__init__`. Docstring signal: Base CudaGraphManager for Eagle with a dedicated graph pool.
**CN:** 这里定义 `EagleCudaGraphManagerBase` 类，其基类包括 `CudaGraphManager`。核心方法包括 `__init__`。

### `EagleCudaGraphManagerBase.__init__` method / `EagleCudaGraphManagerBase.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        device: torch.device,
        cudagraph_mode: CUDAGraphMode,
        decode_query_len: int,
    ):
        super().__init__(vllm_config, device, cudagraph_mode, decode_query_len)

        # Use a dedicated pool for Eagle to avoid memory overlap with the main
        # model's cudagraph. The base class uses a shared global pool, but Eagle's
        # internal allocations (e.g., gumbel_sample temporaries) can conflict with
        # the main model's allocations when sharing the same pool.
        if cudagraph_mode:
            self.pool = torch.cuda.graph_pool_handle()
```
**EN:** This method initializes the object state within `EagleCudaGraphManagerBase`. Key calls include `__init__`, `graph_pool_handle`, `super`. It touches state such as `pool`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`EagleCudaGraphManagerBase`。 关键调用包括 `__init__`, `graph_pool_handle`, `super`。 它会读写 `pool` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `PrefillEagleCudaGraphManager` class / `PrefillEagleCudaGraphManager` 类
```python
class PrefillEagleCudaGraphManager(EagleCudaGraphManagerBase):
    """Eagle CudaGraphManager for prefill, using pre-built attention states
    from the target model's capture."""
```
**EN:** Introduces the `PrefillEagleCudaGraphManager` class on top of `EagleCudaGraphManagerBase`. Core methods include `capture`. Docstring signal: Eagle CudaGraphManager for prefill, using pre-built attention states from the target model's capture.
**CN:** 这里定义 `PrefillEagleCudaGraphManager` 类，其基类包括 `EagleCudaGraphManagerBase`。核心方法包括 `capture`。

### `PrefillEagleCudaGraphManager.capture` method / `PrefillEagleCudaGraphManager.capture` 方法
```python
    def capture(
        self,
        forward_fn: Callable,
        full_cg_attn_states: dict[BatchExecutionDescriptor, CapturedAttentionState],
        progress_bar_desc: str = "Capturing CUDA graphs",
    ) -> None:
        def create_forward_fn(
            desc: BatchExecutionDescriptor,
        ) -> tuple[Callable[[CUDAGraphMode], None], CapturedAttentionState]:
            num_tokens = desc.num_tokens
            num_reqs = desc.num_reqs or min(num_tokens, self.max_num_reqs)
            num_tokens_across_dp = (
                torch.full((self.dp_size,), num_tokens, dtype=torch.int32, device="cpu")
                if self.dp_size > 1
                else None
            )
            attn_state = full_cg_attn_states[desc]
            attn_metadata, slot_mappings = attn_state
            fwd = lambda cg_mode: forward_fn(
                num_reqs,
                num_tokens,
                attn_metadata,
                slot_mappings,
                num_tokens_across_dp,
                cg_mode,
            )
            return fwd, attn_state

        super().capture(create_forward_fn, progress_bar_desc)
```
**EN:** This method implements `capture` within `PrefillEagleCudaGraphManager`. Key calls include `capture`, `min`, `full`, `forward_fn`, `super`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `capture`，其作用域位于`PrefillEagleCudaGraphManager`。 关键调用包括 `capture`, `min`, `full`, `forward_fn`, `super`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DecodeEagleCudaGraphManager` class / `DecodeEagleCudaGraphManager` 类
```python
class DecodeEagleCudaGraphManager(EagleCudaGraphManagerBase):
    """Eagle CudaGraphManager for decode draft generation, building its own
    attention metadata from scratch."""
```
**EN:** Introduces the `DecodeEagleCudaGraphManager` class on top of `EagleCudaGraphManagerBase`. Core methods include `capture`. Docstring signal: Eagle CudaGraphManager for decode draft generation, building its own attention metadata from scratch.
**CN:** 这里定义 `DecodeEagleCudaGraphManager` 类，其基类包括 `EagleCudaGraphManagerBase`。核心方法包括 `capture`。

### `DecodeEagleCudaGraphManager.capture` method / `DecodeEagleCudaGraphManager.capture` 方法
```python
    def capture(
        self,
        forward_fn: Callable,
        model_state: ModelState,
        input_buffers: InputBuffers,
        block_tables: BlockTables,
        attn_groups: list[list[AttentionGroup]],
        kv_cache_config: KVCacheConfig,
        progress_bar_desc: str = "Capturing CUDA graphs",
    ) -> None:
        def create_forward_fn(
            desc: BatchExecutionDescriptor,
        ) -> tuple[Callable[[CUDAGraphMode], None], CapturedAttentionState]:
            num_tokens = desc.num_tokens
            num_reqs = desc.num_reqs or min(num_tokens, self.max_num_reqs)
            num_tokens_across_dp = (
                torch.full((self.dp_size,), num_tokens, dtype=torch.int32, device="cpu")
                if self.dp_size > 1
                else None
            )
            attn_state = prepare_inputs_to_capture(
                num_reqs,
                num_tokens,
                model_state,
                input_buffers,
                block_tables,
                attn_groups,
                kv_cache_config,
                skip_attn=(desc.cg_mode == CUDAGraphMode.PIECEWISE),
            )
            attn_metadata, slot_mappings = attn_state

            fwd = lambda cg_mode: forward_fn(
                num_reqs,
                num_tokens,
                attn_metadata,
                slot_mappings,
                num_tokens_across_dp,
                cg_mode,
            )
            return fwd, attn_state

        super().capture(create_forward_fn, progress_bar_desc)
```
**EN:** This method implements `capture` within `DecodeEagleCudaGraphManager`. Key calls include `capture`, `prepare_inputs_to_capture`, `min`, `full`, `forward_fn`, `super`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `capture`，其作用域位于`DecodeEagleCudaGraphManager`。 关键调用包括 `capture`, `prepare_inputs_to_capture`, `min`, `full`, `forward_fn`, `super`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `EagleCudaGraphManagerBase`: central class or interface in this module. / `EagleCudaGraphManagerBase`：本模块中的核心类或接口。
- `PrefillEagleCudaGraphManager`: central class or interface in this module. / `PrefillEagleCudaGraphManager`：本模块中的核心类或接口。
- `DecodeEagleCudaGraphManager`: central class or interface in this module. / `DecodeEagleCudaGraphManager`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `collections`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.config.compilation`, `vllm.v1.kv_cache_interface`, `vllm.v1.worker.gpu.block_table`, `vllm.v1.worker.gpu.cudagraph_utils`, `vllm.v1.worker.gpu.input_batch`, `vllm.v1.worker.gpu.model_states.interface`, `vllm.v1.worker.utils`
