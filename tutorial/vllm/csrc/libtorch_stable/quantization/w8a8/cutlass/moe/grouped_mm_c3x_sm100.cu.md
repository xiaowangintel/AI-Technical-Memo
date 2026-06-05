# grouped_mm_c3x_sm100.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/w8a8/cutlass/moe/grouped_mm_c3x_sm100.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements SM100 (Blackwell architecture) optimized grouped GEMM kernels for MoE with FP8 quantization / 实现针对SM100（Blackwell架构）优化的MoE分组GEMM内核，支持FP8量化

## Line-by-Line Analysis / 逐行分析

### Includes / 引入
```cpp
#include <cudaTypedefs.h>

#include "libtorch_stable/torch_utils.h"
#include <torch/csrc/stable/tensor.h>
#include <torch/headeronly/core/ScalarType.h>

#include "cutlass/cutlass.h"
#include "grouped_mm_c3x.cuh"

using namespace cute;
```
**EN:** Includes CUDA types, PyTorch stable API, CUTLASS core, and the grouped_mm_c3x header. Uses CuTe namespace for layout abstractions.  
**CN:** 引入CUDA类型、PyTorch稳定API、CUTLASS核心和grouped_mm_c3x头文件。使用CuTe命名空间进行布局抽象。

### Configuration: sm100_fp8_config_default / 配置：SM100 FP8默认配置
```cpp
template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue>
struct sm100_fp8_config_default {
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule =
      cutlass::gemm::KernelPtrArrayTmaWarpSpecialized1SmSm100;
  using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecialized1Sm;
  using TileShape = cute::Shape<cute::_128, cute::_256, cute::_128>;
  using ClusterShape = cute::Shape<cute::_1, cute::_1, cute::_1>;
  using ArchTag = cutlass::arch::Sm100;

  using Cutlass3xGemm =
      cutlass_3x_group_gemm<InType, OutType, ArchTag, Epilogue, TileShape,
                            ClusterShape, KernelSchedule, EpilogueSchedule>;
};
```
**EN:** Default SM100 configuration for general cases. Uses 128x256x128 tile, single-SM kernel schedule with TMA (Tensor Memory Accelerator) for async data movement, and warp-specialized execution.  
**CN:** SM100的默认配置用于一般情况。使用128x256x128瓦片，单SM内核调度，使用TMA（张量内存加速器）进行异步数据移动和warp专用执行。

### Configuration: sm100_fp8_config_M64 / 配置：SM100 FP8 M64配置
```cpp
template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue>
struct sm100_fp8_config_M64 {
  // M in [1,64]
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule =
      cutlass::gemm::KernelPtrArrayTmaWarpSpecialized1SmSm100;
  using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecialized1Sm;
  using TileShape = cute::Shape<cute::_128, cute::_16, cute::_128>;
  using ClusterShape = cute::Shape<cute::_1, cute::_1, cute::_1>;
  using ArchTag = cutlass::arch::Sm100;

  using Cutlass3xGemm =
      cutlass_3x_group_gemm<InType, OutType, ArchTag, Epilogue, TileShape,
                            ClusterShape, KernelSchedule, EpilogueSchedule,
                            true>;
};
```
**EN:** Optimized for small M (1-64 tokens). Uses narrow N-dimension tile (16) and enables swap_ab to make small M the leading dimension, reducing padding overhead.  
**CN:** 针对小M（1-64个令牌）优化。使用窄N维度瓦片（16）并启用swap_ab使小M成为前导维度，减少填充开销。

### Configuration: sm100_fp8_config_N8192 / 配置：SM100 FP8 N8192配置
```cpp
template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue>
struct sm100_fp8_config_N8192 {
  // N in [8192, inf)
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule =
      cutlass::gemm::KernelPtrArrayTmaWarpSpecialized2SmSm100;
  using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecialized2Sm;
  using TileShape = cute::Shape<cute::_128, cute::_256, cute::_128>;
  using ClusterShape = cute::Shape<cute::_2, cute::_1, cute::_1>;
  using ArchTag = cutlass::arch::Sm100;

  using Cutlass3xGemm =
      cutlass_3x_group_gemm<InType, OutType, ArchTag, Epilogue, TileShape,
                            ClusterShape, KernelSchedule, EpilogueSchedule>;
};
```
**EN:** Optimized for large N dimension (8192+). Uses 2-SM kernel schedule and cluster of 2 SMs in M-dimension to increase parallelism for wide matrices.  
**CN:** 针对大N维度（8192+）优化。使用2-SM内核调度和M维度上的2个SM集群，以增加宽矩阵的并行性。

