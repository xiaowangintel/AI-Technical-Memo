# ForeachTernaryOp.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/ForeachTernaryOp.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `foreach_tensor_lerp_ternary_cuda`, `foreach_tensor_lerp_ternary_cuda_`, `foreach_tensor_lerp_list_cuda`, `foreach_tensor_lerp_list_cuda_`.
- 用途（中文）: 实现与 `foreach_tensor_lerp_ternary_cuda`, `foreach_tensor_lerp_ternary_cuda_`, `foreach_tensor_lerp_list_cuda`, `foreach_tensor_lerp_list_cuda_` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/Dispatch.h>
   3: #include <ATen/native/ForeachUtils.h>
   4: #include <ATen/native/Lerp.h>
   5: #include <ATen/native/cuda/ForeachFunctors.cuh>
   6: #include <ATen/native/cuda/MultiTensorApply.cuh>
   7: 
   8: #ifndef AT_PER_OPERATOR_HEADERS
   9: #include <ATen/NativeFunctions.h>
  10: #else
  11: #include <ATen/ops/_foreach_lerp_native.h>
  12: 
  13: #include <ATen/ops/empty_like_native.h>
  14: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/Dispatch.h>`, `<ATen/native/ForeachUtils.h>`, `<ATen/native/Lerp.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/Dispatch.h>`, `<ATen/native/ForeachUtils.h>`, `<ATen/native/Lerp.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 16-37
```cpp
  16: namespace at::native {
  17: 
  18: template <typename T>
  19: struct LerpFunctor {
  20:   inline C10_DEVICE T operator()(const T self, const T end, const T weight) {
  21:     return lerp(self, end, weight);
  22:   }
  23: };
  24: 
  25: std::vector<at::Tensor> foreach_tensor_lerp_ternary_cuda(
  26:     TensorList tensors1,
  27:     TensorList tensors2,
  28:     TensorList tensors3) {
  29:   check_foreach_api_restrictions(tensors1, tensors2, tensors3);
  30:   if (!can_use_fast_route({tensors1, tensors2, tensors3}, {}, true)) {
  31:     return foreach_tensor_ternary_lerp_slow(tensors1, tensors2, tensors3);
  32:   }
  33: 
  34:   std::vector<at::Tensor> vec_res;
  35:   vec_res.reserve(tensors1.size());
  36:   for (const auto& t : tensors1) {
  37:     vec_res.emplace_back(at::native::empty_like(t));
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `foreach_tensor_lerp_ternary_cuda`.
- CN: 该代码块定义或继续实现 `foreach_tensor_lerp_ternary_cuda`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 38-40
```cpp
  38:   }
  39:   std::vector<std::vector<at::Tensor>> tensor_lists{
  40:       tensors1.vec(), tensors2.vec(), tensors3.vec(), std::move(vec_res)};
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 42-57
```cpp
  42:   AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES_AND2(
  43:       at::ScalarType::Half,
  44:       at::ScalarType::BFloat16,
  45:       tensors1[0].scalar_type(),
  46:       "foreach_tensor_lerp_ternary_cuda",
  47:       [&]() {
  48:         using opmath_t = typename at::opmath_type<scalar_t>;
  49:         multi_tensor_apply<4>(
  50:             tensor_lists,
  51:             TernaryOpListFunctor<
  52:                 scalar_t,
  53:                 /* depth */ 4,
  54:                 /* r_args_depth */ 3,
  55:                 /* res_arg_index */ 3>(),
  56:             LerpFunctor<opmath_t>());
  57:       });
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 59-60
```cpp
  59:   return std::move(tensor_lists[3]);
  60: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 62-83
