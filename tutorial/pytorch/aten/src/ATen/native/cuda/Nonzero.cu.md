# Nonzero.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/Nonzero.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `write_indices`, `write_fill_value`, `compute_agg`, `flag_kernel`.
- 用途（中文）: 实现与 `write_indices`, `write_fill_value`, `compute_agg`, `flag_kernel` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/Dispatch.h>
   3: #include <ATen/EmptyTensor.h>
   4: #include <ATen/core/Tensor.h>
   5: #include <ATen/cuda/CUDAContext.h>
   6: #include <ATen/cuda/EmptyTensor.h>
   7: #include <ATen/cuda/detail/KernelUtils.h>
   8: #include <c10/cuda/CUDACachingAllocator.h>
   9: #include <ATen/cuda/cub.cuh>
  10: #include <ATen/cuda/detail/OffsetCalculator.cuh> //for MAX_DIMS
  11: 
  12: #ifndef AT_PER_OPERATOR_HEADERS
  13: #include <ATen/NativeFunctions.h>
  14: #else
  15: #include <ATen/ops/empty_native.h>
  16: #include <ATen/ops/nonzero_native.h>
  17: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/Dispatch.h>`, `<ATen/EmptyTensor.h>`, `<ATen/core/Tensor.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/Dispatch.h>`, `<ATen/EmptyTensor.h>`, `<ATen/core/Tensor.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 19-40
