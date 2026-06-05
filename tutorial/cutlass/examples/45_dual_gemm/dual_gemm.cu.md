# dual_gemm.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/45_dual_gemm/dual_gemm.cu`  
**Purpose / 用途**: Top-level DualGemm example driver This translation unit is the narrative entry point for the fused example. It defines the problem sizes, chooses data types and epilogues, instantiates baseline and fused kernels, and registers the scenarios that demonstrate plain, batched, broadcasted, and batched+broadcasted DualGemm execution. / DualGemm 顶层示例驱动 这个编译单元是融合示例的叙事入口。它定义问题规模，选择数据类型与 epilogue，实例化基线与融合 kernel，并注册普通、batched、broadcast 以及 batched+broadcast DualGemm 场景。

---

## Line-by-Line Analysis / 逐行分析

### Logical Block 1 / 逻辑块 1 — lines 32-61

```cpp
/*! \file
    \brief CUTLASS Dual-GEMM Example.

    Fused kernel that outputs `D0` and `D1`.
    We assume that B0/B1 have the same shape/layout

```
D0 = epilogue0(X @ B0, C0)
D1 = epilogue1(X @ B1, C1)
D2 = element_wise(D0, D1)
```
    D0 and D1 will be optionally stored in gmem (`kStoreD0` / `kStoreD1`)
*/

#include <iostream>

#include "cutlass/cutlass.h"
#include "cutlass/gemm/device/gemm.h"

#include "cutlass/util/host_tensor.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/gemm.h"

#include "device/dual_gemm.h"
#include "thread/left_silu_and_mul.h"
#include "dual_gemm_run.h"
#include "test_run.h"
```

**EN**: The file header states the mathematical contract clearly: two GEMMs share the same left operand `X`, produce `D0` and `D1` through their own epilogues, and then feed a final elementwise stage that produces `D2`. The includes mirror that layering: device wrapper, custom thread-level functor, run harness, and test harness.
**CN**: 文件头部把数学关系写得很清楚：两个 GEMM 共享左操作数 `X`，分别经过各自 epilogue 生成 `D0` 和 `D1`，然后再进入最终逐元素阶段生成 `D2`。包含关系也正好对应这层分工：设备包装层、自定义线程级 functor、运行框架与测试框架。

### Logical Block 2 / 逻辑块 2 — lines 66-117

```cpp
cutlass::gemm::GemmCoord problem_size(4096, 4096, 8192);
cutlass::gemm::GemmCoord batch_problem_size(321, 256, 512);

constexpr int kStages = 3;
constexpr bool kSplitKSerial = false;
constexpr bool kUseBias = true;
constexpr int kBatchCount = 37;


#if 0
using ElementOperandA = cutlass::bfloat16_t;
using ElementOperandB = cutlass::bfloat16_t;
using ElementOutput = cutlass::bfloat16_t;
using ElementAccumulator = float;
using ElementCompute = float;
#else
using ElementOperandA = cutlass::half_t;
using ElementOperandB = cutlass::half_t;
using ElementOutput = cutlass::half_t;
using ElementAccumulator = cutlass::half_t;
using ElementCompute = cutlass::half_t;
#endif

constexpr auto kScaleType = kUseBias ? cutlass::epilogue::thread::ScaleType::NoBetaScaling : (
  // No bias
  kSplitKSerial ? cutlass::epilogue::thread::ScaleType::Default : cutlass::epilogue::thread::ScaleType::Nothing
);
using EpilogueOutputOp0 = cutlass::epilogue::thread::LinearCombination<
  ElementOutput,
  128 / cutlass::sizeof_bits<ElementOutput>::value,
  ElementAccumulator,
  ElementCompute,
  kScaleType
>;
using EpilogueOutputOp1 = cutlass::epilogue::thread::LinearCombination<
  ElementOutput,
  128 / cutlass::sizeof_bits<ElementOutput>::value,
  ElementAccumulator,
  ElementCompute,
  kScaleType
>;
using EpilogueOutputOp2 = cutlass::epilogue::thread::LeftSiLUAndMul<
  ElementOutput,
  128 / cutlass::sizeof_bits<ElementOutput>::value,
  ElementOutput,
  ElementCompute
>;

const ElementCompute alpha0 = ElementCompute(1);
const ElementCompute beta0 = ElementCompute(kUseBias ? 1 : 0);
const ElementCompute alpha1 = ElementCompute(1);
const ElementCompute beta1 = ElementCompute(kUseBias ? 1 : 0);
```

