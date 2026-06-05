# moe_permute_unpermute_kernel.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/moe/permute_unpermute_kernels/moe_permute_unpermute_kernel.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines low-level permutation/unpermutation CUDA kernels and dispatch helpers for MoE routing. / 定义 MoE 路由所需的底层置换/逆置换 CUDA 内核及分派辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 2-7)
```cpp
#include "moe_permute_unpermute_kernel.h"

// moe_permute kernels require at least CUDA 12.0
#if defined(CUDA_VERSION) && (CUDA_VERSION >= 12000)

// CubKeyValueSorter definition begin
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: CubKeyValueSorter::getWorkspaceSize (lines 25-41)
```cpp
size_t CubKeyValueSorter::getWorkspaceSize(size_t const num_key_value_pairs,
                                           int const num_experts) {
  int num_bits = expertsToBits(num_experts);
  size_t required_storage = 0;
  int* null_int = nullptr;
  cub::DeviceRadixSort::SortPairs(nullptr, required_storage, null_int, null_int,
                                  null_int, null_int, num_key_value_pairs, 0,
                                  num_bits);

  //   when num_key_value_pairs, num_experts, num_bits, required_storage = 64,
  //   4, 3, 0 The required_storage seems to vary between 0 and 1 for the same
  //   inputs
  if (required_storage == 0) {
    required_storage = 1;
  }
  return required_storage;
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: CubKeyValueSorter::run (lines 43-57)
```cpp
void CubKeyValueSorter::run(void* workspace, size_t const workspace_size,
                            int const* keys_in, int* keys_out,
                            int const* values_in, int* values_out,
                            size_t const num_key_value_pairs,
                            cudaStream_t stream) {
  size_t expected_ws_size = getWorkspaceSize(num_key_value_pairs, num_experts_);
  size_t actual_ws_size = workspace_size;

  TORCH_CHECK(expected_ws_size <= workspace_size,
              "[CubKeyValueSorter::run] The allocated workspace is too small "
              "to run this problem.");
  cub::DeviceRadixSort::SortPairs(workspace, actual_ws_size, keys_in, keys_out,
                                  values_in, values_out, num_key_value_pairs, 0,
                                  num_bits_, stream);
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: findTotalEltsLessThanTarget (lines 64-80)
```cpp
template <class T>
__device__ inline int64_t findTotalEltsLessThanTarget(T const* sorted_indices,
                                                      int64_t const arr_length,
                                                      T const target) {
  int64_t low = 0, high = arr_length - 1, target_location = -1;
  while (low <= high) {
    int64_t mid = (low + high) / 2;

    if (sorted_indices[mid] >= target) {
      high = mid - 1;
    } else {
      low = mid + 1;
      target_location = mid;
    }
  }
  return target_location + 1;
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: computeExpertFirstTokenOffsetKernel (lines 84-97)
```cpp
__global__ void computeExpertFirstTokenOffsetKernel(
    int const* sorted_experts, int64_t const sorted_experts_len,
    int const num_experts, int64_t* expert_first_token_offset) {
  // First, compute the global tid. We only need 1 thread per expert.
  int const expert = blockIdx.x * blockDim.x + threadIdx.x;

  // Note that expert goes [0, num_experts] (inclusive) because we want a count
  // for the total number of active tokens at the end of the scan.
  if (expert >= num_experts + 1) {
    return;
  }
  expert_first_token_offset[expert] =
      findTotalEltsLessThanTarget(sorted_experts, sorted_experts_len, expert);
}
```
**EN:** This CUDA kernel reorders token data so expert computation can run on contiguous batches, or restores the original order afterward.
**CN:** 该 CUDA 内核对 token 数据进行重排，使专家计算能够在连续批次上执行，或在计算后恢复原始顺序。

### Function / Kernel: sortAndScanExpert (lines 112-130)
```cpp
void sortAndScanExpert(const int* expert_for_source_row, const int* source_rows,
                       int* permuted_experts, int* permuted_rows,
                       int64_t* expert_first_token_offset, int num_rows,
                       int num_experts, int num_experts_per_node, int k,
                       CubKeyValueSorter& sorter, void* sorter_ws,
                       cudaStream_t stream) {
  int64_t const expanded_num_rows = static_cast<int64_t>(k) * num_rows;
  // We need to use the full num_experts because that is the sentinel value used
  // by topk for disabled experts
  sorter.updateNumExperts(num_experts);
  size_t const sorter_ws_size_bytes = pad_to_multiple_of_16(
      sorter.getWorkspaceSize(expanded_num_rows, num_experts));
  sorter.run((void*)sorter_ws, sorter_ws_size_bytes, expert_for_source_row,
             permuted_experts, source_rows, permuted_rows, expanded_num_rows,
// ...
                                num_experts_per_node, expert_first_token_offset,
                                stream);
}
```
**EN:** This function implements the routing-side top-k selection logic and typically prepares indices, scores, or workspace for later kernels.
**CN:** 该函数实现路由侧的 top-k 选择逻辑，通常为后续内核准备索引、分数或工作空间。

### Function / Kernel: preprocessTopkIdKernel (lines 132-160)
```cpp
__global__ void preprocessTopkIdKernel(int* topk_id_ptr, int size,
                                       const int* expert_map_ptr,
                                       int num_experts) {
  auto tidx = threadIdx.x;
  auto bidx = blockIdx.x;
  auto offset = bidx * blockDim.x;
  auto bound = min(offset + blockDim.x, size);
  extern __shared__ int smem_expert_map[];
  // store expert_map in smem
  for (int i = tidx; i < num_experts; i += blockDim.x) {
    smem_expert_map[i] = expert_map_ptr[i];
  }
  __syncthreads();

// ...
    topk_id_ptr[offset + tidx] = topk_id;
  }
}
```
**EN:** This CUDA kernel maps routing work onto threads/blocks to compute top-k experts and associated scores efficiently on the GPU.
**CN:** 该 CUDA 内核把路由计算映射到线程/线程块上，以便在 GPU 上高效计算 top-k 专家及其分数。

### Function / Kernel: preprocessTopkIdLauncher (lines 161-169)
```cpp
void preprocessTopkIdLauncher(int* topk_id_ptr, int size,
                              const int* expert_map_ptr, int num_experts,
                              cudaStream_t stream) {
  int block = std::min(size, 1024);
  int grid = (size + block - 1) / block;
  int smem_size = (num_experts) * sizeof(int);
  preprocessTopkIdKernel<<<grid, block, smem_size, stream>>>(
      topk_id_ptr, size, expert_map_ptr, num_experts);
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Top-k selection / Top-k 选择
- Token permutation / Token 置换
- Token restoration / Token 还原
- CUDA programming model / CUDA 编程模型
- CUDA kernel launch and thread mapping / CUDA 内核启动与线程映射
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: C++ standard library or platform support
- **Runtime coupling / 运行时耦合**: Launches GPU kernels and relies on CUDA/HIP execution semantics / 启动 GPU 内核并依赖 CUDA/HIP 执行语义; Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
