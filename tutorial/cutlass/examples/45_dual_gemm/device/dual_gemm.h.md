# dual_gemm.h — Code Analysis / 代码分析

**Source / 源文件**: `examples/45_dual_gemm/device/dual_gemm.h`  
**Purpose / 用途**: Public device-layer API for fused DualGemm This header is the user-facing device wrapper. It turns a very large template signature into a single callable object with `Arguments`, `can_implement()`, `get_workspace_size()`, `initialize()`, `update()`, and `run()`. The actual CTA kernel lives in `kernel/dual_gemm.h`, but most host code only touches this class. / 融合 DualGemm 的公开设备层 API 这个头文件是面向用户的设备层包装器。它把庞大的模板参数列表封装成一个可调用对象，并提供 `Arguments`、`can_implement()`、`get_workspace_size()`、`initialize()`、`update()` 和 `run()` 等接口。真正的 CTA kernel 位于 `kernel/dual_gemm.h`，但多数主机端代码只会直接接触这个类。

---

## Line-by-Line Analysis / 逐行分析

### Logical Block 1 / 逻辑块 1 — lines 31-54

```cpp
/*! \file
    \brief Performs a dual gemm in one fused kernel:
```
D0 = epilogue0(X @ B0, C0)
D1 = epilogue1(X @ B1, C1)
D2 = element_wise(D0, D1)
```
*/

#pragma once

#include "cutlass/cutlass.h"
#include "cutlass/numeric_types.h"
#include "cutlass/arch/arch.h"
#include "cutlass/device_kernel.h"

#include "cutlass/gemm/threadblock/threadblock_swizzle.h"

#include "cutlass/gemm/device/default_gemm_configuration.h"
#include "cutlass/gemm/threadblock/default_mma.h"
#include "cutlass/epilogue/thread/linear_combination_relu.h"
#include "cutlass/epilogue/threadblock/default_epilogue_tensor_op.h"

#include "../kernel/dual_gemm.h"
#include "../dual_gemm_common.h"
```

**EN**: The file header repeats the mathematical contract and includes only the pieces required to synthesize a device operator: architecture tags, kernel launcher helpers, default GEMM configuration, default MMA builders, default tensor-op epilogues, and the custom kernel/common headers.
**CN**: 文件头部再次强调了数学契约，并只包含构建设备算子所需的组件：架构标签、kernel 启动辅助、默认 GEMM 配置、默认 MMA 构建器、默认 tensor-op epilogue，以及自定义 kernel/common 头文件。

### Logical Block 2 / 逻辑块 2 — lines 65-205

