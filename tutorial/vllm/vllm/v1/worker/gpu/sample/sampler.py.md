# sampler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/sample/sampler.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `Sampler` for the V1 `worker/gpu/sample` subsystem. / 为 V1 的 `worker/gpu/sample` 子系统实现 `Sampler`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import numpy as np
import torch

import vllm.envs as envs
from vllm.config.model import LogprobsMode
from vllm.sampling_params import SamplingParams
from vllm.v1.worker.gpu.input_batch import InputBatch
from vllm.v1.worker.gpu.metrics.logits import get_num_nans
from vllm.v1.worker.gpu.sample.bad_words import BadWordsState
from vllm.v1.worker.gpu.sample.gumbel import gumbel_sample
from vllm.v1.worker.gpu.sample.logit_bias import LogitBiasState
from vllm.v1.worker.gpu.sample.logprob import (
    LogprobTokenIdsState,
    compute_topk_logprobs,
)
from vllm.v1.worker.gpu.sample.output import SamplerOutput
from vllm.v1.worker.gpu.sample.penalties import PenaltiesState
from vllm.v1.worker.gpu.sample.states import NO_LOGPROBS, SamplingStates
from vllm.v1.worker.gpu.states import RequestState
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, and internal vLLM modules such as `vllm.envs`, `vllm.config.model`, `vllm.sampling_params`, `vllm.v1.worker.gpu.input_batch`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖，以及 `vllm.envs`, `vllm.config.model`, `vllm.sampling_params`, `vllm.v1.worker.gpu.input_batch` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `Sampler` class / `Sampler` 类
```python
class Sampler:
```
**EN:** Introduces the `Sampler` class. Core methods include `__init__`, `add_request`, `apply_staged_writes`, `__call__`, `apply_sampling_params`, `sample`.
**CN:** 这里定义 `Sampler` 类。核心方法包括 `__init__`, `add_request`, `apply_staged_writes`, `__call__`, `apply_sampling_params`, `sample`。

### `Sampler.__init__` method / `Sampler.__init__` 方法
```python
    def __init__(
        self,
        max_num_reqs: int,
        vocab_size: int,
        device: torch.device,
        req_states: RequestState,
        logprobs_mode: LogprobsMode = "raw_logprobs",
        num_speculative_tokens: int = 1,
        use_fp64_gumbel: bool = False,
    ):
        if logprobs_mode not in ("processed_logprobs", "raw_logprobs"):
            raise NotImplementedError(f"Unsupported logprobs_mode: {logprobs_mode}")
        self.logprobs_mode = logprobs_mode
        self.compute_nans = envs.VLLM_COMPUTE_NANS_IN_LOGITS  # False by default.
        self.use_fp64_gumbel = use_fp64_gumbel

        self.sampling_states = SamplingStates(max_num_reqs, vocab_size)
        self.penalties_state = PenaltiesState(req_states)
        self.logit_bias_state = LogitBiasState(max_num_reqs, device)
        self.bad_words_state = BadWordsState(req_states)
        self.logprob_token_ids_state = LogprobTokenIdsState(max_num_reqs, device)
        self.num_speculative_tokens = num_speculative_tokens
```
**EN:** This method initializes the object state within `Sampler`. Key calls include `SamplingStates`, `PenaltiesState`, `LogitBiasState`, `BadWordsState`, `LogprobTokenIdsState`, `NotImplementedError`. It touches state such as `logprobs_mode`, `compute_nans`, `use_fp64_gumbel`, `sampling_states`, `penalties_state`, `logit_bias_state`, `bad_words_state`, `logprob_token_ids_state`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`Sampler`。 关键调用包括 `SamplingStates`, `PenaltiesState`, `LogitBiasState`, `BadWordsState`, `LogprobTokenIdsState`, `NotImplementedError`。 它会读写 `logprobs_mode`, `compute_nans`, `use_fp64_gumbel`, `sampling_states`, `penalties_state`, `logit_bias_state`, `bad_words_state`, `logprob_token_ids_state` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `Sampler.add_request` method / `Sampler.add_request` 方法
```python
    def add_request(
        self, req_idx: int, prompt_len: int, sampling_params: SamplingParams
    ) -> None:
        self.sampling_states.add_request(req_idx, sampling_params)
        self.penalties_state.add_request(req_idx, sampling_params)
        self.logit_bias_state.add_request(req_idx, prompt_len, sampling_params)
        self.bad_words_state.add_request(req_idx, sampling_params)
        self.logprob_token_ids_state.add_request(req_idx, sampling_params)
```
**EN:** This method implements `add_request` within `Sampler`. Key calls include `add_request`.
**CN:** 该方法会实现 `add_request`，其作用域位于`Sampler`。 关键调用包括 `add_request`。

