# FlightRecorder.hpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/FlightRecorder.hpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides interface and type declarations for flight recorder in the c10d distributed process-group subsystem. Key types include `TORCH_API`, `FlightRecorder`, `Entry`.
- 用途 (CN): 该文件在c10d 分布式进程组子系统中提供flight recorder 的接口与类型声明。 关键类型包括 `TORCH_API`、`FlightRecorder`、`Entry`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
1: #pragma once
2: #include <cstdio>
3: #include <cstdlib>
4: 
5: #include <memory>
6: #include <mutex>
7: 
8: #include <ATen/ATen.h>
9: #include <c10/util/Exception.h>
10: #include <torch/csrc/distributed/c10d/TraceUtils.h>
11: #include <torch/csrc/distributed/c10d/logger.hpp>
12: #include <optional>
13: 
14: namespace c10d {
15: 
16: #define DEFINE_CONSTANT(name, value) \
```

- EN: Lines 1-16 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 1-16 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 17-32 / 第 17-32 行

```cpp
17:   static c10::IValue name = value;   \
18:   static std::string name##_str = value;
19: // Update whenever changing contents or formatting of the dump
20: // (minor when adding fields, major when changing existing fields)
21: // Also update both JSON and Pickle dumps to make use of the newly defined
22: // field(s).
23: DEFINE_CONSTANT(version_val, "2.10")
24: DEFINE_CONSTANT(entries_key, "entries")
25: DEFINE_CONSTANT(nccl_comm_key, "nccl_comm_state")
26: DEFINE_CONSTANT(comm_lib_version_key, "comm_lib_version")
27: DEFINE_CONSTANT(version_key, "version")
28: DEFINE_CONSTANT(pg_config_key, "pg_config")
29: DEFINE_CONSTANT(pg_status_key, "pg_status")
30: DEFINE_CONSTANT(record_id_key, "record_id")
31: DEFINE_CONSTANT(pg_id_key, "pg_id")
32: DEFINE_CONSTANT(pg_name_key, "process_group")
```

- EN: Lines 17-32 continues the local implementation details and data flow for this file.
- CN: 第 17-32 行继续展开本文件的局部实现细节与数据流。

### Lines 33-48 / 第 33-48 行

```cpp
33: DEFINE_CONSTANT(collective_seq_id_key, "collective_seq_id")
34: DEFINE_CONSTANT(p2p_seq_id_key, "p2p_seq_id")
35: DEFINE_CONSTANT(is_p2p_key, "is_p2p")
36: DEFINE_CONSTANT(op_id_key, "op_id")
37: DEFINE_CONSTANT(profiling_name_key, "profiling_name")
38: DEFINE_CONSTANT(input_sizes_key, "input_sizes")
39: DEFINE_CONSTANT(input_dtypes_key, "input_dtypes")
40: DEFINE_CONSTANT(output_sizes_key, "output_sizes")
41: DEFINE_CONSTANT(output_dtypes_key, "output_dtypes")
42: DEFINE_CONSTANT(time_created_key, "time_created_ns")
43: DEFINE_CONSTANT(duration_key, "duration_ms")
44: DEFINE_CONSTANT(timeout_key, "timeout_ms")
45: DEFINE_CONSTANT(frames_key, "frames")
46: DEFINE_CONSTANT(state_key, "state")
47: DEFINE_CONSTANT(line_key, "line")
48: DEFINE_CONSTANT(name_key, "name")
```

- EN: Lines 33-48 continues the local implementation details and data flow for this file.
- CN: 第 33-48 行继续展开本文件的局部实现细节与数据流。

### Lines 49-64 / 第 49-64 行

```cpp
49: DEFINE_CONSTANT(filename_key, "filename")
50: DEFINE_CONSTANT(retired_key, "retired")
51: DEFINE_CONSTANT(time_discovered_started_key, "time_discovered_started_ns")
52: DEFINE_CONSTANT(time_discovered_completed_key, "time_discovered_completed_ns")
53: DEFINE_CONSTANT(completed_state, "completed")
54: DEFINE_CONSTANT(scheduled_state, "scheduled")
55: DEFINE_CONSTANT(started_state, "started")
56: DEFINE_CONSTANT(thread_id_key, "thread_id")
57: DEFINE_CONSTANT(thread_name_key, "thread_name")
58: #undef DEFINE_CONSTANT
59: 
60: // Write NCCL debug info to local disk or any storage users define.
61: // There are some constrains we set for the debug info writer:
62: // 1. The writer should only be registered once.
63: // 2. Once registered, users cannot change it including un-register.
64: // 3. It is recommended to register the customized writer in the trainer setup,
```

- EN: Lines 49-64 continues the local implementation details and data flow for this file.
- CN: 第 49-64 行继续展开本文件的局部实现细节与数据流。

### Lines 65-80 / 第 65-80 行

```cpp
65: //    If users don't register before calling launchAsyncDebugDump, then users
66: //    lose the chance to register (and the default writer will be
67: //    auto-registered).
68: class TORCH_API DebugInfoWriter {
69:  public:
70:   virtual ~DebugInfoWriter() = default;
71:   virtual void write(const std::string& trace);
72:   static DebugInfoWriter& getWriter(int rank);
73:   static void registerWriter(std::unique_ptr<DebugInfoWriter> writer);
74:   virtual std::string getWriterTarget() {
75:     return filename_;
76:   }
77: 
78:  protected:
79:   DebugInfoWriter(
80:       const std::string& namePrefix,
```

- EN: Lines 65-80 declares or defines types such as `TORCH_API`; introduces executable logic in routines such as `write`, `getWriter`, `registerWriter`.
- CN: 第 65-80 行声明或定义了 `TORCH_API` 等类型；在 `write`、`getWriter`、`registerWriter` 等例程中引入具体执行逻辑。

### Lines 81-96 / 第 81-96 行

```cpp
81:       int rank,
82:       bool enableDynamicFilename = false) {
83:     filename_ = c10::str(namePrefix, rank);
84:     enable_dynamic_filename_ = enableDynamicFilename;
85:     rank_ = rank;
86:   }
87:   std::string filename_;
88:   int rank_;
89:   bool enable_dynamic_filename_;
90: 
91:  private:
92:   static std::unique_ptr<DebugInfoWriter> writer_;
93:   static std::atomic<bool> hasWriterRegistered_;
94: };
95: 
96: template <typename EventType>
```

- EN: Lines 81-96 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 81-96 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 97-112 / 第 97-112 行

```cpp
97: struct FlightRecorder {
98:   static FlightRecorder<EventType>* get() {
99:     // intentionally leak on exit
100:     // because this will hold python state that may get destructed
101:     static FlightRecorder<EventType>* instance =
102:         new FlightRecorder<EventType>();
103:     return instance;
104:   }
105:   FlightRecorder() {
106:     // NOTE: This default value (2000) is duplicated in ProcessGroupNCCL.cpp
107:     // and ProcessGroupNCCL.hpp because they cannot directly query max_entries_
108:     // (no public accessor). Keep these values in sync.
109:     max_entries_ = getCvarInt(
110:         {"TORCH_FR_BUFFER_SIZE", "TORCH_NCCL_TRACE_BUFFER_SIZE"}, 2000);
111:     capture_cpp_stack_ = getCvarBool(
112:         {"TORCH_FR_CPP_STACK", "TORCH_NCCL_TRACE_CPP_STACK"}, false);
```

- EN: Lines 97-112 declares or defines types such as `FlightRecorder`; introduces executable logic in routines such as `get`, `FlightRecorder`.
- CN: 第 97-112 行声明或定义了 `FlightRecorder` 等类型；在 `get`、`FlightRecorder` 等例程中引入具体执行逻辑。

### Lines 113-128 / 第 113-128 行

```cpp
113:     enabled_ = max_entries_ > 0;
114:     reset_epoch_start_idx_[0] = 0;
115:   }
116:   struct Entry {
117:     size_t id_; // incremented id in the trace buffer
118:                 // used to figure out where in the circular entries
119:                 // buffer this entry will be located to
120:                 // update state information
121:     size_t reset_epoch_; // epoch when this entry was created
122:     size_t pg_id_;
123:     std::tuple<std::string, std::string> pg_name_; // <group_name, group_desc>
124: 
125:     // collective_seq_id and p2p_seq_id refer to actual kernel launches (e.g. 1
126:     // per coalesced group).
127:     // collective_seq_id only increments for true collective operations (over
128:     // all ranks in the group). p2p_seq_id only increments over non-collective
```

- EN: Lines 113-128 declares or defines types such as `Entry`.
- CN: 第 113-128 行声明或定义了 `Entry` 等类型。

### Lines 129-144 / 第 129-144 行

```cpp
129:     // operations in the group. op_id refers to logical operations (e.g. one per
130:     // op inside coalesced group)
131:     size_t collective_seq_id_;
132:     size_t p2p_seq_id_;
133:     size_t op_id_;
134:     std::string profiling_name_;
135: 
136:     std::shared_ptr<torch::CapturedTraceback> traceback_;
137:     // we borrow pointers to start_ and end_ so we can query the state
138:     // on reporting. However, once the event is completed, the call
139:     // to `complete` will clear these.
140:     EventType *start_, *end_;
141: 
142:     // timestamp when the entry was created, likely close to the time the work
143:     // was 'enqueued'- not necessarily started
144:     c10::time_t time_created_;
```

- EN: Lines 129-144 continues the local implementation details and data flow for this file.
- CN: 第 129-144 行继续展开本文件的局部实现细节与数据流。

### Lines 145-160 / 第 145-160 行

```cpp
145: 
146:     // configured timeout for this entry
147:     c10::time_t timeout_ms_;
148: 
149:     // Is this a P2P event?
150:     bool isP2P_;
151: 
152:     std::optional<float> duration_;
153: 
154:     // timestamp when our CPU threads discovered that the kernel started.
155:     // will always be _after_ it actually started, and can be very late
156:     // if the watchdog thread got stuck on CUDA APIs.
157:     std::optional<c10::time_t> time_discovered_started_;
158: 
159:     // timestamp when our CPU threads discovered that the kernel completed.
160:     // will always be _after_ it actually completed, and can be the same time
```

- EN: Lines 145-160 continues the local implementation details and data flow for this file.
- CN: 第 145-160 行继续展开本文件的局部实现细节与数据流。

### Lines 161-176 / 第 161-176 行

```cpp
161:     // as the discovery of the start if the watchdog thread is stuck on CUDA
162:     // APIs
163:     std::optional<c10::time_t> time_discovered_completed_;
164: 
165:     // size information for input/output tensors
166:     c10::SmallVector<int64_t, 4> input_dims_;
167:     std::vector<c10::ScalarType> input_dtypes_;
168:     c10::SmallVector<int64_t, 4> output_dims_;
169:     std::vector<c10::ScalarType> output_dtypes_;
170:     c10::SmallVector<int64_t, 8> sizes_; // flattened from inputs, outputs
171:     std::thread::id thread_id_;
172:     std::string thread_name_;
173:     bool retired_ = false; // is this work entry no longer in the workMetaList_?
174:                            // a retired but not completed event has timed out
175: 
176:     // Returns the traceback of current entry, in string form.
```

- EN: Lines 161-176 continues the local implementation details and data flow for this file.
- CN: 第 161-176 行继续展开本文件的局部实现细节与数据流。

### Lines 177-192 / 第 177-192 行

```cpp
177:     // Note: `getTraceback` invokes `torch::symbolize`, which may need to
178:     // acquire the GIL. If you don't want to block the current thread or take
179:     // the risk of a GIL deadlock, you can use an asynchronous calling mechanism
180:     // like std::async.
181:     TORCH_API std::string getTraceback();
182:   };
183: 
184:   bool enabled_ = false;
185:   bool capture_cpp_stack_ = false;
186:   std::mutex mutex_;
187:   std::vector<Entry> entries_;
188:   size_t max_entries_ = 0;
189:   size_t next_ = 0;
190:   size_t id_ = 0;
191:   size_t reset_epoch_ = 0;
192:   std::unordered_map<size_t, size_t>
```

- EN: Lines 177-192 introduces executable logic in routines such as `getTraceback`.
- CN: 第 177-192 行在 `getTraceback` 等例程中引入具体执行逻辑。

### Lines 193-208 / 第 193-208 行

```cpp
193:       reset_epoch_start_idx_; // maps reset_epoch to the idx where it starts
194:   std::map<size_t, std::shared_ptr<ProcessGroupStatus>> all_pg_status_;
195:   std::map<std::tuple<std::string, std::string>, std::vector<uint64_t>>
196:       pg_name_to_ranks_;
197:   std::string comm_lib_version_;
198: 
199:   struct TraceIdentifier {
200:     std::optional<size_t> id;
201:     std::optional<size_t> reset_epoch;
202:   };
203: 
204:   TraceIdentifier recordWithResetEnabled(
205:       size_t pg_id,
206:       const std::tuple<std::string, std::string>& pg_name,
207:       size_t collective_seq_id,
208:       size_t p2p_seq_id,
```

- EN: Lines 193-208 declares or defines types such as `TraceIdentifier`.
- CN: 第 193-208 行声明或定义了 `TraceIdentifier` 等类型。

### Lines 209-224 / 第 209-224 行

```cpp
209:       size_t op_id,
210:       std::string profiling_name,
211:       const std::vector<at::Tensor>& inputs,
212:       const std::vector<at::Tensor>& outputs,
213:       EventType* start,
214:       EventType* end,
215:       std::chrono::milliseconds timeout_ms,
216:       std::shared_ptr<ProcessGroupStatus> pg_status,
217:       bool isP2P);
218: 
219:   std::optional<size_t> record(
220:       size_t pg_id,
221:       const std::tuple<std::string, std::string>& pg_name,
222:       size_t collective_seq_id,
223:       size_t p2p_seq_id,
224:       size_t op_id,
```

- EN: Lines 209-224 continues the local implementation details and data flow for this file.
- CN: 第 209-224 行继续展开本文件的局部实现细节与数据流。

### Lines 225-240 / 第 225-240 行

```cpp
225:       std::string profiling_name,
226:       const std::vector<at::Tensor>& inputs,
227:       const std::vector<at::Tensor>& outputs,
228:       EventType* start,
229:       EventType* end,
230:       std::chrono::milliseconds timeout_ms,
231:       std::shared_ptr<ProcessGroupStatus> pg_status,
232:       bool isP2P);
233: 
234:   TORCH_API void record_pg_ranks(
235:       const std::tuple<std::string, std::string>& pg_name,
236:       std::vector<uint64_t> ranks);
237: 
238:   void record_accelerator_version(const std::string comm_lib_version);
239: 
240:   void update_state(Entry& r);
```

- EN: Lines 225-240 introduces executable logic in routines such as `record_pg_ranks`, `record_accelerator_version`, `update_state`.
- CN: 第 225-240 行在 `record_pg_ranks`、`record_accelerator_version`、`update_state` 等例程中引入具体执行逻辑。

### Lines 241-256 / 第 241-256 行

```cpp
241: 
242:   std::vector<Entry> dump_entries();
243: 
244:   // Returns the index in entries_ for the given id and reset_epoch.
245:   // Caller must hold mutex_lock before calling this method.
246:   size_t getIdxFromId(size_t id, size_t reset_epoch) const;
247: 
248:   // Returns the entry with the given id and reset_epoch, if it exists.
249:   // Otherwise, returns std::nullopt.
250:   TORCH_API std::optional<Entry> getEntry(
251:       std::optional<size_t> id,
252:       std::optional<size_t> reset_epoch);
253: 
254:   TORCH_API std::optional<Entry> getEntry(std::optional<size_t> id);
255: 
256:   /*
```

- EN: Lines 241-256 introduces executable logic in routines such as `dump_entries`, `getIdxFromId`, `getEntry`.
- CN: 第 241-256 行在 `dump_entries`、`getIdxFromId`、`getEntry` 等例程中引入具体执行逻辑。

### Lines 257-272 / 第 257-272 行

```cpp
257:   Mark an Event as completed and free its events.
258:   This is called by the watchdog thread, and is asynchronous from the
259:   perspective of the main thread.
260:   compute_duration defaults to true since retire_id is only called in the
261:   watchdog thread, which is currently a place we call cuda APIs which may hang,
262:   but care should be taken to avoid computing duration in any function that must
263:   never hang. (timing must also be enabled for compute_duration - see
264:   TORCH_NCCL_ENABLE_TIMING).
265:   */
266:   TORCH_API void retire_id(
267:       std::optional<size_t> id,
268:       std::optional<size_t> reset_epoch,
269:       bool compute_duration = true);
270: 
271:   TORCH_API void retire_id(
272:       std::optional<size_t> id,
```

- EN: Lines 257-272 introduces executable logic in routines such as `retire_id`.
- CN: 第 257-272 行在 `retire_id` 等例程中引入具体执行逻辑。

### Lines 273-288 / 第 273-288 行

```cpp
273:       bool compute_duration = true);
274: 
275:   TORCH_API void reset_all();
276: 
277:   const c10::List<c10::IValue> getCollectiveTrace(
278:       bool includeStacktraces,
279:       bool onlyActive);
280: 
281:   // dump pg_entries
282:   const c10::Dict<c10::IValue, c10::IValue> getPgConfig();
283: 
284:   const std::map<std::string, std::map<std::string, std::string>>
285:   getPgConfigJson();
286: 
287:   // dump pg_status
288:   const c10::Dict<c10::IValue, c10::IValue> getPgStatus();
```

- EN: Lines 273-288 introduces executable logic in routines such as `reset_all`, `getCollectiveTrace`, `getPgConfig`.
- CN: 第 273-288 行在 `reset_all`、`getCollectiveTrace`、`getPgConfig` 等例程中引入具体执行逻辑。

### Lines 289-304 / 第 289-304 行

```cpp
289: 
290:   const std::map<std::string, std::map<std::string, std::string>>
291:   getPgStatusJson();
292: 
293:   std::string dump_json(
294:       const std::optional<std::unordered_map<
295:           std::string,
296:           std::unordered_map<std::string, std::string>>>& extraDumpMap,
297:       bool includeCollectives,
298:       bool onlyActive);
299: 
300:   std::string dump(
301:       const std::optional<std::unordered_map<
302:           std::string,
303:           std::unordered_map<std::string, std::string>>>& extraDumpMap,
304:       bool includeCollectives,
```

- EN: Lines 289-304 introduces executable logic in routines such as `getPgStatusJson`, `dump_json`.
- CN: 第 289-304 行在 `getPgStatusJson`、`dump_json` 等例程中引入具体执行逻辑。

### Lines 305-320 / 第 305-320 行

```cpp
305:       bool includeStackTraces,
306:       bool onlyActive);
307: };
308: 
309: // Whether to include stack trace in the Flight Recorder trace (default true)
310: static std::vector<std::string> TORCH_INCLUDE_STACK_TRACE = {
311:     "TORCH_INCLUDE_STACK_TRACE"};
312: 
313: // Whether to include only active collectives in the Flight Recorder trace
314: // (default false)
315: static std::vector<std::string> TORCH_INCLUDE_ONLY_ACTIVE = {
316:     "TORCH_INCLUDE_ONLY_ACTIVE"};
317: 
318: // Dumps the fr traces and additional information about the Process
319: // Group.
320: TORCH_API std::string dump_fr_trace(
```

- EN: Lines 305-320 mainly closes scopes and completes the surrounding declaration or implementation block.
- CN: 第 305-320 行主要用于关闭作用域并结束当前声明或实现块。

### Lines 321-331 / 第 321-331 行

```cpp
321:     bool includeCollectives,
322:     bool includeStackTraces,
323:     bool onlyActive);
324: 
325: // Dumps the fr traces and additional information about the Process
326: // Group in JSON formatted string.
327: // We don't include stack traces in JSON format as it is far too much data.
328: TORCH_API std::string dump_fr_trace_json(
329:     bool includeCollectives,
330:     bool onlyActive);
331: } // namespace c10d
```

- EN: Lines 321-331 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `dump_fr_trace_json`.
- CN: 第 321-331 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `dump_fr_trace_json` 等例程中引入具体执行逻辑。

## Key Concepts / 关键概念

- EN: Subsystem: c10d distributed process-group subsystem.
- CN: 子系统：c10d 分布式进程组子系统。
- EN: File role: interface/definition.
- CN: 文件角色：接口/定义。
- EN: Primary symbols: `TORCH_API`, `FlightRecorder`, `Entry`, `TraceIdentifier`
- CN: 核心符号：`TORCH_API`、`FlightRecorder`、`Entry`、`TraceIdentifier`
- EN: Notable themes: process-group orchestration, collective communication logic.
- CN: 值得关注的主题：进程组编排、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/TraceUtils.h`, `torch/csrc/distributed/c10d/logger.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/ATen.h`, `c10/util/Exception.h`
- External or system headers / 外部或系统头文件: `cstdio`, `cstdlib`, `memory`, `mutex`, `optional`
- Local symbols / 本地符号: `TORCH_API`, `FlightRecorder`, `Entry`, `TraceIdentifier`