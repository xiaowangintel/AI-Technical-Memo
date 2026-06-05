# observer.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/profiler/orchestration/observer.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Coordinates high-level profiler setup, collection, and trace export.
  - CN: 协调高层 profiler 的配置、采集与跟踪导出。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
 1 | #pragma once
 2 | 
 3 | #include <ATen/record_function.h>
 4 | #include <torch/csrc/Export.h>
 5 | 
 6 | #include <utility>
 7 | 
 8 | namespace torch::profiler::impl {
 9 | 
10 | // ----------------------------------------------------------------------------
11 | // -- Profiler Config ---------------------------------------------------------
12 | // ----------------------------------------------------------------------------
13 | enum class C10_API_ENUM ActivityType {
14 |   CPU = 0,
15 |   XPU, // XPU kernels, runtime
16 |   CUDA, // CUDA kernels, runtime
17 |   HPU, // HPU kernels, runtime
18 |   MTIA, // MTIA kernels, runtime
19 |   PrivateUse1, // PrivateUse1 kernels, runtime
20 |   NUM_KINETO_ACTIVITIES, // must be the last one
21 | };
22 | 
23 | inline std::string actToString(ActivityType t) {
24 |   const std::array<
25 |       std::string,
26 |       static_cast<size_t>(ActivityType::NUM_KINETO_ACTIVITIES)>
27 |       ActivityTypeNames = {"CPU", "XPU", "CUDA", "MTIA", "PrivateUse1"};
28 |   return ActivityTypeNames[static_cast<int>(t)];
29 | }
30 | 
```
- EN: Brings in project headers such as `<ATen/record_function.h>`, `<torch/csrc/Export.h>` and system or third-party headers such as `<utility>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::profiler::impl`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `ActivityType`, `C10_API_ENUM` that structure the state handled by this file.
- CN: 这里引入了项目头文件，例如 `<ATen/record_function.h>`、`<torch/csrc/Export.h>`以及系统或第三方头文件，例如 `<utility>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::profiler::impl`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `ActivityType`、`C10_API_ENUM` 等数据抽象，用来组织本文件处理的状态。

### Lines 31-54
```cpp
31 | enum class C10_API_ENUM ProfilerState {
32 |   Disabled = 0,
33 |   CPU, // CPU-only profiling
34 |   CUDA, // CPU + CUDA events
35 |   NVTX, // only emit NVTX markers
36 |   ITT, // only emit ITT markers
37 |   PRIVATEUSE1, // only emit PRIVATEUSE1 markers
38 |   KINETO, // use libkineto
39 |   KINETO_GPU_FALLBACK, // use CUDA events when CUPTI is not available
40 |   KINETO_PRIVATEUSE1_FALLBACK, // use PrivateUse1 events
41 |   KINETO_PRIVATEUSE1, // use Kineto with registered IActivityProfiler
42 |   KINETO_ONDEMAND, // run the profiler in on-demand mode
43 |   NUM_PROFILER_STATES, // must be the last one
44 | };
45 | 
46 | enum class C10_API_ENUM ActiveProfilerType {
47 |   NONE = 0,
48 |   LEGACY,
49 |   KINETO,
50 |   NVTX,
51 |   ITT,
52 |   PRIVATEUSE1
53 | };
54 | 
```
- EN: Defines or extends data abstractions such as `ProfilerState`, `ActiveProfilerType`, `C10_API_ENUM` that structure the state handled by this file.
- CN: 定义或扩展了 `ProfilerState`、`ActiveProfilerType`、`C10_API_ENUM` 等数据抽象，用来组织本文件处理的状态。

### Lines 55-86
```cpp
55 | struct TORCH_API ExperimentalConfig {
56 |   ExperimentalConfig(
57 |       std::vector<std::string> profiler_metrics = {},
58 |       bool profiler_measure_per_kernel = false,
59 |       bool verbose = false,
60 |       std::vector<std::string> performance_events = {},
61 |       bool enable_cuda_sync_events = false,
62 |       bool adjust_profiler_step = false,
63 |       bool disable_external_correlation = false,
64 |       bool profile_all_threads = false,
65 |       bool capture_overload_names = false,
66 |       bool record_python_gc_info = false,
67 |       bool expose_kineto_event_metadata = false,
68 |       std::string custom_profiler_config = "",
69 |       bool adjust_timestamps = false);
70 |   explicit operator bool() const;
71 | 
72 |   std::vector<std::string> profiler_metrics;
73 |   bool profiler_measure_per_kernel;
74 |   bool verbose;
75 |   /*
76 |    * List of performance events to be profiled.
77 |    * An empty list will disable performance event based profiling altogether.
78 |    */
79 |   std::vector<std::string> performance_events;
80 |   /*
81 |    * For CUDA profiling mode, enable adding CUDA synchronization events
82 |    * that expose CUDA device, stream and event synchronization activities.
83 |    * This feature is new and currently disabled by default.
84 |    */
85 |   bool enable_cuda_sync_events;
86 |   /*
```
- EN: Defines or extends data abstractions such as `ExperimentalConfig` that structure the state handled by this file. Implements routines such as `bool` that expose the key API or control flow of this region.
- CN: 定义或扩展了 `ExperimentalConfig` 等数据抽象，用来组织本文件处理的状态。 实现了 `bool` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 87-123
```cpp
 87 |    * Controls whether or not timestamp adjustment for ProfilerStep and parent
 88 |    * Python events occurs after profiling. This occurs at an O(n) cost and
 89 |    * affects only the start of profiler step events.
 90 |    */
 91 |   bool adjust_profiler_step;
 92 |   /*
 93 |    * Controls whether or not external correlation is disabled. This is used to
 94 |    * lower the amount of events received by CUPTI as correlation events are
 95 |    * paired with runtime/gpu events for each kind of correlation
 96 |    */
 97 |   bool disable_external_correlation;
 98 | 
 99 |   /* controls whether profiler records cpu events on threads
100 |    * that are not spawned from the main thread on which the
101 |    * profiler was enabled, similar to on_demand mode */
102 |   bool profile_all_threads;
103 | 
104 |   /* controls whether overload names are queried from an ATen
105 |    * function schema and stored in the profile  */
106 |   bool capture_overload_names;
107 | 
108 |   /*
109 |    * Controls whether or not python gc info is recorded. This is used to
110 |    * determine if gc collect is slowing down your profile.
111 |    */
112 |   bool record_python_gc_info;
113 | 
114 |   /* controls whether KinetoEvent metadata is exposed to FunctionEvent
115 |    * in the PyTorch Profiler as a JSON string */
116 |   bool expose_kineto_event_metadata;
117 | 
118 |   /*
119 |    * A custom_profiler_config option is introduced to allow custom backends
120 |    * to apply custom configurations as needed.
121 |    */
122 |   std::string custom_profiler_config;
123 | 
```
- EN: Continues the file's main role: Coordinates high-level profiler setup, collection, and trace export.
- CN: 继续承担本文件的主要职责：协调高层 profiler 的配置、采集与跟踪导出。

### Lines 124-160
```cpp
124 |   /*
125 |    * Controls whether or not timestamp adjustment occurs after profiling.
126 |    * The purpose of this is to adjust Vulkan event timelines to align with those
127 |    * of their parent CPU events.
128 |    * This sometimes requires increasing CPU event durations (to fully contain
129 |    * their child events) and delaying CPU event start times (to
130 |    * prevent overlaps), so this should not be used unless Vulkan events are
131 |    * being profiled and it is ok to use this modified timestamp/duration
132 |    * information instead of the original information.
133 |    */
134 |   bool adjust_timestamps;
135 | };
136 | 
137 | struct TORCH_API ProfilerConfig {
138 |   explicit ProfilerConfig(
139 |       ProfilerState state,
140 |       bool report_input_shapes = false,
141 |       bool profile_memory = false,
142 |       bool with_stack = false,
143 |       bool with_flops = false,
144 |       bool with_modules = false,
145 |       ExperimentalConfig experimental_config = ExperimentalConfig(),
146 |       std::string trace_id = "");
147 | 
148 |   bool disabled() const;
149 |   bool global() const;
150 |   bool pushGlobalCallbacks() const;
151 | 
152 |   ProfilerState state;
153 |   ExperimentalConfig experimental_config;
154 |   bool report_input_shapes;
155 |   bool profile_memory;
156 |   bool with_stack;
157 |   bool with_flops;
158 |   bool with_modules;
159 |   std::string trace_id;
160 | 
```
- EN: Defines or extends data abstractions such as `ProfilerConfig` that structure the state handled by this file. Implements routines such as `ProfilerConfig`, `disabled`, `global`, `pushGlobalCallbacks` that expose the key API or control flow of this region.
- CN: 定义或扩展了 `ProfilerConfig` 等数据抽象，用来组织本文件处理的状态。 实现了 `ProfilerConfig`、`disabled`、`global`、`pushGlobalCallbacks` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 161-197
```cpp
161 |   // For serialization
162 |   at::IValue toIValue() const;
163 |   static ProfilerConfig fromIValue(const at::IValue& profilerConfigIValue);
164 | };
165 | 
166 | // ----------------------------------------------------------------------------
167 | // -- Profiler base class -----------------------------------------------------
168 | // ----------------------------------------------------------------------------
169 | struct TORCH_API ProfilerStateBase : public c10::MemoryReportingInfoBase {
170 |   explicit ProfilerStateBase(ProfilerConfig config);
171 |   ProfilerStateBase(const ProfilerStateBase&) = delete;
172 |   ProfilerStateBase(ProfilerStateBase&&) = delete;
173 |   ProfilerStateBase& operator=(const ProfilerStateBase&) = delete;
174 |   ProfilerStateBase& operator=(ProfilerStateBase&&) = delete;
175 |   ~ProfilerStateBase() override;
176 | 
177 |   static ProfilerStateBase* get(bool global);
178 |   static ProfilerStateBase* get() {
179 |     auto* out = get(/*global=*/true);
180 |     return out ? out : get(/*global=*/false);
181 |   }
182 | 
183 |   static void push(std::shared_ptr<ProfilerStateBase>&& state);
184 | 
185 |   static std::shared_ptr<ProfilerStateBase> pop(bool global);
186 |   static std::shared_ptr<ProfilerStateBase> pop() {
187 |     auto out = pop(/*global=*/true);
188 |     return out ? std::move(out) : pop(/*global=*/false);
189 |   }
190 | 
191 |   const ProfilerConfig& config() const {
192 |     return config_;
193 |   }
194 | 
195 |   void setCallbackHandle(at::CallbackHandle handle);
196 |   void removeCallback();
197 | 
```
- EN: Defines or extends data abstractions such as `ProfilerStateBase` that structure the state handled by this file. Implements routines such as `toIValue`, `fromIValue`, `ProfilerStateBase`, `get`, `push` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; stores long-lived member state for later calls.
- CN: 定义或扩展了 `ProfilerStateBase` 等数据抽象，用来组织本文件处理的状态。 实现了 `toIValue`、`fromIValue`、`ProfilerStateBase`、`get`、`push` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；保存供后续调用使用的长期成员状态。

### Lines 198-218
```cpp
198 |   bool memoryProfilingEnabled() const override {
199 |     return config_.profile_memory;
200 |   }
201 | 
202 |   virtual ActiveProfilerType profilerType() = 0;
203 | 
204 |  protected:
205 |   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
206 |   std::mutex state_mutex_;
207 |   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
208 |   ProfilerConfig config_ = ProfilerConfig(ProfilerState::Disabled);
209 |   // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
210 |   at::CallbackHandle handle_ = 0;
211 | };
212 | 
213 | // Note: The following are only for the active *thread local* profiler.
214 | TORCH_API bool profilerEnabled();
215 | TORCH_API ActiveProfilerType profilerType();
216 | TORCH_API ProfilerConfig getProfilerConfig();
217 | 
218 | } // namespace torch::profiler::impl
```
- EN: Implements routines such as `memoryProfilingEnabled`, `profilerType`, `profilerEnabled`, `getProfilerConfig` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 实现了 `memoryProfilingEnabled`、`profilerType`、`profilerEnabled`、`getProfilerConfig` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `ActivityType`, `ProfilerState`, `ActiveProfilerType`, `ExperimentalConfig`, `ProfilerConfig`, `ProfilerStateBase`.
  - CN: `ActivityType`、`ProfilerState`、`ActiveProfilerType`、`ExperimentalConfig`、`ProfilerConfig`、`ProfilerStateBase`。
- **Important routines / 重要例程**
  - EN: `actToString`, `bool`, `ProfilerConfig`, `disabled`, `global`, `pushGlobalCallbacks`, `toIValue`, `fromIValue`.
  - CN: `actToString`、`bool`、`ProfilerConfig`、`disabled`、`global`、`pushGlobalCallbacks`、`toIValue`、`fromIValue`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::profiler::impl`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::profiler::impl` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<ATen/record_function.h>`, `<torch/csrc/Export.h>`
- External includes / 外部头文件: `<utility>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
