# dual_gemm_run.h — Code Analysis / 代码分析

**Source / 源文件**: `examples/45_dual_gemm/dual_gemm_run.h`  
**Purpose / 用途**: Host harnesses for non-fused and fused DualGemm experiments This header contains almost all host-side mechanics of the example: tensor allocation, random initialization, CUTLASS argument construction, warm-up/profiling loops, reference computation, and result checking. It is the bridge between the simple driver in `dual_gemm.cu` and the highly templated device code. / 非融合与融合 DualGemm 实验的主机端运行框架 这个头文件承载了示例几乎全部的主机端执行细节：张量分配、随机初始化、CUTLASS 参数构造、预热/计时循环、参考计算以及结果校验。它把 `dual_gemm.cu` 中简洁的驱动层与底层高度模板化的设备代码连接起来。

---

## Line-by-Line Analysis / 逐行分析

### Logical Block 1 / 逻辑块 1 — lines 31-60

```cpp
#pragma once

#include <iostream>
#include <fstream>
#include <sstream>
#include <type_traits>

#include "cutlass/util/host_tensor.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_norm.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/device/tensor_relu.h"

#include "cutlass/platform/platform.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/gemm/device/gemm_universal.h"

#include "dual_gemm_common.h"
#include "helper.h"

#define CHECK_GT(val1, val2) \
    if((val1) <= (val2)) \
        std::cerr << __FILE__ << " " << __LINE__ << ": CHECK_GT failed\n";
#define CHECK_TRUE(val) \
    if(!(val)) \
        std::cerr << __FILE__ << " " << __LINE__ << ": CHECK_TRUE failed\n";
```

**EN**: The includes pull in host/device reference utilities, tensor comparison helpers, and `GemmUniversal`. The lightweight `CHECK_GT` and `CHECK_TRUE` macros only log failures instead of aborting immediately, which makes the file more of a diagnostic harness than a strict unit-test framework.
**CN**: 这些头文件引入了主机/设备参考实现、张量比较工具以及 `GemmUniversal`。这里定义的 `CHECK_GT` 与 `CHECK_TRUE` 只是记录错误而不会立即中止，因此整个文件更像诊断型运行框架，而不是严格的单元测试框架。

### Logical Block 2 / 逻辑块 2 — lines 62-132

```cpp
template <
  typename OutputOp,
  typename Element,
  typename Layout>
struct TensorEpilogueForEachFunc {
  /// View type
  using TensorView = cutlass::TensorView<Element, Layout>;

  /// Coordinate in tensor's index space
  using TensorCoord = typename TensorView::TensorCoord;

  /// Parameters structure
  struct Params {

    //
    // Data members
    //

    TensorView view_x0;
    TensorView view_x1;
    TensorView view_y;
    OutputOp output_op;


    //
    // Methods
    //

    Params(
      TensorView view_x0_ = TensorView(),
      TensorView view_x1_ = TensorView(),
      TensorView view_y_ = TensorView(),
      OutputOp output_op_ = OutputOp(typename OutputOp::Params{})
    ):
      view_x0(view_x0_), view_x1(view_x1_), view_y(view_y_), output_op(output_op_) {
    }
  };

  Params params;

  CUTLASS_DEVICE
  TensorEpilogueForEachFunc(Params const &params): params(params) {

  }

  CUTLASS_DEVICE
  void operator()(TensorCoord const &coord) {
    Element const & x0 = params.view_x0.at(coord);
    Element const & x1 = params.view_x1.at(coord);
    Element& y = params.view_y.at(coord);
    y = params.output_op(x0, x1);
  }
};

template <
  typename OutputOp,
  typename Element,
  typename Layout>
void TensorEpilogueForEach(
  cutlass::TensorView<Element, Layout> x0,
  cutlass::TensorView<Element, Layout> x1,
  cutlass::TensorView<Element, Layout> y) {
  
  using Func = TensorEpilogueForEachFunc<OutputOp, Element, Layout>;
  using Params = typename Func::Params;

  cutlass::reference::device::TensorForEach<Func, Layout::kRank, Params>(
    y.extent(),
    Params(x0, x1, y)
  );
}
```

**EN**: `TensorEpilogueForEachFunc` and `TensorEpilogueForEach` form a reusable device-side elementwise launcher. Given two tensor views and an output view, they apply `OutputOp(x0, x1)` at every coordinate. In this example that means reproducing `D2 = SiLU(D0) * D1` for reference validation after the two reference GEMMs finish.
**CN**: `TensorEpilogueForEachFunc` 与 `TensorEpilogueForEach` 组成了一个可复用的设备端逐元素执行器。它接受两个输入 view 和一个输出 view，在每个坐标上计算 `OutputOp(x0, x1)`。在本示例中，这正是参考校验所需的 `D2 = SiLU(D0) * D1` 重建步骤。

### Logical Block 3 / 逻辑块 3 — lines 136-202

