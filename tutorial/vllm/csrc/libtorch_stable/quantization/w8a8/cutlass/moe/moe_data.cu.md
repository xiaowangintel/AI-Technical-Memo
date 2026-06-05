# moe_data.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/w8a8/cutlass/moe/moe_data.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides CUDA kernels for computing MoE data structures (problem sizes, expert offsets, permutations) required for grouped GEMM execution / 提供CUDA内核用于计算MoE数据结构（问题尺寸、专家偏移量、排列），为分组GEMM执行做准备

## Line-by-Line Analysis / 逐行分析

### Constants and Includes / 常量与引入
```cpp
#include <cudaTypedefs.h>

#include "libtorch_stable/torch_utils.h"
#include <torch/csrc/stable/tensor.h>
#include <torch/csrc/stable/ops.h>
#include <torch/headeronly/core/ScalarType.h>

#include "libtorch_stable/dispatch_utils.h"

#include <iostream>

constexpr uint64_t THREADS_PER_EXPERT = 512;
// threshold must match the dispatch logic in run_cutlass_moe_mm_sm90()
constexpr int SWAP_AB_THRESHOLD = 64;
```
**EN:** Defines constants for kernel configuration. Each expert gets 512 threads for counting tokens. SWAP_AB_THRESHOLD=64 determines when to transpose matrices (matches dispatch logic in SM90/SM100 kernels).  
**CN:** 定义内核配置的常量。每个专家获得512个线程用于计数令牌。SWAP_AB_THRESHOLD=64确定何时转置矩阵（与SM90/SM100内核中的调度逻辑匹配）。

### Kernel: compute_problem_sizes / 内核：计算问题尺寸
```cpp
template <bool SWAP_AB>
__global__ void compute_problem_sizes(const int32_t* __restrict__ topk_ids,
                                      int32_t* problem_sizes1,
                                      int32_t* problem_sizes2,
                                      int32_t* atomic_buffer,
                                      const int topk_length, const int n,
                                      const int k, const bool is_gated) {
  int expert_id = blockIdx.x;
  // For gated activations (gate + up), first GEMM output is 2*n.
  // For non-gated activations (up only), first GEMM output is n.
  int const n1 = is_gated ? 2 * n : n;

  int occurrences = 0;
  for (int i = threadIdx.x; i < topk_length; i += THREADS_PER_EXPERT) {
    occurrences += (topk_ids[i] == expert_id);
  }
  atomicAdd(&atomic_buffer[expert_id], occurrences);
  __syncthreads();
```
**EN:** Each block handles one expert. Threads count how many tokens are routed to this expert by iterating over topk_ids. Uses atomic addition to accumulate counts. Handles gated activations (2x wider intermediate for gate+up projections).  
**CN:** 每个块处理一个专家。线程通过遍历topk_ids计算有多少令牌被路由到此专家。使用原子加法累加计数。处理门控激活（gate+up投影的中间层宽度为2倍）。

### Problem Shape Assignment / 问题形状分配
```cpp
  if (threadIdx.x == 0) {
    int final_occurrences = atomic_buffer[expert_id];
    if constexpr (!SWAP_AB) {
      problem_sizes1[expert_id * 3] = final_occurrences;
      problem_sizes1[expert_id * 3 + 1] = n1;
      problem_sizes1[expert_id * 3 + 2] = k;
      problem_sizes2[expert_id * 3] = final_occurrences;
      problem_sizes2[expert_id * 3 + 1] = k;
      problem_sizes2[expert_id * 3 + 2] = n;
    } else {
      problem_sizes1[expert_id * 3] = n1;
      problem_sizes1[expert_id * 3 + 1] = final_occurrences;
      problem_sizes1[expert_id * 3 + 2] = k;
      problem_sizes2[expert_id * 3] = k;
      problem_sizes2[expert_id * 3 + 1] = final_occurrences;
      problem_sizes2[expert_id * 3 + 2] = n;
    }
  }
}
```
**EN:** Thread 0 writes problem shapes for two GEMMs. GEMM1 computes intermediate (gate+up), GEMM2 computes final output (down projection). When SWAP_AB is true, M and N are swapped to make small batch dimension the leading dimension.  
**CN:** 线程0写入两个GEMM的问题形状。GEMM1计算中间层（gate+up），GEMM2计算最终输出（down投影）。当SWAP_AB为true时，M和N被交换以使小批量维度成为前导维度。

