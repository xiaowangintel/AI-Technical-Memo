# scaled_mm_sm90_fp8_dispatch.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_sm90_fp8_dispatch.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines the main SM90 FP8 CUTLASS kernel families and runtime heuristics for ordinary, swapped, and batch-invariant scaled GEMM. / 定义主要的 SM90 FP8 CUTLASS 内核家族，以及普通、交换式和 batch-invariant scaled GEMM 的运行时启发式。

## Line-by-Line Analysis / 逐行分析
### Kernel wrapper / 内核包装器
```cpp
template <typename ElementAB_, typename ElementD_,
          template <typename, typename, typename> typename Epilogue_,
          typename TileShape, typename ClusterShape, typename KernelSchedule,
          typename EpilogueSchedule, bool swap_ab_ = false>
struct cutlass_3x_gemm_sm90_fp8 {
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
          cutlass::arch::Sm90, cutlass::arch::OpClassTensorOp, TileShape,
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
          cutlass::arch::Sm90, cutlass::arch::OpClassTensorOp, ElementAB,
          LayoutB_T, AlignmentAB,             // Swapped B (as A)
          ElementAB, LayoutA_T, AlignmentAB,  // Swapped A (as B)
          ElementAcc, TileShape, ClusterShape, Stages,
          KernelSchedule>::CollectiveOp,
      typename cutlass::gemm::collective::CollectiveBuilder<
          cutlass::arch::Sm90, cutlass::arch::OpClassTensorOp, ElementAB,
          LayoutA, AlignmentAB, ElementAB, LayoutB, AlignmentAB, ElementAcc,
          TileShape, ClusterShape, Stages, KernelSchedule>::CollectiveOp>;

  // -----------------------------------------------------------
  // Kernel definition
  // -----------------------------------------------------------
  using KernelType = enable_sm90_or_later<cutlass::gemm::kernel::GemmUniversal<
      cute::Shape<int, int, int, int>, CollectiveMainloop, CollectiveEpilogue,
      cutlass::gemm::PersistentScheduler>>;

  struct GemmKernel : public KernelType {};
```
**EN:** This template builds an SM90 GEMM kernel with optional operand swapping and reuses CUTLASS collective builders to adapt both the mainloop and epilogue layouts.
**CN:** 该模板构造了一个支持可选操作数交换的 SM90 GEMM 内核，并复用 CUTLASS collective builder 来适配 mainloop 与 epilogue 的布局。

### Default and mid-size configs / 默认与中等尺寸配置
```cpp
template <typename InType, typename OutType, bool EnableBias>
struct sm90_fp8_config_default {
  // M in (128, inf)
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule =
      cutlass::gemm::KernelTmaWarpSpecializedPingpongFP8FastAccum;
  using EpilogueSchedule = typename cutlass::epilogue::TmaWarpSpecialized;
  using TileShape = Shape<_128, _128, _128>;
  using ClusterShape = Shape<_2, _1, _1>;

  using Cutlass3xGemm = conditional_t<
      EnableBias,
      cutlass_3x_gemm_sm90_fp8<InType, OutType, c3x::ScaledEpilogueBias,
                               TileShape, ClusterShape, KernelSchedule,
                               EpilogueSchedule>,
      cutlass_3x_gemm_sm90_fp8<InType, OutType, c3x::ScaledEpilogue, TileShape,
                               ClusterShape, KernelSchedule, EpilogueSchedule>>;
};

template <typename InType, typename OutType, bool EnableBias>
struct sm90_fp8_config_M8192_K6144 {
  // M >= 8192, K >= 6144
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule =
      cutlass::gemm::KernelTmaWarpSpecializedCooperativeFP8FastAccum;
  using EpilogueSchedule =
      typename cutlass::epilogue::TmaWarpSpecializedCooperative;
  using TileShape = Shape<_256, _128, _128>;
  using ClusterShape = Shape<_2, _1, _1>;

  using Cutlass3xGemm = conditional_t<
      EnableBias,
      cutlass_3x_gemm_sm90_fp8<InType, OutType, c3x::ScaledEpilogueBias,
                               TileShape, ClusterShape, KernelSchedule,
                               EpilogueSchedule>,
      cutlass_3x_gemm_sm90_fp8<InType, OutType, c3x::ScaledEpilogue, TileShape,
                               ClusterShape, KernelSchedule, EpilogueSchedule>>;
};

template <typename InType, typename OutType, bool EnableBias>
struct sm90_fp8_config_M128 {
  // M in (64, 128]
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule =
      cutlass::gemm::KernelTmaWarpSpecializedPingpongFP8FastAccum;
  using EpilogueSchedule = typename cutlass::epilogue::TmaWarpSpecialized;
  using TileShape = Shape<_64, _128, _128>;
  using ClusterShape = Shape<_2, _1, _1>;
  using Cutlass3xGemm = conditional_t<
      EnableBias,
      cutlass_3x_gemm_sm90_fp8<InType, OutType, c3x::ScaledEpilogueBias,
                               TileShape, ClusterShape, KernelSchedule,
                               EpilogueSchedule>,
      cutlass_3x_gemm_sm90_fp8<InType, OutType, c3x::ScaledEpilogue, TileShape,
                               ClusterShape, KernelSchedule, EpilogueSchedule>>;
};
```
**EN:** The first config group covers large generic problems, a specialized large-M cooperative path, and a narrower M128 configuration.
**CN:** 第一组配置覆盖通用的大问题规模、针对超大 M 的 cooperative 路径，以及较窄的 M128 配置。

