# gemm_with_softmax.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/35_gemm_softmax/gemm_with_softmax.h`
**Purpose / 用途**: Builds a complete GEMM+Softmax pipeline by composing a visitor-enabled GEMM kernel, a final reduction kernel for row statistics, and a streaming softmax-apply kernel. / 通过组合 visitor 化 GEMM 内核、逐行统计的最终归约内核以及流式 softmax 应用内核，构建完整的 GEMM+Softmax 流水线。
---
## Line-by-Line Analysis / 逐行分析
```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/

/**

*/
```
**EN**: The file banner is minimal, but this header is the real composition layer of the example: it defines both the postprocessing kernel that materializes softmax values and the higher-level GemmSoftmax wrapper that launches all stages.
**CN**: 文件头比较简短，但这个头文件实际上是示例的“组合层”：它既定义了负责生成 softmax 值的后处理内核，也定义了启动全部阶段的高层 GemmSoftmax 封装。

```cpp
#pragma once

/////////////////////////////////////////////////////////////////////////////////////////////////

#include <cmath>
#include <iostream>
#include <vector>
#include <limits>

#include "cutlass/cutlass.h"
#include "cutlass/arch/memory.h"
#include "cutlass/arch/memory_sm75.h"

#include "cutlass/gemm/kernel/default_gemm.h"
#include "cutlass/gemm/kernel/default_gemm_complex.h"
#include "cutlass/gemm/device/default_gemm_configuration.h"
#include "cutlass/epilogue/threadblock/epilogue_visitor_with_softmax.h"
#include "cutlass/epilogue/threadblock/epilogue_with_visitor.h"
#include "cutlass/reduction/kernel/reduce_softmax_final.h"

/////////////////////////////////////////////////////////////////////////////////////////////////

#include "gemm_with_epilogue_visitor.h"
```
**EN**: These includes bring in CUTLASS GEMM defaults, device-configuration helpers, the epilogue-visitor implementation specialized for softmax, the final reduction kernel, and the local custom kernel wrapper from gemm_with_epilogue_visitor.h.
**CN**: 这些头文件引入了 CUTLASS GEMM 默认配置、设备端配置助手、面向 softmax 的 epilogue visitor 实现、最终归约内核，以及本示例本地定义的 gemm_with_epilogue_visitor.h 自定义封装。

```cpp
namespace cutlass {

/////////////////////////////////////////////////////////////////////////////////////////////////

namespace kernel {
```
**EN**: The outer namespace is cutlass, while ApplySoftmax is placed inside a nested kernel namespace to distinguish raw CUDA kernels from the higher-level orchestration class.
**CN**: 最外层命名空间是 cutlass，而 ApplySoftmax 被放在内部的 kernel 命名空间中，用来区分“原始 CUDA 内核”与“更高层的调度封装类”。

```cpp
template <
  typename ElementD_,
  typename ElementNorm_,
  typename ElementSum_,
  typename ElementSoft_,
  typename ElementSoftmaxCompute_,
  int Alignment,
  typename ApplyShape_ = MatrixShape<1, 1024>
>
class ApplySoftmax {
public:

  using ElementD = ElementD_;
  using ElementNorm = ElementNorm_;
  using ElementSum = ElementSum_;
  using ElementSoft = ElementSoft_;
  using ElementSoftmaxCompute = ElementSoftmaxCompute_;

  static int const kAlignment = Alignment;
  using ApplyShape = ApplyShape_;

  using Layout = cutlass::layout::RowMajor;

  using TensorRefD = TensorRef<ElementD, Layout>;
  using TensorRefN = TensorRef<ElementNorm, Layout>;
  using TensorRefSum = TensorRef<ElementSum, Layout>;
  using TensorRefSoft = TensorRef<ElementSoft, Layout>;

  using FragmentSoftmax = Array<ElementSoftmaxCompute, kAlignment>;
```
**EN**: ApplySoftmax is parameterized by the GEMM output type, row-max type, row-sum type, final softmax output type, internal compute type, vector alignment, and a tile shape. The aliases fix the layout to RowMajor because softmax is reduced across each row, and define a vector fragment type of kAlignment elements processed together by each thread.
**CN**: ApplySoftmax 由 GEMM 输出类型、行最大值类型、行和类型、最终 softmax 输出类型、内部计算类型、向量对齐宽度以及 tile 形状参数化。这里把布局固定为 RowMajor，因为 softmax 是按行归约；同时定义了长度为 kAlignment 的向量片段，让每个线程一次处理一组元素。

