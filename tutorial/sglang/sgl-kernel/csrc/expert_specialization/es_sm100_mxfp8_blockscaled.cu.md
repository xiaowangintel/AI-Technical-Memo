# es_sm100_mxfp8_blockscaled.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/expert_specialization/es_sm100_mxfp8_blockscaled.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Runtime integration and dispatch
```cpp
#include <torch/all.h>

#include "es_sm100_mxfp8_blockscaled_launcher.cuh"

void es_sm100_mxfp8_blockscaled_grouped_mm(
    const torch::Tensor& a,
    const torch::Tensor& b,
    const torch::Tensor& sfa,
    const torch::Tensor& sfb,
    torch::Tensor& d,
    const torch::Tensor& problem_sizes,
    const torch::Tensor& expert_offsets,
    const torch::Tensor& blockscale_offsets) {
#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
  TORCH_CHECK(problem_sizes.dim() == 2, "problem_sizes must be 2D tensor");
  TORCH_CHECK(problem_sizes.size(1) == 3, "problem_sizes must have shape (num_experts, 3)");
  TORCH_CHECK(
      problem_sizes.size(0) == expert_offsets.size(0), "Number of experts in problem_sizes must match expert_offsets");
```
**EN:** This section uses `es_sm100_mxfp8_blockscaled_grouped_mm`, `defined`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`es_sm100_mxfp8_blockscaled_grouped_mm`、`defined`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 19-36: Runtime integration and dispatch
```cpp
  TORCH_CHECK(problem_sizes.dtype() == torch::kInt32, "problem_sizes must be int32");
  TORCH_CHECK(a.dim() == 2, "a must be a 2D tensor of shape (num_tokens, k)");
  TORCH_CHECK(b.dim() == 3, "b must be a 3D tensor of shape (num_experts, k, n)");
  TORCH_CHECK(a.size(1) == b.size(1) && a.size(1) % 128 == 0, "k should align 128");
  TORCH_CHECK(b.size(2) % 128 == 0, "n should align 128");
  TORCH_CHECK(a.strides()[1] == 1, "a must be row major");
  TORCH_CHECK(b.strides()[1] == 1, "a must be column major");

  auto stream = at::cuda::getCurrentCUDAStream();
  if (d.dtype() == torch::kBFloat16) {
    expert_specialization::es_sm100_mxfp8_blockscaled_group_mm_dispatch_out_dtype<cutlass::bfloat16_t>(
        a, b, sfa, sfb, d, problem_sizes, expert_offsets, blockscale_offsets, stream);
  } else if (d.dtype() == torch::kFloat16) {
    expert_specialization::es_sm100_mxfp8_blockscaled_group_mm_dispatch_out_dtype<cutlass::half_t>(
        a, b, sfa, sfb, d, problem_sizes, expert_offsets, blockscale_offsets, stream);
  } else {
    TORCH_CHECK(false, "dtype must be kFloat16 or kBFloat16");
  }
```
**EN:** This section uses `TORCH_CHECK`, `getCurrentCUDAStream` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`、`getCurrentCUDAStream`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 37-40: Runtime integration and dispatch
```cpp
#else
  TORCH_CHECK(false, "No implemented es_sm100_mxfp8_blockscaled_grouped_mm for current device");
#endif
}
```
**EN:** This section uses `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `es_sm100_mxfp8_blockscaled_launcher.cuh`
- **External headers / 外部头文件**: `torch/all.h`
- **Path context / 路径上下文**: expert_specialization / es_sm100_mxfp8_blockscaled.cu
