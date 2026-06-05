# ProcessGroupUCC.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/ProcessGroupUCC.cpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides process-group backend logic in the c10d distributed process-group subsystem. Key types include `torch_ucc_config_t`, `HealthCheckData`.
- 用途 (CN): 该文件在c10d 分布式进程组子系统中提供进程组后端逻辑。 关键类型包括 `torch_ucc_config_t`、`HealthCheckData`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24 / 第 1-24 行

```cpp
1: #ifdef USE_C10D_UCC
2: 
3: #include <ATen/cuda/nvrtc_stub/ATenNVRTC.h>
4: #include <c10/util/CallOnce.h>
5: #include <c10/util/env.h>
6: #include <torch/csrc/distributed/c10d/ProcessGroup.hpp>
7: #include <torch/csrc/distributed/c10d/ProcessGroupUCC.hpp>
8: #include <torch/csrc/distributed/c10d/UCCTracing.hpp>
9: #include <torch/csrc/distributed/c10d/UCCUtils.hpp>
10: #include <list>
11: #include <memory>
12: #include <unordered_map>
13: #include <unordered_set>
14: 
15: namespace c10d {
16: 
17: namespace {
18: 
19: const std::map<c10::DeviceType, ucc_memory_type_t> ucc_mtype_map = {
20:     {c10::kCPU, UCC_MEMORY_TYPE_HOST},
21:     {c10::kCUDA, UCC_MEMORY_TYPE_CUDA},
22: };
23: 
24: ucc_memory_type_t to_ucc_memType(c10::DeviceType _c10_type) {
```

- EN: Lines 1-24 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 1-24 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 25-48 / 第 25-48 行

```cpp
25:   if (ucc_mtype_map.find(_c10_type) != ucc_mtype_map.end())
26:     return ucc_mtype_map.at(_c10_type);
27:   else
28:     return UCC_MEMORY_TYPE_UNKNOWN;
29: }
30: 
31: const std::map<at::ScalarType, ucc_datatype_t> ucc_dtype_map = {
32:     {at::kByte, UCC_DT_UINT8},
33:     {at::kChar, UCC_DT_INT8},
34:     {at::kHalf, UCC_DT_FLOAT16},
35:     {at::kBFloat16, UCC_DT_BFLOAT16},
36:     {at::kDouble, UCC_DT_FLOAT64},
37:     {at::kFloat, UCC_DT_FLOAT32},
38:     {at::kInt, UCC_DT_INT32},
39:     {at::kLong, UCC_DT_INT64},
40:     {at::kBool, UCC_DT_UINT8},
41: };
42: 
43: ucc_datatype_t to_ucc_dType(at::Tensor _tensor) {
44:   if (_tensor.scalar_type() == at::kBool && _tensor.element_size() != 1) {
45:     TORCH_CHECK(
46:         false, "Size of Boolean type larger than 1 is not supported in UCC");
47:   }
48:   try {
```

- EN: Lines 25-48 introduces executable logic in routines such as `to_ucc_dType`, `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 25-48 行在 `to_ucc_dType`、`TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 49-72 / 第 49-72 行

```cpp
49:     return ucc_dtype_map.at(_tensor.scalar_type());
50:   } catch (const std::out_of_range&) {
51:     TORCH_CHECK(false, "Not supported data type for UCC");
52:   }
53: }
54: 
55: const std::map<ReduceOp, ucc_reduction_op_t> ucc_op_map = {
56:     {ReduceOp::SUM, UCC_OP_SUM},
57:     {ReduceOp::PRODUCT, UCC_OP_PROD},
58:     {ReduceOp::MIN, UCC_OP_MIN},
59:     {ReduceOp::MAX, UCC_OP_MAX},
60:     {ReduceOp::BAND, UCC_OP_BAND},
61:     {ReduceOp::BOR, UCC_OP_BOR},
62:     {ReduceOp::BXOR, UCC_OP_BXOR},
63:     {ReduceOp::AVG, UCC_OP_AVG},
64: };
65: 
66: ucc_reduction_op_t to_ucc_reduceOp(
67:     const ReduceOp _op,
68:     const at::ScalarType _dt) {
69:   if (_dt == at::kBool) {
70:     if (_op == ReduceOp::SUM) {
71:       // bitwise or
72:       return UCC_OP_MAX;
```

- EN: Lines 49-72 introduces executable logic in routines such as `TORCH_CHECK`, `to_ucc_reduceOp`; performs validation and error handling to keep distributed state consistent.
- CN: 第 49-72 行在 `TORCH_CHECK`、`to_ucc_reduceOp` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 73-96 / 第 73-96 行

```cpp
73:     } else if (_op == ReduceOp::PRODUCT) {
74:       // bitwise and
75:       return UCC_OP_MIN;
76:     } else if (_op == ReduceOp::AVG) {
77:       TORCH_CHECK(false, "Cannot use ReduceOp.AVG with boolean inputs");
78:     }
79:   }
80: 
81:   try {
82:     return ucc_op_map.at(_op);
83:   } catch (const std::out_of_range&) {
84:     TORCH_CHECK(false, "Not supported ReduceOp for UCC");
85:   }
86: }
87: 
88: struct torch_ucc_config_t {
89:   c10::once_flag flag;
90:   std::array<bool, 32> blocking_wait;
91:   bool enable_comms_logger;
92:   bool use_future;
93:   // Sharing UCC communicator among multiple PGs to save resource.
94:   bool shared_comm;
95:   // Using allgatherv to achieve allgather, without flattening the list of
96:   // (potentially non-contiguous) tensors.
```

- EN: Lines 73-96 declares or defines types such as `torch_ucc_config_t`; introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 73-96 行声明或定义了 `torch_ucc_config_t` 等类型；在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 97-120 / 第 97-120 行

```cpp
97:   bool use_allgatherv;
98:   bool enable_health_check;
99: } torch_ucc_config;
100: 
101: std::unordered_map<std::string, std::string> torch_ucc_envs_map = {
102:     // TORCH_UCC_BLOCKING_WAIT allowed syntax:
103:     // - TORCH_UCC_BLOCKING_WAIT=none --> blocking wait completely disabled
104:     // - TORCH_UCC_BLOCKING_WAIT=all --> blocking wait completely enabled
105:     // - TORCH_UCC_BLOCKING_WAIT=allreduce,send,recv --> blocking wait enabled
106:     //                                                   on selected operations
107:     // Supported operations:
108:     // [allgather,allgather_base,allreduce,alltoall,broadcast,
109:     //  gather,reduce,reduce_scatter, reduce_scatter_base,scatter,send,recv]
110:     {"TORCH_UCC_BLOCKING_WAIT", "none"},
111: 
112:     {"TORCH_UCC_USE_FUTURE", "1"},
113:     {"TORCH_UCC_PROFILING_ENABLE", "0"},
114:     {"TORCH_UCC_SHARED_COMM", "1"},
115:     {"TORCH_UCC_USE_ALLGATHERV", "0"},
116:     {"TORCH_UCC_ENABLE_HEALTH_CHECK", "0"},
117:     {"TORCH_UCC_ENABLE_COMMS_LOGGER", "0"},
118: };
119: 
120: std::vector<OpType> parse_blocking_wait(std::string op_list_string) {
```

- EN: Lines 97-120 introduces executable logic in routines such as `parse_blocking_wait`.
- CN: 第 97-120 行在 `parse_blocking_wait` 等例程中引入具体执行逻辑。

### Lines 121-144 / 第 121-144 行

```cpp
121:   const static std::unordered_map<std::string, OpType> str2op = {
122:       {"allgather", OpType::ALLGATHER},
123:       {"allgather_base", OpType::_ALLGATHER_BASE},
124:       {"allreduce", OpType::ALLREDUCE},
125:       {"alltoall_base", OpType::ALLTOALL_BASE},
126:       {"broadcast", OpType::BROADCAST},
127:       {"gather", OpType::GATHER},
128:       {"reduce", OpType::REDUCE},
129:       {"reduce_scatter", OpType::REDUCE_SCATTER},
130:       {"reduce_scatter_base", OpType::_REDUCE_SCATTER_BASE},
131:       {"scatter", OpType::SCATTER},
132:       {"send", OpType::SEND},
133:       {"recv", OpType::RECV},
134:   };
135:   auto op_list = parse_list(op_list_string);
136:   if (op_list == std::vector<std::string>{"none"}) {
137:     return {};
138:   }
139:   std::vector<OpType> result;
140:   if (op_list == std::vector<std::string>{"all"}) {
141:     for (auto entry : str2op) {
142:       result.push_back(entry.second);
143:     }
144:   } else {
```

- EN: Lines 121-144 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 121-144 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 145-168 / 第 145-168 行

```cpp
145:     for (auto op_string : op_list) {
146:       result.push_back(str2op.at(op_string));
147:     }
148:   }
149:   return result;
150: }
151: 
152: } // namespace
153: 
154: void read_config() {
155:   // default configuration
156:   torch_ucc_config.blocking_wait.fill(false);
157:   torch_ucc_config.use_future = true;
158:   torch_ucc_config.shared_comm = false;
159:   torch_ucc_config.use_allgatherv = false;
160:   torch_ucc_config.enable_health_check = false;
161:   torch_ucc_config.enable_comms_logger = false;
162: 
163:   // read all torch_ucc env. variables and update the map
164:   for (auto& [env_name, value] : torch_ucc_envs_map) {
165:     auto env = c10::utils::get_env(env_name.c_str());
166:     if (env.has_value()) {
167:       value = std::move(env.value());
168:     }
```

- EN: Lines 145-168 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `read_config`.
- CN: 第 145-168 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `read_config` 等例程中引入具体执行逻辑。

### Lines 169-192 / 第 169-192 行

```cpp
169:   }
170: 
171:   auto blocking_wait_str = torch_ucc_envs_map.at("TORCH_UCC_BLOCKING_WAIT");
172:   for (auto op : parse_blocking_wait(blocking_wait_str)) {
173:     torch_ucc_config.blocking_wait[(std::uint8_t)op] = true;
174:   }
175:   // barrier is always blocking
176:   torch_ucc_config.blocking_wait[(std::uint8_t)OpType::BARRIER] = true;
177: 
178:   torch_ucc_config.use_future =
179:       std::stoi(torch_ucc_envs_map.at("TORCH_UCC_USE_FUTURE"));
180:   torch_ucc_config.shared_comm =
181:       std::stoi(torch_ucc_envs_map.at("TORCH_UCC_SHARED_COMM"));
182:   torch_ucc_config.use_allgatherv =
183:       std::stoi(torch_ucc_envs_map.at("TORCH_UCC_USE_ALLGATHERV"));
184:   torch_ucc_config.enable_health_check =
185:       std::stoi(torch_ucc_envs_map.at("TORCH_UCC_ENABLE_HEALTH_CHECK"));
186:   torch_ucc_config.enable_comms_logger =
187:       std::stoi(torch_ucc_envs_map.at("TORCH_UCC_ENABLE_COMMS_LOGGER"));
188: }
189: 
190: void check_device(c10::Device dev1, c10::Device dev2) {
191:   if (dev1.is_cuda() && dev2.is_cuda() && dev1 != dev2) {
192:     throw std::invalid_argument("ProcessGroupUCC multidevice is not supported");
```

- EN: Lines 169-192 introduces executable logic in routines such as `check_device`; performs validation and error handling to keep distributed state consistent.
- CN: 第 169-192 行在 `check_device` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 193-216 / 第 193-216 行