```cpp
  struct Arguments {

    MatrixCoord     extent;             ///< Extent of D and Softmax matrices
    int             batch_count;        ///< Batch count
    TensorRefD      ref_D;              ///< D matrix computed by GEMM+Max (input)
    TensorRefN      ref_N;              ///< Norm tensor (input)
    TensorRefSum    ref_S;              ///< Sum  tensor (input)
    TensorRefSoft   ref_Soft;           ///< Softmax tensor (output)
    int64_t         batch_stride_D;     ///< Batch stride for D tensor
    int64_t         batch_stride_N;     ///< Batch stride for N tensor
    int64_t         batch_stride_S;     ///< Batch stride for S tensor
    int64_t         batch_stride_Soft;  ///< Batch stride for softmax tensor

    //
    // Methods
    //
    Arguments():
      batch_count(1),
      batch_stride_D(0),
      batch_stride_N(0),
      batch_stride_S(0),
      batch_stride_Soft(0)
    { }

    Arguments(
      MatrixCoord     extent_,             ///< Extent of D and Softmax matrices
      int             batch_count_,        ///< Batch count
      TensorRefD      ref_D_,              ///< D matrix computed by GEMM+PartialReduce
      TensorRefN      ref_N_,              ///< Output parameter for N
      TensorRefSum    ref_S_,              ///< Output parameter for N
      TensorRefSoft   ref_Soft_,           ///< Softmax
      int64_t         batch_stride_D_ = 0,
      int64_t         batch_stride_N_ = 0,
      int64_t         batch_stride_S_ = 0,
      int64_t         batch_stride_Soft_ = 0
    ):
      extent(extent_),
      batch_count(batch_count_),
      ref_D(ref_D_),
      ref_N(ref_N_),
      ref_S(ref_S_),
      ref_Soft(ref_Soft_),
      batch_stride_D(batch_stride_D_),
      batch_stride_N(batch_stride_N_),
      batch_stride_S(batch_stride_S_),
      batch_stride_Soft(batch_stride_Soft_)
    {

    }
  };
```
**EN**: ApplySoftmax::Arguments is the runtime interface of the streaming softmax kernel. It passes the matrix extent, batch count, input/output TensorRefs, and batch strides for D, N, S, and Soft tensors. The constructor keeps the launch code readable by bundling everything in one object.
**CN**: ApplySoftmax::Arguments 是这个流式 softmax 内核的运行时接口。它传递矩阵尺寸、batch 数、输入输出 TensorRef，以及 D、N、S、Soft 张量的 batch 跨距。构造函数把这些信息集中打包，使启动代码更清晰。

```cpp
  struct Params {
    Arguments args;

    //
    // Methods
    //
    Params() { }

    Params(Arguments const &args_): args(args_) { }
  };

  //
  // SharedStorage
  //

  struct SharedStorage {

  };
```
**EN**: Params is intentionally thin: unlike the GEMM kernel, this kernel does not need heavy precomputation, so it simply stores Arguments. SharedStorage is empty because the kernel is dominated by global-memory streaming and register-level vector math.
**CN**: Params 被设计得非常轻量：与 GEMM 内核不同，这个内核几乎不需要复杂预计算，因此只是简单保存 Arguments。SharedStorage 为空，因为该内核主要由全局内存流式访问和寄存器级向量运算构成。

```cpp
  CUTLASS_DEVICE
  ApplySoftmax() { }

  CUTLASS_DEVICE
  void operator()(Params const &params, SharedStorage &shared_storage) {
    apply(params, shared_storage);
  }
```
**EN**: The public call operator forwards straight to apply(), which is a common CUTLASS style for device kernel functors.
**CN**: 公开的函数调用运算符直接转发到 apply()，这是 CUTLASS 设备端 functor 的常见写法。

