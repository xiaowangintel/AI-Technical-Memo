# grouped_mm_c3x_sm90.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/w8a8/cutlass/moe/grouped_mm_c3x_sm90.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements SM90 (Hopper architecture) optimized grouped GEMM kernels for MoE with FP8 quantization / 实现针对SM90（Hopper架构）优化的MoE分组GEMM内核，支持FP8量化

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
**EN:** Standard includes for CUDA types, PyTorch stable API, CUTLASS core, and the grouped_mm_c3x header. Uses CuTe namespace.  
**CN:** CUDA类型、PyTorch稳定API、CUTLASS核心和grouped_mm_c3x头文件的标准引入。使用CuTe命名空间。

### Configuration: sm90_fp8_config_default / 配置：SM90 FP8默认配置
```cpp
template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue>
struct sm90_fp8_config_default {
  // M in (16, inf)
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule =
      cutlass::gemm::KernelPtrArrayTmaWarpSpecializedPingpongFP8FastAccum;
  using EpilogueSchedule =
      cutlass::epilogue::PtrArrayTmaWarpSpecializedPingpong;
  using TileShape = cute::Shape<cute::_64, cute::_256, cute::_128>;
  using ClusterShape = cute::Shape<cute::_1, cute::_2, cute::_1>;
  using ArchTag = cutlass::arch::Sm90;

  using Cutlass3xGemm =
      cutlass_3x_group_gemm<InType, OutType, ArchTag, Epilogue, TileShape,
                            ClusterShape, KernelSchedule, EpilogueSchedule>;
};
```
**EN:** Default SM90 configuration for M > 16. Uses 64x256x128 tile with pingpong scheduling (double-buffering) and FP8 fast accumulation. Cluster of 2 in N-dimension for parallelism.  
**CN:** M > 16的SM90默认配置。使用64x256x128瓦片，乒乓调度（双缓冲）和FP8快速累加。N维度上的2个集群用于并行性。

### Configuration: sm90_fp8_config_M4 / 配置：SM90 FP8 M4配置
```cpp
template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue>
struct sm90_fp8_config_M4 {
  // M in [1, 4]
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule =
      cutlass::gemm::KernelPtrArrayTmaWarpSpecializedPingpongFP8FastAccum;
  using EpilogueSchedule =
      cutlass::epilogue::PtrArrayTmaWarpSpecializedPingpong;
  using TileShape = cute::Shape<cute::_128, cute::_16, cute::_128>;
  using ClusterShape = cute::Shape<cute::_1, cute::_1, cute::_1>;
  using ArchTag = cutlass::arch::Sm90;

  using Cutlass3xGemm =
      cutlass_3x_group_gemm<InType, OutType, ArchTag, Epilogue, TileShape,
                            ClusterShape, KernelSchedule, EpilogueSchedule,
                            true>;
};
```
**EN:** Optimized for very small M (1-4 tokens). Uses narrow N-tile (16) and swap_ab to avoid padding. Single-cluster configuration for minimal overhead.  
**CN:** 针对非常小的M（1-4个令牌）优化。使用窄N瓦片（16）和swap_ab以避免填充。单集群配置以获得最小开销。

### Configuration: sm90_fp8_config_M64 / 配置：SM90 FP8 M64配置
```cpp
template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue>
struct sm90_fp8_config_M64 {
  // M in (4, 64]
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule =
      cutlass::gemm::KernelPtrArrayTmaWarpSpecializedPingpongFP8FastAccum;
  using EpilogueSchedule =
      cutlass::epilogue::PtrArrayTmaWarpSpecializedPingpong;
  using TileShape = cute::Shape<cute::_128, cute::_16, cute::_256>;
  using ClusterShape = cute::Shape<cute::_2, cute::_1, cute::_1>;
  using ArchTag = cutlass::arch::Sm90;

  using Cutlass3xGemm =
      cutlass_3x_group_gemm<InType, OutType, ArchTag, Epilogue, TileShape,
                            ClusterShape, KernelSchedule, EpilogueSchedule,
                            true>;
};
```
**EN:** For small-to-medium M (5-64 tokens). Larger K-tile (256) than M4 config. Cluster of 2 in M-dimension with swap_ab for better parallelism on small batches.  
**CN:** 用于小到中等M（5-64个令牌）。比M4配置更大的K瓦片（256）。M维度上的2个集群和swap_ab，为小批量提供更好的并行性。

