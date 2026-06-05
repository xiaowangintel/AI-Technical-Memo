# w4a8_grouped_mm_entry.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/cutlass_w4a8/w4a8_grouped_mm_entry.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides the CUTLASS SM90 grouped W4A8 MoE GEMM entry point, dispatch logic, and int4 preprocessing helpers. / 提供基于 CUTLASS SM90 的 grouped W4A8 MoE GEMM 入口、调度逻辑以及 int4 预处理辅助函数。

## Line-by-Line Analysis / 逐行分析
### Static CUTLASS configuration / 静态 CUTLASS 配置
```cpp
using ProblemShape =
    cutlass::gemm::GroupProblemShape<Shape<int, int, int>>;  // <M,N,K> per
                                                             // group
using MmaType = cutlass::float_e4m3_t;
using QuantType = cutlass::int4b_t;

constexpr int TileShapeK = 128 * 8 / sizeof_bits<MmaType>::value;
static int constexpr PackFactor = 8;  // 8 int4 packed into int32

// A matrix configuration
using ElementA = MmaType;
using LayoutA = cutlass::layout::RowMajor;  // Layout type for A matrix operand
constexpr int AlignmentA =
    128 /
    cutlass::sizeof_bits<ElementA>::value;  // Alignment of A matrix in units of
                                            // elements (up to 16 bytes)

// B matrix configuration
using ElementB = QuantType;  // Element type for B matrix operand
using LayoutB =
    cutlass::layout::ColumnMajor;  // Layout type for B matrix operand
constexpr int AlignmentB =
    128 / cutlass::sizeof_bits<
              ElementB>::value;  // Memory access granularity/alignment of B
                                 // matrix in units of elements (up to 16 bytes)

// This example manually swaps and transposes, so keep transpose of input
// layouts
using LayoutA_Transpose =
    typename cutlass::layout::LayoutTranspose<LayoutA>::type;
using LayoutB_Transpose =
    typename cutlass::layout::LayoutTranspose<LayoutB>::type;

// Need to pass a pointer type to make the 3rd dimension of Stride be _0
using StrideA =
    cute::remove_pointer_t<cutlass::detail::TagToStrideA_t<LayoutA*>>;
using StrideB =
    cute::remove_pointer_t<cutlass::detail::TagToStrideB_t<LayoutB*>>;

// Define the CuTe layout for reoredered quantized tensor B
// LayoutAtomQuant places values that will be read by the same thread in
// contiguous locations in global memory. It specifies the reordering within a
// single warp's fragment
using LayoutAtomQuant =
    decltype(cutlass::compute_memory_reordering_atom<MmaType>());
using LayoutB_Reordered = decltype(cute::tile_to_shape(
    LayoutAtomQuant{}, Layout<Shape<int, int, Int<1>>, StrideB>{}));
```
**EN:** The file defines grouped problem shapes, operand layouts, alignment rules, and the reordered layout used for quantized B. This metadata is shared by every grouped kernel instantiation.
**CN:** 该文件定义了 grouped 问题形状、操作数布局、对齐规则，以及量化 B 使用的重排布局。这些元数据由所有 grouped 内核实例共享。

