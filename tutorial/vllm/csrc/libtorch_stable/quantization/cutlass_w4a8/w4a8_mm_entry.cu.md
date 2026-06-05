# w4a8_mm_entry.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/cutlass_w4a8/w4a8_mm_entry.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the single-matrix CUTLASS W4A8 GEMM path on Hopper, including dispatch heuristics and preprocessing utilities. / 实现 Hopper 上的单矩阵 CUTLASS W4A8 GEMM 路径，并包含调度启发式与预处理工具。

## Line-by-Line Analysis / 逐行分析
### Static configuration and layouts / 静态配置与布局定义
```cpp
using MmaType = cutlass::float_e4m3_t;  // A/scale element type
using QuantType = cutlass::int4b_t;     // B element type (packed int4)

static int constexpr TileShapeK = 128 * 8 / sizeof_bits<MmaType>::value;
static int constexpr ScalePackSize = 8;  // pack 8 scale elements together
static int constexpr PackFactor = 8;     // 8 4-bit packed into int32

// A matrix configuration
using ElementA = MmaType;                   // Element type for A matrix operand
using LayoutA = cutlass::layout::RowMajor;  // Layout type for A matrix operand
using LayoutA_Transpose =
    typename cutlass::layout::LayoutTranspose<LayoutA>::type;
constexpr int AlignmentA =
    128 / cutlass::sizeof_bits<
              ElementA>::value;  // Memory access granularity/alignment of A
                                 // matrix in units of elements (up to 16 bytes)
using StrideA = cutlass::detail::TagToStrideA_t<LayoutA>;

// B matrix configuration
using ElementB = QuantType;  // Element type for B matrix operand
using LayoutB =
    cutlass::layout::ColumnMajor;  // Layout type for B matrix operand
using LayoutB_Transpose =
    typename cutlass::layout::LayoutTranspose<LayoutB>::type;
constexpr int AlignmentB =
    128 / cutlass::sizeof_bits<
              ElementB>::value;  // Memory access granularity/alignment of B
                                 // matrix in units of elements (up to 16 bytes)
using StrideB = cutlass::detail::TagToStrideB_t<LayoutB>;

// Define the CuTe layout for reordered quantized tensor B
// LayoutAtomQuant places values that will be read by the same thread in
// contiguous locations in global memory. It specifies the reordering within a
// single warp's fragment
using LayoutAtomQuant =
    decltype(cutlass::compute_memory_reordering_atom<MmaType>());
using LayoutB_Reordered = decltype(cute::tile_to_shape(
    LayoutAtomQuant{}, Layout<Shape<int, int, int>, StrideB>{}));

// Group-wise scales
using ElementScale = MmaType;
using LayoutScale = cutlass::layout::RowMajor;
```
**EN:** This block defines the FP8 activation type, int4 weight type, pack factors, and runtime layout objects used to describe reordered B tensors and packed group scales.
**CN:** 该代码块定义了 FP8 激活类型、int4 权重类型、pack factor，以及用于描述重排后 B 张量与打包 group scale 的运行时布局对象。