### Kernel: compute_expert_offsets / 内核：计算专家偏移量
```cpp
__global__ void compute_expert_offsets(
    const int32_t* __restrict__ problem_sizes1, int32_t* expert_offsets,
    int32_t* atomic_buffer, const int num_experts, const bool swap_ab) {
  int32_t tot_offset = 0;
  expert_offsets[0] = 0;
  for (int i = 0; i < num_experts; ++i) {
    atomic_buffer[i] = tot_offset;
    tot_offset += swap_ab ? problem_sizes1[i * 3 + 1] : problem_sizes1[i * 3];
    expert_offsets[i + 1] = tot_offset;
  }
}
```
**EN:** Sequential kernel (1 block, 1 thread) that computes prefix sum of token counts to get starting offset for each expert's data in the packed tensor. Also stores offsets in atomic_buffer for later use.  
**CN:** 顺序内核（1个块，1个线程），计算令牌计数的前缀和以获得每个专家在打包张量中的起始偏移量。还将偏移量存储在atomic_buffer中供以后使用。

### Kernel: compute_expert_blockscale_offsets / 内核：计算专家块缩放偏移量
```cpp
__global__ void compute_expert_blockscale_offsets(
    const int32_t* __restrict__ problem_sizes1, int32_t* expert_offsets,
    int32_t* blockscale_offsets, int32_t* atomic_buffer, const int num_experts,
    const bool swap_ab) {
  int32_t tot_offset = 0;
  int32_t tot_offset_round = 0;
  expert_offsets[0] = 0;
  blockscale_offsets[0] = 0;
  for (int i = 0; i < num_experts; ++i) {
    int32_t cur_offset =
        swap_ab ? problem_sizes1[i * 3 + 1] : problem_sizes1[i * 3];
    atomic_buffer[i] = tot_offset;
    tot_offset += cur_offset;
    expert_offsets[i + 1] = tot_offset;
    tot_offset_round += (cur_offset + (128 - 1)) / 128 * 128;
    blockscale_offsets[i + 1] = tot_offset_round;
  }
}
```
**EN:** Extended version of compute_expert_offsets for FP4 quantization. Additionally computes blockscale_offsets which are rounded to 128-element boundaries for block-wise quantization alignment.  
**CN:** compute_expert_offsets的扩展版本，用于FP4量化。此外计算blockscale_offsets，这些偏移量舍入到128元素边界以对齐块级量化。

### Kernel: compute_arg_sorts / 内核：计算参数排序
```cpp
__global__ void compute_arg_sorts(const int32_t* __restrict__ topk_ids,
                                  const int32_t* __restrict__ expert_offsets,
                                  int32_t* input_permutation,
                                  int32_t* output_permutation,
                                  int32_t* atomic_buffer, const int topk_length,
                                  const int topk) {
  int const blk_expert_id = blockIdx.x;
  int const num_experts = gridDim.x;
  int32_t const num_tokens = expert_offsets[num_experts];

  for (int i = threadIdx.x; i < topk_length; i += THREADS_PER_EXPERT) {
    int const expert_id = topk_ids[i];
    if (expert_id == -1 && blockIdx.x == 0) {
      // output_permutation is used to re-order the moe outputs. It is
      // used as c2 = c2[c_map], where c2 is a torch.tensor that is the
      // output of the cutlass kernels and c_map is the output_permutation.
      // c2 is initialized to zeros, therefore by setting the output_permutation
      // to num_tokens, we are guaranteed to fill the moe outputs to zero
      // for "invalid" topk_ids.
      output_permutation[i] = num_tokens;
    } else if (expert_id == blk_expert_id) {
      int start = atomicAdd(&atomic_buffer[expert_id], 1);
      input_permutation[start] = i / topk;
      output_permutation[i] = start;
    }
  }
}
```
**EN:** Computes permutation indices to reorder tokens. input_permutation maps packed position to original token index. output_permutation maps (token, top-k index) to packed position. Invalid expert IDs (-1) map to zero-padded position.  
**CN:** 计算排列索引以重新排序令牌。input_permutation将打包位置映射到原始令牌索引。output_permutation将（令牌，top-k索引）映射到打包位置。无效的专家ID（-1）映射到零填充位置。