### Kernel template builds collectives / 内核模板构建 collective 组件
```cpp
template <class TileShape_MN, class ClusterShape_MNK, class KernelSchedule,
          class EpilogueSchedule>
struct W4A8GroupedGemmKernel {
  using TileShape =
      decltype(cute::append(TileShape_MN{}, cute::Int<TileShapeK>{}));
  using ClusterShape = ClusterShape_MNK;

  // per-channel, per-token scales epilogue
  using ChTokScalesEpilogue =
      typename vllm::c3x::ScaledEpilogueArray<ElementAccumulator, ElementD,
                                              TileShape>;
  using EVTCompute = typename ChTokScalesEpilogue::EVTCompute;
  using CollectiveEpilogue =
      typename cutlass::epilogue::collective::CollectiveBuilder<
          ArchTag, OperatorClass, TileShape, ClusterShape,
          cutlass::epilogue::collective::EpilogueTileAuto, ElementAccumulator,
          ElementSChannel, ElementC,
          typename cutlass::layout::LayoutTranspose<LayoutC>::type*, AlignmentC,
          ElementD, typename cutlass::layout::LayoutTranspose<LayoutD>::type*,
          AlignmentD, EpilogueSchedule, EVTCompute>::CollectiveOp;

  // =========================================================== MIXED INPUT
  // WITH SCALES
  // ===========================================================================
  // The Scale information must get paired with the operand that will be scaled.
  // In this example, B is scaled so we make a tuple of B's information and the
  // scale information.
  using CollectiveMainloopShuffled =
      typename cutlass::gemm::collective::CollectiveBuilder<
          ArchTag, OperatorClass,
          cute::tuple<ElementB, cutlass::Array<ElementScale, 8>>,
          LayoutB_Reordered*, AlignmentB, ElementA, LayoutA_Transpose*,
          AlignmentA, ElementAccumulator, TileShape, ClusterShape,
          cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(
              sizeof(typename CollectiveEpilogue::SharedStorage))>,
          KernelSchedule>::CollectiveOp;

  using GemmKernelShuffled = cutlass::gemm::kernel::GemmUniversal<
      ProblemShape, CollectiveMainloopShuffled, CollectiveEpilogue>;

  using GemmShuffled =
      cutlass::gemm::device::GemmUniversalAdapter<GemmKernelShuffled>;
```
**EN:** `W4A8GroupedGemmKernel` binds together the CUTLASS mainloop and epilogue builders. It pairs int4 weights with packed group scales and uses a scaled epilogue to apply per-channel and per-token scaling.
**CN:** `W4A8GroupedGemmKernel` 把 CUTLASS 的 mainloop 与 epilogue builder 组合起来。它将 int4 权重与打包后的 group scale 配对，并通过带缩放的 epilogue 应用 per-channel 与 per-token 缩放。

