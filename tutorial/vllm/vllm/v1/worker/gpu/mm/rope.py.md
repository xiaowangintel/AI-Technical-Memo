# rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/mm/rope.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `RopeState`, `get_rope_state`, `_prepare_rope_positions_kernel` for the V1 `worker/gpu/mm` subsystem. / 为 V1 的 `worker/gpu/mm` 子系统实现 `RopeState`, `get_rope_state`, `_prepare_rope_positions_kernel`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from typing import cast

import torch
import torch.nn as nn

from vllm.config import ModelConfig
from vllm.model_executor.models.interfaces import SupportsMRoPE, SupportsXDRoPE
from vllm.triton_utils import tl, triton
from vllm.v1.worker.gpu.buffer_utils import StagedWriteTensor, UvaBackedTensor
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.config`, `vllm.model_executor.models.interfaces`, `vllm.triton_utils`, `vllm.v1.worker.gpu.buffer_utils`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.config`, `vllm.model_executor.models.interfaces`, `vllm.triton_utils`, `vllm.v1.worker.gpu.buffer_utils` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `RopeState` class / `RopeState` 类
```python
class RopeState:
    """Unified state for multi-dimensional RoPE variants (M-RoPE, XD-RoPE).

    M-RoPE: 3 dims, uses position delta for decode.
    XD-RoPE: 3 or 4 dims, delta is 0 (decode uses orig_pos for all dims).

    NOTE: `positions` is implemented with one additional dummy position on
    purpose to make it non-contiguous so that it can work with torch compile.
    See detailed explanation in
    https://github.com/vllm-project/vllm/pull/12128#discussion_r1926431923

    NOTE: When M-RoPE is enabled, position ids are 3D regardless of the
    modality of inputs. For text-only inputs, each dimension has identical
    position IDs, making M-RoPE functionally equivalent to 1D-RoPE.
    See page 5 of https://arxiv.org/abs/2409.12191
    """
```
**EN:** Introduces the `RopeState` class. Core methods include `__init__`, `init_prefill_positions`, `apply_staged_writes`, `get_positions`, `prepare_positions`. Docstring signal: Unified state for multi-dimensional RoPE variants (M-RoPE, XD-RoPE).
**CN:** 这里定义 `RopeState` 类。核心方法包括 `__init__`, `init_prefill_positions`, `apply_staged_writes`, `get_positions`, `prepare_positions`。

### `RopeState.__init__` method / `RopeState.__init__` 方法
```python
    def __init__(
        self,
        num_dims: int,
        has_delta: bool,
        max_num_reqs: int,
        max_num_tokens: int,
        max_model_len: int,
        device: torch.device,
    ):
        self.num_dims = num_dims
        self.has_delta = has_delta
        self.max_num_reqs = max_num_reqs
        self.max_num_tokens = max_num_tokens
        self.max_model_len = max_model_len
        self.device = device

        # NOTE(woosuk): This tensor can be extremely large (e.g., several GBs)
        # wasting a lot of CPU memory.
        self.prefill_positions = StagedWriteTensor(
            (max_num_reqs * num_dims, max_model_len),
            dtype=torch.int32,
            device=device,
            uva_instead_of_gpu=True,
        )
        self.positions = torch.zeros(
            (num_dims, max_num_tokens + 1), dtype=torch.int64, device=device
        )

        # Delta is non-zero for M-RoPE, always 0 for XD-RoPE.
        self.prefill_delta = UvaBackedTensor(max_num_reqs, dtype=torch.int32)
```
**EN:** This method initializes the object state within `RopeState`. Key calls include `StagedWriteTensor`, `zeros`, `UvaBackedTensor`. It touches state such as `num_dims`, `has_delta`, `max_num_reqs`, `max_num_tokens`, `max_model_len`, `device`, `prefill_positions`, `positions`.
**CN:** 该方法会初始化对象状态，其作用域位于`RopeState`。 关键调用包括 `StagedWriteTensor`, `zeros`, `UvaBackedTensor`。 它会读写 `num_dims`, `has_delta`, `max_num_reqs`, `max_num_tokens`, `max_model_len`, `device`, `prefill_positions`, `positions` 等状态。

### `RopeState.init_prefill_positions` method / `RopeState.init_prefill_positions` 方法
```python
    def init_prefill_positions(
        self,
        req_idx: int,
        model: nn.Module,
        prefill_token_ids: list[int],
        mm_features: list,
    ) -> None:
        if self.has_delta:
            mrope_model = cast(SupportsMRoPE, model)
            prefill_positions, delta = mrope_model.get_mrope_input_positions(
                prefill_token_ids, mm_features
            )
            self.prefill_delta.np[req_idx] = delta
        else:
            xdrope_model = cast(SupportsXDRoPE, model)
            prefill_positions = xdrope_model.get_xdrope_input_positions(
                prefill_token_ids, mm_features
            )

        for i in range(self.num_dims):
            pos = prefill_positions[i].tolist()
            self.prefill_positions.stage_write(self.num_dims * req_idx + i, 0, pos)
```
**EN:** This method implements `init_prefill_positions` within `RopeState`. Key calls include `range`, `cast`, `get_mrope_input_positions`, `get_xdrope_input_positions`, `tolist`, `stage_write`. It touches state such as `prefill_delta`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `init_prefill_positions`，其作用域位于`RopeState`。 关键调用包括 `range`, `cast`, `get_mrope_input_positions`, `get_xdrope_input_positions`, `tolist`, `stage_write`。 它会读写 `prefill_delta` 等状态。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `RopeState.apply_staged_writes` method / `RopeState.apply_staged_writes` 方法
```python
    def apply_staged_writes(self) -> None:
        self.prefill_positions.apply_write()
        if self.has_delta:
            self.prefill_delta.copy_to_uva()
```
**EN:** This method implements `apply_staged_writes` within `RopeState`. Key calls include `apply_write`, `copy_to_uva`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `apply_staged_writes`，其作用域位于`RopeState`。 关键调用包括 `apply_write`, `copy_to_uva`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RopeState.get_positions` method / `RopeState.get_positions` 方法
```python
    def get_positions(self, num_tokens: int) -> torch.Tensor:
        return self.positions[:, :num_tokens]