```cpp
  62: void foreach_tensor_lerp_ternary_cuda_(
  63:     TensorList tensors1,
  64:     TensorList tensors2,
  65:     TensorList tensors3) {
  66:   check_foreach_api_restrictions(tensors1, tensors2, tensors3);
  67:   if (!can_use_fast_route({tensors1, tensors2, tensors3}, {}, true)) {
  68:     return foreach_tensor_ternary_lerp_slow_(tensors1, tensors2, tensors3);
  69:   }
  70: 
  71:   std::vector<std::vector<at::Tensor>> tensor_lists{
  72:       tensors1.vec(), tensors2.vec(), tensors3.vec()};
  73:   AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES_AND2(
  74:       at::ScalarType::Half,
  75:       at::ScalarType::BFloat16,
  76:       tensors1[0].scalar_type(),
  77:       "foreach_tensor_lerp_ternary_cuda_",
  78:       [&]() {
  79:         using opmath_t = typename at::opmath_type<scalar_t>;
  80:         multi_tensor_apply<3>(
  81:             tensor_lists,
  82:             TernaryOpListFunctor<
  83:                 scalar_t,
```
- EN: This block defines or continues the implementation of `foreach_tensor_lerp_ternary_cuda_`.
- CN: 该代码块定义或继续实现 `foreach_tensor_lerp_ternary_cuda_`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 84-86
```cpp
  84:                 /* depth */ 3,
  85:                 /* r_args_depth */ 3,
  86:                 /* res_arg_index */ 0>(),
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 87-90
```cpp
  87:             LerpFunctor<opmath_t>());
  88:       });
  89:   increment_version(tensors1);
  90: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 92-113
```cpp
  92: std::vector<at::Tensor> foreach_tensor_lerp_list_cuda(
  93:     TensorList tensors1,
  94:     TensorList tensors2,
  95:     const Scalar& weight) {
  96:   check_foreach_api_restrictions(tensors1, tensors2);
  97:   if (!can_use_fast_route({tensors1, tensors2}, {}, true)) {
  98:     return foreach_tensor_lerp_list_kernel_slow(tensors1, tensors2, weight);
  99:   }
 100: 
 101:   std::vector<at::Tensor> vec_res;
 102:   vec_res.reserve(tensors1.size());
 103:   for (const auto& t : tensors1) {
 104:     vec_res.emplace_back(at::native::empty_like(t));
 105:   }
 106:   std::vector<std::vector<at::Tensor>> tensor_lists{
 107:       tensors1.vec(), tensors2.vec(), std::move(vec_res)};
 108: 
 109:   AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES_AND2(
 110:       at::ScalarType::Half,
 111:       at::ScalarType::BFloat16,
 112:       tensors1[0].scalar_type(),
 113:       "foreach_tensor_lerp_scalar_cuda",
```
- EN: This block defines or continues the implementation of `foreach_tensor_lerp_list_cuda`.
- CN: 该代码块定义或继续实现 `foreach_tensor_lerp_list_cuda`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 114-125
```cpp
 114:       [&]() {
 115:         using opmath_t = typename at::opmath_type<scalar_t>;
 116:         multi_tensor_apply<3>(
 117:             tensor_lists,
 118:             TernaryOpScalarFunctor<
 119:                 scalar_t,
 120:                 /* depth */ 3,
 121:                 /* r_args_depth */ 2,
 122:                 /* res_arg_index */ 2>(),
 123:             LerpFunctor<opmath_t>(),
 124:             weight.to<opmath_t>());
 125:       });
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 127-128
```cpp
 127:   return std::move(tensor_lists[2]);
 128: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 130-151
```cpp
 130: void foreach_tensor_lerp_list_cuda_(
 131:     TensorList tensors1,
 132:     TensorList tensors2,
 133:     const Scalar& weight) {
 134:   check_foreach_api_restrictions(tensors1, tensors2);
 135:   if (!can_use_fast_route({tensors1, tensors2}, {}, true)) {
 136:     return foreach_tensor_lerp_list_kernel_slow_(tensors1, tensors2, weight);
 137:   }
 138: 
 139:   std::vector<std::vector<at::Tensor>> tensor_lists{
 140:       tensors1.vec(), tensors2.vec()};
 141:   AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES_AND2(
 142:       at::ScalarType::Half,
 143:       at::ScalarType::BFloat16,
 144:       tensors1[0].scalar_type(),
 145:       "foreach_tensor_lerp_scalar_cuda_",
 146:       [&]() {
 147:         using opmath_t = typename at::opmath_type<scalar_t>;
 148:         multi_tensor_apply<2>(
 149:             tensor_lists,
 150:             TernaryOpScalarFunctor<
 151:                 scalar_t,
```
- EN: This block defines or continues the implementation of `foreach_tensor_lerp_list_cuda_`.
- CN: 该代码块定义或继续实现 `foreach_tensor_lerp_list_cuda_`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 152-154
```cpp
 152:                 /* depth */ 2,
 153:                 /* r_args_depth */ 2,
 154:                 /* res_arg_index */ 0>(),
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 155-158
```cpp
 155:             LerpFunctor<opmath_t>(),
 156:             weight.to<opmath_t>());
 157:       });
 158: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 160-181
