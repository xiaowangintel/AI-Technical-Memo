# sampling_batch_info.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/sampling/sampling_batch_info.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the sampling and penalty control part of the SRT runtime and implements logic centered on `sampling_batch_info`. It exposes primary entry points such as `SamplingBatchInfo`, `merge_bias_tensor`. / 该模块属于 SRT 运行时的采样与惩罚控制部分，主要实现围绕 `sampling_batch_info` 的逻辑。 它对外提供的主要入口包括 `SamplingBatchInfo`, `merge_bias_tensor`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Module imports, constants, and setup
```python
from __future__ import annotations

import dataclasses
import logging
from typing import TYPE_CHECKING, Any, Callable, Dict, List, Optional, Tuple

import torch

import sglang.srt.sampling.penaltylib as penaltylib
from sglang.srt.sampling.custom_logit_processor import CustomLogitProcessor
from sglang.srt.sampling.penaltylib.repetition_penalty import apply_scaling_penalties
from sglang.srt.sampling.sampling_params import TOP_K_ALL
from sglang.srt.server_args import get_global_server_args

if TYPE_CHECKING:
    from sglang.srt.managers.schedule_batch import ScheduleBatch


logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; adjusts sampling behavior and decoding controls.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；调整采样行为与解码控制。

### Lines 22-72: Class SamplingBatchInfo
```python
@dataclasses.dataclass
class SamplingBatchInfo:
    # Basic batched sampling params
    temperatures: torch.Tensor
    top_ps: torch.Tensor
    top_ks: torch.Tensor
    min_ps: torch.Tensor

    # Whether all requests use greedy sampling
    is_all_greedy: bool

    # Whether any requests use top_p sampling
    need_top_p_sampling: bool

    # Whether any requests use top_k sampling
    need_top_k_sampling: bool

    # Whether any request needs min_p sampling
    need_min_p_sampling: bool

    # Masking tensors for grammar-guided structured outputs
    vocab_size: int
    grammars: Optional[List] = None
    vocab_mask: Optional[torch.Tensor] = None
    apply_mask_func: Optional[Callable[[torch.Tensor, torch.Tensor], None]] = None

    # Penalizer
    penalizer_orchestrator: Optional[penaltylib.BatchedPenalizerOrchestrator] = None
    acc_additive_penalties: Optional[torch.Tensor] = None  # Used in the overlap mode
    acc_scaling_penalties: Optional[torch.Tensor] = (
        None  # Used in the overlap mode for repetition penalty
    )

    # Whether any request has custom logit processor
    has_custom_logit_processor: bool = False
    # Custom parameters
    custom_params: Optional[List[Optional[Dict[str, Any]]]] = None
    # Custom logit processor
    custom_logit_processor: Optional[
        Dict[int, Tuple[CustomLogitProcessor, torch.Tensor]]
    ] = None

    # Used for deterministic sampling
    sampling_seed: Optional[torch.Tensor] = None

    # Device
    device: str = "cuda"

    # Handle logit bias
    logit_bias: Optional[torch.Tensor] = None

