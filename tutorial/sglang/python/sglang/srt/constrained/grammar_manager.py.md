# grammar_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/constrained/grammar_manager.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the constrained decoding and grammar control part of the SRT runtime and implements logic centered on `grammar_manager`. It exposes primary entry points such as `GrammarManager`. / 该模块属于 SRT 运行时的约束解码与语法控制部分，主要实现围绕 `grammar_manager` 的逻辑。 它对外提供的主要入口包括 `GrammarManager`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Module imports, constants, and setup
```python
from __future__ import annotations

import logging
import time
from concurrent import futures
from typing import TYPE_CHECKING, List

import torch

from sglang.srt.constrained.base_grammar_backend import (
    InvalidGrammarObject,
    create_grammar_backend,
)
from sglang.srt.constrained.reasoner_grammar_backend import ReasonerGrammarObject
from sglang.srt.environ import envs

if TYPE_CHECKING:
    from sglang.srt.managers.io_struct import AbortReq
    from sglang.srt.managers.schedule_batch import Req
    from sglang.srt.managers.scheduler import Scheduler

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; reads environment-driven configuration.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；读取环境变量驱动的配置。

### Lines 25-25: Class GrammarManager
```python
class GrammarManager:
```
**EN:** This range introduces `GrammarManager` and defines the structure or metadata that its methods rely on. In this range it handles grammar or regular-expression constraints.
**CN:** 这一段引入 `GrammarManager`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理语法或正则约束。

### Lines 26-55: Method GrammarManager.__init__
```python
    def __init__(self, scheduler: Scheduler):
        self.scheduler = scheduler
        self.server_args = scheduler.server_args
        self.grammar_queue: List[Req] = []
        if not self.server_args.skip_tokenizer_init:
            self.grammar_backend = create_grammar_backend(
                self.server_args,
                scheduler.tokenizer,
                scheduler.model_config.vocab_size,
                scheduler.model_config.hf_eos_token_id,
                think_end_id=scheduler.model_config.think_end_id,
            )
        else:
            self.grammar_backend = None

        self._enable_strict_thinking = (
            self.grammar_backend.enable_strict_thinking
            if self.grammar_backend is not None
            else False
        )

        self.grammar_sync_group = scheduler.dp_tp_cpu_group
        self.grammar_sync_size = scheduler.dp_tp_group.world_size
        self.grammar_sync_entry = scheduler.dp_tp_group.first_rank
        self.is_grammar_sync_entry = scheduler.dp_tp_group.is_first_rank

        self.SGLANG_GRAMMAR_POLL_INTERVAL = envs.SGLANG_GRAMMAR_POLL_INTERVAL.get()
        self.SGLANG_GRAMMAR_MAX_POLL_ITERATIONS = (
            envs.SGLANG_GRAMMAR_MAX_POLL_ITERATIONS.get()
        )
```
**EN:** This callable implements `GrammarManager.__init__`. It takes `scheduler` and mainly initializes instance state and defaults. In this range it reads environment-driven configuration; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `GrammarManager.__init__`。它接收 `scheduler`，主要用于初始化实例状态与默认值。 在这一范围内，它会读取环境变量驱动的配置；处理语法或正则约束。

### Lines 57-58: Method GrammarManager.__len__
```python
    def __len__(self):
        return len(self.grammar_queue)
```
**EN:** This callable implements `GrammarManager.__len__` and mainly implements len. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `GrammarManager.__len__`，主要用于实现 len 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 60-62: Method GrammarManager.clear
```python
    def clear(self):
        if self.grammar_backend:
            self.grammar_backend.reset()
```
**EN:** This callable implements `GrammarManager.clear` and mainly implements clear. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `GrammarManager.clear`，主要用于实现 clear 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 64-65: Method GrammarManager.has_waiting_grammars
```python
    def has_waiting_grammars(self) -> bool:
        return len(self.grammar_queue) > 0
```
**EN:** This callable implements `GrammarManager.has_waiting_grammars` and mainly implements has waiting grammars. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `GrammarManager.has_waiting_grammars`，主要用于实现 has waiting grammars 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 67-73: Method GrammarManager.abort_requests
```python
    def abort_requests(self, recv_req: AbortReq):
        for req in self.grammar_queue:
            if recv_req.abort_all or req.rid.startswith(recv_req.rid):
                logger.debug(f"Abort grammar queue request. {req.rid=}")
                if isinstance(req.grammar, futures.Future) and req.grammar:
                    req.grammar.cancel()
                req.set_finish_with_abort("Aborted by AbortReq.")
