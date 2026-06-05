# outputs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/outputs.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `LogprobsLists`, `LogprobsTensors`, `RoutedExpertsTensors` for the V1 `v1` subsystem. / 为 V1 的 `v1` 子系统实现 `LogprobsLists`, `LogprobsTensors`, `RoutedExpertsTensors`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from typing import TYPE_CHECKING, NamedTuple, TypeAlias

import numpy as np
import torch

from vllm.compilation.cuda_graph import CUDAGraphStat
from vllm.v1.core.sched.output import SchedulerOutput

if TYPE_CHECKING:
    from vllm.distributed.kv_events import KVConnectorKVEvents
    from vllm.distributed.kv_transfer.kv_connector.v1.base import (
        KVConnectorWorkerMetadata,
    )
    from vllm.distributed.kv_transfer.kv_connector.v1.metrics import KVConnectorStats
else:
    KVConnectorStats = object
    KVConnectorWorkerMetadata = object
    KVConnectorKVEvents = object
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, and internal vLLM modules such as `vllm.compilation.cuda_graph`, `vllm.v1.core.sched.output`, `vllm.distributed.kv_events`, `vllm.distributed.kv_transfer.kv_connector.v1.base`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖，以及 `vllm.compilation.cuda_graph`, `vllm.v1.core.sched.output`, `vllm.distributed.kv_events`, `vllm.distributed.kv_transfer.kv_connector.v1.base` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `LogprobsLists` class / `LogprobsLists` 类
```python
class LogprobsLists(NamedTuple):
    # [num_reqs x num_generated_tokens, max_num_logprobs + 1]
    logprob_token_ids: np.ndarray
    # [num_reqs x num_generated_tokens, max_num_logprobs + 1]
    logprobs: np.ndarray
    # [num_reqs x num_generated_tokens]
    sampled_token_ranks: np.ndarray
    # [num_reqs]
    # Used for slicing the logprobs in cases like speculative
    # decoding where the number of generated tokens may be
    # different for each request.
    cu_num_generated_tokens: list[int] | None = None
```
**EN:** Introduces the `LogprobsLists` class on top of `NamedTuple`. Core methods include `slice_request`.
**CN:** 这里定义 `LogprobsLists` 类，其基类包括 `NamedTuple`。核心方法包括 `slice_request`。

### `LogprobsLists.slice_request` method / `LogprobsLists.slice_request` 方法
```python
    def slice_request(self, req_idx: int, num_positions: int):
        if self.cu_num_generated_tokens is not None:
            req_idx = self.cu_num_generated_tokens[req_idx]
        end_idx = req_idx + num_positions
        return LogprobsLists(
            self.logprob_token_ids[req_idx:end_idx],
            self.logprobs[req_idx:end_idx],
            self.sampled_token_ranks[req_idx:end_idx],
            None,
        )
```
**EN:** This method implements `slice_request` within `LogprobsLists`. Key calls include `LogprobsLists`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `slice_request`，其作用域位于`LogprobsLists`。 关键调用包括 `LogprobsLists`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `LogprobsTensors` class / `LogprobsTensors` 类
```python
class LogprobsTensors(NamedTuple):
    # [num_reqs x num_generated_tokens, max_num_logprobs + 1]
    logprob_token_ids: torch.Tensor
    # [num_reqs x num_generated_tokens, max_num_logprobs + 1]
    logprobs: torch.Tensor
    # [num_reqs x num_generated_tokens]
    selected_token_ranks: torch.Tensor
    # [num_reqs]
    cu_num_generated_tokens: list[int] | None = None
```
**EN:** Introduces the `LogprobsTensors` class on top of `NamedTuple`. Core methods include `tolists`, `to_cpu_nonblocking`, `filter`, `empty_cpu`.
**CN:** 这里定义 `LogprobsTensors` 类，其基类包括 `NamedTuple`。核心方法包括 `tolists`, `to_cpu_nonblocking`, `filter`, `empty_cpu`。