```
**EN:** This range introduces `SamplingBatchInfo` and defines the structure or metadata that its methods rely on. In this range it handles grammar or regular-expression constraints; adjusts sampling behavior and decoding controls.
**CN:** 这一段引入 `SamplingBatchInfo`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理语法或正则约束；调整采样行为与解码控制。

### Lines 73-132: Method SamplingBatchInfo.from_schedule_batch (part 1/2)
```python
    @classmethod
    def from_schedule_batch(cls, batch: ScheduleBatch, vocab_size: int):
        global_server_args = get_global_server_args()
        enable_deterministic = global_server_args.enable_deterministic_inference

        reqs = batch.reqs
        device = batch.device
        temperatures = torch.tensor(
            [r.sampling_params.temperature for r in reqs],
            dtype=torch.float,
            device=device,
        ).view(-1, 1)
        top_ps = torch.tensor(
            [r.sampling_params.top_p for r in reqs], dtype=torch.float, device=device
        )
        top_ks = torch.tensor(
            [r.sampling_params.top_k for r in reqs], dtype=torch.int32, device=device
        )
        min_ps = torch.tensor(
            [r.sampling_params.min_p for r in reqs], dtype=torch.float, device=device
        )
        sampling_seed = (
            torch.tensor(
                [
                    (
                        r.sampling_params.sampling_seed
                        if r.sampling_params.sampling_seed is not None
                        else 42
                    )
                    for r in reqs
                ],
                dtype=torch.int64,
                device=device,
            )
            if enable_deterministic
            else None
        )

        logit_bias = None
        if any(r.sampling_params.logit_bias is not None for r in reqs):
            logit_bias = torch.zeros(len(reqs), vocab_size, device=device)
            for i, r in enumerate(reqs):
                if r.sampling_params.logit_bias is not None:
                    for key, value in r.sampling_params.logit_bias.items():
                        logit_bias[i, int(key)] = value

        # Check if any request has custom logit processor
        has_custom_logit_processor = (
            global_server_args.enable_custom_logit_processor
            and any(r.custom_logit_processor for r in reqs)  # check the flag first.
        )  # then check the requests.

        if has_custom_logit_processor:
            # Merge the same type of custom logit processors together
            processor_dict = {}
            for i, r in enumerate(reqs):
                if r.custom_logit_processor is None:
                    continue
                processor_str = r.custom_logit_processor
                if processor_str not in processor_dict:
```
**EN:** This callable implements `SamplingBatchInfo.from_schedule_batch`. It takes `cls`, `batch`, `vocab_size` and mainly constructs data from an external representation. This chunk is part 1 of 2 for the same logical block. In this range it adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `SamplingBatchInfo.from_schedule_batch`。它接收 `cls`, `batch`, `vocab_size`，主要用于从外部表示构造数据。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会调整采样行为与解码控制。

### Lines 133-189: Method SamplingBatchInfo.from_schedule_batch (part 2/2)
```python
                    processor_dict[processor_str] = []
                processor_dict[processor_str].append(i)

            merged_custom_logit_processor = {
                hash(processor_str): (
                    # The deserialized custom logit processor object
                    CustomLogitProcessor.from_str(processor_str),
                    # The mask tensor for the requests that use this custom logit processor
                    torch.zeros(len(reqs), dtype=torch.bool)
                    .scatter_(0, torch.tensor(true_indices), True)
                    .to(device, non_blocking=True),
                )
                for processor_str, true_indices in processor_dict.items()
            }
            custom_params = [r.sampling_params.custom_params for r in reqs]
        else:
            merged_custom_logit_processor = None
            custom_params = None

        # Each penalizers will do nothing if they evaluate themselves as not required by looking at
        # the sampling_params of the requests (See {_is_required()} of each penalizers). So this
        # should not add hefty computation overhead other than simple checks.
        #
        # While we can choose not to even create the class instances if they are not required, this
        # could add additional complexity to the {ScheduleBatch} class, especially we need to
        # handle {filter_batch()} and {merge_batch()} cases as well.
        penalizer_orchestrator = penaltylib.BatchedPenalizerOrchestrator(
            vocab_size=vocab_size,
            batch=batch,
            penalizers={
                penaltylib.BatchedFrequencyPenalizer,
                penaltylib.BatchedMinNewTokensPenalizer,
                penaltylib.BatchedPresencePenalizer,
                penaltylib.BatchedRepetitionPenalizer,
            },
        )

        ret = cls(
            temperatures=temperatures,
            top_ps=top_ps,
            top_ks=top_ks,
            min_ps=min_ps,
            sampling_seed=sampling_seed,
            is_all_greedy=all(r.sampling_params.top_k <= 1 for r in reqs),
            need_top_p_sampling=any(r.sampling_params.top_p != 1.0 for r in reqs),
            need_top_k_sampling=any(r.sampling_params.top_k != TOP_K_ALL for r in reqs),
            need_min_p_sampling=any(r.sampling_params.min_p > 0 for r in reqs),
            vocab_size=vocab_size,
            penalizer_orchestrator=penalizer_orchestrator,
            has_custom_logit_processor=has_custom_logit_processor,
            custom_params=custom_params,
            custom_logit_processor=merged_custom_logit_processor,
            device=device,
            logit_bias=logit_bias,
        )
        ret.adjusted_from_schedule_batch(batch, vocab_size)
        return ret
