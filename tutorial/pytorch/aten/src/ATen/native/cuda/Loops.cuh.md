# Loops.cuh — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/Loops.cuh`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Declares or defines CUDA helpers/templates associated with `make_input_offset_calculator`, `make_output_offset_calculator`, `elementwise_kernel_helper`, `gpu_kernel_nocast`.
- 用途（中文）: 声明或定义与 `make_input_offset_calculator`, `make_output_offset_calculator`, `elementwise_kernel_helper`, `gpu_kernel_nocast` 相关的 CUDA 辅助函数/模板。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
   1: #pragma once
   2: 
   3: #include <ATen/OpMathType.h>
   4: #include <ATen/cuda/detail/OffsetCalculator.cuh>
   5: #include <ATen/detail/FunctionTraits.h>
   6: #include <ATen/native/TensorIterator.h>
   7: #include <ATen/native/TensorIteratorDynamicCasting.h>
   8: #include <ATen/native/cuda/thread_constants.h>
   9: #include <ATen/native/cuda/MemoryAccess.cuh>
  10: 
  11: #include <c10/util/C++17.h>
  12: #include <tuple>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/OpMathType.h>`, `<ATen/cuda/detail/OffsetCalculator.cuh>`, `<ATen/detail/FunctionTraits.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/OpMathType.h>`, `<ATen/cuda/detail/OffsetCalculator.cuh>`, `<ATen/detail/FunctionTraits.h>`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 16-37
```cpp
  16: namespace at::native {
  17: 
  18: template<int N>
  19: static OffsetCalculator<N> make_input_offset_calculator(const TensorIteratorBase& iter) {
  20:   // array size can not be 0, this happens when N == 0
  21:   constexpr int array_size = std::max<int>(N, 1);
  22:   TORCH_INTERNAL_ASSERT(N == iter.ntensors() - iter.noutputs());
  23:   std::array<const int64_t*, array_size> strides;
  24:   int64_t element_sizes[array_size];
  25:   for (int i = 0; i < N; i++) {
  26:     strides[i] = iter.strides(i + iter.noutputs()).data();
  27:     element_sizes[i] = iter.element_size(i + iter.noutputs());
  28:   }
  29:   return OffsetCalculator<N>(iter.ndim(), iter.shape().data(), strides.data(), element_sizes);
  30: }
  31: 
  32: template <int num_outputs = 1>
  33: static OffsetCalculator<num_outputs> make_output_offset_calculator(const TensorIteratorBase& iter) {
  34:   TORCH_INTERNAL_ASSERT(num_outputs == iter.noutputs());
  35:   std::array<const int64_t*, num_outputs> strides;
  36:   int64_t element_sizes[num_outputs];
  37:   for (int i = 0; i < num_outputs; i++) {
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `make_input_offset_calculator`, `make_output_offset_calculator`.
- CN: 该代码块定义或继续实现 `make_input_offset_calculator`, `make_output_offset_calculator`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 38-42
```cpp
  38:     strides[i] = iter.strides(i).data();
  39:     element_sizes[i] = iter.element_size(i);
  40:   }
  41:   return OffsetCalculator<num_outputs>(iter.ndim(), iter.shape().data(), strides.data(), element_sizes);
  42: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 44-65
```cpp
  44: template <bool reverted_idx = false, typename func_t, typename policy_t>
  45: __device__ inline void elementwise_kernel_helper(func_t f, policy_t policy) {
  46:   using traits = function_traits<func_t>;
  47:   using return_t = typename traits::result_type;
  48:   using args_t = typename traits::ArgsTuple;
  49:   constexpr int elems_per_thread = policy_t::tws;
  50: 
  51:   int idx = blockIdx.x;
  52:   if constexpr (reverted_idx)
  53:     idx = gridDim.x - blockIdx.x - 1;
  54: 
  55:   return_t results[elems_per_thread];
  56:   args_t args[elems_per_thread];
  57: 
  58:   // load
  59:   policy.load(args, idx);
  60: 
  61:   // compute
  62:   #pragma unroll
  63:   for (int i = 0; i < elems_per_thread; i++) {
  64:     if (policy.check_inbounds(i)) {
  65: #if defined(__HIP__)
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `elementwise_kernel_helper`.
- CN: 该代码块定义或继续实现 `elementwise_kernel_helper`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 66-71
```cpp
  66:       results[i] = c10::guts::apply(f, args[i]);
  67: #else
  68:       results[i] = std::apply(f, args[i]);
  69: #endif
  70:     }
  71:   }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 73-73
