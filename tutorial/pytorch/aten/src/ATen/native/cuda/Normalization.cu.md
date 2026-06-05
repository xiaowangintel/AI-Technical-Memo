# Normalization.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/Normalization.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `first_type`, `is_mixed_type`, `batch_norm_use_channels_last_kernels`, `batch_norm_choose_impl`.
- 用途（中文）: 实现与 `first_type`, `is_mixed_type`, `batch_norm_use_channels_last_kernels`, `batch_norm_choose_impl` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/cuda/detail/IndexUtils.cuh>
   3: #include <ATen/detail/CUDAHooksInterface.h>
   4: #include <ATen/native/Normalization.h>
   5: #include <ATen/native/TensorIterator.h>
   6: #include <ATen/native/ReduceOps.h>
   7: #include <ATen/native/Resize.h>
   8: #include <ATen/native/cuda/Loops.cuh>
   9: #include <ATen/native/cuda/Resize.h>
  10: #include <ATen/native/cuda/Normalization.cuh>
  11: #include <c10/cuda/CUDAMathCompat.h>
  12: 
  13: #ifndef AT_PER_OPERATOR_HEADERS
  14: #include <ATen/Functions.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/cuda/detail/IndexUtils.cuh>`, `<ATen/detail/CUDAHooksInterface.h>`, `<ATen/native/Normalization.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/cuda/detail/IndexUtils.cuh>`, `<ATen/detail/CUDAHooksInterface.h>`, `<ATen/native/Normalization.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 15-28
```cpp
  15: #include <ATen/NativeFunctions.h>
  16: #else
  17: #include <ATen/ops/_batch_norm_with_update_native.h>
  18: #include <ATen/ops/batch_norm_backward_native.h>
  19: #include <ATen/ops/batch_norm_backward_elemt_native.h>
  20: #include <ATen/ops/batch_norm_backward_reduce_native.h>
  21: #include <ATen/ops/batch_norm_elemt_native.h>
  22: #include <ATen/ops/batch_norm_gather_stats_native.h>
  23: #include <ATen/ops/batch_norm_gather_stats_with_counts_native.h>
  24: #include <ATen/ops/batch_norm_stats_native.h>
  25: #include <ATen/ops/batch_norm_update_stats_native.h>
  26: #include <ATen/ops/cudnn_batch_norm.h>
  27: #include <ATen/ops/cudnn_batch_norm_backward.h>
  28: #include <ATen/ops/empty_like.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/NativeFunctions.h>`, `<ATen/ops/_batch_norm_with_update_native.h>`, `<ATen/ops/batch_norm_backward_native.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/NativeFunctions.h>`, `<ATen/ops/_batch_norm_with_update_native.h>`, `<ATen/ops/batch_norm_backward_native.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 29-35
```cpp
  29: #include <ATen/ops/from_blob.h>
  30: #include <ATen/ops/miopen_batch_norm.h>
  31: #include <ATen/ops/miopen_batch_norm_backward.h>
  32: #include <ATen/ops/native_batch_norm_backward_native.h>
  33: #include <ATen/ops/native_batch_norm_native.h>
  34: #include <ATen/ops/scalar_tensor.h>
  35: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/ops/from_blob.h>`, `<ATen/ops/miopen_batch_norm.h>`, `<ATen/ops/miopen_batch_norm_backward.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/ops/from_blob.h>`, `<ATen/ops/miopen_batch_norm.h>`, `<ATen/ops/miopen_batch_norm_backward.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 37-58
```cpp
  37: namespace at::native {
  38: 
  39: namespace {
  40: 
  41: ScalarType first_type() {
  42:   return ScalarType::Undefined;
  43: }
  44: 
  45: template <typename... Args>
  46: ScalarType first_type(const Tensor& arg, const Args&... parameters) {
  47:   return arg.defined() ? arg.scalar_type() : first_type(parameters...);
  48: }
  49: 
  50: // A transform is mixed type if the parameters are higher precision than the input
  51: template <typename... Args>
  52: bool is_mixed_type(const Tensor& input, const Args&... parameters) {
  53:   const auto parameter_type = first_type(parameters...);
  54:   return ((parameter_type != ScalarType::Undefined) &&
  55:           (parameter_type != input.scalar_type()));
  56: }
  57: 
  58: inline bool batch_norm_use_channels_last_kernels(const at::Tensor& self) {
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `first_type`, `is_mixed_type`, `batch_norm_use_channels_last_kernels`.
- CN: 该代码块定义或继续实现 `first_type`, `is_mixed_type`, `batch_norm_use_channels_last_kernels`。

### Lines 59-64
```cpp
  59:   return (
  60:     self.is_contiguous(at::MemoryFormat::ChannelsLast) ||
  61:     self.is_contiguous(at::MemoryFormat::ChannelsLast3d) ||
  62:     (self.is_contiguous() && self.strides()[1] == 1)
  63:   );
  64: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 66-70
```cpp
  66: enum class Impl {
  67:   Contiguous,
  68:   ChannelsLast,
  69:   General,
  70: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 72-86
```cpp
  72: inline Impl batch_norm_choose_impl(const Tensor& self) {
  73:   if (!at::cuda::detail::canUse32BitIndexMath(self)) {
  74:     return Impl::General;
  75:   }
  76: 
  77:   if (self.is_contiguous()) {
  78:     return self.strides()[1] == 1 ? Impl::ChannelsLast : Impl::Contiguous;
  79:   }
  80: 
  81:   if (self.is_contiguous(at::MemoryFormat::ChannelsLast)) {
  82:     return Impl::ChannelsLast;
  83:   }
  84: 
  85:   return Impl::General;
  86: }
```
- EN: This block defines or continues the implementation of `batch_norm_choose_impl`.
- CN: 该代码块定义或继续实现 `batch_norm_choose_impl`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 88-95
```cpp
  88: inline Impl batch_norm_choose_impl(const Tensor& in1, const Tensor& in2) {
  89:   auto imp1 = batch_norm_choose_impl(in1);
  90:   if (imp1 == Impl::General) {
  91:     return imp1;
  92:   }
  93:   auto imp2 = batch_norm_choose_impl(in2);
  94:   return imp1 == imp2 ? imp1 : Impl::General;
  95: }
