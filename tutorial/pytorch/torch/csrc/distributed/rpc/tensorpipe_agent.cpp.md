# tensorpipe_agent.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/rpc/tensorpipe_agent.cpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides implementation logic for tensorpipe agent in the distributed RPC layer. Representative routines include `getDevicesForTensors`, `getStreamsFromPoolForDevices`, `getCurrentStreamsForDevices`, `getDevicesOfTensors`, `makeStreamsWaitOnOthers`.
- 用途 (CN): 该文件在分布式 RPC 层中提供tensorpipe agent 的实现逻辑。 代表性例程包括 `getDevicesForTensors`、`getStreamsFromPoolForDevices`、`getCurrentStreamsForDevices`、`getDevicesOfTensors`、`makeStreamsWaitOnOthers`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24 / 第 1-24 行

```cpp
1: #include <torch/csrc/distributed/rpc/tensorpipe_agent.h>
2: 
3: #ifdef USE_TENSORPIPE
4: 
5: #include <limits>
6: #include <tuple>
7: #include <utility>
8: 
9: #include <fmt/format.h>
10: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wdeprecated")
11: #include <tensorpipe/tensorpipe.h>
12: C10_DIAGNOSTIC_POP()
13: 
14: #include <torch/csrc/distributed/rpc/agent_utils.h>
15: #include <torch/csrc/distributed/rpc/tensorpipe_utils.h>
16: #include <torch/csrc/distributed/rpc/utils.h>
17: 
18: #include <c10/core/StreamGuard.h>
19: #include <c10/util/irange.h>
20: 
21: namespace torch::distributed::rpc {
22: 
23: namespace {
24: 
```

- EN: Lines 1-24 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 1-24 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 25-48 / 第 25-48 行

```cpp
25: // An environment variable along the lines of GLOO_ and NCCL_SOCKET_IFNAME that
26: // allows the user to specify a device to bind to, instead of binding to the
27: // address that the hostname resolves to.
28: const std::string kSocketIfnameEnvVar = "TP_SOCKET_IFNAME";
29: const std::string kDefaultUvAddress = "127.0.0.1";
30: 
31: const std::string kGilAverageWaitTime = "agent.gil_average_wait_time_us";
32: const std::string kThreadPoolSize = "agent.thread_pool_size";
33: const std::string kNumIdleThreads = "agent.num_idle_threads";
34: const std::string kClientActiveCalls = "agent.client_active_calls";
35: const std::string kServerActiveCalls = "agent.server_active_calls";
36: const std::string kServerActiveAsyncCalls = "agent.server_active_async_calls";
37: 
38: std::vector<c10::Device> getDevicesForTensors(
39:     const std::vector<torch::Tensor>& tensors,
40:     const DeviceMap& deviceMap,
41:     const std::string& remoteName) {
42:   // If the deviceMap is overridden, use that instead.
43:   const auto errStr = c10::str(
44:       "TensorPipe RPC backend only supports CPU tensors by default, please "
45:       "move your tensors to CPU before sending them over RPC, or call "
46:       "`set_device_map` on `TensorPipeRpcBackendOptions` to explicitly "
47:       "configure device mapping. ",
48:       "Request device mapping is not available for destination ",
```

- EN: Lines 25-48 introduces executable logic in routines such as `getDevicesForTensors`.
- CN: 第 25-48 行在 `getDevicesForTensors` 等例程中引入具体执行逻辑。

### Lines 49-72 / 第 49-72 行

```cpp
49:       remoteName);
50:   std::vector<c10::Device> devices;
51:   devices.reserve(tensors.size());
52:   bool hasMappedDevice = false;
53:   for (const auto& t : tensors) {
54:     if (t.device().is_cpu()) {
55:       const auto deviceIter = deviceMap.find(c10::kCPU);
56:       if (deviceIter == deviceMap.end()) {
57:         devices.emplace_back(c10::kCPU);
58:       } else {
59:         devices.emplace_back(deviceIter->second);
60:         hasMappedDevice = true;
61:       }
62:     } else {
63:       const auto deviceIter = deviceMap.find(t.device());
64:       TORCH_CHECK(
65:           deviceIter != deviceMap.end(),
66:           errStr,
67:           " for device ",
68:           t.device(),
69:           " but received a tensor on that device.");
70:       devices.push_back(deviceIter->second);
71:       hasMappedDevice = true;
72:     }
```

- EN: Lines 49-72 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 49-72 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 73-96 / 第 73-96 行

```cpp
73:   }
74:   if (!hasMappedDevice) {
75:     devices.clear();
76:   }
77:   return devices;
78: }
79: 
80: std::vector<c10::Stream> getStreamsFromPoolForDevices(
81:     const std::vector<c10::Device>& devices) {
82:   if (devices.empty()) {
83:     return {};
84:   }
85:   c10::impl::VirtualGuardImpl impl(devices[0].type());
86:   std::vector<c10::Stream> streams;
87:   streams.reserve(devices.size());
88:   for (const c10::Device& device : devices) {
89:     TORCH_INTERNAL_ASSERT(device.type() == impl.type());
90:     streams.push_back(impl.getStreamFromGlobalPool(device));
91:   }
92:   return streams;
93: }
94: 
95: std::vector<c10::Stream> getCurrentStreamsForDevices(
96:     const std::vector<c10::Device>& devices) {
```

- EN: Lines 73-96 introduces executable logic in routines such as `getStreamsFromPoolForDevices`, `getCurrentStreamsForDevices`; returns computed state or forwards results to the surrounding caller.
- CN: 第 73-96 行在 `getStreamsFromPoolForDevices`、`getCurrentStreamsForDevices` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 97-120 / 第 97-120 行

```cpp
97:   if (devices.empty()) {
98:     return {};
99:   }
100:   c10::impl::VirtualGuardImpl impl(devices[0].type());
101:   std::vector<c10::Stream> streams;
102:   streams.reserve(devices.size());
103:   for (const c10::Device& device : devices) {
104:     TORCH_INTERNAL_ASSERT(device.type() == impl.type());
105:     streams.push_back(impl.getStream(device));
106:   }
107:   return streams;
108: }
109: 
110: std::vector<c10::Device> getDevicesOfTensors(
111:     const std::vector<torch::Tensor>& tensors) {
112:   std::optional<c10::impl::VirtualGuardImpl> impl;
113:   size_t deviceCount = 0;
114:   std::vector<bool> indexBitset;
115:   for (const torch::Tensor& tensor : tensors) {
116:     if (!tensor.is_cpu()) {
117:       c10::Device device = tensor.device();
118:       if (!impl.has_value()) {
119:         impl.emplace(device.type());
120:         indexBitset.resize(impl->deviceCount());
```

- EN: Lines 97-120 introduces executable logic in routines such as `getDevicesOfTensors`; returns computed state or forwards results to the surrounding caller.
- CN: 第 97-120 行在 `getDevicesOfTensors` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 121-144 / 第 121-144 行

```cpp
121:       }
122:       TORCH_INTERNAL_ASSERT(device.type() == impl->type());
123:       TORCH_INTERNAL_ASSERT(device.has_index());
124:       if (!indexBitset[device.index()]) {
125:         deviceCount++;
126:         indexBitset[device.index()] = true;
127:       }
128:     }
129:   }
130:   std::vector<c10::Device> devices;
131:   devices.reserve(deviceCount);
132:   for (const auto idx : c10::irange(indexBitset.size())) {
133:     if (indexBitset[idx]) {
134:       // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
135:       devices.emplace_back(impl->type(), static_cast<c10::DeviceIndex>(idx));
136:     }
137:   }
138:   return devices;
139: }
140: 
141: void makeStreamsWaitOnOthers(
142:     const std::vector<c10::Stream>& consumers,
143:     const std::vector<c10::Stream>& producers) {
144:   for (const c10::Stream& producer : producers) {
```

- EN: Lines 121-144 introduces executable logic in routines such as `makeStreamsWaitOnOthers`; returns computed state or forwards results to the surrounding caller.
- CN: 第 121-144 行在 `makeStreamsWaitOnOthers` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 145-168 / 第 145-168 行

```cpp
145:     const c10::Stream& consumer =
146:         getStreamForDevice(consumers, producer.device());
147:     c10::Event event(producer.device_type());
148:     event.record(producer);
149:     event.block(consumer);
150:   }
151: }
152: 
153: } // namespace
154: 
155: C10_DEFINE_REGISTRY_WITHOUT_WARNING(
156:     TensorPipeTransportRegistry,
157:     TransportRegistration)
158: 
159: C10_DEFINE_REGISTRY_WITHOUT_WARNING(
160:     TensorPipeChannelRegistry,
161:     ChannelRegistration)
162: 
163: const std::string& TensorPipeAgent::guessAddress() {
164:   static const std::string uvAddress = []() {
165:     auto ifnameEnv = c10::utils::get_env(kSocketIfnameEnvVar.c_str());
166:     if (ifnameEnv.has_value()) {
167:       auto [error, result] =
168:           tensorpipe::transport::uv::lookupAddrForIface(ifnameEnv.value());
```

- EN: Lines 145-168 opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 145-168 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 169-192 / 第 169-192 行

