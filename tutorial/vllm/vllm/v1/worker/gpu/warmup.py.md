# warmup.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/warmup.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `warmup_kernels` for the V1 `worker/gpu` subsystem. / 为 V1 的 `worker/gpu` 子系统实现 `warmup_kernels`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from collections.abc import Callable
from typing import Any

import numpy as np
import torch

from vllm import PoolingParams, SamplingParams
from vllm.utils.math_utils import cdiv
from vllm.v1.core.sched.output import (
    CachedRequestData,
    GrammarOutput,
    NewRequestData,
    SchedulerOutput,
)
from vllm.v1.request import Request
from vllm.v1.worker.gpu.model_runner import GPUModelRunner
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, and internal vLLM modules such as `vllm`, `vllm.utils.math_utils`, `vllm.v1.core.sched.output`, `vllm.v1.request`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖，以及 `vllm`, `vllm.utils.math_utils`, `vllm.v1.core.sched.output`, `vllm.v1.request` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `warmup_kernels` function / `warmup_kernels` 函数
```python
@torch.inference_mode()
def warmup_kernels(
    model_runner: GPUModelRunner,
    worker_execute_model: Callable[[SchedulerOutput], Any],
    worker_sample_tokens: Callable[[GrammarOutput | None], Any],
) -> None:
    """Run two execute_model + sample_tokens iterations to JIT compile
    triton kernels. We must call the provided worker's execute_model for
    pipeline parallel coordination.

    The first iteration simulates a prefill with requests of
    2 + num_spec_steps prompt tokens each. The second iteration simulates
    a decode step with all requests generating 1 + num_spec_steps tokens.
    """
    num_spec_steps = model_runner.num_speculative_steps
    # Use 1 + num_spec_steps + 1 tokens so the prefill batch's per-request
    # query length exceeds decode_query_len (= 1 + num_spec_steps), preventing
    # it from being misclassified as a uniform decode batch.
    prompt_len = 2 + num_spec_steps
    prompt_token_ids = list(range(prompt_len))
    # After prefill, decode generates 1 verified + num_spec_steps draft tokens.
    decode_len = prompt_len + 1 + num_spec_steps

    kv_cache_groups = model_runner.kv_cache_config.kv_cache_groups
    num_kv_cache_groups = len(kv_cache_groups)

    # Compute per-request block counts for each KV cache group.
    group_block_sizes = [g.kv_cache_spec.block_size for g in kv_cache_groups]
    prefill_block_counts = [cdiv(prompt_len, bs) for bs in group_block_sizes]
    decode_block_counts = [cdiv(decode_len, bs) for bs in group_block_sizes]
    decode_block_deltas = [
        d - p for d, p in zip(decode_block_counts, prefill_block_counts)
    ]
    max_blocks_per_req = sum(decode_block_counts)

    num_reqs = min(
        model_runner.scheduler_config.max_num_seqs,
        model_runner.scheduler_config.max_num_batched_tokens
        // max(prompt_len, 1 + num_spec_steps),
        # Reserve block 0 (null block) and ensure we have enough blocks.
        max(1, (model_runner.kv_cache_config.num_blocks - 1) // max_blocks_per_req),
    )

    req_ids = [f"_warmup_{i}_" for i in range(num_reqs)]

    # SamplingParams exercising all sampling features.
    if model_runner.is_pooling_model:
        sampling_params = None
        pooling_params = PoolingParams()
    else:
# ... omitted for brevity ...
        ]

        decode_output = SchedulerOutput.make_empty()
        decode_output.scheduled_cached_reqs = cached_req_data
        decode_output.num_scheduled_tokens = {
            req_id: 1 + num_spec_steps for req_id in req_ids
        }
        if num_spec_steps > 0:
            decode_output.scheduled_spec_decode_tokens = {
                req_id: [0] * num_spec_steps for req_id in req_ids
            }
        decode_output.total_num_scheduled_tokens = sum(
            decode_output.num_scheduled_tokens.values()
        )
        decode_output.num_common_prefix_blocks = [0] * num_kv_cache_groups

        worker_execute_model(decode_output)
        worker_sample_tokens(None)

    # Clean up - process finish_req_ids.
    cleanup_output = SchedulerOutput.make_empty()
    cleanup_output.finished_req_ids = set(req_ids)
    worker_execute_model(cleanup_output)
    model_runner.kv_connector.set_disabled(False)
    torch.accelerator.synchronize()
```
**EN:** This function implements `warmup_kernels` within the module. The docstring frames it as: Run two execute_model + sample_tokens iterations to JIT compile triton kernels. Key calls include `inference_mode`, `list`, `len`, `sum`, `min`, `make_empty`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `warmup_kernels`，其作用域位于the module。 关键调用包括 `inference_mode`, `list`, `len`, `sum`, `min`, `make_empty`。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

## Key Concepts / 关键概念
- `warmup_kernels`: top-level helper or orchestration entry point. / `warmup_kernels`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `collections`, `typing`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm`, `vllm.utils.math_utils`, `vllm.v1.core.sched.output`, `vllm.v1.request`, `vllm.v1.worker.gpu.model_runner`