### `Sampler.apply_staged_writes` method / `Sampler.apply_staged_writes` 方法
```python
    def apply_staged_writes(self) -> None:
        self.sampling_states.apply_staged_writes()
        self.penalties_state.apply_staged_writes()
        self.logit_bias_state.apply_staged_writes()
        self.bad_words_state.apply_staged_writes()
        self.logprob_token_ids_state.apply_staged_writes()
```
**EN:** This method implements `apply_staged_writes` within `Sampler`. Key calls include `apply_staged_writes`.
**CN:** 该方法会实现 `apply_staged_writes`，其作用域位于`Sampler`。 关键调用包括 `apply_staged_writes`。

### `Sampler.__call__` method / `Sampler.__call__` 方法
```python
    def __call__(
        self,
        logits: torch.Tensor,
        input_batch: InputBatch,
    ) -> SamplerOutput:
        expanded_idx_mapping = input_batch.expanded_idx_mapping
        idx_mapping_np = input_batch.idx_mapping_np
        cu_num_logits_np = input_batch.cu_num_logits_np
        expanded_local_pos = input_batch.expanded_local_pos
        pos = input_batch.positions[input_batch.logits_indices]
        input_ids = input_batch.input_ids[input_batch.logits_indices]

        # NOTE(woosuk): We intentionally compute num_nans before sampling to make clear
        # that num_nans is computed before applying penalties and temperature.
        num_nans = get_num_nans(logits) if self.compute_nans else None
        sampled, processed_logits = self.sample(
            logits,
            expanded_idx_mapping,
            idx_mapping_np,
            pos,
            input_ids,
            expanded_local_pos,
        )

        max_num_logprobs = self.sampling_states.max_num_logprobs(idx_mapping_np)
        max_per_req_token_ids = self.logprob_token_ids_state.max_num_token_ids(
            idx_mapping_np
        )
        if max_num_logprobs != NO_LOGPROBS or max_per_req_token_ids > 0:
            if self.logprobs_mode == "processed_logprobs":
                logits = processed_logits
            expanded_logits = logits.shape[0] != idx_mapping_np.shape[0]
            cu_num_logits = cu_num_logits_np.tolist() if expanded_logits else None
            num_logprobs = max_num_logprobs if max_num_logprobs != NO_LOGPROBS else 0
            logprobs_tensors = compute_topk_logprobs(
                logits,
                num_logprobs,
                sampled,
                cu_num_logits,
                logprob_token_ids_state=self.logprob_token_ids_state,
                expanded_idx_mapping=input_batch.expanded_idx_mapping,
                max_per_req_token_ids=max_per_req_token_ids,
            )
        else:
            logprobs_tensors = None

        # These are GPU tensors.
        sampler_output = SamplerOutput(
            # The sampled tokens are expanded to 2D tensor with shape
            # [num_requests, 1], where each row represents one generated
            # token per request.
            sampled_token_ids=sampled.view(-1, 1),
            logprobs_tensors=logprobs_tensors,
            num_nans=num_nans,
            num_sampled=input_batch.seq_lens.new_ones(input_batch.num_reqs),
        )
        return sampler_output
```
**EN:** This method implements `__call__` within `Sampler`. Key calls include `sample`, `max_num_logprobs`, `max_num_token_ids`, `SamplerOutput`, `get_num_nans`, `compute_topk_logprobs`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `__call__`，其作用域位于`Sampler`。 关键调用包括 `sample`, `max_num_logprobs`, `max_num_token_ids`, `SamplerOutput`, `get_num_nans`, `compute_topk_logprobs`。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `Sampler.apply_sampling_params` method / `Sampler.apply_sampling_params` 方法
```python
    def apply_sampling_params(
        self,
        logits: torch.Tensor,
        expanded_idx_mapping: torch.Tensor,
        idx_mapping_np: np.ndarray,
        pos: torch.Tensor,
        input_ids: torch.Tensor,
        expanded_local_pos: torch.Tensor,
    ) -> torch.Tensor:
        # Copy logits to a new FP32 tensor.
        logits = torch.empty_like(logits, dtype=torch.float32).copy_(logits)

        # Apply logit bias (e.g., allowed_token_ids, min_tokens) in place.
        self.logit_bias_state.apply_logit_bias(
            logits, expanded_idx_mapping, idx_mapping_np, pos
        )

        # Apply penalties in place.
        self.penalties_state.apply_penalties(
            logits,
            expanded_idx_mapping,
            idx_mapping_np,
            input_ids,
            expanded_local_pos,
        )

        # Apply bad words masking in place.
        self.bad_words_state.apply_bad_words(
            logits,
            expanded_idx_mapping,
            idx_mapping_np,
            input_ids,
            expanded_local_pos,
        )

        # Apply temperature in place.
        self.sampling_states.apply_temperature(
            logits, expanded_idx_mapping, idx_mapping_np
        )

        # Apply min_p in place.
        self.sampling_states.apply_min_p(logits, expanded_idx_mapping, idx_mapping_np)

        # Apply top_k and/or top_p. This might or might not return a new tensor.
        return self.sampling_states.apply_top_k_top_p(
            logits, expanded_idx_mapping, idx_mapping_np
        )
```
**EN:** This method implements `apply_sampling_params` within `Sampler`. Key calls include `copy_`, `apply_logit_bias`, `apply_penalties`, `apply_bad_words`, `apply_temperature`, `apply_min_p`.
**CN:** 该方法会实现 `apply_sampling_params`，其作用域位于`Sampler`。 关键调用包括 `copy_`, `apply_logit_bias`, `apply_penalties`, `apply_bad_words`, `apply_temperature`, `apply_min_p`。