```cpp
169:       if (error) {
170:         LOG(WARNING) << "Failed to look up the IP address for interface "
171:                      << ifnameEnv.value() << " (" << error.what()
172:                      << "), defaulting to " << kDefaultUvAddress;
173:         return kDefaultUvAddress;
174:       }
175:       return result;
176:     }
177:     auto [error, result] = tensorpipe::transport::uv::lookupAddrForHostname();
178:     if (error) {
179:       LOG(WARNING) << "Failed to look up the IP address for the hostname ("
180:                    << error.what() << "), defaulting to " << kDefaultUvAddress;
181:       return kDefaultUvAddress;
182:     }
183:     return result;
184:   }();
185:   return uvAddress;
186: }
187: 
188: namespace {
189: 
190: std::unique_ptr<TransportRegistration> makeUvTransport() {
191:   auto context = tensorpipe::transport::uv::create();
192:   std::string address = TensorPipeAgent::guessAddress();
```

- EN: Lines 169-192 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `makeUvTransport`.
- CN: 第 169-192 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `makeUvTransport` 等例程中引入具体执行逻辑。

### Lines 193-216 / 第 193-216 行

```cpp
193:   return std::make_unique<TransportRegistration>(TransportRegistration{
194:       std::move(context), kUvTransportPriority, std::move(address)});
195: }
196: 
197: // The UV transport is implemented using standard TCP connections. It leverages
198: // libuv (https://github.com/libuv/libuv) in order to be cross-platform.
199: C10_REGISTER_CREATOR(TensorPipeTransportRegistry, uv, makeUvTransport)
200: 
201: #if TENSORPIPE_HAS_SHM_TRANSPORT
202: 
203: std::unique_ptr<TransportRegistration> makeShmTransport() {
204:   auto context = tensorpipe::transport::shm::create();
205:   return std::make_unique<TransportRegistration>(
206:       TransportRegistration{std::move(context), kShmTransportPriority, ""});
207: }
208: 
209: // The SHM implements connections using ringbuffers residing in anonymous shared
210: // memory (plus UNIX domain sockets to bootstrap the connection and exchange
211: // file descriptors). It is Linux-only due to some advanced features (O_TMPFILE,
212: // eventfd, ...).
213: C10_REGISTER_CREATOR(TensorPipeTransportRegistry, shm, makeShmTransport)
214: 
215: #endif // TENSORPIPE_HAS_SHM_TRANSPORT
216: 
```

- EN: Lines 193-216 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `makeShmTransport`.
- CN: 第 193-216 行使用条件编译来适配特性开关、平台或可选后端；在 `makeShmTransport` 等例程中引入具体执行逻辑。

### Lines 217-240 / 第 217-240 行

```cpp
217: #if TENSORPIPE_HAS_IBV_TRANSPORT
218: 
219: std::unique_ptr<TransportRegistration> makeIbvTransport() {
220:   auto context = tensorpipe::transport::ibv::create();
221:   std::string address = TensorPipeAgent::guessAddress();
222:   return std::make_unique<TransportRegistration>(TransportRegistration{
223:       std::move(context), kIbvTransportPriority, std::move(address)});
224: }
225: 
226: // The IBV transport sends data across using an InfiniBand queue pair, locally
227: // copying data to and from a staging buffer (registered with libibverbs) and
228: // issuing a RDMA write for transferring data across machines (plus a send for
229: // acknowledging it). It bootstraps using a standard TCP connection to exchange
230: // setup information. It is Linux-only.
231: C10_REGISTER_CREATOR(TensorPipeTransportRegistry, ibv, makeIbvTransport)
232: 
233: #endif // TENSORPIPE_HAS_IBV_TRANSPORT
234: 
235: std::unique_ptr<ChannelRegistration> makeBasicChannel() {
236:   auto context = tensorpipe::channel::basic::create();
237:   return std::make_unique<ChannelRegistration>(
238:       ChannelRegistration{std::move(context), kBasicChannelPriority});
239: }
240: 
```

- EN: Lines 217-240 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `makeIbvTransport`, `makeBasicChannel`.
- CN: 第 217-240 行使用条件编译来适配特性开关、平台或可选后端；在 `makeIbvTransport`、`makeBasicChannel` 等例程中引入具体执行逻辑。

### Lines 241-264 / 第 241-264 行

```cpp
241: // The basic channel is just a straightforward adapter wrapper that allows any
242: // transport to be used as a channel.
243: C10_REGISTER_CREATOR(TensorPipeChannelRegistry, basic, makeBasicChannel)
244: 
245: #if TENSORPIPE_HAS_CMA_CHANNEL
246: 
247: std::unique_ptr<ChannelRegistration> makeCmaChannel() {
248:   auto context = tensorpipe::channel::cma::create();
249:   return std::make_unique<ChannelRegistration>(
250:       ChannelRegistration{std::move(context), kCmaChannelPriority});
251: }
252: 
253: // The CMA channel uses the Linux cross-memory attach syscalls (process_vm_readv
254: // and _writev), which allow one process to access the private memory of another
255: // process (as long as they belong to the same user and other security
256: // constraints are satisfied). It does, more or less, what GDB does when it's
257: // attached to a running process.
258: C10_REGISTER_CREATOR(TensorPipeChannelRegistry, cma, makeCmaChannel)
259: 
260: #endif // TENSORPIPE_HAS_CMA_CHANNEL
261: 
262: constexpr static int kNumUvThreads = 16;
263: 
264: std::unique_ptr<ChannelRegistration> makeMultiplexedUvChannel() {
```

- EN: Lines 241-264 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `makeCmaChannel`, `makeMultiplexedUvChannel`.
- CN: 第 241-264 行使用条件编译来适配特性开关、平台或可选后端；在 `makeCmaChannel`、`makeMultiplexedUvChannel` 等例程中引入具体执行逻辑。

### Lines 265-288 / 第 265-288 行

```cpp
265:   std::vector<std::shared_ptr<tensorpipe::transport::Context>> contexts;
266:   contexts.reserve(kNumUvThreads);
267:   std::vector<std::shared_ptr<tensorpipe::transport::Listener>> listeners;
268:   listeners.reserve(kNumUvThreads);
269:   for ([[maybe_unused]] const auto laneIdx : c10::irange(kNumUvThreads)) {
270:     auto context = tensorpipe::transport::uv::create();
271:     const std::string& address = TensorPipeAgent::guessAddress();
272:     contexts.push_back(std::move(context));
273:     listeners.push_back(contexts.back()->listen(address));
274:   }
275:   auto context = tensorpipe::channel::mpt::create(
276:       std::move(contexts), std::move(listeners));
277:   return std::make_unique<ChannelRegistration>(
278:       ChannelRegistration{std::move(context), kMultiplexedUvChannelPriority});
279: }
280: 
281: // The multiplexed UV channel encapsulates multiple UV transports (each with its
282: // own event loop thread). Each channel will, in turn, contain multiple UV
283: // connections, one for each of those contexts. When sending a tensor, its data
284: // is split in equal chunks and each chunks is sent on a different connection
285: // and thus driven by a different thread. This is needed to reach very high
286: // bandwidths.
287: C10_REGISTER_CREATOR(
288:     TensorPipeChannelRegistry,
```

- EN: Lines 265-288 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 265-288 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 289-312 / 第 289-312 行

```cpp
289:     mpt_uv,
290:     makeMultiplexedUvChannel)
291: 
292: } // namespace
293: 
294: //////////////////////////  MetricsTracker  /////////////////////////////////
295: 
296: TensorPipeAgent::TimeSeriesMetricsTracker::TimeSeriesMetricsTracker(
297:     uint64_t currentSum,
298:     uint64_t currentCount)
299:     : currentSum_(currentSum), currentCount_(currentCount) {}
300: 
301: void TensorPipeAgent::TimeSeriesMetricsTracker::addData(uint64_t dataPoint) {
302:   currentSum_ += dataPoint;
303:   ++currentCount_;
304: }
305: 
306: float TensorPipeAgent::TimeSeriesMetricsTracker::computeAverage() const {
307:   return currentCount_ == 0 ? 0
308:                             : static_cast<float>(
309:                                   static_cast<double>(currentSum_) /
310:                                   static_cast<double>(currentCount_));
311: }
312: 
```

- EN: Lines 289-312 opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 289-312 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 313-336 / 第 313-336 行

```cpp
313: ////////////////////////  TensorpipeRpcAgent  /////////////////////////////////
314: 
315: void TensorPipeAgent::removeFromTimeoutMap(uint64_t messageId) {
316:   // Remove entry from timeoutMap_.
317:   {
318:     std::unique_lock<std::mutex> lock(timeoutMapMutex_);
319:     auto it = messageIdToTimeout_.find(messageId);
320:     if (it == messageIdToTimeout_.end()) {
321:       // Already removed from the map by pollTimeoutRpcs(), no need to
322:       // process further.
323:       return;
324:     }
325: 
326:     auto& expirationTime = it->second;
327: 
328:     auto& timedOutFuturesVector = timeoutMap_[expirationTime];
329:     for (auto it = timedOutFuturesVector.begin();
330:          it != timedOutFuturesVector.end();
331:          it++) {
332:       if (it->messageId == messageId) {
333:         it = timedOutFuturesVector.erase(it);
334:         break;
335:       }
336:     }
```

- EN: Lines 313-336 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 313-336 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 337-360 / 第 337-360 行

