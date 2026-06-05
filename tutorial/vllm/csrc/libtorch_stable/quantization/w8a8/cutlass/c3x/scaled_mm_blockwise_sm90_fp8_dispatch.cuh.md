# scaled_mm_blockwise_sm90_fp8_dispatch.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_blockwise_sm90_fp8_dispatch.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines the SM90 blockwise FP8 kernel and a simple fixed dispatch path for that architecture. / 定义 SM90 blockwise FP8 内核，并为该架构提供固定的分发路径。

## Line-by-Line Analysis / 逐行分析
### Blockwise scale template / 分块缩放模板
```cpp
template <class OutType, int ScaleGranularityM,
          int ScaleGranularityN, int ScaleGranularityK,
          class MmaTileShape, class ClusterShape,
          class EpilogueScheduler, class MainloopScheduler>
struct cutlass_3x_gemm_fp8_blockwise {
  using ElementAB = cutlass::float_e4m3_t;

  using ElementA = ElementAB;
  using LayoutA = cutlass::layout::RowMajor;
  static constexpr int AlignmentA = 128 / cutlass::sizeof_bits<ElementA>::value;

  using ElementB = ElementAB;
  using LayoutB = cutlass::layout::ColumnMajor;
  static constexpr int AlignmentB = 128 / cutlass::sizeof_bits<ElementB>::value;

  using ElementD = OutType;
  using LayoutD = cutlass::layout::RowMajor;
  static constexpr int AlignmentD = 128 / cutlass::sizeof_bits<ElementD>::value;

  using ElementC = void; // TODO: support bias
  using LayoutC = LayoutD;
  static constexpr int AlignmentC = AlignmentD;

  using ElementAccumulator = float;
  using ElementCompute = float;
  using ElementBlockScale = float;

  using ScaleConfig = cutlass::detail::Sm90BlockwiseScaleConfig<
        ScaleGranularityM, ScaleGranularityN, ScaleGranularityK,
        cute::GMMA::Major::MN, cute::GMMA::Major::K>;

  using LayoutSFA = decltype(ScaleConfig::deduce_layoutSFA());
  using LayoutSFB = decltype(ScaleConfig::deduce_layoutSFB());

  using ArchTag = cutlass::arch::Sm90;
  using OperatorClass = cutlass::arch::OpClassTensorOp;

  static constexpr auto RoundStyle = cutlass::FloatRoundStyle::round_to_nearest;
  using ElementScalar = float;
  using DefaultOperation = cutlass::epilogue::fusion::LinearCombination<ElementD, ElementCompute, ElementC, ElementScalar, RoundStyle>;
  using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
      ArchTag,
      OperatorClass,
      MmaTileShape,
      ClusterShape,
      cutlass::epilogue::collective::EpilogueTileAuto,
      ElementAccumulator,
      ElementCompute,
      ElementC,
      LayoutC,
      AlignmentC,
      ElementD,
      LayoutD,
      AlignmentD,
      EpilogueScheduler,
      DefaultOperation
  >::CollectiveOp;
```
**EN:** The template ties together FP8 inputs, floating-point block scales, SM90-specific scale layout deduction, and the default linear-combination epilogue.
**CN:** 该模板把 FP8 输入、浮点 block scale、SM90 专用的 scale 布局推导，以及默认线性组合 epilogue 组合在一起。

### Mainloop and kernel type / Mainloop 与内核类型
```cpp
  using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
      ArchTag,
      OperatorClass,
      ElementA,
      cute::tuple<LayoutA, LayoutSFA>,
      AlignmentA,
      ElementB,
      cute::tuple<LayoutB, LayoutSFB>,
      AlignmentB,
      ElementAccumulator,
      MmaTileShape,
      ClusterShape,
      cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
      MainloopScheduler
  >::CollectiveOp;

  using KernelType = enable_sm90_or_later<cutlass::gemm::kernel::GemmUniversal<
      Shape<int, int, int, int>, CollectiveMainloop, CollectiveEpilogue>>;

  struct GemmKernel : public KernelType {};
};
```
**EN:** This block instantiates the CUTLASS mainloop and wraps it inside a `GemmUniversal` kernel that is enabled for SM90 or newer GPUs.
**CN:** 这一段实例化 CUTLASS mainloop，并把它包装进一个面向 SM90 及更新 GPU 启用的 `GemmUniversal` 内核。

