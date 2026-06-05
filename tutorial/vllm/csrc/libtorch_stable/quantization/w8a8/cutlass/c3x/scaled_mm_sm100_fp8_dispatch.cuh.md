# scaled_mm_sm100_fp8_dispatch.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_sm100_fp8_dispatch.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines SM100 FP8 CUTLASS kernels, runtime configuration heuristics, and epilogue wrappers for scaled matrix multiplication. / 定义 SM100 FP8 的 CUTLASS 内核、运行时配置启发式和 scaled matmul 的 epilogue 包装器。

## Line-by-Line Analysis / 逐行分析
### Kernel wrapper type / 内核包装类型
```cpp
template <typename ElementAB_, typename ElementD_,
          template <typename, typename, typename> typename Epilogue_,
          typename TileShape, typename ClusterShape, typename KernelSchedule,
          typename EpilogueSchedule, bool swap_ab_ = false>
struct cutlass_3x_gemm_sm100_fp8 {
  using ElementAB = ElementAB_;
  using ElementC = ElementD_;
  using ElementD = ElementD_;
  using ElementAcc =
      typename std::conditional<std::is_same_v<ElementAB, int8_t>, int32_t,
                                float>::type;

  using Epilogue = Epilogue_<ElementAcc, ElementD, TileShape>;

  using EVTCompute = typename Epilogue::EVTCompute;

  static constexpr int AlignmentAB =
      128 / cutlass::sizeof_bits<ElementAB>::value;
  static constexpr int AlignmentCD =
      128 / cutlass::sizeof_bits<ElementD>::value;

  // Compile-time swap_ab flag
  static constexpr bool swap_ab = swap_ab_;
// ...
  using CollectiveEpilogue =
      typename cutlass::epilogue::collective::CollectiveBuilder<
          cutlass::arch::Sm100, cutlass::arch::OpClassTensorOp, TileShape,
          ClusterShape, cutlass::epilogue::collective::EpilogueTileAuto,
          ElementAcc, float, ElementC,
          conditional_t<swap_ab, LayoutC_Transpose, LayoutC>, AlignmentCD,
          ElementD, conditional_t<swap_ab, LayoutD_Transpose, LayoutD>,
          AlignmentCD, EpilogueSchedule, EVTCompute>::CollectiveOp;

  static constexpr size_t CEStorageSize =
      sizeof(typename CollectiveEpilogue::SharedStorage);

  using Stages = typename cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(CEStorageSize)>;

  // -----------------------------------------------------------
  // Collective mainloop (conditionally swap operands and layouts)
  // -----------------------------------------------------------
  using CollectiveMainloop = conditional_t<
      swap_ab,
      typename cutlass::gemm::collective::CollectiveBuilder<
          cutlass::arch::Sm100, cutlass::arch::OpClassTensorOp, ElementAB,
          LayoutB_T, AlignmentAB,             // Swapped B (as A)
          ElementAB, LayoutA_T, AlignmentAB,  // Swapped A (as B)
          ElementAcc, TileShape, ClusterShape, Stages,
          KernelSchedule>::CollectiveOp,
      typename cutlass::gemm::collective::CollectiveBuilder<
          cutlass::arch::Sm100, cutlass::arch::OpClassTensorOp, ElementAB,
          LayoutA, AlignmentAB, ElementAB, LayoutB, AlignmentAB, ElementAcc,
          TileShape, ClusterShape, Stages, KernelSchedule>::CollectiveOp>;

  // -----------------------------------------------------------
  // Kernel definition
  // -----------------------------------------------------------
  using GemmKernel = enable_sm100_to_sm120<cutlass::gemm::kernel::GemmUniversal<
      Shape<int, int, int, int>, CollectiveMainloop, CollectiveEpilogue, void>>;
```
**EN:** This template wraps the generic SM100 GEMM definition with an optional compile-time `swap_ab` flag and uses CUTLASS collective builders to adapt layouts for swapped operands.
**CN:** 该模板用可选的编译期 `swap_ab` 标志包装通用 SM100 GEMM 定义，并使用 CUTLASS collective builder 为交换操作数调整布局。

