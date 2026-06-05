# FlightRecorderDetail.hpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/FlightRecorderDetail.hpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides interface and type declarations for flight recorder detail in the c10d distributed process-group subsystem. Representative routines include `getDurationFromEvent`, `recordWithResetEnabled`, `getEntry`, `update_state`.
- 用途 (CN): 该文件在c10d 分布式进程组子系统中提供flight recorder detail 的接口与类型声明。 代表性例程包括 `getDurationFromEvent`、`recordWithResetEnabled`、`getEntry`、`update_state`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

```cpp
1: #include <nlohmann/json.hpp>
2: 
3: #include <c10/util/WaitCounter.h>
4: #include <c10/util/thread_name.h>
5: 
6: #include <torch/csrc/distributed/c10d/FlightRecorder.hpp>
7: 
8: namespace c10d {
9: 
10: template <typename EventType>
11: float getDurationFromEvent(EventType& start, EventType& end);
12: 
13: // Returns the traceback of current entry, in string form.
14: // Note: `getTraceback` invokes `torch::symbolize`, which may need to acquire
15: // the GIL. If you don't want to block the current thread or take the risk of a
16: // GIL deadlock, you can use an asynchronous calling mechanism like std::async.
17: template <typename EventType>
18: std::string FlightRecorder<EventType>::Entry::getTraceback() {
19:   torch::CapturedTraceback* traceback = traceback_.get();
20:   torch::SymbolizedTracebacks s_tbs = torch::symbolize({traceback});
```

- EN: Lines 1-20 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `getDurationFromEvent`.
- CN: 第 1-20 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `getDurationFromEvent` 等例程中引入具体执行逻辑。

### Lines 21-40 / 第 21-40 行

```cpp
21:   // We use 0 because we only have one traceback here.
22:   const auto& s_tb = s_tbs.tracebacks.at(0);
23:   std::stringstream oss;
24:   for (auto idx : c10::irange(s_tb.size())) {
25:     auto frame_id = s_tb[idx];
26:     const auto& frame = s_tbs.all_frames.at(frame_id);
27:     oss << '#' << idx << ' ' << frame.funcname << " from " << frame.filename
28:         << ':' << frame.lineno << '\n';
29:   }
30:   /* Resulted format is like:
31:     #0 all_reduce from pytorch/torch/distributed/distributed_c10d.py:2696
32:     #1 wrapper from pytorch/torch/distributed/c10d_logger.py:83
33:     #2 bar from /home/user/repro.py:15
34:     #3 foo from /home/user/repro.py:24
35:     #4 main from /home/user/repro.py:34
36:     #5 <module> from /home/user/repro.py:40
37:   */
38:   return oss.str();
39: }
40: 
```

- EN: Lines 21-40 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 21-40 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 41-60 / 第 41-60 行

```cpp
41: template <typename EventType>
42: std::optional<size_t> FlightRecorder<EventType>::record(
43:     size_t pg_id,
44:     const std::tuple<std::string, std::string>& pg_name,
45:     size_t collective_seq_id,
46:     size_t p2p_seq_id,
47:     size_t op_id,
48:     std::string profiling_name,
49:     const std::vector<at::Tensor>& inputs,
50:     const std::vector<at::Tensor>& outputs,
51:     EventType* start,
52:     EventType* end,
53:     std::chrono::milliseconds timeout_ms,
54:     std::shared_ptr<ProcessGroupStatus> pg_status,
55:     bool isP2P) {
56:   auto result = recordWithResetEnabled(
57:       pg_id,
58:       pg_name,
59:       collective_seq_id,
60:       p2p_seq_id,
```

- EN: Lines 41-60 continues the local implementation details and data flow for this file.
- CN: 第 41-60 行继续展开本文件的局部实现细节与数据流。

### Lines 61-80 / 第 61-80 行

