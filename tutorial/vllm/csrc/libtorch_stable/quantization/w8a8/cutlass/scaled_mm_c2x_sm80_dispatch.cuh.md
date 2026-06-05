# scaled_mm_c2x_sm80_dispatch.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/w8a8/cutlass/scaled_mm_c2x_sm80_dispatch.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Defines GEMM kernel configurations and dispatch logic for NVIDIA Ampere (SM80) GPUs with enhanced tensor core capabilities and larger shared memory. / [CN] 为 NVIDIA Ampere（SM80）GPU 定义 GEMM 内核配置和分发逻辑，具有增强的张量核心功能和更大的共享内存。

## Line-by-Line Analysis / 逐行分析

### Default Configuration
```cpp
template <typename InType, typename OutType,
          template <typename, typename> typename Epilogue>
struct sm80_config_default {
  // This config is used in 2 cases,
  //  - M in (128, inf)
  //  - M in (64, 128] and N >= 8192
  // Shared Memory required by this Gemm - 81920 bytes
  static_assert(std::is_same<InType, int8_t>());
  using TileShape = typename cutlass::gemm::GemmShape<128, 128, 64>;
  using WarpShape = typename cutlass::gemm::GemmShape<64, 64, 64>;
  using InstructionShape = typename cutlass::gemm::GemmShape<16, 8, 32>;
  using Cutlass2xGemm =
      cutlass_2x_gemm<cutlass::arch::Sm80, enable_sm80_to_sm89, InType, OutType,
                      Epilogue, TileShape, WarpShape, InstructionShape, 5>;
};
```
**EN:** Primary configuration for large M or large N problems. Uses 5-stage pipeline (vs SM75's 2-stage) and requires 80KB shared memory. SM80 tensor cores use 16x8x32 instruction shape.
**CN:** 大 M 或大 N 问题的主要配置。使用 5 级流水线（而非 SM75 的 2 级），需要 80KB 共享内存。SM80 张量核心使用 16x8x32 指令形状。

### M64 Configuration
```cpp
template <typename InType, typename OutType,
          template <typename, typename> typename Epilogue>
struct sm80_config_M64 {
  // This config is used in 2 cases,
  // - M in (32, 64]
  // - M in (64, 128] and N < 8192
  // Shared Memory required by this Gemm - 122880 bytes
  static_assert(std::is_same<InType, int8_t>());
  using TileShape = typename cutlass::gemm::GemmShape<64, 128, 128>;
  using WarpShape = typename cutlass::gemm::GemmShape<64, 64, 64>;
  using InstructionShape = typename cutlass::gemm::GemmShape<16, 8, 32>;
  using Cutlass2xGemm =
      cutlass_2x_gemm<cutlass::arch::Sm80, enable_sm80_to_sm89, InType, OutType,
                      Epilogue, TileShape, WarpShape, InstructionShape, 5>;
};
```
**EN:** Optimized for small-to-medium M with small N. Increases K dimension to 128 for better data reuse, consuming 120KB shared memory (80% of SM80's 164KB max).
**CN:** 针对中小型 M 和小 N 优化。将 K 维度增加到 128 以获得更好的数据重用，消耗 120KB 共享内存（SM80 最大 164KB 的 80%）。

### M32 Configuration
```cpp
template <typename InType, typename OutType,
          template <typename, typename> typename Epilogue>
struct sm80_config_M32 {
  // M in (16, 32]
  // Shared Memory required by this Gemm - 61440 bytes
  static_assert(std::is_same<InType, int8_t>());
  using TileShape = typename cutlass::gemm::GemmShape<32, 64, 128>;
  using WarpShape = typename cutlass::gemm::GemmShape<32, 64, 64>;
  using InstructionShape = typename cutlass::gemm::GemmShape<16, 8, 32>;
  using Cutlass2xGemm =
      cutlass_2x_gemm<cutlass::arch::Sm80, enable_sm80_to_sm89, InType, OutType,
                      Epilogue, TileShape, WarpShape, InstructionShape, 5>;
};
```
**EN:** For very small M (16 < M ≤ 32), uses narrow 32x64 output tile with deep K=128. Only 60KB shared memory enables higher occupancy.
**CN:** 对于极小 M（16 < M ≤ 32），使用窄 32x64 输出 tile，深 K=128。仅 60KB 共享内存实现更高占用率。

### M16 Configuration
```cpp
template <typename InType, typename OutType,
          template <typename, typename> typename Epilogue>
struct sm80_config_M16 {
  // M in [1, 16]
  // Shared Memory required by this Gemm - 51200 bytes
  static_assert(std::is_same<InType, int8_t>());
  using TileShape = typename cutlass::gemm::GemmShape<16, 64, 128>;
  using WarpShape = typename cutlass::gemm::GemmShape<16, 64, 64>;
  using InstructionShape = typename cutlass::gemm::GemmShape<16, 8, 32>;
  using Cutlass2xGemm =
      cutlass_2x_gemm<cutlass::arch::Sm80, enable_sm80_to_sm89, InType, OutType,
                      Epilogue, TileShape, WarpShape, InstructionShape, 5>;
};
```
**EN:** Minimal configuration for single-row or tiny batch inference. 16x64x128 tile with minimal 50KB shared memory footprint.
**CN:** 单行或微批推理的最小配置。16x64x128 tile，最小 50KB 共享内存占用。

### N-Aware Dispatch
```cpp
} else if (mp2 <= 128) {
  // M in (64, 128]
  uint32_t const n = out.size(1);
  bool const small_n = n < 8192;
  if (small_n) {
    return fallback_cutlass_gemm_caller<Cutlass2xGemmM128SmallN,
                                        FallbackGemm>(
        out, a, b, std::forward<EpilogueArgs>(args)...);
  } else {
    return fallback_cutlass_gemm_caller<Cutlass2xGemmM128BigN, FallbackGemm>(
        out, a, b, std::forward<EpilogueArgs>(args)...);
  }
}
```
**EN:** SM80 dispatch considers both M and N dimensions. For M in (64, 128], selects config based on N threshold of 8192, balancing shared memory vs compute intensity.
**CN:** SM80 分发同时考虑 M 和 N 维度。对于 M 在 (64, 128]，基于 N 阈值 8192 选择配置，平衡共享内存与计算强度。

### Fallback Strategy
```cpp
// sm80_config_M16 has the least shared-memory requirement. However,
// based on some profiling, we select sm80_config_M32 as a better alternative
// performance wise.
using FallbackGemm =
    typename sm80_config_M32<InType, OutType, Epilogue>::Cutlass2xGemm;
```
**EN:** Unlike SM75, SM80 uses M32 config (not smallest M16) as fallback based on empirical profiling, prioritizing performance over minimal memory usage.
**CN:** 与 SM75 不同，SM80 基于经验性能分析使用 M32 配置（而非最小的 M16）作为后备，优先考虑性能而非最小内存使用。

### 5-Stage Pipeline Advantage
```cpp
using Cutlass2xGemm =
    cutlass_2x_gemm<..., 5>;  // MainLoopStages = 5
```
**EN:** SM80's larger 164KB shared memory enables 5-stage software pipelining, overlapping 4 global loads with 1 compute stage for better latency hiding.
**CN:** SM80 更大的 164KB 共享内存支持 5 级软件流水线，将 4 个全局加载与 1 个计算阶段重叠，更好地隐藏延迟。

### Instruction Shape Enhancement
```cpp
using InstructionShape = typename cutlass::gemm::GemmShape<16, 8, 32>;
```
**EN:** SM80 tensor cores process 16x8x32 instructions (vs SM75's 8x8x16), doubling throughput per instruction for int8 operations.
**CN:** SM80 张量核心处理 16x8x32 指令（相比 SM75 的 8x8x16），int8 操作的每指令吞吐量翻倍。

## Key Concepts / 关键概念

- **SM80 / Ampere Architecture / SM80 / Ampere 架构**: [EN] NVIDIA Ampere GPUs (A100, A10, RTX 30-series) with 2nd-gen tensor cores / [CN] NVIDIA Ampere GPU（A100、A10、RTX 30 系列）具有第二代张量核心
- **5-Stage Pipeline / 5 级流水线**: [EN] Advanced software pipelining enabled by 164KB shared memory, vs SM75's 64KB limit / [CN] 由 164KB 共享内存支持的高级软件流水线，相比 SM75 的 64KB 限制
- **N-Dimension Sensitivity / N 维度敏感性**: [EN] First architecture to dispatch based on both M and N for better optimization / [CN] 首个基于 M 和 N 进行分发的架构，实现更好优化
- **Shared Memory Headroom / 共享内存余量**: [EN] 164KB max allows configs up to 120KB, enabling aggressive tile sizes / [CN] 164KB 最大值允许配置高达 120KB，支持激进的 tile 大小
- **Instruction Throughput / 指令吞吐量**: [EN] 16x8x32 instructions deliver 2x int8 throughput vs SM75 / [CN] 16x8x32 指令提供比 SM75 2倍的 int8 吞吐量
- **Empirical Tuning / 经验调优**: [EN] Fallback selection based on profiling rather than memory minimization / [CN] 基于性能分析而非内存最小化的后备选择

## Dependencies / 依赖关系

- **Base Template / 基础模板**: [EN] `cutlass_2x_gemm` from `scaled_mm_c2x.cuh` / [CN] 来自 `scaled_mm_c2x.cuh`
- **Architecture Guard / 架构保护**: [EN] `enable_sm80_to_sm89` restricts compilation to SM80-SM89 / [CN] 限制编译为 SM80-SM89
- **CUTLASS Ampere Features / CUTLASS Ampere 特性**: [EN] Uses SM80-specific optimizations and instruction shapes / [CN] 使用 SM80 特定优化和指令形状
- **Fallback Mechanism / 后备机制**: [EN] `fallback_cutlass_gemm_caller` with M32 config / [CN] 使用 M32 配置的后备调用器
- **Runtime Utilities / 运行时实用程序**: [EN] `next_pow_2()` for M bucketing, N dimension checks / [CN] 用于 M 分桶的工具，N 维度检查