```
**EN:** This callable implements `SamplingBatchInfo.from_schedule_batch`. It takes `cls`, `batch`, `vocab_size` and mainly constructs data from an external representation. This chunk is part 2 of 2 for the same logical block. In this range it adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `SamplingBatchInfo.from_schedule_batch`。它接收 `cls`, `batch`, `vocab_size`，主要用于从外部表示构造数据。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会调整采样行为与解码控制。

### Lines 190-191: Class-level scaffolding for SamplingBatchInfo
```python

    # placeholder for override
```
**EN:** This callable implements `None.SamplingBatchInfo` and mainly implements Sampling Batch Info.
**CN:** 这一可调用对象实现了 `None.SamplingBatchInfo`，主要用于实现 Sampling Batch Info 相关逻辑。

### Lines 192-193: Method SamplingBatchInfo.adjusted_from_schedule_batch
```python
    def adjusted_from_schedule_batch(self, batch: ScheduleBatch, vocab_size: int):
        pass
```
**EN:** This callable implements `SamplingBatchInfo.adjusted_from_schedule_batch`. It takes `batch`, `vocab_size` and mainly constructs data from an external representation.
**CN:** 这一可调用对象实现了 `SamplingBatchInfo.adjusted_from_schedule_batch`。它接收 `batch`, `vocab_size`，主要用于从外部表示构造数据。

### Lines 194-195: Class-level scaffolding for SamplingBatchInfo
```python

    # placeholder for override
```
**EN:** This callable implements `None.SamplingBatchInfo` and mainly implements Sampling Batch Info.
**CN:** 这一可调用对象实现了 `None.SamplingBatchInfo`，主要用于实现 Sampling Batch Info 相关逻辑。

### Lines 196-197: Method SamplingBatchInfo.adjusted_merge_batch
```python
    def adjusted_merge_batch(self, other: "SamplingBatchInfo"):
        pass
```
**EN:** This callable implements `SamplingBatchInfo.adjusted_merge_batch`. It takes `other` and mainly merges related state.
**CN:** 这一可调用对象实现了 `SamplingBatchInfo.adjusted_merge_batch`。它接收 `other`，主要用于合并相关状态。

### Lines 198-199: Class-level scaffolding for SamplingBatchInfo
```python

    # placeholder for override
```
**EN:** This callable implements `None.SamplingBatchInfo` and mainly implements Sampling Batch Info.
**CN:** 这一可调用对象实现了 `None.SamplingBatchInfo`，主要用于实现 Sampling Batch Info 相关逻辑。

### Lines 200-203: Method SamplingBatchInfo.adjusted_filter_batch
```python
    def adjusted_filter_batch(
        self, keep_indices: List[int], keep_indices_device: torch.Tensor
    ):
        pass
```
**EN:** This callable implements `SamplingBatchInfo.adjusted_filter_batch`. It takes `keep_indices`, `keep_indices_device` and mainly implements adjusted filter batch.
**CN:** 这一可调用对象实现了 `SamplingBatchInfo.adjusted_filter_batch`。它接收 `keep_indices`, `keep_indices_device`，主要用于实现 adjusted filter batch 相关逻辑。

### Lines 205-206: Method SamplingBatchInfo.__len__
```python
    def __len__(self):
        return len(self.temperatures)
```
**EN:** This callable implements `SamplingBatchInfo.__len__` and mainly implements len. In this range it adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `SamplingBatchInfo.__len__`，主要用于实现 len 相关逻辑。 在这一范围内，它会调整采样行为与解码控制。

### Lines 208-233: Method SamplingBatchInfo.update_regex_vocab_mask
```python
    def update_regex_vocab_mask(self):
        if not self.grammars:
            self.vocab_mask = None
            self.apply_mask_func = None
            return

        # Find a grammar from the list
        first_grammar = next(grammar for grammar in self.grammars if grammar)

        # TODO(lianmin): Maybe we can reuse the existing mask?
        self.vocab_mask = first_grammar.allocate_vocab_mask(
            vocab_size=self.vocab_size,
            batch_size=len(self.temperatures),
            device=self.device,
        )
        self.apply_mask_func = (
            first_grammar.apply_vocab_mask
        )  # force to use static method

        # Apply the mask
        for i, grammar in enumerate(self.grammars):
            if grammar and not grammar.finished and not grammar.is_terminated():
                grammar.fill_vocab_mask(self.vocab_mask, i)

        # Move the mask to the device if needed
        self.vocab_mask = first_grammar.move_vocab_mask(self.vocab_mask, self.device)
