# scaled_mm_common.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `benchmarks/operator_benchmark/pt/scaled_mm_common.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator benchmark cases, parameter grids, and registration calls used to measure PyTorch operator performance. Backend or dtype coverage is a visible axis of variation in the implementation.
- **Purpose (CN)**: 定义用于衡量 PyTorch 算子性能的 benchmark 用例、参数网格以及注册调用。 后端或 dtype 覆盖是该实现中明显的变化维度。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7 / 第 1-7 行

```python
import importlib.util
import os
from types import ModuleType

import torch
from torch.torch_version import TorchVersion

```

- **EN:** This range mainly pulls in the imports/includes that the later benchmark or integration logic depends on.
- **CN:** 这一段主要引入后续基准或集成逻辑依赖的 import/include。

### Lines 8-16 / 第 8-16 行

```python

"""
Shared utilities for scaled_mm and scaled_grouped_mm benchmarks.

Both benchmarks use FP8, MX, and NVFP4 quantization with helpers from
test/test_scaled_matmul_cuda.py. This module provides common imports
and utilities to reduce code duplication.
"""

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 17-28 / 第 17-28 行

```python
_TEST_SCALED_MATMUL_CUDA_MOD: ModuleType | None = None


# Shared benchmark shapes for scaled matmul operations
# These shapes are used by both scaled_mm and scaled_grouped_mm benchmarks
SCALED_MM_BASE_SHAPES = [
    # Small shapes for faster benchmarking
    (1024, 1024, 1024),
    (2048, 4096, 2048),
    (4096, 2048, 4096),
    # Original larger shapes
    (16384, 8192, 5120),
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 29-35 / 第 29-35 行

```python
    (128000, 8192, 5120),
    (16384, 1536, 5120),
    (128000, 1536, 5120),
    (16384, 2048, 7168),
    (128000, 2048, 7168),
]

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 36-44 / 第 36-44 行

```python

def get_test_scaled_matmul_cuda() -> ModuleType:
    """
    Reuse scale/quantization helpers from `test/test_scaled_matmul_cuda.py`.

    `test/` isn't a package, so we import by path and cache the module.
    """
    global _TEST_SCALED_MATMUL_CUDA_MOD
    if _TEST_SCALED_MATMUL_CUDA_MOD is not None:
```

- **EN:** Important local symbols in this block include get_test_scaled_matmul_cuda.
- **CN:** 该代码块中的重要局部符号包括 get_test_scaled_matmul_cuda。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 45-51 / 第 45-51 行

```python
        return _TEST_SCALED_MATMUL_CUDA_MOD

    pytorch_root = os.path.abspath(
        os.path.join(os.path.dirname(__file__), "..", "..", "..")
    )
    test_file = os.path.join(pytorch_root, "test", "test_scaled_matmul_cuda.py")
    if not os.path.exists(test_file):
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 52-59 / 第 52-59 行

```python
        raise RuntimeError(
            f"Expected to find {test_file} to reuse scaled matmul test helpers, but it does not exist."
        )

    spec = importlib.util.spec_from_file_location(
        "_test_scaled_matmul_cuda_bench_import", test_file
    )
    if spec is None or spec.loader is None:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 60-65 / 第 60-65 行

```python
        raise RuntimeError(f"Failed to create import spec for {test_file}")

    mod = importlib.util.module_from_spec(spec)
    spec.loader.exec_module(mod)
    _TEST_SCALED_MATMUL_CUDA_MOD = mod
    return mod
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 66-71 / 第 66-71 行

```python


def get_float8_dtype(float8_dtype):
    """Normalize the FP8 dtype arg (handles ROCm fnuz variants via test aliases)."""
    from torch.testing._internal.common_device_type import e4m3_type, e5m2_type

```

- **EN:** Important local symbols in this block include get_float8_dtype.
- **CN:** 该代码块中的重要局部符号包括 get_float8_dtype。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 72-77 / 第 72-77 行

```python
    if float8_dtype in ("e4m3fn", e4m3_type, torch.float8_e4m3fn):
        return e4m3_type
    if float8_dtype in ("e5m2", e5m2_type, torch.float8_e5m2):
        return e5m2_type
    return e4m3_type  # default

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 78-85 / 第 78-85 行

```python

def build_equal_k_group_offs(total_k: int, groups: int, device: str) -> torch.Tensor:
    """
    Build equal-sized group offsets for grouped-K operations.

    Used by scaled_grouped_mm to partition the K dimension into equal groups.
    Returns a tensor of group end offsets.

```

- **EN:** Important local symbols in this block include build_equal_k_group_offs.
- **CN:** 该代码块中的重要局部符号包括 build_equal_k_group_offs。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 86-94 / 第 86-94 行

```python
    Args:
        total_k: Total K dimension to partition
        groups: Number of groups
        device: Device to create tensor on

    Returns:
        Tensor of shape (groups,) with group end offsets
    """
    if groups <= 0:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 95-103 / 第 95-103 行

```python
        raise ValueError(f"groups must be > 0, got {groups}")
    if total_k % groups != 0:
        raise ValueError(f"total_k ({total_k}) must be divisible by groups ({groups})")
    k_per_group = total_k // groups
    if k_per_group % 32 != 0:
        raise ValueError(
            f"K per group must be divisible by 32 for these kernels, got {k_per_group}"
        )
    return torch.arange(
```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 104-111 / 第 104-111 行

```python
        k_per_group, total_k + 1, k_per_group, device=device, dtype=torch.int32
    )


def supports_fp8_deepseek_blockwise_scaling() -> bool:
    """
    Check if the platform supports DeepSeek-style FP8 blockwise scaling.

```

- **EN:** Important local symbols in this block include supports_fp8_deepseek_blockwise_scaling.
- **CN:** 该代码块中的重要局部符号包括 supports_fp8_deepseek_blockwise_scaling。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 112-118 / 第 112-118 行

```python
    PyTorch currently gates "DeepSeek style" FP8 blockwise scaling (1x128/128x128)
    to SM90 (H100) only. On SM100 (B200) this errors with NotImplementedError.
    """
    if not torch.cuda.is_available() or torch.version.cuda is None:
        return False
    # These scaling modes require CUDA 12.9+ (see `aten/src/ATen/cuda/CUDABlas.cpp:get_scale_mode`).
    if torch.version.hip is None and TorchVersion(torch.version.cuda) < "12.9":
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 119-124 / 第 119-124 行

```python
        return False
    if (
        torch.version.hip is not None
        and "gfx950" not in torch.cuda.get_device_properties(0).gcnArchName
    ):  # Blockwise scaling is not supported on ROCm
        return False
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 125-125 / 第 125-125 行

```python
    return torch.cuda.get_device_capability(0) == (9, 0)
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

## Key Concepts / 关键概念

- **PyTorch benchmark suite** — PyTorch 基准测试套件
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **Data type coverage** — 体现文件如何在不同张量 dtype 间覆盖行为差异。
- **Device coverage** — 突出 CPU/CUDA/移动端等后端相关执行路径。
- **Benchmark workflow** — 记录或实现性能测量场景。
- **Representative symbols: get_test_scaled_matmul_cuda, get_float8_dtype, build_equal_k_group_offs, supports_fp8_deepseek_blockwise_scaling** — 代表性符号：get_test_scaled_matmul_cuda、get_float8_dtype、build_equal_k_group_offs、supports_fp8_deepseek_blockwise_scaling

## Dependencies / 依赖关系

- `importlib.util`
- `os`
- `types`
- `torch`
- `torch.torch_version`
- `torch.testing._internal.common_device_type`