```
**EN:** This callable implements `GrammarManager.abort_requests`. It takes `recv_req` and mainly implements abort requests. In this range it emits logs for diagnostics; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `GrammarManager.abort_requests`。它接收 `recv_req`，主要用于实现 abort requests 相关逻辑。 在这一范围内，它会输出日志以便诊断；处理语法或正则约束。

### Lines 75-80: Method GrammarManager._get_request_thinking_budget
```python
    def _get_request_thinking_budget(self, req: Req) -> int | None:
        custom_params = req.sampling_params.custom_params
        if not isinstance(custom_params, dict):
            return None
        thinking_budget = custom_params.get("thinking_budget")
        return thinking_budget if isinstance(thinking_budget, int) else None
```
**EN:** This callable implements `GrammarManager._get_request_thinking_budget`. It takes `req` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `GrammarManager._get_request_thinking_budget`。它接收 `req`，主要用于获取某个值或派生视图。

### Lines 82-87: Method GrammarManager._apply_request_reasoning_budget
```python
    def _apply_request_reasoning_budget(self, req: Req) -> None:
        thinking_budget = self._get_request_thinking_budget(req)
        if thinking_budget is None:
            return
        if isinstance(req.grammar, ReasonerGrammarObject):
            req.grammar.max_think_tokens = thinking_budget
```
**EN:** This callable implements `GrammarManager._apply_request_reasoning_budget`. It takes `req` and mainly retrieves a value or derived view. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `GrammarManager._apply_request_reasoning_budget`。它接收 `req`，主要用于获取某个值或派生视图。 在这一范围内，它会处理语法或正则约束。

### Lines 89-140: Method GrammarManager.process_req_with_grammar
```python
    def process_req_with_grammar(self, req: Req) -> bool:
        # Init grammar cache for this request
        add_to_grammar_queue = False
        if (
            req.sampling_params.json_schema is not None
            or req.sampling_params.regex is not None
            or req.sampling_params.ebnf is not None
            or req.sampling_params.structural_tag is not None
        ):
            if self.grammar_backend is None:
                error_msg = "Grammar-based generation (json_schema, regex, ebnf, structural_tag) is not supported when the server is launched with --grammar-backend none"
                req.set_finish_with_abort(error_msg)
            else:
                if req.sampling_params.json_schema is not None:
                    key = ("json", req.sampling_params.json_schema)
                elif req.sampling_params.regex is not None:
                    key = ("regex", req.sampling_params.regex)
                elif req.sampling_params.ebnf is not None:
                    key = ("ebnf", req.sampling_params.ebnf)
                elif req.sampling_params.structural_tag:
                    key = ("structural_tag", req.sampling_params.structural_tag)

                value, cache_hit = self.grammar_backend.get_cached_or_future_value(
                    key, req.require_reasoning
                )
                req.grammar = value

                if not cache_hit:
                    req.grammar_key = key
                    add_to_grammar_queue = True
                else:
                    if isinstance(
                        value, InvalidGrammarObject
                    ):  # We hit a cached invalid grammar.
                        error_msg = (
                            f"Failed to compile {key[0]} grammar: {value.error_message}"
                        )
                        req.set_finish_with_abort(error_msg)
                    else:
                        self._apply_request_reasoning_budget(req)
        elif self._enable_strict_thinking:
            grammar_obj = self.grammar_backend.init_strict_reasoning_grammar(
                req.require_reasoning
            )
            if grammar_obj is not None:
                req.grammar = grammar_obj
                self._apply_request_reasoning_budget(req)

        if add_to_grammar_queue:
            self.grammar_queue.append(req)

        return add_to_grammar_queue