### Function: run_cutlass_moe_mm_sm100 / 函数：运行CUTLASS MoE矩阵乘法SM100
```cpp
template <typename InType, typename OutType>
void run_cutlass_moe_mm_sm100(torch::stable::Tensor& out_tensors,
                              torch::stable::Tensor const& a_tensors,
                              torch::stable::Tensor const& b_tensors,
                              torch::stable::Tensor const& a_scales,
                              torch::stable::Tensor const& b_scales,
                              torch::stable::Tensor const& expert_offsets,
                              torch::stable::Tensor const& problem_sizes,
                              torch::stable::Tensor const& a_strides,
                              torch::stable::Tensor const& b_strides,
                              torch::stable::Tensor const& c_strides,
                              bool per_act_token, bool per_out_ch) {
  STD_TORCH_CHECK(a_tensors.size(0) > 0, "No input A tensors provided.");
  STD_TORCH_CHECK(b_tensors.size(0) > 0, "No input B tensors provided.");
  STD_TORCH_CHECK(out_tensors.size(0) > 0, "No output tensors provided.");

  STD_TORCH_CHECK(
      a_tensors.scalar_type() == torch::headeronly::ScalarType::Float8_e4m3fn,
      "A tensors must be of type float8_e4m3fn.");
  STD_TORCH_CHECK(
      b_tensors.scalar_type() == torch::headeronly::ScalarType::Float8_e4m3fn,
      "B tensors must be of type float8_e4m3fn.");
```
**EN:** Validates input tensors are non-empty and of correct FP8 type. Enforces Float8_e4m3fn format for both activations and weights.  
**CN:** 验证输入张量非空且类型正确。对激活值和权重都强制使用Float8_e4m3fn格式。

### Kernel Selection Logic / 内核选择逻辑
```cpp
  using Cutlass3xGemmDefault = typename sm100_fp8_config_default<
      InType, OutType, vllm::c3x::ScaledEpilogueArray>::Cutlass3xGemm;
  using Cutlass3xGemmN8192 = typename sm100_fp8_config_N8192<
      InType, OutType, vllm::c3x::ScaledEpilogueArray>::Cutlass3xGemm;
  using Cutlass3xGemmM64 = typename sm100_fp8_config_M64<
      InType, OutType, vllm::c3x::ScaledEpilogueArray>::Cutlass3xGemm;

  uint32_t const m = a_tensors.size(0);
  uint32_t const n = out_tensors.size(1);

  if (m <= 64) {
    cutlass_group_gemm_caller<Cutlass3xGemmM64>(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, expert_offsets,
        problem_sizes, a_strides, b_strides, c_strides, per_act_token,
        per_out_ch);
  } else if (n >= 8192) {
    cutlass_group_gemm_caller<Cutlass3xGemmN8192>(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, expert_offsets,
        problem_sizes, a_strides, b_strides, c_strides, per_act_token,
        per_out_ch);
  } else {
    cutlass_group_gemm_caller<Cutlass3xGemmDefault>(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, expert_offsets,
        problem_sizes, a_strides, b_strides, c_strides, per_act_token,
        per_out_ch);
  }
}
```
**EN:** Implements heuristic-based kernel selection: M64 config for small batches (≤64 tokens), N8192 for wide matrices (≥8192 output features), default otherwise. Optimizes for different problem geometries.  
**CN:** 实现基于启发式的内核选择：M64配置用于小批量（≤64个令牌），N8192用于宽矩阵（≥8192输出特征），其他情况使用默认配置。针对不同问题几何形状优化。