```cpp
template <typename Gemm0_, typename Gemm1_>
struct NonFusedDualGemmRun
{

  using Gemm0 = Gemm0_;
  using Gemm1 = Gemm1_;
  using ElementAccumulator = typename Gemm0::ElementAccumulator;
  using ElementCompute = typename Gemm0::GemmKernel::Epilogue::OutputOp::ElementCompute;

  /// Initialization
  cutlass::Distribution::Kind init_A;
  cutlass::Distribution::Kind init_B;
  cutlass::Distribution::Kind init_C;
  cutlass::Distribution::Kind init_Bias;
  uint64_t seed;

  //
  // Methods
  //

  NonFusedDualGemmRun(
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform, 
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform, 
    cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform, 
    cutlass::Distribution::Kind init_Bias_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = 2080
  ):
    init_A(init_A_), init_B(init_B_), init_C(init_C_), init_Bias(init_Bias_), seed(seed_) { }

  /// Helper to initialize a tensor view
  template <typename Element, typename Layout>
  bool initialize_tensor(
    cutlass::TensorView<Element, Layout> view, 
    cutlass::Distribution::Kind dist_kind,
    uint64_t seed) {

    if (dist_kind == cutlass::Distribution::Uniform) {

      cutlass::reference::host::TensorFillRandomUniform(
        view, seed, 2, -2, 0);
    } 
    else if (dist_kind == cutlass::Distribution::Identity) {

      cutlass::reference::host::TensorFillIdentity(view);
    }
    else if (dist_kind == cutlass::Distribution::Gaussian) {

      cutlass::reference::host::TensorFillRandomGaussian(view, seed, 0, 0.5);
    }
    else if (dist_kind == cutlass::Distribution::Sequential) {

      cutlass::reference::host::BlockFillSequential(
        view.data(), view.capacity());
    }
    else if (dist_kind == cutlass::Distribution::AllZeros) {
      cutlass::reference::host::TensorFill(view, Element(0));
    }
    else if (dist_kind == cutlass::Distribution::AllOnes) {
      cutlass::reference::host::TensorFill(view, Element(1));
    }
    else {
      std::cerr << "Not implemented\n";
      return false;
    }

    return true;
  }
```

**EN**: `NonFusedDualGemmRun` stores random-distribution policy and seed, then provides a generic `initialize_tensor()` helper. The initializer supports uniform, identity, gaussian, sequential, all-zero, and all-one modes, so the harness can be reused for debugging as well as benchmarking.
**CN**: `NonFusedDualGemmRun` 保存随机分布策略和种子，并提供通用的 `initialize_tensor()` 辅助函数。初始化器支持 uniform、identity、gaussian、sequential、全零和全一等模式，因此它不仅能做 benchmark，也适合调试和构造可控输入。

### Logical Block 4 / 逻辑块 4 — lines 208-295

```cpp
    cutlass::HostTensor<
      typename Gemm0::ElementA, 
      typename Gemm0::LayoutA> tensor_A0(problem_size.mk());

    cutlass::HostTensor<
      typename Gemm0::ElementB, 
      typename Gemm0::LayoutB> tensor_B0(problem_size.kn());

    cutlass::HostTensor<
      typename Gemm0::ElementC, 
      typename Gemm0::LayoutC> tensor_C0(problem_size.mn());

    cutlass::HostTensor<
      typename Gemm1::ElementC,
      typename Gemm0::LayoutC> tensor_Bias0({1, problem_size.n()});

    cutlass::HostTensor<
      typename Gemm0::ElementC, 
      typename Gemm0::LayoutC> tensor_D0(problem_size.mn());

    cutlass::HostTensor<
      typename Gemm0::ElementC, 
      typename Gemm0::LayoutC> reference_D0(problem_size.mn());

    cutlass::HostTensor<
      typename Gemm1::ElementB, 
      typename Gemm1::LayoutB> tensor_B1(problem_size.kn());

    cutlass::HostTensor<
      typename Gemm1::ElementC, 
      typename Gemm1::LayoutC> tensor_C1(problem_size.mn());

    cutlass::HostTensor<
      typename Gemm1::ElementC,
      typename Gemm1::LayoutC> tensor_Bias1({1, problem_size.n()});

    cutlass::HostTensor<
      typename Gemm1::ElementC, 
      typename Gemm1::LayoutC> tensor_D1(problem_size.mn());

    cutlass::HostTensor<
      typename Gemm1::ElementC, 
      typename Gemm1::LayoutC> reference_D1(problem_size.mn());


    CHECK_TRUE(initialize_tensor(tensor_A0.host_view(), init_A, seed + 2019));
    CHECK_TRUE(initialize_tensor(tensor_B0.host_view(), init_B, seed + 2018));
    CHECK_TRUE(initialize_tensor(tensor_C0.host_view(), init_C, seed + 2017));
    CHECK_TRUE(initialize_tensor(tensor_Bias0.host_view(), init_Bias, seed + 2014));
    CHECK_TRUE(initialize_tensor(tensor_B1.host_view(), init_B, seed + 2016));
    CHECK_TRUE(initialize_tensor(tensor_C1.host_view(), init_C, seed + 2015));
    CHECK_TRUE(initialize_tensor(tensor_Bias1.host_view(), init_Bias, seed + 2013));

    cutlass::reference::host::TensorFill(
      tensor_D0.host_view());
    cutlass::reference::host::TensorFill(
      tensor_D1.host_view());
    cutlass::reference::host::TensorFill(
      reference_D0.host_view());
    cutlass::reference::host::TensorFill(
      reference_D1.host_view());

    tensor_A0.sync_device();
    tensor_B0.sync_device();
    tensor_C0.sync_device();
    tensor_Bias0.sync_device();
    tensor_D0.sync_device();
    reference_D0.sync_device();
    tensor_B1.sync_device();
    tensor_C1.sync_device();
    tensor_Bias1.sync_device();
    tensor_D1.sync_device();
    reference_D1.sync_device();
```

