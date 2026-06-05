# schedule_policy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/schedule_policy.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements schedule policy logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 调度 policy 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-3: Import runtime dependencies / 导入运行时依赖
```python
import logging
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 5-7: Import runtime dependencies / 导入运行时依赖
```python
from sglang.srt.environ import envs
from sglang.srt.managers.prefill_delayer import PrefillDelayerSinglePassExecutor
from sglang.srt.utils import get_bool_env_var
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 9-25: Provide supporting module logic / 提供辅助模块逻辑
```python
_ROUTING_KEY_POLICY_DEBUG_LOG = get_bool_env_var("SGLANG_ROUTING_KEY_POLICY_DEBUG_LOG")
logger = logging.getLogger(__name__)

# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
"""Request scheduler policy"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 27-32: Import runtime dependencies / 导入运行时依赖
```python
import os
import random
from collections import Counter, defaultdict
from contextlib import contextmanager
from enum import Enum, auto
from typing import TYPE_CHECKING, Dict, List, Optional, Set, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 34-34: Import runtime dependencies / 导入运行时依赖
```python
import torch
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 36-52: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.dllm.config import DllmConfig
from sglang.srt.layers.attention.nsa.utils import is_nsa_prefill_cp_in_seq_split
from sglang.srt.layers.utils.cp_utils import is_prefill_context_parallel_enabled
from sglang.srt.managers.schedule_batch import Req, ScheduleBatch
from sglang.srt.mem_cache.base_prefix_cache import (
    BasePrefixCache,
    InitLoadBackParams,
    InsertParams,
    MatchPrefixParams,
    zero_match_result,
)
from sglang.srt.mem_cache.hisparse_memory_pool import (
    DeepSeekV4HiSparseTokenToKVPoolAllocator,
)
from sglang.srt.mem_cache.radix_cache import RadixCache, RadixKey, TreeNode
from sglang.srt.mem_cache.swa_memory_pool import SWATokenToKVPoolAllocator
from sglang.srt.server_args import ServerArgs
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 54-78: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.mem_cache.allocator import BaseTokenToKVPoolAllocator

# Clip the estimation of max_new_tokens for the request whose max_new_tokens is very large.
# This can prevent the server from being too conservative.
# Note that this only clips the estimation in the scheduler but does not change the stop
# condition. The request can still generate tokens until it hits the unclipped max_new_tokens.
CLIP_MAX_NEW_TOKENS = int(
    os.environ.get("SGLANG_CLIP_MAX_NEW_TOKENS_ESTIMATION", "4096")
)

# Threshold for in-batch prefix cache.
# If a request has a matched prefix length (against existing cache) less than this value,
# the scheduler runs the in-batch prefix caching check for this request.
# If we set it to -1, it means we disable in-batch prefix caching.
IN_BATCH_PREFIX_CACHING_CHECK_THRESHOLD = int(
    os.environ.get("IN_BATCH_PREFIX_CACHING_CHECK_THRESHOLD", "32")
)

# Threshold for in-batch prefix cache.
# If a request has a matched prefix length (within the waiting queue) larger than this value,
# the scheduler deprioritizes this request
IN_BATCH_PREFIX_CACHING_DEPRIORITIZE_THRESHOLD = int(
    os.environ.get("IN_BATCH_PREFIX_CACHING_DEPRIORITIZE_THRESHOLD", "32")
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 81-81: Provide supporting module logic / 提供辅助模块逻辑
```python
IGNORE_EOS_RESERVE_TOKENS = 1
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 84-121: Implement match prefix for req / 实现match prefix for req
```python
def match_prefix_for_req(
    tree_cache: BasePrefixCache,
    req: Req,
    token_ids: Optional[List[int]] = None,
    *,
    cow_mamba: bool = False,
    include_req: bool = False,
):
    if token_ids is None:
        token_ids = req.origin_input_ids + req.output_ids

    match_result = tree_cache.match_prefix(
        MatchPrefixParams(
            key=RadixKey(token_ids=token_ids, extra_key=req.extra_key),
            cow_mamba=cow_mamba,
            req=req if include_req else None,
        )
    )
    if envs.SGLANG_RADIX_FORCE_MISS.get():
        match_result = zero_match_result(tree_cache, match_result)
    (
        req.prefix_indices,
        req.last_node,
        req.last_host_node,
        req.best_match_node,
        req.host_hit_length,
    ) = (
        match_result.device_indices,
        match_result.last_device_node,
        match_result.last_host_node,
        match_result.best_match_node,
        match_result.host_hit_length,
    )
    if match_result.mamba_branching_seqlen is not None:
        req.mamba_branching_seqlen = match_result.mamba_branching_seqlen
    if match_result.cache_protected_len is not None:
        req.cache_protected_len = match_result.cache_protected_len
    return match_result
```
**EN:** This block implements the function `match_prefix_for_req(tree_cache, req, token_ids, cow_mamba, include_req)`. It focuses on handling the schedule policy responsibilities represented by `match_prefix_for_req`, providing reusable behavior for the schedule policy pipeline.
**CN:** 该代码块实现函数 `match_prefix_for_req(tree_cache, req, token_ids, cow_mamba, include_req)`。它围绕 `match_prefix_for_req` 所承担的 调度 policy 相关职责展开，为对应处理链路提供可复用能力。

### Lines 124-128: Define class CacheAwarePolicy / 定义类 CacheAwarePolicy
```python
class CacheAwarePolicy(Enum):
    """Scheduling policies that are aware of the tree cache."""

    LPM = "lpm"  # longest prefix match
    DFS_WEIGHT = "dfs-weight"  # depth-first search weighting
```
**EN:** This block declares the class `CacheAwarePolicy`. It centers on Scheduling policies that are aware of the tree cache..
**CN:** 该代码块声明类 `CacheAwarePolicy`。它负责承载与 调度 policy 相关的核心状态与行为。

### Lines 131-137: Define class CacheAgnosticPolicy / 定义类 CacheAgnosticPolicy
```python
class CacheAgnosticPolicy(Enum):
    """Scheduling policies that are not aware of the tree cache."""

    FCFS = "fcfs"  # first come first serve
    LOF = "lof"  # longest output first
    RANDOM = "random"
    ROUTING_KEY = "routing-key"  # prioritize by routing key frequency in running batch
```
**EN:** This block declares the class `CacheAgnosticPolicy`. It centers on Scheduling policies that are not aware of the tree cache..
**CN:** 该代码块声明类 `CacheAgnosticPolicy`。它负责承载与 调度 policy 相关的核心状态与行为。

### Lines 140-141: Provide supporting module logic / 提供辅助模块逻辑
```python
class SchedulePolicy:
    Policy = Union[CacheAwarePolicy, CacheAgnosticPolicy]
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 143-159: Initialize SchedulePolicy / 初始化 SchedulePolicy
```python
    def __init__(
        self,
        policy: str,
        tree_cache: BasePrefixCache,
        enable_hierarchical_cache: bool,
        enable_priority_scheduling: bool,
        schedule_low_priority_values_first: bool,
    ):
        self.policy = self._validate_and_adjust_policy(policy, tree_cache)
        self.tree_cache = tree_cache
        self.enable_hierarchical_cache = enable_hierarchical_cache
        self.enable_priority_scheduling = enable_priority_scheduling
        self.schedule_low_priority_values_first = schedule_low_priority_values_first
        self.priority_sign = 1 if schedule_low_priority_values_first else -1

        # It is used to find the matching prefix for in-batch prefix caching.
        self.waiting_queue_radix_tree = RadixCache.create_simulated()
