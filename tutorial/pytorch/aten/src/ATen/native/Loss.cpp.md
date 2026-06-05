# Loss.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/Loss.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Loss. It also wires backend dispatch paths.
- **Purpose (CN)**: 实现或声明与 损失函数 相关的 ATen 原生逻辑。 它还负责连接不同后端的调度路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/core/Tensor.h>
0003: #include <ATen/core/Reduction.h>
0004: #include <ATen/Dispatch.h>
0005: #include <ATen/TensorIterator.h>
0006: #include <ATen/TensorMeta.h>
0007: #include <ATen/TensorOperators.h>
0008: #include <ATen/native/BinaryOps.h>
0009: #include <ATen/native/PointwiseOps.h>
0010: #include <ATen/native/cpu/Loops.h>
0011: #include <c10/util/Exception.h>
0012: #include <ATen/TensorSubclassLikeUtils.h>
0013: 
0014: #ifndef AT_PER_OPERATOR_HEADERS
0015: #include <ATen/Functions.h>
0016: #include <ATen/NativeFunctions.h>
0017: #else
0018: #include <ATen/ops/binary_cross_entropy_backward_native.h>
0019: #include <ATen/ops/binary_cross_entropy_native.h>
0020: #include <ATen/ops/binary_cross_entropy_with_logits_native.h>
0021: #include <ATen/ops/clamp_min.h>
0022: #include <ATen/ops/cosine_embedding_loss_native.h>
0023: #include <ATen/ops/empty.h>
0024: #include <ATen/ops/empty_like.h>
0025: #include <ATen/ops/exp.h>
0026: #include <ATen/ops/hinge_embedding_loss_native.h>
0027: #include <ATen/ops/huber_loss_backward.h>
0028: #include <ATen/ops/huber_loss_backward_native.h>
0029: #include <ATen/ops/huber_loss_native.h>
0030: #include <ATen/ops/kl_div_native.h>
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, macro-based glue.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、宏定义或宏调用。

### Lines 31-60 / 第 31-60 行
```cpp
0031: #include <ATen/ops/l1_loss_native.h>
0032: #include <ATen/ops/log.h>
0033: #include <ATen/ops/log_sigmoid.h>
0034: #include <ATen/ops/margin_ranking_loss_native.h>
0035: #include <ATen/ops/mean.h>
0036: #include <ATen/ops/min.h>
0037: #include <ATen/ops/mse_loss_backward.h>
0038: #include <ATen/ops/mse_loss_backward_native.h>
0039: #include <ATen/ops/mse_loss_meta.h>
0040: #include <ATen/ops/mse_loss_native.h>
0041: #include <ATen/ops/mul.h>
0042: #include <ATen/ops/neg.h>
0043: #include <ATen/ops/pairwise_distance.h>
0044: #include <ATen/ops/poisson_nll_loss_native.h>
0045: #include <ATen/ops/smooth_l1_loss_backward.h>
0046: #include <ATen/ops/smooth_l1_loss_backward_native.h>
0047: #include <ATen/ops/smooth_l1_loss_meta.h>
0048: #include <ATen/ops/smooth_l1_loss_native.h>
0049: #include <ATen/ops/soft_margin_loss.h>
0050: #include <ATen/ops/soft_margin_loss_backward.h>
0051: #include <ATen/ops/soft_margin_loss_backward_native.h>
0052: #include <ATen/ops/soft_margin_loss_native.h>
0053: #include <ATen/ops/squeeze.h>
0054: #include <ATen/ops/sum.h>
0055: #include <ATen/ops/triplet_margin_loss_native.h>
0056: #include <ATen/ops/where.h>
0057: #include <ATen/ops/xlogy.h>
0058: #include <ATen/ops/zeros_like.h>
0059: #endif
0060: 
```
- **EN**: Lines 31-60 mainly cover header inclusion, conditional compilation.
- **CN**: 第 31-60 行主要涉及头文件包含、预处理条件。