```cpp
61:       op_id,
62:       std::move(profiling_name),
63:       inputs,
64:       outputs,
65:       start,
66:       end,
67:       timeout_ms,
68:       std::move(pg_status),
69:       isP2P);
70:   return result.id;
71: }
72: 
73: template <typename EventType>
74: typename FlightRecorder<EventType>::TraceIdentifier FlightRecorder<EventType>::
75:     recordWithResetEnabled(
76:         size_t pg_id,
77:         const std::tuple<std::string, std::string>& pg_name,
78:         size_t collective_seq_id,
79:         size_t p2p_seq_id,
80:         size_t op_id,
```

- EN: Lines 61-80 returns computed state or forwards results to the surrounding caller.
- CN: 第 61-80 行返回计算结果，或将结果继续传递给外围调用方。

### Lines 81-100 / 第 81-100 行

```cpp
81:         std::string profiling_name,
82:         const std::vector<at::Tensor>& inputs,
83:         const std::vector<at::Tensor>& outputs,
84:         EventType* start,
85:         EventType* end,
86:         std::chrono::milliseconds timeout_ms,
87:         std::shared_ptr<ProcessGroupStatus> pg_status,
88:         bool isP2P) {
89:   if (!enabled_) {
90:     return TraceIdentifier{std::nullopt, std::nullopt};
91:   }
92:   auto traceback =
93:       torch::CapturedTraceback::gather(true, true, capture_cpp_stack_);
94:   std::lock_guard<std::mutex> guard(mutex_);
95:   if (all_pg_status_.find(pg_id) == all_pg_status_.end()) {
96:     // Current pg_status is not in FR.
97:     all_pg_status_[pg_id] = std::move(pg_status);
98:   }
99: 
100:   TORCH_CHECK(
```

- EN: Lines 81-100 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 81-100 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 101-120 / 第 101-120 行

```cpp
101:       reset_epoch_start_idx_.find(reset_epoch_) !=
102:       reset_epoch_start_idx_.end());
103: 
104:   auto te = Entry{
105:       id_,
106:       reset_epoch_,
107:       pg_id,
108:       pg_name,
109:       collective_seq_id,
110:       p2p_seq_id,
111:       op_id,
112:       std::move(profiling_name),
113:       std::move(traceback),
114:       start,
115:       end,
116:       c10::getTime(),
117:       timeout_ms.count(),
118:       isP2P,
119:       std::nullopt,
120:       std::nullopt,
```

- EN: Lines 101-120 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 101-120 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 121-140 / 第 121-140 行

```cpp
121:       std::nullopt,
122:       {},
123:       {},
124:       {},
125:       {},
126:       {},
127:       std::this_thread::get_id(),
128:       c10::getThreadName(),
129:       false};
130: 
131:   for (const auto& input : inputs) {
132:     c10::IntArrayRef sizes = input.sizes();
133:     te.input_dtypes_.push_back(input.dtype().toScalarType());
134:     te.input_dims_.push_back(static_cast<int64_t>(sizes.size()));
135:     te.sizes_.insert(te.sizes_.end(), sizes.begin(), sizes.end());
136:   }
137: 
138:   for (const auto& output : outputs) {
139:     c10::IntArrayRef sizes = output.sizes();
140:     te.output_dtypes_.push_back(output.dtype().toScalarType());
```

- EN: Lines 121-140 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 121-140 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 141-160 / 第 141-160 行

```cpp
141:     te.output_dims_.push_back(static_cast<int64_t>(sizes.size()));
142:     te.sizes_.insert(te.sizes_.end(), sizes.begin(), sizes.end());
143:   }
144: 
145:   const auto next = next_++;
146: 
147:   if (entries_.size() < max_entries_) {
148:     entries_.emplace_back(std::move(te));
149:   } else {
150:     entries_[next] = std::move(te);
151:   }
152: 
153:   if (next_ == max_entries_) {
154:     next_ = 0;
155:   }
156: 
157:   const auto id = id_++;
158:   return TraceIdentifier{id, reset_epoch_};
159: }
160: 
```

- EN: Lines 141-160 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 141-160 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 161-180 / 第 161-180 行

