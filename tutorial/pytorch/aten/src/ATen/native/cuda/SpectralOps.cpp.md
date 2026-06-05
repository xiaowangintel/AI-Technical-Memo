# SpectralOps.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/SpectralOps.cpp`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Provides host-side CUDA entry points, orchestration, or dispatch glue around `exec_cufft_plan`, `cufft_get_plan_cache_max_size_impl`, `cufft_set_plan_cache_max_size_impl`, `cufft_get_plan_cache_size_impl`.
- 用途（中文）: 提供围绕 `exec_cufft_plan`, `cufft_get_plan_cache_max_size_impl`, `cufft_set_plan_cache_max_size_impl`, `cufft_get_plan_cache_size_impl` 的主机端 CUDA 入口、调度编排或分发胶水代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/cuda/CUDAContext.h>
   4: #include <ATen/Config.h>
   5: #include <ATen/Dispatch.h>
   6: #include <ATen/ScalarOps.h>
   7: #include <ATen/TensorIterator.h>
   8: #include <ATen/detail/CUDAHooksInterface.h>
   9: #include <ATen/native/Resize.h>
  10: #include <ATen/native/SpectralOpsUtils.h>
  11: #include <ATen/native/cuda/CuFFTUtils.h>
  12: #include <ATen/native/cuda/CuFFTPlanCache.h>
  13: #include <ATen/cuda/nvrtc_stub/ATenNVRTC.h>
  14: #include <c10/util/irange.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/cuda/CUDAContext.h>`, `<ATen/Config.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/cuda/CUDAContext.h>`, `<ATen/Config.h>`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 15-28
```cpp
  15: 
  16: #ifndef AT_PER_OPERATOR_HEADERS
  17: #include <ATen/Functions.h>
  18: #include <ATen/NativeFunctions.h>
  19: #else
  20: #include <ATen/ops/_fft_c2c_native.h>
  21: #include <ATen/ops/_fft_c2r_native.h>
  22: #include <ATen/ops/_fft_r2c_native.h>
  23: #include <ATen/ops/empty.h>
  24: #include <ATen/ops/mul.h>
  25: #endif
  26: 
  27: #include <cufft.h>
  28: #include <cufftXt.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/_fft_c2c_native.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/_fft_c2c_native.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 29-30
```cpp
  29: 
  30: #include <cmath>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<cmath>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<cmath>`。