```
**EN:** This callable implements `SamplingBatchInfo.update_regex_vocab_mask` and mainly updates existing runtime state. In this range it sets up imports and shared symbols; handles grammar or regular-expression constraints; adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `SamplingBatchInfo.update_regex_vocab_mask`，主要用于更新现有运行时状态。 在这一范围内，它会建立导入关系并准备共享符号；处理语法或正则约束；调整采样行为与解码控制。

### Lines 235-250: Method SamplingBatchInfo.update_penalties
```python
    def update_penalties(self):
        if self.penalizer_orchestrator.is_required:
            self.acc_additive_penalties = torch.zeros(
                (len(self.temperatures), self.vocab_size),
                dtype=torch.float32,
                device=self.temperatures.device,
            )
            self.penalizer_orchestrator.accumulate_additive_penalties(
                self.acc_additive_penalties
            )
            self.acc_scaling_penalties = (
                self.penalizer_orchestrator.accumulate_scaling_penalties()
            )
        else:
            self.acc_additive_penalties = None
            self.acc_scaling_penalties = None
```
**EN:** This callable implements `SamplingBatchInfo.update_penalties` and mainly updates existing runtime state. In this range it adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `SamplingBatchInfo.update_penalties`，主要用于更新现有运行时状态。 在这一范围内，它会调整采样行为与解码控制。

### Lines 252-269: Method SamplingBatchInfo.apply_logits_bias
```python
    def apply_logits_bias(self, logits: torch.Tensor):
        if self.acc_additive_penalties is not None:
            # Used in the overlap mode
            logits.add_(self.acc_additive_penalties)

        if self.acc_scaling_penalties is not None:
            # Used in the overlap mode
            apply_scaling_penalties(logits, self.acc_scaling_penalties)

        if self.penalizer_orchestrator and self.penalizer_orchestrator.is_required:
            # Used in the non-overlap mode
            self.penalizer_orchestrator.apply(logits)

        if self.vocab_mask is not None:
            self.apply_mask_func(logits=logits, vocab_mask=self.vocab_mask)

        if self.logit_bias is not None:
            logits.add_(self.logit_bias)
```
**EN:** This callable implements `SamplingBatchInfo.apply_logits_bias`. It takes `logits` and mainly implements apply logits bias.
**CN:** 这一可调用对象实现了 `SamplingBatchInfo.apply_logits_bias`。它接收 `logits`，主要用于实现 apply logits bias 相关逻辑。

### Lines 271-291: Method SamplingBatchInfo.filter_batch
```python
    def filter_batch(self, keep_indices: List[int], keep_indices_device: torch.Tensor):
        self.penalizer_orchestrator.filter(keep_indices_device)

        if self.has_custom_logit_processor:
            self._filter_batch_custom_logit_processor(keep_indices, keep_indices_device)

        for item in [
            "temperatures",
            "top_ps",
            "top_ks",
            "min_ps",
            "sampling_seed",
        ]:
            value = getattr(self, item, None)
            if value is not None:
                setattr(self, item, value[keep_indices_device])

        if self.logit_bias is not None:
            self.logit_bias = self.logit_bias[keep_indices_device]

        self.adjusted_filter_batch(keep_indices, keep_indices_device)
