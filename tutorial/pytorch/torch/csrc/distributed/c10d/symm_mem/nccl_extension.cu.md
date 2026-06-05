# nccl_extension.cu — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/symm_mem/nccl_extension.cu`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides implementation logic for nccl extension in the c10d symmetric-memory support. Representative routines include `get_remote_ptr`, `copy_bytes_vec16`, `lsa_put_kernel`, `__syncthreads`, `__threadfence_system`. GPU-oriented code paths are present in this file.
- 用途 (CN): 该文件在c10d 对称内存支持中提供nccl extension 的实现逻辑。 代表性例程包括 `get_remote_ptr`、`copy_bytes_vec16`、`lsa_put_kernel`、`__syncthreads`、`__threadfence_system`。 该文件还包含面向 GPU 的代码路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
1: #include <c10/cuda/CUDAGuard.h>
2: #include <ATen/native/cuda/MemoryAccess.cuh>
3: #include <torch/csrc/distributed/c10d/NCCLUtils.hpp>
4: #include <torch/csrc/distributed/c10d/symm_mem/nccl_dev_cap.hpp>
5: #include <torch/csrc/distributed/c10d/symm_mem/nccl_extension.hpp>
6: #include <torch/csrc/distributed/c10d/symm_mem/nccl_devcomm_manager.hpp>
7: #include <torch/csrc/distributed/c10d/symm_mem/NCCLSymmetricMemory.hpp>
8: 
9: namespace c10d::nccl_extension {
10: 
11: using namespace c10d::symmetric_memory;
12: 
13: #define THREADS_PER_BLOCK 512
14: 
15: #ifdef NCCL_HAS_SYMMEM_SUPPORT
16: __device__ __forceinline__ char* get_remote_ptr(
```

- EN: Lines 1-16 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 1-16 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 17-32 / 第 17-32 行

```cpp
17:     void** buffer,  // buffers_dev_
18:     int peer,  // peer index
19:     size_t byte_offset  // buffer byte offset, default 0
20: ) {
21:     char* base = reinterpret_cast<char*>(buffer[peer]);
22:     return base + byte_offset;
23: }
24: 
25: __device__ inline void copy_bytes_vec16(
26:     const char* src_base,
27:     char* dst_base,
28:     size_t nbytes,
29:     size_t tid,
30:     size_t stride)
31: {
32:     if (nbytes == 0) return;
```

- EN: Lines 17-32 introduces executable logic in routines such as `copy_bytes_vec16`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 17-32 行在 `copy_bytes_vec16` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 33-48 / 第 33-48 行

```cpp
33: 
34:     uintptr_t src_addr = reinterpret_cast<uintptr_t>(src_base);
35:     uintptr_t dst_addr = reinterpret_cast<uintptr_t>(dst_base);
36: 
37:     // head: try to align both to 16B
38:     // We can only align both with a small head copy if they share the
39:     // same offset modulo 16.
40:     size_t head = 0;
41:     if ((src_addr & 0xF) == (dst_addr & 0xF)) {
42:         size_t misalign = src_addr & 0xF;
43:         if (misalign != 0) {
44:             size_t to_align = min(nbytes, 16 - misalign);
45:             if (tid == 0) {
46:                 for (size_t i = 0; i < to_align; ++i) {
47:                     dst_base[i] = src_base[i];
48:                 }
```

- EN: Lines 33-48 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 33-48 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 49-64 / 第 49-64 行

```cpp
49:             }
50:             head = to_align;
51:             src_addr += head;
52:             dst_addr += head;
53:             src_base += head;
54:             dst_base += head;
55:             nbytes -= head;
56:         }
57:     }
58:     if (nbytes == 0) return;
59: 
60:     // If either pointer is still not 16B aligned, we *must not* issue
61:     // 16B vector loads/stores. Fall back to scalar grid-stride copy.
62:     if ((src_addr & 0xF) != 0 || (dst_addr & 0xF) != 0) {
63:         for (size_t i = tid; i < nbytes; i += stride) {
64:             dst_base[i] = src_base[i];
```

- EN: Lines 49-64 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 49-64 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 65-80 / 第 65-80 行

```cpp
65:         }
66:         return;
67:     }
68: 
69:     // middle: 16B vectorized copy
70:     size_t n_vec = nbytes / 16;
71: 
72:     for (size_t vec_idx = tid; vec_idx < n_vec; vec_idx += stride) {
73:         const char* src_ptr = src_base + vec_idx * 16;
74:         char* dst_ptr = dst_base + vec_idx * 16;
75:         auto v = at::native::memory::ld_vec<16>(src_ptr);   // load 16 bytes
76:         at::native::memory::st_vec<16>(dst_ptr, v);         // store 16 bytes
77:     }
78: 
79:     // tail: leftover bytes (< 16)
80:     size_t copied = n_vec * 16;
```

- EN: Lines 65-80 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 65-80 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 81-96 / 第 81-96 行

```cpp
81:     size_t tail   = nbytes - copied;
82:     for (size_t i = tid; i < tail; i += stride) {
83:         dst_base[copied + i] = src_base[copied + i];
84:     }
85: }
86: 
87: __global__ void lsa_put_kernel(
88:     void** buffer,  // buffers_dev_
89:     int dst_peer,
90:     size_t dst_byte_offset,
91:     const void* src,
92:     size_t nbytes
93: ) {
94:     // Calculate index
95:     const size_t tid    = blockIdx.x * blockDim.x + threadIdx.x;
96:     const size_t stride = blockDim.x * gridDim.x;
```

- EN: Lines 81-96 introduces executable logic in routines such as `lsa_put_kernel`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 81-96 行在 `lsa_put_kernel` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 97-112 / 第 97-112 行

```cpp
97: 
98:     // Calculate remote dst pointer
99:     auto dst = get_remote_ptr(buffer, dst_peer, dst_byte_offset);
100:     const char* src_bytes = reinterpret_cast<const char*>(src);
101:     copy_bytes_vec16(src_bytes, dst, nbytes, tid, stride);
102: }
103: 
104: __global__ void lsa_put_signal_kernel(
105:     void**  buffer,  // buffers_dev_
106:     void**  signal_pad,  // signal pointer table (uint64_t-based)
107:     int  dst_peer,
108:     size_t  dst_byte_offset,  // data target offset (bytes)
109:     const void*  src,  // local src
110:     size_t  nbytes,
111:     unsigned int* blocks_done,  // global counter of blocks done
112:     uint64_t  signal_value     // value to write
```

- EN: Lines 97-112 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 97-112 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 113-128 / 第 113-128 行

```cpp
113: ) {
114:     const size_t tid    = blockIdx.x * blockDim.x + threadIdx.x;
115:     const size_t stride = blockDim.x * gridDim.x;
116: 
117:     // 1) data copy without signal set
118:     auto dst = get_remote_ptr(buffer, dst_peer, dst_byte_offset);
119:     const char* src_bytes = reinterpret_cast<const char*>(src);
120:     copy_bytes_vec16(src_bytes, dst, nbytes, tid, stride);
121: 
122:     __syncthreads();
123:     // Ensure all global writes from all SMs are visible system-wide
124:     __threadfence_system();
125: 
126:     // 2) system fence + signal set
127:     if (threadIdx.x == 0) {
128:         // This block is done; increment global completion counter
```

- EN: Lines 113-128 introduces executable logic in routines such as `__syncthreads`, `__threadfence_system`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 113-128 行在 `__syncthreads`、`__threadfence_system` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 129-144 / 第 129-144 行

```cpp
129:         unsigned int prev = atomicAdd(blocks_done, 1);
130: 
131:         // If this was the last block to finish:
132:         if (prev == gridDim.x - 1) {
133:             uint64_t* signal_pad_peer =
134:             reinterpret_cast<uint64_t*>(signal_pad[dst_peer]);
135: 
136:             // Single-writer: atomicExch is conservative but safe.
137:             atomicExch(
138:                 reinterpret_cast<unsigned long long*>(signal_pad_peer),
139:                 static_cast<unsigned long long>(signal_value));
140:         }
141:     }
142: }
143: 
144: __global__ void nccl_wait_for_signal_kernel(
```

- EN: Lines 129-144 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 129-144 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 145-160 / 第 145-160 行

```cpp
145:     void**  signal_pad,
146:     int  cur_rank,
147:     uint64_t  target_signal_value
148: ) {
149:     if (blockIdx.x == 0 && threadIdx.x == 0) {
150:         volatile unsigned long long* sig_ptr =
151:             reinterpret_cast<volatile unsigned long long*>(signal_pad[cur_rank]);
152: 
153:         while (true) {
154:             unsigned long long val = *sig_ptr;
155:             if (val >= static_cast<unsigned long long>(target_signal_value)) break;
156: #if defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 700)
157:             __nanosleep(64);
158: #endif
159:         }
160:     }
```

- EN: Lines 145-160 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 145-160 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 161-176 / 第 161-176 行

```cpp
161: }
162: #endif
163: 
164: void nccl_put(at::Tensor& tensor, const int64_t peer) {
165: #ifdef NCCL_HAS_SYMMEM_SUPPORT
166:   // TODO: support non-contiguous tensors
167:   TORCH_CHECK(tensor.is_contiguous(),
168:       "put op currently supports contiguous tensors only");
169:   // TODO: rendezvous should remember the group name
170:   auto symm_mem = c10d::symmetric_memory::rendezvous(tensor, "0");
171:   int threads = THREADS_PER_BLOCK;
172:   int blocks  = (tensor.numel() + threads - 1) / threads;
173:   c10::cuda::CUDAGuard guard(tensor.device());
174:   size_t nbytes = tensor.numel() * c10::elementSize(tensor.scalar_type());
175: 
176:   lsa_put_kernel<<<blocks, threads, 0, at::cuda::getCurrentCUDAStream()>>>(
```

- EN: Lines 161-176 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `nccl_put`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 161-176 行使用条件编译来适配特性开关、平台或可选后端；在 `nccl_put` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 177-192 / 第 177-192 行

```cpp
177:     symm_mem->get_buffer_ptrs_dev(),
178:     peer,
179:     0,
180:     tensor.data_ptr(),
181:     nbytes);
182:   C10_CUDA_KERNEL_LAUNCH_CHECK();
183: #else
184:   TORCH_CHECK(false, "NCCL symmetric memory is not supported. Requires NCCL >= 2.28.9");
185: #endif
186: }
187: 
188: void nccl_wait_for_signal(at::Tensor& sigpad, int64_t signal) {
189: #ifdef NCCL_HAS_SYMMEM_SUPPORT
190:   c10::cuda::CUDAGuard guard(sigpad.device());
191:   auto stream = at::cuda::getCurrentCUDAStream();
192:   auto symm_mem = c10d::symmetric_memory::rendezvous(sigpad, "0");
```

- EN: Lines 177-192 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`, `TORCH_CHECK`, `nccl_wait_for_signal`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 177-192 行使用条件编译来适配特性开关、平台或可选后端；在 `C10_CUDA_KERNEL_LAUNCH_CHECK`、`TORCH_CHECK`、`nccl_wait_for_signal` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 193-208 / 第 193-208 行

```cpp
193: 
194:   // Always use device-side kernel because this function waits for a SPECIFIC signal value.
195:   // ncclWaitSignal only synchronizes on a channel without checking values, so it's not
196:   // suitable for this API which expects to wait for signal pad to reach a specific value.
197:   int cur_rank = symm_mem->get_rank();
198:   nccl_wait_for_signal_kernel<<<1, THREADS_PER_BLOCK, 0, stream>>>(
199:     symm_mem->get_signal_pad_ptrs_dev(),
200:     cur_rank,
201:     signal);
202:   C10_CUDA_KERNEL_LAUNCH_CHECK();
203: #else
204:   TORCH_CHECK(false, "NCCL symmetric memory is not supported. Requires NCCL >= 2.28.9");
205: #endif
206: }
207: 
208: void nccl_put_with_signal(at::Tensor& tensor, int64_t signal, int64_t peer) {
```

- EN: Lines 193-208 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`, `TORCH_CHECK`, `nccl_put_with_signal`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 193-208 行使用条件编译来适配特性开关、平台或可选后端；在 `C10_CUDA_KERNEL_LAUNCH_CHECK`、`TORCH_CHECK`、`nccl_put_with_signal` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 209-224 / 第 209-224 行

```cpp
209: #ifdef NCCL_HAS_SYMMEM_SUPPORT
210:   // TODO: support non-contiguous tensors
211:   TORCH_CHECK(tensor.is_contiguous(),
212:       "put op currently supports contiguous tensors only");
213:   // TODO: rendezvous should remember the group name
214:   auto symm_mem = c10d::symmetric_memory::rendezvous(tensor, "0");
215:   c10::cuda::CUDAGuard guard(tensor.device());
216:   auto stream = at::cuda::getCurrentCUDAStream();
217: 
218:   // Always use device-side kernel because this function writes a SPECIFIC signal value.
219:   // ncclPutSignal expects a channel index (0-7) for the signal parameter, not a signal value,
220:   // so it's not suitable for this API which needs to write specific values to the signal pad.
221:   int threads = THREADS_PER_BLOCK;
222:   int blocks = (tensor.numel() + threads - 1) / threads;
223:   auto opts = at::TensorOptions()
224:     .dtype(at::kInt)
```

- EN: Lines 209-224 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 209-224 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 225-240 / 第 225-240 行

```cpp
225:     .device(tensor.device());
226:   at::Tensor blocks_done = at::zeros({1}, opts);
227:   unsigned int* blocks_done_dev =
228:     reinterpret_cast<unsigned int*>(blocks_done.data_ptr<int>());
229:   size_t nbytes = tensor.numel() * c10::elementSize(tensor.scalar_type());
230: 
231:   lsa_put_signal_kernel<<<blocks, threads, 0, stream>>>(
232:     symm_mem->get_buffer_ptrs_dev(),
233:     symm_mem->get_signal_pad_ptrs_dev(),
234:     peer,
235:     0,
236:     tensor.data_ptr(),
237:     nbytes,
238:     blocks_done_dev,
239:     signal);
240:   C10_CUDA_KERNEL_LAUNCH_CHECK();
```

- EN: Lines 225-240 introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 225-240 行在 `C10_CUDA_KERNEL_LAUNCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 241-256 / 第 241-256 行

```cpp
241: #else
242:   TORCH_CHECK(false, "NCCL symmetric memory is not supported. Requires NCCL >= 2.28.9");
243: #endif
244: }
245: 
246: #ifdef NCCL_HAS_SYMMEM_SUPPORT
247: __global__ void lsa_get_kernel(
248:     void**  buffer,  // buffers_dev_
249:     int  peer,
250:     size_t  src_byte_offset, // byte offset inside that peer's buffer
251:     void*  dst,
252:     size_t  nbytes
253: ) {
254:     const size_t tid = blockIdx.x * blockDim.x + threadIdx.x;
255:     const size_t stride = blockDim.x * gridDim.x;
256: 
```

- EN: Lines 241-256 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `TORCH_CHECK`, `lsa_get_kernel`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 241-256 行使用条件编译来适配特性开关、平台或可选后端；在 `TORCH_CHECK`、`lsa_get_kernel` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 257-272 / 第 257-272 行

```cpp
257:     // remote src pointer
258:     auto src = get_remote_ptr(buffer, peer, src_byte_offset);
259:     char* dst_bytes = reinterpret_cast<char*>(dst);
260:     copy_bytes_vec16(src, dst_bytes, nbytes, tid, stride);
261: }
262: #endif
263: 
264: void nccl_get(at::Tensor& tensor, const int64_t peer) {
265: #ifdef NCCL_HAS_SYMMEM_SUPPORT
266:   // TODO: support non-contiguous tensors
267:   TORCH_CHECK(tensor.is_contiguous(),
268:       "get op currently supports contiguous tensors only");
269:   // TODO: rendezvous should remember the group name
270:   auto symm_mem = c10d::symmetric_memory::rendezvous(tensor, "0");
271:   c10::cuda::CUDAGuard guard(tensor.device());
272:   int threads = THREADS_PER_BLOCK;
```

- EN: Lines 257-272 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `nccl_get`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 257-272 行使用条件编译来适配特性开关、平台或可选后端；在 `nccl_get` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 273-288 / 第 273-288 行

```cpp
273:   int blocks  = (tensor.numel() + threads - 1) / threads;
274:   size_t nbytes = tensor.numel() * c10::elementSize(tensor.scalar_type());
275: 
276:   lsa_get_kernel<<<blocks, threads, 0, at::cuda::getCurrentCUDAStream()>>>(
277:     symm_mem->get_buffer_ptrs_dev(),
278:     peer,
279:     0,
280:     tensor.data_ptr(),
281:     nbytes);
282:   C10_CUDA_KERNEL_LAUNCH_CHECK();
283: #else
284:   TORCH_CHECK(false, "NCCL symmetric memory is not supported. Requires NCCL >= 2.28.9");
285: #endif
286: }
287: 
288: bool is_nccl_symmem_available() {
```

- EN: Lines 273-288 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`, `TORCH_CHECK`, `is_nccl_symmem_available`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 273-288 行使用条件编译来适配特性开关、平台或可选后端；在 `C10_CUDA_KERNEL_LAUNCH_CHECK`、`TORCH_CHECK`、`is_nccl_symmem_available` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 289-304 / 第 289-304 行

```cpp
289: #ifdef NCCL_HAS_SYMMEM_SUPPORT
290:     return true;
291: #else
292:     return false;
293: #endif
294: }
295: 
296: void nccl_put_signal(at::Tensor& tensor, const c10::intrusive_ptr<SymmetricMemory>& hdl, int64_t peer) {
297: #ifdef NCCL_HAS_ONE_SIDED_API
298:   // Check input arguments
299:   TORCH_CHECK(tensor.is_contiguous(),
300:       "nccl_put_signal op currently supports contiguous tensors only");
301:   TORCH_CHECK(peer < hdl->get_world_size(), "peer must be smaller than world size");
302: 
303:   // Context setup
304:   auto device = tensor.device();
```

- EN: Lines 289-304 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `nccl_put_signal`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 289-304 行使用条件编译来适配特性开关、平台或可选后端；在 `nccl_put_signal` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 305-320 / 第 305-320 行

```cpp
305:   c10::cuda::CUDAGuard guard(device);
306:   auto stream = at::cuda::getCurrentCUDAStream();
307: 
308:   // Get window etc
309:   auto nccl_hdl = dynamic_cast<NCCLSymmetricMemory*>(hdl.get());
310:   auto window = nccl_hdl->get_window();
311:   TORCH_CHECK(window != nullptr, "window is nullptr");
312:   auto offset = nccl_hdl->get_offset();
313:   auto byte_size = tensor.numel() * tensor.element_size();
314: 
315:   // Get the NCCL communicator
316:   auto& manager = NCCLDevCommManager::get(device);
317:   ncclComm_t comm = manager.get_comm(nccl_hdl->get_group_name());
318: 
319:   // Issue the NCCL API
320:   C10D_NCCL_CHECK(
```

- EN: Lines 305-320 introduces executable logic in routines such as `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 305-320 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 321-336 / 第 321-336 行

```cpp
321:       ncclPutSignal(
322:           tensor.data_ptr(), byte_size, ncclChar,
323:           peer, window, offset,
324:           /*sigIdx=*/0, /*ctx=*/0, /*flags=*/0, // Need to be 0 for now
325:           comm, stream),
326:       c10::str("ncclPutSignal failed"));
327: #else
328:   TORCH_CHECK(false, "NCCL one-sided API is not supported. Requires NCCL >= 2.29.0");
329: #endif // NCCL_HAS_ONE_SIDED_API
330: }
331: 
332: void nccl_wait_signal(const c10::intrusive_ptr<SymmetricMemory>& hdl, int64_t peer) {
333: #ifdef NCCL_HAS_ONE_SIDED_API
334:   // Check input arguments
335:   TORCH_CHECK(peer < hdl->get_world_size(), "peer must be smaller than world size");
336: 
```

- EN: Lines 321-336 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `TORCH_CHECK`, `nccl_wait_signal`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 321-336 行使用条件编译来适配特性开关、平台或可选后端；在 `TORCH_CHECK`、`nccl_wait_signal` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 337-352 / 第 337-352 行

```cpp
337:   // Context setup
338:   auto device = hdl->get_device();
339:   c10::cuda::CUDAGuard guard(device);
340:   auto stream = at::cuda::getCurrentCUDAStream();
341: 
342:   // Get the NCCL handle
343:   auto nccl_hdl = dynamic_cast<NCCLSymmetricMemory*>(hdl.get());
344:   // Get the NCCL communicator
345:   auto& manager = NCCLDevCommManager::get(device);
346:   ncclComm_t comm = manager.get_comm(nccl_hdl->get_group_name());
347: 
348:   ncclWaitSignalDesc_t signalDesc;
349:   signalDesc.opCnt = 1;
350:   signalDesc.peer = peer;
351:   signalDesc.sigIdx = 0; // Need to be 0 for now
352:   signalDesc.ctx = 0; // Need to be 0 for now
```

- EN: Lines 337-352 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 337-352 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 353-368 / 第 353-368 行

```cpp
353: 
354:   C10D_NCCL_CHECK(
355:       ncclWaitSignal(1, &signalDesc, comm, stream),
356:       c10::str("ncclWaitSignal failed"));
357: #else
358:   TORCH_CHECK(false, "NCCL one-sided API is not supported. Requires NCCL >= 2.29.0");
359: #endif // NCCL_HAS_ONE_SIDED_API
360: }
361: 
362: } // namespace c10d::nccl_extension
363: 
364: 
365: namespace {
366: // Boxed functions for the APIs that use custom class `SymmetricMemory`.
367: // Why do we need boxed functions?
368: // Problem:
```

- EN: Lines 353-368 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `TORCH_CHECK`.
- CN: 第 353-368 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `TORCH_CHECK` 等例程中引入具体执行逻辑。

### Lines 369-384 / 第 369-384 行

```cpp
369: // `SymmetricMemory` is a custom class that needs to be TorchBind'ed first
370: // before it can be supported by the dispatcher. Since both TorchBind
371: // registration and m.impl registration happen in static initialization, we can
372: // hit static initialization order fiasco since C++ provides no ordering
373: // guarantees for static initializers across translation units.
374: // Solution:
375: // Use boxed kernels that operate on the IValue stack directly. The
376: // makeFromBoxedFunction path never calls inferFunctionSchemaFromFunctor at all:
377: // For details, see https://github.com/pytorch/pytorch/pull/174034.
378: 
379: void nccl_put_signal_boxed(
380:     const c10::OperatorHandle& op,
381:     c10::DispatchKeySet ks,
382:     c10::Stack* stack) {
383:   auto peer = torch::jit::pop(*stack).toInt();
384:   auto hdl = torch::jit::pop(*stack).toCustomClass<
```

- EN: Lines 369-384 introduces executable logic in routines such as `nccl_put_signal_boxed`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 369-384 行在 `nccl_put_signal_boxed` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 385-400 / 第 385-400 行

```cpp
385:       c10d::symmetric_memory::SymmetricMemory>();
386:   auto tensor = torch::jit::pop(*stack).toTensor();
387:   c10d::nccl_extension::nccl_put_signal(tensor, hdl, peer);
388: }
389: 
390: void nccl_wait_signal_boxed(
391:     const c10::OperatorHandle& op,
392:     c10::DispatchKeySet ks,
393:     c10::Stack* stack) {
394:   auto peer = torch::jit::pop(*stack).toInt();
395:   auto hdl = torch::jit::pop(*stack).toCustomClass<
396:       c10d::symmetric_memory::SymmetricMemory>();
397:   c10d::nccl_extension::nccl_wait_signal(hdl, peer);
398: }
399: } // namespace
400: 
```

- EN: Lines 385-400 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `nccl_wait_signal_boxed`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 385-400 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `nccl_wait_signal_boxed` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 401-416 / 第 401-416 行

```cpp
401: TORCH_LIBRARY_IMPL(symm_mem, CUDA, m) {
402:   m.impl("nccl_put", c10d::nccl_extension::nccl_put);
403:   m.impl("nccl_get", c10d::nccl_extension::nccl_get);
404:   // APIs that accept a signal pad from user
405:   // TODO: rename with more descriptive name
406:   m.impl("nccl_wait_for_signal", c10d::nccl_extension::nccl_wait_for_signal);
407:   m.impl("nccl_put_with_signal", c10d::nccl_extension::nccl_put_with_signal);
408:   // API that uses internal signal mechanism and accepts handle
409:   m.impl("nccl_put_signal",
410:       torch::CppFunction::makeFromBoxedFunction<&nccl_put_signal_boxed>());
411:   m.impl("nccl_reduce_scatter_offset", c10d::nccl_extension::nccl_reduce_scatter_offset);
412: }
413: 
414: // Use CompositeExplicitAutograd as key since ops do not accept tensor as input
415: TORCH_LIBRARY_IMPL(symm_mem, CompositeExplicitAutograd, m) {
416:   // API that uses internal signal mechanism and accepts handle
```

- EN: Lines 401-416 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 401-416 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 417-419 / 第 417-419 行

```cpp
417:   m.impl("nccl_wait_signal",
418:       torch::CppFunction::makeFromBoxedFunction<&nccl_wait_signal_boxed>());
419: }
```

- EN: Lines 417-419 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 417-419 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

## Key Concepts / 关键概念

- EN: Subsystem: c10d symmetric-memory support.
- CN: 子系统：c10d 对称内存支持。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `get_remote_ptr`, `copy_bytes_vec16`, `lsa_put_kernel`, `__syncthreads`, `__threadfence_system`, `nccl_wait_for_signal_kernel`
- CN: 核心符号：`get_remote_ptr`、`copy_bytes_vec16`、`lsa_put_kernel`、`__syncthreads`、`__threadfence_system`、`nccl_wait_for_signal_kernel`
- EN: Notable themes: CUDA paths.
- CN: 值得关注的主题：CUDA 路径。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/NCCLUtils.hpp`, `torch/csrc/distributed/c10d/symm_mem/nccl_dev_cap.hpp`, `torch/csrc/distributed/c10d/symm_mem/nccl_extension.hpp`, `torch/csrc/distributed/c10d/symm_mem/nccl_devcomm_manager.hpp`, `torch/csrc/distributed/c10d/symm_mem/NCCLSymmetricMemory.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `c10/cuda/CUDAGuard.h`, `ATen/native/cuda/MemoryAccess.cuh`
- External or system headers / 外部或系统头文件: None / 无
- Local symbols / 本地符号: `get_remote_ptr`, `copy_bytes_vec16`, `lsa_put_kernel`, `__syncthreads`, `__threadfence_system`, `nccl_wait_for_signal_kernel`, `nccl_put`, `C10_CUDA_KERNEL_LAUNCH_CHECK`