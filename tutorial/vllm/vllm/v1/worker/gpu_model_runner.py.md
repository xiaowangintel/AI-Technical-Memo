# gpu_model_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu_model_runner.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `AsyncGPUModelRunnerOutput`, `_copy_pooler_output_to_cpu`, `AsyncGPUPoolingModelRunnerOutput` for the V1 `worker` subsystem. / 为 V1 的 `worker` 子系统实现 `AsyncGPUModelRunnerOutput`, `_copy_pooler_output_to_cpu`, `AsyncGPUPoolingModelRunnerOutput`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import functools
import gc
import itertools
import threading
import time
from collections import defaultdict
from collections.abc import Callable, Iterable, Iterator, Sequence
from contextlib import contextmanager
from copy import copy, deepcopy
from dataclasses import dataclass, replace
from functools import reduce
from typing import TYPE_CHECKING, Any, NamedTuple, TypeAlias, cast

import numpy as np
import torch
import torch.distributed
import torch.nn as nn
from tqdm import tqdm

import vllm.envs as envs
from vllm.compilation.breakable_cudagraph import (
    BreakableCUDAGraphWrapper,
    is_breakable_cudagraph_enabled,
)
from vllm.compilation.counter import compilation_counter
from vllm.compilation.cuda_graph import CUDAGraphStat, CUDAGraphWrapper
from vllm.compilation.monitor import set_cudagraph_capturing_enabled
from vllm.config import (
    CompilationMode,
    CUDAGraphMode,
    VllmConfig,
    get_layers_from_vllm_config,
    set_current_vllm_config,
    update_config,
)
from vllm.config.cache import CacheConfig
from vllm.distributed.ec_transfer import get_ec_transfer, has_ec_transfer
from vllm.distributed.eplb.eplb_state import EplbState
from vllm.distributed.kv_transfer import get_kv_transfer_group, has_kv_transfer_group
from vllm.distributed.kv_transfer.kv_connector.utils import copy_kv_blocks
from vllm.distributed.parallel_state import (
    get_dcp_group,
    get_pp_group,
    get_tp_group,
    graph_capture,
    is_global_first_rank,
    prepare_communication_buffer_for_model,
)
from vllm.forward_context import (
    BatchDescriptor,
# ... omitted for brevity ...
    maybe_create_ubatch_slices,
    split_attn_metadata,
)
from vllm.v1.worker.utils import is_residual_scattered_for_sp
from vllm.v1.worker.workspace import lock_workspace

from .utils import (
    AttentionGroup,
    KVBlockZeroer,
    add_kv_sharing_layers_to_kv_cache_groups,
    bind_kv_cache,
    prepare_kernel_block_sizes,
    sanity_check_mm_encoder_outputs,
)

if TYPE_CHECKING:
    from vllm.v1.core.sched.output import GrammarOutput, SchedulerOutput
    from vllm.v1.spec_decode.ngram_proposer import NgramProposer
    from vllm.v1.worker.encoder_cudagraph import EncoderCudaGraphManager

logger = init_logger(__name__)

AttnMetadataDict: TypeAlias = dict[str, AttentionMetadata]
# list when ubatching is enabled
PerLayerAttnMetadata: TypeAlias = list[AttnMetadataDict] | AttnMetadataDict
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `AttnMetadataDict`, `PerLayerAttnMetadata`. This excerpt omits repetitive declarations.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `AttnMetadataDict`, `PerLayerAttnMetadata`。 该片段省略了重复性声明。

### `AsyncGPUModelRunnerOutput` class / `AsyncGPUModelRunnerOutput` 类
```python
class AsyncGPUModelRunnerOutput(AsyncModelRunnerOutput):
```
**EN:** Introduces the `AsyncGPUModelRunnerOutput` class on top of `AsyncModelRunnerOutput`. Core methods include `__init__`, `get_output`.
**CN:** 这里定义 `AsyncGPUModelRunnerOutput` 类，其基类包括 `AsyncModelRunnerOutput`。核心方法包括 `__init__`, `get_output`。