```cpp
161: template <typename EventType>
162: void FlightRecorder<EventType>::record_pg_ranks(
163:     const std::tuple<std::string, std::string>& pg_name,
164:     std::vector<uint64_t> ranks) {
165:   if (!enabled_) {
166:     return;
167:   }
168:   std::lock_guard<std::mutex> guard(mutex_);
169:   pg_name_to_ranks_[pg_name] = std::move(ranks);
170: }
171: 
172: template <typename EventType>
173: void FlightRecorder<EventType>::record_accelerator_version(
174:     const std::string comm_lib_version) {
175:   if (!enabled_) {
176:     return;
177:   }
178:   std::lock_guard<std::mutex> guard(mutex_);
179:   comm_lib_version_ = std::move(comm_lib_version);
180: }
```

- EN: Lines 161-180 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 161-180 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 181-200 / 第 181-200 行

```cpp
181: 
182: template <typename EventType>
183: void FlightRecorder<EventType>::update_state(Entry& r) {
184:   try {
185:     if (r.start_ != nullptr) {
186:       bool started = r.start_->query();
187:       if (started && !r.time_discovered_started_) {
188:         r.time_discovered_started_ = c10::getTime();
189:       }
190:     }
191:     if (r.end_ != nullptr) {
192:       bool completed = r.end_->query();
193:       if (completed && !r.time_discovered_completed_) {
194:         r.time_discovered_completed_ = c10::getTime();
195:       }
196:     }
197:   } catch (std::exception& e) {
198:     LOG(ERROR) << "Failed to update state for entry " << r.id_ << ": "
199:                << r.profiling_name_ << " with error: " << e.what();
200:   }
```

- EN: Lines 181-200 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 181-200 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 201-220 / 第 201-220 行

```cpp
201: }
202: 
203: template <typename EventType>
204: std::vector<typename FlightRecorder<EventType>::Entry> FlightRecorder<
205:     EventType>::dump_entries() {
206:   std::vector<Entry> result;
207:   {
208:     std::lock_guard<std::mutex> guard(mutex_);
209:     // Filter entries during insertion - only keep entries from current epoch
210:     auto filter = [this](const Entry& e) {
211:       return e.reset_epoch_ == reset_epoch_;
212:     };
213:     std::copy_if(
214:         entries_.begin() + static_cast<std::ptrdiff_t>(next_),
215:         entries_.end(),
216:         std::back_inserter(result),
217:         filter);
218:     std::copy_if(
219:         entries_.begin(),
220:         entries_.begin() + static_cast<std::ptrdiff_t>(next_),
```

- EN: Lines 201-220 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 201-220 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 221-240 / 第 221-240 行

```cpp
221:         std::back_inserter(result),
222:         filter);
223:   }
224:   // query any remaining events
225:   for (auto& r : result) {
226:     update_state(r);
227:     r.start_ = r.end_ = nullptr;
228:   }
229:   return result;
230: }
231: 
232: template <typename EventType>
233: // Returns the index in entries_ for the given id and reset_epoch.
234: // Caller must hold mutex_lock before calling this method.
235: size_t FlightRecorder<EventType>::getIdxFromId(size_t id, size_t reset_epoch)
236:     const {
237:   // Look up the starting idx for the given reset epoch
238:   auto it = reset_epoch_start_idx_.find(reset_epoch);
239:   TORCH_CHECK(it != reset_epoch_start_idx_.end());
240:   // Calculate idx based on where the epoch started
```

- EN: Lines 221-240 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 221-240 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 241-260 / 第 241-260 行

```cpp
241:   return (it->second + id) % max_entries_;
242: }
243: 
244: template <typename EventType>
245: // Returns the entry with the given id and reset_epoch, if it exists. Otherwise,
246: // returns std::nullopt.
247: std::optional<typename FlightRecorder<EventType>::Entry> FlightRecorder<
248:     EventType>::
249:     getEntry(std::optional<size_t> id, std::optional<size_t> reset_epoch) {
250:   if (!enabled_ || !id || !reset_epoch) {
251:     return std::nullopt;
252:   }
253: 
254:   std::unique_lock<std::mutex> guard(mutex_);
255:   Entry entry = entries_.at(getIdxFromId(*id, *reset_epoch));
256:   if (entry.id_ == *id && entry.reset_epoch_ == *reset_epoch) {
257:     return entry;
258:   }
259:   return std::nullopt;
260: }
```

