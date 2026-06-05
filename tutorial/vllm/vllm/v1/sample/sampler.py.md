# sampler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/sample/sampler.py`
- **Repository**: vllm-project/vllm
- **Purpose**: A layer that samples the next tokens from the model's outputs. / 该模块位于 `sample` 子系统，主要围绕 `Sampler` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""A layer that samples the next tokens from the model's outputs."""

import torch
import torch.nn as nn

from vllm.config.model import LogprobsMode
from vllm.utils.platform_utils import is_pin_memory_available
from vllm.v1.outputs import LogprobsTensors, SamplerOutput
from vllm.v1.sample.metadata import SamplingMetadata
from vllm.v1.sample.ops.bad_words import apply_bad_words
from vllm.v1.sample.ops.logprobs import batched_count_greater_than
from vllm.v1.sample.ops.penalties import apply_all_penalties
from vllm.v1.sample.ops.topk_topp_sampler import TopKTopPSampler
from vllm.v1.worker.gpu.sample.logprob import compute_token_logprobs

_SAMPLING_EPS = 1e-5
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `_SAMPLING_EPS`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `_SAMPLING_EPS`。

### `Sampler` class / `Sampler` 类
```python
class Sampler(nn.Module):
    """
    A layer that samples the next tokens from the model's outputs
    with the following steps in order:

    1. If logprobs are requested:
        a) If `logprobs_mode` is `raw_logprobs`, compute logprobs
           as the final logprobs to return.
        b) If `logprobs_mode` is `raw_logits`, clone the logits
           as the final logprobs to return.
    2. Convert logits to float32.
    3. Apply allowed token ids whitelist.
    4. Apply bad words exclusion.
    5. Apply logit processors which are not argmax-invariant,
       i.e. that can impact greedy sampling.
        a) Min tokens processor
        b) Logit bias processor
    6. Apply penalties
        a) Repetition penalty
        b) Frequency penalty
        c) Presence penalty
    7. Sample the next tokens. `sample` method performs the following steps:
        a) If not `all_random`, perform greedy sampling. If `all_greedy`,
           return the greedily sampled tokens and final logprobs if requested.
        b) Apply temperature.
        c) Apply logit processors which are argmax-invariant, by default
           the min_p processor.
        d) Apply top_k and/or top_p.
        e) Sample the next tokens with the probability distribution.
        f) If `all_random` or temperature >= epsilon (1e-5), return the
           randomly sampled tokens and final logprobs if requested. Else,
           return the greedily sampled tokens and logprobs if requested.
    8. Gather the logprobs of the top `max_num_logprobs` and sampled token
       (if requested). Note that if the sampled token is within the top
       `max_num_logprobs`, the logprob will be eventually merged in
       `LogprobsProcessor` during output processing. Therefore, the
       final output may contain either `max_num_logprobs + 1` or
       `max_num_logprobs` logprobs.
    9. Return the final `SamplerOutput`.
    """
```
**EN:** Introduces the `Sampler` class on top of `nn.Module`. Core methods include `__init__`, `forward`, `gather_specific_token_logprobs`, `apply_temperature`, `greedy_sample`, `sample`. Docstring signal: A layer that samples the next tokens from the model's outputs with the following steps in order: 1.
**CN:** 这里定义 `Sampler` 类，其基类包括 `nn.Module`。核心方法包括 `__init__`, `forward`, `gather_specific_token_logprobs`, `apply_temperature`, `greedy_sample`, `sample`。

### `Sampler.forward` method / `Sampler.forward` 方法
```python
    def forward(
        self,
        logits: torch.Tensor,
        sampling_metadata: SamplingMetadata,
        predict_bonus_token: bool = False,
        logprobs_mode_override: LogprobsMode | None = None,
    ) -> SamplerOutput:
        logprobs_mode = logprobs_mode_override or self.logprobs_mode
        # NOTE(woosuk): Use the original logits (before any penalties or
        # temperature scaling) for the top-k logprobs.
        # This is different from the V0 sampler, which uses the logits that
        # is used for sampling (after penalties and temperature scaling).
        num_logprobs = sampling_metadata.max_num_logprobs
        if num_logprobs is not None:
            if logprobs_mode == "raw_logprobs":
                raw_logprobs = self.compute_logprobs(logits)
            elif logprobs_mode == "raw_logits":
                if logits.dtype == torch.float32:
                    raw_logprobs = logits.clone()
                else:
                    raw_logprobs = logits.to(torch.float32)

        # Use float32 for the logits.
        logits = logits.to(torch.float32)

        logits = self.apply_logits_processors(
            logits, sampling_metadata, predict_bonus_token
        )
        # Sample the next token.
        sampled, processed_logprobs = self.sample(logits, sampling_metadata)
        if processed_logprobs is not None:
            raw_logprobs = processed_logprobs
        # Convert sampled token ids to int64 (long) type to ensure compatibility
        # with subsequent operations that may use these values as indices.
        # This conversion is necessary because FlashInfer sampling operations
        # return int32 (while PyTorch argmax and topk return int64).
        sampled = sampled.long()

        # Handle logprob_token_ids if specified (more efficient than full vocab)
        # This is used by generative_scoring API to get logprobs for specific tokens
        logprob_token_ids_tensors = None
        if sampling_metadata.logprob_token_ids:
            logprob_token_ids_tensors = self.gather_specific_token_logprobs(
                logits, sampling_metadata.logprob_token_ids, sampled
            )

        if num_logprobs is None:
            logprobs_tensors = logprob_token_ids_tensors
        elif num_logprobs == -1:
            # Return the full unsorted and unranked logprobs.
            logprobs_tensors = LogprobsTensors(
                torch.empty(0), raw_logprobs, torch.empty(0)
            )
        else:
            # Gather the logprobs and ranks of the topk and sampled token.
            logprobs_tensors = self.gather_logprobs(
                raw_logprobs, num_logprobs, token_ids=sampled
            )

        # If we have both num_logprobs and logprob_token_ids, prefer
        # logprob_token_ids as it's more specific
        if logprob_token_ids_tensors is not None and num_logprobs is not None:
            logprobs_tensors = logprob_token_ids_tensors

        # Use int32 to reduce the tensor size.
        sampled = sampled.to(torch.int32)

        # These are GPU tensors.
        sampler_output = SamplerOutput(
            # The sampled tokens are expanded to 2D tensor with shape
            # [num_requests, 1], where each row represents one generated
            # token per request.
            sampled_token_ids=sampled.unsqueeze(-1),
            logprobs_tensors=logprobs_tensors,
        )
        return sampler_output
```
**EN:** This method drives the forward-pass computation within `Sampler`. Key calls include `to`, `apply_logits_processors`, `sample`, `long`, `SamplerOutput`, `gather_specific_token_logprobs`. The control flow contains 9 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会驱动前向计算流程，其作用域位于`Sampler`。 关键调用包括 `to`, `apply_logits_processors`, `sample`, `long`, `SamplerOutput`, `gather_specific_token_logprobs`。 控制流包含 9 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `Sampler.sample` method / `Sampler.sample` 方法
```python
    def sample(
        self,
        logits: torch.Tensor,
        sampling_metadata: SamplingMetadata,
        logprobs_mode_override: LogprobsMode | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        """Sample logits based on sampling metadata.

        The various logits processing functions called in this method
        may update the logits tensor in-place.
        """

        logprobs_mode = logprobs_mode_override or self.logprobs_mode
        assert not (sampling_metadata.all_greedy and sampling_metadata.all_random)
        if sampling_metadata.all_random:
            greedy_sampled = None
        else:
            greedy_sampled = self.greedy_sample(logits)
            if sampling_metadata.all_greedy:
                processed_logprobs = None
                if sampling_metadata.max_num_logprobs is not None:
                    if logprobs_mode == "processed_logits":
                        processed_logprobs = logits
                    elif logprobs_mode == "processed_logprobs":
                        processed_logprobs = self.compute_logprobs(logits)
                return greedy_sampled, processed_logprobs

        assert sampling_metadata.temperature is not None

        # Apply temperature.
        logits = self.apply_temperature(
            logits, sampling_metadata.temperature, sampling_metadata.all_random
        )

        # Apply logits processors that only apply to random sampling
        # (argmax invariant)
        for processor in sampling_metadata.logitsprocs.argmax_invariant:
            logits = processor.apply(logits)

        # Apply top_k and/or top_p.
        random_sampled, processed_logprobs = self.topk_topp_sampler(
            logits,
            sampling_metadata.generators,
            sampling_metadata.top_k,
            sampling_metadata.top_p,
        )

        if greedy_sampled is None:
            return random_sampled, processed_logprobs

        sampled = torch.where(
            sampling_metadata.temperature < _SAMPLING_EPS,
            greedy_sampled,
            random_sampled,
            out=greedy_sampled,  # Reuse tensor
        )
        return sampled, processed_logprobs
```
**EN:** This method samples outputs from model state within `Sampler`. The docstring frames it as: Sample logits based on sampling metadata. Key calls include `apply_temperature`, `topk_topp_sampler`, `where`, `greedy_sample`, `apply`, `compute_logprobs`. The control flow contains 6 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会从模型状态中采样输出，其作用域位于`Sampler`。 关键调用包括 `apply_temperature`, `topk_topp_sampler`, `where`, `greedy_sample`, `apply`, `compute_logprobs`。 控制流包含 6 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `Sampler.compute_logprobs` method / `Sampler.compute_logprobs` 方法
```python
    @staticmethod
    def compute_logprobs(logits: torch.Tensor) -> torch.Tensor:
        return logits.log_softmax(dim=-1, dtype=torch.float32)
```
**EN:** This method computes derived values within `Sampler`. Key calls include `log_softmax`.
**CN:** 该方法会计算派生值，其作用域位于`Sampler`。 关键调用包括 `log_softmax`。

## Key Concepts / 关键概念
- `Sampler`: central class or interface in this module. / `Sampler`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config.model`, `vllm.utils.platform_utils`, `vllm.v1.outputs`, `vllm.v1.sample.metadata`, `vllm.v1.sample.ops.bad_words`, `vllm.v1.sample.ops.logprobs`, `vllm.v1.sample.ops.penalties`, `vllm.v1.sample.ops.topk_topp_sampler`, `vllm.v1.worker.gpu.sample.logprob`
