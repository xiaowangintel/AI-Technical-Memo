# profiler_legacy.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/autograd/profiler_legacy.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Implements autograd profiler integration used to observe operator execution and runtime behavior.
- 目的 (CN): 实现自动求导性能分析集成，用于观测算子执行与运行时行为。
- Lines: 678
- Language: C++ / C++

## Line-by-Line Analysis / 逐行分析

### Lines 1-16

```cpp
 1: #include <torch/csrc/autograd/profiler_legacy.h>
 2: 
 3: #include <torch/csrc/autograd/function.h>
 4: #include <torch/csrc/jit/frontend/tracer.h>
 5: #include <torch/csrc/jit/runtime/interpreter.h>
 6: #include <torch/csrc/jit/runtime/operator.h>
 7: 
 8: #include <ATen/code_template.h>
 9: #include <ATen/core/op_registration/op_registration.h>
10: #include <torch/library.h>
11: 
12: #include <fstream>
13: #include <mutex>
14: #include <string>
15: #include <vector>
16: 
```

- EN: These lines pull in dependencies such as `torch/csrc/autograd/profiler_legacy.h`, `torch/csrc/autograd/function.h`, `torch/csrc/jit/frontend/tracer.h`, establishing the headers needed by the implementation.
- CN: 这些行引入了依赖，例如 `torch/csrc/autograd/profiler_legacy.h`, `torch/csrc/autograd/function.h`, `torch/csrc/jit/frontend/tracer.h`，为后续实现建立所需的头文件基础。
### Lines 17-32

```cpp
17: #include <ATen/record_function.h>
18: #include <c10/core/Allocator.h>
19: #include <c10/util/ApproximateClock.h>
20: #include <c10/util/ThreadLocalDebugInfo.h>
21: #include <c10/util/irange.h>
22: 
23: #include <iostream>
24: 
25: namespace torch::autograd::profiler {
26: 
27: // We decompose the profiler logic into the following components:
28: //
29: // ThreadLocalDebugInfo:
30: //
31: // ThreadLocalDebugInfo is a thread local mapping from slots into
32: // the debug information structs.
```

- EN: These lines pull in dependencies such as `ATen/record_function.h`, `c10/core/Allocator.h`, `c10/util/ApproximateClock.h`, establishing the headers needed by the implementation. They also open, refine, or close the relevant C++ namespace scope.
- CN: 这些行引入了依赖，例如 `ATen/record_function.h`, `c10/core/Allocator.h`, `c10/util/ApproximateClock.h`，为后续实现建立所需的头文件基础。 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。
### Lines 33-48

```cpp
33: // ThreadLocalDebugInfo is automatically propagated across thread
34: // boundaries, including the cases of:
35: //  - launching async jobs with at::launch
36: //  - executing JIT continuations
37: //  - moving from the forward threads into autograd (backward) threads
38: //
39: // Entries in ThreadLocalDebugInfo are managed by DebugInfoGuard
40: // which can be used to add or overwrite an entry in the thread local
41: // mapping. A corresponding entry is removed when the guard is destroyed,
42: // potentially revealing the previously set value for the same slot.
43: //
44: // For the async tasks, slots previously set in the main thread before
45: // launching of an async task are shared and visible in the async task.
46: //
47: // On the other hand, any adding or overwriting of the mapping by the
48: // async task is not visible to the main thread and any modification
```

- EN: This range is comment-only; it documents the surrounding implementation, generated provenance, or usage notes.
- CN: 这一段仅包含注释，用于说明周围实现、生成来源或使用说明。
### Lines 49-64

```cpp
49: // (including removal of the entries) in the main thread is not visible
50: // to the async task if it happens after launching the task.
51: //
52: // We use ThreadLocalDebugInfo (slot PROFILER_STATE) to store profiler config,
53: // as well as a list of events that happen during profiling.
54: // An instance of ThreadLocalDebugInfo is created each time we enter
55: // profiler (i.e. enter profiling context manager/call enableConfig) and
56: // uniquely identifies a profiling run.
57: //
58: // We automatically propagate ThreadLocalDebugInfo into async tasks,
59: // as well as across JIT continuations and autograd thread, so all
60: // the operations that happen between profiling start and end
61: // (not necessarily within the same thread) are recorded.
62: // Unless the profiling slot is overwritten as in the case of nested
63: // profiling ranges (in this case events for the subrange are handled
64: // by the nested profiler)
```

- EN: This range is comment-only; it documents the surrounding implementation, generated provenance, or usage notes.
- CN: 这一段仅包含注释，用于说明周围实现、生成来源或使用说明。
### Lines 65-80

```cpp
65: //
66: // When we exit a profiling range (either by exiting profiling context
67: // manager or by calling disableProfiler), we remove the previously set
68: // profiling entry for the given thread local mapping, and consolidate
69: // events in the profiling result
70: //
71: //
72: // ThreadLocalState:
73: //
74: // ThreadLocalState takes a 'snapshot' of thread local variables
75: // using provided getters. It is used together with ThreadLocalStateGuard
76: // to transfer the snapshot across thread boundary and set the thread local
77: // values as in the parent task.
78: //
79: // Profiler uses ThreadLocalState to propagate profiler's thread local state.
80: // ThreadLocalState also automatically propagates profiler callbacks.
```