### Configuration families / 配置族
```cpp
template <typename InType, typename OutType, bool EnableBias>
struct sm100_fp8_config_default {
  // M in (256, inf)
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule = cutlass::gemm::collective::KernelScheduleAuto;
  using EpilogueSchedule = cutlass::epilogue::collective::EpilogueScheduleAuto;
  using TileShape = Shape<_256, _128, _128>;
  using ClusterShape = Shape<_2, _2, _1>;
  using Cutlass3xGemm =
      conditional_t<EnableBias,
                    cutlass_3x_gemm_sm100_fp8<
                        InType, OutType, c3x::ScaledEpilogueBias, TileShape,
                        ClusterShape, KernelSchedule, EpilogueSchedule>,
                    cutlass_3x_gemm_sm100_fp8<
                        InType, OutType, c3x::ScaledEpilogue, TileShape,
                        ClusterShape, KernelSchedule, EpilogueSchedule>>;
};

template <typename InType, typename OutType, bool EnableBias>
struct sm100_fp8_config_M256 {
  // M in (64, 256]
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule = cutlass::gemm::collective::KernelScheduleAuto;
  using EpilogueSchedule = cutlass::epilogue::collective::EpilogueScheduleAuto;
  using TileShape = Shape<_128, _128, _128>;
  using ClusterShape = Shape<_2, _1, _1>;
  using Cutlass3xGemm =
      conditional_t<EnableBias,
                    cutlass_3x_gemm_sm100_fp8<
                        InType, OutType, c3x::ScaledEpilogueBias, TileShape,
                        ClusterShape, KernelSchedule, EpilogueSchedule>,
                    cutlass_3x_gemm_sm100_fp8<
                        InType, OutType, c3x::ScaledEpilogue, TileShape,
                        ClusterShape, KernelSchedule, EpilogueSchedule>>;
};
// ...
template <typename InType, typename OutType, bool EnableBias>
struct sm100_fp8_config_M64_swap_ab {
  // This config is for M in (16, 64] and K >= 4096
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule = cutlass::gemm::collective::KernelScheduleAuto;
  using EpilogueSchedule = cutlass::epilogue::collective::EpilogueScheduleAuto;
  using TileShape = Shape<_128, _64, _256>;
  using ClusterShape = Shape<_4, _1, _1>;

  // Use ScaledEpilogueColumnBias instead of ScaledEpilogueBias when doing swap
  // AB
  using Cutlass3xGemm = conditional_t<
      EnableBias,
      cutlass_3x_gemm_sm100_fp8<InType, OutType, c3x::ScaledEpilogueColumnBias,
                                TileShape, ClusterShape, KernelSchedule,
                                EpilogueSchedule, true>,
      cutlass_3x_gemm_sm100_fp8<InType, OutType, c3x::ScaledEpilogue, TileShape,
                                ClusterShape, KernelSchedule, EpilogueSchedule,
                                true>>;
};

template <typename InType, typename OutType, bool EnableBias>
struct sm100_fp8_config_M64 {
  // This config is for M = 64 and K < 4096 (do not enable swap AB in such case)
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule = cutlass::gemm::collective::KernelScheduleAuto;
  using EpilogueSchedule = cutlass::epilogue::collective::EpilogueScheduleAuto;
  using TileShape = Shape<_64, _64, _128>;
  using ClusterShape = Shape<_1, _1, _1>;

  using Cutlass3xGemm =
      conditional_t<EnableBias,
                    cutlass_3x_gemm_sm100_fp8<
                        InType, OutType, c3x::ScaledEpilogueBias, TileShape,
                        ClusterShape, KernelSchedule, EpilogueSchedule>,
                    cutlass_3x_gemm_sm100_fp8<
                        InType, OutType, c3x::ScaledEpilogue, TileShape,
                        ClusterShape, KernelSchedule, EpilogueSchedule>>;
};

template <typename InType, typename OutType, bool EnableBias>
struct sm100_fp8_config_M16_swap_ab {
  // M in [1, 16]
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule = cutlass::gemm::collective::KernelScheduleAuto;
  using EpilogueSchedule = cutlass::epilogue::collective::EpilogueScheduleAuto;
  using TileShape = Shape<_128, _32, _128>;
  using ClusterShape = Shape<_4, _1, _1>;

  // Use ScaledEpilogueColumnBias instead of ScaledEpilogueBias when doing swap
  // AB
  using Cutlass3xGemm = conditional_t<
      EnableBias,
      cutlass_3x_gemm_sm100_fp8<InType, OutType, c3x::ScaledEpilogueColumnBias,
                                TileShape, ClusterShape, KernelSchedule,
                                EpilogueSchedule, true>,
      cutlass_3x_gemm_sm100_fp8<InType, OutType, c3x::ScaledEpilogue, TileShape,
                                ClusterShape, KernelSchedule, EpilogueSchedule,
                                true>>;
};
```
**EN:** The file defines multiple shape-specialized configs: large-M defaults, a mid-sized 128x128 path, and small-M variants that may swap operands and use column-bias epilogues.
**CN:** 该文件定义了多组按形状特化的配置：大 M 默认路径、中等尺寸的 128x128 路径，以及可能交换操作数并使用 column-bias epilogue 的小 M 变体。