```cpp
template <
    /// Element type for A matrix operand
    typename ElementA_,
    /// Layout type for A matrix operand
    typename LayoutA_,
    /// Element type for B matrix operand
    typename ElementB_,
    /// Layout type for B0 matrix operand
    typename LayoutB0_,
    /// Layout type for B1 matrix operand
    typename LayoutB1_,
    /// Element type for C and D matrix operands
    typename ElementC_,
    /// Layout type for C and D matrix operands
    typename LayoutC_,
    /// Element type for internal accumulation
    typename ElementAccumulator_,
    /// Operator class tag
    typename OperatorClass_,
    /// Tag indicating architecture to tune for
    typename ArchTag_,
    /// Threadblock-level tile size (concept: GemmShape)
    typename ThreadblockShape_,
    /// Warp-level tile size (concept: GemmShape)
    typename WarpShape_,
    /// Instruction-level tile size (concept: GemmShape)
    typename InstructionShape_,
    /// Epilogue output operator
    typename EpilogueOutputOp0_,
    typename EpilogueOutputOp1_,
    typename EpilogueOutputOp2_,
    /// Threadblock-level swizzling operator
    typename ThreadblockSwizzle_ = threadblock::GemmIdentityThreadblockSwizzle<>,
    /// Number of stages used in the pipelined mainloop
    int Stages =
        DefaultGemmConfiguration<OperatorClass_, ArchTag_, ElementA_, ElementB_,
                                 ElementC_, ElementAccumulator_>::kStages,
    bool StoreD0 = true,
    bool StoreD1 = true,
    /// If true, kernel supports split-K with serial reduction
    bool SplitKSerial = false,
    /// Access granularity of A matrix in units of elements
    int AlignmentA =
        DefaultGemmConfiguration<OperatorClass_, ArchTag_, ElementA_, ElementB_,
                                 ElementC_, ElementAccumulator_>::kAlignmentA,
    /// Access granularity of B matrix in units of elements
    int AlignmentB =
        DefaultGemmConfiguration<OperatorClass_, ArchTag_, ElementA_, ElementB_,
                                 ElementC_, ElementAccumulator_>::kAlignmentB,
    /// Operation performed by GEMM
    typename Operator_ = typename DefaultGemmConfiguration<
        OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
        ElementAccumulator_>::Operator>
class DualGemm {
 public:

  using ElementA = ElementA_;
  using LayoutA = LayoutA_;
  using TensorRefA = TensorRef<ElementA const, LayoutA>;
  using ElementB = ElementB_;
  using LayoutB0 = LayoutB0_;
  using LayoutB1 = LayoutB1_;
  using TensorRefB0 = TensorRef<ElementB const, LayoutB0>;
  using TensorRefB1 = TensorRef<ElementB const, LayoutB1>;
  using ElementC = ElementC_;
  using LayoutC = LayoutC_;
  using TensorRefC = TensorRef<ElementC const, LayoutC>;
  using TensorRefD = TensorRef<ElementC, LayoutC>;
  using ElementAccumulator = ElementAccumulator_;
  using OperatorClass = OperatorClass_;
  using ArchTag = ArchTag_;
  using ThreadblockShape = ThreadblockShape_;
  using WarpShape = WarpShape_;
  using InstructionShape = InstructionShape_;
  using EpilogueOutputOp0 = EpilogueOutputOp0_;
  using EpilogueOutputOp1 = EpilogueOutputOp1_;
  using EpilogueOutputOp2 = EpilogueOutputOp2_;
  using ThreadblockSwizzle = ThreadblockSwizzle_;
  using Operator = Operator_;
  static int const kStages = Stages;
  static int const kAlignmentA = AlignmentA;
  static int const kAlignmentB = AlignmentB;
  static int const kAlignmentC = EpilogueOutputOp1::kCount;
  static bool const kSplitKSerial = SplitKSerial;
  static bool constexpr kStoreD0 = StoreD0;
  static bool constexpr kStoreD1 = StoreD1;
  static ComplexTransform const kTransformA = ComplexTransform::kNone;
  static ComplexTransform const kTransformB = ComplexTransform::kNone;

  using LayoutScaleBias = layout::RowMajor;
  /// Define the kernel
  /// Define the threadblock-scoped matrix multiply-accumulate
  static_assert(ArchTag::kMinComputeCapability >= 80, "Only multistage is implemented");
  static_assert(kStages >= 3, "Only multistage is implemented");
  using Mma0 = typename cutlass::gemm::threadblock::DefaultMma<
      ElementA, LayoutA, kAlignmentA, ElementB, LayoutB0, kAlignmentB,
      ElementAccumulator, layout::RowMajor, arch::OpClassTensorOp, ArchTag,
      ThreadblockShape, WarpShape,
      InstructionShape, Stages, Operator>::ThreadblockMma;
  using Mma1 = typename cutlass::gemm::threadblock::DefaultMma<
      ElementA, LayoutA, kAlignmentA, ElementB, LayoutB1, kAlignmentB,
      ElementAccumulator, layout::RowMajor, arch::OpClassTensorOp, ArchTag,
      ThreadblockShape, WarpShape, 
      InstructionShape, Stages, Operator>::ThreadblockMma;
  using DualMma = threadblock::DualMmaMultistage<
    typename Mma0::Shape,
    typename Mma0::IteratorA,
    typename Mma0::SmemIteratorA,
    Mma0::kCacheOpA,
    typename Mma0::IteratorB,
    typename Mma0::SmemIteratorB,
    Mma0::kCacheOpB,
    typename Mma1::IteratorB,
    typename Mma1::SmemIteratorB,
    typename Mma0::ElementC,
    typename Mma0::LayoutC,
    typename Mma0::Policy,
    typename Mma1::Policy,
    Mma0::kStages,
    SharedMemoryClearOption::kNone
  >;

  static const int kPartitionsK = ThreadblockShape::kK / WarpShape::kK;

  /// Define the epilogue
  using Epilogue0 =
      typename cutlass::epilogue::threadblock::DefaultEpilogueTensorOp<
          ThreadblockShape, typename DualMma::Operator0, kPartitionsK, EpilogueOutputOp0,
          EpilogueOutputOp0::kCount>::Epilogue;
  using Epilogue1 =
      typename cutlass::epilogue::threadblock::DefaultEpilogueTensorOp<
          ThreadblockShape, typename DualMma::Operator1, kPartitionsK, EpilogueOutputOp1,
          EpilogueOutputOp1::kCount>::Epilogue;

  /// Define the kernel-level GEMM operator.
  using DualGemmKernel = kernel::DualGemm<
    DualMma,
    Epilogue0, Epilogue1, EpilogueOutputOp2,
    ThreadblockSwizzle, kSplitKSerial,
    kStoreD0, kStoreD1>;
```