```cpp
  /// Compute Softmax
  CUTLASS_DEVICE
  void apply(Params const &params, SharedStorage &shared_storage) {

    using AccessTypeD = AlignedArray<ElementD, kAlignment>;

    int block_batch = blockIdx.z;
    int block_m = blockIdx.x * ApplyShape::kRow;
    int block_n = 0;

    int thread_m = threadIdx.y;
    int thread_n = threadIdx.x * kAlignment;

    int idx_m = block_m + thread_m;
    int idx_n = block_n + thread_n;

    int batch_offset_norm = block_batch * params.args.batch_stride_N;
    int batch_offset_sum = block_batch * params.args.batch_stride_S;

    // Kill off thread if it is outside the row boundary
    if (params.args.extent.row() <= idx_m) {
      return;
    }

    //
    // Setup pointers to load D again
    //

    using AccessTypeD = AlignedArray<ElementD, kAlignment>;
    using AccessTypeSoft = AlignedArray<ElementSoft, kAlignment>;
    using FragmentSoft = Array<ElementSoft, kAlignment>;
    using ConvertSoftCompute = cutlass::NumericArrayConverter<ElementSoftmaxCompute, ElementD, kAlignment>;
    using ConvertSoftOutput = cutlass::NumericArrayConverter<ElementSoft, ElementSoftmaxCompute, kAlignment>;

    using Mul = cutlass::multiplies<FragmentSoftmax>;
    using Minus = cutlass::minus<FragmentSoftmax>;
    using Exp   = cutlass::fast_exp_op<FragmentSoftmax>;

    ConvertSoftCompute   convert_soft_compute;
    ConvertSoftOutput  convert_soft_output;

    Minus     minus;
    Mul       mul;
    Exp       exponential;

    using ConvertSum = cutlass::NumericConverter<ElementSoftmaxCompute, ElementSum>;
    using ConvertNorm = cutlass::NumericConverter<ElementSoftmaxCompute, ElementNorm>;

    ConvertSum   convert_sum;
    ConvertNorm  convert_norm;

    AccessTypeD *access_d = reinterpret_cast<AccessTypeD *>(
      params.args.ref_D.data() +
      params.args.batch_stride_D * block_batch +
      params.args.ref_D.layout()({idx_m, idx_n}));

    AccessTypeSoft *access_soft = reinterpret_cast<AccessTypeSoft *>(
      params.args.ref_Soft.data() +
      params.args.batch_stride_Soft * block_batch +
      params.args.ref_Soft.layout()({idx_m, idx_n}));

    ElementSum inv_sum = (params.args.ref_S.data())[idx_m + batch_offset_sum];
    ElementNorm norm = (params.args.ref_N.data())[idx_m + batch_offset_norm];

    //
    // Loop
    //
    CUTLASS_PRAGMA_UNROLL
    for (
      int idx = 0;
      idx < params.args.extent.column();
      idx += ApplyShape::kColumn * kAlignment) {

      if (idx_n < params.args.extent.column()) {
        AccessTypeD fetch;
        arch::global_load<AccessTypeD, sizeof(AccessTypeD)>(fetch, access_d, true);

        FragmentSoftmax result = mul(exponential(minus(convert_soft_compute(fetch), convert_norm(norm))),  convert_sum(inv_sum));
        FragmentSoft soft  = convert_soft_output(result);

        arch::global_store<FragmentSoft, sizeof(FragmentSoft)>(soft, access_soft, true);
      }

      access_d += ApplyShape::kColumn;
      access_soft += ApplyShape::kColumn;
      idx_n += ApplyShape::kColumn * kAlignment;
    }
  }
```
**EN**: apply() maps blockIdx.z to batch, blockIdx.x to a row tile, and threadIdx.x to an aligned vector lane. Each thread reloads a vector from D, converts it to the compute type, subtracts the per-row normalization term, exponentiates, multiplies by the reduced scaling factor from S, converts back to the output type, and stores the softmax result. Because block_n is fixed to zero and the kernel loops across columns internally, ApplyShape and Alignment must be chosen so one thread block efficiently sweeps the row.
**CN**: apply() 把 blockIdx.z 映射到 batch，把 blockIdx.x 映射到若干行组成的 tile，把 threadIdx.x 映射到按对齐宽度切分的向量 lane。每个线程从 D 重新加载一段向量，转换到计算类型后减去该行的归一化项，再做指数、乘上来自 S 的归约缩放因子，最后转换回输出类型并写出 softmax 结果。由于 block_n 被固定为 0，列方向主要依靠内核内部循环推进，因此 ApplyShape 与 Alignment 的选择必须保证单个线程块可以高效扫完整行。