```cpp
  19: namespace at::native {
  20: 
  21: namespace {
  22: template <typename T>
  23: struct NonZeroOp {
  24:   __host__ __device__ __forceinline__ bool operator()(const T& a) const {
  25:     return (a != T(0));
  26:   }
  27: };
  28: 
  29: // TODO: actually support int64_t index_t
  30: template <typename index_t>
  31: struct TensorDims {
  32:   index_t sizes[MAX_DIMS];
  33: };
  34: 
  35: template <typename index_t>
  36: __global__ void write_indices(
  37:     int64_t* inp,
  38:     TensorDims<index_t> dims,
  39:     int ndim,
  40:     index_t n,
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `write_indices`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `write_indices`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 41-62
```cpp
  41:     int64_t * total = nullptr,
  42:     int64_t fill_value = -1) {
  43:   auto index = threadIdx.x + (int64_t)blockIdx.x * blockDim.x;
  44:   bool cond = (total == nullptr || index < *total);
  45:   if (index < n && cond) {
  46:     index_t div = 1;
  47:     int64_t idx_flat = inp[index];
  48: #pragma unroll
  49:     for (int dim = MAX_DIMS; dim >= 0; dim--) {
  50:       if (dim > ndim - 1)
  51:         continue;
  52:       auto dim_size = dims.sizes[dim];
  53:       inp[index + dim * n] = (idx_flat / div) % dim_size;
  54:       div *= dim_size;
  55:     }
  56:   } else if (index < n) {
  57:     // 0th dim has correct values already
  58:     for (int dim = ndim - 1; dim > 0; dim--) {
  59:       inp[index + dim * n] = fill_value;
  60:     }
  61:   }
  62: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 64-71
```cpp
  64: __global__ void write_fill_value(int64_t * inp, int64_t * total, int64_t fill_value, int64_t n){
  65:   int64_t total_val = *total;
  66:   // not aiming for vectorized stores
  67: 
  68:   for (int64_t idx = total_val + (int64_t)blockIdx.x * blockDim.x + threadIdx.x; idx < n; idx += blockDim.x * gridDim.x) {
  69:       inp[idx] = fill_value;
  70:   }
  71: }
```
- EN: This block defines GPU kernel entry point(s) `write_fill_value`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `write_fill_value`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 73-85
```cpp
  73: template <int BLOCK_THREADS>
  74: __global__ void compute_agg(int32_t * agg, int64_t * agg_cum, uint32_t n_blocks) {
  75: 
  76:   using BlockScanT = ROCM_HIPCUB(at_cuda_detail::cub)::BlockScan<int64_t, BLOCK_THREADS, ROCM_HIPCUB(at_cuda_detail::cub)::BLOCK_SCAN_WARP_SCANS>;
  77:   __shared__ typename BlockScanT::TempStorage temp_storage;
  78:   int agg_data;
  79:   int64_t agg_cum_data;
  80:   agg_data = threadIdx.x < n_blocks ? agg[threadIdx.x] : 0;
  81:   BlockScanT(temp_storage).InclusiveSum(agg_data, agg_cum_data);
  82:   if (threadIdx.x < n_blocks) {
  83:     agg_cum[threadIdx.x] = agg_cum_data;
  84:   }
  85: }
```
- EN: This block defines GPU kernel entry point(s) `compute_agg`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `compute_agg`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 87-108
```cpp
  87: template<int BLOCK_THREADS, int ITEMS_PER_THREAD, typename T>
  88: __global__ void flag_kernel(const T* d_in, int64_t * d_out, const int64_t * agg, int64_t input_nelem, int64_t output_nelem, int iters_per_cta) {
  89:   int64_t start_idx = BLOCK_THREADS * ITEMS_PER_THREAD * iters_per_cta * (int64_t)blockIdx.x;
  90:   if (start_idx >= input_nelem) return;
  91:   d_in += start_idx;
  92: 
  93:   using BlockLoadT = ROCM_HIPCUB(at_cuda_detail::cub)::BlockLoad<int, BLOCK_THREADS, ITEMS_PER_THREAD, ROCM_HIPCUB(at_cuda_detail::cub)::BLOCK_LOAD_WARP_TRANSPOSE>;
  94: 
  95:   // Specialize BlockScan type for our thread block
  96:   using BlockScanT = ROCM_HIPCUB(at_cuda_detail::cub)::BlockScan<int, BLOCK_THREADS, ROCM_HIPCUB(at_cuda_detail::cub)::BLOCK_SCAN_WARP_SCANS>;
  97:   using TransformInputIteratorT = ATEN_CUB_TRANSFORM_ITERATOR(int, NonZeroOp<T>, const T*);
  98:   using BlockExchangeT =  ROCM_HIPCUB(at_cuda_detail::cub)::BlockExchange<int, BLOCK_THREADS, ITEMS_PER_THREAD>;
  99: 
 100:   // Shared memory
 101:   __shared__ union TempStorage
 102:   {
 103:     typename BlockLoadT::TempStorage load;
 104:     typename BlockScanT::TempStorage scan;
 105:     typename BlockExchangeT::TempStorage exchange;
 106:   } temp_storage;
 107: 
 108:   int64_t aggregate = blockIdx.x == 0 ? 0 : agg[blockIdx.x - 1];
```
- EN: This block defines GPU kernel entry point(s) `flag_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `flag_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 109-109
```cpp
 109:   d_out += aggregate;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 111-111
```cpp
 111:   TransformInputIteratorT t_input_itr(d_in, NonZeroOp<T>());
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 113-113
```cpp
 113:   // Per-thread tile data
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 114-115
```cpp
 114:   int data[ITEMS_PER_THREAD];
 115:   int out_indices[ITEMS_PER_THREAD];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 117-138
```cpp
 117:   int64_t remaining =  input_nelem - start_idx;
 118:   int64_t out_remaining = output_nelem - aggregate;
 119:   for (int i=0; i<iters_per_cta; i++){
 120: 
 121:   // Load items into a blocked arrangement
 122:     if (remaining >= BLOCK_THREADS * ITEMS_PER_THREAD) {
 123:       BlockLoadT(temp_storage.load).Load(t_input_itr, data);
 124:     } else {
 125:       BlockLoadT(temp_storage.load).Load(t_input_itr, data, remaining, int(0));
 126:     }
 127: 
 128:     // Barrier for smem reuse
 129:     __syncthreads();
 130: 
 131:     // Compute inclusive prefix sum
 132:     int aggregate;
 133:     __shared__ int aggregate_sh;
 134:     BlockScanT(temp_storage.scan).ExclusiveSum(data, out_indices, aggregate);
 135: 
 136:     if (threadIdx.x == 0){
 137:       aggregate_sh = aggregate;
 138:     }
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 140-140
```cpp
 140:     // Barrier for smem reuse
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 141-152
```cpp
 141:     __syncthreads();
 142:     // striped arrangement will provide a slightly better
 143:     // coalescing for writes (although it's still bad because it's indirect indexing)
 144:     BlockExchangeT(temp_storage.exchange).BlockedToStriped(data);
 145:     __syncthreads();
 146:     BlockExchangeT(temp_storage.exchange).BlockedToStriped(out_indices);
 147:     for (int ii=0; ii<ITEMS_PER_THREAD; ii++){
 148:       if (data[ii] != 0 && out_indices[ii] < out_remaining) {
 149:         int64_t inp_idx = start_idx + threadIdx.x + blockDim.x * ii;
 150:         d_out[out_indices[ii]] = inp_idx;
 151:       }
 152:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 154-161
```cpp
 154:     out_remaining -= aggregate_sh;
 155:     remaining -= BLOCK_THREADS * ITEMS_PER_THREAD;
 156:     if (remaining <= 0 || out_remaining <= 0) return;
 157:     d_out += aggregate_sh;
 158:     t_input_itr += BLOCK_THREADS * ITEMS_PER_THREAD;
 159:     start_idx += BLOCK_THREADS * ITEMS_PER_THREAD;
 160:     __syncthreads();
 161:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 163-163
```cpp
 163: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 167-167
```cpp
 167: } // anonymous namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 169-190
