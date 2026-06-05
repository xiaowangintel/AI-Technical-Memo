# scaled_mm_c2x_sm75_dispatch.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/w8a8/cutlass/scaled_mm_c2x_sm75_dispatch.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Defines GEMM kernel configurations and dispatch logic for NVIDIA Turing (SM75) GPUs, optimized for different problem sizes. / [CN] 为 NVIDIA Turing（SM75）GPU 定义 GEMM 内核配置和分发逻辑，针对不同问题大小进行优化。

## Line-by-Line Analysis / 逐行分析

### Default Configuration
```cpp
template <typename InType, typename OutType,
          template <typename, typename> typename Epilogue>
struct sm75_config_default {
  // This config is used in 2 cases,
  // - M in (256, inf]
  // - M in (64, 128]
  // Shared memory required by this Gemm 32768
  static_assert(std::is_same<InType, int8_t>());
  using TileShape = typename cutlass::gemm::GemmShape<128, 128, 64>;
  using WarpShape = typename cutlass::gemm::GemmShape<64, 64, 64>;
  using InstructionShape = typename cutlass::gemm::GemmShape<8, 8, 16>;
  using Cutlass2xGemm =
      cutlass_2x_gemm<cutlass::arch::Sm75, enable_sm75_to_sm80, InType, OutType,
                      Epilogue, TileShape, WarpShape, InstructionShape, 2>;
};
```
**EN:** Default configuration for large and medium-small problem sizes (M > 256 or M in (64, 128]). Uses 128x128x64 tile with 32KB shared memory and 2-stage pipeline.
**CN:** 大型和中小型问题大小的默认配置（M > 256 或 M 在 (64, 128]）。使用 128x128x64 tile，32KB 共享内存和 2 级流水线。

### M256 Configuration
```cpp
template <typename InType, typename OutType,
          template <typename, typename> typename Epilogue>
struct sm75_config_M256 {
  // M in (128, 256]
  // Shared memory required by this Gemm 65536
  static_assert(std::is_same<InType, int8_t>());
  using TileShape = typename cutlass::gemm::GemmShape<128, 128, 128>;
  using WarpShape = typename cutlass::gemm::GemmShape<64, 64, 64>;
  using InstructionShape = typename cutlass::gemm::GemmShape<8, 8, 16>;
  using Cutlass2xGemm =
      cutlass_2x_gemm<cutlass::arch::Sm75, enable_sm75_to_sm80, InType, OutType,
                      Epilogue, TileShape, WarpShape, InstructionShape, 2>;
};
```
**EN:** Specialized for M in (128, 256] range. Increases K dimension to 128 for better data reuse, requiring 64KB shared memory.
**CN:** 专为 M 在 (128, 256] 范围定制。将 K 维度增加到 128 以获得更好的数据重用，需要 64KB 共享内存。

### M64 Configuration
```cpp
template <typename InType, typename OutType,
          template <typename, typename> typename Epilogue>
struct sm75_config_M64 {
  // M in (32, 64]
  // Shared memory required by this Gemm 49152
  static_assert(std::is_same<InType, int8_t>());
  using TileShape = typename cutlass::gemm::GemmShape<64, 128, 128>;
  using WarpShape = typename cutlass::gemm::GemmShape<64, 64, 64>;
  using InstructionShape = typename cutlass::gemm::GemmShape<8, 8, 16>;
  using Cutlass2xGemm =
      cutlass_2x_gemm<cutlass::arch::Sm75, enable_sm75_to_sm80, InType, OutType,
                      Epilogue, TileShape, WarpShape, InstructionShape, 2>;
};
```
**EN:** For small M (32 < M ≤ 64), reduces M dimension to 64 while keeping N=128, K=128 for better occupancy with 48KB shared memory.
**CN:** 对于小 M（32 < M ≤ 64），将 M 维度减小到 64，同时保持 N=128、K=128，使用 48KB 共享内存获得更好的占用率。

### M32 Configuration
```cpp
template <typename InType, typename OutType,
          template <typename, typename> typename Epilogue>
struct sm75_config_M32 {
  // M in [1, 32]
  // Shared memory required by this Gemm 49152
  static_assert(std::is_same<InType, int8_t>());
  using TileShape = typename cutlass::gemm::GemmShape<32, 128, 64>;
  using WarpShape = typename cutlass::gemm::GemmShape<32, 64, 64>;
  using InstructionShape = typename cutlass::gemm::GemmShape<8, 8, 16>;
  using Cutlass2xGemm =
      cutlass_2x_gemm<cutlass::arch::Sm75, enable_sm75_to_sm80, InType, OutType,
                      Epilogue, TileShape, WarpShape, InstructionShape, 2>;
};
```
**EN:** Smallest configuration for very small M (1 ≤ M ≤ 32). Uses 32x128x64 tile with reduced warp dimensions for minimal shared memory waste.
**CN:** 极小 M（1 ≤ M ≤ 32）的最小配置。使用 32x128x64 tile 和减小的 warp 维度以最小化共享内存浪费。