### Kernel template composition / 内核模板组合
```cpp
template <class TileShape_MN, class ClusterShape_MNK>
struct W4A8GemmKernel {
  using TileShape =
      decltype(cute::append(TileShape_MN{}, cute::Int<TileShapeK>{}));
  using ClusterShape = ClusterShape_MNK;

  // Epilogue per-tok, per-chan scales
  using ChTokScalesEpilogue =
      typename vllm::c3x::ScaledEpilogue<ElementAccumulator, ElementD,
                                         TileShape>;
  using EVTCompute = typename ChTokScalesEpilogue::EVTCompute;
  using CollectiveEpilogue =
      typename cutlass::epilogue::collective::CollectiveBuilder<
          ArchTag, OperatorClass, TileShape, ClusterShape, EpilogueTileType,
          ElementAccumulator, ElementSChannel,
          // Transpose layout of D here since we use explicit swap + transpose
          // the void type for C tells the builder to allocate 0 smem for the C
          // matrix. We can enable this if beta == 0 by changing ElementC to
          // void below.
          ElementC, typename cutlass::layout::LayoutTranspose<LayoutC>::type,
          AlignmentC, ElementD,
          typename cutlass::layout::LayoutTranspose<LayoutD>::type, AlignmentD,
          EpilogueSchedule,  // This is the only epi supporting the required
                             // swap + transpose.
          EVTCompute>::CollectiveOp;

  // The Scale information must get paired with the operand that will be scaled.
  // In this example, B is scaled so we make a tuple of B's information and the
  // scale information.
  using CollectiveMainloopShuffled =
      typename cutlass::gemm::collective::CollectiveBuilder<
          ArchTag, OperatorClass,
          cute::tuple<ElementB, cutlass::Array<ElementScale, ScalePackSize>>,
          LayoutB_Reordered, AlignmentB, ElementA, LayoutA_Transpose,
          AlignmentA, ElementAccumulator, TileShape, ClusterShape,
          cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(
              sizeof(typename CollectiveEpilogue::SharedStorage))>,
          KernelSchedule>::CollectiveOp;

  using GemmKernelShuffled = cutlass::gemm::kernel::GemmUniversal<
      Shape<int, int, int, int>,  // Indicates ProblemShape
      CollectiveMainloopShuffled, CollectiveEpilogue>;
  using GemmShuffled =
      cutlass::gemm::device::GemmUniversalAdapter<GemmKernelShuffled>;
```
**EN:** `W4A8GemmKernel` assembles a CUTLASS collective epilogue for token/channel scaling and a mainloop that reads reordered int4 weights paired with packed FP8 scales.
**CN:** `W4A8GemmKernel` 组装了一个负责 token/channel 缩放的 CUTLASS collective epilogue，以及一个读取“重排 int4 权重 + 打包 FP8 scale”组合的 mainloop。

### Building arguments and running GEMM / 构造参数并执行 GEMM
```cpp
  static torch::stable::Tensor mm(
      torch::stable::Tensor const& A,
      torch::stable::Tensor const& B,             // already packed
      torch::stable::Tensor const& group_scales,  // already packed
      int64_t group_size, torch::stable::Tensor const& channel_scales,
      torch::stable::Tensor const& token_scales,
      std::optional<torch::headeronly::ScalarType> const& maybe_out_type) {
    // TODO: param validation
    int m = A.size(0);
    int k = A.size(1);
    int n = B.size(1);

    // safely cast group_size to int
    STD_TORCH_CHECK(
        group_size > 0 && group_size <= std::numeric_limits<int>::max(),
        "group_size out of supported range for int: ", group_size);
    int const group_size_int = static_cast<int>(group_size);

    // Allocate output
    const torch::stable::accelerator::DeviceGuard device_guard(
        A.get_device_index());
    auto device = A.device();
    auto stream = get_current_cuda_stream(device.index());
    torch::stable::Tensor D = torch::stable::empty(
        {m, n}, equivalent_scalar_type_v<ElementD>, std::nullopt, device);
    // prepare arg pointers
    auto A_ptr = static_cast<MmaType const*>(A.const_data_ptr());
    auto B_ptr = static_cast<QuantType const*>(B.const_data_ptr());
    auto D_ptr = static_cast<ElementD*>(D.data_ptr());
    // can we avoid hardcode the 8 here
    auto S_ptr =
        static_cast<cutlass::Array<ElementScale, ScalePackSize> const*>(
            group_scales.const_data_ptr());

    // runtime layout for B
    auto shape_B = cute::make_shape(n, k, 1);
    LayoutB_Reordered layout_B_reordered =
        cute::tile_to_shape(LayoutAtomQuant{}, shape_B);

    // strides
    int const scale_k = cutlass::ceil_div(k, group_size_int);
    StrideA stride_A =
        cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(m, k, 1));
    // Reverse stride here due to swap and transpose
    StrideD stride_D =
        cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(n, m, 1));
    StrideS stride_S = cutlass::make_cute_packed_stride(
        StrideS{}, cute::make_shape(n, scale_k, 1));

    // Create a structure of gemm kernel arguments suitable for invoking an
    // instance of Gemm auto arguments =
    // args_from_options<GemmShuffled>(options);
    /// Populates a Gemm::Arguments structure from the given arguments
    /// Swap the A and B tensors, as well as problem shapes here.
    using Args = typename GemmShuffled::Arguments;
    using MainloopArguments = typename GemmKernelShuffled::MainloopArguments;
    using EpilogueArguments = typename GemmKernelShuffled::EpilogueArguments;

    MainloopArguments mainloop_arguments{
        B_ptr, layout_B_reordered, A_ptr,         stride_A,
        S_ptr, stride_S,           group_size_int};

    EpilogueArguments epilogue_arguments{
        ChTokScalesEpilogue::prepare_args(channel_scales, token_scales),
        nullptr,
        {},  // no C
        D_ptr,
        stride_D};

    Args arguments{cutlass::gemm::GemmUniversalMode::kGemm,
                   {n, m, k, 1},  // shape
                   mainloop_arguments,
                   epilogue_arguments};

    // Workspace
    size_t workspace_size = GemmShuffled::get_workspace_size(arguments);
    torch::stable::Tensor workspace = torch::stable::empty(
        workspace_size, torch::headeronly::ScalarType::Byte, std::nullopt,
        device);

    // Run GEMM
    GemmShuffled gemm;
    CUTLASS_CHECK(gemm.can_implement(arguments));
    CUTLASS_CHECK(gemm.initialize(arguments, workspace.data_ptr(), stream));
    CUTLASS_CHECK(gemm.run(stream));

    return D;
```
**EN:** The `mm` method allocates the output tensor, creates runtime strides and layouts, builds mainloop/epilogue arguments, allocates workspace, and launches the CUTLASS adapter on the current stream.
**CN:** `mm` 方法会分配输出张量，构造运行时 stride 与布局，创建 mainloop/epilogue 参数，分配工作区，并在当前 stream 上启动 CUTLASS 适配器。

