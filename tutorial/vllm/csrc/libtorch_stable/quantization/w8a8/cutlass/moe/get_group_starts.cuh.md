# get_group_starts.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/w8a8/cutlass/moe/get_group_starts.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides kernel to compute starting pointers for grouped GEMM operations in MoE (Mixture of Experts) with 8-bit weight and activation quantization / 提供用于计算MoE（专家混合）模型中分组GEMM操作起始指针的内核，支持8位权重和激活量化

## Line-by-Line Analysis / 逐行分析

### Header Guards and Includes / 头文件保护与引入
```cpp
#pragma once

#include <cuda.h>
#include <torch/csrc/stable/tensor.h>
#include <torch/headeronly/core/ScalarType.h>
#include "libtorch_stable/torch_utils.h"

#include "cutlass/bfloat16.h"
#include "cutlass/float8.h"
```
**EN:** Sets up header guard and includes necessary CUDA, PyTorch stable API, and CUTLASS libraries for FP8 and BF16 data types.  
**CN:** 设置头文件保护并引入必要的CUDA、PyTorch稳定API和CUTLASS库，支持FP8和BF16数据类型。

### CUDA Kernel: get_group_gemm_starts / CUDA内核：获取分组GEMM起始位置
```cpp
template <typename ElementAB, typename ElementC, typename ElementAccumulator>
__global__ void get_group_gemm_starts(
    int64_t* expert_offsets, ElementAB** a_offsets, ElementAB** b_offsets,
    ElementC** out_offsets, ElementAccumulator** a_scales_offsets,
    ElementAccumulator** b_scales_offsets, ElementAB* a_base_as_int,
    ElementAB* b_base_as_int, ElementC* out_base_as_int,
    ElementAccumulator* a_scales_base_as_int,
    ElementAccumulator* b_scales_base_as_int, int64_t n, int64_t k,
    bool per_act_token, bool per_out_ch) {
  int expert_id = threadIdx.x;

  int64_t expert_offset = expert_offsets[expert_id];

  a_offsets[expert_id] = a_base_as_int + expert_offset * k;
  b_offsets[expert_id] = b_base_as_int + expert_id * k * n;
  out_offsets[expert_id] = out_base_as_int + expert_offset * n;
  a_scales_offsets[expert_id] =
      a_scales_base_as_int + (per_act_token ? expert_offset : 0);
  b_scales_offsets[expert_id] =
      b_scales_base_as_int + (per_out_ch ? n * expert_id : expert_id);
}
```
**EN:** Computes pointer offsets for each expert in grouped GEMM. Each thread handles one expert, calculating pointers to input matrices A (activations), B (weights), output C, and their quantization scales. Supports per-token and per-output-channel scaling modes.  
**CN:** 计算分组GEMM中每个专家的指针偏移量。每个线程处理一个专家，计算输入矩阵A（激活值）、B（权重）、输出C及其量化缩放因子的指针。支持按令牌和按输出通道的缩放模式。

### Macro for Kernel Dispatch / 内核调度宏
```cpp
#define __CALL_GET_STARTS_KERNEL(TENSOR_C_TYPE, C_TYPE)                    \
  else if (out_tensors.scalar_type() == TENSOR_C_TYPE) {                   \
    get_group_gemm_starts<cutlass::float_e4m3_t, C_TYPE, float>            \
        <<<1, num_experts, 0, stream>>>(                                   \
            static_cast<int64_t*>(expert_offsets.data_ptr()),              \
            static_cast<cutlass::float_e4m3_t**>(a_ptrs.data_ptr()),       \
            static_cast<cutlass::float_e4m3_t**>(b_ptrs.data_ptr()),       \
            static_cast<C_TYPE**>(out_ptrs.data_ptr()),                    \
            static_cast<float**>(a_scales_ptrs.data_ptr()),                \
            static_cast<float**>(b_scales_ptrs.data_ptr()),                \
            static_cast<cutlass::float_e4m3_t*>(a_tensors.data_ptr()),     \
            static_cast<cutlass::float_e4m3_t*>(b_tensors.data_ptr()),     \
            static_cast<C_TYPE*>(out_tensors.data_ptr()),                  \
            static_cast<float*>(a_scales.data_ptr()),                      \
            static_cast<float*>(b_scales.data_ptr()), out_tensors.size(1), \
            a_tensors.size(1), per_act_token, per_out_ch);                 \
  }
```
**EN:** Macro that dispatches the kernel with appropriate type parameters. Launches one block with num_experts threads, where input is FP8 (float_e4m3_t) and output can be BF16 or FP16.  
**CN:** 宏，使用适当的类型参数调度内核。启动一个块包含num_experts个线程，输入为FP8（float_e4m3_t），输出可以是BF16或FP16。