### Dispatch Function
```cpp
template <typename InType, typename OutType,
          template <typename, typename> typename Epilogue,
          typename... EpilogueArgs>
inline void cutlass_gemm_sm75_dispatch(torch::stable::Tensor& out,
                                       torch::stable::Tensor const& a,
                                       torch::stable::Tensor const& b,
                                       EpilogueArgs&&... args) {
  static_assert(std::is_same<InType, int8_t>());
  STD_TORCH_CHECK(a.scalar_type() == torch::headeronly::ScalarType::Char);
  STD_TORCH_CHECK(b.scalar_type() == torch::headeronly::ScalarType::Char);
```
**EN:** Runtime dispatch function that selects optimal kernel based on M dimension. Validates inputs are int8.
**CN:** 根据 M 维度选择最优内核的运行时分发函数。验证输入为 int8。

### Power-of-2 Based Selection
```cpp
uint32_t const m = a.size(0);
uint32_t const mp2 =
    std::max(static_cast<uint32_t>(32), next_pow_2(m));  // next power of 2
if (mp2 <= 32) {
  // M in [1, 32]
  return fallback_cutlass_gemm_caller<Cutlass2xGemmM32, FallbackGemm>(
      out, a, b, std::forward<EpilogueArgs>(args)...);
} else if (mp2 <= 64) {
  // M in (32, 64]
  return fallback_cutlass_gemm_caller<Cutlass2xGemmM64, FallbackGemm>(
      out, a, b, std::forward<EpilogueArgs>(args)...);
```
**EN:** Uses next power of 2 of M to select configuration, ensuring consistent performance for similar problem sizes. Minimum considered M is 32.
**CN:** 使用 M 的下一个 2 的幂次来选择配置，确保类似问题大小的一致性能。考虑的最小 M 为 32。

### Fallback Strategy
```cpp
// sm75_config_default has the least shared-memory requirements.
using FallbackGemm = Cutlass2xGemmDefault;
```
**EN:** Default configuration selected as fallback since it has minimal 32KB shared memory requirement, ensuring compatibility with all SM75 devices.
**CN:** 选择默认配置作为后备，因为它具有最小的 32KB 共享内存要求，确保与所有 SM75 设备兼容。

### Instruction Shape Details
```cpp
using InstructionShape = typename cutlass::gemm::GemmShape<8, 8, 16>;
```
**EN:** SM75 Turing tensor cores operate on 8x8x16 matrix fragments (8x8 output accumulated from 16 dot products).
**CN:** SM75 Turing 张量核心在 8x8x16 矩阵片段上操作（从 16 个点积累积的 8x8 输出）。

## Key Concepts / 关键概念

- **SM75 / Turing Architecture / SM75 / Turing 架构**: [EN] NVIDIA Turing GPUs (RTX 20-series, T4) with first-generation int8 tensor cores / [CN] NVIDIA Turing GPU（RTX 20 系列，T4）具有第一代 int8 张量核心
- **Tile Shape Tuning / Tile 形状调优**: [EN] Different MxNxK configurations optimized for various problem sizes to maximize SM utilization / [CN] 针对各种问题大小优化的不同 MxNxK 配置，以最大化 SM 利用率
- **Shared Memory Constraints / 共享内存约束**: [EN] SM75 has 64KB max shared memory per block; configs range from 32KB to 64KB / [CN] SM75 每个块最多 64KB 共享内存；配置范围从 32KB 到 64KB
- **2-Stage Pipeline / 2 级流水线**: [EN] Double-buffering strategy overlapping compute with global memory access / [CN] 双缓冲策略，计算与全局内存访问重叠
- **Power-of-2 Bucketing / 2 的幂次分桶**: [EN] Groups similar problem sizes to reduce kernel compilation variants / [CN] 将相似问题大小分组以减少内核编译变体
- **Warp Shape / Warp 形状**: [EN] Defines workload per warp (32 threads); SM75 uses 64x64x64 for most configs / [CN] 定义每个 warp（32 个线程）的工作负载；SM75 在大多数配置中使用 64x64x64

## Dependencies / 依赖关系

- **Base Template / 基础模板**: [EN] `cutlass_2x_gemm` from `scaled_mm_c2x.cuh` / [CN] 来自 `scaled_mm_c2x.cuh` 的基础模板
- **Architecture Guard / 架构保护**: [EN] `enable_sm75_to_sm80` ensures kernels only compile for SM75-SM80 / [CN] 确保内核仅为 SM75-SM80 编译
- **CUTLASS GemmShape / CUTLASS GemmShape**: [EN] `cutlass::gemm::GemmShape<M, N, K>` defines tile dimensions / [CN] 定义 tile 维度
- **Fallback Caller / 后备调用器**: [EN] `fallback_cutlass_gemm_caller` handles shared memory overflow / [CN] 处理共享内存溢出
- **Utility Functions / 实用函数**: [EN] `next_pow_2()` from `torch/headeronly/util/shim_utils.h` / [CN] 来自工具头文件
