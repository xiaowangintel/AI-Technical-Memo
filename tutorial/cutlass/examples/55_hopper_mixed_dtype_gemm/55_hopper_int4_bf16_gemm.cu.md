# 55_hopper_int4_bf16_gemm.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/55_hopper_mixed_dtype_gemm/55_hopper_int4_bf16_gemm.cu`  
**Purpose / 用途**: Shows Hopper mixed-input GEMM where a narrow INT4/FP4 operand is converted or dequantized into the MMA type, optionally scaled and zero-point adjusted, while preserving a high-performance TMA epilogue through an explicit operand swap and optional offline weight reordering. / 展示 Hopper 上的混合输入 GEMM：把窄类型 INT4/FP4 操作数在进入 MMA 前转换或反量化到 MMA 类型，可选地应用 scale 和 zero-point，同时通过显式交换操作数与可选的离线权重重排来保留高性能 TMA epilogue。

---

## Line-by-Line Analysis / 逐行分析

### Problem statement and mixed-dtype constraints (Lines 32-83)

```cpp
/*! \file
    \brief Hopper GEMM example with different data types using CUTLASS 3.0 APIs for NVIDIA Hopper architecture

    This example shows how to perform INT4 x BF16 GEMM and scale up the INT4 weight during dequantization.

    The narrower type always passes through the register file. Therefore, in cases where the narrower type is operand B, the collective will implicitly swap 
    A and B in the main loop. However, as a result of this collective performing implicit swaps, it does not support TMA epilogues. Consequently, it is essential to consider this when constructing the epilogue, 
    as illustrated in this example.

    Note that in this example, we explicitly swap A and B in order to use TMA epilogues. We do this since TMA epilogues are more performant on problem sizes of interest.

    As an additional optimization, we can reorder the narrow data type tensor such that elements read into register file by the same thread are contiguous in global and shared memory.
    This promotes vectorization of shared memory loads and removes additional instructions on the critical path. For example, when MMA is performed in 16-bit data type, each thread reads
    4 groups of 2 elements that are logically contiguous in the same row (refer to https://docs.nvidia.com/cuda/parallel-thread-execution/index.html#wgmma-64n16-a for thread-value layout).
    If the narrow type is INT4 and tensor is major in K dim, only 8 bits can be read at a time, leading to extra load instructions and suboptimal utilization of shared memory throughput.
    If we reorder the data offline to place all 16 elements read by a thread contiguously in memory, a single 64-bit load is sufficient. This reordering is often feasible when the quantized
    tensor is static (e.g. weight tensor of a NN layer at inference time). This example demonstrates how such a reordering can be performed and communicated to the kernel when the options.shuffle is set to true.

    Furthermore, the conversion from {INT4, UINT4} to {FP16, BF16} can benefit from pre-shuffling the weights in the order [0,2,4,6,1,3,5,7]. This allows multiple nibbles to be efficiently extracted and up-converted
    in parallel. The reordering is enabled by defining the layout type `ValueShuffle`. Refer to the partial specializations of `NumericArrayShuffleConverter` in "include/cutlass/detail/collective/mixed_input_utils.hpp"
    for more details.

    It is expected that the scale's K dimension be scale_k = ceil_div(problem_k, group_size). 
    
    Scales are always expected to be MN major. This means the fastest changing dimension must be M if A is scaled or N if B is scaled.
    
    If A is being scaled, the scales must have shape [M, scale_k],  while if B is scaled, it must have shape [N, scale_k].

    The implementation only supports "group-wise" scales. However, we can make it work for per-column scales by setting the group's size
    equal to the gemm problem K.

    Limitations:
      1) The INT4 weights have additional encoding requirements.
      2) The scales must be MN major. That means if A is scaled, it must be column major, but if B is scaled it must be row major.
      3) The scales must have the same layout and groupsize.
      4) The groupsize must be greater or equal to the tile shape k.
      5) Currently, TMA epilogues cannot be used when the narrow type is the B operand. This limitation arises because the implementation always swaps the 
         operands to ensure that the narrow type passes through the register file, and TMA epilogues do not currently support implicit swap + transpose operations. 
         We plan to address this limitation in the future. However, we address this in the example by explicitly swapping and transposing the operands.
    
    Optimizing suggestions:
      1) Use a small tile size, since the register pressure for this GEMM (and RS GEMM in general) is high (it uses a lot of register space).

    Examples:
      
      Runs the mixed input batched gemm (with batch size 2), converting B to the type of A (mode 0)
      $ ./examples/55_hopper_mixed_dtype_gemm/55_hopper_int4_bf16_gemm --m=2048 --n=2048 --k=2048 --l=2 --mode=0

      Runs the mixed input gemm, and applies a scaling factor to B before mma (mode 1). Applies a vector of scales to the entire
      matrix (group size is the same as the gemm k dimension).
      $ ./examples/55_hopper_mixed_dtype_gemm/55_hopper_int4_bf16_gemm --m=4096 --n=5120 --k=8192 --g=8192 --mode=1
*/
```