```
- EN: This block defines or continues the implementation of `batch_norm_choose_impl`.
- CN: 该代码块定义或继续实现 `batch_norm_choose_impl`。

### Lines 97-118
```cpp
  97: void batch_norm_elementwise(
  98:     const Tensor& out, const Tensor& self, const std::optional<Tensor>& weight_opt,
  99:     const std::optional<Tensor>& bias_opt, const Tensor& mean_, const Tensor& invstd_) {
 100:   switch (batch_norm_choose_impl(self)) {
 101:   case Impl::Contiguous: {
 102:     c10::MaybeOwned<Tensor> weight = at::borrow_from_optional_tensor(weight_opt);
 103:     c10::MaybeOwned<Tensor> bias = at::borrow_from_optional_tensor(bias_opt);
 104:     resize_output(out, self.sizes());
 105:     AT_DISPATCH_FLOATING_TYPES_AND2(kBFloat16, kHalf, self.scalar_type(),
 106:                                     "batch_norm_elementwise_cuda", [&] {
 107:       using accscalar_t = at::acc_type<scalar_t, true>;
 108:       const bool mixed_type = is_mixed_type(self, *weight, *bias);
 109:       if (mixed_type) {
 110:         batch_norm_elemt_cuda_template<scalar_t, accscalar_t, int32_t>(
 111:             out, self, *weight, *bias, mean_, invstd_);
 112:       } else {
 113:         batch_norm_elemt_cuda_template<scalar_t, scalar_t, int32_t>(
 114:             out, self, *weight, *bias, mean_, invstd_);
 115:       }
 116:     });
 117:     return;
 118:   }
```
- EN: This block defines or continues the implementation of `batch_norm_elementwise`.
- CN: 该代码块定义或继续实现 `batch_norm_elementwise`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 119-136
```cpp
 119:   case Impl::ChannelsLast: {
 120:     auto weight = at::borrow_from_optional_tensor(weight_opt);
 121:     auto bias = at::borrow_from_optional_tensor(bias_opt);
 122: 
 123:     if (resize_output_check(out, self.sizes())) {
 124:         resize_impl_cuda_(out.unsafeGetTensorImpl(), self.sizes(), self.strides());
 125:     }
 126:     if ((out.strides() == self.strides()) &&
 127:         (!weight->defined() || weight->is_contiguous()) &&
 128:         (!bias->defined() || bias->is_contiguous()) &&
 129:         (!mean_.defined() || mean_.is_contiguous()) &&
 130:         (!invstd_.defined() || invstd_.is_contiguous())) {
 131:       batch_norm_elemt_channels_last_cuda_template(
 132:           out, self, *weight, *bias, mean_, invstd_);
 133:       return;
 134:     }
 135:     [[fallthrough]];
 136:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 137-158
```cpp
 137:   case Impl::General: {
 138:     const int64_t ndim = self.dim();
 139:     DimVector sizes(ndim, 1), strides(ndim, 0);
 140:     // Helper to convert 1d tensors to an nd tensor that broadcasts with input
 141:     // All elements go into the channel dimension
 142:     auto as_nd = [&](const Tensor& t) {
 143:       TORCH_INTERNAL_ASSERT(t.defined() && t.dim() == 1);
 144:       sizes[1] = t.sizes()[0];
 145:       strides[1] = t.strides()[0];
 146:       return t.as_strided(sizes, strides);
 147:     };
 148: 
 149:     auto weight = weight_opt.has_value() && weight_opt->defined() ?
 150:         as_nd(*weight_opt) : at::scalar_tensor(1, mean_.options());
 151:     auto bias = bias_opt.has_value() && bias_opt->defined() ?
 152:         as_nd(*bias_opt) : at::scalar_tensor(0, mean_.options());
 153:     auto mean = as_nd(mean_);
 154:     auto invstd = as_nd(invstd_);
 155: 
 156:     auto iter = TensorIteratorConfig()
 157:         .add_output(out)
 158:         .add_input(self)
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 159-165
```cpp
 159:         .add_input(weight)
 160:         .add_input(bias)
 161:         .add_input(mean)
 162:         .add_input(invstd)
 163:         .check_all_same_dtype(false)
 164:         .promote_inputs_to_common_dtype(false)
 165:         .build();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 167-178
```cpp
 167:     AT_DISPATCH_FLOATING_TYPES_AND2(kBFloat16, kHalf, self.scalar_type(),
 168:                                     "batch_norm_elementwise_cuda", [&] {
 169:       using acc_t = at::acc_type<scalar_t, true>;
 170:       gpu_kernel(iter, [] GPU_LAMBDA (scalar_t input, acc_t weight, acc_t bias,
 171:                                       acc_t mean, acc_t invstd) -> scalar_t {
 172:         return (input - mean) * weight * invstd + bias;
 173:       });
 174:     });
 175:     return;
 176:   }
 177:   }
 178: }
```
- EN: This block defines or continues the implementation of `gpu_kernel`.
- CN: 该代码块定义或继续实现 `gpu_kernel`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 180-201
```cpp
 180: Tensor batch_norm_elementwise_backward_train(
 181:     const Tensor& grad_out, const Tensor& input, const Tensor& mean, const Tensor& invstd,
 182:     const Tensor& weight, const Tensor& sum_dy, const Tensor& sum_dy_xmu) {
 183:   switch (batch_norm_choose_impl(input, grad_out)) {
 184:   case Impl::Contiguous: {
 185:     return AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, input.scalar_type(),
 186:                                            "batch_norm_backward_elemt", [&] {
 187:       using accscalar_t = at::acc_type<scalar_t, true>;
 188:       const bool mixed_type = is_mixed_type(input, weight);
 189:       if (mixed_type) {
 190:         return batch_norm_backward_elemt_cuda_template<scalar_t, accscalar_t, int32_t>(
 191:             grad_out, input, mean, invstd, weight, sum_dy, sum_dy_xmu);
 192:       } else {
 193:         return batch_norm_backward_elemt_cuda_template<scalar_t, scalar_t, int32_t>(
 194:             grad_out, input, mean, invstd, weight, sum_dy, sum_dy_xmu);
 195:       }
 196:     });
 197:   }
 198:   case Impl::ChannelsLast: {
 199:     if ((!weight.defined() || weight.is_contiguous()) &&
 200:         mean.is_contiguous() && invstd.is_contiguous()) {
 201:       return batch_norm_backward_elemt_channels_last_cuda_template(
```
- EN: This block defines or continues the implementation of `batch_norm_elementwise_backward_train`.
- CN: 该代码块定义或继续实现 `batch_norm_elementwise_backward_train`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 202-215
```cpp
 202:           grad_out, input, mean, invstd, weight, sum_dy, sum_dy_xmu);
 203:     }
 204:     [[fallthrough]];
 205:   }
 206:   case Impl::General: {
 207:     const auto ndim = input.dim();
 208:     DimVector sizes(ndim, 1), strides(ndim, 0);
 209:     auto as_nd = [&](const Tensor& t) {
 210:       TORCH_INTERNAL_ASSERT(t.defined() && t.dim() == 1);
 211:       sizes[1] = t.sizes()[0];
 212:       strides[1] = t.strides()[0];
 213:       return t.as_strided(sizes, strides);
 214:     };
 215:     auto invstd_nd = as_nd(invstd);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 216-220
```cpp
 216:     auto mean_nd = as_nd(mean);
 217:     auto sum_dy_nd = as_nd(sum_dy);
 218:     auto sum_dy_xmu_nd = as_nd(sum_dy_xmu);
 219:     auto weight_nd = weight.defined() ? as_nd(weight) :
 220:         at::scalar_tensor(1.0, input.options().dtype(mean.scalar_type()));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 222-234
```cpp
 222:     Tensor grad_input = at::empty(input.sizes(), grad_out.options().memory_format(input.suggest_memory_format()));
 223:     auto iter = TensorIteratorConfig()
 224:         .add_output(grad_input)
 225:         .add_input(grad_out)
 226:         .add_input(input)
 227:         .add_input(weight_nd)
 228:         .add_input(mean_nd)
 229:         .add_input(invstd_nd)
 230:         .add_input(sum_dy_xmu_nd)
 231:         .add_input(sum_dy_nd)
 232:         .check_all_same_dtype(false)
 233:         .promote_inputs_to_common_dtype(false)
 234:         .build();
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 236-249
```cpp
 236:     AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, grad_out.scalar_type(),
 237:                                     "batch_norm_eval_backward", [&]{
 238:       using accscalar_t = at::acc_type<scalar_t, true>;
 239:       auto norm_fct = static_cast<accscalar_t>(1.0 / (input.numel() /input.size(1)) );
 240:       gpu_kernel(iter, [norm_fct] GPU_LAMBDA (scalar_t gO, scalar_t input, accscalar_t weight,
 241:                                               accscalar_t mean, accscalar_t invstd,
 242:                                               accscalar_t xmu, accscalar_t dy) -> scalar_t {
 243:         auto factor_1_c = invstd * invstd * xmu * norm_fct;
 244:         auto factor_2_c = weight * invstd;
 245:         auto m_dy_c = dy * norm_fct;
 246:         return (gO - m_dy_c - (input - mean) * factor_1_c) * factor_2_c;
 247:       });
 248:     });
 249:     return grad_input;
```
- EN: This block defines or continues the implementation of `gpu_kernel`.
- CN: 该代码块定义或继续实现 `gpu_kernel`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 250-253
```cpp
 250:   }
 251:   }
 252:   TORCH_INTERNAL_ASSERT(false);
 253: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 255-275
