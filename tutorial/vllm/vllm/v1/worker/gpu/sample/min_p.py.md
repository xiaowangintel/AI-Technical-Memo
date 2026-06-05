# min_p.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/sample/min_p.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_min_p_kernel`, `apply_min_p` for the V1 `worker/gpu/sample` subsystem. / 为 V1 的 `worker/gpu/sample` 子系统实现 `_min_p_kernel`, `apply_min_p`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import torch

from vllm.triton_utils import tl, triton
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.triton_utils`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.triton_utils` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `_min_p_kernel` function / `_min_p_kernel` 函数
```python
@triton.jit
def _min_p_kernel(
    logits_ptr,
    logits_stride,
    expanded_idx_mapping_ptr,
    min_p_ptr,
    vocab_size,
    BLOCK_SIZE: tl.constexpr,
):
    token_idx = tl.program_id(0)
    req_state_idx = tl.load(expanded_idx_mapping_ptr + token_idx)
    min_p = tl.load(min_p_ptr + req_state_idx).to(tl.float32)
    if min_p == 0.0:
        return

    max_val = float("-inf")
    for i in range(0, vocab_size, BLOCK_SIZE):
        block = i + tl.arange(0, BLOCK_SIZE)
        mask = block < vocab_size
        logits = tl.load(
            logits_ptr + token_idx * logits_stride + block,
            mask=mask,
            other=float("-inf"),
        )
        max_val = tl.max(tl.maximum(logits, max_val))
    max_val = max_val.to(tl.float32)  # type: ignore

    threshold = max_val + tl.log(min_p)
    for i in range(0, vocab_size, BLOCK_SIZE):
        block = i + tl.arange(0, BLOCK_SIZE)
        mask = block < vocab_size
        logits = tl.load(
            logits_ptr + token_idx * logits_stride + block,
            mask=mask,
            other=float("-inf"),
        )
        logits = tl.where(logits < threshold, float("-inf"), logits)
        tl.store(logits_ptr + token_idx * logits_stride + block, logits, mask=mask)
```
**EN:** This function implements `_min_p_kernel` within the module. Key calls include `program_id`, `load`, `to`, `float`, `range`, `max`. The control flow contains 1 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_min_p_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `to`, `float`, `range`, `max`。 控制流包含 1 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `apply_min_p` function / `apply_min_p` 函数
```python
def apply_min_p(
    logits: torch.Tensor, expanded_idx_mapping: torch.Tensor, min_p: torch.Tensor
) -> None:
    num_tokens, vocab_size = logits.shape
    BLOCK_SIZE = 1024
    _min_p_kernel[(num_tokens,)](
        logits,
        logits.stride(0),
        expanded_idx_mapping,
        min_p,
        vocab_size,
        BLOCK_SIZE=BLOCK_SIZE,
    )
```
**EN:** This function implements `apply_min_p` within the module. Key calls include `stride`.
**CN:** 该函数会实现 `apply_min_p`，其作用域位于the module。 关键调用包括 `stride`。

## Key Concepts / 关键概念
- `_min_p_kernel`: top-level helper or orchestration entry point. / `_min_p_kernel`：顶层辅助函数或编排入口。
- `apply_min_p`: top-level helper or orchestration entry point. / `apply_min_p`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.triton_utils`