- EN: Lines 241-260 introduces executable logic in routines such as `getEntry`; returns computed state or forwards results to the surrounding caller.
- CN: 第 241-260 行在 `getEntry` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 261-280 / 第 261-280 行

```cpp
261: 
262: template <typename EventType>
263: std::optional<typename FlightRecorder<EventType>::Entry> FlightRecorder<
264:     EventType>::getEntry(std::optional<size_t> id) {
265:   return getEntry(id, 0);
266: }
267: 
268: template <typename EventType>
269: void FlightRecorder<EventType>::retire_id(
270:     std::optional<size_t> id,
271:     std::optional<size_t> reset_epoch,
272:     bool compute_duration) {
273:   if (!enabled_ || !id || !reset_epoch) {
274:     return;
275:   }
276: 
277:   bool can_compute_duration = false;
278:   EventType* startEvent = nullptr;
279:   EventType* endEvent = nullptr;
280:   std::optional<float> duration = std::nullopt;
```

- EN: Lines 261-280 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 261-280 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 281-300 / 第 281-300 行

```cpp
281: 
282:   std::unique_lock<std::mutex> guard(mutex_);
283: 
284:   Entry* entry = &entries_.at(getIdxFromId(*id, *reset_epoch));
285:   if (entry->id_ == *id && entry->reset_epoch_ == *reset_epoch) {
286:     update_state(*entry);
287: 
288:     if (compute_duration) {
289:       can_compute_duration = entry->time_discovered_completed_.has_value() &&
290:           entry->start_ && entry->end_;
291:       startEvent = entry->start_;
292:       endEvent = entry->end_;
293:     }
294:     entry->retired_ = true;
295:     entry->start_ = entry->end_ = nullptr;
296:   }
297: 
298:   if (can_compute_duration) {
299:     // Compute duration without without holding the lock, because
300:     // cudaEventDuration() can hang, and we need to acquire the lock before we
```

- EN: Lines 281-300 introduces executable logic in routines such as `update_state`.
- CN: 第 281-300 行在 `update_state` 等例程中引入具体执行逻辑。

### Lines 301-320 / 第 301-320 行

```cpp
301:     // can dump(), which we never want to block.
302:     guard.unlock();
303:     duration = getDurationFromEvent<EventType>(*startEvent, *endEvent);
304:     guard.lock();
305: 
306:     // Refresh the entry pointer, see if the entry has been overwritten
307:     entry = &entries_.at(getIdxFromId(*id, *reset_epoch));
308:     if (!(entry->id_ == *id && entry->reset_epoch_ == *reset_epoch)) {
309:       LOG(INFO) << "retire_id abandoned for id " << *id
310:                 << ", event was overwritten while waiting to compute duration.";
311:       return;
312:     }
313:     if (duration.has_value()) {
314:       entry->duration_ = duration;
315:     }
316:   }
317: }
318: 
319: template <typename EventType>
320: void FlightRecorder<EventType>::retire_id(
```

- EN: Lines 301-320 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 301-320 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 321-340 / 第 321-340 行

```cpp
321:     std::optional<size_t> id,
322:     bool compute_duration) {
323:   retire_id(id, 0, compute_duration);
324: }
325: 
326: template <typename EventType>
327: void FlightRecorder<EventType>::reset_all() {
328:   std::lock_guard<std::mutex> guard(mutex_);
329:   if (!entries_.empty()) {
330:     // Soft delete: increment epoch to mark all existing entries as old
331:     // Store where the new epoch starts in the circular buffer
332:     reset_epoch_++;
333:     reset_epoch_start_idx_[reset_epoch_] = next_;
334:     id_ = 0;
335:   }
336: }
337: 
338: template <typename EventType>
339: const c10::List<c10::IValue> FlightRecorder<EventType>::getCollectiveTrace(
340:     bool includeStacktraces,
```