```
**EN:** This method returns or derives a value within `RopeState`.
**CN:** 该方法会返回或推导一个值，其作用域位于`RopeState`。

### `RopeState.prepare_positions` method / `RopeState.prepare_positions` 方法
```python
    def prepare_positions(
        self,
        idx_mapping: torch.Tensor,
        query_start_loc: torch.Tensor,
        prefill_lens: torch.Tensor,
        num_computed_tokens: torch.Tensor,
    ) -> None:
        num_reqs = idx_mapping.shape[0]
        _prepare_rope_positions_kernel[(num_reqs,)](
            self.positions,
            self.positions.stride(0),
            self.prefill_positions.gpu,
            self.num_dims * self.max_model_len,
            self.max_model_len,
            self.prefill_delta.gpu,
            idx_mapping,
            query_start_loc,
            prefill_lens,
            num_computed_tokens,
            BLOCK_SIZE=1024,
            NUM_DIMS=self.num_dims,
        )
```
**EN:** This method prepares inputs and state within `RopeState`. Key calls include `stride`.
**CN:** 该方法会准备输入与状态，其作用域位于`RopeState`。 关键调用包括 `stride`。

### `get_rope_state` function / `get_rope_state` 函数
```python
def get_rope_state(
    model_config: ModelConfig,
    model: nn.Module,
    max_num_reqs: int,
    max_num_tokens: int,
    max_model_len: int,
    device: torch.device,
) -> RopeState | None:
    """Create a RopeState if the model uses multi-dimensional RoPE."""
    if model_config.uses_mrope:
        assert isinstance(model, SupportsMRoPE)
        return RopeState(
            num_dims=3,
            has_delta=True,
            max_num_reqs=max_num_reqs,
            max_num_tokens=max_num_tokens,
            max_model_len=max_model_len,
            device=device,
        )
    if model_config.uses_xdrope_dim > 0:
        assert isinstance(model, SupportsXDRoPE)
        return RopeState(
            num_dims=model_config.uses_xdrope_dim,
            has_delta=False,
            max_num_reqs=max_num_reqs,
            max_num_tokens=max_num_tokens,
            max_model_len=max_model_len,
            device=device,
        )
    return None
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Create a RopeState if the model uses multi-dimensional RoPE. Key calls include `isinstance`, `RopeState`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `isinstance`, `RopeState`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_prepare_rope_positions_kernel` function / `_prepare_rope_positions_kernel` 函数
```python
@triton.jit
def _prepare_rope_positions_kernel(
    positions_ptr,
    positions_stride,
    prefill_positions_ptr,
    prefill_positions_stride0,
    prefill_positions_stride1,
    prefill_delta_ptr,
    idx_mapping_ptr,
    query_start_loc_ptr,
    prefill_lens_ptr,
    num_computed_tokens_ptr,
    BLOCK_SIZE: tl.constexpr,
    NUM_DIMS: tl.constexpr,
):
    batch_idx = tl.program_id(0)
    req_state_idx = tl.load(idx_mapping_ptr + batch_idx)

    prefill_len = tl.load(prefill_lens_ptr + req_state_idx)
    num_computed = tl.load(num_computed_tokens_ptr + req_state_idx)
    is_prefill = num_computed < prefill_len

    query_start = tl.load(query_start_loc_ptr + batch_idx)
    query_end = tl.load(query_start_loc_ptr + batch_idx + 1)
    query_len = query_end - query_start

    delta = tl.load(prefill_delta_ptr + req_state_idx)

    for i in range(0, query_len, BLOCK_SIZE):
        block = i + tl.arange(0, BLOCK_SIZE)
        mask = block < query_len
        orig_pos = num_computed + block

        for j in tl.static_range(NUM_DIMS):
            if is_prefill:
                pos = tl.load(
                    prefill_positions_ptr
                    + req_state_idx * prefill_positions_stride0
                    + j * prefill_positions_stride1
                    + orig_pos,
                    mask=mask,
                )
            else:
                pos = orig_pos + delta
            tl.store(
                positions_ptr + j * positions_stride + query_start + block,
                pos,
                mask=mask,
            )
```
**EN:** This function implements `_prepare_rope_positions_kernel` within the module. Key calls include `program_id`, `load`, `range`, `static_range`, `arange`, `store`. The control flow contains 1 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_prepare_rope_positions_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `range`, `static_range`, `arange`, `store`。 控制流包含 1 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `RopeState`: central class or interface in this module. / `RopeState`：本模块中的核心类或接口。
- `get_rope_state`: top-level helper or orchestration entry point. / `get_rope_state`：顶层辅助函数或编排入口。
- `_prepare_rope_positions_kernel`: top-level helper or orchestration entry point. / `_prepare_rope_positions_kernel`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.model_executor.models.interfaces`, `vllm.triton_utils`, `vllm.v1.worker.gpu.buffer_utils`
