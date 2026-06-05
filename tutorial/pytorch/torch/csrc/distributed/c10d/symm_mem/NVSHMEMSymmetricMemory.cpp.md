# NVSHMEMSymmetricMemory.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/symm_mem/NVSHMEMSymmetricMemory.cpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides implementation logic for nvshmemsymmetric memory in the c10d symmetric-memory support. Key types include `NVSHMEMAllocation`, `NVSHMEMPeerAllocInfo`, `NVSHMEMSymmetricMemory`.
- 用途 (CN): 该文件在c10d 对称内存支持中提供nvshmemsymmetric memory 的实现逻辑。 关键类型包括 `NVSHMEMAllocation`、`NVSHMEMPeerAllocInfo`、`NVSHMEMSymmetricMemory`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

```cpp
1: #include <torch/csrc/distributed/c10d/GroupRegistry.hpp>
2: #include <torch/csrc/distributed/c10d/cuda/utils.hpp>
3: #include <torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryTypes.hpp>
4: #include <torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryUtils.hpp>
5: #include <torch/csrc/distributed/c10d/symm_mem/SymmetricMemory.hpp>
6: #include <torch/csrc/distributed/c10d/symm_mem/nvshmem_extension.hpp>
7: #include <torch/csrc/distributed/c10d/symm_mem/nvshmem_team_manager.hpp>
8: 
9: #include <ATen/ceil_div.h>
10: #include <ATen/cuda/CUDAContext.h>
11: #include <c10/cuda/CUDACachingAllocator.h>
12: #include <c10/cuda/CUDAGuard.h>
13: #include <c10/util/error.h>
14: #include <c10/util/flat_hash_map.h>
15: 
16: #include <mutex>
17: 
18: // Starting from NVSHMEM 3.3.9, nvshmem_host.h exists so that we can cleanly
19: // include only the nvshmem host library headers:
20: // #include <nvshmem_host.h>
```

- EN: Lines 1-20 pulls in the headers required by this translation unit or interface.
- CN: 第 1-20 行引入该实现单元或接口所需的头文件。

### Lines 21-40 / 第 21-40 行

```cpp
21: // It translates into the following two lines:
22: #if !defined(USE_ROCM)
23: #include <host/nvshmem_api.h>
24: #include <host/nvshmemx_api.h>
25: #endif
26: // For maximum compatibility, we use the "host/" style for now.
27: 
28: namespace c10d {
29: namespace symmetric_memory {
30: 
31: /* Start of NVSHMEMSymmetricMemory implementation */
32: 
33: static StoreExchange storeExchange = StoreExchange("NVSHMEMSymmetricMemory");
34: 
35: struct NVSHMEMAllocation {
36:   void* ptr;
37:   size_t buffer_size;
38:   int device_idx;
39: 
40:   NVSHMEMAllocation(void* ptr, size_t buffer_size, int device_idx)
```

- EN: Lines 21-40 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 21-40 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 41-60 / 第 41-60 行

```cpp
41:       : ptr(ptr), buffer_size(buffer_size), device_idx(device_idx) {}
42: 
43:   // Delete copy and move operations to prevent double-free
44:   NVSHMEMAllocation(const NVSHMEMAllocation&) = delete;
45:   NVSHMEMAllocation& operator=(const NVSHMEMAllocation&) = delete;
46:   NVSHMEMAllocation(NVSHMEMAllocation&&) = delete;
47:   NVSHMEMAllocation& operator=(NVSHMEMAllocation&&) = delete;
48: 
49:   ~NVSHMEMAllocation() {
50:     // Avoid calling CUDA functions after driver shutting down
51:     if (is_finalizing()) {
52:       return;
53:     }
54:     c10::cuda::CUDAGuard guard(device_idx);
55:     nvshmem_free(ptr); // nvshmem_free has no return value
56:   }
57: };
58: 
59: // A map from group name to rank-to-global rank mapping
60: static std::mutex rank_map_mutex;
```

