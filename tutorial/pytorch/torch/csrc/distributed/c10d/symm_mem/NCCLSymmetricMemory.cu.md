# NCCLSymmetricMemory.cu — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/symm_mem/NCCLSymmetricMemory.cu`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides implementation logic for ncclsymmetric memory in the c10d symmetric-memory support. Key types include `NCCLAllocation`, `NCCLPeerAllocInfo`, `NCCLSymmetricMemoryAllocator`. GPU-oriented code paths are present in this file.
- 用途 (CN): 该文件在c10d 对称内存支持中提供ncclsymmetric memory 的实现逻辑。 关键类型包括 `NCCLAllocation`、`NCCLPeerAllocInfo`、`NCCLSymmetricMemoryAllocator`。 该文件还包含面向 GPU 的代码路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

```cpp
1: #include <torch/csrc/distributed/c10d/symm_mem/nccl_dev_cap.hpp>
2: 
3: #ifdef NCCL_HAS_SYMMEM_SUPPORT
4: 
5: #include <algorithm>
6: #include <vector_types.h>
7: #include <torch/csrc/distributed/c10d/GroupRegistry.hpp>
8: #include <torch/csrc/distributed/c10d/NCCLUtils.hpp>
9: #include <torch/csrc/distributed/c10d/ProcessGroupNCCL.hpp>
10: #include <torch/csrc/distributed/c10d/cuda/utils.hpp>
11: #include <torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemory-inl.cuh>
12: #include <torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryTypes.hpp>
13: #include <torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryUtils.hpp>
14: #include <torch/csrc/distributed/c10d/symm_mem/NCCLSymmetricMemory.hpp>
15: #include <torch/csrc/distributed/c10d/symm_mem/nccl_devcomm_manager.hpp>
16: 
17: #include <ATen/ceil_div.h>
18: #include <ATen/cuda/CUDAContext.h>
19: #include <c10/cuda/CUDACachingAllocator.h>
20: #include <c10/cuda/CUDAGuard.h>
```

- EN: Lines 1-20 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 1-20 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端；包含面向 CUDA 的声明、内核或启动流程。

### Lines 21-40 / 第 21-40 行

```cpp
21: #include <c10/util/error.h>
22: #include <mutex>
23: #include <c10/util/flat_hash_map.h>
24: #include <c10/util/hash.h>
25: 
26: #if !defined(USE_ROCM) && defined(PYTORCH_C10_DRIVER_API_SUPPORTED)
27: #include <c10/cuda/driver_api.h>
28: #endif
29: 
30: namespace c10d {
31: namespace symmetric_memory {
32: 
33: /* Start of NCCLAllocation implementation */
34: 
35: static StoreExchange storeExchange = StoreExchange("NCCLAllocation");
36: 
37: struct NCCLAllocation {
38:   void* ptr;
39:   size_t buffer_size;
40:   int device_idx;
```

- EN: Lines 21-40 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 21-40 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 41-60 / 第 41-60 行

```cpp
41:   std::mutex mutex;
42:   // Map of group name to peer alloc info
43:   ska::flat_hash_map<std::string, c10::intrusive_ptr<NCCLPeerAllocInfo>>
44:       peer_alloc_infos_;
45: 
46:   NCCLAllocation(void* ptr, size_t buffer_size, int device_idx)
47:       : ptr(ptr), buffer_size(buffer_size), device_idx(device_idx) {}
48: 
49:   ~NCCLAllocation() {
50:     // Avoid calling CUDA functions after driver shutting down
51:     if (is_finalizing()) {
52:       return;
53:     }
54:     c10::cuda::CUDAGuard guard(device_idx);
55:     ncclResult_t res = ncclMemFree(ptr);
56:     if (res != ncclSuccess) {
57:         LOG(WARNING) << "ncclMemFree failed in NCCLAllocation dtor: "
58:                       << ncclGetErrorString(res);
59:     }
60:   }
```

- EN: Lines 41-60 introduces executable logic in routines such as `~NCCLAllocation`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 41-60 行在 `~NCCLAllocation` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 61-80 / 第 61-80 行