- EN: This range is comment-only; it documents the surrounding implementation, generated provenance, or usage notes.
- CN: 这一段仅包含注释，用于说明周围实现、生成来源或使用说明。
### Lines 81-96

```cpp
81: //
82: //
83: // at::RecordFunction and observers
84: //
85: // Profiler uses observers mechanism to add a pair of thread local callbacks
86: // that are executed on a number of predetermined ranges, including:
87: //  - c10/ATen ops
88: //  - TorchScript functions/methods
89: //  - user defined named ranges (see `record_function` python context manager)
90: //
91: // Profiler setups a pair of callbacks that record profiling events and save
92: // them into the thread local profiler struct (ThreadLocalDebugInfo,
93: // PROFILER_STATE slot)
94: //
95: //
96: // Thus, the overall logic is:
```

- EN: This range is comment-only; it documents the surrounding implementation, generated provenance, or usage notes.
- CN: 这一段仅包含注释，用于说明周围实现、生成来源或使用说明。
### Lines 97-112

```cpp
 97: //
 98: // enableProfiler:
 99: //  - checks that profiler is not enabled (otherwise throws)
100: //  - pushes new ThreadLocalDebugInfo (slot PROFILER_STATE) as the profiler
101: //    config for the current thread
102: //  - pushes profiling callbacks for the current thread
103: //
104: // disableProfiler:
105: //  - pops PROFILER_STATE slot from the current ThreadLocalDebugInfo and
106: //    consolidates events
107: //  - removes profiling callbacks
108: //
109: // ThreadLocalState:
110: //  - propagates ThreadLocalDebugInfo across threads
111: //  - propagates profiler callbacks across threads
112: //
```

- EN: This range is comment-only; it documents the surrounding implementation, generated provenance, or usage notes.
- CN: 这一段仅包含注释，用于说明周围实现、生成来源或使用说明。
### Lines 113-128

```cpp
113: // Profiler callbacks:
114: //  - get the current profiling state (PROFILER slot in ThreadLocalDebugInfo)
115: //  - save profiling events into the profiling state
116: //
117: 
118: namespace {
119: using torch::profiler::impl::ActiveProfilerType;
120: using torch::profiler::impl::ProfilerStateBase;
121: 
122: struct ProfilerLegacyThreadLocalState : public ProfilerStateBase {
123:   explicit ProfilerLegacyThreadLocalState(
124:       const torch::profiler::impl::ProfilerConfig& config)
125:       : ProfilerStateBase(config), remoteProfiledEvents_{std::nullopt} {}
126:   ~ProfilerLegacyThreadLocalState() override = default;
127: 
128:   static ProfilerLegacyThreadLocalState* getTLS() {
```

- EN: They also open, refine, or close the relevant C++ namespace scope. This range declares or shapes types such as `ProfilerLegacyThreadLocalState`. The main execution path in this span is carried by `state`, `ProfilerLegacyThreadLocalState`, `ProfilerStateBase`.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段声明或塑造了 ``ProfilerLegacyThreadLocalState`` 等类型。 这一段的主要执行路径由 `state`, `ProfilerLegacyThreadLocalState`, `ProfilerStateBase` 等函数/方法承载。
### Lines 129-144

```cpp
129:     auto tls = ProfilerStateBase::get(/*global=*/false);
130:     TORCH_INTERNAL_ASSERT_DEBUG_ONLY(
131:         tls == nullptr || tls->profilerType() == ActiveProfilerType::LEGACY);
132:     return static_cast<ProfilerLegacyThreadLocalState*>(tls);
133:   }
134: 
135:   thread_event_lists consolidate();
136: 
137:   void mark(std::string name, bool include_cuda = true);
138: 
139:   void setOrAddRemoteProfiledEvents(
140:       std::vector<LegacyEvent>&& remoteProfiledEvents);
141: 
142:   void pushRange(
143:       const at::RecordFunction& fn,
144:       const bool record_cuda,
```

- EN: The main execution path in this span is carried by `get`, `TORCH_INTERNAL_ASSERT_DEBUG_ONLY`, `consolidate`. The logic emits runtime diagnostics or assertions to guard assumptions. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `get`, `TORCH_INTERNAL_ASSERT_DEBUG_ONLY`, `consolidate` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 145-160

```cpp
145:       std::vector<std::vector<int64_t>>&& shapes = {});
146: 
147:   void popRange(const at::RecordFunction& fn, const bool record_cuda);
148: 
149:   void reportMemoryUsage(
150:       void* /* unused */,
151:       int64_t alloc_size,
152:       size_t /* total_allocated, unused for legacy */,
153:       size_t /* total_reserved, unused for legacy */,
154:       c10::Device device) override;
155: 
156:   ActiveProfilerType profilerType() override {
157:     return ActiveProfilerType::LEGACY;
158:   }
159: 
160:   void leakHandle() {
```

- EN: The main execution path in this span is carried by `popRange`, `reportMemoryUsage`, `profilerType`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `popRange`, `reportMemoryUsage`, `profilerType` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 161-176