- EN: Lines 41-60 introduces executable logic in routines such as `~NVSHMEMAllocation`; returns computed state or forwards results to the surrounding caller.
- CN: 第 41-60 行在 `~NVSHMEMAllocation` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 61-80 / 第 61-80 行

```cpp
61: static std::unordered_map<std::string, std::vector<int>>
62:     rank_to_global_rank_map{};
63: // A map from group name to rank-to-global rank device array
64: static std::unordered_map<std::string, int*> rank_to_global_rank_dev_map{};
65: 
66: // A class to hold the base pointers and signal pad pointers for a group of
67: // peers. One `NVSHMEMPeerAllocInfo` object can be shared by multiple
68: // `NVSHMEMSymmetricMemory` objects when latter reside on the same allocation
69: // and rendezvous over the same group. (The `NVSHMEMSymmetricMemory` objects may
70: // have different offsets compared to the base address.)
71: class NVSHMEMPeerAllocInfo : public c10::intrusive_ptr_target {
72:  public:
73:   NVSHMEMPeerAllocInfo(
74:       NVSHMEMAllocation* allocation,
75:       const std::string& group_name)
76:       : base_ptr_(allocation->ptr), buffer_size_(allocation->buffer_size) {
77:     // For logging only
78:     static int exchanged_n_times = 0;
79:     c10::cuda::CUDAGuard guard(allocation->device_idx);
80: 
```

- EN: Lines 61-80 declares or defines types such as `NVSHMEMPeerAllocInfo`; introduces executable logic in routines such as `guard`.
- CN: 第 61-80 行声明或定义了 `NVSHMEMPeerAllocInfo` 等类型；在 `guard` 等例程中引入具体执行逻辑。

### Lines 81-100 / 第 81-100 行

```cpp
81:     auto group = resolve_process_group(group_name);
82:     rank_ = group->getRank();
83:     world_size_ = group->getSize();
84:     auto store = group->getStore();
85: 
86:     // Exchange rank to global rank mapping for this group.
87:     // If it is already available, skip the exchange.
88:     std::lock_guard<std::mutex> rank_map_lock(rank_map_mutex);
89:     auto it = rank_to_global_rank_map.find(group_name);
90:     if (it == rank_to_global_rank_map.end()) {
91:       auto global_group = resolve_process_group("0");
92:       auto global_rank = global_group->getRank();
93:       auto rank_to_global_rank =
94:           storeExchange.all_gather(store, rank_, world_size_, global_rank);
95:       exchanged_n_times++;
96:       if (rank_ == 0) {
97:         LOG(INFO) << "[rank " << rank_ << ']'
98:                   << " rank_to_global_rank: " << rank_to_global_rank
99:                   << ", group_name: " << group_name
100:                   << ", exchanged_n_times: " << exchanged_n_times;
```

- EN: Lines 81-100 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 81-100 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 101-120 / 第 101-120 行

```cpp
101:       }
102:       it = rank_to_global_rank_map.emplace_hint(
103:           it, group_name, rank_to_global_rank);
104: 
105:       // Emplace a device array of rank to global rank mapping
106:       auto rank_to_global_rank_dev =
107:           reinterpret_cast<int*>(c10::cuda::CUDACachingAllocator::raw_alloc(
108:               sizeof(int) * world_size_));
109:       AT_CUDA_CHECK(cudaMemcpy(
110:           rank_to_global_rank_dev,
111:           rank_to_global_rank.data(),
112:           sizeof(int) * world_size_,
113:           cudaMemcpyHostToDevice));
114:       rank_to_global_rank_dev_map[group_name] = rank_to_global_rank_dev;
115:     }
116:     auto& rank_to_global_rank = it->second;
117: 
118:     world_within_cuda_p2p_ = true;
119:     for (int r = 0; r < world_size_; ++r) {
120:       auto peer_ptr = nvshmem_ptr(base_ptr_, rank_to_global_rank[r]);
```

- EN: Lines 101-120 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 101-120 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 121-140 / 第 121-140 行