### Preparing grouped GEMM arguments / 构造 grouped GEMM 参数
```cpp
  static void grouped_mm(torch::stable::Tensor& out_tensors,
                         const torch::stable::Tensor& a_tensors,
                         const torch::stable::Tensor& b_tensors,
                         const torch::stable::Tensor& a_scales,
                         const torch::stable::Tensor& b_scales,
                         const torch::stable::Tensor& b_group_scales,
                         const int64_t b_group_size,
                         const torch::stable::Tensor& expert_offsets,
                         const torch::stable::Tensor& problem_sizes_torch,
                         const torch::stable::Tensor& a_strides,
                         const torch::stable::Tensor& b_strides,
                         const torch::stable::Tensor& c_strides,
                         const torch::stable::Tensor& group_scale_strides) {
    auto device = a_tensors.device();
    auto device_id = device.index();
    const torch::stable::accelerator::DeviceGuard device_guard(device_id);
    auto stream = get_current_cuda_stream(device_id);

    int num_experts = static_cast<int>(expert_offsets.size(0));
    int n = static_cast<int>(b_tensors.size(1));
    int k = static_cast<int>(b_tensors.size(2)) * PackFactor;

    torch::stable::Tensor a_ptrs = torch::stable::empty(
        num_experts, torch::headeronly::ScalarType::Long, std::nullopt, device);
    torch::stable::Tensor b_ptrs = torch::stable::empty(
        num_experts, torch::headeronly::ScalarType::Long, std::nullopt, device);
    torch::stable::Tensor out_ptrs = torch::stable::empty(
        num_experts, torch::headeronly::ScalarType::Long, std::nullopt, device);
    torch::stable::Tensor a_scales_ptrs = torch::stable::empty(
        num_experts, torch::headeronly::ScalarType::Long, std::nullopt, device);
    torch::stable::Tensor b_scales_ptrs = torch::stable::empty(
        num_experts, torch::headeronly::ScalarType::Long, std::nullopt, device);
    torch::stable::Tensor b_group_scales_ptrs = torch::stable::empty(
        num_experts, torch::headeronly::ScalarType::Long, std::nullopt, device);

    // get the correct offsets to pass to gemm
    run_get_group_gemm_starts(expert_offsets, a_ptrs, b_ptrs, out_ptrs,
                              a_scales_ptrs, b_scales_ptrs, b_group_scales_ptrs,
                              a_tensors, b_tensors, out_tensors, a_scales,
                              b_scales, b_group_scales, b_group_size);

    // construct args
    using Args = typename GemmShuffled::Arguments;
    using MainloopArguments = typename GemmKernelShuffled::MainloopArguments;
    using EpilogueArguments = typename GemmKernelShuffled::EpilogueArguments;
    Args arguments;

    ProblemShape::UnderlyingProblemShape* problem_sizes_as_shapes =
        static_cast<ProblemShape::UnderlyingProblemShape*>(
            problem_sizes_torch.data_ptr());
    ProblemShape prob_shape{num_experts, problem_sizes_as_shapes, nullptr};

    // SwapAB so B operands come first
    MainloopArguments mainloop_arguments{
        static_cast<const QuantType**>(b_ptrs.data_ptr()),
        static_cast<LayoutB_Reordered*>(b_strides.data_ptr()),
        static_cast<const MmaType**>(a_ptrs.data_ptr()),
        static_cast<StrideA*>(a_strides.data_ptr()),
        static_cast<const cutlass::Array<ElementScale, 8>**>(
            b_group_scales_ptrs.data_ptr()),
        static_cast<StrideS*>(group_scale_strides.data_ptr()),
        static_cast<int>(b_group_size)};
```
**EN:** The `grouped_mm` method allocates pointer arrays, calls `run_get_group_gemm_starts` to fill them, converts the problem-size tensor to CUTLASS shapes, and prepares the mainloop arguments with swapped A/B ordering.
**CN:** `grouped_mm` 方法会分配指针数组，调用 `run_get_group_gemm_starts` 填充它们，把 problem-size 张量转换为 CUTLASS 形状，并按照交换后的 A/B 顺序构造 mainloop 参数。

### Epilogue args, workspace, and run / Epilogue 参数、工作区与执行
```cpp
    EpilogueArguments epilogue_arguments{
        // since we are doing SwapAB the channel scales comes first, then token
        // scales
        ChTokScalesEpilogue::prepare_args(  // see ScaledEpilogueArray
            static_cast<const ElementAccumulator**>(
                b_scales_ptrs.data_ptr()),  // per-channel
            static_cast<const ElementAccumulator**>(
                a_scales_ptrs.data_ptr()),  // per-token
            true, true),
        nullptr,                                       // C
        static_cast<StrideC*>(c_strides.data_ptr()),   // C
        static_cast<ElementD**>(out_ptrs.data_ptr()),  // D
        static_cast<StrideC*>(c_strides.data_ptr())    // D
    };

    static const cutlass::KernelHardwareInfo hw_info{
        device_id,
        cutlass::KernelHardwareInfo::query_device_multiprocessor_count(
            device_id)};

    arguments = Args{cutlass::gemm::GemmUniversalMode::kGrouped, prob_shape,
                     mainloop_arguments, epilogue_arguments, hw_info};

    // Allocate workspace
    size_t workspace_size = GemmShuffled::get_workspace_size(arguments);
    torch::stable::Tensor workspace = torch::stable::empty(
        workspace_size, torch::headeronly::ScalarType::Byte, std::nullopt,
        device);

    // Run GEMM
    GemmShuffled gemm;
    CUTLASS_CHECK(gemm.can_implement(arguments));
    CUTLASS_CHECK(gemm.initialize(arguments, workspace.data_ptr(), stream));
    CUTLASS_CHECK(gemm.run(stream));
```
**EN:** The epilogue arguments point to channel and token scales, while the CUTLASS adapter computes workspace size, initializes the kernel, and launches it on the active stream.
**CN:** epilogue 参数指向 channel scale 与 token scale，而 CUTLASS 适配器负责计算工作区大小、初始化内核，并在当前 stream 上启动执行。

