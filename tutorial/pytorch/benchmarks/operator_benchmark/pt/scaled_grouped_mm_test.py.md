# scaled_grouped_mm_test.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `benchmarks/operator_benchmark/pt/scaled_grouped_mm_test.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator benchmark cases, parameter grids, and registration calls used to measure PyTorch operator performance. Benchmark registration and workload-shape coverage are important to understanding the file. Backend or dtype coverage is a visible axis of variation in the implementation.
- **Purpose (CN)**: 定义用于衡量 PyTorch 算子性能的 benchmark 用例、参数网格以及注册调用。 理解该文件时，benchmark 注册与工作负载形状覆盖是关键。 后端或 dtype 覆盖是该实现中明显的变化维度。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8 / 第 1-8 行

```python
from pt import configs  # noqa: F401
from pt.scaled_mm_common import (
    build_equal_k_group_offs,
    get_float8_dtype,
    get_test_scaled_matmul_cuda,
    SCALED_MM_BASE_SHAPES,
)

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 9-20 / 第 9-20 行

```python
import operator_benchmark as op_bench

import torch
from torch.nn.functional import ScalingType
from torch.testing._internal.common_cuda import (
    IS_SM100,
    IS_SM90,
    PLATFORM_SUPPORTS_FP8_GROUPED_GEMM,
    PLATFORM_SUPPORTS_MXFP8_GROUPED_GEMM,
)
from torch.torch_version import TorchVersion

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 21-32 / 第 21-32 行