**EN**: The opening comment is unusually important because it explains the core design constraint: the narrower type must travel through registers, which would normally trigger an implicit A/B swap inside the collective and disable TMA epilogues. The example therefore chooses an explicit swap-and-transpose strategy so it can still use Hopper TMA epilogues, and it documents scale layout, group size, and offline shuffle requirements precisely.

**CN**: 开头注释非常关键，因为它解释了本示例的核心约束：窄类型必须经过寄存器路径，这通常会让 collective 在内部隐式交换 A/B，并导致无法使用 TMA epilogue。为此，示例改用“显式交换 + 转置”的策略，从而仍能使用 Hopper 的 TMA epilogue；同时它还明确说明了 scale 布局、group size 和离线 shuffle 的要求。

### Selecting MMA type, quant type, and reorder layout (Lines 119-175)

```cpp
// Select MMA type via compile flag
#if defined(CUTLASS_USE_FP16)
  using MmaType = cutlass::half_t;        // FP16
#elif defined(CUTLASS_USE_TF32)
  using MmaType = cutlass::tfloat32_t;    // TF32 (FP32 format with reduced precision)
#else
  using MmaType = cutlass::bfloat16_t;    // BF16 (default)
#endif

// Select quantization type via compile flag for this example
#if defined(CUTLASS_MIXED_DTYPE_E2M1)
  using QuantType = cutlass::float_e2m1_t;  // E2M1 (FP4)
#else
  using QuantType = cutlass::int4b_t;       // INT4 Two's Complement (default)
#endif

// TF32 requires K to be multiple of 8; BF16/FP16 can go higher
#if defined(CUTLASS_USE_TF32)
  constexpr int TileShapeK = 64;  // TF32: K must be multiple of 8, use 64 for good performance
#else
  constexpr int TileShapeK = 128 * 8 / sizeof_bits<MmaType>::value;
#endif

// A matrix configuration
using         ElementA    = MmaType;                                        // Element type for A matrix operand
using         LayoutA     = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
constexpr int AlignmentA  = 128 / cutlass::sizeof_bits<ElementA>::value;    // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)

// B matrix configuration
using         ElementB    = QuantType;                                      // Element type for B matrix operand
using         LayoutB     = cutlass::layout::ColumnMajor;                   // Layout type for B matrix operand
constexpr int AlignmentB  = 128 / cutlass::sizeof_bits<ElementB>::value;    // Memory access granularity/alignment of B matrix in units of elements (up to 16 bytes)

// This example manually swaps and transposes, so keep transpose of input layouts
using LayoutA_Transpose = typename cutlass::layout::LayoutTranspose<LayoutA>::type;
using LayoutB_Transpose = typename cutlass::layout::LayoutTranspose<LayoutB>::type;

using StrideA = cutlass::detail::TagToStrideA_t<LayoutA>;
using StrideB = cutlass::detail::TagToStrideB_t<LayoutB>;

// Define the CuTe layout for reoredered quantized tensor B
// LayoutAtomQuant places values that will be read by the same thread in contiguous locations in global memory.
// It specifies the reordering within a single warp's fragment
#if defined(CUTLASS_MIXED_DTYPE_E2M1) || defined(CUTLASS_USE_TF32)
// E2M1 & TF32: Use simpler layout without ValueShuffle (like FP8 example)
// ValueShuffle currrently isn't enabled for E2M1 until LayoutAwareConvertImpl specializations support shuffle reordering.
// and for TF32 until we support both the K=8 vs K=16 dimensions for tiles
using LayoutAtomQuant = decltype(cutlass::compute_memory_reordering_atom<MmaType>());
#else
// INT4: Use ValueShuffle for optimal performance with FP16/BF16
// using ValueShuffle = Layout<_1>;                          // no value reordering
using ValueShuffle = Layout<Shape<_2,_4>, Stride<_4,_1>>; // order [0,2,4,6,1,3,5,7]
int constexpr NumShuffleAtoms = 1;
using MmaAtomShape = Layout<Shape<_1,Int<NumShuffleAtoms>>>;
using LayoutAtomQuant = decltype(cutlass::compute_memory_reordering_atom<MmaType, MmaAtomShape, ValueShuffle>());
#endif
using LayoutB_Reordered = decltype(cute::tile_to_shape(LayoutAtomQuant{}, Layout<Shape<int,int,int>, StrideB>{}));
```

**EN**: Compile-time flags choose the wide MMA type (`BF16` by default, optionally FP16 or TF32) and the narrow quantized type (`INT4` by default, optionally E2M1/FP4). The code then defines `LayoutAtomQuant` for reordered B storage. In the INT4 case it enables `ValueShuffle = [0,2,4,6,1,3,5,7]`, which matches the hardware-friendly nibble extraction order used by the mixed-input converters.

