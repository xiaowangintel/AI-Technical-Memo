# test_fused_moe_lora_kernel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/lora/test_fused_moe_lora_kernel.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates fused moe lora kernel behavior in SGLang's lora area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 lora 领域中与 fused moe lora kernel 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting source context / 辅助源码上下文
```python
# Temporarily adapted from https://github.com/vllm-project/vllm/blob/main/tests/lora/test_fused_moe_lora_kernel.py, will optimize in future refactor
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 2-14: module imports and dependencies / 模块导入与依赖
```python
import random
import sys

import pytest
import torch

# ==============================================================================
# IMPORT PREBUILT KERNEL
# ==============================================================================
from sglang.jit_kernel.moe_lora_align import moe_lora_align_block_size
from sglang.srt.lora.triton_ops import fused_moe_lora
from sglang.srt.utils import set_random_seed
from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `random`, `sys`, `pytest`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `random`, `sys`, `pytest`, `torch`。

### Lines 15-17: supporting source context / 辅助源码上下文
```python

# ==============================================================================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 18-18: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=28, stage="base-b", runner_config="1-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 21-22: function round up / 函数 round up
```python
def round_up(x, base):
    return ((x + base - 1) // base) * base
```
**EN:** This block implements `round_up` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `round_up`，承担模块行为中的一个聚焦逻辑片段。

### Lines 25-26: function CEILDIV / 函数 CEILDIV
```python
def CEILDIV(x, y):
    return (x + y - 1) // y
```
**EN:** This block implements `CEILDIV` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `CEILDIV`，承担模块行为中的一个聚焦逻辑片段。

### Lines 29-75: function assign loras to tokens / 函数 assign loras to tokens
```python
def assign_loras_to_tokens(num_tokens: int, num_sequences: int, max_loras: int):
    """
    Split `num_tokens` into `num_sequences` sequences.
    Each sequence randomly selects 1 LoRA index from [0, max_loras),
    and all tokens in that sequence are assigned this LoRA index.

    Args:
        num_tokens (int): Total number of tokens.
        num_sequences (int): Number of sequences to split the tokens into.
        max_loras (int): Total number of available LoRA modules.

    Returns:
        token_lora_mapping (torch.Tensor): 1D tensor of shape [num_tokens]
        seg_indptr (torch.Tensor): 1D tensor of shape [num_sequences + 1]
        req_to_lora (torch.Tensor): 1D tensor of shape [num_sequences]
    """
    assert num_sequences > 0 and max_loras > 0
    assert num_tokens >= num_sequences, "num_tokens must be >= num_sequences"

    # Compute token distribution per sequence (distribute remainder evenly)
    tokens_per_seq = num_tokens // num_sequences
    remainder = num_tokens % num_sequences

    token_lora_mapping = torch.empty(num_tokens, dtype=torch.int32)
    seg_indptr = [0]
    req_to_lora = []

    start = 0
    for seq_idx in range(num_sequences):
        # Determine the token range for this sequence
        end = start + tokens_per_seq + (1 if seq_idx < remainder else 0)

        # Randomly select one LoRA ID for this sequence
        lora_id = random.randint(0, max_loras - 1)

        # Assign the same LoRA ID to all tokens in this sequence
        token_lora_mapping[start:end] = lora_id

        seg_indptr.append(end)
        req_to_lora.append(lora_id)

        start = end

    seg_indptr = torch.tensor(seg_indptr, dtype=torch.int32)
    req_to_lora = torch.tensor(req_to_lora, dtype=torch.int32)

    return token_lora_mapping, seg_indptr, req_to_lora
```
**EN:** Split `num_tokens` into `num_sequences` sequences. This block implements `assign_loras_to_tokens` and captures one focused piece of the module's behavior.
**CN:** Split `num_tokens` into `num_sequences` sequences. 该代码块实现 `assign_loras_to_tokens`，承担模块行为中的一个聚焦逻辑片段。

### Lines 78-107: function assign experts to tokens / 函数 assign experts to tokens
```python
def assign_experts_to_tokens(num_tokens: int, num_experts: int, top_k_num: int):
    """
    For each token, randomly select `top_k_num` distinct experts out of `num_experts`,
    and assign normalized random weights that sum to 1.

    Args:
        num_tokens (int): Total number of tokens.
        num_experts (int): Total number of available experts.
        top_k_num (int): Number of experts to select per token.

    Returns:
        expert_indices (torch.Tensor): shape [num_tokens, top_k_num],
                                       expert index for each token.
        expert_weights (torch.Tensor): shape [num_tokens, top_k_num],
                                       normalized weights (sum = 1 per row).
    """
    assert top_k_num <= num_experts, "top_k_num must be <= num_experts"

    # Randomly select top_k_num distinct experts for each token
    expert_indices = torch.empty((num_tokens, top_k_num), dtype=torch.int32)
    for i in range(num_tokens):
        # Randomly choose unique expert indices
        selected = torch.randperm(num_experts)[:top_k_num]
        expert_indices[i] = selected

    # Generate random weights and normalize along dim=1
    expert_weights = torch.rand((num_tokens, top_k_num), dtype=torch.float32)
    expert_weights = expert_weights / expert_weights.sum(dim=1, keepdim=True)

    return expert_indices, expert_weights
```
**EN:** For each token, randomly select `top_k_num` distinct experts out of `num_experts`, and assign normalized random weights that sum to 1. This block implements `assign_experts_to_tokens` and captures one focused piece of the module's behavior.
**CN:** For each token, randomly select `top_k_num` distinct experts out of `num_experts`, and assign normalized random weights that sum to 1. 该代码块实现 `assign_experts_to_tokens`，承担模块行为中的一个聚焦逻辑片段。

### Lines 110-123: function sample data / 函数 sample data
```python
def sample_data(
    num_tokens: int,
    num_sequences: int,
    max_loras: int,
    num_experts: int,
    top_k_num: int,
):
    topk_ids, topk_weights = assign_experts_to_tokens(
        num_tokens, num_experts, top_k_num
    )
    token_lora_mapping, seg_indptr, req_to_lora = assign_loras_to_tokens(
        num_tokens, num_sequences, max_loras
    )
    return topk_ids, topk_weights, token_lora_mapping, seg_indptr, req_to_lora
```
**EN:** This block implements `sample_data` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `sample_data`，承担模块行为中的一个聚焦逻辑片段。

### Lines 126-205: function use fused moe lora kernel (part 1/2) / 函数 use fused moe lora kernel（第 1/2 部分）
```python
def use_fused_moe_lora_kernel(
    topk_ids,
    topk_weights,
    seg_indptr,
    req_to_lora,
    max_lora_rank,
    top_k_num,
    lora_a_stacked,
    lora_b_stacked,
    hidden_states,
    output,
    max_loras,
    num_experts,
    block_size,
    mul_routed_weight,
    fully_sharded=False,
    offset=0,
):
    max_num_tokens_padded = topk_ids.numel() + num_experts * (block_size - 1)
    max_num_tokens_padded = round_up(max_num_tokens_padded, block_size)
    max_num_m_blocks = CEILDIV(max_num_tokens_padded, block_size)

    # Important: Ensure output tensors are on the same device as inputs
    device = topk_ids.device

    # init output tensors
    sorted_token_ids = torch.empty(
        (max_loras * max_num_tokens_padded,), dtype=torch.int32, device=device
    )
    expert_ids = torch.empty(
        (max_loras * max_num_m_blocks,), dtype=torch.int32, device=device
    )
    num_tokens_post_padded = torch.empty((max_loras,), dtype=torch.int32, device=device)
    adapter_enabled = torch.ones(max_loras + 1, dtype=torch.int32, device=device)
    lora_ids = torch.arange(max_loras, dtype=torch.int32, device=device)

    # call kernel
    moe_lora_align_block_size(
        topk_ids,
        seg_indptr,
        req_to_lora,
        num_experts,
        block_size,
        max_loras,
        max_num_tokens_padded,
        max_num_m_blocks,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        adapter_enabled,
        lora_ids,
        None,  # maybe_expert_map
    )

    config = {
        "BLOCK_SIZE_M": 16,
        "BLOCK_SIZE_N": 32,
        "BLOCK_SIZE_K": 64,
        "GROUP_SIZE_M": 1,
        "NUM_WARPS": 4,
        "NUM_STAGES": 3,
        "SPLIT_K": 1,
    }

    expert_ids = expert_ids.view(max_loras, -1)
    sorted_token_ids = sorted_token_ids.view(max_loras, -1)

    fused_moe_lora(
        output,
        hidden_states,
        lora_a_stacked,
        lora_b_stacked,
        topk_weights,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        max_lora_rank,
        top_k_num,
        lora_ids,
        adapter_enabled,
```
**EN:** This block implements `use_fused_moe_lora_kernel` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** 该代码块实现 `use_fused_moe_lora_kernel`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 206-223: function use fused moe lora kernel (part 2/2) / 函数 use fused moe lora kernel（第 2/2 部分）
```python
        config["BLOCK_SIZE_M"],
        config["BLOCK_SIZE_N"],
        config["BLOCK_SIZE_K"],
        config["GROUP_SIZE_M"],
        config["NUM_WARPS"],
        config["NUM_STAGES"],
        config["SPLIT_K"],
        config["BLOCK_SIZE_M"],
        config["BLOCK_SIZE_N"],
        config["BLOCK_SIZE_K"],
        config["GROUP_SIZE_M"],
        config["NUM_WARPS"],
        config["NUM_STAGES"],
        config["SPLIT_K"],
        mul_routed_weight,
        fully_sharded=fully_sharded,
        offset=offset,
    )
```
**EN:** This block implements `use_fused_moe_lora_kernel` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** 该代码块实现 `use_fused_moe_lora_kernel`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 226-262: function use torch / 函数 use torch
```python
def use_torch(
    hidden_states,
    token_lora_mapping,
    topk_ids,
    topk_weights,
    lora_a_stacked,
    lora_b_stacked,
    top_k_num,
    mul_routed_weight,
):
    outputs = []

    orig_dtype = hidden_states.dtype
    for i in range(hidden_states.shape[0]):
        lora_idx = token_lora_mapping[i]
        expert_ids = topk_ids[i]
        expert_weights = topk_weights[i]

        lora_a = lora_a_stacked[0][lora_idx][expert_ids]
        lora_b = lora_b_stacked[0][lora_idx][expert_ids]

        h_f32 = hidden_states[i].to(torch.float32)
        la_f32 = lora_a.to(torch.float32)
        lb_f32 = lora_b.to(torch.float32)

        if mul_routed_weight:
            tensors = [
                ((h_f32 @ la_f32[x].T @ lb_f32[x].T) * expert_weights[x]).to(orig_dtype)
                for x in range(top_k_num)
            ]
        else:
            tensors = [
                (h_f32 @ la_f32[x].T @ lb_f32[x].T).to(orig_dtype)
                for x in range(top_k_num)
            ]
        outputs.append(torch.stack(tensors, dim=0))
    return torch.stack(outputs, dim=0)
```
**EN:** This block implements `use_torch` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `use_torch`，承担模块行为中的一个聚焦逻辑片段。

### Lines 265-267: module-level constants and configuration / 模块级常量与配置
```python
DTYPES = [torch.float32, torch.float16, torch.bfloat16]
DEVICES = [f"cuda:{0}"]
SEED = [42]
```
**EN:** This block defines shared names such as `DTYPES`, `DEVICES`, `SEED`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `DTYPES`, `DEVICES`, `SEED` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 270-349: test case fused moe lora kernel (part 1/2) / 测试用例 fused moe lora kernel（第 1/2 部分）
```python
@pytest.mark.parametrize("mul_routed_weight", [False, True])
@pytest.mark.parametrize("num_tokens", [100])
@pytest.mark.parametrize("top_k_num", [6, 12])
@pytest.mark.parametrize("num_experts", [64])
@pytest.mark.parametrize("max_loras", [4, 6, 16])
@pytest.mark.parametrize("N", [1408])
@pytest.mark.parametrize("K", [2048])
@pytest.mark.parametrize("max_lora_rank", [16, 32, 64])
@pytest.mark.parametrize("block_size", [16])
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("device", DEVICES)
@pytest.mark.parametrize("seed", SEED)
def test_fused_moe_lora_kernel(
    mul_routed_weight,
    num_tokens,
    top_k_num,
    num_experts,
    max_loras,
    N,
    K,
    max_lora_rank,
    block_size,
    dtype,
    device,
    seed,
):
    torch.set_default_device(device)
    set_random_seed(seed)
    # the number of randomly generated sentences.
    num_sequences = 10
    # generate data
    topk_ids, topk_weights, token_lora_mapping, seg_indptr, req_to_lora = sample_data(
        num_tokens, num_sequences, max_loras, num_experts, top_k_num
    )

    # Ensure generated data is on the correct device
    topk_ids = topk_ids.to(device)
    topk_weights = topk_weights.to(device)
    token_lora_mapping = token_lora_mapping.to(device)
    seg_indptr = seg_indptr.to(device)
    req_to_lora = req_to_lora.to(device)

    # init lora weights
    lora_a_stacked = [
        torch.rand(
            (
                max_loras,
                num_experts,
                max_lora_rank,
                K,
            ),
            dtype=dtype,
            device=device,
        )
    ]
    lora_b_stacked = [
        torch.rand(
            (
                max_loras,
                num_experts,
                N,
                max_lora_rank,
            ),
            dtype=dtype,
            device=device,
        )
    ]
    hidden_states = torch.rand(
        (
            num_tokens,
            K,
        ),
        dtype=dtype,
        device=device,
    )

    # fused_moe_lora_kernel output
    output = torch.zeros((num_tokens, top_k_num, N), dtype=dtype, device=device)

    use_fused_moe_lora_kernel(
```
**EN:** This test exercises `test_fused_moe_lora_kernel` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_moe_lora_kernel`。 这一段对应同一逻辑块的第 1 部分。

### Lines 350-377: test case fused moe lora kernel (part 2/2) / 测试用例 fused moe lora kernel（第 2/2 部分）
```python
        topk_ids,
        topk_weights,
        seg_indptr,
        req_to_lora,
        max_lora_rank,
        top_k_num,
        lora_a_stacked,
        lora_b_stacked,
        hidden_states,
        output,
        max_loras,
        num_experts,
        block_size,
        mul_routed_weight=mul_routed_weight,
    )
    # pytorch output
    output2 = use_torch(
        hidden_states,
        token_lora_mapping,
        topk_ids,
        topk_weights,
        lora_a_stacked,
        lora_b_stacked,
        top_k_num,
        mul_routed_weight=mul_routed_weight,
    )

    torch.testing.assert_close(output, output2, atol=1e-2, rtol=1e-2)
```
**EN:** This test exercises `test_fused_moe_lora_kernel` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_moe_lora_kernel`。 这一段对应同一逻辑块的第 2 部分。

### Lines 380-381: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `round_up`: This block implements `round_up` and captures one focused piece of the module's behavior. / 该代码块实现 `round_up`，承担模块行为中的一个聚焦逻辑片段。
- `CEILDIV`: This block implements `CEILDIV` and captures one focused piece of the module's behavior. / 该代码块实现 `CEILDIV`，承担模块行为中的一个聚焦逻辑片段。
- `assign_loras_to_tokens`: Split `num_tokens` into `num_sequences` sequences. / 该代码块实现 `assign_loras_to_tokens`，承担模块行为中的一个聚焦逻辑片段。
- `assign_experts_to_tokens`: For each token, randomly select `top_k_num` distinct experts out of `num_experts`, and assign normalized random weights that sum to 1. / 该代码块实现 `assign_experts_to_tokens`，承担模块行为中的一个聚焦逻辑片段。
- `sample_data`: This block implements `sample_data` and captures one focused piece of the module's behavior. / 该代码块实现 `sample_data`，承担模块行为中的一个聚焦逻辑片段。
- `use_fused_moe_lora_kernel`: This block implements `use_fused_moe_lora_kernel` and captures one focused piece of the module's behavior. / 该代码块实现 `use_fused_moe_lora_kernel`，承担模块行为中的一个聚焦逻辑片段。
- `use_torch`: This block implements `use_torch` and captures one focused piece of the module's behavior. / 该代码块实现 `use_torch`，承担模块行为中的一个聚焦逻辑片段。
- `test_fused_moe_lora_kernel`: This test exercises `test_fused_moe_lora_kernel` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_moe_lora_kernel`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `random`, `sys`
- **Third-party modules / 第三方模块**: `pytest`, `torch`
- **Internal modules / 内部模块**: `sglang.jit_kernel.moe_lora_align`, `sglang.srt.lora.triton_ops`, `sglang.srt.utils`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 381