### `AsyncGPUModelRunnerOutput.__init__` method / `AsyncGPUModelRunnerOutput.__init__` 方法
```python
    def __init__(
        self,
        model_runner_output: ModelRunnerOutput,
        sampled_token_ids: torch.Tensor,
        logprobs_tensors: LogprobsTensors | None,
        invalid_req_indices: list[int],
        async_output_copy_stream: torch.cuda.Stream,
        vocab_size: int,
        routed_experts: RoutedExpertsTensors | None = None,
    ):
        self._model_runner_output = model_runner_output
        self._invalid_req_indices = invalid_req_indices

        # Event on the copy stream so we can synchronize the non-blocking copy.
        self.async_copy_ready_event = torch.Event()

        # Keep a reference to the device tensor to avoid it being
        # deallocated until we finish copying it to the host.
        self._sampled_token_ids = sampled_token_ids
        self.vocab_size = vocab_size
        self._logprobs_tensors = logprobs_tensors
        self._routed_experts = routed_experts

        # Initiate the copy on a separate stream, but do not synchronize it.
        default_stream = torch.cuda.current_stream()
        with torch.cuda.stream(async_output_copy_stream):
            async_output_copy_stream.wait_stream(default_stream)
            self.sampled_token_ids_cpu = self._sampled_token_ids.to(
                "cpu", non_blocking=True
            )
            self._logprobs_tensors_cpu = (
                self._logprobs_tensors.to_cpu_nonblocking()
                if self._logprobs_tensors
                else None
            )
            self._routed_experts_cpu = (
                self._routed_experts.to_cpu_nonblocking()
                if self._routed_experts is not None
                else None
            )
            self.async_copy_ready_event.record()
```
**EN:** This method initializes the object state within `AsyncGPUModelRunnerOutput`. Key calls include `Event`, `current_stream`, `stream`, `wait_stream`, `to`, `record`. It touches state such as `_model_runner_output`, `_invalid_req_indices`, `async_copy_ready_event`, `_sampled_token_ids`, `vocab_size`, `_logprobs_tensors`, `_routed_experts`, `sampled_token_ids_cpu`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`AsyncGPUModelRunnerOutput`。 关键调用包括 `Event`, `current_stream`, `stream`, `wait_stream`, `to`, `record`。 它会读写 `_model_runner_output`, `_invalid_req_indices`, `async_copy_ready_event`, `_sampled_token_ids`, `vocab_size`, `_logprobs_tensors`, `_routed_experts`, `sampled_token_ids_cpu` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AsyncGPUModelRunnerOutput.get_output` method / `AsyncGPUModelRunnerOutput.get_output` 方法
```python
    def get_output(self) -> ModelRunnerOutput:
        """Copy the device tensors to the host and return a ModelRunnerOutput.

        This function blocks until the copy is finished.
        """
        max_gen_len = self.sampled_token_ids_cpu.shape[-1]
        self.async_copy_ready_event.synchronize()

        # Release the device tensors once the copy has completed.
        del self._logprobs_tensors
        del self._sampled_token_ids
        if max_gen_len == 1:
            valid_sampled_token_ids = self.sampled_token_ids_cpu.tolist()
            for i in self._invalid_req_indices:
                valid_sampled_token_ids[i].clear()
            logprobs_lists = None
            if self._logprobs_tensors_cpu is not None:
                logprobs_lists = self._logprobs_tensors_cpu.tolists()
        else:
            valid_sampled_token_ids, logprobs_lists = RejectionSampler.parse_output(
                self.sampled_token_ids_cpu,
                self.vocab_size,
                self._invalid_req_indices,
                logprobs_tensors=self._logprobs_tensors_cpu,
            )

        output = self._model_runner_output
        output.sampled_token_ids = valid_sampled_token_ids
        output.logprobs = logprobs_lists

        if self._routed_experts_cpu is not None:
            output.routed_experts = self._routed_experts_cpu.tolists()
        del self._routed_experts

        return output
