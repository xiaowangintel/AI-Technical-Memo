# CUDASymmetricMemory.cu — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemory.cu`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides implementation logic for cudasymmetric memory in the c10d symmetric-memory support. Key types include `RendezvousRequest`, `RegisterCUDASymmetricMemoryAllocator`. GPU-oriented code paths are present in this file.
- 用途 (CN): 该文件在c10d 对称内存支持中提供cudasymmetric memory 的实现逻辑。 关键类型包括 `RendezvousRequest`、`RegisterCUDASymmetricMemoryAllocator`。 该文件还包含面向 GPU 的代码路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24 / 第 1-24 行

```cpp
1: #include <torch/csrc/distributed/c10d/GroupRegistry.hpp>
2: #include <torch/csrc/distributed/c10d/ParamCommsUtils.hpp>
3: #include <torch/csrc/distributed/c10d/cuda/utils.hpp>
4: #include <torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemory-inl.cuh>
5: #include <torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemory.hpp>
6: #include <torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryUtils.hpp>
7: 
8: #include <ATen/ceil_div.h>
9: #include <ATen/cuda/CUDAContext.h>
10: #include <ATen/cuda/PeerToPeerAccess.h>
11: #include <c10/cuda/CUDACachingAllocator.h>
12: #include <c10/cuda/CUDAGuard.h>
13: #include <c10/util/env.h>
14: #include <c10/util/error.h>
15: 
16: #include <sys/socket.h>
17: #include <unistd.h>
18: 
19: #if !defined(USE_ROCM) && defined(PYTORCH_C10_DRIVER_API_SUPPORTED)
20: #include <c10/cuda/driver_api.h>
21: #elif defined(USE_ROCM)
22: #include <hip/hip_runtime_api.h>
23: #endif
24: 
```

- EN: Lines 1-24 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 1-24 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端；包含面向 CUDA 的声明、内核或启动流程。

### Lines 25-48 / 第 25-48 行

```cpp
25: #if defined(CUDART_VERSION) && CUDART_VERSION >= 12030
26: #define CUDART_SUPPORTS_MULTICAST
27: #endif
28: 
29: namespace c10d::symmetric_memory {
30: 
31: /* Start of CUDASymmetricMemory implementation */
32: 
33: // A set of exchange methods with prefix "CUDASymmetricMemory"
34: static StoreExchange storeExchange = StoreExchange("CUDASymmetricMemory");
35: 
36: AllocationRef::AllocationRef(
37:     void* ptr,
38:     HandleType handle,
39:     size_t block_size,
40:     int device_idx,
41:     bool is_multicast)
42:     : ptr(ptr),
43:       handle(handle),
44:       block_size(block_size),
45:       device_idx(device_idx),
46:       is_multicast(is_multicast) {}
47: 
48: AllocationRef::~AllocationRef() {
```

- EN: Lines 25-48 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 25-48 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 49-72 / 第 49-72 行

```cpp
49:   if (is_finalizing()) {
50:     return;
51:   }
52:   c10::cuda::CUDAGuard guard(device_idx);
53:   C10_CUDA_CHECK(cudaDeviceSynchronize());
54: #if !defined(USE_ROCM) && defined(PYTORCH_C10_DRIVER_API_SUPPORTED)
55:   // Leak the cuda allocations during static deinitialization
56:   auto driver_api = c10::cuda::DriverAPI::get();
57:   C10_CUDA_DRIVER_CHECK(
58:       driver_api->cuMemUnmap_(reinterpret_cast<CUdeviceptr>(ptr), block_size));
59: #if defined(CUDART_SUPPORTS_MULTICAST)
60:   if (is_multicast) {
61:     C10_CUDA_DRIVER_CHECK(
62:         driver_api->cuMulticastUnbind_(handle, device_idx, 0, block_size));
63:   }
64: #endif
65:   C10_CUDA_DRIVER_CHECK(driver_api->cuMemRelease_(handle));
66: #elif defined(USE_ROCM)
67:   C10_CUDA_CHECK(hipMemUnmap(reinterpret_cast<hipDeviceptr_t>(ptr), block_size));
68:   C10_CUDA_CHECK(hipMemRelease(handle));
69: #else
70:   TORCH_CHECK(
71:       false, "CUDASymmetricMemory requires PYTORCH_C10_DRIVER_API_SUPPORTED");
72: #endif
```