**EN**: This configuration block fixes representative problem sizes, enables three-stage multistage MMA, chooses FP16 as the active type path, and defines the three epilogue operators. `EpilogueOutputOp0/1` are standard `LinearCombination` operators, while `EpilogueOutputOp2` is the custom `LeftSiLUAndMul`, which means the fused pipeline computes `D2 = SiLU(D0) * D1` after the two GEMMs finish.
**CN**: 这一段配置了代表性问题规模，启用三阶段 multistage MMA，选择 FP16 作为当前生效的数据路径，并定义了三个 epilogue 算子。`EpilogueOutputOp0/1` 是标准 `LinearCombination`，而 `EpilogueOutputOp2` 则是自定义的 `LeftSiLUAndMul`，因此整个融合流水线最终计算的是 `D2 = SiLU(D0) * D1`。

### Logical Block 3 / 逻辑块 3 — lines 119-184

```cpp
bool run_nonfused_gemm_f16_sm80() {
  using ThreadblockShape = cutlass::gemm::GemmShape<128, 128, 32>;
  using WarpShape = cutlass::gemm::GemmShape<64, 64, 32>;
  using InstructionShape = cutlass::gemm::GemmShape<16, 8, 16>;

  using Gemm0 = cutlass::gemm::device::Gemm<
    ElementOperandA,
    cutlass::layout::RowMajor,
    ElementOperandB,
    cutlass::layout::ColumnMajor,
    ElementOutput,
    cutlass::layout::RowMajor,
    ElementAccumulator,
    cutlass::arch::OpClassTensorOp,
    cutlass::arch::Sm80,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueOutputOp0,
    cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<1>,
    kStages,
    8,
    8,
    kSplitKSerial
  >;
  using Gemm1 = cutlass::gemm::device::Gemm<
    ElementOperandA,
    cutlass::layout::RowMajor,
    ElementOperandB,
    cutlass::layout::ColumnMajor,
    ElementOutput,
    cutlass::layout::RowMajor,
    ElementAccumulator,
    cutlass::arch::OpClassTensorOp,
    cutlass::arch::Sm80,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueOutputOp1,
    cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<1>,
    kStages,
    8,
    8,
    kSplitKSerial
  >;

  NonFusedDualGemmRun<Gemm0, Gemm1> nonFusedGemm;

  std::cout << "Running Non-fused GEMMs FP16 TN GEMMs...\n";

  bool pass = nonFusedGemm.run(
    problem_size,
    alpha0,
    beta0,
    alpha1,
    beta1,
    true  /* is_profiling */
  );

  if(pass)
    std::cout << "Pass\n";
  else
    std::cout << "Fail\n";

  return pass;
}
```

**EN**: The non-fused baseline builds two independent `cutlass::gemm::device::Gemm` operators that share the same A matrix but use different B matrices and epilogues. It is important for performance comparison: the rest of the example can measure whether the fused path saves global-memory traffic and launch overhead compared with separate kernels.
**CN**: 这个非融合基线路径构造了两个独立的 `cutlass::gemm::device::Gemm` 算子：它们共享同一个 A 矩阵，但使用不同的 B 矩阵和 epilogue。它对性能对比非常重要，因为后续融合版本正是用来衡量是否能相对独立 kernel 节省全局内存访问与 launch 开销。

### Logical Block 4 / 逻辑块 4 — lines 186-211

```cpp
template <typename T>
struct LeftSiLUAndMul {
  struct Params{};
  CUTLASS_HOST_DEVICE LeftSiLUAndMul(Params p) {}

  CUTLASS_HOST_DEVICE void set_k_partition(int, int) {}

  CUTLASS_HOST_DEVICE T operator() (
    T const &lhs, 
    T const &rhs) const {
    cutlass::epilogue::thread::SiLu<T> silu;
    cutlass::multiplies<T> mul;
    auto silu_lhs = silu(lhs);
    return mul(silu_lhs, rhs);
  }

  template <int kCount>
  CUTLASS_HOST_DEVICE cutlass::Array<T, kCount> operator() (
    cutlass::Array<T, kCount> const &lhs, 
    cutlass::Array<T, kCount> const &rhs) const {
    cutlass::epilogue::thread::SiLu<T> silu;
    cutlass::multiplies<T> mul;
    auto silu_lhs = silu(lhs);
    return mul(silu_lhs, rhs);
  }
};
```

