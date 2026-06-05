# rocshmem_extension.cu — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/symm_mem/rocshmem_extension.cu`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides implementation logic for rocshmem extension in the c10d symmetric-memory support. Top-of-file note: ROCm implementation of the NVSHMEM symmetric memory extension ops. This is a separate file from nvshmem_extension.cu (rather than a hipified copy) for the following reasons: 1. ... Representative routines include `parse_rocshmem_version_ge`, `is_nvshmem_available`, `nvshmemx_cumodule_init`, `nvshmem_broadcast`, `TORCH_CHECK`. GPU-oriented code paths are present in this file.
- 用途 (CN): 该文件在c10d 对称内存支持中提供rocshmem extension 的实现逻辑。文件开头备注：ROCm implementation of the NVSHMEM symmetric memory extension ops. This is a separate file from nvshmem_extension.cu (rather than a hipified copy) for the following reasons: 1. ... 代表性例程包括 `parse_rocshmem_version_ge`、`is_nvshmem_available`、`nvshmemx_cumodule_init`、`nvshmem_broadcast`、`TORCH_CHECK`。 该文件还包含面向 GPU 的代码路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

```cpp
1: // ROCm implementation of the NVSHMEM symmetric memory extension ops.
2: //
3: // This is a separate file from nvshmem_extension.cu (rather than a hipified
4: // copy) for the following reasons:
5: //
6: // 1. API differences: NVSHMEM and rocSHMEM device APIs diverge enough that
7: //    #ifdef'ing would be more noise than signal. Key differences include:
8: //    - nvshmemx_collective_launch (grid-wide sync) has no rocSHMEM equivalent;
9: //      ROCm uses regular hip kernel launches with host-side barriers instead.
10: //    - nvshmemx_getmem_nbi_block → rocshmem_getmem_nbi_wg (workgroup scope).
11: //
12: // 2. Missing features: rocSHMEM does not yet support tiled communication
13: //    (nvshmemx::Tensor, nvshmemx::tile_sum_reduce_block, etc.), so the
14: //    tile_reduce and multi_root_tile_reduce ops are not included here.
15: //
16: // 3. Offset writeback: without grid-wide sync, multi-block kernels cannot
17: //    safely write output offsets in-kernel (race with blocks still reading
18: //    source_offsets). A separate writeOutputOffsets kernel runs after the data
19: //    exchange completes.
20: 
```

- EN: Lines 1-20 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 1-20 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 21-40 / 第 21-40 行

```cpp
21: #include <hip/hip_runtime.h>
22: #include <algorithm>
23: #include <cstdlib>
24: #include <vector>
25: #include <ATen/ceil_div.h>
26: #include <c10/hip/HIPGuard.h>
27: 
28: #include <torch/csrc/distributed/c10d/symm_mem/env.hpp>
29: #include <torch/csrc/distributed/c10d/symm_mem/nvshmem_extension.hpp>
30: #include <torch/csrc/distributed/c10d/symm_mem/nvshmem_team_manager.hpp>
31: #include <torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemory-inl.cuh>
32: #include <torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryUtils.hpp>
33: #include <torch/csrc/distributed/c10d/symm_mem/SymmetricMemory.hpp>
34: 
35: #include <ATen/hip/cub.cuh>
36: 
37: #include <c10/hip/HIPException.h>
38: #include <rocshmem/rocshmem.hpp>
39: 
40: using namespace rocshmem;
```

- EN: Lines 21-40 pulls in the headers required by this translation unit or interface; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 21-40 行引入该实现单元或接口所需的头文件；包含面向 CUDA 的声明、内核或启动流程。

### Lines 41-60 / 第 41-60 行

```cpp
41: namespace c10d::nvshmem_extension {
42: 
43: #define THREADS_PER_BLOCK 512
44: #define WARP_SIZE 64
45: 
46: namespace {
47: 
48: bool parse_rocshmem_version_ge(
49:     const char* version,
50:     unsigned min_major,
51:     unsigned min_minor,
52:     unsigned min_patch) {
53:   if (version == nullptr) {
54:     return false;
55:   }
56:   char* end = nullptr;
57:   unsigned long major = std::strtoul(version, &end, 10);
58:   if (end == version || *end != '.') {
59:     return false;
60:   }
```

- EN: Lines 41-60 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `parse_rocshmem_version_ge`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 41-60 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `parse_rocshmem_version_ge` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 61-80 / 第 61-80 行

```cpp
61:   version = end + 1;
62:   unsigned long minor = std::strtoul(version, &end, 10);
63:   if (end == version || *end != '.') {
64:     return false;
65:   }
66:   version = end + 1;
67:   unsigned long patch = std::strtoul(version, &end, 10);
68:   if (end == version) {
69:     return false;
70:   }
71:   if (major > min_major) {
72:     return true;
73:   }
74:   if (major < min_major) {
75:     return false;
76:   }
77:   if (minor > min_minor) {
78:     return true;
79:   }
80:   if (minor < min_minor) {
```

- EN: Lines 61-80 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 61-80 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 81-100 / 第 81-100 行

```cpp
81:     return false;
82:   }
83:   return patch >= min_patch;
84: }
85: 
86: } // namespace
87: 
88: extern "C" void rocshmem_init() __attribute__((weak));
89: 
90: bool is_nvshmem_available() {
91:   static const bool ok =
92:       parse_rocshmem_version_ge(rocshmem::VERSION, 3, 3, 0);
93:   return ok;
94: }
95: 
96: void nvshmemx_cumodule_init(uintptr_t module) {
97:   auto hipmodule = reinterpret_cast<hipModule_t>(module);
98:   NVSHMEM_CHECK(
99:     rocshmem_hipmodule_init(hipmodule),
100:     "rocshmem_hipmodule_init failed");
```

- EN: Lines 81-100 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `is_nvshmem_available`, `parse_rocshmem_version_ge`, `nvshmemx_cumodule_init`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 81-100 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `is_nvshmem_available`、`parse_rocshmem_version_ge`、`nvshmemx_cumodule_init` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 101-120 / 第 101-120 行

```cpp
101: }
102: 
103: at::Tensor nvshmem_broadcast(at::Tensor& input, const int64_t root, const std::string& group_name) {
104:   auto input_hdl = c10d::symmetric_memory::rendezvous(input, group_name);
105:   int rank = input_hdl->get_rank();
106:   void* buffer_ptr = input.mutable_data_ptr();
107:   auto buffer_size = input.numel() * input.element_size();
108:   auto& team_manager = TeamManager::get(input.device());
109:   auto team = team_manager.get_team(group_name, input_hdl->get_rank_to_global_rank());
110:   int team_size = rocshmem_team_n_pes(team);
111:   TORCH_CHECK(root < team_size, "root must be smaller than group size");
112: 
113:   auto stream = at::cuda::getCurrentCUDAStream();
114:   rocshmem_broadcastmem_on_stream(team, buffer_ptr, buffer_ptr, buffer_size, root, stream);
115:   return input;
116: }
117: 
118: void nvshmem_put(at::Tensor& tensor, const int64_t peer) {
119:   // TODO: support non-contiguous tensors
120:   TORCH_CHECK(tensor.is_contiguous(),
```

- EN: Lines 101-120 introduces executable logic in routines such as `nvshmem_broadcast`, `TORCH_CHECK`, `nvshmem_put`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 101-120 行在 `nvshmem_broadcast`、`TORCH_CHECK`、`nvshmem_put` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 121-140 / 第 121-140 行