```cpp
337: 
338:     if (timedOutFuturesVector.empty()) {
339:       timeoutMap_.erase(expirationTime);
340:     }
341: 
342:     // Remove from messageId to timeout map as well.
343:     messageIdToTimeout_.erase(messageId);
344:   }
345: }
346: 
347: void TensorPipeAgent::prepareNames(bool isStaticGroup) {
348:   std::unordered_map<std::string, worker_id_t> nameToId;
349:   if (isStaticGroup) {
350:     nameToId = collectNames(
351:         rankToNameStore_, workerInfo_.id_, workerInfo_.name_, worldSize_);
352:   } else {
353:     nameToId = collectCurrentNames(
354:         rankToNameStore_, workerInfo_.id_, workerInfo_.name_);
355:   }
356: 
357:   for (const auto& entry : nameToId) {
358:     const auto& workerName = entry.first;
359:     const auto& workerId = entry.second;
360:     workerIdToInfo_.emplace(workerId, WorkerInfo(workerName, workerId));
```

- EN: Lines 337-360 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 337-360 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 361-384 / 第 361-384 行

```cpp
361:     workerNameToInfo_.emplace(workerName, WorkerInfo(workerName, workerId));
362:   }
363: }
364: 
365: void TensorPipeAgent::checkAndSetStaticGroup(
366:     const c10::intrusive_ptr<::c10d::Store>& store) {
367:   std::string isStaticGroupKey("rpcIsStaticGroup");
368: 
369:   std::string isStaticGroupStr = isStaticGroup_ ? "true" : "false";
370:   std::vector<uint8_t> isStaticGroupVec(
371:       (uint8_t*)isStaticGroupStr.c_str(),
372:       (uint8_t*)isStaticGroupStr.c_str() + isStaticGroupStr.length());
373:   std::vector<uint8_t> returnedVec;
374:   returnedVec = store->compareSet(
375:       isStaticGroupKey, std::vector<uint8_t>(), isStaticGroupVec);
376:   std::string returnedVal = std::string(returnedVec.begin(), returnedVec.end());
377:   // In both cases, the returned value should be the value of isStaticGroupStr,
378:   // otherwise there is a discrepancy with initialization among one of the
379:   // members
380:   TORCH_CHECK(
381:       returnedVal == isStaticGroupStr,
382:       fmt::format(
383:           "RPC group mixes statically and dynamically initialized members which is not supported. ",
384:           "Static group property is initialized as {} and is trying to be set as {} ",
```

- EN: Lines 361-384 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 361-384 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 385-408 / 第 385-408 行

```cpp
385:           isStaticGroup_,
386:           returnedVal));
387: }
388: 
389: TensorPipeAgent::TensorPipeAgent(
390:     const c10::intrusive_ptr<::c10d::Store>& store,
391:     std::string selfName,
392:     worker_id_t selfId,
393:     std::optional<int> worldSize,
394:     TensorPipeRpcBackendOptions opts,
395:     std::unordered_map<std::string, DeviceMap> reverseDeviceMaps,
396:     std::vector<c10::Device> devices,
397:     std::unique_ptr<RequestCallback> cb)
398:     : RpcAgent(
399:           WorkerInfo(std::move(selfName), selfId),
400:           std::move(cb),
401:           std::chrono::milliseconds(
402:               static_cast<long>(opts.rpcTimeoutSeconds * kSecToMsConversion))),
403:       isStaticGroup_(worldSize.has_value()),
404:       store_(store),
405:       opts_(std::move(opts)),
406:       reverseDeviceMaps_(std::move(reverseDeviceMaps)),
407:       devices_(std::move(devices)),
408:       threadPool_(opts_.numWorkerThreads),
```

- EN: Lines 385-408 mainly closes scopes and completes the surrounding declaration or implementation block.
- CN: 第 385-408 行主要用于关闭作用域并结束当前声明或实现块。

### Lines 409-432 / 第 409-432 行

```cpp
409:       context_(std::make_shared<tensorpipe::Context>(
410:           tensorpipe::ContextOptions().name(workerInfo_.name_))),
411:       rankToNameStore_("names", store),
412:       nameToAddressStore_("addrs", store),
413:       shutdownStore_("shutdown", store) {
414:   if (isStaticGroup_) {
415:     worldSize_ = worldSize.value();
416:   }
417: 
418:   // check the static group attribute against store
419:   checkAndSetStaticGroup(store);
420: 
421:   // collect worker names
422:   prepareNames(isStaticGroup_);
423: 
424:   // Initialize the time-series metrics tracking map
425:   timeSeriesMetrics_.emplace(kGilAverageWaitTime, TimeSeriesMetricsTracker());
426: }
427: 
428: TensorPipeAgent::~TensorPipeAgent() {
429:   VLOG(1) << "RPC agent for " << workerInfo_.name_ << " is being destroyed";
430:   shutdown();
431: }
432: 
```

- EN: Lines 409-432 introduces executable logic in routines such as `shutdown`.
- CN: 第 409-432 行在 `shutdown` 等例程中引入具体执行逻辑。

### Lines 433-456 / 第 433-456 行

```cpp
433: void TensorPipeAgent::startImpl() {
434:   VLOG(1) << "RPC agent for " << workerInfo_.name_ << " is starting";
435: 
436:   std::vector<std::string> addresses;
437:   int64_t lowestPriority = std::numeric_limits<int64_t>::max();
438:   std::string lowestPriorityTransport;
439: 
440:   // Register transports
441:   for (auto& key : TensorPipeTransportRegistry()->Keys()) {
442:     int64_t priority = -1;
443:     if (opts_.transports.has_value()) {
444:       auto iter =
445:           std::find(opts_.transports->begin(), opts_.transports->end(), key);
446:       if (iter == opts_.transports->end()) {
447:         continue;
448:       }
449:       // Assign priorities in reverse order of occurrence in the vector, so that
450:       // a transport that comes before another receives a higher priority.
451:       priority = static_cast<std::ptrdiff_t>(opts_.transports->size()) - 1 -
452:           (iter - opts_.transports->begin());
453:     }
454:     std::unique_ptr<TransportRegistration> reg =
455:         TensorPipeTransportRegistry()->Create(key);
456:     if (!reg->transport->isViable()) {
```

- EN: Lines 433-456 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 433-456 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 457-480 / 第 457-480 行

```cpp
457:       continue;
458:     }
459:     if (priority == -1) {
460:       priority = reg->priority;
461:     }
462:     if (priority < lowestPriority) {
463:       lowestPriority = priority;
464:       lowestPriorityTransport = key;
465:     }
466:     addresses.push_back(c10::str(key, "://", reg->address));
467:     context_->registerTransport(priority, key, reg->transport);
468:   }
469: 
470:   // Register channels
471:   for (auto& key : TensorPipeChannelRegistry()->Keys()) {
472:     int64_t priority = -1;
473:     if (opts_.channels.has_value()) {
474:       auto iter =
475:           std::find(opts_.channels->begin(), opts_.channels->end(), key);
476:       if (iter == opts_.channels->end()) {
477:         continue;
478:       }
479:       // Assign priorities in reverse order of occurrence in the vector, so
480:       // that a channel that comes before another receives a higher priority.
```

- EN: Lines 457-480 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 457-480 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 481-504 / 第 481-504 行

```cpp
481:       priority = static_cast<std::ptrdiff_t>(opts_.channels->size()) - 1 -
482:           (iter - opts_.channels->begin());
483:     }
484:     std::unique_ptr<ChannelRegistration> reg =
485:         TensorPipeChannelRegistry()->Create(key);
486:     if (!reg->channel->isViable()) {
487:       continue;
488:     }
489:     if (priority == -1) {
490:       priority = reg->priority;
491:     }
492:     context_->registerChannel(priority, key, reg->channel);
493:   }
494: 
495:   listener_ = context_->listen(addresses);
496: 
497:   // Store our own url.
498:   const auto address = listener_->url(lowestPriorityTransport);
499:   nameToAddressStore_.set(workerInfo_.name_, address);
500: 
501:   VLOG(1) << "RPC agent for " << workerInfo_.name_ << " is using address "
502:           << address;
503: 
504:   for (const auto& p : workerNameToInfo_) {
```

- EN: Lines 481-504 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 481-504 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 505-528 / 第 505-528 行

```cpp
505:     const auto& name = p.first;
506:     auto nodeAddrData = nameToAddressStore_.get(name);
507:     auto nodeAddrStr = std::string(
508:         reinterpret_cast<const char*>(nodeAddrData.data()),
509:         nodeAddrData.size());
510:     workerNameToURL_.insert({name, nodeAddrStr});
511:   }
512: 
513:   // Start the Timeout Thread
514:   timeoutThread_ = std::thread(&TensorPipeAgent::pollTimeoutRpcs, this);
515: 
516:   listener_->accept([this](
517:                         const tensorpipe::Error& error,
518:                         std::shared_ptr<tensorpipe::Pipe> pipe) {
519:     onListenerAccepted(error, pipe);
520:   });
521: }
522: 
523: void TensorPipeAgent::onListenerAccepted(
524:     const tensorpipe::Error& error,
525:     std::shared_ptr<tensorpipe::Pipe>& pipe) {
526:   if (error) {
527:     if (error.isOfType<tensorpipe::ListenerClosedError>() &&
528:         !rpcAgentRunning_.load()) {
```

- EN: Lines 505-528 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 505-528 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 529-552 / 第 529-552 行

```cpp
529:       // This is expected.
530:     } else {
531:       LOG(WARNING) << "RPC agent for " << workerInfo_.name_
532:                    << " encountered error when accepting incoming pipe: "
533:                    << error.what();
534:     }
535:     return;
536:   }
537: 
538:   // Accept the next connection request
539:   listener_->accept([this](
540:                         const tensorpipe::Error& error,
541:                         std::shared_ptr<tensorpipe::Pipe> pipe) {
542:     onListenerAccepted(error, pipe);
543:   });
544: 
545:   VLOG(1) << "RPC agent for " << workerInfo_.name_
546:           << " accepted incoming pipe from " << pipe->getRemoteName();
547: 
548:   // Arm for server read
549:   respond(pipe);
550: }
551: 
552: void TensorPipeAgent::pipeRead(
```

