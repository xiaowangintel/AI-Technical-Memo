# output.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/core/sched/output.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `NewRequestData`, `CachedRequestData`, `SchedulerOutput` for the V1 `core/sched` subsystem. / 为 V1 的 `core/sched` 子系统实现 `NewRequestData`, `CachedRequestData`, `SchedulerOutput`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from dataclasses import dataclass
from functools import cached_property
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    import numpy as np
    import numpy.typing as npt
    import torch

    from vllm.distributed.ec_transfer.ec_connector.base import ECConnectorMetadata
    from vllm.distributed.kv_transfer.kv_connector.v1.base import KVConnectorMetadata
    from vllm.lora.request import LoRARequest
    from vllm.multimodal.inputs import MultiModalFeatureSpec
    from vllm.pooling_params import PoolingParams
    from vllm.sampling_params import SamplingParams
    from vllm.v1.request import Request
else:
    ECConnectorMetadata = object
    KVConnectorMetadata = object
    LoRARequest = object
    MultiModalFeatureSpec = object
    PoolingParams = object
    SamplingParams = object
    Request = object
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, and internal vLLM modules such as `vllm.distributed.ec_transfer.ec_connector.base`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.lora.request`, `vllm.multimodal.inputs`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖，以及 `vllm.distributed.ec_transfer.ec_connector.base`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.lora.request`, `vllm.multimodal.inputs` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `NewRequestData` class / `NewRequestData` 类
```python
@dataclass
class NewRequestData:
    req_id: str
    prompt_token_ids: list[int] | None
    mm_features: list[MultiModalFeatureSpec]
    sampling_params: SamplingParams | None
    pooling_params: PoolingParams | None
    block_ids: tuple[list[int], ...]
    num_computed_tokens: int
    lora_request: LoRARequest | None
    prompt_embeds: "torch.Tensor | None" = None
    prompt_is_token_ids: list[bool] | None = None

    # Only used for v2 model runner.
    prefill_token_ids: list[int] | None = None
```
**EN:** Uses `@dataclass` to package related state for `NewRequestData`. Typical fields include `req_id`, `prompt_token_ids`, `mm_features`, `sampling_params`, `pooling_params`, `block_ids`.
**CN:** `NewRequestData` 使用 `@dataclass` 打包相关状态。典型字段包括 `req_id`, `prompt_token_ids`, `mm_features`, `sampling_params`, `pooling_params`, `block_ids`。

### `NewRequestData.from_request` method / `NewRequestData.from_request` 方法
```python
    @classmethod
    def from_request(
        cls,
        request: Request,
        block_ids: tuple[list[int], ...],
        prefill_token_ids: list[int] | None = None,
    ) -> "NewRequestData":
        return cls(
            req_id=request.request_id,
            prompt_token_ids=request.prompt_token_ids,
            mm_features=request.mm_features,
            sampling_params=request.sampling_params,
            pooling_params=request.pooling_params,
            block_ids=block_ids,
            num_computed_tokens=request.num_computed_tokens,
            lora_request=request.lora_request,
            prompt_embeds=request.prompt_embeds,
            prompt_is_token_ids=request.prompt_is_token_ids,
            prefill_token_ids=prefill_token_ids,
        )
```
**EN:** This method reconstructs data from another representation within `NewRequestData`. Key calls include `cls`.
**CN:** 该方法会从另一种表示恢复数据，其作用域位于`NewRequestData`。 关键调用包括 `cls`。