```cpp
61: };
62: 
63: namespace {
64: 
65: // Base allocation ptr -> owning NCCL allocation metadata.
66: using NCCLAllocMap = ska::flat_hash_map<void*, std::unique_ptr<NCCLAllocation>>;
67: // (Tensor storage/data ptr, group name) -> cached SymmetricMemory handle.
68: using NCCLSymmMemMap = ska::flat_hash_map<
69:     SymmMemKey,
70:     c10::intrusive_ptr<NCCLSymmetricMemory>,
71:     SymmMemKeyHash>;
72: // Base allocation ptr -> cached `(tensor ptr, group)` keys derived from it.
73: using NCCLSymmMemKeysByAlloc =
74:     ska::flat_hash_map<void*, ska::flat_hash_set<SymmMemKey, SymmMemKeyHash>>;
75: 
76: bool pointer_in_allocation(void* ptr, const NCCLAllocation& allocation) {
77:   auto ptr_int = reinterpret_cast<uintptr_t>(ptr);
78:   auto base_ptr = reinterpret_cast<uintptr_t>(allocation.ptr);
79:   return ptr_int >= base_ptr && ptr_int < base_ptr + allocation.buffer_size;
80: }
```

- EN: Lines 61-80 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `pointer_in_allocation`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 61-80 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `pointer_in_allocation` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 81-100 / 第 81-100 行

```cpp
81: 
82: NCCLAllocMap::iterator find_allocation_covering_linear(
83:     void* ptr,
84:     NCCLAllocMap& allocations) {
85:   return std::find_if(
86:       allocations.begin(),
87:       allocations.end(),
88:       [&](const auto& entry) {
89:         return pointer_in_allocation(ptr, *entry.second);
90:       });
91: }
92: 
93: NCCLAllocMap::iterator find_allocation_covering(
94:     void* ptr,
95:     NCCLAllocMap& allocations) {
96:   auto alloc_it = allocations.find(ptr);
97:   if (alloc_it != allocations.end()) {
98:     return alloc_it;
99:   }
100: #if !defined(USE_ROCM) && defined(PYTORCH_C10_DRIVER_API_SUPPORTED)
```

- EN: Lines 81-100 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `find_allocation_covering_linear`, `pointer_in_allocation`, `find_allocation_covering`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 81-100 行使用条件编译来适配特性开关、平台或可选后端；在 `find_allocation_covering_linear`、`pointer_in_allocation`、`find_allocation_covering` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 101-120 / 第 101-120 行

```cpp
101:   auto driver_api = c10::cuda::DriverAPI::get();
102:   CUdeviceptr base_ptr = 0;
103:   // Recover the CUDA allocation base for interior pointers before falling
104:   // back to the linear scan below when the direct lookup cannot help.
105:   auto status = driver_api->cuMemGetAddressRange_(
106:       &base_ptr,
107:       nullptr,
108:       reinterpret_cast<CUdeviceptr>(ptr));
109:   if (status == CUDA_SUCCESS) {
110:     alloc_it = allocations.find(reinterpret_cast<void*>(base_ptr));
111:     if (alloc_it != allocations.end()) {
112:       return alloc_it;
113:     }
114:   }
115: #else
116:   // No driver API support here, so fall through to the linear scan below.
117: #endif
118:   return find_allocation_covering_linear(ptr, allocations);
119: }
120: 
```

- EN: Lines 101-120 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 101-120 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 121-140 / 第 121-140 行

```cpp
121: } // namespace
122: 
123: // Before NCCL 2.29, we can use device-side APIs to get peer pointers.
124: #if NCCL_VERSION_CODE < NCCL_VERSION(2, 29, 0)
125: #ifdef NCCL_HAS_SYMMEM_DEVICE_SUPPORT
126: static __global__ void build_ptr_dev(
127:   ncclWindow_t  handle,
128:   size_t  offset,  // byte offset inside the window
129:   void**  buffer,  // symmetric memory buffer
130:   int  world_size)
131: {
132:   int tid = blockIdx.x * blockDim.x + threadIdx.x;
133:   int stride = blockDim.x * gridDim.x;
134:   for (int peer = tid; peer < world_size; peer += stride) {
135:       buffer[peer] = ncclGetLsaPointer(handle, offset, peer);
136:   }
137: }
138: #endif // NCCL_HAS_SYMMEM_DEVICE_SUPPORT
139: #endif // NCCL_VERSION_CODE < NCCL_VERSION(2, 29, 0)
140: 
```

- EN: Lines 121-140 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `build_ptr_dev`.
- CN: 第 121-140 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `build_ptr_dev` 等例程中引入具体执行逻辑。

### Lines 141-160 / 第 141-160 行