```cpp
 169: template <typename scalar_t>
 170: void nonzero_cuda_out_impl(const Tensor& self, Tensor& out) {
 171:   Tensor self_ = self.contiguous();
 172:   const cudaStream_t stream = at::cuda::getCurrentCUDAStream();
 173:   int64_t chunk_size, num_chunks;
 174:   if (self.numel() < std::numeric_limits<int>::max()) {
 175:     chunk_size = self.numel();
 176:     num_chunks = 1;
 177:   } else {
 178:     chunk_size = std::numeric_limits<int>::max() / 2 + 1; // 2**30
 179:     num_chunks = (self.numel() + chunk_size - 1) / chunk_size;
 180:   }
 181:   // compute number of nonzero elements
 182:   size_t temp_storage_bytes = 0;
 183:   auto& allocator = *c10::cuda::CUDACachingAllocator::get();
 184:   auto num_nonzeros = allocator.allocate(sizeof(int) * num_chunks);
 185:   for (int64_t idx = 0; idx < num_chunks; idx++) {
 186:     int64_t remaining = std::min<int64_t>(chunk_size, self.numel() - idx * chunk_size);
 187:     ATEN_CUB_TRANSFORM_ITERATOR(bool, NonZeroOp<scalar_t>, const scalar_t*) itr(
 188:         self_.const_data_ptr<scalar_t>() + idx * chunk_size,
 189:         NonZeroOp<scalar_t>());
 190:     AT_CUDA_CHECK(cub::DeviceReduce::Sum(
```
- EN: This block defines or continues the implementation of `nonzero_cuda_out_impl`.
- CN: 该代码块定义或继续实现 `nonzero_cuda_out_impl`。

