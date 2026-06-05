# scaled_mm_blockwise_sm100_fp8_dispatch.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_blockwise_sm100_fp8_dispatch.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines SM100 blockwise FP8 CUTLASS kernels and runtime heuristics for choosing tile shapes, swap mode, and epilogue style. / 定义 SM100 blockwise FP8 的 CUTLASS 内核，以及选择 tile 形状、交换模式和 epilogue 风格的运行时启发式逻辑。

## Line-by-Line Analysis / 逐行分析
### Scale-aware kernel template / 带缩放感知的内核模板
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
      cutlass::detail::Sm100BlockwiseScaleConfig<
        ScaleGranularityM, ScaleGranularityN, ScaleGranularityK,
        cute::UMMA::Major::K, cute::UMMA::Major::MN>,
      cutlass::detail::Sm100BlockwiseScaleConfig<
        ScaleGranularityM, ScaleGranularityN, ScaleGranularityK,
        cute::UMMA::Major::MN, cute::UMMA::Major::K>>;
```
**EN:** The template binds FP8 input types, output type, optional `swap_ab` behavior, and the `Sm100BlockwiseScaleConfig` that describes how blockwise scale tensors are interpreted.
**CN:** 该模板绑定 FP8 输入类型、输出类型、可选 `swap_ab` 行为，以及用于描述 blockwise scale 张量解释方式的 `Sm100BlockwiseScaleConfig`。

### Collective builders / Collective 构建器
```cpp
  using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
      ArchTag,
      OperatorClass,
      MmaTileShape,
      ClusterShape,
      cutlass::epilogue::collective::EpilogueTileAuto,
      ElementAccumulator,
      ElementCompute,
      ElementC,
      conditional_t<swap_ab, LayoutC_Transpose, LayoutC>,
      AlignmentC,
      ElementD,
      conditional_t<swap_ab, LayoutD_Transpose, LayoutD>,
      AlignmentD,
      EpilogueScheduler,
      DefaultOperation
  >::CollectiveOp;
 
  using StageCountType = cutlass::gemm::collective::StageCountAuto; 
  using CollectiveMainloop = conditional_t<swap_ab,
      typename cutlass::gemm::collective::CollectiveBuilder<
          ArchTag,
          OperatorClass,
          ElementB,
          cute::tuple<LayoutB_Transpose, LayoutSFA>,
          AlignmentB,
          ElementA,
          cute::tuple<LayoutA_Transpose, LayoutSFB>,
          AlignmentA,
          ElementAccumulator,
          MmaTileShape,
          ClusterShape,
          cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
          MainloopScheduler
      >::CollectiveOp,
      typename cutlass::gemm::collective::CollectiveBuilder<
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
      >::CollectiveOp>;
