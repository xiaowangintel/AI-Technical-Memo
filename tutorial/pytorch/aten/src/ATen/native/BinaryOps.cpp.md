# BinaryOps.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/BinaryOps.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Binary Ops. It also wires backend dispatch paths.
- **Purpose (CN)**: 实现或声明与 二元运算、算子 相关的 ATen 原生逻辑。 它还负责连接不同后端的调度路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/native/BinaryOps.h>
0003: 
0004: #include <type_traits>
0005: #include <utility>
0006: 
0007: #include <ATen/core/Tensor.h>
0008: #include <ATen/ScalarOps.h>
0009: #include <ATen/TensorIterator.h>
0010: #include <ATen/TensorOperators.h>
0011: #include <ATen/TensorMeta.h>
0012: 
0013: #ifndef AT_PER_OPERATOR_HEADERS
0014: #include <ATen/Functions.h>
0015: #include <ATen/NativeFunctions.h>
0016: #else
0017: #include <ATen/ops/_add_relu_native.h>
0018: #include <ATen/ops/_efficientzerotensor.h>
0019: #include <ATen/ops/_test_serialization_subcmul_native.h>
0020: #include <ATen/ops/_to_copy.h>
0021: #include <ATen/ops/add.h>
0022: #include <ATen/ops/add_native.h>
0023: #include <ATen/ops/add_ops.h>
0024: #include <ATen/ops/and_native.h>
0025: #include <ATen/ops/arctan2_native.h>
0026: #include <ATen/ops/atan2.h>
0027: #include <ATen/ops/atan2_native.h>
0028: #include <ATen/ops/bitwise_and.h>
0029: #include <ATen/ops/bitwise_and_native.h>
0030: #include <ATen/ops/bitwise_left_shift.h>
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, macro-based glue.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、宏定义或宏调用。

### Lines 31-60 / 第 31-60 行
```cpp
0031: #include <ATen/ops/bitwise_left_shift_native.h>
0032: #include <ATen/ops/bitwise_or.h>
0033: #include <ATen/ops/bitwise_or_native.h>
0034: #include <ATen/ops/bitwise_right_shift.h>
0035: #include <ATen/ops/bitwise_right_shift_native.h>
0036: #include <ATen/ops/bitwise_xor.h>
0037: #include <ATen/ops/bitwise_xor_native.h>
0038: #include <ATen/ops/copysign.h>
0039: #include <ATen/ops/copysign_native.h>
0040: #include <ATen/ops/div.h>
0041: #include <ATen/ops/div_native.h>
0042: #include <ATen/ops/div_ops.h>
0043: #include <ATen/ops/divide_native.h>
0044: #include <ATen/ops/empty.h>
0045: #include <ATen/ops/eq_native.h>
0046: #include <ATen/ops/floor_divide.h>
0047: #include <ATen/ops/floor_divide_native.h>
0048: #include <ATen/ops/fmax_native.h>
0049: #include <ATen/ops/fmin_native.h>
0050: #include <ATen/ops/fmod.h>
0051: #include <ATen/ops/fmod_native.h>
0052: #include <ATen/ops/full.h>
0053: #include <ATen/ops/gcd_native.h>
0054: #include <ATen/ops/ge.h>
0055: #include <ATen/ops/ge_native.h>
0056: #include <ATen/ops/greater_equal_native.h>
0057: #include <ATen/ops/greater_native.h>
0058: #include <ATen/ops/gt.h>
0059: #include <ATen/ops/gt_native.h>
0060: #include <ATen/ops/heaviside_native.h>
```
- **EN**: Lines 31-60 mainly cover header inclusion.
- **CN**: 第 31-60 行主要涉及头文件包含。

### Lines 61-90 / 第 61-90 行
```cpp
0061: #include <ATen/ops/hypot_native.h>
0062: #include <ATen/ops/igamma.h>
0063: #include <ATen/ops/igamma_native.h>
0064: #include <ATen/ops/igammac.h>
0065: #include <ATen/ops/igammac_native.h>
0066: #include <ATen/ops/lcm_native.h>
0067: #include <ATen/ops/ldexp.h>
0068: #include <ATen/ops/ldexp_native.h>
0069: #include <ATen/ops/le.h>
0070: #include <ATen/ops/le_native.h>
0071: #include <ATen/ops/less_equal_native.h>
0072: #include <ATen/ops/less_native.h>
0073: #include <ATen/ops/linalg_cross_native.h>
0074: #include <ATen/ops/linalg_cross_ops.h>
0075: #include <ATen/ops/logaddexp2_native.h>
0076: #include <ATen/ops/logaddexp_native.h>
0077: #include <ATen/ops/logical_and.h>
0078: #include <ATen/ops/logical_and_native.h>
0079: #include <ATen/ops/logical_or.h>
0080: #include <ATen/ops/logical_or_native.h>
0081: #include <ATen/ops/logical_xor.h>
0082: #include <ATen/ops/logical_xor_native.h>
0083: #include <ATen/ops/logit_backward_native.h>
0084: #include <ATen/ops/lshift_native.h>
0085: #include <ATen/ops/lt.h>
0086: #include <ATen/ops/lt_native.h>
0087: #include <ATen/ops/max_native.h>
0088: #include <ATen/ops/maximum.h>
0089: #include <ATen/ops/maximum_native.h>
0090: #include <ATen/ops/min_native.h>
```
- **EN**: Lines 61-90 mainly cover header inclusion.
- **CN**: 第 61-90 行主要涉及头文件包含。

### Lines 91-120 / 第 91-120 行
```cpp
0091: #include <ATen/ops/minimum.h>
0092: #include <ATen/ops/minimum_native.h>
0093: #include <ATen/ops/mul.h>
0094: #include <ATen/ops/mul_native.h>
0095: #include <ATen/ops/mul_ops.h>
0096: #include <ATen/ops/multiply_native.h>
0097: #include <ATen/ops/ne.h>
0098: #include <ATen/ops/ne_native.h>
0099: #include <ATen/ops/nextafter_native.h>
0100: #include <ATen/ops/not_equal_native.h>
0101: #include <ATen/ops/or_native.h>
0102: #include <ATen/ops/pow.h>
0103: #include <ATen/ops/remainder.h>
0104: #include <ATen/ops/remainder_native.h>
0105: #include <ATen/ops/rshift_native.h>
0106: #include <ATen/ops/rsub_native.h>
0107: #include <ATen/ops/sigmoid_backward_native.h>
0108: #include <ATen/ops/special_chebyshev_polynomial_t.h>
0109: #include <ATen/ops/special_chebyshev_polynomial_t_native.h>
0110: #include <ATen/ops/special_chebyshev_polynomial_u.h>
0111: #include <ATen/ops/special_chebyshev_polynomial_u_native.h>
0112: #include <ATen/ops/special_chebyshev_polynomial_v.h>
0113: #include <ATen/ops/special_chebyshev_polynomial_v_native.h>
0114: #include <ATen/ops/special_chebyshev_polynomial_w.h>
0115: #include <ATen/ops/special_chebyshev_polynomial_w_native.h>
0116: #include <ATen/ops/special_gammainc_native.h>
0117: #include <ATen/ops/special_gammaincc_native.h>
0118: #include <ATen/ops/special_hermite_polynomial_h.h>
0119: #include <ATen/ops/special_hermite_polynomial_h_native.h>
0120: #include <ATen/ops/special_hermite_polynomial_he.h>
```
- **EN**: Lines 91-120 mainly cover header inclusion.
- **CN**: 第 91-120 行主要涉及头文件包含。

### Lines 121-150 / 第 121-150 行
```cpp
0121: #include <ATen/ops/special_hermite_polynomial_he_native.h>
0122: #include <ATen/ops/special_laguerre_polynomial_l.h>
0123: #include <ATen/ops/special_laguerre_polynomial_l_native.h>
0124: #include <ATen/ops/special_legendre_polynomial_p.h>
0125: #include <ATen/ops/special_legendre_polynomial_p_native.h>
0126: #include <ATen/ops/special_shifted_chebyshev_polynomial_t.h>
0127: #include <ATen/ops/special_shifted_chebyshev_polynomial_t_native.h>
0128: #include <ATen/ops/special_shifted_chebyshev_polynomial_u.h>
0129: #include <ATen/ops/special_shifted_chebyshev_polynomial_u_native.h>
0130: #include <ATen/ops/special_shifted_chebyshev_polynomial_v.h>
0131: #include <ATen/ops/special_shifted_chebyshev_polynomial_v_native.h>
0132: #include <ATen/ops/special_shifted_chebyshev_polynomial_w.h>
0133: #include <ATen/ops/special_shifted_chebyshev_polynomial_w_native.h>
0134: #include <ATen/ops/special_xlog1py.h>
0135: #include <ATen/ops/special_xlog1py_native.h>
0136: #include <ATen/ops/special_xlogy_native.h>
0137: #include <ATen/ops/special_zeta.h>
0138: #include <ATen/ops/special_zeta_native.h>
0139: #include <ATen/ops/sub.h>
0140: #include <ATen/ops/sub_native.h>
0141: #include <ATen/ops/subtract_native.h>
0142: #include <ATen/ops/tanh_backward_native.h>
0143: #include <ATen/ops/true_divide_native.h>
0144: #include <ATen/ops/xlogy.h>
0145: #include <ATen/ops/xlogy_native.h>
0146: #include <ATen/ops/xor_native.h>
0147: #endif
0148: 
0149: namespace at::meta {
0150: 
```
- **EN**: Lines 121-150 mainly cover header inclusion, conditional compilation, namespace structuring.
- **CN**: 第 121-150 行主要涉及头文件包含、预处理条件、命名空间组织。

### Lines 151-180 / 第 151-180 行
```cpp
0151: TORCH_META_FUNC2(add, Tensor) (
0152:   const Tensor& self, const Tensor& other, const Scalar& alpha
0153: ) {
0154:   build_borrowing_binary_op(maybe_get_output(), self, other);
0155:   native::alpha_check(dtype(), alpha);
0156: }
0157: 
0158: TORCH_META_FUNC2(sub, Tensor) (
0159:   const Tensor& self, const Tensor& other, const Scalar& alpha
0160: ) {
0161:   native::sub_check(self, other);
0162:   build_borrowing_binary_op(maybe_get_output(), self, other);
0163:   native::alpha_check(dtype(), alpha);
0164: }
0165: 
0166: TORCH_META_FUNC2(mul, Tensor) (
0167:   const Tensor& self, const Tensor& other
0168: ) {
0169:   build_borrowing_binary_op(maybe_get_output(), self, other);
0170: }
0171: 
0172: TORCH_META_FUNC2(div, Tensor) (const Tensor& self, const Tensor& other) {
0173:   build_borrowing_binary_float_op(maybe_get_output(), self, other);
0174: }
0175: 
0176: TORCH_META_FUNC2(div, Tensor_mode) (const Tensor& self, const Tensor& other, std::optional<std::string_view> rounding_mode) {
0177:   if (!rounding_mode.has_value()) {
0178:     build_borrowing_binary_float_op(maybe_get_output(), self, other);
0179:   // NOLINTNEXTLINE(bugprone-branch-clone)
0180:   } else if (*rounding_mode == "trunc") {
```
- **EN**: Lines 151-180 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: TORCH_META_FUNC2, build_borrowing_binary_op, maybe_get_output, alpha_check.
- **CN**: 第 151-180 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：TORCH_META_FUNC2, build_borrowing_binary_op, maybe_get_output, alpha_check。

### Lines 181-210 / 第 181-210 行
```cpp
0181:     build_borrowing_binary_op(maybe_get_output(), self, other);
0182:   } else if (*rounding_mode == "floor") {
0183:     build_borrowing_binary_op(maybe_get_output(), self, other);
0184:   } else {
0185:     TORCH_CHECK(false,
0186:         "div expected rounding_mode to be one of None, 'trunc', or 'floor' "
0187:         "but found '", *rounding_mode, "'");
0188:   }
0189: }
0190: 
0191: TORCH_META_FUNC(special_xlog1py) (const Tensor& self, const Tensor& other) {
0192:   build_borrowing_binary_float_op(maybe_get_output(), self, other);
0193: }
0194: 
0195: TORCH_META_FUNC(special_zeta) (const Tensor& self, const Tensor& other) {
0196:   build_borrowing_binary_float_op(maybe_get_output(), self, other);
0197: }
0198: 
0199: TORCH_META_FUNC(special_chebyshev_polynomial_t) (const Tensor& self, const Tensor& n) {
0200:   build_borrowing_binary_float_op(maybe_get_output(), self, n);
0201: }
0202: 
0203: TORCH_META_FUNC(special_chebyshev_polynomial_u) (const Tensor& self, const Tensor& n) {
0204:   build_borrowing_binary_float_op(maybe_get_output(), self, n);
0205: }
0206: 
0207: TORCH_META_FUNC(special_chebyshev_polynomial_v) (const Tensor& self, const Tensor& n) {
0208:   build_borrowing_binary_float_op(maybe_get_output(), self, n);
0209: }
0210: 
```
- **EN**: Lines 181-210 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: build_borrowing_binary_op, maybe_get_output, TORCH_CHECK, TORCH_META_FUNC.
- **CN**: 第 181-210 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：build_borrowing_binary_op, maybe_get_output, TORCH_CHECK, TORCH_META_FUNC。