### `NewRequestData.__repr__` method / `NewRequestData.__repr__` 方法
```python
    def __repr__(self) -> str:
        prompt_embeds_shape = (
            self.prompt_embeds.shape if self.prompt_embeds is not None else None
        )
        return (
            f"NewRequestData("
            f"req_id={self.req_id},"
            f"prompt_token_ids={self.prompt_token_ids},"
            f"prefill_token_ids={self.prefill_token_ids},"
            f"mm_features={self.mm_features},"
            f"sampling_params={self.sampling_params},"
            f"block_ids={self.block_ids},"
            f"num_computed_tokens={self.num_computed_tokens},"
            f"lora_request={self.lora_request},"
            f"prompt_embeds_shape={prompt_embeds_shape}"
            ")"
        )
```
**EN:** This method implements `__repr__` within `NewRequestData`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `__repr__`，其作用域位于`NewRequestData`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `NewRequestData.anon_repr` method / `NewRequestData.anon_repr` 方法
```python
    def anon_repr(self) -> str:
        prompt_token_ids_len = (
            len(self.prompt_token_ids) if self.prompt_token_ids is not None else None
        )
        prompt_embeds_shape = (
            self.prompt_embeds.shape if self.prompt_embeds is not None else None
        )
        prefill_token_ids_len = (
            len(self.prefill_token_ids) if self.prefill_token_ids is not None else None
        )
        return (
            f"NewRequestData("
            f"req_id={self.req_id},"
            f"prompt_token_ids_len={prompt_token_ids_len},"
            f"prefill_token_ids_len={prefill_token_ids_len},"
            f"mm_features={self.mm_features},"
            f"sampling_params={self.sampling_params},"
            f"block_ids={self.block_ids},"
            f"num_computed_tokens={self.num_computed_tokens},"
            f"lora_request={self.lora_request},"
            f"prompt_embeds_shape={prompt_embeds_shape}"
            ")"
        )
```
**EN:** This method implements `anon_repr` within `NewRequestData`. Key calls include `len`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `anon_repr`，其作用域位于`NewRequestData`。 关键调用包括 `len`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CachedRequestData` class / `CachedRequestData` 类
```python
@dataclass
class CachedRequestData:
    req_ids: list[str]
    # For request ids not in resumed_req_ids, new_block_ids will be appended to
    # the request's block IDs. For those in the set, new_block_ids will be used as the
    # request's block IDs instead of appending to the existing block IDs.
    resumed_req_ids: set[str]
    # NOTE(woosuk): new_token_ids is only used for pipeline parallelism.
    # When PP is not used, new_token_ids will be empty.
    new_token_ids: list[list[int]]
    # For requests not scheduled in the last step, propagate the token ids to the
    # connector. Won't contain requests that were scheduled in the prior step.
    all_token_ids: dict[str, list[int]]
    new_block_ids: list[tuple[list[int], ...] | None]
    num_computed_tokens: list[int]
    num_output_tokens: list[int]

    # Version of dataclass repr with token IDs obfuscated.
```
**EN:** Uses `@dataclass` to package related state for `CachedRequestData`. Typical fields include `req_ids`, `resumed_req_ids`, `new_token_ids`, `all_token_ids`, `new_block_ids`, `num_computed_tokens`.
**CN:** `CachedRequestData` 使用 `@dataclass` 打包相关状态。典型字段包括 `req_ids`, `resumed_req_ids`, `new_token_ids`, `all_token_ids`, `new_block_ids`, `num_computed_tokens`。

### `CachedRequestData.anon_repr` method / `CachedRequestData.anon_repr` 方法
```python
    def anon_repr(self) -> str:
        new_token_ids_lens = [len(toks) for toks in self.new_token_ids]
        all_token_ids_lens = {
            req_id: len(toks) for req_id, toks in self.all_token_ids.items()
        }
        return (
            f"CachedRequestData("
            f"req_ids={self.req_ids},"
            f"resumed_req_ids={self.resumed_req_ids},"
            f"new_token_ids_lens={new_token_ids_lens},"
            f"all_token_ids_lens={all_token_ids_lens},"
            f"new_block_ids={self.new_block_ids},"
            f"num_computed_tokens={self.num_computed_tokens},"
            f"num_output_tokens={self.num_output_tokens}"
            f")"
        )
```
**EN:** This method implements `anon_repr` within `CachedRequestData`. Key calls include `len`, `items`.
**CN:** 该方法会实现 `anon_repr`，其作用域位于`CachedRequestData`。 关键调用包括 `len`, `items`。

### `CachedRequestData.__repr__` method / `CachedRequestData.__repr__` 方法
```python
    def __repr__(self) -> str:
        return self.anon_repr()