- EN: Lines 321-340 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 321-340 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 341-360 / 第 341-360 行

```cpp
341:     bool onlyActive) {
342:   auto entries = new_list();
343:   // Entries are returned in the order they were recorded
344:   auto result = dump_entries();
345:   std::vector<torch::CapturedTraceback*> tracebacks;
346:   torch::SymbolizedTracebacks stracebacks;
347:   std::vector<c10::IValue> all_frames;
348:   if (includeStacktraces) {
349:     for (auto& e : result) {
350:       tracebacks.push_back(e.traceback_.get());
351:     }
352:     stracebacks = torch::symbolize(tracebacks);
353:     for (const auto& f : stracebacks.all_frames) {
354:       auto d = new_dict();
355:       d.insert(name_key, f.funcname);
356:       d.insert(filename_key, f.filename);
357:       d.insert(line_key, int64_t(f.lineno));
358:       all_frames.emplace_back(std::move(d));
359:     }
360:   }
```

- EN: Lines 341-360 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 341-360 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 361-380 / 第 361-380 行

```cpp
361:   for (auto i : c10::irange(result.size())) {
362:     auto dict = new_dict();
363:     auto& e = result.at(i);
364:     // Skip completed events
365:     if (onlyActive && e.time_discovered_completed_.has_value()) {
366:       continue;
367:     }
368:     if (includeStacktraces) {
369:       auto& tb = stracebacks.tracebacks.at(i);
370:       auto frames = new_list();
371:       for (auto frame : tb) {
372:         frames.push_back(all_frames.at(frame));
373:       }
374:       dict.insert(frames_key, frames);
375:     }
376: 
377:     dict.insert(record_id_key, int64_t(e.id_));
378:     dict.insert(pg_id_key, int64_t(e.pg_id_));
379:     dict.insert(pg_name_key, e.pg_name_);
380:     dict.insert(thread_name_key, e.thread_name_);
```

- EN: Lines 361-380 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 361-380 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 381-400 / 第 381-400 行

```cpp
381:     dict.insert(thread_id_key, c10::str(e.thread_id_));
382:     dict.insert(collective_seq_id_key, int64_t(e.collective_seq_id_));
383:     dict.insert(p2p_seq_id_key, int64_t(e.p2p_seq_id_));
384:     dict.insert(op_id_key, int64_t(e.op_id_));
385:     dict.insert(profiling_name_key, e.profiling_name_);
386:     dict.insert(time_created_key, int64_t(e.time_created_));
387:     if (e.duration_) {
388:       dict.insert(duration_key, *e.duration_);
389:     }
390: 
391:     auto it = e.sizes_.begin();
392:     auto read_sizes = [&](const c10::SmallVector<int64_t, 4>& dims) {
393:       auto sizes = new_list();
394:       for (auto dim : dims) {
395:         auto arg_sizes = new_list();
396:         for ([[maybe_unused]] auto i : c10::irange(dim)) {
397:           arg_sizes.push_back(*it++);
398:         }
399:         sizes.push_back(arg_sizes);
400:       }
```

- EN: Lines 381-400 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 381-400 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 401-420 / 第 401-420 行

```cpp
401:       return sizes;
402:     };
403: 
404:     dict.insert(input_sizes_key, read_sizes(e.input_dims_));
405:     std::vector<std::string> input_dtypes_strs;
406:     input_dtypes_strs.reserve(e.input_dtypes_.size());
407:     for (const auto& input_dtype : e.input_dtypes_) {
408:       input_dtypes_strs.emplace_back(c10::toString(input_dtype));
409:     }
410:     dict.insert(input_dtypes_key, input_dtypes_strs);
411:     dict.insert(output_sizes_key, read_sizes(e.output_dims_));
412:     std::vector<std::string> output_dtypes_strs;
413:     output_dtypes_strs.reserve(e.output_dtypes_.size());
414:     for (const auto& output_dtype : e.output_dtypes_) {
415:       output_dtypes_strs.emplace_back(c10::toString(output_dtype));
416:     }
417:     dict.insert(output_dtypes_key, output_dtypes_strs);
418:     if (e.time_discovered_completed_.has_value()) {
419:       dict.insert(state_key, completed_state);
420:     } else if (e.time_discovered_started_.has_value()) {
```