```cpp
121:       buffers_.push_back(peer_ptr);
122:       // If a peer is over network, `nvshmem_ptr` returns null
123:       if (peer_ptr == nullptr) {
124:         world_within_cuda_p2p_ = false;
125:       }
126:     }
127: 
128:     // TODO: use the same allocation for signal pad
129:     const size_t signal_pad_size = get_signal_pad_size();
130:     void* signal_pad_ptr = nvshmem_malloc(signal_pad_size);
131:     TORCH_CHECK(signal_pad_ptr != nullptr, "nvshmem_malloc failed");
132:     AT_CUDA_CHECK(cudaMemset(signal_pad_ptr, 0, signal_pad_size));
133: 
134:     for (int r = 0; r < world_size_; ++r) {
135:       signal_pads_.push_back(
136:           nvshmem_ptr(signal_pad_ptr, rank_to_global_rank[r]));
137:     }
138: 
139:     const size_t arr_size = sizeof(void*) * world_size_;
140:     buffers_dev_ = reinterpret_cast<void**>(
```

- EN: Lines 121-140 introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 121-140 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 141-160 / 第 141-160 行

```cpp
141:         c10::cuda::CUDACachingAllocator::raw_alloc(arr_size));
142:     signal_pads_dev_ = reinterpret_cast<void**>(
143:         c10::cuda::CUDACachingAllocator::raw_alloc(arr_size));
144: 
145:     AT_CUDA_CHECK(cudaMemcpy(
146:         buffers_dev_, buffers_.data(), arr_size, cudaMemcpyHostToDevice));
147:     AT_CUDA_CHECK(cudaMemcpy(
148:         signal_pads_dev_,
149:         signal_pads_.data(),
150:         arr_size,
151:         cudaMemcpyHostToDevice));
152: 
153: #if !defined(USE_ROCM) // Multi-cast is not supported on ROCm yet
154:     // Initialize multicast address
155:     // On unsupported platforms, this API returns a nullptr.
156:     auto device = c10::Device(c10::DeviceType::CUDA, allocation->device_idx);
157:     auto& team_manager = c10d::nvshmem_extension::TeamManager::get(device);
158:     auto team = team_manager.get_team(group_name, rank_to_global_rank);
159:     mc_addr_ = nvshmemx_mc_ptr(team, base_ptr_);
160: #endif
```

- EN: Lines 141-160 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 141-160 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 161-180 / 第 161-180 行

```cpp
161:   }
162: 
163:  private:
164:   void* base_ptr_;
165:   size_t buffer_size_;
166:   int rank_;
167:   int world_size_;
168:   std::vector<void*> buffers_;
169:   std::vector<void*> signal_pads_;
170:   void** buffers_dev_;
171:   void** signal_pads_dev_;
172:   // Whether the world is within CUDA P2P only, not network
173:   bool world_within_cuda_p2p_;
174:   // Multicast address
175:   void* mc_addr_{nullptr};
176: 
177:   friend class NVSHMEMSymmetricMemory;
178: };
179: 
180: class NVSHMEMSymmetricMemory : public SymmetricMemory {
```

- EN: Lines 161-180 declares or defines types such as `NVSHMEMSymmetricMemory`.
- CN: 第 161-180 行声明或定义了 `NVSHMEMSymmetricMemory` 等类型。

### Lines 181-200 / 第 181-200 行

```cpp
181:  public:
182:   NVSHMEMSymmetricMemory(
183:       NVSHMEMAllocation* allocation,
184:       const std::string& group_name)
185:       : device_idx_(allocation->device_idx), group_name_(group_name) {
186:     // A handle stores two types of info:
187:     // (i) allocation's base ptrs and base signal pads, ours and peers'
188:     pai_ = c10::make_intrusive<NVSHMEMPeerAllocInfo>(allocation, group_name);
189:     // (ii) offset of tensor compared to base ptr (in byte)
190:     offset_ = 0;
191:   }
192: 
193:   // Exact copy is not needed / supported
194:   NVSHMEMSymmetricMemory(const NVSHMEMSymmetricMemory& other) = delete;
195: 
196:   // Copy with offset is allowed
197:   // This is mostly a shallow copy that shares the pointer to
198:   // `NVSHMEMPeerAllocInfo` which has been created by `other`
199:   NVSHMEMSymmetricMemory(const NVSHMEMSymmetricMemory& other, size_t offset)
200:       : device_idx_(other.device_idx_),
```