### Lines 211-240 / 第 211-240 行
```cpp
0211: TORCH_META_FUNC(special_chebyshev_polynomial_w) (const Tensor& self, const Tensor& n) {
0212:   build_borrowing_binary_float_op(maybe_get_output(), self, n);
0213: }
0214: 
0215: TORCH_META_FUNC(special_hermite_polynomial_h) (const Tensor& self, const Tensor& n) {
0216:   build_borrowing_binary_float_op(maybe_get_output(), self, n);
0217: }
0218: 
0219: TORCH_META_FUNC(special_hermite_polynomial_he) (const Tensor& self, const Tensor& n) {
0220:   build_borrowing_binary_float_op(maybe_get_output(), self, n);
0221: }
0222: 
0223: TORCH_META_FUNC(special_laguerre_polynomial_l) (const Tensor& self, const Tensor& n) {
0224:   build_borrowing_binary_float_op(maybe_get_output(), self, n);
0225: }
0226: 
0227: TORCH_META_FUNC(special_legendre_polynomial_p) (const Tensor& self, const Tensor& n) {
0228:   build_borrowing_binary_float_op(maybe_get_output(), self, n);
0229: }
0230: 
0231: TORCH_META_FUNC(special_shifted_chebyshev_polynomial_t) (const Tensor& self, const Tensor& n) {
0232:   build_borrowing_binary_float_op(maybe_get_output(), self, n);
0233: }
0234: 
0235: TORCH_META_FUNC(special_shifted_chebyshev_polynomial_u) (const Tensor& self, const Tensor& n) {
0236:   build_borrowing_binary_float_op(maybe_get_output(), self, n);
0237: }
0238: 
0239: TORCH_META_FUNC(special_shifted_chebyshev_polynomial_v) (const Tensor& self, const Tensor& n) {
0240:   build_borrowing_binary_float_op(maybe_get_output(), self, n);
```
- **EN**: Lines 211-240 mainly cover macro-based glue, state/variable declarations, expressions/calls. Notable symbols: TORCH_META_FUNC, build_borrowing_binary_float_op, maybe_get_output.
- **CN**: 第 211-240 行主要涉及宏定义或宏调用、变量/别名声明、表达式或调用。 值得关注的符号包括：TORCH_META_FUNC, build_borrowing_binary_float_op, maybe_get_output。

### Lines 241-270 / 第 241-270 行
```cpp
0241: }
0242: 
0243: TORCH_META_FUNC(special_shifted_chebyshev_polynomial_w) (const Tensor& self, const Tensor& n) {
0244:   build_borrowing_binary_float_op(maybe_get_output(), self, n);
0245: }
0246: 
0247: TORCH_META_FUNC2(copysign, Tensor) (
0248:   const Tensor& self, const Tensor& other
0249: ) {
0250:   build_borrowing_binary_float_op(maybe_get_output(), self, other);
0251: }
0252: 
0253: TORCH_META_FUNC(heaviside) (
0254:   const Tensor& self, const Tensor& other
0255: ) {
0256:   TORCH_CHECK(!self.is_complex() && !other.is_complex() &&
0257:               (maybe_get_output().defined() ? !maybe_get_output().is_complex() : true),
0258:               "heaviside is not yet implemented for complex tensors.");
0259:   TORCH_CHECK(self.dtype() == other.dtype() &&
0260:               (maybe_get_output().defined() ? maybe_get_output().dtype() == self.dtype() : true),
0261:               "heaviside is not yet implemented for tensors with different dtypes.");
0262: 
0263:   build_binary_op(maybe_get_output(), self, other);
0264: }
0265: 
0266: TORCH_META_FUNC(atan2) (const Tensor& self, const Tensor& other) {
0267:   build_borrowing_binary_float_op(maybe_get_output(), self, other);
0268: }
0269: 
0270: TORCH_META_FUNC2(remainder, Tensor)(const Tensor& self, const Tensor& other) {
```
- **EN**: Lines 241-270 mainly cover expressions/calls, macro-based glue, state/variable declarations. Notable symbols: TORCH_META_FUNC, build_borrowing_binary_float_op, maybe_get_output, TORCH_META_FUNC2.
- **CN**: 第 241-270 行主要涉及表达式或调用、宏定义或宏调用、变量/别名声明。 值得关注的符号包括：TORCH_META_FUNC, build_borrowing_binary_float_op, maybe_get_output, TORCH_META_FUNC2。

### Lines 271-300 / 第 271-300 行
```cpp
0271:   build_borrowing_binary_op(maybe_get_output(), self, other);
0272: }
0273: 
0274: TORCH_META_FUNC2(bitwise_left_shift, Tensor) (
0275:   const Tensor& self, const Tensor& other
0276: ) {
0277:   build_borrowing_binary_op(maybe_get_output(), self, other);
0278: }
0279: 
0280: TORCH_META_FUNC2(bitwise_right_shift, Tensor) (
0281:   const Tensor& self, const Tensor& other
0282: ) {
0283:   build_borrowing_binary_op(maybe_get_output(), self, other);
0284: }
0285: 
0286: TORCH_META_FUNC2(bitwise_and, Tensor) (const Tensor& self, const Tensor& other) {
0287:   build_borrowing_binary_op(maybe_get_output(), self, other);
0288: }
0289: 
0290: TORCH_META_FUNC2(bitwise_or, Tensor) (const Tensor& self, const Tensor& other) {
0291:   build_borrowing_binary_op(maybe_get_output(), self, other);
0292: }
0293: 
0294: TORCH_META_FUNC2(bitwise_xor, Tensor) (const Tensor& self, const Tensor& other) {
0295:   build_borrowing_binary_op(maybe_get_output(), self, other);
0296: }
0297: 
0298: TORCH_META_FUNC2(fmod, Tensor) (const Tensor& self, const Tensor& other) {
0299:   build_borrowing_binary_op(maybe_get_output(), self, other);
0300: }
```
- **EN**: Lines 271-300 mainly cover expressions/calls, state/variable declarations, macro-based glue. Notable symbols: build_borrowing_binary_op, maybe_get_output, TORCH_META_FUNC2.
- **CN**: 第 271-300 行主要涉及表达式或调用、变量/别名声明、宏定义或宏调用。 值得关注的符号包括：build_borrowing_binary_op, maybe_get_output, TORCH_META_FUNC2。

### Lines 301-330 / 第 301-330 行
```cpp
0301: 
0302: TORCH_META_FUNC2(xlogy, Tensor) (const Tensor& self, const Tensor& other) {
0303:   build_borrowing_binary_float_op(maybe_get_output(), self, other);
0304: }
0305: 
0306: TORCH_META_FUNC(logit_backward) (const Tensor& grad_output, const Tensor& input, std::optional<double> eps) {
0307:   build_borrowing_binary_op(maybe_get_output(), grad_output, input);
0308: }
0309: 
0310: TORCH_META_FUNC(sigmoid_backward) (const Tensor& grad_output, const Tensor& output) {
0311:   build_borrowing_binary_op(maybe_get_output(), grad_output, output);
0312: }
0313: 
0314: TORCH_META_FUNC(tanh_backward) (const Tensor& grad_output, const Tensor& output) {
0315:   build_borrowing_binary_op(maybe_get_output(), grad_output, output);
0316: }
0317: 
0318: // These are normal binary ops that preserve dtype
0319: #define CREATE_BINARY_META_FUNC(func)                                 \
0320:   TORCH_META_FUNC(func) (const Tensor& self, const Tensor& other) {   \
0321:     build_borrowing_binary_op(maybe_get_output(), self, other);                 \
0322:   }
0323: 
0324: CREATE_BINARY_META_FUNC(logaddexp)
0325: CREATE_BINARY_META_FUNC(logaddexp2)
0326: CREATE_BINARY_META_FUNC(gcd)
0327: CREATE_BINARY_META_FUNC(lcm)
0328: CREATE_BINARY_META_FUNC(hypot)
0329: CREATE_BINARY_META_FUNC(igamma)
0330: CREATE_BINARY_META_FUNC(igammac)
```
- **EN**: Lines 301-330 mainly cover macro-based glue, expressions/calls, state/variable declarations. Notable symbols: TORCH_META_FUNC2, build_borrowing_binary_float_op, maybe_get_output, TORCH_META_FUNC.
- **CN**: 第 301-330 行主要涉及宏定义或宏调用、表达式或调用、变量/别名声明。 值得关注的符号包括：TORCH_META_FUNC2, build_borrowing_binary_float_op, maybe_get_output, TORCH_META_FUNC。

### Lines 331-360 / 第 331-360 行
```cpp
0331: CREATE_BINARY_META_FUNC(nextafter)
0332: 
0333: TORCH_META_FUNC(maximum) (const Tensor& self, const Tensor& other) {
0334:   TORCH_CHECK(!self.is_complex() && !other.is_complex(), "maximum not implemented for complex tensors.");
0335:   build_borrowing_binary_op(maybe_get_output(), self, other);
0336: }
0337: 
0338: TORCH_META_FUNC(minimum) (const Tensor& self, const Tensor& other) {
0339:   TORCH_CHECK(!self.is_complex() && !other.is_complex(), "minimum not implemented for complex tensors.");
0340:   build_borrowing_binary_op(maybe_get_output(), self, other);
0341: }
0342: 
0343: TORCH_META_FUNC(fmax) (const Tensor& self, const Tensor& other) {
0344:     TORCH_CHECK(!self.is_complex() && !other.is_complex(), "fmax not implemented for complex tensors.");
0345:     build_binary_op(maybe_get_output(), self, other);
0346: }
0347: 
0348: TORCH_META_FUNC(fmin) (const Tensor& self, const Tensor& other) {
0349:     TORCH_CHECK(!self.is_complex() && !other.is_complex(), "fmin not implemented for complex tensors.");
0350:     build_binary_op(maybe_get_output(), self, other);
0351: }
0352: 
0353: #define CREATE_COMPARISON_SCALAR_TENSOR_META_FUNC(func)                     \
0354:   TORCH_META_FUNC2(func, Tensor)(const Tensor& self, const Tensor& other) { \
0355:     const Tensor& result = maybe_get_output();                              \
0356:     build_borrowing_comparison_op(result, self, other);                     \
0357:   }                                                                         \
0358:                                                                             \
0359:   TORCH_META_FUNC2(func, Scalar)(const Tensor& self, const Scalar& other) { \
0360:     auto other_tensor =                                                     \
```
- **EN**: Lines 331-360 mainly cover macro-based glue, expressions/calls, state/variable declarations. Notable symbols: CREATE_BINARY_META_FUNC, TORCH_META_FUNC, TORCH_CHECK, is_complex.
- **CN**: 第 331-360 行主要涉及宏定义或宏调用、表达式或调用、变量/别名声明。 值得关注的符号包括：CREATE_BINARY_META_FUNC, TORCH_META_FUNC, TORCH_CHECK, is_complex。

### Lines 361-390 / 第 361-390 行
```cpp
0361:         native::wrapped_scalar_tensor(other);                               \
0362:     build_borrowing_except_last_argument_comparison_op(maybe_get_output(), self, other_tensor);  \
0363:   }
0364: 
0365: CREATE_COMPARISON_SCALAR_TENSOR_META_FUNC(eq)
0366: CREATE_COMPARISON_SCALAR_TENSOR_META_FUNC(ne)
0367: CREATE_COMPARISON_SCALAR_TENSOR_META_FUNC(lt)
0368: CREATE_COMPARISON_SCALAR_TENSOR_META_FUNC(le)
0369: CREATE_COMPARISON_SCALAR_TENSOR_META_FUNC(gt)
0370: CREATE_COMPARISON_SCALAR_TENSOR_META_FUNC(ge)
0371: 
0372: } // namespace at::meta
0373: 
0374: 
0375: namespace at::native {
0376: 
0377: DEFINE_DISPATCH(add_clamp_stub);
0378: DEFINE_DISPATCH(mul_stub);
0379: DEFINE_DISPATCH(sub_stub);
0380: DEFINE_DISPATCH(div_true_stub);
0381: DEFINE_DISPATCH(div_floor_stub);
0382: DEFINE_DISPATCH(div_trunc_stub);
0383: DEFINE_DISPATCH(remainder_stub);
0384: DEFINE_DISPATCH(atan2_stub);
0385: DEFINE_DISPATCH(bitwise_and_stub);
0386: DEFINE_DISPATCH(bitwise_or_stub);
0387: DEFINE_DISPATCH(bitwise_xor_stub);
0388: DEFINE_DISPATCH(lshift_stub);
0389: DEFINE_DISPATCH(rshift_stub);
0390: DEFINE_DISPATCH(logical_and_stub);
```
- **EN**: Lines 361-390 mainly cover macro-based glue, function signatures/definitions, namespace structuring. Notable symbols: wrapped_scalar_tensor, build_borrowing_except_last_argument_comparison_op, maybe_get_output, CREATE_COMPARISON_SCALAR_TENSOR_META_FUNC.
- **CN**: 第 361-390 行主要涉及宏定义或宏调用、函数签名或实现、命名空间组织。 值得关注的符号包括：wrapped_scalar_tensor, build_borrowing_except_last_argument_comparison_op, maybe_get_output, CREATE_COMPARISON_SCALAR_TENSOR_META_FUNC。

### Lines 391-420 / 第 391-420 行
```cpp
0391: DEFINE_DISPATCH(logical_or_stub);
0392: DEFINE_DISPATCH(logical_xor_stub);
0393: DEFINE_DISPATCH(lt_stub);
0394: DEFINE_DISPATCH(le_stub);
0395: DEFINE_DISPATCH(gt_stub);
0396: DEFINE_DISPATCH(ge_stub);
0397: DEFINE_DISPATCH(eq_stub);
0398: DEFINE_DISPATCH(ne_stub);
0399: DEFINE_DISPATCH(sigmoid_backward_stub);
0400: DEFINE_DISPATCH(logit_backward_stub);
0401: DEFINE_DISPATCH(tanh_backward_stub);
0402: DEFINE_DISPATCH(maximum_stub);
0403: DEFINE_DISPATCH(minimum_stub);
0404: DEFINE_DISPATCH(fmax_stub);
0405: DEFINE_DISPATCH(fmin_stub);
0406: DEFINE_DISPATCH(fmod_stub);
0407: DEFINE_DISPATCH(logaddexp_stub);
0408: DEFINE_DISPATCH(logaddexp2_stub);
0409: DEFINE_DISPATCH(gcd_stub);
0410: DEFINE_DISPATCH(lcm_stub);
0411: DEFINE_DISPATCH(hypot_stub);
0412: DEFINE_DISPATCH(igamma_stub);
0413: DEFINE_DISPATCH(igammac_stub);
0414: DEFINE_DISPATCH(nextafter_stub);
0415: DEFINE_DISPATCH(heaviside_stub);
0416: DEFINE_DISPATCH(copysign_stub);
0417: DEFINE_DISPATCH(xlogy_stub);
0418: DEFINE_DISPATCH(xlog1py_stub);
0419: DEFINE_DISPATCH(zeta_stub);
0420: DEFINE_DISPATCH(chebyshev_polynomial_t_stub);
```
- **EN**: Lines 391-420 mainly cover macro-based glue. Notable symbols: DEFINE_DISPATCH.
- **CN**: 第 391-420 行主要涉及宏定义或宏调用。 值得关注的符号包括：DEFINE_DISPATCH。

