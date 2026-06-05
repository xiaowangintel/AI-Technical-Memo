# es_fp8_blockwise.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/expert_specialization/es_fp8_blockwise.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Headers and compile-time setup
```cpp
#include <ATen/cuda/CUDAEvent.h>
#include <torch/all.h>

#include <tuple>

#include "es_fp8_blockwise_launcher.cuh"

/**
 * @brief Performs blockwise grouped matrix multiplication on FP8 quantized inputs,
 *        with per-block scaling.
 *
 * This function dispatches to hardware-specific implementations (e.g., SM100 FP8)
 * to compute:
 *     C_i = scale_a[i] * A_i * scale_b[i] * B_i
 * for each expert group `i`, using input `problem_sizes` and `expert_offsets`
 * to describe the individual matrix dimensions and their offsets.
 *
 * Input tensors A and B must be quantized to 8-bit formats and dequantized before multiplication.
 * The output tensor is written with bfloat16 or half precision.
 *
 * @param output         Output tensor (must be of type bfloat16 or half).
 * @param a              Input tensor A (must be kFloat8_e4m3fn).
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 23-44: Runtime integration and dispatch
```cpp
 * @param b              Input tensor B (must be kFloat8_e4m3fn).
 * @param scales_a       Scaling factors for tensor A, float32 per expert group.
 * @param scales_b       Scaling factors for tensor B, float32 per expert group.
 * @param stride_a       Stride information for tensor A (int32).
 * @param stride_b       Stride information for tensor B (int32).
 * @param stride_c       Stride information for output tensor C (int32).
 * @param problem_sizes  2D int32 tensor of shape (num_experts, 3), specifying (M, N, K)
 *                       for each grouped matrix multiplication problem.
 * @param expert_offsets 1D int32 tensor of size (num_experts), used to index into
 *                       the grouped input tensors for dispatch.
 */