### Host Function: launch_compute_problem_sizes / 主机函数：启动计算问题尺寸
```cpp
namespace {
inline void launch_compute_problem_sizes(const torch::stable::Tensor& topk_ids,
                                         torch::stable::Tensor& problem_sizes1,
                                         torch::stable::Tensor& problem_sizes2,
                                         torch::stable::Tensor& atomic_buffer,
                                         int64_t num_experts, int64_t n,
                                         int64_t k, cudaStream_t stream,
                                         const bool swap_ab,
                                         const bool is_gated) {
  int num_threads = min(THREADS_PER_EXPERT, topk_ids.numel());

  auto const* topk_ptr = topk_ids.const_data_ptr<int32_t>();
  auto* ps1_ptr = problem_sizes1.mutable_data_ptr<int32_t>();
  auto* ps2_ptr = problem_sizes2.mutable_data_ptr<int32_t>();
  auto* atomic_ptr = atomic_buffer.mutable_data_ptr<int32_t>();

  VLLM_STABLE_DISPATCH_BOOL(swap_ab, SwapAB, [&] {
    compute_problem_sizes<SwapAB><<<num_experts, num_threads, 0, stream>>>(
        topk_ptr, ps1_ptr, ps2_ptr, atomic_ptr,
        static_cast<int>(topk_ids.numel()), static_cast<int>(n),
        static_cast<int>(k), is_gated);
  });
}
}  // namespace
```
**EN:** Helper that dispatches compute_problem_sizes kernel with appropriate template parameter. Uses vLLM's dispatch macro to compile two versions (swap_ab true/false).  
**CN:** 使用适当的模板参数调度compute_problem_sizes内核的辅助函数。使用vLLM的调度宏编译两个版本（swap_ab为true/false）。

### Kernel: compute_problem_sizes_from_expert_offsets / 内核：从专家偏移量计算问题尺寸
```cpp
template <bool SWAP_AB>
__global__ void compute_problem_sizes_from_expert_offsets(
    const int64_t* __restrict__ expert_first_token_offset,
    int32_t* __restrict__ problem_sizes1, int32_t* __restrict__ problem_sizes2,
    const int num_experts, const int n, const int k) {
  int const expert_id = blockIdx.x * blockDim.x + threadIdx.x;
  if (expert_id >= num_experts) {
    return;
  }

  int64_t const m64 = expert_first_token_offset[expert_id + 1] -
                      expert_first_token_offset[expert_id];
  int32_t const m = static_cast<int32_t>(m64);

  int32_t* ps1 = problem_sizes1 + expert_id * 3;
  int32_t* ps2 = problem_sizes2 + expert_id * 3;

  if constexpr (!SWAP_AB) {
    // [M, 2*N, K]
    ps1[0] = m;
    ps1[1] = 2 * n;
    ps1[2] = k;
    // [M, K, N]
    ps2[0] = m;
    ps2[1] = k;
    ps2[2] = n;
  } else {
    // swap logical M/N in the problem shape
    // [2*N, M, K]
    ps1[0] = 2 * n;
    ps1[1] = m;
    ps1[2] = k;
    // [K, M, N]
    ps2[0] = k;
    ps2[1] = m;
    ps2[2] = n;
  }
}
```
**EN:** Alternative problem size computation when expert token counts are pre-computed and provided as offsets. Directly computes M from offset differences without scanning topk_ids. Assumes gated activation (2*n for first GEMM).  
**CN:** 当专家令牌计数预先计算并作为偏移量提供时的备选问题尺寸计算。直接从偏移量差异计算M，无需扫描topk_ids。假设门控激活（第一个GEMM为2*n）。

### Public API: get_cutlass_moe_mm_problem_sizes_from_expert_offsets_caller / 公共API：从专家偏移量获取CUTLASS MoE矩阵乘法问题尺寸
```cpp
void get_cutlass_moe_mm_problem_sizes_from_expert_offsets_caller(
    const torch::stable::Tensor& expert_first_token_offset,
    torch::stable::Tensor& problem_sizes1,
    torch::stable::Tensor& problem_sizes2, const int64_t n, const int64_t k,
    const bool swap_ab) {
  STD_TORCH_CHECK(expert_first_token_offset.is_cuda(),
                  "expert_first_token_offset must be a CUDA tensor");
  STD_TORCH_CHECK(expert_first_token_offset.scalar_type() ==
                      torch::headeronly::ScalarType::Long,
                  "expert_first_token_offset must be int64");

  STD_TORCH_CHECK(problem_sizes1.is_cuda() && problem_sizes2.is_cuda(),
                  "problem_sizes must be CUDA tensors");
  STD_TORCH_CHECK(
      problem_sizes1.scalar_type() == torch::headeronly::ScalarType::Int &&
          problem_sizes2.scalar_type() == torch::headeronly::ScalarType::Int,
      "problem_sizes must be int32");
  // ... more validation ...

  int const num_experts = static_cast<int>(num_experts64);
  auto stream =
      get_current_cuda_stream(expert_first_token_offset.get_device_index());

  int const threads = (num_experts < 256) ? num_experts : 256;
  int const blocks = (num_experts + threads - 1) / threads;

  auto const* offsets_ptr = expert_first_token_offset.const_data_ptr<int64_t>();
  auto* ps1_ptr = problem_sizes1.mutable_data_ptr<int32_t>();
  auto* ps2_ptr = problem_sizes2.mutable_data_ptr<int32_t>();

  VLLM_STABLE_DISPATCH_BOOL(swap_ab, SwapAB, [&] {
    compute_problem_sizes_from_expert_offsets<SwapAB>
        <<<blocks, threads, 0, stream>>>(offsets_ptr, ps1_ptr, ps2_ptr,
                                         num_experts, static_cast<int>(n),
                                         static_cast<int>(k));
  });
}
```
**EN:** Public entry point for offset-based problem size computation. Validates tensors, determines grid/block config, and launches kernel.  
**CN:** 基于偏移量的问题尺寸计算的公共入口点。验证张量，确定网格/块配置，并启动内核。