```
**EN:** These aliases assemble the CUTLASS epilogue and mainloop. When `swap_ab` is enabled, the code switches to transposed layouts so small-M cases can run more efficiently.
**CN:** 这些别名负责组装 CUTLASS 的 epilogue 与 mainloop。当启用 `swap_ab` 时，代码会切换到转置布局，以提升小 M 场景的效率。

### Runtime argument packing / 运行时参数打包
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
**EN:** The blockwise caller computes tensor strides, derives block-scale layouts, swaps pointers and scale tensors when needed, and then invokes the shared CUTLASS launcher.
**CN:** 该 blockwise 调用器会计算张量 stride、推导 block-scale 布局，并在需要时交换操作数指针与缩放张量，最后调用共享的 CUTLASS 启动器。

### Shape heuristics / 形状启发式逻辑
```cpp
  int32_t m = a.size(0), n = b.size(1), k = a.size(1), sms;
  cudaDeviceGetAttribute(&sms, cudaDevAttrMultiProcessorCount, a.get_device());

  constexpr int TILE_K = 128;
  // TODO: better heuristics
  bool swap_ab = (m < 16) || (m % 4 != 0);
  bool use_tma_epilogue = (m * n) % 4 == 0;
  if (!swap_ab) {
    constexpr int TILE_N = 128;
    int tile_m = 256;
    if (cuda_utils::ceil_div(n, TILE_N) * cuda_utils::ceil_div(m, 64) <= sms) {
      tile_m = 64;
    }
    else if (cuda_utils::ceil_div(n, TILE_N) * cuda_utils::ceil_div(m, 128) <= sms) {
// ...
    if (tile_m == 64) {
      if (use_tma_epilogue) {
        cutlass_gemm_caller_blockwise<cutlass_3x_gemm_fp8_blockwise<
            OutType, 1, TILE_N, TILE_K, Shape<_64, Int<TILE_N>, Int<TILE_K>>,
            Shape<_1, _1, _1>, cutlass::epilogue::TmaWarpSpecialized1Sm,
            cutlass::gemm::KernelTmaWarpSpecializedBlockwise1SmSm100>>(
            out, a, b, a_scales, b_scales);
      } else {
        cutlass_gemm_caller_blockwise<cutlass_3x_gemm_fp8_blockwise<
            OutType, 1, TILE_N, TILE_K, Shape<_64, Int<TILE_N>, Int<TILE_K>>,
            Shape<_1, _1, _1>, cutlass::epilogue::BlockwiseNoSmemWarpSpecialized1Sm,
            cutlass::gemm::KernelTmaWarpSpecializedBlockwise1SmSm100>>(
            out, a, b, a_scales, b_scales);
      }
    } else if (tile_m == 128) {
      if (use_tma_epilogue) {
        cutlass_gemm_caller_blockwise<cutlass_3x_gemm_fp8_blockwise<
            OutType, 1, TILE_N, TILE_K, Shape<_128, Int<TILE_N>, Int<TILE_K>>,
            Shape<_1, _1, _1>, cutlass::epilogue::TmaWarpSpecialized1Sm,
            cutlass::gemm::KernelTmaWarpSpecializedBlockwise1SmSm100>>(
            out, a, b, a_scales, b_scales);
      } else {
        cutlass_gemm_caller_blockwise<cutlass_3x_gemm_fp8_blockwise<
            OutType, 1, TILE_N, TILE_K, Shape<_128, Int<TILE_N>, Int<TILE_K>>,
            Shape<_1, _1, _1>, cutlass::epilogue::BlockwiseNoSmemWarpSpecialized1Sm,
            cutlass::gemm::KernelTmaWarpSpecializedBlockwise1SmSm100>>(
            out, a, b, a_scales, b_scales);
      }
    } else { // tile_m == 256
      if (use_tma_epilogue) {
          cutlass_gemm_caller_blockwise<cutlass_3x_gemm_fp8_blockwise<
              OutType, 1, TILE_N, TILE_K, Shape<_256, Int<TILE_N>, Int<TILE_K>>,
            Shape<_2, _1, _1>, cutlass::epilogue::TmaWarpSpecialized2Sm,
            cutlass::gemm::KernelTmaWarpSpecializedBlockwise2SmSm100>>(
            out, a, b, a_scales, b_scales);
      } else {
          cutlass_gemm_caller_blockwise<cutlass_3x_gemm_fp8_blockwise<
              OutType, 1, TILE_N, TILE_K, Shape<_256, Int<TILE_N>, Int<TILE_K>>,
            Shape<_2, _1, _1>, cutlass::epilogue::BlockwiseNoSmemWarpSpecialized2Sm,
            cutlass::gemm::KernelTmaWarpSpecializedBlockwise2SmSm100>>(
            out, a, b, a_scales, b_scales);
      }
    }
  } else {
    // TODO: Test more tile N configs
    constexpr int TILE_M = 128;
    constexpr int TILE_N = 16;
    // TMA epilogue isn't compatible with Swap A/B
    cutlass_gemm_caller_blockwise<cutlass_3x_gemm_fp8_blockwise<
        OutType, TILE_M, 1, TILE_K, Shape<Int<TILE_M>, Int<TILE_N>, Int<TILE_K>>,
        Shape<_1, _1, _1>, cutlass::epilogue::BlockwiseNoSmemWarpSpecialized1Sm,
        cutlass::gemm::KernelTmaWarpSpecializedBlockwise1SmSm100, true>>(
        out, a, b, a_scales, b_scales);
  }
```
**EN:** The dispatcher queries SM count, chooses among 64/128/256-sized M tiles, decides whether TMA epilogues are legal, and falls back to a swap-AB path for tiny or misaligned M values.
**CN:** 分发器会查询 SM 数量，在 64/128/256 的 M tile 之间做选择，判断是否可以使用 TMA epilogue，并在 M 很小或不对齐时回退到 swap-AB 路径。

## Key Concepts / 关键概念
- **Blockwise scale layouts / 分块缩放布局**: Scale tensors are treated as first-class kernel inputs with layouts derived from CUTLASS scale configs. / 缩放张量被当作一等输入，其布局由 CUTLASS 的 scale config 推导。
- **Swap-AB optimization / 交换 A/B 优化**: Small-M workloads may run faster when operands and layouts are logically swapped. / 对于小 M 工作负载，逻辑上交换操作数和布局往往更高效。

## Dependencies / 依赖关系
- **cutlass_gemm_caller.cuh / cutlass_gemm_caller.cuh**: Relies on the shared launcher helper to allocate workspace and invoke the prepared CUTLASS kernel. / 依赖共享启动辅助函数来分配工作区并调用已准备好的 CUTLASS 内核。
- **CUTLASS / CUTLASS**: Uses CUTLASS collective builders, kernel adapters, and architecture tags to assemble GEMM kernels. / 使用 CUTLASS 的 collective builder、kernel adapter 与架构标签来组装 GEMM 内核。
- **cuda_utils / cuda_utils**: Uses `ceil_div` and device SM-count queries to drive runtime kernel heuristics. / 使用 `ceil_div` 与设备 SM 数查询来驱动运行时内核启发式逻辑。
