# moe_align_sum_kernels.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/moe/moe_align_sum_kernels.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements CUDA kernels that align and reduce MoE routing outputs. / 实现对 MoE 路由结果进行对齐与归约的 CUDA 内核。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-13)
```cpp
#include <torch/all.h>
#include <ATen/cuda/CUDAContext.h>
#include <c10/cuda/CUDAGuard.h>
#include <cub/cub.cuh>

#include <ATen/ATen.h>
#include <ATen/cuda/Atomic.cuh>

#include "../cuda_compat.h"
#include "../dispatch_utils.h"
#include "core/math.hpp"

#define CEILDIV(x, y) (((x) + (y) - 1) / (y))
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: moe_align_block_size (lines 495-587)
```cpp
void moe_align_block_size(torch::Tensor topk_ids, int64_t num_experts,
                          int64_t block_size, torch::Tensor sorted_token_ids,
                          torch::Tensor experts_ids,
                          torch::Tensor num_tokens_post_pad,
                          std::optional<torch::Tensor> maybe_expert_map) {
  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();

  int64_t padded_num_experts =
      ((num_experts + WARP_SIZE - 1) / WARP_SIZE) * WARP_SIZE;
  int experts_per_warp = WARP_SIZE;
  int threads = 1024;
  threads = ((threads + WARP_SIZE - 1) / WARP_SIZE) * WARP_SIZE;

  // BlockScan uses 1024 threads and assigns one thread per expert.
// ...
        }
      });
}
```
**EN:** This function implements the routing-side top-k selection logic and typically prepares indices, scores, or workspace for later kernels.
**CN:** 该函数实现路由侧的 top-k 选择逻辑，通常为后续内核准备索引、分数或工作空间。

### Function / Kernel: batched_moe_align_block_size (lines 589-614)
```cpp
void batched_moe_align_block_size(int64_t max_tokens_per_batch,
                                  int64_t block_size,
                                  torch::Tensor const& batch_num_tokens,
                                  torch::Tensor sorted_ids,
                                  torch::Tensor batch_ids,
                                  torch::Tensor num_tokens_post_pad) {
  namespace batched_kernel = vllm::moe::batched_moe_align_block_size;

  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();
  int32_t const B = batch_num_tokens.size(0);
  int32_t const num_blocks_per_batch =
      round_to_next_multiple_of(max_tokens_per_batch, block_size) / block_size;
  int32_t const num_blocks = num_blocks_per_batch * B;
  int64_t const sorted_ids_size = num_blocks * block_size;
// ...
      sorted_ids.data_ptr<int32_t>(), batch_ids.data_ptr<int32_t>(),
      num_tokens_post_pad.data_ptr<int32_t>());
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: moe_sum (lines 616-657)
```cpp
void moe_sum(torch::Tensor& input,   // [num_tokens, topk, hidden_size]
             torch::Tensor& output)  // [num_tokens, hidden_size]
{
  const int hidden_size = input.size(-1);
  const auto num_tokens = output.numel() / hidden_size;
  const int topk = input.size(1);

  dim3 grid(num_tokens);
  dim3 block(std::min(hidden_size, 1024));
  const at::cuda::OptionalCUDAGuard device_guard(device_of(output));
  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();

  switch (topk) {
    case 2:
// ...
      break;
  }
}
```
**EN:** This function implements the routing-side top-k selection logic and typically prepares indices, scores, or workspace for later kernels.
**CN:** 该函数实现路由侧的 top-k 选择逻辑，通常为后续内核准备索引、分数或工作空间。

### Function / Kernel: moe_lora_align_block_size (lines 659-796)
```cpp
void moe_lora_align_block_size(
    torch::Tensor topk_ids, torch::Tensor token_lora_mapping,
    int64_t num_experts, int64_t block_size, int64_t max_loras,
    int64_t max_num_tokens_padded, int64_t max_num_m_blocks,
    torch::Tensor sorted_token_ids, torch::Tensor expert_ids,
    torch::Tensor num_tokens_post_pad, torch::Tensor adapter_enabled,
    torch::Tensor lora_ids, std::optional<torch::Tensor> maybe_expert_map) {
  const int topk_num = topk_ids.size(1);

  TORCH_CHECK(block_size > 0, "block_size should be greater than 0. ");

  int device_max_shared_mem;
  auto dev = topk_ids.get_device();
  cudaDeviceGetAttribute(&device_max_shared_mem,
// ...
        }
      });
}
```
**EN:** This function implements the routing-side top-k selection logic and typically prepares indices, scores, or workspace for later kernels.
**CN:** 该函数实现路由侧的 top-k 选择逻辑，通常为后续内核准备索引、分数或工作空间。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Top-k selection / Top-k 选择
- CUDA programming model / CUDA 编程模型
- CUDA kernel launch and thread mapping / CUDA 内核启动与线程映射
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `../cuda_compat.h`, `../dispatch_utils.h`, `core/math.hpp`
- **External libraries / 外部库**: PyTorch / ATen, PyTorch / c10, CUB block primitives
- **Runtime coupling / 运行时耦合**: Launches GPU kernels and relies on CUDA/HIP execution semantics / 启动 GPU 内核并依赖 CUDA/HIP 执行语义; Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