**CN**: 编译期宏决定宽类型 MMA 数据类型（默认 BF16，也可选 FP16 或 TF32）和窄类型量化类型（默认 INT4，也可选 E2M1/FP4）。随后代码定义 `LayoutAtomQuant` 来描述重排后的 B 存储方式。在 INT4 情况下，它启用了 `ValueShuffle = [0,2,4,6,1,3,5,7]`，这与混合输入转换器更高效的 nibble 提取顺序相匹配。

### Epilogue built around explicit swap + transpose (Lines 181-214)

```cpp
// C/D matrix configuration
using         ElementC    = MmaType;                                        // Element type for C and D matrix operands (matches MMA type)
using         LayoutC     = cutlass::layout::RowMajor;                      // Layout type for C and D matrix operands
constexpr int AlignmentC  = 128 / cutlass::sizeof_bits<ElementC>::value;    // Memory access granularity/alignment of C matrix in units of elements (up to 16 bytes)

// D matrix configuration
using         ElementD    = ElementC;
using         LayoutD     = LayoutC;
constexpr int AlignmentD  = 128 / cutlass::sizeof_bits<ElementD>::value;

// Core kernel configurations
using ElementAccumulator  = float;                                          // Element type for internal accumulation
using ElementCompute      = float;                                          // Element type for epilogue computation
using ArchTag             = cutlass::arch::Sm90;                            // Tag indicating the minimum SM that supports the intended feature
using OperatorClass       = cutlass::arch::OpClassTensorOp;                 // Operator class tag
using TileShape           = Shape<_128,_128,cute::Int<TileShapeK>>;         // Threadblock-level tile size
using ClusterShape        = Shape<_1,_1,_1>;                                // Shape of the threadblocks in a cluster
using KernelSchedule      = cutlass::gemm::KernelTmaWarpSpecializedCooperative;  // Kernel to launch based on the default setting in the Collective Builder 
using EpilogueSchedule    = cutlass::epilogue::TmaWarpSpecializedCooperative;
using EpilogueTileType    = cutlass::epilogue::collective::EpilogueTileAuto;

using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
    cutlass::arch::Sm90, cutlass::arch::OpClassTensorOp,
    TileShape, ClusterShape,
    EpilogueTileType,
    ElementAccumulator, ElementAccumulator,
    // Transpose layout of D here since we use explicit swap + transpose
    // the void type for C tells the builder to allocate 0 smem for the C matrix.
    // We can enable this if beta == 0 by changing ElementC to void below.
    ElementC, typename cutlass::layout::LayoutTranspose<LayoutC>::type, AlignmentC,
    ElementD, typename cutlass::layout::LayoutTranspose<LayoutD>::type, AlignmentD,
    EpilogueSchedule // This is the only epi supporting the required swap + transpose.
  >::CollectiveOp;
```

**EN**: Because the example manually swaps A and B to keep TMA epilogues legal, it also transposes the C and D layouts when building the epilogue collective. This is the subtle but crucial detail: the kernel computes the transposed problem shape internally, and the epilogue is configured to land results back correctly even though the input interpretation was intentionally swapped.

**CN**: 由于示例手动交换了 A 与 B，以保持 TMA epilogue 可用，因此在构造 epilogue collective 时也同步转置了 C 和 D 的布局。这是最关键的细节之一：内核内部实际上计算的是交换后的转置问题，而 epilogue 的配置保证输出仍能正确落到目标矩阵上。

### Builder variants for convert-only, scaled, and zero-point modes (Lines 215-340)

