# moe_permute_unpermute_op.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/moe/moe_permute_unpermute_op.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements MoE token permutation/unpermutation operators and launch code. / 实现 MoE token 置换/逆置换算子及其启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-9)
```cpp
#include <c10/core/ScalarType.h>
#include <torch/all.h>
#include <ATen/cuda/CUDAContext.h>
#include "permute_unpermute_kernels/moe_permute_unpermute_kernel.h"
#include "permute_unpermute_kernels/dispatch.h"
#include "core/registration.h"

// moe_permute kernels require at least CUDA 12.0
#if defined(CUDA_VERSION) && (CUDA_VERSION >= 12000)
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: moe_permute (lines 11-80)
```cpp
void moe_permute(
    const torch::Tensor& input,                      // [n_token, hidden]
    const torch::Tensor& topk_ids,                   // [n_token, topk]
    const torch::Tensor& token_expert_indices,       // [n_token, topk]
    const std::optional<torch::Tensor>& expert_map,  // [n_expert]
    int64_t n_expert, int64_t n_local_expert, int64_t topk,
    torch::Tensor& permuted_input,             // [permuted_size, hidden]
    torch::Tensor& expert_first_token_offset,  // [n_local_expert + 1]
    torch::Tensor& inv_permuted_idx,           // [n_token, topk]
    torch::Tensor& permuted_idx) {             // [permute_size]
  TORCH_CHECK(expert_first_token_offset.scalar_type() == at::ScalarType::Long,
              "expert_first_token_offset must be int64");
  TORCH_CHECK(topk_ids.scalar_type() == at::ScalarType::Int,
              "topk_ids must be int32");
// ...
        n_token, valid_num_ptr, n_hidden, topk, n_local_expert, stream);
  });
}
```
**EN:** This function implements the routing-side top-k selection logic and typically prepares indices, scores, or workspace for later kernels.
**CN:** 该函数实现路由侧的 top-k 选择逻辑，通常为后续内核准备索引、分数或工作空间。

### Function / Kernel: moe_unpermute (lines 82-112)
```cpp
void moe_unpermute(
    const torch::Tensor& permuted_hidden_states,  // [n_token * topk, hidden]
    const torch::Tensor& topk_weights,            // [n_token, topk]
    const torch::Tensor& inv_permuted_idx,        // [n_token, topk]
    const std::optional<torch::Tensor>&
        expert_first_token_offset,  // [n_local_expert+1]
    int64_t topk,
    torch::Tensor& hidden_states  // [n_token, hidden]
) {
  TORCH_CHECK(
      permuted_hidden_states.scalar_type() == hidden_states.scalar_type(),
      "permuted_hidden_states dtype must be same as hidden_states");
  auto n_token = hidden_states.size(0);
  auto n_hidden = hidden_states.size(1);
// ...
        stream);
  });
}
```
**EN:** This function implements the routing-side top-k selection logic and typically prepares indices, scores, or workspace for later kernels.
**CN:** 该函数实现路由侧的 top-k 选择逻辑，通常为后续内核准备索引、分数或工作空间。

### Function / Kernel: shuffleInputRowsKernel (lines 114-142)
```cpp
template <typename T>
__global__ void shuffleInputRowsKernel(const T* input,
                                       const int32_t* dst2src_map, T* output,
                                       int64_t num_src_rows,
                                       int64_t num_dst_rows, int64_t num_cols) {
  int64_t dest_row_idx = blockIdx.x;
  int64_t const source_row_idx = dst2src_map[dest_row_idx];

  if (blockIdx.x < num_dst_rows) {
    // Load 128-bits per thread
    constexpr int64_t ELEM_PER_THREAD = 128 / sizeof(T) / 8;
    using DataElem = cutlass::Array<T, ELEM_PER_THREAD>;

    // Duplicate and permute rows
// ...
    }
  }
}
```
**EN:** This CUDA kernel reorders token data so expert computation can run on contiguous batches, or restores the original order afterward.
**CN:** 该 CUDA 内核对 token 数据进行重排，使专家计算能够在连续批次上执行，或在计算后恢复原始顺序。

### Function / Kernel: shuffle_rows (lines 144-168)
```cpp
void shuffle_rows(const torch::Tensor& input_tensor,
                  const torch::Tensor& dst2src_map,
                  torch::Tensor& output_tensor) {
  TORCH_CHECK(input_tensor.scalar_type() == output_tensor.scalar_type(),
              "Input and output tensors must have the same data type");

  auto stream = at::cuda::getCurrentCUDAStream().stream();
  int64_t const blocks = output_tensor.size(0);
  int64_t const threads = 256;
  int64_t const num_dest_rows = output_tensor.size(0);
  int64_t const num_src_rows = input_tensor.size(0);
  int64_t const num_cols = input_tensor.size(1);

  TORCH_CHECK(!(num_cols % (128 / sizeof(input_tensor.scalar_type()) / 8)),
// ...
        num_dest_rows, num_cols);
  });
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: moe_permute (lines 172-180)
```cpp
void moe_permute(const torch::Tensor& input, const torch::Tensor& topk_ids,
                 const torch::Tensor& token_expert_indices,
                 const std::optional<torch::Tensor>& expert_map,
                 int64_t n_expert, int64_t n_local_expert, int64_t topk,
                 torch::Tensor& permuted_input,
                 torch::Tensor& expert_first_token_offset,
                 torch::Tensor& inv_permuted_idx, torch::Tensor& permuted_idx) {
  TORCH_CHECK(false, "moe_permute is not supported on CUDA < 12.0");
}
```
**EN:** This function implements the routing-side top-k selection logic and typically prepares indices, scores, or workspace for later kernels.
**CN:** 该函数实现路由侧的 top-k 选择逻辑，通常为后续内核准备索引、分数或工作空间。

