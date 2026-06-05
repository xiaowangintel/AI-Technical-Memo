# test_moe_lora_align_sum.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/lora/test_moe_lora_align_sum.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises MoE LoRA Align Sum behavior in the LoRA test area through focused pytest scenarios. It focuses on scenarios such as Round Up, Ceildiv, Sample Data. / 该文件在 LoRA 测试域中，通过有针对性的 pytest 场景验证 MoE LoRA Align Sum 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-11)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import random

import pytest
import torch

from vllm import _custom_ops as ops
from vllm.platforms import current_platform

DEVICE_TYPE = current_platform.device_type
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `random`, `pytest`, `torch`, `vllm`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: round_up (lines 14-15)
```python
def round_up(x, base):
    return ((x + base - 1) // base) * base
```
**EN:** Implements a reusable helper for Round Up, reducing duplication across related tests.
**CN:** 该辅助函数为 Round Up 提供可复用逻辑，用于减少相关测试之间的重复代码。

### Helper: CEILDIV (lines 18-19)
```python
def CEILDIV(x, y):
    return (x + y - 1) // y
```
**EN:** Implements a reusable helper for Ceildiv, reducing duplication across related tests.
**CN:** 该辅助函数为 Ceildiv 提供可复用逻辑，用于减少相关测试之间的重复代码。

### Test: test_moe_lora_align_block_size (lines 36-104)
```python
@pytest.mark.parametrize("num_tokens", [100, 200, 1024, 4096])  # 81920
@pytest.mark.parametrize("topk_num", [6])
@pytest.mark.parametrize("num_experts", [64, 128, 256, 512])
@pytest.mark.parametrize("max_loras", [2, 32])
@pytest.mark.parametrize("block_size", [16])
def test_moe_lora_align_block_size(
    num_tokens, topk_num, num_experts, max_loras, block_size
):
    # sample data
    random.seed(1)
    topk_ids, token_lora_mapping = sample_data(
        num_experts, max_loras, num_tokens, topk_num
    )

    # compute paddings
    max_num_tokens_padded = topk_ids.numel() + num_experts * (block_size - 1)
    max_num_tokens_padded = round_up(max_num_tokens_padded, block_size)
    if topk_ids.numel() < num_experts:
        max_num_tokens_padded = topk_ids.numel() * block_size
# ... omitted for brevity ...
        lora_ids,
    )

    # verify values
    expert_ids = expert_ids.view(max_loras, -1)
    sorted_token_ids = sorted_token_ids.view(max_loras, -1, block_size)

    for lora_idx in range(max_loras):
        for token_idx in range(sorted_token_ids.size(1)):
            block = sorted_token_ids[lora_idx][token_idx]
            indices = block[block != topk_ids.numel()]
            if indices.numel() > 0:
                expert_id = expert_ids[lora_idx][token_idx]
                assert torch.all(topk_ids.view(-1)[indices] == expert_id)
```
**EN:** Checks MoE LoRA Align Block Size under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `random.seed`, `sample_data` before asserting the expected outcome.
**CN:** 该测试用例验证 MoE LoRA Align Block Size 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `random.seed`, `sample_data` 驱动目标逻辑，再断言预期结果。

### Constants / assignments (lines 112-112)
```python
SENTINEL_EXPERT = -2
```
**EN:** Defines shared constants or configuration objects like `SENTINEL_EXPERT`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `SENTINEL_EXPERT`），供后续测试重复使用。

### Test: test_moe_lora_align_block_size_mixed_base_and_lora (lines 222-262)
```python
@pytest.mark.parametrize(
    "max_loras",
    [
        1,
        2,
    ],
)
def test_moe_lora_align_block_size_mixed_base_and_lora(max_loras):
    """Regression test for issue #32235: real LoRA slot must not be skipped
    when ``active_lora_ids`` has -1 at position 0."""
    out = _build_and_run_align(
        num_lora_tokens=8, num_base_tokens=8, max_loras=max_loras
    )

    # Sanity check on the layout being tested.
    assert out["lora_ids"][0].item() == -1, (
        "prepare_tensors layout mismatch: -1 expected at position 0 for mixed batch"
    )

# ... omitted for brevity ...
        and post_pad % out["block_size"] == 0
    ), f"num_tokens_post_pad[{real_slot}]={post_pad} is not a valid block-aligned count"

    expert_row = out["expert_ids"].view(max_loras, -1)[real_slot]
    assert (expert_row != SENTINEL_EXPERT).all(), (
        f"expert_ids row for slot {real_slot} has unwritten sentinel entries; "
        "the align kernel skipped the real LoRA slot."
    )

    sorted_row = out["sorted_token_ids"].view(max_loras, -1)[real_slot]
    assert (sorted_row != SENTINEL_TOKEN).all(), (
        f"sorted_token_ids row for slot {real_slot} has unwritten sentinel "
        "entries; the align kernel skipped the real LoRA slot."
    )
```
**EN:** Regression test for issue #32235: real LoRA slot must not be skipped when ``active_lora_ids`` has -1 at position 0. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `_build_and_run_align`, `out['num_tokens_post_pad'][real_slot].item` before asserting the expected outcome.
**CN:** 该测试用例验证 MoE LoRA Align Block Size Mixed Base And LoRA 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `_build_and_run_align`, `out['num_tokens_post_pad'][real_slot].item` 驱动目标逻辑，再断言预期结果。

