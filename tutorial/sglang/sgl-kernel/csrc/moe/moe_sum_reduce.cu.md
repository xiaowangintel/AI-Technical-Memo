# moe_sum_reduce.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/moe/moe_sum_reduce.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements mixture-of-experts routing, combination, or expert computation utilities. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 实现 MoE 的路由、聚合或专家计算辅助逻辑。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Headers and compile-time setup
```cpp
#include <ATen/OpMathType.h>
#include <ATen/cuda/CUDAContext.h>
#include <c10/cuda/CUDAGuard.h>
#include <cuda.h>
#include <cudaTypedefs.h>
#include <cuda_runtime.h>
#include <torch/all.h>

#include <iostream>
#include <type_traits>

#include "cutlass/array.h"
#include "utils.h"

template <typename T>
using opmath_t = at::opmath_type<T>;

template <typename T>
__device__ __forceinline__ opmath_t<T> to_acc(T x) {
  return static_cast<opmath_t<T>>(x);
}

template <typename T>
__device__ __forceinline__ T from_acc(opmath_t<T> x) {
  return static_cast<T>(x);
}
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 27-50: Device helpers and synchronization
```cpp

template <>
__device__ __forceinline__ opmath_t<at::Half> to_acc<at::Half>(at::Half x) {
  return __half2float(__nv_half(x));
}
template <>
__device__ __forceinline__ at::Half from_acc<at::Half>(opmath_t<at::Half> x) {
  return __float2half_rn(x);
}

template <>
__device__ __forceinline__ opmath_t<at::BFloat16> to_acc<at::BFloat16>(at::BFloat16 x) {
  return __bfloat162float(__nv_bfloat16(x));
}
template <>
__device__ __forceinline__ at::BFloat16 from_acc<at::BFloat16>(opmath_t<at::BFloat16> x) {
  return __float2bfloat16_rn(x);
}

template <typename T>
__device__ __forceinline__ T ldg_cg(const T* p) {
  return __ldg(p);
}
```
**EN:** This section implements `ldg_cg`, `__half2float`, `__float2half_rn`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ldg_cg`、`__half2float`、`__float2half_rn`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 51-74: Kernel implementation
```cpp
union Pack16B {
  uint4 v;
  __nv_bfloat16 u16[8];
};

template <int WARPS_PER_BLOCK>
__global__ void moe_sum_reduce_warp_per_token_vec_kernel(
    const at::BFloat16* __restrict__ x,
    at::BFloat16* __restrict__ y,
    const int64_t token_num,
    const int64_t hidden_dim,
    const int64_t topk_num,
    const int64_t stride_token,      // in elements
    const int64_t stride_topk,       // in elements
    const int64_t out_stride_token,  // in elements
    const float scale) {
  constexpr int VEC = 16;
  constexpr int PACKS = VEC / 8;

  const int warp_id = threadIdx.x / 32;
  const int lane = threadIdx.x % 32;
  const int64_t t = (int64_t)blockIdx.y * WARPS_PER_BLOCK + warp_id;
  if (t >= token_num) return;
```
**EN:** This section implements `moe_sum_reduce_warp_per_token_vec_kernel`, `VEC`, `PACKS`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`moe_sum_reduce_warp_per_token_vec_kernel`、`VEC`、`PACKS`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 75-98: Device helpers and synchronization
```cpp
  const int64_t n_chunks = hidden_dim / VEC;

  for (int64_t chunk = (int64_t)blockIdx.x * 32 + lane; chunk < n_chunks; chunk += (int64_t)gridDim.x * 32) {
    const int64_t d = chunk * VEC;
    const int64_t base = t * stride_token + d;

    float acc[VEC];
#pragma unroll
    for (int i = 0; i < VEC; ++i)
      acc[i] = 0.f;

#pragma unroll
    for (int k = 0; k < topk_num; ++k) {
#pragma unroll
      for (int p = 0; p < PACKS; ++p) {
        const int64_t offset = base + (int64_t)k * stride_topk + p * 8;
        Pack16B pack = {ldg_cg(reinterpret_cast<const uint4*>(x + offset))};

#pragma unroll
        for (int i = 0; i < 8; ++i) {
          acc[p * 8 + i] += __bfloat162float(pack.u16[i]);
        }
      }
    }
```
**EN:** This section implements `__bfloat162float`, `n_chunks`, `d`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__bfloat162float`、`n_chunks`、`d`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 99-116: Control flow and branching
```cpp