- EN: Lines 529-552 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 529-552 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 553-576 / 第 553-576 行

```cpp
553:     const std::shared_ptr<tensorpipe::Pipe>& pipe,
554:     std::function<void(
555:         const tensorpipe::Error&,
556:         c10::intrusive_ptr<Message>,
557:         std::vector<c10::Stream>)> fn) noexcept {
558:   pipe->readDescriptor([this, fn{std::move(fn)}, pipe](
559:                            const tensorpipe::Error& error,
560:                            tensorpipe::Descriptor tpDescriptor) mutable {
561:     if (error) {
562:       fn(error, c10::intrusive_ptr<Message>(), {});
563:       return;
564:     }
565: 
566:     std::vector<c10::Stream> streams;
567:     {
568:       GroupMembershipLockGuard guard(groupMembershipMutex_, isStaticGroup_);
569:       streams = getStreamsFromPoolForDevices(devices_);
570:     }
571:     auto [tpAllocation, tpBuffers] = tensorpipeAllocate(tpDescriptor, streams);
572: 
573:     pipe->read(
574:         std::move(tpAllocation),
575:         [tpDescriptor{std::move(tpDescriptor)},
576:          tpBuffers{
```

- EN: Lines 553-576 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 553-576 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 577-600 / 第 577-600 行

```cpp
577:              std::make_shared<TensorpipeReadBuffers>(std::move(tpBuffers))},
578:          fn{std::move(fn)},
579:          streams{std::move(streams)}](const tensorpipe::Error& error) mutable {
580:           if (error) {
581:             fn(error, c10::intrusive_ptr<Message>(), {});
582:             return;
583:           }
584: 
585:           // FIXME This does some unpickling, which could be a bit expensive:
586:           // perhaps it would be best to perform it inside the worker threads?
587:           c10::intrusive_ptr<Message> rpcMessage =
588:               tensorpipeDeserialize(tpDescriptor, std::move(*tpBuffers));
589: 
590:           fn(error, std::move(rpcMessage), std::move(streams));
591:         });
592:   });
593: }
594: 
595: void TensorPipeAgent::pipeWrite(
596:     const std::shared_ptr<tensorpipe::Pipe>& pipe,
597:     const c10::intrusive_ptr<Message>& rpcMessage,
598:     std::vector<c10::Device>&& devices,
599:     std::vector<c10::Stream> streams,
600:     std::function<void(const tensorpipe::Error&)> fn) noexcept {
```

- EN: Lines 577-600 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 577-600 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 601-624 / 第 601-624 行

```cpp
601:   auto [tpMessage, tpBuffers] =
602:       tensorpipeSerialize(rpcMessage, std::move(devices), streams);
603: 
604:   pipe->write(
605:       std::move(tpMessage),
606:       [tpBuffers{
607:            std::make_shared<TensorpipeWriteBuffers>(std::move(tpBuffers))},
608:        fn{std::move(fn)},
609:        streams{std::move(streams)}](const tensorpipe::Error& error) {
610:         fn(error);
611:       });
612: }
613: 
614: void TensorPipeAgent::sendCompletedResponseMessage(
615:     std::shared_ptr<tensorpipe::Pipe>& pipe,
616:     JitFuture& futureResponseMessage,
617:     uint64_t messageId,
618:     std::vector<c10::Stream> streams) {
619:   if (!rpcAgentRunning_.load()) {
620:     LOG(WARNING) << "RPC agent for " << workerInfo_.name_
621:                  << " won't send response to request #" << messageId << " to "
622:                  << pipe->getRemoteName() << ", as the agent is shutting down";
623:     return;
624:   }
```

- EN: Lines 601-624 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 601-624 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 625-648 / 第 625-648 行

```cpp
625: 
626:   VLOG(1) << "RPC agent for " << workerInfo_.name_
627:           << " is sending response to request #" << messageId << " to "
628:           << pipe->getRemoteName();
629: 
630:   if (!futureResponseMessage.hasError()) {
631:     c10::intrusive_ptr<Message> responseMessage =
632:         futureResponseMessage.value().toCustomClass<Message>();
633:     responseMessage->setId(static_cast<int64_t>(messageId));
634: 
635:     std::vector<c10::Device> devices;
636:     try {
637:       devices = getDevicesForRemote(pipe->getRemoteName(), *responseMessage);
638:     } catch (const std::exception& e) {
639:       responseMessage =
640:           createExceptionResponse(e.what(), static_cast<int64_t>(messageId));
641:     }
642: 
643:     for (const auto& tensor : responseMessage->tensors()) {
644:       const auto device = tensor.device();
645:       if (!device.is_cpu()) {
646:         GroupMembershipLockGuard guard(groupMembershipMutex_, isStaticGroup_);
647:         if (std::find(devices_.begin(), devices_.end(), device) ==
648:             devices_.end()) {
```

- EN: Lines 625-648 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 625-648 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 649-672 / 第 649-672 行

```cpp
649:           std::ostringstream oss;
650:           std::copy(
651:               devices_.begin(),
652:               devices_.end(),
653:               std::ostream_iterator<c10::Device>(oss, ", "));
654:           responseMessage = createExceptionResponse(
655:               c10::str(
656:                   "RPC detected that a user-function output tensor on device ",
657:                   device,
658:                   ". This device is not one of the input tensor devices: ",
659:                   oss.str(),
660:                   "which is not yet supported. Please file a feature request "
661:                   "issue in PyTorch GitHub repo."),
662:               static_cast<int64_t>(messageId));
663:           break;
664:         }
665:       }
666:     }
667: 
668:     pipeWrite(
669:         pipe,
670:         responseMessage,
671:         std::move(devices),
672:         std::move(streams),
```

- EN: Lines 649-672 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 649-672 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 673-696 / 第 673-696 行

```cpp
673:         [this, pipe, messageId](const tensorpipe::Error& error) {
674:           if (error) {
675:             LOG(WARNING)
676:                 << "RPC agent for " << workerInfo_.name_
677:                 << " encountered error when sending response to request #"
678:                 << messageId << " to " << pipe->getRemoteName() << ": "
679:                 << error.what();
680:             return;
681:           }
682: 
683:           VLOG(1) << "RPC agent for " << workerInfo_.name_
684:                   << " done sending response to request #" << messageId
685:                   << " to " << pipe->getRemoteName();
686:         });
687:   } else {
688:     pipeWrite(
689:         pipe,
690:         createExceptionResponse(
691:             futureResponseMessage.tryRetrieveErrorMessage(),
692:             static_cast<int64_t>(messageId)),
693:         /* devices */ {},
694:         std::move(streams),
695:         [this, pipe, messageId](const tensorpipe::Error& error) {
696:           if (error) {
```

- EN: Lines 673-696 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 673-696 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 697-720 / 第 697-720 行

```cpp
697:             LOG(WARNING)
698:                 << "RPC agent for " << workerInfo_.name_
699:                 << " encountered error when sending response to request #"
700:                 << messageId << " to " << pipe->getRemoteName() << ": "
701:                 << error.what();
702:             return;
703:           }
704: 
705:           VLOG(1) << "RPC agent for " << workerInfo_.name_
706:                   << " done sending response to request #" << messageId
707:                   << " to " << pipe->getRemoteName();
708:         });
709:   }
710: }
711: 
712: void TensorPipeAgent::respond(std::shared_ptr<tensorpipe::Pipe>& pipe) {
713:   pipeRead(
714:       pipe,
715:       [this, pipe](
716:           const tensorpipe::Error& error,
717:           c10::intrusive_ptr<Message> requestMessage,
718:           std::vector<c10::Stream> streams) mutable {
719:         if (error) {
720:           if (shuttingDown_) {
```

- EN: Lines 697-720 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 697-720 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 721-744 / 第 721-744 行

```cpp
721:             // This is expected.
722:           } else {
723:             LOG(WARNING)
724:                 << "RPC agent for " << workerInfo_.name_
725:                 << " encountered error when reading incoming request from "
726:                 << pipe->getRemoteName() << ": " << error.what();
727:           }
728:           return;
729:         }
730: 
731:         // Arm for next read
732:         respond(pipe);
733: 
734:         uint64_t messageId = requestMessage->id();
735:         increaseCallCount(serverActiveCalls_);
736: 
737:         VLOG(1) << "RPC agent for " << workerInfo_.name_
738:                 << " received request #" << messageId << " from "
739:                 << pipe->getRemoteName();
740: 
741:         // Defer user RPC UDF run to thread pool
742:         threadPool_.run([this,
743:                          pipe,
744:                          messageId,
```

- EN: Lines 721-744 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 721-744 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 745-768 / 第 745-768 行