### `LogprobsTensors.tolists` method / `LogprobsTensors.tolists` 方法
```python
    def tolists(self, cu_num_generated_tokens: list[int] | None = None):
        return LogprobsLists(
            self.logprob_token_ids.cpu().numpy(),
            self.logprobs.cpu().numpy(),
            self.selected_token_ranks.cpu().numpy(),
            cu_num_generated_tokens
            if cu_num_generated_tokens is not None
            else self.cu_num_generated_tokens,
        )
```
**EN:** This method implements `tolists` within `LogprobsTensors`. Key calls include `LogprobsLists`, `numpy`, `cpu`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `tolists`，其作用域位于`LogprobsTensors`。 关键调用包括 `LogprobsLists`, `numpy`, `cpu`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `LogprobsTensors.to_cpu_nonblocking` method / `LogprobsTensors.to_cpu_nonblocking` 方法
```python
    def to_cpu_nonblocking(self) -> "LogprobsTensors":
        if self.logprob_token_ids.device.type == "cpu":
            return self
        return LogprobsTensors(
            self.logprob_token_ids.to("cpu", non_blocking=True),
            self.logprobs.to("cpu", non_blocking=True),
            self.selected_token_ranks.to("cpu", non_blocking=True),
            self.cu_num_generated_tokens,
        )
```
**EN:** This method converts data to another representation within `LogprobsTensors`. Key calls include `LogprobsTensors`, `to`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会将数据转换为另一种表示，其作用域位于`LogprobsTensors`。 关键调用包括 `LogprobsTensors`, `to`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `LogprobsTensors.filter` method / `LogprobsTensors.filter` 方法
```python
    def filter(self, mask: torch.Tensor) -> "LogprobsTensors":
        """Filter the logprobs tensors with the given bool mask."""
        assert self.cu_num_generated_tokens is None, (
            "filter can't be used with cu_num_generated_tokens"
        )
        return LogprobsTensors(
            self.logprob_token_ids[mask],
            self.logprobs[mask],
            self.selected_token_ranks[mask],
        )
```
**EN:** This method implements `filter` within `LogprobsTensors`. The docstring frames it as: Filter the logprobs tensors with the given bool mask. Key calls include `LogprobsTensors`.
**CN:** 该方法会实现 `filter`，其作用域位于`LogprobsTensors`。 关键调用包括 `LogprobsTensors`。

### `LogprobsTensors.empty_cpu` method / `LogprobsTensors.empty_cpu` 方法
```python
    @staticmethod
    def empty_cpu(
        num_positions: int, num_tokens_per_position: int
    ) -> "LogprobsTensors":
        """Create empty LogprobsTensors on CPU."""

        logprob_token_ids = torch.empty(
            (num_positions, num_tokens_per_position), dtype=torch.int32, device="cpu"
        )
        logprobs = torch.empty_like(logprob_token_ids, dtype=torch.float32)
        selected_token_ranks = torch.empty(
            num_positions, dtype=torch.int32, device="cpu"
        )
        return LogprobsTensors(
            logprob_token_ids=logprob_token_ids,
            logprobs=logprobs,
            selected_token_ranks=selected_token_ranks,
        )
```
**EN:** This method implements `empty_cpu` within `LogprobsTensors`. The docstring frames it as: Create empty LogprobsTensors on CPU. Key calls include `empty`, `empty_like`, `LogprobsTensors`.
**CN:** 该方法会实现 `empty_cpu`，其作用域位于`LogprobsTensors`。 关键调用包括 `empty`, `empty_like`, `LogprobsTensors`。

