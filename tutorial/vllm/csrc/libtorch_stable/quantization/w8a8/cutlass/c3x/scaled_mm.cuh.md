# scaled_mm.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines reusable CUTLASS 3.x GEMM building blocks for SM90, SM100, and SM120 scaled matrix multiplication kernels. / 为 SM90、SM100 和 SM120 的缩放矩阵乘内核定义可复用的 CUTLASS 3.x GEMM 构建模块。

## Line-by-Line Analysis / 逐行分析
### Epilogue contract / Epilogue 契约
```cpp
/*
  Epilogues defined in,
  csrc/cutlass_extensions/epilogue/scaled_mm_epilogues_c3x.hpp,
  must contain a public type named EVTCompute of type Sm90EVT, as well as a
  static prepare_args function that constructs an EVTCompute::Arguments struct.
```
**EN:** The header documents the required interface for epilogue templates: they must expose `EVTCompute` and a `prepare_args` helper that builds the epilogue argument object.
**CN:** 该头文件说明了 epilogue 模板必须满足的接口：需要暴露 `EVTCompute`，并提供 `prepare_args` 辅助函数来构造 epilogue 参数对象。

### SM90 generic GEMM / SM90 通用 GEMM
```cpp
template <typename ElementAB_, typename ElementD_,
          template <typename, typename, typename> typename Epilogue_,
          typename TileShape, typename ClusterShape, typename KernelSchedule,
          typename EpilogueSchedule>
struct cutlass_3x_gemm {
  using ElementAB = ElementAB_;
  using ElementD = ElementD_;
  using ElementAcc =
      typename std::conditional<std::is_same_v<ElementAB, int8_t>, int32_t,
                                float>::type;

  using Epilogue = Epilogue_<ElementAcc, ElementD, TileShape>;

  using StrideD = Stride<int64_t, Int<1>, Int<0>>;
  using ElementC = void;
  using StrideC = StrideD;

  using EVTCompute = typename Epilogue::EVTCompute;

  // These are the minimum alignments needed for the kernels to compile
  static constexpr int AlignmentAB =
      128 / cutlass::sizeof_bits<ElementAB>::value;
  static constexpr int AlignmentCD =
      128 / cutlass::sizeof_bits<ElementD>::value;

  using CollectiveEpilogue =
      typename cutlass::epilogue::collective::CollectiveBuilder<
          cutlass::arch::Sm90, cutlass::arch::OpClassTensorOp, TileShape,
          ClusterShape, cutlass::epilogue::collective::EpilogueTileAuto,
          ElementAcc, float, ElementC, StrideC, AlignmentCD, ElementD, StrideD,
          AlignmentCD, EpilogueSchedule, EVTCompute>::CollectiveOp;

  static constexpr size_t CEStorageSize =
      sizeof(typename CollectiveEpilogue::SharedStorage);
  using Stages = typename cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(CEStorageSize)>;

  // clang-format off
  using CollectiveMainloop =
      typename cutlass::gemm::collective::CollectiveBuilder<
          cutlass::arch::Sm90, cutlass::arch::OpClassTensorOp, 
          ElementAB, cutlass::layout::RowMajor, AlignmentAB, 
          ElementAB, cutlass::layout::ColumnMajor, AlignmentAB, 
          ElementAcc, TileShape, ClusterShape,
          Stages,
          KernelSchedule>::CollectiveOp;
  // clang-format on

  using KernelType = enable_sm90_or_later<cutlass::gemm::kernel::GemmUniversal<
      cute::Shape<int, int, int, int>, CollectiveMainloop, CollectiveEpilogue,
      cutlass::gemm::PersistentScheduler>>;

  struct GemmKernel : public KernelType {};
```
**EN:** This template builds an SM90 CUTLASS GEMM from input/output element types, tile shapes, and epilogue policy, including automatic stage sizing based on epilogue shared storage.
**CN:** 该模板根据输入/输出元素类型、tile 形状和 epilogue 策略构造 SM90 CUTLASS GEMM，并依据 epilogue 共享存储自动计算 stage 数。

### SM100 generic GEMM / SM100 通用 GEMM
```cpp
struct cutlass_3x_gemm_sm100 {
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
          cutlass::arch::Sm100, cutlass::arch::OpClassTensorOp, TileShape,
          ClusterShape, cutlass::epilogue::collective::EpilogueTileAuto,
          ElementAccumulator, ElementCompute, ElementC, LayoutC, AlignmentC,
          ElementD, LayoutD, AlignmentD, EpilogueSchedule,
          EVTCompute>::CollectiveOp;

  using CollectiveMainloop =
      typename cutlass::gemm::collective::CollectiveBuilder<
          cutlass::arch::Sm100, cutlass::arch::OpClassTensorOp, ElementAB,
          LayoutA, AlignmentA, ElementAB, LayoutB, AlignmentB,
          ElementAccumulator, TileShape, ClusterShape,
          cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(
              sizeof(typename CollectiveEpilogue::SharedStorage))>,
          KernelSchedule>::CollectiveOp;

  using GemmKernel = enable_sm100_to_sm120<cutlass::gemm::kernel::GemmUniversal<
      Shape<int, int, int, int>, CollectiveMainloop, CollectiveEpilogue, void>>;
};
```
**EN:** The SM100 version switches to layout-based builders required by the newer architecture and keeps the epilogue contract compatible with the shared scaled-mm epilogues.
**CN:** SM100 版本切换到新架构所需的基于布局的 builder，同时保持与共享 scaled-mm epilogue 的契约兼容。

### SM120 generic GEMM / SM120 通用 GEMM
```cpp
struct cutlass_3x_gemm_sm120 {
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
          ClusterShape, cutlass::epilogue::collective::EpilogueTileAuto,
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
          KernelSchedule>::CollectiveOp;

  using GemmKernel = enable_sm120_family<cutlass::gemm::kernel::GemmUniversal<
      Shape<int, int, int, int>, CollectiveMainloop, CollectiveEpilogue, void>>;
```
**EN:** The SM120 variant mirrors the SM100 structure but targets the SM120 architecture family and its corresponding `enable_sm120_family` wrapper.
**CN:** SM120 变体延续了 SM100 的结构，但面向 SM120 架构家族，并使用对应的 `enable_sm120_family` 包装器。

## Key Concepts / 关键概念
- **Architecture-specific builders / 架构特定构建器**: A single header centralizes the type-level differences between SM90, SM100, and SM120 kernels. / 单个头文件集中管理 SM90、SM100 与 SM120 内核在类型层面的差异。
- **Epilogue as a policy / 以后处理作为策略**: The GEMM type stays generic by injecting the epilogue as a template parameter. / 通过把 epilogue 注入为模板参数，GEMM 类型保持通用性。

## Dependencies / 依赖关系
- **CUTLASS / CUTLASS**: Uses CUTLASS collective builders, kernel adapters, and architecture tags to assemble GEMM kernels. / 使用 CUTLASS 的 collective builder、kernel adapter 与架构标签来组装 GEMM 内核。
- **scaled_mm_epilogues_c3x.hpp contract / scaled_mm_epilogues_c3x.hpp 契约**: All epilogue templates used here must satisfy the documented interface. / 这里使用的所有 epilogue 模板都必须满足文档中约定的接口。
