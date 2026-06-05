# scaled_mm_c2x_sm89_int8_dispatch.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/w8a8/cutlass/scaled_mm_c2x_sm89_int8_dispatch.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Defines INT8 GEMM kernel configurations for NVIDIA Ada Lovelace (SM89) GPUs, optimized for int8 quantization workloads with improved tensor core efficiency. / [CN] 为 NVIDIA Ada Lovelace（SM89）GPU 定义 INT8 GEMM 内核配置，针对 int8 量化工作负载优化，具有改进的张量核心效率。

## Line-by-Line Analysis / 逐行分析

### INT8 Fallback Configuration
```cpp
template <typename InType, typename OutType,
          template <typename, typename> typename Epilogue>
struct sm89_int8_fallback_gemm {
  // Shared mem requirement : 61440
  static_assert(std::is_same<InType, int8_t>());
  using TileShape = cutlass::gemm::GemmShape<32, 64, 128>;
  using WarpShape = cutlass::gemm::GemmShape<16, 64, 64>;
  using InstructionShape = typename cutlass::gemm::GemmShape<16, 8, 32>;
  static int32_t const MainLoopStages = 5;

  using Cutlass2xGemm =
      cutlass_2x_gemm<cutlass::arch::Sm89, enable_sm89_to_sm90, InType, OutType,
                      Epilogue, TileShape, WarpShape, InstructionShape, 5>;
};
```
**EN:** Conservative fallback uses small 32x64x128 tile with narrow 16x64 warp shape, requiring only 60KB shared memory for maximum device compatibility.
**CN:** 保守的后备使用小的 32x64x128 tile 和窄的 16x64 warp 形状，仅需 60KB 共享内存以实现最大设备兼容性。

### Default Configuration Structure
```cpp
struct sm89_int8_config_default {
  // M in (256, inf)
  using WarpShape = typename cutlass::gemm::GemmShape<64, 64, 64>;
  using InstructionShape = typename cutlass::gemm::GemmShape<16, 8, 32>;

  template <typename InType, typename OutType,
            template <typename, typename> typename Epilogue,
            typename... EpilogueArgs>
  static void dispatch(torch::stable::Tensor& out,
                       torch::stable::Tensor const& a,
                       torch::stable::Tensor const& b, EpilogueArgs&&... args) {
```
**EN:** Uses struct-based config pattern with static dispatch method. Default config for large M uses standard 64x64x64 warp shape.
**CN:** 使用基于结构的配置模式和静态分发方法。大 M 的默认配置使用标准的 64x64x64 warp 形状。

### N-Based Tile Selection for Large M
```cpp
uint32_t const n = out.size(1);
uint32_t const np2 = next_pow_2(n);

if (np2 <= 4096) {
  using TileShape = cutlass::gemm::GemmShape<128, 128, 64>;
  // 5-stage pipeline
} else if (np2 <= 8192) {
  using TileShape = cutlass::gemm::GemmShape<256, 128, 64>;
  // 3-stage pipeline
} else if (np2 <= 16384) {
  using TileShape = cutlass::gemm::GemmShape<128, 128, 64>;
  // 5-stage pipeline
} else {
  using TileShape = cutlass::gemm::GemmShape<256, 128, 64>;
  // 3-stage pipeline
}
```
**EN:** Complex N-based selection: uses wide 256x128 tile for N ∈ (4096, 8192] and N > 16384 with 3-stage pipeline; standard 128x128 otherwise with 5-stage.
**CN:** 复杂的基于 N 的选择：对 N ∈ (4096, 8192] 和 N > 16384 使用宽 256x128 tile 和 3 级流水线；否则使用标准 128x128 和 5 级流水线。

### M128 Configuration with Variable Warp Shapes
```cpp
struct sm89_int8_config_M128 {
  // M in (64, 128]
  template <...>
  static void dispatch(...) {
    if (np2 <= 8192) {
      using TileShape = cutlass::gemm::GemmShape<64, 128, 128>;
      using WarpShape = cutlass::gemm::GemmShape<64, 64, 64>;
    } else if (np2 <= 16384) {
      using TileShape = cutlass::gemm::GemmShape<128, 128, 64>;
      using WarpShape = cutlass::gemm::GemmShape<64, 64, 64>;
    } else {
      using TileShape = cutlass::gemm::GemmShape<64, 64, 128>;
      using WarpShape = cutlass::gemm::GemmShape<32, 64, 64>;
```
**EN:** M128 config switches warp shapes based on N: uses full 64x64x64 warp for small/medium N, reduced 32x64x64 warp for very large N to improve occupancy.
**CN:** M128 配置根据 N 切换 warp 形状：小/中等 N 使用完整的 64x64x64 warp，极大 N 使用减小的 32x64x64 warp 以提高占用率。

### M64 Balanced Configuration
```cpp
struct sm89_int8_config_M64 {
  // M in (32, 64]
  template <...>
  static void dispatch(...) {
    if (np2 <= 8192) {
      using TileShape = cutlass::gemm::GemmShape<64, 64, 128>;
      using WarpShape = cutlass::gemm::GemmShape<32, 64, 64>;
      // 5-stage pipeline
    } else {
      using TileShape = cutlass::gemm::GemmShape<64, 128, 128>;
      using WarpShape = cutlass::gemm::GemmShape<64, 64, 64>;
      // 3-stage pipeline
```
**EN:** M64 uses smaller warp (32x64) for small N with deep K=128 and 5-stage pipeline; switches to larger warp and wider N tile for big N with 3-stage.
**CN:** M64 对小 N 使用较小 warp（32x64）、深 K=128 和 5 级流水线；对大 N 切换到较大 warp 和更宽 N tile 以及 3 级流水线。