- EN: Lines 181-200 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 181-200 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 201-220 / 第 201-220 行

```cpp
201:         group_name_(other.group_name_),
202:         pai_(other.pai_) {
203:     offset_ = offset;
204:   }
205: 
206:   ~NVSHMEMSymmetricMemory() override {
207:     // TODO
208:   };
209: 
210:   std::vector<void*> get_buffer_ptrs() override {
211:     return pai_->buffers_;
212:   }
213: 
214:   std::vector<void*> get_signal_pad_ptrs() override {
215:     return pai_->signal_pads_;
216:   }
217: 
218:   void** get_buffer_ptrs_dev() override {
219:     return pai_->buffers_dev_;
220:   }
```

- EN: Lines 201-220 introduces executable logic in routines such as `~NVSHMEMSymmetricMemory`, `get_buffer_ptrs`, `get_signal_pad_ptrs`; returns computed state or forwards results to the surrounding caller.
- CN: 第 201-220 行在 `~NVSHMEMSymmetricMemory`、`get_buffer_ptrs`、`get_signal_pad_ptrs` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 221-240 / 第 221-240 行

```cpp
221: 
222:   void** get_signal_pad_ptrs_dev() override {
223:     return pai_->signal_pads_dev_;
224:   }
225: 
226:   size_t get_buffer_size() override {
227:     return pai_->buffer_size_;
228:   }
229: 
230:   bool has_multicast_support() override {
231:     return pai_->mc_addr_ != nullptr;
232:   }
233: 
234:   void* get_multicast_ptr() override {
235:     if (!has_multicast_support()) {
236:       return nullptr;
237:     }
238:     return static_cast<char*>(pai_->mc_addr_) + offset_;
239:   }
240: 
```

- EN: Lines 221-240 introduces executable logic in routines such as `get_signal_pad_ptrs_dev`, `get_buffer_size`, `has_multicast_support`; returns computed state or forwards results to the surrounding caller.
- CN: 第 221-240 行在 `get_signal_pad_ptrs_dev`、`get_buffer_size`、`has_multicast_support` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 241-260 / 第 241-260 行

```cpp
241:   size_t get_offset() override {
242:     return offset_;
243:   }
244: 
245:   void barrier(int channel, size_t timeout_ms) override {
246:     // TODO
247:   }
248: 
249:   void put_signal(int dst_rank, int channel, size_t timeout_ms) override {
250:     // TODO
251:   }
252: 
253:   void wait_signal(int src_rank, int channel, size_t timeout_ms) override {
254:     // TODO
255:   }
256: 
257:   int get_rank() override {
258:     return pai_->rank_;
259:   }
260: 
```

- EN: Lines 241-260 introduces executable logic in routines such as `get_offset`, `barrier`, `put_signal`; returns computed state or forwards results to the surrounding caller.
- CN: 第 241-260 行在 `get_offset`、`barrier`、`put_signal` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 261-280 / 第 261-280 行

```cpp
261:   int get_world_size() override {
262:     return pai_->world_size_;
263:   }
264: 
265:   c10::Device get_device() override {
266:     return c10::Device(c10::DeviceType::CUDA, device_idx_);
267:   }
268: 
269:   const std::vector<int>& get_rank_to_global_rank() override {
270:     std::lock_guard<std::mutex> lock(rank_map_mutex);
271:     auto it = rank_to_global_rank_map.find(group_name_);
272:     if (it == rank_to_global_rank_map.end()) {
273:       TORCH_CHECK(false, "Group name not found in rank_to_global_rank_map");
274:     }
275:     return it->second;
276:   };
277: 
278:   int* get_rank_to_global_rank_dev() override {
279:     std::lock_guard<std::mutex> lock(rank_map_mutex);
280:     auto it = rank_to_global_rank_dev_map.find(group_name_);
```