- EN: Lines 401-420 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 401-420 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 421-440 / 第 421-440 行

```cpp
421:       dict.insert(state_key, started_state);
422:     } else {
423:       dict.insert(state_key, scheduled_state);
424:     }
425: 
426:     dict.insert(
427:         time_discovered_started_key,
428:         e.time_discovered_started_.has_value()
429:             ? int64_t(*e.time_discovered_started_)
430:             : c10::IValue());
431:     dict.insert(
432:         time_discovered_completed_key,
433:         e.time_discovered_completed_.has_value()
434:             ? int64_t(*e.time_discovered_completed_)
435:             : c10::IValue());
436:     dict.insert(retired_key, e.retired_);
437:     dict.insert(timeout_key, e.timeout_ms_);
438:     dict.insert(is_p2p_key, e.isP2P_);
439: 
440:     entries.push_back(dict);
```

- EN: Lines 421-440 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 421-440 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 441-460 / 第 441-460 行

```cpp
441:   }
442:   return entries;
443: }
444: 
445: template <typename EventType>
446: const c10::Dict<c10::IValue, c10::IValue> FlightRecorder<
447:     EventType>::getPgConfig() {
448:   auto pg_config = new_dict();
449:   for (const auto& [pg_name, ranks] : pg_name_to_ranks_) {
450:     auto pg_info = new_dict();
451:     pg_info.insert("name", std::get<0>(pg_name));
452:     pg_info.insert("desc", std::get<1>(pg_name));
453:     pg_info.insert("ranks", ranks_str(ranks));
454:     pg_config.insert(std::get<0>(pg_name), pg_info);
455:   }
456:   return pg_config;
457: }
458: 
459: template <typename EventType>
460: const std::map<std::string, std::map<std::string, std::string>> FlightRecorder<
```

- EN: Lines 441-460 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 441-460 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 461-480 / 第 461-480 行

```cpp
461:     EventType>::getPgConfigJson() {
462:   std::map<std::string, std::map<std::string, std::string>> result;
463:   for (const auto& [pg_name, ranks] : pg_name_to_ranks_) {
464:     auto pg_info = std::map<std::string, std::string>();
465:     pg_info["name"] = std::get<0>(pg_name);
466:     pg_info["desc"] = std::get<1>(pg_name);
467:     pg_info["ranks"] = ranks_str(ranks);
468:     result.emplace(std::get<0>(pg_name), pg_info);
469:   }
470:   return result;
471: }
472: 
473: template <typename EventType>
474: const c10::Dict<c10::IValue, c10::IValue> FlightRecorder<
475:     EventType>::getPgStatus() {
476:   auto all_pg_status = new_dict();
477:   for (const auto& [pg_id, status] : all_pg_status_) {
478:     auto pg_status = new_dict();
479:     pg_status.insert("last_enqueued_collective", status->lastEnqueuedSeq);
480:     pg_status.insert("last_started_collective", status->lastStartedSeq);
```

- EN: Lines 461-480 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 461-480 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 481-500 / 第 481-500 行

```cpp
481:     pg_status.insert("last_completed_collective", status->lastCompletedSeq);
482:     all_pg_status.insert(std::to_string(pg_id), pg_status);
483:   }
484:   return all_pg_status;
485: }
486: 
487: template <typename EventType>
488: const std::map<std::string, std::map<std::string, std::string>> FlightRecorder<
489:     EventType>::getPgStatusJson() {
490:   std::map<std::string, std::map<std::string, std::string>> result;
491:   for (const auto& [pg_id, status] : all_pg_status_) {
492:     auto pg_status = std::map<std::string, std::string>();
493:     pg_status["last_enqueued_collective"] =
494:         std::to_string(status->lastEnqueuedSeq);
495:     pg_status["last_started_collective"] =
496:         std::to_string(status->lastStartedSeq);
497:     pg_status["last_completed_collective"] =
498:         std::to_string(status->lastCompletedSeq);
499:     result[std::to_string(pg_id)] = pg_status;
500:   }
```