```
**EN:** This block implements the initializer `__init__(policy, tree_cache, enable_hierarchical_cache, enable_priority_scheduling, schedule_low_priority_values_first)` for `SchedulePolicy`. It prepares the object state and connects the instance to the surrounding schedule policy workflow.
**CN:** 该代码块实现 `SchedulePolicy` 的初始化方法 `__init__(policy, tree_cache, enable_hierarchical_cache, enable_priority_scheduling, schedule_low_priority_values_first)`。它负责准备对象状态，并把实例接入 调度 policy 相关的运行流程。

### Lines 161-203: Implement calc priority / 实现calc priority
```python
    def calc_priority(
        self, waiting_queue: List[Req], running_batch: Optional[ScheduleBatch] = None
    ) -> bool:
        if self.policy == CacheAgnosticPolicy.FCFS:
            if self.enable_priority_scheduling:
                SchedulePolicy._sort_by_priority_and_fcfs(
                    waiting_queue, self.priority_sign
                )
            return False

        policy = self._determine_active_policy(waiting_queue)

        prefix_computed = False
        if isinstance(policy, CacheAwarePolicy):
            prefix_computed = True
            temporary_deprioritized = self._compute_prefix_matches(
                waiting_queue, policy
            )
            if policy == CacheAwarePolicy.LPM:
                SchedulePolicy._sort_by_longest_prefix(
                    waiting_queue, temporary_deprioritized
                )
            elif policy == CacheAwarePolicy.DFS_WEIGHT:
                SchedulePolicy._sort_by_dfs_weight(waiting_queue, self.tree_cache)
            else:
                raise ValueError(f"Unknown CacheAware Policy: {policy=}")
        else:
            if policy == CacheAgnosticPolicy.FCFS:
                pass
            elif policy == CacheAgnosticPolicy.LOF:
                SchedulePolicy._sort_by_longest_output(
                    waiting_queue,
                    self.enable_priority_scheduling,
                    self.priority_sign,
                )
            elif policy == CacheAgnosticPolicy.RANDOM:
                SchedulePolicy._sort_randomly(waiting_queue)
            elif policy == CacheAgnosticPolicy.ROUTING_KEY:
                if running_batch is not None:
                    SchedulePolicy._sort_by_routing_key(waiting_queue, running_batch)
            else:
                raise ValueError(f"Unknown CacheAgnostic Policy: {policy=}")
        return prefix_computed
```
**EN:** This block implements the method `calc_priority(waiting_queue, running_batch)` on `SchedulePolicy`. It focuses on handling the schedule policy responsibilities represented by `calc_priority`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulePolicy` 上的方法 `calc_priority(waiting_queue, running_batch)`。它围绕 `calc_priority` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 205-209: Implement determine active policy / 实现determine active policy
```python
    def _determine_active_policy(self, waiting_queue: List[Req]) -> Policy:
        if self.policy == CacheAwarePolicy.LPM and len(waiting_queue) > 128:
            # Turn off the expensive prefix matching and sorting when the #queue is large.
            return CacheAgnosticPolicy.FCFS
        return self.policy
```
**EN:** This block implements the method `_determine_active_policy(waiting_queue)` on `SchedulePolicy`. It focuses on handling the schedule policy responsibilities represented by `_determine_active_policy`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulePolicy` 上的方法 `_determine_active_policy(waiting_queue)`。它围绕 `_determine_active_policy` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 211-227: Implement validate and adjust policy / 实现validate and adjust policy
```python
    def _validate_and_adjust_policy(
        self, policy: str, tree_cache: BasePrefixCache
    ) -> Policy:
        """
        Validates the policy and adjusts it if necessary based on tree cache settings.
        """
        try:
            policy_enum = CacheAwarePolicy(policy)
            if getattr(tree_cache, "disable", True):
                # If tree_cache is disabled, using CacheAgnosticPolicy policy
                return CacheAgnosticPolicy.FCFS
            return policy_enum
        except ValueError:
            try:
                return CacheAgnosticPolicy(policy)
            except ValueError:
                raise ValueError(f"Unknown schedule_policy: {policy=}")