- EN: Lines 261-280 introduces executable logic in routines such as `get_world_size`, `get_device`, `get_rank_to_global_rank`; performs validation and error handling to keep distributed state consistent.
- CN: 第 261-280 行在 `get_world_size`、`get_device`、`get_rank_to_global_rank` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 281-300 / 第 281-300 行

```cpp
281:     if (it == rank_to_global_rank_dev_map.end()) {
282:       TORCH_CHECK(false, "Group name not found in rank_to_global_rank_dev_map");
283:     }
284:     return it->second;
285:   };
286: 
287:   bool world_within_direct_access() override {
288:     return pai_->world_within_cuda_p2p_;
289:   }
290: 
291:  private:
292:   int device_idx_;
293:   std::string group_name_;
294:   c10::intrusive_ptr<NVSHMEMPeerAllocInfo> pai_;
295:   size_t offset_{0}; // in byte
296: };
297: 
298: // Bootstrap based on user's setting for NCCL
299: // Long term, this may be a bit unclean; short term, it improves UX
300: static void maybe_initialize_env_vars() {
```

- EN: Lines 281-300 introduces executable logic in routines such as `TORCH_CHECK`, `world_within_direct_access`, `maybe_initialize_env_vars`; performs validation and error handling to keep distributed state consistent.
- CN: 第 281-300 行在 `TORCH_CHECK`、`world_within_direct_access`、`maybe_initialize_env_vars` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 301-320 / 第 301-320 行

```cpp
301:   auto nccl_socket_if_name = c10::utils::get_env("NCCL_SOCKET_IFNAME");
302:   auto nccl_hca_list = c10::utils::get_env("NCCL_IB_HCA");
303:   auto nccl_ib_gid_index = c10::utils::get_env("NCCL_IB_GID_INDEX");
304:   auto nvshmem_socket_if_name =
305:       c10::utils::get_env("NVSHMEM_BOOTSTRAP_UID_SOCK_IFNAME");
306:   auto nvshmem_hca_list = c10::utils::get_env("NVSHMEM_HCA_LIST");
307:   auto nvshmem_ib_gid_index = c10::utils::get_env("NVSHMEM_IB_GID_INDEX");
308: 
309:   if (!nvshmem_socket_if_name.has_value() && nccl_socket_if_name.has_value()) {
310:     c10::utils::set_env(
311:         "NVSHMEM_BOOTSTRAP_UID_SOCK_IFNAME", nccl_socket_if_name->c_str());
312:   }
313:   if (!nvshmem_hca_list.has_value() && nccl_hca_list.has_value()) {
314:     c10::utils::set_env("NVSHMEM_ENABLE_NIC_PE_MAPPING", "1");
315:     c10::utils::set_env("NVSHMEM_HCA_LIST", nccl_hca_list->c_str());
316:   }
317:   if (!nvshmem_ib_gid_index.has_value() && nccl_ib_gid_index.has_value()) {
318:     c10::utils::set_env("NVSHMEM_IB_GID_INDEX", nccl_ib_gid_index->c_str());
319:   }
320: }
```

- EN: Lines 301-320 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 301-320 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 321-340 / 第 321-340 行

```cpp
321: 
322: static void initialize_nvshmem_with_store(
323:     c10::intrusive_ptr<c10d::Store> store,
324:     int rank,
325:     int world_size,
326:     int device_idx) {
327:   static bool is_initialized = false;
328:   if (is_initialized) {
329:     return;
330:   }
331: 
332:   c10::cuda::CUDAGuard guard(device_idx);
333:   maybe_initialize_env_vars();
334:   // Make sure the CUDA runtime is initialized.
335:   cudaFree(nullptr);
336: 
337:   nvshmemx_uniqueid_t unique_id;
338:   NVSHMEM_CHECK(
339:       nvshmemx_get_uniqueid(&unique_id), "nvshmemx_get_uniqueid failed");
340: 
```

- EN: Lines 321-340 introduces executable logic in routines such as `initialize_nvshmem_with_store`, `maybe_initialize_env_vars`; returns computed state or forwards results to the surrounding caller.
- CN: 第 321-340 行在 `initialize_nvshmem_with_store`、`maybe_initialize_env_vars` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 341-360 / 第 341-360 行

