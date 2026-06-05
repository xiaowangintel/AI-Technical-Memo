# scaled_mm_test.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `benchmarks/operator_benchmark/pt/scaled_mm_test.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator benchmark cases, parameter grids, and registration calls used to measure PyTorch operator performance. Benchmark registration and workload-shape coverage are important to understanding the file. Backend or dtype coverage is a visible axis of variation in the implementation.
- **Purpose (CN)**: 定义用于衡量 PyTorch 算子性能的 benchmark 用例、参数网格以及注册调用。 理解该文件时，benchmark 注册与工作负载形状覆盖是关键。 后端或 dtype 覆盖是该实现中明显的变化维度。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11 / 第 1-11 行

```python
from types import ModuleType

from pt.scaled_mm_common import (
    get_float8_dtype,
    get_test_scaled_matmul_cuda,
    SCALED_MM_BASE_SHAPES,
    supports_fp8_deepseek_blockwise_scaling,
)

import operator_benchmark as op_bench

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 12-27 / 第 12-27 行

```python
import torch
from torch.nn.functional import ScalingType, SwizzleType
from torch.testing._internal.common_cuda import (
    PLATFORM_SUPPORTS_FP8,
    PLATFORM_SUPPORTS_MX_GEMM,
    SM90OrLater,
)
from torch.torch_version import TorchVersion


"""
Operator microbenchmarks for `scaled_mm`.
Uses the same dtype + scale/quantize helpers as `test/test_scaled_matmul_cuda.py`
(bf16/fp16/fp32, fp8 e4m3/e5m2, MX e8m0 scales, NVFP4 packed fp4).
"""

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 28-42 / 第 28-42 行

```python

def _should_generate_scaled_mm_configs() -> bool:
    # Benchmarks are CUDA-centric; register nothing when the op can't run.
    # Minimum requirements:
    # - PyTorch 2.9+ (scaled_mm introduced)
    # - CUDA SM90+ (compute capability >= 9.0) OR ROCm MI300+ (gfx94x)
    return (
        TorchVersion(torch.__version__) >= "2.9"
        and hasattr(torch.nn.functional, "scaled_mm")
        and (
            (torch.version.hip is None and bool(SM90OrLater))
            or (torch.version.hip is not None and bool(PLATFORM_SUPPORTS_FP8))
        )
    )

```

- **EN:** Important local symbols in this block include _should_generate_scaled_mm_configs.
- **CN:** 该代码块中的重要局部符号包括 _should_generate_scaled_mm_configs。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 43-53 / 第 43-53 行

```python

def _supports_fp8_rowwise_fp32_output() -> bool:
    # Mirrors test_scaled_mm_vs_emulated_row_wise gating:
    # fp32 rowwise kernels are cuBLAS-only, CUDA 12.9+, and SM90-only.
    if torch.version.hip is not None:
        return False
    if not torch.cuda.is_available() or torch.version.cuda is None:
        return False
    if TorchVersion(torch.version.cuda) < "12.9":
        return False
    return torch.cuda.get_device_capability(0) >= (9, 0)
```

- **EN:** Important local symbols in this block include _supports_fp8_rowwise_fp32_output.
- **CN:** 该代码块中的重要局部符号包括 _supports_fp8_rowwise_fp32_output。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 54-66 / 第 54-66 行

```python


def _supports_scaled_mm_benchmark() -> tuple[bool, str]:
    # `scaled_mm` was introduced in PyTorch 2.9.
    if not hasattr(torch.nn.functional, "scaled_mm"):
        return False, "torch.nn.functional.scaled_mm requires PyTorch 2.9+"

    if not torch.cuda.is_available():
        return False, "CUDA not available"

    # Mirror torch._scaled_mm support message:
    # "torch._scaled_mm is only supported on CUDA devices with compute capability >= 9.0 or 8.9, or ROCm MI300+"
    if torch.version.hip is not None:
```

- **EN:** Important local symbols in this block include _supports_scaled_mm_benchmark.
- **CN:** 该代码块中的重要局部符号包括 _supports_scaled_mm_benchmark。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 67-79 / 第 67-79 行