```cpp
} // namespace kernel

/////////////////////////////////////////////////////////////////////////////////////////////////

///
template <
  typename ElementA_,
  typename LayoutA_,
  typename ElementB_,
  typename LayoutB_,
  typename ElementC_,
  typename ElementCompute_,
  typename OperatorClass_,
  typename ArchTag_,
  typename ThreadblockShape_,
  typename WarpShape_,
  typename InstructionShape_,
  typename EpilogueFunctorOp_,
  int kStages_,
  typename ApplyShape_ = MatrixShape<1, 1024>,
  int AlignmentA_ = 128 / cutlass::sizeof_bits<ElementA_>::value,
  int AlignmentB_ = 128 / cutlass::sizeof_bits<ElementB_>::value,
  int AlignmentSoftmax_ = 128 / cutlass::sizeof_bits<ElementC_>::value,
  typename ElementNorm_ = float,
  typename ElementSum_ = float,
  typename ElementSoftmax_ = ElementC_
>
class GemmSoftmax {
```
**EN**: The file then leaves the nested kernel namespace and declares the higher-level GemmSoftmax template. Its parameters expose CUTLASS’s key customization axes: A/B/C element types, layouts, compute type, operator class, architecture, threadblock/warp/instruction shapes, epilogue functor, pipeline stages, apply-tile shape, access alignments, and optional norm/sum/output element overrides.
**CN**: 接着文件离开内部 kernel 命名空间，并声明高层 GemmSoftmax 模板。它把 CUTLASS 的主要定制维度都暴露出来：A/B/C 元素类型与布局、计算类型、运算类别、目标架构、线程块/warp/指令形状、epilogue functor、流水线 stage 数、softmax 应用 tile 形状、访存对齐宽度，以及 norm/sum/output 元素类型的可选覆盖。

```cpp
  //
  // Type definitions
  //

  using ElementA = ElementA_;
  using ElementB = ElementB_;
  using ElementC = ElementC_;
  using ElementCompute = ElementCompute_;
  using ElementSum = ElementSum_;
  using ElementSoft = ElementSoftmax_;
  using ElementSoftmaxCompute = float;

  using LayoutA = LayoutA_;
  using LayoutB = LayoutB_;

  using EpilogueFunctorOp = EpilogueFunctorOp_;
  using ElementNorm = ElementNorm_;

  using ApplyShape = ApplyShape_;

  // These are mandatory layouts.
  using LayoutC = cutlass::layout::RowMajor;
  using LayoutN = cutlass::layout::RowMajor;
  using LayoutS = cutlass::layout::RowMajor;
  using LayoutSoft = cutlass::layout::RowMajor;

  using TensorRefA = TensorRef<ElementA, LayoutA>;
  using TensorRefB = TensorRef<ElementB, LayoutB>;
  using TensorRefC = TensorRef<ElementC, LayoutC>;
  using TensorRefN = TensorRef<ElementNorm, LayoutN>;
  using TensorRefSum = TensorRef<ElementSum, LayoutS>;
  using TensorRefSoft = TensorRef<ElementSoft, LayoutSoft>;

  using ThreadblockShape = ThreadblockShape_;
  using WarpShape        = WarpShape_;
  using InstructionShape = InstructionShape_;

  using OperatorClass = OperatorClass_;
  using ArchTag = ArchTag_;

  static int const kStages  = kStages_;
  static int const AlignmentA = AlignmentA_;
  static int const AlignmentB = AlignmentB_;
  static int const AlignmentSoftmax = AlignmentSoftmax_;

  using ThreadblockSwizzle = cutlass::gemm::threadblock::GemmBatchedIdentityThreadblockSwizzle;
```
**EN**: This alias section specializes the template into a concrete pipeline vocabulary. One important API choice is that C, N, S, and Soft are all fixed to RowMajor: the epilogue visitor emits row-wise partial reductions, the final reduction consumes row-major workspaces, and the apply kernel streams rows. Alignment defaults are derived from 128-bit accesses, matching CUTLASS’s vectorized iterator conventions.
**CN**: 这一大段别名把模板具体化为一套“流水线词汇”。其中一个重要 API 选择是：C、N、S 与 Soft 全部固定为 RowMajor，因为 epilogue visitor 输出的是按行组织的局部归约，最终归约内核也按行消费这些工作区，而 softmax 应用内核则沿着行流式处理。默认对齐宽度按 128-bit 访存推导，符合 CUTLASS 常见的向量化迭代器约定。