```cpp
// ============================================================ MIXED INPUT NO SCALES ============================================================================
// The collective will infer that the narrow type should be upcasted to the wide type.
// We swap A and B operands to the builder here
using CollectiveMainloopConvertOnly = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    ElementB, LayoutB_Transpose, AlignmentB,
    ElementA, LayoutA_Transpose, AlignmentA,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))
    >,
    KernelSchedule
  >::CollectiveOp;

using GemmKernelConvertOnly = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>, // Indicates ProblemShape
    CollectiveMainloopConvertOnly,
    CollectiveEpilogue
>;

using GemmConvertOnly = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelConvertOnly>;

using CollectiveMainloopConvertOnlyShuffled = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    ElementB, LayoutB_Reordered, AlignmentB,
    ElementA, LayoutA_Transpose, AlignmentA,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))
    >,
    KernelSchedule
  >::CollectiveOp;

using GemmKernelConvertOnlyShuffled = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>, // Indicates ProblemShape
    CollectiveMainloopConvertOnlyShuffled,
    CollectiveEpilogue
>;

using GemmConvertOnlyShuffled = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelConvertOnlyShuffled>;

// =========================================================== MIXED INPUT WITH SCALES ===========================================================================
// The Scale information must get paired with the operand that will be scaled. In this example, B is scaled so we make a tuple of B's information and the scale information.
using CollectiveMainloopScaleOnly = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    cute::tuple<ElementB, ElementScale>, LayoutB_Transpose, AlignmentB,
    ElementA, LayoutA_Transpose, AlignmentA,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))
    >,
    KernelSchedule
  >::CollectiveOp;

using GemmKernelScaleOnly = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>, // Indicates ProblemShape
    CollectiveMainloopScaleOnly,
    CollectiveEpilogue
>;

using GemmScaleOnly = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelScaleOnly>;

using CollectiveMainloopScaleOnlyShuffled = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    cute::tuple<ElementB, ElementScale>, LayoutB_Reordered, AlignmentB,
    ElementA, LayoutA_Transpose, AlignmentA,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))
    >,
    KernelSchedule
  >::CollectiveOp;

using GemmKernelScaleOnlyShuffled = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>, // Indicates ProblemShape
    CollectiveMainloopScaleOnlyShuffled,
    CollectiveEpilogue
>;

using GemmScaleOnlyShuffled = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelScaleOnlyShuffled>;

// =========================================================== MIXED INPUT WITH SCALES AND ZEROS ==================================================================
// We specify scale + zero elements to indicate that we require both. Scales and biases have the same format.
using CollectiveMainloopScaleWithZeroPoint = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    cute::tuple<ElementB, ElementScale, ElementZero>, LayoutB_Transpose, AlignmentB,
    ElementA, LayoutA_Transpose, AlignmentA,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))
    >,
    KernelSchedule
  >::CollectiveOp;

using GemmKernelScaleWithZeroPoint = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>, // Indicates ProblemShape
    CollectiveMainloopScaleWithZeroPoint,
    CollectiveEpilogue
>;

using GemmScaleWithZeroPoint = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelScaleWithZeroPoint>;

using CollectiveMainloopScaleWithZeroPointShuffled = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    cute::tuple<ElementB, ElementScale, ElementZero>, LayoutB_Reordered, AlignmentB,
    ElementA, LayoutA_Transpose, AlignmentA,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))
    >,
    KernelSchedule
  >::CollectiveOp;

using GemmKernelScaleWithZeroPointShuffled = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>, // Indicates ProblemShape
    CollectiveMainloopScaleWithZeroPointShuffled,
    CollectiveEpilogue
>;

using GemmScaleWithZeroPointShuffled = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelScaleWithZeroPointShuffled>;
```

**EN**: This long type block encodes the mixed-dtype API surface. `CollectiveMainloopConvertOnly` asks the builder to up-convert B into the MMA type. `CollectiveMainloopScaleOnly` replaces B’s descriptor with `tuple<ElementB, ElementScale>` so the mainloop knows it must fetch scales and apply group-wise dequantization. `CollectiveMainloopScaleWithZeroPoint` extends that tuple with `ElementZero`. Each of these comes in both original-layout and shuffled-layout forms, so the runtime can toggle offline swizzling without changing kernel logic.

**CN**: 这一大段类型定义实际上编码了混合数据类型 API 的不同模式。`CollectiveMainloopConvertOnly` 告诉 builder 只需把 B 上转换到 MMA 类型；`CollectiveMainloopScaleOnly` 则把 B 的描述改成 `tuple<ElementB, ElementScale>`，表示主循环还要加载 scale 并做 group-wise 反量化；`CollectiveMainloopScaleWithZeroPoint` 再额外加入 `ElementZero`。每种模式又各有“原布局”和“shuffle 后布局”两个版本，因此运行时可以切换离线 swizzle，而不需要改内核主体。

### Global allocations and command-line surface (Lines 353-421)

```cpp
/// Initialization
StrideA stride_A;
StrideB stride_B;
StrideC stride_C;
StrideC_ref stride_C_ref;
StrideD stride_D;
StrideD_ref stride_D_ref;
uint64_t seed;

LayoutB_Reordered layout_B_reordered;

using StrideS = typename CollectiveMainloopScaleOnly::StrideScale;
using StrideS_ref = cutlass::detail::TagToStrideB_t<LayoutScale>;
StrideS stride_S;
StrideS_ref stride_S_ref;

cutlass::DeviceAllocation<ElementA> block_A;
cutlass::DeviceAllocation<ElementB> block_B;
cutlass::DeviceAllocation<ElementA> block_B_dq;
cutlass::DeviceAllocation<ElementScale> block_scale;
cutlass::DeviceAllocation<ElementZero> block_zero;
cutlass::DeviceAllocation<ElementC> block_C;
cutlass::DeviceAllocation<typename GemmScaleOnly::EpilogueOutputOp::ElementOutput> block_D;
cutlass::DeviceAllocation<typename GemmScaleOnly::EpilogueOutputOp::ElementOutput> block_ref_D;

#endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

/////////////////////////////////////////////////////////////////////////////////////////////////
/// Testbed utility types
/////////////////////////////////////////////////////////////////////////////////////////////////

// Command line options parsing
struct Options : MixedDtypeOptions{
  bool shuffle = true;

  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);
    cmd.get_cmd_line_argument("shuffle", shuffle);

    this->MixedDtypeOptions::parse(argc, args);
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "55_hopper_int4_bf16_gemm\n\n"
      << "  Hopper Mixed Data Type GEMM using a Warp Specialized kernel.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --m=<int>                   Sets the M extent of the GEMM\n"
      << "  --n=<int>                   Sets the N extent of the GEMM\n"
      << "  --k=<int>                   Sets the K extent of the GEMM\n"
      << "  --l=<int>                   The number of independent gemm problems with mnk shape\n"
      << "  --g=<int>                   The size of each group for the scales. To broadcast a vector of scales or zeros, set the group size to K.\n"
      << "  --mode=<int>                The mode to run the gemm. 0 does (A @ B), 1 means A @ (scale * B), 2 means A @ (scale * B + zero-point).\n"
      << "  --alpha=<f32>               Epilogue scalar alpha\n"
      << "  --beta=<f32>                Epilogue scalar beta\n\n"
      << "  --iterations=<int>          Number of profiling iterations to perform.\n\n"
      << "  --warmup=<int>              Number of warmup iterations to perform.\n\n"
      << "  --shuffle=<boolean>         Enable the offline layout swizzling.\n\n";

    out
      << "\n\nExamples:\n\n"
      << "$ " << "55_hopper_int4_bf16_gemm" << " --m=1024 --n=512 --k=1024 -g=1024 --l=10 --alpha=2 --mode=2 --beta=0.707 \n\n";

    return out;
  }
};
```