### `RoutedExpertsTensors` class / `RoutedExpertsTensors` 类
```python
class RoutedExpertsTensors(NamedTuple):
    """Device-side snapshot of routed experts data, pending async D2H.

    Produced by :class:`GPUModelRunner` at the end of each async-scheduled
    step. The copy stream waits on the default stream, then issues
    non-blocking D2H via :meth:`to_cpu_nonblocking` into a pinned CPU
    buffer; :class:`AsyncGPUModelRunnerOutput.get_output` synchronizes
    the copy before the scheduler reads it.

    Sliced to ``total_num_scheduled_tokens`` (step-level, across all
    requests — NOT per-request). Both ``routing_data`` and
    ``slot_mapping`` must be private clones when sourced from shared
    capturer / prepare-input buffers, so the next forward pass /
    ``_prepare_inputs`` on the default stream does not race with a
    D2H still pending on the copy stream.
    """

    # (num_scheduled_tokens, num_layers, num_experts_per_tok)
    routing_data: torch.Tensor
    # (num_scheduled_tokens,)
    slot_mapping: torch.Tensor
```
**EN:** Introduces the `RoutedExpertsTensors` class on top of `NamedTuple`. Core methods include `to_cpu_nonblocking`, `tolists`. Docstring signal: Device-side snapshot of routed experts data, pending async D2H.
**CN:** 这里定义 `RoutedExpertsTensors` 类，其基类包括 `NamedTuple`。核心方法包括 `to_cpu_nonblocking`, `tolists`。

### `RoutedExpertsTensors.to_cpu_nonblocking` method / `RoutedExpertsTensors.to_cpu_nonblocking` 方法
```python
    def to_cpu_nonblocking(self) -> "RoutedExpertsTensors":
        """Issue non-blocking D2H on the current stream.

        NOTE: ``non_blocking=True`` only delivers true overlap when the
        CPU target is pinned. The current fallback here allocates a
        new pageable CPU tensor per call, which silently degrades to a
        synchronous copy; acceptable because the sync happens on the
        dedicated copy stream, not the default stream.
        """
        if self.routing_data.device.type == "cpu":
            return self
        return RoutedExpertsTensors(
            self.routing_data.to("cpu", non_blocking=True),
            self.slot_mapping.to("cpu", non_blocking=True),
        )
```
**EN:** This method converts data to another representation within `RoutedExpertsTensors`. The docstring frames it as: Issue non-blocking D2H on the current stream. Key calls include `RoutedExpertsTensors`, `to`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会将数据转换为另一种表示，其作用域位于`RoutedExpertsTensors`。 关键调用包括 `RoutedExpertsTensors`, `to`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RoutedExpertsTensors.tolists` method / `RoutedExpertsTensors.tolists` 方法
```python
    def tolists(self) -> "RoutedExpertsLists":
        """Convert to the numpy-backed form consumed by the scheduler.

        ``.cpu()`` is a no-op when the tensor is already on CPU, so this
        is cheap for the post-D2H case; for raw device tensors it will
        synchronously block, which is only reached in tests.
        """
        return RoutedExpertsLists(
            self.routing_data.cpu().numpy(),
            self.slot_mapping.cpu().numpy(),
        )
```
**EN:** This method implements `tolists` within `RoutedExpertsTensors`. The docstring frames it as: Convert to the numpy-backed form consumed by the scheduler. Key calls include `RoutedExpertsLists`, `numpy`, `cpu`.
**CN:** 该方法会实现 `tolists`，其作用域位于`RoutedExpertsTensors`。 关键调用包括 `RoutedExpertsLists`, `numpy`, `cpu`。

### `RoutedExpertsLists` class / `RoutedExpertsLists` 类
```python
class RoutedExpertsLists(NamedTuple):
    """CPU-side routed experts, the form :meth:`RoutedExpertsManager.store_batch`
    consumes.

    Batched per scheduler step: the leading dim is the number of tokens
    scheduled across all requests in this step (``total_num_scheduled_tokens``),
    not per-request tokens. ``slot_mapping[i]`` tells the scheduler which
    physical KV-cache slot row ``i`` of ``routing_data`` belongs to.
    """

    # (num_scheduled_tokens, num_layers, num_experts_per_tok)
    routing_data: np.ndarray
    # (num_scheduled_tokens,)
    slot_mapping: np.ndarray