### Lines 191-204
```cpp
 191:         nullptr,
 192:         temp_storage_bytes,
 193:         itr,
 194:         ((int*)num_nonzeros.get()) + idx,
 195:         remaining,
 196:         stream));
 197:     auto temp_storage = allocator.allocate(temp_storage_bytes);
 198:     AT_CUDA_CHECK(cub::DeviceReduce::Sum(
 199:         temp_storage.get(),
 200:         temp_storage_bytes,
 201:         itr,
 202:         ((int*)num_nonzeros.get()) + idx,
 203:         remaining,
 204:         stream));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 205-218
```cpp
 205:   }
 206:   auto pinned_num_nonzeros_h = at::detail::empty_cpu(
 207:       {num_chunks}, /* size */
 208:       c10::CppTypeToScalarType<int>(), /* dtype */
 209:       std::nullopt, /* layout */
 210:       std::nullopt, /* device */
 211:       true, /* pin_memory */
 212:       std::nullopt /* memory format */
 213:   );
 214:   at::cuda::memcpy_and_sync(
 215:       pinned_num_nonzeros_h.template data_ptr<int>(),
 216:       num_nonzeros.get(),
 217:       sizeof(int) * num_chunks,
 218:       cudaMemcpyDeviceToHost,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 219-220
```cpp
 219:       stream);
 220:   int64_t num_nonzeros_h = 0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 222-235
```cpp
 222:   for (int64_t idx = 0; idx < num_chunks; idx++) {
 223:     num_nonzeros_h += pinned_num_nonzeros_h.template const_data_ptr<int>()[idx];
 224:   }
 225:   // num_nonzeros_h = (int)*(pinned_num_nonzeros_h.const_data_ptr<int>());
 226:   // expected output size is num_nonzeros x ndim
 227:   // we are producing output with size {num_nonzeros, ndim} and strides {1,
 228:   // num_nonzeros} (that is, transposed ndim x num_nonzeros output) we are able
 229:   // to directly use passed output with this size and strides, and we can also
 230:   // (per contract) resize passed output with incorrect sizes anyway we want.
 231:   // However, out with correct sizes and incorrect strides will have to be
 232:   // copied to from the intermediate we've produced.
 233:   bool need_to_copy = out.dim() == 2 && out.sizes()[0] == num_nonzeros_h &&
 234:       out.sizes()[1] == self.dim() && !out.t().is_contiguous();
 235:   at::Tensor out_temp = need_to_copy
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 236-257
```cpp
 236:       ? Tensor(
 237:             at::detail::empty_cuda({self.dim(), num_nonzeros_h}, out.options()))
 238:       : out.resize_({self.dim(), num_nonzeros_h});
 239:   // Scalars are expected to produce output of size (1,0), so we can't write to
 240:   // it
 241:   int64_t curr_nonzeros = 0;
 242:   if (self.dim() > 0) {
 243:     for (int64_t idx = 0; idx < num_chunks; idx++) {
 244:       int remaining = std::min<int64_t>(chunk_size, self.numel() - idx * chunk_size);
 245: 
 246:       ATEN_CUB_COUNTING_ITERATOR(int64_t) counting_itr(idx * chunk_size);
 247:       ATEN_CUB_TRANSFORM_ITERATOR(bool, NonZeroOp<scalar_t>, const scalar_t*)
 248:           itr(self_.const_data_ptr<scalar_t>() + idx * chunk_size,
 249:               NonZeroOp<scalar_t>());
 250:       temp_storage_bytes = 0;
 251:       AT_CUDA_CHECK(cub::DeviceSelect::Flagged(
 252:           nullptr,
 253:           temp_storage_bytes,
 254:           counting_itr,
 255:           itr,
 256:           out_temp.mutable_data_ptr<int64_t>(),
 257:           ((int*)num_nonzeros.get()) + idx,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 258-271
```cpp
 258:           remaining,
 259:           stream));
 260:       auto temp_storage = allocator.allocate(temp_storage_bytes);
 261:       AT_CUDA_CHECK(cub::DeviceSelect::Flagged(
 262:           temp_storage.get(),
 263:           temp_storage_bytes,
 264:           counting_itr,
 265:           itr,
 266:           out_temp.mutable_data_ptr<int64_t>() + curr_nonzeros,
 267:           ((int*)num_nonzeros.get()) + idx,
 268:           remaining,
 269:           stream));
 270:       curr_nonzeros += pinned_num_nonzeros_h.template const_data_ptr<int>()[idx];
 271:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 272-285
