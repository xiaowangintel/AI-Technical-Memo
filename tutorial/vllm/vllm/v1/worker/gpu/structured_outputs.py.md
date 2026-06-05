# structured_outputs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/structured_outputs.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `StructuredOutputsWorker`, `_apply_grammar_bitmask_kernel` for the V1 `worker/gpu` subsystem. / 为 V1 的 `worker/gpu` 子系统实现 `StructuredOutputsWorker`, `_apply_grammar_bitmask_kernel`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import numpy as np
import torch

from vllm.triton_utils import tl, triton
from vllm.utils.math_utils import cdiv
from vllm.v1.worker.gpu.buffer_utils import async_copy_to_gpu
from vllm.v1.worker.gpu.input_batch import InputBatch
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, and internal vLLM modules such as `vllm.triton_utils`, `vllm.utils.math_utils`, `vllm.v1.worker.gpu.buffer_utils`, `vllm.v1.worker.gpu.input_batch`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖，以及 `vllm.triton_utils`, `vllm.utils.math_utils`, `vllm.v1.worker.gpu.buffer_utils`, `vllm.v1.worker.gpu.input_batch` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `StructuredOutputsWorker` class / `StructuredOutputsWorker` 类
```python
class StructuredOutputsWorker:
```
**EN:** Introduces the `StructuredOutputsWorker` class. Core methods include `__init__`, `apply_grammar_bitmask`.
**CN:** 这里定义 `StructuredOutputsWorker` 类。核心方法包括 `__init__`, `apply_grammar_bitmask`。

### `StructuredOutputsWorker.__init__` method / `StructuredOutputsWorker.__init__` 方法
```python
    def __init__(self, max_num_logits: int, vocab_size: int, device: torch.device):
        self.logits_indices = torch.zeros(
            max_num_logits, dtype=torch.int32, device=device
        )
        self.grammar_bitmask = torch.zeros(
            (max_num_logits, cdiv(vocab_size, 32)), dtype=torch.int32, device=device
        )
        self.device = device
        self.copy_stream = torch.cuda.Stream()
```
**EN:** This method initializes the object state within `StructuredOutputsWorker`. Key calls include `zeros`, `Stream`, `cdiv`. It touches state such as `logits_indices`, `grammar_bitmask`, `device`, `copy_stream`.
**CN:** 该方法会初始化对象状态，其作用域位于`StructuredOutputsWorker`。 关键调用包括 `zeros`, `Stream`, `cdiv`。 它会读写 `logits_indices`, `grammar_bitmask`, `device`, `copy_stream` 等状态。

### `StructuredOutputsWorker.apply_grammar_bitmask` method / `StructuredOutputsWorker.apply_grammar_bitmask` 方法
```python
    def apply_grammar_bitmask(
        self,
        logits: torch.Tensor,
        input_batch: InputBatch,
        grammar_req_ids: list[str],
        grammar_bitmask: np.ndarray,
    ) -> None:
        if not grammar_req_ids:
            return

        # Asynchronously copy the bitmask to GPU.
        with torch.cuda.stream(self.copy_stream):
            bitmask = async_copy_to_gpu(
                grammar_bitmask, out=self.grammar_bitmask[: grammar_bitmask.shape[0]]
            )

        # Construct bitmask -> logits mapping
        mapping: list[int] = []
        req_ids = input_batch.req_ids
        cu_num_logits = input_batch.cu_num_logits_np.tolist()
        req_id_to_idx = {req_id: i for i, req_id in enumerate(req_ids)}
        for grammar_req_id in grammar_req_ids:
            req_idx = req_id_to_idx[grammar_req_id]
            logits_start_idx = cu_num_logits[req_idx]
            logits_end_idx = cu_num_logits[req_idx + 1]
            mapping.extend(range(logits_start_idx, logits_end_idx))

        # Asynchronously copy the mapping to GPU.
        with torch.cuda.stream(self.copy_stream):
            logits_indices = torch.tensor(
                mapping, dtype=torch.int32, device="cpu", pin_memory=True
            )
            logits_indices = self.logits_indices[: len(mapping)].copy_(
                logits_indices, non_blocking=True
            )

        # Ensure all async copies are complete before launching the kernel.
        current_stream = torch.cuda.current_stream()
        current_stream.wait_stream(self.copy_stream)

        num_masks = bitmask.shape[0]
        assert num_masks == len(mapping)
        vocab_size = logits.shape[-1]
        BLOCK_SIZE = 8192
        grid = (num_masks, triton.cdiv(vocab_size, BLOCK_SIZE))
        _apply_grammar_bitmask_kernel[grid](
            logits,
            logits.stride(0),
            logits_indices,
            bitmask,
            bitmask.stride(0),
            vocab_size,
            BLOCK_SIZE=BLOCK_SIZE,
        )

        # Ensure the copy stream waits for the device tensors to finish being used
        # before it re-uses or deallocates them
        self.copy_stream.wait_stream(current_stream)
```
**EN:** This method implements `apply_grammar_bitmask` within `StructuredOutputsWorker`. Key calls include `tolist`, `current_stream`, `wait_stream`, `stream`, `async_copy_to_gpu`, `extend`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `apply_grammar_bitmask`，其作用域位于`StructuredOutputsWorker`。 关键调用包括 `tolist`, `current_stream`, `wait_stream`, `stream`, `async_copy_to_gpu`, `extend`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `_apply_grammar_bitmask_kernel` function / `_apply_grammar_bitmask_kernel` 函数
```python
@triton.jit
def _apply_grammar_bitmask_kernel(
    logits_ptr,
    logits_stride,
    logits_indices_ptr,
    bitmask_ptr,
    bitmask_stride,
    vocab_size,
    BLOCK_SIZE: tl.constexpr,
):
    bitmask_idx = tl.program_id(0)
    logits_idx = tl.load(logits_indices_ptr + bitmask_idx)

    # Load the bitmask.
    block_id = tl.program_id(1)
    bitmask_offset = (block_id * BLOCK_SIZE) // 32 + tl.arange(0, BLOCK_SIZE // 32)
    packed_bitmask = tl.load(
        bitmask_ptr + bitmask_idx * bitmask_stride + bitmask_offset,
        mask=bitmask_offset < bitmask_stride,
    )
    # Unpack the bitmask.
    bitmask = ((packed_bitmask[:, None] >> (tl.arange(0, 32)[None, :])) & 1) == 0
    bitmask = bitmask.reshape(BLOCK_SIZE)

    # Apply the bitmask to the logits.
    block_offset = block_id * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    tl.store(
        logits_ptr + logits_idx * logits_stride + block_offset,
        -float("inf"),
        mask=bitmask & (block_offset < vocab_size),
    )
```
**EN:** This function implements `_apply_grammar_bitmask_kernel` within the module. Key calls include `program_id`, `load`, `reshape`, `store`, `arange`, `float`.
**CN:** 该函数会实现 `_apply_grammar_bitmask_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `reshape`, `store`, `arange`, `float`。

## Key Concepts / 关键概念
- `StructuredOutputsWorker`: central class or interface in this module. / `StructuredOutputsWorker`：本模块中的核心类或接口。
- `_apply_grammar_bitmask_kernel`: top-level helper or orchestration entry point. / `_apply_grammar_bitmask_kernel`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.triton_utils`, `vllm.utils.math_utils`, `vllm.v1.worker.gpu.buffer_utils`, `vllm.v1.worker.gpu.input_batch`