### Small-M swapped configs / 小 M 交换式配置
```cpp
template <typename InType, typename OutType, bool EnableBias>
struct sm90_fp8_config_M64_N1280 {
  // M in (16, 64], N in [1 1280]
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule = cutlass::gemm::KernelTmaWarpSpecializedFP8FastAccum;
  using EpilogueSchedule = typename cutlass::epilogue::TmaWarpSpecialized;
  using TileShape = Shape<_64, _16, _256>;
  using ClusterShape = Shape<_1, _4, _1>;

  // enable swap AB for M < 64
  using Cutlass3xGemm = conditional_t<
      EnableBias,
      cutlass_3x_gemm_sm90_fp8<InType, OutType, c3x::ScaledEpilogueColumnBias,
                               TileShape, ClusterShape, KernelSchedule,
                               EpilogueSchedule, true>,
      cutlass_3x_gemm_sm90_fp8<InType, OutType, c3x::ScaledEpilogue, TileShape,
                               ClusterShape, KernelSchedule, EpilogueSchedule,
                               true>>;
};

template <typename InType, typename OutType, bool EnableBias>
struct sm90_fp8_config_M64_N8192 {
  // M in (16, 64], N > 1280
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule = cutlass::gemm::KernelTmaWarpSpecializedFP8FastAccum;
  using EpilogueSchedule = typename cutlass::epilogue::TmaWarpSpecialized;
  using TileShape = Shape<_64, _64, _256>;
  using ClusterShape = Shape<_1, _1, _1>;

  // enable swap AB for M < 64
  using Cutlass3xGemm = conditional_t<
      EnableBias,
      cutlass_3x_gemm_sm90_fp8<InType, OutType, c3x::ScaledEpilogueColumnBias,
                               TileShape, ClusterShape, KernelSchedule,
                               EpilogueSchedule, true>,
      cutlass_3x_gemm_sm90_fp8<InType, OutType, c3x::ScaledEpilogue, TileShape,
                               ClusterShape, KernelSchedule, EpilogueSchedule,
                               true>>;
};

template <typename InType, typename OutType, bool EnableBias>
struct sm90_fp8_config_M16_N1280 {
  // M in [1, 16], N in [1, 1280]
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule = cutlass::gemm::KernelTmaWarpSpecializedFP8FastAccum;
  using EpilogueSchedule = typename cutlass::epilogue::TmaWarpSpecialized;
  using TileShape = Shape<_64, _16, _256>;
  using ClusterShape = Shape<_1, _2, _1>;

  // enable swap AB for M < 64
  using Cutlass3xGemm = conditional_t<
      EnableBias,
      cutlass_3x_gemm_sm90_fp8<InType, OutType, c3x::ScaledEpilogueColumnBias,
                               TileShape, ClusterShape, KernelSchedule,
                               EpilogueSchedule, true>,
      cutlass_3x_gemm_sm90_fp8<InType, OutType, c3x::ScaledEpilogue, TileShape,
                               ClusterShape, KernelSchedule, EpilogueSchedule,
                               true>>;
};

template <typename InType, typename OutType, bool EnableBias>
struct sm90_fp8_config_M16_N8192 {
  // M in [1, 16], N > 1280
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule = cutlass::gemm::KernelTmaWarpSpecializedFP8FastAccum;
  using EpilogueSchedule = typename cutlass::epilogue::TmaWarpSpecialized;
  using TileShape = Shape<_64, _16, _256>;
  using ClusterShape = Shape<_1, _1, _1>;

  // enable swap AB for M < 64
  using Cutlass3xGemm = conditional_t<
      EnableBias,
      cutlass_3x_gemm_sm90_fp8<InType, OutType, c3x::ScaledEpilogueColumnBias,
                               TileShape, ClusterShape, KernelSchedule,
                               EpilogueSchedule, true>,
      cutlass_3x_gemm_sm90_fp8<InType, OutType, c3x::ScaledEpilogue, TileShape,
                               ClusterShape, KernelSchedule, EpilogueSchedule,
                               true>>;
};
```
**EN:** For `M <= 64`, the file defines several swapped-operand kernels split by `N` range, each using column-bias epilogues when bias is enabled.
**CN:** 对于 `M <= 64` 的情况，文件定义了多组按 `N` 范围切分的交换操作数内核，并在启用 bias 时使用 column-bias epilogue。

