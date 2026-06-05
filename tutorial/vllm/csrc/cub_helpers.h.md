# cub_helpers.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cub_helpers.h`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Normalizes CUB/HIPCUB includes and reduction operator names across CUDA versions and ROCm builds. / [CN] 在不同 CUDA 版本与 ROCm 构建之间统一 CUB/HIPCUB 的头文件与归约算子命名。

## Line-by-Line Analysis / 逐行分析
### Backend and version compatibility aliases / 后端与版本兼容别名
```cpp
#ifndef USE_ROCM
  #include <cub/cub.cuh>
  #if CUB_VERSION >= 200800
    #include <cuda/std/functional>
using CubAddOp = cuda::std::plus<>;
using CubMaxOp = cuda::maximum<>;
  #else   // if CUB_VERSION < 200800
using CubAddOp = cub::Sum;
using CubMaxOp = cub::Max;
  #endif  // CUB_VERSION
#else
  #include <hipcub/hipcub.hpp>
namespace cub = hipcub;
using CubAddOp = hipcub::Sum;
using CubMaxOp = hipcub::Max;
#endif  // USE_ROCM
```
**EN:** On CUDA builds, the header chooses between modern `cuda::std` functors and legacy `cub::Sum`/`cub::Max` depending on `CUB_VERSION`. On ROCm, it aliases `hipcub` into the `cub` namespace and exports matching operator names.
**CN:** 在 CUDA 构建中，这个头文件会根据 `CUB_VERSION` 在较新的 `cuda::std` 函子与传统的 `cub::Sum`/`cub::Max` 之间切换。在 ROCm 上，它把 `hipcub` 映射到 `cub` 命名空间，并导出对应的算子别名。

## Key Concepts / 关键概念
- **EN:** The file is tiny but strategically important because it hides API drift in the CUB ecosystem.
  **CN:** 这个文件虽小，但非常关键，因为它屏蔽了 CUB 生态中的 API 漂移问题。
- **EN:** Higher-level kernels can refer only to `CubAddOp` and `CubMaxOp` without caring about backend/version details.
  **CN:** 上层内核只需使用 `CubAddOp` 与 `CubMaxOp`，无需关心后端和版本差异。

## Dependencies / 依赖关系
- **EN:** Depends on either `<cub/cub.cuh>` or `<hipcub/hipcub.hpp>`.
  **CN:** 依赖 `<cub/cub.cuh>` 或 `<hipcub/hipcub.hpp>` 之一。
- **EN:** The selected aliases are typically consumed by reduction and scan kernels elsewhere in the CUDA extension.
  **CN:** 这些别名通常会被 CUDA 扩展中其他归约与扫描内核使用。