### Dispatcher: dispatch_moe_mm_sm100 / 调度器：分派MoE矩阵乘法SM100
```cpp
void dispatch_moe_mm_sm100(torch::stable::Tensor& out_tensors,
                           torch::stable::Tensor const& a_tensors,
                           torch::stable::Tensor const& b_tensors,
                           torch::stable::Tensor const& a_scales,
                           torch::stable::Tensor const& b_scales,
                           torch::stable::Tensor const& expert_offsets,
                           torch::stable::Tensor const& problem_sizes,
                           torch::stable::Tensor const& a_strides,
                           torch::stable::Tensor const& b_strides,
                           torch::stable::Tensor const& c_strides,
                           bool per_act_token, bool per_out_ch) {
  if (out_tensors.scalar_type() == torch::headeronly::ScalarType::BFloat16) {
    run_cutlass_moe_mm_sm100<cutlass::float_e4m3_t, cutlass::bfloat16_t>(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, expert_offsets,
        problem_sizes, a_strides, b_strides, c_strides, per_act_token,
        per_out_ch);
  } else {
    run_cutlass_moe_mm_sm100<cutlass::float_e4m3_t, cutlass::half_t>(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, expert_offsets,
        problem_sizes, a_strides, b_strides, c_strides, per_act_token,
        per_out_ch);
  }
}
```
**EN:** Dispatches to appropriate output type (BFloat16 or Float16) while keeping FP8 input. Provides flexibility in output precision without changing computation path.  
**CN:** 分派到适当的输出类型（BFloat16或Float16），同时保持FP8输入。在不更改计算路径的情况下提供输出精度灵活性。

### Public API: cutlass_moe_mm_sm100 / 公共API：CUTLASS MoE矩阵乘法SM100
```cpp
void cutlass_moe_mm_sm100(torch::stable::Tensor& out_tensors,
                          torch::stable::Tensor const& a_tensors,
                          torch::stable::Tensor const& b_tensors,
                          torch::stable::Tensor const& a_scales,
                          torch::stable::Tensor const& b_scales,
                          torch::stable::Tensor const& expert_offsets,
                          torch::stable::Tensor const& problem_sizes,
                          torch::stable::Tensor const& a_strides,
                          torch::stable::Tensor const& b_strides,
                          torch::stable::Tensor const& c_strides,
                          bool per_act_token, bool per_out_ch) {
  dispatch_moe_mm_sm100(out_tensors, a_tensors, b_tensors, a_scales, b_scales,
                        expert_offsets, problem_sizes, a_strides, b_strides,
                        c_strides, per_act_token, per_out_ch);
}
```
**EN:** Public entry point for SM100 MoE GEMM. Simply forwards to dispatcher, providing clean interface for external callers.  
**CN:** SM100 MoE GEMM的公共入口点。仅转发到调度器，为外部调用者提供清晰的接口。

## Key Concepts / 关键概念

**EN:**
- **SM100 (Blackwell)**: Latest NVIDIA GPU architecture with enhanced tensor cores and TMA for improved FP8 performance
- **TMA (Tensor Memory Accelerator)**: Hardware unit that asynchronously loads/stores tensors between global and shared memory, reducing warp overhead
- **Warp Specialization**: Different warps in a CTA perform different roles (data loading vs. computation), improving efficiency
- **Multi-SM Kernels**: For large problems, kernels can span 2 SMs to increase parallelism and resource utilization
- **Problem-Size Heuristics**: Selects optimal tile shape and schedule based on M/N dimensions for best performance

**CN:**
- **SM100（Blackwell）**：最新的NVIDIA GPU架构，具有增强的张量核心和TMA，可改善FP8性能
- **TMA（张量内存加速器）**：在全局内存和共享内存之间异步加载/存储张量的硬件单元，减少warp开销
- **Warp专用化**：CTA中的不同warp执行不同角色（数据加载vs.计算），提高效率
- **多SM内核**：对于大问题，内核可以跨越2个SM以增加并行性和资源利用率
- **问题大小启发式**：根据M/N维度选择最佳瓦片形状和调度以获得最佳性能

## Dependencies / 依赖关系

**EN:**
- **CUTLASS 3.x**: Template library providing SM100 kernel schedules and TMA support
- **grouped_mm_c3x.cuh**: Core grouped GEMM template and caller function
- **PyTorch Stable API**: Tensor interface and device management
- **vllm::c3x::ScaledEpilogueArray**: Custom epilogue for scaling in MoE context
- **Architecture Requirement**: SM100 (Blackwell) or later GPUs only

**CN:**
- **CUTLASS 3.x**：提供SM100内核调度和TMA支持的模板库
- **grouped_mm_c3x.cuh**：核心分组GEMM模板和调用函数
- **PyTorch稳定API**：张量接口和设备管理
- **vllm::c3x::ScaledEpilogueArray**：用于MoE上下文中缩放的自定义尾声
- **架构要求**：仅限SM100（Blackwell）或更高版本的GPU