```cpp
193:   }
194: }
195: 
196: void check_tensor(const std::vector<at::Tensor>& tensors) {
197:   if (tensors.size() != 1) {
198:     throw std::invalid_argument(
199:         "ProcessGroupUCC takes 1 tensor. Got " +
200:         std::to_string(tensors.size()) + ". ");
201:   }
202:   if (!tensors[0].is_contiguous()) {
203:     throw std::invalid_argument(
204:         "ProcessGroupUCC input tensor has to be contiguous");
205:   }
206:   if (tensors[0].is_sparse()) {
207:     throw std::invalid_argument("ProcessGroupUCC input tensor has to be dense");
208:   }
209:   // TODO: check cuda case
210: }
211: 
212: ProcessGroupUCC::WorkUCC::~WorkUCC() {
213: #ifdef USE_CUDA
214:   if (fence && ep) {
215:     std::lock_guard<std::mutex> lock(ep->event_pool_mutex);
216:     ep->event_pool.push(std::move(fence));
```

- EN: Lines 193-216 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `check_tensor`, `lock`; performs validation and error handling to keep distributed state consistent.
- CN: 第 193-216 行使用条件编译来适配特性开关、平台或可选后端；在 `check_tensor`、`lock` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 217-240 / 第 217-240 行

```cpp
217:   }
218: #endif
219: }
220: 
221: void ProcessGroupUCC::WorkUCC::setException() {
222:   if (exception() || !entry_) {
223:     return;
224:   }
225:   exception_ = entry_->eptr_;
226: }
227: 
228: void ProcessGroupUCC::WorkUCC::setAndThrowException() {
229:   setException();
230:   if (exception()) {
231:     std::rethrow_exception(exception());
232:   }
233: }
234: 
235: bool ProcessGroupUCC::WorkUCC::isCompleted() {
236:   if (!entry_) {
237:     return true;
238:   }
239:   setException();
240:   // status_ <= 0 to avoid listing all possible status codes.  The main thread
```

- EN: Lines 217-240 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `setException`.
- CN: 第 217-240 行使用条件编译来适配特性开关、平台或可选后端；在 `setException` 等例程中引入具体执行逻辑。

### Lines 241-264 / 第 241-264 行

```cpp
241:   // needs to be unblocked when UCC (in progress thread) returns success (== 0)
242:   // or any error code (< 0).
243:   return exception() || entry_->status_ <= 0;
244: }
245: 
246: bool ProcessGroupUCC::WorkUCC::isSuccess() const {
247:   if (!entry_) {
248:     return true;
249:   }
250:   return !exception() && entry_->status_ == 0;
251: }
252: 
253: bool ProcessGroupUCC::WorkUCC::wait(std::chrono::milliseconds /* unused */) {
254:   if (torch_ucc_config.enable_comms_logger && logger_) {
255:     logger_->trace_generator->recordComms("wait", (uintptr_t)this, rank_);
256:   }
257: #ifdef USE_CUDA
258:   if (fence && !torch_ucc_config.blocking_wait[(int)opType_]) {
259:     // block user stream
260:     setAndThrowException();
261:     fence->block(at::cuda::getCurrentCUDAStream());
262:     return true;
263:   }
264: #endif
```

- EN: Lines 241-264 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `setAndThrowException`.
- CN: 第 241-264 行使用条件编译来适配特性开关、平台或可选后端；在 `setAndThrowException` 等例程中引入具体执行逻辑。

### Lines 265-288 / 第 265-288 行

```cpp
265:   // wait for complete.  For blocking case, the main thread will be blocked in
266:   // this loop until the progress thread changes the status of this request.
267:   // If timeout occurs, UCC will return UCC_ERR_TIMEOUT as the status.  The
268:   // main thread will throw out the exception then. There is no "abort"
269:   // function in UCC currently.
270:   while (!isCompleted())
271:     ;
272:   setAndThrowException();
273:   // manually call profiling end callbacks if they are set,
274:   // since progress thread does not own WorkUCC
275:   if (Work::recordFunctionEndCallback_) {
276:     Work::recordFunctionEndCallback_();
277:     Work::recordFunctionEndCallback_ = nullptr;
278:   }
279:   if (c10d::allow_inflight_collective_as_graph_input()) {
280:     c10d::unregister_work(
281:         c10::intrusive_ptr<
282:             ProcessGroupUCC::WorkUCC>::unsafe_reclaim_from_nonowning(this));
283:   }
284:   return true;
285: }
286: 
287: c10::intrusive_ptr<c10::ivalue::Future> ProcessGroupUCC::WorkUCC::getFuture() {
288:   return future_;
```

- EN: Lines 265-288 introduces executable logic in routines such as `setAndThrowException`; performs validation and error handling to keep distributed state consistent.
- CN: 第 265-288 行在 `setAndThrowException` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 289-312 / 第 289-312 行

```cpp
289: }
290: 
291: int ProcessGroupUCC::WorkUCC::sourceRank() const {
292:   if (opType_ != OpType::RECV && opType_ != OpType::RECVANYSOURCE) {
293:     // Throw an error
294:     return Work::sourceRank();
295:   }
296:   return sourceRank_;
297: }
298: 
299: std::vector<at::Tensor> ProcessGroupUCC::WorkUCC::result() {
300:   return *outputs_;
301: }
302: 
303: void ProcessGroupUCC::ProgressEntry::finalize(std::exception_ptr eptr) {
304:   ucc_status_t status = UCC_OK;
305: 
306:   if (request_ != nullptr) {
307:     status = request_->status;
308:     comm_->free_request(request_);
309:   }
310:   if (eptr) {
311:     eptr_ = eptr;
312:   } else {
```

- EN: Lines 289-312 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 289-312 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 313-336 / 第 313-336 行

```cpp
313:     status_ = status;
314:   }
315:   if (future_) {
316:     if (eptr) {
317:       future_->setError(eptr);
318:     } else {
319:       future_->markCompleted(
320:           c10::IValue(data ? data->dst : std::vector<at::Tensor>()));
321:     }
322:   }
323: }
324: 
325: Comm::Comm(
326:     const c10::intrusive_ptr<ProcessGroupUCCLogger>& logger_,
327:     std::shared_ptr<torch_ucc_oob_coll_info_t> oob_,
328:     c10::Device dev,
329:     bool is_health_check)
330:     : logger(logger_),
331:       oob(oob_),
332:       ucc_comm(oob, logger),
333:       finalize_phase(
334:           is_health_check ? TORCH_UCC_HEALTH_CHECK : TORCH_UCC_FINALIZE),
335:       cuda_device_index(TORCH_UCC_DEVICE_NOT_SET) {
336:   if (dev.is_cuda()) {
```

- EN: Lines 313-336 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 313-336 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 337-360 / 第 337-360 行

```cpp
337:     cuda_device_index = dev.index();
338:   }
339:   stop_progress_loop = false;
340:   collective_inprogress = false;
341:   progress_thread = std::thread(&Comm::progress_loop, this);
342: #ifdef _GNU_SOURCE
343:   pthread_setname_np(progress_thread.native_handle(), "ucc-progress");
344: #endif
345: }
346: 
347: Comm::~Comm() {
348:   std::unique_lock<std::mutex> lock(mutex);
349:   queue_consume_cv.wait(
350:       lock, [&] { return progress_queue.empty() && !collective_inprogress; });
351:   stop_progress_loop = true;
352:   lock.unlock();
353:   queue_produce_cv.notify_all();
354:   progress_thread.join();
355: }
356: 
357: std::shared_ptr<Comm> Comm::get_comm(
358:     uint32_t& id,
359:     c10::Device dev,
360:     std::shared_ptr<torch_ucc_oob_coll_info_t> oob,
```

- EN: Lines 337-360 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 337-360 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 361-384 / 第 361-384 行

```cpp
361:     const c10::intrusive_ptr<ProcessGroupUCCLogger>& logger,
362:     bool is_health_check) {
363:   static std::mutex m;
364:   static std::weak_ptr<Comm> comm;
365:   static uint32_t comm_id;
366: 
367:   std::lock_guard<std::mutex> lock(m);
368:   id = comm_id;
369: 
370:   std::string group_id = "group_id";
371:   if (is_health_check) {
372:     group_id = c10::str(dev.type()) + "/" + group_id;
373:   }
374: 
375:   std::vector<uint8_t> remote_comm_id;
376:   oob->store->deleteKey(group_id + std::to_string(0));
377:   if (oob->rank != 0) {
378:     std::vector<uint8_t> val = std::vector<uint8_t>(
379:         reinterpret_cast<uint8_t*>(&id),
380:         reinterpret_cast<uint8_t*>(&id) + sizeof(id));
381:     oob->store->set(group_id + std::to_string(oob->rank), val);
382:   } else {
383:     for (int i = 1; i < oob->size; i++) {
384:       remote_comm_id = oob->store->get(group_id + std::to_string(i));
```

- EN: Lines 361-384 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 361-384 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 385-408 / 第 385-408 行

```cpp
385:       oob->store->deleteKey(group_id + std::to_string(i));
386:       // Find the highest id.
387:       id = std::max(id, *(reinterpret_cast<uint32_t*>(remote_comm_id.data())));
388:     }
389:     std::vector<uint8_t> val = std::vector<uint8_t>(
390:         reinterpret_cast<uint8_t*>(&id),
391:         reinterpret_cast<uint8_t*>(&id) + sizeof(id));
392:     oob->store->set(group_id + std::to_string(oob->rank), val);
393:   }
394:   remote_comm_id = oob->store->get(group_id + std::to_string(0));
395:   oob->comm_id = *(reinterpret_cast<uint32_t*>(remote_comm_id.data()));
396:   // Prepare comm_id (static variable) to the next id.
397:   comm_id = oob->comm_id + 1;
398: 
399:   if (torch_ucc_config.shared_comm) {
400:     std::shared_ptr<Comm> shared_comm = comm.lock();
401:     if (!shared_comm) {
402:       shared_comm = std::make_shared<Comm>(logger, oob, dev, is_health_check);
403:       comm = shared_comm;
404:     } else {
405:       if (dev.is_cuda() && !is_health_check) {
406:         if ((shared_comm->cuda_device_index != TORCH_UCC_DEVICE_NOT_SET) &&
407:             (shared_comm->cuda_device_index != dev.index())) {
408:           TORCH_UCC_LOG_ERROR(
```

- EN: Lines 385-408 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 385-408 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 409-432 / 第 409-432 行

```cpp
409:               is_health_check ? TORCH_UCC_HEALTH_CHECK : TORCH_UCC_INIT,
410:               "ucc communicator was initialized with different cuda device,"
411:               "multi device is not supported");
412:           throw std::invalid_argument(ucc_status_string(UCC_ERR_NOT_SUPPORTED));
413:         }
414:         shared_comm->cuda_device_index = dev.index();
415:       }
416:     }
417:     return shared_comm;
418:   } else {
419:     return std::make_shared<Comm>(logger, oob, dev, is_health_check);
420:   }
421: }
422: 
423: void Comm::ucc_create_team(
424:     ucc_team_h& team,
425:     std::shared_ptr<torch_ucc_oob_coll_info_t> oob) {
426:   ucc_status_t st;
427:   ucc_team_params_t team_params;
428:   team_params.mask = UCC_TEAM_PARAM_FIELD_EP | UCC_TEAM_PARAM_FIELD_EP_RANGE |
429:       UCC_TEAM_PARAM_FIELD_OOB;
430:   team_params.oob.allgather = oob_allgather;
431:   team_params.oob.req_test = oob_allgather_test;
432:   team_params.oob.req_free = oob_allgather_free;
```

- EN: Lines 409-432 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 409-432 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 433-456 / 第 433-456 行