### Lines 61-90 / 第 61-90 行
```cpp
0061: constexpr float EPSILON = 1e-12;
0062: 
0063: namespace {
0064:   inline at::Tensor apply_loss_reduction(const at::Tensor& unreduced, int64_t reduction) {
0065:     if (reduction == at::Reduction::Mean) {
0066:       return unreduced.mean();
0067:     } else if (reduction == at::Reduction::Sum) {
0068:       return unreduced.sum();
0069:     }
0070:     return unreduced;
0071:   }
0072: }
0073: 
0074: namespace at::meta {
0075: 
0076: TORCH_META_FUNC(smooth_l1_loss)
0077: (const Tensor& input, const Tensor& target, const int64_t reduction, double beta) {
0078:   TORCH_CHECK(beta >= 0, "smooth_l1_loss does not support negative values for beta.")
0079:   // TODO: Reduce this extra TensorIterator construction for Reduction::Mean & Sum.
0080:   // We do another TensorIterator construction in the IMPL for the two cases.
0081:   build_borrowing_binary_op(maybe_get_output(), input, target);
0082:   if (reduction == Reduction::None) {
0083:     return;
0084:   }
0085: 
0086:   TORCH_INTERNAL_ASSERT(reduction == Reduction::Mean || reduction == Reduction::Sum);
0087:   maybe_get_output().resize_({});
0088: }
0089: 
0090: TORCH_META_FUNC(mse_loss)
```
- **EN**: Lines 61-90 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: apply_loss_reduction, mean, sum, TORCH_META_FUNC.
- **CN**: 第 61-90 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：apply_loss_reduction, mean, sum, TORCH_META_FUNC。

### Lines 91-120 / 第 91-120 行
```cpp
0091: (const Tensor& input, const Tensor& target, const int64_t reduction) {
0092:   build_borrowing_binary_op(maybe_get_output(), input, target);
0093:   if (reduction == Reduction::None) {
0094:     return;
0095:   }
0096: 
0097:   TORCH_INTERNAL_ASSERT(reduction == Reduction::Mean || reduction == Reduction::Sum);
0098:   maybe_get_output().resize_({});
0099: }
0100: 
0101: } // namespace at::meta
0102: 
0103: namespace at::native {
0104: 
0105: DEFINE_DISPATCH(smooth_l1_stub);
0106: DEFINE_DISPATCH(smooth_l1_backward_stub);
0107: DEFINE_DISPATCH(huber_stub);
0108: DEFINE_DISPATCH(huber_backward_stub);
0109: DEFINE_DISPATCH(mse_stub);
0110: DEFINE_DISPATCH(mse_backward_stub);
0111: 
0112: TORCH_IMPL_FUNC(smooth_l1_loss_out)
0113: (const Tensor& input, const Tensor& target, int64_t reduction, double beta, const Tensor& result) {
0114:   if (reduction != Reduction::None) {
0115:     Tensor loss;
0116:     auto iter = TensorIterator::borrowing_binary_op(loss, input, target);
0117:     smooth_l1_stub(iter.device_type(), iter, beta);
0118:     if (reduction == Reduction::Mean) {
0119:       at::mean_out(const_cast<Tensor&>(result), iter.output(), IntArrayRef{});
0120:     } else {
```
- **EN**: Lines 91-120 mainly cover macro-based glue, state/variable declarations, function signatures/definitions. Notable symbols: build_borrowing_binary_op, maybe_get_output, TORCH_INTERNAL_ASSERT, resize_.
- **CN**: 第 91-120 行主要涉及宏定义或宏调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：build_borrowing_binary_op, maybe_get_output, TORCH_INTERNAL_ASSERT, resize_。

