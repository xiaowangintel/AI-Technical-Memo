# 49_collective_builder.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/49_hopper_gemm_with_collective_builder/49_collective_builder.cu`  
**Purpose / 用途**: Demonstrates how CUTLASS 3’s `CollectiveBuilder` assembles Hopper GEMM mainloops and epilogues from a compact set of template inputs, while also showing schedule overrides, stage-count overrides, stream-K scheduling, and a custom epilogue visitor tree (EVT). / 演示 CUTLASS 3 的 `CollectiveBuilder` 如何根据少量模板输入组装 Hopper GEMM 的 mainloop 与 epilogue，同时展示调度覆盖、stage 数覆盖、stream-K 调度以及自定义 epilogue visitor tree（EVT）。

---

## Line-by-Line Analysis / 逐行分析

### Block 1 (Lines 1-119)
```cpp
/*! \file
    \brief Hopper GEMM example leveraging collective operation builders.
    ...
    CUTLASS 3 introduces the CollectiveBuilder ...
*/
```
**EN**: The long prologue is not filler; it explains the exact design pressure behind `CollectiveBuilder`. CUTLASS 2’s `DefaultGemmConfigurations` offered fixed default configurations, but Hopper expands the configuration space dramatically: TMA vs. non-TMA loading, warp specialization, persistence, different epilogues, and stage counts. The file positions `CollectiveBuilder` as a type-level factory that fills this gap by inferring compatible collectives from generic properties and `Auto` tags.

**CN**: 这段很长的开场说明并不是“背景废话”，而是在交代 `CollectiveBuilder` 的设计动机。CUTLASS 2 的 `DefaultGemmConfigurations` 只能提供固定默认配置，但 Hopper 把配置空间显著放大了：是否用 TMA、是否 warp specialization、是否 persistent、不同的 epilogue、不同 stage 数等等。该文件把 `CollectiveBuilder` 定位成一个类型级工厂，用通用属性和 `Auto` tag 去推断兼容的 collective，从而填补这一空白。

### Block 2 (Lines 121-227)
```cpp
struct Options {
  bool help;
  bool error;
  int m, n, k, l;
  float alpha, beta;
  ...
};

template <class Element>
bool initialize_block(cutlass::DeviceAllocation<Element>& block, uint64_t seed=2023) {
  ...
}
```
**EN**: The runtime surface is intentionally small: batched GEMM size (`m/n/k/l`) and epilogue scalars (`alpha`, `beta`). `initialize_block()` is a generic device-buffer initializer that chooses a value range from the element bit width. This keeps the rest of the example focused on builder mechanics rather than tensor setup.

**CN**: 运行时接口被刻意保持得很小：只暴露 batched GEMM 尺寸（`m/n/k/l`）和 epilogue 标量（`alpha`、`beta`）。`initialize_block()` 是一个通用设备缓冲区初始化器，会根据元素位宽选择随机取值范围，从而让后续示例更专注于 builder 机制本身，而不是张量准备细节。

### Block 3 (Lines 231-337)
```cpp
template <
  class MainloopScheduleType = cutlass::gemm::collective::KernelScheduleAuto,
  class EpilogueScheduleType = cutlass::epilogue::collective::EpilogueScheduleAuto,
  class StageCountType = cutlass::gemm::collective::StageCountAuto,
  class TileSchedulerType = cutlass::gemm::PersistentScheduler,
  bool UseCustomEVT = false>
struct ExampleRunner {
  ...
  using CustomEVT = cutlass::epilogue::fusion::Sm90EVT<...>;
  using DefaultOperation = cutlass::epilogue::fusion::LinearCombination<...>;
  using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<...>::CollectiveOp;
  using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<...>::CollectiveOp;
  using GemmKernel = cutlass::gemm::kernel::GemmUniversal<...>;
};
```
**EN**: `ExampleRunner` is the core experiment harness. Its template parameters are exactly the knobs `CollectiveBuilder` is meant to simplify: mainloop schedule, epilogue schedule, stage count, tile scheduler, and whether to use a custom EVT. The epilogue builder is parameterized with either a hand-written `CustomEVT` or the stock `LinearCombination`. The mainloop builder shows a particularly important pattern: when stage count is `Auto`, it becomes `StageCountAutoCarveout<sizeof(CollectiveEpilogue::SharedStorage)>`, so automatic stage selection reserves shared memory for the chosen epilogue. That is the precise collective-builder contract in this example.