```cpp
121:       "put op currently supports contiguous tensors only");
122:   // TODO: rendezvous should remember the group name
123:   auto hdl = c10d::symmetric_memory::rendezvous(tensor, "0");
124:   auto rank = hdl->get_rank();
125:   void* buffer_ptr = hdl->get_buffer_ptrs()[rank];
126:   auto buffer_size = tensor.numel() * tensor.element_size();
127:   TORCH_CHECK(peer < hdl->get_world_size(), "peer must be smaller than world size");
128: 
129:   c10::cuda::CUDAGuard guard(tensor.device());
130:   auto stream = at::cuda::getCurrentCUDAStream();
131:   rocshmem_putmem_on_stream(buffer_ptr, tensor.data_ptr(), buffer_size, peer, stream);
132: }
133: 
134: void nvshmem_wait_for_signal(at::Tensor& sigpad, int64_t signal, int64_t peer) {
135:   c10::cuda::CUDAGuard guard(sigpad.device());
136:   auto stream = at::cuda::getCurrentCUDAStream();
137:   rocshmem_signal_wait_until_on_stream(static_cast<uint64_t*>(sigpad.data_ptr()), ROCSHMEM_CMP_EQ, signal, stream);
138: }
139: 
140: void nvshmem_put_with_signal(at::Tensor& tensor, at::Tensor& sigpad, int64_t signal, int64_t peer) {
```

- EN: Lines 121-140 introduces executable logic in routines such as `nvshmem_wait_for_signal`, `nvshmem_put_with_signal`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 121-140 行在 `nvshmem_wait_for_signal`、`nvshmem_put_with_signal` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 141-160 / 第 141-160 行

```cpp
141:   auto buffer_size = tensor.numel() * tensor.element_size();
142: 
143:   c10::cuda::CUDAGuard guard(tensor.device());
144:   auto stream = at::cuda::getCurrentCUDAStream();
145:   rocshmem_putmem_signal_on_stream(
146:     tensor.mutable_data_ptr(),
147:     tensor.mutable_data_ptr(),
148:     buffer_size,
149:     static_cast<uint64_t*>(sigpad.mutable_data_ptr()),
150:     signal,
151:     ROCSHMEM_SIGNAL_SET,
152:     peer,
153:     stream);
154: }
155: 
156: void nvshmem_get(at::Tensor& tensor, const int64_t peer) {
157:   // TODO: support non-contiguous tensors
158:   TORCH_CHECK(tensor.is_contiguous(),
159:       "get op currently supports contiguous tensors only");
160:   // TODO: rendezvous should remember the group name
```

- EN: Lines 141-160 introduces executable logic in routines such as `nvshmem_get`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 141-160 行在 `nvshmem_get` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 161-180 / 第 161-180 行

```cpp
161:   auto hdl = c10d::symmetric_memory::rendezvous(tensor, "0");
162:   auto rank = hdl->get_rank();
163:   void* buffer_ptr = hdl->get_buffer_ptrs()[rank];
164:   auto buffer_size = tensor.numel() * tensor.element_size();
165:   TORCH_CHECK(peer < hdl->get_world_size(), "peer must be smaller than world size");
166: 
167:   c10::cuda::CUDAGuard guard(tensor.device());
168:   auto stream = at::cuda::getCurrentCUDAStream();
169:   rocshmem_getmem_on_stream(tensor.mutable_data_ptr(), buffer_ptr, buffer_size, peer, stream);
170: }
171: 
172: at::Tensor nvshmem_all_to_all(
173:     at::Tensor& input,
174:     at::Tensor& out,
175:     std::string group_name) {
176:   auto input_hdl = c10d::symmetric_memory::rendezvous(input, group_name);
177:   auto out_hdl = c10d::symmetric_memory::rendezvous(out, group_name);
178:   int rank = input_hdl->get_rank();
179:   int world_size = input_hdl->get_world_size();
180:   auto& team_manager = TeamManager::get(input.device());
```

- EN: Lines 161-180 introduces executable logic in routines such as `nvshmem_all_to_all`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 161-180 行在 `nvshmem_all_to_all` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 181-200 / 第 181-200 行

```cpp
181:   auto team = team_manager.get_team(group_name, input_hdl->get_rank_to_global_rank());
182: 
183:   void* input_ptr = input.data_ptr();
184:   void* output_ptr = out.mutable_data_ptr();
185:   TORCH_CHECK(input.is_contiguous() && out.is_contiguous());
186:   TORCH_CHECK_EQ(input.numel(), out.numel());
187:   TORCH_CHECK_EQ(input.dtype(), out.dtype());
188:   TORCH_CHECK_EQ(input.numel() % world_size, 0);
189:   auto buffer_size = input.numel() * input.element_size();
190:   size_t bytes_per_rank = buffer_size / world_size;
191: 
192:   auto stream = at::cuda::getCurrentCUDAStream(input.device().index());
193:   rocshmem_alltoallmem_on_stream(team, output_ptr, input_ptr, bytes_per_rank, stream);
194:   return out;
195: }
196: 
197: // This is an exclusive prefix sum function that calculates read (or write) offsets for each peer.
198: __device__ int64_t prefixSum(int64_t *odata, int64_t *idata, int n) {
199:   // Specialize BlockScan for a 1D block of threads, of type int64_t.
200:   // - `BLOCK_SCAN_WARP_SCANS` is a low-latency scan algorithm (instead of high
```

- EN: Lines 181-200 introduces executable logic in routines such as `prefixSum`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 181-200 行在 `prefixSum` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 201-220 / 第 201-220 行

```cpp
201:   // throughput which we don't need here).
202:   // - `at_cuda_detail::cub` is torch's cub wrapper, see #55292.
203:   using BlockScanT = ROCM_HIPCUB(at_cuda_detail::cub)::BlockScan<int64_t,
204:         THREADS_PER_BLOCK, ROCM_HIPCUB(at_cuda_detail::cub)::BLOCK_SCAN_WARP_SCANS>;
205:   // Allocate shared memory for BlockScan
206:   __shared__ typename BlockScanT::TempStorage temp_storage;
207: 
208:   // TODO: currently it is assumed that the number of PE's is smaller than
209:   // `THREADS_PER_BLOCK`
210:   CUDA_KERNEL_ASSERT(n <= THREADS_PER_BLOCK);
211: 
212:   // Obtain input item for each thread
213:   int tid = threadIdx.x;
214:   int64_t thread_data = (tid < n) ? idata[tid] : 0;
215: 
216:   // Collectively compute the block-wide exclusive prefix sum
217:   int64_t block_aggregate;
218:   BlockScanT(temp_storage).ExclusiveSum(thread_data, thread_data, block_aggregate);
219: 
220:   // Store the result
```

- EN: Lines 201-220 introduces executable logic in routines such as `CUDA_KERNEL_ASSERT`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 201-220 行在 `CUDA_KERNEL_ASSERT` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 221-240 / 第 221-240 行

```cpp
221:   odata[tid] = thread_data;
222:   return block_aggregate;
223: }
224: 
225: static int get_a2a_nblocks(size_t size, int world_size, bool intra_node) {
226:   // Check user setting first
227:   int num_blocks = c10d::symmetric_memory::getenv_nblocks();
228:   if (num_blocks > 0) {  // set by user
229:     return num_blocks;
230:   }
231:   // 16B per thread, 8 loops
232:   constexpr size_t chunk_size = 16 * THREADS_PER_BLOCK * 8;
233:   num_blocks = at::ceil_div(size, chunk_size);
234:   // Allow kernel to target even number of blocks per peer
235:   num_blocks = at::round_up(num_blocks, world_size);
236:   const int max_blocks = intra_node ? 64 : 16;
237:   return ::min(num_blocks, max_blocks);
238: }
239: 
240: // ROCm-only offset writeback kernel.
```

