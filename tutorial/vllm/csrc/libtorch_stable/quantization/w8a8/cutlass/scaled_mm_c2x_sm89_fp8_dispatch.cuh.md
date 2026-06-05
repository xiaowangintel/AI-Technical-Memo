# scaled_mm_c2x_sm89_fp8_dispatch.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/w8a8/cutlass/scaled_mm_c2x_sm89_fp8_dispatch.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Defines FP8 (E4M3) GEMM kernel configurations for NVIDIA Ada Lovelace (SM89) GPUs with native FP8 tensor core support. / [CN] 为 NVIDIA Ada Lovelace（SM89）GPU 定义 FP8（E4M3）GEMM 内核配置，具有原生 FP8 张量核心支持。

## Line-by-Line Analysis / 逐行分析

### FP8 Type Assertion and Include
```cpp
#pragma once
#include <torch/headeronly/util/shim_utils.h>
#include "scaled_mm_c2x.cuh"
#include "cutlass/float8.h"

template <typename InType, typename OutType,
          template <typename, typename> typename Epilogue>
struct sm89_fp8_fallback_gemm {
  // Shared Memory required by this Gemm - 61440 bytes
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using TileShape = typename cutlass::gemm::GemmShape<64, 128, 64>;
```
**EN:** Includes FP8 type definitions from CUTLASS. Fallback config uses conservative 64x128x64 tile requiring only 60KB shared memory for maximum compatibility.
**CN:** 从 CUTLASS 包含 FP8 类型定义。后备配置使用保守的 64x128x64 tile，仅需 60KB 共享内存以实现最大兼容性。

### Fast Accumulation Math Operator
```cpp
struct sm89_fp8_config_default {
  using WarpShape = typename cutlass::gemm::GemmShape<64, 64, 64>;
  using InstructionShape = typename cutlass::gemm::GemmShape<16, 8, 32>;
  using FP8MathOperator = typename cutlass::arch::OpMultiplyAddFastAccum;
```
**EN:** Uses `OpMultiplyAddFastAccum` for FP8, enabling faster but slightly less precise accumulation compared to standard `OpMultiplyAdd`.
**CN:** 对 FP8 使用 `OpMultiplyAddFastAccum`，实现更快但精度略低的累加，相比标准 `OpMultiplyAdd`。

### N-Aware Default Configuration
```cpp
template <typename InType, typename OutType,
          template <typename, typename> typename Epilogue,
          typename... EpilogueArgs>
static void dispatch(torch::stable::Tensor& out,
                     torch::stable::Tensor const& a,
                     torch::stable::Tensor const& b, EpilogueArgs&&... args) {
  uint32_t const n = out.size(1);
  uint32_t const np2 = next_pow_2(n);

  if (np2 <= 4096) {
    using TileShape = typename cutlass::gemm::GemmShape<128, 128, 64>;
    // 5-stage pipeline
  } else if (np2 <= 8192) {
    using TileShape = typename cutlass::gemm::GemmShape<256, 128, 64>;
    // 3-stage pipeline
```
**EN:** Default config (M > 256) further dispatches based on N. Uses wider 256x128 tile for medium N (4096 < N ≤ 8192) with 3-stage pipeline to fit shared memory.
**CN:** 默认配置（M > 256）进一步基于 N 分发。对中等 N（4096 < N ≤ 8192）使用更宽的 256x128 tile 和 3 级流水线以适应共享内存。

### M64 Mixed Math Operators
```cpp
struct sm89_fp8_config_M64 {
  // M in (32, 64]
  template <...>
  static void dispatch(...) {
    if (np2 <= 8196) {
      using TileShape = cutlass::gemm::GemmShape<64, 64, 128>;
      using WarpShape = cutlass::gemm::GemmShape<32, 64, 64>;
      using FP8MathOperator = typename cutlass::arch::OpMultiplyAdd;
    } else if (np2 <= 16384) {
      using FP8MathOperator = typename cutlass::arch::OpMultiplyAddFastAccum;
```
**EN:** M64 config switches between standard and fast accumulation based on N. Small N uses precise `OpMultiplyAdd`, large N uses `OpMultiplyAddFastAccum` for speed.
**CN:** M64 配置根据 N 在标准和快速累加之间切换。小 N 使用精确的 `OpMultiplyAdd`，大 N 使用 `OpMultiplyAddFastAccum` 以提高速度。

### M16 Fine-Grained N Thresholds
```cpp
struct sm89_fp8_config_M16 {
  // M in [1, 16]
  static void dispatch(...) {
    if (np2 <= 8192) {
      using TileShape = typename cutlass::gemm::GemmShape<16, 64, 128>;
    } else if (np2 <= 24576) {
      using TileShape = typename cutlass::gemm::GemmShape<16, 128, 64>;
    } else {
      using TileShape = typename cutlass::gemm::GemmShape<32, 64, 128>;
```
**EN:** Smallest M config has three N-based variants: narrow 16x64 for small N, balanced 16x128 for medium N, and promoted 32x64 for very large N.
**CN:** 最小 M 配置有三个基于 N 的变体：小 N 用窄 16x64，中等 N 用平衡 16x128，极大 N 用提升的 32x64。