### Launch adapter / 启动适配器
```cpp
template <typename Gemm, typename... EpilogueArgs>
void cutlass_gemm_caller_sm90_fp8(torch::stable::Tensor& out,
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
```
**EN:** The caller computes the problem shape, builds packed strides, swaps raw pointers when necessary, and packages epilogue arguments before launching CUTLASS.
**CN:** 该调用适配器会计算问题形状、构造紧凑 stride、在必要时交换原始指针，并在启动 CUTLASS 之前打包 epilogue 参数。

### Runtime heuristic dispatch / 运行时启发式分发
```cpp
template <typename InType, typename OutType, bool EnableBias,
          typename... EpilogueArgs>
inline void cutlass_gemm_sm90_fp8_dispatch(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales, EpilogueArgs&&... args) {
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  STD_TORCH_CHECK(a.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
  STD_TORCH_CHECK(b.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);

  using Cutlass3xGemmDefault =
      typename sm90_fp8_config_default<InType, OutType,
                                       EnableBias>::Cutlass3xGemm;
  using Cutlass3xGemmM8192_K6144 =
      typename sm90_fp8_config_M8192_K6144<InType, OutType,
                                           EnableBias>::Cutlass3xGemm;
  using Cutlass3xGemmM128 =
      typename sm90_fp8_config_M128<InType, OutType, EnableBias>::Cutlass3xGemm;

  using Cutlass3xGemmM64_N1280 =
      typename sm90_fp8_config_M64_N1280<InType, OutType,
                                         EnableBias>::Cutlass3xGemm;
  using Cutlass3xGemmM64_N8192 =
      typename sm90_fp8_config_M64_N8192<InType, OutType,
                                         EnableBias>::Cutlass3xGemm;
  using Cutlass3xGemmM16_N1280 =
      typename sm90_fp8_config_M16_N1280<InType, OutType,
                                         EnableBias>::Cutlass3xGemm;
  using Cutlass3xGemmM16_N8192 =
      typename sm90_fp8_config_M16_N8192<InType, OutType,
                                         EnableBias>::Cutlass3xGemm;

  uint32_t const m = a.size(0);
  uint32_t const n = b.size(1);
  uint32_t const k = a.size(1);

  if (m <= 16) {
    // m in [1, 16]
    if (n <= 1280) {
      return cutlass_gemm_caller_sm90_fp8<Cutlass3xGemmM16_N1280>(
          out, a, b, b_scales, a_scales, std::forward<EpilogueArgs>(args)...);
    }
    return cutlass_gemm_caller_sm90_fp8<Cutlass3xGemmM16_N8192>(
        out, a, b, b_scales, a_scales, std::forward<EpilogueArgs>(args)...);
  } else if (m <= 64) {
    // m in (16, 64]
    if (n <= 1280) {
      return cutlass_gemm_caller_sm90_fp8<Cutlass3xGemmM64_N1280>(
          out, a, b, b_scales, a_scales, std::forward<EpilogueArgs>(args)...);
    }
    return cutlass_gemm_caller_sm90_fp8<Cutlass3xGemmM64_N8192>(
        out, a, b, b_scales, a_scales, std::forward<EpilogueArgs>(args)...);
  } else if (m <= 128) {
    // m in (64, 128]
    return cutlass_gemm_caller_sm90_fp8<Cutlass3xGemmM128>(
        out, a, b, a_scales, b_scales, std::forward<EpilogueArgs>(args)...);
  } else if (m >= 8192 && k >= 6144) {
    return cutlass_gemm_caller_sm90_fp8<Cutlass3xGemmM8192_K6144>(
        out, a, b, a_scales, b_scales, std::forward<EpilogueArgs>(args)...);
  } else {
    // m in (128, inf)
    return cutlass_gemm_caller_sm90_fp8<Cutlass3xGemmDefault>(
        out, a, b, a_scales, b_scales, std::forward<EpilogueArgs>(args)...);
  }
```
**EN:** Dispatch depends on `m`, `n`, and sometimes `k`: tiny matrices choose small swapped kernels, medium matrices choose narrower tiles, and very large matrices use a dedicated cooperative configuration.
**CN:** 分发逻辑依赖 `m`、`n`，有时也参考 `k`：极小矩阵选择小型交换内核，中等矩阵选择更窄的 tile，非常大的矩阵则使用专门的 cooperative 配置。