**EN**: The non-fused `run()` method allocates A, B0, C0/bias0, D0, B1, C1/bias1, and D1 host tensors, initializes them, clears output/reference tensors, and syncs everything to the device. Notice that bias vectors are represented by `LayoutC::Stride(0)` later, so the epilogue treats them as broadcast rows rather than full matrices.
**CN**: 非融合 `run()` 先分配 A、B0、C0/bias0、D0、B1、C1/bias1、D1 等 HostTensor，完成初始化，清零输出/参考张量，再同步到设备。需要注意的是，后面 bias 会通过 `LayoutC::Stride(0)` 的方式构造成广播向量，因此 epilogue 会把它当作按行广播的偏置，而不是完整矩阵。

### Logical Block 5 / 逻辑块 5 — lines 301-379

```cpp
    int split_k_slices = Gemm0::kSplitKSerial ? 2 : 1;
    typename Gemm0::Arguments arguments_0{
      problem_size,
      tensor_A0.device_ref(),
      tensor_B0.device_ref(),
      {tensor_Bias0.device_data(), typename Gemm0::LayoutC::Stride(0)},
      tensor_D0.device_ref(),
      {alpha0, beta0},
      split_k_slices
    };

    split_k_slices = Gemm1::kSplitKSerial ? 2 : 1;
    typename Gemm1::Arguments arguments_1{
      problem_size,
      tensor_A0.device_ref(),
      tensor_B1.device_ref(),
      {tensor_Bias1.device_data(), typename Gemm1::LayoutC::Stride(0)},
      tensor_D1.device_ref(),
      {alpha1, beta1},
      split_k_slices
    };


    Gemm0 gemm_op_0;
    Gemm1 gemm_op_1;

    // Allocate workspace memory
    cutlass::device_memory::allocation<uint8_t> workspace0(gemm_op_0.get_workspace_size(arguments_0));
    cutlass::device_memory::allocation<uint8_t> workspace1(gemm_op_1.get_workspace_size(arguments_1));

    cutlass::Status status = gemm_op_0.initialize(arguments_0, workspace0.get());

    CUTLASS_CHECK(status);

    status = gemm_op_1.initialize(arguments_1, workspace1.get());

    CUTLASS_CHECK(status);

    for(int i = 0; i < warm_ups; i++) {
        status = gemm_op_0();
        CUTLASS_CHECK(status);
        status = gemm_op_1();
        CUTLASS_CHECK(status);
    }

    if (is_profiling) {
      //
      // Profile the GEMM
      //

      cudaEvent_t start, stop1, stop2;
      cudaEventCreate(&start);
      cudaEventCreate(&stop1);
      cudaEventCreate(&stop2);

      cudaEventRecord(start);

      for(int i = 0; i < runs; i++) {
          status = gemm_op_0();
 
          CUTLASS_CHECK(status);
      }
      cudaEventRecord(stop1);
      for(int i = 0; i < runs; i++) {
          status = gemm_op_1();

          CUTLASS_CHECK(status);
      }

      cudaEventRecord(stop2);
      cudaDeviceSynchronize();
      float gemm0Time, gemm1Time, totalTime;
      cudaEventElapsedTime(&gemm0Time, start, stop1);
      cudaEventElapsedTime(&gemm1Time, stop1, stop2);
      cudaEventElapsedTime(&totalTime, start, stop2);
      std::cout << "gemm 0 time " << gemm0Time / (float)runs << " ms\n";
      std::cout << "gemm 1 time " << gemm1Time / (float)runs << " ms\n";
      std::cout << "Non-fusion GEMM only time " << totalTime / (float)runs << " ms\n";
    }
```

**EN**: This block builds two ordinary CUTLASS GEMM argument objects, allocates their workspaces independently, initializes each operator, performs warm-up iterations, and optionally profiles them with CUDA events. The output prints separate times for GEMM0, GEMM1, and the combined non-fused cost.
**CN**: 这一段构造两个普通 CUTLASS GEMM 参数对象，分别申请 workspace，初始化两个算子，执行预热，并在需要时用 CUDA events 计时。输出结果会分别打印 GEMM0、GEMM1 以及二者合计的非融合开销。

### Logical Block 6 / 逻辑块 6 — lines 387-470

