# tokenizer_control_mixin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/tokenizer_control_mixin.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements tokenizer control mixin logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 分词器 control mixin 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-14: Provide supporting module logic / 提供辅助模块逻辑
```python
import asyncio
import logging
import time
import uuid
from typing import (
    TYPE_CHECKING,
    Any,
    Dict,
    List,
    Optional,
    Tuple,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 16-16: Import runtime dependencies / 导入运行时依赖
```python
import fastapi
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 18-18: Import runtime dependencies / 导入运行时依赖
```python
from sglang.srt.managers.communicator import FanOutCommunicator
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 19-81: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.managers.io_struct import (
    AddExternalCorpusReqInput,
    AddExternalCorpusReqOutput,
    AttachHiCacheStorageReqInput,
    AttachHiCacheStorageReqOutput,
    CheckWeightsReqInput,
    CheckWeightsReqOutput,
    ClearHiCacheReqInput,
    ClearHiCacheReqOutput,
    CloseSessionReqInput,
    DestroyWeightsUpdateGroupReqInput,
    DestroyWeightsUpdateGroupReqOutput,
    DetachHiCacheStorageReqInput,
    DetachHiCacheStorageReqOutput,
    DumperControlReqInput,
    DumperControlReqOutput,
    ExpertDistributionReq,
    ExpertDistributionReqOutput,
    ExpertDistributionReqType,
    FlushCacheReqInput,
    FlushCacheReqOutput,
    GetInternalStateReq,
    GetInternalStateReqOutput,
    GetLoadsReqInput,
    GetLoadsReqOutput,
    GetWeightsByNameReqInput,
    GetWeightsByNameReqOutput,
    InitWeightsSendGroupForRemoteInstanceReqInput,
    InitWeightsSendGroupForRemoteInstanceReqOutput,
    InitWeightsUpdateGroupReqInput,
    InitWeightsUpdateGroupReqOutput,
    ListExternalCorporaReqInput,
    ListExternalCorporaReqOutput,
    LoadLoRAAdapterFromTensorsReqInput,
    LoadLoRAAdapterFromTensorsReqOutput,
    LoadLoRAAdapterReqInput,
    LoadLoRAAdapterReqOutput,
    LoRAUpdateOutput,
    OpenSessionReqInput,
    ProfileReq,
    ProfileReqOutput,
    ProfileReqType,
    ReleaseMemoryOccupationReqInput,
    ReleaseMemoryOccupationReqOutput,
    RemoveExternalCorpusReqInput,
    RemoveExternalCorpusReqOutput,
    ResumeMemoryOccupationReqInput,
    ResumeMemoryOccupationReqOutput,
    SendWeightsToRemoteInstanceReqInput,
    SendWeightsToRemoteInstanceReqOutput,
    SetInternalStateReq,
    SetInternalStateReqOutput,
    SlowDownReqInput,
    SlowDownReqOutput,
    UnloadLoRAAdapterReqInput,
    UnloadLoRAAdapterReqOutput,
    UpdateWeightsFromDistributedReqInput,
    UpdateWeightsFromDistributedReqOutput,
    UpdateWeightsFromIPCReqInput,
    UpdateWeightsFromIPCReqOutput,
    UpdateWeightsFromTensorReqInput,
    UpdateWeightsFromTensorReqOutput,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 82-84: Import runtime dependencies / 导入运行时依赖
```python
from sglang.srt.server_args import LoRARef, ServerArgs
from sglang.srt.utils import get_bool_env_var
from sglang.utils import TypeBasedDispatcher
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 86-87: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.managers.tokenizer_manager import TokenizerManager
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 89-124: Register helpers or handlers / 注册辅助逻辑或处理器
```python
logger = logging.getLogger(__name__)

# Declarative spec: (attr_name_prefix, response_type[, mode])
# Each entry creates self.{prefix}_communicator and registers
# response_type -> communicator.handle_recv in the dispatch table.
_COMMUNICATOR_SPECS = [
    ("init_weights_update_group", InitWeightsUpdateGroupReqOutput),
    ("destroy_weights_update_group", DestroyWeightsUpdateGroupReqOutput),
    ("update_weights_from_distributed", UpdateWeightsFromDistributedReqOutput),
    (
        "init_weights_send_group_for_remote_instance",
        InitWeightsSendGroupForRemoteInstanceReqOutput,
    ),
    ("send_weights_to_remote_instance", SendWeightsToRemoteInstanceReqOutput),
    ("update_weights_from_tensor", UpdateWeightsFromTensorReqOutput),
    ("update_weights_from_ipc", UpdateWeightsFromIPCReqOutput),
    ("get_weights_by_name", GetWeightsByNameReqOutput),
    ("release_memory_occupation", ReleaseMemoryOccupationReqOutput),
    ("resume_memory_occupation", ResumeMemoryOccupationReqOutput),
    ("check_weights", CheckWeightsReqOutput),
    ("slow_down", SlowDownReqOutput),
    ("flush_cache", FlushCacheReqOutput),
    ("add_external_corpus", AddExternalCorpusReqOutput),
    ("remove_external_corpus", RemoveExternalCorpusReqOutput),
    ("list_external_corpora", ListExternalCorporaReqOutput),
    ("clear_hicache_storage", ClearHiCacheReqOutput),
    ("attach_hicache_storage", AttachHiCacheStorageReqOutput),
    ("detach_hicache_storage", DetachHiCacheStorageReqOutput),
    ("profile", ProfileReqOutput),
    ("get_internal_state", GetInternalStateReqOutput),
    ("set_internal_state", SetInternalStateReqOutput),
    ("expert_distribution", ExpertDistributionReqOutput),
    ("update_lora_adapter", LoRAUpdateOutput),
    ("get_loads", GetLoadsReqOutput, "watching"),
    ("dumper_control", DumperControlReqOutput),
]
```
**EN:** This block wires callbacks, registries, or handlers into the surrounding runtime.
**CN:** 该代码块把回调、注册表或处理器接入周边运行时流程。