```cpp
341:   // Using an existing store_all_gather due to laziness.
342:   // TODO(yifu): should use broadcast
343:   auto unique_ids =
344:       storeExchange.all_gather(store, rank, world_size, unique_id);
345: 
346:   nvshmemx_init_attr_t attr;
347:   nvshmemx_set_attr_uniqueid_args(rank, world_size, &unique_ids[0], &attr);
348: 
349:   NVSHMEM_CHECK(
350:       nvshmemx_init_attr(NVSHMEMX_INIT_WITH_UNIQUEID, &attr),
351:       "nvshmemx_init_attr failed");
352: 
353:   is_initialized = true;
354: 
355:   // Print version
356: #if !defined(USE_ROCM)
357:   int major, minor;
358:   ::nvshmem_info_get_version(&major, &minor);
359:   LOG(INFO) << "NVSHMEM is available, version: " << major << '.' << minor;
360: #endif
```

- EN: Lines 341-360 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `nvshmemx_set_attr_uniqueid_args`.
- CN: 第 341-360 行使用条件编译来适配特性开关、平台或可选后端；在 `nvshmemx_set_attr_uniqueid_args` 等例程中引入具体执行逻辑。

### Lines 361-380 / 第 361-380 行

```cpp
361: }
362: 
363: class NVSHMEMSymmetricMemoryAllocator : public SymmetricMemoryAllocator {
364:  public:
365:   void* alloc(
366:       size_t size,
367:       int device_idx,
368:       const std::optional<std::string>& group_name) override {
369:     TORCH_CHECK(
370:         group_name == std::nullopt,
371:         "NVSHMEMSymmetricMemoryAllocator::alloc "
372:         "must not be called with a group_name");
373:     c10::cuda::CUDAGuard guard(device_idx);
374: 
375:     // NVSHMEM needs to be initialized with the global group
376:     auto group = resolve_process_group("0");
377:     initialize_nvshmem_with_store(
378:         group->getStore(), group->getRank(), group->getSize(), device_idx);
379: 
380:     auto ptr = nvshmem_malloc(size);
```

- EN: Lines 361-380 declares or defines types such as `NVSHMEMSymmetricMemoryAllocator`; introduces executable logic in routines such as `alloc`, `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 361-380 行声明或定义了 `NVSHMEMSymmetricMemoryAllocator` 等类型；在 `alloc`、`TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 381-400 / 第 381-400 行

```cpp
381:     // If size is 0 (which is legal allocation request) we shouldn't error out
382:     TORCH_CHECK(ptr != nullptr || size == 0, "nvshmem_malloc failed");
383:     {
384:       std::lock_guard<std::mutex> lock(mutex_);
385:       allocations_.try_emplace(
386:           ptr, std::make_unique<NVSHMEMAllocation>(ptr, size, device_idx));
387:     }
388:     return ptr;
389:   }
390: 
391:   void free(void* ptr) override {
392:     std::lock_guard<std::mutex> lock(mutex_);
393:     allocations_.erase(ptr);
394:   };
395: 
396:   size_t get_alloc_size(void* ptr) override {
397:     std::lock_guard<std::mutex> lock(mutex_);
398:     auto it = allocations_.find(ptr);
399:     if (it == allocations_.end()) {
400:       TORCH_CHECK(
```

- EN: Lines 381-400 introduces executable logic in routines such as `TORCH_CHECK`, `free`, `get_alloc_size`; performs validation and error handling to keep distributed state consistent.
- CN: 第 381-400 行在 `TORCH_CHECK`、`free`、`get_alloc_size` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 401-420 / 第 401-420 行

