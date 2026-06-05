# Distance.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/Distance.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Distance. It also wires backend dispatch paths.
- **Purpose (CN)**: 实现或声明与 距离 相关的 ATen 原生逻辑。 它还负责连接不同后端的调度路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/core/Tensor.h>
0003: #include <ATen/core/grad_mode.h>
0004: #include <ATen/ExpandUtils.h>
0005: #include <ATen/NamedTensorUtils.h>
0006: #include <ATen/TensorOperators.h>
0007: #include <ATen/native/Distance.h>
0008: #include <c10/util/accumulate.h>
0009: 
0010: #ifndef AT_PER_OPERATOR_HEADERS
0011: #include <ATen/Functions.h>
0012: #include <ATen/NativeFunctions.h>
0013: #else
0014: #include <ATen/ops/_cdist_backward_native.h>
0015: #include <ATen/ops/_cdist_forward.h>
0016: #include <ATen/ops/_cdist_forward_native.h>
0017: #include <ATen/ops/_euclidean_dist.h>
0018: #include <ATen/ops/_euclidean_dist_native.h>
0019: #include <ATen/ops/_pdist_backward_native.h>
0020: #include <ATen/ops/_pdist_forward.h>
0021: #include <ATen/ops/_pdist_forward_native.h>
0022: #include <ATen/ops/cat.h>
0023: #include <ATen/ops/cdist_native.h>
0024: #include <ATen/ops/cosine_similarity_native.h>
0025: #include <ATen/ops/empty.h>
0026: #include <ATen/ops/empty_like.h>
0027: #include <ATen/ops/linalg_vector_norm.h>
0028: #include <ATen/ops/norm.h>
0029: #include <ATen/ops/ones_like.h>
0030: #include <ATen/ops/pairwise_distance_native.h>
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, macro-based glue.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、宏定义或宏调用。

### Lines 31-60 / 第 31-60 行
```cpp
0031: #include <ATen/ops/pdist_native.h>
0032: #include <ATen/ops/pow.h>
0033: #include <ATen/ops/result_type.h>
0034: #include <ATen/ops/scalar_tensor.h>
0035: #include <ATen/ops/sum.h>
0036: #include <ATen/ops/zeros.h>
0037: #include <ATen/ops/zeros_like.h>
0038: 
0039: #include <utility>
0040: #endif
0041: 
0042: namespace at::native {
0043: 
0044: DEFINE_DISPATCH(pdist_forward_stub);
0045: DEFINE_DISPATCH(pdist_backward_stub);
0046: DEFINE_DISPATCH(cdist_stub);
0047: DEFINE_DISPATCH(cdist_backward_stub);
0048: 
0049: Tensor pairwise_distance(const Tensor& x1, const Tensor& x2, double p, double eps, bool keepdim) {
0050:   // Since either x1 or x2 could be broadcasted
0051:   auto x1_dim = x1.dim();
0052:   auto x2_dim = x2.dim();
0053:   auto output_dim = x1_dim > x2_dim ? x1_dim : x2_dim;
0054:   auto innermost_dim = output_dim - 1;
0055:   return at::norm(x1 - x2 + eps, p, innermost_dim, keepdim);
0056: }
0057: 
0058: // This is to guarantee that the contiguous memory is passed to the backward pass
0059: Tensor pdist(const Tensor& self, const double p) {
0060:   TORCH_CHECK(self.dim() == 2,
```
- **EN**: Lines 31-60 mainly cover header inclusion, macro-based glue, state/variable declarations. Notable symbols: DEFINE_DISPATCH, pairwise_distance, dim, norm.
- **CN**: 第 31-60 行主要涉及头文件包含、宏定义或宏调用、变量/别名声明。 值得关注的符号包括：DEFINE_DISPATCH, pairwise_distance, dim, norm。