**EN**: The large template signature exposes all major specialization knobs: operand/output layouts, accumulator type, operator class, architecture, tile shapes, epilogues, swizzle, number of stages, storage flags, alignment, and math operator. Inside the class, `Mma0` and `Mma1` are built independently with `DefaultMma`, then fused into `DualMmaMultistage`, and finally wrapped by two default tensor-op epilogues plus the custom third-stage output op. The static assertions enforce the intended implementation envelope: SM80+ and at least three pipeline stages.
**CN**: 庞大的模板签名暴露了几乎所有关键特化参数：输入/输出布局、累加器类型、算子类别、目标架构、tile 形状、epilogue、swizzle、stage 数、是否保存中间结果、对齐要求以及数学算子。类内部先用 `DefaultMma` 独立构建 `Mma0` 和 `Mma1`，再融合成 `DualMmaMultistage`，最后叠加两个默认 tensor-op epilogue 和自定义第三阶段输出算子。静态断言则明确限定了实现适用范围：SM80+ 且流水线 stage 至少为 3。

### Logical Block 3 / 逻辑块 3 — lines 206-294

```cpp
  /// Argument structure
  struct Arguments {

    //
    // Data members
    //

    DualGemmMode mode;
    GemmCoord problem_size;
    TensorRef<ElementA const, LayoutA> ref_A0;
    TensorRef<ElementB const, LayoutB0> ref_B0;
    TensorRef<ElementC const, LayoutC> ref_C0;
    TensorRef<ElementC, LayoutC> ref_D0;
    TensorRef<ElementB const, LayoutB1> ref_B1;
    TensorRef<ElementC const, LayoutC> ref_C1;
    TensorRef<ElementC, LayoutC> ref_D1;
    TensorRef<ElementC, LayoutC> ref_D2;
    typename EpilogueOutputOp0::Params epilogue0;
    typename EpilogueOutputOp1::Params epilogue1;
    typename EpilogueOutputOp2::Params epilogue2;
    int split_k_slices;

    int batch_count;
    int64_t batch_stride_A;
    int64_t batch_stride_B0;
    int64_t batch_stride_B1;
    int64_t batch_stride_C;
    int64_t batch_stride_D;

    //
    // Methods
    //

    /// Default ctor
    CUTLASS_HOST_DEVICE
    Arguments(): problem_size(0, 0, 0), split_k_slices(1) {

    }

    /// Constructs an Arguments structure 
    CUTLASS_HOST_DEVICE
    Arguments(
      DualGemmMode mode,
      GemmCoord problem_size_,
      TensorRef<ElementA const, LayoutA> ref_A0_,
      TensorRef<ElementB const, LayoutB0> ref_B0_,
      TensorRef<ElementC const, LayoutC> ref_C0_,
      TensorRef<ElementC, LayoutC> ref_D0_,
      TensorRef<ElementB const, LayoutB1> ref_B1_,
      TensorRef<ElementC const, LayoutC> ref_C1_,
      TensorRef<ElementC, LayoutC> ref_D1_,
      TensorRef<ElementC, LayoutC> ref_D2_,
      typename EpilogueOutputOp0::Params epilogue0_ =
        typename EpilogueOutputOp0::Params(),
      typename EpilogueOutputOp1::Params epilogue1_ =
        typename EpilogueOutputOp1::Params(),
      typename EpilogueOutputOp2::Params epilogue2_ =
        typename EpilogueOutputOp2::Params(),
      int split_k_slices_ = 1,
      int batch_count = 1,
      int64_t batch_stride_A = 0,
      int64_t batch_stride_B0 = 0,
      int64_t batch_stride_B1 = 0,
      int64_t batch_stride_C = 0,
      int64_t batch_stride_D = 0
    ):
      mode(mode),
      problem_size(problem_size_),
      ref_A0(ref_A0_),
      ref_B0(ref_B0_),
      ref_C0(ref_C0_),
      ref_D0(ref_D0_),
      ref_B1(ref_B1_),
      ref_C1(ref_C1_),
      ref_D1(ref_D1_),
      ref_D2(ref_D2_),
      epilogue0(epilogue0_),
      epilogue1(epilogue1_),
      epilogue2(epilogue2_),
      split_k_slices(split_k_slices_),
      batch_count(batch_count),
      batch_stride_A(batch_stride_A),
      batch_stride_B0(batch_stride_B0),
      batch_stride_B1(batch_stride_B1),
      batch_stride_C(batch_stride_C),
      batch_stride_D(batch_stride_D) {

    }
  };
```