```python
        arch = torch.cuda.get_device_properties(0).gcnArchName
        if "gfx94" in arch:
            return True, ""
        return False, f"unsupported ROCm arch {arch} (requires MI300+ / gfx94x)"

    cap = torch.cuda.get_device_capability(0)
    if cap >= (9, 0) or cap == (8, 9):
        return True, ""
    return (
        False,
        f"unsupported CUDA compute capability {cap[0]}.{cap[1]} (requires >= 9.0 or 8.9)",
    )

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 80-93 / 第 80-93 行

```python

class ScaledMMBenchmark(op_bench.TorchBenchmarkBase):
    _MX_BLOCK_SIZE: int = 32
    _NVFP4_BLOCK_SIZE: int = 16
    _FP8_BLOCK_K: int = 128

    def _set_output_dtype(self, output_dtype: str) -> None:
        if output_dtype == "bfloat16":
            self.output_dtype = torch.bfloat16
        elif output_dtype == "float32":
            self.output_dtype = torch.float32
        else:
            self.output_dtype = torch.bfloat16  # default

```

- **EN:** Important local symbols in this block include ScaledMMBenchmark, _set_output_dtype.
- **CN:** 该代码块中的重要局部符号包括 ScaledMMBenchmark、_set_output_dtype。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 94-107 / 第 94-107 行

```python
    def _set_scaled_mm_call_config(
        self,
        *,
        scale_recipe_a: ScalingType | list[ScalingType],
        scale_recipe_b: ScalingType | list[ScalingType],
        swizzle_a: SwizzleType | list[SwizzleType] | None = None,
        swizzle_b: SwizzleType | list[SwizzleType] | None = None,
    ) -> None:
        # Store call-time config so forward() is a single straight-line call.
        self._scale_recipe_a = scale_recipe_a
        self._scale_recipe_b = scale_recipe_b
        self._swizzle_a = swizzle_a
        self._swizzle_b = swizzle_b

```

- **EN:** Important local symbols in this block include _set_scaled_mm_call_config.
- **CN:** 该代码块中的重要局部符号包括 _set_scaled_mm_call_config。

### Lines 108-128 / 第 108-128 行

```python
    def _init_fp8_tensorwise(
        self, M: int, N: int, K: int, device: str, helpers: ModuleType
    ) -> None:
        self.float8_dtype = get_float8_dtype(self._float8_dtype_arg)

        # Base tensors carry grad in backward benches; fp8 tensors are created as leaves.
        x_base = torch.randn(
            M,
            K,
            device=device,
            dtype=self.base_dtype,
            requires_grad=self.auto_set(),
        )
        y_base = torch.randn(
            N,
            K,
            device=device,
            dtype=self.base_dtype,
            requires_grad=self.auto_set(),
        ).t()

```

- **EN:** Important local symbols in this block include _init_fp8_tensorwise.
- **CN:** 该代码块中的重要局部符号包括 _init_fp8_tensorwise。
- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 129-145 / 第 129-145 行

```python
        # Tensorwise scales; detach so the backward bench doesn't include scale computation.
        x_scale = helpers.tensor_to_scale(x_base, self.float8_dtype).float().detach()
        y_scale = helpers.tensor_to_scale(y_base, self.float8_dtype).float().detach()

        # Quantize with the same saturation logic as the reference tests.
        with torch.no_grad():
            x_lp = (
                helpers.to_fp8_saturated(x_base * x_scale, self.float8_dtype)
                .detach()
                .requires_grad_(self.auto_set())
            )
            y_lp = (
                helpers.to_fp8_saturated(y_base * y_scale, self.float8_dtype)
                .detach()
                .requires_grad_(self.auto_set())
            )

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 146-156 / 第 146-156 行

```python
        self.inputs = {
            "x": x_lp,
            "y": y_lp,
            "scale_a": x_scale.reciprocal(),
            "scale_b": y_scale.reciprocal(),
        }
        self._set_scaled_mm_call_config(
            scale_recipe_a=ScalingType.TensorWise,
            scale_recipe_b=ScalingType.TensorWise,
        )

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 157-178 / 第 157-178 行

```python
    def _init_fp8_rowwise(
        self, M: int, N: int, K: int, device: str, helpers: ModuleType
    ) -> None:
        # Row-wise scaling (per-row A scales and per-column B scales).
        # Mirrors `test_scaled_mm_vs_emulated_row_wise` in test_scaled_matmul_cuda.py.
        self.float8_dtype = get_float8_dtype(self._float8_dtype_arg)

        x_base = torch.randn(
            M,
            K,
            device=device,
            dtype=self.base_dtype,
            requires_grad=self.auto_set(),
        )
        # Start from (N, K) and transpose so mat_b is (K, N), matching scaled_mm signature.
        y_base = torch.randn(
            N,
            K,
            device=device,
            dtype=self.base_dtype,
            requires_grad=self.auto_set(),
        ).t()
