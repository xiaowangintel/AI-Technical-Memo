# rejection_sampler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/spec_decode/rejection_sampler.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_flatten_sampled_kernel`, `RejectionSampler` for the V1 `worker/gpu/spec_decode` subsystem. / 为 V1 的 `worker/gpu/spec_decode` 子系统实现 `_flatten_sampled_kernel`, `RejectionSampler`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import torch

from vllm.config import SpeculativeConfig
from vllm.triton_utils import tl, triton
from vllm.v1.outputs import LogprobsTensors
from vllm.v1.spec_decode.utils import unconditional_to_conditional_rates
from vllm.v1.worker.gpu.input_batch import InputBatch
from vllm.v1.worker.gpu.metrics.logits import get_num_nans
from vllm.v1.worker.gpu.sample.logprob import compute_topk_logprobs
from vllm.v1.worker.gpu.sample.output import SamplerOutput
from vllm.v1.worker.gpu.sample.sampler import Sampler
from vllm.v1.worker.gpu.sample.states import NO_LOGPROBS
from vllm.v1.worker.gpu.spec_decode.rejection_sampler_utils import (
    rejection_sample,
)
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.config`, `vllm.triton_utils`, `vllm.v1.outputs`, `vllm.v1.spec_decode.utils`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.config`, `vllm.triton_utils`, `vllm.v1.outputs`, `vllm.v1.spec_decode.utils` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `_flatten_sampled_kernel` function / `_flatten_sampled_kernel` 函数
```python
@triton.jit
def _flatten_sampled_kernel(
    # [num_logits]
    flat_sampled_ptr,
    # [num_reqs, num_speculative_steps + 1]
    sampled_ptr,
    sampled_stride,
    # [num_reqs]
    num_sampled_ptr,
    # [num_reqs + 1]
    cu_num_logits_ptr,
):
    req_idx = tl.program_id(0)
    start_idx = tl.load(cu_num_logits_ptr + req_idx)
    num_sampled = tl.load(num_sampled_ptr + req_idx)
    for i in range(num_sampled):
        token_id = tl.load(sampled_ptr + req_idx * sampled_stride + i)
        tl.store(flat_sampled_ptr + start_idx + i, token_id)
```
**EN:** This function implements `_flatten_sampled_kernel` within the module. Key calls include `program_id`, `load`, `range`, `store`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_flatten_sampled_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `range`, `store`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `RejectionSampler` class / `RejectionSampler` 类
```python
class RejectionSampler:
```
**EN:** Introduces the `RejectionSampler` class. Core methods include `__init__`, `_get_logprobs_tensors`, `__call__`.
**CN:** 这里定义 `RejectionSampler` 类。核心方法包括 `__init__`, `_get_logprobs_tensors`, `__call__`。

### `RejectionSampler.__init__` method / `RejectionSampler.__init__` 方法
```python
    def __init__(
        self,
        sampler: Sampler,
        spec_config: SpeculativeConfig,
        device: torch.device,
    ):
        self.sampler = sampler
        self.num_speculative_steps = spec_config.num_speculative_tokens
        self.rejection_sample_method = spec_config.rejection_sample_method
        self.synthetic_conditional_rates: torch.Tensor | None = None
        if self.rejection_sample_method == "synthetic":
            assert spec_config.synthetic_acceptance_rates is not None
            self.synthetic_conditional_rates = torch.tensor(
                unconditional_to_conditional_rates(
                    spec_config.synthetic_acceptance_rates
                ),
                dtype=torch.float32,
                device=device,
            )
```
**EN:** This method initializes the object state within `RejectionSampler`. Key calls include `tensor`, `unconditional_to_conditional_rates`. It touches state such as `sampler`, `num_speculative_steps`, `rejection_sample_method`, `synthetic_conditional_rates`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`RejectionSampler`。 关键调用包括 `tensor`, `unconditional_to_conditional_rates`。 它会读写 `sampler`, `num_speculative_steps`, `rejection_sample_method`, `synthetic_conditional_rates` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RejectionSampler._get_logprobs_tensors` method / `RejectionSampler._get_logprobs_tensors` 方法
```python
    def _get_logprobs_tensors(
        self,
        input_batch: InputBatch,
        sampled: torch.Tensor,
        num_sampled: torch.Tensor,
        logits: torch.Tensor,
    ) -> LogprobsTensors | None:
        max_num_logprobs = self.sampler.sampling_states.max_num_logprobs(
            input_batch.idx_mapping_np
        )
        if max_num_logprobs == NO_LOGPROBS:
            return None

        num_reqs = input_batch.cu_num_logits.shape[0] - 1
        num_logits = logits.shape[0]
        flat_sampled = torch.zeros(
            num_logits, dtype=sampled.dtype, device=sampled.device
        )
        _flatten_sampled_kernel[(num_reqs,)](
            flat_sampled,
            sampled,
            sampled.stride(0),
            num_sampled,
            input_batch.cu_num_logits,
            num_warps=1,
        )
        expanded_logits = num_logits != input_batch.idx_mapping.shape[0]
        return compute_topk_logprobs(
            logits,
            max_num_logprobs,
            flat_sampled,
            input_batch.cu_num_logits_np.tolist() if expanded_logits else None,
        )