**CN**: `ExampleRunner` 是整个文件的核心实验框架。它的模板参数正是 `CollectiveBuilder` 想要简化的那些关键旋钮：mainloop 调度、epilogue 调度、stage 数、tile scheduler，以及是否使用自定义 EVT。epilogue builder 会在手写 `CustomEVT` 与默认 `LinearCombination` 之间二选一。mainloop builder 还展示了一个非常关键的模式：当 stage 数为 `Auto` 时，会替换成 `StageCountAutoCarveout<sizeof(CollectiveEpilogue::SharedStorage)>`，即自动 stage 选择时要先给 epilogue 预留共享内存。这正是该示例里 collective builder 契约的精髓。

### Block 4 (Lines 339-430)
```cpp
using StrideA = typename Gemm::GemmKernel::StrideA;
...
bool verify(const ProblemShapeType& problem_size, float alpha, float beta) {
  ...
  cutlass::reference::device::GemmComplex(..., L, M * K, K * N, M * N, M * N);
}

void initialize(const ProblemShapeType& problem_size) {
  ...
  stride_A = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(M, K, L));
  ...
}
```
**EN**: Once the builders have produced a concrete `GemmKernel`, the example extracts the exact stride types from that kernel and uses them consistently for initialization and verification. `initialize()` derives packed CuTe strides from the kernel’s internal layout model, allocates one contiguous block per tensor, and fills A/B/C. `verify()` runs a reference batched GEMM (`GemmComplex` is reused as a convenient batched helper) and then compares device outputs.

**CN**: 一旦 builder 生成了具体的 `GemmKernel`，示例就从该内核中提取精确的 stride 类型，并在初始化和校验中统一使用。`initialize()` 按照内核内部布局模型生成 packed CuTe stride，分别为各张量分配连续内存并填充 A/B/C。`verify()` 运行一个参考 batched GEMM（这里复用 `GemmComplex` 作为方便的 batched helper），随后比较设备输出结果。

### Block 5 (Lines 432-512)
```cpp
typename Gemm::Arguments arguments{
  cutlass::gemm::GemmUniversalMode::kGemm,
  problem_size,
  {block_A.get(), stride_A, block_B.get(), stride_B},
  {{}, block_C.get(), stride_C, block_D.get(), stride_D},
  hw_info
};

if constexpr (UseCustomEVT) {
  arguments.epilogue.thread = {
    {{options.beta}},
    {},
    {{{options.alpha}}, {}, {}},
    {}
  };
}
```
**EN**: `run()` turns the builder-produced types into a real launch. The important detail is how epilogue arguments differ between predefined fusion ops and a custom EVT. For the default linear combination, `alpha` and `beta` are named flat fields. For the custom EVT, the argument object mirrors the tree structure exactly: each node stores child arguments followed by its own operator arguments. The rest of `run()` follows the standard CUTLASS lifecycle: workspace query, `can_implement`, `initialize`, `run`, synchronize, verify.

**CN**: `run()` 把 builder 生成的类型真正转成一次内核 launch。这里最值得注意的是自定义 EVT 与预定义 fusion op 在参数组织方式上的差异：默认线性组合使用扁平、具名的 `alpha`/`beta` 字段；自定义 EVT 则必须严格镜像树结构——每个节点先放子节点参数，再放自身算子参数。其余流程则遵循标准 CUTLASS 生命周期：查询 workspace、`can_implement`、`initialize`、`run`、同步、校验。

### Block 6 (Lines 520-585)
```cpp
void print_result(const std::string& description, bool passed) {
  std::cout << description << ": " << (passed ? "Passed" : "Failed") << std::endl;
}

int main(int argc, char const **args) {
  cudaDeviceProp props;
  ...
  cutlass::KernelHardwareInfo hw_info;
  hw_info.device_id = 0;
  hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);
```
**EN**: Before the main experiment loop, the file defines a tiny `print_result()` helper so every runner emits a uniform label plus pass/fail summary. `main()` then enforces the environment contract—CUDA 12+ and Hopper (SM90)—and materializes `KernelHardwareInfo`, which Hopper kernels use for scheduler decisions such as persistent and stream-K behavior. This makes the later runner instantiations comparable under one consistent hardware descriptor.