```cpp
745:                          requestMessage{std::move(requestMessage)},
746:                          streams{std::move(streams)}]() mutable {
747:           VLOG(1) << "RPC agent for " << workerInfo_.name_
748:                   << " is running request #" << messageId << " from "
749:                   << pipe->getRemoteName() << " in thread pool";
750: 
751:           c10::intrusive_ptr<JitFuture> futureResponseMessage;
752:           try {
753:             // Instead of creating a MultiStreamGuard here, the ctx is passed
754:             // to the callback and the MultiStreamGuard is created there,
755:             // because subsequent processing can switch threads due to 1)
756:             // waiting for RRef arguments to become ready 2) async_execution.
757:             // Besides, the `ctx` also needs to be propagated to
758:             // `process***Call` methods to synchronize CUDA streams there
759:             // to make sure that we fetch the correct value from `to_here()`
760:             // call.
761:             futureResponseMessage =
762:                 cb_->operator()(*requestMessage, std::move(streams));
763:           } catch (const std::exception& /* unused */) {
764:             futureResponseMessage =
765:                 c10::make_intrusive<JitFuture>(at::AnyClassType::get());
766:             futureResponseMessage->setError(std::current_exception());
767:           }
768: 
```

- EN: Lines 745-768 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 745-768 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 769-792 / 第 769-792 行

```cpp
769:           increaseCallCount(serverActiveAsyncCalls_);
770:           futureResponseMessage->addCallback(
771:               [this, pipe, messageId](
772:                   JitFuture& futureResponseMessage) mutable {
773:                 decreaseCallCount(serverActiveCalls_);
774:                 decreaseCallCount(serverActiveAsyncCalls_);
775:                 auto streams = getCurrentStreamsForDevices(
776:                     futureResponseMessage.devices());
777:                 sendCompletedResponseMessage(
778:                     pipe, futureResponseMessage, messageId, std::move(streams));
779:               });
780: 
781:           VLOG(1) << "RPC agent for " << workerInfo_.name_
782:                   << " done running request #" << messageId << " from "
783:                   << pipe->getRemoteName() << " in thread pool";
784:         });
785:       });
786: }
787: 
788: c10::intrusive_ptr<JitFuture> TensorPipeAgent::send(
789:     const WorkerInfo& toWorkerInfo,
790:     c10::intrusive_ptr<Message> requestMessage,
791:     const float rpcTimeoutSeconds,
792:     const DeviceMap& deviceMap) {
```

- EN: Lines 769-792 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 769-792 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 793-816 / 第 793-816 行

```cpp
793:   TORCH_CHECK(
794:       requestMessage->isRequest(),
795:       "TensorPipeAgent::send(..) is only for sending requests.");
796: 
797:   if (!rpcAgentRunning_.load()) {
798:     auto err = c10::str(
799:         "Node ",
800:         RpcAgent::getWorkerInfo().id_,
801:         "tried to send() a message of type ",
802:         requestMessage->type(),
803:         " but RPC is no longer running on this node.");
804:     TORCH_CHECK(false, err);
805:   }
806: 
807:   const auto& url = findWorkerURL(toWorkerInfo);
808: 
809:   decltype(connectedPipes_)::iterator it;
810:   {
811:     std::unique_lock<std::mutex> lock(connectedPipesMutex_);
812: 
813:     // See if we already have a connection to this address or not
814:     it = connectedPipes_.find(toWorkerInfo.id_);
815:     if (it == connectedPipes_.end()) {
816:       // An instance of ClientPipe cannot be copied or moved as it contains a
```

- EN: Lines 793-816 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 793-816 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 817-840 / 第 817-840 行

```cpp
817:       // mutex, and to force in-place construction in GCC 5 we need piecewise
818:       // construction in order to work around an issue.
819:       it = connectedPipes_
820:                .emplace(
821:                    std::piecewise_construct,
822:                    std::forward_as_tuple(toWorkerInfo.id_),
823:                    std::forward_as_tuple(context_->connect(
824:                        url,
825:                        tensorpipe::PipeOptions().remoteName(
826:                            toWorkerInfo.name_))))
827:                .first;
828:     }
829:   }
830:   ClientPipe& clientPipe = it->second;
831: 
832:   std::shared_ptr<torch::distributed::rpc::TensorPipeAgent::AtomicJitFuture>
833:       futureResponseMessage;
834:   {
835:     GroupMembershipLockGuard guard(groupMembershipMutex_, isStaticGroup_);
836:     futureResponseMessage = std::make_shared<AtomicJitFuture>(devices_);
837:   }
838:   uint64_t messageId = nextMessageID_++;
839:   requestMessage->setId(static_cast<int64_t>(messageId));
840: 
```

- EN: Lines 817-840 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 817-840 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 841-864 / 第 841-864 行

```cpp
841:   {
842:     std::unique_lock<std::mutex> lock(clientPipe.mutex_);
843:     clientPipe.pendingResponseMessage_[messageId] = futureResponseMessage;
844:   }
845: 
846:   // Get devices for tensors in the request message. This can throw if device
847:   // maps are not configured properly for this request.
848:   std::vector<c10::Device> devices;
849:   if (deviceMap.empty()) {
850:     devices =
851:         getDevicesForRemote(clientPipe.pipe_->getRemoteName(), *requestMessage);
852:   } else {
853:     // If deviceMap is specified, use that instead.
854:     devices = getDevicesForTensors(
855:         requestMessage->tensors(),
856:         deviceMap,
857:         clientPipe.pipe_->getRemoteName());
858:   }
859: 
860:   futureResponseMessage->jitFuture->addCallback(
861:       [this](JitFuture& /* unused */) {
862:         TORCH_INTERNAL_ASSERT(
863:             this->threadPool_.inThreadPool(),
864:             "Future marked complete from outside the thread pool");
```

- EN: Lines 841-864 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 841-864 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 865-888 / 第 865-888 行

```cpp
865:       });
866: 
867:   increaseCallCount(clientActiveCalls_);
868:   // Use the default RPC timeout if no timeout is specified for this send call
869:   auto timeout = rpcTimeoutSeconds == kUnsetRpcTimeout
870:       ? getRpcTimeout()
871:       : std::chrono::milliseconds(
872:             static_cast<int>(rpcTimeoutSeconds * kSecToMsConversion));
873: 
874:   // We only add to the timeoutMap_ if the timeout is not 0. Per our
875:   // documentation, a user-provided timeout of 0 indicates the RPC should never
876:   // expire (infinite timeout), so there is no need to track it in the
877:   // timeoutMap_.
878:   steady_clock_time_point expirationTime;
879:   if (timeout.count() != 0) {
880:     // Compute the expiration time for this message based on the timeout
881:     expirationTime = computeRpcMessageExpiryTime(timeout);
882: 
883:     // Add the Future to the right vector in the timeoutMap_
884:     {
885:       std::unique_lock<std::mutex> lock(timeoutMapMutex_);
886:       auto& timeoutFuturesVector = timeoutMap_[expirationTime];
887:       messageIdToTimeout_.emplace(messageId, expirationTime);
888:       timeoutFuturesVector.emplace_back(
```

- EN: Lines 865-888 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 865-888 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 889-912 / 第 889-912 行

```cpp
889:           messageId, futureResponseMessage, timeout);
890:     }
891:     timeoutThreadCV_.notify_one();
892:   }
893: 
894:   VLOG(1) << "RPC agent for " << workerInfo_.name_ << " is sending request #"
895:           << messageId << " to " << clientPipe.pipe_->getRemoteName();
896: 
897:   std::vector<c10::Stream> streams;
898:   {
899:     GroupMembershipLockGuard guard(groupMembershipMutex_, isStaticGroup_);
900:     streams = getStreamsFromPoolForDevices(devices_);
901:   }
902:   makeStreamsWaitOnOthers(
903:       streams,
904:       getCurrentStreamsForDevices(
905:           getDevicesOfTensors(requestMessage->tensors())));
906:   pipeWrite(
907:       clientPipe.pipe_,
908:       requestMessage,
909:       std::move(devices),
910:       std::move(streams),
911:       [this, &clientPipe, messageId](const tensorpipe::Error& error) mutable {
912:         if (error) {
```

- EN: Lines 889-912 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 889-912 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 913-936 / 第 913-936 行

```cpp
913:           if (error.isOfType<tensorpipe::PipeClosedError>() &&
914:               !rpcAgentRunning_.load()) {
915:             // This is expected.
916:           } else {
917:             LOG(WARNING) << "RPC agent for " << workerInfo_.name_
918:                          << " encountered error when sending outgoing request #"
919:                          << messageId << " to "
920:                          << clientPipe.pipe_->getRemoteName() << ": "
921:                          << error.what();
922:           }
923:           handleClientError(clientPipe, error);
924:           return;
925:         }
926: 
927:         VLOG(1) << "RPC agent for " << workerInfo_.name_ << " sent request #"
928:                 << messageId << " to " << clientPipe.pipe_->getRemoteName();
929: 
930:         pipeRead(
931:             clientPipe.pipe_,
932:             [this, &clientPipe](
933:                 const tensorpipe::Error& error,
934:                 c10::intrusive_ptr<Message> responseMessage,
935:                 std::vector<c10::Stream> streams) {
936:               if (error) {
```

- EN: Lines 913-936 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 913-936 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 937-960 / 第 937-960 行

```cpp
937:                 if (error.isOfType<tensorpipe::PipeClosedError>() &&
938:                     !rpcAgentRunning_.load()) {
939:                   // This is expected.
940:                 } else {
941:                   LOG(WARNING)
942:                       << "RPC agent for " << workerInfo_.name_
943:                       << " encountered error when reading incoming response from "
944:                       << clientPipe.pipe_->getRemoteName() << ": "
945:                       << error.what();
946:                 }
947:                 handleClientError(clientPipe, error);
948:                 return;
949:               }
950: 
951:               // Identify future response message by message ID
952:               uint64_t messageId = responseMessage->id();
953: 
954:               VLOG(1) << "RPC agent for " << workerInfo_.name_
955:                       << " received response #" << messageId << " from "
956:                       << clientPipe.pipe_->getRemoteName();
957: 
958:               std::shared_ptr<AtomicJitFuture> futureResponseMessage;
959:               {
960:                 std::lock_guard<std::mutex> lock(clientPipe.mutex_);
```

