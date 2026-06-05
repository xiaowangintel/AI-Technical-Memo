# DistributionTemplates.h — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/DistributionTemplates.h`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Distribution Templates. As a header, it exposes declarations and shared helpers.
- **Purpose (CN)**: 实现或声明与 distribution、templates 相关的 ATen 原生逻辑。 作为头文件，它暴露声明与共享辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #pragma once
0002: 
0003: #include <ATen/core/Tensor.h>
0004: #include <ATen/Dispatch.h>
0005: #include <ATen/Dispatch_v2.h>
0006: #include <ATen/Generator.h>
0007: #include <ATen/ExpandUtils.h>
0008: #include <ATen/Tensor.h>
0009: #include <ATen/MemoryOverlap.h>
0010: #include <ATen/NamedTensorUtils.h>
0011: #include <ATen/native/Resize.h>
0012: #include <ATen/native/TensorIterator.h>
0013: #include <cmath>
0014: #include <limits>
0015: #include <optional>
0016: 
0017: #ifndef AT_PER_OPERATOR_HEADERS
0018: #include <ATen/Functions.h>
0019: #else
0020: #include <ATen/ops/empty_like.h>
0021: #include <ATen/ops/empty.h>
0022: #include <ATen/ops/full.h>
0023: #include <ATen/ops/view_as_real.h>
0024: #endif
0025: 
0026: namespace at::native::templates {
0027: 
0028: // ==================================================== Random ========================================================
0029: 
0030: // The purpose of `update_from` and `update_to` is to find the closest valid int64_t number that can be used as actual `from`.
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, comments/documentation.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、注释或说明。

### Lines 31-60 / 第 31-60 行
```cpp
0031: // The current implementation of `random_` uses uint64_t arithmetic and casts the result to the target dtype(scalar_t).
0032: // This casting can result in generating numbers that happen to be greater or equal to `to` value. For instance:
0033: //
0034: //    auto actual = torch::empty({3, 3}, torch::half);
0035: //    actual.random_(0, 65504);
0036: //
0037: // If random's uint64_t arithmetic produces 65503 as a random value after casting to torch::half it becomes 65504
0038: // and violates the requirement that random value must be less than `to`. To resolve this issue `update_from` and `update_to`
0039: // moves `from` to the right and `to` to the left to the next closest value that won't go outside [from, to) after casting to
0040: // the target dtype. For `to` = 65504 it moves left for (1 << (log2(to) - 11 + 1)) = 32 and becomes 65472, which is previous
0041: // available number for torch::half dtype.
0042: template<typename scalar_t>
0043: int64_t update_from(int64_t from) {
0044:   static_assert(
0045:     std::is_floating_point_v<scalar_t> ||
0046:     std::is_same_v<scalar_t, at::Half> ||
0047:     std::is_same_v<scalar_t, at::BFloat16>, "scalar_t must be floating-point type");
0048:   const auto from_plus_1 = static_cast<int64_t>(static_cast<scalar_t>(from + 1));
0049:   if (from_plus_1 < from) {
0050:     int64_t from_ = std::abs(from + 1);
0051:     int n = 0;
0052:     while (from_ >>= 1) ++n;
0053:     // NOLINTNEXTLINE(clang-analyzer-core.UndefinedBinaryOperatorResult)
0054:     from = from_plus_1 + (1LL << (n - std::numeric_limits<scalar_t>::digits + 1));
0055:   }
0056:   return from;
0057: }
0058: 
0059: template<typename scalar_t>
0060: int64_t update_to(int64_t to) {
```
- **EN**: Lines 31-60 mainly cover comments/documentation, expressions/calls, state/variable declarations. Notable symbols: dtype, empty, random_, log2.
- **CN**: 第 31-60 行主要涉及注释或说明、表达式或调用、变量/别名声明。 值得关注的符号包括：dtype, empty, random_, log2。

### Lines 61-90 / 第 61-90 行
```cpp
0061:   static_assert(
0062:     std::is_floating_point_v<scalar_t> ||
0063:     std::is_same_v<scalar_t, at::Half> ||
0064:     std::is_same_v<scalar_t, at::BFloat16>, "scalar_t must be floating-point type");
0065:   const auto to_minus_1 = static_cast<int64_t>(static_cast<scalar_t>(to - 1));
0066:   if (to_minus_1 >= to) {
0067:     int64_t to_ = std::abs(to - 1);
0068:     int n = 0;
0069:     while (to_ >>= 1) ++n;
0070:     // NOLINTNEXTLINE(clang-analyzer-core.UndefinedBinaryOperatorResult)
0071:     to = to_minus_1 - (1LL << (n - std::numeric_limits<scalar_t>::digits + 1));
0072:   }
0073:   return to;
0074: }
0075: 
0076: // Return earlier for not invoking kernel.
0077: // See https://github.com/pytorch/pytorch/issues/103418 for more details
0078: #define CHECK_EMPTY_AND_RETURN(tensor) \
0079:   if (tensor.numel() == 0) {  \
0080:     return tensor;  \
0081:   }
0082: 
0083: template<template<typename> class random_kernel, typename RNG>
0084: at::Tensor& random_impl(at::Tensor& self, std::optional<Generator> generator) {
0085:   CHECK_EMPTY_AND_RETURN(self);
0086:   auto iter = at::TensorIterator::borrowing_nullary_op(self);
0087:   random_kernel<RNG>()(iter, generator);
0088:   return self;
0089: }
0090: 
```
- **EN**: Lines 61-90 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: static_assert, abs, NOLINTNEXTLINE, CHECK_EMPTY_AND_RETURN.
- **CN**: 第 61-90 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：static_assert, abs, NOLINTNEXTLINE, CHECK_EMPTY_AND_RETURN。

### Lines 91-120 / 第 91-120 行
```cpp
0091: #define CHECK_OUT_OF_BOUNDS(var, name, min, max, dtype) \
0092:   TORCH_CHECK(var >= min && var <= max, name , " is out of bounds for ", dtype); \
0093: 
0094: #define WARN_OUT_OF_BOUNDS(var, name, digits, dtype) \
0095:   if (var < -(1LL << digits) || var > (1LL << digits)) { \
0096:     TORCH_WARN(name , " is out of bounds [-(2^", digits, "), 2^", digits, "]. ", \
0097:       "Due to precision limitations ", dtype, " can support discrete uniform distribution only within this range. ", \
0098:       "This warning will become an error in version 1.7 release, please fix the code in advance"); \
0099:   }
0100: 
0101: inline void check_from_to_in_range(int64_t from, int64_t to_inc, caffe2::TypeMeta dtype) {
0102:   const auto scalar_type = typeMetaToScalarType(dtype);
0103:   if (isFloatingType(scalar_type)) {
0104:     AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, scalar_type, "check_random_fp_bounds", [&] {
0105:       const auto min = static_cast<double>(std::numeric_limits<scalar_t>::lowest());
0106:       const auto max = static_cast<double>(std::numeric_limits<scalar_t>::max());
0107:       CHECK_OUT_OF_BOUNDS(from, "from", min, max, dtype);
0108:       CHECK_OUT_OF_BOUNDS(to_inc, "to - 1", min, max, dtype);
0109: 
0110:       constexpr auto digits = std::numeric_limits<scalar_t>::digits;
0111:       WARN_OUT_OF_BOUNDS(from, "from", digits, dtype);
0112:       WARN_OUT_OF_BOUNDS(to_inc, "to - 1", digits, dtype);
0113:     });
0114:   } else if (scalar_type == kUInt64) {
0115:     // When you do a comparison between int64_t and uint64_t, the usual
0116:     // arithmetic conversions say that the int64_t value is promoted to
0117:     // unsigned. But this conversion wraps around: if I had -1 as my int64_t,
0118:     // then it will promote to 0xFFFFFFFFFFFFFFFF in uint64_t. This is never
0119:     // the right thing to do.
0120:     CHECK_OUT_OF_BOUNDS(from, "from", 0, INT64_MAX, dtype);
```
- **EN**: Lines 91-120 mainly cover macro-based glue, state/variable declarations, comments/documentation. Notable symbols: CHECK_OUT_OF_BOUNDS, TORCH_CHECK, WARN_OUT_OF_BOUNDS, TORCH_WARN.
- **CN**: 第 91-120 行主要涉及宏定义或宏调用、变量/别名声明、注释或说明。 值得关注的符号包括：CHECK_OUT_OF_BOUNDS, TORCH_CHECK, WARN_OUT_OF_BOUNDS, TORCH_WARN。

### Lines 121-150 / 第 121-150 行
```cpp
0121:     CHECK_OUT_OF_BOUNDS(to_inc, "to - 1", 0, INT64_MAX, dtype);
0122:   } else if (isIntegralType(scalar_type, /*includeBool=*/true)) {
0123:     AT_DISPATCH_V2(scalar_type, "check_random_integral_bounds", AT_WRAP([&]() {
0124:       const auto min = static_cast<int64_t>(std::numeric_limits<scalar_t>::lowest());
0125:       const auto max = static_cast<int64_t>(std::numeric_limits<scalar_t>::max());
0126:       CHECK_OUT_OF_BOUNDS(from, "from", min, max, dtype);
0127:       CHECK_OUT_OF_BOUNDS(to_inc, "to - 1", min, max, dtype);
0128:     }), AT_EXPAND(AT_INTEGRAL_TYPES), kUInt16, kUInt32, kBool);
0129:   } else {
0130:     TORCH_CHECK(false, "check_random_bounds handles only integral, floating-point and boolean types");
0131:   }
0132: }
0133: 
0134: template<template<typename> class random_from_to_kernel, typename RNG>
0135: at::Tensor& random_from_to_impl(at::Tensor& self, int64_t from, std::optional<int64_t> to_opt, std::optional<Generator> generator) {
0136:   uint64_t range = 0;
0137:   auto iter = at::TensorIterator::borrowing_nullary_op(self);
0138:   if (to_opt.has_value()) {
0139:     // [from, to)
0140:     int64_t to = *to_opt;
0141:     TORCH_CHECK(from < to, "random_ expects 'from' to be less than 'to', but got from=", from, " >= to=", to);
0142:     if (isFloatingType(iter.dtype())) {
0143:       AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, self.scalar_type(), "random_update_from_to", [&] {
0144:         from = update_from<scalar_t>(from);
0145:         to = update_to<scalar_t>(to);
0146:         TORCH_CHECK(from < to, "random_ expects 'from' casted to dtype to be less than 'to' casted to dtype, but got from=", from, " >= to=", to);
0147:       });
0148:     }
0149:     check_from_to_in_range(from, to - 1, self.dtype());
0150:     CHECK_EMPTY_AND_RETURN(self);
```
- **EN**: Lines 121-150 mainly cover macro-based glue, state/variable declarations, function signatures/definitions. Notable symbols: CHECK_OUT_OF_BOUNDS, isIntegralType, AT_DISPATCH_V2, AT_WRAP.
- **CN**: 第 121-150 行主要涉及宏定义或宏调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：CHECK_OUT_OF_BOUNDS, isIntegralType, AT_DISPATCH_V2, AT_WRAP。

### Lines 151-180 / 第 151-180 行
```cpp
0151:     range = static_cast<uint64_t>(to) - static_cast<uint64_t>(from);
0152:     random_from_to_kernel<RNG>()(iter, range, from, generator);
0153:   } else if (from != std::numeric_limits<int64_t>::lowest()) {
0154:     // [from, std::numeric_limits<int64_t>::max()]
0155:     int64_t to_inc = 0;
0156:     if (isFloatingType(iter.dtype())) {
0157:       AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, self.scalar_type(), "random_from_to_range_calc", [&] {
0158:         constexpr int64_t scalar_t_max = static_cast<int64_t>(1) << std::numeric_limits<scalar_t>::digits;
0159:         to_inc = scalar_t_max > std::numeric_limits<int64_t>::max() ? std::numeric_limits<int64_t>::max() : static_cast<int64_t>(scalar_t_max);
0160:         from = update_from<scalar_t>(from);
0161:         TORCH_CHECK(from < to_inc, "random_ expects 'from' casted to dtype to be less than or equal to 'to_inc' casted to dtype, but got from=", from, " > to_inc=", to_inc);
0162:       });
0163:     } else if (isIntegralType(iter.dtype(), /*includeBool=*/true)) {
0164:       AT_DISPATCH_V2(self.scalar_type(), "random_from_to_range_calc", AT_WRAP([&] {
0165:         if constexpr (std::is_same_v<scalar_t, bool>) {
0166:           to_inc = static_cast<int64_t>(true);
0167:         } else {
0168:           to_inc = static_cast<int64_t>(std::numeric_limits<scalar_t>::max());
0169:         }
0170:       }), AT_EXPAND(AT_INTEGRAL_TYPES_V2), kBool);
0171:     } else {
0172:       TORCH_CHECK(false, "random_from_to_impl handles only integral, floating-point and boolean types");
0173:     }
0174:     check_from_to_in_range(from, to_inc, self.dtype());
0175:     CHECK_EMPTY_AND_RETURN(self);
0176:     range = static_cast<uint64_t>(to_inc) - static_cast<uint64_t>(from) + 1;
0177:     random_from_to_kernel<RNG>()(iter, range, from, generator);
0178:   } else {
0179:     // [std::numeric_limits<int64_t>::lowest(), std::numeric_limits<int64_t>::max()]
0180:     // range = 2^64
```
- **EN**: Lines 151-180 mainly cover state/variable declarations, macro-based glue, function signatures/definitions. Notable symbols: lowest, max, isFloatingType, dtype.
- **CN**: 第 151-180 行主要涉及变量/别名声明、宏定义或宏调用、函数签名或实现。 值得关注的符号包括：lowest, max, isFloatingType, dtype。

### Lines 181-210 / 第 181-210 行
```cpp
0181:     CHECK_EMPTY_AND_RETURN(self);
0182:     random_from_to_kernel<RNG>()(iter, generator);
0183:   }
0184:   return self;
0185: }
0186: 
0187: // ==================================================== Normal ========================================================
0188: 
0189: #define CHECK_NORMAL_TENSOR_STD(std) \
0190:   do { \
0191:     TORCH_CHECK( \
0192:       !std.is_complex(), \
0193:       "normal expects standard deviation to be non-complex"); \
0194:     TORCH_CHECK( \
0195:       std.numel() == 0 || std.is_meta() || std.min().ge(0).item<bool>(), \
0196:       "normal expects all elements of std >= 0.0"); \
0197:   } while (0)
0198: 
0199: #define CHECK_NORMAL_STD(std) \
0200:   TORCH_CHECK(std >= 0.0, "normal expects std >= 0.0, but found std ", std);
0201: 
0202: template<template<typename> class normal_kernel, typename RNG>
0203: Tensor& normal_impl_(Tensor& self, double mean, double std, std::optional<Generator> gen) {
0204:   CHECK_NORMAL_STD(std);
0205:   CHECK_EMPTY_AND_RETURN(self);
0206: 
0207:   if (self.is_complex()) {
0208:     auto float_tensor = at::view_as_real(self);
0209:     // variance for normal distribution of the real and imaginary values
0210:     // is half of the input variance
```
- **EN**: Lines 181-210 mainly cover macro-based glue, expressions/calls, function signatures/definitions. Notable symbols: CHECK_EMPTY_AND_RETURN, CHECK_NORMAL_TENSOR_STD, TORCH_CHECK, is_complex.
- **CN**: 第 181-210 行主要涉及宏定义或宏调用、表达式或调用、函数签名或实现。 值得关注的符号包括：CHECK_EMPTY_AND_RETURN, CHECK_NORMAL_TENSOR_STD, TORCH_CHECK, is_complex。

### Lines 211-240 / 第 211-240 行
```cpp
0211:     normal_kernel<RNG>()(float_tensor, mean, std/(std::sqrt(2)), gen);
0212:   } else {
0213:     normal_kernel<RNG>()(self, mean, std, gen);
0214:   }
0215:   return self;
0216: }
0217: 
0218: template<template<typename> class normal_kernel, typename RNG>
0219: Tensor& normal_out_impl(Tensor& output, const Tensor& mean, double std, std::optional<Generator> gen) {
0220:   CHECK_NORMAL_STD(std);
0221:   auto std_tensor = at::empty_like(output, MemoryFormat::Contiguous);
0222:   auto shape = at::infer_size(mean.sizes(), std_tensor.sizes());
0223:   at::native::resize_output(output, shape);
0224:   normal_impl_<normal_kernel, RNG>(output, 0, std, gen);
0225:   output.add_(mean);
0226:   return output;
0227: }
0228: 
0229: template<template<typename> class normal_kernel, typename RNG>
0230: Tensor& normal_out_impl(Tensor& output, double mean, const Tensor& std, std::optional<Generator> gen) {
0231:   CHECK_NORMAL_TENSOR_STD(std);
0232:   auto mean_tensor = at::full({}, mean, output.options());
0233:   auto shape = at::infer_size(mean_tensor.sizes(), std.sizes());
0234:   at::native::resize_output(output, shape);
0235:   normal_impl_<normal_kernel, RNG>(output, 0, 1, gen);
0236:   // CUDA NB: addcmul_out copies the tensor to be added into the output.
0237:   // The previous function here was addcmul_out(output, mean_tensor, output, std, 1);
0238:   // The third argument is not a constant reference and hence the samples in output are overwritten.
0239:   // Consequently, the computation performed is mean_tensor + mean_tensor * std instead of mean_tensor + output * std
0240:   output.mul_(std).add_(mean_tensor);
```
- **EN**: Lines 211-240 mainly cover state/variable declarations, comments/documentation, function signatures/definitions. Notable symbols: sqrt, normal_out_impl, CHECK_NORMAL_STD, empty_like.
- **CN**: 第 211-240 行主要涉及变量/别名声明、注释或说明、函数签名或实现。 值得关注的符号包括：sqrt, normal_out_impl, CHECK_NORMAL_STD, empty_like。

### Lines 241-270 / 第 241-270 行
```cpp
0241:   return output;
0242: }
0243: 
0244: template<template<typename> class normal_kernel, typename RNG>
0245: Tensor& normal_out_impl(Tensor& output, const Tensor& mean, const Tensor& std, std::optional<Generator> gen) {
0246:   CHECK_NORMAL_TENSOR_STD(std);
0247:   auto shape = at::infer_size(mean.sizes(), std.sizes());
0248:   at::native::resize_output(output, shape);
0249:   normal_impl_<normal_kernel, RNG>(output, 0, 1, gen);
0250:   // CUDA NB: addcmul_out copies the tensor to be added into the output.
0251:   // The previous function here was addcmul_out(output, mean, output, std, 1);
0252:   // The third argument is not a constant reference and hence the samples in output are overwritten.
0253:   // Consequently, the computation performed is mean + mean * std instead of mean + output * std
0254:   output.mul_(std).add_(mean);
0255:   return output;
0256: }
0257: 
0258: template<template<typename> class normal_kernel, typename RNG>
0259: Tensor normal_impl(const Tensor& mean, double std, std::optional<Generator> gen) {
0260:   CHECK_NORMAL_STD(std);
0261:   Tensor ret = at::empty_like(mean, MemoryFormat::Contiguous);
0262:   normal_out_impl<normal_kernel, RNG>(ret, mean, std, gen);
0263:   return ret;
0264: }
0265: 
0266: template<template<typename> class normal_kernel, typename RNG>
0267: Tensor normal_impl(double mean, const Tensor& std, std::optional<Generator> gen) {
0268:   CHECK_NORMAL_TENSOR_STD(std);
0269:   Tensor ret = at::empty_like(std, MemoryFormat::Contiguous);
0270:   normal_out_impl<normal_kernel, RNG>(ret, mean, std, gen);
```
- **EN**: Lines 241-270 mainly cover state/variable declarations, comments/documentation, return paths. Notable symbols: normal_out_impl, CHECK_NORMAL_TENSOR_STD, infer_size, sizes.
- **CN**: 第 241-270 行主要涉及变量/别名声明、注释或说明、返回路径。 值得关注的符号包括：normal_out_impl, CHECK_NORMAL_TENSOR_STD, infer_size, sizes。

### Lines 271-300 / 第 271-300 行
```cpp
0271:   return ret;
0272: }
0273: 
0274: template<template<typename> class normal_kernel, typename RNG>
0275: Tensor normal_impl(const Tensor& mean, const Tensor& std, std::optional<Generator> gen) {
0276:   CHECK_NORMAL_TENSOR_STD(std);
0277:   auto shape = at::infer_size(mean.sizes(), std.sizes());
0278:   Tensor ret = at::empty(shape, mean.options(), MemoryFormat::Contiguous);
0279:   normal_out_impl<normal_kernel, RNG>(ret, mean, std, gen);
0280:   return ret;
0281: }
0282: 
0283: // ==================================================== Uniform =======================================================
0284: 
0285: template<template<typename> class uniform_kernel, typename RNG>
0286: at::Tensor& uniform_impl_(at::Tensor& self, double from, double to, std::optional<Generator> generator) {
0287:   if (self.is_complex()) {
0288:     CHECK_EMPTY_AND_RETURN(self);
0289:     auto float_tensor = at::view_as_real(self);
0290:     uniform_impl_<uniform_kernel, RNG>(float_tensor, from, to, generator);
0291:   } else {
0292:     AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, self.scalar_type(), "check_uniform_bounds", [&] {
0293:       [[maybe_unused]] const auto dtype = self.dtype();
0294:       const auto min = static_cast<double>(std::numeric_limits<scalar_t>::lowest());
0295:       const auto max = static_cast<double>(std::numeric_limits<scalar_t>::max());
0296:       CHECK_OUT_OF_BOUNDS(from, "from", min, max, dtype);
0297:       CHECK_OUT_OF_BOUNDS(to, "to", min, max, dtype);
0298:       TORCH_CHECK(from <= to, "uniform_ expects to return a [from, to) range, but found from=", from, " > to=", to);
0299:       TORCH_CHECK((to - from) <= std::numeric_limits<scalar_t>::max(),
0300:             "uniform_ expects to-from <= std::numeric_limits<", toString(self.scalar_type()),
```
- **EN**: Lines 271-300 mainly cover state/variable declarations, macro-based glue, function signatures/definitions. Notable symbols: normal_impl, CHECK_NORMAL_TENSOR_STD, infer_size, sizes.
- **CN**: 第 271-300 行主要涉及变量/别名声明、宏定义或宏调用、函数签名或实现。 值得关注的符号包括：normal_impl, CHECK_NORMAL_TENSOR_STD, infer_size, sizes。

### Lines 301-330 / 第 301-330 行
```cpp
0301:             ">::max(), but found to=", to, " and from=", from,
0302:             " which result in to-from to exceed the limit");
0303:       from = std::min(std::max(from, min), max);
0304:       to = std::max(std::min(to, max), min);
0305:     });
0306:     CHECK_EMPTY_AND_RETURN(self);
0307:     auto iter = at::TensorIterator::borrowing_nullary_op(self);
0308:     uniform_kernel<RNG>()(iter, from, to, generator);
0309:   }
0310:   return self;
0311: }
0312: 
0313: // ================================================== LogNormal =======================================================
0314: 
0315: template<template<typename> class log_normal_kernel, typename RNG>
0316: at::Tensor& log_normal_impl_(at::Tensor& self, double mean, double std, std::optional<Generator> gen) {
0317:   TORCH_CHECK(std > 0.0, "log_normal_ expects std > 0.0, but found std=", std);
0318:   CHECK_EMPTY_AND_RETURN(self);
0319:   auto iter = TensorIterator::borrowing_nullary_op(self);
0320:   log_normal_kernel<RNG>()(iter, mean, std, gen);
0321:   return self;
0322: }
0323: 
0324: // =================================================== Geometric ======================================================
0325: 
0326: template<template<typename> class geometric_kernel, typename RNG>
0327: Tensor& geometric_impl_(Tensor& self, double p, std::optional<Generator> gen) {
0328:   TORCH_CHECK(0 < p && p < 1, "geometric_ expects p to be in (0, 1), but got p=", p);
0329:   CHECK_EMPTY_AND_RETURN(self);
0330:   auto iter = TensorIterator::borrowing_nullary_op(self);
```
- **EN**: Lines 301-330 mainly cover state/variable declarations, macro-based glue, function signatures/definitions. Notable symbols: max, min, CHECK_EMPTY_AND_RETURN, borrowing_nullary_op.
- **CN**: 第 301-330 行主要涉及变量/别名声明、宏定义或宏调用、函数签名或实现。 值得关注的符号包括：max, min, CHECK_EMPTY_AND_RETURN, borrowing_nullary_op。

### Lines 331-360 / 第 331-360 行
```cpp
0331:   geometric_kernel<RNG>()(iter, p, gen);
0332:   return self;
0333: }
0334: 
0335: // ================================================== Exponential =====================================================
0336: 
0337: template<template<typename> class exponential_kernel, typename RNG>
0338: Tensor& exponential_impl_(Tensor& self, double lambda, std::optional<Generator> gen) {
0339:   TORCH_CHECK(lambda > 0.0, "exponential_ expects lambda > 0.0, but found lambda=", lambda);
0340:   CHECK_EMPTY_AND_RETURN(self);
0341:   auto iter = TensorIterator::borrowing_nullary_op(self);
0342:   exponential_kernel<RNG>()(iter, lambda, gen);
0343:   return self;
0344: }
0345: 
0346: // ==================================================== Cauchy ========================================================
0347: 
0348: template<template<typename> class cauchy_kernel, typename RNG>
0349: Tensor& cauchy_impl_(Tensor& self, double median, double sigma, std::optional<Generator> gen) {
0350:   // TODO: instead of variable name 'sigma', use 'gamma' or 'scale'
0351:   // the variance, squared sigma, is undefined for cauchy distribution
0352:   TORCH_CHECK(sigma > 0.0, "cauchy_ expects sigma > 0.0, but found sigma=", sigma);
0353:   TORCH_CHECK(at::isFloatingType(self.scalar_type()), "Cauchy distribution is a continuous probability distribution. dtype must be a floating point but you specified ", self.dtype());
0354:   CHECK_EMPTY_AND_RETURN(self);
0355:   auto iter = TensorIterator::borrowing_nullary_op(self);
0356:   cauchy_kernel<RNG>()(iter, median, sigma, gen);
0357:   return self;
0358: }
0359: 
0360: // ==================================================== Bernoulli =====================================================
```
- **EN**: Lines 331-360 mainly cover state/variable declarations, comments/documentation, macro-based glue. Notable symbols: exponential_impl_, TORCH_CHECK, CHECK_EMPTY_AND_RETURN, borrowing_nullary_op.
- **CN**: 第 331-360 行主要涉及变量/别名声明、注释或说明、宏定义或宏调用。 值得关注的符号包括：exponential_impl_, TORCH_CHECK, CHECK_EMPTY_AND_RETURN, borrowing_nullary_op。

### Lines 361-390 / 第 361-390 行
```cpp
0361: 
0362: template<template<typename> class bernoulli_tensor_kernel, typename RNG>
0363: Tensor& bernoulli_impl_(Tensor& self, const Tensor& p_, std::optional<Generator> gen) {
0364:   CHECK_EMPTY_AND_RETURN(self);
0365:   NoNamesGuard guard;
0366:   at::assert_no_internal_overlap(self);
0367:   bernoulli_tensor_kernel<RNG>()(self, p_, gen);
0368:   return self;
0369: }
0370: 
0371: template<template<typename> class bernoulli_scalar_kernel, typename RNG>
0372: Tensor& bernoulli_impl_(Tensor& self, double p, std::optional<Generator> gen) {
0373:   TORCH_CHECK(0 <= p && p <= 1, "bernoulli_ expects p to be in [0, 1], but got p=", p);
0374:   CHECK_EMPTY_AND_RETURN(self);
0375:   at::assert_no_internal_overlap(self);
0376:   bernoulli_scalar_kernel<RNG>()(self, p, gen);
0377:   return self;
0378: }
0379: 
0380: template<template<typename> class bernoulli_tensor_kernel, typename RNG>
0381: Tensor& bernoulli_out_impl(Tensor& result, const Tensor& self, std::optional<Generator> gen) {
0382:   // result.resize_as_(self) requires self to have same dtype as result, so we
0383:   // use resize_ instead.
0384:   // TODO: Fix resize_as_. See pytorch/pytorch#11665.
0385:   result.resize_(self.sizes());
0386:   bernoulli_impl_<bernoulli_tensor_kernel, RNG>(result, self, gen);
0387:   namedinference::propagate_names(result, self);
0388:   return result;
0389: }
0390: 
```
- **EN**: Lines 361-390 mainly cover state/variable declarations, template setup, function signatures/definitions. Notable symbols: bernoulli_impl_, CHECK_EMPTY_AND_RETURN, assert_no_internal_overlap, TORCH_CHECK.
- **CN**: 第 361-390 行主要涉及变量/别名声明、模板声明、函数签名或实现。 值得关注的符号包括：bernoulli_impl_, CHECK_EMPTY_AND_RETURN, assert_no_internal_overlap, TORCH_CHECK。

### Lines 391-394 / 第 391-394 行
```cpp
0391: #undef CHECK_OUT_OF_BOUNDS
0392: #undef WARN_OUT_OF_BOUNDS
0393: 
0394: } // namespace at::native::templates
```
- **EN**: Lines 391-394 mainly cover macro-based glue, namespace structuring.
- **CN**: 第 391-394 行主要涉及宏定义或宏调用、命名空间组织。

## Key Concepts / 关键概念
- **EN**: TensorIterator-driven traversal  
  **CN**: 基于 TensorIterator 的遍历
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/Dispatch_v2.h>`, `<ATen/Generator.h>`, `<ATen/ExpandUtils.h>`, `<ATen/Tensor.h>`, `<ATen/MemoryOverlap.h>`, `<ATen/NamedTensorUtils.h>`, `<ATen/native/Resize.h>`, `<ATen/native/TensorIterator.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `TORCH_WARN`, `AT_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `std::`