```cpp
141: class NCCLPeerAllocInfo : public c10::intrusive_ptr_target {
142:  public:
143:   NCCLPeerAllocInfo(
144:       NCCLAllocation* allocation,
145:       std::string group_name)
146:       : buffer_size_(allocation->buffer_size),
147:         device_idx_(allocation->device_idx),
148:         group_name_(std::move(group_name))
149:   {
150:     c10::cuda::CUDAGuard guard(device_idx_);
151:     auto group = resolve_process_group(group_name_);
152:     rank_ = group->getRank();
153:     world_size_ = group->getSize();
154:     auto* ncclPg = dynamic_cast<c10d::ProcessGroupNCCL*>(
155:         group->getBackend(c10::DeviceType::CUDA).get());
156:     TORCH_CHECK(ncclPg != nullptr, "backend must be a NCCL process group");
157:     comm_ = reinterpret_cast<ncclComm_t>(ncclPg->getCommPtr());
158: 
159:     C10D_NCCL_CHECK(
160:       ncclCommWindowRegister(comm_, allocation->ptr, buffer_size_, &buffer_win_, NCCL_WIN_COLL_SYMMETRIC),
```

- EN: Lines 141-160 declares or defines types such as `NCCLPeerAllocInfo`; introduces executable logic in routines such as `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 141-160 行声明或定义了 `NCCLPeerAllocInfo` 等类型；在 `TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 161-180 / 第 161-180 行

```cpp
161:       c10::str(
162:           "Failed to window register segment with ptr ",
163:           allocation->ptr,
164:           ", size ",
165:           buffer_size_,
166:           " on rank ",
167:           rank_));
168: 
169:     const size_t signal_pad_size = get_signal_pad_size();
170:     C10D_NCCL_CHECK(
171:         ncclMemAlloc(&signal_pad_ptr_, signal_pad_size), "ncclMemAlloc failed");
172:     C10D_NCCL_CHECK(
173:     ncclCommWindowRegister(comm_, signal_pad_ptr_, signal_pad_size, &signal_handle_, NCCL_WIN_COLL_SYMMETRIC),
174:     c10::str(
175:         "Failed to window register segment with ptr ",
176:         signal_pad_ptr_,
177:         ", size ",
178:         signal_pad_size,
179:         " on rank ",
180:         rank_));
```

- EN: Lines 161-180 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 161-180 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 181-200 / 第 181-200 行

```cpp
181: 
182: #ifdef NCCL_HAS_SYMMEM_DEVICE_SUPPORT
183:     // Register the host-side communicator for device communicator management.
184:     // `ncclDevCommCreate` will require it.
185:     auto& manager = NCCLDevCommManager::get(c10::Device(c10::DeviceType::CUDA, device_idx_));
186:     manager.register_comm(group_name_, comm_);
187: 
188:     // Starting from NCCL 2.28, we can get peer pointers.
189:     const size_t arr_size = sizeof(void*) * world_size_;
190:     buffers_dev_ = reinterpret_cast<void**>(
191:         c10::cuda::CUDACachingAllocator::raw_alloc(arr_size));
192:     signal_pads_dev_ = reinterpret_cast<void**>(
193:         c10::cuda::CUDACachingAllocator::raw_alloc(arr_size));
194:     buffers_.resize(world_size_);
195:     signal_pads_.resize(world_size_);
196: 
197:     // Fill out the peer pointer array
198: #if NCCL_VERSION_CODE < NCCL_VERSION(2, 29, 0)
199:     // Lack of host-side API to get peer pointers, so we get them inside a
200:     // kernel and copy the result to host.
```

- EN: Lines 181-200 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 181-200 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 201-220 / 第 201-220 行

```cpp
201:     int threads = std::min(128, world_size_);
202:     auto stream = at::cuda::getCurrentCUDAStream();
203:     build_ptr_dev<<<1, threads, 0, stream>>>(buffer_win_, 0, buffers_dev_, world_size_);
204:     C10_CUDA_KERNEL_LAUNCH_CHECK();
205:     build_ptr_dev<<<1, threads, 0, stream>>>(signal_handle_, 0, signal_pads_dev_, world_size_);
206:     C10_CUDA_KERNEL_LAUNCH_CHECK();
207:     C10_CUDA_CHECK(cudaStreamSynchronize(stream));
208:     C10_CUDA_CHECK(cudaMemcpy(
209:       buffers_.data(),  // dst (host)
210:       buffers_dev_,  // src (device)
211:       arr_size,
212:       cudaMemcpyDeviceToHost));
213:     C10_CUDA_CHECK(cudaMemcpy(
214:       signal_pads_.data(),  // dst (host)
215:       signal_pads_dev_,  // src (device)
216:       arr_size,
217:       cudaMemcpyDeviceToHost));
218: #else
219:   // Starting from NCCL 2.29, we can use host-side APIs to get peer pointers.
220:   for (int i = 0; i < world_size_; i++) {
```