```cpp
  using DefaultGemmKernel = typename cutlass::gemm::kernel::DefaultGemm<
    ElementA,
    LayoutA,
    AlignmentA,
    ElementB,
    LayoutB,
    AlignmentB,
    ElementC,
    LayoutC,
    ElementCompute,
    OperatorClass,
    ArchTag,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueFunctorOp,
    ThreadblockSwizzle,
    kStages,
    true,
    typename cutlass::gemm::device::DefaultGemmConfiguration<
        OperatorClass, ArchTag, ElementA, ElementB, ElementC, ElementCompute>::Operator,
    cutlass::gemm::SharedMemoryClearOption::kNone
  >::GemmKernel;
```
**EN**: DefaultGemmKernel instantiates a standard CUTLASS GEMM kernel using the chosen layouts, shapes, architecture tag, epilogue functor, and swizzle. The higher-level design here is important: instead of rewriting the whole GEMM, the example first asks CUTLASS for a known-good mainloop/epilogue skeleton and only customizes the epilogue behavior later.
**CN**: DefaultGemmKernel 使用选定的布局、tile 形状、架构标签、epilogue functor 与 swizzle 实例化一个标准 CUTLASS GEMM 内核。这里的设计很关键：示例并没有重写整个 GEMM，而是先从 CUTLASS 获取一套成熟可靠的主循环/尾处理骨架，再在后面只替换尾处理行为。

```cpp
  // Epilogue visitor
  using EpilogueVisitor = typename cutlass::epilogue::threadblock::EpilogueVisitorSoftmax<
    ThreadblockShape,
    DefaultGemmKernel::kThreadCount,
    typename DefaultGemmKernel::Epilogue::OutputTileIterator,
    ElementCompute,
    ElementNorm,
    ElementSum,
    ElementSoftmaxCompute,
    EpilogueFunctorOp
  >;

  /// Epilogue
  using Epilogue = typename cutlass::epilogue::threadblock::EpilogueWithVisitorFromExistingEpilogue<
    EpilogueVisitor,
    typename DefaultGemmKernel::Epilogue
  >::Epilogue;

  // GEMM
  using GemmKernel = gemm::kernel::GemmWithEpilogueVisitor<
    typename DefaultGemmKernel::Mma,
    Epilogue,
    ThreadblockSwizzle
  >;

  // Softmax kernel
  using SoftmaxApplyKernel = kernel::ApplySoftmax<
    ElementC,
    ElementNorm,
    ElementSum,
    ElementSoft,
    ElementSoftmaxCompute,
    AlignmentSoftmax,
    ApplyShape
  >;

  using ApplyFinalReductionKernel = cutlass::reduction::kernel::ApplySoftmaxFinalReduction<
    ElementNorm,
    ElementSum,
    ElementSoftmaxCompute,
    ThreadblockShape
  >;
```
**EN**: This is the core composition chain. EpilogueVisitorSoftmax attaches row-wise max/sum accumulation to the original epilogue iterator flow; EpilogueWithVisitorFromExistingEpilogue reuses the existing epilogue machinery while swapping in the visitor; GemmKernel plugs that epilogue into the custom kernel wrapper; SoftmaxApplyKernel converts normalized logits to final outputs; and ApplyFinalReductionKernel merges the per-tile partial statistics across the N dimension. Together they implement softmax staging as: GEMM+partial-reduce -> final row reduction -> normalization/apply.
**CN**: 这里就是整个组合链的核心。EpilogueVisitorSoftmax 在原有 epilogue 迭代流程上附加逐行 max/sum 累积；EpilogueWithVisitorFromExistingEpilogue 复用现有 epilogue 机制，但换入 visitor；GemmKernel 再把这个 visitor 化 epilogue 插入到自定义内核封装中；SoftmaxApplyKernel 把归一化后的 logits 转成最终输出；ApplyFinalReductionKernel 则在 N 维上合并各 tile 产生的局部统计。三者合起来实现的 softmax 分阶段流程就是：GEMM+局部归约 -> 行级最终归约 -> 归一化应用。