```cpp
 255: Tensor batch_norm_elementwise_backward_eval(
 256:     const Tensor& grad_out, const Tensor& input,
 257:     const Tensor& invstd, const Tensor& weight) {
 258:   const auto ndim = input.dim();
 259:   DimVector shape(ndim, 1), strides(ndim, 0);
 260:   shape[1] = invstd.sizes()[0];
 261:   strides[1] = invstd.strides()[0];
 262:   auto invstd_nd = invstd.as_strided(shape, strides);
 263:   Tensor grad_input = at::empty(input.sizes(), grad_out.options());
 264: 
 265:   if (weight.defined()) {
 266:     strides[1] = weight.strides()[0];
 267:     auto weight_nd = weight.as_strided(shape, strides);
 268:     auto iter = TensorIteratorConfig()
 269:         .add_output(grad_input)
 270:         .add_const_input(grad_out)
 271:         .add_const_input(invstd_nd)
 272:         .add_const_input(weight_nd)
 273:         .check_all_same_dtype(false)
 274:         .promote_inputs_to_common_dtype(false)
 275:         .build();
```
- EN: This block defines or continues the implementation of `batch_norm_elementwise_backward_eval`.
- CN: 该代码块定义或继续实现 `batch_norm_elementwise_backward_eval`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 277-290
```cpp
 277:     AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, grad_out.scalar_type(),
 278:                                     "batch_norm_eval_backward", [&]{
 279:       using accscalar_t = at::acc_type<scalar_t, true>;
 280:       gpu_kernel(iter, [] GPU_LAMBDA (scalar_t gO, accscalar_t invstd, accscalar_t weight)
 281:                  -> scalar_t {
 282:           return gO * weight * invstd;
 283:       });
 284:     });
 285:   } else {
 286:     auto iter = TensorIteratorConfig()
 287:         .add_output(grad_input)
 288:         .add_const_input(grad_out)
 289:         .add_const_input(invstd_nd)
 290:         .check_all_same_dtype(false)
```
- EN: This block defines or continues the implementation of `gpu_kernel`.
- CN: 该代码块定义或继续实现 `gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 291-292
```cpp
 291:         .promote_inputs_to_common_dtype(false)
 292:         .build();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 294-303
```cpp
 294:     AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, grad_out.scalar_type(),
 295:                                     "batch_norm_eval_backward", [&]{
 296:       using accscalar_t = at::acc_type<scalar_t, true>;
 297:       gpu_kernel(iter, [] GPU_LAMBDA (scalar_t gO, accscalar_t invstd) -> scalar_t {
 298:           return gO * invstd;
 299:       });
 300:     });
 301:   }
 302:   return grad_input;
 303: }
```
- EN: This block defines or continues the implementation of `gpu_kernel`.
- CN: 该代码块定义或继续实现 `gpu_kernel`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 306-327
```cpp
 306: void batch_norm_mean_var(const Tensor& self, Tensor& save_mean, Tensor& save_var) {
 307:   // NOTE: Epsilon is only used for InvStd, not Var. The value here is ignored.
 308:   const double dummy_epsilon = 1e-5;
 309:   switch (batch_norm_choose_impl(self)) {
 310:   case Impl::Contiguous: {
 311:     AT_DISPATCH_FLOATING_TYPES_AND2(
 312:         kHalf, kBFloat16, self.scalar_type(), "batch_norm_stats_cuda", [&] {
 313:       batch_norm_stats_cuda_template<scalar_t, int32_t, Var>(
 314:           save_mean, save_var, self, dummy_epsilon);
 315:     });
 316:     return;
 317:   }
 318:   case Impl::ChannelsLast: {
 319:     if ((!save_mean.defined() || save_mean.is_contiguous()) &&
 320:         (!save_var.defined() || save_var.is_contiguous())) {
 321:       AT_DISPATCH_FLOATING_TYPES_AND2(
 322:           kHalf, kBFloat16, self.scalar_type(), "batch_norm_stats_cuda", [&] {
 323:         batch_norm_stats_channels_last_cuda_template<scalar_t, Var>(
 324:             save_mean, save_var, self, dummy_epsilon);
 325:       });
 326:       return;
 327:     }
```
- EN: This block defines or continues the implementation of `batch_norm_mean_var`.
- CN: 该代码块定义或继续实现 `batch_norm_mean_var`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 328-336
```cpp
 328:     [[fallthrough]];
 329:   }
 330:   case Impl::General: {
 331:     const int64_t ndim = self.dim();
 332:     DimVector reduce_dims(ndim - 1);
 333:     reduce_dims[0] = 0;
 334:     for (int64_t i = 2; i < ndim; ++i) {
 335:       reduce_dims[i - 1] = i;
 336:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 338-338
```cpp
 338:     // For some reason this isn't an actual operator but it exists anyway...
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 339-344
```cpp
 339:     at::native::var_mean_out(save_var, save_mean, self, /*dims=*/reduce_dims,
 340:                             /*unbiased=*/false, /*keepdim=*/false);
 341:     return;
 342:   }
 343:   }
 344: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 346-367
```cpp
 346: void batch_norm_update_stats(
 347:     const Tensor& save_mean, const Tensor& save_var,
 348:     const Tensor& running_mean, const Tensor& running_var,
 349:     double momentum_, int64_t N) {
 350: 
 351:   auto iter = TensorIteratorConfig()
 352:       .add_output(running_mean)
 353:       .add_output(running_var)
 354:       .add_input(save_mean)
 355:       .add_input(save_var)
 356:       .add_input(running_mean)
 357:       .add_input(running_var)
 358:       .check_all_same_dtype(false)
 359:       .promote_inputs_to_common_dtype(false)
 360:       .build();
 361: 
 362:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, running_mean.scalar_type(),
 363:                                   "batch_norm_update_stats_cuda", [&] {
 364:       using acc_t = at::acc_type<scalar_t, true>;
 365:       const auto bessel_correction_factor = static_cast<acc_t>(
 366:           static_cast<double>(N) / static_cast<double>(N - 1));
 367:       const auto momentum = static_cast<acc_t>(momentum_);
```
- EN: This block defines or continues the implementation of `batch_norm_update_stats`.
- CN: 该代码块定义或继续实现 `batch_norm_update_stats`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 368-378
```cpp
 368:       gpu_kernel_multiple_outputs(
 369:           iter, [=] GPU_LAMBDA (acc_t mean, acc_t var, scalar_t running_mean, scalar_t running_var)
 370:                -> thrust::tuple<scalar_t, scalar_t> {
 371:         const auto unbiased_var = var * bessel_correction_factor;
 372:         return thrust::tuple<scalar_t, scalar_t>{
 373:           mean * momentum + (1 - momentum) * running_mean,
 374:           unbiased_var * momentum + (1 - momentum) * running_var,
 375:         };
 376:       });
 377:   });
 378: }
