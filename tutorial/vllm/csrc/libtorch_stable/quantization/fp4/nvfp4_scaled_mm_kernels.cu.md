# nvfp4_scaled_mm_kernels.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/fp4/nvfp4_scaled_mm_kernels.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the SM100 CUTLASS backend for scaled NVFP4 matrix multiplication, including argument building and tile-shape dispatch. / 实现面向 SM100 的缩放 NVFP4 矩阵乘 CUTLASS 后端，包括参数构建与 tile 形状分发。

## Line-by-Line Analysis / 逐行分析
### SM100 tile configuration structs
```cpp
// Configuration for M in (256, inf)
struct sm100_fp4_config_default {
  using KernelSchedule = cutlass::gemm::collective::KernelScheduleAuto;
  using EpilogueSchedule = cutlass::epilogue::collective::EpilogueScheduleAuto;
  using TileShape = Shape<_256, _256, _256>;
  using ClusterShape = Shape<_2, _1, _1>;
  using PerSmTileShape_MNK = Shape<_128, _256, _256>;
};

// Configuration for M in (16, 256]
```
**EN:** The file defines several tile-shape presets so the GEMM launcher can choose a configuration that matches the effective M dimension and hardware occupancy characteristics.
**CN:** 文件定义了多组 tile 形状预设，使 GEMM 启动器能够根据有效 M 维度与硬件占用特征选择合适配置。

### Building CUTLASS argument objects from tensors
```cpp
  auto stride_A = cutlass::make_cute_packed_stride(StrideA{}, {m, k, 1});
  auto stride_B = cutlass::make_cute_packed_stride(StrideB{}, {n, k, 1});
  auto stride_D = cutlass::make_cute_packed_stride(StrideD{}, {m, n, 1});

  auto layout_SFA = Sm100BlkScaledConfig::tile_atom_to_shape_SFA(
      cute::make_shape(m, n, k, 1));
  auto layout_SFB = Sm100BlkScaledConfig::tile_atom_to_shape_SFB(
      cute::make_shape(m, n, k, 1));
```
**EN:** This helper converts stable torch tensors into CUTLASS argument objects, deriving packed strides and scale-factor layouts for both input matrices.
**CN:** 这个辅助函数会把 stable torch 张量转换成 CUTLASS 参数对象，并为两个输入矩阵推导 packed stride 与尺度布局。

### Workspace allocation and GEMM launch
```cpp
  size_t workspace_size = Config::Gemm::get_workspace_size(arguments);
  auto workspace =
      torch::stable::empty(workspace_size, torch::headeronly::ScalarType::Byte,
                           std::nullopt, A.device());

  CUTLASS_CHECK(gemm.can_implement(arguments));

  CUTLASS_CHECK(gemm.initialize(arguments, workspace.data_ptr(), stream));
```
**EN:** Before launching CUTLASS, the wrapper queries workspace size, allocates a temporary byte tensor, checks implementability, and then initializes the GEMM object.
**CN:** 在启动 CUTLASS 之前，包装器会查询工作区大小、分配临时字节张量、检查可实现性，然后初始化 GEMM 对象。

### Dispatch by rounded-up M dimension
```cpp
  uint32_t const mp2 = std::max(static_cast<uint32_t>(16), next_pow_2(m));

  if (mp2 <= 16) {
    // m in [1, 16]
    runGemm<Fp4GemmSm100<sm100_fp4_config_M16, OutType>>(
        D, A, B, A_sf, B_sf, alpha, m, n, k, stream);
  } else if (mp2 <= 256) {
    // m in (16, 256]
    runGemm<Fp4GemmSm100<sm100_fp4_config_M256, OutType>>(
        D, A, B, A_sf, B_sf, alpha, m, n, k, stream);
  } else {
    // m in (256, inf)
    runGemm<Fp4GemmSm100<sm100_fp4_config_default, OutType>>(
```
**EN:** The dispatcher rounds M up to the next power of two and uses that value to pick a small, medium, or default tile strategy for better occupancy across different batch sizes.
**CN:** 分发器会把 M 向上取整到最近的 2 次幂，并据此在小、中、默认三种 tile 策略之间选择，以便在不同批大小下获得更好的占用率。

### Public entry validation and scale-shape checks
```cpp
  STD_TORCH_CHECK(k % alignment == 0, "Expected k to be divisible by ",
                  alignment, ", but got a shape: (", A.size(0), "x", A.size(1),
                  "), k: ", k, ".");
  STD_TORCH_CHECK(n % alignment == 0, "Expected n to be divisible by ",
                  alignment, ", but got b shape: (", B.size(0), "x", B.size(1),
                  ").");

  auto round_up = [](int x, int y) { return (x + y - 1) / y * y; };
  int rounded_m = round_up(m, 128);
  int rounded_n = round_up(n, 128);
  // Since k is divisible by 32 (alignment), k / 16 is guaranteed to be an
  // integer.
  int rounded_k = round_up(k / 16, 4);
```
**EN:** The exported entry verifies dtype, contiguity, alignment, and the padded/swizzled shapes of the scale tensors before dispatching the CUTLASS backend.
**CN:** 公开入口在分发 CUTLASS 后端之前，会校验数据类型、连续性、对齐要求以及尺度张量的填充/swizzle 形状。

## Key Concepts / 关键概念
- CUTLASS argument construction from torch tensors / 从 torch 张量构建 CUTLASS 参数
- Tile-shape selection based on effective M / 基于有效 M 的 tile 形状选择
- Workspace-managed SM100 FP4 GEMM launch / 通过工作区管理的 SM100 FP4 GEMM 启动

## Dependencies / 依赖关系
- CUTLASS GEMM builders, layouts, and packed-stride helpers / CUTLASS GEMM 构建器、布局与 packed-stride 辅助工具
- `core/math.hpp` for `next_pow_2` and related helpers / 通过 `core/math.hpp` 使用 `next_pow_2` 等辅助函数
- `torch::stable` tensor/device utilities for workspace allocation and stream selection / 使用 `torch::stable` 张量/设备工具进行工作区分配与 stream 选择