- EN: Lines 481-500 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 481-500 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 501-520 / 第 501-520 行

```cpp
501:   return result;
502: }
503: 
504: using json = nlohmann::json;
505: template <typename EventType>
506: std::string FlightRecorder<EventType>::dump_json(
507:     const std::optional<std::unordered_map<
508:         std::string,
509:         std::unordered_map<std::string, std::string>>>& extraDumpMap,
510:     bool includeCollectives,
511:     bool onlyActive) {
512:   json result;
513:   result[version_key_str] = version_val_str;
514:   result[comm_lib_version_key_str] = comm_lib_version_;
515:   result[pg_config_key_str] = getPgConfigJson();
516:   result[pg_status_key_str] = getPgStatusJson();
517: 
518:   // collective trace
519:   if (includeCollectives) {
520:     std::list<json> entries;
```

- EN: Lines 501-520 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 501-520 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 521-540 / 第 521-540 行

```cpp
521:     for (auto& e : dump_entries()) {
522:       json j;
523:       if (onlyActive && e.time_discovered_completed_.has_value()) {
524:         continue;
525:       }
526:       j[record_id_key_str] = int64_t(e.id_);
527:       j[pg_id_key_str] = int64_t(e.pg_id_);
528:       j[pg_name_key_str] = e.pg_name_;
529:       j[thread_name_key_str] = e.thread_name_;
530:       j[thread_id_key_str] = c10::str(e.thread_id_);
531:       j[collective_seq_id_key_str] = int64_t(e.collective_seq_id_);
532:       j[p2p_seq_id_key_str] = int64_t(e.p2p_seq_id_);
533:       j[op_id_key_str] = int64_t(e.op_id_);
534:       j[profiling_name_key_str] = e.profiling_name_;
535:       j[time_created_key_str] = int64_t(e.time_created_);
536:       if (e.duration_) {
537:         j[duration_key_str] = *e.duration_;
538:       }
539:       auto it = e.sizes_.begin();
540:       auto read_sizes = [&](const c10::SmallVector<int64_t, 4>& dims) {
```

- EN: Lines 521-540 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 521-540 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 541-560 / 第 541-560 行

```cpp
541:         auto sizes = std::list<std::list<int64_t>>();
542:         for (auto dim : dims) {
543:           auto arg_sizes = std::list<int64_t>();
544:           for (auto i : c10::irange(dim)) {
545:             (void)i;
546:             arg_sizes.push_back(*it++);
547:           }
548:           sizes.push_back(arg_sizes);
549:         }
550:         return sizes;
551:       };
552:       j[input_sizes_key_str] = read_sizes(e.input_dims_);
553:       std::vector<std::string> input_dtypes_strs;
554:       input_dtypes_strs.reserve(e.input_dtypes_.size());
555:       for (const auto& input_dtype : e.input_dtypes_) {
556:         input_dtypes_strs.emplace_back(c10::toString(input_dtype));
557:       }
558:       j[input_dtypes_key_str] = input_dtypes_strs;
559:       j[output_sizes_key_str] = read_sizes(e.output_dims_);
560:       std::vector<std::string> output_dtypes_strs;
```

- EN: Lines 541-560 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 541-560 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 561-580 / 第 561-580 行