```cpp
 160: std::vector<at::Tensor> foreach_tensor_lerp_scalarlist_cuda(
 161:     TensorList tensors1,
 162:     TensorList tensors2,
 163:     at::ArrayRef<Scalar> scalars) {
 164:   check_foreach_api_restrictions(tensors1, tensors2, scalars);
 165:   if (!can_use_fast_route({tensors1, tensors2}, scalars, true)) {
 166:     return foreach_tensor_lerp_scalarlist_kernel_slow(
 167:         tensors1, tensors2, scalars);
 168:   }
 169: 
 170:   std::vector<at::Tensor> vec_res;
 171:   vec_res.reserve(tensors1.size());
 172:   for (const auto& t : tensors1) {
 173:     vec_res.emplace_back(at::native::empty_like(t));
 174:   }
 175:   std::vector<std::vector<at::Tensor>> tensor_lists{
 176:       tensors1.vec(), tensors2.vec(), std::move(vec_res)};
 177: 
 178:   AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES_AND2(
 179:       at::ScalarType::Half,
 180:       at::ScalarType::BFloat16,
 181:       tensors1[0].scalar_type(),
```
- EN: This block defines or continues the implementation of `foreach_tensor_lerp_scalarlist_cuda`.
- CN: 该代码块定义或继续实现 `foreach_tensor_lerp_scalarlist_cuda`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 182-194
```cpp
 182:       "foreach_tensor_lerp_scalarlist_cuda",
 183:       [&]() {
 184:         using opmath_t = typename at::opmath_type<scalar_t>;
 185:         multi_tensor_apply<3, opmath_t>(
 186:             tensor_lists,
 187:             scalars,
 188:             TernaryOpScalarListFunctor<
 189:                 scalar_t,
 190:                 /* depth */ 3,
 191:                 /* r_args_depth */ 2,
 192:                 /* res_arg_index */ 2>(),
 193:             LerpFunctor<opmath_t>());
 194:       });
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 196-197
```cpp
 196:   return std::move(tensor_lists[2]);
 197: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 199-220
```cpp
 199: void foreach_tensor_lerp_scalarlist_cuda_(
 200:     TensorList tensors1,
 201:     TensorList tensors2,
 202:     at::ArrayRef<Scalar> scalars) {
 203:   check_foreach_api_restrictions(tensors1, tensors2, scalars);
 204:   if (!can_use_fast_route({tensors1, tensors2}, scalars, true)) {
 205:     return foreach_tensor_lerp_scalarlist_kernel_slow_(
 206:         tensors1, tensors2, scalars);
 207:   }
 208: 
 209:   std::vector<std::vector<at::Tensor>> tensor_lists{
 210:       tensors1.vec(), tensors2.vec()};
 211: 
 212:   AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES_AND2(
 213:       at::ScalarType::Half,
 214:       at::ScalarType::BFloat16,
 215:       tensors1[0].scalar_type(),
 216:       "foreach_tensor_lerp_scalarlist_cuda_",
 217:       [&]() {
 218:         using opmath_t = typename at::opmath_type<scalar_t>;
 219:         multi_tensor_apply<2, opmath_t>(
 220:             tensor_lists,
```
- EN: This block defines or continues the implementation of `foreach_tensor_lerp_scalarlist_cuda_`.
- CN: 该代码块定义或继续实现 `foreach_tensor_lerp_scalarlist_cuda_`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 221-230
```cpp
 221:             scalars,
 222:             TernaryOpScalarListFunctor<
 223:                 scalar_t,
 224:                 /* depth */ 2,
 225:                 /* r_args_depth */ 2,
 226:                 /* res_arg_index */ 0>(),
 227:             LerpFunctor<opmath_t>());
 228:       });
 229: }
 230: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/Dispatch.h>`
  - `<ATen/native/ForeachUtils.h>`
  - `<ATen/native/Lerp.h>`
  - `<ATen/native/cuda/ForeachFunctors.cuh>`
  - `<ATen/native/cuda/MultiTensorApply.cuh>`
  - `<ATen/NativeFunctions.h>`
  - `<ATen/ops/_foreach_lerp_native.h>`
  - `<ATen/ops/empty_like_native.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES_AND2`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
