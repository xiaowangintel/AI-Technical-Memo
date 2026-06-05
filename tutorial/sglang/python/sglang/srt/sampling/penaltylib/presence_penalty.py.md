# presence_penalty.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/sampling/penaltylib/presence_penalty.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the sampling and penalty control part of the SRT runtime and implements logic centered on `presence_penalty`. It exposes primary entry points such as `BatchedPresencePenalizer`. / 该模块属于 SRT 运行时的采样与惩罚控制部分，主要实现围绕 `presence_penalty` 的逻辑。 它对外提供的主要入口包括 `BatchedPresencePenalizer`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Module imports, constants, and setup
```python
import torch

from sglang.srt.sampling.penaltylib.orchestrator import _BatchedPenalizer


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; adjusts sampling behavior and decoding controls.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；调整采样行为与解码控制。

### Lines 6-10: Class BatchedPresencePenalizer
```python
class BatchedPresencePenalizer(_BatchedPenalizer):
    """
    Presence penalizer penalizes tokens based on their presence in the output.
    """

```
**EN:** This range introduces `BatchedPresencePenalizer` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Presence penalizer penalizes tokens based on their presence in the output."
**CN:** 这一段引入 `BatchedPresencePenalizer`，并定义其后续方法依赖的结构或元数据。

### Lines 11-15: Method BatchedPresencePenalizer._is_required
```python
    def _is_required(self) -> bool:
        return any(
            req.sampling_params.presence_penalty != 0.0
            for req in self.orchestrator.reqs()
        )
```
**EN:** This callable implements `BatchedPresencePenalizer._is_required` and mainly implements is required. In this range it adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `BatchedPresencePenalizer._is_required`，主要用于实现 is required 相关逻辑。 在这一范围内，它会调整采样行为与解码控制。

### Lines 17-33: Method BatchedPresencePenalizer._prepare
```python
    def _prepare(self):
        self.cumulated_presence_penalties = torch.zeros(
            (len(self.orchestrator.reqs()), self.orchestrator.vocab_size),
            dtype=torch.float32,
            device=self.orchestrator.device,
        )

        self.presence_penalties = (
            torch.tensor(
                data=[
                    req.sampling_params.presence_penalty
                    for req in self.orchestrator.reqs()
                ],
                dtype=torch.float32,
                device=self.orchestrator.device,
            )
        ).unsqueeze_(1)
```
**EN:** This callable implements `BatchedPresencePenalizer._prepare` and mainly prepares runtime inputs. In this range it adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `BatchedPresencePenalizer._prepare`，主要用于准备运行时输入。 在这一范围内，它会调整采样行为与解码控制。

### Lines 35-40: Method BatchedPresencePenalizer._cumulate_output_tokens
```python
    def _cumulate_output_tokens(self, output_ids: torch.Tensor):
        self.cumulated_presence_penalties.scatter_(
            dim=1,
            index=output_ids.unsqueeze(1),
            src=self.presence_penalties,
        )
```
**EN:** This callable implements `BatchedPresencePenalizer._cumulate_output_tokens`. It takes `output_ids` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `BatchedPresencePenalizer._cumulate_output_tokens`。它接收 `output_ids`，主要用于将数据转换为另一种表示。

### Lines 42-43: Method BatchedPresencePenalizer._apply
```python
    def _apply(self, logits: torch.Tensor) -> torch.Tensor:
        logits.sub_(self.cumulated_presence_penalties)
```
**EN:** This callable implements `BatchedPresencePenalizer._apply`. It takes `logits` and mainly implements apply.
**CN:** 这一可调用对象实现了 `BatchedPresencePenalizer._apply`。它接收 `logits`，主要用于实现 apply 相关逻辑。

### Lines 45-49: Method BatchedPresencePenalizer._filter
```python
    def _filter(self, keep_indices: torch.Tensor):
        self.presence_penalties = self.presence_penalties[keep_indices]
        self.cumulated_presence_penalties = self.cumulated_presence_penalties[
            keep_indices
        ]
```
**EN:** This callable implements `BatchedPresencePenalizer._filter`. It takes `keep_indices` and mainly implements filter.
**CN:** 这一可调用对象实现了 `BatchedPresencePenalizer._filter`。它接收 `keep_indices`，主要用于实现 filter 相关逻辑。

### Lines 51-58: Method BatchedPresencePenalizer._merge
```python
    def _merge(self, their: "BatchedPresencePenalizer"):
        self.presence_penalties = torch.cat(
            [self.presence_penalties, their.presence_penalties], dim=0
        )
        self.cumulated_presence_penalties = torch.cat(
            [self.cumulated_presence_penalties, their.cumulated_presence_penalties],
            dim=0,
        )
```
**EN:** This callable implements `BatchedPresencePenalizer._merge`. It takes `their` and mainly merges related state.
**CN:** 这一可调用对象实现了 `BatchedPresencePenalizer._merge`。它接收 `their`，主要用于合并相关状态。

### Lines 60-63: Method BatchedPresencePenalizer._teardown
```python
    def _teardown(self) -> None:
        for name in ("presence_penalties", "cumulated_presence_penalties"):
            if hasattr(self, name):
                delattr(self, name)
```
**EN:** This callable implements `BatchedPresencePenalizer._teardown` and mainly implements teardown.
**CN:** 这一可调用对象实现了 `BatchedPresencePenalizer._teardown`，主要用于实现 teardown 相关逻辑。

## Key Concepts / 关键概念
- `BatchedPresencePenalizer`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.sampling.penaltylib.orchestrator`
