# moe_wna16.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/moe/moe_wna16.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements CUDA weight-only low-bit MoE kernels. / 实现 CUDA 仅权重量化低比特 MoE 内核。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 2-11)
```cpp
#include <torch/all.h>
#include <c10/cuda/CUDAGuard.h>
#include <ATen/cuda/CUDAContext.h>
#include <cuda_runtime.h>

#include <cuda_fp16.h>
#include <cuda_bf16.h>
#include "moe_wna16_utils.h"

#define DIVIDE(x, size) (((x) + (size) - 1) / (size))
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: run_moe_wna16_gemm (lines 225-272)
```cpp
template <typename scalar_t>
void run_moe_wna16_gemm(const scalar_t* input, scalar_t* output,
                        const uint32_t* b_qweight, const scalar_t* b_scales,
                        const uint32_t* b_qzeros, const float* topk_weights,
                        const int32_t* sorted_token_ids,
                        const int32_t* expert_ids,
                        const int32_t* num_tokens_post_pad, int num_experts,
                        int group_size, int num_token_blocks, int top_k,
                        int size_m, int size_n, int size_k, int BLOCK_SIZE_M,
                        int BLOCK_SIZE_N, int BLOCK_SIZE_K, int bit,
                        bool has_zp, bool mul_topk_weight) {
  dim3 blockDim, gridDim;
  blockDim.x = BLOCK_SIZE_N;
  blockDim.y = 1;
// ...
      group_size, top_k, size_m, size_n, size_k, BLOCK_SIZE_M, BLOCK_SIZE_N,
      BLOCK_SIZE_K, has_zp, mul_topk_weight);
}
```
**EN:** This function implements the routing-side top-k selection logic and typically prepares indices, scores, or workspace for later kernels.
**CN:** 该函数实现路由侧的 top-k 选择逻辑，通常为后续内核准备索引、分数或工作空间。

### Function / Kernel: moe_wna16_gemm (lines 274-342)
```cpp
torch::Tensor moe_wna16_gemm(torch::Tensor input, torch::Tensor output,
                             torch::Tensor b_qweight, torch::Tensor b_scales,
                             std::optional<torch::Tensor> b_qzeros,
                             std::optional<torch::Tensor> topk_weights,
                             torch::Tensor sorted_token_ids,
                             torch::Tensor expert_ids,
                             torch::Tensor num_tokens_post_pad, int64_t top_k,
                             int64_t BLOCK_SIZE_M, int64_t BLOCK_SIZE_N,
                             int64_t BLOCK_SIZE_K, int64_t bit) {
  const at::cuda::OptionalCUDAGuard device_guard(device_of(input));
  output.zero_();

  const int num_experts = b_qweight.size(0);
  const int size_m = input.size(0);
// ...
  }
  return output;
}
```
**EN:** This function implements the routing-side top-k selection logic and typically prepares indices, scores, or workspace for later kernels.
**CN:** 该函数实现路由侧的 top-k 选择逻辑，通常为后续内核准备索引、分数或工作空间。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Top-k selection / Top-k 选择
- Tiled matrix multiplication / 分块矩阵乘法
- Integer kernel specialization / 整数内核专用化
- Weight-only quantization / 仅权重量化
- CUDA programming model / CUDA 编程模型
- CUDA kernel launch and thread mapping / CUDA 内核启动与线程映射
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: PyTorch / ATen, PyTorch / c10, CUDA runtime / CUDA headers, C++ standard library or platform support
- **Runtime coupling / 运行时耦合**: Launches GPU kernels and relies on CUDA/HIP execution semantics / 启动 GPU 内核并依赖 CUDA/HIP 执行语义; Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