### Lines 421-450 / 第 421-450 行
```cpp
0421: DEFINE_DISPATCH(chebyshev_polynomial_u_stub);
0422: DEFINE_DISPATCH(chebyshev_polynomial_v_stub);
0423: DEFINE_DISPATCH(chebyshev_polynomial_w_stub);
0424: DEFINE_DISPATCH(hermite_polynomial_h_stub);
0425: DEFINE_DISPATCH(hermite_polynomial_he_stub);
0426: DEFINE_DISPATCH(laguerre_polynomial_l_stub);
0427: DEFINE_DISPATCH(legendre_polynomial_p_stub);
0428: DEFINE_DISPATCH(shifted_chebyshev_polynomial_t_stub);
0429: DEFINE_DISPATCH(shifted_chebyshev_polynomial_u_stub);
0430: DEFINE_DISPATCH(shifted_chebyshev_polynomial_v_stub);
0431: DEFINE_DISPATCH(shifted_chebyshev_polynomial_w_stub);
0432: DEFINE_DISPATCH(ldexp_stub);
0433: 
0434: TORCH_IMPL_FUNC(sub_out) (
0435:   const Tensor& self, const Tensor& other, const Scalar& alpha, const Tensor& result
0436: ) {
0437:   add_stub(device_type(), *this, -alpha);
0438:   TORCH_INTERNAL_ASSERT(result.scalar_type() == output().dtype());
0439: }
0440: 
0441: TORCH_IMPL_FUNC(mul_out) (
0442:   const Tensor& self, const Tensor& other, const Tensor& result
0443: ) {
0444:   mul_stub(device_type(), *this);
0445: }
0446: 
0447: TORCH_IMPL_FUNC(div_out) (const Tensor& self, const Tensor& other, const Tensor& result) {
0448:   div_true_stub(device_type(), *this);
0449: }
0450: 
```
- **EN**: Lines 421-450 mainly cover macro-based glue, expressions/calls, state/variable declarations. Notable symbols: DEFINE_DISPATCH, TORCH_IMPL_FUNC, add_stub, device_type.
- **CN**: 第 421-450 行主要涉及宏定义或宏调用、表达式或调用、变量/别名声明。 值得关注的符号包括：DEFINE_DISPATCH, TORCH_IMPL_FUNC, add_stub, device_type。

### Lines 451-480 / 第 451-480 行
```cpp
0451: TORCH_IMPL_FUNC(div_out_mode) (
0452:   const Tensor& self, const Tensor& other, std::optional<std::string_view> rounding_mode, const Tensor& result
0453: ) {
0454:   if (!rounding_mode.has_value()) {
0455:     div_true_stub(device_type(), *this);
0456:   } else if (*rounding_mode == "trunc") {
0457:     div_trunc_stub(device_type(), *this);
0458:   } else if (*rounding_mode == "floor") {
0459:     div_floor_stub(device_type(), *this);
0460:   }
0461: }
0462: 
0463: TORCH_IMPL_FUNC(logit_backward_out) (const Tensor& grad_output, const Tensor& input, std::optional<double> eps, const Tensor& result) {
0464:   logit_backward_stub(device_type(), *this, Scalar(eps ? eps.value() : -1.0));
0465: }
0466: 
0467: TORCH_IMPL_FUNC(sigmoid_backward_out) (const Tensor& grad_output, const Tensor& output, const Tensor& result) {
0468:   sigmoid_backward_stub(device_type(), *this);
0469: }
0470: 
0471: TORCH_IMPL_FUNC(special_xlog1py_out) (const Tensor& self, const Tensor& other, const Tensor& result) {
0472:   xlog1py_stub(device_type(), *this);
0473: }
0474: 
0475: TORCH_IMPL_FUNC(special_zeta_out) (const Tensor& self, const Tensor& other, const Tensor& result) {
0476:   zeta_stub(device_type(), *this);
0477: }
0478: 
0479: TORCH_IMPL_FUNC(special_chebyshev_polynomial_t_out) (const Tensor& self, const Tensor& n, const Tensor& result) {
0480:   chebyshev_polynomial_t_stub(device_type(), *this);
```
- **EN**: Lines 451-480 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: TORCH_IMPL_FUNC, has_value, div_true_stub, device_type.
- **CN**: 第 451-480 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：TORCH_IMPL_FUNC, has_value, div_true_stub, device_type。

### Lines 481-510 / 第 481-510 行
```cpp
0481: }
0482: 
0483: TORCH_IMPL_FUNC(special_chebyshev_polynomial_u_out) (const Tensor& self, const Tensor& n, const Tensor& result) {
0484:   chebyshev_polynomial_u_stub(device_type(), *this);
0485: }
0486: 
0487: TORCH_IMPL_FUNC(special_chebyshev_polynomial_v_out) (const Tensor& self, const Tensor& n, const Tensor& result) {
0488:   chebyshev_polynomial_v_stub(device_type(), *this);
0489: }
0490: 
0491: TORCH_IMPL_FUNC(special_chebyshev_polynomial_w_out) (const Tensor& self, const Tensor& n, const Tensor& result) {
0492:   chebyshev_polynomial_w_stub(device_type(), *this);
0493: }
0494: 
0495: TORCH_IMPL_FUNC(special_hermite_polynomial_h_out) (const Tensor& self, const Tensor& n, const Tensor& result) {
0496:   hermite_polynomial_h_stub(device_type(), *this);
0497: }
0498: 
0499: TORCH_IMPL_FUNC(special_hermite_polynomial_he_out) (const Tensor& self, const Tensor& n, const Tensor& result) {
0500:   hermite_polynomial_he_stub(device_type(), *this);
0501: }
0502: 
0503: TORCH_IMPL_FUNC(special_laguerre_polynomial_l_out) (const Tensor& self, const Tensor& n, const Tensor& result) {
0504:   laguerre_polynomial_l_stub(device_type(), *this);
0505: }
0506: 
0507: TORCH_IMPL_FUNC(special_legendre_polynomial_p_out) (const Tensor& self, const Tensor& n, const Tensor& result) {
0508:   legendre_polynomial_p_stub(device_type(), *this);
0509: }
0510: 
```
- **EN**: Lines 481-510 mainly cover expressions/calls, macro-based glue, state/variable declarations. Notable symbols: TORCH_IMPL_FUNC, chebyshev_polynomial_u_stub, device_type, chebyshev_polynomial_v_stub.
- **CN**: 第 481-510 行主要涉及表达式或调用、宏定义或宏调用、变量/别名声明。 值得关注的符号包括：TORCH_IMPL_FUNC, chebyshev_polynomial_u_stub, device_type, chebyshev_polynomial_v_stub。

### Lines 511-540 / 第 511-540 行
```cpp
0511: TORCH_IMPL_FUNC(special_shifted_chebyshev_polynomial_t_out) (const Tensor& self, const Tensor& n, const Tensor& result) {
0512:   shifted_chebyshev_polynomial_t_stub(device_type(), *this);
0513: }
0514: 
0515: TORCH_IMPL_FUNC(special_shifted_chebyshev_polynomial_u_out) (const Tensor& self, const Tensor& n, const Tensor& result) {
0516:   shifted_chebyshev_polynomial_u_stub(device_type(), *this);
0517: }
0518: 
0519: TORCH_IMPL_FUNC(special_shifted_chebyshev_polynomial_v_out) (const Tensor& self, const Tensor& n, const Tensor& result) {
0520:   shifted_chebyshev_polynomial_v_stub(device_type(), *this);
0521: }
0522: 
0523: TORCH_IMPL_FUNC(special_shifted_chebyshev_polynomial_w_out) (const Tensor& self, const Tensor& n, const Tensor& result) {
0524:   shifted_chebyshev_polynomial_w_stub(device_type(), *this);
0525: }
0526: 
0527: TORCH_IMPL_FUNC(tanh_backward_out) (const Tensor& grad_output, const Tensor& output, const Tensor& result) {
0528:   tanh_backward_stub(device_type(), *this);
0529: }
0530: 
0531: #define CREATE_BINARY_TORCH_IMPL_FUNC(func_out, func_stub)                                                    \
0532: TORCH_IMPL_FUNC(func_out) (const Tensor& self, const Tensor& other, const Tensor& result) {  \
0533:   func_stub(device_type(), *this);                                                           \
0534: }
0535: 
0536: CREATE_BINARY_TORCH_IMPL_FUNC(bitwise_and_out, bitwise_and_stub)
0537: CREATE_BINARY_TORCH_IMPL_FUNC(bitwise_or_out, bitwise_or_stub)
0538: CREATE_BINARY_TORCH_IMPL_FUNC(bitwise_xor_out, bitwise_xor_stub)
0539: CREATE_BINARY_TORCH_IMPL_FUNC(maximum_out, maximum_stub)
0540: CREATE_BINARY_TORCH_IMPL_FUNC(minimum_out, minimum_stub)
```
- **EN**: Lines 511-540 mainly cover macro-based glue, expressions/calls, state/variable declarations. Notable symbols: TORCH_IMPL_FUNC, shifted_chebyshev_polynomial_t_stub, device_type, shifted_chebyshev_polynomial_u_stub.
- **CN**: 第 511-540 行主要涉及宏定义或宏调用、表达式或调用、变量/别名声明。 值得关注的符号包括：TORCH_IMPL_FUNC, shifted_chebyshev_polynomial_t_stub, device_type, shifted_chebyshev_polynomial_u_stub。

### Lines 541-570 / 第 541-570 行
```cpp
0541: CREATE_BINARY_TORCH_IMPL_FUNC(fmax_out, fmax_stub)
0542: CREATE_BINARY_TORCH_IMPL_FUNC(fmin_out, fmin_stub)
0543: CREATE_BINARY_TORCH_IMPL_FUNC(fmod_out, fmod_stub)
0544: CREATE_BINARY_TORCH_IMPL_FUNC(logaddexp_out, logaddexp_stub)
0545: CREATE_BINARY_TORCH_IMPL_FUNC(logaddexp2_out, logaddexp2_stub)
0546: CREATE_BINARY_TORCH_IMPL_FUNC(gcd_out, gcd_stub)
0547: CREATE_BINARY_TORCH_IMPL_FUNC(lcm_out, lcm_stub)
0548: CREATE_BINARY_TORCH_IMPL_FUNC(hypot_out, hypot_stub)
0549: CREATE_BINARY_TORCH_IMPL_FUNC(igamma_out, igamma_stub)
0550: CREATE_BINARY_TORCH_IMPL_FUNC(igammac_out, igammac_stub)
0551: CREATE_BINARY_TORCH_IMPL_FUNC(nextafter_out, nextafter_stub)
0552: CREATE_BINARY_TORCH_IMPL_FUNC(remainder_out, remainder_stub)
0553: CREATE_BINARY_TORCH_IMPL_FUNC(xlogy_out, xlogy_stub)
0554: 
0555: Tensor special_xlog1py(const Scalar& x, const Tensor& y) {
0556:   return at::special_xlog1py(wrapped_scalar_tensor(x), y);
0557: }
0558: 
0559: Tensor special_xlog1py(const Tensor& x, const Scalar& y) {
0560:   return at::special_xlog1py(x, wrapped_scalar_tensor(y));
0561: }
0562: 
0563: Tensor& special_xlog1py_out(const Scalar& self, const Tensor& other, Tensor& result) {
0564:   return at::special_xlog1py_out(result, wrapped_scalar_tensor(self), other);
0565: }
0566: 
0567: Tensor& special_xlog1py_out(const Tensor& self, const Scalar& other, Tensor& result) {
0568:   return at::special_xlog1py_out(result, self, wrapped_scalar_tensor(other));
0569: }
0570: 
```
- **EN**: Lines 541-570 mainly cover macro-based glue, function signatures/definitions, return paths. Notable symbols: CREATE_BINARY_TORCH_IMPL_FUNC, special_xlog1py, wrapped_scalar_tensor, special_xlog1py_out.
- **CN**: 第 541-570 行主要涉及宏定义或宏调用、函数签名或实现、返回路径。 值得关注的符号包括：CREATE_BINARY_TORCH_IMPL_FUNC, special_xlog1py, wrapped_scalar_tensor, special_xlog1py_out。

### Lines 571-600 / 第 571-600 行
```cpp
0571: Tensor special_zeta(const Scalar& x, const Tensor& y) {
0572:   return at::special_zeta(wrapped_scalar_tensor(x), y);
0573: }
0574: 
0575: Tensor special_zeta(const Tensor& x, const Scalar& y) {
0576:   return at::special_zeta(x, wrapped_scalar_tensor(y));
0577: }
0578: 
0579: Tensor& special_zeta_out(const Scalar& self, const Tensor& other, Tensor& result) {
0580:   return at::special_zeta_out(result, wrapped_scalar_tensor(self), other);
0581: }
0582: 
0583: Tensor& special_zeta_out(const Tensor& self, const Scalar& other, Tensor& result) {
0584:   return at::special_zeta_out(result, self, wrapped_scalar_tensor(other));
0585: }
0586: 
0587: Tensor special_chebyshev_polynomial_t(const Scalar& x, const Tensor& n) {
0588:   return at::special_chebyshev_polynomial_t(wrapped_scalar_tensor(x), n);
0589: }
0590: 
0591: Tensor special_chebyshev_polynomial_t(const Tensor& x, const Scalar& n) {
0592:   return at::special_chebyshev_polynomial_t(x, wrapped_scalar_tensor(n));
0593: }
0594: 
0595: Tensor& special_chebyshev_polynomial_t_out(const Scalar& self, const Tensor& n, Tensor& result) {
0596:   return at::special_chebyshev_polynomial_t_out(result, wrapped_scalar_tensor(self), n);
0597: }
0598: 
0599: Tensor& special_chebyshev_polynomial_t_out(const Tensor& self, const Scalar& n, Tensor& result) {
0600:   return at::special_chebyshev_polynomial_t_out(result, self, wrapped_scalar_tensor(n));
```
- **EN**: Lines 571-600 mainly cover function signatures/definitions, return paths, expressions/calls. Notable symbols: special_zeta, wrapped_scalar_tensor, special_zeta_out, special_chebyshev_polynomial_t.
- **CN**: 第 571-600 行主要涉及函数签名或实现、返回路径、表达式或调用。 值得关注的符号包括：special_zeta, wrapped_scalar_tensor, special_zeta_out, special_chebyshev_polynomial_t。