```cpp
    cutlass::reference::device::Gemm<
        typename Gemm0::ElementA, typename Gemm0::LayoutA,
        typename Gemm0::ElementB, typename Gemm0::LayoutB,
        typename Gemm0::ElementC, typename Gemm0::LayoutC, ElementCompute,
        ElementAccumulator, typename Gemm0::Operator>
        reference_gemm_0;

    cutlass::reference::device::Gemm<
        typename Gemm1::ElementA, typename Gemm1::LayoutA,
        typename Gemm1::ElementB, typename Gemm1::LayoutB,
        typename Gemm1::ElementC, typename Gemm1::LayoutC, ElementCompute,
        ElementAccumulator, typename Gemm1::Operator>
        reference_gemm_1;

    reference_gemm_0(
      problem_size,
      alpha0, 
      tensor_A0.device_ref(), 
      tensor_B0.device_ref(), 
      beta0, 
      {tensor_Bias0.device_data(), typename Gemm0::LayoutC::Stride(0)},
      reference_D0.device_ref()
    );

    if(relu) {
       cutlass::reference::device::TensorReLu(reference_D0.device_view()); 
    }

    reference_gemm_1(
      problem_size,
      alpha1, 
      tensor_A0.device_ref(), 
      tensor_B1.device_ref(), 
      beta1,
      {tensor_Bias1.device_data(), typename Gemm1::LayoutC::Stride(0)},
      reference_D1.device_ref()
    );
    
    if(relu) {
       cutlass::reference::device::TensorReLu(reference_D1.device_view()); 
    }
   
    // Wait for kernels to finish
    cudaDeviceSynchronize();
    reference_D0.sync_host();
    reference_D1.sync_host();

    CHECK_GT(cutlass::reference::host::TensorNorm(tensor_D0.host_view()), 0);
    CHECK_GT(cutlass::reference::host::TensorNorm(reference_D0.host_view()), 0);
    CHECK_GT(cutlass::reference::host::TensorNorm(tensor_D1.host_view()), 0);
    CHECK_GT(cutlass::reference::host::TensorNorm(reference_D1.host_view()), 0);

    bool passed0 = cutlass::reference::host::TensorEquals(
      reference_D1.host_view(), 
      tensor_D1.host_view());
    CHECK_TRUE(passed0);

    bool passed1 = cutlass::reference::host::TensorEquals(
      reference_D1.host_view(), 
      tensor_D1.host_view());
    CHECK_TRUE(passed1);
    if (!passed0 || !passed1) {

      std::stringstream fname;

      fname << "error_DualGemm_device_nonfused.txt";
      std::cerr << "Dumping results in " << fname.str() << "\n";

      std::ofstream file(fname.str());

      file 
        << "A0 =\n" << tensor_A0.host_view()
        << "\nB0 =\n" << tensor_B0.host_view()
        << "\nC0 =\n" << tensor_C0.host_view()
        << "\nBias0:\n" << tensor_Bias0.host_view() << "\n"
        << "\nD0 =\n" << tensor_D0.host_view()
        << "\nB1 =\n" << tensor_B1.host_view()
        << "\nC1 =\n" << tensor_C1.host_view()
        << "\nBias1:\n" << tensor_Bias1.host_view() << "\n"
        << "\n\nReference =\n" << reference_D1.host_view()
        << "\nComputed =\n" << tensor_D1.host_view();
    }
    return passed0 && passed1;
  }
```

**EN**: Reference validation for the non-fused path uses `cutlass::reference::device::Gemm` twice and then compares host copies. One subtle implementation detail: the current code compares `reference_D1` with `tensor_D1` twice, so when you read this file treat that as the implemented behavior, even though the surrounding context suggests the first check was intended for `D0`.
**CN**: 非融合路径的参考校验通过两次 `cutlass::reference::device::Gemm` 完成，然后回传到主机比较。这里有个实现细节需要注意：当前代码把 `reference_D1` 与 `tensor_D1` 比较了两次，因此阅读时应以“当前实现行为”为准，尽管上下文看起来第一次比较原本更像是想校验 `D0`。

### Logical Block 7 / 逻辑块 7 — lines 473-542

```cpp
template <typename DualGemm_>
struct DualFusedGemmRun
{

  using DualGemm = DualGemm_;
  using ElementAccumulator = typename DualGemm::ElementAccumulator;
  using ElementCompute = typename DualGemm::DualGemmKernel::Epilogue0::OutputOp::ElementCompute;
  using EpilogueOutputOp2 = typename DualGemm::EpilogueOutputOp2;

  /// Initialization
  cutlass::Distribution::Kind init_A;
  cutlass::Distribution::Kind init_B;
  cutlass::Distribution::Kind init_C;
  cutlass::Distribution::Kind init_Scale;
  cutlass::Distribution::Kind init_Bias;
  uint64_t seed;

  //
  // Methods
  //

  DualFusedGemmRun(
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform, 
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform, 
    cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform, 
    cutlass::Distribution::Kind init_Scale_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_Bias_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = 2080
  ):
    init_A(init_A_), init_B(init_B_), init_C(init_C_),
    init_Scale(init_Scale_), init_Bias(init_Bias_), seed(seed_) { }

  /// Helper to initialize a tensor view
  template <typename Element, typename Layout>
  bool initialize_tensor(
    cutlass::TensorView<Element, Layout> view, 
    cutlass::Distribution::Kind dist_kind,
    uint64_t seed) {

    if (dist_kind == cutlass::Distribution::Uniform) {

      cutlass::reference::host::TensorFillRandomUniform(
        view, seed, 2, -2, 0);
    } 
    else if (dist_kind == cutlass::Distribution::Identity) {

      cutlass::reference::host::TensorFillIdentity(view);
    } 
    else if (dist_kind == cutlass::Distribution::Gaussian) {

      cutlass::reference::host::TensorFillRandomGaussian(view, seed, 0, 0.5);
    }
    else if (dist_kind == cutlass::Distribution::Sequential) {

      cutlass::reference::host::BlockFillSequential(
        view.data(), view.capacity());
    }
    else if (dist_kind == cutlass::Distribution::AllZeros) {
      cutlass::reference::host::TensorFill(view, Element(0));
    }
    else if (dist_kind == cutlass::Distribution::AllOnes) {
      cutlass::reference::host::TensorFill(view, Element(1));
    }
    else {
      std::cerr << "Not implemented\n";
      return false;
    }

    return true;
  }
```