```
**EN:** This block implements the method `_validate_and_adjust_policy(policy, tree_cache)` on `SchedulePolicy`. It focuses on Validates the policy and adjusts it if necessary based on tree cache settings., so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulePolicy` 上的方法 `_validate_and_adjust_policy(policy, tree_cache)`。它围绕 `_validate_and_adjust_policy` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 229-275: Implement compute prefix matches / 实现compute prefix matches
```python
    def _compute_prefix_matches(
        self, waiting_queue: List[Req], policy: CacheAwarePolicy
    ) -> Set[int]:
        """
        Computes and caches the matching prefixes for requests in the waiting queue,
            and handles in-batch prefix caching logic.
        """
        temporary_deprioritized: Set[int] = set()
        self.waiting_queue_radix_tree.reset()

        for r in waiting_queue:
            prefix_ids = r.origin_input_ids + r.output_ids
            extra_key = r.extra_key
            match_result = match_prefix_for_req(self.tree_cache, r, prefix_ids)

            # NOTE(sang): This logic is for in-batch prefix caching;
            # If there are more than 1 request that have small matching prefix from
            # existing cache, but all those requests share the same prefix, we prefer
            # to schedule only one of them so that we can increase the cache hit rate.
            # We prefer to set IN_BATCH_PREFIX_CACHING_CHECK_THRESHOLD > 0 because too small
            # threshold means we cannot use in-batch prefix caching for short prefixes.
            # It is kind of common when the engine is long running (e.g., imagine the prefix "the").
            if len(r.prefix_indices) <= IN_BATCH_PREFIX_CACHING_CHECK_THRESHOLD:
                match_result = self.waiting_queue_radix_tree.match_prefix(
                    MatchPrefixParams(
                        key=RadixKey(token_ids=prefix_ids, extra_key=extra_key)
                    )
                )
                if envs.SGLANG_RADIX_FORCE_MISS.get():
                    match_result = zero_match_result(
                        self.waiting_queue_radix_tree, match_result
                    )
                in_batch_matching_prefixes = match_result.device_indices
                if (
                    len(in_batch_matching_prefixes)
                    >= IN_BATCH_PREFIX_CACHING_DEPRIORITIZE_THRESHOLD
                ):
                    temporary_deprioritized.add(r.rid)
                else:
                    # Insert with a dummy key
                    self.waiting_queue_radix_tree.insert(
                        InsertParams(
                            key=RadixKey(token_ids=prefix_ids, extra_key=extra_key),
                            value=torch.empty(len(prefix_ids), dtype=torch.bool),
                        )
                    )
        return temporary_deprioritized
```
**EN:** This block implements the method `_compute_prefix_matches(waiting_queue, policy)` on `SchedulePolicy`. It focuses on Computes and caches the matching prefixes for requests in the waiting queue, and handles in-batch prefix caching logic., so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulePolicy` 上的方法 `_compute_prefix_matches(waiting_queue, policy)`。它围绕 `_compute_prefix_matches` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 276-277: Provide supporting module logic / 提供辅助模块逻辑
```python

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 278-288: Implement sort by longest prefix / 实现sort by longest prefix
```python
    def _sort_by_longest_prefix(
        waiting_queue: List[Req], temporary_deprioritized: Set[int]
    ) -> None:
        """Sorts the waiting queue based on the longest prefix match."""
        waiting_queue.sort(
            key=lambda r: (
                -len(r.prefix_indices)
                if r.rid not in temporary_deprioritized
                else float("inf")
            )
        )
```
**EN:** This block implements the method `_sort_by_longest_prefix(waiting_queue, temporary_deprioritized)` on `SchedulePolicy`. It focuses on Sorts the waiting queue based on the longest prefix match., so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulePolicy` 上的方法 `_sort_by_longest_prefix(waiting_queue, temporary_deprioritized)`。它围绕 `_sort_by_longest_prefix` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 289-290: Provide supporting module logic / 提供辅助模块逻辑
```python

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 291-310: Implement sort by dfs weight / 实现sort by dfs weight
```python
    def _sort_by_dfs_weight(
        waiting_queue: List[Req], tree_cache: BasePrefixCache
    ) -> None:
        """Sorts the waiting queue based on a depth-first search weighting."""
        last_node_to_reqs = defaultdict(list)
        for req in waiting_queue:
            last_node_to_reqs[req.last_node].append(req)

        node_to_weight = defaultdict(int)
        for node in last_node_to_reqs:
            node_to_weight[node] = len(last_node_to_reqs[node])
        SchedulePolicy._calc_weight(tree_cache.root_node, node_to_weight)

        waiting_queue.clear()
        SchedulePolicy._get_dfs_priority(
            tree_cache.root_node,
            node_to_weight,
            last_node_to_reqs,
            waiting_queue,
        )
```
**EN:** This block implements the method `_sort_by_dfs_weight(waiting_queue, tree_cache)` on `SchedulePolicy`. It focuses on Sorts the waiting queue based on a depth-first search weighting., so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulePolicy` 上的方法 `_sort_by_dfs_weight(waiting_queue, tree_cache)`。它围绕 `_sort_by_dfs_weight` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 311-312: Provide supporting module logic / 提供辅助模块逻辑
```python

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 313-327: Implement sort by longest output / 实现sort by longest output
```python
    def _sort_by_longest_output(
        waiting_queue: List[Req],
        enable_priority_scheduling: bool,
        priority_sign: int,
    ) -> None:
        """Sorts the waiting queue based on the longest output (max_new_tokens). If using priority scheduling, sort by priority first."""
        if enable_priority_scheduling:
            waiting_queue.sort(
                key=lambda x: (
                    x.priority * priority_sign,
                    -x.sampling_params.max_new_tokens,
                )
            )
        else:
            waiting_queue.sort(key=lambda x: -x.sampling_params.max_new_tokens)
```
**EN:** This block implements the method `_sort_by_longest_output(waiting_queue, enable_priority_scheduling, priority_sign)` on `SchedulePolicy`. It focuses on Sorts the waiting queue based on the longest output (max_new_tokens)., so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulePolicy` 上的方法 `_sort_by_longest_output(waiting_queue, enable_priority_scheduling, priority_sign)`。它围绕 `_sort_by_longest_output` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 328-329: Provide supporting module logic / 提供辅助模块逻辑
```python

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 330-332: Implement sort randomly / 实现sort randomly
```python
    def _sort_randomly(waiting_queue: List[Req]) -> None:
        """Shuffles the waiting queue randomly."""
        random.shuffle(waiting_queue)
```
**EN:** This block implements the method `_sort_randomly(waiting_queue)` on `SchedulePolicy`. It focuses on Shuffles the waiting queue randomly., so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulePolicy` 上的方法 `_sort_randomly(waiting_queue)`。它围绕 `_sort_randomly` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 333-334: Provide supporting module logic / 提供辅助模块逻辑
```python

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 335-344: Implement sort by priority and fcfs / 实现sort by priority and fcfs
```python
    def _sort_by_priority_and_fcfs(
        waiting_queue: List[Req], priority_sign: int
    ) -> None:
        """Sorts the waiting queue based on the request priority then received titmestamp."""
        waiting_queue.sort(
            key=lambda x: (
                x.priority * priority_sign,
                x.time_stats.wait_queue_entry_time,
            )
        )
```
**EN:** This block implements the method `_sort_by_priority_and_fcfs(waiting_queue, priority_sign)` on `SchedulePolicy`. It focuses on Sorts the waiting queue based on the request priority then received titmestamp., so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulePolicy` 上的方法 `_sort_by_priority_and_fcfs(waiting_queue, priority_sign)`。它围绕 `_sort_by_priority_and_fcfs` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 345-346: Provide supporting module logic / 提供辅助模块逻辑
```python

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 347-377: Implement sort by routing key / 实现sort by routing key
```python
    def _sort_by_routing_key(
        waiting_queue: List[Req], running_batch: ScheduleBatch
    ) -> None:
        """Sorts waiting queue by routing key frequency in running batch."""
        routing_key_counts = Counter(
            r.routing_key for r in running_batch.reqs if r.routing_key
        )

        if _ROUTING_KEY_POLICY_DEBUG_LOG:
            waiting_keys_before = [r.routing_key for r in waiting_queue]
            logger.info(
                f"routing_key_counts={dict(routing_key_counts)}, "
                f"waiting_keys_before={waiting_keys_before}"
            )

        if not routing_key_counts:
            return

        def sort_key(req: Req):
            key = req.routing_key
            if key and key in routing_key_counts:
                count = routing_key_counts[key]
                return (0, -count, key)
            else:
                return (1, 0, key or "")

        waiting_queue.sort(key=sort_key)

        if _ROUTING_KEY_POLICY_DEBUG_LOG:
            waiting_keys_after = [r.routing_key for r in waiting_queue]
            logger.info(f"waiting_keys_after={waiting_keys_after}")
```
**EN:** This block implements the method `_sort_by_routing_key(waiting_queue, running_batch)` on `SchedulePolicy`. It focuses on Sorts waiting queue by routing key frequency in running batch., so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulePolicy` 上的方法 `_sort_by_routing_key(waiting_queue, running_batch)`。它围绕 `_sort_by_routing_key` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 378-379: Provide supporting module logic / 提供辅助模块逻辑
```python

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 380-383: Implement calc weight / 实现calc weight
```python
    def _calc_weight(cur_node: TreeNode, node_to_weight: Dict[TreeNode, int]) -> None:
        for child in cur_node.children.values():
            SchedulePolicy._calc_weight(child, node_to_weight)
            node_to_weight[cur_node] += node_to_weight[child]
```
**EN:** This block implements the method `_calc_weight(cur_node, node_to_weight)` on `SchedulePolicy`. It focuses on handling the schedule policy responsibilities represented by `_calc_weight`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulePolicy` 上的方法 `_calc_weight(cur_node, node_to_weight)`。它围绕 `_calc_weight` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 384-385: Provide supporting module logic / 提供辅助模块逻辑
```python

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 386-398: Implement get dfs priority / 实现get dfs priority
```python
    def _get_dfs_priority(
        cur_node: TreeNode,
        node_to_priority: Dict[TreeNode, int],
        last_node_to_reqs: Dict[TreeNode, List[Req]],
        q: List,
    ) -> None:
        children = [child for child in cur_node.children.values()]
        children.sort(key=lambda x: -node_to_priority[x])
        for child in children:
            SchedulePolicy._get_dfs_priority(
                child, node_to_priority, last_node_to_reqs, q
            )
        q.extend(last_node_to_reqs[cur_node])
```
**EN:** This block implements the method `_get_dfs_priority(cur_node, node_to_priority, last_node_to_reqs, q)` on `SchedulePolicy`. It focuses on handling the schedule policy responsibilities represented by `_get_dfs_priority`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulePolicy` 上的方法 `_get_dfs_priority(cur_node, node_to_priority, last_node_to_reqs, q)`。它围绕 `_get_dfs_priority` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 401-404: Define class AddReqResult / 定义类 AddReqResult
```python
class AddReqResult(Enum):
    CONTINUE = auto()  # Continue to add requests
    NO_TOKEN = auto()  # No token left
    OTHER = auto()  # Other reasons to stop adding requests