### Lines 601-630 / 第 601-630 行
```cpp
0601: }
0602: 
0603: Tensor special_chebyshev_polynomial_u(const Scalar& x, const Tensor& n) {
0604:   return at::special_chebyshev_polynomial_u(wrapped_scalar_tensor(x), n);
0605: }
0606: 
0607: Tensor special_chebyshev_polynomial_u(const Tensor& x, const Scalar& n) {
0608:   return at::special_chebyshev_polynomial_u(x, wrapped_scalar_tensor(n));
0609: }
0610: 
0611: Tensor& special_chebyshev_polynomial_u_out(const Scalar& self, const Tensor& n, Tensor& result) {
0612:   return at::special_chebyshev_polynomial_u_out(result, wrapped_scalar_tensor(self), n);
0613: }
0614: 
0615: Tensor& special_chebyshev_polynomial_u_out(const Tensor& self, const Scalar& n, Tensor& result) {
0616:   return at::special_chebyshev_polynomial_u_out(result, self, wrapped_scalar_tensor(n));
0617: }
0618: 
0619: Tensor special_chebyshev_polynomial_v(const Scalar& x, const Tensor& n) {
0620:   return at::special_chebyshev_polynomial_v(wrapped_scalar_tensor(x), n);
0621: }
0622: 
0623: Tensor special_chebyshev_polynomial_v(const Tensor& x, const Scalar& n) {
0624:   return at::special_chebyshev_polynomial_v(x, wrapped_scalar_tensor(n));
0625: }
0626: 
0627: Tensor& special_chebyshev_polynomial_v_out(const Scalar& self, const Tensor& n, Tensor& result) {
0628:   return at::special_chebyshev_polynomial_v_out(result, wrapped_scalar_tensor(self), n);
0629: }
0630: 
```
- **EN**: Lines 601-630 mainly cover expressions/calls, function signatures/definitions, return paths. Notable symbols: special_chebyshev_polynomial_u, wrapped_scalar_tensor, special_chebyshev_polynomial_u_out, special_chebyshev_polynomial_v.
- **CN**: 第 601-630 行主要涉及表达式或调用、函数签名或实现、返回路径。 值得关注的符号包括：special_chebyshev_polynomial_u, wrapped_scalar_tensor, special_chebyshev_polynomial_u_out, special_chebyshev_polynomial_v。

### Lines 631-660 / 第 631-660 行
```cpp
0631: Tensor& special_chebyshev_polynomial_v_out(const Tensor& self, const Scalar& n, Tensor& result) {
0632:   return at::special_chebyshev_polynomial_v_out(result, self, wrapped_scalar_tensor(n));
0633: }
0634: 
0635: Tensor special_chebyshev_polynomial_w(const Scalar& x, const Tensor& n) {
0636:   return at::special_chebyshev_polynomial_w(wrapped_scalar_tensor(x), n);
0637: }
0638: 
0639: Tensor special_chebyshev_polynomial_w(const Tensor& x, const Scalar& n) {
0640:   return at::special_chebyshev_polynomial_w(x, wrapped_scalar_tensor(n));
0641: }
0642: 
0643: Tensor& special_chebyshev_polynomial_w_out(const Scalar& self, const Tensor& n, Tensor& result) {
0644:   return at::special_chebyshev_polynomial_w_out(result, wrapped_scalar_tensor(self), n);
0645: }
0646: 
0647: Tensor& special_chebyshev_polynomial_w_out(const Tensor& self, const Scalar& n, Tensor& result) {
0648:   return at::special_chebyshev_polynomial_w_out(result, self, wrapped_scalar_tensor(n));
0649: }
0650: 
0651: Tensor special_hermite_polynomial_h(const Scalar& x, const Tensor& n) {
0652:   return at::special_hermite_polynomial_h(wrapped_scalar_tensor(x), n);
0653: }
0654: 
0655: Tensor special_hermite_polynomial_h(const Tensor& x, const Scalar& n) {
0656:   return at::special_hermite_polynomial_h(x, wrapped_scalar_tensor(n));
0657: }
0658: 
0659: Tensor& special_hermite_polynomial_h_out(const Scalar& self, const Tensor& n, Tensor& result) {
0660:   return at::special_hermite_polynomial_h_out(result, wrapped_scalar_tensor(self), n);
```
- **EN**: Lines 631-660 mainly cover function signatures/definitions, return paths, expressions/calls. Notable symbols: special_chebyshev_polynomial_v_out, wrapped_scalar_tensor, special_chebyshev_polynomial_w, special_chebyshev_polynomial_w_out.
- **CN**: 第 631-660 行主要涉及函数签名或实现、返回路径、表达式或调用。 值得关注的符号包括：special_chebyshev_polynomial_v_out, wrapped_scalar_tensor, special_chebyshev_polynomial_w, special_chebyshev_polynomial_w_out。

### Lines 661-690 / 第 661-690 行
```cpp
0661: }
0662: 
0663: Tensor& special_hermite_polynomial_h_out(const Tensor& self, const Scalar& n, Tensor& result) {
0664:   return at::special_hermite_polynomial_h_out(result, self, wrapped_scalar_tensor(n));
0665: }
0666: 
0667: Tensor special_hermite_polynomial_he(const Scalar& x, const Tensor& n) {
0668:   return at::special_hermite_polynomial_he(wrapped_scalar_tensor(x), n);
0669: }
0670: 
0671: Tensor special_hermite_polynomial_he(const Tensor& x, const Scalar& n) {
0672:   return at::special_hermite_polynomial_he(x, wrapped_scalar_tensor(n));
0673: }
0674: 
0675: Tensor& special_hermite_polynomial_he_out(const Scalar& self, const Tensor& n, Tensor& result) {
0676:   return at::special_hermite_polynomial_he_out(result, wrapped_scalar_tensor(self), n);
0677: }
0678: 
0679: Tensor& special_hermite_polynomial_he_out(const Tensor& self, const Scalar& n, Tensor& result) {
0680:   return at::special_hermite_polynomial_he_out(result, self, wrapped_scalar_tensor(n));
0681: }
0682: 
0683: Tensor special_laguerre_polynomial_l(const Scalar& x, const Tensor& n) {
0684:   return at::special_laguerre_polynomial_l(wrapped_scalar_tensor(x), n);
0685: }
0686: 
0687: Tensor special_laguerre_polynomial_l(const Tensor& x, const Scalar& n) {
0688:   return at::special_laguerre_polynomial_l(x, wrapped_scalar_tensor(n));
0689: }
0690: 
```
- **EN**: Lines 661-690 mainly cover expressions/calls, function signatures/definitions, return paths. Notable symbols: special_hermite_polynomial_h_out, wrapped_scalar_tensor, special_hermite_polynomial_he, special_hermite_polynomial_he_out.
- **CN**: 第 661-690 行主要涉及表达式或调用、函数签名或实现、返回路径。 值得关注的符号包括：special_hermite_polynomial_h_out, wrapped_scalar_tensor, special_hermite_polynomial_he, special_hermite_polynomial_he_out。

### Lines 691-720 / 第 691-720 行
```cpp
0691: Tensor& special_laguerre_polynomial_l_out(const Scalar& self, const Tensor& n, Tensor& result) {
0692:   return at::special_laguerre_polynomial_l_out(result, wrapped_scalar_tensor(self), n);
0693: }
0694: 
0695: Tensor& special_laguerre_polynomial_l_out(const Tensor& self, const Scalar& n, Tensor& result) {
0696:   return at::special_laguerre_polynomial_l_out(result, self, wrapped_scalar_tensor(n));
0697: }
0698: 
0699: Tensor special_legendre_polynomial_p(const Scalar& x, const Tensor& n) {
0700:   return at::special_legendre_polynomial_p(wrapped_scalar_tensor(x), n);
0701: }
0702: 
0703: Tensor special_legendre_polynomial_p(const Tensor& x, const Scalar& n) {
0704:   return at::special_legendre_polynomial_p(x, wrapped_scalar_tensor(n));
0705: }
0706: 
0707: Tensor& special_legendre_polynomial_p_out(const Scalar& self, const Tensor& n, Tensor& result) {
0708:   return at::special_legendre_polynomial_p_out(result, wrapped_scalar_tensor(self), n);
0709: }
0710: 
0711: Tensor& special_legendre_polynomial_p_out(const Tensor& self, const Scalar& n, Tensor& result) {
0712:   return at::special_legendre_polynomial_p_out(result, self, wrapped_scalar_tensor(n));
0713: }
0714: 
0715: Tensor special_shifted_chebyshev_polynomial_t(const Scalar& x, const Tensor& n) {
0716:   return at::special_shifted_chebyshev_polynomial_t(wrapped_scalar_tensor(x), n);
0717: }
0718: 
0719: Tensor special_shifted_chebyshev_polynomial_t(const Tensor& x, const Scalar& n) {
0720:   return at::special_shifted_chebyshev_polynomial_t(x, wrapped_scalar_tensor(n));
```
- **EN**: Lines 691-720 mainly cover function signatures/definitions, return paths, expressions/calls. Notable symbols: special_laguerre_polynomial_l_out, wrapped_scalar_tensor, special_legendre_polynomial_p, special_legendre_polynomial_p_out.
- **CN**: 第 691-720 行主要涉及函数签名或实现、返回路径、表达式或调用。 值得关注的符号包括：special_laguerre_polynomial_l_out, wrapped_scalar_tensor, special_legendre_polynomial_p, special_legendre_polynomial_p_out。

### Lines 721-750 / 第 721-750 行
```cpp
0721: }
0722: 
0723: Tensor& special_shifted_chebyshev_polynomial_t_out(const Scalar& self, const Tensor& n, Tensor& result) {
0724:   return at::special_shifted_chebyshev_polynomial_t_out(result, wrapped_scalar_tensor(self), n);
0725: }
0726: 
0727: Tensor& special_shifted_chebyshev_polynomial_t_out(const Tensor& self, const Scalar& n, Tensor& result) {
0728:   return at::special_shifted_chebyshev_polynomial_t_out(result, self, wrapped_scalar_tensor(n));
0729: }
0730: 
0731: Tensor special_shifted_chebyshev_polynomial_u(const Scalar& x, const Tensor& n) {
0732:   return at::special_shifted_chebyshev_polynomial_u(wrapped_scalar_tensor(x), n);
0733: }
0734: 
0735: Tensor special_shifted_chebyshev_polynomial_u(const Tensor& x, const Scalar& n) {
0736:   return at::special_shifted_chebyshev_polynomial_u(x, wrapped_scalar_tensor(n));
0737: }
0738: 
0739: Tensor& special_shifted_chebyshev_polynomial_u_out(const Scalar& self, const Tensor& n, Tensor& result) {
0740:   return at::special_shifted_chebyshev_polynomial_u_out(result, wrapped_scalar_tensor(self), n);
0741: }
0742: 
0743: Tensor& special_shifted_chebyshev_polynomial_u_out(const Tensor& self, const Scalar& n, Tensor& result) {
0744:   return at::special_shifted_chebyshev_polynomial_u_out(result, self, wrapped_scalar_tensor(n));
0745: }
0746: 
0747: Tensor special_shifted_chebyshev_polynomial_v(const Scalar& x, const Tensor& n) {
0748:   return at::special_shifted_chebyshev_polynomial_v(wrapped_scalar_tensor(x), n);
0749: }
0750: 
```
- **EN**: Lines 721-750 mainly cover expressions/calls, function signatures/definitions, return paths. Notable symbols: special_shifted_chebyshev_polynomial_t_out, wrapped_scalar_tensor, special_shifted_chebyshev_polynomial_u, special_shifted_chebyshev_polynomial_u_out.
- **CN**: 第 721-750 行主要涉及表达式或调用、函数签名或实现、返回路径。 值得关注的符号包括：special_shifted_chebyshev_polynomial_t_out, wrapped_scalar_tensor, special_shifted_chebyshev_polynomial_u, special_shifted_chebyshev_polynomial_u_out。

### Lines 751-780 / 第 751-780 行
```cpp
0751: Tensor special_shifted_chebyshev_polynomial_v(const Tensor& x, const Scalar& n) {
0752:   return at::special_shifted_chebyshev_polynomial_v(x, wrapped_scalar_tensor(n));
0753: }
0754: 
0755: Tensor& special_shifted_chebyshev_polynomial_v_out(const Scalar& self, const Tensor& n, Tensor& result) {
0756:   return at::special_shifted_chebyshev_polynomial_v_out(result, wrapped_scalar_tensor(self), n);
0757: }
0758: 
0759: Tensor& special_shifted_chebyshev_polynomial_v_out(const Tensor& self, const Scalar& n, Tensor& result) {
0760:   return at::special_shifted_chebyshev_polynomial_v_out(result, self, wrapped_scalar_tensor(n));
0761: }
0762: 
0763: Tensor special_shifted_chebyshev_polynomial_w(const Scalar& x, const Tensor& n) {
0764:   return at::special_shifted_chebyshev_polynomial_w(wrapped_scalar_tensor(x), n);
0765: }
0766: 
0767: Tensor special_shifted_chebyshev_polynomial_w(const Tensor& x, const Scalar& n) {
0768:   return at::special_shifted_chebyshev_polynomial_w(x, wrapped_scalar_tensor(n));
0769: }
0770: 
0771: Tensor& special_shifted_chebyshev_polynomial_w_out(const Scalar& self, const Tensor& n, Tensor& result) {
0772:   return at::special_shifted_chebyshev_polynomial_w_out(result, wrapped_scalar_tensor(self), n);
0773: }
0774: 
0775: Tensor& special_shifted_chebyshev_polynomial_w_out(const Tensor& self, const Scalar& n, Tensor& result) {
0776:   return at::special_shifted_chebyshev_polynomial_w_out(result, self, wrapped_scalar_tensor(n));
0777: }
0778: 
0779: Tensor& special_gammainc_out(const Tensor& self, const Tensor& other, Tensor& result) {
0780:   return at::igamma_out(result, self, other);
```
- **EN**: Lines 751-780 mainly cover function signatures/definitions, return paths, expressions/calls. Notable symbols: special_shifted_chebyshev_polynomial_v, wrapped_scalar_tensor, special_shifted_chebyshev_polynomial_v_out, special_shifted_chebyshev_polynomial_w.
- **CN**: 第 751-780 行主要涉及函数签名或实现、返回路径、表达式或调用。 值得关注的符号包括：special_shifted_chebyshev_polynomial_v, wrapped_scalar_tensor, special_shifted_chebyshev_polynomial_v_out, special_shifted_chebyshev_polynomial_w。

