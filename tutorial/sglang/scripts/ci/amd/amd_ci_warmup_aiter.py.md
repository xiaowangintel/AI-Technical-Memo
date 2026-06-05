# amd_ci_warmup_aiter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/amd/amd_ci_warmup_aiter.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `amd_ci_warmup_aiter` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `amd_ci_warmup_aiter` 流程，主要负责CI 编排、测试执行。它属于 `amd` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-10: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Warmup script to pre-build AITER JIT kernels.

This script triggers compilation of commonly used AITER kernels by importing
the relevant modules and calling functions with sample data. This avoids
timeouts during actual tests when kernels need to be compiled on first use.

Run this after clearing pre-built AITER kernels from the Docker image.
"""
```
**EN:** Warmup script to pre-build AITER JIT kernels.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 12-14: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import os
import sys
import time
```
**EN:** This block loads os, sys, time. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 os, sys, time。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 17-17: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
os.environ["SGLANG_USE_AITER"] = "1"
```
**EN:** This section defines shared constants, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 共享常量，用于把可复用的默认值集中在模块顶部。

### Lines 20-147: Defines the `warmup_aiter_kernels` routine / 定义 `warmup_aiter_kernels` 例程
```python
def warmup_aiter_kernels():
    """Trigger AITER JIT kernel compilation."""
    import torch

    if not torch.cuda.is_available():
        print("CUDA/ROCm not available, skipping AITER warmup")
        return

    print("=" * 60)
    print("AITER JIT Kernel Warmup")
    print("=" * 60)

    device = torch.device("cuda:0")
    start_time = time.time()

    # Warmup module_rmsnorm_quant (small module, ~2MB)
    # Triggered by rmsnorm2d_fwd when hidden_size <= 8192
    try:
        print(
            "\n[1/5] Warming up module_rmsnorm_quant (rmsnorm2d_fwd, hidden<=8192)..."
        )
        from aiter import rmsnorm2d_fwd

        hidden_size = 4096
        batch_size = 512  # Use larger batch to match CUDA graph capture
        x = torch.randn(batch_size, hidden_size, dtype=torch.bfloat16, device=device)
        weight = torch.ones(hidden_size, dtype=torch.bfloat16, device=device)
        eps = 1e-6

        # hidden_size=4096 <= 8192 -> takes rmsnorm() path -> compiles module_rmsnorm_quant
        _ = rmsnorm2d_fwd(x, weight, eps)
        torch.cuda.synchronize()
        print("   module_rmsnorm_quant compiled successfully")
    except Exception as e:
        print(f"   module_rmsnorm_quant warmup failed: {e}")

    # Warmup module_rmsnorm (large CK module, ~159MB)
    # Triggered by rmsnorm2d_fwd_with_add (always uses CK path)
    # NOTE: rmsnorm2d_fwd_with_add signature is:
    #   rmsnorm2d_fwd_with_add(out, input, residual_in, residual_out, weight, epsilon)
    try:
        print("\n[2/5] Warming up module_rmsnorm (rmsnorm2d_fwd_with_add, CK path)...")
        from aiter import rmsnorm2d_fwd_with_add

        hidden_size = 4096
        batch_size = 512
        x = torch.randn(batch_size, hidden_size, dtype=torch.bfloat16, device=device)
        residual_in = torch.randn(
            batch_size, hidden_size, dtype=torch.bfloat16, device=device
        )
        output = torch.empty_like(x)
        residual_out = torch.empty_like(x)
        weight = torch.ones(hidden_size, dtype=torch.bfloat16, device=device)
        eps = 1e-6

        # This triggers JIT compilation of module_rmsnorm (CK kernels)
        rmsnorm2d_fwd_with_add(output, x, residual_in, residual_out, weight, eps)
        torch.cuda.synchronize()
        print("   module_rmsnorm compiled successfully")
    except Exception as e:
        print(f"   module_rmsnorm warmup failed: {e}")

    # Warmup module_rmsnorm via rmsnorm2d_fwd with large hidden_size (CK path)
    # When hidden_size > 8192, rmsnorm2d_fwd takes the rmsnorm2d_fwd_ck path
    # which also uses module_rmsnorm (already compiled in step 2, but this
    # ensures the CK rmsnorm2d_fwd path is exercised as well)
    try:
        print("\n[3/5] Warming up rmsnorm2d_fwd CK path (hidden>8192)...")
        from aiter import rmsnorm2d_fwd

        hidden_size = 16384  # > 8192 to trigger rmsnorm2d_fwd_ck (module_rmsnorm)
        batch_size = 32
        x = torch.randn(batch_size, hidden_size, dtype=torch.bfloat16, device=device)
        weight = torch.ones(hidden_size, dtype=torch.bfloat16, device=device)
        eps = 1e-6

        _ = rmsnorm2d_fwd(x, weight, eps)
        torch.cuda.synchronize()
        print("   rmsnorm2d_fwd CK path compiled successfully")
    except Exception as e:
        print(f"   rmsnorm2d_fwd CK path warmup skipped: {e}")

    # Warmup rotary embedding kernel if available
    try:
        print("\n[4/5] Warming up rotary embedding kernel...")
        from aiter import rotary_embedding

        head_size = 128
        seq_len = 32
        num_heads = 32
        positions = torch.arange(seq_len, device=device)
        query = torch.randn(
            seq_len, num_heads, head_size, dtype=torch.bfloat16, device=device
        )
        key = torch.randn(
            seq_len, num_heads, head_size, dtype=torch.bfloat16, device=device
        )
        cos = torch.ones(seq_len, head_size // 2, dtype=torch.bfloat16, device=device)
        sin = torch.zeros(seq_len, head_size // 2, dtype=torch.bfloat16, device=device)

        _ = rotary_embedding(positions, query, key, head_size, cos, sin, True)
        torch.cuda.synchronize()
        print("   Rotary embedding kernel compiled successfully")
    except Exception as e:
        print(f"   Rotary embedding warmup skipped (may not be available): {e}")

    # Warmup activation kernels if available
    try:
        print("\n[5/5] Warming up activation kernels...")
        from aiter import silu_and_mul

        hidden_size = 4096
        batch_size = 512
        x = torch.randn(
            batch_size, hidden_size * 2, dtype=torch.bfloat16, device=device
        )
        out = torch.empty(batch_size, hidden_size, dtype=torch.bfloat16, device=device)

        silu_and_mul(out, x)
        torch.cuda.synchronize()
        print("   Activation kernel compiled successfully")
    except Exception as e:
        print(f"   Activation warmup skipped (may not be available): {e}")

    elapsed = time.time() - start_time
    print("\n" + "=" * 60)
    print(f"AITER warmup completed in {elapsed:.1f}s")
    print("=" * 60 + "\n")
```
**EN:** This block defines `warmup_aiter_kernels`. Trigger AITER JIT kernel compilation. It takes no explicit parameters at the top level. Internally it emits status messages.
**CN:** 该代码块定义了 `warmup_aiter_kernels`。其文档字符串说明了该逻辑的预期职责。它在顶层定义中不接收显式参数。内部会输出状态信息。

### Lines 150-151: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    warmup_aiter_kernels()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Container execution** / 容器执行

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `sys`, `time`
- **Third-party modules / 第三方模块**: `aiter`, `torch`