```cpp
433:   team_params.oob.coll_info = oob.get();
434:   team_params.oob.n_oob_eps = oob->size;
435:   team_params.oob.oob_ep = oob->rank;
436:   team_params.ep = oob->rank;
437:   team_params.ep_range = UCC_COLLECTIVE_EP_RANGE_CONTIG;
438:   TORCH_UCC_CHECK(
439:       ucc_team_create_post(&ucc_comm.context, 1, &team_params, &team),
440:       "failed to post team create");
441:   do {
442:     st = ucc_team_create_test(team);
443:     ucc_context_progress(ucc_comm.context);
444:   } while (st == UCC_INPROGRESS);
445:   TORCH_UCC_CHECK(st, "failed to create UCC team");
446: }
447: 
448: void Comm::ucc_destroy_team(ucc_team_h& team) {
449:   std::unique_lock<std::mutex> lock(mutex);
450:   queue_consume_cv.wait(
451:       lock, [&] { return progress_queue.empty() && !collective_inprogress; });
452: 
453:   ucc_status_t status;
454:   while (UCC_INPROGRESS == (status = ucc_team_destroy(team))) {
455:     if (UCC_OK != status) {
456:       TORCH_UCC_LOG_ERROR(
```

- EN: Lines 433-456 introduces executable logic in routines such as `TORCH_UCC_CHECK`; returns computed state or forwards results to the surrounding caller.
- CN: 第 433-456 行在 `TORCH_UCC_CHECK` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 457-480 / 第 457-480 行

```cpp
457:           finalize_phase,
458:           c10::str("ucc team destroy error: ", ucc_status_string(status)));
459:       break;
460:     }
461:   }
462: 
463:   lock.unlock();
464: }
465: 
466: void Comm::enqueue_collective(
467:     std::unique_ptr<ProcessGroupUCC::WorkData> data,
468:     c10::intrusive_ptr<ProcessGroupUCC::WorkUCC> work,
469:     ucc_coll_args_t& coll,
470:     ucc_team_h team) {
471:   ucc_coll_req_h request;
472:   TORCH_UCC_CHECK(
473:       ucc_collective_init(&coll, &request, team), "failed to init collective");
474:   TORCH_UCC_CHECK_REQUEST(
475:       request, ucc_collective_post(request), "failed to post collective");
476: 
477:   auto entry =
478:       std::make_shared<ProcessGroupUCC::ProgressEntry>(&ucc_comm, request);
479:   entry->data = std::move(data);
480:   entry->future_ = work->getFuture();
```

- EN: Lines 457-480 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 457-480 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 481-504 / 第 481-504 行

```cpp
481:   work->entry_ = entry;
482:   std::unique_lock<std::mutex> lock(mutex);
483:   progress_queue.push_back(entry);
484:   lock.unlock();
485:   queue_produce_cv.notify_one();
486: }
487: 
488: #ifdef USE_CUDA
489: void Comm::enqueue_cuda_collective(
490:     std::unique_ptr<ProcessGroupUCC::WorkData> data,
491:     c10::intrusive_ptr<ProcessGroupUCC::WorkUCC> work,
492:     ucc_coll_args_t& coll,
493:     ucc_team_h team,
494:     ucc_ee_h ee) {
495:   ucc_coll_req_h request;
496:   TORCH_UCC_CHECK(
497:       ucc_collective_init(&coll, &request, team),
498:       "failed to init cuda collective");
499:   ucc_ev_t comp_ev, *post_ev;
500:   comp_ev.ev_type = UCC_EVENT_COMPUTE_COMPLETE;
501:   comp_ev.ev_context = nullptr;
502:   comp_ev.ev_context_size = 0;
503:   comp_ev.req = request;
504:   TORCH_UCC_CHECK_REQUEST(
```

- EN: Lines 481-504 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 481-504 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 505-528 / 第 505-528 行

```cpp
505:       request,
506:       ucc_collective_triggered_post(ee, &comp_ev),
507:       "failed to post triggered collective");
508:   ucc_status_t st = ucc_ee_get_event(ee, &post_ev);
509:   TORCH_CHECK(st == UCC_OK && post_ev->ev_type == UCC_EVENT_COLLECTIVE_POST);
510:   ucc_ee_ack_event(ee, post_ev);
511:   auto entry =
512:       std::make_shared<ProcessGroupUCC::ProgressEntry>(&ucc_comm, request);
513:   entry->data = std::move(data);
514:   work->entry_ = entry;
515:   std::unique_lock<std::mutex> lock(mutex);
516:   progress_queue.push_back(entry);
517:   lock.unlock();
518:   queue_produce_cv.notify_one();
519: }
520: #endif
521: 
522: void Comm::progress_loop() {
523:   std::unique_lock<std::mutex> lock(mutex);
524: #ifdef USE_CUDA
525:   bool device_set = false;
526: #endif
527:   while (!stop_progress_loop) {
528:     if (progress_queue.empty()) {
```

- EN: Lines 505-528 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 505-528 行使用条件编译来适配特性开关、平台或可选后端；在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 529-552 / 第 529-552 行

```cpp
529:       queue_produce_cv.wait(lock);
530:       continue;
531:     }
532:     collective_inprogress = true;
533:     auto work = progress_queue.front();
534:     progress_queue.pop_front();
535:     lock.unlock();
536: #ifdef USE_CUDA
537:     if ((!device_set) && (cuda_device_index != TORCH_UCC_DEVICE_NOT_SET)) {
538:       c10::cuda::set_device(cuda_device_index);
539:       CUcontext pctx = nullptr;
540:       at::globalContext().getNVRTC().cuCtxGetCurrent(&pctx);
541:       if (C10_UNLIKELY(!pctx)) {
542:         at::globalContext().getNVRTC().cuDevicePrimaryCtxRetain(
543:             &pctx, cuda_device_index);
544:         at::globalContext().getNVRTC().cuCtxSetCurrent(pctx);
545:       }
546:       device_set = true;
547:     }
548: #endif
549:     std::exception_ptr eptr;
550:     try {
551:       while (work->request_->status > 0) {
552:         ucc_comm.progress();
```

- EN: Lines 529-552 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 529-552 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 553-576 / 第 553-576 行

```cpp
553:       }
554:       if (work->request_->status < 0) {
555:         eptr = std::make_exception_ptr(
556:             std::runtime_error(ucc_status_string(work->request_->status)));
557:         std::string err_log = c10::str(
558:             "Failed to progress communication", // TODO: report exact op type or
559:                                                 // id?
560:             ucc_status_string(work->request_->status));
561:         TORCH_UCC_LOG_ERROR(TORCH_UCC_COLL_PROGRESS, err_log);
562:       }
563:     } catch (...) {
564:       eptr = std::current_exception();
565:     }
566:     work->finalize(eptr);
567:     work = nullptr;
568:     collective_inprogress = false;
569:     queue_consume_cv.notify_one();
570:     lock.lock();
571:   }
572: }
573: 
574: ProcessGroupUCC::ProcessGroupUCC(
575:     const c10::intrusive_ptr<Store>& store,
576:     int rank,
```

- EN: Lines 553-576 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 553-576 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 577-600 / 第 577-600 行

```cpp
577:     int size,
578:     std::chrono::duration<float> timeout)
579:     : Backend(rank, size), timeout_(timeout) {
580:   c10::call_once(torch_ucc_config.flag, read_config);
581:   oob = std::make_shared<torch_ucc_oob_coll_info_t>();
582:   oob->rank = rank;
583:   oob->size = size;
584:   oob->store = store;
585:   comm = nullptr;
586:   cuda_ee = nullptr;
587:   static uint32_t id = 0;
588:   uint32_t pg_id = id++;
589: 
590:   logger = c10::make_intrusive<ProcessGroupUCCLogger>(
591:       c10::str("[Rank ", rank_, "]", "[ProcessGroupUCC-", pg_id, "]"),
592:       TORCH_UCC_INIT);
593:   TORCH_UCC_LOG_INFO(
594:       TORCH_UCC_INIT,
595:       c10::str(
596:           "Created ProcessGroupUCC with ",
597:           size,
598:           " ranks, with timeout ",
599:           timeout_.count(),
600:           " secs"));
```

- EN: Lines 577-600 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 577-600 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 601-624 / 第 601-624 行

```cpp
601:   std::string envs = "";
602:   for (auto& torch_ucc_env : torch_ucc_envs_map) {
603:     envs += ("\n\t" + torch_ucc_env.first + "=" + torch_ucc_env.second);
604:   }
605:   TORCH_UCC_LOG_INFO(
606:       TORCH_UCC_INIT,
607:       c10::str(
608:           "Successfully read and set ProcessGroupUCC env. variables as followings",
609:           envs));
610: 
611:   if (torch_ucc_config.enable_health_check) {
612:     // Perform health check by initializing dummy communicators and destroying
613:     // them. This will help indicate any UCC/UCX-related issues prior to the
614:     // first collective. Run it in a separate thread and wait on CV to handle
615:     // timeouts so that if there are hangs, the main thread can still run
616:     // correctly.
617:     runHealthCheck();
618:   }
619:   if (torch_ucc_config.enable_comms_logger) {
620:     logger->initCommsTracer();
621:   }
622: }
623: 
624: ProcessGroupUCC::~ProcessGroupUCC() {
```

- EN: Lines 601-624 introduces executable logic in routines such as `runHealthCheck`.
- CN: 第 601-624 行在 `runHealthCheck` 等例程中引入具体执行逻辑。

### Lines 625-648 / 第 625-648 行

```cpp
625:   if (torch_ucc_config.enable_comms_logger) {
626:     logger->flushComms(this->getRank(), this->getSize());
627:   }
628:   if (comm) {
629:     logger->setPhase(TORCH_UCC_FINALIZE);
630:     comm->ucc_destroy_team(team);
631:     TORCH_UCC_LOG_INFO(
632:         TORCH_UCC_FINALIZE, "Successfully destroyed UCC library");
633:     try {
634:       if (cuda_ee) {
635:         ucc_ee_destroy(cuda_ee);
636:         ucc_ee_destroy(cuda_ee_p2p[0]);
637:         ucc_ee_destroy(cuda_ee_p2p[1]);
638:       }
639:     } catch (std::exception& ex) {
640:       TORCH_UCC_LOG_INFO(
641:           TORCH_UCC_FINALIZE,
642:           c10::str(
643:               "(~ProcessGroupUCC) Caught error in Store Operation .. ",
644:               "[",
645:               ex.what(),
646:               "]"));
647:     }
648:     comm = nullptr;
```

- EN: Lines 625-648 introduces executable logic in routines such as `TORCH_UCC_LOG_INFO`, `ucc_ee_destroy`.
- CN: 第 625-648 行在 `TORCH_UCC_LOG_INFO`、`ucc_ee_destroy` 等例程中引入具体执行逻辑。

### Lines 649-672 / 第 649-672 行

```cpp
649:   }
650: }
651: 
652: #ifdef USE_CUDA
653: // Return CUDA device with ordinal given by input rank.
654: c10::Device getCUDADeviceForRank(int rank) {
655:   TORCH_CHECK(rank >= 0, "Invalid rank ", rank);
656:   auto numGPUs = at::cuda::getNumGPUs();
657:   auto deviceIdx = static_cast<c10::DeviceIndex>(rank % numGPUs);
658:   return c10::Device(c10::DeviceType::CUDA, deviceIdx);
659: }
660: #endif
661: 
662: void ProcessGroupUCC::runHealthCheck() {
663:   // Run health check in a separate thread and wait on CV to handle timeouts.
664:   // This design allows us to handle hangs.
665: 
666:   // When size_ is 1, there is no need to do any communication at all.
667:   if (size_ == 1)
668:     return;
669: 
670:   struct HealthCheckData {
671:     std::mutex healthCheckMutex;
672:     std::condition_variable healthCheckCv;
```

- EN: Lines 649-672 uses conditional compilation to adapt to feature flags, platforms, or optional backends; declares or defines types such as `HealthCheckData`; introduces executable logic in routines such as `getCUDADeviceForRank`, `TORCH_CHECK`.
- CN: 第 649-672 行使用条件编译来适配特性开关、平台或可选后端；声明或定义了 `HealthCheckData` 等类型；在 `getCUDADeviceForRank`、`TORCH_CHECK` 等例程中引入具体执行逻辑。

### Lines 673-696 / 第 673-696 行