### Lines 781-810 / 第 781-810 行
```cpp
0781: }
0782: 
0783: Tensor special_gammainc(const Tensor& self, const Tensor& other) {
0784:   return at::igamma(self, other);
0785: }
0786: 
0787: Tensor& special_gammaincc_out(const Tensor& self, const Tensor& other, Tensor& result) {
0788:   return at::igammac_out(result, self, other);
0789: }
0790: 
0791: Tensor special_gammaincc(const Tensor& self, const Tensor& other) {
0792:   return at::igammac(self, other);
0793: }
0794: 
0795: TORCH_IMPL_FUNC(atan2_out) (const Tensor& self, const Tensor& other, const Tensor& result) {
0796:   atan2_stub(device_type(), *this);
0797: }
0798: 
0799: Tensor arctan2(const Tensor& self, const Tensor& other) {
0800:   return at::atan2(self, other);
0801: }
0802: 
0803: Tensor& arctan2_(Tensor& self, const Tensor& other) {
0804:   return self.atan2_(other);
0805: }
0806: 
0807: Tensor& arctan2_out(const Tensor& self, const Tensor& other, Tensor& result) {
0808:   return at::atan2_out(result, self, other);
0809: }
0810: 
```
- **EN**: Lines 781-810 mainly cover expressions/calls, function signatures/definitions, return paths. Notable symbols: special_gammainc, igamma, special_gammaincc_out, igammac_out.
- **CN**: 第 781-810 行主要涉及表达式或调用、函数签名或实现、返回路径。 值得关注的符号包括：special_gammainc, igamma, special_gammaincc_out, igammac_out。

### Lines 811-840 / 第 811-840 行
```cpp
0811: static Tensor& add_relu_impl(
0812:     Tensor& result, const Tensor& self, const Tensor& other, const Scalar& alpha) {
0813:   auto iter = TensorIterator::binary_op(result, self, other);
0814:   Scalar min_val;
0815:   Scalar max_val;
0816:   if (self.dtype() == at::kInt) {
0817:     min_val = 0;
0818:     max_val = std::numeric_limits<int32_t>::max();
0819:   } else if (self.dtype() == at::kLong) {
0820:     min_val = 0;
0821:     max_val = std::numeric_limits<int64_t>::max();
0822:   } else if (self.dtype() == at::kShort) {
0823:     min_val = 0;
0824:     max_val = std::numeric_limits<int16_t>::max();
0825:   } else if (self.dtype() == at::kChar) {
0826:     min_val = 0;
0827:     max_val = std::numeric_limits<int8_t>::max();
0828:   } else if (self.dtype() == at::kFloat) {
0829:     min_val = 0.0;
0830:     max_val = std::numeric_limits<float>::max();
0831:   } else if (self.dtype() == at::kDouble) {
0832:     min_val = 0.0;
0833:     max_val = std::numeric_limits<double>::max();
0834:   } else {
0835:     TORCH_INTERNAL_ASSERT(
0836:         false, "Unsupported datatype for add_relu:", self.dtype().name());
0837:   }
0838: 
0839:   result = iter.output();
0840:   add_clamp_stub(iter.device_type(), iter, alpha, min_val, max_val);
```
- **EN**: Lines 811-840 mainly cover state/variable declarations, function signatures/definitions, control-flow checks. Notable symbols: add_relu_impl, binary_op, dtype, max.
- **CN**: 第 811-840 行主要涉及变量/别名声明、函数签名或实现、控制流逻辑。 值得关注的符号包括：add_relu_impl, binary_op, dtype, max。

### Lines 841-870 / 第 841-870 行
```cpp
0841:   return result;
0842: }
0843: 
0844: Tensor& add_relu_out(const Tensor& self, const Tensor& other, const Scalar& alpha, Tensor& result) {
0845:   return add_relu_impl(result, self, other, alpha);
0846: }
0847: 
0848: Tensor add_relu(const Tensor& self, const Tensor& other, const Scalar& alpha) {
0849:   Tensor result;
0850:   return add_relu_impl(result, self, other, alpha);
0851: }
0852: 
0853: Tensor add_relu(const Tensor& self, const Scalar& other, const Scalar& alpha) {
0854:   return add_relu(self, wrapped_scalar_tensor(other), alpha);
0855: }
0856: 
0857: Tensor& add_relu_(Tensor& self, const Tensor& other, const Scalar& alpha) {
0858:   return add_relu_impl(self, self, other, alpha);
0859: }
0860: 
0861: Tensor& add_relu_(Tensor& self, const Scalar& other, const Scalar& alpha) {
0862:   return add_relu_(self, wrapped_scalar_tensor(other), alpha);
0863: }
0864: 
0865: TORCH_IMPL_FUNC(copysign_out) (
0866:   const Tensor& self, const Tensor& other, const Tensor& result
0867: ) {
0868:   copysign_stub(device_type(), *this);
0869: }
0870: 
```
- **EN**: Lines 841-870 mainly cover expressions/calls, return paths, function signatures/definitions. Notable symbols: add_relu_out, add_relu_impl, add_relu, wrapped_scalar_tensor.
- **CN**: 第 841-870 行主要涉及表达式或调用、返回路径、函数签名或实现。 值得关注的符号包括：add_relu_out, add_relu_impl, add_relu, wrapped_scalar_tensor。

### Lines 871-900 / 第 871-900 行
```cpp
0871: Tensor copysign(const Tensor& self, const Scalar& other) {
0872:   // redispatch!
0873:   return at::copysign(self, wrapped_scalar_tensor(other));
0874: }
0875: 
0876: Tensor& copysign_(Tensor& self, const Scalar& other) {
0877:   // redispatch!
0878:   return self.copysign_(wrapped_scalar_tensor(other));
0879: }
0880: 
0881: Tensor& copysign_out(const Tensor& self, const Scalar& other, Tensor& result) {
0882:   // redispatch!
0883:   return at::copysign_out(result, self, wrapped_scalar_tensor(other));
0884: }
0885: 
0886: // WARNING: There doesn't appear to be any testing for this function
0887: // with sparse self input.
0888: Tensor div(const Tensor& self, const Scalar& other) {
0889:   return self.div(wrapped_scalar_tensor(other)); // redispatch!
0890: }
0891: 
0892: // WARNING: This function, with a sparse self, is currently only
0893: // exercised by DistributedDataParallelTest.test_sparse_gradients
0894: // (you need to exercise it from C++, because this overload is never
0895: // used for Python)
0896: Tensor& div_(Tensor& self, const Scalar& other) {
0897:   return self.div_(wrapped_scalar_tensor(other)); // redispatch!
0898: }
0899: 
0900: Tensor div(const Tensor& self, const Scalar& other, std::optional<std::string_view> rounding_mode) {
```
- **EN**: Lines 871-900 mainly cover comments/documentation, function signatures/definitions, return paths. Notable symbols: copysign, wrapped_scalar_tensor, copysign_, copysign_out.
- **CN**: 第 871-900 行主要涉及注释或说明、函数签名或实现、返回路径。 值得关注的符号包括：copysign, wrapped_scalar_tensor, copysign_, copysign_out。

### Lines 901-930 / 第 901-930 行
```cpp
0901:   return self.div(wrapped_scalar_tensor(other), std::move(rounding_mode)); // redispatch!
0902: }
0903: 
0904: Tensor& div_(Tensor& self, const Scalar& other, std::optional<std::string_view> rounding_mode) {
0905:   return self.div_(wrapped_scalar_tensor(other), std::move(rounding_mode)); // redispatch!
0906: }
0907: 
0908: // divide, alias for div
0909: Tensor& divide_out(const Tensor& self, const Tensor& other, Tensor& result) {
0910:   return at::div_out(result, self, other);
0911: }
0912: 
0913: Tensor divide(const Tensor& self, const Tensor& other) {
0914:   return self.div(other);
0915: }
0916: 
0917: Tensor& divide_(Tensor& self, const Tensor& other) {
0918:   return self.div_(other);
0919: }
0920: 
0921: Tensor divide(const Tensor& self, const Scalar& other) {
0922:   return self.div(other);
0923: }
0924: 
0925: Tensor& divide_(Tensor& self, const Scalar& other) {
0926:   return self.div_(other);
0927: }
0928: 
0929: Tensor& divide_out(const Tensor& self, const Tensor& other, std::optional<std::string_view> rounding_mode, Tensor& result) {
0930:   return at::div_out(result, self, other, std::move(rounding_mode));
```
- **EN**: Lines 901-930 mainly cover return paths, expressions/calls, function signatures/definitions. Notable symbols: div, wrapped_scalar_tensor, move, div_.
- **CN**: 第 901-930 行主要涉及返回路径、表达式或调用、函数签名或实现。 值得关注的符号包括：div, wrapped_scalar_tensor, move, div_。

### Lines 931-960 / 第 931-960 行
```cpp
0931: }
0932: 
0933: Tensor divide(const Tensor& self, const Tensor& other, std::optional<std::string_view> rounding_mode) {
0934:   return self.div(other, std::move(rounding_mode));
0935: }
0936: 
0937: Tensor& divide_(Tensor& self, const Tensor& other, std::optional<std::string_view> rounding_mode) {
0938:   return self.div_(other, std::move(rounding_mode));
0939: }
0940: 
0941: Tensor divide(const Tensor& self, const Scalar& other, std::optional<std::string_view> rounding_mode) {
0942:   return self.div(other, std::move(rounding_mode));
0943: }
0944: 
0945: Tensor& divide_(Tensor& self, const Scalar& other, std::optional<std::string_view> rounding_mode) {
0946:   return self.div_(other, std::move(rounding_mode));
0947: }
0948: 
0949: // true_divide, an alias for div
0950: Tensor& true_divide_out(const Tensor& self, const Tensor& divisor, Tensor& result) {
0951:   return at::div_out(result, self, divisor);
0952: }
0953: 
0954: Tensor true_divide(const Tensor& self, const Tensor& divisor) {
0955:   return self.div(divisor);
0956: }
0957: 
0958: Tensor& true_divide_(Tensor& self, const Tensor& divisor) {
0959:   return self.div_(divisor);
0960: }
```
- **EN**: Lines 931-960 mainly cover expressions/calls, function signatures/definitions, return paths. Notable symbols: divide, div, move, divide_.
- **CN**: 第 931-960 行主要涉及表达式或调用、函数签名或实现、返回路径。 值得关注的符号包括：divide, div, move, divide_。

### Lines 961-990 / 第 961-990 行
```cpp
0961: 
0962: Tensor true_divide(const Tensor& self, const Scalar& divisor) {
0963:   return self.div(divisor);
0964: }
0965: 
0966: Tensor& true_divide_(Tensor& self, const Scalar& divisor) {
0967:   return self.div_(divisor);
0968: }
0969: 
0970: Tensor& floor_divide_out(const Tensor& self, const Tensor& other, Tensor& result) {
0971:   auto iter = TensorIterator::binary_op(result, self, other);
0972:   div_floor_stub(iter.device_type(), iter);
0973:   if (!result.defined()) {
0974:     result = iter.output();
0975:   }
0976:   return result;
0977: }
0978: 
0979: Tensor floor_divide(const Tensor& self, const Tensor& other) {
0980:   Tensor result;
0981:   auto iter = TensorIterator::binary_op(result, self, other);
0982:   div_floor_stub(iter.device_type(), iter);
0983:   return iter.output();
0984: }
0985: 
0986: Tensor& floor_divide_(Tensor& self, const Tensor& other) {
0987:   return native::floor_divide_out(self, other, self);
0988: }
0989: 
0990: // TODO: Make this structured to undo the perf regression from native:: removal
```
- **EN**: Lines 961-990 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: true_divide, div, true_divide_, div_.
- **CN**: 第 961-990 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：true_divide, div, true_divide_, div_。

### Lines 991-1020 / 第 991-1020 行
```cpp
0991: // in call here
0992: Tensor mul(const Tensor& self, const Scalar& other) {
0993:   return at::mul(self, wrapped_scalar_tensor(other)); // redispatch!
0994: }
0995: 
0996: Tensor& mul_(Tensor& self, const Scalar& other) {
0997:   return at::mul_out(self, wrapped_scalar_tensor(other), self); // redispatch!
0998: }
0999: 
1000: Tensor& mul__scalar_sparse_csr(Tensor& self, const Scalar& other) {
1001:   self.values().mul_(other);
1002:   return self;
1003: }
1004: 
1005: static Device correct_out_device(const Tensor& self, const Tensor& other) {
1006:   if (self.device() == at::kCPU){
1007:       return other.device();
1008:   } else {
1009:     return self.device();
1010:   }
1011: }
1012: 
1013: static Tensor send_to_meta(const Tensor& self, const Device& device) {
1014:   Tensor out_meta;
1015:   if (self._is_zerotensor() && self.unsafeGetTensorImpl()->is_wrapped_number()) {
1016:     out_meta = at::_efficientzerotensor(self.sizes(), self.options().device(device));
1017:     out_meta.unsafeGetTensorImpl()->set_wrapped_number(true);
1018:   } else {
1019:     out_meta = self.to(device);
1020:   }
```
- **EN**: Lines 991-1020 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: mul, wrapped_scalar_tensor, mul_, mul_out.
- **CN**: 第 991-1020 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：mul, wrapped_scalar_tensor, mul_, mul_out。