```
**EN:** This method returns or derives a value within `AsyncGPUModelRunnerOutput`. The docstring frames it as: Copy the device tensors to the host and return a ModelRunnerOutput. Key calls include `synchronize`, `tolist`, `parse_output`, `tolists`, `clear`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`AsyncGPUModelRunnerOutput`。 关键调用包括 `synchronize`, `tolist`, `parse_output`, `tolists`, `clear`。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `_copy_pooler_output_to_cpu` function / `_copy_pooler_output_to_cpu` 函数
```python
def _copy_pooler_output_to_cpu(
    raw_pooler_output: PoolerOutput, finished_mask: list[bool]
) -> list[torch.Tensor | None]:
    num_reqs = len(finished_mask)

    if isinstance(raw_pooler_output, torch.Tensor):
        if raw_pooler_output.shape[0] != num_reqs:
            raise ValueError(
                "Pooler output batch size does not match finished mask size: "
                f"{raw_pooler_output.shape[0]} != {num_reqs}."
            )

        num_finished = sum(finished_mask)
        if num_finished == 0:
            return [None] * num_reqs
        if num_finished == num_reqs:
            return list(raw_pooler_output.to("cpu", non_blocking=True))

        # partial finished
        finished_indices = [i for i, include in enumerate(finished_mask) if include]
        index_tensor = torch.tensor(
            finished_indices, device=raw_pooler_output.device, dtype=torch.long
        )
        finished_outputs = raw_pooler_output.index_select(0, index_tensor).to(
            "cpu", non_blocking=True
        )
        partial_pooler_output: list[torch.Tensor | None] = [None] * num_reqs
        for i, out in zip(finished_indices, finished_outputs):
            partial_pooler_output[i] = out
        return partial_pooler_output

    assert isinstance(raw_pooler_output, list)
    if len(raw_pooler_output) != num_reqs:
        raise ValueError(
            "Pooler output batch size does not match finished mask size: "
            f"{len(raw_pooler_output)} != {num_reqs}."
        )

    pooler_output: list[torch.Tensor | None] = [None] * num_reqs
    for i, (out, include) in enumerate(zip(raw_pooler_output, finished_mask)):
        if include and out is not None:
            pooler_output[i] = out.to("cpu", non_blocking=True)
    return pooler_output
```
**EN:** This function implements `_copy_pooler_output_to_cpu` within the module. Key calls include `len`, `isinstance`, `enumerate`, `sum`, `tensor`, `to`. The control flow contains 6 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_copy_pooler_output_to_cpu`，其作用域位于the module。 关键调用包括 `len`, `isinstance`, `enumerate`, `sum`, `tensor`, `to`。 控制流包含 6 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `AsyncGPUPoolingModelRunnerOutput` class / `AsyncGPUPoolingModelRunnerOutput` 类
```python
class AsyncGPUPoolingModelRunnerOutput(AsyncModelRunnerOutput):
```
**EN:** Introduces the `AsyncGPUPoolingModelRunnerOutput` class on top of `AsyncModelRunnerOutput`. Core methods include `__init__`, `get_output`.
**CN:** 这里定义 `AsyncGPUPoolingModelRunnerOutput` 类，其基类包括 `AsyncModelRunnerOutput`。核心方法包括 `__init__`, `get_output`。

### `AsyncGPUPoolingModelRunnerOutput.__init__` method / `AsyncGPUPoolingModelRunnerOutput.__init__` 方法
```python
    def __init__(
        self,
        model_runner_output: ModelRunnerOutput,
        raw_pooler_output: PoolerOutput,
        finished_mask: list[bool],
        async_output_copy_stream: torch.cuda.Stream,
    ):
        self._model_runner_output = model_runner_output

        # Event on the copy stream so we can synchronize the non-blocking copy.
        self.async_copy_ready_event = torch.Event()

        # Keep a reference to the device tensors to avoid them being
        # deallocated until we finish copying it to the host.
        self._raw_pooler_output = raw_pooler_output

        # Initiate the copy on a separate stream, but do not synchronize it.
        default_stream = torch.cuda.current_stream()
        with torch.cuda.stream(async_output_copy_stream):
            async_output_copy_stream.wait_stream(default_stream)
            self._model_runner_output.pooler_output = _copy_pooler_output_to_cpu(
                raw_pooler_output=self._raw_pooler_output,
                finished_mask=finished_mask,
            )
            self.async_copy_ready_event.record()
```
**EN:** This method initializes the object state within `AsyncGPUPoolingModelRunnerOutput`. Key calls include `Event`, `current_stream`, `stream`, `wait_stream`, `_copy_pooler_output_to_cpu`, `record`. It touches state such as `_model_runner_output`, `async_copy_ready_event`, `_raw_pooler_output`.
**CN:** 该方法会初始化对象状态，其作用域位于`AsyncGPUPoolingModelRunnerOutput`。 关键调用包括 `Event`, `current_stream`, `stream`, `wait_stream`, `_copy_pooler_output_to_cpu`, `record`。 它会读写 `_model_runner_output`, `async_copy_ready_event`, `_raw_pooler_output` 等状态。