- EN: Lines 221-240 introduces executable logic in routines such as `get_a2a_nblocks`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 221-240 行在 `get_a2a_nblocks` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 241-260 / 第 241-260 行

```cpp
241: //
242: // On ROCm, allToAllV is a regular multi-block kernel with no grid-wide barrier.
243: // Writing source_offsets in-kernel can race with other blocks still reading it for
244: // remote gets. We therefore compute output offsets in a separate kernel after
245: // allToAllV has completed on the stream.
246: __global__ void writeOutputOffsets1d(int64_t* out_splits_offsets, int world_size) {
247:   auto output_splits = out_splits_offsets;
248:   auto output_offsets = out_splits_offsets + world_size;
249:   int tid = threadIdx.x;
250: 
251:   CUDA_KERNEL_ASSERT(world_size <= THREADS_PER_BLOCK);
252:   __shared__ int64_t peer_offsets[THREADS_PER_BLOCK];
253:   prefixSum(peer_offsets, output_splits, world_size);
254:   __syncthreads();
255: 
256:   if (tid < world_size) {
257:     output_offsets[tid] = peer_offsets[tid];
258:   }
259: }
260: // This kernel is used to exchange output splits and source offsets between peers.
```

- EN: Lines 241-260 introduces executable logic in routines such as `writeOutputOffsets1d`, `CUDA_KERNEL_ASSERT`, `__syncthreads`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 241-260 行在 `writeOutputOffsets1d`、`CUDA_KERNEL_ASSERT`、`__syncthreads` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 261-280 / 第 261-280 行

```cpp
261: // `in_out_splits` is of size (3, npes) and contains:
262: // - input splits (IN)
263: // - output splits (OUT) and
264: // - source offsets (OUT).
265: __global__ void exchangeSplitAndOffset(int64_t* input_splits, int64_t* out_splits_offsets, rocshmem_team_t team) {
266:   CUDA_KERNEL_ASSERT(team != ROCSHMEM_TEAM_INVALID);
267:   int mype = rocshmem_team_my_pe(team);
268:   int npes = rocshmem_team_n_pes(team);
269:   auto output_splits = out_splits_offsets;
270:   auto source_offsets = out_splits_offsets + npes;
271:   int tid = threadIdx.x;
272: 
273:   CUDA_KERNEL_ASSERT(npes <= THREADS_PER_BLOCK);
274:   __shared__ int64_t peer_offsets[THREADS_PER_BLOCK];
275: 
276:   // Scan input splits to get the source offsets
277:   prefixSum(peer_offsets, input_splits, npes);
278:   __syncthreads();;
279: 
280:   // Use 1 block to do the exchange
```

- EN: Lines 261-280 introduces executable logic in routines such as `exchangeSplitAndOffset`, `CUDA_KERNEL_ASSERT`, `__syncthreads`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 261-280 行在 `exchangeSplitAndOffset`、`CUDA_KERNEL_ASSERT`、`__syncthreads` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 281-300 / 第 281-300 行

```cpp
281:   if (tid < npes) {
282:     // tid is peer index within team, but put calls require global rank
283:     int peer_global = rocshmem_team_translate_pe(team, tid, ROCSHMEM_TEAM_WORLD);
284:     rocshmem_int64_p(source_offsets + mype, peer_offsets[tid], peer_global);
285:     rocshmem_int64_p(output_splits + mype, input_splits[tid], peer_global);
286:   }
287:   rocshmem_barrier_wg();
288: }
289: 
290: // This kernel is used to do the actual data exchange.
291: // `in_out_splits` has the same definition as in `exchangeSplitAndOffset`.
292: // `stride` is the stride at dim 0, unit in byte.
293: __global__ void allToAllV(void *send_data, void *recv_data, int64_t* out_splits_offsets, size_t stride, rocshmem_team_t team) {
294:   CUDA_KERNEL_ASSERT(team != ROCSHMEM_TEAM_INVALID);
295:   int mype = rocshmem_team_my_pe(team);
296:   int npes = rocshmem_team_n_pes(team);
297:   auto output_splits = out_splits_offsets;
298:   auto source_offsets = out_splits_offsets + npes;
299:   int bid = blockIdx.x;
300:   int tid = threadIdx.x;
```

- EN: Lines 281-300 introduces executable logic in routines such as `rocshmem_int64_p`, `rocshmem_barrier_wg`, `allToAllV`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 281-300 行在 `rocshmem_int64_p`、`rocshmem_barrier_wg`、`allToAllV` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 301-320 / 第 301-320 行

```cpp
301:   int blocks_per_peer = max(gridDim.x / npes, 1);
302: 
303:   // Calculate the output offsets
304:   CUDA_KERNEL_ASSERT(npes <= THREADS_PER_BLOCK);
305:   __shared__ int64_t peer_offsets[THREADS_PER_BLOCK];
306:   prefixSum(peer_offsets, output_splits, npes);
307:   __syncthreads();
308: 
309:   // Target a different peer based on bid
310:   for (int i = bid / blocks_per_peer; i < npes; i += gridDim.x / blocks_per_peer) {
311:     int peer = (mype + i) % npes;
312:     auto peer_global = rocshmem_team_translate_pe(team, peer, ROCSHMEM_TEAM_WORLD);
313:     // Total amount from `peer`
314:     auto peer_size = output_splits[peer] * stride;
315:     // Amount to get from `peer` in this block
316:     auto block_size = peer_size / blocks_per_peer;
317:     // Being lazy here, we should handle the residual if the division is not exact
318:     CUDA_KERNEL_ASSERT(block_size * blocks_per_peer == peer_size);
319:     // This block's offset in the data from `peer`
320:     auto block_offset = block_size * (bid % blocks_per_peer);
```

- EN: Lines 301-320 introduces executable logic in routines such as `CUDA_KERNEL_ASSERT`, `__syncthreads`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 301-320 行在 `CUDA_KERNEL_ASSERT`、`__syncthreads` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 321-340 / 第 321-340 行

```cpp
321:     auto source_offset = source_offsets[peer] * stride + block_offset;
322:     auto write_offset = peer_offsets[peer] * stride + block_offset;
323:     rocshmem_getmem_nbi_wg(
324:       (char*)recv_data + write_offset,
325:       (char*)send_data + source_offset,
326:       block_size,
327:       peer_global);
328:   }
329:   rocshmem_quiet();
330: }
331: 
332: void all_to_all_vdev(
333:     at::Tensor& input,
334:     at::Tensor& out,
335:     at::Tensor& in_splits,
336:     at::Tensor& out_splits_offsets,
337:     std::string group_name) {
338:   /* Perform AllToAllv operation using NVSHMEM, with split information provided on device.
339:    * Step 1: Rendezvous tensors so all ranks have symmetric (device) pointers.
340:    * Step 2: Launch exchangeSplitAndOffset kernel to exchange per-rank split counts
```

- EN: Lines 321-340 introduces executable logic in routines such as `rocshmem_quiet`, `all_to_all_vdev`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 321-340 行在 `rocshmem_quiet`、`all_to_all_vdev` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 341-360 / 第 341-360 行