- EN: Lines 937-960 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 937-960 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 961-984 / 第 961-984 行

```cpp
961:                 // A read error will lead all following callbacks to be
962:                 // invoked with error, and shouldn't reach here.
963:                 TORCH_INTERNAL_ASSERT(
964:                     !clientPipe.inError_, "Shouldn't be in error state");
965:                 auto it = clientPipe.pendingResponseMessage_.find(messageId);
966:                 TORCH_INTERNAL_ASSERT(
967:                     it != clientPipe.pendingResponseMessage_.end(),
968:                     "message ID ",
969:                     messageId,
970:                     " is not recognized");
971:                 futureResponseMessage = std::move(it->second);
972:                 clientPipe.pendingResponseMessage_.erase(it);
973:               }
974: 
975:               // Remove entry from timeoutMap_.
976:               removeFromTimeoutMap(messageId);
977: 
978:               if (responseMessage->type() == MessageType::EXCEPTION) {
979:                 markFutureWithError(
980:                     std::move(futureResponseMessage),
981:                     std::string(
982:                         responseMessage->payload().begin(),
983:                         responseMessage->payload().end()));
984:               } else {
```

- EN: Lines 961-984 introduces executable logic in routines such as `TORCH_INTERNAL_ASSERT`.
- CN: 第 961-984 行在 `TORCH_INTERNAL_ASSERT` 等例程中引入具体执行逻辑。

### Lines 985-1008 / 第 985-1008 行

```cpp
985:                 markFutureAsComplete(
986:                     std::move(futureResponseMessage),
987:                     std::move(responseMessage),
988:                     std::move(streams));
989:               }
990:             });
991:       });
992: 
993:   return futureResponseMessage->jitFuture;
994: }
995: 
996: void TensorPipeAgent::handleClientError(
997:     ClientPipe& clientPipe,
998:     const tensorpipe::Error& error) {
999:   // When an error occurs on a pipe all pending operations will be aborted and
1000:   // all callbacks invoked with error, hence we immediately flush all future
1001:   // messages belonging to this pipe.
1002:   decltype(clientPipe.pendingResponseMessage_) pendingMsgs;
1003:   {
1004:     std::lock_guard<std::mutex> lock(clientPipe.mutex_);
1005:     std::swap(clientPipe.pendingResponseMessage_, pendingMsgs);
1006:     clientPipe.inError_ = true;
1007:   }
1008:   std::string errorMsg = error.what();
```

- EN: Lines 985-1008 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 985-1008 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1009-1032 / 第 1009-1032 行

```cpp
1009:   for (auto& p : pendingMsgs) {
1010:     markFutureWithError(std::move(p.second), errorMsg);
1011: 
1012:     // Remove entry from timeoutMap_.
1013:     removeFromTimeoutMap(p.first);
1014:   }
1015: }
1016: 
1017: void TensorPipeAgent::pollTimeoutRpcs() {
1018:   while (rpcAgentRunning_.load()) {
1019:     std::unique_lock<std::mutex> lock(timeoutMapMutex_);
1020: 
1021:     // We sleep until the earliest expiring RPC in the timeoutMap_. We must
1022:     // also ensure that we sleep while the map is empty, and we exit sleeping
1023:     // if the RPC Agent has been shutdown.
1024:     for (;;) {
1025:       if (!rpcAgentRunning_.load()) {
1026:         return;
1027:       }
1028: 
1029:       if (!timeoutMap_.empty()) {
1030:         steady_clock_time_point earliestTimeout = timeoutMap_.begin()->first;
1031:         if (std::chrono::steady_clock::now() >= earliestTimeout) {
1032:           break;
```

- EN: Lines 1009-1032 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1009-1032 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1033-1056 / 第 1033-1056 行

```cpp
1033:         }
1034:         timeoutThreadCV_.wait_until(lock, earliestTimeout);
1035:       } else {
1036:         timeoutThreadCV_.wait(lock);
1037:       }
1038:     }
1039: 
1040:     // Move all these futures to a separate vector so we can process them
1041:     // outside the lock.
1042:     std::vector<TimeoutMessageMetadata> timedOutFutures =
1043:         std::move(timeoutMap_.begin()->second);
1044: 
1045:     // We can safely remove this key from the timeoutMap_ since all these
1046:     // futures will be processed.
1047:     timeoutMap_.erase(timeoutMap_.begin());
1048: 
1049:     for (auto& timeoutMetadata : timedOutFutures) {
1050:       // Remove from messageIdToTimeout map.
1051:       messageIdToTimeout_.erase(timeoutMetadata.messageId);
1052:     }
1053:     lock.unlock();
1054: 
1055:     // Set an error on futures added to the timedOutFutures vector. We do this
1056:     // outside the lock to prevent potential lock-order-inversions by callbacks
```

- EN: Lines 1033-1056 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1033-1056 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1057-1080 / 第 1057-1080 行

```cpp
1057:     // triggered by the setError call.
1058:     for (auto& timeoutMetadata : timedOutFutures) {
1059:       std::string errorMsg =
1060:           fmt::format(kRpcTimeoutErrorStr, timeoutMetadata.timeout.count());
1061:       auto err = makeRPCError(errorMsg, RPCErrorType::TIMEOUT);
1062:       markFutureWithError(
1063:           std::move(timeoutMetadata.responseFuture), std::move(err));
1064:     }
1065:   }
1066: }
1067: 
1068: void TensorPipeAgent::leaveGroup() {
1069:   std::unique_lock<std::mutex> lock(callCountMutex_);
1070:   // local worker ActiveCallCount is 0 at this point and we will shutdown
1071:   // (any future calls will be dropped)
1072:   callCountCV_.wait(lock, [this] { return clientActiveCalls_ == 0; });
1073: 
1074:   // Remove this agent's WorkerInfo from store
1075:   removeCurrentName(rankToNameStore_, workerInfo_.id_, workerInfo_.name_);
1076: 
1077:   // Set internal variable to be used during destructor
1078:   shuttingDown_ = true;
1079: }
1080: 
```

- EN: Lines 1057-1080 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1057-1080 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1081-1104 / 第 1081-1104 行

```cpp
1081: // TODO: Remove join()
1082: void TensorPipeAgent::join(bool shutdown, float /* unused */) {
1083:   VLOG(1) << "RPC agent for " << workerInfo_.name_ << " is joining";
1084:   if (!isStaticGroup_) {
1085:     leaveGroup();
1086:     return;
1087:   }
1088: 
1089:   // This method behaves like a barrier, as it can only return once all workers
1090:   // have no more requests pending, including "nested" requests (triggered from
1091:   // within the remote code of another call) and "follow-up" requests (triggered
1092:   // from the callback of a future).
1093:   while (true) {
1094:     {
1095:       std::unique_lock<std::mutex> lock(callCountMutex_);
1096:       // It is enough to wait for there to be no more active client calls, since
1097:       // each server call corresponds to a client call for some other worker.
1098:       callCountCV_.wait(lock, [this] { return clientActiveCalls_ == 0; });
1099: 
1100:       // We'd like to immediately proceed with the allreduce, but it's a call
1101:       // that may block for some time, as it waits for other workers to also
1102:       // complete all their active client calls. While we call allreduce we must
1103:       // hold the mutex, or else the count we send to other workers may get
1104:       // stale (e.g., if some nested call happens in the meantime). But we can't
```

- EN: Lines 1081-1104 introduces executable logic in routines such as `leaveGroup`; returns computed state or forwards results to the surrounding caller.
- CN: 第 1081-1104 行在 `leaveGroup` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1105-1128 / 第 1105-1128 行

```cpp
1105:       // hold the lock for an indeterminately long time, as that would block
1106:       // other operations (e.g., send). Thus we must release the lock and only
1107:       // re-acquire it when all workers are ready to proceed with the allreduce.
1108:       // We perform this synchronization using a barrier.
1109:     }
1110:     VLOG(1) << "RPC agent for " << workerInfo_.name_
1111:             << " completed all client calls and is entering a barrier";
1112:     syncCallCount(shutdownStore_, worldSize_);
1113:     {
1114:       std::unique_lock<std::mutex> lock(callCountMutex_);
1115:       // At this point, the count may have become non-zero again. We can't wait
1116:       // for those calls to complete as other workers are waiting for us in the
1117:       // allreduce and we would block them. Thus we send our count even if it is
1118:       // non-zero and if anyone (be it us or another worker) has a non-zero
1119:       // count we'll just do another round.
1120:       VLOG(1) << "RPC agent for " << workerInfo_.name_
1121:               << " exited the barrier and found " << clientActiveCalls_
1122:               << " active client calls";
1123:       int totalClientActiveCalls =
1124:           syncCallCount(shutdownStore_, worldSize_, clientActiveCalls_);
1125:       VLOG(1) << "RPC agent for " << workerInfo_.name_
1126:               << " completed sync call counts and got a total of "
1127:               << totalClientActiveCalls
1128:               << " active client calls across all workers";
```

- EN: Lines 1105-1128 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1105-1128 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1129-1152 / 第 1129-1152 行