- EN: Lines 201-220 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 201-220 行使用条件编译来适配特性开关、平台或可选后端；在 `C10_CUDA_KERNEL_LAUNCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 221-240 / 第 221-240 行

```cpp
221:     // If peer is not accessible within LSA domain, `ncclGetPeerDevicePointer`
222:     // returns nullptr.
223:     C10D_NCCL_CHECK(
224:       ncclGetPeerDevicePointer(buffer_win_, 0, i, &buffers_[i]),
225:       "ncclGetPeerDevicePointer failed");
226:     C10D_NCCL_CHECK(
227:       ncclGetPeerDevicePointer(signal_handle_, 0, i, &signal_pads_[i]),
228:       "ncclGetPeerDevicePointer failed");
229:   }
230:   // Copy the peer access pointers to device arrays.
231:   C10_CUDA_CHECK(cudaMemcpy(
232:     buffers_dev_,  // dst (device)
233:     buffers_.data(),  // src (host)
234:     arr_size,
235:     cudaMemcpyHostToDevice));
236:   C10_CUDA_CHECK(cudaMemcpy(
237:     signal_pads_dev_,  // dst (device)
238:     signal_pads_.data(),  // src (host)
239:     arr_size,
240:     cudaMemcpyHostToDevice));
```

- EN: Lines 221-240 contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 221-240 行包含面向 CUDA 的声明、内核或启动流程。

### Lines 241-260 / 第 241-260 行

```cpp
241: 
242:   // Starting from NCCL 2.29, we can use `ncclGetLsaMultimemDevicePointer`
243:   // to get multicast address.
244:   void* mc_addr = nullptr;
245:   // Skip CHECK on purpose to improve fault tolerance since some machine's
246:   // Fabric Manager may be in bad NVLink Sharp state.
247:   if (ncclGetLsaMultimemDevicePointer(buffer_win_, 0, &mc_addr) == ncclSuccess) {
248:     mc_addr_ = mc_addr;
249:   }
250: #endif // NCCL_VERSION_CODE < NCCL_VERSION(2, 29, 0)
251: #endif // NCCL_HAS_SYMMEM_DEVICE_SUPPORT
252:   }
253: 
254:   // Exact copy is not needed / supported
255:   NCCLPeerAllocInfo(const NCCLPeerAllocInfo& other) = delete;
256:   NCCLPeerAllocInfo& operator=(const NCCLPeerAllocInfo& other) = delete;
257:   NCCLPeerAllocInfo(NCCLPeerAllocInfo&& other) = default;
258:   NCCLPeerAllocInfo& operator=(NCCLPeerAllocInfo&& other) = default;
259: 
260:   ~NCCLPeerAllocInfo() {
```

- EN: Lines 241-260 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `~NCCLPeerAllocInfo`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 241-260 行使用条件编译来适配特性开关、平台或可选后端；在 `~NCCLPeerAllocInfo` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 261-280 / 第 261-280 行

```cpp
261:     if (is_finalizing()) {
262:       return;
263:     }
264:     c10::cuda::CUDAGuard guard(device_idx_);
265:     if (buffer_win_ != nullptr) {
266:       auto res = ncclCommWindowDeregister(comm_, buffer_win_);
267:       if (res != ncclSuccess) {
268:         LOG(WARNING) << "ncclCommWindowDeregister failed for buffer_win: "
269:                      << ncclGetErrorString(res);
270:       }
271:     }
272:     if (signal_handle_ != nullptr) {
273:       auto res = ncclCommWindowDeregister(comm_, signal_handle_);
274:       if (res != ncclSuccess) {
275:         LOG(WARNING) << "ncclCommWindowDeregister failed for signal_handle: "
276:                      << ncclGetErrorString(res);
277:       }
278:     }
279:     if (signal_pad_ptr_ != nullptr) {
280:       auto res = ncclMemFree(signal_pad_ptr_);
```

- EN: Lines 261-280 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 261-280 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 281-300 / 第 281-300 行

```cpp
281:       if (res != ncclSuccess) {
282:         LOG(WARNING) << "ncclMemFree failed for signal_pad: "
283:                      << ncclGetErrorString(res);
284:       }
285:     }
286:     if (buffers_dev_ != nullptr) {
287:       c10::cuda::CUDACachingAllocator::raw_delete(buffers_dev_);
288:     }
289:     if (signal_pads_dev_ != nullptr) {
290:       c10::cuda::CUDACachingAllocator::raw_delete(signal_pads_dev_);
291:     }
292:   }
293: 
294:  private:
295:   size_t buffer_size_;
296:   int device_idx_;
297:   int rank_;
298:   int world_size_;
299:   std::vector<void*> buffers_;
300:   std::vector<void*> signal_pads_;
```

- EN: Lines 281-300 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 281-300 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 301-320 / 第 301-320 行

```cpp
301:   void** buffers_dev_{nullptr};
302:   void** signal_pads_dev_{nullptr};
303:   std::string group_name_;
304:   ncclWindow_t buffer_win_{nullptr};
305:   ncclWindow_t signal_handle_{nullptr};
306:   void* signal_pad_ptr_{nullptr};
307:   // Multicast address
308:   void* mc_addr_{nullptr};
309:   ncclComm_t comm_{nullptr};
310:   friend class NCCLSymmetricMemory;
311: };
312: 
313: NCCLSymmetricMemory::NCCLSymmetricMemory(
314:     c10::intrusive_ptr<NCCLPeerAllocInfo> pai,
315:     size_t offset)
316:     : pai_(std::move(pai)),
317:       offset_(offset),
318:       rank_(pai_->rank_),
319:       world_size_(pai_->world_size_),
320:       device_idx_(pai_->device_idx_) {
```

- EN: Lines 301-320 introduces executable logic in routines such as `device_idx_`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 301-320 行在 `device_idx_` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 321-340 / 第 321-340 行

```cpp
321:   TORCH_INTERNAL_ASSERT(offset_ < pai_->buffer_size_, "offset out of range");
322: }
323: 
324: std::vector<void*> NCCLSymmetricMemory::get_buffer_ptrs() {
325:   return pai_->buffers_;
326: }
327: 
328: std::vector<void*> NCCLSymmetricMemory::get_signal_pad_ptrs() {
329:   return pai_->signal_pads_;
330: }
331: 
332: void** NCCLSymmetricMemory::get_buffer_ptrs_dev() {
333:   return pai_->buffers_dev_;
334: }
335: 
336: void** NCCLSymmetricMemory::get_signal_pad_ptrs_dev() {
337:   return pai_->signal_pads_dev_;
338: }
339: 
340: size_t NCCLSymmetricMemory::get_buffer_size() {
```

- EN: Lines 321-340 introduces executable logic in routines such as `TORCH_INTERNAL_ASSERT`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 321-340 行在 `TORCH_INTERNAL_ASSERT` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 341-360 / 第 341-360 行

```cpp
341:   return pai_->buffer_size_;
342: }
343: 
344: bool NCCLSymmetricMemory::has_multicast_support() {
345:   return pai_->mc_addr_ != nullptr;
346: }
347: 
348: void* NCCLSymmetricMemory::get_multicast_ptr() {
349:   if (!has_multicast_support()) {
350:     return nullptr;
351:   }
352:   return static_cast<char*>(pai_->mc_addr_) + offset_;
353: }
354: 
355: void NCCLSymmetricMemory::barrier(int channel, size_t timeout_ms) {
356:   TORCH_CHECK(false, "NYI");
357: }
358: 
359: void NCCLSymmetricMemory::put_signal(int dst_rank, int channel, size_t timeout_ms) {
360: #ifdef NCCL_HAS_ONE_SIDED_API
```

- EN: Lines 341-360 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 341-360 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 361-380 / 第 361-380 行

```cpp
361:   TORCH_CHECK(channel == 0, "channel must be 0 (sigIdx is reserved for future use)");
362: 
363:   c10::cuda::CUDAGuard guard(device_idx_);
364:   auto stream = at::cuda::getCurrentCUDAStream();
365: 
366:   auto& manager = NCCLDevCommManager::get(c10::Device(c10::DeviceType::CUDA, device_idx_));
367:   ncclComm_t comm = manager.get_comm(pai_->group_name_);
368: 
369:   // use ncclSignal for pure signaling without data transfer
370:   C10D_NCCL_CHECK(
371:       ncclSignal(
372:           dst_rank,
373:           channel,
374:           0,
375:           0,
376:           comm,
377:           stream),
378:       c10::str("ncclSignal failed for dst_rank=", dst_rank, ", channel=", channel));
379: #else
380:   TORCH_CHECK(false, "NYI");
```

- EN: Lines 361-380 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 361-380 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 381-400 / 第 381-400 行

```cpp
381: #endif
382: }
383: 
384: void NCCLSymmetricMemory::wait_signal(int src_rank, int channel, size_t timeout_ms) {
385: #ifdef NCCL_HAS_ONE_SIDED_API
386:   TORCH_CHECK(channel == 0, "channel must be 0 (sigIdx is reserved for future use)");
387: 
388:   c10::cuda::CUDAGuard guard(device_idx_);
389:   auto stream = at::cuda::getCurrentCUDAStream();
390: 
391:   auto& manager = NCCLDevCommManager::get(c10::Device(c10::DeviceType::CUDA, device_idx_));
392:   ncclComm_t comm = manager.get_comm(pai_->group_name_);
393: 
394:   // create signal descriptor for waiting - populate all fields
395:   ncclWaitSignalDesc_t signalDesc;
396:   signalDesc.opCnt = 1;
397:   signalDesc.peer = src_rank;
398:   signalDesc.sigIdx = channel;
399:   signalDesc.ctx = 0;
400: 
```

- EN: Lines 381-400 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 381-400 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 401-420 / 第 401-420 行

```cpp
401:   C10D_NCCL_CHECK(
402:       ncclWaitSignal(
403:           1,
404:           &signalDesc,
405:           comm,
406:           stream),
407:       c10::str("ncclWaitSignal failed for src_rank=", src_rank, ", channel=", channel));
408: #else
409:   TORCH_CHECK(false, "NYI");
410: #endif
411: }
412: 
413: int NCCLSymmetricMemory::get_rank() {
414:   return rank_;
415: }
416: 
417: int NCCLSymmetricMemory::get_world_size() {
418:   return world_size_;
419: }
420: 
```

- EN: Lines 401-420 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 401-420 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 421-440 / 第 421-440 行

```cpp
421: c10::Device NCCLSymmetricMemory::get_device() {
422:   return c10::Device(c10::DeviceType::CUDA, device_idx_);
423: }
424: 
425: ncclWindow_t NCCLSymmetricMemory::get_window() {
426:   return pai_->buffer_win_;
427: }
428: 
429: ncclWindow_t NCCLSymmetricMemory::get_signal_pad_handle() {
430:   return pai_->signal_handle_;
431: }
432: 
433: size_t NCCLSymmetricMemory::get_offset() {
434:   return offset_;
435: }
436: 
437: std::string NCCLSymmetricMemory::get_group_name() {
438:   return pai_->group_name_;
439: }
440: 
```

- EN: Lines 421-440 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 421-440 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 441-460 / 第 441-460 行

```cpp
441: class NCCLSymmetricMemoryAllocator : public SymmetricMemoryAllocator {
442:  public:
443:   void* alloc(
444:       size_t size,
445:       int device_idx,
446:       const std::optional<std::string>& group_name) override {
447:     TORCH_CHECK(
448:         group_name == std::nullopt,
449:         "NCCLSymmetricMemoryAllocator::alloc "
450:         "must not be called with a group_name");
451: 
452:     c10::cuda::CUDAGuard guard(device_idx);
453:     // TODO: we might need to use a roundup or mempool for mem allocation.
454:     void* ptr;
455:     C10D_NCCL_CHECK(ncclMemAlloc(&ptr, size), "ncclMemAlloc");
456:     {
457:       std::lock_guard<std::mutex> lock(mutex_);
458:       allocations_.emplace(
459:           ptr, std::make_unique<NCCLAllocation>(ptr, size, device_idx));
460:     }
```

- EN: Lines 441-460 declares or defines types such as `NCCLSymmetricMemoryAllocator`; introduces executable logic in routines such as `alloc`, `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 441-460 行声明或定义了 `NCCLSymmetricMemoryAllocator` 等类型；在 `alloc`、`TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 461-480 / 第 461-480 行

```cpp
461:     return ptr;
462:   }
463: 
464:   void free(void* ptr) override {
465:     std::lock_guard<std::mutex> lock(mutex_);
466:     auto alloc_it = allocations_.find(ptr);
467:     if (alloc_it == allocations_.end()) {
468:       return;
469:     }
470:     auto cache_keys_it = symm_mem_keys_by_alloc_.find(ptr);
471:     if (cache_keys_it != symm_mem_keys_by_alloc_.end()) {
472:       for (const auto& key : cache_keys_it->second) {
473:         symm_mems_.erase(key);
474:       }
475:       symm_mem_keys_by_alloc_.erase(cache_keys_it);
476:     }
477:     allocations_.erase(alloc_it);
478:   };
479: 
480:   size_t get_alloc_size(void* ptr) override {
```

- EN: Lines 461-480 introduces executable logic in routines such as `free`, `get_alloc_size`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 461-480 行在 `free`、`get_alloc_size` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 481-500 / 第 481-500 行

```cpp
481:     std::lock_guard<std::mutex> lock(mutex_);
482:     auto it = allocations_.find(ptr);
483:     if (it == allocations_.end()) {
484:       TORCH_CHECK(
485:           false, ptr, " is not allocated with NCCLSymmetricMemoryAllocator");
486:     }
487:     return it->second->buffer_size;
488:   };
489: 
490:   c10::intrusive_ptr<SymmetricMemory> rendezvous(
491:       void* ptr,
492:       const std::optional<std::string>& group_name) override {
493:     TORCH_CHECK(group_name.has_value(), "group_name must be provided");
494:     NCCLAllocation* allocation;
495:     SymmMemKey key{ptr, *group_name};
496:     {
497:       std::lock_guard<std::mutex> lock(mutex_);
498:       auto it = symm_mems_.find(key);
499:       if (it != symm_mems_.end()) {
500:         return it->second;
```

- EN: Lines 481-500 introduces executable logic in routines such as `TORCH_CHECK`, `rendezvous`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 481-500 行在 `TORCH_CHECK`、`rendezvous` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 501-520 / 第 501-520 行

```cpp
501:       }
502: 
503:       // Find the allocation covering the ptr under the allocator lock.
504:       // We grab a raw pointer to the NCCLAllocation so we can release the
505:       // allocator lock before doing expensive per-allocation work.
506:       auto alloc_it = find_allocation_covering(ptr, allocations_);
507:       TORCH_CHECK(
508:           alloc_it != allocations_.end(),
509:           "Pointer not within any SymmetricMemory allocation, "
510:           "is the tensor allocated from SymmetricMemory?");
511:       allocation = alloc_it->second.get();
512:     }
513: 
514:     // Get or create peer alloc info for the group under the per-allocation
515:     // lock. This serializes concurrent rendezvous on the same allocation
516:     // for different groups (e.g., forward vs backward).
517:     std::lock_guard<std::mutex> alloc_lock(allocation->mutex);
518:     auto& peer_alloc_infos = allocation->peer_alloc_infos_;
519:     auto& pai = peer_alloc_infos[*group_name];
520:     if (!pai) {
```

- EN: Lines 501-520 introduces executable logic in routines such as `alloc_lock`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 501-520 行在 `alloc_lock` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 521-540 / 第 521-540 行

```cpp
521:       pai = c10::make_intrusive<NCCLPeerAllocInfo>(allocation, *group_name);
522:     }
523:     size_t offset =
524:         reinterpret_cast<uintptr_t>(ptr) -
525:         reinterpret_cast<uintptr_t>(allocation->ptr);
526:     // Create the SymmetricMemory handle.
527:     auto symm_mem = c10::make_intrusive<NCCLSymmetricMemory>(pai, offset);
528:     {
529:       std::lock_guard<std::mutex> lock(mutex_);
530:       // Insert the SymmetricMemory handle into the map (cache), keyed by the
531:       // (Tensor storage ptr, group name) pair.
532:       auto [it, inserted] = symm_mems_.emplace(key, symm_mem);
533:       if (!inserted) {
534:         // This condition should rarely happen, only when another thread happens
535:         // to be concurrently rendezvousing with the same allocation for the
536:         // same group.  For safety, we return the existing SymmetricMemory
537:         // handle and discard the new one.
538:         return it->second;
539:       }
540:       // There is no more use of `key`; we can move it into the per-allocation
```

- EN: Lines 521-540 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 521-540 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 541-560 / 第 541-560 行

```cpp
541:       // key set to avoid an extra copy.
542:       symm_mem_keys_by_alloc_[allocation->ptr].insert(std::move(key));
543:     }
544:     return symm_mem;
545:   }
546: 
547:   bool has_multicast_support(int device_idx) override {
548:     return device_has_multicast_support(device_idx);
549:   }
550: 
551:   bool has_allocation(void* ptr) override {
552:     std::lock_guard<std::mutex> lock(mutex_);
553:     return find_allocation_covering(ptr, allocations_) != allocations_.end();
554:   }
555: 
556:   c10::DeviceType supported_device_type() override {
557:     return c10::DeviceType::CUDA;
558:   }
559: 
560:   std::string name() override {
```

- EN: Lines 541-560 introduces executable logic in routines such as `has_multicast_support`, `has_allocation`, `supported_device_type`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 541-560 行在 `has_multicast_support`、`has_allocation`、`supported_device_type` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 561-580 / 第 561-580 行

```cpp
561:     return "NCCL";
562:   }
563: 
564:  private:
565:   std::mutex mutex_;
566:   NCCLAllocMap allocations_;
567:   NCCLSymmMemMap symm_mems_;
568:   NCCLSymmMemKeysByAlloc symm_mem_keys_by_alloc_;
569: };
570: 
571: struct RegisterNCCLSymmetricMemoryAllocator {
572:     RegisterNCCLSymmetricMemoryAllocator() {
573:     auto allocator = c10::make_intrusive<NCCLSymmetricMemoryAllocator>();
574:     // Query backend used for CUDA tensor
575:     if (getSymmMemBackendCUDA() == "NCCL") {
576:       // Direct set (static registration)
577:       register_allocator(
578:           c10::DeviceType::CUDA,
579:           allocator);
580:     } else {
```

- EN: Lines 561-580 declares or defines types such as `RegisterNCCLSymmetricMemoryAllocator`; introduces executable logic in routines such as `RegisterNCCLSymmetricMemoryAllocator`, `register_allocator`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 561-580 行声明或定义了 `RegisterNCCLSymmetricMemoryAllocator` 等类型；在 `RegisterNCCLSymmetricMemoryAllocator`、`register_allocator` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 581-591 / 第 581-591 行

```cpp
581:       // Register availability in case `set_backend` is called dynamically
582:       register_availability("NCCL", allocator);
583:     }
584:   }
585: };
586: 
587: static RegisterNCCLSymmetricMemoryAllocator register_allocator_;
588: 
589: } // namespace symmetric_memory
590: } // namespace c10d
591: #endif // NCCL_HAS_SYMMEM_SUPPORT
```

- EN: Lines 581-591 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 581-591 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

## Key Concepts / 关键概念

- EN: Subsystem: c10d symmetric-memory support.
- CN: 子系统：c10d 对称内存支持。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `NCCLAllocation`, `NCCLPeerAllocInfo`, `NCCLSymmetricMemoryAllocator`, `RegisterNCCLSymmetricMemoryAllocator`
- CN: 核心符号：`NCCLAllocation`、`NCCLPeerAllocInfo`、`NCCLSymmetricMemoryAllocator`、`RegisterNCCLSymmetricMemoryAllocator`
- EN: Notable themes: CUDA paths, process-group orchestration, store/state coordination.
- CN: 值得关注的主题：CUDA 路径、进程组编排、存储/状态协调。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/symm_mem/nccl_dev_cap.hpp`, `torch/csrc/distributed/c10d/GroupRegistry.hpp`, `torch/csrc/distributed/c10d/NCCLUtils.hpp`, `torch/csrc/distributed/c10d/ProcessGroupNCCL.hpp`, `torch/csrc/distributed/c10d/cuda/utils.hpp`, `torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemory-inl.cuh`, `torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryTypes.hpp`, `torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryUtils.hpp`, `torch/csrc/distributed/c10d/symm_mem/NCCLSymmetricMemory.hpp`, `torch/csrc/distributed/c10d/symm_mem/nccl_devcomm_manager.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/ceil_div.h`, `ATen/cuda/CUDAContext.h`, `c10/cuda/CUDACachingAllocator.h`, `c10/cuda/CUDAGuard.h`, `c10/util/error.h`, `c10/util/flat_hash_map.h`, `c10/util/hash.h`, `c10/cuda/driver_api.h`
- External or system headers / 外部或系统头文件: `algorithm`, `vector_types.h`, `mutex`
- Local symbols / 本地符号: `NCCLAllocation`, `NCCLPeerAllocInfo`, `NCCLSymmetricMemoryAllocator`, `RegisterNCCLSymmetricMemoryAllocator`