### Lines 1021-1050 / 第 1021-1050 行
```cpp
1021:   return out_meta;
1022: }
1023: 
1024: Tensor mul_zerotensor(const Tensor& self, const Tensor& other) {
1025:   auto out_device = correct_out_device(self, other);
1026:   // hack to use the TensorIterator to get the correct broadcasting and type promotion logic
1027:   auto device_ = Device(DeviceType::Meta);
1028:   constexpr c10::DispatchKeySet meta_dks(at::DispatchKey::Meta);
1029:   auto self_meta = send_to_meta(self, device_);
1030:   auto other_meta = send_to_meta(other, device_);
1031:   auto meta_out = at::_ops::mul_Tensor::redispatch(meta_dks, self_meta, other_meta);
1032:   return at::_efficientzerotensor(meta_out.sizes(), meta_out.options().device(out_device));
1033: }
1034: 
1035: Tensor div_zerotensor(const Tensor& self, const Tensor& other) {
1036:   auto out_device = correct_out_device(self, other);
1037:   // hack to use the TensorIterator to get the correct broadcasting and type promotion logic
1038:   auto device_ = Device(DeviceType::Meta);
1039:   constexpr c10::DispatchKeySet meta_dks(at::DispatchKey::Meta);
1040:   auto self_meta = send_to_meta(self, device_);
1041:   auto other_meta = send_to_meta(other, device_);
1042:   auto meta_out = at::_ops::div_Tensor::redispatch(meta_dks, self_meta, other_meta);
1043: 
1044:   if (self._is_zerotensor()) {
1045:     if (other._is_zerotensor()) {
1046:       // 0/0, return full NAN
1047:       return at::full(meta_out.sizes(), std::numeric_limits<float>::quiet_NaN(), meta_out.options().device(out_device));
1048:     }
1049:     else {
1050:       // 0/x, return zero tensor
```
- **EN**: Lines 1021-1050 mainly cover state/variable declarations, comments/documentation, return paths. Notable symbols: mul_zerotensor, correct_out_device, Device, meta_dks.
- **CN**: 第 1021-1050 行主要涉及变量/别名声明、注释或说明、返回路径。 值得关注的符号包括：mul_zerotensor, correct_out_device, Device, meta_dks。

### Lines 1051-1080 / 第 1051-1080 行
```cpp
1051:       return at::_efficientzerotensor(meta_out.sizes(), meta_out.options().device(out_device));
1052:     }
1053:   }
1054:   else {
1055:     if (other._is_zerotensor()) {
1056:       // x/0, return full INF
1057:       return at::full(meta_out.sizes(), std::numeric_limits<float>::infinity(), meta_out.options().device(out_device));
1058:     }
1059:     else {
1060:       // x/y -- unreachable, see TORCH_INTERNAL_ASSERT above
1061:       return at::_efficientzerotensor(meta_out.sizes(), meta_out.options().device(out_device));
1062:     }
1063:   }
1064: }
1065: 
1066: static Tensor maybe_add_maybe_sub(const Tensor& self, const Tensor& other, const Scalar& alpha) {
1067:   auto out_device = correct_out_device(self, other);
1068:   // hack to use the TensorIterator to get the correct broadcasting and type promotion logic
1069:   auto device_ = Device(DeviceType::Meta);
1070:   constexpr c10::DispatchKeySet meta_dks(at::DispatchKey::Meta);
1071:   auto self_meta = send_to_meta(self, device_);
1072:   auto other_meta = send_to_meta(other, device_);
1073:   auto meta_out = at::_ops::add_Tensor::redispatch(meta_dks, self_meta, other_meta, alpha);
1074: 
1075:   auto get_out_like = [&] (const Tensor& tensor)
1076:   {
1077:       auto sizes = meta_out.sizes();
1078:       return at::_to_copy(tensor.expand(sizes), meta_out.options().device(out_device));
1079:   };
1080: 
```
- **EN**: Lines 1051-1080 mainly cover state/variable declarations, expressions/calls, return paths. Notable symbols: _efficientzerotensor, sizes, options, device.
- **CN**: 第 1051-1080 行主要涉及变量/别名声明、表达式或调用、返回路径。 值得关注的符号包括：_efficientzerotensor, sizes, options, device。

### Lines 1081-1110 / 第 1081-1110 行
```cpp
1081:   if (self._is_zerotensor()) {
1082:     if (other._is_zerotensor()) {
1083:       return at::_efficientzerotensor(meta_out.sizes(), meta_out.options().device(out_device));
1084:     }
1085:     auto res = get_out_like(other);
1086:     return alpha.equal(1) ? std::move(res) : res.mul(alpha);
1087:   } else {
1088:     return get_out_like(self);
1089:   }
1090: }
1091: Tensor add_zerotensor(const Tensor& self, const Tensor& other, const Scalar& alpha) {
1092:   return maybe_add_maybe_sub(self, other, alpha);
1093: }
1094: 
1095: Tensor sub_zerotensor(const Tensor& self, const Tensor& other, const Scalar& alpha) {
1096:   return maybe_add_maybe_sub(self, other, -alpha);
1097: }
1098: 
1099: Tensor linalg_cross_zerotensor(
1100:   const Tensor& input,
1101:   const Tensor& other,
1102:   const int64_t dim)
1103: {
1104:   auto out_device = correct_out_device(input, other);
1105:   // hack to use the TensorIterator to get the correct broadcasting and type
1106:   // promotion logic (see add_zerotensor)
1107:   auto device = Device(DeviceType::Meta);
1108:   auto meta_out = at::_ops::linalg_cross::redispatch(
1109:     c10::DispatchKeySet(at::DispatchKey::Meta),
1110:     input.to(device),
```
- **EN**: Lines 1081-1110 mainly cover expressions/calls, function signatures/definitions, return paths. Notable symbols: _is_zerotensor, _efficientzerotensor, sizes, options.
- **CN**: 第 1081-1110 行主要涉及表达式或调用、函数签名或实现、返回路径。 值得关注的符号包括：_is_zerotensor, _efficientzerotensor, sizes, options。

### Lines 1111-1140 / 第 1111-1140 行
```cpp
1111:     other.to(device),
1112:     dim);
1113: 
1114:   return at::_efficientzerotensor(
1115:     meta_out.sizes(),
1116:     meta_out.options().device(out_device));
1117: }
1118: 
1119: // multiply, alias for mul
1120: Tensor& multiply_out(const Tensor& self, const Tensor& other, Tensor& result) {
1121:   return at::mul_out(result, self, other);
1122: }
1123: 
1124: Tensor multiply(const Tensor& self, const Tensor& other) {
1125:   return self.mul(other);
1126: }
1127: 
1128: Tensor& multiply_(Tensor& self, const Tensor& other) {
1129:   return self.mul_(other);
1130: }
1131: 
1132: Tensor multiply(const Tensor& self, const Scalar& other) {
1133:   return self.mul(other);
1134: }
1135: 
1136: Tensor& multiply_(Tensor& self, const Scalar& other) {
1137:   return self.mul_(other);
1138: }
1139: 
1140: Tensor sub(const Tensor& self, const Scalar& other, const Scalar& alpha) {
```
- **EN**: Lines 1111-1140 mainly cover function signatures/definitions, return paths, expressions/calls. Notable symbols: to, _efficientzerotensor, sizes, options.
- **CN**: 第 1111-1140 行主要涉及函数签名或实现、返回路径、表达式或调用。 值得关注的符号包括：to, _efficientzerotensor, sizes, options。

### Lines 1141-1170 / 第 1141-1170 行
```cpp
1141:   return at::sub(self, wrapped_scalar_tensor(other), alpha); // redispatch!
1142: }
1143: 
1144: Tensor& sub_(Tensor& self, const Scalar& other, const Scalar& alpha) {
1145:   return self.sub_(wrapped_scalar_tensor(other), alpha); // redispatch!
1146: }
1147: 
1148: // subtract, alias for sub
1149: Tensor& subtract_out(const Tensor& self, const Tensor& other, const Scalar& alpha, Tensor& result) {
1150:   return at::sub_out(result, self, other, alpha);
1151: }
1152: 
1153: Tensor subtract(const Tensor& self, const Tensor& other, const Scalar& alpha) {
1154:   return self.sub(other, alpha);
1155: }
1156: 
1157: Tensor& subtract_(Tensor& self, const Tensor& other, const Scalar& alpha) {
1158:   return self.sub_(other, alpha);
1159: }
1160: 
1161: Tensor subtract(const Tensor& self, const Scalar& other, const Scalar& alpha) {
1162:   return self.sub(other, alpha);
1163: }
1164: 
1165: Tensor& subtract_(Tensor& self, const Scalar& other, const Scalar& alpha) {
1166:   return self.sub_(other, alpha);
1167: }
1168: 
1169: Tensor rsub(const Tensor& self, const Tensor& other, const Scalar& alpha) {
1170:   return at::sub(other, self, alpha); // redispatch!
```
- **EN**: Lines 1141-1170 mainly cover return paths, expressions/calls, function signatures/definitions. Notable symbols: sub, wrapped_scalar_tensor, sub_, subtract_out.
- **CN**: 第 1141-1170 行主要涉及返回路径、表达式或调用、函数签名或实现。 值得关注的符号包括：sub, wrapped_scalar_tensor, sub_, subtract_out。

### Lines 1171-1200 / 第 1171-1200 行
```cpp
1171: }
1172: 
1173: // TODO: Make this structured to undo the perf regression from native:: removal
1174: // in call here
1175: 
1176: Tensor add(const Tensor& self, const Scalar& other, const Scalar& alpha) {
1177:   return at::add(self, wrapped_scalar_tensor(other), alpha);
1178: }
1179: 
1180: Tensor& add_(Tensor& self, const Scalar& other, const Scalar& alpha) {
1181:   return self.add_(wrapped_scalar_tensor(other), alpha);
1182: }
1183: 
1184: Tensor remainder(const Tensor& self, const Scalar& other) {
1185:   // redispatch
1186:   return at::remainder(self, wrapped_scalar_tensor(other));
1187: }
1188: 
1189: Tensor& remainder_(Tensor& self, const Scalar& other) {
1190:   // redispatch
1191:   return self.remainder_(wrapped_scalar_tensor(other));
1192: }
1193: 
1194: Tensor& remainder_out(const Tensor& self, const Scalar& other, Tensor& result) {
1195:   // redispatch
1196:   return at::remainder_out(result, self, wrapped_scalar_tensor(other));
1197: }
1198: 
1199: Tensor remainder(const Scalar& self, const Tensor& other) {
1200:   return at::remainder(wrapped_scalar_tensor(self), other);
```
- **EN**: Lines 1171-1200 mainly cover expressions/calls, function signatures/definitions, return paths. Notable symbols: add, wrapped_scalar_tensor, add_, remainder.
- **CN**: 第 1171-1200 行主要涉及表达式或调用、函数签名或实现、返回路径。 值得关注的符号包括：add, wrapped_scalar_tensor, add_, remainder。

### Lines 1201-1230 / 第 1201-1230 行
```cpp
1201: }
1202: 
1203: Tensor rsub(const Tensor& self, const Scalar& other, const Scalar& alpha) {
1204:   return native::rsub(self, wrapped_scalar_tensor(other), alpha);
1205: }
1206: 
1207: Tensor& bitwise_and_out(const Tensor& self, const Scalar& other, Tensor& result) {
1208:   return at::bitwise_and_out(result, self, wrapped_scalar_tensor(other));
1209: }
1210: 
1211: Tensor bitwise_and(const Tensor& self, const Scalar& other) {
1212:   return at::bitwise_and(self, wrapped_scalar_tensor(other));
1213: }
1214: 
1215: Tensor bitwise_and(const Scalar& self, const Tensor& other) {
1216:   return at::bitwise_and(wrapped_scalar_tensor(self), other);
1217: }
1218: 
1219: Tensor& bitwise_and_(Tensor& self, const Scalar& other) {
1220:   return self.bitwise_and_(wrapped_scalar_tensor(other));
1221: }
1222: 
1223: // Legacy and interfaces. They are aliased to bitwise_and* functions
1224: Tensor __and__(const Tensor& self, const Tensor& other) {
1225:   return at::bitwise_and(self, other);
1226: }
1227: 
1228: Tensor __and__(const Tensor& self, const Scalar& other) {
1229:   return at::bitwise_and(self, other);
1230: }
```
- **EN**: Lines 1201-1230 mainly cover expressions/calls, function signatures/definitions, return paths. Notable symbols: rsub, wrapped_scalar_tensor, bitwise_and_out, bitwise_and.
- **CN**: 第 1201-1230 行主要涉及表达式或调用、函数签名或实现、返回路径。 值得关注的符号包括：rsub, wrapped_scalar_tensor, bitwise_and_out, bitwise_and。

### Lines 1231-1260 / 第 1231-1260 行
```cpp
1231: 
1232: Tensor& __iand__(Tensor& self, const Tensor& other) {
1233:   return self.bitwise_and_(other);
1234: }
1235: 
1236: Tensor& __iand__(Tensor& self, const Scalar& other) {
1237:   return self.bitwise_and_(other);
1238: }
1239: 
1240: Tensor& bitwise_or_out(const Tensor& self, const Scalar& other, Tensor& result) {
1241:   return at::bitwise_or_out(result, self, wrapped_scalar_tensor(other));
1242: }
1243: 
1244: Tensor bitwise_or(const Tensor& self, const Scalar& other) {
1245:   return at::bitwise_or(self, wrapped_scalar_tensor(other));
1246: }
1247: 
1248: Tensor bitwise_or(const Scalar& self, const Tensor& other) {
1249:   return at::bitwise_or(wrapped_scalar_tensor(self), other);
1250: }
1251: 
1252: Tensor& bitwise_or_(Tensor& self, const Scalar& other) {
1253:   return self.bitwise_or_(wrapped_scalar_tensor(other));
1254: }
1255: 
1256: // Legacy or interfaces. They are aliased to bitwise_or* functions
1257: Tensor __or__(const Tensor& self, const Tensor& other) {
1258:   return at::bitwise_or(self, other);
1259: }
1260: 
```
- **EN**: Lines 1231-1260 mainly cover function signatures/definitions, return paths, expressions/calls. Notable symbols: __iand__, bitwise_and_, bitwise_or_out, wrapped_scalar_tensor.
- **CN**: 第 1231-1260 行主要涉及函数签名或实现、返回路径、表达式或调用。 值得关注的符号包括：__iand__, bitwise_and_, bitwise_or_out, wrapped_scalar_tensor。