```
- EN: This block defines or continues the implementation of `gpu_kernel_multiple_outputs`.
- CN: 该代码块定义或继续实现 `gpu_kernel_multiple_outputs`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 380-401
```cpp
 380: void batch_norm_update_stats_and_invert(
 381:     const Tensor& save_mean, const Tensor& save_var,
 382:     const Tensor& running_mean, const Tensor& running_var,
 383:     double momentum_, double epsilon, int64_t N) {
 384: 
 385:   auto iter = TensorIteratorConfig()
 386:       .add_output(running_mean)
 387:       .add_output(running_var)
 388:       .add_output(save_var)
 389:       .add_const_input(save_mean)
 390:       .add_input(save_var)
 391:       .add_input(running_mean)
 392:       .add_input(running_var)
 393:       .check_all_same_dtype(false)
 394:       .promote_inputs_to_common_dtype(false)
 395:       .build();
 396: 
 397:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, running_mean.scalar_type(),
 398:                                   "batch_norm_update_stats_cuda", [&] {
 399:       using acc_t = at::acc_type<scalar_t, true>;
 400:       const auto bessel_correction_factor = static_cast<acc_t>(
 401:           static_cast<double>(N) / static_cast<double>(N - 1));
```
- EN: This block defines or continues the implementation of `batch_norm_update_stats_and_invert`.
- CN: 该代码块定义或继续实现 `batch_norm_update_stats_and_invert`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 402-415
```cpp
 402:       const auto eps = static_cast<acc_t>(epsilon);
 403:       const auto momentum = static_cast<acc_t>(momentum_);
 404:       gpu_kernel_multiple_outputs(
 405:           iter, [=] GPU_LAMBDA (acc_t mean, acc_t var, scalar_t running_mean, scalar_t running_var)
 406:                -> thrust::tuple<scalar_t, scalar_t, acc_t> {
 407:         const auto unbiased_var = var * bessel_correction_factor;
 408:         return thrust::tuple<scalar_t, scalar_t, acc_t>{
 409:           mean * momentum + (1 - momentum) * running_mean,
 410:           unbiased_var * momentum + (1 - momentum) * running_var,
 411:           c10::cuda::compat::rsqrt(var + eps)
 412:         };
 413:       });
 414:   });
 415: }
```
- EN: This block defines or continues the implementation of `gpu_kernel_multiple_outputs`.
- CN: 该代码块定义或继续实现 `gpu_kernel_multiple_outputs`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 417-432
```cpp
 417: void batch_norm_calc_invstd(const Tensor& out_invstd, const Tensor& running_var, double epsilon) {
 418:   auto iter = TensorIteratorConfig()
 419:       .add_output(out_invstd)
 420:       .add_input(running_var)
 421:       .check_all_same_dtype(false)
 422:       .build();
 423: 
 424:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, running_var.scalar_type(),
 425:                                   "batch_norm_invert_std_cuda", [&] {
 426:     using acc_t = at::acc_type<scalar_t, true>;
 427:     auto eps = static_cast<acc_t>(epsilon);
 428:     gpu_kernel(iter, [eps] GPU_LAMBDA (scalar_t var) -> acc_t {
 429:       return c10::cuda::compat::rsqrt(var + eps);
 430:     });
 431:   });
 432: }