```cpp
673:     bool uccHealthCheckSuccess = false;
674:     std::exception_ptr healthCheckException;
675:   } healthCheckData;
676: 
677:   auto t = std::thread([&healthCheckData, this]() {
678:     std::list<c10::Device> devices{c10::kCPU};
679: #ifdef USE_CUDA
680:     c10::cuda::OptionalCUDAGuard gpuGuard;
681:     if (at::cuda::is_available()) {
682:       devices.emplace_front(getCUDADeviceForRank(rank_));
683:     }
684: #endif
685:     for (auto device : devices) {
686:       bool is_last_device = (device == devices.back());
687:       try {
688:         auto oob = std::make_shared<torch_ucc_oob_coll_info_t>();
689:         oob->rank = this->oob->rank;
690:         oob->size = this->oob->size;
691:         oob->store = this->oob->store;
692:         ucc_team_h team = nullptr;
693:         uint32_t comm_id;
694: #ifdef USE_CUDA
695:         if (device.is_cuda()) {
696:           gpuGuard.set_index(device.index());
```

- EN: Lines 673-696 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 673-696 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 697-720 / 第 697-720 行

```cpp
697:         }
698: #endif
699:         auto comm = Comm::get_comm(comm_id, device, oob, logger, true);
700:         comm->ucc_create_team(team, oob);
701:         comm->ucc_destroy_team(team);
702:         TORCH_UCC_LOG_INFO(
703:             TORCH_UCC_HEALTH_CHECK,
704:             c10::str(
705:                 "UCC library health check succeed for device ",
706:                 c10::DeviceTypeName(device.type())));
707:         // Mark ucc health check as complete.
708:         if (is_last_device) {
709:           std::lock_guard<std::mutex> lk(healthCheckData.healthCheckMutex);
710:           healthCheckData.uccHealthCheckSuccess = true;
711:         }
712: 
713:         comm = nullptr;
714:         oob = nullptr;
715:         // Notify main thread the health check is complete.
716:         if (is_last_device) {
717:           healthCheckData.healthCheckCv.notify_one();
718:         }
719:       } catch (const std::exception&) {
720:         // Populate exception ptr.
```

- EN: Lines 697-720 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 697-720 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 721-744 / 第 721-744 行

```cpp
721:         healthCheckData.healthCheckException = std::current_exception();
722:         // Unblock waiting main thread which will report exception.
723:         healthCheckData.healthCheckCv.notify_one();
724:       } // Unknown exceptions will just cause the program to terminate.
725:     }
726:   });
727:   // We don't need to join the thread, just need to verify health check via the
728:   // CV. Hence we detach the thread here.
729:   t.detach(); // NOLINT
730:   TORCH_UCC_LOG_INFO(
731:       TORCH_UCC_HEALTH_CHECK,
732:       c10::str(
733:           "will wait up to ",
734:           timeout_.count(),
735:           " msec for UCC health check to complete."));
736:   std::unique_lock<std::mutex> lock(healthCheckData.healthCheckMutex);
737:   healthCheckData.healthCheckCv.wait_for(lock, timeout_, [&healthCheckData]() {
738:     return healthCheckData.uccHealthCheckSuccess;
739:   });
740: 
741:   if (healthCheckData.healthCheckException) {
742:     std::rethrow_exception(healthCheckData.healthCheckException);
743:   }
744:   // If there is no exception, the likely culprit is a timeout/hang
```

- EN: Lines 721-744 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 721-744 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 745-768 / 第 745-768 行

```cpp
745:   TORCH_CHECK(
746:       healthCheckData.uccHealthCheckSuccess,
747:       "ProcessGroupUCC: Health check failure: Failed to initialize UCC on rank ",
748:       rank_);
749: }
750: 
751: void ProcessGroupUCC::set_timeout(ucc_coll_args_t& args) {
752:   args.mask |= UCC_COLL_ARGS_FIELD_FLAGS;
753:   args.flags |= UCC_COLL_ARGS_FLAG_TIMEOUT;
754:   args.timeout = timeout_.count();
755: }
756: 
757: #ifdef USE_CUDA
758: std::unique_ptr<at::cuda::CUDAEvent> ProcessGroupUCC::getPooledEvent() {
759:   std::unique_ptr<at::cuda::CUDAEvent> ev;
760:   std::lock_guard<std::mutex> lock(ep.event_pool_mutex);
761:   if (ep.event_pool.empty()) {
762:     ev = std::make_unique<at::cuda::CUDAEvent>();
763:   } else {
764:     ev = std::move(ep.event_pool.front());
765:     ep.event_pool.pop();
766:   }
767:   return ev;
768: }
```

- EN: Lines 745-768 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 745-768 行使用条件编译来适配特性开关、平台或可选后端；在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 769-792 / 第 769-792 行

```cpp
769: #endif
770: 
771: template <typename PreProcess, typename PostProcess>
772: c10::intrusive_ptr<Work> ProcessGroupUCC::collective_post(
773:     OpType opType,
774:     PreProcess preproc,
775:     PostProcess postproc,
776:     ucc_coll_args_t& coll,
777:     std::unique_ptr<ProcessGroupUCC::WorkData> data,
778:     c10::Device dev,
779:     std::vector<at::Tensor>& inputTensors,
780:     std::vector<at::Tensor>& outputTensors,
781:     const char* prof_title) {
782:   seq_++;
783:   set_timeout(coll);
784:   auto work = c10::make_intrusive<ProcessGroupUCC::WorkUCC>(
785:       opType, seq_, prof_title, inputTensors, logger);
786: 
787:   if (opType == OpType::RECV) {
788:     work->sourceRank_ = coll.root;
789:   }
790: 
791:   RECORD_COMMS_TRACE(
792:       logger->trace_generator,
```

- EN: Lines 769-792 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 769-792 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 793-816 / 第 793-816 行

```cpp
793:       work,
794:       opType,
795:       this->getRank(),
796:       this->getSize(),
797:       inputTensors,
798:       outputTensors);
799: 
800:   // Store references to outputs to be used by result
801:   work->outputs_ = std::make_shared<std::vector<at::Tensor>>(outputTensors);
802:   switch (dev.type()) {
803:     case c10::DeviceType::CPU: {
804:       if (torch_ucc_config.use_future) {
805:         work->future_ = c10::make_intrusive<at::ivalue::Future>(
806:             c10::ListType::create(c10::TensorType::get()));
807:       }
808:       preproc();
809:       comm->enqueue_collective(std::move(data), work, coll, team);
810:       postproc();
811:       return work;
812:     }
813: #ifdef USE_CUDA
814:     case c10::DeviceType::CUDA: {
815:       auto cuda_ev = getPooledEvent();
816:       at::cuda::CUDAStream* op_stream;
```

- EN: Lines 793-816 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `preproc`, `postproc`.
- CN: 第 793-816 行使用条件编译来适配特性开关、平台或可选后端；在 `preproc`、`postproc` 等例程中引入具体执行逻辑。

### Lines 817-840 / 第 817-840 行

```cpp
817:       ucc_ee_h* op_ee;
818:       if (opType == OpType::SEND) {
819:         op_stream = stream_p2p[0].get();
820:         op_ee = &cuda_ee_p2p[0];
821:       } else if (opType == OpType::RECV) {
822:         op_stream = stream_p2p[1].get();
823:         op_ee = &cuda_ee_p2p[1];
824:       } else {
825:         op_stream = stream.get();
826:         op_ee = &cuda_ee;
827:       }
828: 
829:       cuda_ev->record(at::cuda::getCurrentCUDAStream(dev.index()));
830:       cuda_ev->block(*op_stream);
831:       at::cuda::CUDAStreamGuard guard(*op_stream);
832:       preproc();
833:       comm->enqueue_cuda_collective(std::move(data), work, coll, team, *op_ee);
834:       postproc();
835:       cuda_ev->record(*op_stream);
836:       work->fence = std::move(cuda_ev);
837:       work->ep = &ep;
838:       if (torch_ucc_config.use_future) {
839:         c10::cuda::CUDAMultiStreamGuard streamGuard(*op_stream);
840:         std::vector<c10::Device> devList{dev};
```

- EN: Lines 817-840 introduces executable logic in routines such as `guard`, `preproc`, `postproc`.
- CN: 第 817-840 行在 `guard`、`preproc`、`postproc` 等例程中引入具体执行逻辑。

### Lines 841-864 / 第 841-864 行

```cpp
841:         work->future_ = c10::make_intrusive<at::ivalue::Future>(
842:             c10::ListType::create(c10::TensorType::get()), devList);
843:         // Add a callback that runs profiling end callbacks
844:         if (work->recordFunctionEndCallback_) {
845:           work->future_->addCallback([work](at::ivalue::Future& /* unused */) {
846:             work->recordFunctionEndCallback_();
847:           });
848:         }
849: 
850:         work->future_->markCompleted(c10::IValue(outputTensors));
851:       }
852:       return work;
853:     }
854: #endif // #ifdef USE_CUDA
855:     default: {
856:       TORCH_UCC_LOG_ERROR(
857:           TORCH_UCC_COLL_POST, c10::str("unsupported device type ", dev.str()));
858:       throw std::invalid_argument(ucc_status_string(UCC_ERR_NOT_SUPPORTED));
859:     }
860:   }
861: }
862: 
863: c10::intrusive_ptr<Work> ProcessGroupUCC::allgather(
864:     std::vector<std::vector<at::Tensor>>& outputTensors,
```

- EN: Lines 841-864 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 841-864 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 865-888 / 第 865-888 行

```cpp
865:     std::vector<at::Tensor>& inputTensors,
866:     const AllgatherOptions& /* unused */) {
867:   auto& tensor = inputTensors[0];
868:   check_device(tensor.device(), outputTensors[0][0].device());
869:   initComm(tensor.device());
870: 
871:   if (tensor.device().is_cpu() || torch_ucc_config.use_allgatherv) {
872:     AllgathervWorkData* data = new AllgathervWorkData(size_);
873:     for (int i = 0; i < size_; i++) {
874:       data->recv_lengths[i] = tensor.element_size() * tensor.numel();
875:       data->recv_offsets[i] = (uint64_t)outputTensors[0][i].data_ptr();
876:     }
877:     ucc_coll_args_t coll;
878:     coll.mask = UCC_COLL_ARGS_FIELD_FLAGS;
879:     coll.flags =
880:         UCC_COLL_ARGS_FLAG_COUNT_64BIT | UCC_COLL_ARGS_FLAG_DISPLACEMENTS_64BIT;
881:     coll.coll_type = UCC_COLL_TYPE_ALLGATHERV;
882:     coll.src.info.buffer = tensor.data_ptr();
883:     coll.src.info.count = tensor.element_size() * tensor.numel();
884:     coll.src.info.datatype = UCC_DT_UINT8;
885:     coll.src.info.mem_type = to_ucc_memType(tensor.device().type());
886:     coll.dst.info_v.buffer = nullptr;
887:     coll.dst.info_v.counts = (ucc_count_t*)data->recv_lengths.data();
888:     coll.dst.info_v.displacements = (ucc_aint_t*)data->recv_offsets.data();
```

- EN: Lines 865-888 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 865-888 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 889-912 / 第 889-912 行

```cpp
889:     coll.dst.info_v.datatype = UCC_DT_UINT8;
890:     coll.dst.info_v.mem_type =
891:         to_ucc_memType(outputTensors[0][0].device().type());
892:     SAVE_TENSORS(inputTensors, data->src);
893:     SAVE_TENSORS(outputTensors[0], data->dst);
894: 
895:     return collective_post(
896:         OpType::ALLGATHER,
897:         []() {},
898:         []() {},
899:         coll,
900:         std::unique_ptr<WorkData>(data),
901:         tensor.device(),
902:         inputTensors,
903:         outputTensors[0],
904:         "ucc:all_gather");
905:   } else {
906:     WorkData* data = new WorkData();
907:     std::vector<at::Tensor> flat_output(outputTensors.size());
908:     for (size_t i = 0; i < outputTensors.size(); i++) {
909:       TORCH_CHECK(
910:           outputTensors[i].size() == outputTensors.size() * size_,
911:           "Tensor output list is not valid for the number of participants");
912:       flat_output[i] = c10d::newLikeFlat(outputTensors, i);
```