```cpp
341:    *         and compute source offsets (prefix sum); uses team barrier.
342:    * Step 3: Launch allToAllV kernel to copy data between peers according to
343:    *         the exchanged splits/offsets.
344:    * Arguments:
345:    *  - `input` is the send buffer
346:    *  - `out` is the receive buffer
347:    *  - `in_splits`: 1D[npes] num of elements this rank sends to each peer
348:    *  - `out_splits_offsets`:2D (2, npes). row0 = output splits, row1 = output offsets
349:    */
350:   auto input_hdl = c10d::symmetric_memory::rendezvous(input, group_name);
351:   auto out_hdl = c10d::symmetric_memory::rendezvous(out, group_name);
352:   auto in_splits_hdl = c10d::symmetric_memory::rendezvous(in_splits, group_name);
353:   auto out_splits_offsets_hdl = c10d::symmetric_memory::rendezvous(out_splits_offsets, group_name);
354:   int world_size = input_hdl->get_world_size();
355: 
356:   void* input_ptr = input.data_ptr();
357:   void* output_ptr = out.mutable_data_ptr();
358:   int64_t* in_splits_ptr = (int64_t*)(in_splits.const_data_ptr());
359:   int64_t* out_splits_offsets_ptr = (int64_t*)(out_splits_offsets.mutable_data_ptr());
360: 
```

- EN: Lines 341-360 introduces executable logic in routines such as `offsets`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 341-360 行在 `offsets` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 361-380 / 第 361-380 行

```cpp
361:   TORCH_CHECK_EQ(input.device(), out.device());
362:   auto device = input.device();
363:   c10::cuda::CUDAGuard guard(device);
364:   auto& team_manager = TeamManager::get(device);
365:   auto team = team_manager.get_team(group_name, input_hdl->get_rank_to_global_rank());
366:   auto stream = at::cuda::getCurrentCUDAStream(device.index());
367: 
368:   exchangeSplitAndOffset<<<dim3(1), dim3(THREADS_PER_BLOCK), 0, stream>>>(
369:       in_splits_ptr, out_splits_offsets_ptr, team);
370:   C10_CUDA_KERNEL_LAUNCH_CHECK();
371:   C10_CUDA_CHECK(hipStreamSynchronize(stream));
372:   rocshmem::rocshmem_barrier_all();
373:   // CTA Tuning
374:   auto input_size = input.numel() * input.element_size();
375:   int num_blocks = get_a2a_nblocks(
376:     input_size,
377:     input_hdl->get_world_size(),
378:     input_hdl->world_within_direct_access());
379: 
380:   // Stride at dim 0 (assuming input is contiguous, TODO)
```

- EN: Lines 361-380 introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 361-380 行在 `C10_CUDA_KERNEL_LAUNCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 381-400 / 第 381-400 行

```cpp
381:   size_t stride_bytes = input.stride(0) * input.element_size();
382: 
383:   allToAllV<<<dim3(num_blocks), dim3(THREADS_PER_BLOCK), 0, stream>>>(
384:       input_ptr, output_ptr, out_splits_offsets_ptr,
385:       stride_bytes, team);
386:   C10_CUDA_KERNEL_LAUNCH_CHECK();
387:   C10_CUDA_CHECK(hipStreamSynchronize(stream));
388:   // `allToAllV` reads source_offsets while fetching remote shards. Since ROCm has
389:   // no grid-wide sync here, writing output offsets in the same kernel can race
390:   // with those reads. Write output offsets in a follow-up kernel instead.
391:   writeOutputOffsets1d<<<dim3(1), dim3(THREADS_PER_BLOCK), 0, stream>>>(
392:       out_splits_offsets_ptr, world_size);
393:   C10_CUDA_KERNEL_LAUNCH_CHECK();
394: }
395: 
396: // Start of `all_to_all_vdev_2d`
397: 
398: // This is an warp-scope, exclusive prefix sum. When called by a block of
399: // threads, each warp will perform an independent prefix sum, concurrently.
400: // Returns the sum of all elements in the warp.
```

- EN: Lines 381-400 introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 381-400 行在 `C10_CUDA_KERNEL_LAUNCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 401-420 / 第 401-420 行

```cpp
401: // `NUM_WARPS` is the number of warps participating the concurrent prefix sum.
402: template <int NUM_WARPS>
403: __device__ int64_t prefixSum_warp(int64_t *odata, int64_t *idata, int n) {
404:   CUDA_KERNEL_ASSERT(n <= WARP_SIZE);
405: 
406:   // Specialize WarpScan for type int
407:   using WarpScan = ROCM_HIPCUB(at_cuda_detail::cub)::WarpScan<int64_t>;
408:   // Allocate WarpScan shared memory for N warps
409:   __shared__ typename WarpScan::TempStorage temp_storage[NUM_WARPS];
410: 
411:   int warp_id = threadIdx.x / WARP_SIZE;
412:   if (warp_id >= NUM_WARPS) {
413:     return 0;
414:   }
415: 
416:   // Obtain input item for each thread
417:   int tid = threadIdx.x % WARP_SIZE;
418:   int64_t thread_data = (tid < n) ? idata[tid] : 0;
419: 
420:   // Total sum of all elements in the warp
```

- EN: Lines 401-420 introduces executable logic in routines such as `prefixSum_warp`, `CUDA_KERNEL_ASSERT`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 401-420 行在 `prefixSum_warp`、`CUDA_KERNEL_ASSERT` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 421-440 / 第 421-440 行

```cpp
421:   int64_t warp_aggregate;
422:   // Compute the warp-wide exclusive prefix sum
423:   WarpScan(temp_storage[warp_id]).ExclusiveSum(thread_data, thread_data, warp_aggregate);
424: 
425:   // Store only valid lanes to avoid out-of-bounds writes when n < WARP_SIZE.
426:   if (tid < n) {
427:     odata[tid] = thread_data;
428:   }
429:   return warp_aggregate;
430: }
431: 
432: // This is for abstracting a thread-group-scope, exclusive prefix sum.
433: // Since we use warp-scope prefix sum, the thread group size is limited to warp size.
434: #define A2AV_TILE_SIZE WARP_SIZE
435: 
436: 
437: __global__ void writeOutputOffsets_2d(
438:     int64_t* out_splits_offsets,
439:     int minor_size,
440:     int major_size,
```

- EN: Lines 421-440 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 421-440 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 441-460 / 第 441-460 行

```cpp
441:     int64_t major_align) {
442:   int nsplits = minor_size * major_size;
443:   auto output_splits = out_splits_offsets;
444:   auto source_offsets = out_splits_offsets + nsplits;
445:   int tid = threadIdx.x;
446: 
447:   constexpr int NUM_TILES = THREADS_PER_BLOCK / A2AV_TILE_SIZE;
448:   int tileId = tid / A2AV_TILE_SIZE;
449:   int laneId = tid % A2AV_TILE_SIZE;
450:   __shared__ int64_t tile_prefix_sums[NUM_TILES][A2AV_TILE_SIZE];
451:   int nsplits_per_tile = min(minor_size, nsplits - tileId * minor_size);
452: 
453:   __shared__ int64_t len_per_tile[NUM_TILES];
454:   if (nsplits_per_tile > 0) {
455:     int64_t my_tile_len = prefixSum_warp<NUM_TILES>(tile_prefix_sums[tileId], output_splits + tileId * minor_size, nsplits_per_tile);
456:     if (laneId == A2AV_TILE_SIZE - 1) {
457:       if (major_align != 0) {
458:         auto aligned_len = (my_tile_len + major_align - 1) / major_align * major_align;
459:         len_per_tile[tileId] = max(aligned_len, major_align);
460:       } else {
```

- EN: Lines 441-460 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 441-460 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 461-480 / 第 461-480 行