```

- **EN:** Important local symbols in this block include _init_fp8_rowwise.
- **CN:** 该代码块中的重要局部符号包括 _init_fp8_rowwise。
- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 179-198 / 第 179-198 行

```python

        x_scales = (
            helpers.tensor_to_scale(x_base, self.float8_dtype, dim=1).float().detach()
        )  # (M, 1)
        y_scales = (
            helpers.tensor_to_scale(y_base, self.float8_dtype, dim=0).float().detach()
        )  # (1, N)

        with torch.no_grad():
            x_lp = (
                helpers.to_fp8_saturated(x_base * x_scales, self.float8_dtype)
                .detach()
                .requires_grad_(self.auto_set())
            )
            y_lp = (
                helpers.to_fp8_saturated(y_base * y_scales, self.float8_dtype)
                .detach()
                .requires_grad_(self.auto_set())
            )

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 199-209 / 第 199-209 行

```python
        self.inputs = {
            "x": x_lp,
            "y": y_lp,
            "scale_a": x_scales.reciprocal(),
            "scale_b": y_scales.reciprocal(),
        }
        self._set_scaled_mm_call_config(
            scale_recipe_a=ScalingType.RowWise,
            scale_recipe_b=ScalingType.RowWise,
        )

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 210-224 / 第 210-224 行

```python
    def _init_fp8_blockwise_common(
        self,
        M: int,
        N: int,
        K: int,
        device: str,
        helpers: ModuleType,
        block_m: int,
        block_k: int,
        scaling_type: ScalingType,
        use_padding: bool,
    ) -> None:
        """
        Common initialization for FP8 blockwise scaling.

```

- **EN:** Important local symbols in this block include _init_fp8_blockwise_common.
- **CN:** 该代码块中的重要局部符号包括 _init_fp8_blockwise_common。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 225-239 / 第 225-239 行

```python
        Args:
            block_m: Block size for M dimension (1 for 1x128, 128 for 128x128)
            block_k: Block size for K dimension (always 128)
            scaling_type: ScalingType enum value
            use_padding: If True, pad scales for 128x128; if False, use simple transpose for 1x128
        """
        self.float8_dtype = get_float8_dtype(self._float8_dtype_arg)

        # Validate SM90 support
        if device == "cuda" and torch.cuda.get_device_capability(0) != (9, 0):
            mode_name = "1x128" if block_m == 1 else "128x128"
            raise RuntimeError(
                f"FP8 BlockWise{mode_name} (DeepSeek style) scaling is only supported on CUDA SM90 (H100)."
            )

```

- **EN:** Important local symbols in this block include value.
- **CN:** 该代码块中的重要局部符号包括 value。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 240-253 / 第 240-253 行

```python
        # Validate dimension divisibility
        if block_m == 1:
            # 1x128 only requires K divisible by block_k
            if K % block_k != 0:
                raise RuntimeError(
                    f"FP8 BlockWise1x128 requires K divisible by {block_k}, got K={K}"
                )
        else:
            # 128x128 requires M, N, K all divisible by block size
            if (M % block_k) != 0 or (N % block_k) != 0 or (K % block_k) != 0:
                raise RuntimeError(
                    f"FP8 BlockWise128x128 requires M,N,K divisible by {block_k}, got M={M}, N={N}, K={K}"
                )

```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 254-272 / 第 254-272 行

```python
        # Create high-precision input tensors
        x_hp = torch.randn(
            M, K, device=device, dtype=self.base_dtype, requires_grad=self.auto_set()
        )
        y_hp = torch.randn(
            N, K, device=device, dtype=self.base_dtype, requires_grad=self.auto_set()
        )

        # Quantize to FP8 with block-wise scaling
        with torch.no_grad():
            x_lp, x_scales = helpers.tensor_to_scale_block(
                x_hp, self.float8_dtype, block_m, block_k
            )
            y_lp, y_scales = helpers.tensor_to_scale_block(
                y_hp, self.float8_dtype, block_m, block_k
            )
            x_lp = x_lp.detach().requires_grad_(self.auto_set())
            y_lp = y_lp.detach().requires_grad_(self.auto_set())