- EN: Lines 889-912 introduces executable logic in routines such as `SAVE_TENSORS`; performs validation and error handling to keep distributed state consistent.
- CN: 第 889-912 行在 `SAVE_TENSORS` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 913-936 / 第 913-936 行

```cpp
913:     }
914:     SAVE_TENSORS(flat_output, data->flat);
915:     ucc_coll_args_t coll;
916:     coll.mask = 0;
917:     coll.flags = 0;
918:     coll.coll_type = UCC_COLL_TYPE_ALLGATHER;
919:     coll.src.info.buffer = tensor.data_ptr();
920:     coll.src.info.count = tensor.numel();
921:     coll.src.info.datatype = to_ucc_dType(tensor);
922:     coll.src.info.mem_type = to_ucc_memType(tensor.device().type());
923:     coll.dst.info.buffer = flat_output[0].data_ptr();
924:     coll.dst.info.count = flat_output[0].numel();
925:     coll.dst.info.datatype = to_ucc_dType(flat_output[0]);
926:     coll.dst.info.mem_type =
927:         to_ucc_memType(outputTensors[0][0].device().type());
928: 
929:     auto copy_from_flat = [&] {
930:       bool asyncCopy = false;
931: #ifdef USE_CUDA
932:       bool isCuda = outputTensors[0][0].device().is_cuda();
933:       ;
934: #endif
935:       for (size_t i = 0; i < outputTensors.size(); i++) {
936:         auto inumel = inputTensors[i].numel();
```

- EN: Lines 913-936 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `SAVE_TENSORS`.
- CN: 第 913-936 行使用条件编译来适配特性开关、平台或可选后端；在 `SAVE_TENSORS` 等例程中引入具体执行逻辑。

### Lines 937-960 / 第 937-960 行

```cpp
937:         for (size_t j = 0; j < outputTensors[i].size(); j++) {
938:           TORCH_CHECK(
939:               (outputTensors[i][j].numel() == inumel),
940:               "Tensor operand counts must be same");
941: #ifdef USE_CUDA
942:           if (isCuda) {
943:             c10::cuda::CUDACachingAllocator::recordStream(
944:                 outputTensors[i][j].storage().data_ptr(), (*stream));
945:             asyncCopy = true;
946:           }
947: #endif
948:           outputTensors[i][j].copy_(flat_output[i][j], asyncCopy);
949:         }
950:       }
951:     };
952:     return collective_post(
953:         OpType::ALLGATHER,
954:         []() {},
955:         copy_from_flat,
956:         coll,
957:         std::unique_ptr<WorkData>(data),
958:         tensor.device(),
959:         inputTensors,
960:         outputTensors[0],
```

- EN: Lines 937-960 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 937-960 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 961-984 / 第 961-984 行

```cpp
961:         "ucc:all_gather");
962:   }
963: }
964: 
965: c10::intrusive_ptr<Work> ProcessGroupUCC::_allgather_base(
966:     at::Tensor& outputTensor,
967:     at::Tensor& inputTensor,
968:     const AllgatherOptions& opts) {
969:   check_tensor({outputTensor});
970:   check_tensor({inputTensor});
971:   initComm(outputTensor.device());
972: 
973:   WorkData* data = new WorkData();
974: 
975:   ucc_coll_args_t coll;
976:   coll.mask = 0;
977:   coll.flags = 0;
978:   coll.coll_type = UCC_COLL_TYPE_ALLGATHER;
979:   coll.src.info.buffer = inputTensor.data_ptr();
980:   coll.src.info.count = inputTensor.numel();
981:   coll.src.info.datatype = ucc_dtype_map.at(inputTensor.scalar_type());
982:   coll.src.info.mem_type = to_ucc_memType(inputTensor.device().type());
983:   coll.dst.info.buffer = outputTensor.data_ptr();
984:   coll.dst.info.count = outputTensor.numel();
```

- EN: Lines 961-984 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 961-984 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 985-1008 / 第 985-1008 行

```cpp
985:   coll.dst.info.datatype = ucc_dtype_map.at(outputTensor.scalar_type());
986:   coll.dst.info.mem_type = to_ucc_memType(outputTensor.device().type());
987: 
988:   std::vector<at::Tensor> inputTensors = {inputTensor};
989:   std::vector<at::Tensor> outputTensors = {outputTensor};
990:   SAVE_TENSORS(inputTensors, data->src);
991:   SAVE_TENSORS(outputTensors, data->dst);
992: 
993:   return collective_post(
994:       OpType::_ALLGATHER_BASE,
995:       []() {},
996:       []() {},
997:       coll,
998:       std::unique_ptr<WorkData>(data),
999:       outputTensor.device(),
1000:       inputTensors,
1001:       outputTensors,
1002:       "ucc:allgather_base");
1003: }
1004: 
1005: c10::intrusive_ptr<Work> ProcessGroupUCC::allreduce(
1006:     std::vector<at::Tensor>& tensors,
1007:     const AllreduceOptions& opts) {
1008:   check_tensor(tensors);
```

- EN: Lines 985-1008 introduces executable logic in routines such as `SAVE_TENSORS`; returns computed state or forwards results to the surrounding caller.
- CN: 第 985-1008 行在 `SAVE_TENSORS` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1009-1032 / 第 1009-1032 行

```cpp
1009:   auto& tensor = tensors[0];
1010:   initComm(tensor.device());
1011:   WorkData* data = new WorkData();
1012: 
1013:   ucc_coll_args_t coll;
1014:   coll.mask = UCC_COLL_ARGS_FIELD_FLAGS;
1015:   coll.flags = UCC_COLL_ARGS_FLAG_IN_PLACE;
1016:   coll.coll_type = UCC_COLL_TYPE_ALLREDUCE;
1017:   coll.op = to_ucc_reduceOp(opts.reduceOp, tensor.scalar_type());
1018:   coll.src.info.buffer = nullptr;
1019:   coll.src.info.count = tensor.numel();
1020:   coll.src.info.datatype = to_ucc_dType(tensor);
1021:   coll.src.info.mem_type = to_ucc_memType(tensor.device().type());
1022:   coll.dst.info.buffer = tensor.data_ptr();
1023:   coll.dst.info.count = tensor.numel();
1024:   coll.dst.info.datatype = to_ucc_dType(tensor);
1025:   coll.dst.info.mem_type = to_ucc_memType(tensor.device().type());
1026:   SAVE_TENSORS(tensors, data->dst);
1027:   return collective_post(
1028:       OpType::ALLREDUCE,
1029:       []() {},
1030:       []() {},
1031:       coll,
1032:       std::unique_ptr<WorkData>(data),
```

- EN: Lines 1009-1032 introduces executable logic in routines such as `SAVE_TENSORS`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1009-1032 行在 `SAVE_TENSORS` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1033-1056 / 第 1033-1056 行

```cpp
1033:       tensor.device(),
1034:       tensors,
1035:       tensors,
1036:       "ucc:all_reduce");
1037: }
1038: 
1039: c10::intrusive_ptr<Work> ProcessGroupUCC::allreduce_coalesced(
1040:     std::vector<at::Tensor>& /* unused */,
1041:     const AllreduceCoalescedOptions& /* unused */) {
1042:   throw std::invalid_argument(
1043:       "ProcessGroupUCC does not support allreduce_coalesced");
1044: }
1045: 
1046: c10::intrusive_ptr<Work> ProcessGroupUCC::alltoall(
1047:     std::vector<at::Tensor>& outputTensors,
1048:     std::vector<at::Tensor>& inputTensors,
1049:     const AllToAllOptions& /* unused */) {
1050:   auto device = outputTensors[0].device();
1051:   for (const auto r : c10::irange(outputTensors.size())) {
1052:     TORCH_CHECK(
1053:         device == outputTensors[r].device() &&
1054:             device == inputTensors[r].device(),
1055:         "Tensors must be on the same device")
1056:   }
```

- EN: Lines 1033-1056 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 1033-1056 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 1057-1080 / 第 1057-1080 行

```cpp
1057: 
1058:   initComm(device);
1059:   ucc_coll_args_t coll;
1060:   AlltoallWorkData* data;
1061:   data = new AlltoallWorkData(size_);
1062: 
1063:   /* to avoid flatten the tensors, we use alltoallv to achieve Alltoall as
1064:      follow.
1065:       1. store addresses of each tensor directly in displacements, keep buffer
1066:      to nullptr, i.e., 0
1067:       2. convert datatype to UINT8, which is always 1 bytes, to avoid wrong size
1068:      calculation in UCC layer
1069:       3. post Alltoallv
1070:   */
1071:   for (const auto i : c10::irange(size_)) {
1072:     data->send_lengths[i] =
1073:         (uint64_t)(inputTensors[i].element_size() * inputTensors[i].numel());
1074:     data->send_offsets[i] = (uint64_t)inputTensors[i].data_ptr();
1075:     data->recv_lengths[i] =
1076:         (uint64_t)(outputTensors[i].element_size() * outputTensors[i].numel());
1077:     data->recv_offsets[i] = (uint64_t)outputTensors[i].data_ptr();
1078:   }
1079: 
1080:   coll.mask = UCC_COLL_ARGS_FIELD_FLAGS;
```

- EN: Lines 1057-1080 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1057-1080 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1081-1104 / 第 1081-1104 行

```cpp
1081:   coll.flags =
1082:       UCC_COLL_ARGS_FLAG_COUNT_64BIT | UCC_COLL_ARGS_FLAG_DISPLACEMENTS_64BIT;
1083:   coll.coll_type = UCC_COLL_TYPE_ALLTOALLV;
1084:   coll.src.info_v.buffer = 0;
1085:   coll.src.info_v.counts = (ucc_count_t*)data->send_lengths.data();
1086:   coll.src.info_v.displacements = (ucc_aint_t*)data->send_offsets.data();
1087:   coll.src.info_v.datatype = UCC_DT_UINT8;
1088:   coll.src.info_v.mem_type = to_ucc_memType(inputTensors[0].device().type());
1089:   coll.dst.info_v.buffer = 0;
1090:   coll.dst.info_v.counts = (ucc_count_t*)data->recv_lengths.data();
1091:   coll.dst.info_v.displacements = (ucc_aint_t*)data->recv_offsets.data();
1092:   coll.dst.info_v.datatype = UCC_DT_UINT8;
1093:   coll.dst.info_v.mem_type = to_ucc_memType(outputTensors[0].device().type());
1094: 
1095:   SAVE_TENSORS(inputTensors, data->src);
1096:   SAVE_TENSORS(outputTensors, data->dst);
1097: 
1098:   return collective_post(
1099:       OpType::ALLTOALL,
1100:       []() {},
1101:       []() {},
1102:       coll,
1103:       std::unique_ptr<WorkData>(data),
1104:       device,
```

- EN: Lines 1081-1104 introduces executable logic in routines such as `SAVE_TENSORS`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1081-1104 行在 `SAVE_TENSORS` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1105-1128 / 第 1105-1128 行