**EN**: The data members allocate space for quantized B, its dequantized reference copy, scales, zero points, and output buffers. `Options` inherits from `MixedDtypeOptions`, adds the `shuffle` flag, and exposes the user-facing controls that select plain conversion, scale-only dequantization, or scale-plus-zero-point dequantization.

**CN**: 这里的数据成员为量化后的 B、其反量化参考副本、scale、zero-point 以及输出缓冲区分配空间。`Options` 继承自 `MixedDtypeOptions`，新增 `shuffle` 开关，并对外暴露“仅转换”“带 scale 的反量化”“带 scale 和 zero-point 的反量化”等模式。

### Initialization, dequantization, and optional offline shuffle (Lines 429-482)

```cpp
/// Initialize operands to be used in the GEMM and reference GEMM
void initialize(Options const& options) {

  auto shape_B = cute::make_shape(options.n, options.k, options.l);
  int const scale_k = cutlass::ceil_div(options.k, options.g);
  stride_A = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(options.m, options.k, options.l));
  stride_B = cutlass::make_cute_packed_stride(StrideB{}, shape_B);
  // Reverse stride here due to swap and transpose
  stride_C = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(options.n, options.m, options.l));
  stride_C_ref = cutlass::make_cute_packed_stride(StrideC_ref{}, cute::make_shape(options.m, options.n, options.l));
  // Reverse stride here due to swap and transpose
  stride_D = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(options.n, options.m, options.l));
  stride_D_ref = cutlass::make_cute_packed_stride(StrideD_ref{}, cute::make_shape(options.m, options.n, options.l));

  auto layout_B = make_layout(shape_B, stride_B);

  auto a_coord = cutlass::make_Coord(options.m * options.l, options.k);
  auto b_coord = cutlass::make_Coord(options.k, options.n * options.l);
  auto c_coord = cutlass::make_Coord(options.m * options.l, options.n);

  block_A.reset(a_coord.product());
  block_B.reset(b_coord.product());
  block_B_dq.reset(b_coord.product());
  block_C.reset(c_coord.product());
  block_D.reset(c_coord.product());
  block_ref_D.reset(c_coord.product());

  block_scale.reset(scale_k * options.l * options.n);
  block_zero.reset(scale_k * options.l * options.n);

  initialize_tensor(block_A, seed + 2022);
  initialize_tensor(block_B, seed + 2021);
  initialize_tensor(block_C, seed + 2020);
  initialize_scale(block_scale, options);
  initialize_zero(block_zero, options);

  auto shape_scale_zero = cute::make_shape(options.n, scale_k, options.l);
  stride_S = cutlass::make_cute_packed_stride(StrideS{}, cute::make_shape(options.n, scale_k, options.l));
  stride_S_ref = cutlass::make_cute_packed_stride(StrideS_ref{}, cute::make_shape(options.n, scale_k, options.l));
  auto layout_scale_zero = cute::make_layout(shape_scale_zero, stride_S_ref);

  cudaStream_t stream = cudaStreamDefault;
  cutlass::dequantize(block_B_dq.get(), block_B.get(), layout_B, block_scale.get(), block_zero.get(), layout_scale_zero, options.g, stream);

  if (options.shuffle) {
    // Repeat the reorder layout atom to tile the whole tensor shape 
    layout_B_reordered = cute::tile_to_shape(LayoutAtomQuant{}, shape_B);
    cutlass::reorder_tensor(block_B.get(), layout_B, layout_B_reordered);

    print("Quantized tensor layout: ");
    print(layout_B_reordered);
    print("\n");
  }
}
```

