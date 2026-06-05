# ngram_proposer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/spec_decode/ngram_proposer.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `NgramProposer`, `batch_propose_numba`, `_find_longest_matched_ngram_and_propose_tokens` for the V1 `spec_decode` subsystem. / 为 V1 的 `spec_decode` 子系统实现 `NgramProposer`, `batch_propose_numba`, `_find_longest_matched_ngram_and_propose_tokens`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import os

import numpy as np
import torch
from numba import get_num_threads, jit, njit, prange, set_num_threads

from vllm.config import VllmConfig
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, `numba`, and internal vLLM modules such as `vllm.config`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch`, `numba` 等外部依赖，以及 `vllm.config` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `NgramProposer` class / `NgramProposer` 类
```python
class NgramProposer:
```
**EN:** Introduces the `NgramProposer` class. Core methods include `__init__`, `batch_propose`, `propose`, `load_model`.
**CN:** 这里定义 `NgramProposer` 类。核心方法包括 `__init__`, `batch_propose`, `propose`, `load_model`。

### `NgramProposer.__init__` method / `NgramProposer.__init__` 方法
```python
    def __init__(self, vllm_config: VllmConfig):
        assert vllm_config.speculative_config is not None
        assert vllm_config.speculative_config.prompt_lookup_min is not None
        assert vllm_config.speculative_config.prompt_lookup_max is not None

        # Minimum length of the n-gram to match.
        self.min_n = vllm_config.speculative_config.prompt_lookup_min
        # Maximum length of the n-gram to match.
        self.max_n = vllm_config.speculative_config.prompt_lookup_max
        # Number of tokens follow the match. If there are less than k
        # tokens follow the match, we will return the maximum amount of
        # tokens until the end.
        self.k = vllm_config.speculative_config.num_speculative_tokens
        # Maximum length of the model.
        self.max_model_len = vllm_config.model_config.max_model_len

        # Pre-allocate buffers for numba batch propose.
        max_num_seqs = vllm_config.scheduler_config.max_num_seqs
        self.valid_ngram_draft = np.zeros((max_num_seqs, self.k), dtype=np.int32)
        self.valid_ngram_num_drafts = np.zeros((max_num_seqs), dtype=np.int32)

        # Threshold of total number of tokens in the batch to enable
        # multi-threading in numba batch propose.
        self.num_tokens_threshold = 8192
        tp_size = vllm_config.parallel_config.tensor_parallel_size
        cpu_count = os.cpu_count()
        # Max number of threads for numba parallel processing.
        if cpu_count:
            # Divide by 2 to use physical cores
            # and not logical cores (hyper-threading).
            # Cap the number of threads to 8 to avoid using too many threads
            # since other components like frontend (incl tokenization)
            # and Structured Outputs also use multiple threads.
            # TODO(ekagra-ranjan): bump up the cap from 1 to 8
            # when TP parallelization for ngram is implemented.
            self.num_numba_thread_available = min(1, (cpu_count // 2))
            # Divide by tp_size to ensure each tensor parallel rank
            # has some threads since all ranks will run this.
            self.num_numba_thread_available //= tp_size
        else:
            self.num_numba_thread_available = 1

        # Trigger Numba JIT compilation for N-gram proposer.
        # This usually takes less than 1 second.
        self.propose(
            [[]] * 1024,
            np.zeros(1024, dtype=np.int32),
            np.zeros((1024, self.max_model_len), dtype=np.int32),
        )
```
**EN:** This method initializes the object state within `NgramProposer`. Key calls include `zeros`, `cpu_count`, `propose`, `min`. It touches state such as `min_n`, `max_n`, `k`, `max_model_len`, `valid_ngram_draft`, `valid_ngram_num_drafts`, `num_tokens_threshold`, `num_numba_thread_available`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`NgramProposer`。 关键调用包括 `zeros`, `cpu_count`, `propose`, `min`。 它会读写 `min_n`, `max_n`, `k`, `max_model_len`, `valid_ngram_draft`, `valid_ngram_num_drafts`, `num_tokens_threshold`, `num_numba_thread_available` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `NgramProposer.batch_propose` method / `NgramProposer.batch_propose` 方法
```python
    def batch_propose(
        self,
        num_requests: int,
        valid_ngram_requests: list,
        num_tokens_no_spec: np.ndarray,
        token_ids_cpu: np.ndarray,
    ) -> list[list[int]]:
        """Batch version of ngram proposer using numba for acceleration.

        Args:
            valid_ngram_requests:
                Set of indices of requests that need ngram proposals.
            num_tokens_no_spec:
                Numpy array of shape (batch_size,) representing the number
                of tokens without speculative tokens for each request.
            token_ids_cpu:
                Numpy array of shape (batch_size, max_model_len)
                representing the token IDs for each request.

        Returns:
            list[list[int]]:
                A list where each element is a list of proposed
                token IDs for the corresponding request.
        """
        draft_token_ids: list[list[int]] = []

        # Only run batch propose if there are requests needing ngram proposals.
        # avoid calling numba function with empty list which causes error
        # ValueError: cannot compute fingerprint of empty list
        if num_ngram_requests := len(valid_ngram_requests):
            original_num_numba_threads = get_num_threads()
            # Ensure we use at least one thread.
            # If total tokens is small, using multiple threads
            # may slow down due to overhead.
            total_tokens = np.sum(num_tokens_no_spec)
            if total_tokens >= self.num_tokens_threshold:
                final_num_threads = max(
                    1, min(self.num_numba_thread_available, num_ngram_requests)
                )
                set_num_threads(final_num_threads)
            else:
                set_num_threads(1)

            batch_propose_numba(
                valid_ngram_requests,
                num_tokens_no_spec,
                token_ids_cpu,
                self.min_n,
                self.max_n,
                self.max_model_len,
                self.k,
                self.valid_ngram_draft,
                self.valid_ngram_num_drafts,
            )

            # Restore original number of threads.
            set_num_threads(original_num_numba_threads)

        for i in range(num_requests):
            if i in valid_ngram_requests and self.valid_ngram_num_drafts[i] > 0:
                draft_token_ids.append(
                    self.valid_ngram_draft[i, : self.valid_ngram_num_drafts[i]].tolist()
                )
            else:
                draft_token_ids.append([])

        return draft_token_ids
```
**EN:** This method implements `batch_propose` within `NgramProposer`. The docstring frames it as: Batch version of ngram proposer using numba for acceleration. Key calls include `range`, `len`, `get_num_threads`, `sum`, `batch_propose_numba`, `set_num_threads`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `batch_propose`，其作用域位于`NgramProposer`。 关键调用包括 `range`, `len`, `get_num_threads`, `sum`, `batch_propose_numba`, `set_num_threads`。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `NgramProposer.propose` method / `NgramProposer.propose` 方法
```python
    def propose(
        self,
        sampled_token_ids: list[list[int]],
        num_tokens_no_spec: np.ndarray,
        token_ids_cpu: np.ndarray,
        slot_mappings: dict[str, torch.Tensor]
        | list[dict[str, torch.Tensor]]
        | None = None,  # unused
    ) -> list[list[int]]:
        # find which requests need ngram proposals
        valid_ngram_requests = []
        for i, sampled_ids in enumerate(sampled_token_ids):
            num_sampled_ids = len(sampled_ids)
            if not num_sampled_ids:
                # Skip speculative decoding.
                continue

            num_tokens = num_tokens_no_spec[i]
            if num_tokens >= self.max_model_len:
                # Skip requests that have already reached the max model length.
                continue

            valid_ngram_requests.append(i)

        draft_token_ids = self.batch_propose(
            len(sampled_token_ids),
            valid_ngram_requests,
            num_tokens_no_spec,
            token_ids_cpu,
        )

        return draft_token_ids
```
**EN:** This method implements `propose` within `NgramProposer`. Key calls include `enumerate`, `batch_propose`, `len`, `append`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `propose`，其作用域位于`NgramProposer`。 关键调用包括 `enumerate`, `batch_propose`, `len`, `append`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `NgramProposer.load_model` method / `NgramProposer.load_model` 方法
```python
    def load_model(self, *args, **kwargs):
        # No model to load.
        pass
```
**EN:** This method loads external or cached state within `NgramProposer`.
**CN:** 该方法会加载外部或缓存状态，其作用域位于`NgramProposer`。

### `batch_propose_numba` function / `batch_propose_numba` 函数
```python
@njit(parallel=True)
def batch_propose_numba(
    valid_ngram_requests: list,
    num_tokens_no_spec: np.ndarray,
    token_ids_cpu: np.ndarray,
    min_n: int,
    max_n: int,
    max_model_len: int,
    k: int,
    valid_ngram_draft: np.ndarray,
    valid_ngram_num_drafts: np.ndarray,
):
    for i in prange(len(valid_ngram_requests)):
        idx = valid_ngram_requests[i]
        num_tokens = num_tokens_no_spec[idx]
        context_token_ids = token_ids_cpu[idx, :num_tokens]
        drafter_output = _find_longest_matched_ngram_and_propose_tokens(
            origin_tokens=context_token_ids,
            min_ngram=min_n,
            max_ngram=max_n,
            max_model_len=max_model_len,
            k=k,
        )

        valid_ngram_num_drafts[idx] = drafter_output.shape[0]
        if len(drafter_output):
            valid_ngram_draft[idx, : drafter_output.shape[0]] = drafter_output
```
**EN:** This function implements `batch_propose_numba` within the module. Key calls include `njit`, `prange`, `len`, `_find_longest_matched_ngram_and_propose_tokens`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `batch_propose_numba`，其作用域位于the module。 关键调用包括 `njit`, `prange`, `len`, `_find_longest_matched_ngram_and_propose_tokens`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `_find_longest_matched_ngram_and_propose_tokens` function / `_find_longest_matched_ngram_and_propose_tokens` 函数
```python
@jit(nopython=True)
def _find_longest_matched_ngram_and_propose_tokens(
    origin_tokens: np.ndarray,
    min_ngram: int,
    max_ngram: int,
    max_model_len: int,
    k: int,
) -> np.ndarray:
    """
    Find the longest n-gram which matches the suffix of the given tokens
    whose length is within [min_ngram, max_ngram] (inclusive).

    If found, we will extract k right after the matched ngram.
    """
    # Do not generate draft tokens is context is shorter than minimum n-gram
    total_token = origin_tokens.shape[0]
    if total_token < min_ngram:
        return np.empty((0,), dtype=origin_tokens.dtype)

    # Do not generate draft tokens beyond the max model length.
    k = min(k, max_model_len - total_token)
    if k <= 0:
        return np.empty((0,), dtype=origin_tokens.dtype)

    # Flip tokens, and the goal become to find longest ngram
    # on the rightmost position which matches the prefix with
    # length [min_n, max_n] (inclusive).
    tokens = origin_tokens[::-1]

    # Longest prefix (not including itself) which is a suffix of
    # the current position.
    #   lps[i] = max{v, where tokens[0:v] == tokens[i+1-v:i+1]}
    #
    # As ngram is capped by max_ngram to save memory, we only need to
    # store lps for the first max_ngram prefix.
    lps = np.zeros(max_ngram, dtype=np.int32)

    longest_ngram = 0
    position = 0

    # lps[0] always equal to 0, we start with index 1
    prev_lps = 0
    i = 1
    while i < total_token:
        # tokens[:prev_lps] is the longest prefix as a suffix of tokens[:i]
        if tokens[prev_lps] == tokens[i]:
            # Token match: tokens[:prev_lps+1] is the longest prefix as
            # a suffix of tokens[:i+1]
            prev_lps += 1
            # Check if we found a longer valid ngram.
            #
            # Update position when longest_ngram matched prev_lps,
            # as we want to get the target n-gram of the earliest position
            # in the original tokens (i.e.
            # latest position in the reversed tokens)
            if prev_lps >= longest_ngram:
                longest_ngram = prev_lps
                position = i
            if i < max_ngram:
                # Store LPS for the first max_ngram prefix
                lps[i] = prev_lps
            if prev_lps == max_ngram:
                # When prev_lps reached max_ngram, update prev_lps
                # to lps[max_ngram-1] to avoid matching ngram
                # longer than max_ngram
                prev_lps = lps[max_ngram - 1]
            i += 1
        elif prev_lps != 0:
            # Token mismatch: try the second-longest prefix
            # among all suffix of tokens[:i],
            # which is the longest prefix of tokens[:prev_lps]
            prev_lps = lps[prev_lps - 1]
        else:
            # Token mismatch, and no more prefix (except empty string)
            # as a suffix of tokens[:i]
            i += 1

    if longest_ngram < min_ngram:
        # No valid ngram is found
        return np.empty((0,), dtype=origin_tokens.dtype)

    # Flip the position back, so in origin_tokens,
    # origin_tokens[total_token-1-position:total_token-1-position+longest_ngram]
    # is the matched ngram, so we should start drafting tokens from
    # total_token-1-position+longest_ngram
    start_position = total_token - 1 - position + longest_ngram
    k = min(k, total_token - start_position)
    return origin_tokens[start_position : start_position + k]
```
**EN:** This function implements `_find_longest_matched_ngram_and_propose_tokens` within the module. The docstring frames it as: Find the longest n-gram which matches the suffix of the given tokens whose length is within [min_ngram, max_ngram] (inclusive). Key calls include `jit`, `min`, `zeros`, `empty`. The control flow contains 8 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_find_longest_matched_ngram_and_propose_tokens`，其作用域位于the module。 关键调用包括 `jit`, `min`, `zeros`, `empty`。 控制流包含 8 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `NgramProposer`: central class or interface in this module. / `NgramProposer`：本模块中的核心类或接口。
- `batch_propose_numba`: top-level helper or orchestration entry point. / `batch_propose_numba`：顶层辅助函数或编排入口。
- `_find_longest_matched_ngram_and_propose_tokens`: top-level helper or orchestration entry point. / `_find_longest_matched_ngram_and_propose_tokens`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `os`
- External / 外部依赖: `numpy`, `torch`, `numba`
- Internal vLLM / 内部依赖: `vllm.config`