```
**EN:** This block declares the class `AddReqResult`. It centers on coordinating schedule policy behavior.
**CN:** 该代码块声明类 `AddReqResult`。它负责承载与 调度 policy 相关的核心状态与行为。

### Lines 407-407: Provide supporting module logic / 提供辅助模块逻辑
```python
class PrefillAdder:
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 408-443: Initialize PrefillAdder / 初始化 PrefillAdder
```python
    def __init__(
        self,
        page_size: int,
        tree_cache: BasePrefixCache,
        token_to_kv_pool_allocator: BaseTokenToKVPoolAllocator,
        running_batch: ScheduleBatch,
        new_token_ratio: float,
        rem_input_tokens: int,
        rem_chunk_tokens: Optional[int],
        num_mixed_decode_tokens: int = 0,
        priority_scheduling_preemption_threshold: int = 0,
        max_prefill_bs: int = 0,
        max_running_requests: Optional[int] = None,
        prefill_max_requests: Optional[int] = None,
        prefill_delayer_single_pass: Optional[PrefillDelayerSinglePassExecutor] = None,
        dllm_config: Optional[DllmConfig] = None,
        waiting_queue_len: int = 0,
    ):
        self.page_size = page_size
        self.tree_cache = tree_cache
        self.token_to_kv_pool_allocator = token_to_kv_pool_allocator
        self.running_batch = running_batch
        self.new_token_ratio = new_token_ratio
        self.rem_input_tokens = rem_input_tokens - num_mixed_decode_tokens
        self.rem_chunk_tokens = rem_chunk_tokens
        self.dllm_config = dllm_config

        if self.dllm_config is not None:
            self._init_dllm_meta(dllm_config)

        if self.rem_chunk_tokens is not None:
            self.rem_chunk_tokens -= num_mixed_decode_tokens
        self.rem_total_token_offset = num_mixed_decode_tokens
        self.cur_rem_token_offset = num_mixed_decode_tokens

        self.req_states = None
```
**EN:** This block implements the initializer `__init__(page_size, tree_cache, token_to_kv_pool_allocator, running_batch, new_token_ratio, ...)` for `PrefillAdder`. It prepares the object state and connects the instance to the surrounding schedule policy workflow.
**CN:** 该代码块实现 `PrefillAdder` 的初始化方法 `__init__(page_size, tree_cache, token_to_kv_pool_allocator, running_batch, new_token_ratio, ...)`。它负责准备对象状态，并把实例接入 调度 policy 相关的运行流程。

### Lines 444-479: Initialize PrefillAdder (continued) / 初始化 PrefillAdder（续）
```python
        self.can_run_list = []
        self.preempt_list = []
        self.new_chunked_req = None
        self.log_hit_tokens = 0
        # TODO(lsyin): report the real input tokens excluding page alignment
        self.log_input_tokens = 0

        if running_batch is not None:
            # Estimate the offset in the remaining token space
            self.rem_total_token_offset += sum(
                [
                    self._get_running_request_total_token_offset(r)
                    for r in running_batch.reqs
                ]
            )

        # DeepSeek V4 HiSparse wraps an SWATokenToKVPoolAllocator internally and
        # exposes the full SWA allocator interface.
        self.is_hybrid_swa = isinstance(
            self.token_to_kv_pool_allocator,
            (SWATokenToKVPoolAllocator, DeepSeekV4HiSparseTokenToKVPoolAllocator),
        )
        self.is_hybrid_ssm_cache = self.tree_cache.supports_mamba()

        self.rem_swa_token_offset = 0

        self.priority_scheduling_preemption_threshold = (
            priority_scheduling_preemption_threshold
        )
        self.nsa_prefill_cp_in_seq_split = is_nsa_prefill_cp_in_seq_split()
        self.max_running_requests = max_running_requests
        self.prefill_context_parallel_enabled = is_prefill_context_parallel_enabled()
        self.prefill_max_requests = prefill_max_requests
        self.prefill_delayer_single_pass = prefill_delayer_single_pass
        self.max_prefill_bs = max_prefill_bs
        # Snapshot of scheduler waiting_queue length at the start of this
```
**EN:** This block implements the initializer `__init__(page_size, tree_cache, token_to_kv_pool_allocator, running_batch, new_token_ratio, ...)` for `PrefillAdder`. It prepares the object state and connects the instance to the surrounding schedule policy workflow.
**CN:** 该代码块实现 `PrefillAdder` 的初始化方法 `__init__(page_size, tree_cache, token_to_kv_pool_allocator, running_batch, new_token_ratio, ...)`。它负责准备对象状态，并把实例接入 调度 policy 相关的运行流程。

### Lines 480-481: Initialize PrefillAdder (continued) / 初始化 PrefillAdder（续）
```python
        # prefill pass. Used by PrefillDelayer's queue-based trigger.
        self.waiting_queue_len = waiting_queue_len
```
**EN:** This block implements the initializer `__init__(page_size, tree_cache, token_to_kv_pool_allocator, running_batch, new_token_ratio, ...)` for `PrefillAdder`. It prepares the object state and connects the instance to the surrounding schedule policy workflow.
**CN:** 该代码块实现 `PrefillAdder` 的初始化方法 `__init__(page_size, tree_cache, token_to_kv_pool_allocator, running_batch, new_token_ratio, ...)`。它负责准备对象状态，并把实例接入 调度 policy 相关的运行流程。

