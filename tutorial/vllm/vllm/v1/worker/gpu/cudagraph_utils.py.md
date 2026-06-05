# cudagraph_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/cudagraph_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `CapturedAttentionState`, `BatchExecutionDescriptor`, `_is_compatible` for the V1 `worker/gpu` subsystem. / 为 V1 的 `worker/gpu` 子系统实现 `CapturedAttentionState`, `BatchExecutionDescriptor`, `_is_compatible`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from collections import defaultdict
from collections.abc import Callable
from dataclasses import dataclass
from typing import Any, NamedTuple

import torch
import torch.nn as nn
from tqdm import tqdm

from vllm.compilation.counter import compilation_counter
from vllm.config import VllmConfig
from vllm.config.compilation import CUDAGraphMode
from vllm.distributed.parallel_state import (
    get_pp_group,
    graph_capture,
    is_global_first_rank,
)
from vllm.forward_context import BatchDescriptor, set_forward_context
from vllm.logger import init_logger
from vllm.model_executor.offloader.base import get_offloader
from vllm.platforms import current_platform
from vllm.sequence import IntermediateTensors
from vllm.v1.kv_cache_interface import KVCacheConfig
from vllm.v1.worker.gpu.attn_utils import build_slot_mappings_by_layer
from vllm.v1.worker.gpu.block_table import BlockTables
from vllm.v1.worker.gpu.cp_utils import prepare_dcp_local_seq_lens
from vllm.v1.worker.gpu.input_batch import InputBatch, InputBuffers
from vllm.v1.worker.gpu.model_states.interface import ModelState
from vllm.v1.worker.utils import AttentionGroup

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `CapturedAttentionState` class / `CapturedAttentionState` 类
```python
class CapturedAttentionState(NamedTuple):
    attn_metadata: dict[str, Any] | None
    slot_mappings: dict[str, torch.Tensor]
```
**EN:** Introduces the `CapturedAttentionState` class on top of `NamedTuple`. Core methods include its methods defined below.
**CN:** 这里定义 `CapturedAttentionState` 类，其基类包括 `NamedTuple`。核心方法包括 下方定义的方法。

### `BatchExecutionDescriptor` class / `BatchExecutionDescriptor` 类
```python
@dataclass(frozen=True)
class BatchExecutionDescriptor:
    """Describes the shape of the batch and CG mode to run; this is used to make shape
    matches between the capture and runtime."""

    cg_mode: CUDAGraphMode
    num_tokens: int
    num_reqs: int | None  # None means no request padding is needed (PIECEWISE graphs)
    uniform_token_count: int | None = None
```
**EN:** Introduces the `BatchExecutionDescriptor` class. Core methods include its methods defined below. Docstring signal: Describes the shape of the batch and CG mode to run; this is used to make shape matches between the capture and runtime.
**CN:** 这里定义 `BatchExecutionDescriptor` 类。核心方法包括 下方定义的方法。

### `_is_compatible` function / `_is_compatible` 函数
```python
def _is_compatible(
    desc: BatchExecutionDescriptor,
    num_reqs: int,
    num_tokens: int,
    uniform_token_count: int | None,
) -> bool:
    # desc.uniform_token_count=None (PIECEWISE) can handle any uniform_token_count
    # desc.num_reqs=None means no request padding needed (PIECEWISE)
    return (
        (
            desc.uniform_token_count is None
            or desc.uniform_token_count == uniform_token_count
        )
        and (desc.num_reqs is None or desc.num_reqs >= num_reqs)
        and desc.num_tokens >= num_tokens
    )
```
**EN:** This function implements `_is_compatible` within the module.
**CN:** 该函数会实现 `_is_compatible`，其作用域位于the module。