### `AsyncGPUPoolingModelRunnerOutput.get_output` method / `AsyncGPUPoolingModelRunnerOutput.get_output` 方法
```python
    def get_output(self) -> ModelRunnerOutput:
        """Copy the device tensors to the host and return a ModelRunnerOutput.
        This function blocks until the copy is finished.
        """
        self.async_copy_ready_event.synchronize()

        # Release the device tensors once the copy has completed.
        del self._raw_pooler_output
        return self._model_runner_output
```
**EN:** This method returns or derives a value within `AsyncGPUPoolingModelRunnerOutput`. The docstring frames it as: Copy the device tensors to the host and return a ModelRunnerOutput. Key calls include `synchronize`.
**CN:** 该方法会返回或推导一个值，其作用域位于`AsyncGPUPoolingModelRunnerOutput`。 关键调用包括 `synchronize`。

### `ExecuteModelState` class / `ExecuteModelState` 类
```python
class ExecuteModelState(NamedTuple):
    """Ephemeral cached state transferred between execute_model() and
    sample_tokens(), after execute_model() returns None."""

    scheduler_output: "SchedulerOutput"
    logits: torch.Tensor
    spec_decode_metadata: SpecDecodeMetadata | None
    spec_decode_common_attn_metadata: CommonAttentionMetadata | None
    hidden_states: torch.Tensor
    sample_hidden_states: torch.Tensor
    aux_hidden_states: list[torch.Tensor] | None
    ec_connector_output: ECConnectorOutput | None
    cudagraph_stats: CUDAGraphStat | None
    slot_mappings: dict[str, torch.Tensor] | list[dict[str, torch.Tensor]] | None
```
**EN:** Introduces the `ExecuteModelState` class on top of `NamedTuple`. Core methods include its methods defined below. Docstring signal: Ephemeral cached state transferred between execute_model() and sample_tokens(), after execute_model() returns None.
**CN:** 这里定义 `ExecuteModelState` 类，其基类包括 `NamedTuple`。核心方法包括 下方定义的方法。

### `GPUModelRunner` class / `GPUModelRunner` 类
```python
class GPUModelRunner(
    LoRAModelRunnerMixin, KVConnectorModelRunnerMixin, ECConnectorModelRunnerMixin
):
```
**EN:** Introduces the `GPUModelRunner` class on top of `LoRAModelRunnerMixin`, `KVConnectorModelRunnerMixin`, `ECConnectorModelRunnerMixin`. Core methods include `__init__`, `update_max_model_len`, `reset_mm_cache`, `reset_encoder_cache`, `post_kv_cache_wake_up`, `init_fp8_kv_scales`.
**CN:** 这里定义 `GPUModelRunner` 类，其基类包括 `LoRAModelRunnerMixin`, `KVConnectorModelRunnerMixin`, `ECConnectorModelRunnerMixin`。核心方法包括 `__init__`, `update_max_model_len`, `reset_mm_cache`, `reset_encoder_cache`, `post_kv_cache_wake_up`, `init_fp8_kv_scales`。