```cpp
461:         len_per_tile[tileId] = my_tile_len;
462:       }
463:     }
464:   }
465:   __syncthreads();
466: 
467:   __shared__ int64_t start_offset_per_tile[WARP_SIZE];
468:   prefixSum_warp<1>(start_offset_per_tile, len_per_tile, NUM_TILES);
469:   __syncthreads();
470: 
471:   tile_prefix_sums[tileId][laneId] += start_offset_per_tile[tileId];
472:   __syncthreads();
473: 
474:   if (tid < nsplits) {
475:     source_offsets[tid] = tile_prefix_sums[tid / minor_size][tid % minor_size];
476:   }
477: }
478: 
479: // `exchangeSplitAndOffset_2d` is used to exchange output splits and source
480: // offsets between peers.
```

- EN: Lines 461-480 introduces executable logic in routines such as `__syncthreads`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 461-480 行在 `__syncthreads` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 481-500 / 第 481-500 行

```cpp
481: 
482: /* Arguments:
483:  * `in_splits_offsets`: input splits and offsets (optional), of size (2, nsplits), or (1, nsplits) if no offsets are provided.
484:  * `out_splits_offsets`: output splits and offsets, of size (2, nsplits).
485:  * `mype`: the rank of the current PE.
486:  * `npes`: the number of PEs.
487:  * `ne`: the number of experts.
488:  * `input_dim0`: the size of dim 0 of the input tensor.
489:  * `rank_is_row_in` is a boolean flag indicating whether the input has ranks as row or experts as row.
490: */
491: 
492: /* Template parameters:
493:  * `HAS_IN_OFFSETS` is a boolean flag indicating whether `in_splits_offsets` has offsets (2nd row) or not.
494: */
495: 
496: template <bool HAS_IN_OFFSETS>
497: __global__ void exchangeSplitAndOffset_2d(int64_t* in_splits_offsets, int64_t* out_splits_offsets, rocshmem_team_t team, int ne, size_t input_dim0, bool rank_is_row_in) {
498:   CUDA_KERNEL_ASSERT(team != ROCSHMEM_TEAM_INVALID);
499:   int mype = rocshmem_team_my_pe(team);
500:   int npes = rocshmem_team_n_pes(team);
```

- EN: Lines 481-500 introduces executable logic in routines such as `exchangeSplitAndOffset_2d`, `CUDA_KERNEL_ASSERT`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 481-500 行在 `exchangeSplitAndOffset_2d`、`CUDA_KERNEL_ASSERT` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 501-520 / 第 501-520 行

```cpp
501:   int nsplits = npes * ne;
502:   auto input_splits = in_splits_offsets;
503:   auto output_splits = out_splits_offsets;
504:   // Borrowing the space below as a temporary exchange pad.
505:   auto source_offsets = out_splits_offsets + nsplits;
506:   int tid = threadIdx.x;
507: 
508:   int64_t* input_offsets = nullptr;
509:   if (HAS_IN_OFFSETS) {
510:     // input offset are provided, so we can use them directly
511:     input_offsets = in_splits_offsets + nsplits;
512:   } else {
513:     // input offset are not provided, so we need to calculate them.
514:     // Scan input splits to get the source offsets
515:     __shared__ int64_t peer_offsets[THREADS_PER_BLOCK];
516:     auto sum_of_splits = prefixSum(peer_offsets, input_splits, nsplits);
517:     __syncthreads();;
518:     CUDA_KERNEL_ASSERT(sum_of_splits <= input_dim0 && "sum of splits is larger than input dim\n");
519:     // Redirect the input splits to the calculated result
520:     input_offsets = peer_offsets;
```

- EN: Lines 501-520 introduces executable logic in routines such as `__syncthreads`, `CUDA_KERNEL_ASSERT`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 501-520 行在 `__syncthreads`、`CUDA_KERNEL_ASSERT` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 521-540 / 第 521-540 行

```cpp
521:   }
522: 
523:   // Use 1 block to do the exchange
524:   if (tid < nsplits) {
525:     int peer, e, dst_offset;
526:     if (rank_is_row_in) {
527:       peer = tid / ne;
528:       e = tid % ne;
529:       dst_offset = e * npes + mype;
530:     } else {  // expert is row in input
531:       peer = tid % npes;
532:       e = tid / npes;
533:       dst_offset = mype * ne + e;
534:     }
535:     // This does a transpose from rank-major order to expert-major order
536:     // (or vice versa).
537:     auto split_val = input_splits[tid];
538:     CUDA_KERNEL_ASSERT(split_val >= 0 && "split value is negative\n");
539:     auto peer_global = rocshmem_team_translate_pe(team, peer, ROCSHMEM_TEAM_WORLD);
540:     rocshmem_int64_p(source_offsets + dst_offset, input_offsets[tid], peer_global);
```

- EN: Lines 521-540 introduces executable logic in routines such as `CUDA_KERNEL_ASSERT`, `rocshmem_int64_p`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 521-540 行在 `CUDA_KERNEL_ASSERT`、`rocshmem_int64_p` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 541-560 / 第 541-560 行

```cpp
541:     rocshmem_int64_p(output_splits + dst_offset, split_val, peer_global);
542:   }
543:   rocshmem_barrier_wg();
544: }
545: 
546: 
547: // This kernel is used to do the actual data exchange.
548: // `in_out_splits` has the same definition as in `exchangeSplitAndOffset`.
549: // `stride` is the stride at dim 0, unit in byte.
550: // For meaning of `mype` and `npes`, see the docstring of `all_to_all_vdev_2d`.
551: // `major_align` is the alignment at dim 0, unit in element. If 0, no alignment is needed.
552: 
553: // `rank_is_row_out` is a boolean flag indicating whether the output has ranks as rows or experts as rows.
554: // In dispatch case, rank_is_row_out = false, major_size = ne, minor_size = npes.
555: // In combine case, rank_is_row_out = true, major_size = npes, minor_size = ne.
556: 
557: __global__ void allToAllV_2d(void *send_data, void *recv_data, int64_t* in_splits, int64_t* out_splits_offsets, size_t stride, int minor_size, int major_size, int64_t major_align, bool rank_is_row_out, rocshmem_team_t team) {
558:   int nsplits = minor_size * major_size;
559:   auto output_splits = out_splits_offsets;
560:   auto source_offsets = out_splits_offsets + nsplits;
```

- EN: Lines 541-560 introduces executable logic in routines such as `rocshmem_barrier_wg`, `allToAllV_2d`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 541-560 行在 `rocshmem_barrier_wg`、`allToAllV_2d` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 561-580 / 第 561-580 行

```cpp
561:   int bid = blockIdx.x;
562:   int tid = threadIdx.x;
563: 
564:   // Split the thread block into tiles
565:   constexpr int NUM_TILES = THREADS_PER_BLOCK / A2AV_TILE_SIZE;
566:   int tileId = tid / A2AV_TILE_SIZE;
567:   int laneId = tid % A2AV_TILE_SIZE;
568:   // Each tile calculates its own prefix sum
569:   __shared__ int64_t tile_prefix_sums[NUM_TILES][A2AV_TILE_SIZE];
570:   // A tile takes care of minor_size worth of splits
571:   int nsplits_per_tile = min(minor_size, nsplits - tileId * minor_size);
572:   // TODO: currently it is assumed that the number of PE's is smaller than
573:   // `A2AV_TILE_SIZE` bc the warp-scope prefix sum can only handle up to
574:   // WARP_SIZE elements
575:   CUDA_KERNEL_ASSERT(minor_size <= A2AV_TILE_SIZE && "minor_size is too large\n");
576:   // Similarly, the number of experts per rank is also assumed to be smaller
577:   // than `NUM_TILES`
578:   CUDA_KERNEL_ASSERT(major_size <= NUM_TILES && "major_size is too large\n");
579: 
580:   // Total length of each tile
```

- EN: Lines 561-580 introduces executable logic in routines such as `CUDA_KERNEL_ASSERT`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 561-580 行在 `CUDA_KERNEL_ASSERT` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 581-600 / 第 581-600 行