```
**EN:** This method implements `__repr__` within `CachedRequestData`. Key calls include `anon_repr`.
**CN:** 该方法会实现 `__repr__`，其作用域位于`CachedRequestData`。 关键调用包括 `anon_repr`。

### `CachedRequestData.num_reqs` method / `CachedRequestData.num_reqs` 方法
```python
    @property
    def num_reqs(self) -> int:
        return len(self.req_ids)
```
**EN:** This method implements `num_reqs` within `CachedRequestData`. Key calls include `len`.
**CN:** 该方法会实现 `num_reqs`，其作用域位于`CachedRequestData`。 关键调用包括 `len`。

### `CachedRequestData._req_id_to_num_output_tokens` method / `CachedRequestData._req_id_to_num_output_tokens` 方法
```python
    @cached_property
    def _req_id_to_num_output_tokens(self) -> dict[str, int]:
        """Cache mapping of req_id to num_output_tokens for O(1) lookup.

        This cached property is safe because CachedRequestData instances
        are created fresh each scheduling iteration and not mutated during
        computation of iteration details.
        """
        return dict(zip(self.req_ids, self.num_output_tokens))
```
**EN:** This method implements `_req_id_to_num_output_tokens` within `CachedRequestData`. The docstring frames it as: Cache mapping of req_id to num_output_tokens for O(1) lookup. Key calls include `dict`, `zip`.
**CN:** 该方法会实现 `_req_id_to_num_output_tokens`，其作用域位于`CachedRequestData`。 关键调用包括 `dict`, `zip`。

### `CachedRequestData.is_context_phase` method / `CachedRequestData.is_context_phase` 方法
```python
    def is_context_phase(self, req_id: str) -> bool:
        num_output_tokens = self._req_id_to_num_output_tokens.get(req_id)
        return num_output_tokens is not None and num_output_tokens == 0
```
**EN:** This method answers a boolean capability check within `CachedRequestData`. Key calls include `get`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`CachedRequestData`。 关键调用包括 `get`。

### `CachedRequestData.make_empty` method / `CachedRequestData.make_empty` 方法
```python
    @classmethod
    def make_empty(cls) -> "CachedRequestData":
        return cls(
            req_ids=[],
            resumed_req_ids=set(),
            new_token_ids=[],
            all_token_ids={},
            new_block_ids=[],
            num_computed_tokens=[],
            num_output_tokens=[],
        )
```
**EN:** This method implements `make_empty` within `CachedRequestData`. Key calls include `cls`, `set`.
**CN:** 该方法会实现 `make_empty`，其作用域位于`CachedRequestData`。 关键调用包括 `cls`, `set`。

### `SchedulerOutput` class / `SchedulerOutput` 类
```python
@dataclass
class SchedulerOutput:
    # list of the requests that are scheduled for the first time.
    # We cache the request's data in each worker process, so that we don't
    # need to re-send it every scheduling step.
    scheduled_new_reqs: list[NewRequestData]
    # list of the requests that have been scheduled before.
    # Since the request's data is already cached in the worker processes,
    # we only send the diff to minimize the communication cost.
    scheduled_cached_reqs: CachedRequestData

    # req_id -> num_scheduled_tokens
    # Number of tokens scheduled for each request.
    num_scheduled_tokens: dict[str, int]
    # Total number of tokens scheduled for all requests.
    # Equal to sum(num_scheduled_tokens.values())
    total_num_scheduled_tokens: int
    # req_id -> spec_token_ids
    # If a request does not have any spec decode tokens, it will not be
    # included in the dictionary.
    scheduled_spec_decode_tokens: dict[str, list[int]]
    # req_id -> encoder input indices that need processing.
    # E.g., if a request has [0, 1], it could mean the vision encoder needs
    # to process that the request's 0-th and 1-th images in the current step.
    scheduled_encoder_inputs: dict[str, list[int]]
    # Number of common prefix blocks for all requests in each KV cache group.
    # This can be used for cascade attention.
    num_common_prefix_blocks: list[int]

    # Request IDs that are finished in between the previous and the current
    # steps. This is used to notify the workers about the finished requests
    # so that they can free the cached states for those requests.
    finished_req_ids: set[str]
    # list of mm_hash strings associated with the encoder outputs to be
    # freed from the encoder cache.
    free_encoder_mm_hashes: list[str]

    # Request IDs that are preempted in this step.
    # Only used for v2 model runner.
    preempted_req_ids: set[str] | None = None

    # Whether any of the scheduled requests use structured output.
    # Set only in async scheduling case.
    has_structured_output_requests: bool = False

    # Whether the scheduled requests have all the output tokens they
    # need to perform grammar bitmask computation.
    pending_structured_output_tokens: bool = False

    # Used for adjusting acceptance rate calculation.
    num_invalid_spec_tokens: dict[str, int] | None = None

    # KV Cache Connector metadata.
    kv_connector_metadata: KVConnectorMetadata | None = None

    # EC Cache Connector metadata
    ec_connector_metadata: ECConnectorMetadata | None = None

    # Block IDs freshly allocated from the pool during this scheduling step.
    # The worker zeros the corresponding GPU memory before the blocks are used,
    # preventing stale NaN/data from corrupting attention or SSM computation.
    new_block_ids_to_zero: list[int] | None = None