### Launch adapter / 启动适配器
```cpp
template <typename Gemm, typename... EpilogueArgs>
void cutlass_gemm_caller_sm100_fp8(torch::stable::Tensor& out,
                                   torch::stable::Tensor const& a,
                                   torch::stable::Tensor const& b,
                                   EpilogueArgs&&... epilogue_params) {
  static constexpr bool swap_ab = Gemm::swap_ab;
  using ElementAB = typename Gemm::ElementAB;
  using ElementD = typename Gemm::ElementD;
  using GemmKernel = typename Gemm::GemmKernel;

  using StrideA = typename Gemm::GemmKernel::StrideA;
  using StrideB = typename Gemm::GemmKernel::StrideB;
  using StrideC = typename Gemm::GemmKernel::StrideC;

  int32_t m = a.size(0), n = b.size(1), k = a.size(1);
  auto prob_shape =
      swap_ab ? cute::make_shape(n, m, k, 1) : cute::make_shape(m, n, k, 1);

  StrideA a_stride =
      cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(m, k, 1));
  StrideB b_stride =
      cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(n, k, 1));
  StrideC c_stride = cutlass::make_cute_packed_stride(
      StrideC{},
      swap_ab ? cute::make_shape(n, m, 1) : cute::make_shape(m, n, 1));

  auto a_ptr = static_cast<ElementAB*>(a.data_ptr());
  auto b_ptr = static_cast<ElementAB*>(b.data_ptr());
  auto c_ptr = static_cast<ElementD*>(out.data_ptr());

  typename GemmKernel::MainloopArguments mainloop_args =
      swap_ab ? typename GemmKernel::MainloopArguments{b_ptr, b_stride, a_ptr,
                                                       a_stride}
              : typename GemmKernel::MainloopArguments{a_ptr, a_stride, b_ptr,
                                                       b_stride};

  typename GemmKernel::EpilogueArguments epilogue_args{
      Gemm::Epilogue::prepare_args(
          std::forward<EpilogueArgs>(epilogue_params)...),
      c_ptr, c_stride, c_ptr, c_stride};

  c3x::cutlass_gemm_caller<GemmKernel>(a.device(), prob_shape, mainloop_args,
                                       epilogue_args);
}
```
**EN:** The SM100 caller computes problem shapes and strides, optionally swaps A and B, prepares epilogue arguments through the GEMM type, and invokes the shared launcher.
**CN:** SM100 调用适配器会计算问题形状与 stride，可选地交换 A/B，通过 GEMM 类型准备 epilogue 参数，并调用共享启动器。

