# test_merge_attn_states.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/attention/test_merge_attn_states.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / attention / test_merge_attn_states, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / attention / test_merge_attn_states 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-16)
```python
import pytest
import torch

from vllm._custom_ops import (
    merge_attn_states as merge_attn_states_cuda,
)
from vllm._custom_ops import (
    scaled_fp8_quant,
)
from vllm.platforms import current_platform
from vllm.v1.attention.ops.triton_merge_attn_states import (
    merge_attn_states as merge_attn_states_triton,
)
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; and vLLM components like vllm._custom_ops, vllm.platforms, vllm.v1.attention.ops.triton_merge_attn_states.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；vLLM 内部组件，例如 vllm._custom_ops、vllm.platforms、vllm.v1.attention.ops.triton_merge_attn_states。

### Function `merge_attn_states_torch` (lines 21-65)
```python
def merge_attn_states_torch(
    output: torch.Tensor,  # [NUM_TOKENS, NUM_HEADS, HEAD_SIZE]
    prefix_output: torch.Tensor,  # [NUM_TOKENS, NUM_HEADS, HEAD_SIZE]
    prefix_lse: torch.Tensor,  # [NUM_HEADS, NUM_TOKENS]
    suffix_output: torch.Tensor,  # [NUM_TOKENS, NUM_HEADS, HEAD_SIZE]
    suffix_lse: torch.Tensor,  # [NUM_HEADS, NUM_TOKENS]
    output_lse: torch.Tensor | None = None,  # [NUM_HEADS, NUM_TOKENS]
    prefill_tokens_with_context: int | None = None,
    output_scale: torch.Tensor | None = None,  # scalar, per-tensor FP8 scale
):
    # Apply prefill_tokens_with_context mask if needed
    if prefill_tokens_with_context is None:
        prefill_tokens_with_context = output.shape[0]
    p_lse = prefix_lse
    s_lse = suffix_lse
    # inf -> -inf
    p_lse[p_lse == torch.inf] = -torch.inf
    s_lse[s_lse == torch.inf] = -torch.inf
    # max_lse [NUM_HEADS, NUM_TOKENS]
    max_lse = torch.maximum(p_lse, s_lse)

    mask = torch.ones((prefix_lse.shape[1], 1, 1), device=p_lse.device)
    mask[prefill_tokens_with_context:].fill_(0)
    p_lse = p_lse - max_lse
    s_lse = s_lse - max_lse
    p_lse_exp = torch.exp(p_lse)
    s_lse_exp = torch.exp(s_lse)
    out_se = p_lse_exp + s_lse_exp
    if output_lse is not None:
        output_lse = torch.log(out_se) + max_lse
        output_lse[prefill_tokens_with_context:] = suffix_lse[
            prefill_tokens_with_context:
        ]
    p_scale = p_lse_exp / out_se  # [NUM_HEADS, NUM_TOKENS]
    s_scale = s_lse_exp / out_se  # [NUM_HEADS, NUM_TOKENS]
    p_scale = torch.transpose(p_scale, 0, 1).unsqueeze(2)  # [NUM_TOKENS, NUM_HEADS, 1]
    s_scale = torch.transpose(s_scale, 0, 1).unsqueeze(2)  # [NUM_TOKENS, NUM_HEADS, 1]
    output = prefix_output * p_scale * mask + suffix_output * (
        s_scale * mask + (1 - mask)
    )
    if output_scale is not None:
        shape = output.shape
        output, _ = scaled_fp8_quant(output.float().view(-1, shape[-1]), output_scale)
        output = output.view(shape)
    return output, output_lse
```
**EN:** This helper function implements the shared logic for merge attn states torch. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 merge attn states torch 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 68-73)
```python
NUM_BATCH_TOKENS = [256, 512, 613, 1024, 1536, 4096]
NUM_QUERY_HEADS = [4, 8, 16, 32, 48, 64]
HEAD_SIZES = [32, 48, 64, 96, 128, 256]
DTYPES = [torch.float32, torch.half, torch.bfloat16]