**EN**: `initialize()` computes the reversed C/D strides needed by the explicit swap, allocates all device buffers, fills A/B/C plus scale/zero data, and then calls `cutlass::dequantize()` to materialize a floating-point reference copy of B. If `shuffle` is enabled, it tiles `LayoutAtomQuant` over the full B tensor and physically reorders the quantized weights so each thread later reads contiguous packed values.

**CN**: `initialize()` 先计算因显式交换而需要反向设置的 C/D stride，然后分配全部设备缓冲区，填充 A/B/C 与 scale/zero 数据，并调用 `cutlass::dequantize()` 生成一个浮点参考版本的 B。若开启 `shuffle`，它会把 `LayoutAtomQuant` 铺满整个 B tensor，并实际重排量化权重，使后续每个线程都能读取连续打包的数据。

### Runtime argument construction with explicit operand swap (Lines 484-528)

```cpp
/// Populates a Gemm::Arguments structure from the given commandline options
/// Swap the A and B tensors, as well as problem shapes here.
template <typename Gemm>
typename Gemm::Arguments args_from_options(Options const& options)
{
  using Args = typename Gemm::Arguments;
  auto&& dB = [&]() {
    if constexpr (cute::is_same_v<Gemm, GemmConvertOnlyShuffled> ||
                  cute::is_same_v<Gemm, GemmScaleOnlyShuffled> ||
                  cute::is_same_v<Gemm, GemmScaleWithZeroPointShuffled>) {
      // offline swizzling is enabled.
      return layout_B_reordered;
    }
    else {
      return stride_B;
    }
  }();
  if (options.mode == MixedDtypeGemmMode::ConvertOnly) {
    return Args {
      cutlass::gemm::GemmUniversalMode::kGemm,
      {options.n, options.m, options.k, options.l},
      {block_B.get(), dB, block_A.get(), stride_A},
      {{options.alpha, options.beta}, block_C.get(), stride_C, block_D.get(), stride_D}
    };
  } 
  else if (options.mode == MixedDtypeGemmMode::ScaleOnly) {
    return Args {
      cutlass::gemm::GemmUniversalMode::kGemm,
      {options.n, options.m, options.k, options.l},
      {block_B.get(), dB, block_A.get(), stride_A, block_scale.get(), stride_S, options.g},
      {{options.alpha, options.beta}, block_C.get(), stride_C, block_D.get(), stride_D}
    };
  } 
  else if (options.mode == MixedDtypeGemmMode::ScaleWithZeroPoint) {
    return Args {
      cutlass::gemm::GemmUniversalMode::kGemm,
      {options.n, options.m, options.k, options.l},
      {block_B.get(), dB, block_A.get(), stride_A, block_scale.get(), stride_S, options.g, block_zero.get()},
      {{options.alpha, options.beta}, block_C.get(), stride_C, block_D.get(), stride_D}
    };
  } else {
    std::cerr << "Invalid mode " << options.mode << ". Must be 0, 1 or 2." << std::endl;
    exit(-1);
  }
}
```

**EN**: The templated `args_from_options()` function is where the explicit swap becomes concrete. For every GEMM mode it passes problem shape `{N, M, K, L}` instead of `{M, N, K, L}`, feeds `block_B` before `block_A`, and uses either the original B stride or the reordered B layout depending on the chosen kernel type. The scale and zero-point pointers are appended only in the modes whose collective types request them.

**CN**: 模板函数 `args_from_options()` 是“显式交换”真正落地的地方。无论哪种模式，它都会传入 `{N, M, K, L}` 而不是 `{M, N, K, L}` 的问题形状，并把 `block_B` 放在 `block_A` 前面；同时依据所选内核类型，决定使用原始 B stride 还是重排后的 B 布局。只有在 collective 类型要求时，才会把 scale 和 zero-point 指针附加到参数中。

### Reference verification through externally dequantized GEMM (Lines 530-584)

```cpp
bool verify(Options const& options) {
  //
  // Compute reference output
  //

  using CollectiveMainloopRef = typename cutlass::gemm::collective::CollectiveBuilder<
      ArchTag, OperatorClass,
      MmaType, LayoutA, AlignmentA,
      MmaType, LayoutB, AlignmentB,
      ElementAccumulator,
      TileShape, ClusterShape,
      cutlass::gemm::collective::StageCountAuto,
      cutlass::gemm::collective::KernelScheduleAuto
    >::CollectiveOp;

  using CollectiveEpilogueRef = typename cutlass::epilogue::collective::CollectiveBuilder<
      cutlass::arch::Sm90, cutlass::arch::OpClassTensorOp,
      TileShape, ClusterShape,
      cutlass::epilogue::collective::EpilogueTileAuto,
      ElementAccumulator, ElementAccumulator,
      ElementC, LayoutC, AlignmentC,
      ElementD, LayoutD, AlignmentD,
      cutlass::epilogue::NoSmemWarpSpecialized
    >::CollectiveOp;

  using GemmKernelRef = cutlass::gemm::kernel::GemmUniversal<
      Shape<int,int,int,int>, // Indicates ProblemShape
      CollectiveMainloopRef,
      CollectiveEpilogueRef
  >;

  using GemmRef = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelRef>;

  typename GemmRef::Arguments arguments{
    cutlass::gemm::GemmUniversalMode::kGemm,
    {options.m, options.n, options.k, options.l},
    {block_A.get(), stride_A, block_B_dq.get(), stride_B},
    {{options.alpha, options.beta}, block_C.get(), stride_C_ref, block_ref_D.get(), stride_D_ref}
  };

  // Run the gemm where the scaling is performed outside of the kernel.
  GemmRef gemm_ref;
  size_t workspace_size = GemmRef::get_workspace_size(arguments);
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
  CUTLASS_CHECK(gemm_ref.can_implement(arguments));
  CUTLASS_CHECK(gemm_ref.initialize(arguments, workspace.get()));
  CUTLASS_CHECK(gemm_ref.run());

  // compare_reference
  ElementD const epsilon(1e-2f);
  ElementD const non_zero_floor(1e-4f);
  bool passed = cutlass::reference::device::BlockCompareRelativelyEqual(block_ref_D.get(), block_D.get(), block_D.size(), epsilon, non_zero_floor);

  return passed;
}
```