```cpp
561:       output_dtypes_strs.reserve(e.output_dtypes_.size());
562:       for (const auto& output_dtype : e.output_dtypes_) {
563:         output_dtypes_strs.emplace_back(c10::toString(output_dtype));
564:       }
565:       j[output_dtypes_key_str] = output_dtypes_strs;
566:       if (e.time_discovered_completed_.has_value()) {
567:         j[state_key_str] = completed_state_str;
568:       } else if (e.time_discovered_started_.has_value()) {
569:         j[state_key_str] = started_state_str;
570:       } else {
571:         j[state_key_str] = scheduled_state_str;
572:       }
573:       j[time_discovered_started_key_str] =
574:           e.time_discovered_started_.has_value()
575:           ? int64_t(*e.time_discovered_started_)
576:           : 0;
577:       j[time_discovered_completed_key_str] =
578:           e.time_discovered_completed_.has_value()
579:           ? int64_t(*e.time_discovered_completed_)
580:           : 0;
```

- EN: Lines 561-580 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 561-580 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 581-600 / 第 581-600 行

```cpp
581:       j[retired_key_str] = e.retired_;
582:       j[timeout_key_str] = e.timeout_ms_;
583:       j[is_p2p_key_str] = e.isP2P_;
584:       entries.emplace_back(j);
585:     }
586: 
587:     if (!entries.empty()) {
588:       result[entries_key_str] = entries;
589:     }
590:   }
591: 
592:   if (extraDumpMap.has_value()) {
593:     result[nccl_comm_key_str] = extraDumpMap.value();
594:   }
595:   return result.dump();
596: }
597: 
598: template <typename EventType>
599: std::string FlightRecorder<EventType>::dump(
600:     const std::optional<std::unordered_map<
```

- EN: Lines 581-600 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 581-600 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 601-620 / 第 601-620 行

```cpp
601:         std::string,
602:         std::unordered_map<std::string, std::string>>>& extraDumpMap,
603:     bool includeCollectives,
604:     bool includeStackTraces,
605:     bool onlyActive) {
606:   STATIC_SCOPED_WAIT_COUNTER(pytorch.wait_counter.FlightRecorder__dump);
607:   auto result = new_dict();
608:   // common values
609:   result.insert(version_key, version_val);
610:   result.insert(pg_config_key, getPgConfig());
611:   result.insert(comm_lib_version_key_str, comm_lib_version_);
612:   result.insert(pg_status_key, getPgStatus());
613: 
614:   // collective trace
615:   if (includeCollectives) {
616:     result.insert(
617:         entries_key, getCollectiveTrace(includeStackTraces, onlyActive));
618:   }
619: 
620:   // convert extraDumpMap into a dictionary
```

- EN: Lines 601-620 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 601-620 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 621-636 / 第 621-636 行

```cpp
621:   auto per_comm_dict = new_dict();
622:   if (extraDumpMap.has_value()) {
623:     for (const auto& [ncclId, ncclDump] : extraDumpMap.value()) {
624:       auto inner_dict = new_dict();
625:       for (const auto& [key, value] : ncclDump) {
626:         inner_dict.insert(key, value);
627:       }
628:       per_comm_dict.insert(ncclId, inner_dict);
629:     }
630:   }
631:   if (!per_comm_dict.empty()) {
632:     result.insert(nccl_comm_key, per_comm_dict);
633:   }
634:   return pickle_str(result);
635: }
636: } // namespace c10d
```

- EN: Lines 621-636 opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 621-636 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

## Key Concepts / 关键概念

- EN: Subsystem: c10d distributed process-group subsystem.
- CN: 子系统：c10d 分布式进程组子系统。
- EN: File role: interface/definition.
- CN: 文件角色：接口/定义。
- EN: Primary symbols: `getDurationFromEvent`, `recordWithResetEnabled`, `getEntry`, `update_state`
- CN: 核心符号：`getDurationFromEvent`、`recordWithResetEnabled`、`getEntry`、`update_state`
- EN: Notable themes: process-group orchestration, store/state coordination, collective communication logic.
- CN: 值得关注的主题：进程组编排、存储/状态协调、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/FlightRecorder.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `c10/util/WaitCounter.h`, `c10/util/thread_name.h`
- External or system headers / 外部或系统头文件: `nlohmann/json.hpp`
- Local symbols / 本地符号: `getDurationFromEvent`, `recordWithResetEnabled`, `getEntry`, `update_state`