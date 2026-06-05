# test_cutlass_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/quant/test_cutlass_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `cutlass moe` scenario in `test/manual/quant`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/quant` 中的 `cutlass moe` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and shared helpers / 导入与共享辅助项
```python
import argparse

import torch
import triton  # Added import
import triton.testing  # Added import
from transformers import AutoConfig

from sglang.srt.layers.moe.cutlass_moe import cutlass_fused_experts_fp8
from sglang.srt.layers.moe.moe_runner.base import MoeRunnerConfig
from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe import fused_experts
from sglang.srt.layers.moe.topk import StandardTopKOutput


# Copy from: https://github.com/deepseek-ai/DeepGEMM/blob/main/deep_gemm/utils.py
```
**EN:** This range imports `argparse`, `torch`, `triton  # Added import` and `triton.testing  # Added import`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 15-19: Helper routines around calc_diff / 辅助例程
```python
def calc_diff(x, y):
    x, y = x.double(), y.double()
    denominator = (x * x + y * y).sum()
    sim = 2 * (x * y).sum() / denominator
    return 1 - sim
```
**EN:** This range implements helper routine(s) `calc_diff` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `double`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 22-38: Helper routines around get_model_config / 辅助例程
```python
def get_model_config(tp_size: int):
    config = AutoConfig.from_pretrained(
        "deepseek-ai/Deepseek-R1", trust_remote_code=True
    )
    E = config.n_routed_experts
    topk = config.num_experts_per_tok
    intermediate_size = config.moe_intermediate_size
    shard_intermediate_size = 2 * intermediate_size // tp_size

    return {
        "num_experts": E,
        "topk": topk,
        "hidden_size": config.hidden_size,
        "shard_intermediate_size": shard_intermediate_size,
        "dtype": config.dtype,
        "block_shape": config.quantization_config["weight_block_size"],
    }
```
**EN:** This range implements helper routine(s) `get_model_config` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `from_pretrained`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 41-60: Helper routines around to_fp8 / 辅助例程
```python
def to_fp8(tensor: torch.Tensor) -> torch.Tensor:
    """Converts tensor to FP8 E4M3, scaling values to fit the range."""
    finfo = torch.finfo(torch.float8_e4m3fn)
    # Calculate max absolute value safely
    max_val = torch.max(torch.abs(tensor))
    # Avoid division by zero if tensor is all zeros
    if max_val == 0:
        scale_factor = 1.0
    else:
        # Scale factor to bring the max value to finfo.max
        scale_factor = finfo.max / max_val

    # Apply scaling
    scaled_tensor = tensor * scale_factor

    # Clamp and convert
    fp8_tensor = scaled_tensor.clamp(min=finfo.min, max=finfo.max).to(
        dtype=torch.float8_e4m3fn
    )
    return fp8_tensor
```
**EN:** This range implements helper routine(s) `to_fp8` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `finfo`, `abs`, `clamp` and `to`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 63-87: Helper routines around run_test / 辅助例程
```python
def run_test(tp_size, batch_size, model_config, check=False):
    print(f"\n--- Batch Size: {batch_size} ---")
    torch.set_default_device("cuda")
    torch.cuda.manual_seed_all(42)  # For reproducible random numbers

    E = model_config["num_experts"]
    topk = model_config["topk"]
    H = model_config["hidden_size"]
    I = model_config["shard_intermediate_size"]
    block_shape = model_config["block_shape"]  # Tuple (BLOCK_N, BLOCK_K)
    dtype = model_config["dtype"]  # e.g., torch.bfloat16

    print(
        f"Config: E={E}, topk={topk}, H={H}, I_shard={I}, dtype={dtype}, block_shape={block_shape}"
    )

    # --- Input Data ---
    # Use bf16/fp16 for input activation based on model config
    x = torch.randn((batch_size, H), device="cuda", dtype=dtype)
    # --- Weights (Generate in higher precision, then convert to FP8) ---
    # Generate weights suitable for FP8 conversion (e.g., scaled appropriately)
    w1_hp = torch.randn((E, I, H), device="cuda", dtype=torch.float32)
    w2_hp = torch.randn((E, H, I // 2), device="cuda", dtype=torch.float32)

    w1 = to_fp8(w1_hp)
```
**EN:** This range implements helper routine(s) `run_test` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `set_default_device`, `manual_seed_all`, `Tuple` and `randn`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 88-112: Scenario logic / 场景逻辑
```python
    w2 = to_fp8(w2_hp)

    # --- Scales for FP8 Weights ---
    block_n, block_k = block_shape
    # Calculate number of blocks needed
    w1_blocks_dim1 = (I + block_n - 1) // block_n
    w1_blocks_dim2 = (H + block_k - 1) // block_k
    w2_blocks_dim1 = (H + block_n - 1) // block_n
    w2_blocks_dim2 = (I // 2 + block_k - 1) // block_k

    # Scales are typically float32 or float16/bfloat16
    scale_dtype = torch.float32  # Or dtype if scales match model dtype
    w1_scale = torch.full(
        (E, w1_blocks_dim1, w1_blocks_dim2), 1, device="cuda", dtype=scale_dtype
    )  # Avoid zero scales
    w2_scale = torch.full(
        (E, w2_blocks_dim1, w2_blocks_dim2), 1, device="cuda", dtype=scale_dtype
    )  # Avoid zero scales

    # --- Routing Information ---
    topk_weights = torch.softmax(
        torch.rand(batch_size, topk, device="cuda", dtype=dtype), dim=-1
    )
    topk_ids = torch.randint(0, E, (batch_size, topk), dtype=torch.int32, device="cuda")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `to_fp8`, `full`, `softmax` and `rand`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 113-137: Scenario logic / 场景逻辑
```python
    a1_strides = torch.full((E,), H, dtype=torch.int64, device="cuda")
    c1_strides = torch.full((E,), I, dtype=torch.int64, device="cuda")
    a2_strides = torch.full((E,), I // 2, dtype=torch.int64, device="cuda")
    c2_strides = torch.full((E,), H, dtype=torch.int64, device="cuda")

    workspace = torch.empty(
        (7182 * 1024), device="cuda", dtype=torch.uint8
    )  # Allocate sufficient workspace
    # Pointer arrays (often filled by the kernel or a prep step, but needed as args)
    a_ptrs = torch.empty((E,), dtype=torch.int64, device="cuda")
    b_ptrs = torch.empty((E,), dtype=torch.int64, device="cuda")
    out_ptrs = torch.empty((E,), dtype=torch.int64, device="cuda")
    a_scales_ptrs = torch.empty((E,), dtype=torch.int64, device="cuda")
    b_scales_ptrs = torch.empty((E,), dtype=torch.int64, device="cuda")
    expert_offsets = torch.empty((E + 1,), dtype=torch.int32, device="cuda")
    problem_sizes1 = torch.empty((E, 3), dtype=torch.int32, device="cuda")
    problem_sizes2 = torch.empty((E, 3), dtype=torch.int32, device="cuda")

    enable_es = (False, False)
    if torch.cuda.get_device_name(torch.cuda.current_device()) == "NVIDIA H200":
        enable_es = (False, True)
    elif torch.cuda.get_device_name(torch.cuda.current_device()) == "NVIDIA H20":
        enable_es = (True, True)

    # --- Lambdas for Benchmarking ---
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Representative call sites include `full`, `empty`, `arrays` and `get_device_name`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 138-162: Scenario logic / 场景逻辑
```python
    cutlass_lambda = lambda: cutlass_fused_experts_fp8(
        x,
        w1.transpose(1, 2),  # Transposed
        w2.transpose(1, 2),  # Transposed
        w1_scale.transpose(1, 2),
        w2_scale.transpose(1, 2),
        topk_weights,
        topk_ids,
        a1_strides,
        c1_strides,
        a2_strides,
        c2_strides,
        workspace,
        a_ptrs,
        b_ptrs,
        out_ptrs,
        a_scales_ptrs,
        b_scales_ptrs,
        expert_offsets,
        problem_sizes1,
        problem_sizes2,
        enable_es=enable_es,
    )

    topk_output = StandardTopKOutput(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `cutlass_fused_experts_fp8`, `transpose` and `StandardTopKOutput`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 163-187: Scenario logic / 场景逻辑
```python
        topk_weights=topk_weights,
        topk_ids=topk_ids,
        router_logits=torch.randn(
            (batch_size, topk), device=topk_weights.device, dtype=dtype
        ),
    )

    moe_runner_config = MoeRunnerConfig(
        num_experts=E,
        top_k=topk,
        hidden_size=H,
        intermediate_size_per_partition=I,
        params_dtype=dtype,
        activation="silu",
        inplace=False,
    )

    # Note: Triton expects non-transposed weights
    triton_lambda = lambda: fused_experts(
        x,
        w1,
        w2,
        topk_output,
        moe_runner_config,
        use_fp8_w8a8=True,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`, `MoeRunnerConfig` and `fused_experts`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 188-212: Scenario logic / 场景逻辑
```python
        w1_scale=w1_scale,
        w2_scale=w2_scale,
        block_shape=block_shape,
    )

    # --- Warmup ---
    print("Warming up...")
    for _ in range(10):
        _ = cutlass_lambda()
        _ = triton_lambda()
    torch.cuda.synchronize()

    # --- Benchmarking ---
    quantiles = [0.5, 0.2, 0.8]
    print(f"Benchmarking Cutlass fused_experts...")
    cutlass_ms, cutlass_min, cutlass_max = triton.testing.do_bench_cudagraph(
        cutlass_lambda, rep=1000, quantiles=quantiles
    )

    print(f"Benchmarking Triton fused_experts...")
    triton_ms, triton_min, triton_max = triton.testing.do_bench_cudagraph(
        triton_lambda, rep=1000, quantiles=quantiles
    )
    print(
        f"Cutlass fused_experts time: {cutlass_ms:.3f} ms (median) [{cutlass_min:.3f} - {cutlass_max:.3f}]"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Representative call sites include `cutlass_lambda`, `triton_lambda`, `synchronize` and `do_bench_cudagraph`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 213-237: Scenario logic / 场景逻辑
```python
    )
    print(
        f"Triton  fused_experts time: {triton_ms:.3f} ms (median) [{triton_min:.3f} - {triton_max:.3f}]"
    )

    # --- Correctness Check ---
    if check:
        print("Running correctness check...")
        with torch.no_grad():
            # Run CUTLASS version (requires transposed weights)
            y_cutlass = cutlass_fused_experts_fp8(
                x,
                w1.transpose(1, 2),  # Transposed
                w2.transpose(1, 2),  # Transposed
                w1_scale.transpose(1, 2),
                w2_scale.transpose(1, 2),
                topk_weights,
                topk_ids,
                a1_strides,
                c1_strides,
                a2_strides,
                c2_strides,
                workspace,
                a_ptrs,
                b_ptrs,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `ms`, `no_grad`, `version` and `cutlass_fused_experts_fp8`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 238-262: Scenario logic / 场景逻辑
```python
                out_ptrs,
                a_scales_ptrs,
                b_scales_ptrs,
                expert_offsets,
                problem_sizes1,
                problem_sizes2,
                enable_es=enable_es,
            )

            # Run Triton version (requires original shape weights, use inplace=False)
            y_triton = fused_experts(
                x,
                w1,  # Original shape
                w2,  # Original shape
                topk_output,
                moe_runner_config,
                use_fp8_w8a8=True,
                w1_scale=w1_scale,
                w2_scale=w2_scale,
                block_shape=block_shape,
            )

        diff = calc_diff(y_cutlass, y_triton)
        print(f"Diff: {diff:.6f}")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `version`, `fused_experts` and `calc_diff`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 263-266: Assertions and result checks / 断言与结果检查
