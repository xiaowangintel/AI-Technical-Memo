# nvfp4_experts_quant.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/fp4/nvfp4_experts_quant.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Quantizes expert-partitioned activations into NVFP4, with both plain and fused SiLU-mul entry points. / 将按 expert 划分的激活量化为 NVFP4，并提供普通版与融合 SiLU+乘法版入口。

## Line-by-Line Analysis / 逐行分析
### Shared NVFP4 utility import
```cpp
#include "cuda_utils.h"
#include "nvfp4_utils.cuh"
#include "launch_bounds_utils.h"
```
**EN:** This file directly includes the common NVFP4 helper header, reusing the shared packing, scale, and fused-activation routines that power the expert quantization kernels.
**CN:** 该文件直接包含公共的 NVFP4 工具头，以复用 expert 量化内核所依赖的打包、尺度与融合激活辅助逻辑。

### Fast expert lookup for small expert counts
```cpp

    if constexpr (SMALL_NUM_EXPERTS) {
      for (int i = 0; i < n_experts; i++) {
        uint32_t current_offset = __ldca(&input_offset_by_experts[i]);
        uint32_t next_offset = __ldca(&input_offset_by_experts[i + 1]);
        if (rowIdx >= current_offset && rowIdx < next_offset) {
          rowIdx_in_expert = rowIdx - current_offset;
          expert_idx = i;
          break;
        }
      }
    } else {
      // Load input offsets into registers first, then do the computation.
      // Local array size set to 17 because of register limit.
```
**EN:** When the number of experts is small, the kernel performs a straightforward scan over expert offsets to locate the current row's expert interval with minimal setup cost.
**CN:** 当 expert 数量较少时，内核会直接扫描 expert 偏移数组，以较低的准备成本定位当前行所属的 expert 区间。

### Shared-memory offset cache for large expert counts
```cpp
  extern __shared__ uint32_t shared_input_offsets[];

  // Load input offsets into shared memory.
  // If n_experts is larger than 4, use vectorized int4 to save instructions.
  // If n_experts is smaller than 4, read directly.
  if constexpr (SMALL_NUM_EXPERTS) {
    for (int i = threadIdx.x; i < n_experts + 1; i += blockDim.x) {
      shared_input_offsets[i] = input_offset_by_experts[i];
    }
  } else {
    for (int i = threadIdx.x * 4; i < n_experts; i += blockDim.x * 4) {
      *reinterpret_cast<int4*>(&shared_input_offsets[i]) =
          *reinterpret_cast<const int4*>(&input_offset_by_experts[i]);
```
**EN:** For larger routing tables, expert offsets are staged in shared memory and then searched with binary search, reducing repeated global-memory reads.
**CN:** 对于更大的路由表，expert 偏移会先放入共享内存，再通过二分查找定位，从而减少重复的全局内存读取。

### Launcher heuristic between latency and throughput
```cpp
      (totalWorkSize + block.x * grid.x - 1) / (block.x * grid.x);
  if (blockRepeat > 1) {
    size_t shared_mem_size = (n_experts + 1) * sizeof(uint32_t);
    // The shared-memory vectorized offset load only handles full 4-expert
    // chunks. Use the scalar specialization for the remainder cases.
    if (n_experts >= 4 && n_experts % 4 == 0) {
      cvt_fp16_to_fp4<T, FUSE_SILU_MUL, false, false>
          <<<grid, block, shared_mem_size, stream>>>(
              m_topk, k, reinterpret_cast<T*>(input),
              reinterpret_cast<float*>(input_global_scale),
              reinterpret_cast<uint32_t*>(output),
              reinterpret_cast<uint32_t*>(output_scale),
              reinterpret_cast<uint32_t*>(input_offset_by_experts),
              reinterpret_cast<uint32_t*>(output_scale_offset_by_experts),
```
**EN:** The launcher measures available occupancy and repeats-per-block to decide whether shared-memory preprocessing is worthwhile for the given expert layout.
**CN:** 该启动器会结合可用占用率与每个 block 的重复次数，判断在当前 expert 布局下是否值得使用共享内存预处理。

### Validation for standard expert quantization
```cpp
  STD_TORCH_CHECK(output.scalar_type() == UINT8);
  STD_TORCH_CHECK(output_scale.scalar_type() == INT);

  const int BLOCK_SIZE = 16;
  STD_TORCH_CHECK(k % BLOCK_SIZE == 0, "k must be a multiple of 16");
  auto n_experts = input_global_scale.size(0);
  STD_TORCH_CHECK(input_offset_by_experts.size(0) == n_experts + 1);
  STD_TORCH_CHECK(output_scale_offset_by_experts.size(0) == n_experts + 1);
  STD_TORCH_CHECK(output.size(0) == m_topk);
  STD_TORCH_CHECK(output.size(1) == k / 2);
  int scales_k = k / BLOCK_SIZE;
```
**EN:** The standard entry checks output dtypes, block alignment, and metadata tensor sizes before delegating to the quantization launcher.
**CN:** 标准入口在调用量化启动器之前，会检查输出数据类型、块对齐以及元数据张量的尺寸。

### Fused SiLU-mul expert entry point
```cpp
  // Input has gate || up layout, so k = input.size(1) / 2
  auto k_times_2 = input.size(1);
  STD_TORCH_CHECK(k_times_2 % 2 == 0, "input width must be even (gate || up)");
  auto k = k_times_2 / 2;

  validate_fp4_experts_quant_inputs(output, output_scale, input,
                                    input_global_scale, input_offset_by_experts,
                                    output_scale_offset_by_experts, m_topk, k);

  auto n_experts = input_global_scale.size(0);
```
**EN:** The fused variant interprets the input width as concatenated gate and up projections, halves the logical K dimension, and then launches the same core quantization path with fusion enabled.
**CN:** 融合版本把输入宽度解释为拼接的 gate 与 up 投影，将逻辑 K 维减半，然后以启用融合的方式调用同一条核心量化路径。

## Key Concepts / 关键概念
- Expert-offset driven row ownership / 基于 expert 偏移定义行归属
- NVFP4 quantization with optional fused gate-up activation / 支持 gate-up 融合激活的 NVFP4 量化
- Shared-memory acceleration for large routing tables / 面向大路由表的共享内存加速

## Dependencies / 依赖关系
- `nvfp4_utils.cuh` for packing, scale generation, and fused SiLU helpers / 提供打包、尺度生成与融合 SiLU 辅助函数
- `dispatch_utils.h` for Half/BFloat16 specialization / 通过 `dispatch_utils.h` 做 Half/BFloat16 特化
- `torch::stable` tensor utilities for device stream access and validation / 使用 `torch::stable` 张量工具获取设备 stream 并执行校验