### M32 and M16 Small Batch Configs
```cpp
struct sm89_int8_config_M32 {
  // M in (16, 32]
  if (np2 <= 8192) {
    using TileShape = cutlass::gemm::GemmShape<32, 64, 128>;
    using WarpShape = cutlass::gemm::GemmShape<16, 64, 64>;

struct sm89_int8_config_M16 {
  // M in [1, 16]
  if (np2 <= 8192) {
    using TileShape = cutlass::gemm::GemmShape<16, 64, 128>;
    using WarpShape = cutlass::gemm::GemmShape<16, 64, 64>;
```
**EN:** Very small M configs use minimal 16x64 warp shapes and narrow M tiles (16 or 32) to avoid wasted shared memory and thread resources.
**CN:** 极小 M 配置使用最小的 16x64 warp 形状和窄 M tile（16 或 32）以避免浪费共享内存和线程资源。

### Main Dispatcher
```cpp
template <typename InType, typename OutType,
          template <typename, typename> typename Epilogue,
          typename... EpilogueArgs>
inline void cutlass_gemm_sm89_int8_dispatch(torch::stable::Tensor& out,
                                            torch::stable::Tensor const& a,
                                            torch::stable::Tensor const& b,
                                            EpilogueArgs&&... args) {
  static_assert(std::is_same<InType, int8_t>());
  STD_TORCH_CHECK(a.scalar_type() == torch::headeronly::ScalarType::Char);
  STD_TORCH_CHECK(b.scalar_type() == torch::headeronly::ScalarType::Char);

  uint32_t const m = a.size(0);
  uint32_t const mp2 =
      std::max(static_cast<uint32_t>(16), next_pow_2(m));
```
**EN:** Entry point validates int8 inputs, computes next power-of-2 of M (minimum 16), and dispatches to appropriate M-bucket config.
**CN:** 入口点验证 int8 输入，计算 M 的下一个 2 的幂次（最小 16），并分发到适当的 M 桶配置。

### Pipeline Stage Variations
```cpp
// Large problems: 5-stage for small N, 3-stage for medium N
// Medium problems: Mix of 3-stage and 5-stage
// Small problems: Mostly 5-stage, some 4-stage
```
**EN:** SM89 int8 uses adaptive pipeline staging: 5-stage for high latency hiding when possible, 3-4 stage when shared memory is constrained by large tiles.
**CN:** SM89 int8 使用自适应流水线分级：在可能时使用 5 级以实现高延迟隐藏，当大 tile 约束共享内存时使用 3-4 级。

## Key Concepts / 关键概念

- **SM89 INT8 Optimization / SM89 INT8 优化**: [EN] Ada Lovelace int8 tensor cores with 16x8x32 instruction shape, 2x throughput vs SM80 / [CN] Ada Lovelace int8 张量核心，16x8x32 指令形状，相比 SM80 2倍吞吐量
- **Adaptive Warp Sizing / 自适应 Warp 大小**: [EN] Dynamically selects 64x64x64, 32x64x64, or 16x64x64 warp shapes based on problem size / [CN] 根据问题大小动态选择 64x64x64、32x64x64 或 16x64x64 warp 形状
- **Multi-Criteria Dispatch / 多标准分发**: [EN] First architecture to systematically vary tile, warp, and pipeline based on both M and N / [CN] 首个系统化地根据 M 和 N 变化 tile、warp 和流水线的架构
- **Shared Memory Efficiency / 共享内存效率**: [EN] Configs range from 60KB (fallback) to ~120KB (large tiles), optimized for SM89's 164KB max / [CN] 配置范围从 60KB（后备）到约 120KB（大 tile），针对 SM89 的 164KB 最大值优化
- **Power-of-2 Bucketing / 2 的幂次分桶**: [EN] Six M buckets (16, 32, 64, 128, 256, inf) with 2-4 N sub-variants each / [CN] 六个 M 桶（16、32、64、128、256、无穷大），每个有 2-4 个 N 子变体
- **Pipeline vs Tile Trade-off / 流水线与 Tile 权衡**: [EN] 3-stage enables wider tiles, 5-stage enables better latency hiding with smaller tiles / [CN] 3 级支持更宽 tile，5 级支持使用较小 tile 更好地隐藏延迟

## Dependencies / 依赖关系

- **Base Template / 基础模板**: [EN] `cutlass_2x_gemm` from `scaled_mm_c2x.cuh` / [CN] 来自 `scaled_mm_c2x.cuh`
- **Architecture Guard / 架构保护**: [EN] `enable_sm89_to_sm90` restricts to SM89-SM90 / [CN] 限制为 SM89-SM90
- **CUTLASS INT8 Types / CUTLASS INT8 类型**: [EN] Standard `int8_t` with `cutlass::arch::OpMultiplyAddSaturate` / [CN] 标准 `int8_t` 和 `cutlass::arch::OpMultiplyAddSaturate`
- **Fallback Mechanism / 后备机制**: [EN] `sm89_int8_fallback_gemm` with minimal memory footprint / [CN] 具有最小内存占用的后备
- **PyTorch INT8 Type / PyTorch INT8 类型**: [EN] `torch::headeronly::ScalarType::Char` validation / [CN] 类型验证
- **Utility Functions / 实用函数**: [EN] `next_pow_2()` for M/N bucketing, `fallback_cutlass_gemm_caller` / [CN] 用于 M/N 分桶和后备调用
