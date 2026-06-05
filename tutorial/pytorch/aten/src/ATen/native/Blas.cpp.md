# Blas.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/Blas.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Blas.
- **Purpose (CN)**: 实现或声明与 blas 相关的 ATen 原生逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/core/Tensor.h>
0003: #include <ATen/core/NamedTensor.h>
0004: #include <ATen/Dispatch.h>
0005: #include <ATen/ExpandUtils.h>
0006: #include <ATen/NamedTensorUtils.h>
0007: #include <ATen/Config.h>
0008: 
0009: #include <ATen/native/mkldnn/Matmul.h>
0010: #include <ATen/native/mkldnn/Linear.h>
0011: #include <ATen/native/Resize.h>
0012: #include <ATen/native/GroupedMMUtils.h>
0013: #include <ATen/BlasBackend.h>
0014: #if !defined(__s390x__) && !defined(__powerpc__)
0015: #include <cpuinfo.h>
0016: #endif
0017: 
0018: #ifndef AT_PER_OPERATOR_HEADERS
0019: #include <ATen/CPUFunctions.h>
0020: #include <ATen/Functions.h>
0021: #include <ATen/NativeFunctions.h>
0022: #else
0023: #include <ATen/ops/_efficientzerotensor.h>
0024: #include <ATen/ops/addmv.h>
0025: #include <ATen/ops/addmv_native.h>
0026: #include <ATen/ops/copy_native.h>
0027: #include <ATen/ops/dot.h>
0028: #include <ATen/ops/dot_native.h>
0029: #include <ATen/ops/empty.h>
0030: #include <ATen/ops/mul_cpu_dispatch.h>
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, macro-based glue. Notable symbols: defined.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、宏定义或宏调用。 值得关注的符号包括：defined。

### Lines 31-60 / 第 31-60 行
```cpp
0031: #include <ATen/ops/mv_native.h>
0032: #include <ATen/ops/scalar_tensor_native.h>
0033: #include <ATen/ops/vdot_native.h>
0034: #include <ATen/ops/_scaled_mm_native.h>
0035: #include <ATen/ops/mul.h>
0036: #include <ATen/ops/matmul.h>
0037: #endif
0038: 
0039: namespace at::meta {
0040: TORCH_META_FUNC(addmv)(const Tensor &self, const Tensor &mat, const Tensor &vec, const Scalar& beta, const Scalar& alpha) {
0041:   TORCH_CHECK((mat.dim() == 2 && vec.dim() == 1 && self.dim() <= 1),
0042:     "vector + matrix @ vector expected, got ", self.dim(), ", ", mat.dim(), ", ", vec.dim());
0043: 
0044:   TORCH_CHECK(mat.size(1) == vec.size(0) && (mat.size(0) == self.numel() || self.numel() == 1),
0045:     "size mismatch, got input (", self.size(0), "), mat (", mat.size(0), "x", mat.size(1), "), vec (", vec.size(0), ")");
0046: 
0047:   TORCH_CHECK(self.scalar_type() == mat.scalar_type() && mat.scalar_type() == vec.scalar_type(),
0048:     "addmv input tensors must have the same dtype, but got ", self.scalar_type(), ", ", mat.scalar_type(), ", and ", vec.scalar_type());
0049:   auto names = at::namedinference::propagate_names_for_addmv(mat, vec, self);
0050:   set_output_raw_strided(0, IntArrayRef(mat.sizes().data(), 1), {}, vec.options(), names);
0051: }
0052: } // namespace at::meta
0053: 
0054: namespace at::native {
0055: 
0056: template<typename scalar_t>
0057: void gemv(char trans, int64_t m, int64_t n, scalar_t alpha, const scalar_t *a, int64_t lda, const scalar_t *x, int64_t incx, scalar_t beta, scalar_t *y, int64_t incy);
0058: 
0059: template<typename scalar_t>
0060: scalar_t dot_impl(int64_t n, const scalar_t *x, int64_t incx, const scalar_t *y, int64_t incy);
```
- **EN**: Lines 31-60 mainly cover state/variable declarations, header inclusion, macro-based glue. Notable symbols: TORCH_META_FUNC, TORCH_CHECK, dim, size.
- **CN**: 第 31-60 行主要涉及变量/别名声明、头文件包含、宏定义或宏调用。 值得关注的符号包括：TORCH_META_FUNC, TORCH_CHECK, dim, size。

