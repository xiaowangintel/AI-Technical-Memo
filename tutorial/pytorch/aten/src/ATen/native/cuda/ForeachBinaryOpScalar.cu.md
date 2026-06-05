# ForeachBinaryOpScalar.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/ForeachBinaryOpScalar.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `foreach_binary_op`, `foreach_binary_op_`, `all_types_complex_bool_half_bfloat16`, `all_types_complex_bool_half_bfloat16_`.
- 用途（中文）: 实现与 `foreach_binary_op`, `foreach_binary_op_`, `all_types_complex_bool_half_bfloat16`, `all_types_complex_bool_half_bfloat16_` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/Dispatch.h>
   3: #include <ATen/native/BinaryOps.h>
   4: #include <ATen/native/ForeachUtils.h>
   5: #include <ATen/native/cuda/ForeachFunctors.cuh>
   6: #include <ATen/native/cuda/ForeachMinMaxFunctors.cuh>
   7: 
   8: #ifndef AT_PER_OPERATOR_HEADERS
   9: #include <ATen/NativeFunctions.h>
  10: #else
  11: #include <ATen/ops/_foreach_add_native.h>
  12: #include <ATen/ops/_foreach_clamp_max_native.h>
  13: #include <ATen/ops/_foreach_clamp_min_native.h>
  14: #include <ATen/ops/_foreach_div_native.h>
  15: #include <ATen/ops/_foreach_mul_native.h>
  16: #include <ATen/ops/_foreach_pow_native.h>
  17: #include <ATen/ops/_foreach_sub_native.h>
  18: 
  19: #include <ATen/ops/empty_like_native.h>
  20: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/Dispatch.h>`, `<ATen/native/BinaryOps.h>`, `<ATen/native/ForeachUtils.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/Dispatch.h>`, `<ATen/native/BinaryOps.h>`, `<ATen/native/ForeachUtils.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 22-43
```cpp
  22: namespace at::native {
  23: 
  24: template <typename T, template <class> class Op>
  25: std::vector<Tensor> foreach_binary_op(
  26:     TensorList tensors,
  27:     const Scalar& scalar) {
  28:   std::vector<std::vector<at::Tensor>> tensor_lists;
  29:   std::vector<at::Tensor> vec_res;
  30:   vec_res.reserve(tensors.size());
  31:   for (const auto& t : tensors) {
  32:     vec_res.emplace_back(at::native::empty_like(t));
  33:   }
  34: 
  35:   tensor_lists.emplace_back(tensors.vec());
  36:   tensor_lists.emplace_back(std::move(vec_res));
  37: 
  38:   using opmath_t = at::opmath_type<T>;
  39:   multi_tensor_apply<2>(
  40:       tensor_lists,
  41:       BinaryOpScalarFunctor<
  42:           T,
  43:           /* depth */ 2,
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `foreach_binary_op`.
- CN: 该代码块定义或继续实现 `foreach_binary_op`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 44-45
```cpp
  44:           /* r_args_depth */ 1,
  45:           /* res_arg_index */ 1>(),
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 46-49
```cpp
  46:       Op<opmath_t>(),
  47:       scalar.to<opmath_t>());
  48:   return std::move(tensor_lists[1]);
  49: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 51-67
```cpp
  51: template <typename T, template <class> class Op>
  52: void foreach_binary_op_(TensorList tensors, const Scalar& scalar) {
  53:   std::vector<std::vector<at::Tensor>> tensor_lists;
  54:   tensor_lists.emplace_back(tensors.vec());
  55: 
  56:   using opmath_t = at::opmath_type<T>;
  57:   multi_tensor_apply<1>(
  58:       tensor_lists,
  59:       BinaryOpScalarFunctor<
  60:           T,
  61:           /* depth */ 1,
  62:           /* r_args_depth */ 1,
  63:           /* res_arg_index */ 0>(),
  64:       Op<opmath_t>(),
  65:       scalar.to<opmath_t>());
  66:   increment_version(tensors);
  67: }
```
- EN: This block defines or continues the implementation of `foreach_binary_op_`.
- CN: 该代码块定义或继续实现 `foreach_binary_op_`。

### Lines 69-80
```cpp
  69: template <template <class> class Op>
  70: std::vector<Tensor> all_types_complex_bool_half_bfloat16(
  71:     TensorList tensors,
  72:     const Scalar& scalar) {
  73:   return AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3(
  74:       kBool,
  75:       kHalf,
  76:       kBFloat16,
  77:       tensors[0].scalar_type(),
  78:       "foreach_binary_op_scalar_cuda",
  79:       [&]() { return foreach_binary_op<scalar_t, Op>(tensors, scalar); });
  80: }