### Batch-invariant and epilogue wrappers / 批不变与 epilogue 包装器
```cpp
template <typename InType, typename OutType, bool EnableBias,
          typename... EpilogueArgs>
inline void cutlass_gemm_sm90_fp8_batch_invariant_dispatch(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales, EpilogueArgs&&... args) {
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  STD_TORCH_CHECK(a.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
  STD_TORCH_CHECK(b.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);

  using Cutlass3xGemmM64_N1280 =
      typename sm90_fp8_config_M64_N1280<InType, OutType,
                                         EnableBias>::Cutlass3xGemm;
  using Cutlass3xGemmM64_N8192 =
      typename sm90_fp8_config_M64_N8192<InType, OutType,
                                         EnableBias>::Cutlass3xGemm;

  // keep the CUTLASS config independent of M for batch invariance
  uint32_t const n = b.size(1);
  if (n <= 1280) {
    return cutlass_gemm_caller_sm90_fp8<Cutlass3xGemmM64_N1280>(
        out, a, b, b_scales, a_scales, std::forward<EpilogueArgs>(args)...);
  }
  return cutlass_gemm_caller_sm90_fp8<Cutlass3xGemmM64_N8192>(
      out, a, b, b_scales, a_scales, std::forward<EpilogueArgs>(args)...);
}
// ...
template <bool EnableBias, typename... EpilogueArgs>
void cutlass_scaled_mm_sm90_fp8_epilogue(torch::stable::Tensor& out,
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
    return cutlass_gemm_sm90_fp8_dispatch<cutlass::float_e4m3_t,
                                          cutlass::bfloat16_t, EnableBias>(
        out, a, b, a_scales, b_scales,
        std::forward<EpilogueArgs>(epilogue_args)...);
  } else {
    STD_TORCH_CHECK(out.scalar_type() == torch::headeronly::ScalarType::Half);
    return cutlass_gemm_sm90_fp8_dispatch<cutlass::float_e4m3_t,
                                          cutlass::half_t, EnableBias>(
        out, a, b, a_scales, b_scales,
        std::forward<EpilogueArgs>(epilogue_args)...);
  }
}

template <bool EnableBias, typename... EpilogueArgs>
void cutlass_scaled_mm_sm90_fp8_batch_invariant_epilogue(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales, EpilogueArgs&&... epilogue_args) {
  STD_TORCH_CHECK(a.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
  STD_TORCH_CHECK(b.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);

  if (out.scalar_type() == torch::headeronly::ScalarType::BFloat16) {
    return cutlass_gemm_sm90_fp8_batch_invariant_dispatch<
        cutlass::float_e4m3_t, cutlass::bfloat16_t, EnableBias>(
        out, a, b, a_scales, b_scales,
        std::forward<EpilogueArgs>(epilogue_args)...);
  } else {
    STD_TORCH_CHECK(out.scalar_type() == torch::headeronly::ScalarType::Half);
    return cutlass_gemm_sm90_fp8_batch_invariant_dispatch<
        cutlass::float_e4m3_t, cutlass::half_t, EnableBias>(
        out, a, b, a_scales, b_scales,
        std::forward<EpilogueArgs>(epilogue_args)...);
  }
}
```
**EN:** The final helpers stabilize kernel choice for batch-invariant mode and map the runtime output tensor type to BF16 or FP16 instantiations.
**CN:** 最后的辅助函数会在 batch-invariant 模式下稳定内核选择，并把运行时输出张量类型映射为 BF16 或 FP16 实例化。

## Key Concepts / 关键概念
- **Heuristic kernel families / 启发式内核家族**: SM90 uses several hand-tuned kernels instead of a single generic kernel because shape sensitivity is high for FP8 workloads. / 由于 FP8 工作负载对形状十分敏感，SM90 使用多组手工调优内核，而不是单一通用内核。
- **Swap-AB plus column bias / Swap-AB 与列偏置**: When operands are swapped for efficiency, bias handling must rotate with the logical output orientation. / 为了效率而交换操作数时，bias 处理也必须随逻辑输出方向一起调整。

## Dependencies / 依赖关系
- **cutlass_gemm_caller.cuh / cutlass_gemm_caller.cuh**: Relies on the shared launcher helper to allocate workspace and invoke the prepared CUTLASS kernel. / 依赖共享启动辅助函数来分配工作区并调用已准备好的 CUTLASS 内核。
- **scaled_mm epilogues / scaled_mm epilogues**: Connects kernel dispatch to epilogue templates that apply scaling, optional bias, and related post-processing. / 将内核分发逻辑连接到执行缩放、可选 bias 与相关后处理的 epilogue 模板。
- **CUTLASS / CUTLASS**: Uses CUTLASS collective builders, kernel adapters, and architecture tags to assemble GEMM kernels. / 使用 CUTLASS 的 collective builder、kernel adapter 与架构标签来组装 GEMM 内核。