### Lines 61-90 / 第 61-90 行
```cpp
0061: 
0062: template<typename scalar_t>
0063: scalar_t vdot_impl(int64_t n, const scalar_t *x, int64_t incx, const scalar_t *y, int64_t incy);
0064: 
0065: static constexpr bool lda_cond(int64_t m, int64_t n, int64_t lda) {
0066:   return n == 1 || lda >= std::max<int64_t>(1L, m);
0067: }
0068: 
0069: 
0070: 
0071: 
0072: TORCH_IMPL_FUNC(addmv_out_cpu)(const Tensor &self, const Tensor &mat, const Tensor &vec, const Scalar& beta_, const Scalar& alpha_, const Tensor& result) {
0073:   c10::MaybeOwned<Tensor> self_ = expand_size(self, {mat.size(0)});
0074:   auto betaval = beta_.toComplexDouble();
0075:   if (mat.numel() == 0) {
0076:     // shortcut for an empty matrix
0077:     // By definition, when beta==0, values in self should be ignored. nans and infs
0078:     // should not propagate
0079:     if (betaval == 0.0) {
0080:       result.zero_();
0081:     } else {
0082:       at::cpu::mul_out(
0083:           // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
0084:           const_cast<Tensor&>(result),
0085:           self,
0086:           at::native::scalar_tensor(
0087:               beta_, self.scalar_type(), std::nullopt /* layout */, at::kCPU, std::nullopt /* pin_memory */));
0088:     }
0089:   } else {
0090:     if (!result.is_same(*self_) && betaval != 0.0) { //if beta is 0, result contents is ignored
```
- **EN**: Lines 61-90 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: vdot_impl, lda_cond, TORCH_IMPL_FUNC, expand_size.
- **CN**: 第 61-90 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：vdot_impl, lda_cond, TORCH_IMPL_FUNC, expand_size。

### Lines 91-120 / 第 91-120 行
```cpp
0091:       // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
0092:       at::native::copy_(const_cast<Tensor&>(result), *self_);
0093:     }
0094:     if (result.numel() != 0) {
0095: 
0096:       NoNamesGuard guard;
0097:       if (use_mkldnn_matmul(mat, vec, /*result=*/Tensor())){
0098:         mkldnn_matmul(mat, vec, result, beta_.to<float>(), alpha_.to<float>());
0099:         return;
0100:       }
0101: 
0102:       auto r_stride = result.stride(0);
0103:       AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND2(kBFloat16, kHalf, mat.scalar_type(), "addmv_impl_cpu", [&] {
0104:         auto beta = beta_.to<scalar_t>();
0105:         auto alpha = alpha_.to<scalar_t>();
0106:         if (mat.stride(0) == 1 && lda_cond(mat.size(0), mat.size(1), mat.stride(1))) {
0107:           gemv<scalar_t>('n', mat.size(0), mat.size(1), alpha, mat.const_data_ptr<scalar_t>(), mat.stride(1),
0108:               vec.const_data_ptr<scalar_t>(), vec.stride(0), beta, result.mutable_data_ptr<scalar_t>(), r_stride);
0109:         }
0110:         else if (mat.stride(1) == 1 && lda_cond(mat.size(1), mat.size(0), mat.stride(0))) {
0111:           gemv<scalar_t>('t', mat.size(1), mat.size(0), alpha, mat.const_data_ptr<scalar_t>(), mat.stride(0),
0112:               vec.const_data_ptr<scalar_t>(), vec.stride(0), beta, result.mutable_data_ptr<scalar_t>(), r_stride);
0113:         }
0114:         else {
0115:           Tensor cmat = mat.contiguous();
0116:           gemv<scalar_t>('t', mat.size(1), mat.size(0), alpha, cmat.const_data_ptr<scalar_t>(), cmat.stride(0),
0117:               vec.const_data_ptr<scalar_t>(), vec.stride(0), beta, result.mutable_data_ptr<scalar_t>(), r_stride);
0118:         }
0119:       });
0120:     }
```
- **EN**: Lines 91-120 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: NOLINTNEXTLINE, copy_, numel, use_mkldnn_matmul.
- **CN**: 第 91-120 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：NOLINTNEXTLINE, copy_, numel, use_mkldnn_matmul。