```cpp
1105:       inputTensors,
1106:       outputTensors,
1107:       "ucc:alltoall");
1108: }
1109: 
1110: c10::intrusive_ptr<Work> ProcessGroupUCC::alltoall_base(
1111:     at::Tensor& outputTensor,
1112:     at::Tensor& inputTensor,
1113:     std::vector<int64_t>& outputSplitSizes,
1114:     std::vector<int64_t>& inputSplitSizes,
1115:     const AllToAllOptions& /* unused */) {
1116:   check_device(inputTensor.device(), outputTensor.device());
1117:   initComm(inputTensor.device());
1118:   ucc_coll_args_t coll;
1119:   AlltoallWorkData* data;
1120: 
1121:   if ((outputSplitSizes.size() == 0) && (inputSplitSizes.size() == 0)) {
1122:     data = new AlltoallWorkData(0);
1123:     TORCH_CHECK(
1124:         (outputTensor.size(0) % size_ == 0) &&
1125:             (inputTensor.size(0) % size_ == 0),
1126:         "Tensor's dim 0 does not divide equally across group size");
1127:     coll.mask = 0;
1128:     coll.flags = 0;
```

- EN: Lines 1105-1128 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 1105-1128 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 1129-1152 / 第 1129-1152 行

```cpp
1129:     coll.coll_type = UCC_COLL_TYPE_ALLTOALL;
1130:     coll.src.info.buffer = inputTensor.data_ptr();
1131:     coll.src.info.count = inputTensor.element_size() * inputTensor.numel();
1132:     coll.src.info.datatype = UCC_DT_UINT8;
1133:     coll.src.info.mem_type = to_ucc_memType(inputTensor.device().type());
1134:     coll.dst.info.buffer = outputTensor.data_ptr();
1135:     coll.dst.info.count = outputTensor.element_size() * outputTensor.numel();
1136:     coll.dst.info.datatype = UCC_DT_UINT8;
1137:     coll.dst.info.mem_type = to_ucc_memType(outputTensor.device().type());
1138:     coll.flags = 0;
1139:   } else {
1140:     data = new AlltoallWorkData(size_);
1141:     c10d::checkSplitSizes(inputSplitSizes, inputTensor, size_);
1142:     c10d::checkSplitSizes(outputSplitSizes, outputTensor, size_);
1143:     computeLengthsAndOffsets(
1144:         outputSplitSizes,
1145:         outputTensor,
1146:         &data->recv_lengths,
1147:         &data->recv_offsets);
1148:     computeLengthsAndOffsets(
1149:         inputSplitSizes, inputTensor, &data->send_lengths, &data->send_offsets);
1150:     coll.mask = UCC_COLL_ARGS_FIELD_FLAGS;
1151:     coll.coll_type = UCC_COLL_TYPE_ALLTOALLV;
1152:     coll.src.info_v.buffer = inputTensor.data_ptr();
```

- EN: Lines 1129-1152 introduces executable logic in routines such as `computeLengthsAndOffsets`.
- CN: 第 1129-1152 行在 `computeLengthsAndOffsets` 等例程中引入具体执行逻辑。

### Lines 1153-1176 / 第 1153-1176 行

```cpp
1153:     coll.src.info_v.counts = (ucc_count_t*)data->send_lengths.data();
1154:     coll.src.info_v.displacements = (ucc_aint_t*)data->send_offsets.data();
1155:     coll.src.info_v.datatype = to_ucc_dType(inputTensor);
1156:     coll.src.info_v.mem_type = to_ucc_memType(inputTensor.device().type());
1157:     coll.dst.info_v.buffer = outputTensor.data_ptr();
1158:     coll.dst.info_v.counts = (ucc_count_t*)data->recv_lengths.data();
1159:     coll.dst.info_v.displacements = (ucc_aint_t*)data->recv_offsets.data();
1160:     coll.dst.info_v.datatype = to_ucc_dType(outputTensor);
1161:     coll.dst.info_v.mem_type = to_ucc_memType(outputTensor.device().type());
1162:     coll.flags = UCC_COLL_ARGS_FLAG_CONTIG_SRC_BUFFER |
1163:         UCC_COLL_ARGS_FLAG_CONTIG_DST_BUFFER | UCC_COLL_ARGS_FLAG_COUNT_64BIT |
1164:         UCC_COLL_ARGS_FLAG_DISPLACEMENTS_64BIT;
1165: 
1166:     if (torch_ucc_config.enable_comms_logger) {
1167:       logger->trace_generator->recordOptionalInfo(
1168:           outputSplitSizes, inputSplitSizes);
1169:     }
1170:   }
1171:   std::vector<at::Tensor> inputTensors = {inputTensor};
1172:   std::vector<at::Tensor> outputTensors = {outputTensor};
1173:   SAVE_TENSORS(inputTensors, data->src);
1174:   SAVE_TENSORS(outputTensors, data->dst);
1175: 
1176:   return collective_post(
```

- EN: Lines 1153-1176 introduces executable logic in routines such as `SAVE_TENSORS`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1153-1176 行在 `SAVE_TENSORS` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1177-1200 / 第 1177-1200 行

```cpp
1177:       OpType::ALLTOALL_BASE,
1178:       []() {},
1179:       []() {},
1180:       coll,
1181:       std::unique_ptr<WorkData>(data),
1182:       inputTensor.device(),
1183:       inputTensors,
1184:       outputTensors,
1185:       "ucc:alltoall");
1186: }
1187: 
1188: c10::intrusive_ptr<Work> ProcessGroupUCC::barrier(const BarrierOptions& opts) {
1189:   c10::Device device = c10::Device(c10::DeviceType::CPU);
1190: #ifdef USE_CUDA
1191:   auto numGPUs = c10::cuda::device_count();
1192:   if (!opts.device_ids.empty()) {
1193:     device = c10::Device(c10::DeviceType::CUDA, opts.device_ids.front());
1194:   } else if (comm && comm->cuda_device_index != TORCH_UCC_DEVICE_NOT_SET) {
1195:     device = c10::Device(c10::DeviceType::CUDA, comm->cuda_device_index);
1196:   } else if (numGPUs > 0) {
1197:     int8_t deviceIdx = static_cast<int8_t>(c10::cuda::current_device());
1198:     // if current device is 0, likely the device is not set, use the best guess
1199:     if (0 == (int)deviceIdx) {
1200:       deviceIdx = static_cast<int8_t>(this->getRank() % numGPUs);
```

- EN: Lines 1177-1200 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1177-1200 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1201-1224 / 第 1201-1224 行

```cpp
1201:     }
1202:     TORCH_UCC_LOG_INFO(
1203:         TORCH_UCC_COLL_POST,
1204:         c10::str(
1205:             "post barrier before specifying any GPU while there are ",
1206:             numGPUs,
1207:             " GPUs available. ",
1208:             "Not clear if GPU barrier is required, using GPU ",
1209:             (int)deviceIdx,
1210:             " to perform barrier. ",
1211:             "Specify device_ids option in barrier() to force ",
1212:             "use of a particular device"));
1213:     device = c10::Device(c10::DeviceType::CUDA, deviceIdx);
1214:   }
1215: #endif
1216:   initComm(device);
1217: 
1218:   ucc_coll_args_t coll;
1219:   coll.mask = 0;
1220:   coll.flags = 0;
1221:   coll.coll_type = UCC_COLL_TYPE_BARRIER;
1222:   auto dummy_tensor = std::vector<at::Tensor>();
1223:   return collective_post(
1224:       OpType::BARRIER,
```

- EN: Lines 1201-1224 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1201-1224 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1225-1248 / 第 1225-1248 行

```cpp
1225:       []() {},
1226:       []() {},
1227:       coll,
1228:       nullptr,
1229:       device,
1230:       dummy_tensor,
1231:       dummy_tensor,
1232:       "ucc:barrier");
1233: }
1234: 
1235: c10::intrusive_ptr<Work> ProcessGroupUCC::broadcast(
1236:     std::vector<at::Tensor>& tensors,
1237:     const BroadcastOptions& opts) {
1238:   check_tensor(tensors);
1239:   auto& tensor = tensors[0];
1240:   initComm(tensor.device());
1241:   WorkData* data = new WorkData();
1242: 
1243:   ucc_coll_args_t coll;
1244:   coll.mask = 0;
1245:   coll.flags = 0;
1246:   coll.coll_type = UCC_COLL_TYPE_BCAST;
1247:   coll.src.info.buffer = tensor.data_ptr();
1248:   coll.src.info.count = tensor.numel();
```

- EN: Lines 1225-1248 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1225-1248 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1249-1272 / 第 1249-1272 行

```cpp
1249:   coll.src.info.datatype = to_ucc_dType(tensor);
1250:   coll.src.info.mem_type = to_ucc_memType(tensor.device().type());
1251:   coll.root = opts.rootRank;
1252:   SAVE_TENSORS(tensors, data->dst);
1253: 
1254:   if (torch_ucc_config.enable_comms_logger) {
1255:     logger->trace_generator->recordOptionalInfo(opts.rootRank);
1256:   }
1257: 
1258:   return collective_post(
1259:       OpType::BROADCAST,
1260:       []() {},
1261:       []() {},
1262:       coll,
1263:       std::unique_ptr<WorkData>(data),
1264:       tensor.device(),
1265:       tensors,
1266:       tensors,
1267:       "ucc:broadcast");
1268: }
1269: 
1270: c10::intrusive_ptr<Work> ProcessGroupUCC::gather(
1271:     std::vector<std::vector<at::Tensor>>& outputTensors,
1272:     std::vector<at::Tensor>& inputTensors,
```

- EN: Lines 1249-1272 introduces executable logic in routines such as `SAVE_TENSORS`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1249-1272 行在 `SAVE_TENSORS` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1273-1296 / 第 1273-1296 行

```cpp
1273:     const GatherOptions& opts) {
1274:   std::vector<at::Tensor> outputs;
1275:   auto& input = inputTensors[0];
1276:   initComm(input.device());
1277: 
1278:   AllgathervWorkData* data = new AllgathervWorkData(size_);
1279:   ucc_coll_args_t coll;
1280:   coll.root = opts.rootRank;
1281:   coll.mask = UCC_COLL_ARGS_FIELD_FLAGS;
1282:   coll.flags =
1283:       UCC_COLL_ARGS_FLAG_COUNT_64BIT | UCC_COLL_ARGS_FLAG_DISPLACEMENTS_64BIT;
1284:   coll.coll_type = UCC_COLL_TYPE_GATHERV;
1285: 
1286:   /* for non-root ranks, only src is valid */
1287:   coll.src.info.buffer = input.data_ptr();
1288:   coll.src.info.count = (uint64_t)(input.element_size() * input.numel());
1289:   coll.src.info.datatype = UCC_DT_UINT8;
1290:   coll.src.info.mem_type = to_ucc_memType(input.device().type());
1291: 
1292:   if (getRank() == opts.rootRank) {
1293:     if (outputTensors.size() != 1) {
1294:       TORCH_UCC_LOG_ERROR(
1295:           TORCH_UCC_COLL_POST,
1296:           c10::str(
```

- EN: Lines 1273-1296 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1273-1296 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1297-1320 / 第 1297-1320 行

```cpp
1297:               "gather requires a single-element output list containing a list with ",
1298:               getSize(),
1299:               " tensors."));
1300:     } else if (outputTensors[0].size() != static_cast<size_t>(getSize())) {
1301:       TORCH_UCC_LOG_ERROR(
1302:           TORCH_UCC_COLL_POST,
1303:           c10::str(
1304:               "Incorrect output list size ",
1305:               outputTensors[0].size(),
1306:               ". Output list size should be ",
1307:               getSize(),
1308:               ", same as size of the process group."));
1309:     }
1310:     outputs = outputTensors[0];
1311: 
1312:     for (int i = 0; i < size_; i++) {
1313:       data->recv_lengths[i] =
1314:           (uint64_t)(outputs[i].element_size() * outputs[i].numel());
1315:       data->recv_offsets[i] = (uint64_t)outputs[i].data_ptr();
1316:     }
1317:     /* use gatherv and store non-contiguous addresses in displacements to avoid
1318:      * flatten outputTensors */
1319:     coll.dst.info_v.buffer = nullptr;
1320:     coll.dst.info_v.counts = (ucc_count_t*)data->recv_lengths.data();
```