```
**EN:** Introduces the `RoutedExpertsLists` class on top of `NamedTuple`. Core methods include its methods defined below. Docstring signal: CPU-side routed experts, the form :meth:`RoutedExpertsManager.store_batch` consumes.
**CN:** 这里定义 `RoutedExpertsLists` 类，其基类包括 `NamedTuple`。核心方法包括 下方定义的方法。

### Module constants / 模块常量
```python
PoolerOutput: TypeAlias = torch.Tensor | list[torch.Tensor] | list[torch.Tensor | None]
```
**EN:** Defines module-level constants or aliases such as `PoolerOutput`, which are reused by later definitions.
**CN:** 定义 `PoolerOutput` 等模块级常量或别名，供后续定义复用。

### `SamplerOutput` class / `SamplerOutput` 类
```python
@dataclass
class SamplerOutput:
    # [num_reqs, max_num_generated_tokens]
    # Different requests can have different number of generated tokens.
    # All requests are padded to max_num_generated_tokens.
    # PLACEHOLDER_TOKEN_ID (-1 by default) is used for padding.
    sampled_token_ids: torch.Tensor
    logprobs_tensors: LogprobsTensors | None
```
**EN:** Uses `@dataclass` to package related state for `SamplerOutput`. Typical fields include `sampled_token_ids`, `logprobs_tensors`.
**CN:** `SamplerOutput` 使用 `@dataclass` 打包相关状态。典型字段包括 `sampled_token_ids`, `logprobs_tensors`。

### `KVConnectorOutput` class / `KVConnectorOutput` 类
```python
@dataclass
class KVConnectorOutput:
    # [req_ids]
    finished_sending: set[str] | None = None
    finished_recving: set[str] | None = None
    kv_connector_stats: KVConnectorStats | None = None
    kv_cache_events: KVConnectorKVEvents | None = None
    kv_connector_worker_meta: KVConnectorWorkerMetadata | None = None
    # IDs of externally computed KV blocks that failed to load.
    # Requests referencing these blocks should be rescheduled to recompute them
    invalid_block_ids: set[int] = field(default_factory=set)
    # Configuration describing how many finished sending/receiving
    # notifications should be expected for each request. This allows
    # handshake-based connectors like Nixl to update the KVOutputAggregator.
    # It captures a static setup info and should almost always remain constant
    # for a given connector after discovery. Default value entails no change.
    expected_finished_count: int = 0
```
**EN:** Uses `@dataclass` to package related state for `KVConnectorOutput`. Typical fields include `finished_sending`, `finished_recving`, `kv_connector_stats`, `kv_cache_events`, `kv_connector_worker_meta`, `invalid_block_ids`.
**CN:** `KVConnectorOutput` 使用 `@dataclass` 打包相关状态。典型字段包括 `finished_sending`, `finished_recving`, `kv_connector_stats`, `kv_cache_events`, `kv_connector_worker_meta`, `invalid_block_ids`。

### `KVConnectorOutput.is_empty` method / `KVConnectorOutput.is_empty` 方法
```python
    def is_empty(self):
        return (
            not self.finished_sending
            and not self.finished_recving
            and not self.kv_connector_stats
            and not self.kv_cache_events
            and not self.invalid_block_ids
            and not self.kv_connector_worker_meta
        )
```
**EN:** This method answers a boolean capability check within `KVConnectorOutput`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`KVConnectorOutput`。

### `ECConnectorOutput` class / `ECConnectorOutput` 类
```python
@dataclass
class ECConnectorOutput:
    # [mm_hash]
    finished_sending: set[str] | None = None
    finished_recving: set[str] | None = None