**EN**: A second, local `LeftSiLUAndMul` template appears here as a compact scalar/vector mirror of the header implementation. The active alias earlier already points at `cutlass::epilogue::thread::LeftSiLUAndMul`, so this local copy mainly serves as an in-file reference for the intended fused math rather than the selected production type.
**CN**: 这里又定义了一个本地版 `LeftSiLUAndMul` 模板，语义上是头文件实现的精简镜像。前面的活跃别名已经指向 `cutlass::epilogue::thread::LeftSiLUAndMul`，因此这个本地版本更像是为了在文件内展示目标融合数学，而不是当前真正被选中的生产实现。

### Logical Block 5 / 逻辑块 5 — lines 213-264

```cpp
bool run_fused_gemm_f16_sm80_shmem() {
  using ThreadblockShape = cutlass::gemm::GemmShape<128, 64, 32>;
  using WarpShape = cutlass::gemm::GemmShape<64, 32, 32>;
  using InstructionShape = cutlass::gemm::GemmShape<16, 8, 16>;

  // Optionally, we might not need intermediate GEMM outputs
  constexpr bool kStoreD0 = true;
  constexpr bool kStoreD1 = true;

  using DualGemm = cutlass::gemm::device::DualGemm<
    ElementOperandA,
    cutlass::layout::RowMajor,
    ElementOperandB,
    cutlass::layout::ColumnMajor,
    cutlass::layout::ColumnMajor,
    ElementOutput,
    cutlass::layout::RowMajor,
    ElementAccumulator,
    cutlass::arch::OpClassTensorOp,
    cutlass::arch::Sm80,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueOutputOp0,
    EpilogueOutputOp1,
    EpilogueOutputOp2,
    cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<1>,
    kStages,
    kStoreD0,
    kStoreD1,
    kSplitKSerial
  >;

  DualFusedGemmRun<DualGemm> fusedGemm;

  std::cout << "Running Fused FP16 TN GEMMs + Epilogue2...\n";

  bool passed = fusedGemm.run(
    problem_size,
    alpha0,
    beta0,
    alpha1,
    beta1
  );

  if(passed)
    std::cout << "Pass\n";
  else
    std::cout << "Fail\n";

  return passed;
}
```

**EN**: This is the core fused case. It instantiates `cutlass::gemm::device::DualGemm` with one row-major A, two column-major B operands, row-major outputs, Tensor Core SM80 math, the custom epilogue triplet, and storage flags for `D0`/`D1`. The host code then hands everything to `DualFusedGemmRun`, which hides allocation, launch, profiling, and reference checking.
**CN**: 这是最核心的融合场景。这里用一份 row-major A、两份 column-major B、row-major 输出、SM80 Tensor Core 数学路径以及自定义 epilogue 三元组来实例化 `cutlass::gemm::device::DualGemm`，并通过 `kStoreD0/kStoreD1` 控制是否保留中间结果。主机端随后把执行细节交给 `DualFusedGemmRun`，后者负责分配、启动、性能测量和结果校验。

### Logical Block 6 / 逻辑块 6 — lines 266-320

```cpp
bool run_batched_fused_gemm_f16_sm80_shmem() {
  using ThreadblockShape = cutlass::gemm::GemmShape<128, 64, 32>;
  using WarpShape = cutlass::gemm::GemmShape<64, 32, 32>;
  using InstructionShape = cutlass::gemm::GemmShape<16, 8, 16>;

  // Optionally, we might not need intermediate GEMM outputs
  constexpr bool kStoreD0 = true;
  constexpr bool kStoreD1 = true;

  using DualGemm = cutlass::gemm::device::DualGemm<
    ElementOperandA,
    cutlass::layout::RowMajor,
    ElementOperandB,
    cutlass::layout::ColumnMajor,
    cutlass::layout::ColumnMajor,
    ElementOutput,
    cutlass::layout::RowMajor,
    ElementAccumulator,
    cutlass::arch::OpClassTensorOp,
    cutlass::arch::Sm80,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueOutputOp0,
    EpilogueOutputOp1,
    EpilogueOutputOp2,
    cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<1>,
    kStages,
    kStoreD0,
    kStoreD1,
    kSplitKSerial
  >;

  DualFusedGemmRun<DualGemm> fusedGemm;

  std::cout << "Running Batched Fused FP16 TN GEMMs + Epilogue2...\n";

  bool passed = fusedGemm.run(
    batch_problem_size,
    alpha0,
    beta0,
    alpha1,
    beta1,
    kBatchCount,
    false,  /* broadcast_b1 */
    false   /* is_profiling */
  );

  if(passed)
    std::cout << "Pass\n";
  else
    std::cout << "Fail\n";

  return passed;
}
```