### Lines 61-90 / 第 61-90 行
```cpp
0061:       "pdist only supports 2D tensors, got: ", self.dim(), "D");
0062:   TORCH_CHECK(at::isFloatingType(self.scalar_type()), "pdist only supports floating-point dtypes");
0063:   TORCH_CHECK(p >= 0, "pdist only supports non-negative p values");
0064:   return at::_pdist_forward(self.contiguous(), p);
0065: }
0066: 
0067: Tensor _euclidean_dist(const Tensor& x1, const Tensor& x2) {
0068:   /** This function does the fist part of the euclidean distance calculation
0069:    * We divide it in two steps to simplify dealing with subgradients in the
0070:    * backward step */
0071:   Tensor x1_norm = x1.pow(2).sum(-1, true);
0072:   Tensor x1_pad = at::ones_like(x1_norm, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
0073:   Tensor x2_norm = x2.pow(2).sum(-1, true);
0074:   Tensor x2_pad = at::ones_like(x2_norm, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
0075:   Tensor x1_ = at::cat({x1.mul(-2), std::move(x1_norm), std::move(x1_pad)}, -1);
0076:   Tensor x2_ = at::cat({x2, std::move(x2_pad), std::move(x2_norm)}, -1);
0077:   Tensor result = x1_.matmul(x2_.mT());
0078:   result.clamp_min_(0).sqrt_();
0079:   return result;
0080: }
0081: 
0082: static Tensor cdist_impl(const Tensor& x1, const Tensor& x2, const double p, std::optional<int64_t> compute_mode) {
0083:   TORCH_CHECK(at::isFloatingType(x1.scalar_type()), "cdist only supports floating-point dtypes, X1 got: ", x1.scalar_type());
0084:   auto device1 = x1.device().type();
0085:   TORCH_CHECK(at::isFloatingType(x2.scalar_type()), "cdist only supports floating-point dtypes, X2 got: ", x2.scalar_type());
0086:   auto device2 = x2.device().type();
0087:   TORCH_CHECK(p >= 0, "cdist only supports non-negative p values");
0088:   TORCH_CHECK(device1 == device2, "X1 and X2 must have the same device type. X1: ", device1, " X2: ", device2);
0089:   // TODO: This is bad; this test should apply universally
0090:   TORCH_CHECK(!x1.is_cuda() || x1.get_device() == x2.get_device(), "device of X1 (", x1.get_device(), ") must match device of X2 (", x2.get_device(), ")");
```
- **EN**: Lines 61-90 mainly cover state/variable declarations, macro-based glue, comments/documentation. Notable symbols: dim, TORCH_CHECK, isFloatingType, scalar_type.
- **CN**: 第 61-90 行主要涉及变量/别名声明、宏定义或宏调用、注释或说明。 值得关注的符号包括：dim, TORCH_CHECK, isFloatingType, scalar_type。