```cpp
 272:     if (num_nonzeros_h > 0 && self.dim() > 1) {
 273:       TensorDims<int64_t> dims;
 274:       for (int i = 0; i < self.dim(); i++) {
 275:         dims.sizes[i] = self.sizes()[i];
 276:       }
 277:       const int nthreads = 256;
 278:       const int nblocks = (num_nonzeros_h + nthreads - 1) / nthreads;
 279:       write_indices<<<nblocks, nthreads, 0, stream>>>(
 280:           out_temp.mutable_data_ptr<int64_t>(),
 281:           dims,
 282:           self.dim(),
 283:           num_nonzeros_h);
 284:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 285:     }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 286-294
```cpp
 286:   }
 287:   if (need_to_copy) {
 288:     out.copy_(out_temp.t());
 289:   } else {
 290:     // transpose out so it is correct size
 291:     Tensor out_ = out_temp.t();
 292:     out.set_(out_);
 293:   }
 294: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 296-317
```cpp
 296: template <typename scalar_t>
 297: void nonzero_static_cuda_out_impl(
 298:     const Tensor& self,
 299:     int64_t size,
 300:     int64_t fill_value,
 301:     Tensor& out) {
 302:   Tensor self_contiguous_ = self.contiguous();
 303:   // see comment in nonzero_cuda_out_impl on reqs for out
 304:   bool out_correct_size =
 305:       out.dim() == 2 && out.sizes()[0] == size && out.sizes()[1] == self.dim();
 306:   bool need_to_copy = out_correct_size && !out.t().is_contiguous();
 307:   if (!out_correct_size) {
 308:     out.resize_({self.dim(), size}).t();
 309:   }
 310:   if (out.numel() == 0) return;
 311:   // we need to allocate temporary out to then copy to user provided out
 312:   at::Tensor out_temp;
 313:   if (need_to_copy) {
 314:     out_temp =
 315:         Tensor(at::detail::empty_cuda({self.dim(), size}, out.options())).t();
 316:   }
 317:   // If input has zero elements, avoid kernel grid calculations (which can
```
- EN: This block defines or continues the implementation of `nonzero_static_cuda_out_impl`.
- CN: 该代码块定义或继续实现 `nonzero_static_cuda_out_impl`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 318-318
```cpp
 318:   // produce zero divisors) and just fill the output with fill_value.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 319-329
```cpp
 319:   if (self.numel() == 0) {
 320:     if (need_to_copy) {
 321:       out_temp.fill_(fill_value);
 322:       out.copy_(out_temp);
 323:     } else {
 324:       out.fill_(fill_value);
 325:     }
 326:     return;
 327:   }
 328:   int64_t* out_data_ptr = need_to_copy ? out_temp.mutable_data_ptr<int64_t>()
 329:                                        : out.mutable_data_ptr<int64_t>();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 331-344
