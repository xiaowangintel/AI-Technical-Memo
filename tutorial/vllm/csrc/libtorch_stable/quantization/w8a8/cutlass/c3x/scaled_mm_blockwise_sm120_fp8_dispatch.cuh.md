# scaled_mm_blockwise_sm120_fp8_dispatch.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_blockwise_sm120_fp8_dispatch.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements SM120-family blockwise FP8 kernel templates and heuristics, including ping-pong and swap-AB variants. / 实现 SM120 系列 blockwise FP8 的内核模板与启发式逻辑，包括 ping-pong 与 swap-AB 变体。

## Line-by-Line Analysis / 逐行分析
### Kernel template / 内核模板
```cpp
template <class OutType, int ScaleGranularityM,
          int ScaleGranularityN, int ScaleGranularityK,
          class MmaTileShape, class ClusterShape,
          class EpilogueScheduler, class MainloopScheduler,
          bool swap_ab_ = false>
struct cutlass_3x_gemm_fp8_blockwise {
  static constexpr bool swap_ab = swap_ab_;
  using ElementAB = cutlass::float_e4m3_t;

  using ElementA = ElementAB;
  using LayoutA = cutlass::layout::RowMajor;
  using LayoutA_Transpose = typename cutlass::layout::LayoutTranspose<LayoutA>::type;
  static constexpr int AlignmentA = 128 / cutlass::sizeof_bits<ElementA>::value;

  using ElementB = ElementAB;
  // ColumnMajor is used for B to match the CUTLASS convention.
  using LayoutB = cutlass::layout::ColumnMajor;
  using LayoutB_Transpose = typename cutlass::layout::LayoutTranspose<LayoutB>::type;
  static constexpr int AlignmentB = 128 / cutlass::sizeof_bits<ElementB>::value;

  using ElementD = OutType;
  using LayoutD = cutlass::layout::RowMajor;
  using LayoutD_Transpose = typename cutlass::layout::LayoutTranspose<LayoutD>::type;
  static constexpr int AlignmentD = 128 / cutlass::sizeof_bits<ElementD>::value;

  using ElementC = void; // TODO: support bias
  using LayoutC = LayoutD;
  using LayoutC_Transpose = LayoutD_Transpose;
  static constexpr int AlignmentC = AlignmentD;

  using ElementAccumulator = float;
  using ElementCompute = float;
  using ElementBlockScale = float; 

  using ScaleConfig = conditional_t<swap_ab,
      cutlass::detail::Sm120BlockwiseScaleConfig<
        ScaleGranularityM, ScaleGranularityN, ScaleGranularityK,
        cute::UMMA::Major::K, cute::UMMA::Major::MN>,
      cutlass::detail::Sm120BlockwiseScaleConfig<
        ScaleGranularityM, ScaleGranularityN, ScaleGranularityK,
        cute::UMMA::Major::MN, cute::UMMA::Major::K>>;

  // layout_SFA and layout_SFB cannot be swapped since they are deduced.
  using LayoutSFA = decltype(ScaleConfig::deduce_layoutSFA());
  using LayoutSFB = decltype(ScaleConfig::deduce_layoutSFB());

  using ArchTag = cutlass::arch::Sm120;
  using OperatorClass = cutlass::arch::OpClassTensorOp;

  static constexpr auto RoundStyle = cutlass::FloatRoundStyle::round_to_nearest;
  using ElementScalar = float;
  using DefaultOperation = cutlass::epilogue::fusion::LinearCombination<ElementD, ElementCompute, ElementC, ElementScalar, RoundStyle>;
```
**EN:** This template mirrors the SM100 blockwise path but switches to `Sm120BlockwiseScaleConfig` and SM120 architecture tags, while preserving optional operand swapping.
**CN:** 该模板与 SM100 的 blockwise 路径类似，但改用了 `Sm120BlockwiseScaleConfig` 和 SM120 架构标签，同时保留可选的操作数交换能力。