### Lines 91-120 / 第 91-120 行
```cpp
0091:   SymInt c1 = x1.sym_size(-1);
0092:   SymInt c2 = x2.sym_size(-1);
0093:   // 0 - default value. If p = 2 and r1 > 25 or r2 > 25 (these values are based on performance metrics),
0094:   // it will try to compute distance using matrix multiplication approach
0095:   // 1 - force to use matrix multiplication for p = 2
0096:   // 2 - do not use matrix multiplication for p = 2
0097:   int64_t mode = compute_mode.value_or(0);
0098:   TORCH_CHECK(mode >= 0 && mode <= 2, "possible modes: 0, 1, 2, but was: ", mode);
0099: 
0100:   SymInt r1 = x1.sym_size(-2);
0101:   SymInt r2 = x2.sym_size(-2);
0102: 
0103:   // See Note [cdist relies on cdist_impl redispatching]
0104:   // Keep this condition in sync with the condition at the Note
0105:   if (!(p == 2 && (mode == 1 || (mode == 0 && (r1 > 25 || r2 > 25))))) {
0106:     TORCH_CHECK(device1 == kCPU || device1 == kCUDA || device1 == kXPU || device1 == kPrivateUse1, "cdist only supports CPU, XPU, CUDA and PrivateUse1 devices, X1 got: ", device1);
0107:     TORCH_CHECK(device2 == kCPU || device2 == kCUDA || device2 == kXPU || device2 == kPrivateUse1, "cdist only supports CPU, XPU, CUDA and PrivateUse1 devices, X2 got: ", device2);
0108:   }
0109: 
0110:   auto dim1 = x1.dim();
0111:   auto dim2 = x2.dim();
0112: 
0113:   //For batch calculation we expand all dimensions(except the last two) to one, with size that equals to product of them.
0114:   //The last two dimensions will stay the same
0115:   SymIntArrayRef batch_tensor1(x1.sym_sizes().data(), dim1 - 2);
0116:   SymIntArrayRef batch_tensor2(x2.sym_sizes().data(), dim2 - 2);
0117:   std::vector<SymInt> expand_batch_portion = infer_size_symint(batch_tensor1, batch_tensor2);
0118:   std::vector<SymInt> tensor1_expand_size(expand_batch_portion);
0119:   tensor1_expand_size.insert(tensor1_expand_size.end(), {r1, c1});
0120:   std::vector<SymInt> tensor2_expand_size(expand_batch_portion);
```
- **EN**: Lines 91-120 mainly cover state/variable declarations, comments/documentation, macro-based glue. Notable symbols: sym_size, value_or, TORCH_CHECK, dim.
- **CN**: 第 91-120 行主要涉及变量/别名声明、注释或说明、宏定义或宏调用。 值得关注的符号包括：sym_size, value_or, TORCH_CHECK, dim。