```cpp
1129:       if (totalClientActiveCalls == 0) {
1130:         if (shutdown) {
1131:           shuttingDown_ = true;
1132:           syncCallCount(shutdownStore_, worldSize_);
1133:         }
1134:         break;
1135:       }
1136:     }
1137:   }
1138:   VLOG(1) << "RPC agent for " << workerInfo_.name_ << " done joining";
1139: }
1140: 
1141: void TensorPipeAgent::shutdownImpl() {
1142:   // FIXME Isn't it too verbose for a library to print logs in normal operation?
1143:   LOG(INFO) << "RPC agent for " << workerInfo_.name_ << " is shutting down";
1144: 
1145:   // Join the Timeout Thread
1146:   timeoutThreadCV_.notify_one();
1147:   if (timeoutThread_.joinable()) {
1148:     timeoutThread_.join();
1149:   }
1150:   VLOG(1) << "RPC agent for " << workerInfo_.name_
1151:           << " done waiting for timeout thread to join";
1152: 
```

- EN: Lines 1129-1152 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1129-1152 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1153-1176 / 第 1153-1176 行

```cpp
1153:   // This will close all the pipes and listeners, invoke all callbacks with
1154:   // errors, turn down the I/O event loops and wait for everything to terminate.
1155:   context_->join();
1156:   VLOG(1) << "RPC agent for " << workerInfo_.name_
1157:           << " done waiting for TensorPipe context to join";
1158: 
1159:   // NOTE: We need to call waitWorkComplete in the end after we have shutdown
1160:   // all listeners for Tensorpipe. This is to drain any already accepted work
1161:   // in the ThreadPool. If this is done before we shutdown the listeners,
1162:   // additional work could be added after this call and before we shutdown
1163:   // listeners. This work would continue executing in the threadpool and might
1164:   // cause issues during shutdown of the system.
1165:   threadPool_.waitWorkComplete();
1166:   VLOG(1) << "RPC agent for " << workerInfo_.name_
1167:           << " done waiting for thread pool to complete work";
1168: }
1169: 
1170: const WorkerInfo& TensorPipeAgent::getWorkerInfo(
1171:     const std::string& workerName) const {
1172:   std::unordered_map<std::string, WorkerInfo>::const_iterator it;
1173:   {
1174:     GroupMembershipLockGuard guard(groupMembershipMutex_, isStaticGroup_);
1175:     it = workerNameToInfo_.find(workerName);
1176:   }
```

- EN: Lines 1153-1176 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1153-1176 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1177-1200 / 第 1177-1200 行

```cpp
1177:   TORCH_CHECK(
1178:       it != workerNameToInfo_.end(),
1179:       fmt::format(
1180:           "name:{},rank:{} could not find destination name {}",
1181:           workerInfo_.name_,
1182:           workerInfo_.id_,
1183:           workerName));
1184:   return it->second;
1185: }
1186: 
1187: const WorkerInfo& TensorPipeAgent::getWorkerInfo(worker_id_t workerId) const {
1188:   std::unordered_map<worker_id_t, WorkerInfo>::const_iterator it;
1189:   {
1190:     GroupMembershipLockGuard guard(groupMembershipMutex_, isStaticGroup_);
1191:     it = workerIdToInfo_.find(workerId);
1192:   }
1193:   TORCH_CHECK(
1194:       it != workerIdToInfo_.end(),
1195:       fmt::format(
1196:           "name:{},rank:{} could not find destination id {}",
1197:           workerInfo_.name_,
1198:           workerInfo_.id_,
1199:           workerId));
1200:   return it->second;
```

- EN: Lines 1177-1200 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 1177-1200 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 1201-1224 / 第 1201-1224 行

```cpp
1201: }
1202: 
1203: std::vector<WorkerInfo> TensorPipeAgent::getWorkerInfos() const {
1204:   std::vector<WorkerInfo> workerInfos;
1205:   workerInfos.reserve(workerNameToInfo_.size());
1206:   for (auto& item : workerNameToInfo_) {
1207:     workerInfos.emplace_back(item.second);
1208:   }
1209:   return workerInfos;
1210: }
1211: 
1212: const std::string& TensorPipeAgent::findWorkerURL(
1213:     const WorkerInfo& worker) const {
1214:   std::unordered_map<std::string, std::string>::const_iterator it;
1215:   {
1216:     GroupMembershipLockGuard guard(groupMembershipMutex_, isStaticGroup_);
1217:     it = workerNameToURL_.find(worker.name_);
1218:   }
1219:   TORCH_CHECK(
1220:       it != workerNameToURL_.end(),
1221:       fmt::format(
1222:           "name:{},rank:{} could not find destination url for name {}",
1223:           workerInfo_.name_,
1224:           workerInfo_.id_,
```

- EN: Lines 1201-1224 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 1201-1224 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 1225-1248 / 第 1225-1248 行

```cpp
1225:           worker.name_));
1226:   return it->second;
1227: }
1228: 
1229: void TensorPipeAgent::updateGroupMembership(
1230:     const WorkerInfo& workerInfo,
1231:     const std::vector<c10::Device>& devices,
1232:     const std::unordered_map<std::string, DeviceMap>& reverseDeviceMaps,
1233:     bool isJoin) {
1234:   std::string name = workerInfo.name_;
1235:   worker_id_t id = workerInfo.id_;
1236:   // Rank with workerInfo is joining the group, update internal mappings
1237:   if (isJoin) {
1238:     GroupMembershipLockGuard guard(groupMembershipMutex_, isStaticGroup_);
1239:     workerIdToInfo_.emplace(id, workerInfo);
1240:     workerNameToInfo_.emplace(name, workerInfo);
1241: 
1242:     // TODO: we should get nodeAddrStr in the joining process, then pass in as
1243:     // an argument rather than getting from store each time
1244:     auto nodeAddrData = nameToAddressStore_.get(name);
1245:     auto nodeAddrStr = std::string(
1246:         reinterpret_cast<const char*>(nodeAddrData.data()),
1247:         nodeAddrData.size());
1248:     workerNameToURL_.insert({name, nodeAddrStr});
```

- EN: Lines 1225-1248 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1225-1248 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1249-1272 / 第 1249-1272 行

```cpp
1249: 
1250:     for (const auto& it : reverseDeviceMaps) {
1251:       if (reverseDeviceMaps_.find(it.first) == reverseDeviceMaps_.end()) {
1252:         reverseDeviceMaps_[it.first] = it.second;
1253:       }
1254:     }
1255:     // TODO: clean up mutex for devices_ usage
1256:     // Add devices that have not been added yet
1257:     for (const auto& it : devices) {
1258:       if (std::find(devices_.begin(), devices_.end(), it) == devices_.end()) {
1259:         devices_.push_back(it);
1260:       }
1261:     }
1262:   } else {
1263:     workerIdToInfo_.erase(id);
1264:     workerNameToInfo_.erase(name);
1265:     workerNameToURL_.erase(name);
1266: 
1267:     // remove reverse device maps that are no longer used
1268:     std::erase_if(reverseDeviceMaps_, [&reverseDeviceMaps](const auto& kv) {
1269:       return !reverseDeviceMaps.contains(kv.first);
1270:     });
1271: 
1272:     // remove devices that are no longer used
```

- EN: Lines 1249-1272 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1249-1272 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1273-1296 / 第 1273-1296 行

```cpp
1273:     std::erase_if(devices_, [&](const auto& d) {
1274:       return std::find(devices.begin(), devices.end(), d) == devices.end();
1275:     });
1276:   }
1277: }
1278: std::unordered_map<std::string, std::string> TensorPipeAgent::getMetrics() {
1279:   std::unordered_map<std::string, std::string> metrics;
1280:   metrics[kThreadPoolSize] = std::to_string(threadPool_.size());
1281:   metrics[kNumIdleThreads] = std::to_string(threadPool_.numAvailable());
1282:   {
1283:     std::unique_lock<std::mutex> lock(callCountMutex_);
1284:     metrics[kClientActiveCalls] = std::to_string(clientActiveCalls_);
1285:     metrics[kServerActiveCalls] = std::to_string(serverActiveCalls_);
1286:     metrics[kServerActiveAsyncCalls] = std::to_string(serverActiveAsyncCalls_);
1287:   }
1288:   if (isGILProfilingEnabled()) {
1289:     {
1290:       std::unique_lock<std::mutex> lock(metricsMutex_);
1291:       // Include the averages for each time series metric. This is just the GIL
1292:       // Wait Time for now.
1293:       auto averageGilWaitTime =
1294:           timeSeriesMetrics_[kGilAverageWaitTime].computeAverage();
1295:       lock.unlock();
1296:       metrics[kGilAverageWaitTime] = std::to_string(averageGilWaitTime);
```

- EN: Lines 1273-1296 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1273-1296 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1297-1320 / 第 1297-1320 行

```cpp
1297:     }
1298:   }
1299: 
1300:   return metrics;
1301: }
1302: 
1303: void TensorPipeAgent::addGilWaitTime(
1304:     const std::chrono::microseconds gilWaitTime) {
1305:   std::lock_guard<std::mutex> lock(metricsMutex_);
1306:   timeSeriesMetrics_[kGilAverageWaitTime].addData(gilWaitTime.count());
1307: }
1308: 
1309: TensorPipeAgent::NetworkDataDict TensorPipeAgent::getNetworkData() {
1310:   std::lock_guard<std::mutex> lock(networkDataMutex_);
1311:   return networkData_;
1312: }
1313: 
1314: NetworkSourceInfo TensorPipeAgent::getNetworkSourceInfo() {
1315:   NetworkSourceInfo info = {
1316:       RpcAgent::getWorkerInfo().id_,
1317:       nameToAddressStore_.get(RpcAgent::getWorkerInfo().name_)};
1318: 
1319:   return info;
1320: }
```