### Lines 1261-1290 / 第 1261-1290 行
```cpp
1261: Tensor __or__(const Tensor& self, const Scalar& other) {
1262:   return at::bitwise_or(self, other);
1263: }
1264: 
1265: Tensor& __ior__(Tensor& self, const Tensor& other) {
1266:   return self.bitwise_or_(other);
1267: }
1268: 
1269: Tensor& __ior__(Tensor& self, const Scalar& other) {
1270:   return self.bitwise_or_(other);
1271: }
1272: 
1273: Tensor& bitwise_xor_out(const Tensor& self, const Scalar& other, Tensor& result) {
1274:   return at::bitwise_xor_out(result, self, wrapped_scalar_tensor(other));
1275: }
1276: 
1277: Tensor bitwise_xor(const Tensor& self, const Scalar& other) {
1278:   return at::bitwise_xor(self, wrapped_scalar_tensor(other));
1279: }
1280: 
1281: Tensor bitwise_xor(const Scalar& self, const Tensor& other) {
1282:   return at::bitwise_xor(wrapped_scalar_tensor(self), other);
1283: }
1284: 
1285: Tensor& bitwise_xor_(Tensor& self, const Scalar& other) {
1286:   return self.bitwise_xor_(wrapped_scalar_tensor(other));
1287: }
1288: 
1289: // Legacy xor interfaces. They are aliased to bitwise_xor* functions
1290: Tensor __xor__(const Tensor& self, const Tensor& other) {
```
- **EN**: Lines 1261-1290 mainly cover function signatures/definitions, return paths, expressions/calls. Notable symbols: __or__, bitwise_or, __ior__, bitwise_or_.
- **CN**: 第 1261-1290 行主要涉及函数签名或实现、返回路径、表达式或调用。 值得关注的符号包括：__or__, bitwise_or, __ior__, bitwise_or_。

### Lines 1291-1320 / 第 1291-1320 行
```cpp
1291:   return at::bitwise_xor(self, other);
1292: }
1293: 
1294: Tensor __xor__(const Tensor& self, const Scalar& other) {
1295:   return at::bitwise_xor(self, other);
1296: }
1297: 
1298: Tensor& __ixor__(Tensor& self, const Tensor& other) {
1299:   return self.bitwise_xor_(other);
1300: }
1301: 
1302: Tensor& __ixor__(Tensor& self, const Scalar& other) {
1303:   return self.bitwise_xor_(other);
1304: }
1305: 
1306: Tensor __lshift__(const Tensor& self, const Tensor& other) {
1307:   Tensor result;
1308:   auto iter = TensorIterator::binary_op(result, self, other);
1309:   lshift_stub(iter.device_type(), iter);
1310:   return iter.output();
1311: }
1312: 
1313: Tensor __lshift__(const Tensor& self, const Scalar& other) {
1314:   Tensor result;
1315:   auto wrapper = wrapped_scalar_tensor(other);
1316:   auto iter = TensorIterator::binary_op(result, self, wrapper);
1317:   lshift_stub(iter.device_type(), iter);
1318:   return iter.output();
1319: }
1320: 
```
- **EN**: Lines 1291-1320 mainly cover state/variable declarations, return paths, expressions/calls. Notable symbols: bitwise_xor, __xor__, __ixor__, bitwise_xor_.
- **CN**: 第 1291-1320 行主要涉及变量/别名声明、返回路径、表达式或调用。 值得关注的符号包括：bitwise_xor, __xor__, __ixor__, bitwise_xor_。

### Lines 1321-1350 / 第 1321-1350 行
```cpp
1321: Tensor& __ilshift__(Tensor& self, const Tensor& other) {
1322:   auto iter = TensorIterator::binary_op(self, self, other);
1323:   lshift_stub(iter.device_type(), iter);
1324:   return self;
1325: }
1326: 
1327: Tensor& __ilshift__(Tensor& self, const Scalar& other) {
1328:   auto wrapper = wrapped_scalar_tensor(other);
1329:   auto iter = TensorIterator::binary_op(self, self, wrapper);
1330:   lshift_stub(iter.device_type(), iter);
1331:   return self;
1332: }
1333: 
1334: TORCH_IMPL_FUNC(bitwise_left_shift_out) (const Tensor& self, const Tensor& other, const Tensor& result) {
1335:   lshift_stub(device_type(), *this);
1336: }
1337: 
1338: Tensor& bitwise_left_shift_out(const Tensor& self, const Scalar& other, Tensor& result) {
1339:   return at::bitwise_left_shift_out(result, self, wrapped_scalar_tensor(other));
1340: }
1341: 
1342: Tensor bitwise_left_shift(const Tensor& self, const Scalar& other) {
1343:   return at::bitwise_left_shift(self, wrapped_scalar_tensor(other));
1344: }
1345: 
1346: Tensor& bitwise_left_shift_(Tensor& self, const Scalar& other) {
1347:   return at::bitwise_left_shift_out(self, self, wrapped_scalar_tensor(other));
1348: }
1349: 
1350: Tensor bitwise_left_shift(const Scalar& self, const Tensor& other) {
```
- **EN**: Lines 1321-1350 mainly cover function signatures/definitions, state/variable declarations, expressions/calls. Notable symbols: __ilshift__, binary_op, lshift_stub, device_type.
- **CN**: 第 1321-1350 行主要涉及函数签名或实现、变量/别名声明、表达式或调用。 值得关注的符号包括：__ilshift__, binary_op, lshift_stub, device_type。

### Lines 1351-1380 / 第 1351-1380 行
```cpp
1351:   return at::bitwise_left_shift(wrapped_scalar_tensor(self), other);
1352: }
1353: 
1354: Tensor __rshift__(const Tensor& self, const Tensor& other) {
1355:   Tensor result;
1356:   auto iter = TensorIterator::binary_op(result, self, other);
1357:   rshift_stub(iter.device_type(), iter);
1358:   return iter.output();
1359: }
1360: 
1361: Tensor __rshift__(const Tensor& self, const Scalar& other) {
1362:   Tensor result;
1363:   auto wrapper = wrapped_scalar_tensor(other);
1364:   auto iter = TensorIterator::binary_op(result, self, wrapper);
1365:   rshift_stub(iter.device_type(), iter);
1366:   return iter.output();
1367: }
1368: 
1369: Tensor& __irshift__(Tensor& self, const Tensor& other) {
1370:   auto iter = TensorIterator::binary_op(self, self, other);
1371:   rshift_stub(iter.device_type(), iter);
1372:   return self;
1373: }
1374: 
1375: Tensor& __irshift__(Tensor& self, const Scalar& other) {
1376:   auto wrapper = wrapped_scalar_tensor(other);
1377:   auto iter = TensorIterator::binary_op(self, self, wrapper);
1378:   rshift_stub(iter.device_type(), iter);
1379:   return self;
1380: }
```
- **EN**: Lines 1351-1380 mainly cover state/variable declarations, return paths, expressions/calls. Notable symbols: bitwise_left_shift, wrapped_scalar_tensor, __rshift__, binary_op.
- **CN**: 第 1351-1380 行主要涉及变量/别名声明、返回路径、表达式或调用。 值得关注的符号包括：bitwise_left_shift, wrapped_scalar_tensor, __rshift__, binary_op。

### Lines 1381-1410 / 第 1381-1410 行
```cpp
1381: 
1382: TORCH_IMPL_FUNC(bitwise_right_shift_out) (const Tensor& self, const Tensor& other, const Tensor& result) {
1383:   rshift_stub(device_type(), *this);
1384: }
1385: 
1386: Tensor& bitwise_right_shift_out(const Tensor& self, const Scalar& other, Tensor& result) {
1387:   return at::bitwise_right_shift_out(result, self, wrapped_scalar_tensor(other));
1388: }
1389: 
1390: Tensor bitwise_right_shift(const Tensor& self, const Scalar& other) {
1391:   return at::bitwise_right_shift(self, wrapped_scalar_tensor(other));
1392: }
1393: 
1394: Tensor& bitwise_right_shift_(Tensor& self, const Scalar& other) {
1395:   return at::bitwise_right_shift_out(self, self, wrapped_scalar_tensor(other));
1396: }
1397: 
1398: Tensor bitwise_right_shift(const Scalar& self, const Tensor& other) {
1399:   return at::bitwise_right_shift(wrapped_scalar_tensor(self), other);
1400: }
1401: 
1402: template <typename Stub>
1403: static Tensor& comparison_op_out(Tensor& result, const Tensor& self, const Tensor& other, Stub& stub) {
1404:   auto iter = TensorIterator::comparison_op(result, self, other);
1405:   stub(iter.device_type(), iter);
1406:   return result;
1407: }
1408: 
1409: template <typename OutImpl>
1410: static Tensor comparison_op(const Tensor& self, const Tensor& other, OutImpl& out_impl) {
```
- **EN**: Lines 1381-1410 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: TORCH_IMPL_FUNC, rshift_stub, device_type, bitwise_right_shift_out.
- **CN**: 第 1381-1410 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：TORCH_IMPL_FUNC, rshift_stub, device_type, bitwise_right_shift_out。

### Lines 1411-1440 / 第 1411-1440 行
```cpp
1411:   Tensor result = at::empty({0}, self.options().dtype(kBool));
1412:   return out_impl(result, self, other);
1413: }
1414: 
1415: template <typename OutImpl>
1416: static Tensor& comparison_op_(Tensor& self, const Tensor& other, OutImpl& out_impl) {
1417:   return out_impl(self, self, other);
1418: }
1419: 
1420: template <typename OutImpl>
1421: static Tensor& comparison_op_out(Tensor& result, const Tensor& self, const Scalar& other, OutImpl& out_impl) {
1422:   return out_impl(result, self, wrapped_scalar_tensor(other));
1423: }
1424: 
1425: template <typename OutImpl>
1426: static Tensor comparison_op(const Tensor& self, const Scalar& other, OutImpl& out_impl) {
1427:   return comparison_op(self, wrapped_scalar_tensor(other), out_impl);
1428: }
1429: 
1430: template <typename OutImpl>
1431: static Tensor& comparison_op_(Tensor& self, const Scalar& other, OutImpl& out_impl) {
1432:   return out_impl(self, self, wrapped_scalar_tensor(other));
1433: }
1434: 
1435: // We need explicit cast to OutFunc because each *_out func is overloaded twice. Without An explicit cast, merely
1436: // referring to *_out function is ambiguous.
1437: using OutFunc = std::add_const_t<Tensor&(&)(Tensor&, const Tensor&, const Tensor&)>;
1438: 
1439: // less, alias for torch.lt
1440: Tensor& less_out(const Tensor& self, const Tensor& other, Tensor& result) { return at::lt_out(result, self, other); }
```
- **EN**: Lines 1411-1440 mainly cover state/variable declarations, return paths, expressions/calls. Notable symbols: empty, options, dtype, out_impl.
- **CN**: 第 1411-1440 行主要涉及变量/别名声明、返回路径、表达式或调用。 值得关注的符号包括：empty, options, dtype, out_impl。

### Lines 1441-1470 / 第 1441-1470 行
```cpp
1441: Tensor less(const Tensor& self, const Tensor& other) { return self.lt(other); }
1442: Tensor& less_(Tensor& self, const Tensor& other) { return self.lt_(other); }
1443: Tensor& less_out(const Tensor& self, const Scalar& other, Tensor& result) { return at::lt_out(result, self, other); }
1444: Tensor less(const Tensor& self, const Scalar& other) { return self.lt(other); }
1445: Tensor& less_(Tensor& self, const Scalar& other) { return self.lt_(other); }
1446: 
1447: // less_equal, alias for torch.le
1448: Tensor& less_equal_out(const Tensor& self, const Tensor& other, Tensor& result) { return at::le_out(result, self, other); }
1449: Tensor less_equal(const Tensor& self, const Tensor& other) { return self.le(other); }
1450: Tensor& less_equal_(Tensor& self, const Tensor& other) { return self.le_(other); }
1451: Tensor& less_equal_out(const Tensor& self, const Scalar& other, Tensor& result) { return at::le_out(result, self, other); }
1452: Tensor less_equal(const Tensor& self, const Scalar& other) { return self.le(other); }
1453: Tensor& less_equal_(Tensor& self, const Scalar& other) { return self.le_(other); }
1454: 
1455: // greater, alias for torch.gt
1456: Tensor& greater_out(const Tensor& self, const Tensor& other, Tensor& result) { return at::gt_out(result, self, other); }
1457: Tensor greater(const Tensor& self, const Tensor& other) { return self.gt(other); }
1458: Tensor& greater_(Tensor& self, const Tensor& other) { return self.gt_(other); }
1459: Tensor& greater_out(const Tensor& self, const Scalar& other, Tensor& result) { return at::gt_out(result, self, other); }
1460: Tensor greater(const Tensor& self, const Scalar& other) { return self.gt(other); }
1461: Tensor& greater_(Tensor& self, const Scalar& other) { return self.gt_(other); }
1462: 
1463: // greater_equal, alias for torch.ge
1464: Tensor& greater_equal_out(const Tensor& self, const Tensor& other, Tensor& result) { return at::ge_out(result, self, other); }
1465: Tensor greater_equal(const Tensor& self, const Tensor& other) { return self.ge(other); }
1466: Tensor& greater_equal_(Tensor& self, const Tensor& other) { return self.ge_(other); }
1467: Tensor& greater_equal_out(const Tensor& self, const Scalar& other, Tensor& result) { return at::ge_out(result, self, other); }
1468: Tensor greater_equal(const Tensor& self, const Scalar& other) { return self.ge(other); }
1469: Tensor& greater_equal_(Tensor& self, const Scalar& other) { return self.ge_(other); }
1470: 
```
- **EN**: Lines 1441-1470 mainly cover function signatures/definitions, comments/documentation. Notable symbols: less, lt, less_, lt_.
- **CN**: 第 1441-1470 行主要涉及函数签名或实现、注释或说明。 值得关注的符号包括：less, lt, less_, lt_。

