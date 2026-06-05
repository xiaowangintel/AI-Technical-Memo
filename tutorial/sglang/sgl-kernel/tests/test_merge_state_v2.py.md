# test_merge_state_v2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_merge_state_v2.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `merge state v2` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `merge state v2` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Imports and module setup
````python
import sys
from typing import Optional

import pytest
import torch
import triton
import triton.language as tl
from sgl_kernel import merge_state_v2
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 11-66: `merge_state_kernel` definition
````python
@triton.jit
def merge_state_kernel(
    output,  # [NUM_TOKENS, NUM_HEADS, HEAD_SIZE] v_merged
    output_lse,  # [NUM_TOKENS, NUM_HEADS] s_merged
    prefix_output,  # [NUM_TOKENS, NUM_HEADS, HEAD_SIZE] v_a
    prefix_lse,  # [NUM_TOKENS, NUM_HEADS] s_a
    suffix_output,  # [NUM_TOKENS, NUM_HEADS, HEAD_SIZE] v_b
    suffix_lse,  # [NUM_TOKENS, NUM_HEADS] s_b
    HEAD_SIZE: tl.constexpr,
    PADDED_HEAD_SIZE: tl.constexpr,
    OUTPUT_LSE: tl.constexpr,
):
    token_idx = tl.program_id(0)
    num_tokens = tl.num_programs(0)
    head_idx = tl.program_id(1)
    num_heads = tl.num_programs(1)

    p_lse = tl.load(prefix_lse + token_idx * num_heads + head_idx)
    s_lse = tl.load(suffix_lse + token_idx * num_heads + head_idx)
    p_lse = float("-inf") if p_lse == float("inf") else p_lse
    s_lse = float("-inf") if s_lse == float("inf") else s_lse

    max_lse = tl.maximum(p_lse, s_lse)
    p_lse = p_lse - max_lse
    s_lse = s_lse - max_lse
    out_se = tl.exp(p_lse) + tl.exp(s_lse)

    if OUTPUT_LSE:
        out_lse = tl.log(out_se) + max_lse
        tl.store(output_lse + token_idx * num_heads + head_idx, out_lse)

    head_arange = tl.arange(0, PADDED_HEAD_SIZE)
    head_mask = head_arange < HEAD_SIZE
    p_out = tl.load(
        prefix_output
        + token_idx * num_heads * HEAD_SIZE
        + head_idx * HEAD_SIZE
        + head_arange,
        mask=head_mask,
    )
    s_out = tl.load(
        suffix_output
        + token_idx * num_heads * HEAD_SIZE
        + head_idx * HEAD_SIZE
        + head_arange,
        mask=head_mask,
    )

    p_scale = tl.exp(p_lse) / out_se
    s_scale = tl.exp(s_lse) / out_se
    out = p_out * p_scale + s_out * s_scale
    tl.store(
        output + token_idx * num_heads * HEAD_SIZE + head_idx * HEAD_SIZE + head_arange,
        out,
        mask=head_mask,
    )
````
**EN:** This section defines `merge_state_kernel`. It merges the `state kernel` path used by the module.
**CN:** 该部分定义 `merge_state_kernel`。它负责合并模块中与 `state kernel` 相关的处理路径。

### Lines 69-98: `merge_state_triton` definition
````python
def merge_state_triton(
    prefix_output: torch.Tensor,
    prefix_lse: torch.Tensor,
    suffix_output: torch.Tensor,
    suffix_lse: torch.Tensor,
    output: Optional[torch.Tensor] = None,
    output_lse: Optional[torch.Tensor] = None,
) -> None:
    num_tokens = output.shape[0]
    num_query_heads = output.shape[1]
    head_size = output.shape[2]
    padded_head_size = triton.next_power_of_2(head_size)
    # Avoid creating new tensors if they are already provided
    if output is None:
        output = torch.empty_like(prefix_output)
    if output_lse is None:
        output_lse = torch.empty_like(prefix_lse)

    merge_state_kernel[(num_tokens, num_query_heads)](
        output,
        output_lse,
        prefix_output,
        prefix_lse,
        suffix_output,
        suffix_lse,
        head_size,
        padded_head_size,
        output_lse is not None,
    )
    return output, output_lse