```cpp
401:           false, ptr, " is not allocated with NVSHMEMSymmetricMemoryAllocator");
402:     }
403:     return it->second->buffer_size;
404:   };
405: 
406:   c10::intrusive_ptr<SymmetricMemory> rendezvous(
407:       void* ptr,
408:       const std::optional<std::string>& group_name) override {
409:     TORCH_CHECK(group_name.has_value());
410:     std::lock_guard<std::mutex> lock(mutex_);
411:     {
412:       auto it = symm_mems_.find(SymmMemKey{ptr, *group_name});
413:       if (it != symm_mems_.end()) {
414:         return it->second;
415:       }
416:     }
417:     // In case of MemPool, tensor.storage().data_ptr() may not match
418:     // exactly an allocation's base address. Thus we perform the search by
419:     // testing if the former is within an allocation's range.
420:     auto alloc_it = std::find_if(
```

- EN: Lines 401-420 introduces executable logic in routines such as `rendezvous`; performs validation and error handling to keep distributed state consistent.
- CN: 第 401-420 行在 `rendezvous` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 421-440 / 第 421-440 行

```cpp
421:         allocations_.begin(), allocations_.end(), [&](const auto& pair) {
422:           auto& allocation = pair.second;
423:           auto ptr_int = reinterpret_cast<uintptr_t>(ptr);
424:           auto base_ptr = reinterpret_cast<uintptr_t>(allocation->ptr);
425:           return ptr_int >= base_ptr &&
426:               ptr_int < base_ptr + allocation->buffer_size;
427:         });
428:     TORCH_CHECK(
429:         alloc_it != allocations_.end(),
430:         "Pointer not within any SymmetricMemory allocation, "
431:         "is the tensor allocated from SymmetricMemory?");
432: 
433:     auto& allocation = alloc_it->second;
434: 
435:     // Search again using allocation base ptr (which is the key we use for
436:     // caching, see below)
437:     auto it = symm_mems_.find(SymmMemKey{allocation->ptr, *group_name});
438:     c10::intrusive_ptr<NVSHMEMSymmetricMemory> symm_mem;
439:     if (it != symm_mems_.end()) {
440:       // Base allocation has been rendezvoused
```

- EN: Lines 421-440 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 421-440 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 441-460 / 第 441-460 行

```cpp
441:       symm_mem = it->second;
442:     } else {
443:       // Create a new rendezvous
444:       symm_mem = c10::make_intrusive<NVSHMEMSymmetricMemory>(
445:           allocation.get(), *group_name);
446:     }
447: 
448:     // Cache rendezvous using allocation's base address as key
449:     symm_mems_[SymmMemKey{allocation->ptr, *group_name}] = symm_mem;
450: 
451:     // TODO: change the `ptr` below to `tensor.data_ptr()` when adding support
452:     // for user slice/view operations. For MemPool support,
453:     // `tensor.storage().data_ptr()` is fine (today's `ptr`).
454: 
455:     // If the tensor's ptr happen to be the same as allocation ptr
456:     if (ptr == allocation->ptr) {
457:       return symm_mem;
458:     } else {
459:       // Return a copy of the SymmetricMemory with an offset. This is a
460:       // "shallow" copy adjusting the offset field in the handle.
```

- EN: Lines 441-460 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 441-460 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 461-480 / 第 461-480 行

```cpp
461:       return c10::make_intrusive<NVSHMEMSymmetricMemory>(
462:           *symm_mem, (uintptr_t)ptr - (uintptr_t)allocation->ptr);
463:     }
464:   };
465: 
466:   bool has_multicast_support(int device_idx) override {
467:     return device_has_multicast_support(device_idx);
468:   }
469: 
470:   bool has_allocation(void* ptr) override {
471:     std::lock_guard<std::mutex> lock(mutex_);
472:     auto alloc_it = std::find_if(
473:         allocations_.begin(), allocations_.end(), [&](const auto& pair) {
474:           auto ptr_int = reinterpret_cast<uintptr_t>(ptr);
475:           auto base_ptr = reinterpret_cast<uintptr_t>(pair.second->ptr);
476:           return ptr_int >= base_ptr &&
477:               ptr_int < base_ptr + pair.second->buffer_size;
478:         });
479:     return alloc_it != allocations_.end();
480:   }
```

