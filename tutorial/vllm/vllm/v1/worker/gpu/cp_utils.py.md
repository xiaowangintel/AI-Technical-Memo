# cp_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/cp_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `prepare_dcp_local_seq_lens`, `_dcp_local_seq_lens_kernel` for the V1 `worker/gpu` subsystem. / 为 V1 的 `worker/gpu` 子系统实现 `prepare_dcp_local_seq_lens`, `_dcp_local_seq_lens_kernel`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import torch

from vllm.triton_utils import tl, triton
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.triton_utils`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.triton_utils` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `prepare_dcp_local_seq_lens` function / `prepare_dcp_local_seq_lens` 函数
```python
def prepare_dcp_local_seq_lens(
    dcp_local_seq_lens: torch.Tensor,
    seq_lens: torch.Tensor,
    num_reqs: int,
    dcp_size: int,
    dcp_rank: int,
    cp_interleave: int,
) -> None:
    """Populate the persistent DCP local seq_lens buffer (CUDA graph safe)."""
    if dcp_size == 1:
        return

    max_num_reqs = dcp_local_seq_lens.shape[0]
    BLOCK_SIZE = 128
    num_blocks = triton.cdiv(max_num_reqs, BLOCK_SIZE)
    _dcp_local_seq_lens_kernel[(num_blocks,)](
        dcp_local_seq_lens,
        seq_lens,
        dcp_size,
        dcp_rank,
        cp_interleave,
        num_reqs,
        max_num_reqs,
        BLOCK_SIZE,
    )
```
**EN:** This function prepares inputs and state within the module. The docstring frames it as: Populate the persistent DCP local seq_lens buffer (CUDA graph safe). Key calls include `cdiv`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会准备输入与状态，其作用域位于the module。 关键调用包括 `cdiv`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_dcp_local_seq_lens_kernel` function / `_dcp_local_seq_lens_kernel` 函数
```python
@triton.jit
def _dcp_local_seq_lens_kernel(
    out_ptr,
    seq_lens_ptr,
    dcp_size,
    dcp_rank,
    cp_interleave,
    num_reqs,
    max_num_reqs,
    BLOCK_SIZE: tl.constexpr,
):
    pid = tl.program_id(0)
    block = pid * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)

    seq_lens = tl.load(seq_lens_ptr + block, mask=block < num_reqs)

    # Distribute KV cache among different ranks, in a round-robin manner.
    rounds = seq_lens // (dcp_size * cp_interleave)
    remainder = seq_lens % (dcp_size * cp_interleave)

    remainder = tl.maximum(remainder - dcp_rank * cp_interleave, 0)
    remainder = tl.minimum(remainder, cp_interleave)
    local_seq_lens = rounds * cp_interleave + remainder

    # For [num_reqs, max_num_reqs), pad with 0
    local_seq_lens = tl.where(block < num_reqs, local_seq_lens, 0)
    tl.store(out_ptr + block, local_seq_lens, mask=block < max_num_reqs)
```
**EN:** This function implements `_dcp_local_seq_lens_kernel` within the module. Key calls include `program_id`, `load`, `maximum`, `minimum`, `where`, `store`.
**CN:** 该函数会实现 `_dcp_local_seq_lens_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `maximum`, `minimum`, `where`, `store`。

## Key Concepts / 关键概念
- `prepare_dcp_local_seq_lens`: top-level helper or orchestration entry point. / `prepare_dcp_local_seq_lens`：顶层辅助函数或编排入口。
- `_dcp_local_seq_lens_kernel`: top-level helper or orchestration entry point. / `_dcp_local_seq_lens_kernel`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.triton_utils`