```

- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 273-284 / 第 273-284 行

```python
        # Process scales based on block configuration
        if use_padding:
            # 128x128: pad scales to multiple of 4, then transpose
            x_scales, _ = helpers._pad_128x128_scales(x_scales.detach())
            y_scales, _ = helpers._pad_128x128_scales(y_scales.detach())
            x_scales = x_scales.t()
            y_scales = y_scales.t()
        else:
            # 1x128: simple transpose to get "outer-dim-major" layout
            x_scales = x_scales.t().contiguous().t().detach()
            y_scales = y_scales.t().contiguous().t().detach()

```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 285-295 / 第 285-295 行

```python
        self.inputs = {
            "x": x_lp,
            "y": y_lp.t(),  # mat_b is (K, N)
            "scale_a": x_scales.reciprocal(),
            "scale_b": y_scales.reciprocal(),
        }
        self._set_scaled_mm_call_config(
            scale_recipe_a=scaling_type,
            scale_recipe_b=scaling_type,
        )

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 296-311 / 第 296-311 行

```python
    def _init_fp8_blockwise_1x128(
        self, M: int, N: int, K: int, device: str, helpers: ModuleType
    ) -> None:
        # FP8 blockwise scaling with 1x128 blocks.
        self._init_fp8_blockwise_common(
            M,
            N,
            K,
            device,
            helpers,
            block_m=1,
            block_k=self._FP8_BLOCK_K,
            scaling_type=ScalingType.BlockWise1x128,
            use_padding=False,
        )

```

- **EN:** Important local symbols in this block include _init_fp8_blockwise_1x128.
- **CN:** 该代码块中的重要局部符号包括 _init_fp8_blockwise_1x128。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 312-327 / 第 312-327 行

```python
    def _init_fp8_blockwise_128x128(
        self, M: int, N: int, K: int, device: str, helpers: ModuleType
    ) -> None:
        # FP8 blockwise scaling with 128x128 blocks.
        self._init_fp8_blockwise_common(
            M,
            N,
            K,
            device,
            helpers,
            block_m=self._FP8_BLOCK_K,
            block_k=self._FP8_BLOCK_K,
            scaling_type=ScalingType.BlockWise128x128,
            use_padding=True,
        )

```

- **EN:** Important local symbols in this block include _init_fp8_blockwise_128x128.
- **CN:** 该代码块中的重要局部符号包括 _init_fp8_blockwise_128x128。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 328-341 / 第 328-341 行

```python
    def _init_mx_blockwise(
        self, M: int, N: int, K: int, device: str, *, mx_format: str
    ) -> None:
        # MX uses BlockWise1x32 with swizzled scales on CUDA, NO_SWIZZLE on HIP.
        if device != "cuda":
            raise RuntimeError(f"MX scaling requires CUDA device, got: {device}")

        # Important cuBLASLt requirement: mat_b must be column-major.
        # We satisfy this by passing a transpose view (non-contiguous) for `mat_b`.
        #
        # NOTE: we intentionally import from torch.testing._internal to reuse the exact
        # reference implementation used by test/test_scaled_matmul_cuda.py.
        from torch.testing._internal.common_quantized import to_blocked, to_mxfp

```

- **EN:** Important local symbols in this block include _init_mx_blockwise.
- **CN:** 该代码块中的重要局部符号包括 _init_mx_blockwise。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 342-355 / 第 342-355 行

```python
        x_hp = torch.randn(
            M, K, device=device, dtype=self.base_dtype, requires_grad=self.auto_set()
        )
        y_hp = torch.randn(
            N, K, device=device, dtype=self.base_dtype, requires_grad=self.auto_set()
        )

        scale_a, x_lp = to_mxfp(
            x_hp.contiguous(), block_size=self._MX_BLOCK_SIZE, format=mx_format
        )
        scale_b, y_lp = to_mxfp(
            y_hp.contiguous(), block_size=self._MX_BLOCK_SIZE, format=mx_format
        )

```

- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 356-377 / 第 356-377 行