### Runtime argument builder / 运行时参数构建
```cpp
template <typename Gemm>
void cutlass_gemm_caller_blockwise(torch::stable::Tensor& out, torch::stable::Tensor const& a,
                                   torch::stable::Tensor const& b,
                                   torch::stable::Tensor const& a_scales,
                                   torch::stable::Tensor const& b_scales) {
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

  STD_TORCH_CHECK(m % 4 == 0, "m must be divisible by 4");

  StrideA a_stride;
  StrideB b_stride;
  StrideC c_stride;
  a_stride =
      cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(m, k, 1));
  b_stride =
      cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(n, k, 1));
  c_stride =
      cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(m, n, 1));

  LayoutSFA layout_SFA = 
      ScaleConfig::tile_atom_to_shape_SFA(make_shape(m, n, k, 1));
  LayoutSFB layout_SFB = 
      ScaleConfig::tile_atom_to_shape_SFB(make_shape(m, n, k, 1));

  auto a_ptr = static_cast<ElementAB const*>(a.data_ptr());
  auto b_ptr = static_cast<ElementAB const*>(b.data_ptr());
  auto a_scales_ptr = static_cast<ElementBlockScale const*>(a_scales.data_ptr());
  auto b_scales_ptr = static_cast<ElementBlockScale const*>(b_scales.data_ptr());

  typename GemmKernel::MainloopArguments mainloop_args{};
  mainloop_args.ptr_A = a_ptr;
  mainloop_args.dA = a_stride;
  mainloop_args.ptr_B = b_ptr;
  mainloop_args.dB = b_stride;
  mainloop_args.ptr_SFA = a_scales_ptr;
  mainloop_args.layout_SFA = layout_SFA;
  mainloop_args.ptr_SFB = b_scales_ptr;
  mainloop_args.layout_SFB = layout_SFB;
  auto prob_shape = cute::make_shape(m, n, k, 1);

  auto c_ptr = static_cast<ElementD*>(out.data_ptr());
  typename GemmKernel::EpilogueArguments epilogue_args{
      {}, c_ptr, c_stride, c_ptr, c_stride};
  c3x::cutlass_gemm_caller<GemmKernel>(a.device(), prob_shape, mainloop_args,
                                       epilogue_args);
```
**EN:** The caller validates `m % 4 == 0`, builds strides and scale layouts, fills the blockwise mainloop arguments, and launches the prepared kernel.
**CN:** 调用器会验证 `m % 4 == 0`，构造 stride 与 scale 布局，填充 blockwise mainloop 参数，并启动已准备好的内核。

### Fixed heuristic dispatch / 固定启发式分发
```cpp
template <typename OutType>
void cutlass_gemm_blockwise_sm90_fp8_dispatch(torch::stable::Tensor& out,
                                              torch::stable::Tensor const& a,
                                              torch::stable::Tensor const& b,
                                              torch::stable::Tensor const& a_scales,
                                              torch::stable::Tensor const& b_scales) {
  // TODO: better heuristics
  cutlass_gemm_caller_blockwise<cutlass_3x_gemm_fp8_blockwise<
      OutType, 1, 128, 128, Shape<_128, _128, _128>,
      Shape<_1, _2, _1>, cutlass::epilogue::TmaWarpSpecializedCooperative,
      cutlass::gemm::KernelTmaWarpSpecializedCooperativeFP8BlockScaledAccum>>(
      out, a, b, a_scales, b_scales);
```
**EN:** SM90 currently uses one hard-coded kernel configuration, with a TODO noting that future heuristics may choose more variants.
**CN:** SM90 当前只使用一个硬编码的内核配置，代码中的 TODO 表明未来可能增加更多启发式选择。

## Key Concepts / 关键概念
- **SM90 block scaling / SM90 分块缩放**: Uses CUTLASS `Sm90BlockwiseScaleConfig` to align scale tensors with block-scaled MMA instructions. / 使用 CUTLASS 的 `Sm90BlockwiseScaleConfig`，让缩放张量与 block-scaled MMA 指令对齐。

## Dependencies / 依赖关系
- **cutlass_gemm_caller.cuh / cutlass_gemm_caller.cuh**: Relies on the shared launcher helper to allocate workspace and invoke the prepared CUTLASS kernel. / 依赖共享启动辅助函数来分配工作区并调用已准备好的 CUTLASS 内核。
- **CUTLASS / CUTLASS**: Uses CUTLASS collective builders, kernel adapters, and architecture tags to assemble GEMM kernels. / 使用 CUTLASS 的 collective builder、kernel adapter 与架构标签来组装 GEMM 内核。
