# skinny_gemms.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/rocm/skinny_gemms.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements ROCm-optimized skinny GEMM kernels. / 实现面向 ROCm 优化的 skinny GEMM 内核。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-24)
```cpp
#include <torch/all.h>
#include <ATen/cuda/CUDAContext.h>
#include <c10/cuda/CUDAGuard.h>

#include <cuda_runtime.h>
#include <cuda_fp16.h>
#include <cuda_bf16.h>

#include <stdexcept>
#include <algorithm>

#include "../cuda_compat.h"
#include "dispatch_utils.h"
#include "quantization/w8a8/fp8/common.cuh"
// ...
// However, it may be possible to fix these kernels to handle both issues.

#if defined(__HIPCC__) && \
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: LLGemm1_kernel (lines 165-254)
```cpp
template <typename scalar_t, int NUM_A_ROWS_PER_BLOCK>
__global__ void LLGemm1_kernel(const scalar_t* in_a, const scalar_t* in_b,
                               scalar_t* out_c, const int K) {
  using scalar2_t = typename scalar2<scalar_t>::type;
  auto af4 = reinterpret_cast<const float4*>(in_a);
  auto bf4 = reinterpret_cast<const scalar2_t*>(in_b);
  auto c = reinterpret_cast<scalar2_t*>(out_c);
  __shared__ float red_smem[NUM_A_ROWS_PER_BLOCK][WARP_SIZE];
  const int row_addr = blockIdx.x * NUM_A_ROWS_PER_BLOCK * K / 8;
  const int threadid = threadIdx.x;
  const int warp = threadIdx.x / WARP_SIZE;
  const int lane = threadIdx.x % WARP_SIZE;
  const int num_warps = blockDim.x / WARP_SIZE;
  const int qwarpid = threadid / 16;
// ...
    }
  }
}
```
**EN:** This kernel drives a tiled matrix multiplication path, organizing expert or attention work into GPU-friendly fragments.
**CN:** 该内核驱动分块矩阵乘法路径，把专家计算或注意力计算组织成适合 GPU 执行的片段。

### Function / Kernel: LLMM1 (lines 256-308)
```cpp
torch::Tensor LLMM1(at::Tensor& in_a, at::Tensor& in_b,
                    const int64_t rows_per_block) {
  auto M = in_a.size(0);
  auto K = in_a.size(1);
  auto N = in_b.size(0);

  TORCH_CHECK(N == 1, "Row number of activation tensor must be 1.");
  TORCH_CHECK(in_a.dtype() == in_b.dtype());
  TORCH_CHECK(in_b.dtype() == torch::kFloat16 ||
              in_b.dtype() == torch::kBFloat16);

  auto out_c = torch::empty(
      {N, M}, torch::TensorOptions().dtype(in_b.dtype()).device(in_b.device()));

// ...

  return out_c;
}
```
**EN:** This block computes a fused activation path so activation and gating can be applied with minimal extra memory traffic.
**CN:** 该代码块实现融合激活路径，使激活与门控能够在尽量少的额外内存访问下完成。

### Function / Kernel: wvSplitK_hf_sml_ (lines 340-560)
```cpp
template <typename scalar_t, int THRDS, int YTILE, int WvPrGrp, int A_CHUNK,
          int UNRL, int N>
__global__ void __launch_bounds__(WvPrGrp* THRDS)
    wvSplitK_hf_sml_(const int K, const int Kbp, const int Kap, const int M,
                     const int Bx, const int By, const scalar_t* B,
                     const scalar_t* __restrict__ A,
                     const scalar_t* __restrict__ BIAS, scalar_t* C,
                     const int _WvPrGrp, const int CuCount) {
  constexpr int max_lds_len = LDS_SIZE / 2;
  #if defined(__HIP__MI3XX__)
  constexpr bool use_mfma = (std::is_same_v<scalar_t, __hip_bfloat16>);
  #else
  constexpr bool use_mfma = false;
  #endif
// ...
    m += CuCount * _WvPrGrp * YTILE;
  }
}
```
**EN:** This kernel drives a tiled matrix multiplication path, organizing expert or attention work into GPU-friendly fragments.
**CN:** 该内核驱动分块矩阵乘法路径，把专家计算或注意力计算组织成适合 GPU 执行的片段。

### Function / Kernel: wvSplitK_hf_ (lines 576-794)
```cpp
template <typename scalar_t, int THRDS, int YTILE, int WvPrGrp, int A_CHUNK,
          int UNRL, int N>
__global__ void __launch_bounds__(WvPrGrp* THRDS)
    wvSplitK_hf_(const int K, const int Kbp, const int Kap, const int M,
                 const int Bx, const int By, const scalar_t* B,
                 const scalar_t* __restrict__ A,
                 const scalar_t* __restrict__ BIAS, scalar_t* C,
                 const int _WvPrGrp, const int CuCount) {
  constexpr int max_lds_len = LDS_SIZE / 2;
  #if defined(__HIP__MI3XX__)
  constexpr bool use_mfma = (std::is_same_v<scalar_t, __hip_bfloat16>);
  #else
  constexpr bool use_mfma = false;
  #endif
// ...
    }
  }
}
```
**EN:** This kernel drives a tiled matrix multiplication path, organizing expert or attention work into GPU-friendly fragments.
**CN:** 该内核驱动分块矩阵乘法路径，把专家计算或注意力计算组织成适合 GPU 执行的片段。

### Function / Kernel: wvSplitK (lines 1172-1287)
```cpp
torch::Tensor wvSplitK(const at::Tensor& in_a, const at::Tensor& in_b,
                       const std::optional<at::Tensor>& in_bias,
                       const int64_t CuCount) {
  auto M_in = in_a.size(0);
  auto K_in = in_a.size(1);
  auto N_in = in_b.size(0);
  auto Kap_in = in_a.stride(0);
  auto Kbp_in = in_b.stride(0);
  auto Bx_in =
      (in_bias.has_value() && in_bias->numel() > 0)
          ? (in_bias->sizes().size() == 2) ? in_bias->size(1) : in_bias->size(0)
          : 1;
  auto By_in = (in_bias.has_value() && in_bias->numel() > 0 &&
                in_bias->sizes().size() == 2)
// ...
  });
  return out_c;
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

## Key Concepts / 关键概念
- Tiled matrix multiplication / 分块矩阵乘法
- FP8 quantization / compute / FP8 量化与计算
- Fused activation functions / 融合激活函数
- SIMD vectorization / SIMD 向量化
- CUDA programming model / CUDA 编程模型
- ROCm/HIP backend / ROCm/HIP 后端
- CUDA kernel launch and thread mapping / CUDA 内核启动与线程映射
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `../cuda_compat.h`, `quantization/w8a8/fp8/common.cuh`, `core/batch_invariant.hpp`
- **External libraries / 外部库**: PyTorch / ATen, PyTorch / c10, CUDA runtime / CUDA headers, C++ standard library or platform support, C++ standard library
- **Runtime coupling / 运行时耦合**: Launches GPU kernels and relies on CUDA/HIP execution semantics / 启动 GPU 内核并依赖 CUDA/HIP 执行语义; Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