**EN**: The nested `Arguments` struct is the host-visible control surface. It packages execution mode, problem size, all operand/source/destination tensor references, three epilogue parameter objects, split-K count, batch count, and all batch strides. This is where the device/kernel split becomes concrete: all runtime variability is expressed as data, not template changes.
**CN**: `Arguments` 嵌套结构体是主机端真正可见的控制面。它把执行模式、问题规模、全部输入/源/输出张量引用、三个 epilogue 参数对象、split-K 数量、batch 数量以及所有 batch stride 统一封装起来。device/kernel 分层在这里就变得非常具体：所有运行时变化都通过数据表达，而不是重新改模板。

### Logical Block 4 / 逻辑块 4 — lines 303-339

```cpp
  /// Constructs the GEMM.
  DualGemm() = default;

  /// Determines whether the GEMM can execute the given problem.
  static Status can_implement(Arguments const &args) {

    if (args.mode == DualGemmMode::kBatched && kSplitKSerial) {
      return Status::kErrorInvalidProblem;
    }
    if (!kSplitKSerial && args.split_k_slices > 1) {
      return Status::kErrorInvalidProblem;
    }
    if (kStoreD0 != (args.ref_D0.data() != nullptr)) {
      return Status::kErrorInternal;
    }
    if (kStoreD1 != (args.ref_D1.data() != nullptr)) {
      return Status::kErrorInternal;
    }

    Status status = DualGemmKernel::can_implement(
      args.problem_size,
      args.ref_A0.non_const_ref(),
      args.ref_B0.non_const_ref(),
      args.ref_C0.non_const_ref(),
      args.ref_D0,
      args.ref_B1.non_const_ref(),
      args.ref_C1.non_const_ref(),
      args.ref_D1,
      args.ref_D2
    );

    if (status != Status::kSuccess) {
      return status;
    }

    return Status::kSuccess;
  }
```

**EN**: `can_implement()` performs policy checks before reaching the kernel. Batched mode is rejected if split-K serial support is requested, multi-slice split-K is rejected unless the class was specialized for it, and the storage flags are cross-checked against whether `ref_D0/ref_D1` are actually non-null. After that it delegates alignment validation to `DualGemmKernel::can_implement()`.
**CN**: `can_implement()` 在进入 kernel 之前先做策略检查：若类启用了 split-K serial 支持则不允许 batched 模式；若类本身不支持 split-K serial，则拒绝多切片 split-K；同时还会检查 `kStoreD0/kStoreD1` 与 `ref_D0/ref_D1` 是否真正匹配。之后它再把对齐性检查委托给 `DualGemmKernel::can_implement()`。

### Logical Block 5 / 逻辑块 5 — lines 341-359

```cpp
  /// Gets the workspace size
  static size_t get_workspace_size(Arguments const &args) {

    size_t bytes = 0;

    if (kSplitKSerial && args.split_k_slices > 1) {
      // Determine grid shape
      ThreadblockSwizzle threadblock_swizzle;

      cutlass::gemm::GemmCoord tiled_shape = threadblock_swizzle.get_tiled_shape(
        args.problem_size,
        {ThreadblockShape::kM, ThreadblockShape::kN, ThreadblockShape::kK},
        args.split_k_slices);

      bytes += sizeof(int) * size_t(tiled_shape.m()) * size_t(tiled_shape.n());
    }

    return bytes;
  }
```

**EN**: `get_workspace_size()` is intentionally narrow: workspace is only needed for the semaphore array used by serial split-K reduction. Plain single-slice or batched execution consumes no extra workspace here.
**CN**: `get_workspace_size()` 的职责刻意收窄：这里只为串行 split-K 归约所需的 semaphore 数组申请空间。普通单切片执行或 batched 执行在这里都不需要额外 workspace。