### `get_uniform_token_count` function / `get_uniform_token_count` 函数
```python
def get_uniform_token_count(
    num_reqs: int,
    num_tokens: int,
    max_query_len: int,
) -> int | None:
    """
    Return the uniform token count if batch is uniform, else None.
    A batch is uniform if all requests have the same number of tokens.
    """
    if (max_query_len == num_tokens // num_reqs) and (
        num_tokens == max_query_len * num_reqs
    ):
        return max_query_len
    return None
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Return the uniform token count if batch is uniform, else None. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CudaGraphManager` class / `CudaGraphManager` 类
```python
class CudaGraphManager:
```
**EN:** Introduces the `CudaGraphManager` class. Core methods include `__init__`, `_init_candidates`, `needs_capture`, `capture`, `dispatch`, `run_fullgraph`.
**CN:** 这里定义 `CudaGraphManager` 类。核心方法包括 `__init__`, `_init_candidates`, `needs_capture`, `capture`, `dispatch`, `run_fullgraph`。

### `CudaGraphManager.__init__` method / `CudaGraphManager.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        device: torch.device,
        cudagraph_mode: CUDAGraphMode,
        decode_query_len: int,
    ):
        self.vllm_config = vllm_config
        self.device = device
        self.max_num_reqs = vllm_config.scheduler_config.max_num_seqs
        self.compilation_config = vllm_config.compilation_config
        assert self.compilation_config is not None
        self.cudagraph_mode = cudagraph_mode
        self.decode_query_len = decode_query_len

        self.dp_size = vllm_config.parallel_config.data_parallel_size
        self.tp_size = vllm_config.parallel_config.tensor_parallel_size
        self.is_first_pp_rank = get_pp_group().is_first_rank
        self.is_last_pp_rank = get_pp_group().is_last_rank

        self.graphs: dict[BatchExecutionDescriptor, torch.cuda.CUDAGraph] = {}
        self.pool = current_platform.get_global_graph_pool() if cudagraph_mode else None

        self._graphs_captured = False
        self._candidates: list[list[BatchExecutionDescriptor]] = []
        self._capture_descs: dict[CUDAGraphMode, list[BatchExecutionDescriptor]] = {}
        # adjust the cudagraph sizes to be a multiple of the uniform decode query length
        self.compilation_config.adjust_cudagraph_sizes_for_spec_decode(
            self.decode_query_len, self.tp_size
        )
        self._init_candidates()
```
**EN:** This method initializes the object state within `CudaGraphManager`. Key calls include `adjust_cudagraph_sizes_for_spec_decode`, `_init_candidates`, `get_pp_group`, `get_global_graph_pool`. It touches state such as `vllm_config`, `device`, `max_num_reqs`, `compilation_config`, `cudagraph_mode`, `decode_query_len`, `dp_size`, `tp_size`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`CudaGraphManager`。 关键调用包括 `adjust_cudagraph_sizes_for_spec_decode`, `_init_candidates`, `get_pp_group`, `get_global_graph_pool`。 它会读写 `vllm_config`, `device`, `max_num_reqs`, `compilation_config`, `cudagraph_mode`, `decode_query_len`, `dp_size`, `tp_size` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CudaGraphManager.dispatch` method / `CudaGraphManager.dispatch` 方法
```python
    def dispatch(
        self,
        num_reqs: int,
        num_tokens: int,
        uniform_token_count: int | None,
    ) -> BatchExecutionDescriptor:
        """Find matching cudagraph descriptor from priority-ordered candidates."""
        if self._graphs_captured and 0 < num_tokens < len(self._candidates):
            for desc in self._candidates[num_tokens]:
                if _is_compatible(desc, num_reqs, num_tokens, uniform_token_count):
                    return desc
        return BatchExecutionDescriptor(
            cg_mode=CUDAGraphMode.NONE, num_tokens=num_tokens, num_reqs=num_reqs
        )
```
**EN:** This method dispatches work to specialized components within `CudaGraphManager`. The docstring frames it as: Find matching cudagraph descriptor from priority-ordered candidates. Key calls include `BatchExecutionDescriptor`, `len`, `_is_compatible`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会将工作分发到专门组件，其作用域位于`CudaGraphManager`。 关键调用包括 `BatchExecutionDescriptor`, `len`, `_is_compatible`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `CudaGraphManager.run_fullgraph` method / `CudaGraphManager.run_fullgraph` 方法
```python
    def run_fullgraph(self, desc: BatchExecutionDescriptor):
        """Replay a captured FULL cudagraph."""
        assert desc.cg_mode == CUDAGraphMode.FULL, (
            f"Expected FULL mode, got {desc.cg_mode}"
        )
        assert desc in self.graphs, f"No cudagraph for {desc}"
        # Sync offloader before replay - needed when transitioning from
        # eager/piecewise to full cudagraph (e.g., prefill → decode).
        # The previous eager iteration's start_prefetch may have queued
        # H2D copies on copy_stream that the graph's captured events
        # cannot see. Without this, replay could overwrite static buffers
        # while those copies are still in flight.
        get_offloader().sync_prev_onload()
        self.graphs[desc].replay()
```
**EN:** This method runs the main execution path within `CudaGraphManager`. The docstring frames it as: Replay a captured FULL cudagraph. Key calls include `sync_prev_onload`, `replay`, `get_offloader`.
**CN:** 该方法会执行主要运行路径，其作用域位于`CudaGraphManager`。 关键调用包括 `sync_prev_onload`, `replay`, `get_offloader`。