```cpp
  73:   // store
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 74-75
```cpp
  74:   policy.store(results, idx);
  75: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 77-77
```cpp
  77: }  // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 79-79
```cpp
  79: #include <ATen/native/cuda/CUDALoops.cuh>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/CUDALoops.cuh>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/CUDALoops.cuh>`。

### Lines 81-101
```cpp
  81: namespace at:: native {
  82: 
  83: template <typename func_t>
  84: void gpu_kernel_nocast(TensorIteratorBase& iter, const func_t& f, bool check_cast = true) {
  85: 
  86:   for (int arg = 0; arg < iter.ntensors(); arg++) {
  87:     TORCH_INTERNAL_ASSERT(
  88:       iter.device(arg).is_cuda(),
  89:       "argument ", arg, ": expected a CUDA device but found ", iter.device(arg));
  90:   }
  91: 
  92:   if (iter.numel() == 0) {
  93:     return;
  94:   }
  95: 
  96:   if (!iter.can_use_32bit_indexing()) {
  97:     for (auto& sub_iter : iter.with_32bit_indexing()) {
  98:       gpu_kernel_nocast(sub_iter, f, check_cast);
  99:     }
 100:     return;
 101:   }
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `gpu_kernel_nocast`.
- CN: 该代码块定义或继续实现 `gpu_kernel_nocast`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 103-107
```cpp
 103:   if (check_cast) {
 104:     TORCH_INTERNAL_ASSERT(!needs_dynamic_casting<func_t>::check(iter));
 105:   }
 106:   gpu_kernel_impl_nocast(iter, f);
 107: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 109-112
```cpp
 109: template <typename func_t>
 110: void gpu_kernel_opaque(TensorIteratorBase& iter, const func_t& f) {
 111:   gpu_kernel_nocast(iter, f, false);
 112: }
```
- EN: This block defines or continues the implementation of `gpu_kernel_opaque`.
- CN: 该代码块定义或继续实现 `gpu_kernel_opaque`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 114-135
```cpp
 114: template <typename func_t>
 115: void gpu_kernel(TensorIteratorBase& iter, const func_t& f) {
 116: 
 117:   for (int arg = 0; arg < iter.ntensors(); arg++) {
 118:     TORCH_INTERNAL_ASSERT(
 119:       iter.device(arg).is_cuda(),
 120:       "argument ", arg, ": expected a CUDA device but found ", iter.device(arg));
 121:   }
 122: 
 123:   if (iter.numel() == 0) {
 124:     return;
 125:   }
 126: 
 127:   if (!iter.can_use_32bit_indexing()) {
 128:     for (auto& sub_iter : iter.with_32bit_indexing()) {
 129:       gpu_kernel(sub_iter, f);
 130:     }
 131:     return;
 132:   }
 133: 
 134:   gpu_kernel_impl(iter, f);
 135: }
```
- EN: This block defines or continues the implementation of `gpu_kernel`.
- CN: 该代码块定义或继续实现 `gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 137-149
```cpp
 137: template<typename arg1_t, typename arg2_t, typename return_t, typename func_t>
 138: struct AUnaryFunctor {
 139:   using traits = function_traits<func_t>;
 140:   using opmath_arg1_t = typename traits::template arg<0>::type;
 141:   __device__ return_t operator()(arg2_t b) const {
 142:     return f(a, b);
 143:   }
 144:   // NB: scalar is stored in higher precision!
 145:   AUnaryFunctor(func_t f_, opmath_arg1_t a_): f(f_), a(a_) {}
 146:   private:
 147:     func_t f;
 148:     opmath_arg1_t a;
 149: };
```
- EN: This block defines or continues the implementation of `AUnaryFunctor`.
- CN: 该代码块定义或继续实现 `AUnaryFunctor`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 151-163
```cpp
 151: template<typename arg1_t, typename arg2_t, typename return_t, typename func_t>
 152: struct BUnaryFunctor {
 153:   using traits = function_traits<func_t>;
 154:   using opmath_arg2_t = typename traits::template arg<1>::type;
 155:   __device__ return_t operator()(arg1_t a) const {
 156:     return f(a, b);
 157:   }
 158:   // NB: scalar is stored in higher precision!
 159:   BUnaryFunctor(func_t f_, opmath_arg2_t b_): f(f_), b(b_) {}
 160:   private:
 161:     func_t f;
 162:     opmath_arg2_t b;
 163: };