### Named tile configs / 命名 tile 配置
```cpp
// Tile configurations for different M ranges
template <typename OutType>
struct sm120_blockwise_fp8_config_default {
  // use 128x128x128 tile with Cooperative (Auto) schedule
  using KernelSchedule = cutlass::gemm::collective::KernelScheduleAuto;
  using EpilogueSchedule = cutlass::epilogue::collective::EpilogueScheduleAuto;
  using TileShape = Shape<_128, _128, _128>;
  using ClusterShape = Shape<_1, _1, _1>;
  // ScaleGranularity must match the actual quantization block size (1, 128, 128)
  using Gemm = cutlass_3x_gemm_fp8_blockwise<
      OutType, 1, 128, 128, TileShape, ClusterShape,
      EpilogueSchedule, KernelSchedule>;
};

template <typename OutType>
struct sm120_blockwise_fp8_config_pingpong {
  // use 64x128x128 tile with Pingpong schedule
  using KernelSchedule = cutlass::gemm::KernelTmaWarpSpecializedBlockwisePingpongSm120;
  using EpilogueSchedule = cutlass::epilogue::collective::EpilogueScheduleAuto;
  using TileShape = Shape<_64, _128, _128>;
  using ClusterShape = Shape<_1, _1, _1>;
  // ScaleGranularity stays (1, 128, 128) to match actual quantization data
  using Gemm = cutlass_3x_gemm_fp8_blockwise<
      OutType, 1, 128, 128, TileShape, ClusterShape,
      EpilogueSchedule, KernelSchedule>;
};

template <typename OutType>
struct sm120_blockwise_fp8_config_swapab {
  // use 128x32x128 tile with Cooperative schedule
  using KernelSchedule = cutlass::gemm::KernelTmaWarpSpecializedBlockwiseCooperativeSm120;
  using EpilogueSchedule = cutlass::epilogue::collective::EpilogueScheduleAuto;
  using TileShape = Shape<_128, _32, _128>;
  using ClusterShape = Shape<_1, _1, _1>;
  using Gemm = cutlass_3x_gemm_fp8_blockwise<
      OutType, 128, 1, 128, TileShape, ClusterShape,
      EpilogueSchedule, KernelSchedule, true>;
};
```
**EN:** The file exposes three named configurations: a default cooperative kernel, a ping-pong kernel for moderate M, and a swap-AB kernel tuned for very small M.
**CN:** 文件提供了三个命名配置：默认 cooperative 内核、用于中等 M 的 ping-pong 内核，以及针对很小 M 调优的 swap-AB 内核。

### Blockwise caller / Blockwise 调用器
```cpp
template <typename Gemm>
void cutlass_gemm_caller_blockwise(torch::stable::Tensor& out, torch::stable::Tensor const& a,
                                   torch::stable::Tensor const& b,
                                   torch::stable::Tensor const& a_scales,
                                   torch::stable::Tensor const& b_scales) {
  static constexpr bool swap_ab = Gemm::swap_ab;
  using GemmKernel = typename Gemm::GemmKernel;
  using StrideA = typename Gemm::GemmKernel::StrideA;
  using StrideB = typename Gemm::GemmKernel::StrideB;
  using StrideD = typename Gemm::GemmKernel::StrideD;
  using StrideC = typename Gemm::GemmKernel::StrideC;
  using LayoutSFA = typename Gemm::LayoutSFA;
  using LayoutSFB = typename Gemm::LayoutSFB;
  using ScaleConfig = typename Gemm::ScaleConfig;

  using ElementAB = typename Gemm::ElementAB;
  using ElementD = typename Gemm::ElementD;
  using ElementBlockScale = typename Gemm::ElementBlockScale;

  int32_t m = a.size(0), n = b.size(1), k = a.size(1);

  StrideA a_stride;
  StrideB b_stride;
  StrideC c_stride;
  a_stride =
      cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(m, k, 1));
  b_stride =
      cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(n, k, 1));
  c_stride =
      cutlass::make_cute_packed_stride(StrideC{}, swap_ab ? cute::make_shape(n, m, 1) : cute::make_shape(m, n, 1));

  LayoutSFA layout_SFA = swap_ab ?
      ScaleConfig::tile_atom_to_shape_SFA(make_shape(n, m, k, 1)) :
      ScaleConfig::tile_atom_to_shape_SFA(make_shape(m, n, k, 1));
  LayoutSFB layout_SFB = swap_ab ?
      ScaleConfig::tile_atom_to_shape_SFB(make_shape(n, m, k, 1)) :
      ScaleConfig::tile_atom_to_shape_SFB(make_shape(m, n, k, 1));

  auto a_ptr = static_cast<ElementAB const*>(a.data_ptr());
  auto b_ptr = static_cast<ElementAB const*>(b.data_ptr());
  auto a_scales_ptr = static_cast<ElementBlockScale const*>(a_scales.data_ptr());
  auto b_scales_ptr = static_cast<ElementBlockScale const*>(b_scales.data_ptr());

  typename GemmKernel::MainloopArguments mainloop_args{};
  mainloop_args.layout_SFA = layout_SFA;
  mainloop_args.layout_SFB = layout_SFB;
  if (swap_ab) {
    mainloop_args.ptr_A = b_ptr;
    mainloop_args.dA = b_stride;
    mainloop_args.ptr_B = a_ptr;
    mainloop_args.dB = a_stride;
    mainloop_args.ptr_SFA = b_scales_ptr;
    mainloop_args.ptr_SFB = a_scales_ptr;
  } else {
    mainloop_args.ptr_A = a_ptr;
    mainloop_args.dA = a_stride;
    mainloop_args.ptr_B = b_ptr;
    mainloop_args.dB = b_stride;
    mainloop_args.ptr_SFA = a_scales_ptr;
    mainloop_args.ptr_SFB = b_scales_ptr;
  }
  auto prob_shape = swap_ab ? cute::make_shape(n, m, k, 1) : cute::make_shape(m, n, k, 1);

  auto c_ptr = static_cast<ElementD*>(out.data_ptr());
  typename GemmKernel::EpilogueArguments epilogue_args{
      {}, c_ptr, c_stride, c_ptr, c_stride};
  c3x::cutlass_gemm_caller<GemmKernel>(a.device(), prob_shape, mainloop_args,
                                       epilogue_args);
```
**EN:** The caller computes packed strides, derives scale layouts from the chosen config, fills mainloop arguments, and conditionally swaps operands and scale tensors.
**CN:** 调用器会计算紧凑 stride、从所选配置推导 scale 布局、填充 mainloop 参数，并在需要时交换操作数与缩放张量。

