# scaled_mm_sm120_fp8_dispatch.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_sm120_fp8_dispatch.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines SM120 FP8 kernel wrappers, small-M custom epilogue tiling, and runtime dispatch for regular and batch-invariant execution. / 定义 SM120 FP8 内核包装器、小 M 的自定义 epilogue tile，以及常规和 batch-invariant 执行的运行时分发逻辑。

## Line-by-Line Analysis / 逐行分析
### Custom SM120 wrapper / 自定义 SM120 包装器
```cpp
// Custom wrapper to allow specifying EpilogueTile for small M
template <typename ElementAB_, typename ElementD_,
          template <typename, typename, typename> typename Epilogue_,
          typename TileShape, typename ClusterShape, typename KernelSchedule,
          typename EpilogueSchedule, typename EpilogueTile>
struct cutlass_3x_gemm_sm120_custom {
  using ElementAB = ElementAB_;
  using LayoutA = cutlass::layout::RowMajor;
  static constexpr int AlignmentA =
      128 / cutlass::sizeof_bits<ElementAB>::value;

  using LayoutB = cutlass::layout::ColumnMajor;
  static constexpr int AlignmentB =
      128 / cutlass::sizeof_bits<ElementAB>::value;

  using ElementC = void;
  using LayoutC = cutlass::layout::RowMajor;
  static constexpr int AlignmentC =
      128 / cutlass::sizeof_bits<ElementD_>::value;

  using ElementD = ElementD_;
  using LayoutD = cutlass::layout::RowMajor;
  static constexpr int AlignmentD = AlignmentC;

  using ElementAcc =
      typename std::conditional<std::is_same_v<ElementAB, int8_t>, int32_t,
                                float>::type;
  using Epilogue = Epilogue_<ElementAcc, ElementD, TileShape>;

  // MMA type
  using ElementAccumulator = float;

  // Epilogue types
  using ElementBias = cutlass::half_t;
  using ElementCompute = float;
  using ElementAux = ElementD;
  using LayoutAux = LayoutD;
  using ElementAmax = float;

  using EVTCompute = typename Epilogue::EVTCompute;

  using CollectiveEpilogue =
      typename cutlass::epilogue::collective::CollectiveBuilder<
          cutlass::arch::Sm120, cutlass::arch::OpClassTensorOp, TileShape,
          ClusterShape, EpilogueTile,  // Use custom EpilogueTile
          ElementAccumulator, ElementCompute, ElementC, LayoutC, AlignmentC,
          ElementD, LayoutD, AlignmentD, EpilogueSchedule,
          EVTCompute>::CollectiveOp;

  using CollectiveMainloop =
      typename cutlass::gemm::collective::CollectiveBuilder<
          cutlass::arch::Sm120, cutlass::arch::OpClassTensorOp, ElementAB,
          LayoutA, AlignmentA, ElementAB, LayoutB, AlignmentB,
          ElementAccumulator, TileShape, ClusterShape,
          cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(
              sizeof(typename CollectiveEpilogue::SharedStorage))>,
          KernelSchedule, void>::CollectiveOp;

  using GemmKernel = enable_sm120_family<cutlass::gemm::kernel::GemmUniversal<
      Shape<int, int, int, int>, CollectiveMainloop, CollectiveEpilogue, void>>;
};
```
**EN:** The custom wrapper exists so small-M kernels can override the epilogue tile shape while still reusing the shared SM120 GEMM construction pattern.
**CN:** 该自定义包装器的存在，是为了让小 M 内核能够覆盖 epilogue tile 形状，同时继续复用共享的 SM120 GEMM 构造模式。