**EN**: `DualFusedGemmRun` mirrors the baseline harness but binds itself to the fused `DualGemm` type. It exposes the same distribution-driven initialization style and extracts `EpilogueOutputOp2` from the kernel type so the reference path can apply the exact same fused elementwise operator.
**CN**: `DualFusedGemmRun` 在结构上与基线框架对应，但绑定的是融合后的 `DualGemm` 类型。它保留了同样的分布驱动初始化方式，并从 kernel 类型中提取 `EpilogueOutputOp2`，从而保证参考路径能够使用完全一致的最终融合算子。

### Logical Block 8 / 逻辑块 8 — lines 548-620

```cpp
  bool run(
    cutlass::gemm::GemmCoord problem_size,
    ElementCompute alpha0 = ElementCompute(1),
    ElementCompute beta0 = ElementCompute(1),
    ElementCompute alpha1 = ElementCompute(1),
    ElementCompute beta1 = ElementCompute(1),
    int batch_count = 1,
    bool broadcast_b1 = false,
    bool is_profiling = true,
    bool relu = false,
    int warm_ups = 1,
    int runs = 100) {
    
    //
    // Allocate the GEMM workspace
    //

    cutlass::HostTensor<
      typename DualGemm::ElementA,
      typename DualGemm::LayoutA> tensor_A0(
        cutlass::platform::is_same<typename DualGemm::LayoutA, cutlass::layout::RowMajor>::value ?
          cutlass::MatrixCoord(batch_count * problem_size.m(), problem_size.k()) :
          cutlass::MatrixCoord(problem_size.m(), batch_count * problem_size.k()));

    cutlass::HostTensor<
      typename DualGemm::ElementB,
      typename DualGemm::LayoutB0> tensor_B0(
        cutlass::platform::is_same<typename DualGemm::LayoutB0, cutlass::layout::RowMajor>::value ?
          cutlass::MatrixCoord(batch_count * problem_size.k(), problem_size.n()) :
          cutlass::MatrixCoord(problem_size.k(), batch_count * problem_size.n()));

    cutlass::HostTensor<
      typename DualGemm::ElementC,
      typename DualGemm::LayoutC> tensor_C0(
        cutlass::platform::is_same<typename DualGemm::LayoutC, cutlass::layout::RowMajor>::value ?
          cutlass::MatrixCoord(batch_count * problem_size.m(), problem_size.n()) :
          cutlass::MatrixCoord(problem_size.m(), batch_count * problem_size.n()));

    cutlass::HostTensor<
      typename DualGemm::ElementC,
      typename DualGemm::LayoutScaleBias> tensor_Bias0({batch_count, problem_size.n()});

    cutlass::HostTensor<
      typename DualGemm::ElementC,
      typename DualGemm::LayoutC> tensor_D0(
        cutlass::platform::is_same<typename DualGemm::LayoutC, cutlass::layout::RowMajor>::value ?
          cutlass::MatrixCoord(batch_count * problem_size.m(), problem_size.n()) :
          cutlass::MatrixCoord(problem_size.m(), batch_count * problem_size.n()));

    cutlass::HostTensor<
      typename DualGemm::ElementC,
      typename DualGemm::LayoutC> reference_D0(
        cutlass::platform::is_same<typename DualGemm::LayoutC, cutlass::layout::RowMajor>::value ?
          cutlass::MatrixCoord(batch_count * problem_size.m(), problem_size.n()) :
          cutlass::MatrixCoord(problem_size.m(), batch_count * problem_size.n()));

    cutlass::HostTensor<
      typename DualGemm::ElementB,
      typename DualGemm::LayoutB1> tensor_B1(
        cutlass::platform::is_same<typename DualGemm::LayoutB1, cutlass::layout::RowMajor>::value ?
          cutlass::MatrixCoord(batch_count * problem_size.k(), problem_size.n()) :
          cutlass::MatrixCoord(problem_size.k(), batch_count * problem_size.n()));
    if (broadcast_b1) {
      tensor_B1.resize({problem_size.k(), batch_count});
    }

    cutlass::HostTensor<
      typename DualGemm::ElementC,
      typename DualGemm::LayoutC> tensor_C1(
        cutlass::platform::is_same<typename DualGemm::LayoutC, cutlass::layout::RowMajor>::value ?
          cutlass::MatrixCoord(batch_count * problem_size.m(), problem_size.n()) :
          cutlass::MatrixCoord(problem_size.m(), batch_count * problem_size.n()));
```

