# min_new_tokens.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/sampling/penaltylib/min_new_tokens.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the sampling and penalty control part of the SRT runtime and implements logic centered on `min_new_tokens`. It exposes primary entry points such as `BatchedMinNewTokensPenalizer`. / 该模块属于 SRT 运行时的采样与惩罚控制部分，主要实现围绕 `min_new_tokens` 的逻辑。 它对外提供的主要入口包括 `BatchedMinNewTokensPenalizer`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Module imports, constants, and setup
```python
import torch

from sglang.srt.sampling.penaltylib.orchestrator import _BatchedPenalizer


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; adjusts sampling behavior and decoding controls.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；调整采样行为与解码控制。

### Lines 6-10: Class BatchedMinNewTokensPenalizer
```python
class BatchedMinNewTokensPenalizer(_BatchedPenalizer):
    """
    Min new tokens penalizer penalizes tokens based on the length of the output.
    """

```
**EN:** This range introduces `BatchedMinNewTokensPenalizer` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Min new tokens penalizer penalizes tokens based on the length of the output."
**CN:** 这一段引入 `BatchedMinNewTokensPenalizer`，并定义其后续方法依赖的结构或元数据。

### Lines 11-14: Method BatchedMinNewTokensPenalizer._is_required
```python
    def _is_required(self) -> bool:
        return any(
            req.sampling_params.min_new_tokens > 0 for req in self.orchestrator.reqs()
        )
```
**EN:** This callable implements `BatchedMinNewTokensPenalizer._is_required` and mainly implements is required.
**CN:** 这一可调用对象实现了 `BatchedMinNewTokensPenalizer._is_required`，主要用于实现 is required 相关逻辑。

### Lines 16-64: Method BatchedMinNewTokensPenalizer._prepare
```python
    def _prepare(self):
        self.min_new_tokens = torch.tensor(
            data=[
                req.sampling_params.min_new_tokens for req in self.orchestrator.reqs()
            ],
            dtype=torch.int32,
            device=self.orchestrator.device,
        ).unsqueeze_(1)

        padded_stop_token_ids = torch.nn.utils.rnn.pad_sequence(
            sequences=[
                torch.tensor(
                    data=(
                        list(
                            (req.sampling_params.stop_token_ids or set())
                            | (req.tokenizer.additional_stop_token_ids or set())
                            | {req.tokenizer.eos_token_id}
                        )
                    ),
                    dtype=torch.int64,
                    device=self.orchestrator.device,
                )
                for req in self.orchestrator.reqs()
            ],
            batch_first=True,
            padding_value=self.orchestrator.vocab_size,
        )
        self.stop_token_penalties = torch.zeros(
            size=(len(self.orchestrator.reqs()), self.orchestrator.vocab_size + 1),
            dtype=torch.float32,
            device=self.orchestrator.device,
        ).scatter_add_(
            dim=1,
            index=padded_stop_token_ids,
            src=torch.full_like(
                input=padded_stop_token_ids,
                dtype=torch.float32,
                fill_value=float("-inf"),
                device=self.orchestrator.device,
            ),
        )[
            :, : self.orchestrator.vocab_size
        ]

        self.len_output_tokens = torch.zeros(
            size=(len(self.orchestrator.reqs()), 1),
            dtype=torch.int32,
            device=self.orchestrator.device,
        )
```
**EN:** This callable implements `BatchedMinNewTokensPenalizer._prepare` and mainly prepares runtime inputs.
**CN:** 这一可调用对象实现了 `BatchedMinNewTokensPenalizer._prepare`，主要用于准备运行时输入。

### Lines 66-67: Method BatchedMinNewTokensPenalizer._cumulate_output_tokens
```python
    def _cumulate_output_tokens(self, output_ids: torch.Tensor):
        self.len_output_tokens += 1
```
**EN:** This callable implements `BatchedMinNewTokensPenalizer._cumulate_output_tokens`. It takes `output_ids` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `BatchedMinNewTokensPenalizer._cumulate_output_tokens`。它接收 `output_ids`，主要用于将数据转换为另一种表示。

### Lines 69-71: Method BatchedMinNewTokensPenalizer._apply
```python
    def _apply(self, logits: torch.Tensor):
        mask = (self.len_output_tokens < self.min_new_tokens).expand_as(logits)
        logits[mask] += self.stop_token_penalties[mask]
```
**EN:** This callable implements `BatchedMinNewTokensPenalizer._apply`. It takes `logits` and mainly implements apply.
**CN:** 这一可调用对象实现了 `BatchedMinNewTokensPenalizer._apply`。它接收 `logits`，主要用于实现 apply 相关逻辑。

### Lines 73-76: Method BatchedMinNewTokensPenalizer._filter
```python
    def _filter(self, keep_indices: torch.Tensor):
        self.min_new_tokens = self.min_new_tokens[keep_indices]
        self.stop_token_penalties = self.stop_token_penalties[keep_indices]
        self.len_output_tokens = self.len_output_tokens[keep_indices]
```
**EN:** This callable implements `BatchedMinNewTokensPenalizer._filter`. It takes `keep_indices` and mainly implements filter.
**CN:** 这一可调用对象实现了 `BatchedMinNewTokensPenalizer._filter`。它接收 `keep_indices`，主要用于实现 filter 相关逻辑。

### Lines 78-87: Method BatchedMinNewTokensPenalizer._merge
```python
    def _merge(self, their: "BatchedMinNewTokensPenalizer"):
        self.min_new_tokens = torch.cat(
            [self.min_new_tokens, their.min_new_tokens], dim=0
        )
        self.stop_token_penalties = torch.cat(
            [self.stop_token_penalties, their.stop_token_penalties], dim=0
        )
        self.len_output_tokens = torch.cat(
            [self.len_output_tokens, their.len_output_tokens], dim=0
        )
```
**EN:** This callable implements `BatchedMinNewTokensPenalizer._merge`. It takes `their` and mainly merges related state.
**CN:** 这一可调用对象实现了 `BatchedMinNewTokensPenalizer._merge`。它接收 `their`，主要用于合并相关状态。

### Lines 88-89: Class-level scaffolding for BatchedMinNewTokensPenalizer
```python

    # Explicit resource cleanup to aid GC and free CUDA memory promptly
```
**EN:** This callable implements `None.BatchedMinNewTokensPenalizer` and mainly converts data into another representation. The docstring states: "Min new tokens penalizer penalizes tokens based on the length of the output."
**CN:** 这一可调用对象实现了 `None.BatchedMinNewTokensPenalizer`，主要用于将数据转换为另一种表示。

### Lines 90-93: Method BatchedMinNewTokensPenalizer._teardown
```python
    def _teardown(self) -> None:
        for name in ("min_new_tokens", "stop_token_penalties", "len_output_tokens"):
            if hasattr(self, name):
                delattr(self, name)
```
**EN:** This callable implements `BatchedMinNewTokensPenalizer._teardown` and mainly implements teardown.
**CN:** 这一可调用对象实现了 `BatchedMinNewTokensPenalizer._teardown`，主要用于实现 teardown 相关逻辑。

## Key Concepts / 关键概念
- `BatchedMinNewTokensPenalizer`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.sampling.penaltylib.orchestrator`