```python
        # Tolerance might need adjustment based on FP8 specifics and kernel differences
        # FP8 comparisons often require higher tolerance than FP16/BF16
        assert diff < 1e-4, f"Diff too high! {diff}"
        print("Correctness check passed.")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。

### Lines 269-273: Helper routines around main / 辅助例程
```python
def main(tp_size=8, batch_sizes=[1, 4, 8, 16, 32, 64, 128, 256, 512], check=False):
    model_config = get_model_config(tp_size)
    print("Model Config:", model_config)
    for batch_size in batch_sizes:
        run_test(tp_size, batch_size, model_config, check)
```
**EN:** This range implements helper routine(s) `main` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `get_model_config` and `run_test`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 274-306: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--tp-size", type=int, default=8, help="Tensor Parallel size")
    parser.add_argument(
        "--batch-sizes",
        type=int,
        nargs="+",
        default=[
            1,
            4,
            8,
            16,
            32,
            64,
            128,
            256,
            512,
            1024,
            2048,
            4096,
            8192,
        ],  # Adjusted default
        help="List of batch sizes to test",
    )
    parser.add_argument("--check", action="store_true", help="Enable check mode")
    args = parser.parse_args()

    print(f"Running benchmarks with TP size: {args.tp_size}")
    print(f"Testing batch sizes: {args.batch_sizes}")

    main(tp_size=args.tp_size, batch_sizes=args.batch_sizes, check=args.check)
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. The code also ties this block to evaluation or benchmark execution. Representative call sites include `ArgumentParser`, `add_argument`, `parse_args` and `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- HTTP/API interaction / HTTP/API 交互
- Multimodal inputs / 多模态输入
- Disaggregated serving / 解耦式服务

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `argparse`
- **Third-party / 第三方库**: `torch`, `transformers`, `triton`, `triton.testing`
- **Project Modules / 项目模块**: `sglang.srt.layers.moe.cutlass_moe`, `sglang.srt.layers.moe.moe_runner.base`, `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe`, `sglang.srt.layers.moe.topk`