### Lines 121-150 / 第 121-150 行
```cpp
0121:   }
0122: }
0123: 
0124: Tensor &mv_out(const Tensor &self, const Tensor &vec, Tensor& result) {
0125:   //self arg sent to addmv_out cannot be resized
0126:   //here we use result as self argument for addmv, and result is user supplied and can be wrong size
0127:   //it's not a hard error, because we allow resizing result, but it becomes a hard error
0128:   //in addmv, because addmv expects self to satisfy proper conditions
0129:   //to avoid this, supply correctly sized self, its contents doesn't matter because beta is 0
0130:   if (result.dim() > 1 || (result.numel() != self.size(0) && result.numel() != 1)) {
0131:     Tensor self_addmv = at::empty({self.size(0)}, vec.options());
0132:     return at::addmv_out(result, self_addmv, self, vec, 0, 1);
0133:   }
0134:   return at::addmv_out(result, result, self, vec, 0, 1);
0135: }
0136: 
0137: Tensor mv(const Tensor &self, const Tensor &vec) {
0138:   Tensor result = at::empty({self.size(0)}, vec.options());
0139:   //inplace version is more efficient if we can use it
0140:   return at::addmv_(result, self, vec, 0, 1);
0141: }
0142: 
0143: static inline void dot_check(const Tensor& self, const Tensor& other) {
0144:   TORCH_CHECK(
0145:       self.dim() == 1 && other.dim() == 1,
0146:       "1D tensors expected, but got ",
0147:       self.dim(),
0148:       "D and ",
0149:       other.dim(),
0150:       "D tensors");
```
- **EN**: Lines 121-150 mainly cover expressions/calls, comments/documentation, function signatures/definitions. Notable symbols: mv_out, dim, numel, size.
- **CN**: 第 121-150 行主要涉及表达式或调用、注释或说明、函数签名或实现。 值得关注的符号包括：mv_out, dim, numel, size。

### Lines 151-180 / 第 151-180 行
```cpp
0151: 
0152:   TORCH_CHECK(
0153:       self.scalar_type() == other.scalar_type(),
0154:       "dot : expected both vectors to have same dtype, but found ",
0155:       self.scalar_type(),
0156:       " and ",
0157:       other.scalar_type());
0158: 
0159:   TORCH_CHECK(
0160:       self.numel() == other.numel(),
0161:       "inconsistent tensor size, expected tensor [",
0162:       self.numel(),
0163:       "] and src [",
0164:       other.numel(),
0165:       "] to have the same number of elements, but got ",
0166:       self.numel(),
0167:       " and ",
0168:       other.numel(),
0169:       " elements respectively");
0170: }
0171: 
0172: Tensor dot(const Tensor &self, const Tensor &other){
0173:   if (self.is_complex()) {
0174:     if (self.is_conj()) {
0175:       if (other.is_conj()) {
0176:         return (at::native::dot(self.conj(), other.conj())).conj();
0177:        } else {
0178:          return at::native::vdot(self.conj(), other);
0179:        }
0180:     } else if (other.is_conj()) {
```
- **EN**: Lines 151-180 mainly cover function signatures/definitions, expressions/calls, control-flow checks. Notable symbols: TORCH_CHECK, scalar_type, numel, dot.
- **CN**: 第 151-180 行主要涉及函数签名或实现、表达式或调用、控制流逻辑。 值得关注的符号包括：TORCH_CHECK, scalar_type, numel, dot。

