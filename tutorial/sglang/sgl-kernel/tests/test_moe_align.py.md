# test_moe_align.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_moe_align.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `MoE alignment` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `MoE alignment` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Imports and module setup
````python
import itertools
import sys

import pytest
import torch
import triton
import triton.language as tl
from sgl_kernel import moe_align_block_size, moe_sum
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 11-12: `is_hip` definition
````python
def is_hip() -> bool:
    return torch.version.hip is not None
````
**EN:** This section defines `is_hip` and implements the core logic associated with is hip.
**CN:** 该部分定义 `is_hip`，并实现与 is hip 相关的核心逻辑。

### Lines 15-15: Constants and configuration
````python
_is_hip = is_hip()
````
**EN:** This block defines shared constants or configuration values such as `_is_hip`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `_is_hip`），供后续函数或控制流程复用。

### Lines 18-19: `ceil_div` definition
````python
def ceil_div(a, b):
    return (a + b - 1) // b
````
**EN:** This section defines `ceil_div` and implements the core logic associated with ceil div.
**CN:** 该部分定义 `ceil_div`，并实现与 ceil div 相关的核心逻辑。

### Lines 22-38: `moe_align_block_size_stage1` definition
````python
@triton.jit
def moe_align_block_size_stage1(
    topk_ids_ptr,
    tokens_cnts_ptr,
    num_experts: tl.constexpr,
    numel: tl.constexpr,
    tokens_per_thread: tl.constexpr,
):
    pid = tl.program_id(0)
    start_idx = pid * tokens_per_thread
    off_c = (pid + 1) * num_experts

    for i in range(tokens_per_thread):
        if start_idx + i < numel:
            idx = tl.load(topk_ids_ptr + start_idx + i)
            token_cnt = tl.load(tokens_cnts_ptr + off_c + idx)
            tl.store(tokens_cnts_ptr + off_c + idx, token_cnt + 1)
````
**EN:** This section defines `moe_align_block_size_stage1` and implements the core logic associated with MoE alignment block size stage1.
**CN:** 该部分定义 `moe_align_block_size_stage1`，并实现与 MoE alignment block size stage1 相关的核心逻辑。

### Lines 41-51: `moe_align_block_size_stage2` definition
````python
@triton.jit
def moe_align_block_size_stage2(
    tokens_cnts_ptr,
    num_experts: tl.constexpr,
):
    pid = tl.program_id(0)
    last_cnt = 0
    for i in range(1, num_experts + 1):
        token_cnt = tl.load(tokens_cnts_ptr + i * num_experts + pid)
        last_cnt = last_cnt + token_cnt
        tl.store(tokens_cnts_ptr + i * num_experts + pid, last_cnt)
````
**EN:** This section defines `moe_align_block_size_stage2` and implements the core logic associated with MoE alignment block size stage2.
**CN:** 该部分定义 `moe_align_block_size_stage2`，并实现与 MoE alignment block size stage2 相关的核心逻辑。

### Lines 54-68: `moe_align_block_size_stage3` definition
````python
@triton.jit
def moe_align_block_size_stage3(
    total_tokens_post_pad_ptr,
    tokens_cnts_ptr,
    cumsum_ptr,
    num_experts: tl.constexpr,
    block_size: tl.constexpr,
):
    last_cumsum = 0
    off_cnt = num_experts * num_experts
    for i in range(1, num_experts + 1):
        token_cnt = tl.load(tokens_cnts_ptr + off_cnt + i - 1)
        last_cumsum = last_cumsum + tl.cdiv(token_cnt, block_size) * block_size
        tl.store(cumsum_ptr + i, last_cumsum)
    tl.store(total_tokens_post_pad_ptr, last_cumsum)
````
**EN:** This section defines `moe_align_block_size_stage3` and implements the core logic associated with MoE alignment block size stage3.
**CN:** 该部分定义 `moe_align_block_size_stage3`，并实现与 MoE alignment block size stage3 相关的核心逻辑。

