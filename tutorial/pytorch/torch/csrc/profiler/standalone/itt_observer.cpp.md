# itt_observer.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/profiler/standalone/itt_observer.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements standalone profiling and execution-trace support utilities.
  - CN: 实现独立的 profiling 与执行跟踪支持工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21
```cpp
 1 | #include <torch/csrc/profiler/standalone/itt_observer.h>
 2 | 
 3 | #include <torch/csrc/profiler/stubs/base.h>
 4 | 
 5 | namespace torch::profiler::impl {
 6 | 
 7 | struct ITTThreadLocalState : ProfilerStateBase {
 8 |   explicit ITTThreadLocalState(const ProfilerConfig& config)
 9 |       : ProfilerStateBase(config) {
10 |     // Only `report_input_shapes` makes sense in this context.
11 |     TORCH_CHECK(!config.profile_memory);
12 |     TORCH_CHECK(!config.with_stack);
13 |     TORCH_CHECK(!config.with_flops);
14 |     TORCH_CHECK(!config.with_modules);
15 |   }
16 |   ~ITTThreadLocalState() override = default;
17 | 
18 |   ActiveProfilerType profilerType() override {
19 |     return ActiveProfilerType::ITT;
20 |   }
21 | 
```
- EN: Brings in project headers such as `<torch/csrc/profiler/standalone/itt_observer.h>`, `<torch/csrc/profiler/stubs/base.h>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch::profiler::impl`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `ITTThreadLocalState` that structure the state handled by this file. Implements routines such as `ITTThreadLocalState`, `profilerType` that expose the key API or control flow of this region.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/profiler/standalone/itt_observer.h>`、`<torch/csrc/profiler/stubs/base.h>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch::profiler::impl`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `ITTThreadLocalState` 等数据抽象，用来组织本文件处理的状态。 实现了 `ITTThreadLocalState`、`profilerType` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 22-45
```cpp
22 |   void reportMemoryUsage(
23 |       void* /*ptr*/,
24 |       int64_t /*alloc_size*/,
25 |       size_t /*total_allocated*/,
26 |       size_t /*total_reserved*/,
27 |       c10::Device /*device*/) override {}
28 | 
29 |   static ITTThreadLocalState* getTLS() {
30 |     auto tls = ProfilerStateBase::get(/*global=*/false);
31 |     TORCH_INTERNAL_ASSERT_DEBUG_ONLY(
32 |         tls == nullptr || tls->profilerType() == ActiveProfilerType::ITT);
33 |     return static_cast<ITTThreadLocalState*>(tls);
34 |   }
35 | };
36 | 
37 | template <bool report_input_shapes>
38 | static std::unique_ptr<at::ObserverContext> enterITT(
39 |     const at::RecordFunction& fn) {
40 |   if (ITTThreadLocalState::getTLS() != nullptr) {
41 |     torch::profiler::impl::ittStubs()->rangePush(fn.name());
42 |   }
43 |   return nullptr;
44 | }
45 | 
```
- EN: Implements routines such as `reportMemoryUsage`, `getTLS`, `enterITT` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `reportMemoryUsage`、`getTLS`、`enterITT` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 46-59
```cpp
46 | void pushITTCallbacks(
47 |     const ProfilerConfig& config,
48 |     const std::unordered_set<at::RecordScope>& scopes) {
49 |   TORCH_CHECK(
50 |       torch::profiler::impl::ittStubs()->enabled(),
51 |       "Can't use ITT profiler - PyTorch was compiled without ITT");
52 | 
53 |   c10::ThreadLocalDebugInfo::_push(
54 |       c10::DebugInfoKind::PROFILER_STATE,
55 |       std::make_shared<ITTThreadLocalState>(config));
56 | 
57 |   auto state_ptr = ITTThreadLocalState::getTLS();
58 |   TORCH_INTERNAL_ASSERT(state_ptr, "Expected profiler state set");
59 | 
```
- EN: Implements routines such as `pushITTCallbacks` that expose the key API or control flow of this region. At the statement level, this block validates assumptions and reports descriptive failures.
- CN: 实现了 `pushITTCallbacks` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段校验前提条件并报告明确错误。

### Lines 60-73
```cpp
60 |   auto handle = at::addThreadLocalCallback(
61 |       at::RecordFunctionCallback(
62 |           state_ptr->config().report_input_shapes
63 |               ? &enterITT</*report_input_shapes=*/true>
64 |               : &enterITT</*report_input_shapes=*/false>,
65 |           [](const at::RecordFunction&, at::ObserverContext*) {
66 |             torch::profiler::impl::ittStubs()->rangePop();
67 |           })
68 |           .needsInputs(config.report_input_shapes)
69 |           .scopes(scopes));
70 |   state_ptr->setCallbackHandle(handle);
71 | }
72 | 
73 | } // namespace torch::profiler::impl
```
- EN: Continues the file's main role: Implements standalone profiling and execution-trace support utilities.
- CN: 继续承担本文件的主要职责：实现独立的 profiling 与执行跟踪支持工具。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `ITTThreadLocalState`.
  - CN: `ITTThreadLocalState`。
- **Important routines / 重要例程**
  - EN: `ITTThreadLocalState`, `profilerType`, `reportMemoryUsage`, `getTLS`, `enterITT`, `pushITTCallbacks`.
  - CN: `ITTThreadLocalState`、`profilerType`、`reportMemoryUsage`、`getTLS`、`enterITT`、`pushITTCallbacks`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::profiler::impl`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::profiler::impl` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/profiler/standalone/itt_observer.h>`, `<torch/csrc/profiler/stubs/base.h>`
- External includes / 外部头文件: None / 无
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