```
**EN:** This callable implements `SamplingBatchInfo.filter_batch`. It takes `keep_indices`, `keep_indices_device` and mainly implements filter batch. In this range it adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `SamplingBatchInfo.filter_batch`。它接收 `keep_indices`, `keep_indices_device`，主要用于实现 filter batch 相关逻辑。 在这一范围内，它会调整采样行为与解码控制。

### Lines 293-311: Method SamplingBatchInfo._filter_batch_custom_logit_processor
```python
    def _filter_batch_custom_logit_processor(
        self, keep_indices: List[int], keep_indices_device: torch.Tensor
    ):
        """Filter the custom logit processor and custom params"""
        self.custom_logit_processor = {
            k: (p, mask[keep_indices_device])
            for k, (p, mask) in self.custom_logit_processor.items()
            if torch.any(
                mask[keep_indices_device]
            )  # ignore the custom logit processor whose mask is all False
        }
        self.custom_params = [self.custom_params[i] for i in keep_indices]

        # If the custom logit processor is an empty dict, set the flag to False,
        # and set the custom logit processor and custom params to None.
        if len(self.custom_logit_processor) == 0:
            self.custom_logit_processor = None
            self.custom_params = None
            self.has_custom_logit_processor = False
```
**EN:** This callable implements `SamplingBatchInfo._filter_batch_custom_logit_processor`. It takes `keep_indices`, `keep_indices_device` and mainly converts data into another representation. The docstring states: "Filter the custom logit processor and custom params"
**CN:** 这一可调用对象实现了 `SamplingBatchInfo._filter_batch_custom_logit_processor`。它接收 `keep_indices`, `keep_indices_device`，主要用于将数据转换为另一种表示。

### Lines 313-351: Method SamplingBatchInfo.merge_custom_logit_processor
```python
    @staticmethod
    def merge_custom_logit_processor(
        lhs: Optional[Dict[int, Tuple[CustomLogitProcessor, torch.Tensor]]],
        rhs: Optional[Dict[int, Tuple[CustomLogitProcessor, torch.Tensor]]],
        bs1: int,
        bs2: int,
        device: str,
    ):
        if lhs is None and rhs is None:
            return None
        lhs, rhs = lhs or {}, rhs or {}

        keys = set(lhs.keys()).union(set(rhs.keys()))
        merged_dict = {}

        for k in keys:
            # Get the logit processor object
            processor = lhs[k][0] if k in lhs else rhs[k][0]
            # Get and merge the mask tensors from the two dicts
            left_mask = (
                lhs[k][1]
                if k in lhs
                else torch.zeros(bs1, dtype=torch.bool, device=device)
            )
            right_mask = (
                rhs[k][1]
                if k in rhs
                else torch.zeros(bs2, dtype=torch.bool, device=device)
            )
            merged_dict[k] = (processor, torch.cat([left_mask, right_mask]))

            assert merged_dict[k][1].shape[0] == bs1 + bs2, (
                f"The batch size of merged mask ({merged_dict[k][1].shape[0]}) does not match "
                f"the sum of the batch sizes of the two masks ({bs1 + bs2})"
                f"\n{left_mask=}\n{right_mask=}\n{bs1=}\n{bs2=}"
                f"\n{lhs=}\n{rhs=}"
            )

        return merged_dict
```
**EN:** This callable implements `SamplingBatchInfo.merge_custom_logit_processor`. It takes `lhs`, `rhs`, `bs1`, `bs2` and mainly converts data into another representation. In this range it sets up imports and shared symbols; performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `SamplingBatchInfo.merge_custom_logit_processor`。它接收 `lhs`, `rhs`, `bs1`, `bs2`，主要用于将数据转换为另一种表示。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查。

### Lines 353-402: Method SamplingBatchInfo.merge_batch
```python
    def merge_batch(self, other: "SamplingBatchInfo"):
        self.penalizer_orchestrator.merge(other.penalizer_orchestrator)

        # Merge the custom logit processors and custom params lists
        if self.has_custom_logit_processor or other.has_custom_logit_processor:
            # Merge the custom logit processors
            self.custom_logit_processor = (
                SamplingBatchInfo.merge_custom_logit_processor(
                    self.custom_logit_processor,
                    other.custom_logit_processor,
                    len(self),
                    len(other),
                    self.device,
                )
            )
            # Merge the custom params lists
            self.custom_params = self.custom_params or [None] * len(self)
            other.custom_params = other.custom_params or [None] * len(other)
            self.custom_params.extend(other.custom_params)

            # Set the flag to True if any of the two has custom logit processor
            self.has_custom_logit_processor = True

        # Merge logit bias - note this has to come before the temperatures tensor update! Otherwise will cause crashes.
        # See note below on len(self) and len(other).
        self.logit_bias = merge_bias_tensor(
            self.logit_bias, other.logit_bias, len(self), len(other), self.device, 0.0
        )

        # Note: because the __len()__ operator is defined on the temperatures tensor,
        # please make sure any merge operation with len(self) or len(other) is done before
        # the merge operation of the temperatures tensor below.
        for item in [
            "temperatures",
            "top_ps",
            "top_ks",
            "min_ps",
            "sampling_seed",
        ]:
            self_val = getattr(self, item, None)
            other_val = getattr(other, item, None)
            if self_val is not None and other_val is not None:
                setattr(self, item, torch.cat([self_val, other_val]))

        self.is_all_greedy &= other.is_all_greedy
        self.need_top_p_sampling |= other.need_top_p_sampling
        self.need_top_k_sampling |= other.need_top_k_sampling
        self.need_min_p_sampling |= other.need_min_p_sampling

        self.adjusted_merge_batch(other)