```python
        scale_a = to_blocked(scale_a)
        scale_b = to_blocked(scale_b)

        # HIP requires NO_SWIZZLE, CUDA uses SWIZZLE_32_4_4
        swizzle_type = (
            SwizzleType.NO_SWIZZLE
            if torch.version.hip is not None
            else SwizzleType.SWIZZLE_32_4_4
        )

        self.inputs = {
            "x": x_lp,
            "y": y_lp.t(),  # column-major mat_b
            "scale_a": scale_a,
            "scale_b": scale_b,
        }
        self._set_scaled_mm_call_config(
            scale_recipe_a=ScalingType.BlockWise1x32,
            scale_recipe_b=ScalingType.BlockWise1x32,
            swizzle_a=swizzle_type,
            swizzle_b=swizzle_type,
        )
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 378-389 / 第 378-389 行

```python

    def _init_nvfp4_blockwise_and_tensorwise(
        self, M: int, N: int, K: int, device: str, helpers: ModuleType
    ) -> None:
        # NVFP4 uses packed fp4 inputs and two-level scaling:
        # - blockwise (1x16) decode scales (swizzled for CUDA)
        # - tensorwise (global) decode scale (NO_SWIZZLE)
        #
        # scaled_mm expects these as LISTS for both `scale_*` and `scale_recipe_*`.
        if device != "cuda":
            raise RuntimeError(f"NVFP4 scaling requires CUDA device, got: {device}")
        if torch.version.hip is not None:
```

- **EN:** Important local symbols in this block include _init_nvfp4_blockwise_and_tensorwise.
- **CN:** 该代码块中的重要局部符号包括 _init_nvfp4_blockwise_and_tensorwise。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 390-400 / 第 390-400 行

```python
            raise RuntimeError("NVFP4 benchmarks are only wired for CUDA (non-HIP).")
        if K % 32 != 0:
            raise RuntimeError(f"NVFP4 requires K divisible by 32, got K={K}")

        # NOTE: We reuse the same reference implementation as test/test_scaled_matmul_cuda.py.
        from torch.testing._internal.common_quantized import to_blocked

        # Use nontrivial distribution so scaling isn't degenerate.
        a_ref = torch.randn((M, K), device=device, dtype=self.base_dtype) * 1000
        b_ref = torch.randn((N, K), device=device, dtype=self.base_dtype) * 1000

```

- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 401-422 / 第 401-422 行

```python
        a_lp, a_scale, a_global_scale = helpers.data_to_nvfp4_with_global_scale(
            a_ref, self._NVFP4_BLOCK_SIZE
        )
        b_lp, b_scale, b_global_scale = helpers.data_to_nvfp4_with_global_scale(
            b_ref, self._NVFP4_BLOCK_SIZE
        )

        a_scale = to_blocked(a_scale)
        b_scale = to_blocked(b_scale)

        self.inputs = {
            "x": a_lp,
            "y": b_lp.t(),
            "scale_a": [a_scale, a_global_scale],
            "scale_b": [b_scale, b_global_scale],
        }
        self._set_scaled_mm_call_config(
            scale_recipe_a=[ScalingType.BlockWise1x16, ScalingType.TensorWise],
            scale_recipe_b=[ScalingType.BlockWise1x16, ScalingType.TensorWise],
            swizzle_a=[SwizzleType.SWIZZLE_32_4_4, SwizzleType.NO_SWIZZLE],
            swizzle_b=[SwizzleType.SWIZZLE_32_4_4, SwizzleType.NO_SWIZZLE],
        )
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 423-439 / 第 423-439 行

```python

    def init(
        self,
        M,
        N,
        K,
        device,
        float8_dtype="e4m3fn",
        output_dtype="bfloat16",
        scaling="fp8_tensorwise",
    ):
        helpers = get_test_scaled_matmul_cuda()
        self._float8_dtype_arg = float8_dtype
        self.base_dtype = torch.bfloat16
        self.scaling = scaling
        self.float8_dtype = None

```

- **EN:** Important local symbols in this block include init.
- **CN:** 该代码块中的重要局部符号包括 init。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 440-458 / 第 440-458 行