```cpp
581:   __shared__ int64_t len_per_tile[NUM_TILES];
582:   // When `nsplits` is small, not every tile gets data to sum. They can skip
583:   // this local prefix sum.
584:   if (nsplits_per_tile > 0) {
585:     // Each tile calculates its own prefix sum, return value is the sum of all elements in the tile.
586:     int64_t my_tile_len = prefixSum_warp<NUM_TILES>(tile_prefix_sums[tileId], output_splits + tileId * minor_size, nsplits_per_tile);
587:     // Last thread in each tile does the up aligning.
588:     if (laneId == A2AV_TILE_SIZE - 1) {
589:       if (major_align != 0) {  // Needs alignment
590:         auto aligned_len = (my_tile_len + major_align - 1) / major_align * major_align;
591:         // In case `aligned_len` is 0, we set it to `major_align` to avoid an
592:         // empty bin, bc cutlass currently does not support it. See
593:         // https://github.com/pytorch/pytorch/issues/152668.
594:         len_per_tile[tileId] = max(aligned_len, major_align);
595:       } else {  // 0 means alignment not needed
596:         len_per_tile[tileId] = my_tile_len;
597:       }
598:     }
599:   }
600:   __syncthreads();
```

- EN: Lines 581-600 introduces executable logic in routines such as `__syncthreads`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 581-600 行在 `__syncthreads` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 601-620 / 第 601-620 行

```cpp
601: 
602:   // Starting offset of each tile
603:   __shared__ int64_t start_offset_per_tile[NUM_TILES];
604:   // Prefix sum again to get the tiles' start offsets.
605:   // `NUM_TILES` is typically not greater than 32, because 32 tiles * 32 threads
606:   // = 1024 threads, and this kernel is launched within 1024 threads. Thus, we
607:   // can use warp-scope prefix sum.
608:   static_assert(NUM_TILES <= WARP_SIZE);
609:   // Only 1 warp is needed
610:   prefixSum_warp<1>(start_offset_per_tile, len_per_tile, NUM_TILES);
611:   __syncthreads();
612: 
613:   // Add tile offset to every element in the tile
614:   tile_prefix_sums[tileId][laneId] += start_offset_per_tile[tileId];
615:   __syncthreads();
616: 
617:   // Target a different e based on bid
618:   for (int eid = bid; eid < nsplits; eid += gridDim.x) {
619:     int row = eid / minor_size;
620:     int col = eid % minor_size;
```

- EN: Lines 601-620 introduces executable logic in routines such as `static_assert`, `__syncthreads`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 601-620 行在 `static_assert`、`__syncthreads` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 621-640 / 第 621-640 行

```cpp
621:     // Amount from `peer` for `e`
622:     auto peer_size = output_splits[eid] * stride;
623:     auto source_offset = source_offsets[eid] * stride;
624:     auto e_offset = tile_prefix_sums[row][col];
625:     auto write_offset = e_offset * stride;
626:     auto peer_global = rocshmem_team_translate_pe(team, rank_is_row_out ? row : col, ROCSHMEM_TEAM_WORLD);
627:     rocshmem_getmem_nbi_wg(
628:       (char*)recv_data + write_offset,
629:       (char*)send_data + source_offset,
630:       peer_size,
631:       peer_global);  // peer's global index
632:   }
633:   rocshmem_quiet();
634: }
635: 
636: void all_to_all_vdev_2d(
637:     at::Tensor& input,
638:     at::Tensor& out,
639:     at::Tensor& in_splits,
640:     at::Tensor& out_splits_offsets,
```

- EN: Lines 621-640 introduces executable logic in routines such as `rocshmem_quiet`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 621-640 行在 `rocshmem_quiet` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 641-660 / 第 641-660 行

```cpp
641:     std::string group_name,
642:     std::optional<int64_t> major_align) {
643:   /* Perform a 2D AllToAllv shuffle operation using NVSHMEM, with split information provided on device.
644:    * Arguments:
645:    *  - `input` is the input tensor
646:    *  - `out` is the output tensor
647:    *  - `in_out_splits` is a 2D tensor of size (3, `world_size` * `ne`). In the
648:         scenario of Mixture-of-Experts models, `ne` is the number of experts per
649:         rank. The rows of `in_out_splits` are (in order):
650:         input splits (IN)
651:         output splits (OUT) and
652:         output offsets (OUT).
653:    *  - `group_name` is the name of the group to use for the collective operation.
654:    *  - `major_align` is the alignment of the "major dimension" of the output
655:         sequence. See below for details.
656: 
657:    *  A 2D AllToAllv shuffle is illustrated below:
658:         (world_size = 2, ne = 2, total number of experts = 4)
659:         Source: |       Rank 0      |       Rank 1      |
660:                 | c0 | c1 | c2 | c3 | d0 | d1 | d2 | d3 |
```

- EN: Lines 641-660 contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 641-660 行包含面向 CUDA 的声明、内核或启动流程。

### Lines 661-680 / 第 661-680 行

```cpp
661: 
662:         Dest  : |       Rank 0      |       Rank 1      |
663:                 | c0 | d0 | c1 | d1 | c2 | d2 | c3 | d3 |
664:         where each `c_i` / `d_i` are slices of the `input` tensor, targeting
665:         expert `i`, with length indicated by input splits (in
666:         `in_out_splits[0]`).  That is, the 2D AllToAllv shuffle achieves a
667:         transpose from rank-major order at input to expert-major order at
668:         output.
669: 
670:    *  If `major_align` is not 1, the output offsets of c1, c2, c3 will be
671:       up-aligned to this value. For example, if c0 has length 5 and d0 has
672:       length 7 (making a total of 12), and if the `major_align` is set to 16,
673:       the output offset of c1 will be 16. Similar for c2 and c3. This value has
674:       no effect on the offset of the minor dimension, i.e.  d0, d1, d2 and d3.
675:       Note: since cutlass does not support empty bins, we set the aligned length
676:       to `major_align` if it is 0. See
677:       https://github.com/pytorch/pytorch/issues/152668.
678:   */
679:   auto input_hdl = c10d::symmetric_memory::rendezvous(input, group_name);
680:   auto out_hdl = c10d::symmetric_memory::rendezvous(out, group_name);
```

- EN: Lines 661-680 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 661-680 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 681-700 / 第 681-700 行

```cpp
681:   auto in_splits_hdl = c10d::symmetric_memory::rendezvous(in_splits, group_name);
682:   auto out_splits_offsets_hdl = c10d::symmetric_memory::rendezvous(out_splits_offsets, group_name);
683:   int world_size = input_hdl->get_world_size();
684:   // TODO: world_size is currently limited by the number of elements in a WarpScan.
685:   TORCH_CHECK(world_size <= A2AV_TILE_SIZE, "world_size must be smaller than A2AV_TILE_SIZE", A2AV_TILE_SIZE);
686: 
687:   // If `major_align` is not provided, use 1 as the default value.
688:   int64_t major_align_val = major_align.value_or(1);
689:   TORCH_CHECK(major_align_val > 0, "major_align must be positive");
690: 
691:   void* input_ptr = input.data_ptr();
692:   void* output_ptr = out.mutable_data_ptr();
693:   int64_t* in_splits_ptr = (int64_t*)(in_splits.data_ptr());
694:   int64_t* out_splits_offsets_ptr = (int64_t*)(out_splits_offsets.mutable_data_ptr());
695: 
696:   // Shape checks
697:   TORCH_CHECK(in_splits.is_contiguous()
698:       && out_splits_offsets.is_contiguous()
699:       && input.is_contiguous()
700:       && out.is_contiguous(),
```