**EN**: Verification intentionally avoids the mixed-input path. Instead it builds a standard same-type Hopper GEMM whose B operand is the already dequantized `block_B_dq`. This isolates correctness: if the mixed-input kernel matches this reference, then the builder-selected conversion/dequantization logic, scale application, and zero-point handling all behaved as intended.

**CN**: 正确性验证故意不再走 mixed-input 路径，而是构造一个标准的同类型 Hopper GEMM，并把已经反量化好的 `block_B_dq` 作为 B 操作数。这样一来，只要 mixed-input 内核的输出与该参考一致，就说明 builder 选择的转换/反量化逻辑、scale 应用和 zero-point 处理都是正确的。

### Kernel execution and profiling flow (Lines 586-623)

```cpp
/// Execute a given example GEMM computation
template <typename Gemm>
int run(Options &options)
{
  initialize(options);

  // Instantiate CUTLASS kernel depending on templates
  Gemm gemm;

  // Create a structure of gemm kernel arguments suitable for invoking an instance of Gemm
  auto arguments = args_from_options<Gemm>(options);

  // Using the arguments, query for extra workspace required for matrix multiplication computation
  size_t workspace_size = Gemm::get_workspace_size(arguments);

  // Allocate workspace memory
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

  // Check if the problem size is supported or not
  CUTLASS_CHECK(gemm.can_implement(arguments));

  // Initialize CUTLASS kernel with arguments and workspace pointer
  CUTLASS_CHECK(gemm.initialize(arguments, workspace.get()));

  // Correctness / Warmup iteration
  CUTLASS_CHECK(gemm.run());

  // Check if output from CUTLASS kernel and reference kernel are equal or not
  MixedDtypeResult result;
  result.passed = verify(options);
  mixed_dtype_profiling(gemm, options, result);
  std::cout << "  Disposition: " << (result.passed ? "Passed" : "Failed") << std::endl;
  if (!result.passed) {
    exit(-1);
  }

  return 0;
}
```

**EN**: `run()` is a generic driver shared by all kernel variants. It initializes tensors, instantiates the selected CUTLASS adapter, queries workspace, checks `can_implement()`, runs one correctness iteration, validates against the reference path, and then delegates timing to `mixed_dtype_profiling()`. The template parameter is therefore the only thing that changes between the six concrete kernel choices.

**CN**: `run()` 是所有内核变体共用的执行驱动：先初始化张量，再实例化选定的 CUTLASS adapter，查询工作区、检查 `can_implement()`、运行一次正确性迭代、与参考路径比对，然后把计时工作交给 `mixed_dtype_profiling()`。因此六个具体内核之间真正变化的只有模板参数本身。

### Architecture checks and mode dispatch in main (Lines 629-712)