```
**EN:** Uses `@dataclass` to package related state for `SchedulerOutput`. Typical fields include `scheduled_new_reqs`, `scheduled_cached_reqs`, `num_scheduled_tokens`, `total_num_scheduled_tokens`, `scheduled_spec_decode_tokens`, `scheduled_encoder_inputs`.
**CN:** `SchedulerOutput` 使用 `@dataclass` 打包相关状态。典型字段包括 `scheduled_new_reqs`, `scheduled_cached_reqs`, `num_scheduled_tokens`, `total_num_scheduled_tokens`, `scheduled_spec_decode_tokens`, `scheduled_encoder_inputs`。

### `SchedulerOutput.make_empty` method / `SchedulerOutput.make_empty` 方法
```python
    @classmethod
    def make_empty(cls) -> "SchedulerOutput":
        return cls(
            scheduled_new_reqs=[],
            scheduled_cached_reqs=CachedRequestData.make_empty(),
            num_scheduled_tokens={},
            total_num_scheduled_tokens=0,
            scheduled_spec_decode_tokens={},
            scheduled_encoder_inputs={},
            num_common_prefix_blocks=[],
            finished_req_ids=set(),
            free_encoder_mm_hashes=[],
        )
```
**EN:** This method implements `make_empty` within `SchedulerOutput`. Key calls include `cls`, `make_empty`, `set`.
**CN:** 该方法会实现 `make_empty`，其作用域位于`SchedulerOutput`。 关键调用包括 `cls`, `make_empty`, `set`。

### `GrammarOutput` class / `GrammarOutput` 类
```python
@dataclass
class GrammarOutput:
    # ids of structured output requests.
    structured_output_request_ids: list[str]
    # Bitmask ordered as structured_output_request_ids.
    grammar_bitmask: "npt.NDArray[np.int32]"
```
**EN:** Uses `@dataclass` to package related state for `GrammarOutput`. Typical fields include `structured_output_request_ids`, `grammar_bitmask`.
**CN:** `GrammarOutput` 使用 `@dataclass` 打包相关状态。典型字段包括 `structured_output_request_ids`, `grammar_bitmask`。

## Key Concepts / 关键概念
- `NewRequestData`: central class or interface in this module. / `NewRequestData`：本模块中的核心类或接口。
- `CachedRequestData`: central class or interface in this module. / `CachedRequestData`：本模块中的核心类或接口。
- `SchedulerOutput`: central class or interface in this module. / `SchedulerOutput`：本模块中的核心类或接口。
- `GrammarOutput`: central class or interface in this module. / `GrammarOutput`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `functools`, `typing`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.distributed.ec_transfer.ec_connector.base`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.lora.request`, `vllm.multimodal.inputs`, `vllm.pooling_params`, `vllm.sampling_params`, `vllm.v1.request`