### Runtime heuristic / 运行时启发式
```cpp
template <typename OutType>
void cutlass_gemm_blockwise_sm120_fp8_dispatch(torch::stable::Tensor& out,
                                               torch::stable::Tensor const& a,
                                               torch::stable::Tensor const& b,
                                               torch::stable::Tensor const& a_scales,
                                               torch::stable::Tensor const& b_scales) {
  int M = a.size(0);
  // more heuristic tuning can be done here by checking N/K dimensions as well
  bool swap_ab = (M <= 64) || (M % 4 != 0);

  if (!swap_ab) {
    if (M <= 256) {
      using Gemm = typename sm120_blockwise_fp8_config_pingpong<OutType>::Gemm;
      return cutlass_gemm_caller_blockwise<Gemm>(
          out, a, b, a_scales, b_scales);
    }
    // M > 256: use default 128x128x128 config with Cooperative (Auto) schedule
    using Gemm = typename sm120_blockwise_fp8_config_default<OutType>::Gemm;
    return cutlass_gemm_caller_blockwise<Gemm>(
        out, a, b, a_scales, b_scales);
  } else {
    // Swap A/B for small M to improve performance
    // Use TILE_N=32 as the minimum compatible tile size.
    using Gemm = typename sm120_blockwise_fp8_config_swapab<OutType>::Gemm;
    return cutlass_gemm_caller_blockwise<Gemm>(
        out, a, b, a_scales, b_scales);
  }
```
**EN:** The SM120 dispatcher mainly keys off `M`: small values use swap-AB, medium values use the ping-pong kernel, and large values keep the default 128x128x128 configuration.
**CN:** SM120 分发器主要依据 `M` 做决策：小 M 使用 swap-AB，中等 M 使用 ping-pong 内核，大 M 则保持默认的 128x128x128 配置。

## Key Concepts / 关键概念
- **Ping-pong scheduling / Ping-pong 调度**: SM120 introduces a ping-pong kernel schedule that is useful for smaller tiles. / SM120 引入了适用于较小 tile 的 ping-pong 内核调度方式。
- **Family-wide enablement / 家族级启用**: The kernel is enabled for the full SM120 family rather than a single chip revision. / 该内核针对整个 SM120 家族启用，而不是只支持单一芯片修订版。

## Dependencies / 依赖关系
- **cutlass_gemm_caller.cuh / cutlass_gemm_caller.cuh**: Relies on the shared launcher helper to allocate workspace and invoke the prepared CUTLASS kernel. / 依赖共享启动辅助函数来分配工作区并调用已准备好的 CUTLASS 内核。
- **CUTLASS / CUTLASS**: Uses CUTLASS collective builders, kernel adapters, and architecture tags to assemble GEMM kernels. / 使用 CUTLASS 的 collective builder、kernel adapter 与架构标签来组装 GEMM 内核。
- **SM120 family helpers / SM120 家族辅助类型**: Uses `enable_sm120_family` so the same definition can target SM120 and related variants. / 使用 `enable_sm120_family`，使同一套定义可覆盖 SM120 及相关变体。