### Lines 71-98: `moe_align_block_size_stage4` definition
````python
@triton.jit
def moe_align_block_size_stage4(
    topk_ids_ptr,
    sorted_token_ids_ptr,
    expert_ids_ptr,
    tokens_cnts_ptr,
    cumsum_ptr,
    num_experts: tl.constexpr,
    block_size: tl.constexpr,
    numel: tl.constexpr,
    tokens_per_thread: tl.constexpr,
):
    pid = tl.program_id(0)
    start_idx = tl.load(cumsum_ptr + pid)
    end_idx = tl.load(cumsum_ptr + pid + 1)

    for i in range(start_idx, end_idx, block_size):
        tl.store(expert_ids_ptr + i // block_size, pid)

    start_idx = pid * tokens_per_thread
    off_t = pid * num_experts

    for i in range(start_idx, tl.minimum(start_idx + tokens_per_thread, numel)):
        expert_id = tl.load(topk_ids_ptr + i)
        token_cnt = tl.load(tokens_cnts_ptr + off_t + expert_id)
        rank_post_pad = token_cnt + tl.load(cumsum_ptr + expert_id)
        tl.store(sorted_token_ids_ptr + rank_post_pad, i)
        tl.store(tokens_cnts_ptr + off_t + expert_id, token_cnt + 1)
````
**EN:** This section defines `moe_align_block_size_stage4` and implements the core logic associated with MoE alignment block size stage4.
**CN:** 该部分定义 `moe_align_block_size_stage4`，并实现与 MoE alignment block size stage4 相关的核心逻辑。

### Lines 101-145: `moe_align_block_size_triton` definition
````python
def moe_align_block_size_triton(
    topk_ids: torch.Tensor,
    num_experts: int,
    block_size: int,
    sorted_token_ids: torch.Tensor,
    expert_ids: torch.Tensor,
    num_tokens_post_pad: torch.Tensor,
) -> None:
    numel = topk_ids.numel()
    grid = (num_experts,)
    tokens_cnts = torch.zeros(
        (num_experts + 1, num_experts), dtype=torch.int32, device=topk_ids.device
    )
    cumsum = torch.zeros((num_experts + 1,), dtype=torch.int32, device=topk_ids.device)
    tokens_per_thread = ceil_div(numel, num_experts)

    moe_align_block_size_stage1[grid](
        topk_ids,
        tokens_cnts,
        num_experts,
        numel,
        tokens_per_thread,
    )
    moe_align_block_size_stage2[grid](
        tokens_cnts,
        num_experts,
    )
    moe_align_block_size_stage3[(1,)](
        num_tokens_post_pad,
        tokens_cnts,
        cumsum,
        num_experts,
        block_size,
    )
    moe_align_block_size_stage4[grid](
        topk_ids,
        sorted_token_ids,
        expert_ids,
        tokens_cnts,
        cumsum,
        num_experts,
        block_size,
        numel,
        tokens_per_thread,
    )
````
**EN:** This section defines `moe_align_block_size_triton` and implements the core logic associated with MoE alignment block size triton.
**CN:** 该部分定义 `moe_align_block_size_triton`，并实现与 MoE alignment block size triton 相关的核心逻辑。

### Lines 148-256: `test_moe_align_block_size_compare_implementations` definition
````python
@pytest.mark.parametrize(
    "block_size,num_tokens,topk,num_experts,pad_sorted_token_ids",
    list(
        itertools.product(
            [32, 64, 128, 256],  # block_size
            [1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048, 4096],  # num_tokens
            [1, 2, 4, 8, 16, 32, 64],  # topk
            [64, 160, 256, 257, 260, 264],  #  num_experts
            [True, False],  # pad_sorted_token_ids
        )
    ),
)
def test_moe_align_block_size_compare_implementations(
    block_size, num_tokens, topk, num_experts, pad_sorted_token_ids
):

    topk_ids = torch.argsort(torch.rand(num_tokens, num_experts, device="cuda"), dim=1)[
        :, :topk
    ]

    max_num_tokens_padded = topk_ids.numel() + (num_experts + 1) * (block_size - 1)
    if topk_ids.numel() < num_experts + 1:
        max_num_tokens_padded = topk_ids.numel() * block_size

    sorted_ids_cuda = torch.empty(
        (max_num_tokens_padded,), dtype=torch.int32, device=topk_ids.device
    )
    if not pad_sorted_token_ids:
        sorted_ids_cuda.fill_(topk_ids.numel())
    max_num_m_blocks = max_num_tokens_padded // block_size
    expert_ids_cuda = torch.zeros(
        (max_num_m_blocks,), dtype=torch.int32, device=topk_ids.device
    )
    num_tokens_post_pad_cuda = torch.empty(
        (1), dtype=torch.int32, device=topk_ids.device
    )
    cumsum_buffer = torch.empty(
        num_experts + 2, dtype=torch.int32, device=topk_ids.device
    )

    sorted_ids_triton = torch.empty_like(sorted_ids_cuda)
    sorted_ids_triton.fill_(topk_ids.numel())
    expert_ids_triton = torch.zeros_like(expert_ids_cuda)
    num_tokens_post_pad_triton = torch.empty_like(num_tokens_post_pad_cuda)

    moe_align_block_size(
        topk_ids,
        num_experts + 1,
        block_size,
        sorted_ids_cuda,
        expert_ids_cuda,
        num_tokens_post_pad_cuda,
        cumsum_buffer,
        pad_sorted_token_ids,
    )

    moe_align_block_size_triton(
        topk_ids,
        num_experts + 1,
        block_size,
        sorted_ids_triton,
        expert_ids_triton,
        num_tokens_post_pad_triton,
    )

    assert torch.allclose(expert_ids_cuda, expert_ids_triton, atol=0, rtol=0), (
        f"Expert IDs mismatch for block_size={block_size}, "
        f"num_tokens={num_tokens}, topk={topk}\n"
        f"CUDA expert_ids: {expert_ids_cuda}\n"
        f"Triton expert_ids: {expert_ids_triton}"
    )

    assert torch.allclose(
        num_tokens_post_pad_cuda, num_tokens_post_pad_triton, atol=0, rtol=0
    ), (
        f"Num tokens post pad mismatch for block_size={block_size}, "
        f"num_tokens={num_tokens}, topk={topk}\n"
        f"CUDA num_tokens_post_pad: {num_tokens_post_pad_cuda}\n"
        f"Triton num_tokens_post_pad: {num_tokens_post_pad_triton}"
    )

    # Select an expert to check
    expert_idx = expert_ids_cuda.max().item()

    # Get the first and last block id where expert_ids_cuda == expert_idx
    matching_indices = torch.where(expert_ids_cuda == expert_idx)[0]
    block_sorted_start = matching_indices[0].item() * block_size
    block_sorted_end = min(
        (matching_indices[-1].item() + 1) * block_size, num_tokens_post_pad_cuda.item()
    )

    selected_sorted_ids_cuda = sorted_ids_cuda[
        block_sorted_start:block_sorted_end
    ].sort()[0]
    selected_sorted_ids_triton = sorted_ids_triton[
        block_sorted_start:block_sorted_end
    ].sort()[0]

    assert torch.allclose(
        selected_sorted_ids_cuda,
        selected_sorted_ids_triton,
        atol=0,
        rtol=0,
    ), (
        f"Sorted IDs mismatch for block_size={block_size}, "
        f"num_tokens={num_tokens}, topk={topk}\n"
        f"CUDA sorted_ids: {selected_sorted_ids_cuda}\n"
        f"Triton sorted_ids: {selected_sorted_ids_triton}"
    )
````
**EN:** This section defines the test `test_moe_align_block_size_compare_implementations`. It sets up inputs, runs the target path, and checks the expected result. It also performs explicit assertions or shape checks before continuing. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_moe_align_block_size_compare_implementations`。它会准备输入、执行目标路径，并检查预期结果。 它还会在继续执行前进行显式断言或形状检查。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 259-271: `test_moe_sum` definition
````python
@pytest.mark.parametrize("m", [1, 33, 64, 222])
@pytest.mark.parametrize("topk", [2, 6])
@pytest.mark.parametrize("k", [128, 511, 1024])
@pytest.mark.parametrize("dtype", [torch.float32, torch.float16, torch.bfloat16])
@pytest.mark.skipif(_is_hip, reason="Skip for AMD GPU")
def test_moe_sum(m: int, topk: int, k: int, dtype: torch.dtype):
    input = torch.randn((m, topk, k), device="cuda", dtype=dtype)
    actual = torch.empty((m, k), device="cuda", dtype=dtype)

    expected = input.sum(dim=1)
    moe_sum(input, actual)

    torch.testing.assert_close(actual, expected, atol=2e-2, rtol=0)
````
**EN:** This section defines the test `test_moe_sum`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_moe_sum`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 274-275: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `is_hip`, `ceil_div`, `moe_align_block_size_stage1`, `moe_align_block_size_stage2`, `moe_align_block_size_stage3`, `moe_align_block_size_stage4`, `moe_align_block_size_triton`, `test_moe_align_block_size_compare_implementations`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化, Triton benchmarking / Triton 基准工具
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`
- **External / 外部**: `itertools`, `pytest`, `sys`, `torch`, `triton`, `triton.language`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