````
**EN:** This section defines `merge_state_triton`. It merges the `state triton` path used by the module.
**CN:** 该部分定义 `merge_state_triton`。它负责合并模块中与 `state triton` 相关的处理路径。

### Lines 99-101: Comments and local context
````python


# Naive PyTorch Implements of Merge Attn States
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 102-134: `merge_state_torch` definition
````python
def merge_state_torch(
    prefix_output: torch.Tensor,  # [NUM_TOKENS, NUM_HEADS, HEAD_SIZE]
    prefix_lse: torch.Tensor,  # [NUM_TOKENS, NUM_HEADS]
    suffix_output: torch.Tensor,  # [NUM_TOKENS, NUM_HEADS, HEAD_SIZE]
    suffix_lse: torch.Tensor,  # [NUM_TOKENS, NUM_HEADS]
    output: Optional[torch.Tensor] = None,  # [NUM_TOKENS, NUM_HEADS, HEAD_SIZE]
    output_lse: Optional[torch.Tensor] = None,  # [NUM_TOKENS, NUM_HEADS]
):
    # Avoid creating new tensors if they are already provided
    if output is None:
        output = torch.empty_like(prefix_output)
    if output_lse is None:
        output_lse = torch.empty_like(prefix_lse)
    p_lse = prefix_lse
    s_lse = suffix_lse
    # inf -> -inf
    p_lse[p_lse == torch.inf] = -torch.inf
    s_lse[s_lse == torch.inf] = -torch.inf
    # max_lse [NUM_HEADS, NUM_TOKENS]
    max_lse = torch.maximum(p_lse, s_lse)
    p_lse = p_lse - max_lse
    s_lse = s_lse - max_lse
    p_lse_exp = torch.exp(p_lse)
    s_lse_exp = torch.exp(s_lse)
    out_se = p_lse_exp + s_lse_exp
    if output_lse is not None:
        output_lse = torch.log(out_se) + max_lse
    p_scale = p_lse_exp / out_se
    s_scale = s_lse_exp / out_se
    p_scale = p_scale.unsqueeze(2)  # [NUM_TOKENS, NUM_HEADS, 1]
    s_scale = s_scale.unsqueeze(2)  # [NUM_TOKENS, NUM_HEADS, 1]
    output = prefix_output * p_scale + suffix_output * s_scale
    return output, output_lse
````
**EN:** This section defines `merge_state_torch`. It merges the `state torch` path used by the module.
**CN:** 该部分定义 `merge_state_torch`。它负责合并模块中与 `state torch` 相关的处理路径。

### Lines 137-142: Constants and configuration
````python
NUM_BATCH_TOKENS = [256, 512, 613, 1024, 1536]
NUM_QUERY_HEADS = [8, 16, 32]
HEAD_SIZES = [32, 48, 64, 128, 256]
DTYPES = [torch.half, torch.bfloat16]

all_case_info: list[tuple] = []
````
**EN:** This block defines shared constants or configuration values such as `NUM_BATCH_TOKENS`, `NUM_QUERY_HEADS`, `HEAD_SIZES`, `DTYPES`, `all_case_info`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `NUM_BATCH_TOKENS`, `NUM_QUERY_HEADS`, `HEAD_SIZES`, `DTYPES`, `all_case_info`），供后续函数或控制流程复用。

### Lines 145-181: `generate_markdown_table` definition
````python
def generate_markdown_table():
    global all_case_info
    table_header = (
        "| tokens | heads | headsize | dtype "
        "| device | torch | triton | v2 | speedup(vs triton) |"
    )
    table_separator = "| --- | --- | --- | --- | --- | --- | --- | --- | --- |"

    def shortly_dtype(dtype: torch.dtype) -> str:
        return str(dtype).removeprefix("torch.")

    def shortly_device(device: str) -> str:
        return device.removeprefix("NVIDIA").strip()

    print(table_header)
    print(table_separator)
    for info in all_case_info:
        (
            num_tokens,
            num_heads,
            head_size,
            dtype,
            device,
            time_torch,
            time_triton,
            time_v2,
        ) = info
        dtype = shortly_dtype(dtype)
        device = shortly_device(device)
        improved_triton = time_triton / time_v2
        print(
            f"| {num_tokens} | {num_heads} | {head_size} "
            f"| {dtype} | {device} | {time_torch:.4f}ms "
            f"| {time_triton:.4f}ms "
            f"| {time_v2:.4f}ms "
            f"| {improved_triton:.4f}x |"
        )