**EN**: The fused `run()` starts by allocating tensors whose extents depend on layout and batch count. The allocation formulas are careful about row-major vs column-major packing, and `broadcast_b1` can resize B1 into a `K x batch_count` buffer so the same B1 vector can be shared across all rows/tiles of a batch.
**CN**: 融合版 `run()` 先根据布局和 batch 数量分配张量，尺寸公式会仔细区分 row-major 与 column-major 的打包方式。若启用 `broadcast_b1`，B1 还会被改成 `K x batch_count` 的缓冲区，以便同一个 B1 向量在整个 batch 内被重复使用。

### Logical Block 9 / 逻辑块 9 — lines 621-788

```cpp
    //
    // Batch strides (irrelevant when batch_count == 1)
    //

    int64_t batch_stride_A = problem_size.m() * problem_size.k();
    int64_t batch_stride_B0 = problem_size.k() * problem_size.n();
    int64_t batch_stride_B1 = problem_size.k() * problem_size.n();
    if (broadcast_b1) {
      // B1 is a (column) vector
      batch_stride_B1 = problem_size.k();
    }
    int64_t batch_stride_Bias = problem_size.n();
    int64_t batch_stride_D = problem_size.m() * problem_size.n();

    //
    // Initialize the GEMM operator
    //

    int split_k_slices = DualGemm::kSplitKSerial ? 2 : 1;
    typename cutlass::TensorRef<typename DualGemm::ElementC, typename DualGemm::LayoutC> nullptr_ref{};
    decltype(nullptr_ref) ref_B0, ref_B1;
    if (beta0 != ElementCompute(0)) {
      ref_B0 = {tensor_Bias0.device_data(), typename DualGemm::LayoutC::Stride(0)};
    }
    if (beta1 != ElementCompute(0)) {
      ref_B1 = {tensor_Bias1.device_data(), typename DualGemm::LayoutC::Stride(0)};
    }
    typename DualGemm::Arguments arguments{
      (batch_count > 1 ?
        cutlass::gemm::DualGemmMode::kBatched :
        cutlass::gemm::DualGemmMode::kGemm),
      problem_size,
      tensor_A0.device_ref(),
      tensor_B0.device_ref(),
      ref_B0,
      DualGemm::kStoreD0 ? tensor_D0.device_ref() : nullptr_ref,
      (broadcast_b1 ?
        typename DualGemm::TensorRefB1(tensor_B1.device_data(), 0) :
        tensor_B1.device_ref()),
      ref_B1,
      DualGemm::kStoreD1 ? tensor_D1.device_ref() : nullptr_ref,
      tensor_D2.device_ref(),
      {alpha0, beta0},
      {alpha1, beta1},
      {},
      split_k_slices,
      batch_count,
      batch_stride_A,
      batch_stride_B0,
      batch_stride_B1,
      batch_stride_Bias,
      batch_stride_D,
    };

    //
    // Run the GEMM
    //

    DualGemm b2b_gemm_op;

    cutlass::device_memory::allocation<uint8_t> workspace(b2b_gemm_op.get_workspace_size(arguments));
  
    cutlass::Status status = b2b_gemm_op.can_implement(arguments);

    CUTLASS_CHECK(status);

    status = b2b_gemm_op.initialize(arguments, workspace.get());

    CUTLASS_CHECK(status);

    for(int i = 0; i < warm_ups; i++) {
        status = b2b_gemm_op();
        CUTLASS_CHECK(status);
    }

    if (is_profiling) {
      //
      // Profile the GEMM
      //

      cudaEvent_t start, stop;
      cudaEventCreate(&start);
      cudaEventCreate(&stop);

      cudaEventRecord(start);

      for(int i = 0; i < runs; i++) {
          status = b2b_gemm_op();
          CUTLASS_CHECK(status);
      }

      cudaEventRecord(stop);
      cudaDeviceSynchronize();
      float gemmTime;
      cudaEventElapsedTime(&gemmTime, start, stop);
      std::cout << "Fusion time " << gemmTime / (float)runs << " ms\n";
    }
```

**EN**: This is the heart of the fused launch path. It computes batch strides, chooses `DualGemmMode::kGemm` or `kBatched`, optionally constructs a zero-stride `TensorRefB1` for broadcast, fills a `DualGemm::Arguments` object, allocates workspace, checks `can_implement()`, initializes the operator, warms up, and optionally profiles the fused kernel. This is the host-level reflection of the device/kernel split: the host only prepares metadata and pointers; the device code owns actual fusion.
**CN**: 这里是融合路径的核心启动逻辑。它先计算 batch stride，再选择 `DualGemmMode::kGemm` 或 `kBatched`；若启用广播，则为 B1 构造零 stride 的 `TensorRefB1`；随后填充 `DualGemm::Arguments`，申请 workspace，调用 `can_implement()`，初始化算子，完成预热，并在需要时测量融合 kernel 性能。这正好体现了 device/kernel 分层：主机端只负责准备元数据和指针，真正的融合执行完全在设备端完成。