```
**EN:** Uses `@dataclass` to package related state for `ECConnectorOutput`. Typical fields include `finished_sending`, `finished_recving`.
**CN:** `ECConnectorOutput` 使用 `@dataclass` 打包相关状态。典型字段包括 `finished_sending`, `finished_recving`。

### `ModelRunnerOutput` class / `ModelRunnerOutput` 类
```python
@dataclass
class ModelRunnerOutput:
    # [num_reqs]
    req_ids: list[str]
    # req_id -> index
    req_id_to_index: dict[str, int]

    # num_reqs x num_generated_tokens
    # num_generated_tokens is the number of tokens
    # generated in the current step. It can be different for
    # each request due to speculative/jump decoding.
    sampled_token_ids: list[list[int]] = field(default_factory=list)

    # [num_reqs, max_num_logprobs + 1]
    # [num_reqs, max_num_logprobs + 1]
    # [num_reqs]
    logprobs: LogprobsLists | None = None

    # req_id -> (token_ids, logprobs, ranks)
    # [prompt_len, num_prompt_logprobs]
    # [prompt_len, num_prompt_logprobs]
    # [prompt_len]
    prompt_logprobs_dict: dict[str, LogprobsTensors | None] = field(
        default_factory=dict
    )

    # [num_reqs, hidden_size]
    pooler_output: list[torch.Tensor | None] | None = None

    kv_connector_output: KVConnectorOutput | None = None

    ec_connector_output: ECConnectorOutput | None = None

    # req_id -> num_nans_in_logits
    num_nans_in_logits: dict[str, int] | None = None

    # information related to cudagraph execution
    cudagraph_stats: CUDAGraphStat | None = None

    # Per-step routed experts data captured by the worker.
    # ``routing_data`` shape: (num_scheduled_tokens, num_layers,
    #                         num_experts_per_tok); expert IDs as uint8/uint16.
    # ``slot_mapping`` shape: (num_scheduled_tokens,); physical KV-cache
    #                         slot for each row of routing_data.
    # ``num_scheduled_tokens`` is step-level (total across all requests
    # in this step), not per-request. The scheduler persists this into
    # its slot buffer via ``slot_buffer[slot_mapping] = routing_data``.
    # ``None`` when ``enable_return_routed_experts`` is off.
    routed_experts: RoutedExpertsLists | None = None
```
**EN:** Uses `@dataclass` to package related state for `ModelRunnerOutput`. Typical fields include `req_ids`, `req_id_to_index`, `sampled_token_ids`, `logprobs`, `prompt_logprobs_dict`, `pooler_output`.
**CN:** `ModelRunnerOutput` 使用 `@dataclass` 打包相关状态。典型字段包括 `req_ids`, `req_id_to_index`, `sampled_token_ids`, `logprobs`, `prompt_logprobs_dict`, `pooler_output`。

### `AsyncModelRunnerOutput` class / `AsyncModelRunnerOutput` 类
```python
class AsyncModelRunnerOutput(ABC):
```
**EN:** Declares the `AsyncModelRunnerOutput` interface. Downstream implementations are expected to provide methods such as `get_output`.
**CN:** `AsyncModelRunnerOutput` 声明了一组接口约定。下游实现需要提供 `get_output` 等方法。

### `AsyncModelRunnerOutput.get_output` method / `AsyncModelRunnerOutput.get_output` 方法
```python
    @abstractmethod
    def get_output(self) -> ModelRunnerOutput:
        """Get the ModelRunnerOutput for this async output.

        This is a blocking call that waits until the results are ready, which
        might involve copying device tensors to the host.
        This method should only be called once per AsyncModelRunnerOutput.
        """
        pass
```
**EN:** This method returns or derives a value within `AsyncModelRunnerOutput`. The docstring frames it as: Get the ModelRunnerOutput for this async output.
**CN:** 该方法会返回或推导一个值，其作用域位于`AsyncModelRunnerOutput`。

### `DraftTokenIds` class / `DraftTokenIds` 类
```python
@dataclass
class DraftTokenIds:
    # [num_reqs]
    req_ids: list[str]
    # num_reqs x num_draft_tokens
    draft_token_ids: list[list[int]]
