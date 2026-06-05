# joint_threshold.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/dllm/algorithm/joint_threshold.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed LLM coordination part of the SRT runtime and implements logic centered on `joint_threshold`. It exposes primary entry points such as `JointThreshold`. / 该模块属于 SRT 运行时的分布式 LLM 协调部分，主要实现围绕 `joint_threshold` 的逻辑。 它对外提供的主要入口包括 `JointThreshold`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Module imports, constants, and setup
```python
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

### Lines 12-13: Class JointThreshold
```python
class JointThreshold(DllmAlgorithm):

```
**EN:** This range introduces `JointThreshold` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `JointThreshold`，并定义其后续方法依赖的结构或元数据。

### Lines 14-24: Method JointThreshold.__init__
```python
    def __init__(
        self,
        config: DllmConfig,
    ):
        super().__init__(config)
        self.threshold = config.algorithm_config.get("threshold", 0.5)
        self.edit_threshold = config.algorithm_config.get("edit_threshold", 0)
        self.max_post_edit_steps = config.algorithm_config.get(
            "max_post_edit_steps", 16
        )
        self.penalty_lambda = config.algorithm_config.get("penalty_lambda", 0)
```
**EN:** This callable implements `JointThreshold.__init__`. It takes `config` and mainly initializes instance state and defaults. In this range it adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `JointThreshold.__init__`。它接收 `config`，主要用于初始化实例状态与默认值。 在这一范围内，它会调整采样行为与解码控制。

### Lines 26-85: Method JointThreshold.run (part 1/2)
```python
    def run(
        self,
        model_runner: ModelRunner,
        forward_batch: ForwardBatch,
    ) -> tuple[LogitsProcessorOutput | torch.Tensor, torch.Tensor | None, bool]:
        batch_size = forward_batch.batch_size
        device = forward_batch.input_ids.device

        mask_index = forward_batch.input_ids == self.mask_id
        if not mask_index.any():
            out = model_runner.forward(forward_batch, pp_proxy_tensors=None)
            return out.logits_output, [], out.can_run_graph

        start_list = []
        prompt_masks = []
        for i in range(batch_size):
            block_start = i * self.block_size
            block_end = block_start + self.block_size
            block_input_ids = forward_batch.input_ids[block_start:block_end]

            prompt_mask = block_input_ids != self.mask_id
            prompt_masks.append(prompt_mask)
            start_list.append(prompt_mask.sum().item())

        post_edit_steps = torch.zeros(batch_size, dtype=torch.int32, device=device)

        finished = torch.zeros(batch_size, dtype=torch.bool, device=device)
        # Controls whether to perform an additional forward pass for KV cache persistence.
        # For certain decoding rounds where the terminal step yields no state change,
        # this can be set to False to bypass the overhead of an idle forward pass.
        any_changed_in_last_step = False

        max_iterations = self.block_size + self.max_post_edit_steps
        for _ in range(max_iterations):
            if finished.all():
                break

            out = model_runner.forward(forward_batch, pp_proxy_tensors=None)
            logits_output, can_run_cuda_graph = out.logits_output, out.can_run_graph

            any_changed_in_last_step = False

            for i in range(batch_size):
                if finished[i]:
                    continue

                block_start = i * self.block_size
                block_end = block_start + self.block_size

                curr_input_ids = forward_batch.input_ids[block_start:block_end]
                curr_logits = logits_output.full_logits[block_start:block_end]
                curr_prompt_mask = prompt_masks[i]

                if self.penalty_lambda > 0:
                    prev_ids = curr_input_ids[:-1]
                    curr_logits[1:, :].scatter_(
                        1, prev_ids.unsqueeze(-1), -self.penalty_lambda, reduce="add"
                    )

                x = torch.argmax(curr_logits, dim=-1)
```
**EN:** This callable implements `JointThreshold.run`. It takes `model_runner`, `forward_batch` and mainly executes the main workflow. This chunk is part 1 of 2 for the same logical block. In this range it manages graph capture or replay logic; adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `JointThreshold.run`。它接收 `model_runner`, `forward_batch`，主要用于执行主要流程。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会管理图捕获或回放逻辑；调整采样行为与解码控制。

### Lines 86-136: Method JointThreshold.run (part 2/2)
```python
                p = torch.squeeze(
                    torch.gather(
                        F.softmax(curr_logits, dim=-1),
                        dim=-1,
                        index=torch.unsqueeze(x, -1),
                    ),
                    -1,
                )

                mask_index = curr_input_ids == self.mask_id
                has_mask = mask_index.any()

                # Mask to token (M2T)
                mask_transfer_index = torch.zeros_like(mask_index)
                if has_mask:
                    confidence = torch.where(mask_index, p, -np.inf)
                    mask_transfer_index = confidence > self.threshold

                    if not mask_transfer_index.any():
                        _, select_index = torch.topk(confidence, k=1)
                        mask_transfer_index[select_index] = True
                else:
                    post_edit_steps[i] += 1
                    if post_edit_steps[i] > self.max_post_edit_steps:
                        finished[i] = True
                        continue

                # Token to token (T2T)
                edit_mask = ~mask_index & ~curr_prompt_mask
                edit_transfer_index = (
                    (p > self.edit_threshold) & (curr_input_ids != x) & edit_mask
                )

                transfer_index = mask_transfer_index | edit_transfer_index
                if not transfer_index.any():
                    finished[i] = True
                    continue

                curr_input_ids[transfer_index] = x[transfer_index]
                any_changed_in_last_step = True

        if any_changed_in_last_step:
            out = model_runner.forward(forward_batch, pp_proxy_tensors=None)
            logits_output, can_run_cuda_graph = out.logits_output, out.can_run_graph

        next_token_ids = torch.reshape(forward_batch.input_ids, (batch_size, -1))
        next_token_ids_list = [
            next_token_ids[i, start_list[i] :] for i in range(batch_size)
        ]

        return logits_output, next_token_ids_list, can_run_cuda_graph
```
**EN:** This callable implements `JointThreshold.run`. It takes `model_runner`, `forward_batch` and mainly executes the main workflow. This chunk is part 2 of 2 for the same logical block. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `JointThreshold.run`。它接收 `model_runner`, `forward_batch`，主要用于执行主要流程。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 137-139: Module-level constants and helpers
```python


Algorithm = JointThreshold
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

## Key Concepts / 关键概念
- `JointThreshold`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Third-party / 第三方**: `numpy`, `torch`, `torch.nn.functional`
- **Internal modules / 内部模块**: `sglang.srt.dllm.algorithm.base`, `sglang.srt.dllm.config`, `sglang.srt.layers.logits_processor`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.model_executor.model_runner`
