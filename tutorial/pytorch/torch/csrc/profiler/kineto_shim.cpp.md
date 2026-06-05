# kineto_shim.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/profiler/kineto_shim.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Integrates PyTorch profiling with the Kineto tracing backend.
  - CN: 将 PyTorch 性能分析与 Kineto 跟踪后端集成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50
```cpp
 1 | #include <torch/csrc/profiler/collection.h>
 2 | #include <torch/csrc/profiler/kineto_shim.h>
 3 | 
 4 | #include <type_traits>
 5 | 
 6 | #ifdef USE_KINETO
 7 | #include <libkineto.h>
 8 | #endif
 9 | 
10 | #include <c10/util/Exception.h>
11 | #include <c10/util/env.h>
12 | 
13 | namespace torch {
14 | 
15 | namespace profiler::impl::kineto {
16 | 
17 | // Here lies pain and `#ifdef USE_KINETO`
18 | 
19 | #ifdef USE_KINETO
20 | namespace {
21 | 
22 | using ActivityTypeMap =
23 |     std::unordered_map<libkineto::ActivityType, std::string>;
24 | 
25 | // clang-format off
26 | const ActivityTypeMap kCpuTypes{
27 |     {libkineto::ActivityType::CPU_OP,                "CPU_OP"},
28 |     {libkineto::ActivityType::CPU_INSTANT_EVENT,     "CPU_INSTANT_EVENT"},
29 |     {libkineto::ActivityType::USER_ANNOTATION,       "USER_ANNOTATION"},
30 |     {libkineto::ActivityType::EXTERNAL_CORRELATION,  "EXTERNAL_CORRELATION"},
31 |     {libkineto::ActivityType::XPU_RUNTIME,           "XPU_RUNTIME"},
32 |     {libkineto::ActivityType::XPU_DRIVER,            "XPU_DRIVER"},
33 |     {libkineto::ActivityType::CUDA_RUNTIME,          "CUDA_RUNTIME"},
34 |     {libkineto::ActivityType::CUDA_DRIVER,           "CUDA_DRIVER"},
35 |     {libkineto::ActivityType::PYTHON_FUNCTION,       "PYTHON_FUNCTION"},
36 |     {libkineto::ActivityType::PRIVATEUSE1_RUNTIME,   "PRIVATEUSE1_RUNTIME"},
37 |     {libkineto::ActivityType::PRIVATEUSE1_DRIVER,    "PRIVATEUSE1_DRIVER"},
38 | };
39 | 
40 | const ActivityTypeMap kCudaTypes{
41 |     {libkineto::ActivityType::GPU_MEMCPY,            "GPU_MEMCPY"},
42 |     {libkineto::ActivityType::GPU_MEMSET,            "GPU_MEMSET"},
43 |     {libkineto::ActivityType::GPU_USER_ANNOTATION,   "GPU_USER_ANNOTATION"},
44 |     {libkineto::ActivityType::CONCURRENT_KERNEL,     "CONCURRENT_KERNEL"},
45 |     // CUDA_RUNTIME and CUDA_DRIVER appear in both kCpuTypes and kCudaTypes.
46 |     {libkineto::ActivityType::CUDA_RUNTIME,          "CUDA_RUNTIME"},
47 |     {libkineto::ActivityType::CUDA_DRIVER,           "CUDA_DRIVER"},
48 |     {libkineto::ActivityType::OVERHEAD,              "OVERHEAD"},
49 | };
50 | 
```
- EN: Brings in project headers such as `<torch/csrc/profiler/collection.h>`, `<torch/csrc/profiler/kineto_shim.h>`, `<c10/util/Exception.h>`, `<c10/util/env.h>` and system or third-party headers such as `<type_traits>`, `<libkineto.h>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch`, `profiler::impl::kineto`) so ownership matches the PyTorch subsystem layout. Reads environment switches (`CONCURRENT_KERNEL`, `CPU_INSTANT_EVENT`, `CPU_OP`, `CUDA_DRIVER`, `CUDA_RUNTIME`, `EXTERNAL_CORRELATION`) to tune runtime behavior.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/profiler/collection.h>`、`<torch/csrc/profiler/kineto_shim.h>`、`<c10/util/Exception.h>`、`<c10/util/env.h>`以及系统或第三方头文件，例如 `<type_traits>`、`<libkineto.h>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch`、`profiler::impl::kineto`），使其归属与 PyTorch 子系统布局保持一致。 读取环境变量开关（`CONCURRENT_KERNEL`、`CPU_INSTANT_EVENT`、`CPU_OP`、`CUDA_DRIVER`、`CUDA_RUNTIME`、`EXTERNAL_CORRELATION`）来调整运行时行为。

### Lines 51-105
```cpp
 51 | const ActivityTypeMap kXpuTypes{
 52 |     {libkineto::ActivityType::GPU_MEMCPY,            "GPU_MEMCPY"},
 53 |     {libkineto::ActivityType::GPU_MEMSET,            "GPU_MEMSET"},
 54 |     {libkineto::ActivityType::CONCURRENT_KERNEL,     "CONCURRENT_KERNEL"},
 55 |     // XPU_RUNTIME and XPU_DRIVER appear in both kCpuTypes and kXpuTypes.
 56 |     {libkineto::ActivityType::XPU_RUNTIME,           "XPU_RUNTIME"},
 57 |     {libkineto::ActivityType::XPU_DRIVER,            "XPU_DRIVER"},
 58 | };
 59 | 
 60 | const ActivityTypeMap kMtiaTypes{
 61 |     {libkineto::ActivityType::MTIA_CCP_EVENTS,       "MTIA_CCP_EVENTS"},
 62 |     {libkineto::ActivityType::MTIA_RUNTIME,          "MTIA_RUNTIME"},
 63 |     {libkineto::ActivityType::MTIA_INSIGHT,          "MTIA_INSIGHT"},
 64 |     {libkineto::ActivityType::MTIA_COUNTERS,         "MTIA_COUNTERS"},
 65 | };
 66 | 
 67 | const ActivityTypeMap kHpuTypes{
 68 |     {libkineto::ActivityType::HPU_OP,                "HPU_OP"},
 69 | };
 70 | 
 71 | const ActivityTypeMap kPrivateUse1Types{
 72 |     {libkineto::ActivityType::GPU_MEMCPY,            "GPU_MEMCPY"},
 73 |     {libkineto::ActivityType::GPU_MEMSET,            "GPU_MEMSET"},
 74 |     {libkineto::ActivityType::GPU_USER_ANNOTATION,   "GPU_USER_ANNOTATION"},
 75 |     {libkineto::ActivityType::CONCURRENT_KERNEL,     "CONCURRENT_KERNEL"},
 76 |     // PRIVATEUSE1_RUNTIME appears in both kCpuTypes and kPrivateUse1Types.
 77 |     {libkineto::ActivityType::PRIVATEUSE1_RUNTIME,   "PRIVATEUSE1_RUNTIME"},
 78 |     {libkineto::ActivityType::PRIVATEUSE1_DRIVER,    "PRIVATEUSE1_DRIVER"},
 79 | };
 80 | // clang-format on
 81 | 
 82 | // Given a named activity type map and a set of requested name strings,
 83 | // return the matching subset of activity types.
 84 | std::unordered_set<libkineto::ActivityType> filterActivities(
 85 |     const ActivityTypeMap& defaults,
 86 |     const std::unordered_set<std::string>& requested) {
 87 |   std::unordered_set<libkineto::ActivityType> result;
 88 |   for (const auto& name : requested) {
 89 |     bool found = false;
 90 |     for (const auto& [type, type_name] : defaults) {
 91 |       if (type_name == name) {
 92 |         result.insert(type);
 93 |         found = true;
 94 |         break;
 95 |       }
 96 |     }
 97 |     TORCH_CHECK(
 98 |         found, "Unknown or non-member activity type name: '", name, "'");
 99 |   }
100 |   return result;
101 | }
102 | 
103 | } // namespace
104 | #endif // USE_KINETO
105 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `filterActivities` that expose the key API or control flow of this region. Reads environment switches (`CONCURRENT_KERNEL`, `GPU_MEMCPY`, `GPU_MEMSET`, `GPU_USER_ANNOTATION`, `HPU_OP`, `MTIA_CCP_EVENTS`) to tune runtime behavior. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; builds container state that later execution depends on.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `filterActivities` 等例程，它们构成了这一段的关键 API 或控制流程。 读取环境变量开关（`CONCURRENT_KERNEL`、`GPU_MEMCPY`、`GPU_MEMSET`、`GPU_USER_ANNOTATION`、`HPU_OP`、`MTIA_CCP_EVENTS`）来调整运行时行为。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；构建后续执行依赖的容器状态。

### Lines 106-153
```cpp
106 | static_assert(
107 |     std::is_trivial_v<DeviceAndResource>,
108 |     "Kineto specific details should be in `kineto_ids`.");
109 | 
110 | const DeviceAndResource kineto_ids() {
111 | #ifdef USE_KINETO
112 |   return {
113 |       /*device=*/libkineto::processId(),
114 |       /*resource=*/libkineto::systemThreadId()};
115 | #else
116 |   return {};
117 | #endif // USE_KINETO
118 | }
119 | 
120 | void addMetadata(
121 |     activity_t* activity,
122 |     const std::string& key,
123 |     const std::string& value) {
124 | #ifdef USE_KINETO
125 |   activity->addMetadata(key, value);
126 | #endif // USE_KINETO
127 | }
128 | 
129 | TraceWrapper::TraceWrapper(const int64_t start_time, const std::string& name)
130 | #ifdef USE_KINETO
131 |     : cpu_trace_(std::make_unique<libkineto::CpuTraceBuffer>()) {
132 |   cpu_trace_->span.startTime = start_time;
133 |   cpu_trace_->gpuOpCount = -1;
134 |   cpu_trace_->span.name = name;
135 | }
136 | #else
137 | {
138 | }
139 | #endif // USE_KINETO
140 | 
141 | activity_t* TraceWrapper::addCPUActivity(
142 |     const std::string& name,
143 |     const libkineto::ActivityType type,
144 |     const DeviceAndResource device_and_resource,
145 |     const uint64_t correlation_id,
146 |     const int64_t start_time,
147 |     const int64_t end_time) {
148 | #ifdef USE_KINETO
149 |   TORCH_CHECK((bool)(*this), "Cannot add event to non-existent trace.");
150 |   cpu_trace_->emplace_activity(cpu_trace_->span, type, name);
151 |   auto& act = libkineto::CpuTraceBuffer::toRef(cpu_trace_->activities.back());
152 |   act.device = device_and_resource.device;
153 |   act.resource = device_and_resource.resource;
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `kineto_ids`, `addMetadata` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `kineto_ids`、`addMetadata` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 154-204
```cpp
154 |   act.id = static_cast<int32_t>(correlation_id);
155 |   act.startTime = start_time;
156 |   if (type != libkineto::ActivityType::CPU_INSTANT_EVENT) {
157 |     act.endTime = end_time;
158 |   }
159 |   return cpu_trace_->activities.back().get();
160 | #else
161 |   return nullptr;
162 | #endif // USE_KINETO
163 | }
164 | 
165 | void TraceWrapper::transferCpuTrace(int64_t end_time) {
166 | #ifdef USE_KINETO
167 |   cpu_trace_->span.endTime = end_time;
168 |   libkineto::api().activityProfiler().transferCpuTrace(std::move(cpu_trace_));
169 | #endif // USE_KINETO
170 | }
171 | 
172 | TraceWrapper::operator bool() const {
173 | #ifdef USE_KINETO
174 |   return cpu_trace_ != nullptr;
175 | #else
176 |   return false;
177 | #endif // USE_KINETO
178 | }
179 | 
180 | ActivityTraceWrapper::ActivityTraceWrapper(
181 |     std::unique_ptr<interface_trace_t>&& trace)
182 |     : trace_(std::move(trace)) {}
183 | 
184 | ActivityTraceWrapper::operator bool() const {
185 | #ifdef USE_KINETO
186 |   return trace_ != nullptr;
187 | #else
188 |   return false;
189 | #endif // USE_KINETO
190 | }
191 | 
192 | void ActivityTraceWrapper::save(const std::string& path) {
193 | #ifdef USE_KINETO
194 |   TORCH_CHECK(!saved_, "Trace is already saved.");
195 |   TORCH_CHECK(trace_ != nullptr, "Missing trace.")
196 |   trace_->save(path);
197 |   saved_ = true;
198 | #else
199 |   TORCH_CHECK(
200 |       false,
201 |       "Saving a trace requires using torch.profiler with Kineto support (USE_KINETO=1)");
202 | #endif // USE_KINETO
203 | }
204 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `bool` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; uses move semantics to avoid unnecessary copies.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `bool` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；使用移动语义来避免不必要的拷贝。

### Lines 205-255
```cpp
205 | namespace {
206 | // Handles processing of Experimental Config options for Kineto
207 | class ExperimentalConfigWrapper {
208 |  public:
209 |   explicit ExperimentalConfigWrapper(
210 |       const torch::profiler::impl::ExperimentalConfig& config)
211 |       : config_(config) {}
212 | 
213 |   bool assertValid() {
214 |     return !config_.profiler_metrics.empty();
215 |   }
216 | 
217 |   void prepareTraceWithExperimentalOptions(
218 |       std::set<libkineto::ActivityType>&& enabled_activities) {
219 |     std::set<libkineto::ActivityType> k_activities =
220 |         std::move(enabled_activities);
221 | #ifdef USE_KINETO
222 |     k_activities.insert(libkineto::ActivityType::CUDA_PROFILER_RANGE);
223 | 
224 |     // Add CPU activities if we are measuring per kernel ranges
225 |     if (config_.profiler_measure_per_kernel) {
226 |       for (const auto& [type, name] : kCpuTypes) {
227 |         k_activities.insert(type);
228 |       }
229 |     }
230 | 
231 |     const size_t num_metrics = config_.profiler_metrics.size();
232 |     std::stringstream configss;
233 | 
234 |     LOG(INFO) << "CUPTI profiler metrics size = " << num_metrics;
235 | 
236 |     configss << "ACTIVITIES_WARMUP_PERIOD_SECS=0\n"
237 |              << "CUPTI_PROFILER_METRICS=";
238 | 
239 |     for (size_t i = 0; i < num_metrics; i++) {
240 |       configss << config_.profiler_metrics[i];
241 |       if (num_metrics > 1 && i < (num_metrics - 1)) {
242 |         configss << ',';
243 |       }
244 |     }
245 |     configss << "\nCUPTI_PROFILER_ENABLE_PER_KERNEL="
246 |              << (config_.profiler_measure_per_kernel ? "true" : "false")
247 |              << '\n';
248 |     configss << "CUSTOM_CONFIG=" << config_.custom_profiler_config << '\n';
249 |     LOG(INFO) << "Generated config = " << configss.str();
250 | 
251 |     libkineto::api().activityProfiler().prepareTrace(
252 |         k_activities, configss.str());
253 | #endif // USE_KINETO
254 |   }
255 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Defines or extends data abstractions such as `ExperimentalConfigWrapper` that structure the state handled by this file. Implements routines such as `ExperimentalConfigWrapper`, `assertValid`, `prepareTraceWithExperimentalOptions` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; builds container state that later execution depends on.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 定义或扩展了 `ExperimentalConfigWrapper` 等数据抽象，用来组织本文件处理的状态。 实现了 `ExperimentalConfigWrapper`、`assertValid`、`prepareTraceWithExperimentalOptions` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；构建后续执行依赖的容器状态。

### Lines 256-311
```cpp
256 |  private:
257 |   // NOLINTNEXTLINE(cppcoreguidelines-avoid-const-or-ref-data-members)
258 |   const torch::profiler::impl::ExperimentalConfig& config_;
259 | };
260 | } // namespace
261 | 
262 | bool collectivesProfilerExists() {
263 | #if defined(KINETO_HAS_HCCL_PROFILER)
264 |   return true;
265 | #endif
266 |   const auto val =
267 |       c10::utils::get_env("TORCH_PROFILER_ENABLE_COLLECTIVE_PROFILING");
268 |   return val == "1";
269 | }
270 | 
271 | #ifdef USE_KINETO
272 | static const std::string setTraceID(const std::string& trace_id) {
273 |   if (trace_id.empty()) {
274 |     return "";
275 |   }
276 |   std::stringstream configss;
277 |   configss << "REQUEST_TRACE_ID=" << trace_id << '\n';
278 |   configss << "REQUEST_GROUP_TRACE_ID=" << trace_id << '\n';
279 |   return configss.str();
280 | }
281 | 
282 | static const std::string appendCustomConfig(
283 |     const std::string& config,
284 |     const std::string& custom_profiler_config) {
285 |   if (custom_profiler_config.empty()) {
286 |     return config;
287 |   }
288 |   std::stringstream configss;
289 |   configss << config;
290 |   configss << "CUSTOM_CONFIG=" << custom_profiler_config << '\n';
291 |   return configss.str();
292 | }
293 | #endif
294 | 
295 | void prepareTrace(
296 |     const bool cpuOnly,
297 |     const ActivitySet& activities,
298 |     const torch::profiler::impl::ExperimentalConfig& config,
299 |     const std::string& trace_id,
300 |     const ActivityFilter& activity_filter) {
301 | #ifdef USE_KINETO
302 |   libkineto::api().resetKinetoTLS();
303 |   if (!libkineto::api().isProfilerRegistered()) {
304 |     libkineto_init(/*cpuOnly=*/cpuOnly, /*logOnError=*/true);
305 |     libkineto::api().suppressLogMessages();
306 |   }
307 | 
308 |   if (!libkineto::api().isProfilerInitialized()) {
309 |     libkineto::api().initProfilerIfRegistered();
310 |   }
311 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `collectivesProfilerExists`, `setTraceID`, `appendCustomConfig`, `prepareTrace` that expose the key API or control flow of this region. Reads environment switches (`TORCH_PROFILER_ENABLE_COLLECTIVE_PROFILING`) to tune runtime behavior. At the statement level, this block returns computed state or forwards values to callers.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `collectivesProfilerExists`、`setTraceID`、`appendCustomConfig`、`prepareTrace` 等例程，它们构成了这一段的关键 API 或控制流程。 读取环境变量开关（`TORCH_PROFILER_ENABLE_COLLECTIVE_PROFILING`）来调整运行时行为。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 312-359
```cpp
312 |   std::set<libkineto::ActivityType> k_activities;
313 | 
314 |   // Helper: insert activity types for a group, applying the filter if present.
315 |   auto insertActivities = [&](torch::autograd::profiler::ActivityType group,
316 |                               const ActivityTypeMap& defaults) {
317 |     auto filter_it = activity_filter.find(group);
318 |     if (filter_it != activity_filter.end()) {
319 |       auto filtered = filterActivities(defaults, filter_it->second);
320 |       k_activities.insert(filtered.begin(), filtered.end());
321 |     } else {
322 |       for (const auto& [type, name] : defaults) {
323 |         k_activities.insert(type);
324 |       }
325 |     }
326 |   };
327 | 
328 |   bool has_cpu_activity =
329 |       activities.count(torch::autograd::profiler::ActivityType::CPU);
330 | 
331 |   if (has_cpu_activity) {
332 |     insertActivities(torch::autograd::profiler::ActivityType::CPU, kCpuTypes);
333 |   }
334 |   if (activities.count(torch::autograd::profiler::ActivityType::XPU)) {
335 |     insertActivities(torch::autograd::profiler::ActivityType::XPU, kXpuTypes);
336 |   }
337 |   if (activities.count(torch::autograd::profiler::ActivityType::MTIA)) {
338 |     if (config.custom_profiler_config.empty()) {
339 |       insertActivities(
340 |           torch::autograd::profiler::ActivityType::MTIA, kMtiaTypes);
341 |     } else {
342 |       if (config.custom_profiler_config.find("disable_runtime_events") ==
343 |           std::string::npos) {
344 |         k_activities.insert(libkineto::ActivityType::MTIA_RUNTIME);
345 |       } else {
346 |         LOG(INFO) << "Disabling MTIA runtime events";
347 |       }
348 |       if (config.custom_profiler_config.find("disable_ccp_events") ==
349 |           std::string::npos) {
350 |         k_activities.insert(libkineto::ActivityType::MTIA_CCP_EVENTS);
351 |       } else {
352 |         LOG(INFO) << "Disabling MTIA CCP events";
353 |       }
354 |       if (config.custom_profiler_config.find("disable_insight_events") ==
355 |           std::string::npos) {
356 |         k_activities.insert(libkineto::ActivityType::MTIA_INSIGHT);
357 |       } else {
358 |         LOG(INFO) << "Disabling MTIA insight events";
359 |       }
```
- EN: At the statement level, this block builds container state that later execution depends on.
- CN: 在语句层面，这一段构建后续执行依赖的容器状态。

### Lines 360-412
```cpp
360 |       if (config.custom_profiler_config.find("disable_counter_events") ==
361 |           std::string::npos) {
362 |         k_activities.insert(libkineto::ActivityType::MTIA_COUNTERS);
363 |       } else {
364 |         LOG(INFO) << "Disabling MTIA counter events";
365 |       }
366 |     }
367 |   }
368 |   if (activities.count(torch::autograd::profiler::ActivityType::HPU)) {
369 |     insertActivities(torch::autograd::profiler::ActivityType::HPU, kHpuTypes);
370 |   }
371 |   if (activities.count(torch::autograd::profiler::ActivityType::CUDA)) {
372 |     insertActivities(torch::autograd::profiler::ActivityType::CUDA, kCudaTypes);
373 |     if (config.enable_cuda_sync_events || get_cuda_sync_enabled()) {
374 |       LOG(INFO) << "Enabling CUDA Sync Events";
375 |       k_activities.insert(libkineto::ActivityType::CUDA_SYNC);
376 |     }
377 |   }
378 |   if (collectivesProfilerExists()) {
379 |     k_activities.insert(libkineto::ActivityType::COLLECTIVE_COMM);
380 |   }
381 |   if (activities.count(torch::autograd::profiler::ActivityType::PrivateUse1)) {
382 |     insertActivities(
383 |         torch::autograd::profiler::ActivityType::PrivateUse1,
384 |         kPrivateUse1Types);
385 |   }
386 | 
387 |   ExperimentalConfigWrapper configWrap(config);
388 | 
389 |   // Experimental Configuration options are present
390 |   if (config && configWrap.assertValid()) {
391 |     configWrap.prepareTraceWithExperimentalOptions(std::move(k_activities));
392 |     return;
393 |   }
394 | 
395 |   const std::string traceIdStr = setTraceID(trace_id);
396 |   const std::string configStr =
397 |       appendCustomConfig(traceIdStr, config.custom_profiler_config);
398 | 
399 |   libkineto::api().activityProfiler().prepareTrace(k_activities, configStr);
400 | #endif // USE_KINETO
401 | }
402 | 
403 | void toggleCollectionDynamic(const bool enable) {
404 | #ifdef USE_KINETO
405 |   // TODO: We may want to consider adding another input arg for this function
406 |   // if we want to support turning off certain devices and keeping others on.
407 |   // For now, we can keep it simple at have it turn off all tracing of "CUDA"
408 |   // devices
409 |   libkineto::api().activityProfiler().toggleCollectionDynamic(enable);
410 | #endif // USE_KINETO
411 | }
412 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `configWrap`, `toggleCollectionDynamic` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; builds container state that later execution depends on.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `configWrap`、`toggleCollectionDynamic` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；构建后续执行依赖的容器状态。

### Lines 413-458
```cpp
413 | void startTrace() {
414 | #ifdef USE_KINETO
415 |   libkineto::api().activityProfiler().startTrace();
416 | #endif // USE_KINETO
417 | }
418 | 
419 | ActivityTraceWrapper stopTrace() {
420 |   return ActivityTraceWrapper{
421 | #ifdef USE_KINETO
422 |       libkineto::api().activityProfiler().stopTrace()
423 | #else
424 |       std::make_unique<interface_trace_t>()
425 | #endif // USE_KINETO
426 |   };
427 | }
428 | 
429 | void pushCorrelationId(uint64_t correlation_id) {
430 | #ifdef USE_KINETO
431 |   libkineto::api().activityProfiler().pushCorrelationId(correlation_id);
432 | #endif // USE_KINETO
433 | }
434 | 
435 | void pushUserCorrelationId(uint64_t correlation_id) {
436 | #ifdef USE_KINETO
437 |   libkineto::api().activityProfiler().pushUserCorrelationId(correlation_id);
438 | #endif // USE_KINETO
439 | }
440 | 
441 | void popCorrelationId() {
442 | #ifdef USE_KINETO
443 |   libkineto::api().activityProfiler().popCorrelationId();
444 | #endif // USE_KINETO
445 | }
446 | 
447 | void popUserCorrelationId() {
448 | #ifdef USE_KINETO
449 |   libkineto::api().activityProfiler().popUserCorrelationId();
450 | #endif // USE_KINETO
451 | }
452 | 
453 | void recordThreadInfo() {
454 | #ifdef USE_KINETO
455 |   libkineto::api().activityProfiler().recordThreadInfo();
456 | #endif // USE_KINETO
457 | }
458 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `startTrace`, `stopTrace`, `pushCorrelationId`, `pushUserCorrelationId`, `popCorrelationId` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `startTrace`、`stopTrace`、`pushCorrelationId`、`pushUserCorrelationId`、`popCorrelationId` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 459-506
```cpp
459 | void logInvariantViolation(
460 |     const std::string& assertion,
461 |     const std::string& error,
462 |     const std::string& profile_id,
463 |     const std::string& group_profile_id) {
464 | #ifdef USE_KINETO
465 |   if (libkineto::api().isProfilerInitialized()) {
466 |     libkineto::api().activityProfiler().logInvariantViolation(
467 |         profile_id, assertion, error, group_profile_id);
468 |   }
469 | #endif // USE_KINETO
470 | }
471 | 
472 | } // namespace profiler::impl::kineto
473 | 
474 | namespace autograd::profiler {
475 | c10::DeviceType deviceTypeFromActivity(libkineto::ActivityType activity_type) {
476 |   // PrivateUse1 kineto backend reuse some ActivityTypes,
477 |   // If PrivateUse1 backend is enabled, this should return
478 |   // c10::DeviceType::PrivateUse1.
479 |   auto device_type_privateuse1_or = [](c10::DeviceType device_type) {
480 |     return c10::is_privateuse1_backend_registered()
481 |         ? c10::DeviceType::PrivateUse1
482 |         : device_type;
483 |   };
484 | 
485 |   switch (activity_type) {
486 |     case libkineto::ActivityType::GPU_MEMCPY:
487 |     case libkineto::ActivityType::GPU_MEMSET:
488 |     case libkineto::ActivityType::CONCURRENT_KERNEL:
489 | #if defined(USE_XPU)
490 |       return device_type_privateuse1_or(c10::DeviceType::XPU);
491 | #endif
492 |       [[fallthrough]];
493 |     case libkineto::ActivityType::CUDA_SYNC:
494 |     case libkineto::ActivityType::GPU_USER_ANNOTATION:
495 |     case libkineto::ActivityType::CUDA_PROFILER_RANGE:
496 |       return device_type_privateuse1_or(c10::DeviceType::CUDA);
497 |     // TODO: T151322015
498 |     case libkineto::ActivityType::MTIA_CCP_EVENTS:
499 |     case libkineto::ActivityType::MTIA_INSIGHT:
500 |     case libkineto::ActivityType::MTIA_COUNTERS:
501 |       return device_type_privateuse1_or(c10::DeviceType::MTIA);
502 |     case libkineto::ActivityType::HPU_OP:
503 |       return c10::DeviceType::HPU;
504 |     case libkineto::ActivityType::CPU_OP:
505 |     case libkineto::ActivityType::USER_ANNOTATION:
506 |     case libkineto::ActivityType::EXTERNAL_CORRELATION:
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`autograd::profiler`) so ownership matches the PyTorch subsystem layout. Implements routines such as `logInvariantViolation`, `deviceTypeFromActivity`, `device_type_privateuse1_or` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`autograd::profiler`），使其归属与 PyTorch 子系统布局保持一致。 实现了 `logInvariantViolation`、`deviceTypeFromActivity`、`device_type_privateuse1_or` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 507-555
```cpp
507 |     case libkineto::ActivityType::CUDA_RUNTIME:
508 |     case libkineto::ActivityType::XPU_RUNTIME:
509 |     case libkineto::ActivityType::XPU_DRIVER:
510 |     case libkineto::ActivityType::CPU_INSTANT_EVENT:
511 |     case libkineto::ActivityType::GLOW_RUNTIME:
512 |     case libkineto::ActivityType::MTIA_RUNTIME:
513 |     case libkineto::ActivityType::PYTHON_FUNCTION:
514 |     case libkineto::ActivityType::CUDA_DRIVER:
515 |     case libkineto::ActivityType::PRIVATEUSE1_RUNTIME:
516 |     case libkineto::ActivityType::PRIVATEUSE1_DRIVER:
517 |     case libkineto::ActivityType::OVERHEAD:
518 |       return c10::DeviceType::CPU;
519 |     default: {
520 |       TORCH_WARN(
521 |           "Unknown activity type (",
522 |           (uint8_t)activity_type,
523 |           "), assuming CPU device");
524 |       return c10::DeviceType::CPU;
525 |     }
526 |   }
527 | }
528 | 
529 | void addMetadataJson(const std::string& key, const std::string& value) {
530 | #ifdef USE_KINETO
531 |   if (libkineto::api().isProfilerInitialized()) {
532 |     libkineto::api().activityProfiler().addMetadata(key, value);
533 |   } else {
534 |     LOG(WARNING) << "Profiler is not initialized: skipping profiling metadata";
535 |   }
536 | #else
537 |   LOG(WARNING) << "Adding profiling metadata requires using "
538 |                << "torch.profiler with Kineto support (USE_KINETO=1)";
539 | #endif // USE_KINETO
540 | }
541 | 
542 | void profilerStep() {
543 | #ifdef USE_KINETO
544 |   libkineto::api().initProfilerIfRegistered();
545 | 
546 |   if (libkineto::api().isProfilerInitialized()) {
547 |     libkineto::api().activityProfiler().step();
548 |   } else {
549 |     VLOG(1) << "Profiler is not initialized: skipping step() invocation";
550 |   }
551 | #endif // USE_KINETO
552 | }
553 | 
554 | } // namespace autograd::profiler
555 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `addMetadataJson`, `profilerStep` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `addMetadataJson`、`profilerStep` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 556-556
```cpp
556 | } // namespace torch
```
- EN: Continues the file's main role: Integrates PyTorch profiling with the Kineto tracing backend.
- CN: 继续承担本文件的主要职责：将 PyTorch 性能分析与 Kineto 跟踪后端集成。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `ExperimentalConfigWrapper`.
  - CN: `ExperimentalConfigWrapper`。
- **Important routines / 重要例程**
  - EN: `filterActivities`, `kineto_ids`, `addMetadata`, `bool`, `ExperimentalConfigWrapper`, `assertValid`, `prepareTraceWithExperimentalOptions`, `collectivesProfilerExists`.
  - CN: `filterActivities`、`kineto_ids`、`addMetadata`、`bool`、`ExperimentalConfigWrapper`、`assertValid`、`prepareTraceWithExperimentalOptions`、`collectivesProfilerExists`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch`, `profiler::impl::kineto`, `autograd::profiler`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch`、`profiler::impl::kineto`、`autograd::profiler` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/profiler/collection.h>`, `<torch/csrc/profiler/kineto_shim.h>`, `<c10/util/Exception.h>`, `<c10/util/env.h>`
- External includes / 外部头文件: `<type_traits>`, `<libkineto.h>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