### Logical Block 6 / 逻辑块 6 — lines 361-419

```cpp
  /// Initializes GEMM state from arguments.
  Status initialize(Arguments const &args, void *workspace = nullptr, cudaStream_t stream = nullptr) {

    // Determine grid shape
    ThreadblockSwizzle threadblock_swizzle;

    cutlass::gemm::GemmCoord grid_shape = threadblock_swizzle.get_tiled_shape(
      args.problem_size, 
      {ThreadblockShape::kM, ThreadblockShape::kN, ThreadblockShape::kK},
      args.mode == DualGemmMode::kBatched ? args.batch_count : args.split_k_slices);

    if (kSplitKSerial) {
      if (args.split_k_slices > 1) {
        if (!workspace) {
          return Status::kErrorWorkspaceNull;
        }

        size_t bytes = get_workspace_size(args);
      
        cudaError_t result = cudaMemsetAsync(workspace, 0, bytes, stream);

        if (result != cudaSuccess) {
          return Status::kErrorInternal;
        }
      }
    }
    else {

      if (args.split_k_slices > 1) {
        return Status::kErrorInvalidProblem;
      }
    }

    // Initialize the Params structure
    params_ = typename DualGemmKernel::Params{
      args.mode,
      args.problem_size,
      grid_shape,
      args.ref_A0.non_const_ref(),
      args.ref_B0.non_const_ref(),
      args.ref_C0.non_const_ref(),
      args.ref_D0,
      args.ref_B1.non_const_ref(),
      args.ref_C1.non_const_ref(),
      args.ref_D1,
      args.ref_D2,
      args.epilogue0,
      args.epilogue1,
      args.epilogue2,
      reinterpret_cast<int *>(workspace),
      args.batch_stride_A,
      args.batch_stride_B0,
      args.batch_stride_B1,
      args.batch_stride_C,
      args.batch_stride_D,
    };

    return Status::kSuccess;
  }
```

**EN**: `initialize()` computes the tiled grid shape using the threadblock swizzle, clears semaphore workspace when needed, and materializes a `DualGemmKernel::Params` object. A subtle but important detail is the third argument to `get_tiled_shape()`: batched mode uses `batch_count`, while single-GEMM mode uses `split_k_slices`.
**CN**: `initialize()` 先借助 threadblock swizzle 计算网格的 tiled 形状，在需要时清零 semaphore workspace，然后真正构造出 `DualGemmKernel::Params`。其中一个很关键的细节是 `get_tiled_shape()` 的第三个参数：batched 模式传入 `batch_count`，单 GEMM 模式则传入 `split_k_slices`。

### Logical Block 7 / 逻辑块 7 — lines 421-492

```cpp
  /// Lightweight update given a subset of arguments
  Status update(Arguments const &args, void *workspace = nullptr) {
    
    if (kSplitKSerial && args.split_k_slices > 1) {  
      if (!workspace) {
        return Status::kErrorWorkspaceNull;
      }
    }

    params_.ref_A0.reset(args.ref_A0.non_const_ref().data());
    params_.ref_B0.reset(args.ref_B0.non_const_ref().data());
    params_.ref_C0.reset(args.ref_C0.non_const_ref().data());
    params_.ref_D0.reset(args.ref_D0.data());
    params_.ref_B1.reset(args.ref_B1.non_const_ref().data());
    params_.ref_C1.reset(args.ref_C1.non_const_ref().data());
    params_.ref_D1.reset(args.ref_D1.data());
    params_.ref_D2.reset(args.ref_D2.data());
    params_.output_op_0 = args.epilogue0;
    params_.output_op_1 = args.epilogue1;
    params_.output_op_2 = args.epilogue2;
    params_.semaphore = reinterpret_cast<int *>(workspace);

    return Status::kSuccess;
  }

  /// Runs the kernel using initialized state.
  Status run(cudaStream_t stream = nullptr) {

    ThreadblockSwizzle threadblock_swizzle;

    dim3 grid = threadblock_swizzle.get_grid_shape(params_.grid_tiled_shape);
    dim3 block(DualGemmKernel::kThreadCount, 1, 1);

    cudaError_t result;

    int smem_size = int(sizeof(typename DualGemmKernel::SharedStorage));
    if (smem_size >= (48 << 10)) {
      result = cudaFuncSetAttribute(Kernel<DualGemmKernel>,
                                    cudaFuncAttributeMaxDynamicSharedMemorySize,
                                    smem_size);

      if (result != cudaSuccess) {
        return Status::kErrorInternal;
      }
    }

    cutlass::Kernel<DualGemmKernel><<<grid, block, smem_size, stream>>>(params_);

    result = cudaGetLastError();

    return result == cudaSuccess ? Status::kSuccess : Status::kErrorInternal;
  }

  /// Runs the kernel using initialized state.
  Status operator()(cudaStream_t stream = nullptr) {
    return run(stream);
  }

  /// Runs the kernel using initialized state.
  Status operator()(
    Arguments const &args, 
    void *workspace = nullptr, 
    cudaStream_t stream = nullptr) {
    
    Status status = initialize(args, workspace, stream);
    
    if (status == Status::kSuccess) {
      status = run(stream);
    }

    return status;
  }
```