### Lines 1471-1500 / 第 1471-1500 行
```cpp
1471: #define CREATE_COMPARISON_SCALAR_TENSOR_IMPL_FUNC(func)             \
1472:   TORCH_IMPL_FUNC(func##_Tensor_out)                                \
1473:   (const Tensor& self, const Tensor& other, const Tensor& result) { \
1474:     func##_stub(device_type(), *this);                              \
1475:   }                                                                 \
1476:                                                                     \
1477:   TORCH_IMPL_FUNC(func##_Scalar_out)                                \
1478:   (const Tensor& self, const Scalar& other, const Tensor& result) { \
1479:     func##_stub(device_type(), *this);                              \
1480:   }
1481: 
1482: CREATE_COMPARISON_SCALAR_TENSOR_IMPL_FUNC(eq)
1483: CREATE_COMPARISON_SCALAR_TENSOR_IMPL_FUNC(ne)
1484: CREATE_COMPARISON_SCALAR_TENSOR_IMPL_FUNC(gt)
1485: CREATE_COMPARISON_SCALAR_TENSOR_IMPL_FUNC(ge)
1486: CREATE_COMPARISON_SCALAR_TENSOR_IMPL_FUNC(lt)
1487: CREATE_COMPARISON_SCALAR_TENSOR_IMPL_FUNC(le)
1488: 
1489: // not_equal, alias for torch.ne
1490: Tensor& not_equal_out(const Tensor& self, const Tensor& other, Tensor& result) { return at::ne_out(result, self, other); }
1491: Tensor not_equal(const Tensor& self, const Tensor& other) { return self.ne(other); }
1492: Tensor& not_equal_(Tensor& self, const Tensor& other) { return self.ne_(other); }
1493: Tensor& not_equal_out(const Tensor& self, const Scalar& other, Tensor& result) { return at::ne_out(result, self, other); }
1494: Tensor not_equal(const Tensor& self, const Scalar& other) { return self.ne(other); }
1495: Tensor& not_equal_(Tensor& self, const Scalar& other) { return self.ne_(other); }
1496: 
1497: Tensor& logical_and_out(const Tensor& self, const Tensor& other, Tensor& result) { return comparison_op_out(result, self, other, logical_and_stub); }
1498: Tensor logical_and(const Tensor& self, const Tensor& other) { return comparison_op(self, other, static_cast<OutFunc>(at::logical_and_out)); }
1499: Tensor& logical_and_(Tensor& self, const Tensor& other) { return comparison_op_(self, other, static_cast<OutFunc>(at::logical_and_out)); }
1500: 
```
- **EN**: Lines 1471-1500 mainly cover function signatures/definitions, macro-based glue, expressions/calls. Notable symbols: CREATE_COMPARISON_SCALAR_TENSOR_IMPL_FUNC, TORCH_IMPL_FUNC, _stub, device_type.
- **CN**: 第 1471-1500 行主要涉及函数签名或实现、宏定义或宏调用、表达式或调用。 值得关注的符号包括：CREATE_COMPARISON_SCALAR_TENSOR_IMPL_FUNC, TORCH_IMPL_FUNC, _stub, device_type。

### Lines 1501-1530 / 第 1501-1530 行
```cpp
1501: Tensor& logical_or_out(const Tensor& self, const Tensor& other, Tensor& result) { return comparison_op_out(result, self, other, logical_or_stub); }
1502: Tensor logical_or(const Tensor& self, const Tensor& other) { return comparison_op(self, other, static_cast<OutFunc>(at::logical_or_out)); }
1503: Tensor& logical_or_(Tensor& self, const Tensor& other) { return comparison_op_(self, other, static_cast<OutFunc>(at::logical_or_out)); }
1504: 
1505: Tensor& logical_xor_out(const Tensor& self, const Tensor& other, Tensor& result) { return comparison_op_out(result, self, other, logical_xor_stub); }
1506: Tensor logical_xor(const Tensor& self, const Tensor& other) { return comparison_op(self, other, static_cast<OutFunc>(at::logical_xor_out)); }
1507: Tensor& logical_xor_(Tensor& self, const Tensor& other) { return comparison_op_(self, other, static_cast<OutFunc>(at::logical_xor_out)); }
1508: 
1509: // binary max, alias for maximum
1510: Tensor& max_out(const Tensor& self, const Tensor& other, Tensor& result) {
1511:   return at::maximum_out(result, self, other);
1512: }
1513: 
1514: Tensor max(const Tensor& self, const Tensor& other) {
1515:   return at::maximum(self, other);
1516: }
1517: 
1518: // binary min, alias for minimum
1519: Tensor& min_out(const Tensor& self, const Tensor& other, Tensor& result) {
1520:   return at::minimum_out(result, self, other);
1521: }
1522: 
1523: Tensor min(const Tensor& self, const Tensor& other) {
1524:   return at::minimum(self, other);
1525: }
1526: 
1527: Tensor floor_divide(const Tensor& self, const Scalar& other) {
1528:   return at::floor_divide(self, wrapped_scalar_tensor(other));
1529: }
1530: 
```
- **EN**: Lines 1501-1530 mainly cover function signatures/definitions, return paths, expressions/calls. Notable symbols: logical_or_out, comparison_op_out, logical_or, comparison_op.
- **CN**: 第 1501-1530 行主要涉及函数签名或实现、返回路径、表达式或调用。 值得关注的符号包括：logical_or_out, comparison_op_out, logical_or, comparison_op。

### Lines 1531-1560 / 第 1531-1560 行
```cpp
1531: Tensor& floor_divide_(Tensor& self, const Scalar& other) {
1532:   return at::floor_divide_out(self, self, wrapped_scalar_tensor(other));
1533: }
1534: 
1535: Tensor& fmod_out(const Tensor& self, const Scalar& other, Tensor & result) {
1536:   // redispatch
1537:   return at::fmod_out(result, self, wrapped_scalar_tensor(other));
1538: }
1539: 
1540: Tensor fmod(const Tensor& self, const Scalar& other) {
1541:   // redispatch
1542:   return at::fmod(self, wrapped_scalar_tensor(other));
1543: }
1544: 
1545: Tensor& fmod_(Tensor& self, const Scalar& other) {
1546:   // redispatch
1547:   return self.fmod_(wrapped_scalar_tensor(other));
1548: }
1549: 
1550: // Note: this function is only for testing.
1551: // It is undocumented and should not be used outside of tests.
1552: Tensor _test_serialization_subcmul(const Tensor& self, const Tensor& other, const Scalar& alpha) {
1553:   return self - (other * alpha);
1554: }
1555: 
1556: TORCH_IMPL_FUNC(heaviside_out) (
1557:   const Tensor& self, const Tensor& other, const Tensor& result
1558: ) {
1559:   heaviside_stub(device_type(), *this);
1560: }
```
- **EN**: Lines 1531-1560 mainly cover expressions/calls, function signatures/definitions, return paths. Notable symbols: floor_divide_, floor_divide_out, wrapped_scalar_tensor, fmod_out.
- **CN**: 第 1531-1560 行主要涉及表达式或调用、函数签名或实现、返回路径。 值得关注的符号包括：floor_divide_, floor_divide_out, wrapped_scalar_tensor, fmod_out。

### Lines 1561-1590 / 第 1561-1590 行
```cpp
1561: 
1562: static inline Tensor _pow2(const Tensor& self, const Tensor& other) {
1563:   const auto self_dtype = self.scalar_type();
1564:   // All integral types are promoted to float32
1565:   if (isIntegralType(self_dtype, true) || self_dtype == kFloat) {
1566:       return at::pow(2.0, other);
1567:   }
1568:   // For double and reduced floating types do regular type promotion
1569:   return at::full({}, 2.0, self.options()).pow(other);
1570: }
1571: 
1572: // This function is used to dispatch to kernels that use std::ldexp on CPU and the global namespaces ::ldexp on CUDA
1573: // Both of these require floating types for 'self' and integer types for 'other'.
1574: static inline Tensor& _ldexp_int_exponent(const Tensor& self, const Tensor& other, Tensor& result) {
1575:   auto iter = TensorIteratorConfig()
1576:     .check_all_same_dtype(false)
1577:     .add_output(result)
1578:     .add_input(self)
1579:     .add_input(other)
1580:     .build();
1581: 
1582:   ldexp_stub(iter.device_type(), iter);
1583:   return result;
1584: }
1585: 
1586: Tensor& ldexp_out(const Tensor& self, const Tensor& other, Tensor& result) {
1587:   TORCH_CHECK(!isIntegralType(result.scalar_type(), /*includeBool=*/true),
1588:               "ldexp can't be cast to the desired output type ", result.scalar_type());
1589: 
1590:   if (isIntegralType(other.scalar_type(), /*includeBool=*/true) &&
```
- **EN**: Lines 1561-1590 mainly cover state/variable declarations, function signatures/definitions, comments/documentation. Notable symbols: _pow2, scalar_type, isIntegralType, pow.
- **CN**: 第 1561-1590 行主要涉及变量/别名声明、函数签名或实现、注释或说明。 值得关注的符号包括：_pow2, scalar_type, isIntegralType, pow。

### Lines 1591-1620 / 第 1591-1620 行
```cpp
1591:       isFloatingType(self.scalar_type())) {
1592:     return _ldexp_int_exponent(self, other, result);
1593:   }
1594: 
1595:   return at::mul_out(result, self, _pow2(self, other));
1596: }
1597: 
1598: Tensor ldexp(const Tensor& self, const Tensor& other) {
1599:   if (isIntegralType(other.scalar_type(), /*includeBool=*/true) &&
1600:       isFloatingType(self.scalar_type())) {
1601:     Tensor result = at::empty_like(self);
1602:     return _ldexp_int_exponent(self, other, result);
1603:   }
1604: 
1605:   return at::mul(self, _pow2(self, other));
1606: }
1607: 
1608: Tensor& ldexp_(Tensor& self, const Tensor& other) {
1609:   return at::ldexp_out(self, self, other);
1610: }
1611: 
1612: Tensor& xlogy_out(const Scalar& self, const Tensor& other, Tensor& result) {
1613:   return at::xlogy_out(result, wrapped_scalar_tensor(self), other);
1614: }
1615: 
1616: Tensor& xlogy_out(const Tensor& self, const Scalar& other, Tensor& result) {
1617:   return at::xlogy_out(result, self, wrapped_scalar_tensor(other));
1618: }
1619: 
1620: Tensor xlogy(const Scalar& x, const Tensor& y) {
```
- **EN**: Lines 1591-1620 mainly cover function signatures/definitions, return paths, expressions/calls. Notable symbols: isFloatingType, scalar_type, _ldexp_int_exponent, mul_out.
- **CN**: 第 1591-1620 行主要涉及函数签名或实现、返回路径、表达式或调用。 值得关注的符号包括：isFloatingType, scalar_type, _ldexp_int_exponent, mul_out。

### Lines 1621-1650 / 第 1621-1650 行
```cpp
1621:   return at::xlogy(wrapped_scalar_tensor(x), y);
1622: }
1623: 
1624: Tensor xlogy(const Tensor& x, const Scalar& y) {
1625:   return at::xlogy(x, wrapped_scalar_tensor(y));
1626: }
1627: 
1628: Tensor& xlogy_(Tensor& x, const Scalar& y) {
1629:   return at::xlogy_(x, wrapped_scalar_tensor(y));
1630: }
1631: 
1632: Tensor& special_xlogy_out(const Tensor& self, const Tensor& other, Tensor& result) {
1633:   return at::xlogy_out(result, self, other);
1634: }
1635: 
1636: Tensor& special_xlogy_out(const Scalar& self, const Tensor& other, Tensor& result) {
1637:   return at::xlogy_out(result, self, other);
1638: }
1639: 
1640: Tensor& special_xlogy_out(const Tensor& self, const Scalar& other, Tensor& result) {
1641:   return at::xlogy_out(result, self, other);
1642: }
1643: 
1644: Tensor special_xlogy(const Tensor& x, const Tensor& y) {
1645:   return at::xlogy(x, y);
1646: }
1647: 
1648: Tensor special_xlogy(const Scalar& x, const Tensor& y) {
1649:   return at::xlogy(x, y);
1650: }
```
- **EN**: Lines 1621-1650 mainly cover return paths, expressions/calls, function signatures/definitions. Notable symbols: xlogy, wrapped_scalar_tensor, xlogy_, special_xlogy_out.
- **CN**: 第 1621-1650 行主要涉及返回路径、表达式或调用、函数签名或实现。 值得关注的符号包括：xlogy, wrapped_scalar_tensor, xlogy_, special_xlogy_out。

### Lines 1651-1656 / 第 1651-1656 行
```cpp
1651: 
1652: Tensor special_xlogy(const Tensor& x, const Scalar& y) {
1653:   return at::xlogy(x, y);
1654: }
1655: 
1656: } // namespace at::native
```
- **EN**: Lines 1651-1656 mainly cover function signatures/definitions, return paths, expressions/calls. Notable symbols: special_xlogy, xlogy.
- **CN**: 第 1651-1656 行主要涉及函数签名或实现、返回路径、表达式或调用。 值得关注的符号包括：special_xlogy, xlogy。

## Key Concepts / 关键概念
- **EN**: TensorIterator-driven traversal  
  **CN**: 基于 TensorIterator 的遍历
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Dispatch stub definition  
  **CN**: 调度桩定义
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/native/BinaryOps.h>`, `<type_traits>`, `<utility>`, `<ATen/core/Tensor.h>`, `<ATen/ScalarOps.h>`, `<ATen/TensorIterator.h>`, `<ATen/TensorOperators.h>`, `<ATen/TensorMeta.h>`, `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `DEFINE_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