### `GPUModelRunner._update_states_after_model_execute` method / `GPUModelRunner._update_states_after_model_execute` 方法
```python
    def _update_states_after_model_execute(
        self, output_token_ids: torch.Tensor, scheduler_output: "SchedulerOutput"
    ) -> None:
        """Update the cached states after model execution.

        This is used for MTP/EAGLE for hybrid models, as in linear attention,
        only the last token's state is kept. In MTP/EAGLE, for draft tokens
        the state are kept util we decide how many tokens are accepted for
        each sequence, and a shifting is done during the next iteration
        based on the number of accepted tokens.
        """
        if not self.speculative_config or not self.model_config.is_hybrid:
            return

        # TODO: Remove .cpu() sync to enable fully async for hybrid model;
        # Use num_computed_tokens.gpu instead of req.num_computed_tokens to
        # support aligned mamba cache mode.
        # Count the number of accepted tokens for each sequence.
        # Valid tokens are contiguous from position 0, so counting non-(-1)
        # tokens gives us the first -1 position (i.e., number of accepted).
        num_reqs = output_token_ids.size(0)
        self.num_accepted_tokens.gpu[:num_reqs] = (output_token_ids != -1).sum(dim=1)

        is_align = self.cache_config.mamba_cache_mode == "align"
        if is_align:
            for i, num_tokens in enumerate(
                self.num_accepted_tokens.gpu[:num_reqs].cpu().numpy()
            ):
                self.input_batch.num_accepted_tokens_cpu[i] = num_tokens
        else:
            self.input_batch.num_accepted_tokens_cpu_tensor[:num_reqs].copy_(
                self.num_accepted_tokens.gpu[:num_reqs], non_blocking=True
            )
            assert self.num_accepted_tokens_event is not None
            self.num_accepted_tokens_event.record()

        mamba_utils.postprocess_mamba(
            scheduler_output,
            self.kv_cache_config,
            self.cache_config,
            self.input_batch,
            self.requests,
            self.mamba_state_idx,
            self.num_spec_tokens,
            num_reqs,
            forward_context=(
                self.compilation_config.static_forward_context if is_align else None
            ),
            mamba_state_copy_funcs=(
                self.model.get_mamba_state_copy_func() if is_align else None
            ),
            copy_bufs=self._get_mamba_copy_bufs() if is_align else None,
        )
```
**EN:** This method implements `_update_states_after_model_execute` within `GPUModelRunner`. The docstring frames it as: Update the cached states after model execution. Key calls include `size`, `sum`, `postprocess_mamba`, `enumerate`, `copy_`, `record`. It touches state such as `num_accepted_tokens`, `input_batch`. The control flow contains 5 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_update_states_after_model_execute`，其作用域位于`GPUModelRunner`。 关键调用包括 `size`, `sum`, `postprocess_mamba`, `enumerate`, `copy_`, `record`。 它会读写 `num_accepted_tokens`, `input_batch` 等状态。 控制流包含 5 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `GPUModelRunner.execute_model` method / `GPUModelRunner.execute_model` 方法
```python
    @torch.inference_mode()
    def execute_model(
        self,
        scheduler_output: "SchedulerOutput",
        intermediate_tensors: IntermediateTensors | None = None,
    ) -> ModelRunnerOutput | AsyncModelRunnerOutput | IntermediateTensors | None:
        if self.execute_model_state is not None:
            raise RuntimeError(
                "State error: sample_tokens() must be called "
                "after execute_model() returns None."
            )

        if self.routed_experts_initialized:
            self.routed_experts_capturer.clear_buffer()

        # If ngram_gpu is used, we need to copy the scheduler_output to avoid
        # the modification has influence on the scheduler_output in engine core process.
        # The replace is much faster than deepcopy.
        if (
            self.speculative_config is not None
            and self.speculative_config.use_ngram_gpu()
        ):
            num_scheduled_tokens_copy = scheduler_output.num_scheduled_tokens.copy()
            spec_decode_tokens_copy = (
                scheduler_output.scheduled_spec_decode_tokens.copy()
            )
            scheduler_output = replace(
                scheduler_output,
                num_scheduled_tokens=num_scheduled_tokens_copy,
                scheduled_spec_decode_tokens=spec_decode_tokens_copy,
            )

        if has_kv_transfer_group():
            kv_connector_metadata = scheduler_output.kv_connector_metadata
            assert kv_connector_metadata is not None
            get_kv_transfer_group().handle_preemptions(kv_connector_metadata)

        num_scheduled_tokens = scheduler_output.total_num_scheduled_tokens
        with (
            record_function_or_nullcontext("gpu_model_runner: preprocess"),
            self.synchronize_input_prep(),
        ):
            # Update persistent batch states.
            deferred_state_corrections_fn = self._update_states(scheduler_output)

            if has_ec_transfer() and not get_ec_transfer().is_consumer:
                with self.maybe_get_ec_connector_output(
                    scheduler_output,
                    encoder_cache=self.encoder_cache,
                ) as ec_connector_output:
    # ... omitted for brevity ...
                    model_output_broadcast_data, src=len(get_pp_group().ranks) - 1
                )
                assert broadcasted is not None
                logits = broadcasted["logits"]

        self.execute_model_state = ExecuteModelState(
            scheduler_output,
            logits,
            spec_decode_metadata,
            spec_decode_common_attn_metadata,
            hidden_states,
            sample_hidden_states,
            aux_hidden_states,
            ec_connector_output,
            cudagraph_stats,
            slot_mappings,
        )
        self.kv_connector_output = kv_connector_output

        # Now the batch has been launched we can wait for corrections from the
        # previous model forward without breaking async scheduling.
        if deferred_state_corrections_fn:
            deferred_state_corrections_fn()

        return None