### Main API: get_cutlass_moe_mm_data_caller / 主API：获取CUTLASS MoE矩阵乘法数据
```cpp
void get_cutlass_moe_mm_data_caller(
    const torch::stable::Tensor& topk_ids,
    torch::stable::Tensor& expert_offsets,
    torch::stable::Tensor& problem_sizes1,
    torch::stable::Tensor& problem_sizes2,
    torch::stable::Tensor& input_permutation,
    torch::stable::Tensor& output_permutation, const int64_t num_experts,
    const int64_t n, const int64_t k,
    const std::optional<torch::stable::Tensor>& blockscale_offsets,
    const bool is_gated) {
  auto device = topk_ids.device();
  auto stream = get_current_cuda_stream(device.index());
  torch::stable::Tensor atomic_buffer = torch::stable::new_zeros(
      topk_ids, {num_experts}, torch::headeronly::ScalarType::Int);

  int num_threads = min(THREADS_PER_EXPERT, topk_ids.numel());

  // Swap-AB should be disabled for FP4 path
  bool may_swap_ab = (!blockscale_offsets.has_value()) &&
                     (topk_ids.numel() <= SWAP_AB_THRESHOLD);

  launch_compute_problem_sizes(topk_ids, problem_sizes1, problem_sizes2,
                               atomic_buffer, num_experts, n, k, stream,
                               may_swap_ab, is_gated);

  if (blockscale_offsets.has_value()) {
    // fp4 path
    compute_expert_blockscale_offsets<<<1, 1, 0, stream>>>(
        static_cast<const int32_t*>(problem_sizes1.data_ptr()),
        static_cast<int32_t*>(expert_offsets.data_ptr()),
        static_cast<int32_t*>(blockscale_offsets.value().data_ptr()),
        static_cast<int32_t*>(atomic_buffer.data_ptr()), num_experts,
        may_swap_ab);
  } else {
    compute_expert_offsets<<<1, 1, 0, stream>>>(
        static_cast<const int32_t*>(problem_sizes1.data_ptr()),
        static_cast<int32_t*>(expert_offsets.data_ptr()),
        static_cast<int32_t*>(atomic_buffer.data_ptr()), num_experts,
        may_swap_ab);
  }
  compute_arg_sorts<<<num_experts, num_threads, 0, stream>>>(
      static_cast<const int32_t*>(topk_ids.data_ptr()),
      static_cast<const int32_t*>(expert_offsets.data_ptr()),
      static_cast<int32_t*>(input_permutation.data_ptr()),
      static_cast<int32_t*>(output_permutation.data_ptr()),
      static_cast<int32_t*>(atomic_buffer.data_ptr()), topk_ids.numel(),
      topk_ids.size(1));
}
```
**EN:** Main orchestrator for MoE data preparation. Computes problem sizes, expert offsets (with optional blockscale for FP4), and permutation indices. Disables swap_ab for FP4 quantization and small workloads.  
**CN:** MoE数据准备的主编排器。计算问题尺寸、专家偏移量（FP4的可选块缩放）和排列索引。对于FP4量化和小工作负载禁用swap_ab。