### Named schedule dispatch / 命名 schedule 分发
```cpp
torch::stable::Tensor mm_dispatch(
    torch::stable::Tensor const& A,
    torch::stable::Tensor const& B,             // already packed
    torch::stable::Tensor const& group_scales,  // already packed
    int64_t group_size, torch::stable::Tensor const& channel_scales,
    torch::stable::Tensor const& token_scales,
    std::optional<torch::headeronly::ScalarType> const& maybe_out_type,
    const std::string& schedule) {
  if (schedule == "256x128_1x1x1") {
    return Kernel_256x128_1x1x1::mm(A, B, group_scales, group_size,
                                    channel_scales, token_scales,
                                    maybe_out_type);
  } else if (schedule == "256x64_1x1x1") {
    return Kernel_256x64_1x1x1::mm(A, B, group_scales, group_size,
                                   channel_scales, token_scales,
                                   maybe_out_type);
  } else if (schedule == "256x32_1x1x1") {
    return Kernel_256x32_1x1x1::mm(A, B, group_scales, group_size,
                                   channel_scales, token_scales,
                                   maybe_out_type);
  } else if (schedule == "256x16_1x1x1") {
    return Kernel_256x16_1x1x1::mm(A, B, group_scales, group_size,
                                   channel_scales, token_scales,
                                   maybe_out_type);
  } else if (schedule == "128x256_2x1x1") {
    return Kernel_128x256_2x1x1::mm(A, B, group_scales, group_size,
                                    channel_scales, token_scales,
                                    maybe_out_type);
  } else if (schedule == "128x256_1x1x1") {
    return Kernel_128x256_1x1x1::mm(A, B, group_scales, group_size,
                                    channel_scales, token_scales,
                                    maybe_out_type);
  } else if (schedule == "128x128_1x1x1") {
    return Kernel_128x128_1x1x1::mm(A, B, group_scales, group_size,
                                    channel_scales, token_scales,
                                    maybe_out_type);
  } else if (schedule == "128x64_1x1x1") {
    return Kernel_128x64_1x1x1::mm(A, B, group_scales, group_size,
                                   channel_scales, token_scales,
                                   maybe_out_type);
  } else if (schedule == "128x32_1x1x1") {
    return Kernel_128x32_1x1x1::mm(A, B, group_scales, group_size,
                                   channel_scales, token_scales,
                                   maybe_out_type);
  } else if (schedule == "128x16_1x1x1") {
    return Kernel_128x16_1x1x1::mm(A, B, group_scales, group_size,
                                   channel_scales, token_scales,
                                   maybe_out_type);
  }
  STD_TORCH_CHECK(false, "Unknown W4A8 schedule: ", schedule);
  return {};
```
**EN:** `mm_dispatch` converts a schedule string into one concrete tile/cluster instantiation. This provides an explicit override path for benchmarking or tuning.
**CN:** `mm_dispatch` 会把 schedule 字符串转换成一个具体的 tile/cluster 实例，便于做基准测试或手动调优。