- EN: Lines 681-700 introduces executable logic in routines such as `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 681-700 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 701-720 / 第 701-720 行

```cpp
701:       "input, out, in_splits and out_splits_offsets must be contiguous");
702:   auto in_split_shape = in_splits.sizes();
703:   auto out_split_shape = out_splits_offsets.sizes();
704:   TORCH_CHECK(out_split_shape.size() == 2
705:       && out_split_shape[0] == 2
706:       && out_split_shape[1] == in_split_shape[0]
707:       && in_split_shape[0] % world_size == 0,
708:       "out_splits_offsets must be 2D with 2 rows, "
709:       "each row must be a multiple of world_size");
710: 
711:   // Consistency checks
712:   TORCH_CHECK(input.dtype() == out.dtype()
713:       && input.stride(0) == out.stride(0),
714:       "input and out must have the same dtype and same stride at dim 0");
715:   TORCH_CHECK(in_splits.scalar_type() == at::kLong
716:       && out_splits_offsets.scalar_type() == at::kLong,
717:       "splits and offsets must be int64");
718: 
719:   // Number of experts per rank
720:   int ne = in_split_shape[0] / world_size;
```

- EN: Lines 701-720 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 701-720 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 721-740 / 第 721-740 行

```cpp
721:   constexpr int NUM_TILES = THREADS_PER_BLOCK / A2AV_TILE_SIZE;
722:   TORCH_CHECK(ne <= NUM_TILES, "Number of experts must be smaller than NUM_TILES", NUM_TILES);
723: 
724:   // Set device context for getting the stream and launching kernels below
725:   auto device = input.device();
726:   TORCH_CHECK(device.type() == at::DeviceType::CUDA &&
727:       out.device() == device &&
728:       in_splits.device() == device &&
729:       out_splits_offsets.device() == device,
730:       "all tensor arguments must be on the same CUDA device");
731:   c10::cuda::CUDAGuard guard(device);
732:   auto stream = at::cuda::getCurrentCUDAStream();
733:   auto& team_manager = TeamManager::get(device);
734:   auto team = team_manager.get_team(group_name, input_hdl->get_rank_to_global_rank());
735: 
736:   // Exchange output splits and source offsets
737:   auto input_dim0 = input.size(0);
738:   bool rank_is_row_in = true;
739:   exchangeSplitAndOffset_2d<false><<<dim3(1), dim3(THREADS_PER_BLOCK), 0, stream>>>(
740:       in_splits_ptr, out_splits_offsets_ptr, team,
```

- EN: Lines 721-740 introduces executable logic in routines such as `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 721-740 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 741-760 / 第 741-760 行

```cpp
741:       ne, input_dim0, rank_is_row_in);
742:   C10_CUDA_KERNEL_LAUNCH_CHECK();
743:   C10_CUDA_CHECK(hipStreamSynchronize(stream));
744:   rocshmem::rocshmem_barrier_all();
745:   // CTA Tuning
746:   // Naive for now, use 1 block per expert.
747:   // Total number of blocks is limited to 64 (intra-node) or 8 (inter-node).
748:   int num_blocks = ::min(world_size * ne, world_size > 8 ? 8 : 64);
749: 
750:   // Stride at dim 0
751:   size_t stride_bytes = input.stride(0) * input.element_size();
752:   bool rank_is_row_out = !rank_is_row_in;
753: 
754:   allToAllV_2d<<<dim3(num_blocks), dim3(THREADS_PER_BLOCK), 0, stream>>>(
755:       input_ptr, output_ptr,
756:       in_splits_ptr, out_splits_offsets_ptr,
757:       stride_bytes, world_size,
758:       ne, major_align_val, rank_is_row_out, team);
759:   C10_CUDA_KERNEL_LAUNCH_CHECK();
760:   // allToAllV_2d uses a regular multi-block launch with no grid-wide sync, so
```

- EN: Lines 741-760 introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 741-760 行在 `C10_CUDA_KERNEL_LAUNCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 761-780 / 第 761-780 行

```cpp
761:   // in-kernel writeback can race with other blocks still reading source_offsets.
762:   writeOutputOffsets_2d<<<dim3(1), dim3(THREADS_PER_BLOCK), 0, stream>>>(
763:       out_splits_offsets_ptr, world_size, ne, major_align_val);
764:   C10_CUDA_KERNEL_LAUNCH_CHECK();
765:   C10_CUDA_CHECK(hipStreamSynchronize(stream));
766: }
767: 
768: void all_to_all_vdev_2d_offset(
769:     at::Tensor& input,
770:     at::Tensor& out,
771:     at::Tensor& in_splits_offsets,
772:     at::Tensor& out_splits_offsets,
773:     std::string group_name) {
774:   /* Perform a 2D AllToAllv shuffle operation, with input split and offset
775:    * information provided on device. The input offsets are not required to be
776:    * exact prefix sum of the input splits, i.e. paddings are allowed between the
777:    * split chunks. The paddings, however, will not be transferred to peer
778:    * ranks.
779: 
780:    * In Mixture of Experts models, this operation can be used to combine tokens
```

- EN: Lines 761-780 introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`, `all_to_all_vdev_2d_offset`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 761-780 行在 `C10_CUDA_KERNEL_LAUNCH_CHECK`、`all_to_all_vdev_2d_offset` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 781-800 / 第 781-800 行

```cpp
781:    * processed by experts on parallel ranks. This operation can be viewed as an
782:    * "reverse" operation to the `all_to_all_vdev_2d` operation (which shuffles
783:    * tokens to experts).
784: 
785:    * Arguments:
786:    *  - `input` is the input tensor
787:    *  - `out` is the output tensor
788:    *  - `in_splits_offsets` is a 2D tensor of size (2, `ne` * `world_size`). In the
789:         scenario of Mixture-of-Experts models, `ne` is the number of experts per
790:         rank. The rows of `in_splits_offsets` are (in order):
791:         input splits (IN) and
792:         input offsets (IN)
793:    *  - `out_splits_offsets` is a 2D tensor of size (2, `world_size` * `ne`). The
794:         rows are (in order):
795:         output splits (OUT) and
796:         output offsets (OUT).
797:    *  - `group_name` is the name of the group to use for the collective operation.
798:   */
799:   auto input_hdl = c10d::symmetric_memory::rendezvous(input, group_name);
800:   auto out_hdl = c10d::symmetric_memory::rendezvous(out, group_name);
```

- EN: Lines 781-800 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 781-800 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 801-820 / 第 801-820 行

```cpp
801:   auto out_splits_offsets_hdl = c10d::symmetric_memory::rendezvous(out_splits_offsets, group_name);
802:   auto in_splits_offsets_hdl = c10d::symmetric_memory::rendezvous(in_splits_offsets, group_name);
803:   int rank = input_hdl->get_rank();
804:   int world_size = input_hdl->get_world_size();
805:   constexpr int NUM_TILES = THREADS_PER_BLOCK / A2AV_TILE_SIZE;
806:   TORCH_CHECK(world_size <= NUM_TILES, "world_size must be smaller than NUM_TILES", NUM_TILES);
807: 
808:   int64_t major_align_val = 0;
809: 
810:   void* input_ptr = input.data_ptr();
811:   void* output_ptr = out.mutable_data_ptr();
812:   int64_t* out_splits_offsets_ptr = (int64_t*)(out_splits_offsets.mutable_data_ptr());
813:   int64_t* in_splits_offsets_ptr = (int64_t*)(in_splits_offsets.data_ptr());
814: 
815:   // Shape checks
816:   TORCH_CHECK(out_splits_offsets.is_contiguous()
817:       && in_splits_offsets.is_contiguous()
818:       && input.is_contiguous()
819:       && out.is_contiguous(),
820:       "input, out, in_splits_offsets and out_splits_offsets must be contiguous");
```

- EN: Lines 801-820 introduces executable logic in routines such as `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 801-820 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 821-840 / 第 821-840 行

```cpp
821:   auto out_split_shape = out_splits_offsets.sizes();
822:   auto in_split_shape = in_splits_offsets.sizes();
823:   TORCH_CHECK(in_split_shape.size() == 2
824:       && in_split_shape[0] == 2
825:       && in_split_shape[1] % world_size == 0,
826:       "in_splits_offsets must be 2D with 2 rows, "
827:       "each row must be a multiple of world_size");
828: 
829:   // Consistency checks
830:   TORCH_CHECK(input.dtype() == out.dtype()
831:       && input.stride(0) == out.stride(0),
832:       "input and out must have the same dtype and same stride at dim 0");
833:   TORCH_CHECK(out_splits_offsets.scalar_type() == at::kLong
834:       && in_splits_offsets.scalar_type() == at::kLong,
835:       "splits and offsets must be int64");
836: 
837:   // Number of experts per rank
838:   int ne = in_split_shape[1] / world_size;
839:   // TODO: number of experts is currently limited by the number of elements in a WarpScan.
840:   TORCH_CHECK(ne <= A2AV_TILE_SIZE, "Number of experts must be smaller than A2AV_TILE_SIZE", A2AV_TILE_SIZE);
```

- EN: Lines 821-840 introduces executable logic in routines such as `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 821-840 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 841-860 / 第 841-860 行

```cpp
841: 
842:   // Set device context for getting the stream and launching kernels below
843:   auto device = input.device();
844:   TORCH_CHECK(device.type() == at::DeviceType::CUDA &&
845:       out.device() == device &&
846:       in_splits_offsets.device() == device &&
847:       out_splits_offsets.device() == device,
848:       "all tensor arguments must be on the same CUDA device");
849:   c10::cuda::CUDAGuard guard(device);
850:   auto stream = at::cuda::getCurrentCUDAStream();
851:   auto& team_manager = TeamManager::get(device);
852:   auto team = team_manager.get_team(group_name, input_hdl->get_rank_to_global_rank());
853: 
854:   // Exchange output splits and source offsets
855:   auto input_dim0 = input.size(0);
856:   bool rank_is_row_in = false;
857:   exchangeSplitAndOffset_2d<true><<<dim3(1), dim3(THREADS_PER_BLOCK), 0, stream>>>(
858:       in_splits_offsets_ptr,
859:       out_splits_offsets_ptr,
860:       team,
```

- EN: Lines 841-860 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 841-860 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 861-880 / 第 861-880 行

```cpp
861:       ne, input_dim0, rank_is_row_in);
862:   C10_CUDA_KERNEL_LAUNCH_CHECK();
863:   C10_CUDA_CHECK(hipStreamSynchronize(stream));
864:   rocshmem::rocshmem_barrier_all();
865:   // CTA Tuning
866:   // Naive for now, use 1 block per expert.
867:   // Total number of blocks is limited to 64 (intra-node) or 8 (inter-node).
868:   int num_blocks = ::min(world_size * ne, world_size > 8 ? 8 : 64);
869: 
870:   // Stride at dim 0
871:   size_t stride_bytes = input.stride(0) * input.element_size();
872:   bool rank_is_row_out = !rank_is_row_in;
873: 
874:   allToAllV_2d<<<dim3(num_blocks), dim3(THREADS_PER_BLOCK), 0, stream>>>(
875:       input_ptr,
876:       output_ptr,
877:       in_splits_offsets_ptr,
878:       out_splits_offsets_ptr,
879:       stride_bytes,
880:       ne,
```

- EN: Lines 861-880 introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 861-880 行在 `C10_CUDA_KERNEL_LAUNCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 881-900 / 第 881-900 行

```cpp
881:       world_size,
882:       major_align_val,
883:       rank_is_row_out,
884:       team);
885:   C10_CUDA_KERNEL_LAUNCH_CHECK();
886:   writeOutputOffsets_2d<<<dim3(1), dim3(THREADS_PER_BLOCK), 0, stream>>>(
887:       out_splits_offsets_ptr, ne, world_size, major_align_val);
888:   C10_CUDA_KERNEL_LAUNCH_CHECK();
889:   C10_CUDA_CHECK(hipStreamSynchronize(stream));
890: }
891: 
892: } // namespace c10d::nvshmem_extension
893: 
894: TORCH_LIBRARY_IMPL(symm_mem, CUDA, m) {
895:   m.impl("nvshmem_broadcast", c10d::nvshmem_extension::nvshmem_broadcast);
896:   m.impl("nvshmem_put", c10d::nvshmem_extension::nvshmem_put);
897:   m.impl("nvshmem_get", c10d::nvshmem_extension::nvshmem_get);
898:   m.impl("nvshmem_wait_for_signal", c10d::nvshmem_extension::nvshmem_wait_for_signal);
899:   m.impl("nvshmem_put_with_signal", c10d::nvshmem_extension::nvshmem_put_with_signal);
900:   m.impl("nvshmem_all_to_all", c10d::nvshmem_extension::nvshmem_all_to_all);
```

- EN: Lines 881-900 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 881-900 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `C10_CUDA_KERNEL_LAUNCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 901-904 / 第 901-904 行

```cpp
901:   m.impl("all_to_all_vdev", c10d::nvshmem_extension::all_to_all_vdev);
902:   m.impl("all_to_all_vdev_2d", c10d::nvshmem_extension::all_to_all_vdev_2d);
903:   m.impl("all_to_all_vdev_2d_offset", c10d::nvshmem_extension::all_to_all_vdev_2d_offset);
904: }
```

- EN: Lines 901-904 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 901-904 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

## Key Concepts / 关键概念

- EN: Subsystem: c10d symmetric-memory support.
- CN: 子系统：c10d 对称内存支持。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `parse_rocshmem_version_ge`, `is_nvshmem_available`, `nvshmemx_cumodule_init`, `nvshmem_broadcast`, `TORCH_CHECK`, `nvshmem_put`
- CN: 核心符号：`parse_rocshmem_version_ge`、`is_nvshmem_available`、`nvshmemx_cumodule_init`、`nvshmem_broadcast`、`TORCH_CHECK`、`nvshmem_put`
- EN: Notable themes: CUDA paths, store/state coordination, collective communication logic.
- CN: 值得关注的主题：CUDA 路径、存储/状态协调、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/symm_mem/env.hpp`, `torch/csrc/distributed/c10d/symm_mem/nvshmem_extension.hpp`, `torch/csrc/distributed/c10d/symm_mem/nvshmem_team_manager.hpp`, `torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemory-inl.cuh`, `torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryUtils.hpp`, `torch/csrc/distributed/c10d/symm_mem/SymmetricMemory.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/ceil_div.h`, `c10/hip/HIPGuard.h`, `ATen/hip/cub.cuh`, `c10/hip/HIPException.h`
- External or system headers / 外部或系统头文件: `hip/hip_runtime.h`, `algorithm`, `cstdlib`, `vector`, `rocshmem/rocshmem.hpp`
- Local symbols / 本地符号: `parse_rocshmem_version_ge`, `is_nvshmem_available`, `nvshmemx_cumodule_init`, `nvshmem_broadcast`, `TORCH_CHECK`, `nvshmem_put`, `nvshmem_wait_for_signal`, `nvshmem_put_with_signal`