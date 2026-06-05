# observer.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/profiler/orchestration/observer.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Coordinates high-level profiler setup, collection, and trace export.
  - CN: 协调高层 profiler 的配置、采集与跟踪导出。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
 1 | #include <torch/csrc/profiler/orchestration/observer.h>
 2 | 
 3 | #include <torch/csrc/profiler/util.h>
 4 | 
 5 | #include <utility>
 6 | 
 7 | namespace torch::profiler::impl {
 8 | 
 9 | using GlobalManager = GlobalStateManager<ProfilerStateBase>;
10 | 
11 | // ----------------------------------------------------------------------------
12 | // -- Profiler Config ---------------------------------------------------------
13 | // ----------------------------------------------------------------------------
14 | ExperimentalConfig::ExperimentalConfig(
15 |     std::vector<std::string> profiler_metrics,
16 |     bool profiler_measure_per_kernel,
17 |     bool verbose,
18 |     std::vector<std::string> performance_events,
19 |     bool enable_cuda_sync_events,
20 |     bool adjust_profiler_step,
21 |     bool disable_external_correlation,
22 |     bool profile_all_threads,
23 |     bool capture_overload_names,
24 |     bool record_python_gc_info,
25 |     bool expose_kineto_event_metadata,
26 |     std::string custom_profiler_config,
27 |     bool adjust_timestamps)
28 |     : profiler_metrics{std::move(profiler_metrics)},
29 |       profiler_measure_per_kernel{profiler_measure_per_kernel},
30 |       verbose{verbose},
31 |       performance_events(std::move(performance_events)),
32 |       enable_cuda_sync_events{enable_cuda_sync_events},
```
- EN: Brings in project headers such as `<torch/csrc/profiler/orchestration/observer.h>`, `<torch/csrc/profiler/util.h>` and system or third-party headers such as `<utility>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch::profiler::impl`) so ownership matches the PyTorch subsystem layout. At the statement level, this block uses move semantics to avoid unnecessary copies.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/profiler/orchestration/observer.h>`、`<torch/csrc/profiler/util.h>`以及系统或第三方头文件，例如 `<utility>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch::profiler::impl`），使其归属与 PyTorch 子系统布局保持一致。 在语句层面，这一段使用移动语义来避免不必要的拷贝。

### Lines 33-71
```cpp
33 |       adjust_profiler_step{adjust_profiler_step},
34 |       disable_external_correlation{disable_external_correlation},
35 |       profile_all_threads{profile_all_threads},
36 |       capture_overload_names{capture_overload_names},
37 |       record_python_gc_info{record_python_gc_info},
38 |       expose_kineto_event_metadata{expose_kineto_event_metadata},
39 |       custom_profiler_config(std::move(custom_profiler_config)),
40 |       adjust_timestamps{adjust_timestamps} {}
41 | 
42 | /*explicit*/ ExperimentalConfig::operator bool() const {
43 |   return !profiler_metrics.empty();
44 | }
45 | 
46 | ProfilerConfig::ProfilerConfig(
47 |     ProfilerState state,
48 |     bool report_input_shapes,
49 |     bool profile_memory,
50 |     bool with_stack,
51 |     bool with_flops,
52 |     bool with_modules,
53 |     ExperimentalConfig experimental_config,
54 |     std::string trace_id)
55 |     : state{state},
56 |       experimental_config{std::move(experimental_config)},
57 |       report_input_shapes{report_input_shapes},
58 |       profile_memory{profile_memory},
59 |       with_stack{with_stack},
60 |       with_flops{with_flops},
61 |       with_modules{with_modules},
62 |       trace_id{std::move(trace_id)} {}
63 | 
64 | bool ProfilerConfig::disabled() const {
65 |   return state == torch::profiler::impl::ProfilerState::Disabled;
66 | }
67 | 
68 | bool ProfilerConfig::global() const {
69 |   return state == torch::profiler::impl::ProfilerState::KINETO_ONDEMAND;
70 | }
71 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 72-111
```cpp
 72 | bool ProfilerConfig::pushGlobalCallbacks() const {
 73 |   return global() || experimental_config.profile_all_threads;
 74 | }
 75 | 
 76 | namespace {
 77 | enum ProfilerIValueIdx {
 78 |   STATE = 0,
 79 |   REPORT_INPUT_SHAPES,
 80 |   PROFILE_MEMORY,
 81 |   NUM_PROFILER_CFG_IVALUE_IDX // must be last in list
 82 | };
 83 | } // namespace
 84 | 
 85 | at::IValue ProfilerConfig::toIValue() const {
 86 |   c10::impl::GenericList eventIValueList(at::AnyType::get());
 87 |   eventIValueList.reserve(NUM_PROFILER_CFG_IVALUE_IDX);
 88 |   eventIValueList.emplace_back(static_cast<int64_t>(state));
 89 |   eventIValueList.emplace_back(report_input_shapes);
 90 |   eventIValueList.emplace_back(profile_memory);
 91 |   return eventIValueList;
 92 | }
 93 | 
 94 | ProfilerConfig ProfilerConfig::fromIValue(
 95 |     const at::IValue& profilerConfigIValue) {
 96 |   TORCH_INTERNAL_ASSERT(
 97 |       profilerConfigIValue.isList(),
 98 |       "Expected IValue to contain type c10::impl::GenericList");
 99 |   auto ivalues = profilerConfigIValue.toList();
100 |   TORCH_INTERNAL_ASSERT(
101 |       ivalues.size() == NUM_PROFILER_CFG_IVALUE_IDX,
102 |       c10::str(
103 |           "Expected exactly ",
104 |           NUM_PROFILER_CFG_IVALUE_IDX,
105 |           " ivalues to resconstruct ProfilerConfig."));
106 |   return ProfilerConfig(
107 |       static_cast<ProfilerState>(ivalues.get(ProfilerIValueIdx::STATE).toInt()),
108 |       ivalues.get(ProfilerIValueIdx::REPORT_INPUT_SHAPES).toBool(),
109 |       ivalues.get(ProfilerIValueIdx::PROFILE_MEMORY).toBool());
110 | }
111 | 
```
- EN: Defines or extends data abstractions such as `ProfilerIValueIdx` that structure the state handled by this file. Implements routines such as `eventIValueList`, `ProfilerConfig` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 定义或扩展了 `ProfilerIValueIdx` 等数据抽象，用来组织本文件处理的状态。 实现了 `eventIValueList`、`ProfilerConfig` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 112-145
```cpp
112 | // ----------------------------------------------------------------------------
113 | // -- Profiler base class -----------------------------------------------------
114 | // ----------------------------------------------------------------------------
115 | /*explicit*/ ProfilerStateBase::ProfilerStateBase(ProfilerConfig config)
116 |     : c10::MemoryReportingInfoBase(), config_(std::move(config)) {}
117 | 
118 | ProfilerStateBase::~ProfilerStateBase() {
119 |   if (handle_) {
120 |     auto handle = handle_;
121 |     removeCallback();
122 |     SOFT_ASSERT(false, "Leaked callback handle: ", handle);
123 |   }
124 | }
125 | 
126 | /*static*/ ProfilerStateBase* ProfilerStateBase::get(bool global) {
127 |   auto* out = global
128 |       ? GlobalManager::get()
129 |       : static_cast<ProfilerStateBase*>(
130 |             c10::ThreadLocalDebugInfo::get(c10::DebugInfoKind::PROFILER_STATE));
131 |   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(
132 |       !out || out->config().pushGlobalCallbacks() == global);
133 |   return out;
134 | }
135 | 
136 | /*static*/ void ProfilerStateBase::push(
137 |     std::shared_ptr<ProfilerStateBase>&& state) {
138 |   TORCH_INTERNAL_ASSERT(state != nullptr);
139 |   if (state->config().pushGlobalCallbacks()) {
140 |     GlobalManager::push(std::move(state));
141 |   } else {
142 |     c10::ThreadLocalDebugInfo::_push(c10::DebugInfoKind::PROFILER_STATE, state);
143 |   }
144 | }
145 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 146-184
```cpp
146 | namespace {
147 | std::shared_ptr<ProfilerStateBase> popTLS() {
148 |   // If there is no active thread local profiler then we simply return null.
149 |   // However if there is an active profiler but it is not the top
150 |   // `DebugInfoBase`then `c10::ThreadLocalDebugInfo::_pop` will throw.
151 |   // TODO(robieta): make `noexcept` version.
152 |   return c10::ThreadLocalDebugInfo::get(c10::DebugInfoKind::PROFILER_STATE)
153 |       ? std::static_pointer_cast<ProfilerStateBase>(
154 |             c10::ThreadLocalDebugInfo::_pop(c10::DebugInfoKind::PROFILER_STATE))
155 |       : nullptr;
156 | }
157 | } // namespace
158 | 
159 | /*static*/ std::shared_ptr<ProfilerStateBase> ProfilerStateBase::pop(
160 |     bool global) {
161 |   auto out = global ? GlobalManager::pop() : popTLS();
162 |   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(!out || out->config().global() == global);
163 |   return out;
164 | }
165 | 
166 | void ProfilerStateBase::setCallbackHandle(at::CallbackHandle handle) {
167 |   if (handle_) {
168 |     at::removeCallback(handle_);
169 |     SOFT_ASSERT(
170 |         false,
171 |         "ProfilerStateBase already has a registered callback. "
172 |         "Removing to avoid leaked callback.");
173 |   }
174 | 
175 |   handle_ = handle;
176 | }
177 | 
178 | void ProfilerStateBase::removeCallback() {
179 |   if (handle_) {
180 |     at::removeCallback(handle_);
181 |     handle_ = 0;
182 |   }
183 | }
184 | 
```
- EN: Implements routines such as `popTLS` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers.
- CN: 实现了 `popTLS` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方。

### Lines 185-204
```cpp
185 | bool profilerEnabled() {
186 |   auto* state_ptr = ProfilerStateBase::get(/*global=*/false);
187 |   return state_ptr && !state_ptr->config().disabled();
188 | }
189 | 
190 | TORCH_API ActiveProfilerType profilerType() {
191 |   auto* state_ptr = ProfilerStateBase::get(/*global=*/false);
192 |   return state_ptr == nullptr ? ActiveProfilerType::NONE
193 |                               : state_ptr->profilerType();
194 | }
195 | 
196 | torch::profiler::impl::ProfilerConfig getProfilerConfig() {
197 |   auto* state_ptr = ProfilerStateBase::get(/*global=*/false);
198 |   TORCH_CHECK(
199 |       state_ptr,
200 |       "Tried to access profiler config, but profiler is not enabled!");
201 |   return state_ptr->config();
202 | }
203 | 
204 | } // namespace torch::profiler::impl
```
- EN: Implements routines such as `profilerEnabled`, `profilerType`, `getProfilerConfig` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `profilerEnabled`、`profilerType`、`getProfilerConfig` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

## Key Concepts / 关键概念

- **Important routines / 重要例程**
  - EN: `eventIValueList`, `ProfilerConfig`, `popTLS`, `profilerEnabled`, `profilerType`, `getProfilerConfig`.
  - CN: `eventIValueList`、`ProfilerConfig`、`popTLS`、`profilerEnabled`、`profilerType`、`getProfilerConfig`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::profiler::impl`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::profiler::impl` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/profiler/orchestration/observer.h>`, `<torch/csrc/profiler/util.h>`
- External includes / 外部头文件: `<utility>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