### Configuration: sm90_fp8_config_K8192 / 配置：SM90 FP8 K8192配置
```cpp
template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue>
struct sm90_fp8_config_K8192 {
  // K in [8192, inf)
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule =
      cutlass::gemm::KernelPtrArrayTmaWarpSpecializedPingpongFP8FastAccum;
  using EpilogueSchedule =
      cutlass::epilogue::PtrArrayTmaWarpSpecializedPingpong;
  using TileShape = cute::Shape<cute::_128, cute::_128, cute::_128>;
  using ClusterShape = cute::Shape<cute::_1, cute::_8, cute::_1>;
  using ArchTag = cutlass::arch::Sm90;

  using Cutlass3xGemm =
      cutlass_3x_group_gemm<InType, OutType, ArchTag, Epilogue, TileShape,
                            ClusterShape, KernelSchedule, EpilogueSchedule>;
};
```
**EN:** Optimized for large K dimension (≥8192, e.g., large hidden sizes). Balanced square tile 128x128x128, large cluster of 8 in N-dimension to parallelize across the K reduction.  
**CN:** 针对大K维度（≥8192，例如大隐藏尺寸）优化。平衡的方形瓦片128x128x128，N维度上的8个大集群以并行化K归约。

### Configuration: sm90_fp8_config_N8192 / 配置：SM90 FP8 N8192配置
```cpp
template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue>
struct sm90_fp8_config_N8192 {
  // N in [8192, inf)
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule =
      cutlass::gemm::KernelPtrArrayTmaWarpSpecializedPingpongFP8FastAccum;
  using EpilogueSchedule =
      cutlass::epilogue::PtrArrayTmaWarpSpecializedPingpong;
  using TileShape = cute::Shape<cute::_64, cute::_128, cute::_256>;
  using ClusterShape = cute::Shape<cute::_1, cute::_8, cute::_1>;
  using ArchTag = cutlass::arch::Sm90;

  using Cutlass3xGemm =
      cutlass_3x_group_gemm<InType, OutType, ArchTag, Epilogue, TileShape,
                            ClusterShape, KernelSchedule, EpilogueSchedule>;
};
```
**EN:** For large N dimension (≥8192, wide output). Large K-tile (256) for memory efficiency, cluster of 8 in N-dimension to partition wide output across SMs.  
**CN:** 用于大N维度（≥8192，宽输出）。大K瓦片（256）以提高内存效率，N维度上的8个集群将宽输出分区到各个SM。