### Explicit schedule dispatch / 显式 schedule 分发
```cpp
void mm_dispatch(torch::stable::Tensor& out_tensors,
                 const torch::stable::Tensor& a_tensors,
                 const torch::stable::Tensor& b_tensors,
                 const torch::stable::Tensor& a_scales,
                 const torch::stable::Tensor& b_scales,
                 const torch::stable::Tensor& b_group_scales,
                 const int64_t b_group_size,
                 const torch::stable::Tensor& expert_offsets,
                 const torch::stable::Tensor& problem_sizes,
                 const torch::stable::Tensor& a_strides,
                 const torch::stable::Tensor& b_strides,
                 const torch::stable::Tensor& c_strides,
                 const torch::stable::Tensor& group_scale_strides,
                 const std::string& schedule) {
  if (schedule == "Kernel_128x16_1x1x1_Coop") {
    Kernel_128x16_1x1x1_Coop::grouped_mm(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, b_group_scales,
        b_group_size, expert_offsets, problem_sizes, a_strides, b_strides,
        c_strides, group_scale_strides);
  } else if (schedule == "Kernel_128x16_2x1x1_Coop") {
    Kernel_128x16_2x1x1_Coop::grouped_mm(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, b_group_scales,
        b_group_size, expert_offsets, problem_sizes, a_strides, b_strides,
        c_strides, group_scale_strides);
  } else if (schedule == "Kernel_256x16_1x1x1_Coop") {
    Kernel_256x16_1x1x1_Coop::grouped_mm(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, b_group_scales,
        b_group_size, expert_offsets, problem_sizes, a_strides, b_strides,
        c_strides, group_scale_strides);
  } else if (schedule == "Kernel_256x16_2x1x1_Coop") {
    Kernel_256x16_2x1x1_Coop::grouped_mm(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, b_group_scales,
        b_group_size, expert_offsets, problem_sizes, a_strides, b_strides,
        c_strides, group_scale_strides);
  } else if (schedule == "Kernel_256x32_1x1x1_Coop") {
    Kernel_256x32_1x1x1_Coop::grouped_mm(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, b_group_scales,
        b_group_size, expert_offsets, problem_sizes, a_strides, b_strides,
        c_strides, group_scale_strides);
  } else if (schedule == "Kernel_256x32_2x1x1_Coop") {
    Kernel_256x32_2x1x1_Coop::grouped_mm(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, b_group_scales,
        b_group_size, expert_offsets, problem_sizes, a_strides, b_strides,
        c_strides, group_scale_strides);
  } else if (schedule == "Kernel_256x64_1x1x1_Coop") {
    Kernel_256x64_1x1x1_Coop::grouped_mm(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, b_group_scales,
        b_group_size, expert_offsets, problem_sizes, a_strides, b_strides,
        c_strides, group_scale_strides);
  } else if (schedule == "Kernel_256x64_2x1x1_Coop") {
    Kernel_256x64_2x1x1_Coop::grouped_mm(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, b_group_scales,
        b_group_size, expert_offsets, problem_sizes, a_strides, b_strides,
        c_strides, group_scale_strides);
  } else if (schedule == "Kernel_256x128_1x1x1_Coop") {
    Kernel_256x128_1x1x1_Coop::grouped_mm(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, b_group_scales,
        b_group_size, expert_offsets, problem_sizes, a_strides, b_strides,
        c_strides, group_scale_strides);
  } else if (schedule == "Kernel_256x128_2x1x1_Coop") {
    Kernel_256x128_2x1x1_Coop::grouped_mm(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, b_group_scales,
        b_group_size, expert_offsets, problem_sizes, a_strides, b_strides,
        c_strides, group_scale_strides);
  } else if (schedule == "Kernel_128x256_2x1x1_Coop") {
    Kernel_128x256_2x1x1_Coop::grouped_mm(
        out_tensors, a_tensors, b_tensors, a_scales, b_scales, b_group_scales,
        b_group_size, expert_offsets, problem_sizes, a_strides, b_strides,
        c_strides, group_scale_strides);
  } else {
    STD_TORCH_CHECK(false,
                    "cutlass_w4a8_moe_mm: unknown schedule string: ", schedule);
  }
}
```
**EN:** `mm_dispatch` maps a schedule string to one concrete grouped kernel instantiation. This keeps Python or higher-level code free to request a specific tile/cluster configuration.
**CN:** `mm_dispatch` 会把 schedule 字符串映射到某个具体的 grouped 内核实例上，使 Python 或上层代码可以主动指定 tile/cluster 配置。