### Shape-driven dispatch / 形状驱动分发
```cpp
template <typename InType, typename OutType, bool EnableBias,
          typename... EpilogueArgs>
inline void cutlass_gemm_sm100_fp8_dispatch(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales, EpilogueArgs&&... args) {
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  STD_TORCH_CHECK(a.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
  STD_TORCH_CHECK(b.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);

  using Cutlass3xGemmDefault =
      typename sm100_fp8_config_default<InType, OutType,
                                        EnableBias>::Cutlass3xGemm;
  using Cutlass3xGemmM16SwapAB =
      typename sm100_fp8_config_M16_swap_ab<InType, OutType,
                                            EnableBias>::Cutlass3xGemm;
  using Cutlass3xGemmM64SwapAB =
      typename sm100_fp8_config_M64_swap_ab<InType, OutType,
                                            EnableBias>::Cutlass3xGemm;
  using Cutlass3xGemmM64 =
      typename sm100_fp8_config_M64<InType, OutType, EnableBias>::Cutlass3xGemm;

  using Cutlass3xGemmM256 =
      typename sm100_fp8_config_M256<InType, OutType,
                                     EnableBias>::Cutlass3xGemm;

  uint32_t const m = a.size(0);
  uint32_t const k = a.size(1);

  if (m <= 16) {
    // m in [1, 16]
    return cutlass_gemm_caller_sm100_fp8<Cutlass3xGemmM16SwapAB>(
        out, a, b, b_scales, a_scales, std::forward<EpilogueArgs>(args)...);
  } else if (m <= 64) {
    // m in (16, 64]
    if (m == 64 && k < 4096) {
      // do not enable swap AB
      return cutlass_gemm_caller_sm100_fp8<Cutlass3xGemmM64>(
          out, a, b, a_scales, b_scales, std::forward<EpilogueArgs>(args)...);
    }
    return cutlass_gemm_caller_sm100_fp8<Cutlass3xGemmM64SwapAB>(
        out, a, b, b_scales, a_scales, std::forward<EpilogueArgs>(args)...);

  } else if (m <= 256) {
    // m in (64, 256]
    return cutlass_gemm_caller_sm100_fp8<Cutlass3xGemmM256>(
        out, a, b, a_scales, b_scales, std::forward<EpilogueArgs>(args)...);
  } else {
    // m in (256, inf)
    return cutlass_gemm_caller_sm100_fp8<Cutlass3xGemmDefault>(
        out, a, b, a_scales, b_scales, std::forward<EpilogueArgs>(args)...);
  }
}
```
**EN:** Runtime selection depends mainly on `m` and partly on `k`. Tiny and narrow cases use swap-AB kernels, while larger problems move through M64, M256, or the default configuration.
**CN:** 运行时选择主要依赖 `m`，并部分参考 `k`。极小或较窄的情况会使用 swap-AB 内核，而更大的问题则依次进入 M64、M256 或默认配置。