### Heuristic kernel selection / 启发式内核选择
```cpp
torch::stable::Tensor mm(
    torch::stable::Tensor const& A,
    torch::stable::Tensor const& B,             // already packed
    torch::stable::Tensor const& group_scales,  // already packed
    int64_t group_size, torch::stable::Tensor const& channel_scales,
    torch::stable::Tensor const& token_scales,
    std::optional<torch::headeronly::ScalarType> const& maybe_out_type,
    std::optional<std::string> maybe_schedule) {
  // requested a specific schedule
  if (maybe_schedule) {
    return mm_dispatch(A, B, group_scales, group_size, channel_scales,
                       token_scales, maybe_out_type, *maybe_schedule);
  }
  std::string schedule;
  int M = A.size(0);
  int K = A.size(1);
  int N = B.size(1);
  // heuristic
  if (M <= 16) {
    schedule = (K == 16384 && N == 18432) ? "256x16_1x1x1" : "128x16_1x1x1";
  } else if (M <= 32) {
    schedule = (K == 16384 && N == 18432) ? "256x32_1x1x1" : "128x32_1x1x1";
  } else if (M <= 64) {
    if (K == 16384 && N == 18432)
      schedule = "256x64_1x1x1";
    else if (N <= 8192 && K <= 8192)
      schedule = "128x32_1x1x1";
    else
      schedule = "128x64_1x1x1";
  } else if (M <= 128) {
    if (K == 16384 && N == 18432)
      schedule = "256x128_1x1x1";
    else if (N <= 8192)
      schedule = "128x64_1x1x1";
    else
      schedule = "128x128_1x1x1";
  } else if (M <= 256) {
    if (N <= 4096)
      schedule = "128x64_1x1x1";
    else if (N <= 8192)
      schedule = "128x128_1x1x1";
    else
      schedule = "128x256_1x1x1";
  } else if (M <= 512 && N <= 4096) {
    schedule = "128x128_1x1x1";
  } else if (M <= 1024) {
    schedule = "128x256_1x1x1";
  } else {
    schedule = "128x256_2x1x1";
  }
  return mm_dispatch(A, B, group_scales, group_size, channel_scales,
                     token_scales, maybe_out_type, schedule);
}
```
**EN:** If no schedule is given, the file picks one from `(M, K, N)`. The heuristic prefers smaller tiles for tiny batches, switches to special 256-wide kernels for a known large shape, and uses wider kernels as M grows.
**CN:** 如果没有显式 schedule，代码会根据 `(M, K, N)` 选择方案。该启发式会为小批次选择较小 tile，对特定大形状切换到 256 宽内核，并在 M 增大时逐步使用更宽内核。