```
**EN:** This callable implements `SamplingBatchInfo.merge_batch`. It takes `other` and mainly merges related state. In this range it adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `SamplingBatchInfo.merge_batch`。它接收 `other`，主要用于合并相关状态。 在这一范围内，它会调整采样行为与解码控制。

### Lines 404-407: Method SamplingBatchInfo.copy_for_forward
```python
    def copy_for_forward(self):
        # Accumulate the penalty into a pre-allocated buffer to get rid of the dependency of `penalizer_orchestrator` later
        self.update_penalties()
        return dataclasses.replace(self, penalizer_orchestrator=None)
```
**EN:** This callable implements `SamplingBatchInfo.copy_for_forward` and mainly implements copy for forward. In this range it adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `SamplingBatchInfo.copy_for_forward`，主要用于实现 copy for forward 相关逻辑。 在这一范围内，它会调整采样行为与解码控制。

### Lines 410-446: Function merge_bias_tensor
```python
def merge_bias_tensor(
    lhs: Optional[torch.Tensor],
    rhs: Optional[torch.Tensor],
    bs1: int,
    bs2: int,
    device: str,
    default: float,
):
    """Merge two bias tensors for batch merging.

    Args:
        lhs: Left-hand side tensor
        rhs: Right-hand side tensor
        bs1: Batch size of left-hand side tensor
        bs2: Batch size of right-hand side tensor
        device: Device to place the merged tensor on
        default: Default value for missing tensor elements

    Returns:
        Merged tensor or None if both inputs are None
    """
    if lhs is None and rhs is None:
        return None

    if lhs is not None and rhs is not None:
        return torch.cat([lhs, rhs])
    else:
        if lhs is not None:
            shape, dtype = lhs.shape[1:], lhs.dtype
        else:
            shape, dtype = rhs.shape[1:], rhs.dtype

        if lhs is None:
            lhs = torch.empty((bs1, *shape), device=device, dtype=dtype).fill_(default)
        if rhs is None:
            rhs = torch.empty((bs2, *shape), device=device, dtype=dtype).fill_(default)
        return torch.cat([lhs, rhs])
```
**EN:** This callable implements `merge_bias_tensor`. It takes `lhs`, `rhs`, `bs1`, `bs2` and mainly merges related state. The docstring states: "Merge two bias tensors for batch merging."
**CN:** 这一可调用对象实现了 `merge_bias_tensor`。它接收 `lhs`, `rhs`, `bs1`, `bs2`，主要用于合并相关状态。

## Key Concepts / 关键概念
- `SamplingBatchInfo`: core class or state container / 核心类或状态容器
- `merge_bias_tensor`: merges related state / 合并相关状态

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `logging`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.sampling.penaltylib`, `sglang.srt.sampling.custom_logit_processor`, `sglang.srt.sampling.penaltylib.repetition_penalty`, `sglang.srt.sampling.sampling_params`, `sglang.srt.server_args`, `sglang.srt.managers.schedule_batch`