### Lines 121-150 / 第 121-150 行
```cpp
0121:   tensor2_expand_size.insert(tensor2_expand_size.end(), {r2, c2});
0122: 
0123:   const SymInt expand_batch_product = c10::multiply_integers(expand_batch_portion);
0124:   std::vector<SymInt> tensor1_view{expand_batch_product, r1, c1};
0125:   std::vector<SymInt> tensor2_view{expand_batch_product, r2, c2};
0126: 
0127:   Tensor tensor1_expanded = x1.expand_symint(tensor1_expand_size).contiguous().view_symint(tensor1_view);
0128:   Tensor tensor2_expanded = x2.expand_symint(tensor2_expand_size).contiguous().view_symint(tensor2_view);
0129: 
0130:   std::vector<SymInt> output_shape(std::move(expand_batch_portion));
0131:   output_shape.insert(output_shape.end(), {r1, r2});
0132: 
0133:   Tensor result;
0134:   if (r1 == 0 || r2 == 0 || expand_batch_product == 0) {
0135:     result = at::empty_symint(output_shape, x1.options());
0136:   } else if (c1 == 0) {
0137:     result = at::zeros_symint(output_shape, x1.options());
0138:   } else if (p == 2 && (mode == 1 || (mode == 0 && (r1 > 25 || r2 > 25)))) {
0139:     // See Note [cdist relies on cdist_impl redispatching]
0140:     // Keep the condition above in sync with the condition at the Note
0141:     Tensor dist = (expand_batch_product == 1) ? at::_euclidean_dist(x1, x2) :
0142:                   at::_euclidean_dist(tensor1_expanded, tensor2_expanded);
0143:     result = dist.view_symint(output_shape);
0144:   } else {
0145:     result = at::empty_symint(output_shape, x1.options());
0146:     cdist_stub(device1, result, tensor1_expanded, tensor2_expanded, p);
0147:   }
0148:   return result;
0149: }
0150: 
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, function signatures/definitions, comments/documentation. Notable symbols: insert, end, multiply_integers, expand_symint.
- **CN**: 第 121-150 行主要涉及变量/别名声明、函数签名或实现、注释或说明。 值得关注的符号包括：insert, end, multiply_integers, expand_symint。

### Lines 151-180 / 第 151-180 行
```cpp
0151: Tensor cdist(const Tensor& x1, const Tensor& x2, const double p, std::optional<int64_t> compute_mode) {
0152:   TORCH_CHECK(x1.dim() >= 2, "cdist only supports at least 2D tensors, X1 got: ", x1.dim(), "D");
0153:   TORCH_CHECK(x2.dim() >= 2, "cdist only supports at least 2D tensors, X2 got: ", x2.dim(), "D");
0154:   TORCH_CHECK(x1.sym_size(-1) == x2.sym_size(-1), "X1 and X2 must have the same number of columns. X1: ", x1.sym_size(-1), " X2: ", x2.sym_size(-1));
0155:   auto maybe_outnames = namedinference::compute_cdist_outnames(x1, x2);
0156:   auto result = [&]() {
0157:     NoNamesGuard guard;
0158:     SymInt r1 = x1.sym_size(-2);
0159:     SymInt r2 = x2.sym_size(-2);
0160:     // Special case for empty input: always call the version with explicit autograd to ensure the graph is properly connected
0161:     if (x1.sym_numel() == 0 || x2.sym_numel() == 0) {
0162:         return at::_cdist_forward(x1, x2, p, compute_mode);
0163:     }
0164:     int64_t mode = compute_mode.value_or(0);
0165:     // Note [cdist relies on cdist_impl redispatching]
0166:     // ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
0167:     // This is for pytorch to figure the backward pass itself
0168:     // when p=2.  Keep this condition in sync with the See Note reference
0169:     if (p == 2 && (mode == 1 || (mode == 0 && (r1 > 25 || r2 > 25)))) {
0170:         return cdist_impl(x1, x2, p, compute_mode);
0171:     } else {
0172:         return at::_cdist_forward(x1, x2, p, compute_mode);
0173:     }
0174:   }();
0175:   namedinference::propagate_names_if_nonempty(result, maybe_outnames);
0176:   return result;
0177: }
0178: 
0179: Tensor _cdist_forward(const Tensor& x1, const Tensor& x2, const double p, std::optional<int64_t> compute_mode) {
0180:   TORCH_CHECK(x1.dim() >= 2, "cdist only supports at least 2D tensors, X1 got: ", x1.dim(), "D");
```
- **EN**: Lines 151-180 mainly cover state/variable declarations, comments/documentation, function signatures/definitions. Notable symbols: cdist, TORCH_CHECK, dim, sym_size.
- **CN**: 第 151-180 行主要涉及变量/别名声明、注释或说明、函数签名或实现。 值得关注的符号包括：cdist, TORCH_CHECK, dim, sym_size。

### Lines 181-210 / 第 181-210 行
```cpp
0181:   TORCH_CHECK(x2.dim() >= 2, "cdist only supports at least 2D tensors, X2 got: ", x2.dim(), "D");
0182:   TORCH_CHECK(x1.size(-1) == x2.size(-1), "X1 and X2 must have the same number of columns. X1: ", x1.size(-1), " X2: ", x2.size(-1));
0183:   auto maybe_outnames = namedinference::compute_cdist_outnames(x1, x2);
0184:   auto result = [&]() {
0185:     NoNamesGuard guard;
0186:     return cdist_impl(x1, x2, p, compute_mode);
0187:   }();
0188:   namedinference::propagate_names_if_nonempty(result, maybe_outnames);
0189:   return result;
0190: }
0191: 
0192: Tensor _cdist_backward(const Tensor& _grad, const Tensor& _x1, const Tensor& _x2, const double p, const Tensor& _cdist) {
0193:   // Broadcasting might generate non-contiguous Tensors, so handle it before doing checks
0194:   int64_t c1 = _x1.size(-1);
0195:   int64_t c2 = _x2.size(-1);
0196:   int64_t r1 = _x1.size(-2);
0197:   int64_t r2 = _x2.size(-2);
0198:   auto dim1 = _x1.dim();
0199:   auto dim2 = _x2.dim();
0200:   IntArrayRef batch_tensor1(_x1.sizes().data(), dim1 - 2);
0201:   IntArrayRef batch_tensor2(_x2.sizes().data(), dim2 - 2);
0202:   std::vector<int64_t> expand_batch_portion = infer_size(batch_tensor1, batch_tensor2);
0203:   std::vector<int64_t> tensor1_expand_size(expand_batch_portion);
0204:   tensor1_expand_size.insert(tensor1_expand_size.end(), {r1, c1});
0205:   std::vector<int64_t> tensor2_expand_size(expand_batch_portion);
0206:   tensor2_expand_size.insert(tensor2_expand_size.end(), {r2, c2});
0207: 
0208:   // Compute the linearized batch size
0209:   const int64_t batch_product = c10::multiply_integers(expand_batch_portion);
0210: 
```
- **EN**: Lines 181-210 mainly cover state/variable declarations, macro-based glue, function signatures/definitions. Notable symbols: TORCH_CHECK, dim, size, compute_cdist_outnames.
- **CN**: 第 181-210 行主要涉及变量/别名声明、宏定义或宏调用、函数签名或实现。 值得关注的符号包括：TORCH_CHECK, dim, size, compute_cdist_outnames。

### Lines 211-240 / 第 211-240 行
```cpp
0211:   // Gracefully handle empty Tensors
0212:   if (r1 == 0 || r2 == 0 || c1 == 0 || batch_product == 0) {
0213:     return at::zeros_like(_x1, _x1.options());
0214:   }
0215: 
0216:   Tensor x1 = _x1;
0217:   if (tensor1_expand_size != x1.sizes()) {
0218:     x1 = x1.expand(tensor1_expand_size);
0219:   }
0220:   Tensor x2 = _x2;
0221:   if (tensor2_expand_size != x2.sizes()) {
0222:     x2 = x2.expand(tensor2_expand_size);
0223:   }
0224: 
0225:   x1 = x1.contiguous();
0226:   x2 = x2.contiguous();
0227:   auto cdist = _cdist.contiguous();
0228:   auto grad = _grad.contiguous();
0229:   int64_t n = x1.size(-2);
0230:   int64_t m = x1.size(-1);
0231:   auto device1 = x1.device().type();
0232:   TORCH_CHECK(device1 == kCPU || device1 == kCUDA || device1 == kXPU || device1 == kPrivateUse1, "_cdist_backward only supports CPU, XPU, CUDA and PrivateUse1 devices, X1 got: ", device1);
0233:   auto device2 = x2.device().type();
0234:   TORCH_CHECK(device2 == kCPU || device2 == kCUDA || device2 == kXPU || device2 == kPrivateUse1, "_cdist_backward only supports CPU, XPU, CUDA and PrivateUse1 devices, X2 got: ", device2);
0235: 
0236:   Tensor grad_x1 =
0237:       at::empty({batch_product, n, m}, x1.options(), LEGACY_CONTIGUOUS_MEMORY_FORMAT);
0238:   cdist_backward_stub(device1, grad_x1, grad, x1, x2, p, cdist);
0239: 
0240:   // Use x1.size() here and not the original size of _x1.size() as this gradient is not taking broadcasting into account
```
- **EN**: Lines 211-240 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: zeros_like, options, sizes, expand.
- **CN**: 第 211-240 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：zeros_like, options, sizes, expand。

### Lines 241-270 / 第 241-270 行
```cpp
0241:   // Broadcasting will be handled automatically by the autograd engine
0242:   return grad_x1.view(x1.sizes());
0243: }
0244: 
0245: Tensor _pdist_forward(const Tensor& self, const double p) {
0246:   TORCH_CHECK(self.is_contiguous(), "_pdist_forward requires contiguous input");
0247:   auto device = self.device().type();
0248:   TORCH_CHECK(device == kCPU || device == kCUDA || device == kXPU || device == kPrivateUse1, "_pdist_forward only supports CPU, XPU, CUDA and PrivateUse1 devices, got: ", device);
0249:   Tensor result = at::empty({0}, self.options(), LEGACY_CONTIGUOUS_MEMORY_FORMAT);
0250:   if (self.size(0) <= 1) {
0251:     result.resize_({0});
0252:   } else {
0253:     int64_t n = self.size(0);
0254:     int64_t c = n * (n - 1) / 2;
0255:     result.resize_({c});
0256:     if (self.size(1) == 0) {
0257:       result.fill_(0);
0258:     } else {
0259:       pdist_forward_stub(device, result, self, p);
0260:     }
0261:   }
0262:   return result;
0263: }
0264: 
0265: Tensor _pdist_backward(const Tensor& grad, const Tensor& self, const double p, const Tensor& pdist) {
0266:   TORCH_CHECK(self.is_contiguous(), "_pdist_backward requires self to be contiguous");
0267:   TORCH_CHECK(pdist.is_contiguous(), "_pdist_backward requires pdist to be contiguous");
0268:   auto device = self.device().type();
0269:   TORCH_CHECK(device == kCPU || device == kCUDA || device == kXPU || device == kPrivateUse1, "_pdist_backward only supports CPU, XPU, CUDA and PrivateUse1 devices, got: ", device);
0270:   Tensor result = at::empty_like(self, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
```
- **EN**: Lines 241-270 mainly cover state/variable declarations, macro-based glue, expressions/calls. Notable symbols: view, sizes, _pdist_forward, TORCH_CHECK.
- **CN**: 第 241-270 行主要涉及变量/别名声明、宏定义或宏调用、表达式或调用。 值得关注的符号包括：view, sizes, _pdist_forward, TORCH_CHECK。

### Lines 271-300 / 第 271-300 行
```cpp
0271:   pdist_backward_stub(device, result, grad, self, p, pdist);
0272:   return result;
0273: }
0274: 
0275: Tensor cosine_similarity(const Tensor& x1_, const Tensor& x2_, int64_t dim, double eps) {
0276:     /*
0277:    * cosine_similarity(x1, x2) = <x1, x2> / (||x1|| * ||x2||)
0278:    *
0279:    * The current implementation is an improvement over the previous version.
0280:    *
0281:    * Previous implementation:
0282:    * 1. Compute num = <x1, x2>,
0283:    * 2. Compute denom = ||x1|| * ||x2||,
0284:    * 3. Compute denom = max(denom, eps) to avoid division by zero,
0285:    * 4. Return num / denom.
0286:    *
0287:    * Previous implementation has the following issues:
0288:    * 1. Chance of losing precision in <x1, x2> when ||x1|| and ||x2|| are large.
0289:    * 2. Chance of losing precision in ||x1|| * ||x2|| when ||x1|| and ||x2|| are large.
0290:    * 3. Losing precision may cause |cosing_similarity(x1, x2)| > 1.0.
0291:    *
0292:    * Current implementation:
0293:    * 1. Compute x1_normalized = x1 / max(||x1||, eps),
0294:    *            x2_normalized = x2 / max(||x2||, eps),
0295:    * 2. Return <x1_normalized, x2_normalized>.
0296:    *
0297:    * The current implementation improves over the previous one by:
0298:    * 1. Making sure that <x1, x2> and ||x1|| * ||x2|| are not computed explicitly,
0299:    *    hence avoiding floating point overflows.
0300:    * 2. Both methods might have issues with computing ||x1|| and ||x2||, but for
```
- **EN**: Lines 271-300 mainly cover comments/documentation, state/variable declarations, return paths. Notable symbols: pdist_backward_stub, cosine_similarity, max, cosing_similarity.
- **CN**: 第 271-300 行主要涉及注释或说明、变量/别名声明、返回路径。 值得关注的符号包括：pdist_backward_stub, cosine_similarity, max, cosing_similarity。

### Lines 301-330 / 第 301-330 行
```cpp
0301:    *    the current method this is the only source of the floating point imprecision.
0302:    * 3. Makes sure |cosing_similarity(x1, x2)| <= 1.0.
0303:    *
0304:    */
0305: 
0306:   auto commonDtype = at::result_type(x1_, x2_);
0307:   TORCH_CHECK(at::isFloatingType(commonDtype), "expected common dtype to be floating point, yet common dtype is ", commonDtype);
0308:   TORCH_CHECK(eps >= 0, "eps must be non-negative, got: ", eps);
0309: 
0310:   // We accept integral types (and bools lol) but vector_norm does not
0311:   auto x1_is_int = c10::isIntegralType(x1_.scalar_type(), /*încludeBool=*/true);
0312:   auto x2_is_int = c10::isIntegralType(x2_.scalar_type(), /*încludeBool=*/true);
0313:   auto x1_t = x1_is_int ? x1_.to(commonDtype) : x1_;
0314:   auto x2_t = x2_is_int ? x2_.to(commonDtype) : x2_;
0315:   auto [x1, x2] = expand_outplace(x1_t, x2_t);
0316: 
0317: 
0318:   // We want to divide each tensor by its norm first, as it's more numerically stable.
0319:   // This keeps the result between -1.0 and 1.0
0320:   // We clone them, as we're going to modify them in-place
0321:   // This allows the gradients to propagate properly all the way to x1 and x2
0322:   auto x1_norm = at::linalg_vector_norm(*x1, 2, /*dim=*/dim, /*keepdim=*/true).clone();
0323:   auto x2_norm = at::linalg_vector_norm(*x2, 2, /*dim=*/dim, /*keepdim=*/true).clone();
0324: 
0325:   // Convert eps to a scalar tensor to ensure consistent CPU/CUDA behavior when eps overflows the dtype.
0326:   // Use float32 if commonDtype is a reduced floating type (float16/bfloat16), otherwise use commonDtype.
0327:   // This avoids CUDA errors when creating a scalar_tensor with reduced types if the eps value overflows,
0328:   // while CPU would convert to inf. The eps_tensor is then used to clamp the norms to prevent division by zero.
0329:   auto common_is_reduced = at::isReducedFloatingType(commonDtype);
0330:   auto eps_dtype = common_is_reduced ? at::kFloat : commonDtype;
```
- **EN**: Lines 301-330 mainly cover comments/documentation, state/variable declarations, macro-based glue. Notable symbols: cosing_similarity, result_type, TORCH_CHECK, isFloatingType.
- **CN**: 第 301-330 行主要涉及注释或说明、变量/别名声明、宏定义或宏调用。 值得关注的符号包括：cosing_similarity, result_type, TORCH_CHECK, isFloatingType。

### Lines 331-342 / 第 331-342 行
```cpp
0331:   auto eps_tensor = at::scalar_tensor(eps, at::TensorOptions().dtype(eps_dtype).device(x1_norm.device()));
0332: 
0333:   {
0334:     at::NoGradGuard guard;
0335:     x1_norm.clamp_min_(eps_tensor);
0336:     x2_norm.clamp_min_(eps_tensor);
0337:   }
0338: 
0339:   return ((*x1 / x1_norm) * (*x2 / x2_norm)).sum(dim);
0340: }
0341: 
0342: }  // namespace at::native
```
- **EN**: Lines 331-342 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: scalar_tensor, TensorOptions, dtype, device.
- **CN**: 第 331-342 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：scalar_tensor, TensorOptions, dtype, device。

## Key Concepts / 关键概念
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Dispatch stub definition  
  **CN**: 调度桩定义
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现
- **EN**: Native operator implementation path  
  **CN**: 原生算子实现路径

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/core/grad_mode.h>`, `<ATen/ExpandUtils.h>`, `<ATen/NamedTensorUtils.h>`, `<ATen/TensorOperators.h>`, `<ATen/native/Distance.h>`, `<c10/util/accumulate.h>`, `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/_cdist_backward_native.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `DEFINE_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