#pragma unroll
    for (int i = 0; i < VEC; ++i)
      acc[i] *= scale;

#pragma unroll
    for (int p = 0; p < PACKS; ++p) {
      Pack16B outp;
#pragma unroll
      for (int i = 0; i < 8; ++i) {
        outp.u16[i] = __float2bfloat16_rn(acc[p * 8 + i]);
      }
      const int64_t dst = t * out_stride_token + d + p * 8;
      *reinterpret_cast<uint4*>(y + dst) = outp.v;
    }
  }
}
```
**EN:** This section drives `__float2bfloat16_rn`, `dst` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`__float2bfloat16_rn`、`dst`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 117-139: Kernel implementation
```cpp
template <typename scalar_t, int TOPK, int WARPS_PER_BLOCK>
__global__ void moe_sum_reduce_kernel_warp_token_topk(
    const scalar_t* __restrict__ x,
    scalar_t* __restrict__ y,
    const int64_t token_num,
    const int64_t hidden_dim,
    const int64_t stride_token,
    const int64_t stride_topk,
    const int64_t out_stride_token,
    const opmath_t<scalar_t> scale) {
  const int warp_id = threadIdx.x / 32;
  const int lane = threadIdx.x % 32;
  const int64_t t = (int64_t)blockIdx.y * WARPS_PER_BLOCK + warp_id;
  if (t >= token_num) return;

  for (int64_t d = (int64_t)blockIdx.x * 32 + lane; d < hidden_dim; d += (int64_t)gridDim.x * 32) {
    opmath_t<scalar_t> acc = opmath_t<scalar_t>(0);
    const int64_t base = t * stride_token + d;

#pragma unroll
    for (int k = 0; k < TOPK; ++k) {
      acc += to_acc<scalar_t>(x[base + (int64_t)k * stride_topk]);
    }
```
**EN:** This section implements `moe_sum_reduce_kernel_warp_token_topk`, `warp_id`, `lane`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`moe_sum_reduce_kernel_warp_token_topk`、`warp_id`、`lane`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 140-163: Kernel implementation
```cpp
    acc *= scale;
    y[t * out_stride_token + d] = from_acc<scalar_t>(acc);
  }
}

template <typename scalar_t, int TOPK>
__global__ void moe_sum_reduce_kernel(
    const scalar_t* __restrict__ x,
    scalar_t* __restrict__ y,
    const int64_t token_num,
    const int64_t hidden_dim,
    const int64_t stride_token,
    const int64_t stride_topk,
    const int64_t out_stride_token,
    const opmath_t<scalar_t> scale) {
  for (int t = blockIdx.y; t < token_num; t += gridDim.y) {
    for (int d = blockIdx.x * blockDim.x + threadIdx.x; d < hidden_dim; d += blockDim.x * gridDim.x) {
      const int64_t base = t * stride_token + d;
      opmath_t<scalar_t> acc = opmath_t<scalar_t>(0);

#pragma unroll
      for (int k = 0; k < TOPK; ++k) {
        acc += to_acc<scalar_t>(x[base + (int64_t)k * stride_topk]);
      }
```
**EN:** This section implements `moe_sum_reduce_kernel`, `base`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`moe_sum_reduce_kernel`、`base`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 164-191: Kernel implementation
```cpp

      acc *= scale;
      y[t * out_stride_token + d] = from_acc<scalar_t>(acc);
    }
  }
}