```
- EN: This block defines or continues the implementation of `all_types_complex_bool_half_bfloat16`.
- CN: 该代码块定义或继续实现 `all_types_complex_bool_half_bfloat16`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 82-93
```cpp
  82: template <template <class> class Op>
  83: void all_types_complex_bool_half_bfloat16_(
  84:     TensorList tensors,
  85:     const Scalar& scalar) {
  86:   AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3(
  87:       kBool,
  88:       kHalf,
  89:       kBFloat16,
  90:       tensors[0].scalar_type(),
  91:       "foreach_binary_op_scalar_cuda_",
  92:       [&]() { foreach_binary_op_<scalar_t, Op>(tensors, scalar); });
  93: }
```
- EN: This block defines or continues the implementation of `all_types_complex_bool_half_bfloat16_`.
- CN: 该代码块定义或继续实现 `all_types_complex_bool_half_bfloat16_`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 95-105
```cpp
  95: template <template <class> class Op>
  96: std::vector<Tensor> all_types_half_bfloat16(
  97:     TensorList tensors,
  98:     const Scalar& scalar) {
  99:   return AT_DISPATCH_ALL_TYPES_AND2(
 100:       kHalf,
 101:       kBFloat16,
 102:       tensors[0].scalar_type(),
 103:       "foreach_binary_op_scalar_cuda",
 104:       [&]() { return foreach_binary_op<scalar_t, Op>(tensors, scalar); });
 105: }
```
- EN: This block defines or continues the implementation of `all_types_half_bfloat16`.
- CN: 该代码块定义或继续实现 `all_types_half_bfloat16`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 107-115
```cpp
 107: template <template <class> class Op>
 108: void all_types_half_bfloat16_(TensorList tensors, const Scalar& scalar) {
 109:   AT_DISPATCH_ALL_TYPES_AND2(
 110:       kHalf,
 111:       kBFloat16,
 112:       tensors[0].scalar_type(),
 113:       "foreach_binary_op_scalar_cuda_",
 114:       [&]() { foreach_binary_op_<scalar_t, Op>(tensors, scalar); });
 115: }
```
- EN: This block defines or continues the implementation of `all_types_half_bfloat16_`.
- CN: 该代码块定义或继续实现 `all_types_half_bfloat16_`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 117-127
```cpp
 117: template <template <class> class Op>
 118: std::vector<Tensor> all_types_complex_half_bfloat16(
 119:     TensorList tensors,
 120:     const Scalar& scalar) {
 121:   return AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND2(
 122:       kHalf,
 123:       kBFloat16,
 124:       tensors[0].scalar_type(),
 125:       "foreach_binary_op_scalar_cuda",
 126:       [&]() { return foreach_binary_op<scalar_t, Op>(tensors, scalar); });
 127: }
```
- EN: This block defines or continues the implementation of `all_types_complex_half_bfloat16`.
- CN: 该代码块定义或继续实现 `all_types_complex_half_bfloat16`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 129-139
```cpp
 129: template <template <class> class Op>
 130: void all_types_complex_half_bfloat16_(
 131:     TensorList tensors,
 132:     const Scalar& scalar) {
 133:   AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND2(
 134:       kHalf,
 135:       kBFloat16,
 136:       tensors[0].scalar_type(),
 137:       "foreach_binary_op_scalar_cuda_",
 138:       [&]() { foreach_binary_op_<scalar_t, Op>(tensors, scalar); });
 139: }