**EN**: The batched fused variant keeps the same compute shape but switches the runtime mode to batched execution by passing `batch_problem_size` and a non-unit `kBatchCount`. This demonstrates the first meaning of `DualGemmMode`: the grid K dimension can be interpreted as batch index rather than split-K slice index.
**CN**: batched 融合版本保持同样的计算 tile 形状，但通过传入 `batch_problem_size` 和非 1 的 `kBatchCount` 切换到 batched 执行。这展示了 `DualGemmMode` 的第一层含义：网格的 K 维可以不再表示 split-K 切片，而是解释为 batch 索引。

### Logical Block 7 / 逻辑块 7 — lines 322-377

```cpp
bool run_broadcast_fused_gemm_f16_sm80_shmem() {
  using ThreadblockShape = cutlass::gemm::GemmShape<128, 64, 32>;
  using WarpShape = cutlass::gemm::GemmShape<64, 32, 32>;
  using InstructionShape = cutlass::gemm::GemmShape<16, 8, 16>;

  // Optionally, we might not need intermediate GEMM outputs
  constexpr bool kStoreD0 = true;
  constexpr bool kStoreD1 = true;

  using DualGemm = cutlass::gemm::device::DualGemm<
    ElementOperandA,
    cutlass::layout::RowMajor,
    ElementOperandB,
    // different LayoutB0 and B1
    cutlass::layout::RowMajor,
    cutlass::layout::ColumnMajor,
    ElementOutput,
    cutlass::layout::RowMajor,
    ElementAccumulator,
    cutlass::arch::OpClassTensorOp,
    cutlass::arch::Sm80,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueOutputOp0,
    EpilogueOutputOp1,
    EpilogueOutputOp2,
    cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<1>,
    kStages,
    kStoreD0,
    kStoreD1,
    kSplitKSerial
  >;

  DualFusedGemmRun<DualGemm> fusedGemm;

  std::cout << "Running Broadcast Fused FP16 TN GEMMs + Epilogue2...\n";

  bool passed = fusedGemm.run(
    problem_size,
    alpha0,
    beta0,
    alpha1,
    beta1,
    1,     /* batch_count */
    true,  /* broadcast_b1 */
    true   /* is_profiling */
  );
```

**EN**: The broadcast case changes only `LayoutB0`/`LayoutB1`: `B0` is row-major while `B1` stays column-major, and the run helper is told to broadcast B1. The important idea is that the dual-GEMM machinery is generic enough to let the two right operands use different layouts, as long as their iterators and epilogues are consistent.
**CN**: broadcast 场景只改动了 `LayoutB0`/`LayoutB1`：`B0` 变为 row-major，而 `B1` 仍为 column-major，同时运行辅助器被告知对 B1 做广播。关键点在于 dual-GEMM 框架对两个右操作数的布局是泛化的，只要对应迭代器与 epilogue 能匹配即可。

### Logical Block 8 / 逻辑块 8 — lines 379-456