### Batched MoE Data Kernel / 批量MoE数据内核
```cpp
template <bool SWAP_AB>
__global__ void compute_batched_moe_data(
    int32_t* expert_offsets, int32_t* problem_sizes1, int32_t* problem_sizes2,
    const int32_t* __restrict__ expert_num_tokens, const int padded_m,
    const int n, const int k) {
  int expert_idx = threadIdx.x;
  expert_offsets[expert_idx] = expert_idx * padded_m;

  if constexpr (!SWAP_AB) {
    problem_sizes1[expert_idx * 3] = expert_num_tokens[expert_idx];
    problem_sizes1[expert_idx * 3 + 1] = 2 * n;
    problem_sizes1[expert_idx * 3 + 2] = k;
    problem_sizes2[expert_idx * 3] = expert_num_tokens[expert_idx];
    problem_sizes2[expert_idx * 3 + 1] = k;
    problem_sizes2[expert_idx * 3 + 2] = n;
  } else {
    problem_sizes1[expert_idx * 3] = 2 * n;
    problem_sizes1[expert_idx * 3 + 1] = expert_num_tokens[expert_idx];
    problem_sizes1[expert_idx * 3 + 2] = k;
    problem_sizes2[expert_idx * 3] = k;
    problem_sizes2[expert_idx * 3 + 1] = expert_num_tokens[expert_idx];
    problem_sizes2[expert_idx * 3 + 2] = n;
  }
}

void get_cutlass_batched_moe_mm_data_caller(
    torch::stable::Tensor& expert_offsets,
    torch::stable::Tensor& problem_sizes1,
    torch::stable::Tensor& problem_sizes2,
    const torch::stable::Tensor& expert_num_tokens,
    const int64_t num_local_experts, const int64_t padded_m, const int64_t n,
    const int64_t k) {
  auto stream = get_current_cuda_stream(expert_offsets.get_device_index());

  if (num_local_experts * padded_m > SWAP_AB_THRESHOLD) {
    compute_batched_moe_data<false><<<1, num_local_experts, 0, stream>>>(
        static_cast<int32_t*>(expert_offsets.data_ptr()),
        static_cast<int32_t*>(problem_sizes1.data_ptr()),
        static_cast<int32_t*>(problem_sizes2.data_ptr()),
        static_cast<const int32_t*>(expert_num_tokens.data_ptr()), padded_m, n,
        k);
  } else {
    compute_batched_moe_data<true><<<1, num_local_experts, 0, stream>>>(
        static_cast<int32_t*>(expert_offsets.data_ptr()),
        static_cast<int32_t*>(problem_sizes1.data_ptr()),
        static_cast<int32_t*>(problem_sizes2.data_ptr()),
        static_cast<const int32_t*>(expert_num_tokens.data_ptr()), padded_m, n,
        k);
  }
}
```
**EN:** Alternative path for batched MoE where data is pre-padded and organized. Uses simpler uniform-stride layout instead of packed variable-length. Applies swap_ab heuristic based on total batch size.  
**CN:** 数据预填充和组织的批量MoE的备选路径。使用更简单的统一步幅布局而不是打包的可变长度。根据总批量大小应用swap_ab启发式。

## Key Concepts / 关键概念

**EN:**
- **Token Routing**: MoE layers route each token to top-K experts based on routing scores
- **Packing and Permutation**: Tokens for same expert are gathered together to enable batched GEMM, requiring input/output permutations to reorder data
- **Gated Activations**: SwiGLU-style MoE uses two projections (gate, up) multiplied element-wise, requiring 2N intermediate dimension
- **Swap AB Heuristic**: For small M, transposing problem can reduce padding overhead and improve performance
- **Block Quantization**: FP4 quantization requires 128-element aligned blocks for scale factors

**CN:**
- **令牌路由**：MoE层根据路由分数将每个令牌路由到top-K个专家
- **打包和排列**：将同一专家的令牌收集在一起以启用批量GEMM，需要输入/输出排列来重新排序数据
- **门控激活**：SwiGLU风格的MoE使用两个投影（gate，up）逐元素相乘，需要2N中间维度
- **交换AB启发式**：对于小M，转置问题可以减少填充开销并提高性能
- **块量化**：FP4量化需要128元素对齐的块用于缩放因子

## Dependencies / 依赖关系

**EN:**
- **CUDA Runtime**: Kernel execution and atomic operations
- **PyTorch Stable API**: Tensor allocation and device management
- **vLLM Dispatch Utils**: Template dispatch macros for compile-time branching
- **Used by**: Grouped GEMM kernels in `grouped_mm_c3x_sm90.cu` and `grouped_mm_c3x_sm100.cu` which consume the computed data structures
- **Coordination**: SWAP_AB_THRESHOLD must match threshold in SM90/SM100 dispatch logic

**CN:**
- **CUDA运行时**：内核执行和原子操作
- **PyTorch稳定API**：张量分配和设备管理
- **vLLM调度工具**：用于编译时分支的模板调度宏
- **被使用于**：`grouped_mm_c3x_sm90.cu`和`grouped_mm_c3x_sm100.cu`中的分组GEMM内核，消费计算出的数据结构
- **协调**：SWAP_AB_THRESHOLD必须与SM90/SM100调度逻辑中的阈值匹配