```
**EN:** This method executes the main operation within `GPUModelRunner`. Key calls include `inference_mode`, `has_kv_transfer_group`, `len`, `ExecuteModelState`, `RuntimeError`, `clear_buffer`. It touches state such as `execute_model_state`, `kv_connector_output`, `calculate_kv_scales`, `num_accepted_tokens`. The control flow contains 26 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会执行主要操作，其作用域位于`GPUModelRunner`。 关键调用包括 `inference_mode`, `has_kv_transfer_group`, `len`, `ExecuteModelState`, `RuntimeError`, `clear_buffer`。 它会读写 `execute_model_state`, `kv_connector_output`, `calculate_kv_scales`, `num_accepted_tokens` 等状态。 控制流包含 26 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `GPUModelRunner.sample_tokens` method / `GPUModelRunner.sample_tokens` 方法
```python
    @torch.inference_mode
    def sample_tokens(
        self, grammar_output: "GrammarOutput | None"
    ) -> ModelRunnerOutput | AsyncModelRunnerOutput | IntermediateTensors:
        if self.execute_model_state is None:
            kv_connector_output = self.kv_connector_output
            self.kv_connector_output = None
            # receive sampled token ids from the last PP rank.
            if self.use_async_scheduling and not get_pp_group().is_last_rank:
                self._pp_receive_prev_sampled_token_ids_to_input_batch()
            if not kv_connector_output:
                return None  # type: ignore[return-value]

            # In case of PP with kv transfer, we need to pass through the
            # kv_connector_output
            if kv_connector_output.is_empty():
                return EMPTY_MODEL_RUNNER_OUTPUT

            output = copy(EMPTY_MODEL_RUNNER_OUTPUT)
            output.kv_connector_output = kv_connector_output
            return output

        # Unpack ephemeral state.
        (
            scheduler_output,
            logits,
            spec_decode_metadata,
            spec_decode_common_attn_metadata,
            hidden_states,
            sample_hidden_states,
            aux_hidden_states,
            ec_connector_output,
            cudagraph_stats,
            slot_mappings,
        ) = self.execute_model_state
        # Clear ephemeral state.
        self.execute_model_state = None

        # Apply structured output bitmasks if present.
        if grammar_output is not None:
            apply_grammar_bitmask(
                scheduler_output, grammar_output, self.input_batch, logits
            )

        with record_function_or_nullcontext("gpu_model_runner: sample"):
            sampler_output = self._sample(logits, spec_decode_metadata)

        self._update_states_after_model_execute(
            sampler_output.sampled_token_ids, scheduler_output
        )
    # ... omitted for brevity ...
                    slot_mapping=self.routed_experts_slot_mapping_device[
                        :total
                    ].clone(),
                )

            async_output = AsyncGPUModelRunnerOutput(
                model_runner_output=output,
                sampled_token_ids=sampler_output.sampled_token_ids,
                logprobs_tensors=sampler_output.logprobs_tensors,
                invalid_req_indices=invalid_req_indices,
                async_output_copy_stream=self._get_or_create_async_output_copy_stream(),
                vocab_size=self.input_batch.vocab_size,
                routed_experts=routed_experts_snapshot,
            )
        with record_function_or_nullcontext(
            "gpu_model_runner: set_async_sampled_token_ids"
        ):
            # Save ref of sampled_token_ids CPU tensor if the batch contains
            # any requests with sampling params that require output ids.
            self.input_batch.set_async_sampled_token_ids(
                async_output.sampled_token_ids_cpu,
                async_output.async_copy_ready_event,
            )

        return async_output
