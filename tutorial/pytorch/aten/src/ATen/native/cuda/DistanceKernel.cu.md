# DistanceKernel.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/DistanceKernel.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `sign`, `inc`, `finish`, `agg`.
- 用途（中文）: 实现与 `sign`, `inc`, `finish`, `agg` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/Dispatch.h>
   4: #include <ATen/cuda/Exceptions.h>
   5: #include <ATen/cuda/DeviceUtils.cuh>
   6: #include <ATen/cuda/CUDAContext.h>
   7: #include <math.h>
   8: 
   9: #include <ATen/native/cuda/block_reduce.cuh>
  10: #include <ATen/native/cuda/DeviceSqrt.cuh>
  11: #include <ATen/native/Distance.h>
  12: #include <ATen/NumericUtils.h>
  13: 
  14: #ifndef AT_PER_OPERATOR_HEADERS
  15: #include <ATen/Functions.h>
  16: #else
  17: #include <ATen/ops/empty.h>
  18: #include <ATen/ops/sum.h>
  19: #endif
  20: 
  21: #include <c10/macros/Macros.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/cuda/Exceptions.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/cuda/Exceptions.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 23-44
```cpp
  23: namespace at::native {
  24: 
  25: namespace {
  26: 
  27: constexpr int kCUDANumThreads = 256;
  28: 
  29: template <typename scalar_t>
  30: struct dists {
  31: 
  32:   static __forceinline__ __device__ scalar_t sign(scalar_t val) {
  33:     return (0 < val) - (val < 0);
  34:   }
  35: 
  36:   // Zero norm
  37:   struct zero {
  38:     static __forceinline__ __device__ void inc(scalar_t& agg, const scalar_t diff, const scalar_t /*p*/) {
  39:       if (diff != diff) { // NaN
  40:         agg = diff;
  41:       } else if (diff != 0.0) {
  42:         agg += 1.0;
  43:       }
  44:     }
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `sign`, `inc`.
- CN: 该代码块定义或继续实现 `sign`, `inc`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 45-47
```cpp
  45:     static __forceinline__ __device__ scalar_t finish(const scalar_t agg, const scalar_t /*p*/) { return agg; }
  46:     static __forceinline__ __device__ void agg(scalar_t& update, const scalar_t other) { update += other; }
  47:   };
```
- EN: This block defines or continues the implementation of `finish`, `agg`.
- CN: 该代码块定义或继续实现 `finish`, `agg`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 49-49
```cpp
  49:   // One norm
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 50-55
```cpp
  50:   struct one {
  51:     static __forceinline__ __device__ void inc(scalar_t& agg, const scalar_t diff, const scalar_t /*p*/) { agg += diff; }
  52:     static __forceinline__ __device__ scalar_t finish(const scalar_t agg, const scalar_t /*p*/) { return agg; }
  53:     static __forceinline__ __device__ void agg(scalar_t& update, const scalar_t other) { update += other; }
  54:     static __forceinline__ __device__ scalar_t backward(const scalar_t diff, const scalar_t grad, const scalar_t /*dist*/, const scalar_t /*p*/) { return grad * sign(diff); }
  55:   };
```
- EN: This block defines or continues the implementation of `inc`, `finish`, `agg`.
- CN: 该代码块定义或继续实现 `inc`, `finish`, `agg`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 57-57
```cpp
  57:   // Special case backward when p is less than two
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 58-62
```cpp
  58:   struct lt_two {
  59:     static __forceinline__ __device__ scalar_t backward(const scalar_t diff, const scalar_t grad, const scalar_t dist, const scalar_t p) {
  60:       return (dist == 0.0 || (diff == 0.0 && p < 1)) ? 0 : (sign(diff) * std::pow(std::abs(diff), p - 1) * grad / std::pow(dist, p - 1));
  61:     }
  62:   };