### Lines 121-150 / 第 121-150 行
```cpp
0121:       at::sum_out(const_cast<Tensor&>(result), iter.output(), IntArrayRef{});
0122:     }
0123:   } else {
0124:     smooth_l1_stub(device_type(), *this, beta);
0125:   }
0126: }
0127: 
0128: TORCH_IMPL_FUNC(mse_loss_out)
0129: (const Tensor& input, const Tensor& target, int64_t reduction, const Tensor& result) {
0130:   TORCH_CHECK(input.device() == target.device(),
0131:       "Expected all tensors to be on the same device, but found at least two devices, ",
0132:       input.device(), " and ", target.device(), "!");
0133:   if (reduction != Reduction::None) {
0134:     Tensor loss;
0135:     auto iter = TensorIterator::borrowing_binary_op(loss, input, target);
0136:     mse_stub(iter.device_type(), iter);
0137:     if (reduction == Reduction::Mean) {
0138:       at::mean_out(const_cast<Tensor&>(result), iter.output(), IntArrayRef{});
0139:     } else {
0140:       at::sum_out(const_cast<Tensor&>(result), iter.output(), IntArrayRef{});
0141:     }
0142:   } else {
0143:     mse_stub(device_type(), *this);
0144:   }
0145: }
0146: 
0147: Tensor cosine_embedding_loss(const Tensor& input1, const Tensor& input2, const Tensor& target, double margin, int64_t reduction) {
0148:   auto targ_dim = target.dim();
0149:   TORCH_CHECK(
0150:       targ_dim == 1 || targ_dim == 0,
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: sum_out, output, smooth_l1_stub, device_type.
- **CN**: 第 121-150 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：sum_out, output, smooth_l1_stub, device_type。

### Lines 151-180 / 第 151-180 行
```cpp
0151:       "0D or 1D target tensor expected, multi-target not supported");
0152:   if (targ_dim == 1) {
0153:     TORCH_CHECK(
0154:         input1.dim() == 2 && input2.dim() == 2,
0155:         "1D target tensor expects 2D input tensors, but found inputs with sizes ",
0156:         input1.sizes(),
0157:         " and ",
0158:         input2.sizes(),
0159:         ".");
0160:   } else {
0161:     TORCH_CHECK(
0162:         input1.dim() == 1 && input2.dim() == 1,
0163:         "0D target tensor expects 1D input tensors, but found inputs with sizes ",
0164:         input1.sizes(),
0165:         " and ",
0166:         input2.sizes(),
0167:         ".");
0168:   }
0169: 
0170:   auto prod_sum = (input1 * input2).sum(targ_dim);
0171:   auto mag_square1 = (input1 * input1).sum(targ_dim) + EPSILON;
0172:   auto mag_square2 = (input2 * input2).sum(targ_dim) + EPSILON;
0173:   auto denom = (mag_square1 * mag_square2).sqrt_();
0174:   auto cos = prod_sum / denom;
0175: 
0176:   auto zeros = at::zeros_like(cos, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
0177:   auto pos = 1 - cos;
0178:   auto neg = (cos - margin).clamp_min_(0);
0179:   auto output_pos = at::where(target == 1, pos, zeros);
0180:   auto output_neg = at::where(target == -1, neg, zeros);
```
- **EN**: Lines 151-180 mainly cover state/variable declarations, function signatures/definitions, expressions/calls. Notable symbols: TORCH_CHECK, dim, sizes, sum.
- **CN**: 第 151-180 行主要涉及变量/别名声明、函数签名或实现、表达式或调用。 值得关注的符号包括：TORCH_CHECK, dim, sizes, sum。

### Lines 181-210 / 第 181-210 行
```cpp
0181:   auto output = output_pos + output_neg;
0182:   return apply_loss_reduction(output, reduction);
0183: }
0184: 
0185: Tensor hinge_embedding_loss(const Tensor& self, const Tensor& target, double margin, int64_t reduction) {
0186:   auto zeros = at::zeros_like(self, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
0187:   auto margin_diff = (margin - self);
0188:   // For Composite Compliance,
0189:   // In Forward AD, if `margin_diff` is a CCT but its tangent isn't,
0190:   // using inplace clamp_min doesn't work because we end up writing
0191:   // the CCT in-place to the tangent
0192:   auto margin_clamp = (margin_diff._fw_grad(/*level*/ 0).defined() &&
0193:                        isTensorSubclassLike(margin_diff))
0194:       ? margin_diff.clamp_min(0)
0195:       : margin_diff.clamp_min_(0);
0196:   auto output_margin = at::where(target != 1, margin_clamp, zeros);
0197:   auto output_self = at::where(target != -1, self, zeros);
0198:   auto output = output_margin + output_self;
0199:   return apply_loss_reduction(output, reduction);
0200: }
0201: 
0202: Tensor triplet_margin_loss(const Tensor& anchor, const Tensor& positive, const Tensor& negative, double margin,
0203:                            double p, double eps, bool swap, int64_t reduction) {
0204:   auto a_dim = anchor.dim();
0205:   auto p_dim = positive.dim();
0206:   auto n_dim = negative.dim();
0207:   TORCH_CHECK(
0208:       a_dim == p_dim && p_dim == n_dim,
0209:       "The anchor, positive, and negative tensors are expected to have "
0210:       "the same number of dimensions, but got: anchor ", a_dim, "D, "
```
- **EN**: Lines 181-210 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: apply_loss_reduction, hinge_embedding_loss, zeros_like, _fw_grad.
- **CN**: 第 181-210 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：apply_loss_reduction, hinge_embedding_loss, zeros_like, _fw_grad。

### Lines 211-240 / 第 211-240 行
```cpp
0211:       "positive ", p_dim, "D, and negative ", n_dim, "D inputs")
0212: 
0213:   auto dist_pos = at::pairwise_distance(anchor, positive, p, eps);
0214:   auto dist_neg = at::pairwise_distance(anchor, negative, p, eps);
0215:   // The distance swap is described in the paper "Learning shallow
0216:   // convolutional feature descriptors with triplet losses" by V. Balntas, E.
0217:   // Riba et al.  If True, and if the positive example is closer to the
0218:   // negative example than the anchor is, swaps the positive example and the
0219:   // anchor in the loss computation.
0220:   if (swap) {
0221:     auto dist_swap = at::pairwise_distance(positive, negative, p, eps);
0222:     dist_neg = at::min(dist_neg, dist_swap);
0223:   }
0224:   auto output = at::clamp_min(margin + dist_pos - dist_neg, 0);
0225:   return apply_loss_reduction(output, reduction);
0226: }
0227: 
0228: Tensor margin_ranking_loss(const Tensor& input1, const Tensor& input2, const Tensor& target, double margin, int64_t reduction) {
0229:   auto unclamped_output = (-target * (input1 - input2) + margin);
0230:   // For Composite Compliance,
0231:   // In Forward AD, if `margin_diff` is a CCT but its tangent isn't,
0232:   // using inplace clamp_min doesn't work because we end up writing
0233:   // the CCT in-place to the tangent
0234:   auto output = (unclamped_output._fw_grad(/*level*/ 0).defined() &&
0235:                  isTensorSubclassLike(unclamped_output))
0236:       ? unclamped_output.clamp_min(0)
0237:       : unclamped_output.clamp_min_(0);
0238:   return apply_loss_reduction(output, reduction);
0239: }
0240: 
```
- **EN**: Lines 211-240 mainly cover comments/documentation, state/variable declarations, expressions/calls. Notable symbols: pairwise_distance, min, clamp_min, apply_loss_reduction.
- **CN**: 第 211-240 行主要涉及注释或说明、变量/别名声明、表达式或调用。 值得关注的符号包括：pairwise_distance, min, clamp_min, apply_loss_reduction。

### Lines 241-270 / 第 241-270 行
```cpp
0241: Tensor kl_div(const Tensor& input, const Tensor& target, int64_t reduction, bool log_target) {
0242:   TORCH_CHECK(!input.is_complex() && !target.is_complex(),
0243:               "kl_div: Complex inputs not supported.");
0244:   TORCH_CHECK(!at::isIntegralType(input.scalar_type(), /*include_bool*/true) &&
0245:               !at::isIntegralType(target.scalar_type(), /*include_bool*/true),
0246:               "kl_div: Integral inputs not supported.");
0247:   Tensor output;
0248:   if (log_target) {
0249:     output = at::exp(target) * (target - input);
0250:   } else {
0251:     output = at::xlogy(target, target) - target * input;
0252:   }
0253:   return apply_loss_reduction(output, reduction);
0254: }
0255: 
0256: Tensor binary_cross_entropy_cpu(const Tensor& input, const Tensor& target, const std::optional<Tensor>& weight_opt, int64_t reduction) {
0257:     Tensor loss = at::empty_like(input);
0258:     return at::native::binary_cross_entropy_out_cpu(
0259:         input, target, weight_opt, reduction, loss);
0260: }
0261: 
0262: Tensor& binary_cross_entropy_out_cpu(const Tensor& input, const Tensor& target, const std::optional<Tensor>& weight_opt, int64_t reduction, Tensor& loss) {
0263:     Tensor loss_squeezed = at::squeeze(loss);
0264: 
0265:     auto iter = TensorIteratorConfig()
0266:       .add_output(loss_squeezed)
0267:       .add_owned_const_input(at::squeeze(input))
0268:       .add_owned_const_input(at::squeeze(target))
0269:       .build();
0270: 
```
- **EN**: Lines 241-270 mainly cover function signatures/definitions, state/variable declarations, expressions/calls. Notable symbols: kl_div, TORCH_CHECK, is_complex, isIntegralType.
- **CN**: 第 241-270 行主要涉及函数签名或实现、变量/别名声明、表达式或调用。 值得关注的符号包括：kl_div, TORCH_CHECK, is_complex, isIntegralType。

### Lines 271-300 / 第 271-300 行
```cpp
0271:     AT_DISPATCH_FLOATING_TYPES_AND2(
0272:         ScalarType::Half,
0273:         ScalarType::BFloat16,
0274:         loss.scalar_type(),
0275:         "binary_cross_entropy",
0276:         [&] {
0277:           at::native::cpu_kernel(
0278:               iter, [](scalar_t input_val, scalar_t target_val) {
0279:                 TORCH_CHECK(
0280:                     (input_val >= 0) && (input_val <= 1),
0281:                     "all elements of input should be between 0 and 1");
0282:                 TORCH_CHECK(
0283:                     (target_val >= 0) && (target_val <= 1),
0284:                     "all elements of target should be between 0 and 1");
0285: 
0286:                 // Binary cross entropy tensor is defined by the equation:
0287:                 // L = -w (y ln(x) + (1-y) ln(1-x))
0288:                 return (target_val - scalar_t(1)) *
0289:                     std::max(scalar_t(std::log1p(-input_val)), scalar_t(-100)) -
0290:                     target_val *
0291:                     std::max(scalar_t(std::log(input_val)), scalar_t(-100));
0292:               });
0293:         });
0294: 
0295:     if (weight_opt.has_value() && weight_opt->defined()) {
0296:         loss.mul_(*weight_opt);
0297:     }
0298:     if (reduction != at::Reduction::None) {
0299:         Tensor loss_reduced = apply_loss_reduction(loss, reduction);
0300:         loss.resize_as_(loss_reduced).copy_(loss_reduced);
```
- **EN**: Lines 271-300 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: AT_DISPATCH_FLOATING_TYPES_AND2, scalar_type, cpu_kernel, TORCH_CHECK.
- **CN**: 第 271-300 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：AT_DISPATCH_FLOATING_TYPES_AND2, scalar_type, cpu_kernel, TORCH_CHECK。

### Lines 301-330 / 第 301-330 行
```cpp
0301:     }
0302:     return loss;
0303: }
0304: 
0305: Tensor binary_cross_entropy_backward_cpu(const Tensor& grad, const Tensor& input, const Tensor& target, const std::optional<Tensor>& weight_opt, int64_t reduction) {
0306:     Tensor grad_input = at::empty_like(input);
0307:     return at::native::binary_cross_entropy_backward_out_cpu(
0308:         grad, input, target, weight_opt, reduction, grad_input);
0309: }
0310: 
0311: Tensor& binary_cross_entropy_backward_out_cpu(const Tensor& grad, const Tensor& input, const Tensor& target, const std::optional<Tensor>& weight_opt, int64_t reduction, Tensor& grad_input) {
0312:     Tensor grad_input_squeezed = at::squeeze(grad_input);
0313: 
0314:     auto iter = TensorIteratorConfig()
0315:       .add_output(grad_input_squeezed)
0316:       .add_owned_const_input(at::squeeze(grad))
0317:       .add_owned_const_input(at::squeeze(input))
0318:       .add_owned_const_input(at::squeeze(target))
0319:       .build();
0320: 
0321:     AT_DISPATCH_FLOATING_TYPES_AND2(
0322:         ScalarType::Half,
0323:         ScalarType::BFloat16,
0324:         grad_input.scalar_type(),
0325:         "binary_cross_entropy_backward",
0326:         [&] {
0327:           at::native::cpu_kernel(
0328:               iter,
0329:               [](scalar_t grad_val, scalar_t input_val, scalar_t target_val) {
0330:                 // The gradient is the partial derivative of BCELoss
```
- **EN**: Lines 301-330 mainly cover function signatures/definitions, expressions/calls, state/variable declarations. Notable symbols: binary_cross_entropy_backward_cpu, empty_like, binary_cross_entropy_backward_out_cpu, squeeze.
- **CN**: 第 301-330 行主要涉及函数签名或实现、表达式或调用、变量/别名声明。 值得关注的符号包括：binary_cross_entropy_backward_cpu, empty_like, binary_cross_entropy_backward_out_cpu, squeeze。

### Lines 331-360 / 第 331-360 行
```cpp
0331:                 // with respect to x
0332:                 // d(L)/d(x) = -w (y - x) / (x - x^2)
0333:                 return grad_val * (input_val - target_val) /
0334:                     (scalar_t(std::max(
0335:                         (scalar_t(1) - input_val) * input_val,
0336:                         scalar_t(EPSILON))));
0337:               });
0338:         });
0339: 
0340:     if (weight_opt.has_value() && weight_opt->defined()) {
0341:         grad_input.mul_(*weight_opt);
0342:     }
0343:     if (reduction == at::Reduction::Mean) {
0344:         grad_input.div_(input.numel());
0345:     }
0346:     return grad_input;
0347: }
0348: 
0349: Tensor binary_cross_entropy_with_logits(const Tensor& input, const Tensor& target, const std::optional<Tensor>& weight_opt, const std::optional<Tensor>& pos_weight_opt, int64_t reduction) {
0350:   auto log_sigmoid_input = at::log_sigmoid(input);
0351:   if (pos_weight_opt.has_value() && pos_weight_opt->defined()) {
0352:       // pos_weight need to be broadcasted, thus mul(target) is not inplace.
0353:       auto log_weight = (*pos_weight_opt- 1).mul(target).add_(1);
0354:       log_sigmoid_input.mul_(log_weight);
0355:   }
0356: 
0357:   Tensor loss = (1 - target).mul_(input).sub_(log_sigmoid_input);
0358: 
0359:   if (weight_opt.has_value() && weight_opt->defined()) {
0360:       loss.mul_(*weight_opt);
```
- **EN**: Lines 331-360 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: d, w, scalar_t, max.
- **CN**: 第 331-360 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：d, w, scalar_t, max。

### Lines 361-390 / 第 361-390 行
```cpp
0361:   }
0362: 
0363:   return apply_loss_reduction(loss, reduction);
0364: }
0365: 
0366: Tensor poisson_nll_loss(const Tensor& input, const Tensor& target, const bool log_input, const bool full, const double eps, const int64_t reduction)
0367: {
0368:     Tensor loss;
0369:     if (log_input) {
0370:         loss = at::exp(input) - target * input;
0371:     } else {
0372:         loss = input - target * at::log(input + eps);
0373:     }
0374: 
0375:     if (full) {
0376:         auto stirling_term = target * at::log(target) - target + 0.5 * at::log(2 * c10::pi<double> * target);
0377:         loss += stirling_term.masked_fill(target <= 1, 0);
0378:     }
0379: 
0380:     return apply_loss_reduction(loss, reduction);
0381: }
0382: 
0383: Tensor& soft_margin_loss_backward_out(const Tensor& grad_output, const Tensor& input, const Tensor& target, int64_t reduction, Tensor& grad_input) {
0384:   auto norm = reduction == Reduction::Mean ? 1. / input.numel() : 1.;
0385:   auto z = at::exp(-target * input);
0386:   // inplace version of: grad_input = -norm * target * z / (1. + z) * grad_output;
0387:   at::mul_out(grad_input, target, z).mul_(-norm);
0388:   z.add_(1);
0389:   grad_input.div_(z).mul_(grad_output);
0390:   return grad_input;
```
- **EN**: Lines 361-390 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: apply_loss_reduction, poisson_nll_loss, exp, log.
- **CN**: 第 361-390 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：apply_loss_reduction, poisson_nll_loss, exp, log。

### Lines 391-420 / 第 391-420 行
```cpp
0391: }
0392: 
0393: Tensor soft_margin_loss_backward(const Tensor& grad_output, const Tensor& input, const Tensor& target, int64_t reduction) {
0394:   auto grad_input = at::empty({0}, input.options());
0395:   at::soft_margin_loss_backward_out(grad_input, grad_output, input, target, reduction);
0396:   return grad_input;
0397: }
0398: 
0399: Tensor& soft_margin_loss_out(const Tensor& input,
0400:     const Tensor& target,
0401:     int64_t reduction,
0402:     Tensor& output) {
0403:   // compute inplace variant of: output = at::log1p(at::exp(-input * target));
0404:   at::neg_out(output, input).mul_(target).exp_().log1p_();
0405:   if (reduction != Reduction::None) {
0406:     auto tmp = apply_loss_reduction(output, reduction);
0407:     output.resize_({});
0408:     output.copy_(tmp);
0409:   }
0410:   return output;
0411: }
0412: 
0413: Tensor soft_margin_loss(
0414:     const Tensor& input,
0415:     const Tensor& target,
0416:     int64_t reduction) {
0417:   auto output = at::empty({0}, input.options());
0418:   at::soft_margin_loss_out(output, input, target, reduction);
0419:   return output;
0420: }
```
- **EN**: Lines 391-420 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: soft_margin_loss_backward, empty, options, soft_margin_loss_backward_out.
- **CN**: 第 391-420 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：soft_margin_loss_backward, empty, options, soft_margin_loss_backward_out。

### Lines 421-450 / 第 421-450 行
```cpp
0421: 
0422: Tensor& smooth_l1_loss_backward_out(const Tensor& grad_output, const Tensor& input, const Tensor& target, int64_t reduction, double beta, Tensor& grad_input) {
0423:   auto norm = reduction == Reduction::Mean ? 1. / input.numel() : 1.;
0424:   auto iter = at::TensorIteratorConfig()
0425:     .add_output(grad_input)
0426:     .add_const_input(input)
0427:     .add_const_input(target)
0428:     .add_const_input(grad_output)
0429:     .promote_inputs_to_common_dtype(true)
0430:     .cast_common_dtype_to_outputs(true)
0431:     .enforce_safe_casting_to_output(true)
0432:     .build();
0433:   smooth_l1_backward_stub(iter.device_type(), iter, norm, beta);
0434:   return grad_input;
0435: }
0436: 
0437: Tensor smooth_l1_loss_backward(const Tensor& grad_output, const Tensor& input, const Tensor& target, int64_t reduction, double beta) {
0438:   auto grad_input = at::zeros_like(input, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
0439:   return at::smooth_l1_loss_backward_out(grad_input, grad_output, input, target, reduction, beta);
0440: }
0441: 
0442: Tensor huber_loss(const Tensor& input, const Tensor& target, int64_t reduction, double delta) {
0443:   TORCH_CHECK(delta > 0, "huber_loss does not support non-positive values for delta.")
0444:   Tensor loss = at::empty_like(input);
0445:   auto iter = TensorIterator::borrowing_binary_op(loss, input, target);
0446:   huber_stub(iter.device_type(), iter, delta);
0447:   return apply_loss_reduction(loss, reduction);
0448: }
0449: 
0450: Tensor& huber_loss_out(const Tensor& input, const Tensor& target, int64_t reduction, double delta, Tensor& result) {
```
- **EN**: Lines 421-450 mainly cover function signatures/definitions, state/variable declarations, return paths. Notable symbols: smooth_l1_loss_backward_out, numel, TensorIteratorConfig, add_output.
- **CN**: 第 421-450 行主要涉及函数签名或实现、变量/别名声明、返回路径。 值得关注的符号包括：smooth_l1_loss_backward_out, numel, TensorIteratorConfig, add_output。

### Lines 451-480 / 第 451-480 行
```cpp
0451:   TORCH_CHECK(delta > 0, "huber_loss does not support non-positive values for delta.")
0452:   auto iter = TensorIterator::borrowing_binary_op(result, input, target);
0453:   huber_stub(iter.device_type(), iter, delta);
0454:   if (reduction != Reduction::None) {
0455:     auto reduced = apply_loss_reduction(result, reduction);
0456:     result.resize_({});
0457:     result.copy_(reduced);
0458:   }
0459:   return result;
0460: }
0461: 
0462: Tensor huber_loss_backward(const Tensor& grad_output, const Tensor& input, const Tensor& target, int64_t reduction, double delta) {
0463:   auto grad_input = at::zeros_like(input, MemoryFormat::Contiguous);
0464:   return at::huber_loss_backward_out(grad_input, grad_output, input, target, reduction, delta);
0465: }
0466: 
0467: Tensor& huber_loss_backward_out(const Tensor& grad_output, const Tensor& input, const Tensor& target, int64_t reduction, double delta, Tensor& grad_input) {
0468:   auto norm = (reduction == Reduction::Mean) ? (1. / input.numel()) : 1.;
0469:   auto iter = at::TensorIteratorConfig()
0470:     .add_output(grad_input)
0471:     .add_const_input(input)
0472:     .add_const_input(target)
0473:     .add_const_input(grad_output)
0474:     .build();
0475:   huber_backward_stub(iter.device_type(), iter, norm, delta);
0476:   return grad_input;
0477: }
0478: 
0479: Tensor mse_loss_backward(const Tensor& grad_output, const Tensor& input, const Tensor& target, int64_t reduction) {
0480:   Tensor grad_input = at::zeros_like(input, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
```
- **EN**: Lines 451-480 mainly cover state/variable declarations, function signatures/definitions, expressions/calls. Notable symbols: TORCH_CHECK, borrowing_binary_op, huber_stub, device_type.
- **CN**: 第 451-480 行主要涉及变量/别名声明、函数签名或实现、表达式或调用。 值得关注的符号包括：TORCH_CHECK, borrowing_binary_op, huber_stub, device_type。

### Lines 481-503 / 第 481-503 行
```cpp
0481:   return at::mse_loss_backward_out(grad_input, grad_output, input, target, reduction);
0482: }
0483: 
0484: Tensor& mse_loss_backward_out(const Tensor& grad_output,
0485:     const Tensor& input, const Tensor& target, int64_t reduction, Tensor& grad_input) {
0486:   auto norm = reduction == Reduction::Mean ? 2. / input.numel() : 2.;
0487:   auto iter = at::TensorIteratorConfig()
0488:     .add_output(grad_input)
0489:     .add_const_input(input)
0490:     .add_const_input(target)
0491:     .add_const_input(grad_output)
0492:     .promote_inputs_to_common_dtype(true)
0493:     .cast_common_dtype_to_outputs(true)
0494:     .enforce_safe_casting_to_output(true)
0495:     .build();
0496:   mse_backward_stub(iter.device_type(), iter, norm);
0497:   return grad_input;
0498: }
0499: 
0500: Tensor l1_loss(const Tensor& input, const Tensor& target, int64_t reduction) {
0501:   return apply_loss_reduction((input - target).abs(), reduction);
0502: }
0503: }  // namespace at::native
```
- **EN**: Lines 481-503 mainly cover function signatures/definitions, expressions/calls, return paths. Notable symbols: mse_loss_backward_out, numel, TensorIteratorConfig, add_output.
- **CN**: 第 481-503 行主要涉及函数签名或实现、表达式或调用、返回路径。 值得关注的符号包括：mse_loss_backward_out, numel, TensorIteratorConfig, add_output。

## Key Concepts / 关键概念
- **EN**: TensorIterator-driven traversal  
  **CN**: 基于 TensorIterator 的遍历
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Dispatch stub definition  
  **CN**: 调度桩定义
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/core/Reduction.h>`, `<ATen/Dispatch.h>`, `<ATen/TensorIterator.h>`, `<ATen/TensorMeta.h>`, `<ATen/TensorOperators.h>`, `<ATen/native/BinaryOps.h>`, `<ATen/native/PointwiseOps.h>`, `<ATen/native/cpu/Loops.h>`, `<c10/util/Exception.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `DEFINE_DISPATCH`, `AT_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