```cpp
  /// Arguments class
  struct Arguments {

    typename GemmKernel::Arguments         gemm;
    typename SoftmaxApplyKernel::Arguments softmax;
    typename ApplyFinalReductionKernel::Arguments reduction;
    cutlass::gemm::GemmCoord extend;

    //
    // Methods
    //
    Arguments() { }

    Arguments(
      cutlass::gemm::GemmCoord problem_size,
      int32_t    batch_count_,
      TensorRefA ref_A_,
      TensorRefB ref_B_,
      TensorRefC ref_C_,
      TensorRefC ref_D_,
      typename EpilogueFunctorOp::Params linear_scaling,
      TensorRefN ref_N_,
      TensorRefSum ref_S_,
      TensorRefSoft ref_Softmax_,
      int64_t batch_stride_A_ = 0,
      int64_t batch_stride_B_ = 0,
      int64_t batch_stride_C_ = 0,
      int64_t batch_stride_D_ = 0,
      int64_t batch_stride_Max_ = 0,
      int64_t batch_stride_Sum_ = 0,
      int64_t batch_stride_Softmax_ = 0
    ):
      gemm(
        cutlass::gemm::GemmUniversalMode::kBatched,
        problem_size,
        batch_count_,
        ref_A_,
        ref_B_,
        ref_C_,
        ref_D_,
        ref_N_.data(),
        ref_S_.data(),
        batch_stride_A_,
        batch_stride_B_,
        typename EpilogueVisitor::Arguments(
          linear_scaling,
          batch_stride_C_,
          batch_stride_D_,
          batch_stride_Max_,
          batch_stride_Sum_
        )
      ),
      reduction(
        problem_size,
        ref_N_.data(),
        ref_S_.data(),
        batch_stride_Max_,
        batch_stride_Sum_
      ), 
      softmax(
        MatrixCoord(problem_size.m(), problem_size.n()),
        batch_count_,
        ref_D_,
        ref_N_,
        ref_S_,
        ref_Softmax_,
        batch_stride_D_,
        batch_stride_Max_,
        batch_stride_Sum_,
        batch_stride_Softmax_
      ),
      extend(problem_size)
    {

    }
  };
```
**EN**: GemmSoftmax::Arguments is the single host-facing bundle for the whole pipeline. Its constructor wires the three internal kernels together: the GEMM stage gets batched GEMM arguments plus visitor parameters, the reduction stage gets pointers to the partial max/sum workspaces, and the apply stage gets D/N/S/Soft tensor refs and strides. This API choice hides a fairly complex multi-kernel protocol behind one object.
**CN**: GemmSoftmax::Arguments 是整个流水线统一的主机侧参数包。其构造函数把三个内部内核串起来：GEMM 阶段获得 batched GEMM 参数与 visitor 参数；归约阶段获得局部 max/sum 工作区指针；apply 阶段获得 D/N/S/Soft 的 TensorRef 和跨距。这个 API 选择把一个相当复杂的多内核协议隐藏在单个对象之后。