```cpp
161:     handle_ = 0;
162:   }
163: 
164:  protected:
165:   RangeEventList& getEventList(
166:       std::optional<uint64_t> thread_id = std::nullopt);
167: 
168:   std::mutex state_mutex_;
169:   std::unordered_map<uint64_t, std::shared_ptr<RangeEventList>>
170:       event_lists_map_;
171: 
172:   std::optional<std::vector<std::vector<LegacyEvent>>> remoteProfiledEvents_;
173: };
174: 
175: thread_event_lists ProfilerLegacyThreadLocalState::consolidate() {
176:   std::lock_guard<std::mutex> g(state_mutex_);
```

- EN: The main execution path in this span is carried by `getEventList`, `consolidate`, `g`.
- CN: 这一段的主要执行路径由 `getEventList`, `consolidate`, `g` 等函数/方法承载。
### Lines 177-192

```cpp
177:   thread_event_lists result;
178:   for (auto& kv : event_lists_map_) {
179:     auto& list = kv.second;
180:     result.emplace_back(list->consolidate());
181:   }
182:   // Consolidate remote events if applicable as well.
183:   if (remoteProfiledEvents_) {
184:     result.insert(
185:         result.end(),
186:         std::make_move_iterator(remoteProfiledEvents_->begin()),
187:         std::make_move_iterator(remoteProfiledEvents_->end()));
188:   }
189:   return result;
190: }
191: 
192: void ProfilerLegacyThreadLocalState::mark(std::string name, bool include_cuda) {
```

- EN: The main execution path in this span is carried by `make_move_iterator`, `mark`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `make_move_iterator`, `mark` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 193-208

```cpp
193:   if (config_.disabled()) {
194:     return;
195:   }
196:   if (config_.state == torch::profiler::impl::ProfilerState::NVTX) {
197:     torch::profiler::impl::cudaStubs()->mark(name.c_str());
198:   } else {
199:     LegacyEvent evt(
200:         EventKind::Mark,
201:         at::StringView(std::move(name)),
202:         at::RecordFunction::currentThreadId(),
203:         include_cuda &&
204:             config_.state == torch::profiler::impl::ProfilerState::CUDA);
205:     evt.setNodeId(at::RecordFunction::getDefaultNodeId());
206:     getEventList().record(std::move(evt));
207:   }
208: }
```

- EN: The main execution path in this span is carried by `cudaStubs`, `evt`, `StringView`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `cudaStubs`, `evt`, `StringView` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 209-224

```cpp
209: 
210: void ProfilerLegacyThreadLocalState::setOrAddRemoteProfiledEvents(
211:     std::vector<LegacyEvent>&& remoteProfiledEvents) {
212:   // Lock to serialize access from multiple callback threads.
213:   std::lock_guard<std::mutex> guard(state_mutex_);
214:   if (remoteProfiledEvents_) {
215:     (*remoteProfiledEvents_).emplace_back(remoteProfiledEvents);
216:   } else {
217:     remoteProfiledEvents_ = {std::move(remoteProfiledEvents)};
218:   }
219: }
220: 
221: void ProfilerLegacyThreadLocalState::pushRange(
222:     const at::RecordFunction& fn,
223:     const bool record_cuda,
224:     std::vector<std::vector<int64_t>>&& shapes) {
```

- EN: The main execution path in this span is carried by `setOrAddRemoteProfiledEvents`, `guard`, `move`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `setOrAddRemoteProfiledEvents`, `guard`, `move` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 225-240

```cpp
225:   if (config_.disabled()) {
226:     return;
227:   }
228:   if (config_.state == torch::profiler::impl::ProfilerState::NVTX) {
229:     torch::profiler::impl::cudaStubs()->rangePush(
230:         torch::profiler::impl::getNvtxStr(fn.name(), fn.seqNr(), shapes)
231:             .c_str());
232:   } else {
233:     LegacyEvent evt(
234:         EventKind::PushRange,
235:         at::StringView(std::string(fn.name())),
236:         at::RecordFunction::currentThreadId(),
237:         record_cuda,
238:         fn.handle(),
239:         std::move(shapes),
240:         at::RecordFunction::getDefaultNodeId(),
```

- EN: The main execution path in this span is carried by `cudaStubs`, `getNvtxStr`, `evt`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `cudaStubs`, `getNvtxStr`, `evt` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 241-256

```cpp
241:         fn.isAsync());
242:     evt.setSequenceNr(fn.seqNr());
243:     evt.setFwdThreadId(fn.forwardThreadId());
244:     evt.setScope((uint8_t)fn.scope());
245:     if (config_.with_flops) {
246:       evt.setExtraArgs(torch::profiler::impl::saveExtraArgs(fn));
247:       evt.setFlops(torch::profiler::impl::computeFlops(
248:           std::string(fn.name()), evt.extraArgs()));
249:     }
250: 
251: // TODO: will unify the two macros BUILD_LITE_INTERPRETER and C10_MOBILE soon.
252: #if !defined BUILD_LITE_INTERPRETER && !defined C10_MOBILE
253:     // backward nodes source range corresponds to the forward node
254:     // TODO: consider using C++ stack trace
255:     if (config_.with_stack &&
256:         fn.scope() != at::RecordScope::BACKWARD_FUNCTION) {
```