```
- EN: This block defines or continues the implementation of `BUnaryFunctor`.
- CN: 该代码块定义或继续实现 `BUnaryFunctor`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 165-166
```cpp
 165: // Though seemingly noop, this inserts casts from arg1_t to func_t's type
 166: // (which may be higher precision), as well as casts to return_t
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 167-175
```cpp
 167: template <typename arg1_t, typename arg2_t, typename return_t, typename func_t>
 168: struct BinaryFunctor {
 169:   __device__ return_t operator()(arg1_t a, arg2_t b) const {
 170:     return f(a, b);
 171:   }
 172:   BinaryFunctor(func_t f_): f(f_) {}
 173:   private:
 174:     func_t f;
 175: };
```
- EN: This block defines or continues the implementation of `BinaryFunctor`.
- CN: 该代码块定义或继续实现 `BinaryFunctor`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 177-181
```cpp
 177: // Unlike gpu_kernel_with_scalars, this allows you to pass a func_t which
 178: // accepts inputs at higher precision (typically opmath_t), but then
 179: // ensure that we load from memory at the correct precision (scalar_t)
 180: // to avoid expensive loads.  For the whole sordid story see
 181: // https://dev-discuss.pytorch.org/t/cuda-loops-case-study-code-generation-vs-templates/302
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 182-203
```cpp
 182: template <typename arg1_t, typename arg2_t = arg1_t, typename return_t = arg1_t, typename func_t>
 183: void opmath_gpu_kernel_with_scalars(TensorIteratorBase& iter, const func_t& f) {
 184:   TORCH_INTERNAL_ASSERT(iter.ntensors() == 3);
 185: 
 186:   using traits = function_traits<func_t>;
 187:   using opmath_arg1_t = typename traits::template arg<0>::type;
 188:   using opmath_arg2_t = typename traits::template arg<1>::type;
 189:   static_assert(
 190:       traits::arity == 2,
 191:       "gpu_kernel_with_scalars only supports two input arguments");
 192: 
 193:   if (iter.is_cpu_scalar(1)) {
 194:     AUnaryFunctor<arg1_t, arg2_t, return_t, func_t> af(f, iter.scalar_value<opmath_arg1_t>(1));
 195:     iter.remove_operand(1);
 196:     // TODO: When all kernels that use gpu_kernel_with_scalars are
 197:     // ported to structured, this device guard can be deleted.  This
 198:     // works around incorrect device guard generation for pre-structured
 199:     // kernels device guards, but structured kernels do it right and
 200:     // we can assume the device is already set correctly
 201:     const OptionalDeviceGuard device_guard(iter.device(1));
 202:     gpu_kernel(iter, af);
 203:   } else if (iter.is_cpu_scalar(2)) {
```
- EN: This block defines or continues the implementation of `opmath_gpu_kernel_with_scalars`.
- CN: 该代码块定义或继续实现 `opmath_gpu_kernel_with_scalars`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 204-210
```cpp
 204:     BUnaryFunctor<arg1_t, arg2_t, return_t, func_t> bf(f, iter.scalar_value<opmath_arg2_t>(2));
 205:     iter.remove_operand(2);
 206:     gpu_kernel(iter, bf);
 207:   } else {
 208:     gpu_kernel(iter, BinaryFunctor<arg1_t, arg2_t, return_t, func_t>(f));
 209:   }
 210: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 212-233
```cpp
 212: template <typename scalar_t, typename return_t = scalar_t, typename func_t>
 213: void opmath_symmetric_gpu_kernel_with_scalars(TensorIteratorBase& iter, const func_t& f) {
 214:   // Use symmetric property of the functor to reduce number of kernels,
 215:   // requires f(a, b) == f(b, a)
 216:   TORCH_INTERNAL_ASSERT(iter.ntensors() == 3);
 217: 
 218:   using traits = function_traits<func_t>;
 219:   using opmath_arg_t = typename traits::template arg<0>::type;
 220:   static_assert(
 221:       traits::arity == 2,
 222:       "gpu_kernel_with_scalars only supports two input arguments");
 223:   static_assert(std::is_same_v<opmath_arg_t, typename traits::template arg<1>::type>,
 224:                 "f is not symmetric");
 225: 
 226:   OptionalDeviceGuard device_guard;
 227:   opmath_arg_t scalar_val{};
 228: 
 229:   if (iter.is_cpu_scalar(1)) {
 230:     scalar_val = iter.scalar_value<opmath_arg_t>(1);
 231:     iter.remove_operand(1);
 232: 
 233:     // TODO: When all kernels that use gpu_kernel_with_scalars are
```
- EN: This block defines or continues the implementation of `opmath_symmetric_gpu_kernel_with_scalars`.
- CN: 该代码块定义或继续实现 `opmath_symmetric_gpu_kernel_with_scalars`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 234-237
```cpp
 234:     // ported to structured, this device guard can be deleted.  This
 235:     // works around incorrect device guard generation for pre-structured
 236:     // kernels device guards, but structured kernels do it right and
 237:     // we can assume the device is already set correctly
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 238-242
```cpp
 238:     device_guard.reset_device(iter.device(1));
 239:   } else if (iter.is_cpu_scalar(2)) {
 240:     scalar_val = iter.scalar_value<opmath_arg_t>(2);
 241:     iter.remove_operand(2);
 242:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 244-250
```cpp
 244:   if (iter.ninputs() == 2) {
 245:     gpu_kernel(iter, BinaryFunctor<scalar_t, scalar_t, return_t, func_t>(f));
 246:   } else {
 247:     AUnaryFunctor<scalar_t, scalar_t, return_t, func_t> unary_f(f, scalar_val);
 248:     gpu_kernel(iter, unary_f);
 249:   }
 250: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 252-253
```cpp
 252: // Legacy variant that assumes that func_t has the correct types
 253: // that we expect to load from memory
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 254-264
```cpp
 254: template <typename func_t>
 255: void gpu_kernel_with_scalars(TensorIteratorBase& iter, const func_t& f) {
 256:   using traits = function_traits<func_t>;
 257:   static_assert(
 258:       traits::arity == 2,
 259:       "gpu_kernel_with_scalars only supports two input arguments");
 260:   using arg1_t = typename traits::template arg<0>::type;
 261:   using arg2_t = typename traits::template arg<1>::type;
 262:   using return_t = typename traits::result_type;
 263:   opmath_gpu_kernel_with_scalars<arg1_t, arg2_t, return_t, func_t>(iter, f);
 264: }
```
- EN: This block defines or continues the implementation of `gpu_kernel_with_scalars`.
- CN: 该代码块定义或继续实现 `gpu_kernel_with_scalars`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 266-287
```cpp
 266: namespace { // functions for `gpu_kernel_multiple_outputs`.
 267: 
 268: // check the return type is `thrust::tuple`, not `std::tuple`.
 269: template <typename T> struct is_tuple: std::false_type {};
 270: 
 271: template <typename ...T> struct is_tuple<thrust::tuple<T...>>: std::true_type {};
 272: 
 273: template <int num_outputs, typename func_t, typename array_t, typename inp_calc_t, typename out_calc_t>
 274: C10_LAUNCH_BOUNDS_1(num_threads())
 275: __global__ void unrolled_elementwise_kernel_for_multi_outputs(int N, func_t f, array_t data, inp_calc_t ic, out_calc_t oc) {
 276:   int remaining = N - block_work_size() * blockIdx.x;
 277:   elementwise_kernel_helper(f, memory::policies::multi_outputs_unroll<array_t, inp_calc_t, out_calc_t, num_outputs>(data, remaining, ic, oc));
 278: }
 279: 
 280: template <int num_outputs, typename func_t, typename array_t, typename inp_calc_t, typename out_calc_t>
 281: static inline void launch_unrolled_kernel_for_multi_outputs(int64_t N, const func_t& f, array_t data, inp_calc_t ic, out_calc_t oc) {
 282:   TORCH_INTERNAL_ASSERT(N > 0 && N <= std::numeric_limits<int32_t>::max());
 283:   int64_t grid = (N + block_work_size() - 1) / block_work_size();
 284:   auto stream = at::cuda::getCurrentCUDAStream();
 285:   unrolled_elementwise_kernel_for_multi_outputs<num_outputs, func_t, array_t><<<grid, num_threads(), 0, stream>>>(N, f, data, ic, oc);
 286:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 287: }
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `unrolled_elementwise_kernel_for_multi_outputs`, `launch_unrolled_kernel_for_multi_outputs`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `unrolled_elementwise_kernel_for_multi_outputs`, `launch_unrolled_kernel_for_multi_outputs`，它们会直接在 CUDA 线程上执行。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 289-310
```cpp
 289: template <typename func_t>
 290: void gpu_kernel_multiple_outputs_impl(TensorIteratorBase& iter, const func_t& f) {
 291:   using traits = function_traits<func_t>;
 292:   using output_t = typename traits::result_type;
 293:   static_assert(is_tuple<output_t>::value, "f's return type must be `thrust::tuple`");
 294:   constexpr int num_outputs = thrust::tuple_size<output_t>::value;
 295:   constexpr int num_inputs = traits::arity;
 296:   constexpr int ntensors = num_outputs + num_inputs;
 297: 
 298:   TORCH_INTERNAL_ASSERT(iter.can_use_32bit_indexing());
 299:   TORCH_INTERNAL_ASSERT(iter.ntensors() == ntensors);
 300: 
 301:   std::array<char*, ntensors> data;
 302:   for (int i = 0; i < ntensors; i++) {
 303:     data[i] = (char*)iter.data_ptr(i);
 304:   }
 305: 
 306:   int64_t numel = iter.numel();
 307: 
 308:   if (iter.is_contiguous()) {
 309:     auto input_calc = TrivialOffsetCalculator<num_inputs>();
 310:     auto output_calc = TrivialOffsetCalculator<num_outputs>();
```
- EN: This block defines or continues the implementation of `gpu_kernel_multiple_outputs_impl`.
- CN: 该代码块定义或继续实现 `gpu_kernel_multiple_outputs_impl`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 311-318
```cpp
 311:     launch_unrolled_kernel_for_multi_outputs<num_outputs>(numel, f, data, input_calc, output_calc);
 312:   } else {
 313:     auto input_calc = make_input_offset_calculator<num_inputs>(iter);
 314:     auto output_calc = make_output_offset_calculator<num_outputs>(iter);
 315:     launch_unrolled_kernel_for_multi_outputs<num_outputs>(numel, f, data, input_calc, output_calc);
 316:   }
 317: }
 318: } // namespace
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 320-340
```cpp
 320: template <typename func_t>
 321: void gpu_kernel_multiple_outputs(TensorIteratorBase& iter, const func_t& f) {
 322:   ASSERT_HOST_DEVICE_LAMBDA(func_t);
 323: 
 324:   for (int arg = 0; arg < iter.ntensors(); arg++) {
 325:     TORCH_INTERNAL_ASSERT(iter.device(arg).is_cuda());
 326:   }
 327: 
 328:   if (iter.numel() == 0) {
 329:     return;
 330:   }
 331: 
 332:   if (!iter.can_use_32bit_indexing()) {
 333:     for (auto& sub_iter : iter.with_32bit_indexing()) {
 334:       gpu_kernel_multiple_outputs(sub_iter, f);
 335:     }
 336:     return;
 337:   }
 338: 
 339:   gpu_kernel_multiple_outputs_impl(iter, f);
 340: }
```
- EN: This block defines or continues the implementation of `gpu_kernel_multiple_outputs`.
- CN: 该代码块定义或继续实现 `gpu_kernel_multiple_outputs`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 342-342
```cpp
 342: } //namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `TensorIterator` normalizes tensor shapes/strides and drives elementwise CUDA traversal. / `TensorIterator` 统一张量形状与步长，并驱动逐元素 CUDA 遍历。
- `gpu_kernel` applies a functor over iterator elements on CUDA. / `gpu_kernel` 在 CUDA 上对迭代器元素应用函数对象。
- `gpu_kernel_with_scalars` handles elementwise CUDA work while folding scalar operands efficiently. / `gpu_kernel_with_scalars` 在高效折叠标量操作数的同时执行逐元素 CUDA 计算。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- Thrust utilities supply STL-like CUDA algorithms and containers. / Thrust 工具提供类似 STL 的 CUDA 算法与容器。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/OpMathType.h>`
  - `<ATen/cuda/detail/OffsetCalculator.cuh>`
  - `<ATen/detail/FunctionTraits.h>`
  - `<ATen/native/TensorIterator.h>`
  - `<ATen/native/TensorIteratorDynamicCasting.h>`
  - `<ATen/native/cuda/thread_constants.h>`
  - `<ATen/native/cuda/MemoryAccess.cuh>`
  - `<c10/util/C++17.h>`
  - `<tuple>`
  - `<ATen/native/cuda/CUDALoops.cuh>`
- Runtime symbols / 运行时符号:
  - `launch_unrolled_kernel_for_multi_outputs`
  - `gpu_kernel`
  - `gpu_kernel_with_scalars`
  - `TensorIterator`
  - `TensorIteratorBase`
  - `thrust::tuple`
  - `thrust::tuple_size`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