```cpp
  struct Params {

    typename GemmKernel::Params         gemm;
    typename SoftmaxApplyKernel::Params softmax;
    typename ApplyFinalReductionKernel::Params reduction;
    MatrixCoord extend;
    //
    // Methods
    //
    Params() { }

    Params(Arguments const &args):
      gemm(args.gemm),
      reduction(args.reduction),
      softmax(args.softmax),
      extend(MatrixCoord(args.extend.m(), args.extend.n()))
    {

    }
  };
```
**EN**: Params stores the device-ready versions of the three sub-argument sets plus a MatrixCoord extent. The constructor mirrors Arguments, performing the lightweight conversions once during initialize().
**CN**: Params 保存三个子阶段参数的设备就绪版本，以及一个 MatrixCoord 形式的尺寸。其构造过程与 Arguments 对应，在 initialize() 中一次性完成这些轻量转换。

```cpp
private:

  Params params_;

public:

  /// Ctor
  GemmSoftmax() {

  }

  /// Initialize
  Status initialize(Arguments const &args) {

    params_ = Params(args);

    return cutlass::Status::kSuccess;
  }
```
**EN**: The class caches Params in a private member so launch-time code only needs a stream. This mirrors CUTLASS device operators such as device::Gemm: initialize() prepares all runtime state, and run() performs the actual kernel launches.
**CN**: 该类把 Params 缓存在私有成员中，因此真正运行时只需要传入一个 stream。这与 CUTLASS 的 device::Gemm 等设备级操作器风格一致：initialize() 负责准备运行状态，run() 负责实际启动内核。

```cpp
  Status run(cudaStream_t stream) {

    //
    // Launch the GEMM + max kernel
    //

    dim3 gemm_grid = ThreadblockSwizzle().get_grid_shape(params_.gemm.grid_tiled_shape);
    dim3 gemm_block(GemmKernel::kThreadCount, 1, 1);

    int gemm_smem_size = int(sizeof(typename GemmKernel::SharedStorage));

    cudaError_t result;

    if (gemm_smem_size >= (48 << 10)) {
      result = cudaFuncSetAttribute(cutlass::Kernel<GemmKernel>,
                                    cudaFuncAttributeMaxDynamicSharedMemorySize,
                                    gemm_smem_size);

      if (result != cudaSuccess) {
        return Status::kErrorInternal;
      }
    }

    cutlass::Kernel<GemmKernel><<<gemm_grid, gemm_block, gemm_smem_size, stream>>>(params_.gemm);

    result = cudaGetLastError();

    if (result != cudaSuccess) {
      return cutlass::Status::kErrorInternal;
    }


    //
    // Launch the ApplyFinalReductionKernel
    //

    int thread_per_block = 128;
    int block_per_row = (params_.extend.row() + thread_per_block - 1) / thread_per_block;
    if (block_per_row < 4) {
      thread_per_block = 32;
      block_per_row = (params_.extend.row() + thread_per_block - 1) / thread_per_block;
    }

    dim3 final_reduction_grid(block_per_row, 1, params_.softmax.args.batch_count);
    dim3 final_reduction_block(thread_per_block);

    Kernel<ApplyFinalReductionKernel><<<
      final_reduction_grid, final_reduction_block, sizeof(typename ApplyFinalReductionKernel::SharedStorage), stream
    >>>(params_.reduction);

    result = cudaGetLastError();

    if (result != cudaSuccess) {
      return cutlass::Status::kErrorInternal;
    }

    //
    // Launch the SoftmaxApplyKernel
    //

    dim3 apply_block(SoftmaxApplyKernel::ApplyShape::kColumn, SoftmaxApplyKernel::ApplyShape::kRow);

    int threadblock_rows = SoftmaxApplyKernel::ApplyShape::kRow;
    int threadblock_columns = SoftmaxApplyKernel::ApplyShape::kColumn * SoftmaxApplyKernel::kAlignment;

    dim3 apply_grid(
      (params_.softmax.args.extent.row() + threadblock_rows - 1) / threadblock_rows,
      (params_.softmax.args.extent.column() + threadblock_columns - 1) / threadblock_columns,
      params_.softmax.args.batch_count);

    Kernel<SoftmaxApplyKernel><<<
      apply_grid, apply_block, sizeof(typename SoftmaxApplyKernel::SharedStorage), stream
    >>>(params_.softmax);

    result = cudaGetLastError();

    if (result != cudaSuccess) {
      return cutlass::Status::kErrorInternal;
    }

    return cutlass::Status::kSuccess;
```
**EN**: run() launches three kernels in sequence. First it launches the custom GEMM kernel and, when necessary, requests a larger dynamic shared-memory limit. Then it launches the final reduction kernel, choosing 128 or 32 threads depending on the row count. Finally it launches ApplySoftmax with a block shape derived from ApplyShape and kAlignment. Each launch is followed by cudaGetLastError(), and any CUDA failure is translated to kErrorInternal.
**CN**: run() 按顺序启动三个内核。首先启动自定义 GEMM 内核，并在需要时申请更大的动态共享内存上限；然后启动最终归约内核，根据行数在 128 或 32 线程之间选择；最后根据 ApplyShape 与 kAlignment 派生出的线程块形状启动 ApplySoftmax。每次启动后都通过 cudaGetLastError() 检查错误，任何 CUDA 失败都会被转换为 kErrorInternal。