- EN: Lines 1297-1320 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1297-1320 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1321-1344 / 第 1321-1344 行

```cpp
1321: 
1322: void TensorPipeAgent::trackNetworkData(
1323:     uint64_t requestSize,
1324:     uint64_t responseSize,
1325:     const std::string& destWorkerName) {
1326:   std::lock_guard<std::mutex> lock(networkDataMutex_);
1327:   networkData_[destWorkerName].numCalls++;
1328:   networkData_[destWorkerName].totalSentBytes += requestSize;
1329:   networkData_[destWorkerName].totalRecvBytes += responseSize;
1330: }
1331: 
1332: void TensorPipeAgent::trackNetworkError(
1333:     uint64_t requestSize,
1334:     const std::string& destWorkerName) {
1335:   std::lock_guard<std::mutex> lock(networkDataMutex_);
1336:   networkData_[destWorkerName].numCalls++;
1337:   networkData_[destWorkerName].totalSentBytes += requestSize;
1338:   networkData_[destWorkerName].totalErrors++;
1339: }
1340: 
1341: void TensorPipeAgent::increaseCallCount(int32_t& count) {
1342:   {
1343:     std::unique_lock<std::mutex> lock(callCountMutex_);
1344:     ++count;
```

- EN: Lines 1321-1344 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1321-1344 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1345-1368 / 第 1345-1368 行

```cpp
1345:   }
1346:   callCountCV_.notify_all();
1347: }
1348: 
1349: void TensorPipeAgent::decreaseCallCount(int32_t& count) {
1350:   {
1351:     std::unique_lock<std::mutex> lock(callCountMutex_);
1352:     --count;
1353:   }
1354:   callCountCV_.notify_all();
1355: }
1356: 
1357: void TensorPipeAgent::markFutureAsComplete(
1358:     std::shared_ptr<AtomicJitFuture> atomicFuture,
1359:     c10::intrusive_ptr<Message> message,
1360:     std::vector<c10::Stream> streams) {
1361:   if (!atomicFuture->isComplete.test_and_set()) {
1362:     // Completing the future will run its callbacks, which could execute
1363:     // arbitrary user code. To prevent blocking or stalling the TensorPipe event
1364:     // loops, we defer this to a worker thread.
1365:     threadPool_.run([this,
1366:                      atomicFuture{std::move(atomicFuture)},
1367:                      message{std::move(message)},
1368:                      streams{std::move(streams)}]() mutable {
```

- EN: Lines 1345-1368 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1345-1368 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1369-1392 / 第 1369-1392 行

```cpp
1369:       c10::MultiStreamGuard guard(streams);
1370:       std::vector<c10::weak_intrusive_ptr<c10::StorageImpl>> storages =
1371:           message->getStorages();
1372:       atomicFuture->jitFuture->markCompleted(
1373:           std::move(message), std::move(storages));
1374:       // The future's callbacks may schedule further RPCs, increasing the count.
1375:       // Thus we must decrease it after completing the future, otherwise it may
1376:       // briefly dip to zero and trick join into thinking all work is done.
1377:       decreaseCallCount(clientActiveCalls_);
1378:     });
1379:   }
1380: }
1381: 
1382: void TensorPipeAgent::markFutureWithError(
1383:     std::shared_ptr<AtomicJitFuture> atomicFuture,
1384:     std::string errorMsg) {
1385:   if (!atomicFuture->isComplete.test_and_set()) {
1386:     // Completing the future will run its callbacks, which could execute
1387:     // arbitrary user code. To prevent blocking or stalling the TensorPipe event
1388:     // loops, we defer this to a worker thread.
1389:     threadPool_.run([this,
1390:                      atomicFuture{std::move(atomicFuture)},
1391:                      errorMsg{std::move(errorMsg)}]() mutable {
1392:       atomicFuture->jitFuture->setError(
```

- EN: Lines 1369-1392 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1369-1392 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1393-1416 / 第 1393-1416 行

```cpp
1393:           std::make_exception_ptr(std::runtime_error(errorMsg)));
1394:       // The future's callbacks may schedule further RPCs, increasing the count.
1395:       // Thus we must decrease it after completing the future, otherwise it may
1396:       // briefly dip to zero and trick join into thinking all work is done.
1397:       decreaseCallCount(clientActiveCalls_);
1398:     });
1399:   }
1400: }
1401: 
1402: std::vector<c10::Device> TensorPipeAgent::getDevicesForRemote(
1403:     const std::string& remoteName,
1404:     const Message& message) const {
1405:   std::unordered_map<std::string, DeviceMap> deviceMaps;
1406:   {
1407:     GroupMembershipLockGuard guard(groupMembershipMutex_, isStaticGroup_);
1408:     deviceMaps = message.isRequest() ? opts_.deviceMaps : reverseDeviceMaps_;
1409:   }
1410: 
1411:   const auto errStr = c10::str(
1412:       "TensorPipe RPC backend only supports CPU tensors by default, please "
1413:       "move your tensors to CPU before sending them over RPC, or call "
1414:       "`set_device_map` on `TensorPipeRpcBackendOptions` to explicitly "
1415:       "configure device mapping. ",
1416:       message.isRequest() ? "Request" : "Response",
```

- EN: Lines 1393-1416 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1393-1416 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 1417-1440 / 第 1417-1440 行

```cpp
1417:       " device mapping is not available for destination ",
1418:       remoteName);
1419: 
1420:   const auto& iter = deviceMaps.find(remoteName);
1421:   if (iter == deviceMaps.end()) {
1422:     for (const auto& t : message.tensors()) {
1423:       TORCH_CHECK(
1424:           t.device().is_cpu(),
1425:           errStr,
1426:           ", but found tensor on device: ",
1427:           t.device());
1428:     }
1429:     return {};
1430:   } else {
1431:     return getDevicesForTensors(message.tensors(), iter->second, errStr);
1432:   }
1433: }
1434: 
1435: DeviceMap TensorPipeAgent::getDeviceMap(const WorkerInfo& dst) const {
1436:   auto it = opts_.deviceMaps.find(dst.name_);
1437:   if (it == opts_.deviceMaps.end()) {
1438:     return {};
1439:   }
1440:   return it->second;
```

- EN: Lines 1417-1440 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 1417-1440 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 1441-1464 / 第 1441-1464 行

```cpp
1441: }
1442: 
1443: const c10::intrusive_ptr<::c10d::Store> TensorPipeAgent::getStore() const {
1444:   return store_;
1445: }
1446: 
1447: TensorPipeRpcBackendOptions TensorPipeAgent::getBackendOptions() const {
1448:   return opts_;
1449: }
1450: 
1451: const std::vector<c10::Device>& TensorPipeAgent::getDevices() const {
1452:   GroupMembershipLockGuard guard(groupMembershipMutex_, isStaticGroup_);
1453:   return devices_;
1454: }
1455: 
1456: size_t TensorPipeAgent::timeoutMapSize() {
1457:   std::unique_lock<std::mutex> lock(timeoutMapMutex_);
1458:   return timeoutMap_.size();
1459: }
1460: 
1461: size_t TensorPipeAgent::numPendingResponses() {
1462:   std::unique_lock<std::mutex> lock(callCountMutex_);
1463:   return clientActiveCalls_;
1464: }
```

- EN: Lines 1441-1464 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 1441-1464 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 1465-1473 / 第 1465-1473 行

```cpp
1465: 
1466: size_t TensorPipeAgent::messageIdToTimeoutMapSize() {
1467:   std::unique_lock<std::mutex> lock(timeoutMapMutex_);
1468:   return messageIdToTimeout_.size();
1469: }
1470: 
1471: } // namespace torch::distributed::rpc
1472: 
1473: #endif // USE_TENSORPIPE
```

- EN: Lines 1465-1473 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 1465-1473 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

## Key Concepts / 关键概念

- EN: Subsystem: distributed RPC layer.
- CN: 子系统：分布式 RPC 层。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `getDevicesForTensors`, `getStreamsFromPoolForDevices`, `getCurrentStreamsForDevices`, `getDevicesOfTensors`, `makeStreamsWaitOnOthers`, `makeUvTransport`
- CN: 核心符号：`getDevicesForTensors`、`getStreamsFromPoolForDevices`、`getCurrentStreamsForDevices`、`getDevicesOfTensors`、`makeStreamsWaitOnOthers`、`makeUvTransport`
- EN: Notable themes: store/state coordination, RPC/RRef semantics, collective communication logic.
- CN: 值得关注的主题：存储/状态协调、RPC/RRef 语义、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/rpc/tensorpipe_agent.h`, `torch/csrc/distributed/rpc/agent_utils.h`, `torch/csrc/distributed/rpc/tensorpipe_utils.h`, `torch/csrc/distributed/rpc/utils.h`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `c10/core/StreamGuard.h`, `c10/util/irange.h`
- External or system headers / 外部或系统头文件: `limits`, `tuple`, `utility`, `fmt/format.h`, `tensorpipe/tensorpipe.h`
- Local symbols / 本地符号: `getDevicesForTensors`, `getStreamsFromPoolForDevices`, `getCurrentStreamsForDevices`, `getDevicesOfTensors`, `makeStreamsWaitOnOthers`, `makeUvTransport`, `makeShmTransport`, `makeIbvTransport`