### Configuration structs / 配置结构体
```cpp
template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue>
struct sm120_fp8_config_default {
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule = cutlass::gemm::collective::KernelScheduleAuto;
  using EpilogueSchedule = cutlass::epilogue::collective::EpilogueScheduleAuto;
  using TileShape = Shape<_128, _128, _128>;
  using ClusterShape = Shape<_1, _1, _1>;  // Only work with Shape<_1, _1, _1>
  using Cutlass3xGemm =
      cutlass_3x_gemm_sm120<InType, OutType, Epilogue, TileShape, ClusterShape,
                            KernelSchedule, EpilogueSchedule>;
};

template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue>
struct sm120_fp8_config_M64 {
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  // SM120 Cooperative kernel requires Tile M >= 128.
  // For M=64 tile, we use Pingpong schedule which is more flexible with small
  // tiles.
  using KernelSchedule = cutlass::gemm::KernelTmaWarpSpecializedPingpong;
  using EpilogueSchedule = cutlass::epilogue::collective::EpilogueScheduleAuto;
  using TileShape = Shape<_64, _64, _128>;
  // CUTLASS 3.x on SM120 currently restricts programmatic multicast (Cluster >
  // 1) for certain schedules/types. Reverting to 1x1x1 to ensure compilation.
  using ClusterShape = Shape<_1, _1, _1>;
  using Cutlass3xGemm =
      cutlass_3x_gemm_sm120<InType, OutType, Epilogue, TileShape, ClusterShape,
                            KernelSchedule, EpilogueSchedule>;
};

template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue>
struct sm120_fp8_config_M32 {
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule = cutlass::gemm::KernelTmaWarpSpecializedPingpong;
  using EpilogueSchedule = cutlass::epilogue::collective::EpilogueScheduleAuto;
  using TileShape = Shape<_32, _64, _128>;
  using ClusterShape = Shape<_1, _1, _1>;
  // Use custom gemm to specify EpilogueTile M=32
  using Cutlass3xGemm =
      cutlass_3x_gemm_sm120_custom<InType, OutType, Epilogue, TileShape,
                                   ClusterShape, KernelSchedule,
                                   EpilogueSchedule, Shape<_32, _32>>;
};

template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue>
struct sm120_fp8_config_M16 {
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  using KernelSchedule = cutlass::gemm::KernelTmaWarpSpecializedPingpong;
  using EpilogueSchedule = cutlass::epilogue::collective::EpilogueScheduleAuto;
  using TileShape = Shape<_16, _64, _128>;
  using ClusterShape = Shape<_1, _1, _1>;
  // Use custom gemm to specify EpilogueTile M=16
  using Cutlass3xGemm =
      cutlass_3x_gemm_sm120_custom<InType, OutType, Epilogue, TileShape,
                                   ClusterShape, KernelSchedule,
                                   EpilogueSchedule, Shape<_16, _32>>;
};
```
**EN:** The header defines default, M64, M32, and M16 kernel families. The smaller variants rely on ping-pong scheduling and custom epilogue tile shapes.
**CN:** 该头文件定义了 default、M64、M32 和 M16 四个内核族，其中更小的变体依赖 ping-pong 调度和自定义 epilogue tile 形状。

### Shape-based dispatch / 基于形状的分发
```cpp
template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue,
          typename... EpilogueArgs>
inline void cutlass_gemm_sm120_fp8_dispatch(torch::stable::Tensor& out,
                                            torch::stable::Tensor const& a,
                                            torch::stable::Tensor const& b,
                                            EpilogueArgs&&... args) {
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  STD_TORCH_CHECK(a.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
  STD_TORCH_CHECK(b.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);

  int M = a.size(0);

  if (M <= 16) {
    using Cutlass3xGemmM16 =
        typename sm120_fp8_config_M16<InType, OutType, Epilogue>::Cutlass3xGemm;
    return cutlass_gemm_caller<Cutlass3xGemmM16>(
        out, a, b, std::forward<EpilogueArgs>(args)...);
  }
  if (M <= 32) {
    using Cutlass3xGemmM32 =
        typename sm120_fp8_config_M32<InType, OutType, Epilogue>::Cutlass3xGemm;
    return cutlass_gemm_caller<Cutlass3xGemmM32>(
        out, a, b, std::forward<EpilogueArgs>(args)...);
  }

  if (M <= 256) {
    using Cutlass3xGemmM64 =
        typename sm120_fp8_config_M64<InType, OutType, Epilogue>::Cutlass3xGemm;
    return cutlass_gemm_caller<Cutlass3xGemmM64>(
        out, a, b, std::forward<EpilogueArgs>(args)...);
  }

  using Cutlass3xGemmDefault =
      typename sm120_fp8_config_default<InType, OutType,
                                        Epilogue>::Cutlass3xGemm;
  return cutlass_gemm_caller<Cutlass3xGemmDefault>(
      out, a, b, std::forward<EpilogueArgs>(args)...);
```
**EN:** Runtime dispatch only needs `M`: very small matrices use M16/M32 kernels, medium matrices use the M64 path, and larger ones fall back to the default kernel.
**CN:** 运行时分发只需要看 `M`：很小的矩阵使用 M16/M32 内核，中等矩阵使用 M64 路径，更大的矩阵则回退到默认内核。

