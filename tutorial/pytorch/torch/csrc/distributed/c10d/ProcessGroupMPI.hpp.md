# ProcessGroupMPI.hpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/ProcessGroupMPI.hpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides process-group backend logic in the c10d distributed process-group subsystem. Key types include `WorkEntry`, `TORCH_API`, `WorkMPI`.
- 用途 (CN): 该文件在c10d 分布式进程组子系统中提供进程组后端逻辑。 关键类型包括 `WorkEntry`、`TORCH_API`、`WorkMPI`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
1: #pragma once
2: 
3: #ifdef USE_C10D_MPI
4: 
5: #include <condition_variable>
6: #include <deque>
7: #include <exception>
8: #include <memory>
9: #include <mutex>
10: #include <thread>
11: #include <vector>
12: 
13: #include <ATen/core/ivalue.h>
14: #include <ATen/core/ivalue_inl.h>
15: 
16: #include <torch/csrc/distributed/c10d/Backend.hpp>
```

- EN: Lines 1-16 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends.
- CN: 第 1-16 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端。

### Lines 17-32 / 第 17-32 行

```cpp
17: #include <torch/csrc/distributed/c10d/Types.hpp>
18: #include <torch/csrc/distributed/c10d/Utils.hpp>
19: 
20: #include <mpi.h>
21: 
22: namespace c10d {
23: 
24: constexpr const char* MPI_BACKEND_NAME = "mpi";
25: 
26: // WorkEntry is the state associated with a single MPI run instance.
27: // It include the source Tensor list and destination Tensor list, as well as
28: // The actual run function that will operate either on src or dst or both.
29: struct WorkEntry {
30:   explicit WorkEntry(
31:       std::vector<at::Tensor>* srcPtr,
32:       std::vector<at::Tensor>* dstPtr,
```

- EN: Lines 17-32 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component; declares or defines types such as `WorkEntry`.
- CN: 第 17-32 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中；声明或定义了 `WorkEntry` 等类型。

### Lines 33-48 / 第 33-48 行

```cpp
33:       std::function<void(std::unique_ptr<WorkEntry>&)> run)
34:       : dst(dstPtr ? *dstPtr : std::vector<at::Tensor>()), run(std::move(run)) {
35:     if (srcPtr) {
36:       src = *srcPtr;
37:     }
38:   }
39: 
40:   // Not copyable
41:   WorkEntry(const WorkEntry&) = delete;
42:   // Not copy assignable
43:   WorkEntry& operator=(const WorkEntry&) = delete;
44: 
45:   // For input and output tensors (in-place), we will always use src
46:   std::vector<at::Tensor> src;
47: 
48:   // Copy of user provided outputs.
```

- EN: Lines 33-48 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 33-48 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 49-64 / 第 49-64 行

```cpp
49:   const std::vector<at::Tensor> dst;
50: 
51:   // src rank returned, for recv only
52:   int* srcRank = nullptr;
53:   std::function<void(std::unique_ptr<WorkEntry>&)> run;
54: };
55: 
56: // ProcessGroupMPI implements MPI bindings for c10d.
57: //
58: // All functions on this class are expected to be called in the same
59: // order across processes in the group. This is the only way that we
60: // can guarantee to match up the same calls across processes.
61: //
62: // All MPI functions provided by this class is asynchronously scheduled on a
63: // Worker thread. Therefore, ProcessGroupMPI requires the MPI implementation
64: // that is used to have a minimum thread support value of MPI_THREAD_SERIALIZED.
```

- EN: Lines 49-64 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 49-64 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 65-80 / 第 65-80 行

```cpp
65: // That is, The process may be multi-threaded, and multiple threads may make
66: // MPI calls, but only one at a time: MPI calls are not made concurrently from
67: // two distinct threads (all MPI calls are serialized). However, with
68: // MPI_THREAD_SERIALIZED, ProcessGroupMPI will only support a single process
69: // group. In other words, no more than 1 process group can be created globally.
70: //
71: // If you would like to use multiple ProcessGroupMPI, it requires your MPI
72: // implementation to have a thread support value of MPI_THREAD_MULTIPLE, that
73: // is, multiple threads may call MPI, with no restriction.
74: //
75: // Also note that ProcessGroupMPI only supports a single Tensor operation. In
76: // other words, the size of the input Tensor vector should always be 1.
77: //
78: // CUDA tensor can be supported if the MPI used is CUDA-aware MPI, and
79: // ProcessGroupMPI will automatically detect this support.
80: class TORCH_API ProcessGroupMPI : public Backend {
```

- EN: Lines 65-80 declares or defines types such as `TORCH_API`.
- CN: 第 65-80 行声明或定义了 `TORCH_API` 等类型。

### Lines 81-96 / 第 81-96 行

```cpp
81:  public:
82:   class WorkMPI : public Work {
83:    public:
84:     explicit WorkMPI(
85:         std::vector<at::Tensor> outputTensors,
86:         const char* profilingTitle = nullptr,
87:         const std::optional<std::vector<at::Tensor>>& inputTensors =
88:             std::nullopt)
89:         : Work(-1, OpType::UNKNOWN, profilingTitle, inputTensors),
90:           outputTensors_(std::move(outputTensors)),
91:           future_(c10::make_intrusive<at::ivalue::Future>(
92:               c10::ListType::create(c10::TensorType::get()))) {}
93: 
94:     std::vector<at::Tensor> result() override;
95: 
96:     c10::intrusive_ptr<c10::ivalue::Future> getFuture() override;
```

- EN: Lines 81-96 declares or defines types such as `WorkMPI`; introduces executable logic in routines such as `result`, `getFuture`.
- CN: 第 81-96 行声明或定义了 `WorkMPI` 等类型；在 `result`、`getFuture` 等例程中引入具体执行逻辑。

### Lines 97-112 / 第 97-112 行

```cpp
97: 
98:    protected:
99:     friend class ProcessGroupMPI;
100: 
101:    private:
102:     void finishWorkMPI();
103:     void finishWorkMPIError(const std::exception_ptr& eptr);
104: 
105:     std::vector<at::Tensor> outputTensors_;
106:     c10::intrusive_ptr<at::ivalue::Future> future_;
107:   };
108: 
109:   class AsyncWork : public Work {
110:    public:
111:     AsyncWork(
112:         MPI_Request request,
```

- EN: Lines 97-112 declares or defines types such as `AsyncWork`; introduces executable logic in routines such as `finishWorkMPI`, `finishWorkMPIError`.
- CN: 第 97-112 行声明或定义了 `AsyncWork` 等类型；在 `finishWorkMPI`、`finishWorkMPIError` 等例程中引入具体执行逻辑。

### Lines 113-128 / 第 113-128 行

```cpp
113:         std::vector<at::Tensor> outputTensors,
114:         const char* profilingTitle = nullptr,
115:         const std::optional<std::vector<at::Tensor>>& inputTensors =
116:             std::nullopt);
117: 
118:     ~AsyncWork() override;
119: 
120:     bool isCompleted() override;
121: 
122:     bool isSuccess() const override;
123: 
124:     int sourceRank() const override;
125: 
126:     bool wait(std::chrono::milliseconds timeout = kUnsetTimeout) override;
127: 
128:     void abort() override;
```

- EN: Lines 113-128 introduces executable logic in routines such as `~AsyncWork`, `isCompleted`, `isSuccess`.
- CN: 第 113-128 行在 `~AsyncWork`、`isCompleted`、`isSuccess` 等例程中引入具体执行逻辑。

### Lines 129-144 / 第 129-144 行

```cpp
129: 
130:     std::vector<at::Tensor> result() override;
131: 
132:    protected:
133:     void populateException();
134: 
135:    private:
136:     const std::vector<at::Tensor> outputTensors_;
137:     MPI_Request request_;
138:     MPI_Status status_{};
139:   };
140: 
141:   // Constructor will spawn up the worker thread loop
142:   explicit ProcessGroupMPI(int rank, int size, MPI_Comm pgComm);
143: 
144:   ~ProcessGroupMPI() override;
```

- EN: Lines 129-144 introduces executable logic in routines such as `result`, `populateException`, `ProcessGroupMPI`.
- CN: 第 129-144 行在 `result`、`populateException`、`ProcessGroupMPI` 等例程中引入具体执行逻辑。

### Lines 145-160 / 第 145-160 行

```cpp
145: 
146:   // Abort the MPI program, needs to be called when exception is detected
147:   void abort() override;
148: 
149:   const std::string getBackendName() const override {
150:     return std::string(MPI_BACKEND_NAME);
151:   }
152: 
153:   c10::intrusive_ptr<Work> broadcast(
154:       std::vector<at::Tensor>& data,
155:       const BroadcastOptions& opts = BroadcastOptions()) override;
156: 
157:   c10::intrusive_ptr<Work> allreduce(
158:       std::vector<at::Tensor>& tensors,
159:       const AllreduceOptions& opts = AllreduceOptions()) override;
160: 
```

- EN: Lines 145-160 introduces executable logic in routines such as `abort`, `getBackendName`; returns computed state or forwards results to the surrounding caller.
- CN: 第 145-160 行在 `abort`、`getBackendName` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 161-176 / 第 161-176 行

```cpp
161:   c10::intrusive_ptr<Work> allreduce_coalesced(
162:       std::vector<at::Tensor>& tensors,
163:       const AllreduceCoalescedOptions& opts =
164:           AllreduceCoalescedOptions()) override;
165: 
166:   c10::intrusive_ptr<Work> reduce(
167:       std::vector<at::Tensor>& tensors,
168:       const ReduceOptions& opts = ReduceOptions()) override;
169: 
170:   c10::intrusive_ptr<Work> allgather(
171:       std::vector<std::vector<at::Tensor>>& outputTensors,
172:       std::vector<at::Tensor>& inputTensors,
173:       const AllgatherOptions& opts = AllgatherOptions()) override;
174: 
175:   c10::intrusive_ptr<Work> _allgather_base(
176:       at::Tensor& outputbuffer,
```

- EN: Lines 161-176 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 161-176 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 177-192 / 第 177-192 行

```cpp
177:       at::Tensor& inputbuffer,
178:       const AllgatherOptions& opts = AllgatherOptions()) override;
179: 
180:   c10::intrusive_ptr<Work> allgather_coalesced(
181:       std::vector<std::vector<at::Tensor>>& outputTensorLists,
182:       std::vector<at::Tensor>& inputTensors,
183:       const AllgatherOptions& opts = AllgatherOptions()) override;
184: 
185:   c10::intrusive_ptr<Work> gather(
186:       std::vector<std::vector<at::Tensor>>& outputTensors,
187:       std::vector<at::Tensor>& inputTensors,
188:       const GatherOptions& opts = GatherOptions()) override;
189: 
190:   c10::intrusive_ptr<Work> scatter(
191:       std::vector<at::Tensor>& outputTensors,
192:       std::vector<std::vector<at::Tensor>>& inputTensors,
```

- EN: Lines 177-192 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 177-192 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 193-208 / 第 193-208 行

```cpp
193:       const ScatterOptions& opts = ScatterOptions()) override;
194: 
195:   c10::intrusive_ptr<Work> reduce_scatter(
196:       std::vector<at::Tensor>& outputTensors,
197:       std::vector<std::vector<at::Tensor>>& inputTensors,
198:       const ReduceScatterOptions& opts = ReduceScatterOptions()) override;
199: 
200:   c10::intrusive_ptr<Work> _reduce_scatter_base(
201:       at::Tensor& outputTensor,
202:       at::Tensor& inputTensor,
203:       const ReduceScatterOptions& opts = ReduceScatterOptions()) override;
204: 
205:   c10::intrusive_ptr<Work> alltoall_base(
206:       at::Tensor& outputTensor,
207:       at::Tensor& inputTensor,
208:       std::vector<int64_t>& outputSplitSizes,
```

- EN: Lines 193-208 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 193-208 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 209-224 / 第 209-224 行

```cpp
209:       std::vector<int64_t>& inputSplitSizes,
210:       const AllToAllOptions& opts = AllToAllOptions()) override;
211: 
212:   c10::intrusive_ptr<Work> alltoall(
213:       std::vector<at::Tensor>& outputTensors,
214:       std::vector<at::Tensor>& inputTensors,
215:       const AllToAllOptions& opts = AllToAllOptions()) override;
216: 
217:   c10::intrusive_ptr<Work> send(
218:       std::vector<at::Tensor>& tensors,
219:       int dstRank,
220:       int tag) override;
221: 
222:   c10::intrusive_ptr<Work> recv(
223:       std::vector<at::Tensor>& tensors,
224:       int srcRank,
```

- EN: Lines 209-224 introduces executable logic in routines such as `send`.
- CN: 第 209-224 行在 `send` 等例程中引入具体执行逻辑。

### Lines 225-240 / 第 225-240 行

```cpp
225:       int tag) override;
226: 
227:   c10::intrusive_ptr<Work> recvAnysource(
228:       std::vector<at::Tensor>& tensor,
229:       int tag) override;
230: 
231:   c10::intrusive_ptr<Work> barrier(
232:       const BarrierOptions& opts = BarrierOptions()) override;
233: 
234:   // Creating a new ProcessGroupMPI, will initialize MPI if not initialized
235:   static c10::intrusive_ptr<ProcessGroupMPI> createProcessGroupMPI(
236:       std::vector<int> ranks = {});
237: 
238:  protected:
239:   using WorkType =
240:       std::tuple<std::unique_ptr<WorkEntry>, c10::intrusive_ptr<WorkMPI>>;
```

- EN: Lines 225-240 introduces executable logic in routines such as `recvAnysource`.
- CN: 第 225-240 行在 `recvAnysource` 等例程中引入具体执行逻辑。

### Lines 241-256 / 第 241-256 行

```cpp
241:   // Worker thread loop
242:   void runLoop();
243:   // Helper function that is called by the destructor
244:   void destroy();
245: 
246:   c10::intrusive_ptr<Work> enqueue(
247:       std::unique_ptr<WorkEntry> entry,
248:       const char* profilingTitle = nullptr,
249:       const std::optional<std::vector<at::Tensor>>& inputTensors =
250:           std::nullopt);
251: 
252:   bool stop_{false};
253: 
254:   std::mutex pgMutex_;
255:   std::thread workerThread_;
256: 
```

- EN: Lines 241-256 introduces executable logic in routines such as `runLoop`, `destroy`, `enqueue`.
- CN: 第 241-256 行在 `runLoop`、`destroy`、`enqueue` 等例程中引入具体执行逻辑。

### Lines 257-272 / 第 257-272 行

```cpp
257:   std::deque<WorkType> queue_;
258:   std::condition_variable queueProduceCV_;
259:   std::condition_variable queueConsumeCV_;
260: 
261:   // Global states
262:   static void initMPIOnce();
263:   static void mpiExit();
264: 
265:   static std::mutex pgGlobalMutex_;
266:   static int mpiThreadSupport_;
267: 
268:   MPI_Comm pgComm_;
269: };
270: 
271: } // namespace c10d
272: 
```

- EN: Lines 257-272 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `initMPIOnce`, `mpiExit`.
- CN: 第 257-272 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `initMPIOnce`、`mpiExit` 等例程中引入具体执行逻辑。

### Lines 273-273 / 第 273-273 行

```cpp
273: #endif // USE_C10D_MPI
```

- EN: Lines 273-273 uses conditional compilation to adapt to feature flags, platforms, or optional backends.
- CN: 第 273-273 行使用条件编译来适配特性开关、平台或可选后端。

## Key Concepts / 关键概念

- EN: Subsystem: c10d distributed process-group subsystem.
- CN: 子系统：c10d 分布式进程组子系统。
- EN: File role: interface/definition.
- CN: 文件角色：接口/定义。
- EN: Primary symbols: `WorkEntry`, `TORCH_API`, `WorkMPI`, `AsyncWork`
- CN: 核心符号：`WorkEntry`、`TORCH_API`、`WorkMPI`、`AsyncWork`
- EN: Notable themes: process-group orchestration, collective communication logic.
- CN: 值得关注的主题：进程组编排、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/Backend.hpp`, `torch/csrc/distributed/c10d/Types.hpp`, `torch/csrc/distributed/c10d/Utils.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/core/ivalue.h`, `ATen/core/ivalue_inl.h`
- External or system headers / 外部或系统头文件: `condition_variable`, `deque`, `exception`, `memory`, `mutex`, `thread`, `vector`, `mpi.h`
- Local symbols / 本地符号: `WorkEntry`, `TORCH_API`, `WorkMPI`, `AsyncWork`