// -------------------- general-topk fallback kernels --------------------
// small-token
template <typename scalar_t>
__global__ void moe_sum_reduce_kernel_general(
    const scalar_t* __restrict__ x,
    scalar_t* __restrict__ y,
    const int64_t token_num,
    const int64_t hidden_dim,
    const int64_t stride_token,
    const int64_t stride_topk,
    const int64_t out_stride_token,
    const int topk_num,
    const opmath_t<scalar_t> scale) {
  for (int t = blockIdx.y; t < token_num; t += gridDim.y) {
    for (int d = blockIdx.x * blockDim.x + threadIdx.x; d < hidden_dim; d += blockDim.x * gridDim.x) {
      const int64_t base = t * stride_token + d;
      opmath_t<scalar_t> acc = opmath_t<scalar_t>(0);
#pragma unroll 1
      for (int k = 0; k < topk_num; ++k) {
        acc += to_acc<scalar_t>(x[base + (int64_t)k * stride_topk]);
      }
```
**EN:** This section implements `moe_sum_reduce_kernel_general`, `base`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`moe_sum_reduce_kernel_general`、`base`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 192-214: Kernel implementation
```cpp
      acc *= scale;
      y[t * out_stride_token + d] = from_acc<scalar_t>(acc);
    }
  }
}

// warp-per-token
template <typename scalar_t, int WARPS_PER_BLOCK>
__global__ void moe_sum_reduce_kernel_warp_token_general(
    const scalar_t* __restrict__ x,
    scalar_t* __restrict__ y,
    const int64_t token_num,
    const int64_t hidden_dim,
    const int64_t stride_token,
    const int64_t stride_topk,
    const int64_t out_stride_token,
    const int topk_num,
    const opmath_t<scalar_t> scale) {
  const int warp_id = threadIdx.x / 32;
  const int lane = threadIdx.x % 32;
  const int64_t t = (int64_t)blockIdx.y * WARPS_PER_BLOCK + warp_id;
  if (t >= token_num) return;
```
**EN:** This section implements `moe_sum_reduce_kernel_warp_token_general`, `warp_id`, `lane`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`moe_sum_reduce_kernel_warp_token_general`、`warp_id`、`lane`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 215-237: Device helpers and synchronization
```cpp
  for (int64_t d = (int64_t)blockIdx.x * 32 + lane; d < hidden_dim; d += (int64_t)gridDim.x * 32) {
    opmath_t<scalar_t> acc = opmath_t<scalar_t>(0);
    const int64_t base = t * stride_token + d;
#pragma unroll 1
    for (int k = 0; k < topk_num; ++k) {
      acc += to_acc<scalar_t>(x[base + (int64_t)k * stride_topk]);
    }
    acc *= scale;
    y[t * out_stride_token + d] = from_acc<scalar_t>(acc);
  }
}