```python

"""
Operator microbenchmarks for `scaled_grouped_mm`.

This benchmark supports:
- FP8 (e4m3/e5m2) with TensorWise and RowWise scaling:
  * CUDA SM90 (H100) only - not supported on SM100 (B200)
  * ROCm MI300+ (gfx94x) with grouped GEMM support
- MXFP8/MXFP4/NVFP4 grouped-K path with blocked scaling:
  * CUDA-only (non-HIP), SM90+ and SM100+
  * Requires swizzled scales

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 33-42 / 第 33-42 行

```python
All modes reuse the same conversion helpers as `test/test_scaled_matmul_cuda.py`.
"""


def _should_generate_scaled_grouped_mm_configs() -> bool:
    # Minimum requirements:
    # - PyTorch 2.9+ (scaled_grouped_mm introduced)
    # - CUDA: compute capability exactly 9.0 (SM90) or 10.0 (SM100) and CUDA 12.8+
    # - ROCm: MI300+ (gfx94x) grouped GEMM support
    if TorchVersion(torch.__version__) < "2.9" or not hasattr(
```

- **EN:** Important local symbols in this block include _should_generate_scaled_grouped_mm_configs.
- **CN:** 该代码块中的重要局部符号包括 _should_generate_scaled_grouped_mm_configs。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 43-50 / 第 43-50 行

```python
        torch.nn.functional, "scaled_grouped_mm"
    ):
        return False
    if not torch.cuda.is_available():
        return False

    if torch.version.hip is not None:
        return bool(PLATFORM_SUPPORTS_FP8_GROUPED_GEMM)
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 51-58 / 第 51-58 行

```python

    # CUDA build: some scale modes require CUDA 12.8+ (see `aten/src/ATen/cuda/CUDABlas.cpp:get_scale_mode`).
    if TorchVersion(torch.version.cuda or "0.0") < "12.8":
        return False

    return bool(IS_SM90) or bool(IS_SM100)


```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 59-71 / 第 59-71 行

```python
class ScaledGroupedMMBenchmark(op_bench.TorchBenchmarkBase):
    def init(
        self,
        M,
        N,
        K,
        G,
        device,
        scaling="mxfp8",
        output_dtype="bfloat16",
        float8_dtype="e4m3fn",
    ):
        if output_dtype != "bfloat16":
```

- **EN:** Important local symbols in this block include ScaledGroupedMMBenchmark, init.
- **CN:** 该代码块中的重要局部符号包括 ScaledGroupedMMBenchmark、init。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 72-79 / 第 72-79 行

```python
            raise ValueError(
                "scaled_grouped_mm benchmark currently supports bfloat16 output only"
            )
        self.output_dtype = torch.bfloat16

        if device != "cuda":
            raise ValueError("scaled_grouped_mm benchmark is CUDA-only")
        if torch.version.hip is not None and not PLATFORM_SUPPORTS_FP8_GROUPED_GEMM:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 80-87 / 第 80-87 行

```python
            raise ValueError(
                "scaled_grouped_mm benchmark requires ROCm MI300+ (gfx94x) grouped GEMM support"
            )

        self.scaling = scaling
        self.base_dtype = torch.bfloat16

        if scaling in ("fp8_tensorwise", "fp8_rowwise"):
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 88-95 / 第 88-95 行

```python
            self._init_fp8(M, N, K, G, device, float8_dtype, scaling)
        elif scaling in ("mxfp8", "mxfp4", "nvfp4"):
            self._init_mx_nvfp4(M, N, K, G, device, scaling)
        else:
            raise ValueError(f"Unsupported scaling format: {scaling}")

        self.set_module_name("scaled_grouped_mm")

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 96-103 / 第 96-103 行

```python
    def _init_fp8(self, M, N, K, G, device, float8_dtype, scaling):
        """Initialize FP8 tensorwise or rowwise scaling."""
        self.float8_dtype = get_float8_dtype(float8_dtype)

        # We interpret offs as group end offsets along K (grouped-K).
        # Use deterministic equal-sized groups.
        offs = build_equal_k_group_offs(K, G, device)

```

- **EN:** Important local symbols in this block include _init_fp8.
- **CN:** 该代码块中的重要局部符号包括 _init_fp8。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 104-112 / 第 104-112 行

```python
        # Create FP8 inputs directly (similar to test_scaled_grouped_gemm_2d_2d)
        # Input shapes: (M, K) and (N, K) where K is the total K dimension
        x_lp = torch.randn(M, K, device=device, dtype=self.base_dtype).to(
            self.float8_dtype
        )
        w_lp = torch.randn(N, K, device=device, dtype=self.base_dtype).to(
            self.float8_dtype
        )

```

- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 113-124 / 第 113-124 行

```python
        if scaling == "fp8_tensorwise":
            # Tensorwise scaling: one scale per group
            # For tensorwise, we still use RowWise recipe but with repeated scales
            # scale_a: (M*G,), scale_b: (N*G,)
            # Each group gets the same scale value repeated M or N times
            scale_a = torch.rand(
                G, device=device, dtype=torch.float32
            ).repeat_interleave(M)
            scale_b = torch.rand(
                G, device=device, dtype=torch.float32
            ).repeat_interleave(N)

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 125-134 / 第 125-134 行

```python
            self._scale_recipe_a = ScalingType.RowWise
            self._scale_recipe_b = ScalingType.RowWise

        elif scaling == "fp8_rowwise":
            # Rowwise scaling: M scales per group, N scales per group
            # scale_a: (M*G,), scale_b: (N*G,)
            # Organized as [group0_M_scales, group1_M_scales, ..., group_{G-1}_M_scales]
            scale_a = torch.rand(M * G, device=device, dtype=torch.float32)
            scale_b = torch.rand(N * G, device=device, dtype=torch.float32)

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 135-149 / 第 135-149 行

```python
            self._scale_recipe_a = ScalingType.RowWise
            self._scale_recipe_b = ScalingType.RowWise

        self._swizzle_a = None
        self._swizzle_b = None

        # For grouped-K, mat_b is expected as (K, N).
        self.inputs = {
            "x": x_lp,
            "w_t": w_lp.t(),
            "offs": offs,
            "scale_a": scale_a,
            "scale_b": scale_b,
        }

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 150-157 / 第 150-157 行

```python
    def _init_mx_nvfp4(self, M, N, K, G, device, scaling):
        """Initialize MX or NVFP4 blocked scaling."""
        helpers = get_test_scaled_matmul_cuda()

        # NVFP4 is not supported on HIP
        if scaling == "nvfp4" and torch.version.hip is not None:
            raise ValueError("nvfp4 not supported on HIP/ROCm")

```

- **EN:** Important local symbols in this block include _init_mx_nvfp4.
- **CN:** 该代码块中的重要局部符号包括 _init_mx_nvfp4。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 158-166 / 第 158-166 行

```python
        # We interpret offs as group end offsets along K (grouped-K).
        # Use deterministic equal-sized groups.
        offs = build_equal_k_group_offs(K, G, device)

        # Create high-precision inputs and quantize per-group along K into the requested format.
        # Use modest magnitudes to avoid degenerate saturation.
        x_hp = torch.randn((M, K), device=device, dtype=torch.bfloat16) * 0.1
        w_hp = torch.randn((N, K), device=device, dtype=torch.bfloat16) * 0.1

```

- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 167-174 / 第 167-174 行

```python
        _, xq, x_scales, x_global = helpers._2d_grouped_tensor_to_blocked_scaled(
            x_hp, M, G, offs, format=scaling
        )
        _, wq, w_scales, w_global = helpers._2d_grouped_tensor_to_blocked_scaled(
            w_hp, N, G, offs, format=scaling
        )

        if scaling == "nvfp4":
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 175-185 / 第 175-185 行

```python
            kwargs = helpers._build_scaled_grouped_mm_kwargs(
                [x_scales, x_global],
                [w_scales, w_global],
                offs,
                format=scaling,
            )
        else:
            kwargs = helpers._build_scaled_grouped_mm_kwargs(
                x_scales, w_scales, offs, format=scaling
            )

```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 186-200 / 第 186-200 行

```python
        self._scale_recipe_a = kwargs["scale_recipe_a"]
        self._scale_recipe_b = kwargs["scale_recipe_b"]
        # _build_scaled_grouped_mm_kwargs already handles HIP vs CUDA swizzle selection
        self._swizzle_a = kwargs.get("swizzle_a", None)
        self._swizzle_b = kwargs.get("swizzle_b", None)

        # For grouped-K, mat_b is expected as (K, N).
        self.inputs = {
            "x": xq,
            "w_t": wq.t(),
            "offs": offs,
            "scale_a": kwargs["scale_a"],
            "scale_b": kwargs["scale_b"],
        }

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 201-210 / 第 201-210 行

```python
    def forward(self, x, w_t, offs, scale_a, scale_b):
        call_kwargs = {
            "scale_a": scale_a,
            "scale_recipe_a": self._scale_recipe_a,
            "scale_b": scale_b,
            "scale_recipe_b": self._scale_recipe_b,
            "offs": offs,
            "output_dtype": self.output_dtype,
        }
        if self._swizzle_a is not None:
```

- **EN:** Important local symbols in this block include forward.
- **CN:** 该代码块中的重要局部符号包括 forward。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 211-224 / 第 211-224 行

```python
            call_kwargs["swizzle_a"] = self._swizzle_a
        if self._swizzle_b is not None:
            call_kwargs["swizzle_b"] = self._swizzle_b

        return torch.nn.functional.scaled_grouped_mm(x, w_t, **call_kwargs)


# Generate MNKG shapes from shared base shapes
# First 2 shapes use all group counts [1, 2, 4, 8]
# Remaining shapes use only [1, 8] for faster benchmarking
MNKG_list = [
    (m, n, k, g) for (m, n, k) in SCALED_MM_BASE_SHAPES[:2] for g in [1, 2, 4, 8]
] + [(m, n, k, g) for (m, n, k) in SCALED_MM_BASE_SHAPES[2:] for g in [1, 8]]

```

- **EN:** Looping logic iterates over benchmark dimensions, shape metadata, or container elements.
- **CN:** 循环逻辑会遍历 benchmark 维度、形状元数据或容器元素。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 225-240 / 第 225-240 行

```python
scaled_grouped_mm_configs_long = []

if _should_generate_scaled_grouped_mm_configs():
    # FP8 tensorwise and rowwise: works on both CUDA and ROCm
    # Requires PLATFORM_SUPPORTS_FP8_GROUPED_GEMM (SM90/H100, not SM100/B200)
    if PLATFORM_SUPPORTS_FP8_GROUPED_GEMM:
        scaled_grouped_mm_configs_long += op_bench.config_list(
            attr_names=["M", "N", "K", "G"],
            attrs=[[m, n, k, g] for (m, n, k, g) in MNKG_list],
            cross_product_configs={
                "device": ["cuda"],
                "float8_dtype": ["e4m3fn"],
                "output_dtype": ["bfloat16"],
                "scaling": ["fp8_tensorwise", "fp8_rowwise"],
            },
            tags=["long"],
```

- **EN:** The code is defining benchmark parameter spaces so one benchmark class can be exercised across many shapes, devices, or dtypes.
- **CN:** 这里在定义 benchmark 参数空间，使同一个 benchmark 类可以覆盖多种形状、设备或 dtype。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Looping logic iterates over benchmark dimensions, shape metadata, or container elements.
- **CN:** 循环逻辑会遍历 benchmark 维度、形状元数据或容器元素。

### Lines 241-255 / 第 241-255 行

```python
        )

    # MX supports both CUDA (with swizzle) and HIP (with NO_SWIZZLE).
    if PLATFORM_SUPPORTS_MXFP8_GROUPED_GEMM:
        scaled_grouped_mm_configs_long += op_bench.config_list(
            attr_names=["M", "N", "K", "G"],
            attrs=[[m, n, k, g] for (m, n, k, g) in MNKG_list],
            cross_product_configs={
                "device": ["cuda"],
                "scaling": ["mxfp4", "mxfp8"],
                "output_dtype": ["bfloat16"],
            },
            tags=["long"],
        )

```

- **EN:** The code is defining benchmark parameter spaces so one benchmark class can be exercised across many shapes, devices, or dtypes.
- **CN:** 这里在定义 benchmark 参数空间，使同一个 benchmark 类可以覆盖多种形状、设备或 dtype。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Looping logic iterates over benchmark dimensions, shape metadata, or container elements.
- **CN:** 循环逻辑会遍历 benchmark 维度、形状元数据或容器元素。

### Lines 256-268 / 第 256-268 行

```python
    # NVFP4 is CUDA-only (non-HIP) due to swizzled scale requirements.
    if torch.version.hip is None and PLATFORM_SUPPORTS_MXFP8_GROUPED_GEMM:
        scaled_grouped_mm_configs_long += op_bench.config_list(
            attr_names=["M", "N", "K", "G"],
            attrs=[[m, n, k, g] for (m, n, k, g) in MNKG_list],
            cross_product_configs={
                "device": ["cuda"],
                "scaling": ["nvfp4"],
                "output_dtype": ["bfloat16"],
            },
            tags=["long"],
        )

```

- **EN:** The code is defining benchmark parameter spaces so one benchmark class can be exercised across many shapes, devices, or dtypes.
- **CN:** 这里在定义 benchmark 参数空间，使同一个 benchmark 类可以覆盖多种形状、设备或 dtype。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Looping logic iterates over benchmark dimensions, shape metadata, or container elements.
- **CN:** 循环逻辑会遍历 benchmark 维度、形状元数据或容器元素。

### Lines 269-276 / 第 269-276 行

```python
op_bench.generate_pt_test(
    scaled_grouped_mm_configs_long,
    ScaledGroupedMMBenchmark,
)


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
- **Benchmark workflow** — 记录或实现性能测量场景。
- **Representative symbols: ScaledGroupedMMBenchmark, _should_generate_scaled_grouped_mm_configs, init, _init_fp8, _init_mx_nvfp4, forward** — 代表性符号：ScaledGroupedMMBenchmark、_should_generate_scaled_grouped_mm_configs、init、_init_fp8、_init_mx_nvfp4、forward

## Dependencies / 依赖关系

- `pt`
- `pt.scaled_mm_common`
- `operator_benchmark`
- `torch`
- `torch.nn.functional`
- `torch.testing._internal.common_cuda`
- `torch.torch_version`