### Batch Invariant Specialization
```cpp
template <typename InType, typename OutType,
          template <typename, typename> typename Epilogue,
          typename... EpilogueArgs>
inline void cutlass_gemm_sm89_fp8_batch_invariant_dispatch(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, EpilogueArgs&&... args) {
  // keep the CUTLASS config independent of M for batch invariance
  return sm89_fp8_config_M64::dispatch<InType, OutType, Epilogue>(
      out, a, b, std::forward<EpilogueArgs>(args)...);
}
```
**EN:** Batch-invariant path always uses M64 configuration regardless of actual M, ensuring consistent kernel selection and compilation cache hits across different batch sizes.
**CN:** 批次不变路径无论实际 M 如何都始终使用 M64 配置，确保在不同批次大小下一致的内核选择和编译缓存命中。

### FP8 Main Dispatcher
```cpp
inline void cutlass_gemm_sm89_fp8_dispatch(torch::stable::Tensor& out,
                                           torch::stable::Tensor const& a,
                                           torch::stable::Tensor const& b,
                                           EpilogueArgs&&... args) {
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  STD_TORCH_CHECK(a.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
  STD_TORCH_CHECK(b.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
```
**EN:** Public dispatch entry validates FP8 E4M3 input types and routes to M-bucket, which further dispatches on N dimension.
**CN:** 公共分发入口验证 FP8 E4M3 输入类型，路由到 M 桶，然后进一步基于 N 维度分发。

### Pipeline Stage Trade-offs
```cpp
// M > 256, N ≤ 4096: 5-stage pipeline
// M > 256, 4096 < N ≤ 8192: 3-stage pipeline with wider tile
// M ∈ (128, 256], N ≤ 4096: 3-stage pipeline
```
**EN:** SM89 FP8 configs balance between pipeline depth (more stages = better latency hiding) and tile size (larger tiles = more parallelism but more shared memory).
**CN:** SM89 FP8 配置在流水线深度（更多级 = 更好的延迟隐藏）和 tile 大小（更大的 tile = 更多并行但更多共享内存）之间平衡。

## Key Concepts / 关键概念

- **FP8 E4M3 Format / FP8 E4M3 格式**: [EN] 8-bit floating point with 4-bit exponent and 3-bit mantissa, native on SM89 tensor cores / [CN] 8 位浮点数，4 位指数和 3 位尾数，SM89 张量核心原生支持
- **SM89 / Ada Lovelace / SM89 / Ada Lovelace**: [EN] NVIDIA Ada architecture (RTX 40-series, L4, L40) with hardware FP8 support / [CN] NVIDIA Ada 架构（RTX 40 系列、L4、L40）具有硬件 FP8 支持
- **Fast Accumulation / 快速累加**: [EN] `OpMultiplyAddFastAccum` trades minor precision for ~20% speedup in FP8 / [CN] `OpMultiplyAddFastAccum` 以轻微精度换取 FP8 约 20% 加速
- **Two-Dimensional Dispatch / 二维分发**: [EN] First architecture with systematic M×N dispatch matrix for optimal config selection / [CN] 首个具有系统化 M×N 分发矩阵以实现最优配置选择的架构
- **Batch Invariance / 批次不变性**: [EN] Fixed M64 config eliminates recompilation overhead during dynamic batching / [CN] 固定的 M64 配置消除动态批处理期间的重新编译开销
- **3 vs 5-Stage Pipeline / 3 级与 5 级流水线**: [EN] Trades off shared memory usage (3-stage uses less) vs latency hiding (5-stage hides more) / [CN] 在共享内存使用（3 级用得少）与延迟隐藏（5 级隐藏更多）之间权衡

## Dependencies / 依赖关系

- **CUTLASS FP8 Support / CUTLASS FP8 支持**: [EN] `cutlass/float8.h` for `cutlass::float_e4m3_t` type / [CN] 用于 `cutlass::float_e4m3_t` 类型
- **Base Template / 基础模板**: [EN] `cutlass_2x_gemm` with FP8MathOperator parameter / [CN] 带 FP8MathOperator 参数
- **Architecture Guard / 架构保护**: [EN] `enable_sm89_to_sm90` ensures SM89+ compilation / [CN] 确保 SM89+ 编译
- **PyTorch FP8 Type / PyTorch FP8 类型**: [EN] `torch::headeronly::ScalarType::Float8_e4m3fn` / [CN] PyTorch FP8 E4M3 类型
- **Batch Invariant API / 批次不变 API**: [EN] `cutlass_scaled_mm_sm89_fp8_batch_invariant_epilogue` for optimized serving / [CN] 用于优化服务的批次不变 API