```
- EN: This block defines or continues the implementation of `all_types_complex_half_bfloat16_`.
- CN: 该代码块定义或继续实现 `all_types_complex_half_bfloat16_`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 141-141
```cpp
 141: #define FOREACH_BINARY_OP_SCALAR(FUNCTION, NAME, OP, DIVISION_OP)     \
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 142-162
```cpp
 142:   void foreach_tensor_##NAME##_scalar_kernel_cuda_(                   \
 143:       TensorList tensors, const Scalar& scalar) {                     \
 144:     check_foreach_api_restrictions(tensors);                          \
 145:     if (!can_use_fast_route(tensors, scalar, DIVISION_OP)) {          \
 146:       return at::native::foreach_tensor_##NAME##_scalar_kernel_slow_( \
 147:           tensors, scalar);                                           \
 148:     }                                                                 \
 149:                                                                       \
 150:     FUNCTION##_<OP>(tensors, scalar);                                 \
 151:   }                                                                   \
 152:                                                                       \
 153:   std::vector<Tensor> foreach_tensor_##NAME##_scalar_kernel_cuda(     \
 154:       TensorList tensors, const Scalar& scalar) {                     \
 155:     check_foreach_api_restrictions(tensors);                          \
 156:     if (!can_use_fast_route(tensors, scalar, DIVISION_OP)) {          \
 157:       return at::native::foreach_tensor_##NAME##_scalar_kernel_slow(  \
 158:           tensors, scalar);                                           \
 159:     }                                                                 \
 160:                                                                       \
 161:     return FUNCTION<OP>(tensors, scalar);                             \
 162:   }
```
- EN: This block defines or continues the implementation of `_scalar_kernel_cuda_`, `_scalar_kernel_cuda`.
- CN: 该代码块定义或继续实现 `_scalar_kernel_cuda_`, `_scalar_kernel_cuda`。

### Lines 164-177
```cpp
 164: FOREACH_BINARY_OP_SCALAR(
 165:     all_types_complex_bool_half_bfloat16,
 166:     add,
 167:     std::plus,
 168:     /*div_op*/ false);
 169: FOREACH_BINARY_OP_SCALAR(
 170:     all_types_complex_bool_half_bfloat16,
 171:     mul,
 172:     std::multiplies,
 173:     /*div_op*/ false);
 174: // See [Why is foreach_pow's division_op=true?]
 175: FOREACH_BINARY_OP_SCALAR(
 176:     all_types_complex_half_bfloat16,
 177:     pow,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 178-189
```cpp
 178:     power_functor,
 179:     /*div_op*/ true);
 180: std::vector<Tensor> foreach_scalar_pow_list_kernel_cuda(
 181:     const Scalar& scalar,
 182:     TensorList exponent) {
 183:   check_foreach_api_restrictions(exponent);
 184:   if (!can_use_fast_route(exponent)) {
 185:     return at::native::foreach_scalar_pow_list_kernel_slow(scalar, exponent);
 186:   }
 187:   return all_types_complex_half_bfloat16<reverse_power_functor>(
 188:       exponent, scalar);
 189: }
```
- EN: This block defines or continues the implementation of `foreach_scalar_pow_list_kernel_cuda`.
- CN: 该代码块定义或继续实现 `foreach_scalar_pow_list_kernel_cuda`。

### Lines 191-197
```cpp
 191: // In the case of division, integer inputs will result in float.
 192: // Currently multi tensor apply can only return result of the same type as
 193: // input.
 194: //
 195: // Implement via multiply with reciprocal as it's faster and makes it match
 196: // the behavior of regular Tensor div by scalar.  Loses one bit of
 197: // precision.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 198-208
```cpp
 198: Scalar scalar_reciprocal(const Scalar& scalar) {
 199:   if (scalar.isFloatingPoint()) {
 200:     return Scalar(1. / scalar.toDouble());
 201:   } else if (scalar.isIntegral(/*includeBool*/ true)) {
 202:     return Scalar(1. / static_cast<double>(scalar.toLong()));
 203:   } else if (scalar.isComplex()) {
 204:     return Scalar(1. / scalar.toComplexDouble());
 205:   }
 206:   TORCH_INTERNAL_ASSERT(
 207:       false, "division with ", scalar.type(), " not supported");
 208: }
```
- EN: This block defines or continues the implementation of `scalar_reciprocal`.
- CN: 该代码块定义或继续实现 `scalar_reciprocal`。

### Lines 210-221
```cpp
 210: void foreach_tensor_div_scalar_kernel_cuda_(
 211:     TensorList tensors,
 212:     const Scalar& scalar) {
 213:   check_foreach_api_restrictions(tensors);
 214:   if (!can_use_fast_route(tensors, scalar, true)) {
 215:     return at::native::foreach_tensor_mul_scalar_kernel_slow_(
 216:         tensors, scalar_reciprocal(scalar));
 217:   }
 218: 
 219:   all_types_complex_bool_half_bfloat16_<std::multiplies>(
 220:       tensors, scalar_reciprocal(scalar));
 221: }
```
- EN: This block defines or continues the implementation of `foreach_tensor_div_scalar_kernel_cuda_`.
- CN: 该代码块定义或继续实现 `foreach_tensor_div_scalar_kernel_cuda_`。