### Lines 33-54
```cpp
  33: namespace at::native {
  34: 
  35: using namespace at::native::detail;
  36: 
  37: // Execute a pre-planned transform
  38: static void exec_cufft_plan(
  39:     const CuFFTConfig &config, void* in_data, void* out_data, bool forward) {
  40:   auto& plan = config.plan();
  41:   CUFFT_CHECK(cufftXtExec(plan, in_data, out_data,
  42:                           forward ? CUFFT_FORWARD : CUFFT_INVERSE));
  43: }
  44: 
  45: 
  46: // NOTE [ cuFFT Embedded Strides ]
  47: //
  48: // cuFFT supports a subset of arbitrary strides via their "advanced data layout"
  49: // option (http://docs.nvidia.com/cuda/cufft/index.html#advanced-data-layout).
  50: // Specifically, these are tensors that can be viewed as subtensors resulted
  51: // from slicing a larger contiguous tensors. For such input tensors, let the
  52: // sizes of the enclosing tensor be `inembed`, and we can have in 3d case:
  53: //
  54: //     input[x, y, z] = input[((x * inembed[1] + y) * inembed[2] + z)]
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `exec_cufft_plan`.
- CN: 该代码块定义或继续实现 `exec_cufft_plan`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 55-68
```cpp
  55: //
  56: // Above is the simplified formula ignoring the batch dimension. In fact, the
  57: // last dimension of the enclosing tensor doesn't have to be contiguous, i.e.,
  58: // it can be greater than 1. Then one can set the base stride for the enclosing
  59: // tensor with `istride`. Then we have
  60: //
  61: //     input[x, y, z] = input[((x * inembed[1] + y) * inembed[2] + z) * istride]
  62: //
  63: // For example, consider
  64: //
  65: //     enclosing = torch.zeros(6, 8, 10)  # contiguous
  66: //     input = enclosing[:4, 2:6, 6:]
  67: //     input.size()                       # [ 4,  4,  4]
  68: //     input.stride()                     # [80, 10,  1]
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 69-82
```cpp
  69: //     # inembed = [6, 8, 10]
  70: //     input[2, 1, 3] = input[((2 * 8) + 1) * 10 + 3]   # using above formula
  71: //                    = input[173]
  72: //                    = input[2 * 80 + 1 * 10 + 1 * 3]  # using strides directly
  73: //
  74: // Generally, the embedded strides can be computed as
  75: //
  76: //     embed[i] = stride[i - 1] / stride[i].
  77: //
  78: // Note that the value of embed[0] isn't used to compute indices and doesn't
  79: // matter.
  80: //
  81: // Contrary to advanced data layout, simple layout means that *embeds have
  82: // unit-strides. In particular, unit-stride refers to that the input and output
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 83-87
```cpp
  83: // tensors being contiguous, and that the strides at the innermost signal
  84: // dimension being unit (1) w.r.t. the corresponding data type.
  85: 
  86: // The cuFFT plan cache
  87: // unique_ptr for nullability and to avoid reference invalidation on vector resize
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 88-89
```cpp
  88: static std::vector<std::unique_ptr<CuFFTParamsLRUCache>> plan_caches;
  89: static std::mutex plan_caches_mutex;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 91-106
```cpp
  91: static inline
  92: CuFFTParamsLRUCache &cufft_get_plan_cache(DeviceIndex device_index) {
  93:   std::lock_guard<std::mutex> guard(plan_caches_mutex);
  94: 
  95:   AT_ASSERT(device_index >= 0);
  96: 
  97:   if (device_index >= static_cast<int64_t>(plan_caches.size())) {
  98:     plan_caches.resize(device_index + 1);
  99:   }
 100: 
 101:   if (!plan_caches[device_index]) {
 102:     plan_caches[device_index] = std::make_unique<CuFFTParamsLRUCache>();
 103:   }
 104: 
 105:   return *plan_caches[device_index];
 106: }