### Batch-invariant helper / 批不变辅助逻辑
```cpp
template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue,
          typename... EpilogueArgs>
inline void cutlass_gemm_sm120_fp8_batch_invariant_dispatch(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, EpilogueArgs&&... args) {
  static_assert(std::is_same<InType, cutlass::float_e4m3_t>());
  STD_TORCH_CHECK(a.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
  STD_TORCH_CHECK(b.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);

  using Cutlass3xGemmM64 =
      typename sm120_fp8_config_M64<InType, OutType, Epilogue>::Cutlass3xGemm;

  // keep the CUTLASS config independent of M for batch invariance
  return cutlass_gemm_caller<Cutlass3xGemmM64>(
      out, a, b, std::forward<EpilogueArgs>(args)...);
```
**EN:** For batch-invariant mode, the code intentionally pins execution to the M64 family so kernel choice does not depend on batch size.
**CN:** 在 batch-invariant 模式下，代码有意把执行固定到 M64 家族，以避免内核选择依赖 batch 大小。

### Epilogue wrappers / Epilogue 包装器
```cpp
template <template <typename, typename, typename> typename Epilogue,
          typename... EpilogueArgs>
void cutlass_scaled_mm_sm120_fp8_epilogue(torch::stable::Tensor& out,
                                          torch::stable::Tensor const& a,
                                          torch::stable::Tensor const& b,
                                          EpilogueArgs&&... epilogue_args) {
  STD_TORCH_CHECK(a.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
  STD_TORCH_CHECK(b.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);

  if (out.scalar_type() == torch::headeronly::ScalarType::BFloat16) {
    return cutlass_gemm_sm120_fp8_dispatch<cutlass::float_e4m3_t,
                                           cutlass::bfloat16_t, Epilogue>(
        out, a, b, std::forward<EpilogueArgs>(epilogue_args)...);
  } else {
    STD_TORCH_CHECK(out.scalar_type() == torch::headeronly::ScalarType::Half);
    return cutlass_gemm_sm120_fp8_dispatch<cutlass::float_e4m3_t,
                                           cutlass::half_t, Epilogue>(
        out, a, b, std::forward<EpilogueArgs>(epilogue_args)...);
  }
}

template <template <typename, typename, typename> typename Epilogue,
          typename... EpilogueArgs>
void cutlass_scaled_mm_sm120_fp8_batch_invariant_epilogue(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, EpilogueArgs&&... epilogue_args) {
  STD_TORCH_CHECK(a.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
  STD_TORCH_CHECK(b.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);

  if (out.scalar_type() == torch::headeronly::ScalarType::BFloat16) {
    return cutlass_gemm_sm120_fp8_batch_invariant_dispatch<
        cutlass::float_e4m3_t, cutlass::bfloat16_t, Epilogue>(
        out, a, b, std::forward<EpilogueArgs>(epilogue_args)...);
  } else {
    STD_TORCH_CHECK(out.scalar_type() == torch::headeronly::ScalarType::Half);
    return cutlass_gemm_sm120_fp8_batch_invariant_dispatch<
        cutlass::float_e4m3_t, cutlass::half_t, Epilogue>(
        out, a, b, std::forward<EpilogueArgs>(epilogue_args)...);
  }
}
```
**EN:** The final wrappers validate FP8 inputs and route the runtime output type to the correct BF16 or FP16 instantiation.
**CN:** 最后的包装器会验证 FP8 输入，并把运行时输出类型路由到正确的 BF16 或 FP16 实例化。

## Key Concepts / 关键概念
- **Custom epilogue tiles / 自定义 epilogue tile**: Small-M kernels need non-default epilogue tiles to keep the SM120 path compiling and efficient. / 小 M 内核需要非默认 epilogue tile，才能让 SM120 路径同时保持可编译和高效。

## Dependencies / 依赖关系
- **cutlass_gemm_caller.cuh / cutlass_gemm_caller.cuh**: Relies on the shared launcher helper to allocate workspace and invoke the prepared CUTLASS kernel. / 依赖共享启动辅助函数来分配工作区并调用已准备好的 CUTLASS 内核。
- **CUTLASS / CUTLASS**: Uses CUTLASS collective builders, kernel adapters, and architecture tags to assemble GEMM kernels. / 使用 CUTLASS 的 collective builder、kernel adapter 与架构标签来组装 GEMM 内核。
- **scaled_mm.cuh / scaled_mm.cuh**: Reuses the shared SM120 GEMM building blocks from `scaled_mm.cuh`. / 复用 `scaled_mm.cuh` 中共享的 SM120 GEMM 构建模块。