```cpp
bool run_batched_broadcast_fused_gemm_f16_sm80_shmem() {
  using ThreadblockShape = cutlass::gemm::GemmShape<128, 64, 32>;
  using WarpShape = cutlass::gemm::GemmShape<64, 32, 32>;
  using InstructionShape = cutlass::gemm::GemmShape<16, 8, 16>;

  // Optionally, we might not need intermediate GEMM outputs
  constexpr bool kStoreD0 = true;
  constexpr bool kStoreD1 = true;

  using DualGemm = cutlass::gemm::device::DualGemm<
    ElementOperandA,
    cutlass::layout::RowMajor,
    ElementOperandB,
    // different LayoutB0 and B1
    cutlass::layout::RowMajor,
    cutlass::layout::ColumnMajor,
    ElementOutput,
    cutlass::layout::RowMajor,
    ElementAccumulator,
    cutlass::arch::OpClassTensorOp,
    cutlass::arch::Sm80,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueOutputOp0,
    EpilogueOutputOp1,
    EpilogueOutputOp2,
    cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<1>,
    kStages,
    kStoreD0,
    kStoreD1,
    kSplitKSerial
  >;

  DualFusedGemmRun<DualGemm> fusedGemm;

  std::cout << "Running Batch Broadcast Fused FP16 TN GEMMs + Epilogue2...\n";

  bool passed = fusedGemm.run(
    batch_problem_size,
    alpha0,
    beta0,
    alpha1,
    beta1,
    kBatchCount,
    true,  /* broadcast_b1 */
    false  /* is_profiling */
  );

  if(passed)
    std::cout << "Pass\n";
  else
    std::cout << "Fail\n";

  return passed;
}

int main() {

  std::vector<bool (*)()>funcs = {
    &run_nonfused_gemm_f16_sm80,
    &run_fused_gemm_f16_sm80_shmem,
    &run_batched_fused_gemm_f16_sm80_shmem,
    &run_broadcast_fused_gemm_f16_sm80_shmem,
    &run_batched_broadcast_fused_gemm_f16_sm80_shmem
  };

  std::string test_name = (
    "dual-gemm f16 bias=" +
    std::to_string(kUseBias) +
    " split_k_serial=" +
    std::to_string(kSplitKSerial) +
    " batch_count=" +
    std::to_string(kBatchCount)
  );

  return testRun(80, funcs, test_name);
}
```

**EN**: The final run function combines batching and B1 broadcasting, then `main()` registers all scenarios and defers execution policy to `testRun(80, ...)`. In other words, this file specifies *what* variants to demonstrate, while the helper files decide *how* to allocate, launch, and validate them.
**CN**: 最后一个运行函数把 batching 与 B1 广播组合起来，随后 `main()` 注册全部场景并把执行策略交给 `testRun(80, ...)`。换句话说，这个文件负责定义“展示哪些变体”，而真正的“如何分配、启动和验证”则交由辅助文件完成。

---

## Key Concepts / 关键概念
**EN**: this file is the host-facing driver. `dual_gemm_run.h` provides the harnesses, `device/dual_gemm.h` provides the fused operator, and `thread/left_silu_and_mul.h` provides the custom D2 epilogue functor.
**CN**: 组合关系：该文件是面向主机的驱动层；`dual_gemm_run.h` 提供运行与校验框架，`device/dual_gemm.h` 提供融合算子，`thread/left_silu_and_mul.h` 提供自定义的 D2 epilogue functor。

**EN**: End-to-end flow: choose epilogues → instantiate baseline/fused operators → run host harnesses → compare `D0`, `D1`, and fused `D2` against reference paths.
**CN**: 端到端流程：选择 epilogue → 实例化基线/融合算子 → 调用主机辅助框架 → 将 `D0`、`D1` 与融合得到的 `D2` 与参考路径逐一比较。

## Dependencies / 依赖项
**EN**: Standard-library dependency: `<iostream>` is used directly in this file.
**CN**: 标准库依赖：`<iostream>` 在该文件中被直接使用。

**EN**: CUTLASS dependency: `cutlass/cutlass.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/cutlass.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/gemm/device/gemm.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/gemm/device/gemm.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/host_tensor.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/host_tensor.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/tensor_view_io.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/tensor_view_io.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/reference/host/tensor_fill.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/reference/host/tensor_fill.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/reference/host/tensor_copy.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/reference/host/tensor_copy.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/reference/host/tensor_compare.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/reference/host/tensor_compare.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/reference/host/gemm.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/reference/host/gemm.h` 提供该文件直接使用的库级原语。

**EN**: Project-local dependency: `device/dual_gemm.h` provides helper code included directly by this file.
**CN**: 项目内依赖：`device/dual_gemm.h` 提供该文件直接包含的辅助代码。

**EN**: Project-local dependency: `thread/left_silu_and_mul.h` provides helper code included directly by this file.
**CN**: 项目内依赖：`thread/left_silu_and_mul.h` 提供该文件直接包含的辅助代码。

**EN**: Project-local dependency: `dual_gemm_run.h` provides helper code included directly by this file.
**CN**: 项目内依赖：`dual_gemm_run.h` 提供该文件直接包含的辅助代码。

**EN**: Project-local dependency: `test_run.h` provides helper code included directly by this file.
**CN**: 项目内依赖：`test_run.h` 提供该文件直接包含的辅助代码。