### Test: test_moe_lora_align_block_size_disabled_adapter_untouched (lines 265-295)
```python
def test_moe_lora_align_block_size_disabled_adapter_untouched():
    """Disabled-adapter slot rows must remain untouched by all three align
    kernels. Pins the invariant protected by the ``adapter_enabled`` guard
    in ``lora_count_and_sort_expert_tokens_kernel``: without it the sort
    kernel reads uninitialized ``token_mask`` values for disabled slots and
    pollutes ``sorted_token_ids`` / ``cumsum_buffer``."""
    max_loras = 1
    out = _build_and_run_align(
        num_lora_tokens=16,
        num_base_tokens=0,
        max_loras=max_loras,
        disabled_slots=(0,),
    )
    # Sanity: slot 0 IS present in active_lora_ids (otherwise we would only
    # exercise the lora_id == -1 / >= max_loras guards).
    assert (out["lora_ids"] == 0).any().item()

    assert out["num_tokens_post_pad"][0].item() == SENTINEL_NPAD, (
        "num_tokens_post_pad[0] was modified for a disabled adapter slot."
    )
    expert_row = out["expert_ids"].view(max_loras, -1)[0]
    assert (expert_row == SENTINEL_EXPERT).all(), (
        "expert_ids row for disabled slot 0 was partially written."
    )
    # Row specifically protected by the sort-kernel adapter_enabled guard.
    sorted_row = out["sorted_token_ids"].view(max_loras, -1)[0]
    assert (sorted_row == SENTINEL_TOKEN).all(), (
        "sorted_token_ids row for disabled slot 0 was polluted by the sort "
        "kernel; lora_count_and_sort_expert_tokens_kernel must skip "
        "adapter_enabled == 0 slots."
    )
```
**EN:** Disabled-adapter slot rows must remain untouched by all three align kernels. The body exercises logic via `_build_and_run_align`, `(out['lora_ids'] == 0).any().item`, `(expert_row == SENTINEL_EXPERT).all` before asserting the expected outcome.
**CN:** 该测试用例验证 MoE LoRA Align Block Size Disabled Adapter Untouched 在特定场景下的行为。 函数体会先通过 `_build_and_run_align`, `(out['lora_ids'] == 0).any().item`, `(expert_row == SENTINEL_EXPERT).all` 驱动目标逻辑，再断言预期结果。

### Test: test_moe_lora_align_block_size_lora_id_oob_guard (lines 298-321)
```python
def test_moe_lora_align_block_size_lora_id_oob_guard():
    """Regression test for the ``lora_id >= max_loras`` guard.

    Production ``LoRAKernelMeta.prepare_tensors`` pre-fills the tail of
    ``active_lora_ids`` with -1, so the existing ``lora_id == -1`` check
    covers the extra slot. This test bypasses that invariant and injects
    an out-of-range value (5 with max_loras=1) at the tail to verify the
    explicit guard prevents OOB reads against ``adapter_enabled`` and
    OOB writes against the max_loras-sized output buffers. Without the
    guard, an illegal-memory-access would surface on the next CUDA sync.
    """
    max_loras = 1
    lora_ids_override = torch.tensor([0, 5], dtype=torch.int32)
    out = _build_and_run_align(
        num_lora_tokens=16,
        num_base_tokens=0,
        max_loras=max_loras,
        lora_ids_override=lora_ids_override,
    )
    # The .item() call below syncs and would surface any async
    # illegal-memory-access from the OOB iteration.
    assert out["num_tokens_post_pad"][0].item() != SENTINEL_NPAD, (
        "real LoRA slot 0 was skipped by the align kernel"
    )
```
**EN:** Regression test for the ``lora_id >= max_loras`` guard. The body exercises logic via `torch.tensor`, `_build_and_run_align`, `out['num_tokens_post_pad'][0].item` before asserting the expected outcome.
**CN:** 该测试用例验证 MoE LoRA Align Block Size LoRA Id Oob Guard 在特定场景下的行为。 函数体会先通过 `torch.tensor`, `_build_and_run_align`, `out['num_tokens_post_pad'][0].item` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
sample_data
Constants / assignments
Constants / assignments
_build_and_run_align
Conditional block
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Tokenization / 分词与反分词**
  - **EN:** The logic checks how text and token IDs are converted, streamed, or reconstructed.
  - **CN:** 逻辑重点检查文本与 token ID 的转换、流式处理和重建过程。
- **LoRA adapters / LoRA 适配器**
  - **EN:** The scenarios focus on adapter loading, composition, or low-rank update behavior.
  - **CN:** 这些场景关注适配器加载、组合方式或低秩更新行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `random`
- **Third-party / 第三方依赖**: `pytest`, `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.platforms`