### Lines 483-487: Implement init dllm meta / 实现init dllm meta
```python
    def _init_dllm_meta(self, dllm_config: DllmConfig):
        self.dllm_block_size = dllm_config.block_size
        max_running_reqs = dllm_config.max_running_requests

        self.rem_dllm_tokens = max_running_reqs * self.dllm_block_size
```
**EN:** This block implements the method `_init_dllm_meta(dllm_config)` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `_init_dllm_meta`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `_init_dllm_meta(dllm_config)`。它围绕 `_init_dllm_meta` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 489-496: Implement get running request total token offset / 实现get running 请求 total Token offset
```python
    def _get_running_request_total_token_offset(self, req: Req) -> int:
        return (
            min(
                (req.sampling_params.max_new_tokens - len(req.output_ids)),
                CLIP_MAX_NEW_TOKENS,
            )
            * self.new_token_ratio
        )
```
**EN:** This block implements the method `_get_running_request_total_token_offset(req)` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `_get_running_request_total_token_offset`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `_get_running_request_total_token_offset(req)`。它围绕 `_get_running_request_total_token_offset` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 497-498: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 499-515: Implement rem total tokens / 实现rem total tokens
```python
    def rem_total_tokens(self):
        if self.is_hybrid_swa:
            available_and_evictable = (
                self.token_to_kv_pool_allocator.full_available_size()
                + self.tree_cache.full_evictable_size()
            )
        elif self.is_hybrid_ssm_cache:
            available_and_evictable = (
                self.token_to_kv_pool_allocator.available_size()
                + self.tree_cache.full_evictable_size()
            )
        else:
            available_and_evictable = (
                self.token_to_kv_pool_allocator.available_size()
                + self.tree_cache.evictable_size()
            )
        return available_and_evictable - self.rem_total_token_offset
```
**EN:** This block implements the method `rem_total_tokens()` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `rem_total_tokens`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `rem_total_tokens()`。它围绕 `rem_total_tokens` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 516-517: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 518-523: Implement rem swa tokens / 实现rem swa tokens
```python
    def rem_swa_tokens(self):
        return (
            self.token_to_kv_pool_allocator.swa_available_size()
            + self.tree_cache.swa_evictable_size()
            - self.rem_swa_token_offset
        )
```
**EN:** This block implements the method `rem_swa_tokens()` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `rem_swa_tokens`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `rem_swa_tokens()`。它围绕 `rem_swa_tokens` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 524-525: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 526-543: Implement cur rem tokens / 实现cur rem tokens
```python
    def cur_rem_tokens(self):
        if self.is_hybrid_swa:
            available_and_evictable = (
                self.token_to_kv_pool_allocator.full_available_size()
                + self.tree_cache.full_evictable_size()
            )
        elif self.is_hybrid_ssm_cache:
            available_and_evictable = (
                self.token_to_kv_pool_allocator.available_size()
                + self.tree_cache.full_evictable_size()
            )
        else:
            available_and_evictable = (
                self.token_to_kv_pool_allocator.available_size()
                + self.tree_cache.evictable_size()
            )

        return available_and_evictable - self.cur_rem_token_offset
```
**EN:** This block implements the method `cur_rem_tokens()` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `cur_rem_tokens`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `cur_rem_tokens()`。它围绕 `cur_rem_tokens` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 545-560: Implement swa budget for req / 实现swa budget for req
```python
    def _swa_budget_for_req(self, extend_input_len: int) -> int:
        """SWA pool budget per request. Only valid when is_hybrid_swa is True.

        With chunked prefill + overlap scheduler, the peak SWA occupancy is:
          chunk N (running, not yet in tree) + sliding window (locked in tree)
          + chunk N+1 (new allocation)
        Since chunk N and locked tokens are already excluded from
        swa_available + swa_evictable, the budget only needs to cover the
        chunk N+1 allocation. We floor at sliding_window_size to reserve
        room for the decode phase.
        """
        if self.rem_chunk_tokens is not None:
            alloc = min(extend_input_len, self.rem_chunk_tokens)
        else:
            alloc = extend_input_len
        return max(alloc, self.tree_cache.sliding_window_size) + self.page_size
```
**EN:** This block implements the method `_swa_budget_for_req(extend_input_len)` on `PrefillAdder`. It focuses on SWA pool budget per request., so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `_swa_budget_for_req(extend_input_len)`。它围绕 `_swa_budget_for_req` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 562-563: Implement ceil paged tokens / 实现ceil paged tokens
```python
    def ceil_paged_tokens(self, tokens: int) -> int:
        return -(-tokens // self.page_size) * self.page_size
```
**EN:** This block implements the method `ceil_paged_tokens(tokens)` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `ceil_paged_tokens`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `ceil_paged_tokens(tokens)`。它围绕 `ceil_paged_tokens` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 565-582: Implement budget state / 实现budget 状态
```python
    def budget_state(self):
        no_token = self.rem_total_tokens <= 0 or self.cur_rem_tokens <= 0
        if not no_token and self.is_hybrid_swa:
            no_token = self.rem_swa_tokens <= 0
        if no_token:
            return AddReqResult.NO_TOKEN

        if self.rem_input_tokens <= 0:
            return AddReqResult.OTHER

        if self.dllm_config is not None:
            if self.rem_dllm_tokens <= 0:
                return AddReqResult.OTHER
        else:
            if self.rem_chunk_tokens is not None and self.rem_chunk_tokens <= 0:
                return AddReqResult.OTHER

        return AddReqResult.CONTINUE
```
**EN:** This block implements the method `budget_state()` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `budget_state`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `budget_state()`。它围绕 `budget_state` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 584-605: Implement update prefill budget / 实现update prefill budget
```python
    def _update_prefill_budget(
        self, prefix_len: int, extend_input_len: int, max_new_tokens: int
    ):
        # TODO(lsyin): check this workaround logic, which only ensures the prefill will not out of memory, and may be too conservative
        extend_input_len = self.ceil_paged_tokens(extend_input_len)

        # alloc_extend reserves an extra page_size per request to make sure the budget doesn't over-commit
        page_overhead = self.page_size
        self.rem_total_token_offset += extend_input_len + max_new_tokens + page_overhead
        self.cur_rem_token_offset += extend_input_len + page_overhead
        self.rem_input_tokens -= extend_input_len

        if self.is_hybrid_swa:
            self.rem_swa_token_offset += self._swa_budget_for_req(extend_input_len)

        if self.dllm_config is not None:
            self.rem_dllm_tokens -= extend_input_len
        elif self.rem_chunk_tokens is not None:
            self.rem_chunk_tokens -= extend_input_len

        self.log_hit_tokens += prefix_len
        self.log_input_tokens += extend_input_len
```
**EN:** This block implements the method `_update_prefill_budget(prefix_len, extend_input_len, max_new_tokens)` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `_update_prefill_budget`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `_update_prefill_budget(prefix_len, extend_input_len, max_new_tokens)`。它围绕 `_update_prefill_budget` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 607-616: Implement get dllm remain tokens / 实现get dllm remain tokens
```python
    def _get_dllm_remain_tokens(self) -> int:
        _rem_tokens = min(
            self.rem_dllm_tokens,
            self.dllm_block_size,
            int(self.rem_total_tokens),
        )
        if _rem_tokens <= 0:
            _rem_tokens = self.rem_dllm_tokens

        return _rem_tokens
```
**EN:** This block implements the method `_get_dllm_remain_tokens()` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `_get_dllm_remain_tokens`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `_get_dllm_remain_tokens()`。它围绕 `_get_dllm_remain_tokens` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 618-633: Implement add dllm req / 实现add dllm req
```python
    def _add_dllm_req(self, req: Req, prefix_len: int):
        # FIXME: consider the case when rem_dllm_tokens < dllm_block_size,
        # the diffusion unmask process may have some problems
        # Make sure at least one page is available
        trunc_len = (
            min(self.rem_dllm_tokens, self.dllm_block_size)
            // self.page_size
            * self.page_size
        )

        req.extend_input_len = trunc_len
        req.fill_ids = req.fill_ids[: prefix_len + trunc_len]

        self.can_run_list.append(req)

        self._update_prefill_budget(prefix_len, trunc_len, 0)
```
**EN:** This block implements the method `_add_dllm_req(req, prefix_len)` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `_add_dllm_req`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `_add_dllm_req(req, prefix_len)`。它围绕 `_add_dllm_req` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 635-638: Implement req inc lock ref / 实现req inc lock ref
```python
    def _req_inc_lock_ref(self, req: Req):
        result = self.tree_cache.inc_lock_ref(req.last_node)
        if self.is_hybrid_swa:
            req.swa_uuid_for_lock = result.swa_uuid_for_lock
```
**EN:** This block implements the method `_req_inc_lock_ref(req)` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `_req_inc_lock_ref`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `_req_inc_lock_ref(req)`。它围绕 `_req_inc_lock_ref` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 640-666: Implement add dllm staging req / 实现add dllm staging req
```python
    def add_dllm_staging_req(self, req: Req):
        assert self.dllm_config is not None
        _rem_tokens = self._get_dllm_remain_tokens()

        if _rem_tokens <= 0:
            return AddReqResult.NO_TOKEN

        # Truncate input length to available tokens and update request metadata
        truncated = req.extend_input_len > _rem_tokens
        req.extend_input_len = min(req.extend_input_len, _rem_tokens)
        req.fill_ids = req.fill_ids[: len(req.prefix_indices) + req.extend_input_len]
        self.can_run_list.append(req)

        # Update budget: reserve max_new_tokens only if not truncated
        max_new_tokens = (
            min(req.sampling_params.max_new_tokens, CLIP_MAX_NEW_TOKENS)
            if not truncated
            else 0
        )
        self._update_prefill_budget(0, req.extend_input_len, max_new_tokens)

        # Return based on remaining token availability
        return (
            AddReqResult.NO_TOKEN
            if self._get_dllm_remain_tokens() <= 0
            else AddReqResult.CONTINUE
        )
```
**EN:** This block implements the method `add_dllm_staging_req(req)` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `add_dllm_staging_req`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `add_dllm_staging_req(req)`。它围绕 `add_dllm_staging_req` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 668-701: Implement add chunked req / 实现add chunked req
```python
    def add_chunked_req(self, req: Req):
        if self.dllm_config is not None:
            _rem_tokens = self._get_dllm_remain_tokens()
        else:
            _rem_tokens = min(self.rem_chunk_tokens, int(self.rem_total_tokens))
            if self.is_hybrid_swa:
                # alloc_extend needs extend_num_tokens + page_size per request,
                # so reserve one page here to avoid OOM
                _rem_tokens = min(
                    _rem_tokens, int(self.rem_swa_tokens) - self.page_size
                )
            # The chunked_req must be added to the list; otherwise, it will cause a memory leak.
            # Therefore, in certain cases where _rem_tokens <= 0, it should be replaced with rem_chunk_tokens.
            if _rem_tokens <= 0:
                if self.is_hybrid_swa:
                    return req
                _rem_tokens = self.rem_chunk_tokens

        truncated = req.extend_input_len > _rem_tokens
        req.set_extend_input_len(min(req.extend_input_len, _rem_tokens))
        req.fill_ids = req.fill_ids[: len(req.prefix_indices) + req.extend_input_len]
        self.can_run_list.append(req)
        self._update_prefill_budget(
            0,
            req.extend_input_len,
            (
                min(req.sampling_params.max_new_tokens, CLIP_MAX_NEW_TOKENS)
                if not truncated
                else 0
            ),
        )

        # Return if chunked prefill not finished
        return req if truncated else None
```
**EN:** This block implements the method `add_chunked_req(req)` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `add_chunked_req`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `add_chunked_req(req)`。它围绕 `add_chunked_req` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 702-703: Provide supporting module logic / 提供辅助模块逻辑
```python

    @contextmanager
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 704-718: Implement lock node / 实现lock node
```python
    def _lock_node(self, last_node: TreeNode):
        dec_lock_params = None
        try:
            result = self.tree_cache.inc_lock_ref(last_node)
            if self.tree_cache.is_tree_cache():
                # init_load_back may revive SWA/Mamba tombstones while this
                # temporary admission lock is held. Release must mirror the
                # exact nodes skipped at acquire time.
                dec_lock_params = result.to_dec_params()
            yield None
        finally:
            if dec_lock_params is not None:
                self.tree_cache.dec_lock_ref(last_node, dec_lock_params)
            else:
                self.tree_cache.dec_lock_ref(last_node)