````
**EN:** This section defines `generate_markdown_table` and implements the core logic associated with generate markdown table.
**CN:** 该部分定义 `generate_markdown_table`，并实现与 generate markdown table 相关的核心逻辑。

### Lines 184-377: `test_merge_attn_states` definition
````python
@pytest.mark.parametrize("num_tokens", NUM_BATCH_TOKENS)
@pytest.mark.parametrize("num_query_heads", NUM_QUERY_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("output_dtype", DTYPES)
@torch.inference_mode()
def test_merge_attn_states(
    num_tokens: int, num_query_heads: int, head_size: int, output_dtype: torch.dtype
):
    if not torch.cuda.is_available():
        pytest.skip(
            "Currently only support compare triton merge_attn_states "
            "with custom cuda merge_attn_states kernel"
        )

    NUM_TOKENS = num_tokens
    NUM_HEADS = num_query_heads
    HEAD_SIZE = head_size

    print(
        f"\nNUM_TOKENS:{NUM_TOKENS}, NUM_HEADS:{NUM_HEADS}, "
        f"HEAD_SIZE:{HEAD_SIZE}, DTYPE: {output_dtype}, "
        f"Device: {torch.cuda.get_device_name()}"
    )

    # prefix_lse and suffix_lse contain inf and normal values
    prefix_lse = torch.randn(NUM_TOKENS, NUM_HEADS, dtype=torch.float32, device="cuda")
    suffix_lse = torch.randn(NUM_TOKENS, NUM_HEADS, dtype=torch.float32, device="cuda")

    # Generate boolean masks
    mask_prefix = torch.rand(NUM_TOKENS, NUM_HEADS) < 0.1
    mask_suffix = torch.rand(NUM_TOKENS, NUM_HEADS) < 0.1
    # Ensure that the same position is not True at the same time
    combined_mask = torch.logical_and(mask_prefix, mask_suffix)
    mask_prefix = torch.logical_and(mask_prefix, ~combined_mask)
    mask_suffix = torch.logical_and(mask_suffix, ~combined_mask)

    prefix_lse[mask_prefix] = float("inf")
    suffix_lse[mask_suffix] = float("inf")

    # Other input tensors (need to be initialized but
    # no actual calculation needed)
    output = torch.zeros(
        (NUM_TOKENS, NUM_HEADS, HEAD_SIZE), dtype=output_dtype, device="cuda"
    )
    output_lse = torch.zeros(
        (NUM_TOKENS, NUM_HEADS), dtype=torch.float32, device="cuda"
    )
    prefix_output = torch.randn(
        (NUM_TOKENS, NUM_HEADS, HEAD_SIZE), dtype=output_dtype, device="cuda"
    )
    suffix_output = torch.randn(
        (NUM_TOKENS, NUM_HEADS, HEAD_SIZE), dtype=output_dtype, device="cuda"
    )

    warmup_times = 2
    repeat_times = 20

    def perf_kernel_fn(
        output_fn: torch.Tensor,
        output_lse_fn: torch.Tensor,
        kernel_fn: callable,
        fn_type: str = "torch",
    ):
        # Avoid inplace inf -> -inf, we have to use prefix_lse
        # and suffix_lse for other kernel.
        if fn_type == "torch":
            prefix_lse_ = prefix_lse.clone()
            suffix_lse_ = suffix_lse.clone()
        else:
            prefix_lse_ = prefix_lse
            suffix_lse_ = suffix_lse

        total_time = 0
        start = torch.cuda.Event(enable_timing=True)
        end = torch.cuda.Event(enable_timing=True)

        try:
            for _ in range(warmup_times):
                output_fn, output_lse_fn = kernel_fn(
                    prefix_output,
                    prefix_lse_,
                    suffix_output,
                    suffix_lse_,
                    output_fn,
                    output_lse_fn,
                )
            torch.cuda.synchronize()

            for _ in range(repeat_times):
                start.record()
                output_fn, output_lse_fn = kernel_fn(
                    prefix_output,
                    prefix_lse_,
                    suffix_output,
                    suffix_lse_,
                    output_fn,
                    output_lse_fn,
                )
                end.record()
                torch.cuda.synchronize()
                total_time += start.elapsed_time(end)

            avg_time = total_time / repeat_times
            return avg_time, output_fn, output_lse_fn
        except Exception as e:
            return 0, output_fn, output_lse_fn

    # 0. Run the Torch kernel
    output_torch = output.clone()
    output_lse_torch = output_lse.clone()
    time_torch, output_torch, output_lse_torch = perf_kernel_fn(
        output_torch, output_lse_torch, merge_state_torch, fn_type="torch"
    )

    # 1. Run the Triton kernel
    output_ref_triton = output.clone()
    output_lse_ref_triton = output_lse.clone()
    time_triton, output_ref_triton, output_lse_ref_triton = perf_kernel_fn(
        output_ref_triton,
        output_lse_ref_triton,
        merge_state_triton,
        fn_type="triton",
    )

    # 2. Run the merge_state V2 kernel
    output_v2 = output.clone()
    output_lse_v2 = output_lse.clone()
    time_v2, output_v2, output_lse_v2 = perf_kernel_fn(
        output_v2, output_lse_v2, merge_state_v2, fn_type="cuda_v2"
    )

    # 3. Performance compare
    improved = time_triton / time_v2
    print(f"  Torch time: {time_torch:.6f}ms")
    print(f" Triton time: {time_triton:.6f}ms")
    print(f"CUDA v2 time: {time_v2:.6f}ms, Performance: {improved:.5f}x")
    print("-" * 100)

    # 4. Correctness compare
    # Liger Kernel: Efficient Triton Kernels for LLM Training
    # https://arxiv.org/pdf/2410.10989, 3.3 Correctness
    # use rtol = 1e-2 for bfloat16.
    rtol = 1e-2 if output_dtype == torch.bfloat16 else 1e-3

    def diff(a: torch.Tensor, b: torch.Tensor):
        max_diff = torch.max(torch.abs(a.float() - b.float()))
        return max_diff

    # Use Triton output as reference because we want to replace
    # the Triton kernel with custom CUDA kernel for merge attn
    # states operation.
    output_ref = output_ref_triton
    output_lse_ref = output_lse_ref_triton
    torch.testing.assert_close(
        output_v2.float(), output_ref.float(), atol=1e-3, rtol=rtol
    )
    print("Output all match, max abs diff:")
    print(f"(Triton  vs Torch) : {diff(output_torch, output_ref)}")
    print(f"(CUDA v2 vs Torch) : {diff(output_torch, output_v2)}")
    print(f"(CUDA v2 vs Triton): {diff(output_ref, output_v2)}")
    print("-" * 100)

    torch.testing.assert_close(
        output_lse_v2.float(), output_lse_ref.float(), atol=1e-3, rtol=rtol
    )
    print("Output LSE all match, max abs diff:")
    print(f"(Triton  vs Torch) : {diff(output_lse_torch, output_lse_ref)}")
    print(f"(CUDA v2 vs Torch) : {diff(output_lse_torch, output_lse_v2)}")
    print(f"(CUDA v2 vs Triton): {diff(output_lse_ref, output_lse_v2)}")
    print("-" * 100)

    print(
        "All output values test passed! All inf values "
        "are correctly replaced with -inf."
    )
    print("-" * 100)

    device = torch.cuda.get_device_name()
    all_case_info.append(
        (
            NUM_TOKENS,
            NUM_HEADS,
            HEAD_SIZE,
            output_dtype,
            device,
            time_torch,
            time_triton,
            time_v2,
        )
    )
    if len(all_case_info) == (
        len(NUM_BATCH_TOKENS) * len(HEAD_SIZES) * len(NUM_QUERY_HEADS) * len(DTYPES)
    ):
        generate_markdown_table()
````
**EN:** This section defines the test `test_merge_attn_states`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_merge_attn_states`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 380-381: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `merge_state_kernel`, `merge_state_triton`, `merge_state_torch`, `generate_markdown_table`, `test_merge_attn_states`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化, Triton benchmarking / Triton 基准工具
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`
- **External / 外部**: `pytest`, `sys`, `torch`, `triton`, `triton.language`, `typing`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