```cpp
  /// Function call operator
  Status operator()(cudaStream_t stream = nullptr) {
    return run(stream);
  }
};
```
**EN**: operator() is a convenience wrapper that defaults the CUDA stream to nullptr and forwards to run().
**CN**: operator() 是一个便捷封装：默认使用 nullptr stream，并直接转发到 run()。

```cpp
} // namespace cutlass

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: The namespace closes here; the header now exposes a reusable, templated GEMM+softmax device operator for the example.
**CN**: 命名空间在此结束；至此，这个头文件对外暴露了一个可复用、可模板化的 GEMM+softmax 设备级操作器。

---
## Key Concepts / 关键概念
- Multi-kernel softmax staging / 多内核 softmax 分阶段计算
- CUTLASS default GEMM specialization reuse / 复用 CUTLASS 默认 GEMM 特化
- Epilogue visitor pattern / Epilogue visitor 模式
- Row-major workspace design for row-wise reductions / 面向逐行归约的 RowMajor 工作区设计
- Alignment-driven vector loads and stores / 由对齐宽度驱动的向量加载与存储
- Host-facing argument bundling for several kernels / 面向主机的一体化多内核参数封装
## Dependencies / 依赖项
- `cutlass/gemm/kernel/default_gemm.h` — builds the baseline threadblock GEMM mainloop and default epilogue / 构建基础线程块 GEMM 主循环与默认 epilogue
- `cutlass/gemm/device/default_gemm_configuration.h` — selects architecture-specific operator defaults / 选择架构相关的默认算子配置
- `cutlass/epilogue/threadblock/epilogue_visitor_with_softmax.h` — provides the softmax-specific epilogue visitor / 提供面向 softmax 的 epilogue visitor
- `cutlass/epilogue/threadblock/epilogue_with_visitor.h` — adapts an existing epilogue to the visitor pattern / 把现有 epilogue 适配到 visitor 模式
- `cutlass/reduction/kernel/reduce_softmax_final.h` — final row-wise reduction over partial max/sum results / 对局部 max/sum 结果进行最终逐行归约
- `gemm_with_epilogue_visitor.h` — custom GEMM kernel wrapper that plugs the visitor-enabled epilogue into the mainloop / 将 visitor 化 epilogue 插入主循环的自定义 GEMM 封装
- `cutlass/arch/memory.h` and `cutlass/arch/memory_sm75.h` — vectorized global load/store helpers used by ApplySoftmax / ApplySoftmax 使用的向量化全局加载/存储辅助接口