void moe_sum_reduce(at::Tensor& input, at::Tensor& output, double routed_scaling_factor) {
  TORCH_CHECK(input.is_cuda(), "input must be CUDA tensor");
  TORCH_CHECK(output.is_cuda(), "output must be CUDA tensor");
  TORCH_CHECK(input.dim() == 3, "input must be a 3D tensor like [token_num, topk_num, hidden_dim]");
  TORCH_CHECK(output.dim() == 2, "output must be [token_num, hidden_dim]");
  TORCH_CHECK(input.size(0) == output.size(0), "token dim mismatch");
  TORCH_CHECK(input.size(2) == output.size(1), "hidden_dim mismatch");

  TORCH_CHECK(input.is_contiguous(), "expect input to be contiguous");
  TORCH_CHECK(output.is_contiguous(), "expect output to be contiguous");
```
**EN:** This section implements `moe_sum_reduce`, `TORCH_CHECK`, `base`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`moe_sum_reduce`、`TORCH_CHECK`、`base`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 238-261: Device helpers and synchronization
```cpp
  const int64_t token_num = input.size(0);
  const int64_t topk_num = input.size(1);
  const int64_t hidden_dim = input.size(2);

  const int64_t in_stride_token = input.stride(0);
  const int64_t in_stride_topk = input.stride(1);
  const int64_t out_stride_token = output.stride(0);

  auto stream = at::cuda::getCurrentCUDAStream();

  const bool fast_bf16_vec_ok = (input.scalar_type() == at::kBFloat16) && (token_num > 256) && (hidden_dim % 8 == 0);

  // Fast path for bf16 vectorize
  if (fast_bf16_vec_ok) {
    constexpr int WARPS_PER_BLOCK = 8;
    constexpr int THREADS = WARPS_PER_BLOCK * 32;

    const int64_t n_chunks = hidden_dim / 8;
    int64_t grid_x = (n_chunks + 32 - 1) / 32;
    if (grid_x > 65535) grid_x = 65535;

    int64_t grid_y = (token_num + WARPS_PER_BLOCK - 1) / WARPS_PER_BLOCK;
    if (grid_y > 65535) grid_y = 65535;
```
**EN:** This section implements `stride`, `getCurrentCUDAStream`, `scalar_type`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`stride`、`getCurrentCUDAStream`、`scalar_type`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 262-284: Kernel implementation
```cpp
    dim3 block(THREADS);
    dim3 grid(static_cast<unsigned>(grid_x), static_cast<unsigned>(grid_y));

    auto stream = at::cuda::getCurrentCUDAStream();

    const float scale = static_cast<float>(routed_scaling_factor);
    moe_sum_reduce_warp_per_token_vec_kernel<WARPS_PER_BLOCK><<<grid, block, 0, stream>>>(
        reinterpret_cast<const at::BFloat16*>(input.data_ptr<at::BFloat16>()),
        reinterpret_cast<at::BFloat16*>(output.data_ptr<at::BFloat16>()),
        token_num,
        hidden_dim,
        topk_num,
        in_stride_token,
        in_stride_topk,
        out_stride_token,
        scale);

    TORCH_CHECK(cudaGetLastError() == cudaSuccess, "moe_sum_reduce CUDA kernel (bf16 vec) launch failed");
    return;
  }

  const bool per_token_use_one_warp = (token_num > 128);
```
**EN:** This section implements `block`, `grid`, `getCurrentCUDAStream`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`block`、`grid`、`getCurrentCUDAStream`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 285-311: Kernel implementation
```cpp
  if (!per_token_use_one_warp) {
    // ---------- small-token ----------
    const int block_size = 256;
    int64_t grid_x = (hidden_dim + block_size - 1) / block_size;
    grid_x = grid_x > 65535 ? 65535 : grid_x;
    int64_t grid_y = token_num < 65535 ? token_num : 65535;

    dim3 block(block_size);
    dim3 grid(static_cast<unsigned>(grid_x), static_cast<unsigned>(grid_y));

#define LAUNCH_SMALL_TOKEN_KERNEL(TOPK)                               \
  moe_sum_reduce_kernel<scalar_t_, TOPK><<<grid, block, 0, stream>>>( \
      input.data_ptr<scalar_t_>(),                                    \
      output.data_ptr<scalar_t_>(),                                   \
      token_num,                                                      \
      hidden_dim,                                                     \
      in_stride_token,                                                \
      in_stride_topk,                                                 \
      out_stride_token,                                               \
      scale);

    AT_DISPATCH_FLOATING_TYPES_AND2(
        at::kHalf, at::kBFloat16, input.scalar_type(), "moe_sum_reduce_cuda_small_token", [&] {
          using scalar_t_ = scalar_t;
          using acc_t_ = opmath_t<scalar_t_>;
          const acc_t_ scale = static_cast<acc_t_>(routed_scaling_factor);
```
**EN:** This section implements `block`, `grid`, `LAUNCH_SMALL_TOKEN_KERNEL`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`block`、`grid`、`LAUNCH_SMALL_TOKEN_KERNEL`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 312-336: Kernel implementation
```cpp
          switch (topk_num) {
            case 2:
              LAUNCH_SMALL_TOKEN_KERNEL(2);
              break;
            case 4:
              LAUNCH_SMALL_TOKEN_KERNEL(4);
              break;
            case 8:
              LAUNCH_SMALL_TOKEN_KERNEL(8);
              break;
            case 9:
              LAUNCH_SMALL_TOKEN_KERNEL(9);
              break;
            default:  // launch general kernel
              moe_sum_reduce_kernel_general<scalar_t_><<<grid, block, 0, stream>>>(
                  input.data_ptr<scalar_t_>(),
                  output.data_ptr<scalar_t_>(),
                  token_num,
                  hidden_dim,
                  in_stride_token,
                  in_stride_topk,
                  out_stride_token,
                  static_cast<int>(topk_num),
                  scale);
          }
```
**EN:** This section implements `LAUNCH_SMALL_TOKEN_KERNEL`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`LAUNCH_SMALL_TOKEN_KERNEL`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 337-355: Device helpers and synchronization
```cpp
        });
#undef LAUNCH_SMALL_TOKEN_KERNEL

    TORCH_CHECK(cudaGetLastError() == cudaSuccess, "moe_sum_reduce CUDA kernel (small-token) launch failed");

  } else {
    // ---------- warp-per-token ----------
    constexpr int WARPS_PER_BLOCK = 4;
    constexpr int THREADS = WARPS_PER_BLOCK * 32;

    int64_t gx = (hidden_dim + 32 - 1) / 32;
    gx = gx > 65535 ? 65535 : gx;

    int64_t gy = (token_num + WARPS_PER_BLOCK - 1) / WARPS_PER_BLOCK;
    gy = gy > 65535 ? 65535 : gy;

    dim3 block(THREADS);
    dim3 grid(static_cast<unsigned>(gx), static_cast<unsigned>(gy));
```
**EN:** This section implements `TORCH_CHECK`, `block`, `grid`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`TORCH_CHECK`、`block`、`grid`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 356-379: Kernel implementation
```cpp
#define LAUNCH_WARP_PER_TOKEN_KERNEL(TOPK)                                                             \
  moe_sum_reduce_kernel_warp_token_topk<scalar_t_, TOPK, WARPS_PER_BLOCK><<<grid, block, 0, stream>>>( \
      input.data_ptr<scalar_t_>(),                                                                     \
      output.data_ptr<scalar_t_>(),                                                                    \
      token_num,                                                                                       \
      hidden_dim,                                                                                      \
      in_stride_token,                                                                                 \
      in_stride_topk,                                                                                  \
      out_stride_token,                                                                                \
      scale);

    AT_DISPATCH_FLOATING_TYPES_AND2(
        at::kHalf, at::kBFloat16, input.scalar_type(), "moe_sum_reduce_cuda_large_token", [&] {
          using scalar_t_ = scalar_t;
          using acc_t_ = opmath_t<scalar_t_>;
          const acc_t_ scale = static_cast<acc_t_>(routed_scaling_factor);

          switch (topk_num) {
            case 2:
              LAUNCH_WARP_PER_TOKEN_KERNEL(2);
              break;
            case 4:
              LAUNCH_WARP_PER_TOKEN_KERNEL(4);
              break;
```
**EN:** This section implements `LAUNCH_WARP_PER_TOKEN_KERNEL`, `scalar_t_`, `acc_t_`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`LAUNCH_WARP_PER_TOKEN_KERNEL`、`scalar_t_`、`acc_t_`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 380-403: Kernel implementation
```cpp
            case 8:
              LAUNCH_WARP_PER_TOKEN_KERNEL(8);
              break;
            case 9:
              LAUNCH_WARP_PER_TOKEN_KERNEL(9);
              break;
            default:  // launch general kernel
              moe_sum_reduce_kernel_warp_token_general<scalar_t_, WARPS_PER_BLOCK><<<grid, block, 0, stream>>>(
                  input.data_ptr<scalar_t_>(),
                  output.data_ptr<scalar_t_>(),
                  token_num,
                  hidden_dim,
                  in_stride_token,
                  in_stride_topk,
                  out_stride_token,
                  static_cast<int>(topk_num),
                  scale);
          }
        });
#undef LAUNCH_WARP_PER_TOKEN_KERNEL

    TORCH_CHECK(cudaGetLastError() == cudaSuccess, "moe_sum_reduce CUDA kernel (warp-token) launch failed");
  }
}
```
**EN:** This section implements `LAUNCH_WARP_PER_TOKEN_KERNEL`, `TORCH_CHECK`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`LAUNCH_WARP_PER_TOKEN_KERNEL`、`TORCH_CHECK`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cutlass/array.h`, `utils.h`
- **External headers / 外部头文件**: `ATen/OpMathType.h`, `ATen/cuda/CUDAContext.h`, `c10/cuda/CUDAGuard.h`, `cuda.h`, `cudaTypedefs.h`, `cuda_runtime.h`, `torch/all.h`, `iostream`, `type_traits`
- **Path context / 路径上下文**: moe / moe_sum_reduce.cu