### Lines 223-234
```cpp
 223: std::vector<Tensor> foreach_tensor_div_scalar_kernel_cuda(
 224:     TensorList tensors,
 225:     const Scalar& scalar) {
 226:   check_foreach_api_restrictions(tensors);
 227:   if (!can_use_fast_route(tensors, scalar, true)) {
 228:     return at::native::foreach_tensor_mul_scalar_kernel_slow(
 229:         tensors, scalar_reciprocal(scalar));
 230:   }
 231: 
 232:   return all_types_complex_bool_half_bfloat16<std::multiplies>(
 233:       tensors, scalar_reciprocal(scalar));
 234: }
```
- EN: This block defines or continues the implementation of `foreach_tensor_div_scalar_kernel_cuda`.
- CN: 该代码块定义或继续实现 `foreach_tensor_div_scalar_kernel_cuda`。

### Lines 236-237
```cpp
 236: // In the case of subtraction, we dont allow scalar to be boolean following the
 237: // torch.sub logic
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 238-255
```cpp
 238: void foreach_tensor_sub_scalar_kernel_cuda_(
 239:     TensorList tensors,
 240:     const Scalar& scalar) {
 241:   check_foreach_api_restrictions(tensors);
 242:   at::native::sub_check(tensors[0], scalar);
 243: 
 244:   if (!can_use_fast_route(tensors, scalar)) {
 245:     return at::native::foreach_tensor_sub_scalar_kernel_slow_(tensors, scalar);
 246:   }
 247: 
 248:   AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3(
 249:       kBool,
 250:       kHalf,
 251:       kBFloat16,
 252:       tensors[0].scalar_type(),
 253:       "foreach_binary_op_scalar_cuda_",
 254:       [&]() { foreach_binary_op_<scalar_t, std::minus>(tensors, scalar); });
 255: }
```
- EN: This block defines or continues the implementation of `foreach_tensor_sub_scalar_kernel_cuda_`.
- CN: 该代码块定义或继续实现 `foreach_tensor_sub_scalar_kernel_cuda_`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 257-276
```cpp
 257: std::vector<Tensor> foreach_tensor_sub_scalar_kernel_cuda(
 258:     TensorList tensors,
 259:     const Scalar& scalar) {
 260:   check_foreach_api_restrictions(tensors);
 261:   at::native::sub_check(tensors[0], scalar);
 262: 
 263:   if (!can_use_fast_route(tensors, scalar)) {
 264:     return at::native::foreach_tensor_sub_scalar_kernel_slow(tensors, scalar);
 265:   }
 266: 
 267:   return AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3(
 268:       kBool,
 269:       kHalf,
 270:       kBFloat16,
 271:       tensors[0].scalar_type(),
 272:       "foreach_binary_op_scalar_cuda",
 273:       [&]() {
 274:         return foreach_binary_op<scalar_t, std::minus>(tensors, scalar);
 275:       });
 276: }
```
- EN: This block defines or continues the implementation of `foreach_tensor_sub_scalar_kernel_cuda`.
- CN: 该代码块定义或继续实现 `foreach_tensor_sub_scalar_kernel_cuda`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 278-279
```cpp
 278: // NOTE(crcrpar): `all_types_half_bfloat16` does not cover bool, so temporarily
 279: // set `division_op` to true.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 280-281
```cpp
 280: FOREACH_BINARY_OP_SCALAR(all_types_half_bfloat16, clamp_max, minimum, true);
 281: FOREACH_BINARY_OP_SCALAR(all_types_half_bfloat16, clamp_min, maximum, true);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 283-283
```cpp
 283: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/Dispatch.h>`
  - `<ATen/native/BinaryOps.h>`
  - `<ATen/native/ForeachUtils.h>`
  - `<ATen/native/cuda/ForeachFunctors.cuh>`
  - `<ATen/native/cuda/ForeachMinMaxFunctors.cuh>`
  - `<ATen/NativeFunctions.h>`
  - `<ATen/ops/_foreach_add_native.h>`
  - `<ATen/ops/_foreach_clamp_max_native.h>`
  - `<ATen/ops/_foreach_clamp_min_native.h>`
  - `<ATen/ops/_foreach_div_native.h>`
  - `<ATen/ops/_foreach_mul_native.h>`
  - `<ATen/ops/_foreach_pow_native.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3`
  - `AT_DISPATCH_ALL_TYPES_AND2`
  - `AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND2`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
