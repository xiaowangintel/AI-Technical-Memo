# ops.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/moe/marlin_moe_wna16/ops.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements Marlin-based weight-only MoE GEMM kernels and supporting templates. / 实现基于 Marlin 的仅权重 MoE GEMM 内核及其模板支持代码。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-29)
```cpp
/*
 * Modified by Neural Magic
 * Copyright (C) Marlin.2024 Elias Frantar
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *         http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
// ...
#include "core/registration.h"

#define STATIC_ASSERT_SCALAR_TYPE_VALID(scalar_t)               \
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: permute_cols_kernel (lines 42-116)
```cpp
template <int moe_block_size>
__global__ void permute_cols_kernel(
    int4 const* __restrict__ a_int4_ptr, int const* __restrict__ perm_int_ptr,
    int4* __restrict__ out_int4_ptr,
    const int32_t* __restrict__ sorted_token_ids_ptr,
    const int32_t* __restrict__ expert_ids_ptr,
    const int32_t* __restrict__ num_tokens_past_padded_ptr, int size_m,
    int size_k, int top_k) {
  int num_tokens_past_padded = num_tokens_past_padded_ptr[0];
  int num_moe_blocks = div_ceil(num_tokens_past_padded, moe_block_size);
  int32_t block_sorted_ids[moe_block_size];
  int block_num_valid_tokens = 0;
  int64_t old_expert_id = 0;
  int64_t expert_id = 0;
// ...
      permute_row(block_sorted_ids[i]);
  }
}
```
**EN:** This CUDA kernel reorders token data so expert computation can run on contiguous batches, or restores the original order afterward.
**CN:** 该 CUDA 内核对 token 数据进行重排，使专家计算能够在连续批次上执行，或在计算后恢复原始顺序。

### Function / Kernel: get_kernel_cache_size (lines 175-216)
```cpp
int get_kernel_cache_size(thread_config_t const& th_config, bool m_block_size_8,
                          int thread_m_blocks, int prob_m, int prob_n,
                          int prob_k, int num_bits, int group_size,
                          bool has_act_order, bool is_k_full, int has_zp,
                          int is_zp_float, bool is_a_8bit, int stages) {
  int pack_factor = 32 / num_bits;

  // Get B size
  int tb_k = th_config.thread_k;
  int tb_n = th_config.thread_n;
  int tb_m = thread_m_blocks * 16;

  // shm size for block_sorted_ids/rd_block_sorted_ids/block_topk_weights
  // both of them requires tb_m * 4 bytes (tb_m * int32 or tb_m * float32)
// ...

  return total_size;
}
```
**EN:** This function implements the routing-side top-k selection logic and typically prepares indices, scores, or workspace for later kernels.
**CN:** 该函数实现路由侧的 top-k 选择逻辑，通常为后续内核准备索引、分数或工作空间。

### Function / Kernel: determine_exec_config (lines 266-334)
```cpp
exec_config_t determine_exec_config(
    const vllm::ScalarType& a_type, const vllm::ScalarType& b_type,
    const vllm::ScalarType& c_type, const vllm::ScalarType& s_type, int prob_m,
    int prob_n, int prob_k, int num_experts, int top_k, int thread_m_blocks,
    bool m_block_size_8, int num_bits, int group_size, bool has_act_order,
    bool is_k_full, bool has_zp, bool is_zp_float, bool is_a_8bit, int stages,
    int max_shared_mem, int sms) {
  exec_config_t exec_cfg = exec_config_t{1, thread_config_t{-1, -1, -1}};
  thread_config_t* thread_configs = thread_m_blocks > 1
                                        ? large_batch_thread_configs
                                        : small_batch_thread_configs;
  int thread_configs_size =
      thread_m_blocks > 1
          ? sizeof(large_batch_thread_configs) / sizeof(thread_config_t)
// ...

  return exec_cfg;
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: moe_wna16_marlin_gemm (lines 535-870)
```cpp
torch::Tensor moe_wna16_marlin_gemm(
    torch::Tensor& a, std::optional<torch::Tensor> c_or_none,
    torch::Tensor& b_q_weight,
    std::optional<torch::Tensor> const& b_bias_or_none, torch::Tensor& b_scales,
    std::optional<torch::Tensor> const& a_scales_or_none,
    std::optional<torch::Tensor> const& global_scale_or_none,
    std::optional<torch::Tensor> const& b_zeros_or_none,
    std::optional<torch::Tensor> const& g_idx_or_none,
    std::optional<torch::Tensor> const& perm_or_none, torch::Tensor& workspace,
    torch::Tensor& sorted_token_ids, torch::Tensor& expert_ids,
    torch::Tensor& num_tokens_past_padded, torch::Tensor& topk_weights,
    int64_t moe_block_size, int64_t top_k, bool mul_topk_weights,
    vllm::ScalarTypeId const& b_type_id, int64_t size_m, int64_t size_n,
    int64_t size_k, bool is_k_full, bool use_atomic_add, bool use_fp32_reduce,
// ...

  return c;
}
```
**EN:** This function implements the routing-side top-k selection logic and typically prepares indices, scores, or workspace for later kernels.
**CN:** 该函数实现路由侧的 top-k 选择逻辑，通常为后续内核准备索引、分数或工作空间。

### Registration: TORCH_LIBRARY (lines 872-874)
```cpp
TORCH_LIBRARY_IMPL_EXPAND(TORCH_EXTENSION_NAME, CUDA, m) {
  m.impl("moe_wna16_marlin_gemm", &moe_wna16_marlin_gemm);
}
```
**EN:** This block exposes the low-level implementation as a PyTorch extension entrypoint, so Python code can invoke the kernel through a stable schema.
**CN:** 该代码块把底层实现注册为 PyTorch 扩展入口，使 Python 侧能够通过稳定的 schema 调用这些内核。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Top-k selection / Top-k 选择
- Token permutation / Token 置换
- Tiled matrix multiplication / 分块矩阵乘法
- FP8 quantization / compute / FP8 量化与计算
- Low-bit quantization / 低比特量化
- Integer kernel specialization / 整数内核专用化
- Weight-only quantization / 仅权重量化

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `core/registration.h`
- **External libraries / 外部库**: C++ standard library or platform support
- **Runtime coupling / 运行时耦合**: Registered through the PyTorch extension mechanism / 通过 PyTorch 扩展机制注册; Launches GPU kernels and relies on CUDA/HIP execution semantics / 启动 GPU 内核并依赖 CUDA/HIP 执行语义; Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