### Host Function: run_get_group_gemm_starts / 主机函数：运行分组GEMM起始计算
```cpp
void run_get_group_gemm_starts(
    torch::stable::Tensor const& expert_offsets, torch::stable::Tensor& a_ptrs,
    torch::stable::Tensor& b_ptrs, torch::stable::Tensor& out_ptrs,
    torch::stable::Tensor& a_scales_ptrs, torch::stable::Tensor& b_scales_ptrs,
    torch::stable::Tensor const& a_tensors,
    torch::stable::Tensor const& b_tensors, torch::stable::Tensor& out_tensors,
    torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales) {
  STD_TORCH_CHECK(a_tensors.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
  STD_TORCH_CHECK(b_tensors.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
  STD_TORCH_CHECK(a_scales.scalar_type() ==
                  torch::headeronly::ScalarType::Float);
  STD_TORCH_CHECK(b_scales.scalar_type() ==
                  torch::headeronly::ScalarType::Float);
  STD_TORCH_CHECK(expert_offsets.scalar_type() ==
                  torch::headeronly::ScalarType::Long);

  int num_experts = static_cast<int>(expert_offsets.size(0));
  bool per_act_token = a_scales.numel() != 1;
  bool per_out_ch = b_scales.numel() != num_experts;

  auto stream = get_current_cuda_stream(a_tensors.get_device_index());

  if (false) {
  }
  __CALL_GET_STARTS_KERNEL(torch::headeronly::ScalarType::BFloat16,
                           cutlass::bfloat16_t)
  __CALL_GET_STARTS_KERNEL(torch::headeronly::ScalarType::Half, half)
  else {
    STD_TORCH_CHECK(false, "Invalid output type (must be float16 or bfloat16)");
  }
}
```
**EN:** Host-side function that validates input tensor types (FP8 for inputs, Float32 for scales, Int64 for offsets), determines scaling modes, and dispatches kernel based on output dtype (BF16 or FP16).  
**CN:** 主机端函数，验证输入张量类型（输入为FP8，缩放因子为Float32，偏移量为Int64），确定缩放模式，并根据输出数据类型（BF16或FP16）调度内核。

## Key Concepts / 关键概念

**EN:**
- **Grouped GEMM**: Batch of matrix multiplications with different sizes, used in MoE layers where each expert processes a variable number of tokens
- **FP8 Quantization**: Uses 8-bit floating point (E4M3 format) for weights and activations to reduce memory and computation
- **Per-token vs Per-channel Scaling**: Two quantization granularities - per-token applies one scale per input token, per-channel applies different scales for each output channel
- **Pointer Array Approach**: Creates arrays of pointers for each expert's data to enable efficient batched processing

**CN:**
- **分组GEMM**：不同尺寸的批量矩阵乘法，用于MoE层，其中每个专家处理可变数量的令牌
- **FP8量化**：对权重和激活值使用8位浮点（E4M3格式）以减少内存和计算量
- **按令牌与按通道缩放**：两种量化粒度 - 按令牌为每个输入令牌应用一个缩放因子，按通道为每个输出通道应用不同的缩放因子
- **指针数组方法**：为每个专家的数据创建指针数组，以实现高效的批处理

## Dependencies / 依赖关系

**EN:**
- **CUDA Runtime**: Core GPU kernel execution
- **PyTorch Stable API**: Tensor abstraction and device management
- **CUTLASS**: NVIDIA's template library for high-performance linear algebra, provides FP8 types
- **Used by**: `grouped_mm_c3x.cuh` for initializing grouped GEMM operations

**CN:**
- **CUDA运行时**：核心GPU内核执行
- **PyTorch稳定API**：张量抽象和设备管理
- **CUTLASS**：NVIDIA的高性能线性代数模板库，提供FP8类型
- **被使用于**：`grouped_mm_c3x.cuh`用于初始化分组GEMM操作