all_case_info: list[tuple] = []
```
**EN:** This block centralizes shared constants and parameter grids, including NUM_BATCH_TOKENS, NUM_QUERY_HEADS, HEAD_SIZES, DTYPES. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 NUM_BATCH_TOKENS、NUM_QUERY_HEADS、HEAD_SIZES、DTYPES。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `generate_markdown_table` (lines 76-112)
```python
def generate_markdown_table():
    global all_case_info
    table_header = (
        "| tokens | heads | headsize | dtype "
        "| device | torch | triton | cuda | speedup |"
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
            avg_time_torch_kernel,
            avg_time_triton_kernel,
            avg_time_cuda_kernel,
            performance_improved,
        ) = info
        dtype = shortly_dtype(dtype)
        device = shortly_device(device)
        print(
            f"| {num_tokens} | {num_heads} | {head_size} "
            f"| {dtype} | {device} | {avg_time_torch_kernel:.5f}ms "
            f"| {avg_time_triton_kernel:.5f}ms "
            f"| {avg_time_cuda_kernel:.5f}ms "
            f"| {performance_improved:.4f}x |"
        )
```
**EN:** This helper function implements the shared logic for generate markdown table. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 generate markdown table 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_merge_attn_states` (lines 115-392)
```python
@pytest.mark.parametrize("use_fp8", [False, True])
@pytest.mark.parametrize("prefill_tokens_with_context", [None, 128])
@pytest.mark.parametrize("num_tokens", NUM_BATCH_TOKENS)
@pytest.mark.parametrize("num_query_heads", NUM_QUERY_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("input_dtype", DTYPES)
@torch.inference_mode()
def test_merge_attn_states(
    prefill_tokens_with_context: int | None,
    num_tokens: int,
    num_query_heads: int,
    head_size: int,
    input_dtype: torch.dtype,
    use_fp8: bool,
):
    if not current_platform.is_cuda():
        pytest.skip(
            "Currently only support compare triton merge_attn_states "
            "with custom cuda merge_attn_states kernel"
        )

    NUM_TOKENS = num_tokens
    NUM_HEADS = num_query_heads
    HEAD_SIZE = head_size

    # When use_fp8 is set, inputs stay as input_dtype (bf16/fp16/fp32)
    # and output becomes FP8.
    output_dtype = input_dtype
    output_scale = None
    if use_fp8:
        output_dtype = current_platform.fp8_dtype()
        output_scale = torch.tensor([0.05], dtype=torch.float32, device="cuda")

    print(
        f"\nNUM_TOKENS:{NUM_TOKENS}, NUM_HEADS:{NUM_HEADS}, "
        f"HEAD_SIZE:{HEAD_SIZE}, input_dtype: {input_dtype}, "
        f"output_dtype: {output_dtype}, use_fp8: {use_fp8}, "
        f"prefill_tokens_with_context: {prefill_tokens_with_context}, "
        f"Device: {current_platform.get_device_name()}"
    )
# ... excerpt ...
    _diff = diff(output_torch.float() * scale, output_cuda.float() * scale)
    print(f"  (CUDA vs Torch) : {_diff}")
    _diff = diff(output_ref.float() * scale, output_cuda.float() * scale)
    print(f"  (CUDA vs Triton): {_diff}")
    print("-" * 100)

    torch.testing.assert_close(
        output_lse_cuda.float(), output_lse_ref.float(), atol=atol, rtol=rtol
    )
    print("Output LSE all match, max abs diff:")
    print(f"(Triton vs Torch) : {diff(output_lse_torch, output_lse_ref)}")
    print(f"  (CUDA vs Torch) : {diff(output_lse_torch, output_lse_cuda)}")
    print(f"  (CUDA vs Triton): {diff(output_lse_ref, output_lse_cuda)}")
    print("-" * 100)

    print(
        "All output values test passed! All inf values "
        "are correctly replaced with -inf."
    )
    print("-" * 100)

    device = current_platform.get_device_name()
    all_case_info.append(
        (
            NUM_TOKENS,
            NUM_HEADS,
            HEAD_SIZE,
            output_dtype,
            device,
            avg_time_torch_kernel,
            avg_time_triton_kernel,
            avg_time_cuda_kernel,
            performance_improved,
        )
    )
    if len(all_case_info) == (
        len(NUM_BATCH_TOKENS) * len(HEAD_SIZES) * len(NUM_QUERY_HEADS) * len(DTYPES)
    ):
        generate_markdown_table()
```
**EN:** This pytest case verifies merge attn states. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as prefill_tokens_with_context, num_tokens, num_query_heads, head_size. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 merge attn states 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 prefill_tokens_with_context、num_tokens、num_query_heads、head_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm._custom_ops -> merge_attn_states`
- `vllm._custom_ops -> scaled_fp8_quant`
- `vllm.platforms -> current_platform`
- `vllm.v1.attention.ops.triton_merge_attn_states -> merge_attn_states`