```
**EN:** This method samples outputs from model state within `GPUModelRunner`. Key calls include `_update_states_after_model_execute`, `is_empty`, `copy`, `apply_grammar_bitmask`, `record_function_or_nullcontext`, `_sample`. It touches state such as `execute_model_state`, `_draft_token_ids`, `_draft_probs`, `_draft_prob_req_ids`, `_draft_token_req_ids`, `valid_sampled_token_count_gpu`, `input_batch`, `kv_connector_output`. The control flow contains 21 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会从模型状态中采样输出，其作用域位于`GPUModelRunner`。 关键调用包括 `_update_states_after_model_execute`, `is_empty`, `copy`, `apply_grammar_bitmask`, `record_function_or_nullcontext`, `_sample`。 它会读写 `execute_model_state`, `_draft_token_ids`, `_draft_probs`, `_draft_prob_req_ids`, `_draft_token_req_ids`, `valid_sampled_token_count_gpu`, `input_batch`, `kv_connector_output` 等状态。 控制流包含 21 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `GPUModelRunner.load_model` method / `GPUModelRunner.load_model` 方法
```python
    @instrument(span_name="Loading (GPU)")
    def load_model(self, load_dummy_weights: bool = False) -> None:
        """
        Args:
            load_dummy_weights: load dummy weights instead of real weights.
        """
        logger.info_once(
            "Starting to load model %s...",
            self.model_config.model,
            scope="global",
        )

        if self.parallel_config.enable_eplb:
            self.eplb_state = EplbState(self.parallel_config, self.device)
            eplb_models = 0

        try:
            with DeviceMemoryProfiler() as m:
                time_before_load = time.perf_counter()
                if load_dummy_weights:
                    self.load_config.load_format = "dummy"
                model_loader = get_model_loader(self.load_config)
                self.model = model_loader.load_model(
                    vllm_config=self.vllm_config, model_config=self.model_config
                )
                if self.lora_config:
                    self.model = self.load_lora_model(
                        self.model, self.vllm_config, self.device
                    )
                if hasattr(self, "drafter"):
                    logger.info_once("Loading drafter model...")
                    if hasattr(self.drafter, "load_model"):
                        self.drafter.load_model(self.model)
                    if (
                        hasattr(self.drafter, "model")
                        and is_mixture_of_experts(self.drafter.model)
                        and self.parallel_config.enable_eplb
                    ):
                        assert not self.parallel_config.enable_elastic_ep, (
                            "Elastic EP is not supported with drafter model."
                        )
                        spec_config = self.vllm_config.speculative_config
                        assert spec_config is not None
                        assert spec_config.draft_model_config is not None
                        logger.info_once(
                            "EPLB is enabled for drafter model %s.",
                            spec_config.draft_model_config.model,
                        )
                        if self.eplb_state is None:
                            self.eplb_state = EplbState(
    # ... omitted for brevity ...
        assert cudagraph_mode is not None
        if (
            is_breakable_cudagraph_enabled()
            and cudagraph_mode != CUDAGraphMode.NONE
            and not self.parallel_config.use_ubatching
        ):
            self.model = BreakableCUDAGraphWrapper(self.model, self.vllm_config)
        elif (
            cudagraph_mode.has_full_cudagraphs()
            and not self.parallel_config.use_ubatching
        ):
            self.model = CUDAGraphWrapper(
                self.model, self.vllm_config, runtime_mode=CUDAGraphMode.FULL
            )
        elif self.parallel_config.use_ubatching:
            if cudagraph_mode.has_full_cudagraphs():
                self.model = UBatchWrapper(
                    self.model, self.vllm_config, CUDAGraphMode.FULL, self.device
                )
            else:
                self.model = UBatchWrapper(
                    self.model, self.vllm_config, CUDAGraphMode.NONE, self.device
                )

        get_offloader().post_init()
```
**EN:** This method loads external or cached state within `GPUModelRunner`. The docstring frames it as: Args: load_dummy_weights: load dummy weights instead of real weights. Key calls include `instrument`, `info_once`, `hasattr`, `post_init`, `EplbState`, `format_gib`. It touches state such as `is_multimodal_pruning_enabled`, `requires_sequential_video_encoding`, `eplb_state`, `model_memory_usage`, `model`, `load_config`, `_moe_model`. The control flow contains 22 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会加载外部或缓存状态，其作用域位于`GPUModelRunner`。 关键调用包括 `instrument`, `info_once`, `hasattr`, `post_init`, `EplbState`, `format_gib`。 它会读写 `is_multimodal_pruning_enabled`, `requires_sequential_video_encoding`, `eplb_state`, `model_memory_usage`, `model`, `load_config`, `_moe_model` 等状态。 控制流包含 22 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `GPUModelRunner._dummy_sampler_run` method / `GPUModelRunner._dummy_sampler_run` 方法
```python
    @torch.inference_mode()
    def _dummy_sampler_run(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        # The dummy hidden states may contain special values,
        # like `inf` or `nan`.
        # To avoid breaking the sampler, we use a random tensor here instead.

        mm_config = self.vllm_config.model_config.multimodal_config
        if mm_config and mm_config.mm_encoder_only:
            # MM Encoder only model no need to run sampler.
            return torch.tensor([])

        hidden_states = torch.rand_like(hidden_states)

        logits = self.model.compute_logits(hidden_states)
        num_reqs = logits.size(0)

        dummy_tensors = lambda v: torch.full((num_reqs,), v, device=self.device)

        dummy_metadata = SamplingMetadata(
            temperature=dummy_tensors(0.5),
            all_greedy=False,
            all_random=False,
            top_p=dummy_tensors(0.9),
            top_k=dummy_tensors(logits.size(1) - 1),
            generators={},
            max_num_logprobs=None,
            logprob_token_ids=None,
            no_penalties=True,
            prompt_token_ids=None,
            frequency_penalties=dummy_tensors(0.1),
            presence_penalties=dummy_tensors(0.1),
            repetition_penalties=dummy_tensors(0.1),
            output_token_ids=[[] for _ in range(num_reqs)],
            spec_token_ids=[[] for _ in range(num_reqs)],
            allowed_token_ids_mask=None,
            bad_words_token_ids={},
            logitsprocs=LogitsProcessors(),
        )
        try:
            sampler_output = self.sampler(
                logits=logits, sampling_metadata=dummy_metadata
            )
            # Also warm forward_native (taken when generators dict is non-empty),
            # but skip the extra call in 'processed_logits' / 'processed_logprobs'
            # modes — there TopKTopPSampler binds forward = forward_native at
            # init time, so the warmup call is redundant and only inflates peak
            # memory during profile_run.
    # ... omitted for brevity ...
            draft_probs = None
            if (
                self.speculative_config.rejection_sample_method == "standard"
                and self.speculative_config.draft_sample_method == "probabilistic"
            ):
                draft_probs = torch.rand(
                    num_tokens,
                    logits.shape[-1],
                    device=self.device,
                    dtype=torch.float32,
                )
                draft_probs = torch.softmax(draft_probs, dim=-1)
            logits = torch.randn(
                num_tokens + num_reqs,
                logits.shape[-1],
                device=self.device,
                dtype=logits.dtype,
            )
            self.rejection_sampler(
                dummy_spec_decode_metadata,
                draft_probs,
                logits,
                dummy_metadata,
            )
        return sampler_output
```
**EN:** This method implements `_dummy_sampler_run` within `GPUModelRunner`. Key calls include `inference_mode`, `rand_like`, `compute_logits`, `size`, `SamplingMetadata`, `tensor`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `_dummy_sampler_run`，其作用域位于`GPUModelRunner`。 关键调用包括 `inference_mode`, `rand_like`, `compute_logits`, `size`, `SamplingMetadata`, `tensor`。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `EncoderTimingStats` class / `EncoderTimingStats` 类
```python
@dataclass
class EncoderTimingStats:
    """Per-request timing statistics for encoder forward pass."""

    encoder_forward_secs: float = 0.0
    """Time spent in vision encoder forward pass (seconds)."""

    num_encoder_calls: int = 0
    """Number of times encoder was called for this request."""
```
**EN:** Uses `@dataclass` to package related state for `EncoderTimingStats`. Typical fields include `encoder_forward_secs`, `num_encoder_calls`.
**CN:** `EncoderTimingStats` 使用 `@dataclass` 打包相关状态。典型字段包括 `encoder_forward_secs`, `num_encoder_calls`。

### `EncoderTimingStats.to_dict` method / `EncoderTimingStats.to_dict` 方法
```python
    def to_dict(self) -> dict[str, float | int]:
        return {
            "encoder_forward_secs": self.encoder_forward_secs,
            "num_encoder_calls": self.num_encoder_calls,
        }
```
**EN:** This method converts data to another representation within `EncoderTimingStats`.
**CN:** 该方法会将数据转换为另一种表示，其作用域位于`EncoderTimingStats`。

## Key Concepts / 关键概念
- `AsyncGPUModelRunnerOutput`: central class or interface in this module. / `AsyncGPUModelRunnerOutput`：本模块中的核心类或接口。
- `_copy_pooler_output_to_cpu`: top-level helper or orchestration entry point. / `_copy_pooler_output_to_cpu`：顶层辅助函数或编排入口。
- `AsyncGPUPoolingModelRunnerOutput`: central class or interface in this module. / `AsyncGPUPoolingModelRunnerOutput`：本模块中的核心类或接口。
- `ExecuteModelState`: central class or interface in this module. / `ExecuteModelState`：本模块中的核心类或接口。
- `GPUModelRunner`: central class or interface in this module. / `GPUModelRunner`：本模块中的核心类或接口。
- `EncoderTimingStats`: central class or interface in this module. / `EncoderTimingStats`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `functools`, `gc`, `itertools`, `threading`, `time`, `collections`, `contextlib`, `copy`, `dataclasses`, `typing`
- External / 外部依赖: `numpy`, `torch`, `tqdm`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.compilation.breakable_cudagraph`, `vllm.compilation.counter`, `vllm.compilation.cuda_graph`, `vllm.compilation.monitor`, `vllm.config`, `vllm.config.cache`, `vllm.distributed.ec_transfer`, `vllm.distributed.eplb.eplb_state`, `vllm.distributed.kv_transfer`, `vllm.distributed.kv_transfer.kv_connector.utils`, `vllm.distributed.parallel_state`