### Logical Block 10 / 逻辑块 10 — lines 794-901

```cpp
    using GemmUniversal0 = cutlass::gemm::device::GemmUniversal<
      typename DualGemm::ElementA, typename DualGemm::LayoutA,
      typename DualGemm::ElementB, typename DualGemm::LayoutB0,
      typename DualGemm::ElementC, typename DualGemm::LayoutC,
      ElementAccumulator
    >;

    GemmUniversal0 reference_gemm0;

    typename GemmUniversal0::Arguments args0 {
      (batch_count > 1 ?
        cutlass::gemm::GemmUniversalMode::kBatched :
        cutlass::gemm::GemmUniversalMode::kGemm),
      problem_size,
      batch_count,
      {alpha0, beta0},
      tensor_A0.device_data(),
      tensor_B0.device_data(),
      tensor_Bias0.device_data(),
      reference_D0.device_data(),
      batch_stride_A,
      batch_stride_B0,
      batch_stride_Bias,
      batch_stride_D,
      tensor_A0.stride(0),
      tensor_B0.stride(0),
      0,  // zero stride for the bias vector
      reference_D0.stride(0),
    };

    status = reference_gemm0.can_implement(args0);
    CUTLASS_CHECK(status);
    status = reference_gemm0(args0);
    CUTLASS_CHECK(status);

    using GemmUniversal1 = cutlass::gemm::device::GemmUniversal<
      typename DualGemm::ElementA, typename DualGemm::LayoutA,
      typename DualGemm::ElementB, typename DualGemm::LayoutB1,
      typename DualGemm::ElementC, typename DualGemm::LayoutC,
      ElementAccumulator
    >;

    GemmUniversal1 reference_gemm1;

    typename GemmUniversal1::Arguments args1 {
      (batch_count > 1 ?
        cutlass::gemm::GemmUniversalMode::kBatched :
        cutlass::gemm::GemmUniversalMode::kGemm),
      problem_size,
      batch_count,
      {alpha1, beta1},
      tensor_A0.device_data(),
      tensor_B1.device_data(),
      tensor_Bias1.device_data(),
      reference_D1.device_data(),
      batch_stride_A,
      batch_stride_B1,
      batch_stride_Bias,
      batch_stride_D,
      tensor_A0.stride(0),
      (broadcast_b1 ? 0 : tensor_B1.stride(0)),
      0,  // zero stride for the bias vector
      reference_D1.stride(0),
    };

    status = reference_gemm1.can_implement(args1);
    CUTLASS_CHECK(status);
    status = reference_gemm1(args1);
    CUTLASS_CHECK(status);

    if(relu) {
       cutlass::reference::device::TensorReLu(reference_D0.device_view());
       cutlass::reference::device::TensorReLu(reference_D1.device_view()); 
    }

    TensorEpilogueForEach<EpilogueOutputOp2>(reference_D0.device_view(), reference_D1.device_view(), reference_D2.device_view());
    cudaDeviceSynchronize();
    reference_D0.sync_host();
    reference_D1.sync_host();
    reference_D2.sync_host();

    CHECK_GT(cutlass::reference::host::TensorNorm(reference_D0.host_view()), 0);
    CHECK_GT(cutlass::reference::host::TensorNorm(reference_D1.host_view()), 0);
    CHECK_GT(cutlass::reference::host::TensorNorm(tensor_D2.host_view()), 0);
    CHECK_GT(cutlass::reference::host::TensorNorm(reference_D2.host_view()), 0);

    bool passed_out0 = true;
    if (DualGemm::kStoreD0) {
      CHECK_GT(cutlass::reference::host::TensorNorm(tensor_D0.host_view()), 0);
      passed_out0 = cutlass::reference::host::TensorEquals(
        reference_D0.host_view(), 
        tensor_D0.host_view());
    }
    CHECK_TRUE(passed_out0);

    bool passed_out1 = true;
    if (DualGemm::kStoreD1) {
      CHECK_GT(cutlass::reference::host::TensorNorm(tensor_D1.host_view()), 0);
      passed_out1 = cutlass::reference::host::TensorEquals(
        reference_D1.host_view(), 
        tensor_D1.host_view());
    }
    CHECK_TRUE(passed_out1);

    bool passed_out2 = cutlass::reference::host::TensorEquals(
      reference_D2.host_view(), 
      tensor_D2.host_view());
    CHECK_TRUE(passed_out2);
```

**EN**: Reference checking for the fused case uses two `GemmUniversal` launches to reproduce `D0` and `D1`, including batched mode and broadcasted B1 stride handling, then applies `TensorEpilogueForEach<EpilogueOutputOp2>` to build reference `D2`. This is the cleanest place in the example to see the semantic decomposition of the fused operator: `DualGemm = GEMM0 + GEMM1 + custom epilogue2`.
**CN**: 融合路径的参考校验使用两次 `GemmUniversal` 来重建 `D0` 与 `D1`，同时正确处理 batched 模式以及 B1 广播时的零 stride，之后再通过 `TensorEpilogueForEach<EpilogueOutputOp2>` 生成参考 `D2`。这也是整个示例中最直观体现融合语义分解的地方：`DualGemm = GEMM0 + GEMM1 + 自定义 epilogue2`。