### Heuristic schedule selection / 启发式 schedule 选择
```cpp
  // use heuristic
  int m_full = a_tensors.size(0);
  int n = b_tensors.size(1);
  int k = b_tensors.size(2) * PackFactor;  // logical k
  int num_experts = b_tensors.size(0);
  // per-expert batch size assuming uniform distribution
  int m_expert = m_full / num_experts;

  std::string schedule;
  if (m_expert <= 16) {
    schedule = "Kernel_128x16_2x1x1_Coop";
  } else if (m_expert <= 32) {
    schedule = "Kernel_256x32_1x1x1_Coop";
  } else if (m_expert <= 64) {
    schedule = "Kernel_256x64_1x1x1_Coop";
  } else if (m_expert <= 128) {
    schedule = "Kernel_256x128_2x1x1_Coop";
  } else {  // m_expert > 128
    schedule = "Kernel_128x256_2x1x1_Coop";
  }

  mm_dispatch(out_tensors, a_tensors, b_tensors, a_scales, b_scales,
              b_group_scales, b_group_size, expert_offsets, problem_sizes,
              a_strides, b_strides, c_strides, group_scale_strides, schedule);
```
**EN:** When the caller does not specify a schedule, the file chooses one from the average per-expert token count `m_expert`. Small expert batches prefer narrow tiles, while larger ones choose wider kernels.
**CN:** 当调用方没有指定 schedule 时，代码会根据平均每个 expert 的 token 数 `m_expert` 选择方案。较小批次偏向窄 tile，较大批次则选择更宽的内核。