### Function / Kernel: moe_unpermute (lines 182-188)
```cpp
void moe_unpermute(
    const torch::Tensor& permuted_hidden_states,
    const torch::Tensor& topk_weights, const torch::Tensor& inv_permuted_idx,
    const std::optional<torch::Tensor>& expert_first_token_offset, int64_t topk,
    torch::Tensor& hidden_states) {
  TORCH_CHECK(false, "moe_unpermute is not supported on CUDA < 12.0");
}
```
**EN:** This function implements the routing-side top-k selection logic and typically prepares indices, scores, or workspace for later kernels.
**CN:** 该函数实现路由侧的 top-k 选择逻辑，通常为后续内核准备索引、分数或工作空间。

### Registration: TORCH_LIBRARY (lines 200-203)
```cpp
TORCH_LIBRARY_IMPL_EXPAND(TORCH_EXTENSION_NAME, CUDA, m) {
  m.impl("moe_permute", &moe_permute);
  m.impl("moe_unpermute", &moe_unpermute);
}
```
**EN:** This block exposes the low-level implementation as a PyTorch extension entrypoint, so Python code can invoke the kernel through a stable schema.
**CN:** 该代码块把底层实现注册为 PyTorch 扩展入口，使 Python 侧能够通过稳定的 schema 调用这些内核。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Top-k selection / Top-k 选择
- Token permutation / Token 置换
- Token restoration / Token 还原
- Tiled matrix multiplication / 分块矩阵乘法
- Integer kernel specialization / 整数内核专用化
- CUDA programming model / CUDA 编程模型
- PyTorch custom operator registration / PyTorch 自定义算子注册

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `permute_unpermute_kernels/moe_permute_unpermute_kernel.h`, `permute_unpermute_kernels/dispatch.h`, `core/registration.h`
- **External libraries / 外部库**: PyTorch / c10, PyTorch / ATen
- **Runtime coupling / 运行时耦合**: Registered through the PyTorch extension mechanism / 通过 PyTorch 扩展机制注册; Launches GPU kernels and relies on CUDA/HIP execution semantics / 启动 GPU 内核并依赖 CUDA/HIP 执行语义; Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