### Lines 127-131: Provide supporting module logic / 提供辅助模块逻辑
```python
class TokenizerControlMixin:
    """Mixin for TokenizerManager's control-plane operations (weights, cache, lora,
    profile, internal state, etc.) -- everything that talks to the scheduler via
    FanOutCommunicator, as opposed to data-plane inference requests multiplexed by rid.
    """
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 133-141: Implement init communicators / 实现init communicators
```python
    def init_communicators(self: TokenizerManager, server_args: ServerArgs):
        dispatch_pairs = []
        for spec in _COMMUNICATOR_SPECS:
            name, resp_type = spec[0], spec[1]
            mode = spec[2] if len(spec) > 2 else "queueing"
            comm = FanOutCommunicator(self.send_to_scheduler, server_args.dp_size, mode)
            setattr(self, f"{name}_communicator", comm)
            dispatch_pairs.append((resp_type, comm.handle_recv))
        self._result_dispatcher += TypeBasedDispatcher(dispatch_pairs)
```
**EN:** This block implements the method `init_communicators(server_args)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `init_communicators`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的方法 `init_communicators(server_args)`。它围绕 `init_communicators` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 143-178: Implement async add external corpus / 实现异步add external corpus
```python
    async def add_external_corpus(
        self: TokenizerManager, obj: AddExternalCorpusReqInput
    ) -> AddExternalCorpusReqOutput:
        self.auto_create_handle_loop()
        if self.server_args.speculative_algorithm != "NGRAM":
            return AddExternalCorpusReqOutput(
                success=False,
                message="Ngram speculative decoding is not enabled.",
            )
        truncated = False
        try:
            if not obj.corpus_id:
                import uuid

                obj.corpus_id = uuid.uuid4().hex
            if obj.file_path is not None:
                from sglang.srt.speculative.cpp_ngram.external_corpus import (
                    iter_external_corpus_chunks,
                )

                max_tokens = (
                    self.server_args.speculative_ngram_external_corpus_max_tokens
                )
                obj.token_chunks = list(
                    iter_external_corpus_chunks(
                        obj.file_path, self.tokenizer, max_tokens
                    )
                )
            elif obj.documents is not None:
                from sglang.srt.speculative.cpp_ngram.external_corpus import (
                    SEPARATOR_TOKEN,
                )

                max_tokens = (
                    self.server_args.speculative_ngram_external_corpus_max_tokens
                )
```
**EN:** This block implements the async method `add_external_corpus(obj)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `add_external_corpus`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `add_external_corpus(obj)`。它围绕 `add_external_corpus` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 179-214: Continue async add external corpus / 继续说明异步add external corpus
```python
                token_chunks = []
                total_tokens = 0
                has_prev = False
                for doc in obj.documents:
                    if not doc:
                        continue
                    token_ids = list(
                        self.tokenizer.encode(doc, add_special_tokens=False)
                    )
                    if not token_ids:
                        continue
                    if has_prev:
                        token_ids = [SEPARATOR_TOKEN] + token_ids
                    if total_tokens + len(token_ids) > max_tokens:
                        truncated = True
                        break
                    token_chunks.append(token_ids)
                    total_tokens += len(token_ids)
                    has_prev = True
                obj.token_chunks = token_chunks
            else:
                return AddExternalCorpusReqOutput(
                    success=False,
                    message="Either file_path or documents must be provided.",
                )
            obj.file_path = None
            obj.documents = None
            results = await self.add_external_corpus_communicator(obj)
            all_success, all_message = FanOutCommunicator.merge_results(results)
            if truncated and all_success:
                all_message += f" (truncated: exceeded {max_tokens} token limit)"
            return AddExternalCorpusReqOutput(
                success=all_success,
                corpus_id=results[0].corpus_id if all_success else "",
                message=all_message,
                loaded_token_count=results[0].loaded_token_count if all_success else 0,
```
**EN:** This block implements the async method `add_external_corpus(obj)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `add_external_corpus`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `add_external_corpus(obj)`。它围绕 `add_external_corpus` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 215-217: Continue async add external corpus / 继续说明异步add external corpus
```python
            )
        except Exception as e:
            return AddExternalCorpusReqOutput(success=False, message=str(e))
```
**EN:** This block implements the async method `add_external_corpus(obj)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `add_external_corpus`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `add_external_corpus(obj)`。它围绕 `add_external_corpus` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 219-232: Implement async remove external corpus / 实现异步remove external corpus
```python
    async def remove_external_corpus(
        self: TokenizerManager, corpus_id: str
    ) -> RemoveExternalCorpusReqOutput:
        self.auto_create_handle_loop()
        if self.server_args.speculative_algorithm != "NGRAM":
            return RemoveExternalCorpusReqOutput(
                success=False,
                message="Ngram speculative decoding is not enabled.",
            )
        results = await self.remove_external_corpus_communicator(
            RemoveExternalCorpusReqInput(corpus_id=corpus_id)
        )
        all_success, all_message = FanOutCommunicator.merge_results(results)
        return RemoveExternalCorpusReqOutput(success=all_success, message=all_message)
```
**EN:** This block implements the async method `remove_external_corpus(corpus_id)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `remove_external_corpus`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `remove_external_corpus(corpus_id)`。它围绕 `remove_external_corpus` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 234-253: Implement async list external corpora / 实现异步list external corpora
```python
    async def list_external_corpora(
        self: TokenizerManager,
    ) -> ListExternalCorporaReqOutput:
        self.auto_create_handle_loop()
        if self.server_args.speculative_algorithm != "NGRAM":
            return ListExternalCorporaReqOutput(
                success=False,
                message="Ngram speculative decoding is not enabled.",
            )
        results = await self.list_external_corpora_communicator(
            ListExternalCorporaReqInput()
        )
        all_success, all_message = FanOutCommunicator.merge_results(results)
        # Merge corpus token counts from all DP ranks (each rank loads the same set).
        corpus_token_counts = results[0].corpus_token_counts if all_success else {}
        return ListExternalCorporaReqOutput(
            success=all_success,
            corpus_token_counts=corpus_token_counts,
            message=all_message,
        )
```
**EN:** This block implements the async method `list_external_corpora()` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `list_external_corpora`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `list_external_corpora()`。它围绕 `list_external_corpora` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 255-261: Implement async flush cache / 实现异步flush 缓存
```python
    async def flush_cache(
        self: TokenizerManager, timeout_s: Optional[float] = None
    ) -> FlushCacheReqOutput:
        self.auto_create_handle_loop()
        return (
            await self.flush_cache_communicator(FlushCacheReqInput(timeout_s=timeout_s))
        )[0]
```
**EN:** This block implements the async method `flush_cache(timeout_s)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `flush_cache`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `flush_cache(timeout_s)`。它围绕 `flush_cache` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 263-269: Implement async clear hicache storage / 实现异步clear hicache storage
```python
    async def clear_hicache_storage(self: TokenizerManager) -> ClearHiCacheReqOutput:
        """Clear the hierarchical cache storage."""
        self.auto_create_handle_loop()
        # Delegate to the scheduler to handle HiCacheStorage clearing
        return (await self.clear_hicache_storage_communicator(ClearHiCacheReqInput()))[
            0
        ]
```
**EN:** This block implements the async method `clear_hicache_storage()` on `TokenizerControlMixin`. It focuses on Clear the hierarchical cache storage., so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `clear_hicache_storage()`。它围绕 `clear_hicache_storage` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 271-305: Implement async attach hicache storage / 实现异步attach hicache storage
```python
    async def attach_hicache_storage(
        self: TokenizerManager,
        hicache_storage_backend: str,
        hicache_storage_backend_extra_config_json: Optional[str] = None,
        hicache_storage_prefetch_policy: Optional[str] = None,
        hicache_write_policy: Optional[str] = None,
    ) -> AttachHiCacheStorageReqOutput:
        """Attach (enable) HiCache storage backend at runtime."""
        self.auto_create_handle_loop()
        results = await self.attach_hicache_storage_communicator(
            AttachHiCacheStorageReqInput(
                hicache_storage_backend=hicache_storage_backend,
                hicache_storage_backend_extra_config_json=hicache_storage_backend_extra_config_json,
                hicache_storage_prefetch_policy=hicache_storage_prefetch_policy,
                hicache_write_policy=hicache_write_policy,
            )
        )

        all_success, all_message = FanOutCommunicator.merge_results(results)
        out = AttachHiCacheStorageReqOutput(success=all_success, message=all_message)
        # TODO: partial rollback if failed
        if all_success:
            # Keep tokenizer side server_info consistent with scheduler side.
            self.server_args.hicache_storage_backend = hicache_storage_backend
            if hicache_storage_backend_extra_config_json is not None:
                self.server_args.hicache_storage_backend_extra_config = (
                    hicache_storage_backend_extra_config_json
                )
            if hicache_storage_prefetch_policy is not None:
                self.server_args.hicache_storage_prefetch_policy = (
                    hicache_storage_prefetch_policy
                )
            if hicache_write_policy is not None:
                self.server_args.hicache_write_policy = hicache_write_policy
        return out
```
**EN:** This block implements the async method `attach_hicache_storage(hicache_storage_backend, hicache_storage_backend_extra_config_json, hicache_storage_prefetch_policy, hicache_write_policy)` on `TokenizerControlMixin`. It focuses on Attach (enable) HiCache storage backend at runtime., so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `attach_hicache_storage(hicache_storage_backend, hicache_storage_backend_extra_config_json, hicache_storage_prefetch_policy, hicache_write_policy)`。它围绕 `attach_hicache_storage` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 307-322: Implement async detach hicache storage / 实现异步detach hicache storage
```python
    async def detach_hicache_storage(
        self: TokenizerManager,
    ) -> DetachHiCacheStorageReqOutput:
        """Detach (disable) HiCache storage backend at runtime."""
        self.auto_create_handle_loop()
        results = await self.detach_hicache_storage_communicator(
            DetachHiCacheStorageReqInput()
        )

        all_success, all_message = FanOutCommunicator.merge_results(results)
        out = DetachHiCacheStorageReqOutput(success=all_success, message=all_message)
        # TODO: partial rollback if failed
        if all_success:
            self.server_args.hicache_storage_backend = None
            self.server_args.hicache_storage_backend_extra_config = None
        return out
```
**EN:** This block implements the async method `detach_hicache_storage()` on `TokenizerControlMixin`. It focuses on Detach (disable) HiCache storage backend at runtime., so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `detach_hicache_storage()`。它围绕 `detach_hicache_storage` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 324-358: Implement async start profile / 实现异步start 剖析
```python
    async def start_profile(
        self: TokenizerManager,
        output_dir: Optional[str] = None,
        start_step: Optional[int] = None,
        num_steps: Optional[int] = None,
        activities: Optional[List[str]] = None,
        with_stack: Optional[bool] = None,
        record_shapes: Optional[bool] = None,
        profile_by_stage: bool = False,
        merge_profiles: bool = False,
        profile_prefix: Optional[str] = None,
        profile_stages: Optional[List[str]] = None,
    ):
        self.auto_create_handle_loop()
        env_with_stack: bool = get_bool_env_var("SGLANG_PROFILE_WITH_STACK", "true")
        with_stack = False if with_stack is False or env_with_stack is False else True
        env_record_shapes: bool = get_bool_env_var(
            "SGLANG_PROFILE_RECORD_SHAPES", "true"
        )
        record_shapes = (record_shapes is not False) and env_record_shapes
        req = ProfileReq(
            type=ProfileReqType.START_PROFILE,
            output_dir=output_dir,
            start_step=start_step,
            num_steps=num_steps,
            activities=activities,
            with_stack=with_stack,
            record_shapes=record_shapes,
            profile_by_stage=profile_by_stage,
            profile_id=str(time.time()),
            merge_profiles=merge_profiles,
            profile_prefix=profile_prefix,
            profile_stages=profile_stages,
        )
        return await self._execute_profile(req)
```
**EN:** This block implements the async method `start_profile(output_dir, start_step, num_steps, activities, with_stack, ...)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `start_profile`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `start_profile(output_dir, start_step, num_steps, activities, with_stack, ...)`。它围绕 `start_profile` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 360-363: Implement async stop profile / 实现异步stop 剖析
```python
    async def stop_profile(self: TokenizerManager):
        self.auto_create_handle_loop()
        req = ProfileReq(type=ProfileReqType.STOP_PROFILE)
        return await self._execute_profile(req)
```
**EN:** This block implements the async method `stop_profile()` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `stop_profile`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `stop_profile()`。它围绕 `stop_profile` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 365-369: Implement async execute profile / 实现异步execute 剖析
```python
    async def _execute_profile(self: TokenizerManager, req: ProfileReq):
        result = (await self.profile_communicator(req))[0]
        if not result.success:
            raise RuntimeError(result.message)
        return result
```
**EN:** This block implements the async method `_execute_profile(req)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `_execute_profile`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `_execute_profile(req)`。它围绕 `_execute_profile` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 371-374: Implement async start expert distribution record / 实现异步start expert distribution record
```python
    async def start_expert_distribution_record(self: TokenizerManager):
        self.auto_create_handle_loop()
        req = ExpertDistributionReq(action=ExpertDistributionReqType.START_RECORD)
        await self.expert_distribution_communicator(req)
```
**EN:** This block implements the async method `start_expert_distribution_record()` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `start_expert_distribution_record`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `start_expert_distribution_record()`。它围绕 `start_expert_distribution_record` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 376-379: Implement async stop expert distribution record / 实现异步stop expert distribution record
```python
    async def stop_expert_distribution_record(self: TokenizerManager):
        self.auto_create_handle_loop()
        req = ExpertDistributionReq(action=ExpertDistributionReqType.STOP_RECORD)
        await self.expert_distribution_communicator(req)
```
**EN:** This block implements the async method `stop_expert_distribution_record()` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `stop_expert_distribution_record`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `stop_expert_distribution_record()`。它围绕 `stop_expert_distribution_record` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 381-384: Implement async dump expert distribution record / 实现异步dump expert distribution record
```python
    async def dump_expert_distribution_record(self: TokenizerManager):
        self.auto_create_handle_loop()
        req = ExpertDistributionReq(action=ExpertDistributionReqType.DUMP_RECORD)
        await self.expert_distribution_communicator(req)
```
**EN:** This block implements the async method `dump_expert_distribution_record()` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `dump_expert_distribution_record`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `dump_expert_distribution_record()`。它围绕 `dump_expert_distribution_record` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 386-397: Implement async init weights update group / 实现异步init weights update group
```python
    async def init_weights_update_group(
        self: TokenizerManager,
        obj: InitWeightsUpdateGroupReqInput,
        request: Optional[fastapi.Request] = None,
    ) -> Tuple[bool, str]:
        self.auto_create_handle_loop()
        assert (
            self.server_args.dp_size == 1 or self.server_args.enable_dp_attention
        ), "dp_size must be 1 or dp attention must be enabled for update weights from distributed"

        results = await self.init_weights_update_group_communicator(obj)
        return FanOutCommunicator.merge_results(results)
```
**EN:** This block implements the async method `init_weights_update_group(obj, request)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `init_weights_update_group`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `init_weights_update_group(obj, request)`。它围绕 `init_weights_update_group` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 399-410: Implement async destroy weights update group / 实现异步destroy weights update group
```python
    async def destroy_weights_update_group(
        self: TokenizerManager,
        obj: DestroyWeightsUpdateGroupReqInput,
        request: Optional[fastapi.Request] = None,
    ) -> Tuple[bool, str]:
        self.auto_create_handle_loop()
        assert (
            self.server_args.dp_size == 1 or self.server_args.enable_dp_attention
        ), "dp_size must be 1 or dp attention must be enabled for destroy parameter update group"

        results = await self.destroy_weights_update_group_communicator(obj)
        return FanOutCommunicator.merge_results(results)
```
**EN:** This block implements the async method `destroy_weights_update_group(obj, request)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `destroy_weights_update_group`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `destroy_weights_update_group(obj, request)`。它围绕 `destroy_weights_update_group` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 412-440: Implement async update weights from distributed / 实现异步update weights from distributed
```python
    async def update_weights_from_distributed(
        self: TokenizerManager,
        obj: UpdateWeightsFromDistributedReqInput,
        request: Optional[fastapi.Request] = None,
    ) -> Tuple[bool, str]:
        self.auto_create_handle_loop()
        assert (
            self.server_args.dp_size == 1 or self.server_args.enable_dp_attention
        ), "dp_size must be 1 or dp attention must be enabled for update weights from distributed"

        if obj.abort_all_requests:
            self.abort_request(abort_all=True)

        # Hold is_pause_cond while updating to prevent unpause from racing.
        async with self.is_pause_cond:
            is_paused = self.is_pause
            if is_paused:
                results = await self.update_weights_from_distributed_communicator(obj)

        if not is_paused:
            async with self.model_update_lock.writer_lock:
                results = await self.update_weights_from_distributed_communicator(obj)

        success, message = FanOutCommunicator.merge_results(results)
        if success and obj.weight_version is not None:
            self._update_weight_version_if_provided(obj.weight_version)
            message += f" Weight version updated to {obj.weight_version}."

        return success, message
```
**EN:** This block implements the async method `update_weights_from_distributed(obj, request)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `update_weights_from_distributed`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `update_weights_from_distributed(obj, request)`。它围绕 `update_weights_from_distributed` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 442-455: Implement async init weights send group for remote instance / 实现异步init weights send group for remote instance
```python
    async def init_weights_send_group_for_remote_instance(
        self: TokenizerManager,
        obj: InitWeightsSendGroupForRemoteInstanceReqInput,
        request: Optional[fastapi.Request] = None,
    ) -> Tuple[bool, str]:
        self.auto_create_handle_loop()
        # TODO: support DP
        assert (
            self.server_args.dp_size == 1
        ), "dp_size must be 1 for init_weights_send_group_for_remote_instance"
        result = (
            await self.init_weights_send_group_for_remote_instance_communicator(obj)
        )[0]
        return result.success, result.message
```
**EN:** This block implements the async method `init_weights_send_group_for_remote_instance(obj, request)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `init_weights_send_group_for_remote_instance`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `init_weights_send_group_for_remote_instance(obj, request)`。它围绕 `init_weights_send_group_for_remote_instance` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 457-468: Implement async send weights to remote instance / 实现异步send weights to remote instance
```python
    async def send_weights_to_remote_instance(
        self: TokenizerManager,
        obj: SendWeightsToRemoteInstanceReqInput,
        request: Optional[fastapi.Request] = None,
    ) -> Tuple[bool, str]:
        self.auto_create_handle_loop()
        # TODO: support DP
        assert (
            self.server_args.dp_size == 1
        ), "dp_size must be 1 for send_weights_to_remote_instance"
        result = (await self.send_weights_to_remote_instance_communicator(obj))[0]
        return result.success, result.message
```
**EN:** This block implements the async method `send_weights_to_remote_instance(obj, request)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `send_weights_to_remote_instance`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `send_weights_to_remote_instance(obj, request)`。它围绕 `send_weights_to_remote_instance` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 470-497: Implement async update weights from tensor / 实现异步update weights from 张量
```python
    async def update_weights_from_tensor(
        self: TokenizerManager,
        obj: UpdateWeightsFromTensorReqInput,
        request: Optional[fastapi.Request] = None,
    ) -> Tuple[bool, str]:
        self.auto_create_handle_loop()
        assert (
            self.server_args.dp_size == 1 or self.server_args.enable_dp_attention
        ), "dp_size must be 1 or dp attention must be enabled for update weights from tensor"

        if obj.abort_all_requests:
            self.abort_request(abort_all=True)

        async with self.is_pause_cond:
            is_paused = self.is_pause
            if is_paused:
                results = await self.update_weights_from_tensor_communicator(obj)

        if not is_paused:
            async with self.model_update_lock.writer_lock:
                results = await self.update_weights_from_tensor_communicator(obj)

        success, message = FanOutCommunicator.merge_results(results)
        if success and obj.weight_version is not None:
            self._update_weight_version_if_provided(obj.weight_version)
            message += f" Weight version updated to {obj.weight_version}."

        return success, message
```
**EN:** This block implements the async method `update_weights_from_tensor(obj, request)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `update_weights_from_tensor`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `update_weights_from_tensor(obj, request)`。它围绕 `update_weights_from_tensor` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 499-532: Implement async update weights from ipc / 实现异步update weights from ipc
```python
    async def update_weights_from_ipc(
        self: TokenizerManager,
        obj: UpdateWeightsFromIPCReqInput,
        request: Optional[fastapi.Request] = None,
    ) -> Tuple[bool, str]:
        """Update weights via IPC for checkpoint-engine integration."""
        self.auto_create_handle_loop()
        try:
            # For now, we only support single data parallel instance
            assert (
                self.server_args.dp_size == 1 or self.server_args.enable_dp_attention
            ), "dp_size must be 1 or dp attention must be enabled for update weights from IPC"
            logger.info("Starting IPC weight update")

            async with self.is_pause_cond:
                is_paused = self.is_pause
                if is_paused:
                    result = (await self.update_weights_from_ipc_communicator(obj))[0]
                    success, message = result.success, result.message

            if not is_paused:
                async with self.model_update_lock.writer_lock:
                    result = (await self.update_weights_from_ipc_communicator(obj))[0]
                    success, message = result.success, result.message
        except Exception as e:
            error_msg = f"IPC weight update failed: {str(e)}"
            logger.error(error_msg)
            success, message = False, error_msg

        if success and obj.weight_version is not None:
            self._update_weight_version_if_provided(obj.weight_version)
            message += f" Weight version updated to {obj.weight_version}."

        return success, message
```
**EN:** This block implements the async method `update_weights_from_ipc(obj, request)` on `TokenizerControlMixin`. It focuses on Update weights via IPC for checkpoint-engine integration., so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `update_weights_from_ipc(obj, request)`。它围绕 `update_weights_from_ipc` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 534-552: Implement async unload lora adapter locked / 实现异步unload lora adapter locked
```python
    async def _unload_lora_adapter_locked(
        self: TokenizerManager,
        obj: UnloadLoRAAdapterReqInput,
    ) -> UnloadLoRAAdapterReqOutput:
        assert (
            self.lora_update_lock.locked()
        ), "self.lora_update_lock must be locked in order for self._unload_lora_adapter_locked() to be called"

        # Unregister the LoRA adapter from the registry to stop new requests for this adapter
        # from being started.
        lora_id = await self.lora_registry.unregister(obj.lora_name)
        obj.lora_id = lora_id

        # Initiate the actual unloading operation at the backend processes only after all
        # ongoing requests using this LoRA adapter are finished.
        await self.lora_registry.wait_for_unload(lora_id)
        result = (await self.update_lora_adapter_communicator(obj))[0]

        return result
```
**EN:** This block implements the async method `_unload_lora_adapter_locked(obj)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `_unload_lora_adapter_locked`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `_unload_lora_adapter_locked(obj)`。它围绕 `_unload_lora_adapter_locked` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 554-589: Implement async load lora adapter / 实现异步load lora adapter
```python
    async def load_lora_adapter(
        self: TokenizerManager,
        obj: LoadLoRAAdapterReqInput,
        _: Optional[fastapi.Request] = None,
    ) -> LoadLoRAAdapterReqOutput:
        self.auto_create_handle_loop()

        try:
            if not self.server_args.enable_lora:
                raise ValueError(
                    "LoRA is not enabled. Please set `--enable-lora` to enable LoRA."
                )

            # TODO (lifuhuang): Remove this after we verify that dynamic lora loading works
            # with dp_size > 1.
            assert (
                self.server_args.dp_size == 1
            ), "dp_size must be 1 for dynamic lora loading"
            logger.info(
                "Start load Lora adapter. Lora name=%s, path=%s",
                obj.lora_name,
                obj.lora_path,
            )

            async with self.lora_update_lock:
                # Generate new uniquely identifiable LoRARef object.
                new_adapter = LoRARef(
                    lora_name=obj.lora_name,
                    lora_path=obj.lora_path,
                    pinned=obj.pinned,
                )

                # Trigger the actual loading operation at the backend processes.
                obj.lora_id = new_adapter.lora_id
                result = (await self.update_lora_adapter_communicator(obj))[0]
```
**EN:** This block implements the async method `load_lora_adapter(obj, _)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `load_lora_adapter`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `load_lora_adapter(obj, _)`。它围绕 `load_lora_adapter` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 590-625: Continue async load lora adapter / 继续说明异步load lora adapter
```python
                # Register the LoRA adapter only after loading is successful.
                if result.success:
                    await self.lora_registry.register(new_adapter)
                    self.lora_ref_cache[obj.lora_name] = new_adapter

                if self.server_args.max_loaded_loras is not None:
                    while (
                        self.lora_registry.num_registered_loras
                        > self.server_args.max_loaded_loras
                    ):
                        lru_lora_name = await self.lora_registry.lru_lora_name(
                            exclude_pinned=True
                        )
                        if lru_lora_name is None:
                            raise ValueError(
                                "Didn't find any LoRA adapters when trying to evict LRU LoRA adapter. "
                                f"LoRA registry is: {self.lora_registry._registry}"
                            )

                        logger.info(
                            f"Unloading least recently used LoRA adapter '{lru_lora_name}' "
                            f"(current number of adapters: {self.lora_registry.num_registered_loras}, "
                            f"max allowed: {self.server_args.max_loaded_loras})"
                        )

                        unload_result = await self._unload_lora_adapter_locked(
                            UnloadLoRAAdapterReqInput(lora_name=lru_lora_name)
                        )
                        if not unload_result.success:
                            raise ValueError(
                                f"Error while unloading LRU LoRA adapter '{lru_lora_name}': "
                                f"{unload_result.error_message}"
                            )
                        del result.loaded_adapters[lru_lora_name]

                return result
```
**EN:** This block implements the async method `load_lora_adapter(obj, _)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `load_lora_adapter`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `load_lora_adapter(obj, _)`。它围绕 `load_lora_adapter` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 626-630: Continue async load lora adapter / 继续说明异步load lora adapter
```python
        except ValueError as e:
            return LoadLoRAAdapterReqOutput(
                success=False,
                error_message=str(e),
            )
```
**EN:** This block implements the async method `load_lora_adapter(obj, _)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `load_lora_adapter`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `load_lora_adapter(obj, _)`。它围绕 `load_lora_adapter` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 632-667: Implement async load lora adapter from tensors / 实现异步load lora adapter from tensors
```python
    async def load_lora_adapter_from_tensors(
        self: TokenizerManager,
        obj: LoadLoRAAdapterFromTensorsReqInput,
        _: Optional[fastapi.Request] = None,
    ) -> LoadLoRAAdapterFromTensorsReqOutput:
        self.auto_create_handle_loop()

        try:
            if not self.server_args.enable_lora:
                raise ValueError(
                    "LoRA is not enabled. Please set `--enable-lora` to enable LoRA."
                )

            assert (
                self.server_args.dp_size == 1
            ), "dp_size must be 1 for dynamic lora loading"
            logger.info(
                "Start load Lora adapter from tensors. Lora name=%s",
                obj.lora_name,
            )

            async with self.lora_update_lock:
                new_adapter = LoRARef(
                    lora_name=obj.lora_name,
                    lora_path="__tensor__",
                    pinned=obj.pinned,
                )
                obj.lora_id = new_adapter.lora_id
                result = (await self.update_lora_adapter_communicator(obj))[0]

                if result.success:
                    await self.lora_registry.register(new_adapter)
                    self.lora_ref_cache[obj.lora_name] = new_adapter
                if self.server_args.max_loaded_loras is not None:
                    while (
                        self.lora_registry.num_registered_loras
```
**EN:** This block implements the async method `load_lora_adapter_from_tensors(obj, _)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `load_lora_adapter_from_tensors`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `load_lora_adapter_from_tensors(obj, _)`。它围绕 `load_lora_adapter_from_tensors` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 668-700: Continue async load lora adapter from tensors / 继续说明异步load lora adapter from tensors
```python
                        > self.server_args.max_loaded_loras
                    ):
                        lru_lora_name = await self.lora_registry.lru_lora_name(
                            exclude_pinned=True
                        )
                        if lru_lora_name is None:
                            raise ValueError(
                                "Didn't find any LoRA adapters when trying to evict LRU LoRA adapter. "
                                f"LoRA registry is: {self.lora_registry._registry}"
                            )

                        logger.info(
                            f"Unloading least recently used LoRA adapter '{lru_lora_name}' "
                            f"(current number of adapters: {self.lora_registry.num_registered_loras}, "
                            f"max allowed: {self.server_args.max_loaded_loras})"
                        )

                        unload_result = await self._unload_lora_adapter_locked(
                            UnloadLoRAAdapterReqInput(lora_name=lru_lora_name)
                        )
                        if not unload_result.success:
                            raise ValueError(
                                f"Error while unloading LRU LoRA adapter '{lru_lora_name}': "
                                f"{unload_result.error_message}"
                            )
                        del result.loaded_adapters[lru_lora_name]

                return result
        except ValueError as e:
            return LoadLoRAAdapterFromTensorsReqOutput(
                success=False,
                error_message=str(e),
            )
```
**EN:** This block implements the async method `load_lora_adapter_from_tensors(obj, _)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `load_lora_adapter_from_tensors`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `load_lora_adapter_from_tensors(obj, _)`。它围绕 `load_lora_adapter_from_tensors` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 702-732: Implement async unload lora adapter / 实现异步unload lora adapter
```python
    async def unload_lora_adapter(
        self: TokenizerManager,
        obj: UnloadLoRAAdapterReqInput,
        _: Optional[fastapi.Request] = None,
    ) -> UnloadLoRAAdapterReqOutput:
        self.auto_create_handle_loop()

        try:
            if not self.server_args.enable_lora:
                raise ValueError(
                    "LoRA is not enabled. Please set `--enable-lora` to enable LoRA."
                )

            assert (
                obj.lora_name is not None
            ), "lora_name must be provided to unload LoRA adapter"

            # TODO (lifuhuang): Remove this after we verify that dynamic lora loading works
            # with dp_size > 1.
            assert (
                self.server_args.dp_size == 1
            ), "dp_size must be 1 for dynamic lora loading"
            logger.info(
                "Start unload Lora adapter. Lora name=%s",
                obj.lora_name,
            )

            async with self.lora_update_lock:
                return await self._unload_lora_adapter_locked(obj)
        except ValueError as e:
            return UnloadLoRAAdapterReqOutput(success=False, error_message=str(e))
```
**EN:** This block implements the async method `unload_lora_adapter(obj, _)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `unload_lora_adapter`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `unload_lora_adapter(obj, _)`。它围绕 `unload_lora_adapter` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 734-745: Implement async get weights by name / 实现异步get weights by name
```python
    async def get_weights_by_name(
        self: TokenizerManager,
        obj: GetWeightsByNameReqInput,
        request: Optional[fastapi.Request] = None,
    ):
        self.auto_create_handle_loop()
        results = await self.get_weights_by_name_communicator(obj)
        all_parameters = [r.parameter for r in results]
        if self.server_args.dp_size == 1:
            return all_parameters[0]
        else:
            return all_parameters
```
**EN:** This block implements the async method `get_weights_by_name(obj, request)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `get_weights_by_name`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `get_weights_by_name(obj, request)`。它围绕 `get_weights_by_name` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 747-753: Implement async release memory occupation / 实现异步release memory occupation
```python
    async def release_memory_occupation(
        self: TokenizerManager,
        obj: ReleaseMemoryOccupationReqInput,
        request: Optional[fastapi.Request] = None,
    ):
        self.auto_create_handle_loop()
        await self.release_memory_occupation_communicator(obj)
```
**EN:** This block implements the async method `release_memory_occupation(obj, request)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `release_memory_occupation`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `release_memory_occupation(obj, request)`。它围绕 `release_memory_occupation` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 755-761: Implement async resume memory occupation / 实现异步resume memory occupation
```python
    async def resume_memory_occupation(
        self: TokenizerManager,
        obj: ResumeMemoryOccupationReqInput,
        request: Optional[fastapi.Request] = None,
    ):
        self.auto_create_handle_loop()
        await self.resume_memory_occupation_communicator(obj)
```
**EN:** This block implements the async method `resume_memory_occupation(obj, request)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `resume_memory_occupation`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `resume_memory_occupation(obj, request)`。它围绕 `resume_memory_occupation` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 763-774: Implement async check weights / 实现异步检查 weights
```python
    async def check_weights(
        self: TokenizerManager,
        obj: CheckWeightsReqInput,
        request: Optional[fastapi.Request] = None,
    ) -> Tuple[bool, str, Optional[List[Dict]]]:
        self.auto_create_handle_loop()
        results = await self.check_weights_communicator(obj)
        success, message = FanOutCommunicator.merge_results(results)
        ranks: Optional[List[Dict]] = None
        if any(r.payload is not None for r in results):
            ranks = [r.payload for r in results]
        return success, message, ranks
```
**EN:** This block implements the async method `check_weights(obj, request)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `check_weights`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `check_weights(obj, request)`。它围绕 `check_weights` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 776-782: Implement async slow down / 实现异步slow down
```python
    async def slow_down(
        self: TokenizerManager,
        obj: SlowDownReqInput,
        request: Optional[fastapi.Request] = None,
    ):
        self.auto_create_handle_loop()
        await self.slow_down_communicator(obj)
```
**EN:** This block implements the async method `slow_down(obj, request)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `slow_down`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `slow_down(obj, request)`。它围绕 `slow_down` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 784-791: Implement async get internal state / 实现异步get internal 状态
```python
    async def get_internal_state(self: TokenizerManager) -> List[Dict[Any, Any]]:
        self.auto_create_handle_loop()
        req = GetInternalStateReq()
        responses: List[GetInternalStateReqOutput] = (
            await self.get_internal_state_communicator(req)
        )
        # Many DP ranks
        return [res.internal_state for res in responses]
```
**EN:** This block implements the async method `get_internal_state()` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `get_internal_state`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `get_internal_state()`。它围绕 `get_internal_state` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 793-800: Implement async set internal state / 实现异步set internal 状态
```python
    async def set_internal_state(
        self: TokenizerManager, obj: SetInternalStateReq
    ) -> List[bool]:
        self.auto_create_handle_loop()
        responses: List[SetInternalStateReqOutput] = (
            await self.set_internal_state_communicator(obj)
        )
        return [res.updated for res in responses]
```
**EN:** This block implements the async method `set_internal_state(obj)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `set_internal_state`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `set_internal_state(obj)`。它围绕 `set_internal_state` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 802-806: Implement async dumper control / 实现异步dumper control
```python
    async def dumper_control(
        self: TokenizerManager, obj: DumperControlReqInput
    ) -> List[DumperControlReqOutput]:
        self.auto_create_handle_loop()
        return await self.dumper_control_communicator(obj)
```
**EN:** This block implements the async method `dumper_control(obj)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `dumper_control`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `dumper_control(obj)`。它围绕 `dumper_control` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 808-848: Implement async get loads / 实现异步get loads
```python
    async def get_loads(
        self: TokenizerManager,
        include: Optional[List[str]] = None,
        dp_rank: Optional[int] = None,
    ) -> List[GetLoadsReqOutput]:
        """
        Get comprehensive load metrics for /v1/loads endpoint.

        Args:
            include: List of sections to include. Options: core, memory, spec, lora, disagg, queues, all
            dp_rank: Optional filter for specific DP rank

        Returns:
            List of GetLoadsReqOutput, one per scheduler (filtered by dp_rank if specified)
        """
        self.auto_create_handle_loop()
        # Always request all sections from scheduler — watching mode shares
        # results across concurrent callers, so we fetch full data and filter here.
        req = GetLoadsReqInput(include=["all"], dp_rank=None)
        results = await self.get_loads_communicator(req)

        # Filter by dp_rank if specified
        if dp_rank is not None:
            results = [r for r in results if r.dp_rank == dp_rank]

        # Filter optional sections client-side (scheduler always returns all)
        if include and "all" not in include:
            include_set = set(include)
            _section_attrs = {
                "memory": "memory",
                "spec": "speculative",
                "lora": "lora",
                "disagg": "disaggregation",
                "queues": "queues",
            }
            for r in results:
                for key, attr in _section_attrs.items():
                    if key not in include_set:
                        setattr(r, attr, None)

        return results
```
**EN:** This block implements the async method `get_loads(include, dp_rank)` on `TokenizerControlMixin`. It focuses on Get comprehensive load metrics for /v1/loads endpoint., so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `get_loads(include, dp_rank)`。它围绕 `get_loads` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 850-875: Implement async open session / 实现异步open 会话
```python
    async def open_session(
        self: TokenizerManager,
        obj: OpenSessionReqInput,
        request: Optional[fastapi.Request] = None,
    ):
        self.auto_create_handle_loop()
        if obj.streaming:
            if not self.server_args.enable_streaming_session:
                raise ValueError(
                    "Streaming sessions are disabled. "
                    "Please relaunch with --enable-streaming-session."
                )

        if obj.session_id is None:
            obj.session_id = uuid.uuid4().hex
        elif obj.session_id in self.session_futures:
            return None

        future = asyncio.Future()
        self.session_futures[obj.session_id] = future
        self.send_to_scheduler.send_pyobj(obj)

        try:
            return await future
        finally:
            self.session_futures.pop(obj.session_id, None)
```
**EN:** This block implements the async method `open_session(obj, request)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `open_session`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `open_session(obj, request)`。它围绕 `open_session` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 877-882: Implement async close session / 实现异步close 会话
```python
    async def close_session(
        self: TokenizerManager,
        obj: CloseSessionReqInput,
        request: Optional[fastapi.Request] = None,
    ):
        await self.send_to_scheduler.send_pyobj(obj)
```
**EN:** This block implements the async method `close_session(obj, request)` on `TokenizerControlMixin`. It focuses on handling the tokenizer control mixin responsibilities represented by `close_session`, so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的异步方法 `close_session(obj, request)`。它围绕 `close_session` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

### Lines 884-889: Implement update weight version if provided / 实现update weight version if provided
```python
    def _update_weight_version_if_provided(
        self: TokenizerManager, weight_version: Optional[str]
    ) -> None:
        """Update weight version if provided."""
        if weight_version is not None:
            self.server_args.weight_version = weight_version
```
**EN:** This block implements the method `_update_weight_version_if_provided(weight_version)` on `TokenizerControlMixin`. It focuses on Update weight version if provided., so the class can advance the tokenizer control mixin workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerControlMixin` 上的方法 `_update_weight_version_if_provided(weight_version)`。它围绕 `_update_weight_version_if_provided` 所承担的 分词器 control mixin 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: TokenizerControlMixin
- **Domain focus / 领域焦点**: tokenizer control mixin / 分词器 control mixin
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: asyncio, logging, time, typing, uuid
- **Third-party / 第三方库**: __future__, fastapi
- **Local Modules / 本地模块**: sglang.srt.managers.communicator, sglang.srt.managers.io_struct, sglang.srt.managers.tokenizer_manager, sglang.srt.server_args, sglang.srt.speculative.cpp_ngram.external_corpus, sglang.srt.utils, sglang.utils