- EN: Lines 461-480 introduces executable logic in routines such as `has_multicast_support`, `has_allocation`; returns computed state or forwards results to the surrounding caller.
- CN: 第 461-480 行在 `has_multicast_support`、`has_allocation` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 481-500 / 第 481-500 行

```cpp
481: 
482:   c10::DeviceType supported_device_type() override {
483:     return c10::DeviceType::CUDA;
484:   }
485: 
486:   std::string name() override {
487:     return "NVSHMEM";
488:   }
489: 
490:  private:
491:   std::mutex mutex_;
492:   std::unordered_map<void*, std::unique_ptr<NVSHMEMAllocation>> allocations_;
493:   ska::flat_hash_map<
494:       SymmMemKey,
495:       c10::intrusive_ptr<NVSHMEMSymmetricMemory>,
496:       SymmMemKeyHash>
497:       symm_mems_;
498: };
499: 
500: struct RegisterNVSHMEMSymmetricMemoryAllocator {
```

- EN: Lines 481-500 declares or defines types such as `RegisterNVSHMEMSymmetricMemoryAllocator`; introduces executable logic in routines such as `supported_device_type`, `name`.
- CN: 第 481-500 行声明或定义了 `RegisterNVSHMEMSymmetricMemoryAllocator` 等类型；在 `supported_device_type`、`name` 等例程中引入具体执行逻辑。

### Lines 501-517 / 第 501-517 行

```cpp
501:   RegisterNVSHMEMSymmetricMemoryAllocator() {
502:     auto allocator = c10::make_intrusive<NVSHMEMSymmetricMemoryAllocator>();
503:     // Query backend used for CUDA tensor
504:     if (getSymmMemBackendCUDA() == "NVSHMEM") {
505:       // Direct set (static registration)
506:       register_allocator(c10::DeviceType::CUDA, allocator);
507:     } else {
508:       // Register availability in case `set_backend` is called dynamically
509:       register_availability("NVSHMEM", allocator);
510:     }
511:   }
512: };
513: 
514: static RegisterNVSHMEMSymmetricMemoryAllocator register_allocator_;
515: 
516: } // namespace symmetric_memory
517: } // namespace c10d
```

- EN: Lines 501-517 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `RegisterNVSHMEMSymmetricMemoryAllocator`, `register_allocator`.
- CN: 第 501-517 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `RegisterNVSHMEMSymmetricMemoryAllocator`、`register_allocator` 等例程中引入具体执行逻辑。

## Key Concepts / 关键概念

- EN: Subsystem: c10d symmetric-memory support.
- CN: 子系统：c10d 对称内存支持。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `NVSHMEMAllocation`, `NVSHMEMPeerAllocInfo`, `NVSHMEMSymmetricMemory`, `NVSHMEMSymmetricMemoryAllocator`
- CN: 核心符号：`NVSHMEMAllocation`、`NVSHMEMPeerAllocInfo`、`NVSHMEMSymmetricMemory`、`NVSHMEMSymmetricMemoryAllocator`
- EN: Notable themes: store/state coordination.
- CN: 值得关注的主题：存储/状态协调。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/GroupRegistry.hpp`, `torch/csrc/distributed/c10d/cuda/utils.hpp`, `torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryTypes.hpp`, `torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryUtils.hpp`, `torch/csrc/distributed/c10d/symm_mem/SymmetricMemory.hpp`, `torch/csrc/distributed/c10d/symm_mem/nvshmem_extension.hpp`, `torch/csrc/distributed/c10d/symm_mem/nvshmem_team_manager.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/ceil_div.h`, `ATen/cuda/CUDAContext.h`, `c10/cuda/CUDACachingAllocator.h`, `c10/cuda/CUDAGuard.h`, `c10/util/error.h`, `c10/util/flat_hash_map.h`
- External or system headers / 外部或系统头文件: `mutex`, `host/nvshmem_api.h`, `host/nvshmemx_api.h`
- Local symbols / 本地符号: `NVSHMEMAllocation`, `NVSHMEMPeerAllocInfo`, `NVSHMEMSymmetricMemory`, `NVSHMEMSymmetricMemoryAllocator`