```
**EN:** This block implements the method `_lock_node(last_node)` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `_lock_node`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `_lock_node(last_node)`。它围绕 `_lock_node` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 720-755: Implement add one req ignore eos / 实现add one req ignore eos
```python
    def add_one_req_ignore_eos(self, req: Req):
        paged_input = self.ceil_paged_tokens(req.extend_input_len)
        if paged_input > min(self.cur_rem_tokens, self.rem_total_tokens):
            return AddReqResult.NO_TOKEN
        if self.is_hybrid_swa:
            if self._swa_budget_for_req(req.extend_input_len) > self.rem_swa_tokens:
                return AddReqResult.NO_TOKEN

        def add_req_state(r, insert_sort=False):
            new_token_ratio = (
                1.0 if r.sampling_params.ignore_eos else self.new_token_ratio
            )
            tokens_left = r.sampling_params.max_new_tokens * new_token_ratio - len(
                r.output_ids
            )
            tokens_occupied = len(r.origin_input_ids) + len(r.output_ids)

            if tokens_left <= 0:
                return

            if not insert_sort:
                self.req_states.append((tokens_left, tokens_occupied))
            else:
                i = 0
                for i in range(len(self.req_states)):
                    if tokens_left <= self.req_states[i][0]:
                        break
                self.req_states.insert(i, (tokens_left, tokens_occupied))

        if self.req_states is None:
            self.req_states = []
            add_req_state(req)
            if self.running_batch is not None:
                for r in self.running_batch.reqs:
                    add_req_state(r)
            for r in self.can_run_list:
```
**EN:** This block implements the method `add_one_req_ignore_eos(req)` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `add_one_req_ignore_eos`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `add_one_req_ignore_eos(req)`。它围绕 `add_one_req_ignore_eos` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 756-791: Continue add one req ignore eos / 继续说明add one req ignore eos
```python
                add_req_state(r)
            self.req_states.sort(key=lambda x: x[0])
        else:
            add_req_state(req, insert_sort=True)

        if not self.is_hybrid_swa:
            # Skip this logic for swa. The SWA has different memory management, and
            # this mechanism is underestimating the memory usage.
            cur_rem_tokens = self.cur_rem_tokens - self.ceil_paged_tokens(
                req.extend_input_len
            )
            tokens_freed = 0
            for i, (tokens_left, tokens_occupied) in enumerate(self.req_states):
                # tokens_left gives a reservative calculation as the last token is not stored
                bs = len(self.req_states) - i
                min_free_tokens = cur_rem_tokens + tokens_freed - tokens_left * bs
                # reserve tokens for corner cases
                if min_free_tokens <= IGNORE_EOS_RESERVE_TOKENS * bs:
                    return AddReqResult.NO_TOKEN
                tokens_freed += tokens_occupied

        if (self.prefill_delayer_single_pass is not None) and (
            not self.prefill_delayer_single_pass.negotiate_should_allow_prefill(
                local_prefillable=True
            )
        ):
            return AddReqResult.OTHER

        if self.dllm_config is not None:
            if self.rem_dllm_tokens <= 0:
                return AddReqResult.OTHER

            self._add_dllm_req(req, 0)
        elif (
            self.rem_chunk_tokens is None  # chunked prefill is disabled
            or req.extend_input_len <= self.rem_chunk_tokens  # it is the last chunk
```
**EN:** This block implements the method `add_one_req_ignore_eos(req)` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `add_one_req_ignore_eos`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `add_one_req_ignore_eos(req)`。它围绕 `add_one_req_ignore_eos` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 792-813: Continue add one req ignore eos / 继续说明add one req ignore eos
```python
        ):
            # Non-chunked prefill
            self.can_run_list.append(req)
            self._update_prefill_budget(
                0,
                req.extend_input_len,
                min(req.sampling_params.max_new_tokens, CLIP_MAX_NEW_TOKENS),
            )
        else:
            if self.rem_chunk_tokens <= 0:
                return AddReqResult.OTHER

            # Chunked prefill
            trunc_len = self.rem_chunk_tokens

            req.set_extend_input_len(trunc_len)
            req.fill_ids = req.fill_ids[:trunc_len]
            self.can_run_list.append(req)
            self.new_chunked_req = req
            self._update_prefill_budget(0, trunc_len, 0)

        return self.budget_state()
```
**EN:** This block implements the method `add_one_req_ignore_eos(req)` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `add_one_req_ignore_eos`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `add_one_req_ignore_eos(req)`。它围绕 `add_one_req_ignore_eos` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 815-850: Implement add one req / 实现add one req
```python
    def add_one_req(
        self, req: Req, has_chunked_req: bool, truncation_align_size: Optional[int]
    ):
        if (self.prefill_delayer_single_pass is not None) and (
            not self.prefill_delayer_single_pass.negotiate_should_allow_prefill(
                local_prefillable=True,
                running_batch=self.running_batch.batch_size(),
                max_prefill_bs=self.max_prefill_bs,
                max_running_requests=self.max_running_requests,
                waiting_queue_len=self.waiting_queue_len,
            )
        ):
            return AddReqResult.OTHER
        # TODO support cp with multiple requests
        # Enabling context parallelism currently presents precision issues;
        # therefore, the prefill-batch setting is temporarily set to 1.
        if (
            self.nsa_prefill_cp_in_seq_split or self.prefill_context_parallel_enabled
        ) and len(self.can_run_list) >= 1:
            return AddReqResult.OTHER

        if (x := self.prefill_max_requests) is not None and len(self.can_run_list) >= x:
            return AddReqResult.OTHER

        if req.sampling_params.ignore_eos and getattr(self.tree_cache, "disable", True):
            return self.add_one_req_ignore_eos(req)

        # Reserve page_size for page-alignment overhead. The paged allocator
        # may consume up to one extra page per request (see alloc_extend), and
        # _update_prefill_budget already accounts for this in the deduction.
        # Without this, admission is more optimistic than the actual budget
        # deduction, allowing over-admission when the pool is nearly full.
        max_new = min(
            max(req.sampling_params.max_new_tokens - len(req.output_ids), 0),
            CLIP_MAX_NEW_TOKENS,
        )
```
**EN:** This block implements the method `add_one_req(req, has_chunked_req, truncation_align_size)` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `add_one_req`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `add_one_req(req, has_chunked_req, truncation_align_size)`。它围绕 `add_one_req` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 851-886: Continue add one req / 继续说明add one req
```python
        total_tokens = req.extend_input_len + max_new + self.page_size

        # adjusting the input_tokens based on host_hit_length and page_size
        real_input_tokens = req.extend_input_len - req.host_hit_length
        real_input_tokens = self.ceil_paged_tokens(real_input_tokens)
        prefix_len = len(req.prefix_indices)

        if total_tokens >= self.rem_total_tokens:
            return AddReqResult.NO_TOKEN

        if self.is_hybrid_swa:
            swa_needed = self._swa_budget_for_req(req.extend_input_len)
            if swa_needed >= self.rem_swa_tokens:
                return AddReqResult.NO_TOKEN

        if real_input_tokens >= self.rem_input_tokens and len(self.can_run_list) != 0:
            return AddReqResult.OTHER

        with self._lock_node(req.last_node):
            # self.rem_total_tokens may decrease after the lock acquisition
            if total_tokens >= self.rem_total_tokens:
                return AddReqResult.NO_TOKEN

            if self.is_hybrid_swa:
                swa_needed = self._swa_budget_for_req(req.extend_input_len)
                if swa_needed >= self.rem_swa_tokens:
                    return AddReqResult.NO_TOKEN

            if req.host_hit_length > 0:
                new_indices, req.last_node = self.tree_cache.init_load_back(
                    InitLoadBackParams(
                        best_match_node=req.best_match_node,
                        host_hit_length=req.host_hit_length,
                        req=req,
                    )
                )