### Lines 181-210 / 第 181-210 行
```cpp
0181:       return at::native::vdot(other.conj(), self);
0182:     }
0183:   }
0184: 
0185:   at::NoNamesGuard guard;
0186:   dot_check(self, other);
0187: 
0188:   if (self._is_zerotensor() || other._is_zerotensor()) {
0189:     return at::_efficientzerotensor({}, self.options());
0190:   }
0191: 
0192:   if (use_mkldnn_matmul(self, other, /*result=*/Tensor())){
0193:     // mkldnn matmul expect result have sizes info to create ideep tensor
0194:     auto r =  at::empty({1, 1}, self.options());
0195:     mkldnn_matmul(self, other, r, /*beta=*/0);
0196:     return r;
0197:   }
0198: 
0199:   return AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND2(at::ScalarType::BFloat16, at::ScalarType::Half, self.scalar_type(), "dot", [&] {
0200:     Tensor result = at::empty({}, self.options());
0201:     result.fill_(dot_impl<scalar_t>(self.numel(), self.const_data_ptr<scalar_t>(), self.stride(0), other.const_data_ptr<scalar_t>(), other.stride(0)));
0202:     return result;
0203:   });
0204: }
0205: 
0206: Tensor vdot(const Tensor &self, const Tensor &other){
0207:   // Dispatch to `dot` for real dtypes.
0208:   if (!self.is_complex()){
0209:     return at::dot(self, other);
0210:   }
```
- **EN**: Lines 181-210 mainly cover state/variable declarations, return paths, expressions/calls. Notable symbols: vdot, conj, dot_check, _is_zerotensor.
- **CN**: 第 181-210 行主要涉及变量/别名声明、返回路径、表达式或调用。 值得关注的符号包括：vdot, conj, dot_check, _is_zerotensor。

### Lines 211-238 / 第 211-238 行
```cpp
0211: 
0212:   if (self.is_conj()) {
0213:     if (other.is_conj()) {
0214:       return at::native::vdot(other.conj(), self.conj());
0215:     } else {
0216:       return at::native::dot(self.conj(), other);
0217:     }
0218:   } else if (other.is_conj()) {
0219:     return (at::native::dot(self, other.conj())).conj();
0220:   }
0221: 
0222:   at::NoNamesGuard guard;
0223:   // For complex dtypes.
0224:   dot_check(self, other);
0225: 
0226:   if (self._is_zerotensor() || other._is_zerotensor()) {
0227:     return at::_efficientzerotensor({}, self.options());
0228:   }
0229: 
0230:   return AT_DISPATCH_COMPLEX_TYPES(self.scalar_type(), "vdot", [&] {
0231:     Tensor result = at::empty({}, self.options());
0232:     result.fill_(vdot_impl<scalar_t>(self.numel(), self.const_data_ptr<scalar_t>(), self.stride(0), other.const_data_ptr<scalar_t>(), other.stride(0)));
0233:     return result;
0234:   });
0235: 
0236: }
0237: 
0238: }  // namespace at::native
```
- **EN**: Lines 211-238 mainly cover return paths, expressions/calls, state/variable declarations. Notable symbols: is_conj, vdot, conj, dot.
- **CN**: 第 211-238 行主要涉及返回路径、表达式或调用、变量/别名声明。 值得关注的符号包括：is_conj, vdot, conj, dot。

## Key Concepts / 关键概念
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现
- **EN**: Scalar/tensor mixed arithmetic  
  **CN**: 标量与张量混合运算

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/core/NamedTensor.h>`, `<ATen/Dispatch.h>`, `<ATen/ExpandUtils.h>`, `<ATen/NamedTensorUtils.h>`, `<ATen/Config.h>`, `<ATen/native/mkldnn/Matmul.h>`, `<ATen/native/mkldnn/Linear.h>`, `<ATen/native/Resize.h>`, `<ATen/native/GroupedMMUtils.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `AT_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
