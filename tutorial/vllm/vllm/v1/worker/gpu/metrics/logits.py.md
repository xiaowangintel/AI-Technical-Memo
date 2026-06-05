# logits.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/metrics/logits.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_num_nans_kernel`, `get_num_nans` for the V1 `worker/gpu/metrics` subsystem. / 为 V1 的 `worker/gpu/metrics` 子系统实现 `_num_nans_kernel`, `get_num_nans`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import torch
from torch._inductor.runtime.triton_helpers import libdevice

from vllm.triton_utils import tl, triton
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.triton_utils`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.triton_utils` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `_num_nans_kernel` function / `_num_nans_kernel` 函数
```python
@triton.jit
def _num_nans_kernel(
    logits_ptr,
    logits_stride,
    num_nans_ptr,
    vocab_size,
    BLOCK_SIZE: tl.constexpr,
):
    req_idx = tl.program_id(0)
    num_nans = 0
    for i in range(0, vocab_size, BLOCK_SIZE):
        block = i + tl.arange(0, BLOCK_SIZE)
        mask = block < vocab_size
        logits = tl.load(
            logits_ptr + req_idx * logits_stride + block, mask=mask, other=0
        )
        logits = logits.to(tl.float32)
        is_nan = libdevice.isnan(logits).to(tl.int1)
        num_nans += tl.sum(is_nan).to(tl.int32)
    tl.store(num_nans_ptr + req_idx, num_nans)
```
**EN:** This function implements `_num_nans_kernel` within the module. Key calls include `program_id`, `range`, `store`, `load`, `to`, `arange`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_num_nans_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `range`, `store`, `load`, `to`, `arange`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `get_num_nans` function / `get_num_nans` 函数
```python
def get_num_nans(logits: torch.Tensor) -> torch.Tensor:
    num_reqs, vocab_size = logits.shape
    BLOCK_SIZE = 8192
    num_nans = torch.empty(num_reqs, dtype=torch.int32, device=logits.device)
    _num_nans_kernel[(num_reqs,)](
        logits,
        logits.stride(0),
        num_nans,
        vocab_size,
        BLOCK_SIZE=BLOCK_SIZE,
    )
    return num_nans
```
**EN:** This function returns or derives a value within the module. Key calls include `empty`, `stride`.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `empty`, `stride`。

## Key Concepts / 关键概念
- `_num_nans_kernel`: top-level helper or orchestration entry point. / `_num_nans_kernel`：顶层辅助函数或编排入口。
- `get_num_nans`: top-level helper or orchestration entry point. / `get_num_nans`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.triton_utils`
