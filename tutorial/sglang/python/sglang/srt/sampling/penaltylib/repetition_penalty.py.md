# repetition_penalty.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/sampling/penaltylib/repetition_penalty.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the sampling and penalty control part of the SRT runtime and implements logic centered on `repetition_penalty`. It exposes primary entry points such as `apply_scaling_penalties`, `BatchedRepetitionPenalizer`. / 该模块属于 SRT 运行时的采样与惩罚控制部分，主要实现围绕 `repetition_penalty` 的逻辑。 它对外提供的主要入口包括 `apply_scaling_penalties`, `BatchedRepetitionPenalizer`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Module imports, constants, and setup
```python
import torch

from sglang.srt.sampling.penaltylib.orchestrator import _BatchedPenalizer
from sglang.srt.utils import get_compiler_backend


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; prepares compilation-related behavior; adjusts sampling behavior and decoding controls.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；处理与编译相关的行为；调整采样行为与解码控制。

### Lines 7-13: Function apply_scaling_penalties
```python
@torch.compile(dynamic=True, backend=get_compiler_backend())
def apply_scaling_penalties(logits, scaling_penalties):
    logits[:] = torch.where(
        logits < 0,
        logits * scaling_penalties,
        logits / scaling_penalties,
    )
```
**EN:** This callable implements `apply_scaling_penalties`. It takes `logits`, `scaling_penalties` and mainly implements apply scaling penalties. In this range it prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `apply_scaling_penalties`。它接收 `logits`, `scaling_penalties`，主要用于实现 apply scaling penalties 相关逻辑。 在这一范围内，它会处理与编译相关的行为。

### Lines 16-22: Class BatchedRepetitionPenalizer
```python
class BatchedRepetitionPenalizer(_BatchedPenalizer):
    """
    Repetition penalizer penalizes tokens based on their presence in the generated output.
    """

    is_multiplicative: bool = True

```
**EN:** This range introduces `BatchedRepetitionPenalizer` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Repetition penalizer penalizes tokens based on their presence in the generated output."
**CN:** 这一段引入 `BatchedRepetitionPenalizer`，并定义其后续方法依赖的结构或元数据。

### Lines 23-27: Method BatchedRepetitionPenalizer._is_required
```python
    def _is_required(self) -> bool:
        return any(
            req.sampling_params.repetition_penalty != 1.0
            for req in self.orchestrator.reqs()
        )
```
**EN:** This callable implements `BatchedRepetitionPenalizer._is_required` and mainly implements is required. In this range it adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `BatchedRepetitionPenalizer._is_required`，主要用于实现 is required 相关逻辑。 在这一范围内，它会调整采样行为与解码控制。

### Lines 29-44: Method BatchedRepetitionPenalizer._prepare
```python
    def _prepare(self):
        self.cumulated_repetition_penalties = torch.ones(
            (len(self.orchestrator.reqs()), self.orchestrator.vocab_size),
            dtype=torch.float32,
            device=self.orchestrator.device,
        )
        self.repetition_penalties = (
            torch.tensor(
                data=[
                    req.sampling_params.repetition_penalty
                    for req in self.orchestrator.reqs()
                ],
                dtype=torch.float32,
                device=self.orchestrator.device,
            )
        ).unsqueeze_(1)
```
**EN:** This callable implements `BatchedRepetitionPenalizer._prepare` and mainly prepares runtime inputs. In this range it adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `BatchedRepetitionPenalizer._prepare`，主要用于准备运行时输入。 在这一范围内，它会调整采样行为与解码控制。

### Lines 46-51: Method BatchedRepetitionPenalizer._cumulate_output_tokens
```python
    def _cumulate_output_tokens(self, output_ids: torch.Tensor):
        self.cumulated_repetition_penalties.scatter_(
            dim=1,
            index=output_ids.unsqueeze(1),
            src=self.repetition_penalties,
        )
```
**EN:** This callable implements `BatchedRepetitionPenalizer._cumulate_output_tokens`. It takes `output_ids` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `BatchedRepetitionPenalizer._cumulate_output_tokens`。它接收 `output_ids`，主要用于将数据转换为另一种表示。

### Lines 53-55: Method BatchedRepetitionPenalizer._apply
```python
    def _apply(self, logits: torch.Tensor) -> torch.Tensor:
        apply_scaling_penalties(logits, self.cumulated_repetition_penalties)
        return logits
```
**EN:** This callable implements `BatchedRepetitionPenalizer._apply`. It takes `logits` and mainly implements apply.
**CN:** 这一可调用对象实现了 `BatchedRepetitionPenalizer._apply`。它接收 `logits`，主要用于实现 apply 相关逻辑。

### Lines 57-58: Method BatchedRepetitionPenalizer.get_scaling_penalties
```python
    def get_scaling_penalties(self) -> torch.Tensor:
        return self.cumulated_repetition_penalties
```
**EN:** This callable implements `BatchedRepetitionPenalizer.get_scaling_penalties` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `BatchedRepetitionPenalizer.get_scaling_penalties`，主要用于获取某个值或派生视图。

### Lines 60-64: Method BatchedRepetitionPenalizer._filter
```python
    def _filter(self, keep_indices: torch.Tensor):
        self.repetition_penalties = self.repetition_penalties[keep_indices]
        self.cumulated_repetition_penalties = self.cumulated_repetition_penalties[
            keep_indices
        ]
```
**EN:** This callable implements `BatchedRepetitionPenalizer._filter`. It takes `keep_indices` and mainly implements filter.
**CN:** 这一可调用对象实现了 `BatchedRepetitionPenalizer._filter`。它接收 `keep_indices`，主要用于实现 filter 相关逻辑。

### Lines 66-73: Method BatchedRepetitionPenalizer._merge
```python
    def _merge(self, their: "BatchedRepetitionPenalizer"):
        self.repetition_penalties = torch.cat(
            [self.repetition_penalties, their.repetition_penalties], dim=0
        )
        self.cumulated_repetition_penalties = torch.cat(
            [self.cumulated_repetition_penalties, their.cumulated_repetition_penalties],
            dim=0,
        )
```
**EN:** This callable implements `BatchedRepetitionPenalizer._merge`. It takes `their` and mainly merges related state.
**CN:** 这一可调用对象实现了 `BatchedRepetitionPenalizer._merge`。它接收 `their`，主要用于合并相关状态。

### Lines 75-78: Method BatchedRepetitionPenalizer._teardown
```python
    def _teardown(self) -> None:
        for name in ("repetition_penalties", "cumulated_repetition_penalties"):
            if hasattr(self, name):
                delattr(self, name)
```
**EN:** This callable implements `BatchedRepetitionPenalizer._teardown` and mainly implements teardown.
**CN:** 这一可调用对象实现了 `BatchedRepetitionPenalizer._teardown`，主要用于实现 teardown 相关逻辑。

## Key Concepts / 关键概念
- `apply_scaling_penalties`: implements apply scaling penalties / 实现 apply scaling penalties 相关逻辑
- `BatchedRepetitionPenalizer`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.sampling.penaltylib.orchestrator`, `sglang.srt.utils`