### Encoding and reordering int4 weights / 编码并重排 int4 权重
```cpp
std::tuple<torch::stable::Tensor, torch::stable::Tensor>
encode_and_reorder_int4b(torch::stable::Tensor const& b_tensors) {
  STD_TORCH_CHECK(b_tensors.scalar_type() ==
                  torch::headeronly::ScalarType::Int);
  STD_TORCH_CHECK(b_tensors.dim() == 3);  // (experts, n, k)
  STD_TORCH_CHECK(b_tensors.is_contiguous());
  STD_TORCH_CHECK(b_tensors.is_cuda());

  int n = static_cast<int>(b_tensors.size(1));
  int k = static_cast<int>(b_tensors.size(2)) * PackFactor;  // logical k

  // CUTLASS reorder_tensor requires k % 256 == 0 and n % 16 == 0.
  // These misalignments cause silent OOB unless run under Compute Sanitizer.
  STD_TORCH_CHECK(k % 256 == 0, "logical k must be divisible by 256");
  STD_TORCH_CHECK(n % 16 == 0, "n must be divisible by 16");

  // we will store the layout to an int32 tensor;
  // this is the number of elements we need per layout
  constexpr size_t layout_width = sizeof(LayoutB_Reordered) / sizeof(int32_t);

  torch::stable::Tensor b_tensors_packed = torch::stable::empty_like(b_tensors);
  int num_experts = static_cast<int>(b_tensors.size(0));

  auto b_ptr = static_cast<QuantType const*>(b_tensors.const_data_ptr());
  auto b_packed_ptr = static_cast<QuantType*>(b_tensors_packed.data_ptr());

  // multiply by ull so result does not overflow int32
  size_t num_int4_elems = 1ull * num_experts * n * k;
  bool ok = vllm::cutlass_w4a8_utils::unified_encode_int4b(b_ptr, b_packed_ptr,
                                                           num_int4_elems);
  STD_TORCH_CHECK(ok, "unified_encode_int4b failed");

  // construct the layout once; assumes each expert has the same layout
  using LayoutType = LayoutB_Reordered;
  std::vector<LayoutType> layout_B_reordered_host(num_experts);
  auto stride_B = cutlass::make_cute_packed_stride(StrideB{}, {n, k, Int<1>{}});
  auto shape_B = cute::make_shape(n, k, Int<1>{});
  auto layout_B = make_layout(shape_B, stride_B);
  LayoutType layout_B_reordered = tile_to_shape(LayoutAtomQuant{}, shape_B);

  // reorder weights for each expert
  for (int i = 0; i < num_experts; i++) {
    // since the storage type of int4b is 1 byte but one element is 4 bits
    // we need to adjust the offset
    int64_t offset =
        1ull * i * n * k * cutlass::sizeof_bits<QuantType>::value / 8;
    cutlass::reorder_tensor(b_packed_ptr + offset, layout_B,
                            layout_B_reordered);
  }

  // save the packed layout to torch tensor so we can re-use it
  torch::stable::Tensor layout_cpu = torch::stable::empty(
      {num_experts, layout_width}, torch::headeronly::ScalarType::Int,
      std::nullopt, torch::stable::Device(torch::stable::DeviceType::CPU));

  int32_t* layout_data = layout_cpu.mutable_data_ptr<int32_t>();
  for (int i = 0; i < num_experts; ++i) {
    std::memcpy(layout_data + i * layout_width,  // dst (int32*)
                &layout_B_reordered,             // src (LayoutType*)
                sizeof(LayoutType));             // number of bytes
  }

  torch::stable::Tensor packed_layout =
      torch::stable::to(layout_cpu, b_tensors.device(),
                        /*non_blocking=*/false);

  return {b_tensors_packed, packed_layout};
}
```
**EN:** The preprocessing helper validates tensor shape constraints, rewrites int4 values through `unified_encode_int4b`, applies CUTLASS memory reordering per expert, and serializes the resulting layout so it can be reused later.
**CN:** 该预处理辅助函数会校验张量形状约束，通过 `unified_encode_int4b` 重写 int4 数值，对每个 expert 应用 CUTLASS 的内存重排，并把布局序列化保存，以便后续复用。

### Torch operator registration / Torch 算子注册
```cpp
STABLE_TORCH_LIBRARY_IMPL(_C, CUDA, m) {
  m.impl("cutlass_w4a8_moe_mm", TORCH_BOX(&mm));
  m.impl("cutlass_encode_and_reorder_int4b_grouped",
         TORCH_BOX(&encode_and_reorder_int4b));
```
**EN:** The final block exposes the grouped GEMM entry point and grouped int4 preprocessing helper through the stable Torch library mechanism.
**CN:** 最后的代码块通过 stable Torch library 机制导出 grouped GEMM 入口与 grouped int4 预处理辅助函数。

## Key Concepts / 关键概念
- Grouped MoE GEMM is modeled as an array of per-expert problem shapes / Grouped MoE GEMM 被建模为一组按 expert 划分的问题形状
- CUTLASS collective builders compose the mainloop, scale handling, and epilogue at compile time / CUTLASS collective builder 在编译期组合 mainloop、scale 处理与 epilogue
- Quantized B must be both numerically re-encoded and memory-reordered before fast Hopper execution / 量化 B 在高效 Hopper 执行前既要做数值重编码，也要做内存重排

## Dependencies / 依赖关系
- Depends on CUTLASS/CuTe, `get_group_starts.cuh`, and `w4a8_utils.cuh` / 依赖 CUTLASS/CuTe、`get_group_starts.cuh` 与 `w4a8_utils.cuh`
- Registers `_C.cutlass_w4a8_moe_mm` and the grouped encode/reorder helper for Torch / 向 Torch 注册 `_C.cutlass_w4a8_moe_mm` 与 grouped encode/reorder 辅助算子