```
**EN:** This callable implements `GrammarManager.process_req_with_grammar`. It takes `req` and mainly implements process req with grammar. In this range it handles grammar or regular-expression constraints; prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `GrammarManager.process_req_with_grammar`。它接收 `req`，主要用于实现 process req with grammar 相关逻辑。 在这一范围内，它会处理语法或正则约束；处理与编译相关的行为。

### Lines 142-201: Method GrammarManager.get_ready_grammar_requests (part 1/2)
```python
    def get_ready_grammar_requests(self) -> List[Req]:
        """
        Move requests whose grammar objects are ready from grammar_queue to waiting_queue.

        Rank i returns two sets ready_reqs_i, failed_reqs_i
        ready_reqs_all = all_gather(ready_reqs_i)
        failed_reqs_all = all_gather(failed_reqs_i)

        ready_reqs = intersect(ready_reqs_all)
        failed_reqs = union(failed_reqs_all)
        """
        assert self.grammar_backend
        ready_req_idxs: set[int] = set()
        failed_req_idxs: set[int] = set()

        # Poll for ready requests
        start_time = time.perf_counter()
        while time.perf_counter() - start_time < self.SGLANG_GRAMMAR_POLL_INTERVAL:
            for i, req in enumerate(self.grammar_queue):
                if i in ready_req_idxs:
                    continue

                if req.finished() or req.grammar is None:  # It is aborted by AbortReq
                    ready_req_idxs.add(i)
                    continue

                assert isinstance(req.grammar, futures.Future), f"{req=}"
                if req.grammar.done():
                    ready_req_idxs.add(i)

            # Sleep a bit to avoid busy waiting
            time.sleep(self.SGLANG_GRAMMAR_POLL_INTERVAL / 10)

        # Check failed requests
        for i, req in enumerate(self.grammar_queue):
            if i not in ready_req_idxs:
                self.grammar_queue[i].grammar_wait_ct += 1
                if (
                    self.grammar_queue[i].grammar_wait_ct
                    >= self.SGLANG_GRAMMAR_MAX_POLL_ITERATIONS
                ):
                    # Timeout after max poll iterations
                    # The actual waiting time is SGLANG_GRAMMAR_MAX_POLL_ITERATIONS * max(SGLANG_GRAMMAR_POLL_INTERVAL, GPU_forward_batch_latency)
                    failed_req_idxs.add(i)

        # Sync ready and failed requests across all ranks
        if self.grammar_sync_size == 1:
            synced_ready_req_idxs = ready_req_idxs
            synced_failed_req_idxs = failed_req_idxs
        else:
            all_gather_output = [None] * self.grammar_sync_size
            torch.distributed.all_gather_object(
                all_gather_output,
                (ready_req_idxs, failed_req_idxs),
                group=self.grammar_sync_group,
            )
            synced_ready_req_idxs = set.intersection(*[x[0] for x in all_gather_output])
            synced_failed_req_idxs = set.union(*[x[1] for x in all_gather_output])

        # Return ready requests
```
**EN:** This callable implements `GrammarManager.get_ready_grammar_requests` and mainly retrieves a value or derived view. The docstring states: "Move requests whose grammar objects are ready from grammar_queue to waiting_queue." This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GrammarManager.get_ready_grammar_requests`，主要用于获取某个值或派生视图。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；协调分布式通信。

### Lines 202-243: Method GrammarManager.get_ready_grammar_requests (part 2/2)
```python
        return_reqs: List[Req] = []
        for i in synced_ready_req_idxs:
            req = self.grammar_queue[i]
            return_reqs.append(req)
            if req.finished() or req.grammar is None:  # It is aborted by AbortReq
                continue

            assert isinstance(req.grammar, futures.Future) and req.grammar_key
            try:
                req.grammar = req.grammar.result()
            except Exception as e:
                logger.error(
                    f"Grammar compilation raised an exception: {e}, "
                    f"grammar_key={req.grammar_key}"
                )
                req.grammar = InvalidGrammarObject(f"Grammar compilation failed: {e}")
            self.grammar_backend.set_cache(req.grammar_key, req.grammar.copy())
            self._apply_request_reasoning_budget(req)
            if isinstance(req.grammar, InvalidGrammarObject):
                error_msg = f"Failed to compile {req.grammar_key[0]} grammar: {req.grammar.error_message}"
                req.set_finish_with_abort(error_msg)

        # Return failed requests
        for i in synced_failed_req_idxs:
            req = self.grammar_queue[i]
            return_reqs.append(req)

            assert isinstance(req.grammar, futures.Future) and req.grammar_key
            req.grammar.cancel()
            self.grammar_backend.set_cache(
                req.grammar_key, InvalidGrammarObject("Grammar preprocessing timed out")
            )
            error_msg = f"Grammar preprocessing timed out: {req.grammar_key=}"
            req.set_finish_with_abort(error_msg)

        # Remove finished requests from grammar_queue
        self.grammar_queue = [
            req
            for i, req in enumerate(self.grammar_queue)
            if i not in synced_ready_req_idxs and i not in synced_failed_req_idxs
        ]
        return return_reqs
```
**EN:** This callable implements `GrammarManager.get_ready_grammar_requests` and mainly retrieves a value or derived view. The docstring states: "Move requests whose grammar objects are ready from grammar_queue to waiting_queue." This chunk is part 2 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `GrammarManager.get_ready_grammar_requests`，主要用于获取某个值或派生视图。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；输出日志以便诊断。

## Key Concepts / 关键概念
- `GrammarManager`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `time`, `concurrent`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.constrained.base_grammar_backend`, `sglang.srt.constrained.reasoner_grammar_backend`, `sglang.srt.environ`, `sglang.srt.managers.io_struct`, `sglang.srt.managers.schedule_batch`, `sglang.srt.managers.scheduler`