**EN**: `update()` is the lightweight fast path for reusing an already-shaped kernel object with new pointers or new epilogue parameters. `run()` then selects grid/block dimensions, requests dynamic shared memory if the kernel needs at least 48 KiB, launches `cutlass::Kernel<DualGemmKernel>`, and converts `cudaGetLastError()` into a CUTLASS `Status`.
**CN**: `update()` 是复用已定型 kernel 对象时的轻量级快速路径，只更新指针和 epilogue 参数。随后 `run()` 会计算 grid/block 尺寸，在 shared memory 需求达到 48 KiB 以上时申请动态共享内存属性，启动 `cutlass::Kernel<DualGemmKernel>`，并把 `cudaGetLastError()` 转换成 CUTLASS `Status`。

### Logical Block 8 / 逻辑块 8 — lines 495-499

```cpp
} // namespace device
} // namespace gemm
} // namespace cutlass

////////////////////////////////////////////////////////////////////////////////
```

**EN**: The namespace closure marks the end of the host-visible wrapper layer. Everything below this layer is kernel implementation detail.
**CN**: 命名空间收尾意味着主机可见包装层到此结束；再往下就是纯 kernel 实现细节了。

---

## Key Concepts / 关键概念
**EN**: default CUTLASS Mma builders create two per-GEMM threadblock MMAs, `threadblock::DualMmaMultistage` fuses them into one mainloop, default tensor-op epilogues create `D0`/`D1`, and `kernel::DualGemm` stitches everything into an executable kernel.
**CN**: 组合关系：默认的 CUTLASS Mma 构建器先生成两个单独 GEMM 的 threadblock MMA，`threadblock::DualMmaMultistage` 再把它们融合到同一个 mainloop 中，默认 tensor-op epilogue 负责 `D0/D1`，最后由 `kernel::DualGemm` 把这些组件缝合成可执行 kernel。

**EN**: Device/kernel split in one sentence: this class turns template specialization into runtime-callable metadata, then delegates actual fusion work to `kernel::DualGemm`.
**CN**: 用一句话概括 device/kernel 分层：这个类把模板特化结果包装成可在运行时调用的元数据，然后把真正的融合执行交给 `kernel::DualGemm`。

## Dependencies / 依赖项
**EN**: CUTLASS dependency: `cutlass/cutlass.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/cutlass.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/numeric_types.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/numeric_types.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/arch/arch.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/arch/arch.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/device_kernel.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/device_kernel.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/gemm/threadblock/threadblock_swizzle.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/gemm/threadblock/threadblock_swizzle.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/gemm/device/default_gemm_configuration.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/gemm/device/default_gemm_configuration.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/gemm/threadblock/default_mma.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/gemm/threadblock/default_mma.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/epilogue/thread/linear_combination_relu.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/epilogue/thread/linear_combination_relu.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/epilogue/threadblock/default_epilogue_tensor_op.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/epilogue/threadblock/default_epilogue_tensor_op.h` 提供该文件直接使用的库级原语。

**EN**: Project-local dependency: `../kernel/dual_gemm.h` provides helper code included directly by this file.
**CN**: 项目内依赖：`../kernel/dual_gemm.h` 提供该文件直接包含的辅助代码。

**EN**: Project-local dependency: `../dual_gemm_common.h` provides helper code included directly by this file.
**CN**: 项目内依赖：`../dual_gemm_common.h` 提供该文件直接包含的辅助代码。