```python
        self._set_output_dtype(output_dtype)

        if scaling == "fp8_tensorwise":
            self._init_fp8_tensorwise(M, N, K, device, helpers)
        elif scaling == "fp8_rowwise":
            self._init_fp8_rowwise(M, N, K, device, helpers)
        elif scaling == "fp8_blockwise_1x128":
            self._init_fp8_blockwise_1x128(M, N, K, device, helpers)
        elif scaling == "fp8_blockwise_128x128":
            self._init_fp8_blockwise_128x128(M, N, K, device, helpers)
        elif scaling == "mxfp8":
            self._init_mx_blockwise(M, N, K, device, mx_format="mxfp8")
        elif scaling == "mxfp4":
            self._init_mx_blockwise(M, N, K, device, mx_format="mxfp4")
        elif scaling == "nvfp4":
            self._init_nvfp4_blockwise_and_tensorwise(M, N, K, device, helpers)
        else:
            raise ValueError(f"Unsupported scaling mode: {scaling}")

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 459-469 / 第 459-469 行

```python
        self.set_module_name("scaled_mm")

    def forward(self, x, y, scale_a, scale_b):
        kwargs = {
            "scale_a": scale_a,
            "scale_recipe_a": self._scale_recipe_a,
            "scale_b": scale_b,
            "scale_recipe_b": self._scale_recipe_b,
            "output_dtype": self.output_dtype,
        }
        if self._swizzle_a is not None:
```

- **EN:** Important local symbols in this block include forward.
- **CN:** 该代码块中的重要局部符号包括 forward。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 470-480 / 第 470-480 行

```python
            kwargs["swizzle_a"] = self._swizzle_a
        if self._swizzle_b is not None:
            kwargs["swizzle_b"] = self._swizzle_b

        return torch.nn.functional.scaled_mm(x, y, **kwargs)


# Use shared base shapes from scaled_mm_common
_scaled_mm_long_shapes = []
_seen = set()
for m, n, k in SCALED_MM_BASE_SHAPES:
```

- **EN:** Looping logic iterates over benchmark dimensions, shape metadata, or container elements.
- **CN:** 循环逻辑会遍历 benchmark 维度、形状元数据或容器元素。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 481-491 / 第 481-491 行

```python
    shape = (m, n, k)
    if shape in _seen:
        continue
    _seen.add(shape)
    _scaled_mm_long_shapes.append([m, n, k])

# Build long configs in groups so we can gate unsupported (scaling, output_dtype)
# combinations based on the running platform.
scaled_mm_configs_long = []

if _should_generate_scaled_mm_configs():
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 492-504 / 第 492-504 行

```python
    # FP8 tensorwise supports bf16 and fp32 output.
    scaled_mm_configs_long += op_bench.config_list(
        attr_names=["M", "N", "K"],
        attrs=_scaled_mm_long_shapes,
        cross_product_configs={
            "device": ["cuda"],
            "float8_dtype": ["e4m3fn"],
            "output_dtype": ["bfloat16"],
            "scaling": ["fp8_tensorwise"],
        },
        tags=["long"],
    )

```

- **EN:** The code is defining benchmark parameter spaces so one benchmark class can be exercised across many shapes, devices, or dtypes.
- **CN:** 这里在定义 benchmark 参数空间，使同一个 benchmark 类可以覆盖多种形状、设备或 dtype。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 505-522 / 第 505-522 行

```python
    # FP8 rowwise requires CUDA 12.9+ on CUDA builds (see `aten/src/ATen/cuda/CUDABlas.cpp:get_scale_mode`).
    if torch.version.hip is None and TorchVersion(torch.version.cuda) < "12.9":
        pass
    else:
        # Keep bf16-only for now.
        rowwise_output_dtypes = ["bfloat16"]
        scaled_mm_configs_long += op_bench.config_list(
            attr_names=["M", "N", "K"],
            attrs=_scaled_mm_long_shapes,
            cross_product_configs={
                "device": ["cuda"],
                "float8_dtype": ["e4m3fn"],
                "output_dtype": rowwise_output_dtypes,
                "scaling": ["fp8_rowwise"],
            },
            tags=["long"],
        )

```

- **EN:** The code is defining benchmark parameter spaces so one benchmark class can be exercised across many shapes, devices, or dtypes.
- **CN:** 这里在定义 benchmark 参数空间，使同一个 benchmark 类可以覆盖多种形状、设备或 dtype。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 523-537 / 第 523-537 行