### `ModelCudaGraphManager` class / `ModelCudaGraphManager` 类
```python
class ModelCudaGraphManager(CudaGraphManager):
    """CudaGraphManager with model-specific capture and hidden state management."""
```
**EN:** Introduces the `ModelCudaGraphManager` class on top of `CudaGraphManager`. Core methods include `__init__`, `capture`, `run_fullgraph`. Docstring signal: CudaGraphManager with model-specific capture and hidden state management.
**CN:** 这里定义 `ModelCudaGraphManager` 类，其基类包括 `CudaGraphManager`。核心方法包括 `__init__`, `capture`, `run_fullgraph`。

### `ModelCudaGraphManager.__init__` method / `ModelCudaGraphManager.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        device: torch.device,
        cudagraph_mode: CUDAGraphMode,
        decode_query_len: int,
    ):
        super().__init__(vllm_config, device, cudagraph_mode, decode_query_len)
        # Used for FULL CUDA graphs. PW CUDA graphs do not use these.
        self.hidden_states: torch.Tensor | None = None
        self.aux_hidden_states: list[torch.Tensor] = []
        self.use_aux_hidden_state_outputs = False
        self.intermediate_tensors: IntermediateTensors | None = None
```
**EN:** This method initializes the object state within `ModelCudaGraphManager`. Key calls include `__init__`, `super`. It touches state such as `hidden_states`, `aux_hidden_states`, `use_aux_hidden_state_outputs`, `intermediate_tensors`.
**CN:** 该方法会初始化对象状态，其作用域位于`ModelCudaGraphManager`。 关键调用包括 `__init__`, `super`。 它会读写 `hidden_states`, `aux_hidden_states`, `use_aux_hidden_state_outputs`, `intermediate_tensors` 等状态。

### `ModelCudaGraphManager.capture` method / `ModelCudaGraphManager.capture` 方法
```python
    def capture(
        self,
        model: nn.Module,
        model_state: ModelState,
        input_buffers: InputBuffers,
        intermediate_tensors: IntermediateTensors | None,
        block_tables: BlockTables,
        attn_groups: list[list[AttentionGroup]],
        kv_cache_config: KVCacheConfig,
        has_lora: bool = False,
        use_aux_hidden_state_outputs: bool = False,
        progress_bar_desc: str = "Capturing CUDA graphs",
    ) -> dict[BatchExecutionDescriptor, CapturedAttentionState]:
        """Capture CUDA graphs for model forward pass."""
        self.use_aux_hidden_state_outputs = use_aux_hidden_state_outputs

        def create_forward_fn(
            desc: BatchExecutionDescriptor,
        ) -> tuple[
            Callable[[CUDAGraphMode], None],
            CapturedAttentionState,
        ]:
            num_tokens = desc.num_tokens
            num_reqs = desc.num_reqs or min(num_tokens, self.max_num_reqs)
            num_tokens_across_dp = (
                torch.full((self.dp_size,), num_tokens, dtype=torch.int32, device="cpu")
                if self.dp_size > 1
                else None
            )

            model_inputs = {
                "input_ids": input_buffers.input_ids[:num_tokens],
                "positions": input_buffers.positions[:num_tokens],
                **model_state.prepare_dummy_inputs(num_reqs, num_tokens),
            }
            if not self.is_first_pp_rank:
                # Update for non-first PP ranks.
                model_inputs["input_ids"] = None
                model_inputs["inputs_embeds"] = None
                assert intermediate_tensors is not None
                model_inputs["intermediate_tensors"] = intermediate_tensors[:num_tokens]

            attn_metadata, slot_mappings = prepare_inputs_to_capture(
                num_reqs,
                num_tokens,
                model_state,
                input_buffers,
                block_tables,
                attn_groups,
                kv_cache_config,
    # ... omitted for brevity ...
                        hidden_states = model_output
                        aux_hidden_states = []
                    if self.hidden_states is None:
                        self.hidden_states = torch.empty_like(hidden_states)
                    self.hidden_states[:num_tokens] = hidden_states
                    if self.use_aux_hidden_state_outputs and not self.aux_hidden_states:
                        self.aux_hidden_states = [
                            torch.empty_like(x) for x in aux_hidden_states
                        ]
                    for i, aux in enumerate(aux_hidden_states):
                        self.aux_hidden_states[i][:num_tokens] = aux
                else:
                    # Non-last PP rank.
                    assert isinstance(model_output, IntermediateTensors)
                    intermediate_tensors = model_output
                    if self.intermediate_tensors is None:
                        self.intermediate_tensors = IntermediateTensors.empty_like(
                            intermediate_tensors
                        )
                    for k, v in intermediate_tensors.tensors.items():
                        self.intermediate_tensors[k][:num_tokens] = v

            return forward_fn, CapturedAttentionState(attn_metadata, slot_mappings)

        return super().capture(create_forward_fn, progress_bar_desc)
```
**EN:** This method implements `capture` within `ModelCudaGraphManager`. The docstring frames it as: Capture CUDA graphs for model forward pass. Key calls include `capture`, `prepare_inputs_to_capture`, `min`, `full`, `prepare_dummy_inputs`, `CapturedAttentionState`. It touches state such as `use_aux_hidden_state_outputs`, `hidden_states`, `aux_hidden_states`, `intermediate_tensors`. The control flow contains 9 branch(es) and 2 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `capture`，其作用域位于`ModelCudaGraphManager`。 关键调用包括 `capture`, `prepare_inputs_to_capture`, `min`, `full`, `prepare_dummy_inputs`, `CapturedAttentionState`。 它会读写 `use_aux_hidden_state_outputs`, `hidden_states`, `aux_hidden_states`, `intermediate_tensors` 等状态。 控制流包含 9 个分支和 2 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `ModelCudaGraphManager.run_fullgraph` method / `ModelCudaGraphManager.run_fullgraph` 方法
```python
    def run_fullgraph(
        self, desc: BatchExecutionDescriptor
    ) -> torch.Tensor | tuple[torch.Tensor, list[torch.Tensor]] | IntermediateTensors:
        """Replay a captured FULL cudagraph and return hidden states."""
        super().run_fullgraph(desc)
        if not self.is_last_pp_rank:
            assert self.intermediate_tensors is not None
            return self.intermediate_tensors[: desc.num_tokens]

        assert self.hidden_states is not None
        hidden_states = self.hidden_states[: desc.num_tokens]
        if not self.use_aux_hidden_state_outputs:
            return hidden_states
        return hidden_states, [x[: desc.num_tokens] for x in self.aux_hidden_states]
```
**EN:** This method runs the main execution path within `ModelCudaGraphManager`. The docstring frames it as: Replay a captured FULL cudagraph and return hidden states. Key calls include `run_fullgraph`, `super`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会执行主要运行路径，其作用域位于`ModelCudaGraphManager`。 关键调用包括 `run_fullgraph`, `super`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `prepare_inputs_to_capture` function / `prepare_inputs_to_capture` 函数
```python
def prepare_inputs_to_capture(
    num_reqs: int,
    num_tokens: int,
    model_state: ModelState,
    input_buffers: InputBuffers,
    block_tables: BlockTables,
    attn_groups: list[list[AttentionGroup]],
    kv_cache_config: KVCacheConfig,
    skip_attn: bool = False,
) -> CapturedAttentionState:
    input_batch = InputBatch.make_dummy(num_reqs, num_tokens, input_buffers)
    input_block_tables = block_tables.get_dummy_block_tables(num_reqs)
    slot_mappings = block_tables.get_dummy_slot_mappings(num_tokens)
    slot_mappings_by_layer = build_slot_mappings_by_layer(
        slot_mappings, kv_cache_config
    )

    # HACK(woosuk): Special handling for DCP.
    if block_tables.cp_size > 1:
        prepare_dcp_local_seq_lens(
            input_buffers.dcp_local_seq_lens,
            input_batch.seq_lens,
            num_reqs,
            block_tables.cp_size,
            block_tables.cp_rank,
            block_tables.cp_interleave,
        )
        input_batch.dcp_local_seq_lens = input_buffers.dcp_local_seq_lens[:num_reqs]

    attn_metadata = None
    if not skip_attn:
        attn_metadata = model_state.prepare_attn(
            input_batch,
            CUDAGraphMode.NONE,
            input_block_tables,
            slot_mappings,
            attn_groups,
            kv_cache_config,
            for_capture=True,
        )
    return CapturedAttentionState(attn_metadata, slot_mappings_by_layer)
```
**EN:** This function prepares inputs and state within the module. Key calls include `make_dummy`, `get_dummy_block_tables`, `get_dummy_slot_mappings`, `build_slot_mappings_by_layer`, `CapturedAttentionState`, `prepare_dcp_local_seq_lens`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会准备输入与状态，其作用域位于the module。 关键调用包括 `make_dummy`, `get_dummy_block_tables`, `get_dummy_slot_mappings`, `build_slot_mappings_by_layer`, `CapturedAttentionState`, `prepare_dcp_local_seq_lens`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `CapturedAttentionState`: central class or interface in this module. / `CapturedAttentionState`：本模块中的核心类或接口。
- `BatchExecutionDescriptor`: central class or interface in this module. / `BatchExecutionDescriptor`：本模块中的核心类或接口。
- `_is_compatible`: top-level helper or orchestration entry point. / `_is_compatible`：顶层辅助函数或编排入口。
- `get_uniform_token_count`: top-level helper or orchestration entry point. / `get_uniform_token_count`：顶层辅助函数或编排入口。
- `CudaGraphManager`: central class or interface in this module. / `CudaGraphManager`：本模块中的核心类或接口。
- `ModelCudaGraphManager`: central class or interface in this module. / `ModelCudaGraphManager`：本模块中的核心类或接口。
- `prepare_inputs_to_capture`: top-level helper or orchestration entry point. / `prepare_inputs_to_capture`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `collections`, `dataclasses`, `typing`
- External / 外部依赖: `torch`, `tqdm`
- Internal vLLM / 内部依赖: `vllm.compilation.counter`, `vllm.config`, `vllm.config.compilation`, `vllm.distributed.parallel_state`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.offloader.base`, `vllm.platforms`, `vllm.sequence`, `vllm.v1.kv_cache_interface`, `vllm.v1.worker.gpu.attn_utils`, `vllm.v1.worker.gpu.block_table`