```
- EN: This block defines or continues the implementation of `backward`.
- CN: 该代码块定义或继续实现 `backward`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 64-64
```cpp
  64:   // Two norm
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 65-70
```cpp
  65:   struct two {
  66:     static __forceinline__ __device__ void inc(scalar_t& agg, const scalar_t diff, const scalar_t /*p*/) { agg += diff * diff; }
  67:     static __forceinline__ __device__ scalar_t finish(const scalar_t agg, const scalar_t /*p*/) { return device_sqrt<scalar_t>(agg); }
  68:     static __forceinline__ __device__ void agg(scalar_t& update, const scalar_t other) { update += other; }
  69:     static __forceinline__ __device__ scalar_t backward(const scalar_t diff, const scalar_t grad, const scalar_t dist, const scalar_t /*p*/) { return dist == 0.0 ? 0 : grad * diff / dist; }
  70:   };
```
- EN: This block defines or continues the implementation of `inc`, `finish`, `agg`.
- CN: 该代码块定义或继续实现 `inc`, `finish`, `agg`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 72-72
```cpp
  72:   // General p norm
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 73-78
```cpp
  73:   struct p {
  74:     static __forceinline__ __device__ void inc(scalar_t& agg, const scalar_t diff, const scalar_t p) { agg += std::pow(diff, p); }
  75:     static __forceinline__ __device__ scalar_t finish(const scalar_t agg, const scalar_t p) { return std::pow(agg, static_cast<scalar_t>(1) / p); }
  76:     static __forceinline__ __device__ void agg(scalar_t& update, const scalar_t other) { update += other; }
  77:     static __forceinline__ __device__ scalar_t backward(const scalar_t diff, const scalar_t grad, const scalar_t dist, const scalar_t p) { return dist == 0.0 ? 0 : diff * std::pow(std::abs(diff), p - 2) * grad / std::pow(dist, p - 1); }
  78:   };
```
- EN: This block defines or continues the implementation of `inc`, `finish`, `agg`.
- CN: 该代码块定义或继续实现 `inc`, `finish`, `agg`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 80-80
```cpp
  80:   // Inf norm
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 81-86
```cpp
  81:   struct inf {
  82:     static __forceinline__ __device__ void inc(scalar_t& agg, const scalar_t diff, const scalar_t /*p*/) { if (diff > agg) { agg = diff; } }
  83:     static __forceinline__ __device__ scalar_t finish(const scalar_t agg, const scalar_t /*p*/) { return agg; }
  84:     static __forceinline__ __device__ void agg(scalar_t& update, const scalar_t other) { if (other > update) { update = other; } }
  85:     static __forceinline__ __device__ scalar_t backward(const scalar_t diff, const scalar_t grad, const scalar_t dist, const scalar_t /*p*/) { return grad * sign(diff) * (std::abs(diff) == dist); }
  86:   };
```
- EN: This block defines or continues the implementation of `inc`, `finish`, `agg`.
- CN: 该代码块定义或继续实现 `inc`, `finish`, `agg`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 88-88
```cpp
  88: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 90-100
```cpp
  90: template <typename scalar_t, typename F>
  91: struct DistReduceOp {
  92:     __forceinline__ __device__ scalar_t combine(scalar_t a, scalar_t b) const {
  93:         F::agg(a, b);
  94:         return a;
  95:     }
  96: 
  97:     __forceinline__ __device__ scalar_t warp_shfl_down(scalar_t data, int offset) const {
  98:         return WARP_SHFL_DOWN(data, offset);
  99:     }
 100: };