### Batch-invariant and epilogue wrappers / 批不变与 epilogue 包装器
```cpp
template <typename InType, typename OutType, bool EnableBias,
          typename... EpilogueArgs>
inline void cutlass_gemm_sm100_fp8_batch_invariant_dispatch(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales, EpilogueArgs&&... args) {
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  STD_TORCH_CHECK(a.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
  STD_TORCH_CHECK(b.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);

  using Cutlass3xGemmM64SwapAB =
      typename sm100_fp8_config_M64_swap_ab<InType, OutType,
                                            EnableBias>::Cutlass3xGemm;
  using Cutlass3xGemmM64 =
      typename sm100_fp8_config_M64<InType, OutType, EnableBias>::Cutlass3xGemm;

  // keep the CUTLASS config independent of M for batch invariance
  uint32_t const k = a.size(1);
  if (k < 4096) {
    return cutlass_gemm_caller_sm100_fp8<Cutlass3xGemmM64>(
        out, a, b, a_scales, b_scales, std::forward<EpilogueArgs>(args)...);
  }
  return cutlass_gemm_caller_sm100_fp8<Cutlass3xGemmM64SwapAB>(
      out, a, b, b_scales, a_scales, std::forward<EpilogueArgs>(args)...);
}
// ...
template <bool EnableBias, typename... EpilogueArgs>
void cutlass_scaled_mm_sm100_fp8_epilogue(torch::stable::Tensor& out,
                                          torch::stable::Tensor const& a,
                                          torch::stable::Tensor const& b,
                                          torch::stable::Tensor const& a_scales,
                                          torch::stable::Tensor const& b_scales,
                                          EpilogueArgs&&... epilogue_args) {
  STD_TORCH_CHECK(a.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
  STD_TORCH_CHECK(b.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);

  if (out.scalar_type() == torch::headeronly::ScalarType::BFloat16) {
    return cutlass_gemm_sm100_fp8_dispatch<cutlass::float_e4m3_t,
                                           cutlass::bfloat16_t, EnableBias>(
        out, a, b, a_scales, b_scales,
        std::forward<EpilogueArgs>(epilogue_args)...);
  } else {
    STD_TORCH_CHECK(out.scalar_type() == torch::headeronly::ScalarType::Half);
    return cutlass_gemm_sm100_fp8_dispatch<cutlass::float_e4m3_t,
                                           cutlass::half_t, EnableBias>(
        out, a, b, a_scales, b_scales,
        std::forward<EpilogueArgs>(epilogue_args)...);
  }
}

template <bool EnableBias, typename... EpilogueArgs>
void cutlass_scaled_mm_sm100_fp8_batch_invariant_epilogue(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales, EpilogueArgs&&... epilogue_args) {
  STD_TORCH_CHECK(a.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
  STD_TORCH_CHECK(b.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);

  if (out.scalar_type() == torch::headeronly::ScalarType::BFloat16) {
    return cutlass_gemm_sm100_fp8_batch_invariant_dispatch<
        cutlass::float_e4m3_t, cutlass::bfloat16_t, EnableBias>(
        out, a, b, a_scales, b_scales,
        std::forward<EpilogueArgs>(epilogue_args)...);
  } else {
    STD_TORCH_CHECK(out.scalar_type() == torch::headeronly::ScalarType::Half);
    return cutlass_gemm_sm100_fp8_batch_invariant_dispatch<
        cutlass::float_e4m3_t, cutlass::half_t, EnableBias>(
        out, a, b, a_scales, b_scales,
        std::forward<EpilogueArgs>(epilogue_args)...);
  }
}
```
**EN:** The final helpers keep batch-invariant execution on a fixed kernel family and map runtime output dtype to either BF16 or FP16 instantiations.
**CN:** 最后这些辅助函数让 batch-invariant 执行固定在同一内核族上，并把运行时输出数据类型映射到 BF16 或 FP16 实例化。

## Key Concepts / 关键概念
- **Shape-specialized kernels / 按形状特化的内核**: Kernel families are explicitly tuned for different M ranges instead of relying on one universal configuration. / 内核族按不同 M 区间显式调优，而不是依赖单一通用配置。
- **Column-bias epilogues / 列偏置后处理**: Swap-AB kernels need a column-oriented bias epilogue because logical output axes change after swapping operands. / swap-AB 内核需要列方向 bias epilogue，因为交换操作数后逻辑输出轴发生了变化。

## Dependencies / 依赖关系
- **cutlass_gemm_caller.cuh / cutlass_gemm_caller.cuh**: Relies on the shared launcher helper to allocate workspace and invoke the prepared CUTLASS kernel. / 依赖共享启动辅助函数来分配工作区并调用已准备好的 CUTLASS 内核。
- **scaled_mm epilogues / scaled_mm epilogues**: Connects kernel dispatch to epilogue templates that apply scaling, optional bias, and related post-processing. / 将内核分发逻辑连接到执行缩放、可选 bias 与相关后处理的 epilogue 模板。
- **CUTLASS / CUTLASS**: Uses CUTLASS collective builders, kernel adapters, and architecture tags to assemble GEMM kernels. / 使用 CUTLASS 的 collective builder、kernel adapter 与架构标签来组装 GEMM 内核。
