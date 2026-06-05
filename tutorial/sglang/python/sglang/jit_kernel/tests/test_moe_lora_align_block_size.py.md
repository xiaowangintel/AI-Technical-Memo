# test_moe_lora_align_block_size.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_moe_lora_align_block_size.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and module setup
```python
# Temporarily adapted from https://github.com/vllm-project/vllm/blob/main/tests/lora/test_moe_lora_align_sum.py, will optimize in future refactor
import random
import sys

import pytest
import torch

# ---------------------------------------------------------
# IMPORT PREBUILT KERNEL
# ---------------------------------------------------------
from sglang.jit_kernel.moe_lora_align import moe_lora_align_block_size
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=28, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=120, suite="nightly-kernel-1-gpu", nightly=True)
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 18-21: Function `round_up`
```python
def round_up(x, base):
    return ((x + base - 1) // base) * base
```
**EN:** This block defines `round_up`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `round_up`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 22-25: Function `CEILDIV`
```python
def CEILDIV(x, y):
    return (x + y - 1) // y
```
**EN:** This block defines `CEILDIV`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `CEILDIV`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 26-67: Function `sample_data`
```python
def sample_data(num_experts, max_loras, num_tokens, topk_num):
    # 1. Generate TopK IDs (Flattened tokens)
    topk_ids = torch.zeros((num_tokens, topk_num), dtype=torch.int32)
    for i in range(num_tokens):
        pool = list(range(num_experts))
        random.shuffle(pool)
        for j in range(topk_num):
            topk_ids[i, j] = pool[j]

    # 2. Generate Random Requests (Segments)
    # We split num_tokens into random chunks to simulate a batch of requests
    remaining_tokens = num_tokens
    seg_lens = []
    while remaining_tokens > 0:
        # Random length between 1 and remaining
        length = random.randint(1, min(32, remaining_tokens))
        if remaining_tokens - length < 0:
            length = remaining_tokens
        seg_lens.append(length)
        remaining_tokens -= length

    # Ensure we cover the full range exactly (cleanup last segment)
    if sum(seg_lens) < num_tokens:
        seg_lens.append(num_tokens - sum(seg_lens))

    # 3. Build seg_indptr [0, len1, len1+len2, ...]
    seg_indptr = torch.cumsum(
        torch.tensor([0] + seg_lens, dtype=torch.int32), dim=0
    ).to(dtype=torch.int32)

    # 4. Assign a LoRA ID to each Request
    num_reqs = len(seg_lens)
    req_to_lora = torch.randint(0, max_loras, (num_reqs,), dtype=torch.int32)

    return (topk_ids.to("cuda"), seg_indptr.to("cuda"), req_to_lora.to("cuda"))


@pytest.mark.parametrize("num_tokens", [100, 200, 1024, 4096])
@pytest.mark.parametrize("topk_num", [6])
@pytest.mark.parametrize("num_experts", [64, 128, 256, 512])
@pytest.mark.parametrize("max_loras", [2, 32])
@pytest.mark.parametrize("block_size", [16])
```
**EN:** This block defines `sample_data`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `sample_data`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 68-168: Function `test_moe_lora_align_block_size`
```python
def test_moe_lora_align_block_size(
    num_tokens, topk_num, num_experts, max_loras, block_size
):
    # sample data
    random.seed(1)
    torch.manual_seed(1)

    if not torch.cuda.is_available():
        pytest.skip("CUDA is not available, skipping moe_lora_align_block_size test.")
    # UPDATED: Get the new 3-step mapping tensors
    topk_ids, seg_indptr, req_to_lora = sample_data(
        num_experts, max_loras, num_tokens, topk_num
    )

    # compute paddings
    max_num_tokens_padded = topk_ids.numel() + num_experts * (block_size - 1)
    max_num_tokens_padded = round_up(max_num_tokens_padded, block_size)
    max_num_m_blocks = CEILDIV(max_num_tokens_padded, block_size)

    # init output tensors
    sorted_token_ids = torch.full(
        (max_loras * max_num_tokens_padded,),
        topk_ids.numel(),
        dtype=torch.int32,
        device="cuda",
    )
    expert_ids = torch.full(
        (max_loras * max_num_m_blocks,), num_experts, dtype=torch.int32, device="cuda"
    )
    num_tokens_post_pad = torch.zeros((max_loras,), dtype=torch.int32, device="cuda")
    adapter_enabled = torch.ones((max_loras + 1,), dtype=torch.int32, device="cuda")
    lora_ids = torch.arange(max_loras, dtype=torch.int32, device="cuda")

    # UPDATED: Call kernel with new signature
    moe_lora_align_block_size(
        topk_ids,
        seg_indptr,  # Arg 2: Pointers
        req_to_lora,  # Arg 3: Request Map
        num_experts,
        block_size,
        max_loras,
        max_num_tokens_padded,
        max_num_m_blocks,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_pad,
        adapter_enabled,
        lora_ids,
        None,
    )

    # verify values
    expert_ids = expert_ids.view(max_loras, -1)
    sorted_token_ids = sorted_token_ids.view(max_loras, -1, block_size)

    # Reconstruct token-level ownership for verification logic
    # We expand req_to_lora back to [num_tokens] on CPU just to check correctness
    # This proves the kernel (which used the compressed format) produced the right result
    cpu_seg_indptr = seg_indptr.cpu()
    cpu_req_to_lora = req_to_lora.cpu()
    token_ownership = torch.zeros(num_tokens, dtype=torch.int32)

    for r in range(len(cpu_req_to_lora)):
        start = cpu_seg_indptr[r]
        end = cpu_seg_indptr[r + 1]
        token_ownership[start:end] = cpu_req_to_lora[r]

    token_ownership = token_ownership.to("cuda")

    for lora_idx in range(max_loras):
        # Count how many tokens actually belong to this LoRA
        expected_count = (token_ownership == lora_idx).sum().item()

        # Verify the kernel processed a reasonable number of tokens (sanity check)
        # Note: num_tokens_post_pad includes padding, so it might be larger than expected_count
        assert num_tokens_post_pad[lora_idx].item() >= expected_count * topk_num

        for token_idx in range(sorted_token_ids.size(1)):
            block = sorted_token_ids[lora_idx][token_idx]
            # Valid indices are those less than total numel
# ...
```
**EN:** This block defines `test_moe_lora_align_block_size`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_moe_lora_align_block_size`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `random`
- `sys`
- `pytest`
- `torch`
- `sglang.jit_kernel.moe_lora_align -> moe_lora_align_block_size`
- `sglang.test.ci.ci_register -> register_cuda_ci`