- EN: The main execution path in this span is carried by `string`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `string` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 257-272

```cpp
257:       auto cs =
258:           torch::profiler::impl::prepareCallstack(jit::currentCallstack());
259:       if (cs.empty()) {
260:         cs = torch::profiler::impl::prepareCallstack(
261:             jit::tracer::pythonCallstack());
262:       }
263:       evt.setStack(callstackStr(cs));
264:     }
265: #endif
266:     getEventList().record(std::move(evt));
267:   }
268: }
269: 
270: void ProfilerLegacyThreadLocalState::popRange(
271:     const at::RecordFunction& fn,
272:     const bool record_cuda) {
```

- EN: The main execution path in this span is carried by `prepareCallstack`, `pythonCallstack`, `getEventList`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `prepareCallstack`, `pythonCallstack`, `getEventList` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 273-288

```cpp
273:   if (config_.disabled()) {
274:     return;
275:   }
276:   if (config_.state == torch::profiler::impl::ProfilerState::NVTX) {
277:     torch::profiler::impl::cudaStubs()->rangePop();
278:   } else {
279:     // In some cases RecordFunction (and popRange) may be
280:     // called on a different thread than pushRange
281:     // As a convention, we put the async pop on the original
282:     // thread and save current thread id in pop event
283:     LegacyEvent evt(
284:         EventKind::PopRange,
285:         at::StringView(""),
286:         at::RecordFunction::currentThreadId(),
287:         record_cuda,
288:         fn.handle());
```

- EN: The main execution path in this span is carried by `cudaStubs`, `RecordFunction`, `evt`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `cudaStubs`, `RecordFunction`, `evt` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 289-304

```cpp
289:     evt.setNodeId(at::RecordFunction::getDefaultNodeId());
290:     getEventList(fn.threadId()).record(std::move(evt));
291:   }
292: }
293: 
294: void ProfilerLegacyThreadLocalState::reportMemoryUsage(
295:     void* /* unused */,
296:     int64_t alloc_size,
297:     size_t /* total_allocated, unused for legacy */,
298:     size_t /* total_reserved, unused for legacy */,
299:     c10::Device device) {
300:   if (config_.profile_memory && !config_.disabled()) {
301:     uint64_t thread_id = at::RecordFunction::currentThreadId();
302:     LegacyEvent evt(
303:         EventKind::MemoryAlloc,
304:         at::StringView(""),
```

- EN: The main execution path in this span is carried by `getEventList`, `reportMemoryUsage`, `currentThreadId`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `getEventList`, `reportMemoryUsage`, `currentThreadId` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 305-320

```cpp
305:         thread_id,
306:         config_.state == torch::profiler::impl::ProfilerState::CUDA);
307:     evt.updateMemoryStats(alloc_size, device);
308:     getEventList(thread_id).record(std::move(evt));
309:   }
310: }
311: 
312: RangeEventList& ProfilerLegacyThreadLocalState::getEventList(
313:     std::optional<uint64_t> thread_id) {
314:   if (!thread_id.has_value()) {
315:     thread_id = at::RecordFunction::currentThreadId();
316:   }
317:   RangeEventList* list_ptr = nullptr;
318:   std::lock_guard<std::mutex> guard(state_mutex_);
319:   auto it = event_lists_map_.find(thread_id.value());
320:   if (it != event_lists_map_.end()) {
```

- EN: The main execution path in this span is carried by `getEventList`, `currentThreadId`, `guard`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `getEventList`, `currentThreadId`, `guard` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 321-336

```cpp
321:     list_ptr = it->second.get();
322:   } else {
323:     auto event_list = std::make_shared<RangeEventList>();
324:     event_lists_map_[thread_id.value()] = event_list;
325:     list_ptr = event_list.get();
326:   }
327:   return *list_ptr;
328: }
329: 
330: enum EventIValueIdx {
331:   KIND = 0,
332:   NAME,
333:   THREAD_ID,
334:   HANDLE,
335:   NODE_ID,
336:   CPU_MEM_USAGE,
```

- EN: This range declares or shapes types such as `EventIValueIdx`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段声明或塑造了 ``EventIValueIdx`` 等类型。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 337-352