### Scale packing and weight reordering / scale 打包与权重重排
```cpp
torch::stable::Tensor pack_scale_fp8(torch::stable::Tensor const& scales) {
  STD_TORCH_CHECK(scales.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
  STD_TORCH_CHECK(scales.is_contiguous());
  STD_TORCH_CHECK(scales.is_cuda());

  auto packed_scales =
      torch::stable::empty({scales.numel() * ScalePackSize},
                           scales.scalar_type(), std::nullopt, scales.device());
  auto scales_ptr = static_cast<MmaType const*>(scales.const_data_ptr());
  auto packed_scales_ptr =
      static_cast<cutlass::Array<ElementScale, ScalePackSize>*>(
          packed_scales.data_ptr());

  cutlass::pack_scale_fp8(scales_ptr, packed_scales_ptr, scales.numel());

  return packed_scales;
}

torch::stable::Tensor encode_and_reorder_int4b(torch::stable::Tensor const& B) {
  STD_TORCH_CHECK(B.scalar_type() == torch::headeronly::ScalarType::Int);
  STD_TORCH_CHECK(B.dim() == 2);

  torch::stable::Tensor B_packed = torch::stable::empty_like(B);

  int k = B.size(0) * PackFactor;  // logical k
  int n = B.size(1);
  STD_TORCH_CHECK((n * k) % 32 == 0,
                  "need multiples of 32 int4s for 16B chunks");

  auto B_ptr = static_cast<QuantType const*>(B.const_data_ptr());
  auto B_packed_ptr = static_cast<QuantType*>(B_packed.data_ptr());
  auto shape_B = cute::make_shape(n, k, 1);
  auto layout_B = make_layout(shape_B, LayoutRight{});  // row major
  LayoutB_Reordered layout_B_reordered =
      cute::tile_to_shape(LayoutAtomQuant{}, shape_B);

  bool ok = vllm::cutlass_w4a8_utils::unified_encode_int4b(B_ptr, B_packed_ptr,
                                                           n * k);
  STD_TORCH_CHECK(ok, "unified_encode_int4b failed");
  cutlass::reorder_tensor(B_packed_ptr, layout_B, layout_B_reordered);

  return B_packed;
```
**EN:** `pack_scale_fp8` reshapes raw FP8 scale values into CUTLASS arrays of eight elements, while `encode_and_reorder_int4b` performs numeric remapping plus layout reordering for packed int4 weights.
**CN:** `pack_scale_fp8` 会把原始 FP8 scale 重新打包成 CUTLASS 需要的 8 元数组，而 `encode_and_reorder_int4b` 则负责完成 packed int4 权重的数值重映射和布局重排。

### Torch operator registration / Torch 算子注册
```cpp
STABLE_TORCH_LIBRARY_IMPL(_C, CUDA, m) {
  m.impl("cutlass_w4a8_mm", TORCH_BOX(&mm));
  m.impl("cutlass_pack_scale_fp8", TORCH_BOX(&pack_scale_fp8));
  m.impl("cutlass_encode_and_reorder_int4b",
         TORCH_BOX(&encode_and_reorder_int4b));
```
**EN:** The file exports the main W4A8 GEMM, FP8 scale packing, and int4 encode/reorder helpers as stable Torch CUDA operators.
**CN:** 该文件将主 W4A8 GEMM、FP8 scale 打包函数以及 int4 编码/重排辅助函数导出为 stable Torch CUDA 算子。

## Key Concepts / 关键概念
- Hopper-specific CUTLASS kernels combine FP8 activations with int4 weights / 面向 Hopper 的 CUTLASS 内核将 FP8 激活与 int4 权重结合起来
- Runtime schedule heuristics trade tile size against matrix shape / 运行时 schedule 启发式会根据矩阵形状权衡 tile 大小
- Preprocessing is mandatory because CUTLASS expects reordered int4 storage and packed scales / 预处理是必需的，因为 CUTLASS 期望重排后的 int4 存储与打包 scale

## Dependencies / 依赖关系
- Depends on `w4a8_utils.cuh`, CUTLASS/CuTe, and stable Torch CUDA bindings / 依赖 `w4a8_utils.cuh`、CUTLASS/CuTe 与 stable Torch CUDA 绑定
- Calls CUTLASS helpers such as `pack_scale_fp8` and `reorder_tensor` / 调用 `pack_scale_fp8`、`reorder_tensor` 等 CUTLASS 辅助函数
