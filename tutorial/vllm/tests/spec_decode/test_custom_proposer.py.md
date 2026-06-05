# test_custom_proposer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/spec_decode/test_custom_proposer.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Integration test for custom proposer class in speculative decoding. / 该文件主要围绕 Custom Proposer 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-18)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

"""Integration test for custom proposer class in speculative decoding.

Usage:
    .venv/bin/python test_custom_proposer.py
"""

import os

import torch

from vllm import LLM, SamplingParams
from vllm.config import VllmConfig

MODEL_ID = "facebook/opt-125m"
NUM_SPEC_TOKENS = 5
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `os`, `torch`, `vllm`, `vllm.config`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Class: DummyDraftProposer (lines 21-72)
```python
class DummyDraftProposer:
    """Custom proposer class that repeats the last token of each sequence.

    This demonstrates the class-based custom proposer interface.
    """

    def __init__(self, vllm_config: VllmConfig):
        """Initialize the custom proposer.

        Args:
            vllm_config: vLLM configuration containing model and speculative settings.
        """
        self.num_speculative_tokens = (
            vllm_config.speculative_config.num_speculative_tokens
        )
        self.max_model_len = vllm_config.model_config.max_model_len
        print(
            f"[DummyDraftProposer.__init__] num_speculative_tokens="
            f"{self.num_speculative_tokens}, max_model_len={self.max_model_len}"
# ... omitted for brevity ...
        """
        # Cross-process flag to prove this method was executed
        with open("proposer_called.flag", "w") as f:
            f.write("called")

        batch_size = len(sampled_token_ids)
        last_tokens = [seq[-1] for seq in sampled_token_ids]
        drafts = [[t] * self.num_speculative_tokens for t in last_tokens]
        print(
            f"[DummyDraftProposer.propose] batch_size={batch_size}, "
            f"num_speculative_tokens={self.num_speculative_tokens}, "
            f"drafts_shape={len(drafts)}x{len(drafts[0])}"
        )
        return drafts
```
**EN:** Groups related scenarios for Dummydraftproposer.
**CN:** 该类把与 Dummydraftproposer 相关的场景组织在一起。

### Conditional block (lines 75-121)
```python
if __name__ == "__main__":
    print("=" * 60)
    print("Custom Proposer Backend Integration Test")
    print("=" * 60)

    # Cleanup any leftover flag from previous failed runs
    if os.path.exists("proposer_called.flag"):
        os.remove("proposer_called.flag")

    llm = LLM(
        model=MODEL_ID,
        speculative_config={
            "model": f"{__name__}.DummyDraftProposer",
            "num_speculative_tokens": NUM_SPEC_TOKENS,
        },
        gpu_memory_utilization=0.4,
        enforce_eager=True,
    )

# ... omitted for brevity ...
        prompt = output.prompt
        generated = output.outputs[0].text
        print(f"Prompt:          {prompt!r}")
        print(f"Generated text:  {generated!r}")
        print("-" * 60)

    # Verify the custom proposer's propose() was actually called across processes
    assert os.path.exists("proposer_called.flag"), (
        "The custom proposer's propose() method was never called!"
    )
    os.remove("proposer_called.flag")

    print("✓ Custom proposer was actively used during generation!")
    print("Test completed successfully.")
```
**EN:** Handles a conditional branch that adjusts test behavior for a specific environment, backend, or runtime constraint.
**CN:** 这里处理条件分支，用于针对特定环境、后端或运行时限制调整测试行为。

## Key Concepts / 关键概念
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Tokenization / 分词与反分词**
  - **EN:** The logic checks how text and token IDs are converted, streamed, or reconstructed.
  - **CN:** 逻辑重点检查文本与 token ID 的转换、流式处理和重建过程。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。
- **Sampling behavior / 采样行为**
  - **EN:** The assertions target decoding controls and the shape of generated outputs.
  - **CN:** 这些断言关注解码控制参数以及生成结果的形态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`
- **Third-party / 第三方依赖**: `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.config`