### `Sampler.sample` method / `Sampler.sample` 方法
```python
    def sample(
        self,
        logits: torch.Tensor,
        expanded_idx_mapping: torch.Tensor,
        idx_mapping_np: np.ndarray,
        pos: torch.Tensor,
        input_ids: torch.Tensor,
        expanded_local_pos: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        processed_logits = self.apply_sampling_params(
            logits,
            expanded_idx_mapping,
            idx_mapping_np,
            pos,
            input_ids,
            expanded_local_pos,
        )

        # Sample the next token.
        sampled = gumbel_sample(
            processed_logits,
            expanded_idx_mapping,
            self.sampling_states.temperature.gpu,
            self.sampling_states.seeds.gpu,
            pos,
            apply_temperature=False,
            use_fp64=self.use_fp64_gumbel,
        )
        return sampled, processed_logits
```
**EN:** This method samples outputs from model state within `Sampler`. Key calls include `apply_sampling_params`, `gumbel_sample`.
**CN:** 该方法会从模型状态中采样输出，其作用域位于`Sampler`。 关键调用包括 `apply_sampling_params`, `gumbel_sample`。

## Key Concepts / 关键概念
- `Sampler`: central class or interface in this module. / `Sampler`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.config.model`, `vllm.sampling_params`, `vllm.v1.worker.gpu.input_batch`, `vllm.v1.worker.gpu.metrics.logits`, `vllm.v1.worker.gpu.sample.bad_words`, `vllm.v1.worker.gpu.sample.gumbel`, `vllm.v1.worker.gpu.sample.logit_bias`, `vllm.v1.worker.gpu.sample.logprob`, `vllm.v1.worker.gpu.sample.output`, `vllm.v1.worker.gpu.sample.penalties`, `vllm.v1.worker.gpu.sample.states`