### Logical Block 11 / 逻辑块 11 — lines 903-938

```cpp
    bool passed = passed_out0 && passed_out1 && passed_out2;
    if (!passed)
    {
      std::stringstream fname;

      fname << "error_DualGemm_device_fused.txt";
      std::cerr << "Dumping results in " << fname.str() << "\n";

      std::ofstream file(fname.str());

      file 
        << "A0 =\n" << tensor_A0.host_view()
        << "\nB0 =\n" << tensor_B0.host_view()
        << "\nC0 =\n" << tensor_C0.host_view()
        << "\nBias0:\n" << tensor_Bias0.host_view() << "\n"
        << "\nB1 =\n" << tensor_B1.host_view()
        << "\nC1 =\n" << tensor_C1.host_view()
        << "\nBias1:\n" << tensor_Bias1.host_view() << "\n"
        << "\n\nReference0 =\n" << reference_D0.host_view()
        << "\nComputed0 =\n" << tensor_D0.host_view()
        << "\n\nReference1 =\n" << reference_D1.host_view()
        << "\nComputed1 =\n" << tensor_D1.host_view()
        << "\n\nReference2 =\n" << reference_D2.host_view()
        << "\nComputed2 =\n" << tensor_D2.host_view();
    }
    //std::cout << "A0 " << tensor_A0.host_view() << std::endl;
    // std::cout << "reference_D0 " << reference_D0.host_view() << std::endl;
    // std::cout << "reference_D1 " << reference_D1.host_view() << std::endl;
    // std::cout << "reference_D2 " << reference_D2.host_view() << std::endl;
    //std::cout << "reference_D0 " << reference_D0.host_view() << std::endl;
    return passed;
  }

};

////////////////////////////////////////////////////////////////////////////////
```

**EN**: The final block aggregates pass/fail status, dumps tensors to a text file when mismatches occur, and returns a boolean result to the caller. For a tutorial reader, this is the “trust but verify” layer that makes every earlier template instantiation observable and debuggable.
**CN**: 最后这段代码汇总通过/失败状态，在比较失败时把张量转储到文本文件，并将布尔结果返回给上层调用者。对教程读者而言，这是“先相信，再验证”的收尾层，让前面所有模板实例化都变得可观察、可调试。

---

## Key Concepts / 关键概念
**EN**: `NonFusedDualGemmRun` runs two separate GEMMs; `DualFusedGemmRun` runs the fused `device::DualGemm`; `TensorEpilogueForEach` reconstructs the final SiLU-and-multiply stage for reference checking.
**CN**: 组合关系：`NonFusedDualGemmRun` 运行两个独立 GEMM；`DualFusedGemmRun` 运行融合后的 `device::DualGemm`；`TensorEpilogueForEach` 则在参考路径中重建最终的 SiLU-and-multiply 阶段。

**EN**: Dual-GEMM host flow: allocate tensors → initialize/broadcast/batch metadata → launch fused operator → reconstruct two reference GEMMs → apply the same SiLU-and-multiply helper to validate `D2`.
**CN**: Dual-GEMM 主机端流程：分配张量 → 初始化并组织广播/batch 元数据 → 启动融合算子 → 重建两个参考 GEMM → 用同一个 SiLU-and-multiply helper 校验 `D2`。

## Dependencies / 依赖项
**EN**: Standard-library dependency: `<iostream>` is used directly in this file.
**CN**: 标准库依赖：`<iostream>` 在该文件中被直接使用。

**EN**: Standard-library dependency: `<fstream>` is used directly in this file.
**CN**: 标准库依赖：`<fstream>` 在该文件中被直接使用。

**EN**: Standard-library dependency: `<sstream>` is used directly in this file.
**CN**: 标准库依赖：`<sstream>` 在该文件中被直接使用。

**EN**: Standard-library dependency: `<type_traits>` is used directly in this file.
**CN**: 标准库依赖：`<type_traits>` 在该文件中被直接使用。

**EN**: CUTLASS dependency: `cutlass/util/host_tensor.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/host_tensor.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/tensor_view_io.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/tensor_view_io.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/distribution.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/distribution.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/reference/host/tensor_fill.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/reference/host/tensor_fill.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/reference/host/tensor_copy.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/reference/host/tensor_copy.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/reference/host/tensor_compare.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/reference/host/tensor_compare.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/reference/host/tensor_norm.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/reference/host/tensor_norm.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/reference/device/gemm.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/reference/device/gemm.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/reference/device/tensor_relu.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/reference/device/tensor_relu.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/platform/platform.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/platform/platform.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/gemm/gemm.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/gemm/gemm.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/gemm/device/gemm_universal.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/gemm/device/gemm_universal.h` 提供该文件直接使用的库级原语。

**EN**: Project-local dependency: `dual_gemm_common.h` provides helper code included directly by this file.
**CN**: 项目内依赖：`dual_gemm_common.h` 提供该文件直接包含的辅助代码。

**EN**: Project-local dependency: `helper.h` provides helper code included directly by this file.
**CN**: 项目内依赖：`helper.h` 提供该文件直接包含的辅助代码。