```
- EN: This block defines or continues the implementation of `cufft_get_plan_cache`.
- CN: 该代码块定义或继续实现 `cufft_get_plan_cache`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 109-130
```cpp
 109: namespace detail {
 110: 
 111: int64_t cufft_get_plan_cache_max_size_impl(DeviceIndex device_index) {
 112:   TORCH_CHECK(0 <= device_index && device_index < at::detail::getCUDAHooks().deviceCount(),
 113:     "cufft_get_plan_cache_max_size: expected 0 <= device_index < ",
 114:     at::detail::getCUDAHooks().deviceCount(), "], but got device_index=",
 115:     device_index);
 116:   return cufft_get_plan_cache(device_index).max_size();
 117: }
 118: 
 119: void cufft_set_plan_cache_max_size_impl(DeviceIndex device_index, int64_t max_size) {
 120:   TORCH_CHECK(0 <= device_index && device_index < at::detail::getCUDAHooks().deviceCount(),
 121:     "cufft_set_plan_cache_max_size: expected 0 <= device_index < ",
 122:     at::detail::getCUDAHooks().deviceCount(), "], but got device_index=",
 123:     device_index);
 124:   cufft_get_plan_cache(device_index).resize(max_size);
 125: }
 126: 
 127: int64_t cufft_get_plan_cache_size_impl(DeviceIndex device_index) {
 128:   TORCH_CHECK(0 <= device_index && device_index < at::detail::getCUDAHooks().deviceCount(),
 129:     "cufft_get_plan_cache_size: expected 0 <= device_index < ",
 130:     at::detail::getCUDAHooks().deviceCount(), "], but got device_index=",
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `cufft_get_plan_cache_max_size_impl`, `cufft_set_plan_cache_max_size_impl`, `cufft_get_plan_cache_size_impl`.
- CN: 该代码块定义或继续实现 `cufft_get_plan_cache_max_size_impl`, `cufft_set_plan_cache_max_size_impl`, `cufft_get_plan_cache_size_impl`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 131-133
```cpp
 131:     device_index);
 132:   return cufft_get_plan_cache(device_index).size();
 133: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 135-141
```cpp
 135: void cufft_clear_plan_cache_impl(DeviceIndex device_index) {
 136:   TORCH_CHECK(0 <= device_index && device_index < at::detail::getCUDAHooks().deviceCount(),
 137:     "cufft_clear_plan_cache: expected 0 <= device_index < ",
 138:     at::detail::getCUDAHooks().deviceCount(), "], but got device_index=",
 139:     device_index);
 140:   cufft_get_plan_cache(device_index).clear();
 141: }
```
- EN: This block defines or continues the implementation of `cufft_clear_plan_cache_impl`.
- CN: 该代码块定义或继续实现 `cufft_clear_plan_cache_impl`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 143-143
```cpp
 143: } // namespace at::native::detail
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 145-166
```cpp
 145: namespace {
 146: constexpr int64_t cufft_max_ndim = 3;
 147: 
 148: // "Large" here means a prime factor not special-cased by cuFFT
 149: // Ref: https://docs.nvidia.com/cuda/cufft/index.html#accuracy-and-performance
 150: bool has_large_prime_factor(int64_t n) {
 151:   constexpr int64_t first_large_prime = 11;
 152:   const std::array<int64_t, 4> prime_radices{{2, 3, 5, 7}};
 153:   for (auto prime : prime_radices) {
 154:     if (n < first_large_prime) {
 155:         return false;
 156:     }
 157: 
 158:     while (n % prime == 0) {
 159:       n /= prime;
 160:     }
 161:   }
 162:   return n != 1;
 163: }
 164: 
 165: // Execute a general fft operation (can be c2c, onesided r2c or onesided c2r)
 166: const Tensor& _exec_fft(Tensor& out, const Tensor& self, IntArrayRef out_sizes,
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `has_large_prime_factor`.
- CN: 该代码块定义或继续实现 `has_large_prime_factor`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 167-187
```cpp
 167:                          IntArrayRef dim, bool forward) {
 168:   const auto ndim = self.dim();
 169:   const int64_t signal_ndim = dim.size();
 170:   const auto batch_dims = ndim - signal_ndim;
 171: 
 172:   // Permute dimensions so batch dimensions come first, and in stride order
 173:   // This maximizes data locality when collapsing to a single batch dimension
 174:   DimVector dim_permute(ndim);
 175:   std::iota(dim_permute.begin(), dim_permute.end(), int64_t{0});
 176: 
 177:   c10::SmallVector<bool, kDimVectorStaticSize> is_transformed_dim(ndim);
 178:   for (const auto& d : dim) {
 179:     is_transformed_dim[d] = true;
 180:   }
 181:   auto batch_end = std::partition(dim_permute.begin(), dim_permute.end(),
 182:                                   [&](int64_t d) {return !is_transformed_dim[d]; });
 183:   auto self_strides = self.strides();
 184:   std::sort(dim_permute.begin(), batch_end,
 185:             [&](int64_t a, int64_t b) { return self_strides[a] > self_strides[b]; });
 186:   std::copy(dim.cbegin(), dim.cend(), batch_end);
 187:   auto input = self.permute(dim_permute);
```
- EN: This block defines or continues the implementation of `partition`, `sort`.
- CN: 该代码块定义或继续实现 `partition`, `sort`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 189-189
```cpp
 189:   // Collapse batch dimensions into a single dimension
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 190-193
```cpp
 190:   DimVector batched_sizes(signal_ndim + 1);
 191:   batched_sizes[0] = -1;
 192:   std::copy(input.sizes().cbegin() + batch_dims, input.sizes().cend(), batched_sizes.begin() + 1);
 193:   input = input.reshape(batched_sizes);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 195-206
```cpp
 195:   const auto batch_size = input.sizes()[0];
 196:   DimVector signal_size(signal_ndim + 1);
 197:   signal_size[0] = batch_size;
 198:   for (const auto i : c10::irange(signal_ndim)) {
 199:     auto in_size = input.sizes()[i + 1];
 200:     auto out_size = out_sizes[dim[i]];
 201:     signal_size[i + 1] = std::max(in_size, out_size);
 202:     TORCH_INTERNAL_ASSERT(in_size == signal_size[i + 1] ||
 203:                           in_size == (signal_size[i + 1] / 2) + 1);
 204:     TORCH_INTERNAL_ASSERT(out_size == signal_size[i + 1] ||
 205:                           out_size == (signal_size[i + 1] / 2) + 1);
 206:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 208-213
```cpp
 208:   batched_sizes[0] = batch_size;
 209:   DimVector batched_out_sizes(batched_sizes.begin(), batched_sizes.end());
 210:   for (const auto i : c10::irange(dim.size())) {
 211:     batched_out_sizes[i + 1] = out_sizes[dim[i]];
 212:   }
 213:   out.resize_(batched_out_sizes, MemoryFormat::Contiguous);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 215-215
```cpp
 215:   // Create the transform plan (either from cache or locally)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 216-222
```cpp
 216:   const auto value_type = c10::toRealValueType(input.scalar_type());
 217:   auto fft_type = GetCuFFTTransformType(input.is_complex(), out.is_complex());
 218:   CuFFTParams Params(input.strides(), out.strides(), signal_size, fft_type, value_type);
 219:   CuFFTParamsLRUCache& plan_cache = cufft_get_plan_cache(input.device().index());
 220:   std::unique_lock<std::mutex> guard(plan_cache.mutex, std::defer_lock);
 221:   std::optional<CuFFTConfig> uncached_plan;
 222:   const CuFFTConfig * config = nullptr;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 224-225
```cpp
 224:   // Bluestein's algorithm is only used when a size has large prime factors,
 225:   // sizes with only small prime factors can still be cached
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 226-231
```cpp
 226:   if (plan_cache.max_size() > 0) {
 227:     guard.lock();
 228:     if (plan_cache.max_size() > 0) {  // check again after acquiring the lock
 229:       config = &plan_cache.lookup(Params);
 230:     }
 231:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 233-236
```cpp
 233:   if (config == nullptr) {
 234:     uncached_plan.emplace(Params);
 235:     config = &uncached_plan.value();
 236:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 238-238
```cpp
 238:   auto & plan = config->plan();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 240-242
```cpp
 240:   if (config->should_clone_input()) {
 241:     input = input.clone(MemoryFormat::Contiguous);
 242:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 244-244
```cpp
 244:   // prepare cufft for execution
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 245-247
```cpp
 245:   CUFFT_CHECK(cufftSetStream(plan, at::cuda::getCurrentCUDAStream()));
 246:   auto workspace = at::empty({ config->workspace_size() }, at::device(at::kCUDA).dtype(at::kByte));
 247:   CUFFT_CHECK(cufftSetWorkArea(plan, workspace.mutable_data_ptr()));
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 249-249
```cpp
 249:   // execute transform plan
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 250-250
```cpp
 250: #if !defined(USE_ROCM)
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 251-261
```cpp
 251:   CUcontext pctx = nullptr;
 252:   at::globalContext().getNVRTC().cuCtxGetCurrent(&pctx);
 253:   if (C10_UNLIKELY(!pctx)) {
 254:     // workaround for corner case where a primary context exists but is not
 255:     // the current context
 256:     TORCH_WARN_ONCE("Attempting to run cuFFT, but there was no current CUDA context! Attempting to set the primary context...");
 257:     at::globalContext().getNVRTC().cuDevicePrimaryCtxRetain(&pctx, 0);
 258:     at::globalContext().getNVRTC().cuCtxSetCurrent(pctx);
 259:   }
 260: #endif /* !defined(USE_ROCM) */
 261:   exec_cufft_plan(*config, const_cast<void*>(input.const_data_ptr()), out.data_ptr(), forward);
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 263-263
```cpp
 263:   // Inplace reshaping to original batch shape and inverting the dimension permutation
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 264-274
```cpp
 264:   DimVector out_strides(ndim);
 265:   int64_t batch_numel = 1;
 266:   for (int64_t i = batch_dims - 1; i >= 0; --i) {
 267:     out_strides[dim_permute[i]] = batch_numel * out.strides()[0];
 268:     batch_numel *= out_sizes[dim_permute[i]];
 269:   }
 270:   for (const auto i : c10::irange(batch_dims, ndim)) {
 271:     out_strides[dim_permute[i]] = out.strides()[1 + (i - batch_dims)];
 272:   }
 273:   return out.as_strided_(out_sizes, out_strides, out.storage_offset());
 274: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 276-277
```cpp
 276: // Calculates the normalization constant and applies it in-place to self
 277: // sizes is the sizes of a twosided tensor and dims are all transformed dims
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 278-291
```cpp
 278: double _fft_normalization_scale(int64_t normalization, IntArrayRef sizes, IntArrayRef dims) {
 279:   auto norm = static_cast<fft_norm_mode>(normalization);
 280:   if (norm == fft_norm_mode::none) {
 281:     return 1.0;
 282:   }
 283: 
 284:   int64_t signal_numel = 1;
 285:   for (auto dim : dims) {
 286:     signal_numel *= sizes[dim];
 287:   }
 288:   const double scale_denom = (norm == fft_norm_mode::by_root_n) ?
 289:     std::sqrt(signal_numel) : static_cast<double>(signal_numel);
 290:   return 1.0 / scale_denom;
 291: }
```
- EN: This block defines or continues the implementation of `_fft_normalization_scale`.
- CN: 该代码块定义或继续实现 `_fft_normalization_scale`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 293-296
```cpp
 293: const Tensor& _fft_apply_normalization(const Tensor& self, int64_t normalization, IntArrayRef sizes, IntArrayRef dims) {
 294:   auto scale = _fft_normalization_scale(normalization, sizes, dims);
 295:   return (scale == 1.0) ? self : self.mul_(scale);
 296: }
```
- EN: This block defines or continues the implementation of `_fft_apply_normalization`.
- CN: 该代码块定义或继续实现 `_fft_apply_normalization`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 298-301
```cpp
 298: Tensor& _fft_apply_normalization_out(Tensor& out, const Tensor& self, int64_t normalization, IntArrayRef sizes, IntArrayRef dims) {
 299:   auto scale = _fft_normalization_scale(normalization, sizes, dims);
 300:   return at::mul_out(out, self, c10::scalar_to_tensor(scale));
 301: }
```
- EN: This block defines or continues the implementation of `_fft_apply_normalization_out`.
- CN: 该代码块定义或继续实现 `_fft_apply_normalization_out`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 303-303
```cpp
 303: }  // namespace (anonymous)
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 305-305
```cpp
 305: // Use the optimized path to perform single R2C or C2R if transformation dim is supported by cuFFT
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 306-315
```cpp
 306: bool use_optimized_cufft_path(IntArrayRef dim) {
 307:   // For performance reason, when dim starts with (0, 1), do not use the optimized path.
 308:   if (dim.size() > cufft_max_ndim || (
 309:     dim.size() >= 2 && dim[0] == 0 && dim[1] == 1
 310:   )) {
 311:     return false;
 312:   } else {
 313:     return true;
 314:   }
 315: }
```
- EN: This block defines or continues the implementation of `use_optimized_cufft_path`, `with`.
- CN: 该代码块定义或继续实现 `use_optimized_cufft_path`, `with`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 317-317
```cpp
 317: // n-dimensional real to complex FFT
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 318-339
```cpp
 318: Tensor _fft_r2c_cufft(const Tensor& self, IntArrayRef dim, int64_t normalization, bool onesided) {
 319:   TORCH_CHECK(self.is_floating_point());
 320:   auto input_sizes = self.sizes();
 321:   DimVector onesided_sizes(input_sizes.begin(), input_sizes.end());
 322:   auto last_dim = dim.back();
 323:   auto last_dim_halfsize = (input_sizes[last_dim]) / 2 + 1;
 324:   onesided_sizes[last_dim] = last_dim_halfsize;
 325:   IntArrayRef out_sizes = onesided ? onesided_sizes : input_sizes;
 326: 
 327:   const auto out_options = self.options().dtype(c10::toComplexType(self.scalar_type()));
 328:   auto output = at::empty(out_sizes, out_options);
 329: 
 330:   // CuFFT requires real input to be over-aligned, as if it were complex
 331:   const auto complex_size = 2 * self.element_size();
 332:   const bool complex_aligned = (
 333:       reinterpret_cast<std::uintptr_t>(self.const_data_ptr()) % complex_size == 0);
 334:   auto working_tensor = self;
 335:   if (!complex_aligned) {
 336:     working_tensor = self.movedim(last_dim, -1)
 337:                          .clone(MemoryFormat::Contiguous)
 338:                          .movedim(-1, last_dim);
 339:   }
```
- EN: This block defines or continues the implementation of `_fft_r2c_cufft`.
- CN: 该代码块定义或继续实现 `_fft_r2c_cufft`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 341-361
```cpp
 341:   if (use_optimized_cufft_path(dim)) {
 342:     _exec_fft(output, working_tensor, out_sizes, dim, /*forward=*/true);
 343:   } else {
 344:     // First do the R2C transform on the last dimension
 345:     {
 346:       auto target_sizes = dim.size() == 1 ? out_sizes : onesided_sizes;
 347:       _exec_fft(output, working_tensor, target_sizes, last_dim, /*forward=*/true);
 348:       if (dim.size() > 1) {
 349:         working_tensor = at::empty(out_sizes, out_options);
 350:       }
 351:     }
 352: 
 353:     // Then any remaining C2C transforms
 354:     DimVector sorted_dims(dim.begin(), dim.end() - 1);
 355:     while (!sorted_dims.empty()) {
 356:       std::swap(output, working_tensor);
 357: 
 358:       // Resort dimensions every time as _exec_fft re-strides the output
 359:       auto strides = working_tensor.strides();
 360:       std::sort(sorted_dims.begin(), sorted_dims.end(),
 361:                 [&](int64_t a, int64_t b) { return strides[a] > strides[b]; });
```
- EN: This block defines or continues the implementation of `sort`.
- CN: 该代码块定义或继续实现 `sort`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 363-364
```cpp
 363:       const auto max_dims = std::min(static_cast<size_t>(cufft_max_ndim), sorted_dims.size());
 364:       auto last_dims = IntArrayRef(sorted_dims).slice(sorted_dims.size() - max_dims, max_dims);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 366-366
```cpp
 366:       // Intermediate results are always onesided
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 367-370
```cpp
 367:       _exec_fft(output, working_tensor, onesided_sizes, last_dims, /*forward=*/true);
 368:       sorted_dims.resize(sorted_dims.size() - max_dims);
 369:     }
 370:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 372-372
```cpp
 372:   // Only need to normalize the onesided slice since data in the other half is overwritten
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 373-374
```cpp
 373:   auto out_slice = output.slice(last_dim, 0, last_dim_halfsize);
 374:   _fft_apply_normalization(out_slice, normalization, input_sizes, dim);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 376-385
```cpp
 376:   if (!onesided) {
 377:     if (output.sizes()[last_dim] != out_sizes[last_dim]) {
 378:       working_tensor.resize_(out_sizes, MemoryFormat::Contiguous);
 379:       working_tensor.slice(last_dim, 0, last_dim_halfsize).copy_(output);
 380:       output = std::move(working_tensor);
 381:     }
 382:     at::native::_fft_fill_with_conjugate_symmetry_(output, dim);
 383:   }
 384:   return output;
 385: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 387-402
```cpp
 387: Tensor& _fft_r2c_cufft_out(const Tensor& self, IntArrayRef dim,
 388:                            int64_t normalization, bool onesided, Tensor& out) {
 389:   auto result = _fft_r2c_cufft(self, dim, static_cast<int64_t>(fft_norm_mode::none), /*onesided=*/true);
 390:   if (onesided) {
 391:     return _fft_apply_normalization_out(out, result, normalization, self.sizes(), dim);
 392:   }
 393: 
 394:   resize_output(out, self.sizes());
 395: 
 396:   auto last_dim = dim.back();
 397:   auto last_dim_halfsize = result.sizes()[last_dim];
 398:   auto out_slice = out.slice(last_dim, 0, last_dim_halfsize);
 399:   _fft_apply_normalization_out(out_slice, result, normalization, self.sizes(), dim);
 400:   at::native::_fft_fill_with_conjugate_symmetry_(out, dim);
 401:   return out;
 402: }
```
- EN: This block defines or continues the implementation of `_fft_r2c_cufft_out`.
- CN: 该代码块定义或继续实现 `_fft_r2c_cufft_out`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 404-404
```cpp
 404: // n-dimensional complex to real IFFT
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 405-426
```cpp
 405: Tensor _fft_c2r_cufft(const Tensor& self, IntArrayRef dim, int64_t normalization, int64_t lastdim) {
 406:   TORCH_CHECK(self.is_complex());
 407:   auto in_sizes = self.sizes();
 408:   DimVector out_sizes(in_sizes.begin(), in_sizes.end());
 409:   out_sizes[dim.back()] = lastdim;
 410: 
 411:   auto output = at::empty(out_sizes, self.options().dtype(c10::toRealValueType(self.scalar_type())));
 412: 
 413:   if (use_optimized_cufft_path(dim)) {
 414:     Tensor temp;
 415:     // Complex to real FFTs may overwrite the input buffer, so must always clone (gh-34551)
 416:     temp = self.clone(MemoryFormat::Contiguous);
 417:     _exec_fft(output, temp, out_sizes, dim, /*forward=*/false);
 418:   } else {
 419:     // First complete any C2C transforms
 420:     Tensor temp;
 421:     if (dim.size() > 1) {
 422:       temp = _fft_c2c_cufft(
 423:           self, dim.slice(0, dim.size() - 1),
 424:           static_cast<int64_t>(fft_norm_mode::none), /*forward=*/false);
 425:     } else {
 426:       // Complex to real FFTs may overwrite the input buffer, so must always clone (gh-34551)
```
- EN: This block defines or continues the implementation of `_fft_c2r_cufft`.
- CN: 该代码块定义或继续实现 `_fft_c2r_cufft`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 427-428
```cpp
 427:       temp = self.clone(MemoryFormat::Contiguous);
 428:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 430-431
```cpp
 430:     // Finally, do a 1D C2R transform
 431:     // TODO: could transform up to 2 other dims in the same cuFFT operation
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 432-433
```cpp
 432:     _exec_fft(output, temp, out_sizes, dim.back(), /*forward=*/false);
 433:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 435-436
```cpp
 435:   return _fft_apply_normalization(output, normalization, out_sizes, dim);
 436: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 438-442
```cpp
 438: Tensor& _fft_c2r_cufft_out(const Tensor& self, IntArrayRef dim,
 439:                            int64_t normalization, int64_t lastdim, Tensor& out) {
 440:   auto result = _fft_c2r_cufft(self, dim, static_cast<int64_t>(fft_norm_mode::none), lastdim);
 441:   return _fft_apply_normalization_out(out, result, normalization, result.sizes(), dim);
 442: }
```
- EN: This block defines or continues the implementation of `_fft_c2r_cufft_out`.
- CN: 该代码块定义或继续实现 `_fft_c2r_cufft_out`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 444-444
```cpp
 444: // n-dimensional complex to complex FFT/IFFT
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 445-466
```cpp
 445: Tensor _fft_c2c_cufft(const Tensor& self, IntArrayRef dim, int64_t normalization, bool forward) {
 446:   TORCH_CHECK(self.is_complex());
 447:   if (dim.empty()) {
 448:     return self.clone();
 449:   }
 450: 
 451:   auto out_sizes = self.sizes();
 452:   auto output = at::empty(out_sizes, self.options());
 453: 
 454:   // Perform any number of C2C transforms
 455:   DimVector sorted_dims(dim.begin(), dim.end());
 456:   auto working_tensor = self;
 457:   while (true) {
 458:     // Sort dimensions every time as _exec_fft re-strides the output
 459:     auto strides = working_tensor.strides();
 460:     std::sort(sorted_dims.begin(), sorted_dims.end(),
 461:               [&](int64_t a, int64_t b) { return strides[a] > strides[b]; });
 462: 
 463:     const auto max_dims = std::min(static_cast<size_t>(cufft_max_ndim), sorted_dims.size());
 464:     auto first_dims = IntArrayRef(sorted_dims).slice(sorted_dims.size() - max_dims, max_dims);
 465: 
 466:     _exec_fft(output, working_tensor, out_sizes, first_dims, forward);
```
- EN: This block defines or continues the implementation of `_fft_c2c_cufft`, `sort`.
- CN: 该代码块定义或继续实现 `_fft_c2c_cufft`, `sort`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 467-467
```cpp
 467:     sorted_dims.resize(sorted_dims.size() - max_dims);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 469-471
```cpp
 469:     if (sorted_dims.empty()) {
 470:       break;
 471:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 473-479
```cpp
 473:     if (working_tensor.is_same(self)) {
 474:       working_tensor = std::move(output);
 475:       output = at::empty(out_sizes, self.options());
 476:     } else {
 477:       std::swap(output, working_tensor);
 478:     }
 479:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 481-482
```cpp
 481:   return _fft_apply_normalization(output, normalization, out_sizes, dim);
 482: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 484-488
```cpp
 484: Tensor& _fft_c2c_cufft_out(const Tensor& self, IntArrayRef dim,
 485:                            int64_t normalization, bool forward, Tensor& out) {
 486:   auto result = _fft_c2c_cufft(self, dim, static_cast<int64_t>(fft_norm_mode::none), forward);
 487:   return _fft_apply_normalization_out(out, result, normalization, result.sizes(), dim);
 488: }
```
- EN: This block defines or continues the implementation of `_fft_c2c_cufft_out`.
- CN: 该代码块定义或继续实现 `_fft_c2c_cufft_out`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 491-491
```cpp
 491: } // at::native
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

## Key Concepts / 关键概念

- `TensorIterator` normalizes tensor shapes/strides and drives elementwise CUDA traversal. / `TensorIterator` 统一张量形状与步长，并驱动逐元素 CUDA 遍历。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/core/Tensor.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/Config.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/ScalarOps.h>`
  - `<ATen/TensorIterator.h>`
  - `<ATen/detail/CUDAHooksInterface.h>`
  - `<ATen/native/Resize.h>`
  - `<ATen/native/SpectralOpsUtils.h>`
  - `<ATen/native/cuda/CuFFTUtils.h>`
  - `<ATen/native/cuda/CuFFTPlanCache.h>`
  - `<ATen/cuda/nvrtc_stub/ATenNVRTC.h>`
- Runtime symbols / 运行时符号:
  - `nvrtc_stub`
  - `TensorIterator`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
