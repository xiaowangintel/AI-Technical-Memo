# low_confidence.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/dllm/algorithm/low_confidence.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed LLM coordination part of the SRT runtime and implements logic centered on `low_confidence`. It exposes primary entry points such as `LowConfidence`. / 该模块属于 SRT 运行时的分布式 LLM 协调部分，主要实现围绕 `low_confidence` 的逻辑。 它对外提供的主要入口包括 `LowConfidence`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Module imports, constants, and setup
```python
from typing import List, Tuple, Union

import numpy as np
import torch
import torch.nn.functional as F

from sglang.srt.dllm.algorithm.base import DllmAlgorithm
from sglang.srt.dllm.config import DllmConfig
from sglang.srt.layers.logits_processor import LogitsProcessorOutput
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_executor.model_runner import ModelRunner


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 14-15: Class LowConfidence
```python
class LowConfidence(DllmAlgorithm):

```
**EN:** This range introduces `LowConfidence` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `LowConfidence`，并定义其后续方法依赖的结构或元数据。

### Lines 16-21: Method LowConfidence.__init__
```python
    def __init__(
        self,
        config: DllmConfig,
    ):
        super().__init__(config)
        self.threshold = config.algorithm_config.get("threshold", 0.95)
```
**EN:** This callable implements `LowConfidence.__init__`. It takes `config` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `LowConfidence.__init__`。它接收 `config`，主要用于初始化实例状态与默认值。

### Lines 23-82: Method LowConfidence.run (part 1/2)
```python
    def run(
        self,
        model_runner: ModelRunner,
        forward_batch: ForwardBatch,
    ) -> Tuple[Union[LogitsProcessorOutput, torch.Tensor], List[torch.Tensor], bool]:
        batch_size = forward_batch.batch_size
        # Here, the forward_batch full logits contains all the blocks
        # such as [dllm_block_size * batch_size, hidden_size]
        start_list = []
        mask_index = forward_batch.input_ids == self.mask_id

        # Fast path: if there is no mask token, forward and save kv cache
        if torch.sum(mask_index).item() == 0:
            out = model_runner.forward(forward_batch, pp_proxy_tensors=None)
            logits_output, can_run_cuda_graph = out.logits_output, out.can_run_graph

            next_token_ids = []
            return logits_output, next_token_ids, can_run_cuda_graph

        # Calculate start positions for each block
        for block_id in range(batch_size):
            block_start = block_id * self.block_size
            block_end = block_start + self.block_size
            block_input_ids = forward_batch.input_ids[block_start:block_end]
            block_mask_index = block_input_ids == self.mask_id
            start = self.block_size - torch.sum(block_mask_index).item()
            start_list.append(start)

        for _ in range(self.block_size):
            mask_index = forward_batch.input_ids == self.mask_id
            if torch.sum(mask_index).item() == 0:
                break

            out = model_runner.forward(forward_batch, pp_proxy_tensors=None)
            logits_output, can_run_cuda_graph = out.logits_output, out.can_run_graph
            assert batch_size == forward_batch.input_ids.shape[0] // self.block_size
            for batch_id in range(batch_size):
                curr_block_start = batch_id * self.block_size
                curr_block_end = curr_block_start + self.block_size
                block_input_ids = forward_batch.input_ids[
                    curr_block_start:curr_block_end,
                ]
                block_mask_index = block_input_ids == self.mask_id
                if torch.sum(block_mask_index).item() == 0:
                    continue
                curr_logits = logits_output.full_logits[
                    curr_block_start:curr_block_end,
                ]

                x = torch.argmax(curr_logits, dim=-1)
                p = torch.squeeze(
                    torch.gather(
                        F.softmax(curr_logits, dim=-1),
                        dim=-1,
                        index=torch.unsqueeze(x, -1),
                    ),
                    -1,
                )
                x = torch.where(block_mask_index, x, block_input_ids)
                confidence = torch.where(block_mask_index, p, -np.inf)
```
**EN:** This callable implements `LowConfidence.run`. It takes `model_runner`, `forward_batch` and mainly executes the main workflow. This chunk is part 1 of 2 for the same logical block. In this range it performs defensive checks on invalid state; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `LowConfidence.run`。它接收 `model_runner`, `forward_batch`，主要用于执行主要流程。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会对非法状态执行防御性检查；管理图捕获或回放逻辑。

### Lines 83-101: Method LowConfidence.run (part 2/2)
```python

                transfer_index = confidence > self.threshold

                if transfer_index.sum().item() == 0:
                    _, select_index = torch.topk(confidence, k=1)
                    transfer_index[select_index] = True

                block_input_ids[transfer_index] = x[transfer_index]

        out = model_runner.forward(forward_batch, pp_proxy_tensors=None)
        logits_output, can_run_cuda_graph = out.logits_output, out.can_run_graph
        # Here next token ids is tricky to implement the dynamic lengths,
        # so we return a list of tensors
        next_token_ids = torch.reshape(forward_batch.input_ids, (batch_size, -1))
        next_token_ids_list = [
            next_token_ids[i, start_list[i] :] for i in range(batch_size)
        ]

        return logits_output, next_token_ids_list, can_run_cuda_graph
```
**EN:** This callable implements `LowConfidence.run`. It takes `model_runner`, `forward_batch` and mainly executes the main workflow. This chunk is part 2 of 2 for the same logical block. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `LowConfidence.run`。它接收 `model_runner`, `forward_batch`，主要用于执行主要流程。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 102-104: Module-level constants and helpers
```python


Algorithm = LowConfidence
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

## Key Concepts / 关键概念
- `LowConfidence`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方**: `numpy`, `torch`, `torch.nn.functional`
- **Internal modules / 内部模块**: `sglang.srt.dllm.algorithm.base`, `sglang.srt.dllm.config`, `sglang.srt.layers.logits_processor`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.model_executor.model_runner`