```
- EN: This block defines or continues the implementation of `combine`, `warp_shfl_down`.
- CN: 该代码块定义或继续实现 `combine`, `warp_shfl_down`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 102-123
```cpp
 102: template <typename scalar_t, typename F>
 103: __global__ static void pdist_kernel_cuda_impl(scalar_t * result, const scalar_t * self, const int64_t n, const int64_t m, const scalar_t p,
 104:                                               const double n2, const double n2_squared_minus_1) {
 105:   const int64_t k = blockIdx.x;
 106:   const int stride = blockDim.x;
 107: 
 108:   // The -1 accounts for floating point truncation issues
 109:   int64_t i = static_cast<int64_t>((n2 - device_sqrt<double>(n2_squared_minus_1 - 2 * k)));
 110:   int64_t j = k - n * i + i * (i + 1) / 2 + i + 1;
 111: 
 112:   const scalar_t * const start = self + i * m;
 113:   const scalar_t * const end = start + m;
 114:   const scalar_t * a = start + threadIdx.x;
 115:   const scalar_t * b = self + j * m + threadIdx.x;
 116:   scalar_t agg = 0.0;
 117:   for (; a < end; a += stride, b += stride) {
 118:     F::inc(agg, std::abs(*a - *b), p);
 119:   }
 120: 
 121:   __shared__ scalar_t agg_smem[kCUDANumThreads];
 122:   scalar_t agg_init{0.0};
 123:   agg = cuda_utils::BlockReduce(agg, DistReduceOp<scalar_t, F>{}, agg_init, agg_smem);
```
- EN: This block defines GPU kernel entry point(s) `pdist_kernel_cuda_impl`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `pdist_kernel_cuda_impl`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 124-127
```cpp
 124:   if (threadIdx.x == 0) {
 125:     result[k] = F::finish(agg, p);
 126:   }
 127: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 129-150
```cpp
 129: template <typename scalar_t, typename F>
 130: __global__ static void cdist_backward_kernel_cuda_impl(scalar_t * buffer, const scalar_t * grad, const scalar_t * x1, const scalar_t * x2, const scalar_t * dist,
 131:                                                        const scalar_t p, const int64_t r1, const int64_t r2, const int64_t m, const int64_t count, const int64_t r_size, const int64_t l1_size, const int64_t l2_size) {
 132:   const int y = (blockIdx.y * gridDim.z + blockIdx.z) * blockDim.y + threadIdx.y;
 133:   const int init = blockIdx.x * blockDim.x + threadIdx.x;
 134:   if (y >= count || init >= m) {
 135:     return;
 136:   }
 137:   const int l = y / r_size;
 138:   const int k = y % r_size;
 139:   const int stride = blockDim.x * gridDim.x;
 140:   const int l_size = r_size * m;
 141: 
 142:   int64_t i = k / r2;
 143:   int64_t j = k % r2;
 144: 
 145:   const scalar_t grad_k = grad[y];
 146:   const scalar_t dist_k = dist[y];
 147: 
 148:   const scalar_t * const start = x1 + l * l1_size + i * m;
 149:   const scalar_t * const end = start + m;
 150:   const scalar_t * self_i = start + init;
```
- EN: This block defines GPU kernel entry point(s) `cdist_backward_kernel_cuda_impl`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `cdist_backward_kernel_cuda_impl`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 151-151
```cpp
 151:   const scalar_t * self_j = x2 + l * l2_size + j * m + init;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 153-153
```cpp
 153:   scalar_t * buff_i = buffer + l * l_size + (r1 * j + i) * m + init;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 155-159
```cpp
 155:   for (; self_i < end; self_i += stride, self_j += stride, buff_i += stride) {
 156:     const scalar_t res = F::backward(*self_i - *self_j, grad_k, dist_k, p);
 157:     *buff_i = res;
 158:   }
 159: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 161-182
```cpp
 161: template <typename scalar_t, typename F>
 162: __global__ static void pdist_backward_kernel_cuda_impl(scalar_t * buffer, const scalar_t * grad, const scalar_t * self, const scalar_t * dist, int64_t gs, const int64_t n, const int64_t m, const int64_t combs, const scalar_t p,
 163:                                                        const double n2, const double n2_squared_minus_1) {
 164:   const int64_t k = ((int64_t) blockIdx.x) * blockDim.x + threadIdx.x;
 165:   const int init = blockIdx.y * blockDim.y + threadIdx.y;
 166:   const int stride = blockDim.y * gridDim.y;
 167: 
 168:   if (k >= combs) {
 169:     return;
 170:   }
 171: 
 172:   // The -1 accounts for floating point truncation issues
 173:   int64_t i = static_cast<int64_t>((n2 - device_sqrt<double>(n2_squared_minus_1 - 2 * k)));
 174:   int64_t j = k - n * i + i * (i + 1) / 2 + i + 1;
 175:   int64_t ib = j - i - 1;
 176:   int64_t jb = n - 2 - i;
 177: 
 178:   const scalar_t grad_k = grad[k * gs];
 179:   const scalar_t dist_k = dist[k];
 180: 
 181:   const scalar_t * const start = self + i * m;
 182:   const scalar_t * const end = start + m;
```
- EN: This block defines GPU kernel entry point(s) `pdist_backward_kernel_cuda_impl`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `pdist_backward_kernel_cuda_impl`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 183-192
```cpp
 183:   const scalar_t * self_i = start + init;
 184:   const scalar_t * self_j = self + j * m + init;
 185:   scalar_t * buff_i = buffer + (ib * n + i) * m + init;
 186:   scalar_t * buff_j = buffer + (jb * n + j) * m + init;
 187:   for (; self_i < end; self_i += stride, self_j += stride, buff_i += stride, buff_j += stride) {
 188:     const scalar_t res = F::backward(*self_i - *self_j, grad_k, dist_k, p);
 189:     *buff_i = res;
 190:     *buff_j = -res;
 191:   }
 192: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 194-215
```cpp
 194: template <typename scalar_t, typename F>
 195: __global__ static void cdist_kernel_cuda_impl(scalar_t * result, const scalar_t * x1, const scalar_t * x2,
 196:     const scalar_t p, const int64_t r2, const int64_t m, const int64_t r_size, const int64_t l1_size, const int64_t l2_size) {
 197:   const int64_t l = blockIdx.x / r_size;
 198:   const int64_t k = blockIdx.x % r_size;
 199:   const int64_t i = k / r2;
 200:   const int64_t j = k % r2;
 201:   const int stride = blockDim.x;
 202: 
 203:   const scalar_t * const start = x1 + l * l1_size + i * m;
 204:   const scalar_t * const end = start + m;
 205:   const scalar_t * a = start + threadIdx.x;
 206:   const scalar_t * b = x2 + l * l2_size + j * m + threadIdx.x;
 207: 
 208:   scalar_t agg = 0.0;
 209:   for (; a < end; a += stride, b += stride) {
 210:     F::inc(agg, std::abs(*a - *b), p);
 211:   }
 212:   __shared__ scalar_t agg_smem[kCUDANumThreads];
 213:   scalar_t agg_init{0.0};
 214:   agg = cuda_utils::BlockReduce(agg, DistReduceOp<scalar_t, F>{}, agg_init, agg_smem);
 215:   if (threadIdx.x == 0) {
```
- EN: This block defines GPU kernel entry point(s) `cdist_kernel_cuda_impl`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `cdist_kernel_cuda_impl`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 216-218
```cpp
 216:     result[blockIdx.x] = F::finish(agg, p);
 217:   }
 218: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 220-241
```cpp
 220: void cdist_kernel_impl(Tensor& result, const Tensor& x1, const Tensor& x2, double p) {
 221:   const int64_t r1 = x1.size(-2);
 222:   const int64_t r2 = x2.size(-2);
 223:   const int64_t m = x1.size(-1);
 224:   const int64_t r_size = r1 * r2;
 225:   const int64_t l1_size = r1 * m;
 226:   const int64_t l2_size = r2 * m;
 227:   const dim3 grid(result.numel());
 228:   const dim3 block(kCUDANumThreads);
 229: 
 230:   AT_DISPATCH_FLOATING_TYPES(x1.scalar_type(), "cdist_cuda", [&] {
 231:     auto impl_fptr = cdist_kernel_cuda_impl<scalar_t, dists<scalar_t>::p>;
 232:     if (p == 0.0) {
 233:       impl_fptr = cdist_kernel_cuda_impl<scalar_t, dists<scalar_t>::zero>;
 234:     } else if (p == 1.0) {
 235:       impl_fptr = cdist_kernel_cuda_impl<scalar_t, dists<scalar_t>::one>;
 236:     } else if (p == 2.0) {
 237:       impl_fptr = cdist_kernel_cuda_impl<scalar_t, dists<scalar_t>::two>;
 238:     } else if (std::isinf(p)) {
 239:       impl_fptr = cdist_kernel_cuda_impl<scalar_t, dists<scalar_t>::inf>;
 240:     }
 241:     impl_fptr<<<grid, block, 0, at::cuda::getCurrentCUDAStream()>>>(result.mutable_data_ptr<scalar_t>(), x1.const_data_ptr<scalar_t>(), x2.const_data_ptr<scalar_t>(), p, r2, m, r_size, l1_size, l2_size);
```
- EN: This block defines or continues the implementation of `cdist_kernel_impl`.
- CN: 该代码块定义或继续实现 `cdist_kernel_impl`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 242-244
```cpp
 242:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 243:   });
 244: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 246-267
```cpp
 246: void pdist_forward_kernel_impl(Tensor& result, const Tensor& self, double p) {
 247:   const dim3 grid(result.numel());
 248:   const dim3 block(kCUDANumThreads);
 249:   int64_t n = self.size(0);
 250:   int64_t m = self.size(1);
 251:   // https://github.com/pytorch/pytorch/issues/15511 demonstrated we need to do
 252:   // some math in fp64 -- this is just minimizing the amount of fp64 math we do on the device.
 253:   const double n2 = n - .5;
 254:   const double n2_squared_minus_1 = n2 * n2 - 1;
 255: 
 256:   AT_DISPATCH_FLOATING_TYPES(self.scalar_type(), "pdist_cuda", [&] {
 257:     auto impl_fptr = pdist_kernel_cuda_impl<scalar_t, dists<scalar_t>::p>;
 258:     if (p == 0.0) {
 259:       impl_fptr = pdist_kernel_cuda_impl<scalar_t, dists<scalar_t>::zero>;
 260:     } else if (p == 1.0) {
 261:       impl_fptr = pdist_kernel_cuda_impl<scalar_t, dists<scalar_t>::one>;
 262:     } else if (p == 2.0) {
 263:       impl_fptr = pdist_kernel_cuda_impl<scalar_t, dists<scalar_t>::two>;
 264:     } else if (std::isinf(p)) {
 265:       impl_fptr = pdist_kernel_cuda_impl<scalar_t, dists<scalar_t>::inf>;
 266:     }
 267:     impl_fptr<<<grid, block, 0, at::cuda::getCurrentCUDAStream()>>>(result.mutable_data_ptr<scalar_t>(), self.const_data_ptr<scalar_t>(), n, m, p, n2, n2_squared_minus_1);
```
- EN: This block defines or continues the implementation of `pdist_forward_kernel_impl`.
- CN: 该代码块定义或继续实现 `pdist_forward_kernel_impl`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 268-270
```cpp
 268:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 269:   });
 270: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 272-293
```cpp
 272: void pdist_backward_kernel_impl(Tensor& result, const Tensor& grad, const Tensor& self, const double p, const Tensor& dist) {
 273:   if (p == 0.0 || grad.numel() == 0 || self.numel() == 0) {
 274:     result.fill_(0);
 275:     return;
 276:   }
 277: 
 278:   const int64_t n = result.size(0);
 279:   int64_t m = self.size(1);
 280:   const int block_x = 16;
 281:   // NB: be careful with changing block_y; as it's currently written, grid_y is limited to be 2^16.
 282:   // block_y of 64 gives us max pdist dim1 of 2**24
 283:   const int block_y = 64;
 284:   const int grid_x = (dist.numel() + block_x - 1) / block_x;
 285:   const int grid_y = (m + block_y * 8 - 1) / (block_y * 8);
 286:   const dim3 grid(grid_x, grid_y);
 287:   const dim3 block(block_x, block_y);
 288:   // https://github.com/pytorch/pytorch/issues/15511 demonstrated we need to do
 289:   // some math in fp64 -- this is just minimizing the amount of fp64 math we do on the device.
 290:   const double n2 = n - .5;
 291:   const double n2_squared_minus_1 = n2 * n2 - 1;
 292: 
 293:   Tensor buffer = at::empty({n - 1, result.size(0), result.size(1)}, result.options());
```
- EN: This block defines or continues the implementation of `pdist_backward_kernel_impl`.
- CN: 该代码块定义或继续实现 `pdist_backward_kernel_impl`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 294-307
```cpp
 294:   AT_DISPATCH_FLOATING_TYPES(self.scalar_type(), "pdist_cuda_backward", [&] {
 295:     auto impl_fptr = pdist_backward_kernel_cuda_impl<scalar_t, dists<scalar_t>::p>;
 296:     if (p == 1.0) {
 297:       impl_fptr = pdist_backward_kernel_cuda_impl<scalar_t, dists<scalar_t>::one>;
 298:     } else if (p < 2.0) {
 299:       impl_fptr = pdist_backward_kernel_cuda_impl<scalar_t, dists<scalar_t>::lt_two>;
 300:     } else if (p == 2.0) {
 301:       impl_fptr = pdist_backward_kernel_cuda_impl<scalar_t, dists<scalar_t>::two>;
 302:     } else if (std::isinf(p)) {
 303:       impl_fptr = pdist_backward_kernel_cuda_impl<scalar_t, dists<scalar_t>::inf>;
 304:     }
 305:     impl_fptr<<<grid, block, 0, at::cuda::getCurrentCUDAStream()>>>(buffer.mutable_data_ptr<scalar_t>(), grad.const_data_ptr<scalar_t>(), self.const_data_ptr<scalar_t>(), dist.const_data_ptr<scalar_t>(), grad.stride(0), n, m, dist.numel(), p, n2, n2_squared_minus_1);
 306:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 307:   });
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 309-310
```cpp
 309:   at::sum_out(result, buffer, 0);
 310: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 312-333
```cpp
 312: void cdist_backward_kernel_impl(Tensor& result, const Tensor& grad, const Tensor& x1, const Tensor& x2, const double p, const Tensor& dist) {
 313:   if (p == 0.0 || grad.numel() == 0 || x1.numel() == 0 || x2.numel() == 0) {
 314:     result.fill_(0);
 315:     return;
 316:   }
 317: 
 318:   const int64_t r1 = x1.size(-2);
 319:   const int64_t r2 = x2.size(-2);
 320:   const int64_t m = x1.size(-1);
 321:   // Just like we do in the CPU code, assume that result is always batched
 322:   int64_t batch = result.size(0);
 323:   const int block_x = 64;
 324:   const int block_y = 16;
 325:   const int grid_x = (m + block_x * 8 - 1) / (block_x * 8);
 326: 
 327:   const int64_t count = dist.numel();
 328:   const int64_t grid_temp = (count + block_y - 1) / block_y;
 329: 
 330:   const int grid_y = (grid_temp - 1) / 65535 + 1;
 331:   const int grid_z = (grid_temp - 1) / grid_y + 1;
 332: 
 333:   const dim3 grid(grid_x, grid_y, grid_z);
```
- EN: This block defines or continues the implementation of `cdist_backward_kernel_impl`.
- CN: 该代码块定义或继续实现 `cdist_backward_kernel_impl`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 334-334
```cpp
 334:   const dim3 block(block_x, block_y);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 336-340
```cpp
 336:   const int64_t r_size = r1 * r2;
 337:   const int64_t l1_size = r1 * m;
 338:   const int64_t l2_size = r2 * m;
 339:   //current implementation supports only gradient that can be collapsed to 1D. However, to avoid checking this assumption,
 340:   //we call grad.contiguous() before backward, so stride is guaranteed to be 1
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 342-358
```cpp
 342:   Tensor buffer = at::empty({batch, r2, r1, m}, result.options());
 343:   AT_DISPATCH_FLOATING_TYPES(result.scalar_type(), "cdist_cuda_backward", [&] {
 344:     auto impl_fptr = cdist_backward_kernel_cuda_impl<scalar_t, dists<scalar_t>::p>;
 345:     if (p == 1.0) {
 346:       impl_fptr = cdist_backward_kernel_cuda_impl<scalar_t, dists<scalar_t>::one>;
 347:     } else if (p < 2.0) {
 348:        impl_fptr = cdist_backward_kernel_cuda_impl<scalar_t, dists<scalar_t>::lt_two>;
 349:     } else if (p == 2.0) {
 350:        impl_fptr = cdist_backward_kernel_cuda_impl<scalar_t, dists<scalar_t>::two>;
 351:     } else if (std::isinf(p)) {
 352:        impl_fptr = cdist_backward_kernel_cuda_impl<scalar_t, dists<scalar_t>::inf>;
 353:     }
 354:     impl_fptr<<<grid, block, 0, at::cuda::getCurrentCUDAStream()>>>(buffer.mutable_data_ptr<scalar_t>(),
 355:       grad.const_data_ptr<scalar_t>(), x1.const_data_ptr<scalar_t>(), x2.const_data_ptr<scalar_t>(), dist.const_data_ptr<scalar_t>(),
 356:       p, r1, r2, m, count, r_size, l1_size, l2_size);
 357:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 358:   });
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 360-360
```cpp
 360:   at::sum_out(result, buffer, 1);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 362-362
```cpp
 362: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 365-365
```cpp
 365: } // anonymous namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 367-370
```cpp
 367: REGISTER_DISPATCH(pdist_forward_stub, &pdist_forward_kernel_impl)
 368: REGISTER_DISPATCH(pdist_backward_stub, &pdist_backward_kernel_impl)
 369: REGISTER_DISPATCH(cdist_stub, &cdist_kernel_impl)
 370: REGISTER_DISPATCH(cdist_backward_stub, &cdist_backward_kernel_impl)
```
- EN: The registration macro binds this implementation into PyTorch dispatch so higher-level operators can call the CUDA specialization.
- CN: 注册宏把该实现接入 PyTorch 分发系统，使上层算子能够调用这个 CUDA 特化版本。

### Lines 372-372
```cpp
 372: } // at::native
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- Dispatch registration exposes the CUDA specialization to higher PyTorch layers. / 分发注册把 CUDA 特化实现暴露给更高层的 PyTorch 调用链。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/core/Tensor.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/cuda/Exceptions.h>`
  - `<ATen/cuda/DeviceUtils.cuh>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<math.h>`
  - `<ATen/native/cuda/block_reduce.cuh>`
  - `<ATen/native/cuda/DeviceSqrt.cuh>`
  - `<ATen/native/Distance.h>`
  - `<ATen/NumericUtils.h>`
  - `<ATen/Functions.h>`
  - `<ATen/ops/empty.h>`
- Runtime symbols / 运行时符号:
  - `pdist_forward_stub`
  - `pdist_backward_stub`
  - `cdist_stub`
  - `cdist_backward_stub`
  - `AT_DISPATCH_FLOATING_TYPES`
  - `REGISTER_DISPATCH`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