```
**EN:** This block implements the method `add_one_req(req, has_chunked_req, truncation_align_size)` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `add_one_req`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `add_one_req(req, has_chunked_req, truncation_align_size)`。它围绕 `add_one_req` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 887-922: Continue add one req / 继续说明add one req
```python
                req.prefix_indices = torch.cat([req.prefix_indices, new_indices])
                req.set_extend_input_len(len(req.fill_ids) - len(req.prefix_indices))
                prefix_len = len(req.prefix_indices)
                req.cache_protected_len = prefix_len

            input_tokens = self.ceil_paged_tokens(req.extend_input_len)

            if input_tokens >= self.rem_input_tokens and len(self.can_run_list) != 0:
                return AddReqResult.OTHER

            if self.dllm_config is not None:
                if self.rem_dllm_tokens <= 0:
                    return AddReqResult.OTHER

                assert (
                    truncation_align_size is None
                ), "truncation_align_size is not supported for dllm prefill"

                self._add_dllm_req(req, prefix_len)
                self._req_inc_lock_ref(req)
            elif self.rem_chunk_tokens is None or input_tokens <= self.rem_chunk_tokens:
                # Non-chunked prefill
                self.can_run_list.append(req)

                self._req_inc_lock_ref(req)
                self._update_prefill_budget(
                    prefix_len,
                    input_tokens,
                    min(
                        req.sampling_params.max_new_tokens,
                        CLIP_MAX_NEW_TOKENS,
                    ),
                )
            else:
                # Make sure at least one page is available
                trunc_len = self.rem_chunk_tokens // self.page_size * self.page_size
```
**EN:** This block implements the method `add_one_req(req, has_chunked_req, truncation_align_size)` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `add_one_req`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `add_one_req(req, has_chunked_req, truncation_align_size)`。它围绕 `add_one_req` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 923-955: Continue add one req / 继续说明add one req
```python

                if trunc_len <= 0:
                    return AddReqResult.OTHER

                # When truncation align size is set, we want to assert that the prefill prefix length is multiple of truncation align size
                # A typical use case is when deterministic inference is enabled with flashinfer attention backend,
                # we need the prefill prefix length to be multiple of attention split size
                if truncation_align_size is not None:
                    if trunc_len < truncation_align_size:
                        return AddReqResult.OTHER
                    else:
                        trunc_len = truncation_align_size * (
                            trunc_len // truncation_align_size
                        )

                now_input_len = trunc_len + len(req.prefix_indices)
                now_input_len = now_input_len // self.page_size * self.page_size
                trunc_len = now_input_len - len(req.prefix_indices)

                if trunc_len <= 0:
                    return AddReqResult.OTHER

                # Chunked prefill
                req.set_extend_input_len(trunc_len)
                req.fill_ids = req.fill_ids[: len(req.prefix_indices) + trunc_len]

                self.can_run_list.append(req)
                self.new_chunked_req = req

                self._req_inc_lock_ref(req)
                self._update_prefill_budget(prefix_len, trunc_len, 0)

        return self.budget_state()
```
**EN:** This block implements the method `add_one_req(req, has_chunked_req, truncation_align_size)` on `PrefillAdder`. It focuses on handling the schedule policy responsibilities represented by `add_one_req`, so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `add_one_req(req, has_chunked_req, truncation_align_size)`。它围绕 `add_one_req` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 957-992: Implement preempt to schedule / 实现preempt to 调度
```python
    def preempt_to_schedule(self, req: Req, server_args: ServerArgs) -> bool:
        """
        Preempt running requests to serve the new request if the priority threshold is met and token count sum is verified.
        Returns True if preemption was committed, and the new request can be scheduled.
        """
        # Iterate running requests to find preemptible requests
        priority_sign = 1 if server_args.schedule_low_priority_values_first else -1

        # NOTE: A request finishes in two phases:
        #   1) check_finished + release_kv_cache  (in process_batch_result)
        #   2) filter out of batch                (in get_next_batch_to_run / update_running_batch)
        # Preemption runs between these two phases (inside get_new_batch_prefill),
        # so running_batch may still contain requests whose KV cache is already freed.
        # We must skip them here to avoid a double-free on release_req.
        valid_running_reqs = (
            r
            for r in self.running_batch.reqs
            if r not in self.preempt_list and not r.finished()
        )

        sorted_valid_running_reqs = sorted(
            valid_running_reqs,
            key=lambda x: (
                x.priority * (-priority_sign),
                -x.time_stats.wait_queue_entry_time,
            ),
        )

        preemptible_reqs = []
        min_tokens_to_remove = (
            req.extend_input_len
            + min(req.sampling_params.max_new_tokens, CLIP_MAX_NEW_TOKENS)
            - self.rem_total_tokens
        )
        for running_req in sorted_valid_running_reqs:
            # Priority difference needs to meet the threshold to be preemptible.
```
**EN:** This block implements the method `preempt_to_schedule(req, server_args)` on `PrefillAdder`. It focuses on Preempt running requests to serve the new request if the priority threshold is met and token count sum is verified., so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `preempt_to_schedule(req, server_args)`。它围绕 `preempt_to_schedule` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

### Lines 993-1026: Continue preempt to schedule / 继续说明preempt to 调度
```python
            priority_diff = (req.priority - running_req.priority) * (-priority_sign)

            if priority_diff > self.priority_scheduling_preemption_threshold:
                preemptible_reqs.append(running_req)
                min_tokens_to_remove -= self._get_running_request_total_token_offset(
                    running_req
                )
                if min_tokens_to_remove <= 0:
                    break
            else:
                break

        # Check max token count limit can be met
        if len(preemptible_reqs) == 0 or min_tokens_to_remove > 0:
            return False

        # Preempt running requests. Release allocated resources for immediate usage.
        preemptible_reqs = set(preemptible_reqs)
        keep_indices = []
        release_counter = 0
        for i, running_req in enumerate(self.running_batch.reqs):
            if running_req in preemptible_reqs:
                self.rem_total_token_offset -= (
                    self._get_running_request_total_token_offset(running_req)
                )
                release_counter += 1
                self.running_batch.release_req(
                    i, len(self.running_batch.reqs) - release_counter, server_args
                )
            else:
                keep_indices.append(i)
        self.running_batch.filter_batch(keep_indices=keep_indices)
        self.preempt_list.extend(preemptible_reqs)
        return True
```
**EN:** This block implements the method `preempt_to_schedule(req, server_args)` on `PrefillAdder`. It focuses on Preempt running requests to serve the new request if the priority threshold is met and token count sum is verified., so the class can advance the schedule policy workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillAdder` 上的方法 `preempt_to_schedule(req, server_args)`。它围绕 `preempt_to_schedule` 所承担的 调度 policy 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: CacheAwarePolicy, CacheAgnosticPolicy, SchedulePolicy, AddReqResult, PrefillAdder
- **Main callables / 主要可调用对象**: match_prefix_for_req
- **Domain focus / 领域焦点**: schedule policy / 调度 policy
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: collections, contextlib, enum, logging, os, typing
- **Third-party / 第三方库**: __future__, random, torch
- **Local Modules / 本地模块**: sglang.srt.dllm.config, sglang.srt.environ, sglang.srt.layers.attention.nsa.utils, sglang.srt.layers.utils.cp_utils, sglang.srt.managers.prefill_delayer, sglang.srt.managers.schedule_batch, sglang.srt.mem_cache.allocator, sglang.srt.mem_cache.base_prefix_cache, sglang.srt.mem_cache.hisparse_memory_pool, sglang.srt.mem_cache.radix_cache, sglang.srt.mem_cache.swa_memory_pool, sglang.srt.server_args