```cpp
int main(int argc, char const **args) {

  // CUTLASS must be compiled with CUDA 12.0 Toolkit to run this example
  // and must have compute capability at least 90.
  if (__CUDACC_VER_MAJOR__ < 12) {
    std::cerr << "This example requires CUDA 12 or newer.\n";
    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
  }

  cudaDeviceProp props;
  int current_device_id;
  CUDA_CHECK(cudaGetDevice(&current_device_id));
  CUDA_CHECK(cudaGetDeviceProperties(&props, current_device_id));
  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (props.major != 9 || props.minor != 0) {
    std::cerr
      << "This example requires a GPU of NVIDIA's Hopper Architecture (compute capability 90).\n";
    return 0;
  }

  
  

  //
  // Parse options
  //

  Options options;

  options.parse(argc, args);

  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }

  //
  // Evaluate CUTLASS kernels
  //

#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
  if (options.mode == MixedDtypeGemmMode::ConvertOnly) {
    std::cout << "Running in no scale mode." << std::endl;
    if (options.shuffle) {
      std::cout << "Offline shuffle enabled." << std::endl;
      run<GemmConvertOnlyShuffled>(options);
    } else {
      std::cout << "Offline shuffle disabled." << std::endl;
      run<GemmConvertOnly>(options);
    }
  }
  else if (options.mode == MixedDtypeGemmMode::ScaleOnly) {
    if (options.g == options.k) {
      std::cout << "Running in per-column scale mode." << std::endl;
    } else {
      std::cout << "Running in group scale mode." << std::endl;
    }
    if (options.shuffle) {
      std::cout << "Offline shuffle enabled." << std::endl;
      run<GemmScaleOnlyShuffled>(options);
    } else {
      std::cout << "Offline shuffle disabled." << std::endl;
      run<GemmScaleOnly>(options);
    }
  }
  else if (options.mode == MixedDtypeGemmMode::ScaleWithZeroPoint) {
    if (options.g == options.k) {
      std::cout << "Running in per-column scale and zero mode." << std::endl;
    } else {
      std::cout << "Running in group scale and zero mode." << std::endl;
    }
    if (options.shuffle) {
      std::cout << "Offline shuffle enabled." << std::endl;
      run<GemmScaleWithZeroPointShuffled>(options);
    } else {
      std::cout << "Offline shuffle disabled." << std::endl;
      run<GemmScaleWithZeroPoint>(options);
    }
  }
#endif

  return 0;
}
```

**EN**: The entrypoint validates CUDA 12+ and SM90 hardware, parses the CLI, and dispatches to one of six instantiations based on `{mode, shuffle}`. The console messages clearly distinguish plain convert-only, per-column or group-wise scaling, and whether offline shuffle is enabled, making runtime behavior match the compile-time collective configuration chosen earlier.

**CN**: 入口函数会先检查 CUDA 12+ 和 SM90 硬件，再解析命令行，并依据 `{mode, shuffle}` 组合分发到六种实例化之一。控制台输出明确区分“仅转换”“按列或按组缩放”以及是否启用了离线 shuffle，使运行时行为与前面编译期选择的 collective 配置一一对应。

---

## Key Concepts / 关键概念

- **EN**: The central Hopper mixed-dtype trick is explicit swap + transpose: it avoids the builder’s implicit swap path so a TMA epilogue remains legal and fast.
  **CN**: Hopper 混合数据类型示例的核心技巧是“显式交换 + 转置”：它绕开 builder 的隐式交换路径，从而保留合法且高性能的 TMA epilogue。
- **EN**: Type tuples (`tuple<ElementB, ElementScale>` and `tuple<ElementB, ElementScale, ElementZero>`) are the compile-time signal that tells CUTLASS which dequantization features to fuse into the mainloop.
  **CN**: 类型元组（`tuple<ElementB, ElementScale>` 与 `tuple<ElementB, ElementScale, ElementZero>`）是编译期信号，用来告诉 CUTLASS 主循环应当融合哪些反量化能力。
- **EN**: Offline weight shuffle is a memory-layout optimization, not a math change: it rearranges packed INT4 values so each thread’s future accesses become more vectorizable.
  **CN**: 离线权重 shuffle 是内存布局优化，而不是数学变换：它重新排列打包后的 INT4 值，使每个线程未来的访问更适合向量化。
- **EN**: Verification dequantizes outside the kernel on purpose, providing a clean same-type GEMM reference for the fused mixed-input path.
  **CN**: 验证阶段特意在内核外完成反量化，从而为融合后的 mixed-input 路径提供一个干净的同类型 GEMM 参考。

## Dependencies / 依赖项

- **EN**: `cutlass/gemm/collective/collective_builder.hpp` and `cutlass/epilogue/collective/collective_builder.hpp` synthesize Hopper mainloop/epilogue collectives from the type-level configuration.
  **CN**: `cutlass/gemm/collective/collective_builder.hpp` 与 `cutlass/epilogue/collective/collective_builder.hpp` 会根据类型级配置合成 Hopper 的主循环和 epilogue collective。
- **EN**: `cutlass/util/mixed_dtype_utils.hpp` plus the local `mixed_dtype_utils.hpp` provide CLI parsing, tensor initialization, dequantization helpers, and profiling routines specific to mixed-input examples.
  **CN**: `cutlass/util/mixed_dtype_utils.hpp` 和本地 `mixed_dtype_utils.hpp` 提供了混合输入示例所需的命令行解析、张量初始化、反量化辅助函数以及性能测试逻辑。
- **EN**: `cute` layout utilities drive the reordered INT4 layout and the transposed stride calculations required by the explicit swap.
  **CN**: `cute` 的布局工具负责表达重排后的 INT4 布局，以及显式交换所需的转置 stride 计算。
- **EN**: Reference correctness depends on `cutlass::dequantize()` and a standard CUTLASS Hopper GEMM instantiated in `verify()`.
  **CN**: 参考正确性路径依赖 `cutlass::dequantize()`，以及 `verify()` 中实例化的标准 CUTLASS Hopper GEMM。