```cpp
 331:   const scalar_t * in_data_ptr = self_contiguous_.const_data_ptr<scalar_t>();
 332:   constexpr int BLOCK_THREADS = 512; //block_threads<sizeof(scalar_t)>();
 333:   constexpr int ITEMS_PER_THREAD = 16;
 334:   auto grid_size = (self.numel() + BLOCK_THREADS * ITEMS_PER_THREAD - 1) / (BLOCK_THREADS * ITEMS_PER_THREAD);
 335:   const int64_t num_sms = at::cuda::getCurrentDeviceProperties()->multiProcessorCount;
 336:   int64_t target_blocks = sizeof(scalar_t) == 1 ? 2 * num_sms : num_sms;
 337:   const int iters_per_cta = (grid_size + target_blocks - 1)/target_blocks;
 338:   grid_size = (self.numel() + iters_per_cta * BLOCK_THREADS * ITEMS_PER_THREAD - 1) / (iters_per_cta * BLOCK_THREADS * ITEMS_PER_THREAD);
 339:   auto& allocator = *c10::cuda::CUDACachingAllocator::get();
 340:   auto agg = allocator.allocate(grid_size * sizeof(int));
 341:   at::cuda::cub::calc_block_sums<BLOCK_THREADS, ITEMS_PER_THREAD, true>
 342:   <<<grid_size, BLOCK_THREADS, 0, at::cuda::getCurrentCUDAStream()>>>(
 343:     in_data_ptr, (int*)agg.get(), self.numel(), iters_per_cta);
 344:   C10_CUDA_KERNEL_LAUNCH_CHECK();
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 345-366
```cpp
 345:   auto agg_cum = allocator.allocate(grid_size * sizeof(int64_t));
 346:   // computing partial sums in int64 in the flag kernel
 347:   // leads to 20-30% slowdown, so compute them in a separate 2 us kernel
 348:   compute_agg<BLOCK_THREADS><<<1, BLOCK_THREADS, 0, at::cuda::getCurrentCUDAStream()>>>(
 349:    (int*)agg.get(), (int64_t*)agg_cum.get(), grid_size
 350:   );
 351:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 352:   flag_kernel<BLOCK_THREADS, ITEMS_PER_THREAD>
 353:   <<<grid_size, BLOCK_THREADS, 0, at::cuda::getCurrentCUDAStream()>>>(
 354:     in_data_ptr, out_data_ptr, (int64_t*)agg_cum.get(), self.numel(), size, iters_per_cta);
 355:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 356:   int64_t out_grid = std::min<int64_t>(num_sms, (size + BLOCK_THREADS - 1)/BLOCK_THREADS);
 357:   write_fill_value<<<out_grid, BLOCK_THREADS, 0, at::cuda::getCurrentCUDAStream()>>>(out_data_ptr, (int64_t *)agg_cum.get() + grid_size - 1, fill_value, size);
 358:   if (self.dim() > 1) {
 359:     TensorDims<int64_t> dims;
 360:     for (int i = 0; i < self.dim(); i++) {
 361:       dims.sizes[i] = self.sizes()[i];
 362:     }
 363:     const int nthreads = 256;
 364:     const int nblocks = (size + nthreads - 1) / nthreads;
 365:     write_indices<<<nblocks, nthreads, 0, at::cuda::getCurrentCUDAStream()>>>(
 366:         out_data_ptr,
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 367-377
```cpp
 367:         dims,
 368:         self.dim(),
 369:         size,
 370:         (int64_t *)agg_cum.get() + grid_size - 1,
 371:         fill_value);
 372:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 373:   }
 374:   if (need_to_copy) {
 375:     out.copy_(out_temp);
 376:   }
 377: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 379-400
```cpp
 379: Tensor& nonzero_out_cuda(const Tensor& self, Tensor& out) {
 380:   TORCH_CHECK(
 381:       out.dtype() == at::kLong,
 382:       "Expected object of scalar type ",
 383:       at::kLong,
 384:       " as out, but got ",
 385:       out.dtype());
 386:   TORCH_CHECK(
 387:       self.device() == out.device(),
 388:       "expected self and out to be on the same device, but got out on ",
 389:       out.device(),
 390:       " and self on ",
 391:       self.device());
 392:   TORCH_CHECK(
 393:       self.dim() <= MAX_DIMS,
 394:       "nonzero is not supported for tensor with more than ",
 395:       MAX_DIMS,
 396:       " dimensions");
 397:   AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND4(
 398:       at::ScalarType::ComplexHalf,
 399:       at::ScalarType::Bool,
 400:       at::ScalarType::BFloat16,
```
- EN: This block defines or continues the implementation of `nonzero_out_cuda`.
- CN: 该代码块定义或继续实现 `nonzero_out_cuda`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 401-406
```cpp
 401:       at::ScalarType::Half,
 402:       self.scalar_type(),
 403:       "nonzero_cuda",
 404:       [&] { nonzero_cuda_out_impl<scalar_t>(self, out); });
 405:   return out;
 406: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 408-411
```cpp
 408: Tensor nonzero_cuda(const Tensor& self) {
 409:   Tensor out = at::detail::empty_cuda({0}, self.options().dtype(kLong));
 410:   return at::native::nonzero_out_cuda(self, out);
 411: }
```
- EN: This block defines or continues the implementation of `nonzero_cuda`.
- CN: 该代码块定义或继续实现 `nonzero_cuda`。

### Lines 413-434
```cpp
 413: Tensor& nonzero_static_out_cuda(
 414:     const Tensor& self,
 415:     int64_t size,
 416:     int64_t fill_value,
 417:     Tensor& out) {
 418:   TORCH_CHECK(
 419:       out.dtype() == at::kLong,
 420:       "nonzero_static: Expected out tensor to have scalar type ",
 421:       at::kLong,
 422:       " but got ",
 423:       out.dtype());
 424:   TORCH_CHECK(
 425:       self.device() == out.device(),
 426:       "expected self and out to be on the same device, but got out on ",
 427:       out.device(),
 428:       " and self on ",
 429:       self.device());
 430:   TORCH_CHECK(
 431:       self.dim() <= MAX_DIMS,
 432:       "nonzero_static is not supported for tensor with more than ",
 433:       MAX_DIMS,
 434:       " dimensions");
```
- EN: This block defines or continues the implementation of `nonzero_static_out_cuda`.
- CN: 该代码块定义或继续实现 `nonzero_static_out_cuda`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 435-448
```cpp
 435:   TORCH_CHECK(
 436:       size >= 0, "nonzero_static: 'size' must be an non-negative integer"
 437:   )
 438:   AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND4(
 439:       at::ScalarType::ComplexHalf,
 440:       at::ScalarType::Bool,
 441:       at::ScalarType::BFloat16,
 442:       at::ScalarType::Half,
 443:       self.scalar_type(),
 444:       "nonzero_cuda",
 445:       [&] {
 446:         nonzero_static_cuda_out_impl<scalar_t>(self, size, fill_value, out);
 447:       });
 448:   return out;
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 449-449
```cpp
 449: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 451-462
```cpp
 451: Tensor nonzero_static_cuda(
 452:     const Tensor& self,
 453:     int64_t size,
 454:     int64_t fill_value) {
 455:   TORCH_CHECK(
 456:       size >= 0, "nonzero_static: 'size' must be an non-negative integer"
 457:   )
 458:   Tensor out = Tensor(at::detail::empty_cuda(
 459:                           {self.dim(), size}, self.options().dtype(kLong)))
 460:                    .t();
 461:   return at::native::nonzero_static_out_cuda(self, size, fill_value, out);
 462: }
```
- EN: This block defines or continues the implementation of `nonzero_static_cuda`.
- CN: 该代码块定义或继续实现 `nonzero_static_cuda`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 464-464
```cpp
 464: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- CUB primitives provide parallel CUDA building blocks such as reductions or scans. / CUB 原语提供归约、扫描等并行 CUDA 基础组件。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/Dispatch.h>`
  - `<ATen/EmptyTensor.h>`
  - `<ATen/core/Tensor.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/EmptyTensor.h>`
  - `<ATen/cuda/detail/KernelUtils.h>`
  - `<c10/cuda/CUDACachingAllocator.h>`
  - `<ATen/cuda/cub.cuh>`
  - `<ATen/cuda/detail/OffsetCalculator.cuh>`
  - `<ATen/NativeFunctions.h>`
  - `<ATen/ops/empty_native.h>`
  - `<ATen/ops/nonzero_native.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND4`
  - `cub::DeviceReduce::Sum`
  - `cub::DeviceSelect::Flagged`
  - `cub::calc_block_sums`
  - `at::cuda::getCurrentCUDAStream`
  - `at::cuda::memcpy_and_sync`
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::cub::calc_block_sums`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