```python
    # MX supports both CUDA (with swizzle) and HIP (with NO_SWIZZLE).
    # NVFP4 is CUDA-only (non-HIP) due to swizzled scale requirements.
    if PLATFORM_SUPPORTS_MX_GEMM:
        scaled_mm_configs_long += op_bench.config_list(
            attr_names=["M", "N", "K"],
            attrs=_scaled_mm_long_shapes,
            cross_product_configs={
                "device": ["cuda"],
                "float8_dtype": ["e4m3fn"],
                "output_dtype": ["bfloat16"],
                "scaling": ["mxfp8", "mxfp4"],
            },
            tags=["long"],
        )

```

- **EN:** The code is defining benchmark parameter spaces so one benchmark class can be exercised across many shapes, devices, or dtypes.
- **CN:** 这里在定义 benchmark 参数空间，使同一个 benchmark 类可以覆盖多种形状、设备或 dtype。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 538-551 / 第 538-551 行

```python
    # NVFP4 is CUDA-only (non-HIP)
    if torch.version.hip is None and PLATFORM_SUPPORTS_MX_GEMM:
        scaled_mm_configs_long += op_bench.config_list(
            attr_names=["M", "N", "K"],
            attrs=_scaled_mm_long_shapes,
            cross_product_configs={
                "device": ["cuda"],
                "float8_dtype": ["e4m3fn"],
                "output_dtype": ["bfloat16"],
                "scaling": ["nvfp4"],
            },
            tags=["long"],
        )

```

- **EN:** The code is defining benchmark parameter spaces so one benchmark class can be exercised across many shapes, devices, or dtypes.
- **CN:** 这里在定义 benchmark 参数空间，使同一个 benchmark 类可以覆盖多种形状、设备或 dtype。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 552-565 / 第 552-565 行

```python
    # DeepSeek FP8 blockwise (1x128 / 128x128) is SM90-only.
    if supports_fp8_deepseek_blockwise_scaling():
        scaled_mm_configs_long += op_bench.config_list(
            attr_names=["M", "N", "K"],
            attrs=_scaled_mm_long_shapes,
            cross_product_configs={
                "device": ["cuda"],
                "float8_dtype": ["e4m3fn"],
                "output_dtype": ["bfloat16"],
                "scaling": ["fp8_blockwise_1x128", "fp8_blockwise_128x128"],
            },
            tags=["long"],
        )

```

- **EN:** The code is defining benchmark parameter spaces so one benchmark class can be exercised across many shapes, devices, or dtypes.
- **CN:** 这里在定义 benchmark 参数空间，使同一个 benchmark 类可以覆盖多种形状、设备或 dtype。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 566-576 / 第 566-576 行

```python
# Generate tests for scaled_mm (register nothing on unsupported platforms).
if _should_generate_scaled_mm_configs():
    _scaled_mm_configs = scaled_mm_configs_long
else:
    _scaled_mm_configs = []

op_bench.generate_pt_test(_scaled_mm_configs, ScaledMMBenchmark)


if __name__ == "__main__":
    op_bench.benchmark_runner.main()
```

- **EN:** This portion registers the prepared benchmark definitions with the operator_benchmark runner.
- **CN:** 这一部分把准备好的 benchmark 定义注册到 operator_benchmark 运行器中。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

## Key Concepts / 关键概念

- **PyTorch benchmark suite** — PyTorch 基准测试套件
- **Operator benchmark harness** — 使用 operator_benchmark 框架注册 PyTorch 性能测试用例。
- **Parameterized benchmark matrix** — 构建形状、设备、dtype 等参数组合形成的基准矩阵。
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **Data type coverage** — 体现文件如何在不同张量 dtype 间覆盖行为差异。
- **Device coverage** — 突出 CPU/CUDA/移动端等后端相关执行路径。
- **Representative symbols: ScaledMMBenchmark, _should_generate_scaled_mm_configs, _supports_fp8_rowwise_fp32_output, _supports_scaled_mm_benchmark, _set_output_dtype, _set_scaled_mm_call_config, _init_fp8_tensorwise, _init_fp8_rowwise** — 代表性符号：ScaledMMBenchmark、_should_generate_scaled_mm_configs、_supports_fp8_rowwise_fp32_output、_supports_scaled_mm_benchmark、_set_output_dtype、_set_scaled_mm_call_config、_init_fp8_tensorwise、_init_fp8_rowwise

## Dependencies / 依赖关系

- `types`
- `pt.scaled_mm_common`
- `operator_benchmark`
- `torch`
- `torch.nn.functional`
- `torch.testing._internal.common_cuda`
- `torch.torch_version`
- `torch.testing._internal.common_quantized`