- EN: Lines 49-72 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 49-72 行使用条件编译来适配特性开关、平台或可选后端；在 `TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 73-96 / 第 73-96 行

```cpp
73: }
74: 
75: CUDAPeerAllocInfo::CUDAPeerAllocInfo(
76:     std::vector<c10::intrusive_ptr<AllocationRef>> alloc_refs,
77:     std::vector<void*> buffers,
78:     std::vector<void*> signal_pads,
79:     HandleType mc_handle,
80:     void* mc_addr,
81:     size_t buffer_size,
82:     int local_device_idx,
83:     int rank,
84:     int world_size,
85:     std::string group_name)
86:     : alloc_refs_(std::move(alloc_refs)),
87:       buffers_(std::move(buffers)),
88:       signal_pads_(std::move(signal_pads)),
89:       mc_handle_(mc_handle),
90:       mc_addr_(mc_addr),
91:       buffer_size_(buffer_size),
92:       local_device_idx_(local_device_idx),
93:       rank_(rank),
94:       world_size_(world_size),
95:       group_name_(std::move(group_name)) {
96:   const size_t arr_size = sizeof(void*) * world_size_;
```

- EN: Lines 73-96 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 73-96 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 97-120 / 第 97-120 行

```cpp
97:   buffers_dev_ = reinterpret_cast<void**>(
98:       c10::cuda::CUDACachingAllocator::raw_alloc(arr_size));
99:   signal_pads_dev_ = reinterpret_cast<void**>(
100:       c10::cuda::CUDACachingAllocator::raw_alloc(arr_size));
101: 
102:   c10::cuda::CUDAGuard guard(local_device_idx);
103:   AT_CUDA_CHECK(cudaMemcpy(
104:       buffers_dev_, buffers_.data(), arr_size, cudaMemcpyHostToDevice));
105:   AT_CUDA_CHECK(cudaMemcpy(
106:       signal_pads_dev_, signal_pads_.data(), arr_size, cudaMemcpyHostToDevice));
107: }
108: 
109: /* Start of CUDASymmetricMemory */
110: 
111: // This is mostly a shallow copy that shares the pointer to `CUDAPeerAllocInfo`
112: // which corresponds to the base Block. The CUDASymmetricMemory handle is
113: // specified by the offset to the base ptr.
114: CUDASymmetricMemory::CUDASymmetricMemory(const c10::intrusive_ptr<CUDAPeerAllocInfo>& pai, size_t offset)
115:     : local_device_idx_(pai->local_device_idx_),
116:       rank_(pai->rank_),
117:       world_size_(pai->world_size_),
118:       pai_(pai),
119:       offset_(offset) {
120:   // offset is specific per symm_mem handle
```

- EN: Lines 97-120 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 97-120 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 121-144 / 第 121-144 行

```cpp
121:   TORCH_INTERNAL_ASSERT(offset_ < pai_->buffer_size_, "offset out of range");
122: }
123: 
124: std::vector<void*> CUDASymmetricMemory::get_buffer_ptrs() {
125:   return pai_->buffers_;
126: }
127: 
128: std::vector<void*> CUDASymmetricMemory::get_signal_pad_ptrs() {
129:   return pai_->signal_pads_;
130: }
131: 
132: void** CUDASymmetricMemory::get_buffer_ptrs_dev() {
133:   return pai_->buffers_dev_;
134: }
135: 
136: void** CUDASymmetricMemory::get_signal_pad_ptrs_dev() {
137:   return pai_->signal_pads_dev_;
138: }
139: 
140: size_t CUDASymmetricMemory::get_buffer_size() {
141:   return pai_->buffer_size_;
142: }
143: 
144: bool CUDASymmetricMemory::has_multicast_support() {
```

- EN: Lines 121-144 introduces executable logic in routines such as `TORCH_INTERNAL_ASSERT`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 121-144 行在 `TORCH_INTERNAL_ASSERT` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 145-168 / 第 145-168 行

```cpp
145:   return pai_->mc_addr_ != nullptr;
146: }
147: 
148: void* CUDASymmetricMemory::get_multicast_ptr() {
149:   if (!has_multicast_support()) {
150:     return nullptr;
151:   }
152:   return static_cast<char*>(pai_->mc_addr_) + offset_;
153: }
154: 
155: size_t CUDASymmetricMemory::get_offset() {
156:   return offset_;
157: }
158: 
159: void check_channel(int channel, int world_size) {
160:   TORCH_CHECK(
161:       channel >= 0,
162:       "channel for barrier(), put_signal() and wait_signal() ",
163:       "must be greater than 0 (got ",
164:       channel,
165:       ")");
166:   const size_t num_channels = c10d::symmetric_memory::get_signal_pad_size() /
167:       sizeof(uint32_t) * world_size;
168:   TORCH_CHECK(
```

- EN: Lines 145-168 introduces executable logic in routines such as `check_channel`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 145-168 行在 `check_channel` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 169-192 / 第 169-192 行

```cpp
169:       static_cast<size_t>(channel) < num_channels,
170:       "The maximum supported channel for barrier(), put_signal() and wait_signal() is ",
171:       num_channels - 1,
172:       " (got ",
173:       channel,
174:       ")");
175: }
176: 
177: static __global__ void barrier_kernel(
178:     uint32_t** signal_pads,
179:     int channel,
180:     int rank,
181:     int world_size,
182:     size_t timeout_ms) {
183:   if (threadIdx.x < world_size) {
184:     auto target_rank = threadIdx.x;
185:     if (target_rank == rank) {
186:       return;
187:     }
188:     auto put_success = try_put_signal<std::memory_order_release>(
189:         signal_pads[target_rank] + world_size * channel + rank, timeout_ms);
190:     if (!put_success) {
191:       printf(
192:           "[FATAL] CUDASymmetricMemory::barrier: rank %d failed to send signal "
```

- EN: Lines 169-192 introduces executable logic in routines such as `barrier_kernel`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 169-192 行在 `barrier_kernel` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 193-216 / 第 193-216 行

```cpp
193:           "to rank %d on channel %d after %lu microseconds\n",
194:           rank,
195:           target_rank,
196:           channel,
197:           timeout_ms);
198:       trap();
199:     }
200:     auto wait_success = try_wait_signal<std::memory_order_acquire>(
201:         signal_pads[rank] + world_size * channel + target_rank, timeout_ms);
202:     if (!wait_success) {
203:       printf(
204:           "[FATAL] CUDASymmetricMemory::barrier: rank %d failed to receive signal "
205:           "from rank %d on channel %d after %lu microseconds\n",
206:           rank,
207:           target_rank,
208:           channel,
209:           timeout_ms);
210:       trap();
211:     }
212:   }
213: }
214: 
215: void CUDASymmetricMemory::barrier(int channel, size_t timeout_ms) {
216:   check_channel(channel, world_size_);
```

- EN: Lines 193-216 introduces executable logic in routines such as `trap`, `printf`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 193-216 行在 `trap`、`printf` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 217-240 / 第 217-240 行

```cpp
217:   auto pg = c10d::resolve_process_group(pai_->group_name_);
218:   RECORD_PARAM_COMMS(
219:       static_cast<int64_t>(0),
220:       std::make_tuple(pg->getGroupName(), pg->getGroupDesc()),
221:       rank_,
222:       "symm_mem::barrier",
223:       0,
224:       0,
225:       at::kByte,
226:       std::vector<int64_t>(),
227:       std::vector<int64_t>(),
228:       -1,
229:       -1,
230:       world_size_);
231:   c10::cuda::CUDAGuard device_guard(local_device_idx_);
232:   barrier_kernel<<<
233:       1,
234:       max(at::cuda::warp_size(), world_size_),
235:       0,
236:       at::cuda::getCurrentCUDAStream()>>>(
237:       reinterpret_cast<uint32_t**>(pai_->signal_pads_dev_),
238:       channel,
239:       rank_,
240:       world_size_,
```

- EN: Lines 217-240 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 217-240 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 241-264 / 第 241-264 行

```cpp
241:       timeout_ms);
242:   C10_CUDA_KERNEL_LAUNCH_CHECK();
243: }
244: 
245: static __global__ void put_signal_kernel(
246:     uint32_t** signal_pads,
247:     int dst_rank,
248:     int channel,
249:     int rank,
250:     int world_size,
251:     size_t timeout_ms) {
252:   if (threadIdx.x == 0) {
253:     bool success = try_put_signal<std::memory_order_release>(
254:         signal_pads[dst_rank] + world_size * channel + rank, timeout_ms);
255:     if (!success) {
256:       printf(
257:           "[FATAL] CUDASymmetricMemory::put_signal: rank %d failed to send signal "
258:           "to rank %d on channel %d after %lu microseconds\n",
259:           rank,
260:           dst_rank,
261:           channel,
262:           timeout_ms);
263:       trap();
264:     }
```

- EN: Lines 241-264 introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`, `put_signal_kernel`, `printf`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 241-264 行在 `C10_CUDA_KERNEL_LAUNCH_CHECK`、`put_signal_kernel`、`printf` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 265-288 / 第 265-288 行

```cpp
265:   }
266: }
267: 
268: void CUDASymmetricMemory::put_signal(
269:     int dst_rank,
270:     int channel,
271:     size_t timeout_ms) {
272:   check_channel(channel, world_size_);
273:   auto pg = c10d::resolve_process_group(pai_->group_name_);
274:   RECORD_PARAM_COMMS(
275:       static_cast<int64_t>(0),
276:       std::make_tuple(pg->getGroupName(), pg->getGroupDesc()),
277:       rank_,
278:       "symm_mem::put_signal",
279:       0,
280:       0,
281:       at::kByte,
282:       std::vector<int64_t>(),
283:       std::vector<int64_t>(),
284:       -1,
285:       -1,
286:       world_size_);
287:   c10::cuda::CUDAGuard device_guard(local_device_idx_);
288:   put_signal_kernel<<<
```

- EN: Lines 265-288 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 265-288 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 289-312 / 第 289-312 行

```cpp
289:       1,
290:       at::cuda::warp_size(),
291:       0,
292:       at::cuda::getCurrentCUDAStream()>>>(
293:       reinterpret_cast<uint32_t**>(pai_->signal_pads_dev_),
294:       dst_rank,
295:       channel,
296:       rank_,
297:       world_size_,
298:       timeout_ms);
299:   C10_CUDA_KERNEL_LAUNCH_CHECK();
300: }
301: 
302: static __global__ void wait_signal_kernel(
303:     uint32_t** signal_pads,
304:     int src_rank,
305:     int channel,
306:     int rank,
307:     int world_size,
308:     size_t timeout_ms) {
309:   if (threadIdx.x == 0) {
310:     bool success = try_wait_signal<std::memory_order_acquire>(
311:         signal_pads[rank] + world_size * channel + src_rank, timeout_ms);
312:     if (!success) {
```

- EN: Lines 289-312 introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`, `wait_signal_kernel`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 289-312 行在 `C10_CUDA_KERNEL_LAUNCH_CHECK`、`wait_signal_kernel` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 313-336 / 第 313-336 行

```cpp
313:       printf(
314:           "[FATAL] CUDASymmetricMemory::wait_signal rank %d failed to receive signal "
315:           "from rank %d on channel %d after %lu microseconds\n",
316:           rank,
317:           src_rank,
318:           channel,
319:           timeout_ms);
320: #if !defined(USE_ROCM)
321:       __trap();
322: #else
323:       assert(0);
324: #endif
325:     }
326:   }
327:   __threadfence_system();
328: }
329: 
330: void CUDASymmetricMemory::wait_signal(
331:     int src_rank,
332:     int channel,
333:     size_t timeout_ms) {
334:   check_channel(channel, world_size_);
335:   auto pg = c10d::resolve_process_group(pai_->group_name_);
336:   RECORD_PARAM_COMMS(
```

- EN: Lines 313-336 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `printf`, `__trap`, `__threadfence_system`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 313-336 行使用条件编译来适配特性开关、平台或可选后端；在 `printf`、`__trap`、`__threadfence_system` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 337-360 / 第 337-360 行

```cpp
337:       static_cast<int64_t>(0),
338:       std::make_tuple(pg->getGroupName(), pg->getGroupDesc()),
339:       rank_,
340:       "symm_mem::wait_signal",
341:       0,
342:       0,
343:       at::kByte,
344:       std::vector<int64_t>(),
345:       std::vector<int64_t>(),
346:       -1,
347:       -1,
348:       world_size_);
349:   c10::cuda::CUDAGuard device_guard(local_device_idx_);
350:   wait_signal_kernel<<<
351:       1,
352:       at::cuda::warp_size(),
353:       0,
354:       at::cuda::getCurrentCUDAStream()>>>(
355:       reinterpret_cast<uint32_t**>(pai_->signal_pads_dev_),
356:       src_rank,
357:       channel,
358:       rank_,
359:       world_size_,
360:       timeout_ms);
```

- EN: Lines 337-360 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 337-360 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 361-384 / 第 361-384 行

```cpp
361:   C10_CUDA_KERNEL_LAUNCH_CHECK();
362: }
363: 
364: int CUDASymmetricMemory::get_rank() {
365:   return rank_;
366: }
367: 
368: int CUDASymmetricMemory::get_world_size() {
369:   return world_size_;
370: }
371: 
372: c10::Device CUDASymmetricMemory::get_device() {
373:   return c10::Device(c10::DeviceType::CUDA, local_device_idx_);
374: }
375: 
376: bool CUDASymmetricMemory::world_within_direct_access() {
377:   return true;
378: }
379: 
380: /* End of CUDASymmetricMemory */
381: 
382: Block::Block(
383:     c10::intrusive_ptr<AllocationRef> alloc_ref,
384:     int device_idx,
```

- EN: Lines 361-384 introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 361-384 行在 `C10_CUDA_KERNEL_LAUNCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 385-408 / 第 385-408 行

```cpp
385:     size_t block_size,
386:     size_t buffer_size,
387:     size_t signal_pad_offset,
388:     const std::optional<std::string>& group_name)
389:     : alloc_ref(std::move(alloc_ref)),
390:       device_idx(device_idx),
391:       block_size(block_size),
392:       buffer_size(buffer_size),
393:       signal_pad_offset(signal_pad_offset),
394:       default_group_name(std::move(group_name)) {}
395: 
396: namespace {
397: using Expandable_Segments_Handle_Type =
398:     c10::cuda::CUDACachingAllocator::Expandable_Segments_Handle_Type;
399: }
400: 
401: void* CUDASymmetricMemoryAllocator::alloc(
402:     size_t size,
403:     int device_idx,
404:     const std::optional<std::string>& group_name) {
405:   size_t signal_pad_offset = at::round_up(size, 16UL);
406:   size_t block_size = signal_pad_offset + get_signal_pad_size();
407:   c10::cuda::CUDAGuard guard(device_idx);
408:   device_idx = static_cast<int>(guard.current_device().index());
```

- EN: Lines 385-408 opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 385-408 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 409-432 / 第 409-432 行

```cpp
409: #if !defined(USE_ROCM) && defined(PYTORCH_C10_DRIVER_API_SUPPORTED)
410:   CUmemAllocationProp prop = {};
411:   prop.type = CU_MEM_ALLOCATION_TYPE_PINNED;
412:   prop.location.type = CU_MEM_LOCATION_TYPE_DEVICE;
413:   // NOLINTNEXTLINE(bugprone-signed-char-misuse)
414:   prop.location.id = device_idx;
415:   bool has_fabric_support = at::cuda::get_fabric_access(device_idx);
416:   LOG(INFO) << "CUDASymmetricMemoryAllocator::alloc: has_fabric_support " << has_fabric_support;
417:   if (handle_type_ == Expandable_Segments_Handle_Type::UNSPECIFIED) {
418:     handle_type_ = has_fabric_support ? Expandable_Segments_Handle_Type::FABRIC_HANDLE : Expandable_Segments_Handle_Type::POSIX_FD;
419:   }
420:   if (handle_type_ == Expandable_Segments_Handle_Type::POSIX_FD) {
421:     prop.requestedHandleTypes = CU_MEM_HANDLE_TYPE_POSIX_FILE_DESCRIPTOR;
422:   } else {
423:     prop.requestedHandleTypes = CU_MEM_HANDLE_TYPE_FABRIC;
424:   }
425: 
426:   size_t granularity;
427:   auto driver_api = c10::cuda::DriverAPI::get();
428:   C10_CUDA_DRIVER_CHECK(driver_api->cuMemGetAllocationGranularity_(
429:       &granularity, &prop, CU_MEM_ALLOC_GRANULARITY_RECOMMENDED));
430:   block_size = at::round_up(block_size, granularity);
431: 
432:   HandleType handle;
```

- EN: Lines 409-432 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 409-432 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 433-456 / 第 433-456 行

```cpp
433:   C10_CUDA_DRIVER_CHECK(driver_api->cuMemCreate_(&handle, block_size, &prop, 0));
434: 
435: #elif defined(USE_ROCM)
436:   handle_type_ = Expandable_Segments_Handle_Type::POSIX_FD;
437:   hipMemAllocationProp prop = {};
438:   prop.type = hipMemAllocationTypePinned;
439:   prop.location.type = hipMemLocationTypeDevice;
440:   // NOLINTNEXTLINE(bugprone-signed-char-misuse)
441:   prop.location.id = device_idx;
442:   prop.requestedHandleType = hipMemHandleTypePosixFileDescriptor;
443: 
444:   size_t granularity;
445:   C10_CUDA_CHECK(hipMemGetAllocationGranularity(
446:       &granularity, &prop, hipMemAllocationGranularityRecommended));
447:   block_size = at::round_up(block_size, granularity);
448: 
449:   HandleType handle;
450:   C10_CUDA_CHECK(hipMemCreate(
451:       reinterpret_cast<hipMemGenericAllocationHandle_t*>(&handle),
452:       block_size,
453:       &prop,
454:       0));
455: 
456: #else
```

- EN: Lines 433-456 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 433-456 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 457-480 / 第 457-480 行

```cpp
457:   TORCH_CHECK(
458:       false, "CUDASymmetricMemory requires PYTORCH_C10_DRIVER_API_SUPPORTED");
459: #endif
460:   void* ptr = nullptr;
461:   map_block(&ptr, handle, block_size, device_idx);
462: 
463:   AT_CUDA_CHECK(cudaMemset(ptr, 0, block_size));
464: 
465:   auto alloc_ref =
466:       c10::make_intrusive<AllocationRef>(ptr, handle, block_size, device_idx);
467:   auto block = c10::make_intrusive<Block>(
468:       std::move(alloc_ref),
469:       device_idx,
470:       block_size,
471:       size,
472:       signal_pad_offset,
473:       group_name);
474:   {
475:     std::unique_lock lock(mutex_);
476:     ptr_to_block_.emplace(ptr, std::move(block));
477:   }
478:   return ptr;
479: }
480: 
```

- EN: Lines 457-480 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `TORCH_CHECK`, `map_block`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 457-480 行使用条件编译来适配特性开关、平台或可选后端；在 `TORCH_CHECK`、`map_block` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 481-504 / 第 481-504 行

```cpp
481: void CUDASymmetricMemoryAllocator::free(void* ptr) {
482:   std::unique_lock lock(mutex_);
483:   ptr_to_block_.erase(ptr);
484: }
485: 
486: size_t CUDASymmetricMemoryAllocator::get_alloc_size(void* ptr) {
487:   auto block = find_block(ptr);
488:   TORCH_CHECK(
489:       block != nullptr,
490:       "CUDASymmetricMemoryAllocator::get_alloc_size: input must be allocated ",
491:       "via CUDASymmetricMemoryAllocator::alloc");
492:   return block->buffer_size;
493: }
494: 
495: struct RendezvousRequest {
496:   int device_idx;
497:   int pid;
498:   size_t block_size;
499:   size_t buffer_size;
500:   size_t signal_pad_offset;
501:   bool has_multicast_support;
502:   int clique_id;
503:   char hostname[HOST_NAME_MAX + 1];
504: };
```

- EN: Lines 481-504 declares or defines types such as `RendezvousRequest`; introduces executable logic in routines such as `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 481-504 行声明或定义了 `RendezvousRequest` 等类型；在 `TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 505-528 / 第 505-528 行

```cpp
505: 
506: static std::string import_err_msg(
507:     int rank,
508:     int peer,
509:     const std::vector<RendezvousRequest>& reqs) {
510:   std::ostringstream oss;
511:   oss << ". Rank " << rank << " (host: " << reqs[rank].hostname
512:       << ", device: " << reqs[rank].device_idx << ", fabric_info: {"
513:       << at::cuda::get_nvml_fabric_info(reqs[rank].device_idx)
514:       << "}) failed to import memory from rank " << peer
515:       << " (host: " << reqs[peer].hostname
516:       << ", device: " << reqs[peer].device_idx << ", NCCL_MNNVL_CLIQUE_ID: "
517:       << c10::utils::get_env("NCCL_MNNVL_CLIQUE_ID").value_or("unset") << ").";
518:   return oss.str();
519: }
520: 
521: void validate_rendezvous_requests(
522:     const std::vector<RendezvousRequest>& reqs,
523:     int world_size) {
524:   TORCH_CHECK(reqs.size() == (size_t)world_size);
525: 
526:   // For NVL72 systems, multiple hosts can be within a single nvlink domain.
527:   // Multiple blocks will have same device_idx but they are on different hosts.
528:   // Use (hostname, device_idx) pair to uniquely identify each allocation.
```

- EN: Lines 505-528 introduces executable logic in routines such as `import_err_msg`, `validate_rendezvous_requests`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 505-528 行在 `import_err_msg`、`validate_rendezvous_requests` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 529-552 / 第 529-552 行

```cpp
529:   std::set<std::pair<std::string, int>> device_host_pairs;
530:   for (auto req : reqs) {
531:     device_host_pairs.insert(
532:         std::make_pair(std::string(req.hostname), req.device_idx));
533:   }
534:   if (!allow_overlapping_devices() &&
535:       device_host_pairs.size() < (size_t)world_size) {
536:     TORCH_CHECK(
537:         false,
538:         "CUDASymmetricMemoryAllocator::rendezvous: ",
539:         "detected allocations from overlapping devices ",
540:         "from different ranks.");
541:   }
542: 
543:   for (int r = 1; r < world_size; ++r) {
544:     TORCH_CHECK(reqs[r].block_size == reqs[0].block_size);
545:     TORCH_CHECK(reqs[r].buffer_size == reqs[0].buffer_size);
546:     TORCH_CHECK(reqs[r].signal_pad_offset == reqs[0].signal_pad_offset);
547:   }
548: }
549: 
550: // All ranks must be in the same NVLink domain (same clique_id). Detect
551: // mismatches early before the import fails with an opaque CUDA error.
552: static void validate_nvlink_fabric_support(
```

- EN: Lines 529-552 introduces executable logic in routines such as `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 529-552 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 553-576 / 第 553-576 行

```cpp
553:     const std::vector<RendezvousRequest>& reqs,
554:     int world_size) {
555:   std::unordered_set<int> clique_ids;
556:   for (const auto& req : reqs) {
557:     if (req.clique_id >= 0) {
558:       clique_ids.insert(req.clique_id);
559:     }
560:   }
561:   if (clique_ids.size() > 1) {
562:     std::ostringstream oss;
563:     oss << "CUDASymmetricMemory::rendezvous: "
564:         << "ranks have mismatched NVLink clique_ids. "
565:         << "All ranks using fabric handles must be in the same NVLink domain. "
566:         << "Per-rank info: ";
567:     for (int r = 0; r < world_size; ++r) {
568:       if (r > 0) {
569:         oss << ", ";
570:       }
571:       oss << "rank " << r << " (host: " << reqs[r].hostname
572:           << ", device: " << reqs[r].device_idx
573:           << ", clique_id: " << reqs[r].clique_id << ")";
574:     }
575:     TORCH_CHECK(false, oss.str());
576:   }
```

- EN: Lines 553-576 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 553-576 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 577-600 / 第 577-600 行

```cpp
577: }
578: 
579: static bool check_group_multicast_support(
580:     const std::vector<RendezvousRequest>& reqs) {
581:   std::vector<size_t> ranks_with_multicast_support;
582:   for (size_t r = 0; r < reqs.size(); ++r) {
583:     if (reqs[r].has_multicast_support) {
584:       ranks_with_multicast_support.push_back(r);
585:     }
586:   }
587:   if (ranks_with_multicast_support.size() == reqs.size()) {
588:     return true;
589:   } else {
590:     // We don't expect this to happen. But we want to let the user to know if
591:     // this happens.
592:     if (ranks_with_multicast_support.size() != 0) {
593:       LOG(WARNING)
594:           << "Only a subset of ranks in the group has multicast support: "
595:           << ranks_with_multicast_support << " (world_size=" << reqs.size()
596:           << "). Skipping multicast initialization because this is unexpected.";
597:     }
598:     return false;
599:   }
600: }
```

- EN: Lines 577-600 introduces executable logic in routines such as `check_group_multicast_support`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 577-600 行在 `check_group_multicast_support` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 601-624 / 第 601-624 行

```cpp
601: 
602: template <bool use_fabric_handle>
603: static void init_multicast_for_block(
604:     HandleType& mc_handle,
605:     void*& mc_addr,
606:     const c10::intrusive_ptr<Block>& block,
607:     std::conditional_t<!use_fabric_handle, IpcChannel&, int&> ipc_channel,
608:     const std::vector<int>& pids,
609:     const c10::intrusive_ptr<c10d::Store>& store,
610:     int rank,
611:     int world_size) {
612: #if !defined(USE_ROCM) && defined(PYTORCH_C10_DRIVER_API_SUPPORTED) && \
613:     defined(CUDART_SUPPORTS_MULTICAST)
614:   auto driver_api = c10::cuda::DriverAPI::get();
615:   auto handleType = use_fabric_handle
616:       ? CU_MEM_HANDLE_TYPE_FABRIC
617:       : CU_MEM_HANDLE_TYPE_POSIX_FILE_DESCRIPTOR;
618:   using McHandleType =
619:       std::conditional_t<use_fabric_handle, CUmemFabricHandle, int>;
620: 
621:   McHandleType invalidator;
622:   std::memset(&invalidator, UINT8_MAX, sizeof(McHandleType));
623: 
624:   // Phase 1: export handle (rank 0 only)
```

- EN: Lines 601-624 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `init_multicast_for_block`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 601-624 行使用条件编译来适配特性开关、平台或可选后端；在 `init_multicast_for_block` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 625-648 / 第 625-648 行

```cpp
625:   McHandleType mc_exported_handle{};
626:   if (rank == 0) {
627:     CUmulticastObjectProp mc_prop{};
628:     mc_prop.numDevices = world_size;
629:     mc_prop.handleTypes = handleType;
630:     mc_prop.size = block->block_size;
631: 
632:     // create a multicast object, which acts as a handle that allows multiple
633:     // devices or processes to access the same memory allocation coherently.
634:     try {
635:       C10_CUDA_DRIVER_CHECK(
636:           driver_api->cuMulticastCreate_(&mc_handle, &mc_prop));
637:       // using the CUDA Driver API to export a multicast object into a POSIX file
638:       // descriptor.
639:       C10_CUDA_DRIVER_CHECK(driver_api->cuMemExportToShareableHandle_(
640:           &mc_exported_handle, mc_handle, handleType, 0));
641:     } catch (const std::exception& e) {
642:       // Allow peers gracefully skip multicast initialization by sending -1
643:       mc_exported_handle = invalidator;
644:       LOG(WARNING)
645:           << "SymmetricMemory: fail to export multicast handle.\n"
646:           << e.what();
647:     }
648:   }
```

- EN: Lines 625-648 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 625-648 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 649-672 / 第 649-672 行

```cpp
649: 
650:   // Phase 2: Exchange handle
651:   McHandleType recv_handle;
652:   if constexpr (!use_fabric_handle) {
653:     recv_handle = ipc_channel.broadcast_fds(rank, 0, pids, mc_exported_handle);
654:   } else {
655:     // TODO implement storeExchange.broadcast
656:     auto gathered_handles = storeExchange.all_gather(store, rank, world_size, mc_exported_handle);
657:     recv_handle = std::move(gathered_handles[0]);
658:   }
659: 
660:   // Check exchange result
661:   if (memcmp(&recv_handle, &invalidator, sizeof(McHandleType)) == 0) {
662:     LOG(WARNING) << "Gracefully skipping multicast initialization.";
663:     return;
664:   }
665: 
666:   // Flip to true after all CUDA steps finish
667:   bool success_end = false;
668: 
669:   // Phase 3: Import handle (non-0 ranks only)
670:   if (rank != 0) {
671:     if constexpr (!use_fabric_handle) {
672:       // Convert back to a handle from the broadcasted POSIX file descriptor.
```

- EN: Lines 649-672 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 649-672 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 673-696 / 第 673-696 行

```cpp
673:       C10_CUDA_DRIVER_CHECK_GOTO(driver_api->cuMemImportFromShareableHandle_(
674:           &mc_handle,
675:           (void*)(uintptr_t)recv_handle,
676:           CU_MEM_HANDLE_TYPE_POSIX_FILE_DESCRIPTOR), check_all);
677:     } else {
678:       C10_CUDA_DRIVER_CHECK_GOTO(driver_api->cuMemImportFromShareableHandle_(
679:           &mc_handle, (void*)&(recv_handle), CU_MEM_HANDLE_TYPE_FABRIC), check_all);
680:     }
681:   }
682: 
683:   // Phase 4: Bind memory
684:   // All rank adds their physical allocation to the multicast object
685:   C10_CUDA_DRIVER_CHECK_GOTO(
686:       driver_api->cuMulticastAddDevice_(mc_handle, block->device_idx), check_all);
687:   C10_CUDA_DRIVER_CHECK_GOTO(driver_api->cuMulticastBindMem_(
688:       mc_handle, 0, block->alloc_ref->handle, 0, block->block_size, 0), check_all);
689: 
690:   success_end = true;
691: 
692: check_all:
693:   // Whether all ranks have succeeded
694:   bool all_succeed = true;
695:   auto rank_successes = storeExchange.all_gather(store, rank, world_size, success_end);
696:   for (int r = 0; r < world_size; ++r) {
```

- EN: Lines 673-696 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 673-696 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 697-720 / 第 697-720 行

```cpp
697:     all_succeed &= rank_successes[r];
698:   }
699:   // Close the file descriptor before exit
700:   if constexpr (!use_fabric_handle) {
701:     close(recv_handle);
702:   }
703:   if (!all_succeed) {
704:     LOG(WARNING) << "Gracefully skipping multicast initialization.";
705:     return;
706:   }
707: 
708:   // Phase 5: Map to virtual memory
709:   map_block(&mc_addr, mc_handle, block->block_size, block->device_idx);
710: #endif
711: }
712: 
713: namespace {
714: template <bool use_fabric_handle>
715: c10::intrusive_ptr<CUDAPeerAllocInfo> make_peer_alloc_info(
716:     void* ptr,
717:     c10::intrusive_ptr<Block> block,
718:     const std::string& group_name) {
719: #if defined(USE_ROCM)
720:   using BlockHandleType = int;
```

- EN: Lines 697-720 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `map_block`, `make_peer_alloc_info`.
- CN: 第 697-720 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `map_block`、`make_peer_alloc_info` 等例程中引入具体执行逻辑。

### Lines 721-744 / 第 721-744 行

```cpp
721: #else
722:   using BlockHandleType =
723:       std::conditional_t<use_fabric_handle, CUmemFabricHandle, int>;
724: #endif
725:   BlockHandleType block_handle;
726:   c10::cuda::CUDAGuard guard(block->device_idx);
727:   if constexpr (!use_fabric_handle) {
728:     LOG(INFO) << "using posix fd to import symmetric memory handles.";
729:   } else {
730:     LOG(INFO) << "using fabric handle to import symmetric memory handles.";
731:   }
732: 
733:   auto group = resolve_process_group(group_name);
734:   auto rank = group->getRank();
735:   auto world_size = group->getSize();
736:   auto store = group->getStore();
737: 
738:   // Currently, IpcChannel is using a file based socket for inter-process
739:   // communication
740:   // Note: don't move ipc_channel construction closer to the use
741:   // there needs to be a barrier between constructor and first use,
742:   // and this barrier is provided when we are exchanging rendezvous requests
743:   using IpcChannelType = std::conditional_t<use_fabric_handle, int, IpcChannel>;
744:   IpcChannelType ipc_channel;
```

- EN: Lines 721-744 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `guard`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 721-744 行使用条件编译来适配特性开关、平台或可选后端；在 `guard` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 745-768 / 第 745-768 行

```cpp
745: 
746: #if !defined(USE_ROCM) && defined(PYTORCH_C10_DRIVER_API_SUPPORTED)
747:   auto driver_api = c10::cuda::DriverAPI::get();
748:   // using the CUDA Driver API to export a GPU memory block as a
749:   // POSIX file descriptor (FD), so it can be shared across processes via IPC.
750:   C10_CUDA_DRIVER_CHECK(driver_api->cuMemExportToShareableHandle_(
751:       &block_handle,
752:       block->alloc_ref->handle,
753:       use_fabric_handle ? CU_MEM_HANDLE_TYPE_FABRIC
754:                         : CU_MEM_HANDLE_TYPE_POSIX_FILE_DESCRIPTOR,
755:       0));
756: #elif defined(USE_ROCM)
757:   C10_CUDA_CHECK(hipMemExportToShareableHandle(
758:       &block_handle,
759:       block->alloc_ref->handle,
760:       hipMemHandleTypePosixFileDescriptor,
761:       0));
762: #else
763:   TORCH_CHECK(
764:       false, "CUDASymmetricMemory requires PYTORCH_C10_DRIVER_API_SUPPORTED");
765: #endif
766: 
767:   auto local_req = RendezvousRequest{
768:       .device_idx = block->device_idx,
```

- EN: Lines 745-768 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 745-768 行使用条件编译来适配特性开关、平台或可选后端；在 `TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 769-792 / 第 769-792 行

```cpp
769:       .pid = getpid(),
770:       .block_size = block->block_size,
771:       .buffer_size = block->buffer_size,
772:       .signal_pad_offset = block->signal_pad_offset,
773:       .has_multicast_support = device_has_multicast_support(block->device_idx),
774:       .clique_id = at::cuda::get_fabric_clique_id(block->device_idx)};
775: 
776:   // Populate hostname field for host identification
777:   gethostname(local_req.hostname, sizeof(local_req.hostname));
778:   auto reqs = storeExchange.all_gather(store, rank, world_size, local_req);
779:   validate_nvlink_fabric_support(reqs, world_size);
780:   validate_rendezvous_requests(reqs, world_size);
781: 
782:   std::vector<int> pids(world_size);
783:   for (int r = 0; r < world_size; ++r) {
784:     pids[r] = reqs[r].pid;
785:   }
786: 
787:   std::vector<BlockHandleType> imported_handles;
788:   if constexpr (!use_fabric_handle) {
789:     imported_handles = ipc_channel.all_gather_fds(rank, pids, block_handle);
790:   } else {
791:     imported_handles =
792:         storeExchange.all_gather(store, rank, world_size, block_handle);
```

- EN: Lines 769-792 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 769-792 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 793-816 / 第 793-816 行

```cpp
793:   }
794: 
795:   std::vector<HandleType> handles(world_size);
796:   std::vector<void*> buffers(world_size, nullptr);
797:   std::vector<void*> signal_pads(world_size, nullptr);
798: 
799:   for (int r = 0; r < world_size; ++r) {
800:     if (r == rank) {
801:       handles[r] = block->alloc_ref->handle;
802:       buffers[r] = ptr;
803:       signal_pads[r] = (void*)((uintptr_t)ptr + block->signal_pad_offset);
804:       continue;
805:     }
806:     // This api imports a GPU memory allocation that was previously exported as
807:     // a file descriptor or fabric handle and it returns a memory handle.
808: #if !defined(USE_ROCM) && defined(PYTORCH_C10_DRIVER_API_SUPPORTED)
809:     // note how in one case it's directly imported_handles[r] and in another
810:     // &(imported_handles[r]) so can't do with just type definitions
811:     if constexpr (!use_fabric_handle) {
812:       C10_CUDA_DRIVER_CHECK_MSG(
813:           driver_api->cuMemImportFromShareableHandle_(
814:               &handles[r],
815:               (void*)(uintptr_t)imported_handles[r],
816:               CU_MEM_HANDLE_TYPE_POSIX_FILE_DESCRIPTOR),
```

- EN: Lines 793-816 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 793-816 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 817-840 / 第 817-840 行

```cpp
817:           import_err_msg(rank, r, reqs));
818:     } else {
819:       C10_CUDA_DRIVER_CHECK_MSG(
820:           driver_api->cuMemImportFromShareableHandle_(
821:               &handles[r],
822:               (void*)&(imported_handles[r]),
823:               CU_MEM_HANDLE_TYPE_FABRIC),
824:           import_err_msg(rank, r, reqs));
825:     }
826: #elif defined(USE_ROCM)
827:     C10_CUDA_CHECK(hipMemImportFromShareableHandle(
828:         &handles[r],
829: #if ROCM_VERSION >= 70100
830:         reinterpret_cast<void*>(static_cast<uintptr_t>(imported_handles[r])),
831: #else
832:         (void*)(uintptr_t) & (imported_handles[r]),
833: #endif
834:         hipMemHandleTypePosixFileDescriptor));
835: #else
836:     TORCH_CHECK(
837:         false, "CUDASymmetricMemory requires PYTORCH_C10_DRIVER_API_SUPPORTED");
838: #endif
839:     map_block(&buffers[r], handles[r], block->block_size, block->device_idx);
840:     signal_pads[r] = (void*)((uintptr_t)buffers[r] + block->signal_pad_offset);
```

- EN: Lines 817-840 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `TORCH_CHECK`, `map_block`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 817-840 行使用条件编译来适配特性开关、平台或可选后端；在 `TORCH_CHECK`、`map_block` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 841-864 / 第 841-864 行

```cpp
841:     if constexpr (!use_fabric_handle) {
842:       close(imported_handles[r]);
843:     }
844:   }
845:   storeExchange.barrier(store, rank, world_size);
846:   if constexpr (!use_fabric_handle) {
847:     close(block_handle);
848:   }
849: 
850:   HandleType mc_handle{};
851:   void* mc_addr = nullptr;
852:   bool group_has_multicast_support = check_group_multicast_support(reqs);
853:   if (!allow_overlapping_devices() && group_has_multicast_support) {
854:     init_multicast_for_block<use_fabric_handle>(
855:         mc_handle, mc_addr, block, ipc_channel, pids, store, rank, world_size);
856:   }
857: 
858:   std::vector<c10::intrusive_ptr<AllocationRef>> alloc_refs;
859:   for (int r = 0; r < world_size; ++r) {
860:     if (r == rank) {
861:       if (mc_addr != nullptr) {
862:         alloc_refs.push_back(c10::make_intrusive<AllocationRef>(
863:             mc_addr, mc_handle, block->block_size, block->device_idx, true));
864:       }
```

- EN: Lines 841-864 introduces executable logic in routines such as `close`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 841-864 行在 `close` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 865-888 / 第 865-888 行

```cpp
865:       // Note that in B200, cuMulticastUnbind can error if the mapped buffers
866:       // are free'd before the multicast object is free'd. That's why the
867:       // alloc_ref for the multicast object is added first into the vector,
868:       // such that ~AllocationRef can release it first. For more context,
869:       // see: https://github.com/pytorch/pytorch/issues/162429
870:       alloc_refs.emplace_back(block->alloc_ref);
871:       continue;
872:     }
873:     alloc_refs.push_back(c10::make_intrusive<AllocationRef>(
874:         buffers[r], handles[r], block->block_size, block->device_idx));
875:   }
876: 
877:   auto pai = c10::make_intrusive<CUDAPeerAllocInfo>(
878:       std::move(alloc_refs),
879:       std::move(buffers),
880:       std::move(signal_pads),
881:       mc_handle,
882:       mc_addr,
883:       block->buffer_size,
884:       block->device_idx,
885:       rank,
886:       world_size,
887:       group_name);
888: 
```

- EN: Lines 865-888 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 865-888 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 889-912 / 第 889-912 行

```cpp
889:   return pai;
890: }
891: 
892: } // namespace
893: 
894: c10::intrusive_ptr<SymmetricMemory> CUDASymmetricMemoryAllocator::rendezvous(
895:     void* ptr,
896:     const std::optional<std::string>& group_name) {
897:   // In case of MemPool, the `ptr` passed in (i.e. tensor storage ptr) may not
898:   // be the same as the allocation base pointer, so we need to find the block
899:   // that covers the `ptr`
900:   size_t offset = 0;
901:   auto block = find_block_covering(ptr, offset);
902:   if (block == nullptr) {
903:     TORCH_WARN(
904:       "Pointer not within any SymmetricMemory allocation, "
905:       "is the tensor allocated from SymmetricMemory?");
906:     return nullptr;
907:   }
908:   // The group_name passed to rendezvous() takes precedence over
909:   // the default group_name specified during allocation.
910:   std::string group_name_;
911:   // Treat empty string and std::nullopt the same as empty string seems to be
912:   // implicitly used that way
```

- EN: Lines 889-912 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `TORCH_WARN`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 889-912 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `TORCH_WARN` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 913-936 / 第 913-936 行

```cpp
913:   if (group_name.has_value() && group_name != "") {
914:     group_name_ = *group_name;
915:   } else {
916:     if (!block->default_group_name.has_value()) {
917:       TORCH_CHECK(
918:           false,
919:           "CUDASymmetricMemory::rendezvous: `group_name` is neither "
920:           "specified during allocation nor passed to rendezvous().");
921:     }
922:     group_name_ = *block->default_group_name;
923:   }
924: 
925:   // If found, this block has been rendezvous by the given group
926:   auto it = block->symm_mems.find(group_name_);
927:   if (it == block->symm_mems.end()) {
928:     // Create PeerAllocInfo for this block (this is the costly part)
929:     TORCH_INTERNAL_ASSERT(
930:         handle_type_ != Expandable_Segments_Handle_Type::UNSPECIFIED)
931:     bool use_fabric =
932:         handle_type_ == Expandable_Segments_Handle_Type::FABRIC_HANDLE;
933:     // PeerAllocInfo captures this block's rendezvous info
934:     auto pai = use_fabric ? make_peer_alloc_info<true>(ptr, block, group_name_)
935:                           : make_peer_alloc_info<false>(ptr, block, group_name_);
936:     // Cache it with the group name
```

- EN: Lines 913-936 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 913-936 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 937-960 / 第 937-960 行

```cpp
937:     it = block->symm_mems.emplace(group_name_, pai).first;
938:   }
939: 
940:   // Create symm mem handle for this tensor, specified by its offset
941:   auto pai = it->second;
942:   return c10::make_intrusive<CUDASymmetricMemory>(pai, offset);
943: }
944: 
945: bool CUDASymmetricMemoryAllocator::has_multicast_support(int device_idx) {
946:   return device_has_multicast_support(device_idx);
947: }
948: 
949: c10::DeviceType CUDASymmetricMemoryAllocator::supported_device_type() {
950:   return c10::DeviceType::CUDA;
951: }
952: 
953: std::string CUDASymmetricMemoryAllocator::name() {
954:   return "CUDA";
955: }
956: 
957: c10::intrusive_ptr<Block> CUDASymmetricMemoryAllocator::find_block(void* ptr) {
958:   std::shared_lock lock(mutex_);
959:   auto it = ptr_to_block_.find(ptr);
960:   if (it == ptr_to_block_.end()) {
```

- EN: Lines 937-960 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 937-960 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 961-984 / 第 961-984 行

```cpp
961:     return nullptr;
962:   }
963:   return it->second;
964: }
965: 
966: /* Search for a block that covers the given ptr, and write back the offset to
967:  * the base ptr; error out if not found */
968: c10::intrusive_ptr<Block> CUDASymmetricMemoryAllocator::find_block_covering(void* ptr, size_t& offset) {
969:   std::shared_lock lock(mutex_);
970:   // In case of MemPool, tensor.storage().data_ptr() may not match
971:   // exactly an allocation's base address. Thus we perform the search by
972:   // testing if the former is within an allocation's range.
973:   auto alloc_it = std::find_if(ptr_to_block_.begin(), ptr_to_block_.end(),
974:                              [&](const auto& pair){
975:                                 auto& block = pair.second;
976:                                 auto& allocation = block->alloc_ref;
977:                                 auto ptr_int = reinterpret_cast<uintptr_t>(ptr);
978:                                 auto base_ptr = reinterpret_cast<uintptr_t>(allocation->ptr);
979:                                 // Modify offset so that it is returned
980:                                 offset = ptr_int - base_ptr;
981:                                 return ptr_int >= base_ptr && offset < block->buffer_size; });
982: 
983:   if (alloc_it == ptr_to_block_.end()) {
984:     return nullptr;
```

- EN: Lines 961-984 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 961-984 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 985-1008 / 第 985-1008 行

```cpp
985:   }
986: 
987:   return alloc_it->second;
988: }
989: 
990: bool CUDASymmetricMemoryAllocator::has_allocation(void* ptr) {
991:   return find_block(ptr) != nullptr;
992: }
993: 
994: struct RegisterCUDASymmetricMemoryAllocator {
995:   RegisterCUDASymmetricMemoryAllocator() {
996:     auto allocator = c10::make_intrusive<CUDASymmetricMemoryAllocator>();
997:     // Query backend used for CUDA tensor
998:     // "CUDA" backend stands for this implementation
999:     if (getSymmMemBackendCUDA() == "CUDA") {
1000:       // Direct set (static registration)
1001:       register_allocator(c10::DeviceType::CUDA, allocator);
1002:     } else {
1003:       // Register availability in case `set_backend` is called dynamically
1004:       register_availability("CUDA", allocator);
1005:     }
1006:   }
1007: };
1008: 
```

- EN: Lines 985-1008 declares or defines types such as `RegisterCUDASymmetricMemoryAllocator`; introduces executable logic in routines such as `RegisterCUDASymmetricMemoryAllocator`, `register_allocator`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 985-1008 行声明或定义了 `RegisterCUDASymmetricMemoryAllocator` 等类型；在 `RegisterCUDASymmetricMemoryAllocator`、`register_allocator` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 1009-1011 / 第 1009-1011 行

```cpp
1009: static RegisterCUDASymmetricMemoryAllocator register_allocator_;
1010: 
1011: } // namespace c10d::symmetric_memory
```

- EN: Lines 1009-1011 opens or closes namespaces to place the code in the correct distributed component; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 1009-1011 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；包含面向 CUDA 的声明、内核或启动流程。

## Key Concepts / 关键概念

- EN: Subsystem: c10d symmetric-memory support.
- CN: 子系统：c10d 对称内存支持。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `RendezvousRequest`, `RegisterCUDASymmetricMemoryAllocator`
- CN: 核心符号：`RendezvousRequest`、`RegisterCUDASymmetricMemoryAllocator`
- EN: Notable themes: CUDA paths, store/state coordination.
- CN: 值得关注的主题：CUDA 路径、存储/状态协调。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/GroupRegistry.hpp`, `torch/csrc/distributed/c10d/ParamCommsUtils.hpp`, `torch/csrc/distributed/c10d/cuda/utils.hpp`, `torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemory-inl.cuh`, `torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemory.hpp`, `torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryUtils.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/ceil_div.h`, `ATen/cuda/CUDAContext.h`, `ATen/cuda/PeerToPeerAccess.h`, `c10/cuda/CUDACachingAllocator.h`, `c10/cuda/CUDAGuard.h`, `c10/util/env.h`, `c10/util/error.h`, `c10/cuda/driver_api.h`
- External or system headers / 外部或系统头文件: `sys/socket.h`, `unistd.h`, `hip/hip_runtime_api.h`
- Local symbols / 本地符号: `RendezvousRequest`, `RegisterCUDASymmetricMemoryAllocator`