```
- EN: This block defines or continues the implementation of `batch_norm_calc_invstd`, `gpu_kernel`.
- CN: 该代码块定义或继续实现 `batch_norm_calc_invstd`, `gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 433-433
```cpp
 433: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 435-456
```cpp
 435: std::tuple<Tensor&, Tensor&, Tensor&> batch_norm_cuda_out(const Tensor& self, const std::optional<Tensor>& weight_opt, const std::optional<Tensor>& bias_opt, const std::optional<Tensor>& running_mean_opt, const std::optional<Tensor>& running_var_opt, bool train, double momentum, double epsilon, Tensor& output, Tensor& save_mean, Tensor& save_invstd) {
 436:   const bool has_running_mean = (running_mean_opt.has_value() && running_mean_opt->defined());
 437:   const bool has_running_var = (running_var_opt.has_value() && running_var_opt->defined());
 438:   TORCH_CHECK_VALUE(has_running_mean == has_running_var,
 439:     "running_mean and running_var must either both be None or neither be None");
 440: 
 441:   if (train) {
 442:     batch_norm_mean_var(self, save_mean, save_invstd);
 443:     if (has_running_mean) {
 444:       const int64_t N = self.numel() / save_mean.numel();
 445:       batch_norm_update_stats_and_invert(
 446:           save_mean, save_invstd, *running_mean_opt, *running_var_opt,
 447:           momentum, epsilon, N);
 448:     } else {
 449:       batch_norm_calc_invstd(save_invstd, save_invstd, epsilon);
 450:     }
 451:   } else {
 452:     TORCH_CHECK(has_running_mean);
 453:     at::native::resize_output(save_mean, running_mean_opt->sizes());
 454:     save_mean.copy_(*running_mean_opt, /*non_blocking=*/true);
 455:     batch_norm_calc_invstd(save_invstd, running_var_opt.value(), epsilon);
 456:   }
```
- EN: This block defines or continues the implementation of `batch_norm_cuda_out`.
- CN: 该代码块定义或继续实现 `batch_norm_cuda_out`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 458-460
```cpp
 458:   batch_norm_elementwise(output, self, weight_opt, bias_opt, save_mean, save_invstd);
 459:   return std::tuple<Tensor&, Tensor&, Tensor&>(output, save_mean, save_invstd);
 460: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 462-483
```cpp
 462: std::tuple<Tensor, Tensor, Tensor> batch_norm_cuda(const Tensor& self, const std::optional<Tensor>& weight_opt, const std::optional<Tensor>& bias_opt, const std::optional<Tensor>& running_mean_opt, const std::optional<Tensor>& running_var_opt, bool train, double momentum, double epsilon) {
 463:   auto output = at::empty_like(self);
 464:   int64_t n_input = self.size(1);
 465:   auto options = self.options().dtype(
 466:       at::toAccumulateType(self.scalar_type(), /*is_cuda=*/true));
 467:   auto save_mean = at::empty({n_input}, options);
 468:   auto save_invstd = at::empty({n_input}, options);
 469: 
 470:   at::native::batch_norm_cuda_out(
 471:       self,
 472:       weight_opt,
 473:       bias_opt,
 474:       running_mean_opt,
 475:       running_var_opt,
 476:       train,
 477:       momentum,
 478:       epsilon,
 479:       output,
 480:       save_mean,
 481:       save_invstd);
 482:   return std::make_tuple(output, save_mean, save_invstd);
 483: }
```
- EN: This block defines or continues the implementation of `batch_norm_cuda`.
- CN: 该代码块定义或继续实现 `batch_norm_cuda`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 485-506
```cpp
 485: std::tuple<Tensor, Tensor, Tensor, Tensor> _batch_norm_with_update_cuda(
 486:     const Tensor& input, const std::optional<Tensor>& weight_opt, const std::optional<Tensor>& bias_opt,
 487:     Tensor& running_mean, Tensor& running_var, double momentum, double eps) {
 488:   // See [Note: hacky wrapper removal for optional tensor]
 489:   c10::MaybeOwned<Tensor> weight_maybe_owned = at::borrow_from_optional_tensor(weight_opt);
 490:   const Tensor& weight = *weight_maybe_owned;
 491:   const Tensor& bias = bias_opt.value_or(Tensor());
 492:   Tensor output, save_mean, save_var, reserve;
 493: 
 494:   BatchNormBackend backend = _select_batch_norm_backend(input, weight, bias, running_mean, running_var, /*training*/true, eps);
 495:   if (backend == BatchNormBackend::Cudnn) {
 496:     return at::cudnn_batch_norm(input, weight, bias, running_mean, running_var, /*training*/true, momentum, eps);
 497:   }
 498:   if (backend == BatchNormBackend::Miopen) {
 499:     reserve = at::empty({0}, input.options().dtype(kByte));
 500:     std::tie(output, save_mean, save_var) =
 501:         at::miopen_batch_norm(input, weight, bias, running_mean, running_var, /*training*/true, momentum, eps);
 502:   } else {
 503:     reserve = at::empty({0}, input.options().dtype(kByte));
 504:     std::tie(output, save_mean, save_var) =
 505:         batch_norm_cuda(input, weight_opt, bias_opt, running_mean, running_var, /*training*/true, momentum, eps);
 506:   }
```
- EN: This block defines or continues the implementation of `_batch_norm_with_update_cuda`.
- CN: 该代码块定义或继续实现 `_batch_norm_with_update_cuda`。

### Lines 507-508
```cpp
 507:   return std::tuple<Tensor, Tensor, Tensor, Tensor>(output, save_mean, save_var, reserve);
 508: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 510-531
```cpp
 510: std::tuple<Tensor&, Tensor&, Tensor&, Tensor&> _batch_norm_with_update_cuda_out(
 511:     const Tensor& input, const std::optional<Tensor>& weight_opt, const std::optional<Tensor>& bias_opt,
 512:     Tensor& running_mean, Tensor& running_var, double momentum, double eps,
 513:     Tensor& out, Tensor& save_mean, Tensor& save_var, Tensor& reserve) {
 514:   // See [Note: hacky wrapper removal for optional tensor]
 515:   c10::MaybeOwned<Tensor> weight_maybe_owned = at::borrow_from_optional_tensor(weight_opt);
 516:   const Tensor& weight = *weight_maybe_owned;
 517:   const Tensor& bias = bias_opt.value_or(Tensor());
 518: 
 519:   BatchNormBackend backend = _select_batch_norm_backend(input, weight, bias, running_mean, running_var, /*training*/true, eps);
 520:   if (backend == BatchNormBackend::Cudnn) {
 521:     std::tie(out, save_mean, save_var, reserve) =
 522:         at::cudnn_batch_norm_out(out, save_mean, save_var, reserve, input, weight, bias, running_mean, running_var, /*training*/true, momentum, eps);
 523:   } else if (backend == BatchNormBackend::Miopen) {
 524:     std::tie(out, save_mean, save_var) =
 525:         at::miopen_batch_norm_out(out, save_mean, save_var, input, weight, bias, running_mean, running_var, /*training*/true, momentum, eps);
 526:   } else {
 527:     std::tie(out, save_mean, save_var) =
 528:       batch_norm_cuda_out(input, weight_opt, bias_opt, running_mean, running_var, /*update*/true, momentum, eps, out, save_mean, save_var);
 529:   }
 530:   return std::tuple<Tensor&, Tensor&, Tensor&, Tensor&>(out, save_mean, save_var, reserve);
 531: }
```
- EN: This block defines or continues the implementation of `_batch_norm_with_update_cuda_out`.
- CN: 该代码块定义或继续实现 `_batch_norm_with_update_cuda_out`。

### Lines 533-535
```cpp
 533: std::tuple<Tensor, Tensor, Tensor> _batch_norm_legit_cuda(const Tensor& self, const std::optional<Tensor>& weight_opt, const std::optional<Tensor>& bias_opt, Tensor& running_mean, Tensor& running_var, bool train, double momentum, double epsilon) {
 534:   return batch_norm_cuda(self, weight_opt, bias_opt, running_mean, running_var, train, momentum, epsilon);
 535: }
```
- EN: This block defines or continues the implementation of `_batch_norm_legit_cuda`.
- CN: 该代码块定义或继续实现 `_batch_norm_legit_cuda`。

### Lines 537-539
```cpp
 537: std::tuple<Tensor, Tensor, Tensor> _batch_norm_legit_no_stats_cuda(const Tensor& self, const std::optional<Tensor>& weight_opt, const std::optional<Tensor>& bias_opt, bool train, double momentum, double epsilon) {
 538:   return batch_norm_cuda(self, weight_opt, bias_opt, Tensor(), Tensor(), train, momentum, epsilon);
 539: }
```
- EN: This block defines or continues the implementation of `_batch_norm_legit_no_stats_cuda`.
- CN: 该代码块定义或继续实现 `_batch_norm_legit_no_stats_cuda`。

### Lines 541-543
```cpp
 541: std::tuple<Tensor&, Tensor&, Tensor&> _batch_norm_legit_cuda_out(const Tensor& self, const std::optional<Tensor>& weight_opt, const std::optional<Tensor>& bias_opt, Tensor& running_mean, Tensor& running_var, bool train, double momentum, double epsilon, Tensor& output, Tensor& save_mean, Tensor& save_invstd) {
 542:   return batch_norm_cuda_out(self, weight_opt, bias_opt, running_mean, running_var, train, momentum, epsilon, output, save_mean, save_invstd);
 543: }
```
- EN: This block defines or continues the implementation of `_batch_norm_legit_cuda_out`.
- CN: 该代码块定义或继续实现 `_batch_norm_legit_cuda_out`。

### Lines 545-547
```cpp
 545: std::tuple<Tensor&, Tensor&, Tensor&> _batch_norm_legit_no_stats_cuda_out(const Tensor& self, const std::optional<Tensor>& weight_opt, const std::optional<Tensor>& bias_opt, bool train, double momentum, double epsilon, Tensor& output, Tensor& save_mean, Tensor& save_invstd) {
 546:   return batch_norm_cuda_out(self, weight_opt, bias_opt, Tensor(), Tensor(), train, momentum, epsilon, output, save_mean, save_invstd);
 547: }
```
- EN: This block defines or continues the implementation of `_batch_norm_legit_no_stats_cuda_out`.
- CN: 该代码块定义或继续实现 `_batch_norm_legit_no_stats_cuda_out`。

### Lines 549-569
```cpp
 549: std::tuple<Tensor, Tensor, Tensor> _new_batch_norm_backward_cuda(
 550:     const Tensor& grad_output, const Tensor& input, const Tensor& weight,
 551:     const std::optional<Tensor>& running_mean_opt, const std::optional<Tensor>& running_var_opt,
 552:     const std::optional<Tensor>& save_mean_opt, const std::optional<Tensor>& save_var_opt,
 553:     bool update, double eps, std::array<bool,3> grad_input_mask, const Tensor& reserve) {
 554:   const Tensor& dummy_bias = at::empty(1);
 555:   const Tensor& running_mean = running_mean_opt.value_or(Tensor());
 556:   const Tensor& running_var = running_var_opt.value_or(Tensor());
 557:   const Tensor& save_mean = save_mean_opt.value_or(Tensor());
 558:   const Tensor& save_var = save_var_opt.value_or(Tensor());
 559: 
 560:   BatchNormBackend backend = _select_batch_norm_backend(input, weight, dummy_bias, running_mean, running_var, /*training*/true, eps);
 561: 
 562:   if (backend == BatchNormBackend::Cudnn) {
 563:     return at::cudnn_batch_norm_backward(input, grad_output, weight, running_mean, running_var, save_mean, save_var, eps, reserve);
 564:   } else if (backend == BatchNormBackend::Miopen) {
 565:     return at::miopen_batch_norm_backward(input, grad_output, weight, running_mean, running_var, save_mean, save_var, eps);
 566:   } else {
 567:     return batch_norm_backward_cuda(grad_output, input, weight, running_mean, running_var, save_mean, save_var, update, eps, grad_input_mask);
 568:   }
 569: }
```
- EN: This block defines or continues the implementation of `_new_batch_norm_backward_cuda`.
- CN: 该代码块定义或继续实现 `_new_batch_norm_backward_cuda`。

### Lines 571-592
```cpp
 571: std::tuple<Tensor, Tensor, Tensor> batch_norm_backward_cuda(const Tensor& grad_out, const Tensor& input, const std::optional<Tensor>& weight_opt, const std::optional<Tensor>& running_mean_opt, const std::optional<Tensor>& running_var_opt, const std::optional<Tensor>& save_mean_opt, const std::optional<Tensor>& save_invstd_opt, bool train, double epsilon, std::array<bool,3> grad_input_mask) {
 572:   // See [Note: hacky wrapper removal for optional tensor]
 573:   c10::MaybeOwned<Tensor> weight = at::borrow_from_optional_tensor(weight_opt);
 574:   c10::MaybeOwned<Tensor> save_mean = at::borrow_from_optional_tensor(save_mean_opt);
 575:   c10::MaybeOwned<Tensor> save_invstd = at::borrow_from_optional_tensor(save_invstd_opt);
 576:   c10::MaybeOwned<Tensor> running_mean = at::borrow_from_optional_tensor(running_mean_opt);
 577:   c10::MaybeOwned<Tensor> running_var = at::borrow_from_optional_tensor(running_var_opt);
 578: 
 579:   const bool needs_reduction = train || grad_input_mask[1] || grad_input_mask[2];
 580: 
 581:   // Fused reduction & elementwise kernel
 582:   if (needs_reduction && grad_input_mask[0] &&
 583:       !batch_norm_use_channels_last_kernels(input) &&
 584:       cuda::detail::canUse32BitIndexMath(input) &&
 585:       cuda::detail::canUse32BitIndexMath(grad_out)) {
 586:     return AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, input.scalar_type(),
 587:                                            "batch_norm_backward_cuda", [&] {
 588:       using accscalar_t = at::acc_type<scalar_t, true>;
 589:       const bool mixed_type = is_mixed_type(input, *weight, *running_mean, *running_var);
 590:       if (mixed_type) {
 591:           return batch_norm_backward_cuda_template<scalar_t, accscalar_t, int32_t>(
 592:               grad_out, input, *weight, *running_mean, *running_var,
```
- EN: This block defines or continues the implementation of `batch_norm_backward_cuda`.
- CN: 该代码块定义或继续实现 `batch_norm_backward_cuda`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 593-593
```cpp
 593:               *save_mean, *save_invstd, train, epsilon, grad_input_mask);
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 594-600
```cpp
 594:       } else {
 595:           return batch_norm_backward_cuda_template<scalar_t, scalar_t, int32_t>(
 596:               grad_out, input, *weight, *running_mean, *running_var,
 597:               *save_mean, *save_invstd, train, epsilon, grad_input_mask);
 598:       }
 599:     });
 600:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 602-604
```cpp
 602:   // NOTE: native_batch_norm always returns save_mean and save_invstd to be reused in backward.
 603:   // However, this is also called from cudnn_batch_norm in eval mode which doesn't give
 604:   // save_mean and save_invstd, so it needs recalculated.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 605-614
```cpp
 605:   const auto acc_type = at::toAccumulateType(input.scalar_type(), /*is_cuda=*/true);
 606:   Tensor mean;
 607:   TORCH_INTERNAL_ASSERT(save_mean->defined(), "save_mean should always be defined\n");
 608:   if (save_mean->numel() != 0) {
 609:     mean = *save_mean;
 610:   } else if (needs_reduction) {
 611:     TORCH_CHECK(!train && running_mean->defined());
 612:     mean = (running_mean->scalar_type() == acc_type) ?
 613:         *running_mean : running_mean->to(acc_type);
 614:   }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 616-625
```cpp
 616:   Tensor invstd;
 617:   TORCH_INTERNAL_ASSERT(save_invstd->defined(), "save_invstd should always be defined\n");
 618:   if (save_invstd->numel() != 0) {
 619:     invstd = *save_invstd;
 620:   } else {
 621:     TORCH_CHECK(!train && running_var->defined());
 622:     auto n_channels = input.sizes()[1];
 623:     invstd = at::empty({n_channels}, input.options().dtype(acc_type));
 624:     batch_norm_calc_invstd(invstd, *running_var, epsilon);
 625:   }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 627-633
```cpp
 627:   Tensor sum_dy, sum_dy_xmu, grad_weight, grad_bias;
 628:   if (needs_reduction) {
 629:     std::tie(sum_dy, sum_dy_xmu, grad_weight, grad_bias) =
 630:         batch_norm_backward_reduce_cuda(
 631:             grad_out, input, mean, invstd, *weight,
 632:             grad_input_mask[0], grad_input_mask[1], grad_input_mask[2]);
 633:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 635-645
```cpp
 635:   Tensor grad_input;
 636:   if (grad_input_mask[0]) {
 637:     if (train) {
 638:       // NOTE: sum_dy and sum_dy_xmy are defined, as train implies needs_reduction
 639:       grad_input = batch_norm_elementwise_backward_train(
 640:           grad_out, input, mean, invstd, *weight, sum_dy, sum_dy_xmu);
 641:     } else {
 642:       grad_input = batch_norm_elementwise_backward_eval(
 643:           grad_out, input, invstd, *weight);
 644:     }
 645:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 647-648
```cpp
 647:   return std::make_tuple(grad_input, grad_weight, grad_bias);
 648: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 650-671
```cpp
 650: std::tuple<Tensor, Tensor> batch_norm_stats_cuda(const Tensor& self, double epsilon) {
 651:   auto options = self.options().dtype(
 652:       at::toAccumulateType(self.scalar_type(), /*is_cuda=*/true));
 653:   auto n_channels = self.size(1);
 654:   auto save_mean = at::empty({n_channels}, options);
 655:   auto save_invstd = at::empty({n_channels}, options);
 656: 
 657:   bool use_channels_last_kernel = batch_norm_use_channels_last_kernels(self);
 658:   AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16,
 659:                                   self.scalar_type(), "batch_norm_stats_cuda", [&] {
 660:     if (cuda::detail::canUse32BitIndexMath(self)) {
 661:       if (use_channels_last_kernel) {
 662:         batch_norm_stats_channels_last_cuda_template<scalar_t, InvStd>(
 663:             save_mean, save_invstd, self, epsilon);
 664:       } else {
 665:         batch_norm_stats_cuda_template<scalar_t, int32_t, InvStd>(
 666:             save_mean, save_invstd, self, epsilon);
 667:       }
 668:     } else {
 669:       batch_norm_stats_cuda_template<scalar_t, int64_t, InvStd>(
 670:           save_mean, save_invstd, self, epsilon);
 671:     }
```
- EN: This block defines or continues the implementation of `batch_norm_stats_cuda`.
- CN: 该代码块定义或继续实现 `batch_norm_stats_cuda`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 672-674
```cpp
 672:   });
 673:   return std::tuple<Tensor, Tensor>(save_mean, save_invstd);
 674: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 676-684
```cpp
 676: Tensor batch_norm_elemt_cuda(
 677:     const Tensor& self, const std::optional<Tensor>& weight_opt,
 678:     const std::optional<Tensor>& bias_opt, const Tensor& mean,
 679:     const Tensor& invstd, double epsilon) {
 680:   auto output = at::empty_like(self);
 681:   // FIXME: Epsilon parameter isn't required, we don't take the reciprocal
 682:   batch_norm_elementwise(output, self, weight_opt, bias_opt, mean, invstd);
 683:   return output;
 684: }
```
- EN: This block defines or continues the implementation of `batch_norm_elemt_cuda`.
- CN: 该代码块定义或继续实现 `batch_norm_elemt_cuda`。

### Lines 686-691
```cpp
 686: Tensor& batch_norm_elemt_cuda_out(const Tensor& self, const std::optional<Tensor>& weight_opt, const std::optional<Tensor>& bias_opt,
 687:                                   const Tensor& mean, const Tensor& invstd, double epsilon, Tensor& output) {
 688:   // FIXME: Epsilon parameter isn't required, we don't take the reciprocal
 689:   batch_norm_elementwise(output, self, weight_opt, bias_opt, mean, invstd);
 690:   return output;
 691: }
```
- EN: This block defines or continues the implementation of `batch_norm_elemt_cuda_out`.
- CN: 该代码块定义或继续实现 `batch_norm_elemt_cuda_out`。

### Lines 693-693
```cpp
 693: // accepting input(self) here to determine template data types, since running_mean/running_var are optional
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 694-704
```cpp
 694: std::tuple<Tensor, Tensor> batch_norm_gather_stats_cuda(const Tensor& self, const Tensor& mean, const Tensor& invstd, const std::optional<Tensor>& running_mean_opt, const std::optional<Tensor>& running_var_opt, double momentum, double epsilon, int64_t count) {
 695:   // See [Note: hacky wrapper removal for optional tensor]
 696:   c10::MaybeOwned<Tensor> running_mean_maybe_owned = at::borrow_from_optional_tensor(running_mean_opt);
 697:   const Tensor& running_mean = *running_mean_maybe_owned;
 698:   const Tensor& running_var = running_var_opt.value_or(Tensor());
 699: 
 700:   std::vector<int64_t> counts(mean.size(0), count);
 701:   Tensor counts_ = at::from_blob((void*)counts.data(), {(int64_t)counts.size()}, self.options().dtype(at::kLong).device(at::kCPU));
 702:   counts_ = counts_.to(self.device()).to(running_mean.defined() ? running_mean.dtype() : self.dtype());
 703:   return batch_norm_gather_stats_with_counts_cuda(self, mean, invstd, running_mean, running_var, momentum, epsilon, counts_);
 704: }
```
- EN: This block defines or continues the implementation of `batch_norm_gather_stats_cuda`.
- CN: 该代码块定义或继续实现 `batch_norm_gather_stats_cuda`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 707-724
```cpp
 707: std::tuple<Tensor, Tensor> batch_norm_gather_stats_with_counts_cuda(
 708:     const Tensor& self, const Tensor& mean, const Tensor& invstd, const std::optional<Tensor>& running_mean_opt /* optional */, const std::optional<Tensor>& running_var_opt /* optional */, double momentum, double epsilon, const Tensor& counts) {
 709:   // See [Note: hacky wrapper removal for optional tensor]
 710:   c10::MaybeOwned<Tensor> running_mean_maybe_owned = at::borrow_from_optional_tensor(running_mean_opt);
 711:   const Tensor& running_mean = *running_mean_maybe_owned;
 712:   const Tensor& running_var = running_var_opt.value_or(Tensor());
 713: 
 714: 
 715:   auto scalar_type = running_mean.defined() ? running_mean.scalar_type() : self.scalar_type();
 716:   return AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, scalar_type, "batch_norm_update_stats_cuda", [&] {
 717:     using accscalar_t = at::acc_type<scalar_t, true>;
 718:     if (cuda::detail::canUse32BitIndexMath(self)) {
 719:       return batch_norm_gather_stats_cuda_template<scalar_t, accscalar_t, int32_t>(mean, invstd, running_mean, running_var, momentum, epsilon, counts);
 720:     } else {
 721:       return batch_norm_gather_stats_cuda_template<scalar_t, accscalar_t, int64_t>(mean, invstd, running_mean, running_var, momentum, epsilon, counts);
 722:     }
 723:   });
 724: }
```
- EN: This block defines or continues the implementation of `batch_norm_gather_stats_with_counts_cuda`.
- CN: 该代码块定义或继续实现 `batch_norm_gather_stats_with_counts_cuda`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 726-747
```cpp
 726: std::tuple<Tensor, Tensor, Tensor, Tensor> batch_norm_backward_reduce_cuda(const Tensor& grad_output, const Tensor& input, const Tensor& mean, const Tensor& invstd, const std::optional<Tensor>& weight_opt, bool input_g, bool weight_g, bool bias_g) {
 727:   // See [Note: hacky wrapper removal for optional tensor]
 728:   c10::MaybeOwned<Tensor> weight_maybe_owned = at::borrow_from_optional_tensor(weight_opt);
 729:   const Tensor& weight = *weight_maybe_owned;
 730: 
 731:   if (at::cuda::detail::canUse32BitIndexMath(grad_output) &&
 732:       batch_norm_use_channels_last_kernels(grad_output) &&
 733:       batch_norm_use_channels_last_kernels(input) &&
 734:       (!weight.defined() || weight.is_contiguous()) &&
 735:       mean.is_contiguous() && invstd.is_contiguous()){
 736:     return batch_norm_backward_reduce_cuda_channels_last_template(
 737:         grad_output, input, mean, invstd, weight, input_g, weight_g, bias_g);
 738:   }
 739: 
 740:   return AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, grad_output.scalar_type(), "batch_norm_backward_reduce", [&] {
 741:     auto mean_st = mean.dtype();
 742:     auto invstd_st = invstd.dtype();
 743:     TORCH_CHECK(mean_st == invstd_st, "mean and invstd need to have the same data types");
 744:     const bool mixed_type = is_mixed_type(input, weight);
 745:     using accscalar_t = at::acc_type<scalar_t, true>;
 746: 
 747:     if (cuda::detail::canUse32BitIndexMath(grad_output)) {
```
- EN: This block defines or continues the implementation of `batch_norm_backward_reduce_cuda`.
- CN: 该代码块定义或继续实现 `batch_norm_backward_reduce_cuda`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 748-761
```cpp
 748:       if (mixed_type) {
 749:         return batch_norm_backward_reduce_cuda_template<scalar_t, accscalar_t, int32_t>(grad_output, input, mean, invstd, weight, input_g, weight_g, bias_g);
 750:       } else {
 751:         return batch_norm_backward_reduce_cuda_template<scalar_t, scalar_t, int32_t>(grad_output, input, mean, invstd, weight, input_g, weight_g, bias_g);
 752:       }
 753:     } else {
 754:       if (mixed_type) {
 755:         return batch_norm_backward_reduce_cuda_template<scalar_t, accscalar_t, int64_t>(grad_output, input, mean, invstd, weight, input_g, weight_g, bias_g);
 756:       } else {
 757:         return batch_norm_backward_reduce_cuda_template<scalar_t, scalar_t, int64_t>(grad_output, input, mean, invstd, weight, input_g, weight_g, bias_g);
 758:       }
 759:     }
 760:   });
 761: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 763-784
```cpp
 763: Tensor batch_norm_backward_elemt_cuda(const Tensor& self, const Tensor& input, const Tensor& mean, const Tensor& invstd, const std::optional<Tensor>& weight_opt, const Tensor& sum_dy, const Tensor& sum_dy_xmu, const Tensor& count) {
 764:   // See [Note: hacky wrapper removal for optional tensor]
 765:   c10::MaybeOwned<Tensor> weight_maybe_owned = at::borrow_from_optional_tensor(weight_opt);
 766:   const Tensor& weight = *weight_maybe_owned;
 767: 
 768:   if (at::cuda::detail::canUse32BitIndexMath(self) &&
 769:       batch_norm_use_channels_last_kernels(self) &&
 770:       batch_norm_use_channels_last_kernels(input))  {
 771:     return batch_norm_backward_elemt_channels_last_cuda_template(self, input, mean, invstd, weight, sum_dy, sum_dy_xmu, count);
 772:   }
 773: 
 774:   return AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, self.scalar_type(), "batch_norm_backward_elemt", [&] {
 775:     auto mean_st = mean.dtype();
 776:     auto invstd_st = invstd.dtype();
 777:     TORCH_CHECK(mean_st == invstd_st, "mean and invstd need to have the same data types");
 778:     bool is_half_float = std::is_same_v<scalar_t, at::Half> && mean_st == at::kFloat;
 779:     bool is_bfloat16_float = std::is_same_v<scalar_t, at::BFloat16> && mean_st == at::kFloat;
 780:     using accscalar_t = at::acc_type<scalar_t, true>;
 781:     if (cuda::detail::canUse32BitIndexMath(self)) {
 782:       if (is_half_float || is_bfloat16_float) {
 783:         return batch_norm_backward_elemt_cuda_template<scalar_t, accscalar_t, int32_t>(self, input, mean, invstd, weight, sum_dy, sum_dy_xmu, count);
 784:       } else {
```
- EN: This block defines or continues the implementation of `batch_norm_backward_elemt_cuda`.
- CN: 该代码块定义或继续实现 `batch_norm_backward_elemt_cuda`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 785-795
```cpp
 785:         return batch_norm_backward_elemt_cuda_template<scalar_t, scalar_t, int32_t>(self, input, mean, invstd, weight, sum_dy, sum_dy_xmu, count);
 786:       }
 787:     } else {
 788:       if (is_half_float || is_bfloat16_float) {
 789:         return batch_norm_backward_elemt_cuda_template<scalar_t, accscalar_t, int64_t>(self, input, mean, invstd, weight, sum_dy, sum_dy_xmu, count);
 790:       } else {
 791:         return batch_norm_backward_elemt_cuda_template<scalar_t, scalar_t, int64_t>(self, input, mean, invstd, weight, sum_dy, sum_dy_xmu, count);
 792:       }
 793:     }
 794:   });
 795: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 797-818
```cpp
 797: std::tuple<Tensor, Tensor> batch_norm_update_stats_cuda(
 798:     const Tensor& self, const std::optional<Tensor>& running_mean_opt,
 799:     const std::optional<Tensor>& running_var_opt, double momentum) {
 800:   c10::MaybeOwned<Tensor> running_mean = at::borrow_from_optional_tensor(running_mean_opt);
 801:   c10::MaybeOwned<Tensor> running_var = at::borrow_from_optional_tensor(running_var_opt);
 802: 
 803:   const int64_t n_input = self.size(1);
 804: 
 805:   TORCH_CHECK(self.numel() != 0, "input tensor must have at least one element, but got input_sizes = ", self.sizes());
 806:   auto options = self.options().dtype(
 807:       at::toAccumulateType(self.scalar_type(), /*is_cuda=*/true));
 808:   auto save_mean = at::empty({n_input}, options);
 809:   auto save_var = at::empty({n_input}, options);
 810: 
 811:   batch_norm_mean_var(self, save_mean, save_var);
 812:   TORCH_CHECK(running_mean->defined() == running_var->defined());
 813:   if (running_mean->defined()) {
 814:     const int64_t N = self.numel() / save_mean.numel();
 815:     batch_norm_update_stats(save_mean, save_var, *running_mean, *running_var, momentum, N);
 816:   }
 817:   return std::tuple<Tensor, Tensor>(save_mean, save_var);
 818: }
```
- EN: This block defines or continues the implementation of `batch_norm_update_stats_cuda`.
- CN: 该代码块定义或继续实现 `batch_norm_update_stats_cuda`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 820-820
```cpp
 820: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `TensorIterator` normalizes tensor shapes/strides and drives elementwise CUDA traversal. / `TensorIterator` 统一张量形状与步长，并驱动逐元素 CUDA 遍历。
- `TensorIteratorConfig` builds iterator state before launch. / `TensorIteratorConfig` 在启动前构建迭代器状态。
- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `gpu_kernel` applies a functor over iterator elements on CUDA. / `gpu_kernel` 在 CUDA 上对迭代器元素应用函数对象。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- Thrust utilities supply STL-like CUDA algorithms and containers. / Thrust 工具提供类似 STL 的 CUDA 算法与容器。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。
- cuDNN calls delegate work to NVIDIA's tuned deep-learning kernels. / cuDNN 调用把工作委托给 NVIDIA 优化过的深度学习内核。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/cuda/detail/IndexUtils.cuh>`
  - `<ATen/detail/CUDAHooksInterface.h>`
  - `<ATen/native/Normalization.h>`
  - `<ATen/native/TensorIterator.h>`
  - `<ATen/native/ReduceOps.h>`
  - `<ATen/native/Resize.h>`
  - `<ATen/native/cuda/Loops.cuh>`
  - `<ATen/native/cuda/Resize.h>`
  - `<ATen/native/cuda/Normalization.cuh>`
  - `<c10/cuda/CUDAMathCompat.h>`
  - `<ATen/Functions.h>`
  - `<ATen/NativeFunctions.h>`
- Runtime symbols / 运行时符号:
  - `gpu_kernel`
  - `TensorIterator`
  - `TensorIteratorConfig`
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `thrust::tuple`
  - `at::cuda::detail::canUse32BitIndexMath`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
