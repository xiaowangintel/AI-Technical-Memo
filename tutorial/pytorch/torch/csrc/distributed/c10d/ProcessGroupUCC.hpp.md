# ProcessGroupUCC.hpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/ProcessGroupUCC.hpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides process-group backend logic in the c10d distributed process-group subsystem. Key types include `event_pool_t`, `Comm`, `TORCH_API`.
- 用途 (CN): 该文件在c10d 分布式进程组子系统中提供进程组后端逻辑。 关键类型包括 `event_pool_t`、`Comm`、`TORCH_API`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
1: #pragma once
2: 
3: #ifdef USE_C10D_UCC
4: 
5: #include <torch/csrc/distributed/c10d/UCCUtils.hpp>
6: 
7: #include <exception>
8: #include <memory>
9: #include <mutex>
10: #include <queue>
11: #include <thread>
12: #include <vector>
13: 
14: #include <torch/csrc/distributed/c10d/Backend.hpp>
15: #include <torch/csrc/distributed/c10d/Store.hpp>
16: #include <torch/csrc/distributed/c10d/Types.hpp>
```

- EN: Lines 1-16 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends.
- CN: 第 1-16 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端。

### Lines 17-32 / 第 17-32 行

```cpp
17: #include <torch/csrc/distributed/c10d/Utils.hpp>
18: #ifdef USE_CUDA
19: #include <ATen/cuda/CUDAEvent.h>
20: #include <c10/cuda/CUDAStream.h>
21: #endif
22: 
23: namespace c10d {
24: 
25: #define TORCH_UCC_DEVICE_NOT_SET -2
26: 
27: #ifdef USE_CUDA
28: #define SAVE_TENSORS(_TENSORS, _DATA)                       \
29:   do {                                                      \
30:     if ((_TENSORS)[0].device().is_cuda()) {                 \
31:       for (const auto i : c10::irange((_TENSORS).size())) { \
32:         c10::cuda::CUDACachingAllocator::recordStream(      \
```

- EN: Lines 17-32 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 17-32 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 33-48 / 第 33-48 行

```cpp
33:             (_TENSORS)[i].storage().data_ptr(), (*stream)); \
34:       }                                                     \
35:     } else {                                                \
36:       (_DATA) = (_TENSORS);                                 \
37:     }                                                       \
38:   } while (0)
39: 
40: #else
41: #define SAVE_TENSORS(_TENSORS, _DATA) (_DATA) = (_TENSORS);
42: #endif
43: 
44: constexpr const char* UCC_BACKEND_NAME = "ucc";
45: 
46: struct event_pool_t {
47: #ifdef USE_CUDA
48:   std::queue<std::unique_ptr<at::cuda::CUDAEvent>> event_pool;
```

- EN: Lines 33-48 uses conditional compilation to adapt to feature flags, platforms, or optional backends; declares or defines types such as `event_pool_t`; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 33-48 行使用条件编译来适配特性开关、平台或可选后端；声明或定义了 `event_pool_t` 等类型；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 49-64 / 第 49-64 行

```cpp
49: #endif
50:   std::mutex event_pool_mutex;
51: };
52: 
53: class Comm;
54: 
55: // UCC does not support multiple CUDA devices per process.
56: class TORCH_API ProcessGroupUCC : public Backend {
57:  private:
58:   void set_timeout(ucc_coll_args_t& args);
59: 
60:  public:
61:   class WorkData {
62:    public:
63:     std::vector<at::Tensor> src;
64:     std::vector<at::Tensor> dst;
```

- EN: Lines 49-64 uses conditional compilation to adapt to feature flags, platforms, or optional backends; declares or defines types such as `Comm`, `TORCH_API`, `WorkData`; introduces executable logic in routines such as `set_timeout`.
- CN: 第 49-64 行使用条件编译来适配特性开关、平台或可选后端；声明或定义了 `Comm`、`TORCH_API`、`WorkData` 等类型；在 `set_timeout` 等例程中引入具体执行逻辑。

### Lines 65-80 / 第 65-80 行

```cpp
65:     std::vector<at::Tensor> flat;
66:     WorkData() {}
67:     virtual ~WorkData() = default;
68:   };
69:   class AlltoallWorkData : public WorkData {
70:    public:
71:     AlltoallWorkData(int size)
72:         : send_lengths(size),
73:           send_offsets(size),
74:           recv_lengths(size),
75:           recv_offsets(size) {}
76:     std::vector<uint64_t> send_lengths;
77:     std::vector<uint64_t> send_offsets;
78:     std::vector<uint64_t> recv_lengths;
79:     std::vector<uint64_t> recv_offsets;
80:   };
```

- EN: Lines 65-80 declares or defines types such as `AlltoallWorkData`; introduces executable logic in routines such as `WorkData`.
- CN: 第 65-80 行声明或定义了 `AlltoallWorkData` 等类型；在 `WorkData` 等例程中引入具体执行逻辑。

### Lines 81-96 / 第 81-96 行

```cpp
81: 
82:   class AllgathervWorkData : public WorkData {
83:    public:
84:     AllgathervWorkData(int size) : recv_lengths(size), recv_offsets(size) {}
85:     std::vector<uint64_t> recv_lengths;
86:     std::vector<uint64_t> recv_offsets;
87:   };
88: 
89:   class ScattervWorkData : public WorkData {
90:    public:
91:     ScattervWorkData(int size) : send_lengths(size), send_offsets(size) {}
92:     std::vector<uint64_t> send_lengths;
93:     std::vector<uint64_t> send_offsets;
94:   };
95: 
96:   class ProgressEntry {
```

- EN: Lines 81-96 declares or defines types such as `AllgathervWorkData`, `ScattervWorkData`, `ProgressEntry`; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 81-96 行声明或定义了 `AllgathervWorkData`、`ScattervWorkData`、`ProgressEntry` 等类型；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 97-112 / 第 97-112 行

```cpp
97:     friend class ProcessGroupUCC;
98:     friend class Comm;
99: 
100:    public:
101:     ProgressEntry(CommBase* comm, ucc_coll_req_h request)
102:         : status_(UCC_INPROGRESS), comm_(comm), request_(request) {}
103:     // Finalizes UCC status or exception of collective request.
104:     void finalize(std::exception_ptr eptr = nullptr);
105:     ucc_status_t status_;
106:     CommBase* comm_;
107:     ucc_coll_req_h request_;
108:     std::unique_ptr<WorkData> data;
109:     c10::intrusive_ptr<c10::ivalue::Future> future_;
110:     std::exception_ptr eptr_;
111:   };
112: 
```

- EN: Lines 97-112 introduces executable logic in routines such as `finalize`.
- CN: 第 97-112 行在 `finalize` 等例程中引入具体执行逻辑。

### Lines 113-128 / 第 113-128 行

```cpp
113:   class WorkUCC : public Work {
114:     friend class ProcessGroupUCC;
115:     friend class Comm;
116: 
117:    public:
118:     WorkUCC(
119:         OpType opType,
120:         uint64_t seq,
121:         const char* prof_title,
122:         const std::optional<std::vector<at::Tensor>>& inputs,
123:         const c10::intrusive_ptr<ProcessGroupUCCLogger>& logger)
124:         : Work(-1, opType, prof_title, inputs), logger_(logger), seq_(seq) {}
125:     ~WorkUCC();
126:     void setException();
127:     void setAndThrowException();
128:     bool isCompleted() override;
```

- EN: Lines 113-128 declares or defines types such as `WorkUCC`; introduces executable logic in routines such as `~WorkUCC`, `setException`, `setAndThrowException`.
- CN: 第 113-128 行声明或定义了 `WorkUCC` 等类型；在 `~WorkUCC`、`setException`、`setAndThrowException` 等例程中引入具体执行逻辑。

### Lines 129-144 / 第 129-144 行

```cpp
129:     bool isSuccess() const override;
130:     bool wait(std::chrono::milliseconds timeout = kUnsetTimeout) override;
131:     c10::intrusive_ptr<c10::ivalue::Future> getFuture() override;
132:     std::vector<at::Tensor> result() override;
133:     int sourceRank() const override;
134: #ifdef USE_CUDA
135:     std::unique_ptr<at::cuda::CUDAEvent> fence = nullptr;
136:     event_pool_t* ep = nullptr;
137: #endif
138:     int sourceRank_;
139: 
140:    protected:
141:     std::shared_ptr<ProgressEntry> entry_;
142:     c10::intrusive_ptr<ProcessGroupUCCLogger> logger_;
143:     uint64_t seq_;
144: 
```

- EN: Lines 129-144 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `isSuccess`, `wait`, `getFuture`.
- CN: 第 129-144 行使用条件编译来适配特性开关、平台或可选后端；在 `isSuccess`、`wait`、`getFuture` 等例程中引入具体执行逻辑。

### Lines 145-160 / 第 145-160 行

```cpp
145:    private:
146:     // The future returned by getFuture.
147:     c10::intrusive_ptr<at::ivalue::Future> future_;
148:     // Store a reference to collective's outputs, used by result
149:     std::shared_ptr<std::vector<at::Tensor>> outputs_;
150:   };
151: 
152:   explicit ProcessGroupUCC(
153:       const c10::intrusive_ptr<Store>& store,
154:       int rank = -1,
155:       int size = -1,
156:       std::chrono::duration<float> timeout = kBackendDefaultTimeout);
157: 
158:   void initComm(c10::Device dev);
159: 
160:   ~ProcessGroupUCC() override;
```

- EN: Lines 145-160 introduces executable logic in routines such as `ProcessGroupUCC`, `initComm`, `~ProcessGroupUCC`.
- CN: 第 145-160 行在 `ProcessGroupUCC`、`initComm`、`~ProcessGroupUCC` 等例程中引入具体执行逻辑。

### Lines 161-176 / 第 161-176 行

```cpp
161: 
162:   const std::string getBackendName() const override {
163:     return std::string(UCC_BACKEND_NAME);
164:   }
165: 
166: #ifdef USE_CUDA
167:   std::unique_ptr<at::cuda::CUDAEvent> getPooledEvent();
168: #endif
169: 
170:   // Performs a health check by initializing dummy UCC & UCX communicators and
171:   // then destroying them. This will help indicate and signal any
172:   // UCC/UCX-related issues prior to the first collective. The actual
173:   // initialization and subsequent destruction is ran on a separate thread and
174:   // the main thread is signalled about timeouts/errors to report to the
175:   // application.
176:   void runHealthCheck();
```

- EN: Lines 161-176 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `getBackendName`, `getPooledEvent`, `runHealthCheck`.
- CN: 第 161-176 行使用条件编译来适配特性开关、平台或可选后端；在 `getBackendName`、`getPooledEvent`、`runHealthCheck` 等例程中引入具体执行逻辑。

### Lines 177-192 / 第 177-192 行

```cpp
177: 
178:   template <typename PreProcess, typename PostProcess>
179:   c10::intrusive_ptr<Work> collective_post(
180:       OpType opType,
181:       PreProcess preproc,
182:       PostProcess postproc,
183:       ucc_coll_args_t& coll,
184:       std::unique_ptr<ProcessGroupUCC::WorkData> data,
185:       c10::Device dev,
186:       std::vector<at::Tensor>& inputTensors,
187:       std::vector<at::Tensor>& outputTensors,
188:       const char* prof_title);
189: 
190:   c10::intrusive_ptr<Work> broadcast(
191:       std::vector<at::Tensor>& data,
192:       const BroadcastOptions& opts = BroadcastOptions()) override;
```

- EN: Lines 177-192 introduces executable logic in routines such as `collective_post`.
- CN: 第 177-192 行在 `collective_post` 等例程中引入具体执行逻辑。

### Lines 193-208 / 第 193-208 行

```cpp
193: 
194:   c10::intrusive_ptr<Work> allreduce(
195:       std::vector<at::Tensor>& tensors,
196:       const AllreduceOptions& opts = AllreduceOptions()) override;
197: 
198:   c10::intrusive_ptr<Work> allreduce_coalesced(
199:       std::vector<at::Tensor>& tensors,
200:       const AllreduceCoalescedOptions& opts =
201:           AllreduceCoalescedOptions()) override;
202: 
203:   c10::intrusive_ptr<Work> reduce(
204:       std::vector<at::Tensor>& tensors,
205:       const ReduceOptions& opts = ReduceOptions()) override;
206: 
207:   c10::intrusive_ptr<Work> allgather(
208:       std::vector<std::vector<at::Tensor>>& outputTensors,
```

- EN: Lines 193-208 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 193-208 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 209-224 / 第 209-224 行

```cpp
209:       std::vector<at::Tensor>& inputTensors,
210:       const AllgatherOptions& opts = AllgatherOptions()) override;
211: 
212:   c10::intrusive_ptr<Work> _allgather_base(
213:       at::Tensor& outputBuffer,
214:       at::Tensor& inputBuffer,
215:       const AllgatherOptions& opts = AllgatherOptions()) override;
216: 
217:   c10::intrusive_ptr<Work> barrier(
218:       const BarrierOptions& opts = BarrierOptions()) override;
219: 
220:   c10::intrusive_ptr<Work> gather(
221:       std::vector<std::vector<at::Tensor>>& outputTensors,
222:       std::vector<at::Tensor>& inputTensors,
223:       const GatherOptions& opts = GatherOptions()) override;
224: 
```

- EN: Lines 209-224 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 209-224 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 225-240 / 第 225-240 行

```cpp
225:   c10::intrusive_ptr<Work> scatter(
226:       std::vector<at::Tensor>& outputTensors,
227:       std::vector<std::vector<at::Tensor>>& inputTensors,
228:       const ScatterOptions& opts = ScatterOptions()) override;
229: 
230:   c10::intrusive_ptr<Work> reduce_scatter(
231:       std::vector<at::Tensor>& outputTensors,
232:       std::vector<std::vector<at::Tensor>>& inputTensors,
233:       const ReduceScatterOptions& opts = ReduceScatterOptions()) override;
234: 
235:   c10::intrusive_ptr<Work> _reduce_scatter_base(
236:       at::Tensor& outputTensor,
237:       at::Tensor& inputTensor,
238:       const ReduceScatterOptions& opts = ReduceScatterOptions()) override;
239: 
240:   c10::intrusive_ptr<Work> alltoall_base(
```

- EN: Lines 225-240 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 225-240 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 241-256 / 第 241-256 行

```cpp
241:       at::Tensor& outputTensor,
242:       at::Tensor& inputTensor,
243:       std::vector<int64_t>& outputSplitSizes,
244:       std::vector<int64_t>& inputSplitSizes,
245:       const AllToAllOptions& opts = AllToAllOptions()) override;
246: 
247:   c10::intrusive_ptr<Work> alltoall(
248:       std::vector<at::Tensor>& outputTensors,
249:       std::vector<at::Tensor>& inputTensors,
250:       const AllToAllOptions& opts = AllToAllOptions()) override;
251: 
252:   c10::intrusive_ptr<Work> send(
253:       std::vector<at::Tensor>& tensors,
254:       int dstRank,
255:       int tag) override;
256: 
```

- EN: Lines 241-256 introduces executable logic in routines such as `send`.
- CN: 第 241-256 行在 `send` 等例程中引入具体执行逻辑。

### Lines 257-272 / 第 257-272 行

```cpp
257:   c10::intrusive_ptr<Work> recv(
258:       std::vector<at::Tensor>& tensors,
259:       int srcRank,
260:       int tag) override;
261: 
262:   // Counting for the sequential number of UCC collective_post call.
263:   uint64_t seq_{0};
264: 
265:   // Agrees on an initial sequence number for the whole group by having rank 0
266:   // create it and broadcast it to other ranks using the store.
267:   void setSequenceNumberForGroup() override;
268: 
269:   // Retrieves the current sequence number for the whole group, which should be
270:   // in sync. If the returned number is not consistent across the group, it
271:   // may indicate that there is some sort of collective desynchronization.
272:   uint64_t getSequenceNumberForGroup() override;
```

- EN: Lines 257-272 introduces executable logic in routines such as `recv`, `setSequenceNumberForGroup`, `getSequenceNumberForGroup`.
- CN: 第 257-272 行在 `recv`、`setSequenceNumberForGroup`、`getSequenceNumberForGroup` 等例程中引入具体执行逻辑。

### Lines 273-288 / 第 273-288 行

```cpp
273: 
274:   static c10::intrusive_ptr<Backend> createProcessGroupUCC(
275:       const c10::intrusive_ptr<::c10d::Store>& store,
276:       int rank,
277:       int size,
278:       const std::chrono::duration<float>& timeout);
279: 
280:  protected:
281:   const std::chrono::duration<float> timeout_;
282:   std::shared_ptr<torch_ucc_oob_coll_info_t> oob;
283:   std::shared_ptr<Comm> comm = {nullptr};
284:   uint32_t comm_id;
285:   ucc_team_h team{nullptr};
286:   ucc_ee_h cuda_ee{nullptr};
287:   ucc_ee_h cuda_ee_p2p[2]{nullptr, nullptr};
288: 
```

- EN: Lines 273-288 introduces executable logic in routines such as `createProcessGroupUCC`.
- CN: 第 273-288 行在 `createProcessGroupUCC` 等例程中引入具体执行逻辑。

### Lines 289-304 / 第 289-304 行

```cpp
289: #ifdef USE_CUDA
290:   std::unique_ptr<at::cuda::CUDAStream> stream = nullptr;
291:   std::unique_ptr<at::cuda::CUDAStream> stream_p2p[2] = {nullptr, nullptr};
292:   event_pool_t ep;
293: #endif
294:   c10::intrusive_ptr<ProcessGroupUCCLogger> logger;
295: };
296: 
297: class Comm {
298:   c10::intrusive_ptr<ProcessGroupUCCLogger> logger;
299:   std::shared_ptr<torch_ucc_oob_coll_info_t> oob;
300:   CommUCC ucc_comm;
301:   std::mutex mutex;
302:   std::thread progress_thread;
303:   std::condition_variable queue_produce_cv;
304:   std::condition_variable queue_consume_cv;
```

- EN: Lines 289-304 uses conditional compilation to adapt to feature flags, platforms, or optional backends; declares or defines types such as `Comm`.
- CN: 第 289-304 行使用条件编译来适配特性开关、平台或可选后端；声明或定义了 `Comm` 等类型。

### Lines 305-320 / 第 305-320 行

```cpp
305:   std::deque<std::shared_ptr<ProcessGroupUCC::ProgressEntry>> progress_queue;
306:   bool stop_progress_loop;
307:   bool collective_inprogress;
308:   torch_ucc_phase_t finalize_phase;
309: 
310:  public:
311:   c10::DeviceIndex cuda_device_index;
312:   Comm(
313:       const c10::intrusive_ptr<ProcessGroupUCCLogger>& logger,
314:       std::shared_ptr<torch_ucc_oob_coll_info_t> oob,
315:       c10::Device dev,
316:       bool is_health_check);
317: 
318:   ~Comm();
319: 
320:   void ucc_create_team(
```

- EN: Lines 305-320 introduces executable logic in routines such as `Comm`, `~Comm`.
- CN: 第 305-320 行在 `Comm`、`~Comm` 等例程中引入具体执行逻辑。

### Lines 321-336 / 第 321-336 行

```cpp
321:       ucc_team_h& team,
322:       std::shared_ptr<torch_ucc_oob_coll_info_t> oob);
323: 
324:   void ucc_destroy_team(ucc_team_h& team);
325: 
326:   c10::intrusive_ptr<Work> enqueue_p2p(
327:       OpType opType,
328:       ucc_coll_req_h request,
329:       const char* prof_title);
330: 
331: #ifdef USE_CUDA
332:   void enqueue_cuda_collective(
333:       std::unique_ptr<ProcessGroupUCC::WorkData> data,
334:       c10::intrusive_ptr<ProcessGroupUCC::WorkUCC> work,
335:       ucc_coll_args_t& coll,
336:       ucc_team_h team,
```

- EN: Lines 321-336 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `ucc_destroy_team`, `enqueue_p2p`.
- CN: 第 321-336 行使用条件编译来适配特性开关、平台或可选后端；在 `ucc_destroy_team`、`enqueue_p2p` 等例程中引入具体执行逻辑。

### Lines 337-352 / 第 337-352 行

```cpp
337:       ucc_ee_h ee);
338: #endif
339: 
340:   void enqueue_collective(
341:       std::unique_ptr<ProcessGroupUCC::WorkData> data,
342:       c10::intrusive_ptr<ProcessGroupUCC::WorkUCC> work,
343:       ucc_coll_args_t& coll,
344:       ucc_team_h team);
345: 
346:   static std::shared_ptr<Comm> get_comm(
347:       uint32_t& id,
348:       c10::Device dev,
349:       std::shared_ptr<torch_ucc_oob_coll_info_t> oob,
350:       const c10::intrusive_ptr<ProcessGroupUCCLogger>& logger,
351:       bool is_health_check = false);
352: 
```

- EN: Lines 337-352 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `enqueue_collective`, `get_comm`.
- CN: 第 337-352 行使用条件编译来适配特性开关、平台或可选后端；在 `enqueue_collective`、`get_comm` 等例程中引入具体执行逻辑。

### Lines 353-358 / 第 353-358 行

```cpp
353:   void progress_loop();
354: };
355: 
356: } // namespace c10d
357: 
358: #endif // USE_C10D_UCC
```

- EN: Lines 353-358 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `progress_loop`.
- CN: 第 353-358 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `progress_loop` 等例程中引入具体执行逻辑。

## Key Concepts / 关键概念

- EN: Subsystem: c10d distributed process-group subsystem.
- CN: 子系统：c10d 分布式进程组子系统。
- EN: File role: interface/definition.
- CN: 文件角色：接口/定义。
- EN: Primary symbols: `event_pool_t`, `Comm`, `TORCH_API`, `WorkData`
- CN: 核心符号：`event_pool_t`、`Comm`、`TORCH_API`、`WorkData`
- EN: Notable themes: process-group orchestration, store/state coordination, collective communication logic.
- CN: 值得关注的主题：进程组编排、存储/状态协调、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/UCCUtils.hpp`, `torch/csrc/distributed/c10d/Backend.hpp`, `torch/csrc/distributed/c10d/Store.hpp`, `torch/csrc/distributed/c10d/Types.hpp`, `torch/csrc/distributed/c10d/Utils.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/cuda/CUDAEvent.h`, `c10/cuda/CUDAStream.h`
- External or system headers / 外部或系统头文件: `exception`, `memory`, `mutex`, `queue`, `thread`, `vector`
- Local symbols / 本地符号: `event_pool_t`, `Comm`, `TORCH_API`, `WorkData`