```
**EN:** Uses `@dataclass` to package related state for `DraftTokenIds`. Typical fields include `req_ids`, `draft_token_ids`.
**CN:** `DraftTokenIds` 使用 `@dataclass` 打包相关状态。典型字段包括 `req_ids`, `draft_token_ids`。

### `make_empty_encoder_model_runner_output` function / `make_empty_encoder_model_runner_output` 函数
```python
def make_empty_encoder_model_runner_output(
    scheduler_output: "SchedulerOutput",
) -> ModelRunnerOutput:
    """
    Create a ModelRunnerOutput stub that contains the correct
    per-request bookkeeping but no generated data yet.
    """
    if not scheduler_output.num_scheduled_tokens:
        return EMPTY_MODEL_RUNNER_OUTPUT

    # Convert to list so we get a deterministic, indexable sequence
    req_ids: list[str] = list(scheduler_output.num_scheduled_tokens.keys())

    # Give every request its own contiguous index
    req_id_to_index: dict[str, int] = {rid: idx for idx, rid in enumerate(req_ids)}

    # No tokens generated yet ⇒ one empty list per request
    sampled_token_ids: list[list[int]] = [[0] for _ in req_ids]

    # Pooler outputs are not available yet ⇒ use None placeholders
    pooler_output: list[torch.Tensor | None] = [None for _ in req_ids]

    return ModelRunnerOutput(
        req_ids=req_ids,
        req_id_to_index=req_id_to_index,
        sampled_token_ids=sampled_token_ids,
        pooler_output=pooler_output,
    )
```
**EN:** This function implements `make_empty_encoder_model_runner_output` within the module. The docstring frames it as: Create a ModelRunnerOutput stub that contains the correct per-request bookkeeping but no generated data yet. Key calls include `list`, `ModelRunnerOutput`, `keys`, `enumerate`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `make_empty_encoder_model_runner_output`，其作用域位于the module。 关键调用包括 `list`, `ModelRunnerOutput`, `keys`, `enumerate`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### Module constants / 模块常量
```python
EMPTY_MODEL_RUNNER_OUTPUT = ModelRunnerOutput(req_ids=[], req_id_to_index={})
```
**EN:** Defines module-level constants or aliases such as `EMPTY_MODEL_RUNNER_OUTPUT`, which are reused by later definitions.
**CN:** 定义 `EMPTY_MODEL_RUNNER_OUTPUT` 等模块级常量或别名，供后续定义复用。

## Key Concepts / 关键概念
- `LogprobsLists`: central class or interface in this module. / `LogprobsLists`：本模块中的核心类或接口。
- `LogprobsTensors`: central class or interface in this module. / `LogprobsTensors`：本模块中的核心类或接口。
- `RoutedExpertsTensors`: central class or interface in this module. / `RoutedExpertsTensors`：本模块中的核心类或接口。
- `RoutedExpertsLists`: central class or interface in this module. / `RoutedExpertsLists`：本模块中的核心类或接口。
- `SamplerOutput`: central class or interface in this module. / `SamplerOutput`：本模块中的核心类或接口。
- `KVConnectorOutput`: central class or interface in this module. / `KVConnectorOutput`：本模块中的核心类或接口。
- `ECConnectorOutput`: central class or interface in this module. / `ECConnectorOutput`：本模块中的核心类或接口。
- `ModelRunnerOutput`: central class or interface in this module. / `ModelRunnerOutput`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `abc`, `dataclasses`, `typing`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.compilation.cuda_graph`, `vllm.v1.core.sched.output`, `vllm.distributed.kv_events`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.distributed.kv_transfer.kv_connector.v1.metrics`
