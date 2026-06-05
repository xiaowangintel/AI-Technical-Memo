# mxfp4_experts_quant.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/fp4/mxfp4_experts_quant.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Quantizes MoE expert activations into MXFP4, with optional fused SiLU-and-mul preprocessing. / 将 MoE expert 激活量化为 MXFP4，并支持可选的 SiLU+乘法融合预处理。

## Line-by-Line Analysis / 逐行分析
### Compile-time packing mode selection
```cpp
// nvfp4_utils.cuh.
#define NVFP4_ENABLE_ELTS16 1

#include <cuda.h>
```
**EN:** The file enables the 16-element packing configuration used by shared FP4 helpers so MXFP4 expert quantization can consume wide vector packs.
**CN:** 该文件启用了共享 FP4 工具所使用的 16 元素打包配置，使 MXFP4 expert 量化能够处理更宽的向量包。

### Optional fused SiLU-mul in the quantization kernel
```cpp
    PackedVec quant_input;
    if constexpr (FUSE_SILU_MUL) {
      PackedVec in_vec_up =
          reinterpret_cast<PackedVec const*>(in)[inOffset + colsPerRow];
      quant_input = compute_silu_mul(in_vec, in_vec_up);
    } else {
      quant_input = in_vec;
    }

    // In PACK16 mode, each thread outputs 16 E2M1 values = u32x2
```
**EN:** Inside the main kernel, the `FUSE_SILU_MUL` template flag decides whether the input should be used directly or first transformed by the gated SiLU-up fusion typical in MoE feed-forward blocks.
**CN:** 在主量化内核中，`FUSE_SILU_MUL` 模板参数决定是直接使用输入，还是先执行 MoE 前馈层常见的 gate 与 up 融合 SiLU 变换。

### Shared-memory binary search across experts
```cpp

    // Binary search through experts using shared memory
    int left = 0, right = n_experts - 1;
    while (left <= right) {
      int mid = (left + right) / 2;
      uint32_t mid_offset = shared_input_offsets[mid];
      uint32_t next_offset = shared_input_offsets[mid + 1];

      if (rowIdx >= mid_offset && rowIdx < next_offset) {
        rowIdx_in_expert = rowIdx - mid_offset;
```
**EN:** For larger expert counts, the kernel caches expert row offsets in shared memory and uses binary search to map a row to its expert segment with lower lookup overhead.
**CN:** 当 expert 数量较多时，内核会把 expert 行偏移缓存到共享内存中，并通过二分查找将一行映射到对应 expert 区段，从而降低查找开销。

### Heuristic launcher selection
```cpp
      (totalWorkSize + block.x * grid.x - 1) / (block.x * grid.x);
  if (blockRepeat > 1) {
    size_t shared_mem_size = (n_experts + 1) * sizeof(uint32_t);
    if (n_experts >= 4) {
      mxfp4_cvt_fp16_to_fp4<T, FUSE_SILU_MUL, false>
          <<<grid, block, shared_mem_size, stream>>>(
              m_topk, k, reinterpret_cast<T*>(input),
              reinterpret_cast<fp4_packed_t*>(output),
              reinterpret_cast<uint32_t*>(output_scale),
              reinterpret_cast<uint32_t*>(input_offset_by_experts),
              reinterpret_cast<uint32_t*>(output_scale_offset_by_experts),
              n_experts);
    } else {
      mxfp4_cvt_fp16_to_fp4<T, FUSE_SILU_MUL, true>
```
**EN:** The launcher picks between a shared-memory kernel and a low-latency direct-lookup kernel according to occupancy and expert count, balancing latency and throughput.
**CN:** 该启动器会根据占用率与 expert 数量在共享内存版本和低延迟直接查找版本之间做选择，以平衡时延与吞吐。

### Input checks and operator registration
```cpp
  STD_TORCH_CHECK(output_scale.scalar_type() == INT);

  STD_TORCH_CHECK(k % MXFP4_BLOCK_SIZE == 0, "k must be a multiple of 32");
  STD_TORCH_CHECK(input_offset_by_experts.size(0) == n_experts + 1);
  STD_TORCH_CHECK(output_scale_offset_by_experts.size(0) == n_experts + 1);
  STD_TORCH_CHECK(output.size(0) == m_topk);
  STD_TORCH_CHECK(output.size(1) == k / 2);
  int scales_k = k / MXFP4_BLOCK_SIZE;
  // K-dimension scale columns padded to a multiple of 4 for swizzle layout
  int padded_k = (scales_k + (4 - 1)) / 4 * 4;
  // 4 = 4 E8M0 values packed into one int32
  STD_TORCH_CHECK(output_scale.size(1) * 4 == padded_k);
}

void mxfp4_experts_quant(
    torch::stable::Tensor& output, torch::stable::Tensor& output_scale,
    torch::stable::Tensor const& input,
```
**EN:** The public entry enforces MXFP4 block-size constraints and matching offset metadata, and the file finishes by registering both the plain and fused expert-quantization operators into the CUDA library.
**CN:** 公开入口会强制执行 MXFP4 的块大小约束以及偏移元数据匹配，文件最后还会把普通版与融合版 expert 量化算子注册到 CUDA 库中。

## Key Concepts / 关键概念
- Expert-aware row partitioning / 感知 expert 边界的行划分
- MXFP4 quantization with optional fused activation / 支持可选融合激活的 MXFP4 量化
- Kernel selection based on occupancy heuristics / 基于占用率启发式的内核选择

## Dependencies / 依赖关系
- `nvfp4_utils.cuh` for shared quantization primitives / 通过 `nvfp4_utils.cuh` 复用公共量化原语
- `cuda_utils.h` and launch-bound helpers for kernel configuration / 使用 `cuda_utils.h` 与 launch bound 辅助函数配置内核
- Stable torch registration APIs for exposing CUDA ops / 使用 stable torch 注册 API 暴露 CUDA 算子