```cpp
337:   CPU_NS,
338:   CUDA_RECORDED,
339:   CUDA_MEM_USAGE,
340:   CUDA_DEVICE,
341:   CUDA_US,
342:   SHAPES,
343:   NUM_EVENT_IVALUE_IDX // must be last in list
344: };
345: 
346: const std::unordered_set<std::string> disable_cuda_profiling = {
347:     "aten::view",
348:     "aten::t",
349:     "aten::transpose",
350:     "aten::stride",
351:     "aten::empty",
352:     "aten::empty_like",
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 353-368

```cpp
353:     "aten::empty_strided",
354:     "aten::as_strided",
355:     "aten::expand",
356:     "aten::resize_",
357:     "aten::squeeze",
358:     "aten::unsqueeze",
359:     "aten::slice",
360:     "aten::_unsafe_view",
361:     "aten::size"};
362: 
363: void pushProfilingCallbacksLegacy() {
364:   auto registration_state_ptr = ProfilerLegacyThreadLocalState::getTLS();
365:   TORCH_INTERNAL_ASSERT(registration_state_ptr, "Expected profiler state set");
366:   auto handle = at::addThreadLocalCallback(
367:       at::RecordFunctionCallback(
368:           [](const at::RecordFunction& fn)
```

- EN: The main execution path in this span is carried by `pushProfilingCallbacksLegacy`, `getTLS`, `TORCH_INTERNAL_ASSERT`. The logic emits runtime diagnostics or assertions to guard assumptions.
- CN: 这一段的主要执行路径由 `pushProfilingCallbacksLegacy`, `getTLS`, `TORCH_INTERNAL_ASSERT` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。
### Lines 369-384

```cpp
369:               -> std::unique_ptr<at::ObserverContext> {
370:             auto state_ptr = ProfilerLegacyThreadLocalState::getTLS();
371:             if (!state_ptr || state_ptr->config().disabled()) {
372:               return nullptr;
373:             }
374:             bool record_cuda = state_ptr->config().state ==
375:                 torch::profiler::impl::ProfilerState::CUDA;
376:             if (record_cuda &&
377:                 disable_cuda_profiling.find(fn.name()) !=
378:                     disable_cuda_profiling.end()) {
379:               record_cuda = false;
380:             }
381: 
382:             if (state_ptr->config().report_input_shapes) {
383:               auto sizes = torch::profiler::impl::inputSizes(fn);
384:               state_ptr->pushRange(fn, record_cuda, std::move(sizes));
```

- EN: The main execution path in this span is carried by `getTLS`, `inputSizes`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `getTLS`, `inputSizes` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 385-400

```cpp
385:             } else {
386:               state_ptr->pushRange(fn, record_cuda);
387:             }
388: 
389:             return nullptr;
390:           },
391:           [](const at::RecordFunction& fn, at::ObserverContext*) {
392:             auto state_ptr = ProfilerLegacyThreadLocalState::getTLS();
393:             if (!state_ptr || state_ptr->config().disabled()) {
394:               return;
395:             }
396:             bool record_cuda = state_ptr->config().state ==
397:                 torch::profiler::impl::ProfilerState::CUDA;
398:             if (record_cuda &&
399:                 disable_cuda_profiling.find(fn.name()) !=
400:                     disable_cuda_profiling.end()) {
```

- EN: The main execution path in this span is carried by `getTLS`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `getTLS` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 401-416

```cpp
401:               record_cuda = false;
402:             }
403:             state_ptr->popRange(fn, record_cuda);
404:           })
405:           .needsInputs(registration_state_ptr->config().report_input_shapes)
406:           .needsIds(true));
407:   registration_state_ptr->setCallbackHandle(handle);
408: }
409: 
410: } // namespace
411: 
412: void enableProfilerLegacy(
413:     const torch::profiler::impl::ProfilerConfig& new_config) {
414:   TORCH_CHECK(
415:       new_config.state != torch::profiler::impl::ProfilerState::NVTX ||
416:           torch::profiler::impl::cudaStubs()->enabled(),
```

- EN: The main execution path in this span is carried by `enableProfilerLegacy`, `TORCH_CHECK`, `cudaStubs`. The logic emits runtime diagnostics or assertions to guard assumptions.
- CN: 这一段的主要执行路径由 `enableProfilerLegacy`, `TORCH_CHECK`, `cudaStubs` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。
### Lines 417-432

```cpp
417:       "Can't use NVTX profiler - PyTorch was compiled without CUDA");
418: 
419:   TORCH_CHECK(new_config.state != torch::profiler::impl::ProfilerState::KINETO);
420: 
421:   auto state_ptr = ProfilerLegacyThreadLocalState::getTLS();
422:   TORCH_CHECK(!state_ptr, "Profiler is already enabled on this thread");
423:   auto state = std::make_shared<ProfilerLegacyThreadLocalState>(new_config);
424:   c10::ThreadLocalDebugInfo::_push(c10::DebugInfoKind::PROFILER_STATE, state);
425: 
426:   pushProfilingCallbacksLegacy();
427: 
428:   state->mark("__start_profile", false);
429: }
430: 
431: thread_event_lists disableProfilerLegacy(
432:     std::optional<ProfilerDisableOptions> profilerDisableOptions) {
```

- EN: The main execution path in this span is carried by `TORCH_CHECK`, `getTLS`, `_push`. The logic emits runtime diagnostics or assertions to guard assumptions.
- CN: 这一段的主要执行路径由 `TORCH_CHECK`, `getTLS`, `_push` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。
### Lines 433-448

```cpp
433:   auto cleanupTLSState =
434:       profilerDisableOptions ? profilerDisableOptions->cleanupTLSState : true;
435:   auto consolidate =
436:       profilerDisableOptions ? profilerDisableOptions->consolidate : true;
437:   // all the DebugInfoBase objects are scope based and supposed to use
438:   // DebugInfoGuard
439:   std::shared_ptr<c10::DebugInfoBase> state;
440:   if (cleanupTLSState) {
441:     state = c10::ThreadLocalDebugInfo::_pop(c10::DebugInfoKind::PROFILER_STATE);
442:   } else {
443:     state =
444:         c10::ThreadLocalDebugInfo::_peek(c10::DebugInfoKind::PROFILER_STATE);
445:   }
446: 
447:   auto state_ptr = static_cast<ProfilerLegacyThreadLocalState*>(state.get());
448:   TORCH_CHECK(
```

- EN: The main execution path in this span is carried by `_pop`, `_peek`, `TORCH_CHECK`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `_pop`, `_peek`, `TORCH_CHECK` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 449-464

```cpp
449:       state_ptr && !state_ptr->config().disabled(),
450:       "Can't disable profiler when it's not running");
451: 
452:   cleanupTLSState ? state_ptr->removeCallback() : state_ptr->leakHandle();
453:   if (!consolidate ||
454:       state_ptr->config().state == torch::profiler::impl::ProfilerState::NVTX) {
455:     return thread_event_lists();
456:   }
457: 
458:   state_ptr->mark("__stop_profile", false);
459:   // Note that this will erase the underlying events.
460:   return state_ptr->consolidate();
461: }
462: 
463: void addEventList(std::vector<LegacyEvent>&& profiledEvents) {
464:   auto state_ptr = ProfilerLegacyThreadLocalState::getTLS();
```

- EN: The main execution path in this span is carried by `thread_event_lists`, `addEventList`, `getTLS`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `thread_event_lists`, `addEventList`, `getTLS` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 465-480

```cpp
465:   TORCH_CHECK(state_ptr, "Profiler must be enabled.");
466:   state_ptr->setOrAddRemoteProfiledEvents(std::move(profiledEvents));
467: }
468: 
469: void LegacyEvent::record(bool record_cuda) {
470:   if (record_cuda) {
471:     torch::profiler::impl::cudaStubs()->record(&device_, &cuda_event, &cpu_ns_);
472:     return;
473:   }
474:   cpu_ns_ = c10::getTime();
475: }
476: 
477: /* static */ LegacyEvent LegacyEvent::fromIValue(
478:     const at::IValue& eventIValue) {
479:   TORCH_INTERNAL_ASSERT(
480:       eventIValue.isList(),
```

- EN: The main execution path in this span is carried by `TORCH_CHECK`, `record`, `cudaStubs`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `TORCH_CHECK`, `record`, `cudaStubs` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 481-496

```cpp
481:       "Expected IValue to contain type c10::impl::GenericList");
482:   auto ivalues = eventIValue.toList();
483:   TORCH_INTERNAL_ASSERT(
484:       ivalues.size() >= NUM_EVENT_IVALUE_IDX,
485:       "Expected at least ",
486:       NUM_EVENT_IVALUE_IDX,
487:       " elements to reconstruct LegacyEvent.");
488: 
489:   // Reconstruct input shapes from ivalues.
490:   const auto& shapeListIValue = ivalues.get(EventIValueIdx::SHAPES);
491:   TORCH_INTERNAL_ASSERT(
492:       shapeListIValue.isList(),
493:       "Expected profiler shapes IValue to contain type c10::impl::GenericList.");
494: 
495:   auto shapeList = shapeListIValue.toList();
496:   std::vector<std::vector<int64_t>> shapes;
```

- EN: The main execution path in this span is carried by `TORCH_INTERNAL_ASSERT`. The logic emits runtime diagnostics or assertions to guard assumptions.
- CN: 这一段的主要执行路径由 `TORCH_INTERNAL_ASSERT` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。
### Lines 497-512

```cpp
497:   shapes.reserve(shapeList.size());
498:   for (const auto i : c10::irange(shapeList.size())) {
499:     std::vector<int64_t> s;
500:     const auto& shapeIValue = shapeList.get(i);
501:     TORCH_INTERNAL_ASSERT(
502:         shapeIValue.isList(),
503:         "Expected each profiler shape element to contain shapes of type c10::impl::GenericList.")
504:     auto curShapesList = shapeIValue.toList();
505:     s.reserve(curShapesList.size());
506:     for (const auto j : c10::irange(curShapesList.size())) {
507:       s.emplace_back(curShapesList.get(j).toInt());
508:     }
509:     shapes.emplace_back(s);
510:   }
511: 
512:   LegacyEvent evt(
```

- EN: The main execution path in this span is carried by `TORCH_INTERNAL_ASSERT`, `evt`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `TORCH_INTERNAL_ASSERT`, `evt` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 513-528

```cpp
513:       static_cast<EventKind>(
514:           ivalues.get(EventIValueIdx::KIND).toInt()), // EventKind
515:       at::StringView(ivalues.get(EventIValueIdx::NAME).toStringRef()), // name
516:       ivalues.get(EventIValueIdx::THREAD_ID).toInt(), // thread_id
517:       static_cast<at::RecordFunctionHandle>(
518:           ivalues.get(EventIValueIdx::HANDLE).toDouble()), // handle
519:       std::move(shapes), // input shapes
520:       ivalues.get(EventIValueIdx::NODE_ID).toInt(), // node id
521:       true, // is remote
522:       ivalues.get(EventIValueIdx::CPU_MEM_USAGE).toInt(), // cpu_mem_usage
523:       ivalues.get(EventIValueIdx::CPU_NS).toInt(), // cpu_ns
524:       ivalues.get(EventIValueIdx::CUDA_RECORDED).toBool(), // was cuda recorded
525:       ivalues.get(EventIValueIdx::CUDA_MEM_USAGE).toInt(), // cuda memory usage
526:       c10::DeviceIndex(
527:           ivalues.get(EventIValueIdx::CUDA_DEVICE).toInt()), // device
528:       static_cast<double>(
```

- EN: The main execution path in this span is carried by `StringView`, `move`, `DeviceIndex`.
- CN: 这一段的主要执行路径由 `StringView`, `move`, `DeviceIndex` 等函数/方法承载。
### Lines 529-544

```cpp
529:           ivalues.get(EventIValueIdx::CUDA_US).toInt()) // cuda_us
530:   );
531:   return evt;
532: }
533: 
534: at::IValue LegacyEvent::toIValue() const {
535:   c10::impl::GenericList eventIValueList(at::AnyType::get());
536:   eventIValueList.reserve(NUM_EVENT_IVALUE_IDX);
537:   eventIValueList.emplace_back(static_cast<int64_t>(kind_));
538:   eventIValueList.emplace_back(std::string(name_.str()));
539:   eventIValueList.emplace_back(static_cast<int64_t>(thread_id_));
540:   eventIValueList.emplace_back(static_cast<double>(handle_));
541:   eventIValueList.emplace_back(node_id_);
542:   eventIValueList.emplace_back(cpu_memory_usage_);
543:   eventIValueList.emplace_back(cpu_ns_);
544:   // CUDA event information
```

- EN: The main execution path in this span is carried by `toIValue`, `eventIValueList`. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `toIValue`, `eventIValueList` 等函数/方法承载。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 545-560

```cpp
545:   bool cuda_profiling_enabled = hasCuda();
546:   eventIValueList.emplace_back(cuda_profiling_enabled);
547:   eventIValueList.emplace_back(static_cast<int64_t>(cuda_memory_usage_));
548:   eventIValueList.emplace_back(device_);
549:   eventIValueList.emplace_back(cuda_us_);
550:   // Shapes
551:   c10::impl::GenericList shapesList =
552:       c10::impl::GenericList(at::ListType::create(at::IntType::get()));
553:   shapesList.reserve(shapes_.size());
554:   for (const auto& shape : shapes_) {
555:     c10::impl::GenericList s = c10::impl::GenericList(at::IntType::get());
556:     s.reserve(shape.size());
557:     for (const auto& k : shape) {
558:       s.emplace_back(k);
559:     }
560:     shapesList.emplace_back(s);
```

- EN: The main execution path in this span is carried by `hasCuda`, `GenericList`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `hasCuda`, `GenericList` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 561-576

```cpp
561:   }
562:   eventIValueList.emplace_back(shapesList);
563:   return at::IValue(eventIValueList);
564: }
565: 
566: double LegacyEvent::cudaElapsedUs(const LegacyEvent& e) const {
567:   TORCH_CHECK(e.hasCuda() && hasCuda(), "Events were not recorded for CUDA");
568:   TORCH_CHECK(
569:       e.device() == device(),
570:       c10::str(
571:           "Events are not on the same device: ", e.device(), " vs ", device()));
572:   if (isRemote() && e.isRemote()) {
573:     // validate that cuda_us_ has been set properly.
574:     TORCH_INTERNAL_ASSERT(cuda_us_ >= 0 && e.cuda_us_ >= 0);
575:     return static_cast<double>(e.cuda_us_ - cuda_us_);
576:   }
```

- EN: The main execution path in this span is carried by `IValue`, `cudaElapsedUs`, `TORCH_CHECK`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `IValue`, `cudaElapsedUs`, `TORCH_CHECK` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 577-592

```cpp
577:   return torch::profiler::impl::cudaStubs()->elapsed(
578:       &cuda_event, &e.cuda_event);
579: }
580: 
581: static const at::jit::CodeTemplate event_template(R"(
582: {
583:   "name": "${name}",
584:   "ph": "X",
585:   "ts": ${ts},
586:   "dur": ${dur},
587:   "tid": ${tid},
588:   "pid": "CPU Functions",
589:   "args": {}
590: })");
591: 
592: void writeProfilerEventsToStream(
```

- EN: The main execution path in this span is carried by `cudaStubs`, `event_template`, `writeProfilerEventsToStream`. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cudaStubs`, `event_template`, `writeProfilerEventsToStream` 等函数/方法承载。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 593-608

```cpp
593:     std::ostream& out,
594:     const std::vector<LegacyEvent*>& events) {
595:   TORCH_CHECK(out, "Could not open file");
596:   LegacyEvent* profiler_start = nullptr;
597:   for (LegacyEvent* e : events) {
598:     if (0 == strcmp(e->name(), "__start_profile")) {
599:       profiler_start = e;
600:       break;
601:     }
602:   }
603:   TORCH_CHECK(profiler_start, "Could not find __start_profile mark");
604: 
605:   struct PairHash {
606:     size_t operator()(
607:         std::pair<at::RecordFunctionHandle, int> p) const noexcept {
608:       return std::hash<at::RecordFunctionHandle>()(p.first) ^
```

- EN: This range declares or shapes types such as `PairHash`. The main execution path in this span is carried by `TORCH_CHECK`, `operator`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段声明或塑造了 ``PairHash`` 等类型。 这一段的主要执行路径由 `TORCH_CHECK`, `operator` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 609-624

```cpp
609:           std::hash<int64_t>()(p.second);
610:     }
611:   };
612:   std::unordered_map<
613:       std::pair<at::RecordFunctionHandle, int64_t>,
614:       LegacyEvent*,
615:       PairHash>
616:       events_map;
617:   out << "[\n";
618:   bool first = true;
619:   for (LegacyEvent* evt : events) {
620:     if (evt->kindStr() == "push") {
621:       events_map[std::make_pair(evt->handle(), evt->nodeId())] = evt;
622:     } else if (evt->kindStr() == "pop") {
623:       if (!first) {
624:         out << ",\n";
```

- EN: The main execution path in this span is carried by `make_pair`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `make_pair` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 625-640

```cpp
625:       }
626:       first = false;
627:       auto it = events_map.find(std::make_pair(evt->handle(), evt->nodeId()));
628:       TORCH_CHECK(it != events_map.end(), "Unmatched pop event");
629:       LegacyEvent* evt_start = it->second;
630:       events_map.erase(it);
631: 
632:       at::jit::TemplateEnv env;
633:       env.s("name", evt_start->name());
634:       env.d("ts", profiler_start->cpuElapsedUs(*evt_start));
635:       env.d("dur", evt_start->cpuElapsedUs(*evt));
636:       env.d("tid", evt_start->threadId());
637:       out << event_template.format(env);
638:     }
639:   }
640:   out << "]\n";
```

- EN: The main execution path in this span is carried by `TORCH_CHECK`. The logic emits runtime diagnostics or assertions to guard assumptions.
- CN: 这一段的主要执行路径由 `TORCH_CHECK` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。
### Lines 641-656

```cpp
641: }
642: 
643: RecordProfile::RecordProfile(std::ostream& out) : out_(out) {
644:   init();
645: }
646: 
647: RecordProfile::RecordProfile(const std::string& filename)
648:     : file_(std::make_unique<std::ofstream>(filename)), out_(*file_) {
649:   init();
650: }
651: 
652: void RecordProfile::init() {
653:   enableProfilerLegacy(torch::profiler::impl::ProfilerConfig(
654:       torch::profiler::impl::ProfilerState::CPU));
655: }
656: 
```

- EN: The main execution path in this span is carried by `RecordProfile`, `init`, `file_`.
- CN: 这一段的主要执行路径由 `RecordProfile`, `init`, `file_` 等函数/方法承载。
### Lines 657-672

```cpp
657: RecordProfile::~RecordProfile() {
658:   try {
659:     thread_event_lists event_lists = disableProfilerLegacy();
660:     std::vector<LegacyEvent*> events;
661:     for (auto& l : event_lists) {
662:       for (auto& e : l) {
663:         events.push_back(&e);
664:       }
665:     }
666:     processEvents(events);
667:   } catch (const std::exception& e) {
668:     LOG(ERROR) << e.what() << '\n';
669:   } catch (...) {
670:     LOG(ERROR) << "Unknown error" << '\n';
671:   }
672: }
```

- EN: The main execution path in this span is carried by `RecordProfile`, `disableProfilerLegacy`, `processEvents`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `RecordProfile`, `disableProfilerLegacy`, `processEvents` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 673-678

```cpp
673: 
674: void RecordProfile::processEvents(const std::vector<LegacyEvent*>& events) {
675:   writeProfilerEventsToStream(out_, events);
676: }
677: 
678: } // namespace torch::autograd::profiler
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `processEvents`, `writeProfilerEventsToStream`.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `processEvents`, `writeProfilerEventsToStream` 等函数/方法承载。

## Key Concepts / 关键概念
- Autograd graph and gradient metadata / 自动求导图与梯度元数据
- Gradient bookkeeping and autograd history / 梯度记录与自动求导历史
- Runtime validation and diagnostics / 运行时校验与诊断
- State coordination and lifetime management / 状态协调与生命周期管理
- Primary symbol `ProfilerLegacyThreadLocalState` / 核心符号 `ProfilerLegacyThreadLocalState`
- Primary symbol `EventIValueIdx` / 核心符号 `EventIValueIdx`

## Dependencies / 依赖关系
- Direct includes / 直接包含: `torch/csrc/autograd/profiler_legacy.h`, `torch/csrc/autograd/function.h`, `torch/csrc/jit/frontend/tracer.h`, `torch/csrc/jit/runtime/interpreter.h`, `torch/csrc/jit/runtime/operator.h`, `ATen/code_template.h`, `ATen/core/op_registration/op_registration.h`, `torch/library.h`, `fstream`, `mutex`
- Include roots / 头文件根模块: `ATen`, `c10`, `torch`
- Key symbols / 关键符号: `ProfilerLegacyThreadLocalState`, `EventIValueIdx`, `PairHash`, `getTLS`, `consolidate`, `mark`, `setOrAddRemoteProfiledEvents`, `popRange`, `leakHandle`, `getEventList`
- Related subsystems / 相关子系统: Autograd / 自动求导, ATen operator layer / ATen 算子层, c10 core utilities / c10 核心工具