**CN**: 在主实验循环之前，文件先定义了一个很小的 `print_result()` 辅助函数，让每个 runner 都以统一格式输出说明文字和通过/失败结果。随后 `main()` 检查运行环境——必须是 CUDA 12+ 且 GPU 为 Hopper（SM90）——并构造 `KernelHardwareInfo`；Hopper 内核会依赖这份硬件信息做 persistent、stream-K 等调度决策。这样后续不同 runner 实例的比较就都建立在一致的硬件描述之上。

### Block 7 (Lines 587-659)
```cpp
ExampleRunner<> auto_schedule_auto_stage_runner;
ExampleRunner<..., _5> auto_schedule_5_stage_runner;
ExampleRunner<cutlass::gemm::KernelTma, cutlass::epilogue::NoSmemWarpSpecialized> ...;
ExampleRunner<cutlass::gemm::KernelTmaWarpSpecialized, cutlass::epilogue::NoSmemWarpSpecialized> ...;
ExampleRunner<cutlass::gemm::KernelTmaWarpSpecializedPingpong, cutlass::epilogue::TmaWarpSpecialized> ...;
ExampleRunner<..., cutlass::gemm::StreamKScheduler> ...;
ExampleRunner<..., true> ws_cooperative_schedule_auto_stage_custom_evt_runner;
```
**EN**: The final section systematically exercises the builder interface. It starts with full auto-selection, then overrides only the stage count, then progressively pins specific Hopper schedules: plain TMA, warp-specialized TMA, ping-pong warp-specialized, cooperative warp-specialized with stream-K, and finally the same cooperative schedule plus a custom EVT. This sequence is the clearest practical answer to “what does CollectiveBuilder buy me?”—you can vary one policy dimension without manually rebuilding every low-level collective type.

**CN**: 最后一段系统化地演示了 builder 接口的使用方式：先完全自动选择，再只覆盖 stage 数，随后逐步固定具体的 Hopper 调度：普通 TMA、warp-specialized TMA、ping-pong warp-specialized、带 stream-K 的 cooperative warp-specialized，最后再在 cooperative 调度基础上叠加自定义 EVT。这一串实验最直接地回答了“CollectiveBuilder 到底带来了什么”：你可以只改一个策略维度，而不必手动重写全部底层 collective 类型。

---

## Key Concepts / 关键概念

- **CollectiveBuilder as a type factory / 作为类型工厂的 CollectiveBuilder**: The builders synthesize compatible mainloop and epilogue collectives from high-level tags. / builder 根据高层 tag 合成兼容的 mainloop 与 epilogue collective。
- **Stage-count carveout / stage 数内存预留**: Automatic stage selection must account for epilogue shared storage. / 自动 stage 选择必须为 epilogue 共享内存预留空间。
- **Schedule compatibility / 调度兼容性**: Mainloop and epilogue schedules must both be `Auto`, or both be explicitly compatible non-`Auto` tags. / mainloop 与 epilogue 要么都使用 `Auto`，要么都使用显式且彼此兼容的非 `Auto` tag。
- **EVT customization / EVT 自定义**: Hopper TMA warp-specialized epilogues can express fused math as a visitor tree instead of a monolithic epilogue class. / Hopper 的 TMA warp-specialized epilogue 可用 visitor tree 表达融合计算，而不必手写整体 epilogue 类。

## Dependencies / 依赖项

- `iostream` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cute/tensor.hpp` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/cutlass.h` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/tensor_ref.h` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/epilogue/collective/default_epilogue.hpp` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/epilogue/thread/linear_combination.h` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/gemm/dispatch_policy.hpp` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/gemm/collective/collective_builder.hpp` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/epilogue/collective/collective_builder.hpp` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/gemm/device/gemm_universal_adapter.h` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/gemm/kernel/gemm_universal.hpp` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/gemm/kernel/tile_scheduler.hpp` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/util/command_line.h` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/util/distribution.h` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/util/host_tensor.h` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/util/packed_stride.hpp` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/util/tensor_view_io.h` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/util/reference/device/gemm_complex.h` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/util/reference/device/tensor_compare.h` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/util/reference/device/tensor_fill.h` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