void es_fp8_blockwise_scaled_grouped_mm(
    torch::Tensor& output,
    const torch::Tensor& a,
    const torch::Tensor& b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b,
    const torch::Tensor& stride_a,
    const torch::Tensor& stride_b,
    const torch::Tensor& stride_d,
    const torch::Tensor& problem_sizes,
    const torch::Tensor& expert_offsets,
```
**EN:** This section uses the surrounding logic to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过相关逻辑等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 45-66: Runtime integration and dispatch
```cpp
    const torch::Tensor& workspace) {
#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED) && defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)
  TORCH_CHECK(problem_sizes.dim() == 2, "problem_sizes must be 2D tensor");
  TORCH_CHECK(problem_sizes.size(1) == 3, "problem_sizes must have shape (num_experts, 3)");
  TORCH_CHECK(
      problem_sizes.size(0) == expert_offsets.size(0), "Number of experts in problem_sizes must match expert_offsets");
  TORCH_CHECK(problem_sizes.dtype() == torch::kInt32, "problem_sizes must be int32");
  TORCH_CHECK(a.scalar_type() == torch::kFloat8_e4m3fn, "a must be kFloat8_e4m3fn");
  TORCH_CHECK(b.scalar_type() == torch::kFloat8_e4m3fn, "b must be kFloat8_e4m3fn");
  TORCH_CHECK(
      output.scalar_type() == torch::kBFloat16 || output.scalar_type() == torch::kHalf,
      "output must be bfloat16 or half");

  int num_experts = (int)problem_sizes.size(0);
  torch::TensorOptions options_int64 = torch::TensorOptions().dtype(torch::kInt64).device(a.device());
  torch::TensorOptions options_int32 = torch::TensorOptions().dtype(torch::kInt32).device(a.device());
  torch::Tensor out_ptrs = torch::empty(num_experts, options_int64);
  torch::Tensor a_ptrs = torch::empty(num_experts, options_int64);
  torch::Tensor b_ptrs = torch::empty(num_experts, options_int64);
  torch::Tensor a_scales_ptrs = torch::empty(num_experts, options_int64);
  torch::Tensor b_scales_ptrs = torch::empty(num_experts, options_int64);
```
**EN:** This section uses `defined`, `TORCH_CHECK`, `TensorOptions` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`defined`、`TORCH_CHECK`、`TensorOptions`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 67-86: Runtime integration and dispatch
```cpp
  torch::Tensor layout_sfa = torch::empty({num_experts, 5}, options_int32);
  torch::Tensor layout_sfb = torch::empty({num_experts, 5}, options_int32);

  torch::Tensor lm_problem_sizes = torch::empty({num_experts, 3}, options_int32);
  torch::Tensor mm_problem_sizes = torch::empty({num_experts, 3}, options_int32);
  torch::Tensor hm_problem_sizes = torch::empty({num_experts, 3}, options_int32);

  torch::Tensor backup_workspace_0 = torch::empty_like(workspace);
  torch::Tensor backup_workspace_1 = torch::empty_like(workspace);

  const std::string H20_device_type_str("NVIDIA H20");
  bool is_h20_device = std::string(at::cuda::getCurrentDeviceProperties()->name) == H20_device_type_str;

  auto stream = at::cuda::getCurrentCUDAStream();
  static auto backup_stream_0 = at::cuda::getStreamFromPool();
  static auto backup_stream_1 = at::cuda::getStreamFromPool();
  at::cuda::CUDAEvent start_event;
  at::cuda::CUDAEvent end_event_0;
  at::cuda::CUDAEvent end_event_1;
```
**EN:** This section uses `empty_like`, `H20_device_type_str`, `getCurrentCUDAStream` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`empty_like`、`H20_device_type_str`、`getCurrentCUDAStream`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 87-108: Runtime integration and dispatch
```cpp
  if (output.dtype() == torch::kBFloat16) {
    expert_specialization::es_sm90_fp8_blockwise_scaled_group_mm_pre_compute<cutlass::bfloat16_t>(
        out_ptrs,
        a_ptrs,
        b_ptrs,
        a_scales_ptrs,
        b_scales_ptrs,
        layout_sfa,
        layout_sfb,
        lm_problem_sizes,
        mm_problem_sizes,
        hm_problem_sizes,
        output,
        a,
        b,
        scales_a,
        scales_b,
        problem_sizes,
        expert_offsets,
        is_h20_device,
        stream.stream());
  } else if (output.dtype() == torch::kFloat16) {
```
**EN:** This section uses `stream` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`stream`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 109-131: Runtime integration and dispatch
```cpp
    expert_specialization::es_sm90_fp8_blockwise_scaled_group_mm_pre_compute<cutlass::half_t>(
        out_ptrs,
        a_ptrs,
        b_ptrs,
        a_scales_ptrs,
        b_scales_ptrs,
        layout_sfa,
        layout_sfb,
        lm_problem_sizes,
        mm_problem_sizes,
        hm_problem_sizes,
        output,
        a,
        b,
        scales_a,
        scales_b,
        problem_sizes,
        expert_offsets,
        is_h20_device,
        stream.stream());
  } else {
    TORCH_CHECK(false, "Invalid output type (must be float16 or bfloat16)");
  }
```
**EN:** This section uses `stream`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`stream`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 132-153: Runtime integration and dispatch
```cpp

  start_event.recordOnce(stream);
  start_event.block(backup_stream_0);
  start_event.block(backup_stream_1);

  if (output.dtype() == torch::kBFloat16) {
    expert_specialization::es_sm90_fp8_blockwise_scaled_group_mm_distpatch_out_dtype<cutlass::bfloat16_t>(
        out_ptrs,
        a_ptrs,
        b_ptrs,
        a_scales_ptrs,
        b_scales_ptrs,
        stride_a,
        stride_b,
        stride_d,
        layout_sfa,
        layout_sfb,
        lm_problem_sizes,
        mm_problem_sizes,
        hm_problem_sizes,
        workspace,
        backup_workspace_0,
```
**EN:** This section uses `recordOnce`, `block` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`recordOnce`、`block`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 154-175: Runtime integration and dispatch
```cpp
        backup_workspace_1,
        is_h20_device,
        stream.stream(),
        backup_stream_0.stream(),
        backup_stream_1.stream());
  } else if (output.dtype() == torch::kFloat16) {
    expert_specialization::es_sm90_fp8_blockwise_scaled_group_mm_distpatch_out_dtype<cutlass::half_t>(
        out_ptrs,
        a_ptrs,
        b_ptrs,
        a_scales_ptrs,
        b_scales_ptrs,
        stride_a,
        stride_b,
        stride_d,
        layout_sfa,
        layout_sfb,
        lm_problem_sizes,
        mm_problem_sizes,
        hm_problem_sizes,
        workspace,
        backup_workspace_0,
```
**EN:** This section uses `stream` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`stream`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 176-193: Runtime integration and dispatch
```cpp
        backup_workspace_1,
        is_h20_device,
        stream.stream(),
        backup_stream_0.stream(),
        backup_stream_1.stream());
  } else {
    TORCH_CHECK(false, "Invalid output type (must be float16 or bfloat16)");
  }

  end_event_0.recordOnce(backup_stream_0);
  end_event_1.recordOnce(backup_stream_1);
  end_event_0.block(stream);
  end_event_1.block(stream);
#else
  TORCH_CHECK_NOT_IMPLEMENTED(
      can_implement, "No implemented fp8_blockwise_scaled_grouped_mm for current compute capability: ", sm_version);
#endif
}
```
**EN:** This section uses `stream`, `TORCH_CHECK`, `recordOnce` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`stream`、`TORCH_CHECK`、`recordOnce`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `es_fp8_blockwise_launcher.cuh`
- **External headers / 外部头文件**: `ATen/cuda/CUDAEvent.h`, `torch/all.h`, `tuple`
- **Path context / 路径上下文**: expert_specialization / es_fp8_blockwise.cu