- EN: Lines 1297-1320 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1297-1320 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1321-1344 / 第 1321-1344 行

```cpp
1321:     coll.dst.info_v.displacements = (ucc_aint_t*)data->recv_offsets.data();
1322:     coll.dst.info_v.datatype = UCC_DT_UINT8;
1323:     coll.dst.info_v.mem_type = to_ucc_memType(outputs[0].device().type());
1324: 
1325:     SAVE_TENSORS(outputs, data->dst);
1326:   } else {
1327:     // for non-root ranks, outputTensors should be an empty list
1328:     if (!outputTensors.empty()) {
1329:       TORCH_UCC_LOG_ERROR(
1330:           TORCH_UCC_COLL_POST, "requires empty output on non-root");
1331:     }
1332:     outputs = {};
1333:     // append a empty tensor to the list to be used by future mark
1334:     outputs.emplace_back();
1335:   }
1336: 
1337:   SAVE_TENSORS(inputTensors, data->src);
1338: 
1339:   return collective_post(
1340:       OpType::GATHER,
1341:       []() {},
1342:       []() {},
1343:       coll,
1344:       std::unique_ptr<WorkData>(data),
```

- EN: Lines 1321-1344 introduces executable logic in routines such as `SAVE_TENSORS`, `TORCH_UCC_LOG_ERROR`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1321-1344 行在 `SAVE_TENSORS`、`TORCH_UCC_LOG_ERROR` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1345-1368 / 第 1345-1368 行

```cpp
1345:       input.device(),
1346:       inputTensors,
1347:       outputs,
1348:       "ucc:gather");
1349: }
1350: 
1351: c10::intrusive_ptr<Work> ProcessGroupUCC::reduce(
1352:     std::vector<at::Tensor>& tensors,
1353:     const ReduceOptions& opts) {
1354:   check_tensor(tensors);
1355:   auto& tensor = tensors[0];
1356:   initComm(tensor.device());
1357:   WorkData* data = new WorkData();
1358: 
1359:   ucc_coll_args_t coll;
1360:   coll.mask = UCC_COLL_ARGS_FIELD_FLAGS;
1361:   coll.flags = UCC_COLL_ARGS_FLAG_IN_PLACE;
1362:   coll.coll_type = UCC_COLL_TYPE_REDUCE;
1363:   coll.op = ucc_op_map.at(opts.reduceOp);
1364:   coll.root = opts.rootRank;
1365:   coll.src.info.buffer = tensor.data_ptr();
1366:   coll.src.info.count = tensor.numel();
1367:   coll.src.info.datatype = ucc_dtype_map.at(tensor.scalar_type());
1368:   coll.src.info.mem_type = to_ucc_memType(tensor.device().type());
```

- EN: Lines 1345-1368 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1345-1368 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1369-1392 / 第 1369-1392 行

```cpp
1369:   coll.dst.info.buffer = tensor.data_ptr();
1370:   coll.dst.info.count = tensor.numel();
1371:   coll.dst.info.datatype = ucc_dtype_map.at(tensor.scalar_type());
1372:   coll.dst.info.mem_type = to_ucc_memType(tensor.device().type());
1373:   SAVE_TENSORS(tensors, data->dst);
1374:   return collective_post(
1375:       OpType::REDUCE,
1376:       []() {},
1377:       []() {},
1378:       coll,
1379:       std::unique_ptr<WorkData>(data),
1380:       tensor.device(),
1381:       tensors,
1382:       tensors,
1383:       "ucc:reduce");
1384: }
1385: 
1386: c10::intrusive_ptr<Work> ProcessGroupUCC::reduce_scatter(
1387:     std::vector<at::Tensor>& outputTensors,
1388:     std::vector<std::vector<at::Tensor>>& inputTensors,
1389:     const ReduceScatterOptions& opts) {
1390:   TORCH_CHECK(
1391:       (outputTensors.size() == inputTensors.size()),
1392:       "Tensor input/output list for reduce_scatter must have same size");
```

- EN: Lines 1369-1392 introduces executable logic in routines such as `SAVE_TENSORS`; performs validation and error handling to keep distributed state consistent.
- CN: 第 1369-1392 行在 `SAVE_TENSORS` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 1393-1416 / 第 1393-1416 行

```cpp
1393:   check_tensor(outputTensors);
1394:   check_device(inputTensors[0][0].device(), outputTensors[0].device());
1395:   initComm(inputTensors[0][0].device());
1396:   auto data = std::make_unique<WorkData>();
1397:   std::vector<at::Tensor> flat_input(inputTensors.size());
1398:   for (size_t i = 0; i < inputTensors.size(); i++) {
1399:     TORCH_CHECK(
1400:         inputTensors[i].size() == inputTensors.size() * size_,
1401:         "Tensor input list is not valid for the number of participants");
1402:     flat_input[i] = c10d::newLikeFlat(inputTensors, i);
1403:   }
1404:   SAVE_TENSORS(flat_input, data->flat);
1405:   check_tensor(flat_input);
1406:   ucc_coll_args_t coll;
1407:   coll.mask = 0;
1408:   coll.flags = 0;
1409:   coll.coll_type = UCC_COLL_TYPE_REDUCE_SCATTER;
1410:   coll.op = to_ucc_reduceOp(opts.reduceOp, flat_input[0].scalar_type());
1411: 
1412:   coll.src.info.buffer = flat_input[0].data_ptr();
1413:   coll.src.info.count = flat_input[0].numel();
1414:   coll.src.info.datatype = to_ucc_dType(flat_input[0]);
1415:   coll.src.info.mem_type = to_ucc_memType(flat_input[0].device().type());
1416:   coll.dst.info.buffer = outputTensors[0].data_ptr();
```

- EN: Lines 1393-1416 introduces executable logic in routines such as `SAVE_TENSORS`; performs validation and error handling to keep distributed state consistent.
- CN: 第 1393-1416 行在 `SAVE_TENSORS` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 1417-1440 / 第 1417-1440 行

```cpp
1417:   coll.dst.info.count = outputTensors[0].numel();
1418:   coll.dst.info.datatype = to_ucc_dType(outputTensors[0]);
1419:   coll.dst.info.mem_type = to_ucc_memType(outputTensors[0].device().type());
1420: 
1421:   SAVE_TENSORS(inputTensors[0], data->src);
1422:   SAVE_TENSORS(outputTensors, data->dst);
1423: 
1424:   auto copy_to_flat = [&] {
1425:     bool asyncCopy = false;
1426:     auto isize = inputTensors.size();
1427: #ifdef USE_CUDA
1428:     bool isCuda = inputTensors[0][0].device().is_cuda();
1429: #endif
1430:     for (size_t i = 0; i < isize; i++) {
1431:       auto onumel = outputTensors[i].numel();
1432:       for (size_t j = 0; j < inputTensors[i].size(); j++) {
1433:         TORCH_CHECK(
1434:             (inputTensors[i][j].numel() == onumel),
1435:             "Tensor operand counts must be same");
1436: #ifdef USE_CUDA
1437:         if (isCuda) {
1438:           c10::cuda::CUDACachingAllocator::recordStream(
1439:               inputTensors[i][j].storage().data_ptr(), (*stream));
1440:           asyncCopy = true;
```

- EN: Lines 1417-1440 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `SAVE_TENSORS`; performs validation and error handling to keep distributed state consistent.
- CN: 第 1417-1440 行使用条件编译来适配特性开关、平台或可选后端；在 `SAVE_TENSORS` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 1441-1464 / 第 1441-1464 行

```cpp
1441:         }
1442: #endif
1443:         flat_input[i][j].copy_(inputTensors[i][j], asyncCopy);
1444:       }
1445:     }
1446:   };
1447: 
1448:   return collective_post(
1449:       OpType::REDUCE_SCATTER,
1450:       copy_to_flat,
1451:       []() {},
1452:       coll,
1453:       std::move(data),
1454:       inputTensors[0][0].device(),
1455:       inputTensors[0],
1456:       outputTensors,
1457:       "ucc:reduce_scatter");
1458: }
1459: 
1460: c10::intrusive_ptr<Work> ProcessGroupUCC::_reduce_scatter_base(
1461:     at::Tensor& outputTensor,
1462:     at::Tensor& inputTensor,
1463:     const ReduceScatterOptions& opts) {
1464:   check_tensor({outputTensor});
```

- EN: Lines 1441-1464 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1441-1464 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1465-1488 / 第 1465-1488 行

```cpp
1465:   check_tensor({inputTensor});
1466:   initComm(outputTensor.device());
1467: 
1468:   auto data = std::make_unique<WorkData>();
1469: 
1470:   ucc_coll_args_t coll;
1471:   coll.mask = 0;
1472:   coll.flags = 0;
1473:   coll.coll_type = UCC_COLL_TYPE_REDUCE_SCATTER;
1474:   coll.op = to_ucc_reduceOp(opts.reduceOp, inputTensor.scalar_type());
1475: 
1476:   coll.src.info.buffer = inputTensor.data_ptr();
1477:   coll.src.info.count = inputTensor.numel();
1478:   coll.src.info.datatype = ucc_dtype_map.at(inputTensor.scalar_type());
1479:   coll.src.info.mem_type = to_ucc_memType(inputTensor.device().type());
1480:   coll.dst.info.buffer = outputTensor.data_ptr();
1481:   coll.dst.info.count = outputTensor.numel();
1482:   coll.dst.info.datatype = ucc_dtype_map.at(outputTensor.scalar_type());
1483:   coll.dst.info.mem_type = to_ucc_memType(outputTensor.device().type());
1484: 
1485:   std::vector<at::Tensor> inputTensors = {inputTensor};
1486:   std::vector<at::Tensor> outputTensors = {outputTensor};
1487:   SAVE_TENSORS(inputTensors, data->src);
1488:   SAVE_TENSORS(outputTensors, data->dst);
```

- EN: Lines 1465-1488 introduces executable logic in routines such as `SAVE_TENSORS`.
- CN: 第 1465-1488 行在 `SAVE_TENSORS` 等例程中引入具体执行逻辑。

### Lines 1489-1512 / 第 1489-1512 行

```cpp
1489: 
1490:   return collective_post(
1491:       OpType::_REDUCE_SCATTER_BASE,
1492:       []() {},
1493:       []() {},
1494:       coll,
1495:       std::move(data),
1496:       outputTensor.device(),
1497:       inputTensors,
1498:       outputTensors,
1499:       "ucc:_reduce_scatter_base");
1500: }
1501: 
1502: c10::intrusive_ptr<Work> ProcessGroupUCC::scatter(
1503:     std::vector<at::Tensor>& outputTensors,
1504:     std::vector<std::vector<at::Tensor>>& inputTensors,
1505:     const ScatterOptions& opts) {
1506:   auto& tensor = outputTensors[0];
1507:   initComm(tensor.device());
1508: 
1509:   ScattervWorkData* data = new ScattervWorkData(size_);
1510:   ucc_coll_args_t coll;
1511:   coll.root = opts.rootRank;
1512:   coll.mask = UCC_COLL_ARGS_FIELD_FLAGS;
```

- EN: Lines 1489-1512 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1489-1512 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1513-1536 / 第 1513-1536 行

```cpp
1513:   coll.flags =
1514:       UCC_COLL_ARGS_FLAG_COUNT_64BIT | UCC_COLL_ARGS_FLAG_DISPLACEMENTS_64BIT;
1515:   coll.coll_type = UCC_COLL_TYPE_SCATTERV;
1516: 
1517:   if (getRank() == opts.rootRank) {
1518:     /* src is only valid at non-root rank */
1519:     if (inputTensors.size() != 1) {
1520:       TORCH_UCC_LOG_ERROR(
1521:           TORCH_UCC_COLL_POST,
1522:           c10::str(
1523:               "gather requires a single-element output list containing a list with ",
1524:               getSize(),
1525:               " tensors."));
1526:     } else if (inputTensors[0].size() != static_cast<size_t>(getSize())) {
1527:       TORCH_UCC_LOG_ERROR(
1528:           TORCH_UCC_COLL_POST,
1529:           c10::str(
1530:               "Incorrect output list size ",
1531:               inputTensors[0].size(),
1532:               ". Output list size should be ",
1533:               getSize(),
1534:               ", same as size of the process group."));
1535:     }
1536: 
```