```
**EN:** This method implements `_get_logprobs_tensors` within `RejectionSampler`. Key calls include `max_num_logprobs`, `zeros`, `compute_topk_logprobs`, `stride`, `tolist`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_get_logprobs_tensors`，其作用域位于`RejectionSampler`。 关键调用包括 `max_num_logprobs`, `zeros`, `compute_topk_logprobs`, `stride`, `tolist`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RejectionSampler.__call__` method / `RejectionSampler.__call__` 方法
```python
    def __call__(
        self,
        logits: torch.Tensor,
        input_batch: InputBatch,
        draft_logits: torch.Tensor | None = None,
    ) -> SamplerOutput:
        # NOTE(woosuk): We intentionally compute num_nans before sampling to make clear
        # that num_nans is computed before applying penalties and temperature.
        num_nans = get_num_nans(logits) if self.sampler.compute_nans else None

        draft_sampled = input_batch.input_ids[input_batch.logits_indices]
        pos = input_batch.positions[input_batch.logits_indices]
        processed_logits = self.sampler.apply_sampling_params(
            logits,
            input_batch.expanded_idx_mapping,
            input_batch.idx_mapping_np,
            pos,
            draft_sampled,
            input_batch.expanded_local_pos,
        )
        sampled, num_sampled = rejection_sample(
            processed_logits,
            draft_logits,
            draft_sampled,
            input_batch.cu_num_logits,
            pos,
            input_batch.idx_mapping,
            input_batch.expanded_idx_mapping,
            input_batch.expanded_local_pos,
            self.sampler.sampling_states.temperature.gpu,
            self.sampler.sampling_states.seeds.gpu,
            self.num_speculative_steps,
            self.synthetic_conditional_rates,
            use_fp64=self.sampler.use_fp64_gumbel,
        )
        logprobs_tensors = self._get_logprobs_tensors(
            input_batch,
            sampled,
            num_sampled,
            processed_logits
            if self.sampler.logprobs_mode == "processed_logprobs"
            else logits,
        )

        return SamplerOutput(
            sampled_token_ids=sampled,
            logprobs_tensors=logprobs_tensors,
            num_nans=num_nans,
            num_sampled=num_sampled,
        )
```
**EN:** This method implements `__call__` within `RejectionSampler`. Key calls include `apply_sampling_params`, `rejection_sample`, `_get_logprobs_tensors`, `SamplerOutput`, `get_num_nans`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `__call__`，其作用域位于`RejectionSampler`。 关键调用包括 `apply_sampling_params`, `rejection_sample`, `_get_logprobs_tensors`, `SamplerOutput`, `get_num_nans`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_flatten_sampled_kernel`: top-level helper or orchestration entry point. / `_flatten_sampled_kernel`：顶层辅助函数或编排入口。
- `RejectionSampler`: central class or interface in this module. / `RejectionSampler`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.triton_utils`, `vllm.v1.outputs`, `vllm.v1.spec_decode.utils`, `vllm.v1.worker.gpu.input_batch`, `vllm.v1.worker.gpu.metrics.logits`, `vllm.v1.worker.gpu.sample.logprob`, `vllm.v1.worker.gpu.sample.output`, `vllm.v1.worker.gpu.sample.sampler`, `vllm.v1.worker.gpu.sample.states`, `vllm.v1.worker.gpu.spec_decode.rejection_sampler_utils`