### Function: run_cutlass_moe_mm_sm90 / 函数：运行CUTLASS MoE矩阵乘法SM90
```cpp
template <typename InType, typename OutType>
void run_cutlass_moe_mm_sm90(torch::stable::Tensor& out_tensors,
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
**EN:** Validates tensor dimensions and types. Ensures FP8 E4M3 format for both activations and weights.  
**CN:** 验证张量维度和类型。确保激活值和权重都使用FP8 E4M3格式。

### Kernel Selection with Multiple Heuristics / 多启发式内核选择
```cpp
  using Cutlass3xGemmN8192 = typename sm90_fp8_config_N8192<
      InType, OutType, vllm::c3x::ScaledEpilogueArray>::Cutlass3xGemm;
  using Cutlass3xGemmK8192 = typename sm90_fp8_config_K8192<
      InType, OutType, vllm::c3x::ScaledEpilogueArray>::Cutlass3xGemm;
  using Cutlass3xGemmM4 = typename sm90_fp8_config_M4<
      InType, OutType, vllm::c3x::ScaledEpilogueArray>::Cutlass3xGemm;
  using Cutlass3xGemmM64 = typename sm90_fp8_config_M64<
      InType, OutType, vllm::c3x::ScaledEpilogueArray>::Cutlass3xGemm;
  using Cutlass3xGemmDefault = typename sm90_fp8_config_default<
      InType, OutType, vllm::c3x::ScaledEpilogueArray>::Cutlass3xGemm;

  uint32_t const m = a_tensors.size(0);
  uint32_t const n = out_tensors.size(1);
  uint32_t const k = a_tensors.size(1);

  // Use swap_ab for M <= 64 by default to reduce padding
  if (m <= 4) {
    cutlass_group_gemm_caller<Cutlass3xGemmM4>(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, expert_offsets,
        problem_sizes, a_strides, b_strides, c_strides, per_act_token,
        per_out_ch);
  } else if (m <= 64) {
    cutlass_group_gemm_caller<Cutlass3xGemmM64>(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, expert_offsets,
        problem_sizes, a_strides, b_strides, c_strides, per_act_token,
        per_out_ch);
  } else if (n >= 8192) {
    cutlass_group_gemm_caller<Cutlass3xGemmN8192>(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, expert_offsets,
        problem_sizes, a_strides, b_strides, c_strides, per_act_token,
        per_out_ch);
  } else if (k >= 8192) {
    cutlass_group_gemm_caller<Cutlass3xGemmK8192>(
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
**EN:** Sophisticated heuristic-based dispatch: prioritizes M-size for small batches (≤4, ≤64), then checks large N (≥8192), then large K (≥8192), finally defaults. Ensures optimal kernel for diverse MoE workloads.  
**CN:** 复杂的基于启发式的调度：优先考虑小批量（≤4，≤64）的M大小，然后检查大N（≥8192），然后大K（≥8192），最后使用默认值。确保为各种MoE工作负载选择最佳内核。

### Dispatchers / 调度器
```cpp
void dispatch_moe_mm_sm90(torch::stable::Tensor& out_tensors,
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
    run_cutlass_moe_mm_sm90<cutlass::float_e4m3_t, cutlass::bfloat16_t>(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, expert_offsets,
        problem_sizes, a_strides, b_strides, c_strides, per_act_token,
        per_out_ch);
  } else {
    run_cutlass_moe_mm_sm90<cutlass::float_e4m3_t, cutlass::half_t>(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, expert_offsets,
        problem_sizes, a_strides, b_strides, c_strides, per_act_token,
        per_out_ch);
  }
}

void cutlass_moe_mm_sm90(torch::stable::Tensor& out_tensors,
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
  dispatch_moe_mm_sm90(out_tensors, a_tensors, b_tensors, a_scales, b_scales,
                       expert_offsets, problem_sizes, a_strides, b_strides,
                       c_strides, per_act_token, per_out_ch);
}
```
**EN:** Two-level dispatch: first by output dtype (BF16 vs FP16), then by problem geometry. Public API `cutlass_moe_mm_sm90` provides clean entry point.  
**CN:** 两级调度：首先按输出数据类型（BF16 vs FP16），然后按问题几何形状。公共API `cutlass_moe_mm_sm90`提供清晰的入口点。

## Key Concepts / 关键概念

**EN:**
- **SM90 (Hopper)**: NVIDIA H100 GPU architecture, first to support FP8 tensor cores natively
- **Pingpong Scheduling**: Double-buffering technique where one buffer loads data while another computes, hiding memory latency
- **FP8 Fast Accumulation**: Hopper feature allowing accumulation in lower precision (FP8/FP16) for better throughput vs. FP32
- **Fine-Grained Tuning**: Five different configurations cover workload spectrum from tiny (M=1-4) to large (N/K≥8192)
- **Thread Block Clusters**: Groups of CTAs that share L1 cache and synchronization, enabling multi-SM cooperation

**CN:**
- **SM90（Hopper）**：NVIDIA H100 GPU架构，首个原生支持FP8张量核心的架构
- **乒乓调度**：双缓冲技术，一个缓冲区加载数据而另一个计算，隐藏内存延迟
- **FP8快速累加**：Hopper特性，允许以较低精度（FP8/FP16）累加，比FP32具有更好的吞吐量
- **细粒度调优**：五种不同配置涵盖从微小（M=1-4）到大型（N/K≥8192）的工作负载范围
- **线程块集群**：共享L1缓存和同步的CTA组，实现多SM协作

## Dependencies / 依赖关系

**EN:**
- **CUTLASS 3.x**: Provides SM90-optimized kernels with pingpong scheduling and FP8 fast accum
- **grouped_mm_c3x.cuh**: Core grouped GEMM infrastructure
- **PyTorch Stable API**: Tensor operations and type checking
- **vllm::c3x::ScaledEpilogueArray**: Scaling epilogue for quantized computation
- **Architecture Requirement**: SM90 (Hopper H100) or later GPUs

**CN:**
- **CUTLASS 3.x**：提供SM90优化的内核，具有乒乓调度和FP8快速累加
- **grouped_mm_c3x.cuh**：核心分组GEMM基础设施
- **PyTorch稳定API**：张量操作和类型检查
- **vllm::c3x::ScaledEpilogueArray**：用于量化计算的缩放尾声
- **架构要求**：SM90（Hopper H100）或更高版本的GPU
