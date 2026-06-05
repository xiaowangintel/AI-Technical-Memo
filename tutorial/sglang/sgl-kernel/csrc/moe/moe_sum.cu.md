# moe_sum.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/moe/moe_sum.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements mixture-of-experts routing, combination, or expert computation utilities. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 实现 MoE 的路由、聚合或专家计算辅助逻辑。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Kernel implementation
```cpp
#include <ATen/ATen.h>
#include <ATen/cuda/CUDAContext.h>
#include <c10/cuda/CUDAGuard.h>
#include <torch/all.h>

#include <ATen/cuda/Atomic.cuh>
#include <cub/cub.cuh>

#include "utils.h"

template <typename scalar_t, int TOPK>
__global__ void moe_sum_kernel(
    scalar_t* __restrict__ out,          // [..., d]
    const scalar_t* __restrict__ input,  // [..., topk, d]
    const int d) {
  const int64_t token_idx = blockIdx.x;
  for (int64_t idx = threadIdx.x; idx < d; idx += blockDim.x) {
    scalar_t x = 0.0;
#pragma unroll
    for (int k = 0; k < TOPK; ++k) {
      x += SGLANG_LDG(&input[token_idx * TOPK * d + k * d + idx]);
    }
```
**EN:** This section implements `moe_sum_kernel`, `SGLANG_LDG`, `token_idx`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`moe_sum_kernel`、`SGLANG_LDG`、`token_idx`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 23-39: Runtime integration and dispatch
```cpp
    out[token_idx * d + idx] = x;
  }
}

void moe_sum(
    torch::Tensor& input,   // [num_tokens, topk, hidden_size]
    torch::Tensor& output)  // [num_tokens, hidden_size]
{
  const int hidden_size = input.size(-1);
  const auto num_tokens = output.numel() / hidden_size;
  const int topk = input.size(1);

  dim3 grid(num_tokens);
  dim3 block(std::min(hidden_size, 1024));
  const at::cuda::OptionalCUDAGuard device_guard(device_of(output));
  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();
```
**EN:** This section uses `moe_sum`, `grid`, `block` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`moe_sum`、`grid`、`block`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 40-54: Kernel implementation
```cpp
  switch (topk) {
    case 2:
      DISPATCH_FLOAT_TYPES(input.scalar_type(), "moe_sum_kernel", [&] {
        moe_sum_kernel<scalar_t, 2>
            <<<grid, block, 0, stream>>>(output.data_ptr<scalar_t>(), input.data_ptr<scalar_t>(), hidden_size);
      });
      break;

    case 3:
      DISPATCH_FLOAT_TYPES(input.scalar_type(), "moe_sum_kernel", [&] {
        moe_sum_kernel<scalar_t, 3>
            <<<grid, block, 0, stream>>>(output.data_ptr<scalar_t>(), input.data_ptr<scalar_t>(), hidden_size);
      });
      break;
```
**EN:** This section implements the surrounding logic, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了相关逻辑等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 55-66: Kernel implementation
```cpp
    case 4:
      DISPATCH_FLOAT_TYPES(input.scalar_type(), "moe_sum_kernel", [&] {
        moe_sum_kernel<scalar_t, 4>
            <<<grid, block, 0, stream>>>(output.data_ptr<scalar_t>(), input.data_ptr<scalar_t>(), hidden_size);
      });
      break;

    default:
      at::sum_out(output, input, 1);
      break;
  }
}
```
**EN:** This section implements `sum_out`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`sum_out`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **Synchronization / atomics / 同步与原子操作**: Uses atomics or explicit synchronization to coordinate parallel work. / 使用原子操作或显式同步协调并行工作。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `utils.h`
- **External headers / 外部头文件**: `ATen/ATen.h`, `ATen/cuda/CUDAContext.h`, `c10/cuda/CUDAGuard.h`, `torch/all.h`, `ATen/cuda/Atomic.cuh`, `cub/cub.cuh`
- **Path context / 路径上下文**: moe / moe_sum.cu