- EN: Lines 1513-1536 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1513-1536 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1537-1560 / 第 1537-1560 行

```cpp
1537:     for (int i = 0; i < size_; i++) {
1538:       data->send_lengths[i] = (uint64_t)tensor.element_size() * tensor.numel();
1539:       data->send_offsets[i] = (uint64_t)inputTensors[0][i].data_ptr();
1540:     }
1541:     /* use scatter and store non-contiguous addresses in displacements to avoid
1542:      * flatten inputTensors */
1543:     coll.src.info_v.buffer = nullptr;
1544:     coll.src.info_v.counts = (ucc_count_t*)data->send_lengths.data();
1545:     coll.src.info_v.displacements = (ucc_aint_t*)data->send_offsets.data();
1546:     coll.src.info_v.datatype = UCC_DT_UINT8;
1547:     coll.src.info_v.mem_type =
1548:         to_ucc_memType(inputTensors[0][0].device().type());
1549: 
1550:     SAVE_TENSORS(inputTensors[0], data->src);
1551:   } else {
1552:     // for non-root ranks, inputTensors should be an empty list
1553:     if (!inputTensors.empty()) {
1554:       TORCH_UCC_LOG_ERROR(
1555:           TORCH_UCC_COLL_POST, "requires empty output on non-root");
1556:     }
1557:   }
1558: 
1559:   coll.dst.info.buffer = tensor.data_ptr();
1560:   coll.dst.info.count = (uint64_t)tensor.element_size() * tensor.numel();
```

- EN: Lines 1537-1560 introduces executable logic in routines such as `SAVE_TENSORS`, `TORCH_UCC_LOG_ERROR`.
- CN: 第 1537-1560 行在 `SAVE_TENSORS`、`TORCH_UCC_LOG_ERROR` 等例程中引入具体执行逻辑。

### Lines 1561-1584 / 第 1561-1584 行

```cpp
1561:   coll.dst.info.datatype = UCC_DT_UINT8;
1562:   coll.dst.info.mem_type = to_ucc_memType(tensor.device().type());
1563:   SAVE_TENSORS(outputTensors, data->dst);
1564: 
1565:   return collective_post(
1566:       OpType::SCATTER,
1567:       []() {},
1568:       []() {},
1569:       coll,
1570:       std::unique_ptr<WorkData>(data),
1571:       tensor.device(),
1572:       (getRank() == opts.rootRank) ? inputTensors[0] : outputTensors,
1573:       outputTensors,
1574:       "ucc:scatter");
1575: }
1576: 
1577: c10::intrusive_ptr<Work> ProcessGroupUCC::send(
1578:     std::vector<at::Tensor>& tensors,
1579:     int dstRank,
1580:     int tag) {
1581:   check_tensor(tensors);
1582:   auto& tensor = tensors[0];
1583:   initComm(tensor.device());
1584: 
```

- EN: Lines 1561-1584 introduces executable logic in routines such as `SAVE_TENSORS`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1561-1584 行在 `SAVE_TENSORS` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1585-1608 / 第 1585-1608 行

```cpp
1585:   WorkData* data = new WorkData();
1586:   ucc_coll_args_t coll;
1587:   coll.tag = tag;
1588:   coll.mask = UCC_COLL_ARGS_FIELD_ACTIVE_SET | UCC_COLL_ARGS_FIELD_TAG;
1589:   coll.flags = 0;
1590:   coll.coll_type = UCC_COLL_TYPE_BCAST;
1591:   coll.src.info.buffer = tensor.data_ptr();
1592:   coll.src.info.count = tensor.numel();
1593:   coll.src.info.datatype = to_ucc_dType(tensor);
1594:   coll.src.info.mem_type = to_ucc_memType(tensor.device().type());
1595:   coll.root = getRank();
1596: 
1597:   coll.active_set.size = 2;
1598:   coll.active_set.start = getRank();
1599:   coll.active_set.stride = dstRank - getRank();
1600:   SAVE_TENSORS(tensors, data->dst);
1601: 
1602:   return collective_post(
1603:       OpType::SEND,
1604:       []() {},
1605:       []() {},
1606:       coll,
1607:       std::unique_ptr<WorkData>(data),
1608:       tensor.device(),
```

- EN: Lines 1585-1608 introduces executable logic in routines such as `SAVE_TENSORS`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1585-1608 行在 `SAVE_TENSORS` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1609-1632 / 第 1609-1632 行

```cpp
1609:       tensors,
1610:       tensors,
1611:       "ucc:send");
1612: }
1613: 
1614: c10::intrusive_ptr<Work> ProcessGroupUCC::recv(
1615:     std::vector<at::Tensor>& tensors,
1616:     int srcRank,
1617:     int tag) {
1618:   check_tensor(tensors);
1619:   auto& tensor = tensors[0];
1620:   initComm(tensor.device());
1621: 
1622:   WorkData* data = new WorkData();
1623:   ucc_coll_args_t coll;
1624:   coll.tag = tag;
1625:   coll.mask = UCC_COLL_ARGS_FIELD_ACTIVE_SET | UCC_COLL_ARGS_FIELD_TAG;
1626:   coll.flags = 0;
1627:   coll.coll_type = UCC_COLL_TYPE_BCAST;
1628:   coll.src.info.buffer = tensor.data_ptr();
1629:   coll.src.info.count = tensor.numel();
1630:   coll.src.info.datatype = to_ucc_dType(tensor);
1631:   coll.src.info.mem_type = to_ucc_memType(tensor.device().type());
1632:   coll.root = srcRank;
```

- EN: Lines 1609-1632 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1609-1632 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1633-1656 / 第 1633-1656 行

```cpp
1633: 
1634:   coll.active_set.size = 2;
1635:   coll.active_set.start = srcRank;
1636:   coll.active_set.stride = getRank() - srcRank;
1637:   SAVE_TENSORS(tensors, data->dst);
1638: 
1639:   return collective_post(
1640:       OpType::RECV,
1641:       []() {},
1642:       []() {},
1643:       coll,
1644:       std::unique_ptr<WorkData>(data),
1645:       tensor.device(),
1646:       tensors,
1647:       tensors,
1648:       "ucc:recv");
1649: }
1650: 
1651: void ProcessGroupUCC::setSequenceNumberForGroup() {}
1652: 
1653: uint64_t ProcessGroupUCC::getSequenceNumberForGroup() {
1654:   return seq_;
1655: }
1656: 
```

- EN: Lines 1633-1656 introduces executable logic in routines such as `SAVE_TENSORS`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1633-1656 行在 `SAVE_TENSORS` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1657-1680 / 第 1657-1680 行

```cpp
1657: c10::intrusive_ptr<Backend> ProcessGroupUCC::createProcessGroupUCC(
1658:     const c10::intrusive_ptr<::c10d::Store>& store,
1659:     int rank,
1660:     int size,
1661:     const std::chrono::duration<float>& timeout) {
1662:   return c10::make_intrusive<ProcessGroupUCC>(store, rank, size, timeout);
1663: }
1664: 
1665: void ProcessGroupUCC::initComm(c10::Device dev) {
1666:   if (!comm) {
1667: #ifdef USE_CUDA
1668:     if (dev.is_cuda()) {
1669:       c10::cuda::set_device(dev.index());
1670:     }
1671: #endif
1672:     comm = Comm::get_comm(comm_id, dev, oob, logger);
1673:     TORCH_UCC_LOG_INFO(TORCH_UCC_INIT, "Successfully initialized UCX library");
1674:     comm->ucc_create_team(team, oob);
1675:     TORCH_UCC_LOG_INFO(TORCH_UCC_INIT, "Successfully initialized UCC library");
1676:     logger->setPhase(TORCH_UCC_READY);
1677:   } else {
1678:     if (dev.is_cuda()) {
1679:       if ((comm->cuda_device_index != TORCH_UCC_DEVICE_NOT_SET) &&
1680:           (comm->cuda_device_index != dev.index())) {
```

- EN: Lines 1657-1680 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `TORCH_UCC_LOG_INFO`.
- CN: 第 1657-1680 行使用条件编译来适配特性开关、平台或可选后端；在 `TORCH_UCC_LOG_INFO` 等例程中引入具体执行逻辑。

### Lines 1681-1704 / 第 1681-1704 行

```cpp
1681:         TORCH_UCC_LOG_ERROR(
1682:             TORCH_UCC_INIT,
1683:             "ucc communicator was initialized with different cuda device,"
1684:             "multi device is not supported");
1685:         throw std::invalid_argument(ucc_status_string(UCC_ERR_NOT_SUPPORTED));
1686:       }
1687:       comm->cuda_device_index = dev.index();
1688:     }
1689:   }
1690: #ifdef USE_CUDA
1691:   // Create UCC execution engine.
1692:   if (!cuda_ee && dev.is_cuda()) {
1693:     stream = std::make_unique<at::cuda::CUDAStream>(
1694:         at::cuda::getStreamFromPool(true, dev.index()));
1695:     ucc_ee_params_t params;
1696:     params.ee_type = UCC_EE_CUDA_STREAM;
1697:     params.ee_context = (void*)stream->stream();
1698:     params.ee_context_size = sizeof(cudaStream_t);
1699:     TORCH_UCC_CHECK(
1700:         ucc_ee_create(team, &params, &cuda_ee),
1701:         "failed to create UCC execution engine");
1702:     for (int i = 0; i < 2; i++) {
1703:       stream_p2p[i] = std::make_unique<at::cuda::CUDAStream>(
1704:           at::cuda::getStreamFromPool(true, dev.index()));
```

- EN: Lines 1681-1704 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `TORCH_UCC_LOG_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 1681-1704 行使用条件编译来适配特性开关、平台或可选后端；在 `TORCH_UCC_LOG_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 1705-1719 / 第 1705-1719 行

```cpp
1705:       ucc_ee_params_t params;
1706:       params.ee_type = UCC_EE_CUDA_STREAM;
1707:       params.ee_context = (void*)stream_p2p[i]->stream();
1708:       params.ee_context_size = sizeof(cudaStream_t);
1709:       TORCH_UCC_CHECK(
1710:           ucc_ee_create(team, &params, &cuda_ee_p2p[i]),
1711:           "failed to create UCC P2P execution engine");
1712:     }
1713:   }
1714: #endif
1715: }
1716: 
1717: } // namespace c10d
1718: 
1719: #endif // USE_C10D_UCC
```

- EN: Lines 1705-1719 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1705-1719 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

## Key Concepts / 关键概念

- EN: Subsystem: c10d distributed process-group subsystem.
- CN: 子系统：c10d 分布式进程组子系统。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `torch_ucc_config_t`, `HealthCheckData`
- CN: 核心符号：`torch_ucc_config_t`、`HealthCheckData`
- EN: Notable themes: process-group orchestration, store/state coordination, collective communication logic.
- CN: 值得关注的主题：进程组编排、存储/状态协调、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/ProcessGroup.hpp`, `torch/csrc/distributed/c10d/ProcessGroupUCC.hpp`, `torch/csrc/distributed/c10d/UCCTracing.hpp`, `torch/csrc/distributed/c10d/UCCUtils.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/cuda/nvrtc_stub/ATenNVRTC.h`, `c10/util/CallOnce.h`, `c10/util/env.h`
- External or system headers / 外部或系统头文件: `list`, `memory`, `unordered_map`, `unordered_set`
- Local symbols / 本地符号: `torch_ucc_config_t`, `HealthCheckData`