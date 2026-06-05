# LossNLL.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/LossNLL.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Loss NLL.
- **Purpose (CN)**: 实现或声明与 损失函数、nll 相关的 ATen 原生逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/core/Tensor.h>
0003: #include <ATen/Dispatch.h>
0004: #include <ATen/Parallel.h>
0005: #include <ATen/TensorIndexing.h>
0006: #include <ATen/TensorMeta.h>
0007: #include <ATen/TensorOperators.h>
0008: #include <ATen/TensorUtils.h>
0009: #include <ATen/native/cpu/utils.h>
0010: #include <ATen/native/Resize.h>
0011: #include <c10/util/SmallBuffer.h>
0012: #include <ATen/TensorSubclassLikeUtils.h>
0013: 
0014: #ifndef AT_PER_OPERATOR_HEADERS
0015: #include <ATen/Functions.h>
0016: #include <ATen/NativeFunctions.h>
0017: #else
0018: #include <ATen/ops/cross_entropy_loss_native.h>
0019: #include <ATen/ops/empty.h>
0020: #include <ATen/ops/log_softmax.h>
0021: #include <ATen/ops/nll_loss.h>
0022: #include <ATen/ops/nll_loss2d.h>
0023: #include <ATen/ops/nll_loss_backward_native.h>
0024: #include <ATen/ops/nll_loss_forward.h>
0025: #include <ATen/ops/nll_loss_forward_native.h>
0026: #include <ATen/ops/nll_loss_native.h>
0027: #include <ATen/ops/nll_loss_nd.h>
0028: #include <ATen/ops/nll_loss_nd_native.h>
0029: #endif
0030: 
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, macro-based glue.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、宏定义或宏调用。

### Lines 31-60 / 第 31-60 行
```cpp
0031: #include <c10/core/TensorOptions.h>
0032: #include <c10/util/irange.h>
0033: 
0034: #include <utility>
0035: 
0036: namespace at::meta {
0037: TORCH_META_FUNC(nll_loss_forward)
0038: (const Tensor& self,
0039:  const Tensor& target,
0040:  const OptionalTensorRef weight_opt,
0041:  int64_t reduction,
0042:  int64_t ignore_index) {
0043:   const Tensor& weight = weight_opt.getTensorRef();
0044: 
0045:   TORCH_CHECK(
0046:       self.dim() > 0 && self.dim() <= 2, "input tensor should be 1D or 2D");
0047:   TORCH_CHECK(
0048:       target.dim() <= 1,
0049:       "0D or 1D target tensor expected, multi-target not supported");
0050:   TORCH_CHECK(
0051:       target.scalar_type() == kLong || target.scalar_type() == kByte,
0052:       "expected target dtype to be Long or Byte, but got ",
0053:       target.scalar_type());
0054:   if (self.dim() == 1 && target.dim() == 1) {
0055:       TORCH_CHECK_VALUE(
0056:           target.size(0) == 1,
0057:           "For 1D input, 1D target must have size 1, but got target size: ",
0058:           target.size(0));
0059:   }
0060:   TORCH_CHECK(
```
- **EN**: Lines 31-60 mainly cover expressions/calls, macro-based glue, state/variable declarations. Notable symbols: TORCH_META_FUNC, getTensorRef, TORCH_CHECK, dim.
- **CN**: 第 31-60 行主要涉及表达式或调用、宏定义或宏调用、变量/别名声明。 值得关注的符号包括：TORCH_META_FUNC, getTensorRef, TORCH_CHECK, dim。

### Lines 61-90 / 第 61-90 行
```cpp
0061:       self.dim() == 1 || (self.size(0) == target.size(0)),
0062:       "size mismatch (got input: ",
0063:       self.sizes(),
0064:       ", target: ",
0065:       target.sizes(),
0066:       ")")
0067: 
0068:   const auto n_classes = self.size(-1);
0069: 
0070:   TORCH_CHECK(
0071:       !weight.defined() || (weight.dim() == 1 && weight.numel() == n_classes),
0072:       "weight tensor should be defined either for all ",
0073:       n_classes,
0074:       " classes or no classes"
0075:       " but got weight tensor of shape: ",
0076:       weight.sizes());
0077: 
0078:   const auto n_dims = self.dim();
0079:   const auto batch_size = self.size(0);
0080: 
0081:   if (reduction == Reduction::None && n_dims == 2) {
0082:     set_output_raw_strided(0, {batch_size}, {}, self.options());
0083:   } else {
0084:     // produce scalar output when reducing or input is 1d
0085:     set_output_raw_strided(0, {}, {}, self.options());
0086:   }
0087: 
0088:   set_output_raw_strided(1, {}, {}, self.options());
0089: }
0090: 
```
- **EN**: Lines 61-90 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: dim, size, mismatch, sizes.
- **CN**: 第 61-90 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：dim, size, mismatch, sizes。

### Lines 91-120 / 第 91-120 行
```cpp
0091: TORCH_META_FUNC(nll_loss_backward)
0092: (const Tensor& grad_output,
0093:  const Tensor& self,
0094:  const Tensor& target,
0095:  OptionalTensorRef weight_opt,
0096:  int64_t reduction,
0097:  int64_t ignore_index,
0098:  const Tensor& total_weight) {
0099:   TORCH_CHECK(
0100:       self.dim() > 0 && self.dim() <= 2, "input tensor should be 1D or 2D");
0101:   TORCH_CHECK(
0102:       target.dim() <= 1,
0103:       "0D or 1D target tensor expected, multi-target not supported");
0104: 
0105:   auto no_batch_dim = self.dim() == 1  && target.dim() == 0;
0106:   TORCH_CHECK(
0107:       no_batch_dim || (self.size(0) == target.size(0)),
0108:       "size mismatch (got input: ",
0109:       self.sizes(),
0110:       ", target: ",
0111:       target.sizes(),
0112:       ")")
0113:   TORCH_CHECK(
0114:       total_weight.numel() == 1,
0115:       "expected total_weight to be a  single element tensor, got: ",
0116:       total_weight.sizes(),
0117:       " (",
0118:       total_weight.numel(),
0119:       " elements)");
0120: 
```
- **EN**: Lines 91-120 mainly cover expressions/calls, function signatures/definitions, macro-based glue. Notable symbols: TORCH_META_FUNC, TORCH_CHECK, dim, size.
- **CN**: 第 91-120 行主要涉及表达式或调用、函数签名或实现、宏定义或宏调用。 值得关注的符号包括：TORCH_META_FUNC, TORCH_CHECK, dim, size。

### Lines 121-150 / 第 121-150 行
```cpp
0121:   const auto& weight = weight_opt.getTensorRef();
0122: 
0123:   TORCH_CHECK(
0124:       !weight.defined() || weight.numel() == self.size(-1),
0125:       "weight tensor should be defined either for all or no classes");
0126: 
0127:   const auto n_dims = self.dim();
0128: 
0129:   if (reduction == Reduction::None && n_dims == 2) {
0130:     const auto batch_size = self.size(0);
0131:     check_dim_size(grad_output, 1, 0, batch_size);
0132:   } else {
0133:     TORCH_CHECK(
0134:         grad_output.dim() <= 1 && grad_output.numel() == 1,
0135:         "Expected a single element grad_output tensor, but got: ",
0136:         grad_output.sizes());
0137:   }
0138: 
0139:   set_output_raw_strided(0, self.sizes(), {}, self.options().memory_format(LEGACY_CONTIGUOUS_MEMORY_FORMAT));
0140: }
0141: } // namespace at::meta
0142: 
0143: namespace at::native {
0144: 
0145: namespace {
0146: 
0147: // Returns a contiguous tensor if the source tensor
0148: // is defined. Otherwise returns the undefined
0149: // source tensor unmodified.
0150: inline Tensor optional_contiguous(const Tensor& source) {
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, function signatures/definitions, expressions/calls. Notable symbols: getTensorRef, TORCH_CHECK, defined, numel.
- **CN**: 第 121-150 行主要涉及变量/别名声明、函数签名或实现、表达式或调用。 值得关注的符号包括：getTensorRef, TORCH_CHECK, defined, numel。

### Lines 151-180 / 第 151-180 行
```cpp
0151:   return source.defined() ? source.contiguous() : source;
0152: }
0153: 
0154: // Returns the address of the first element of a tensor
0155: // or nullptr if the tensor is undefined.
0156: template <typename scalar_t>
0157: inline scalar_t* optional_data(const Tensor& source) {
0158:   if constexpr (std::is_const_v<scalar_t>) {
0159:     return source.defined() ? source.const_data_ptr<scalar_t>() : nullptr;
0160:   } else {
0161:     return source.defined() ? source.data_ptr<scalar_t>() : nullptr;
0162:   }
0163: }
0164: 
0165: template <typename scalar_t, typename target_t>
0166: void nll_loss_out_frame(
0167:     const Tensor& output,
0168:     const Tensor& total_weight,
0169:     const Tensor& input,
0170:     const Tensor& target,
0171:     const Tensor& weight,
0172:     int64_t reduction,
0173:     int64_t ignore_index) {
0174:   const auto n_dims = input.dim();
0175:   const auto n_classes = input.size(-1);
0176: 
0177:   scalar_t* total_weight_data = total_weight.data_ptr<scalar_t>();
0178:   *total_weight_data = 0;
0179: 
0180:   auto weight_contiguous = optional_contiguous(weight);
```
- **EN**: Lines 151-180 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: defined, contiguous, optional_data, constexpr.
- **CN**: 第 151-180 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：defined, contiguous, optional_data, constexpr。

### Lines 181-210 / 第 181-210 行
```cpp
0181:   const scalar_t* weight_data = optional_data<const scalar_t>(weight_contiguous);
0182: 
0183:   if (reduction == Reduction::None && n_dims == 2) {
0184:     const auto batch_size = input.size(0);
0185:     at::native::resize_output(output, {batch_size});
0186: 
0187:     auto input_acc = input.accessor<const scalar_t, 2>();
0188:     auto target_acc = target.accessor<const target_t, 1>();
0189:     auto output_acc = output.accessor<scalar_t, 1>();
0190: 
0191:     at::parallel_for(0, batch_size, 0, [&](int64_t start, int64_t end) {
0192:       for (const auto i : c10::irange(start, end)) {
0193:         const auto cur_target = target_acc[i];
0194: 
0195:         if (cur_target == ignore_index) {
0196:           output_acc[i] = 0;
0197:           continue;
0198:         }
0199: 
0200:         TORCH_CHECK_INDEX(
0201:             cur_target >= 0 && cur_target < n_classes,
0202:             "Target ",
0203:             cur_target,
0204:             " is out of bounds.");
0205: 
0206:         scalar_t cur_weight = weight_data != nullptr ? weight_data[cur_target]
0207:                                                      : static_cast<scalar_t>(1);
0208:         output_acc[i] = -input_acc[i][cur_target] * cur_weight;
0209:       }
0210:     });
```
- **EN**: Lines 181-210 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: size, resize_output, parallel_for, irange.
- **CN**: 第 181-210 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：size, resize_output, parallel_for, irange。

### Lines 211-240 / 第 211-240 行
```cpp
0211: 
0212:     return;
0213:   }
0214: 
0215:   // produce scalar outputs for the reduction case
0216:   at::native::resize_output(output, {});
0217: 
0218:   if (target.numel() == 0) {
0219:     // Here target (and input) have zero elements
0220:     // Mean reduction on empty tensors produces NaN. See the discussion in
0221:     // https://github.com/pytorch/pytorch/pull/64572#issuecomment-926504162
0222:     if (reduction == Reduction::Mean) {
0223:       output.fill_(std::numeric_limits<double>::quiet_NaN());
0224:     } else {
0225:       output.zero_();
0226:     }
0227:     total_weight.zero_();
0228:     return;
0229:   }
0230: 
0231:   auto input_contiguous = input.contiguous();
0232:   auto target_contiguous = target.contiguous();
0233: 
0234:   const scalar_t* input_data = input_contiguous.const_data_ptr<scalar_t>();
0235:   const target_t* target_data = target_contiguous.const_data_ptr<target_t>();
0236: 
0237:   const int64_t ndim = input.dim();
0238:   const int64_t batch_size = ndim == 1 ? 1 : input.size(0);
0239: 
0240:   constexpr int64_t cascade_sum_num_levels = 8;
```
- **EN**: Lines 211-240 mainly cover state/variable declarations, comments/documentation, expressions/calls. Notable symbols: resize_output, numel, target, fill_.
- **CN**: 第 211-240 行主要涉及变量/别名声明、注释或说明、表达式或调用。 值得关注的符号包括：resize_output, numel, target, fill_。

### Lines 241-270 / 第 241-270 行
```cpp
0241:   const int64_t level_power =
0242:       std::max(static_cast<int64_t>(4), utils::CeilLog2(batch_size) / cascade_sum_num_levels);
0243:   const int64_t level_step = (1 << level_power);
0244:   const int64_t level_mask = level_step - 1;
0245: 
0246:   int64_t num_ignored = 0;
0247: 
0248:   // NOLINTNEXTLINE(cppcoreguidelines-avoid-c-arrays,modernize-avoid-c-arrays)
0249:   scalar_t weight_partial_sums[cascade_sum_num_levels] = {0};
0250:   // NOLINTNEXTLINE(cppcoreguidelines-avoid-c-arrays,modernize-avoid-c-arrays)
0251:   scalar_t loss_partial_sums[cascade_sum_num_levels] = {0};
0252:   for (const auto b : c10::irange(batch_size)) {
0253:     const int64_t cur_target = target_data[b];
0254:     if (cur_target == ignore_index) {
0255:       ++num_ignored;
0256:       continue;
0257:     }
0258: 
0259:     TORCH_CHECK_INDEX(
0260:         cur_target >= 0 && cur_target < n_classes,
0261:         "Target ",
0262:         cur_target,
0263:         " is out of bounds.");
0264: 
0265:     const auto data = input_data[b * n_classes + cur_target];
0266:     if (weight_data) {
0267:       const scalar_t weight_val = weight_data[cur_target];
0268:       loss_partial_sums[0] -= data * weight_val;
0269:       weight_partial_sums[0] += weight_val;
0270:     } else {
```
- **EN**: Lines 241-270 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: max, CeilLog2, NOLINTNEXTLINE, irange.
- **CN**: 第 241-270 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：max, CeilLog2, NOLINTNEXTLINE, irange。

### Lines 271-300 / 第 271-300 行
```cpp
0271:       loss_partial_sums[0] -= data;
0272:     }
0273: 
0274:     for (int64_t j = 0; j + 1 < cascade_sum_num_levels; ++j) {
0275:       const auto mask = (level_mask << (j * level_power));
0276:       if (C10_LIKELY((b & mask) != 0)) {
0277:         break;
0278:       }
0279: 
0280:       weight_partial_sums[j + 1] += weight_partial_sums[j];
0281:       loss_partial_sums[j + 1] += loss_partial_sums[j];
0282: 
0283:       weight_partial_sums[j] = 0;
0284:       loss_partial_sums[j] = 0;
0285:     }
0286:   }
0287: 
0288:   const scalar_t total_weight_val = !weight_data ?
0289:     static_cast<scalar_t>(batch_size - num_ignored) :
0290:     std::accumulate(std::begin(weight_partial_sums),
0291:                     std::end(weight_partial_sums),
0292:                     scalar_t{0});
0293: 
0294:   scalar_t output_val = std::accumulate(std::begin(loss_partial_sums),
0295:                                         std::end(loss_partial_sums),
0296:                                         scalar_t{0});
0297: 
0298:   if (reduction == Reduction::Mean) {
0299:     output_val /= total_weight_val;
0300:   }
```
- **EN**: Lines 271-300 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: C10_LIKELY, accumulate, begin, end.
- **CN**: 第 271-300 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：C10_LIKELY, accumulate, begin, end。

### Lines 301-330 / 第 301-330 行
```cpp
0301: 
0302:   // write result to output tensors
0303:   *output.data_ptr<scalar_t>() = output_val;
0304:   *total_weight_data = total_weight_val;
0305: }
0306: 
0307: void nll_loss_forward_out_cpu_template(
0308:     const Tensor& output,
0309:     const Tensor& total_weight,
0310:     const Tensor& input,
0311:     const Tensor& target,
0312:     const Tensor& weight,
0313:     int64_t reduction,
0314:     int64_t ignore_index) {
0315:   AT_DISPATCH_FLOATING_TYPES_AND2(
0316:       ScalarType::BFloat16,
0317:       ScalarType::Half,
0318:       input.scalar_type(),
0319:       "nll_loss_out_frame",
0320:       [&] {
0321:         if (target.scalar_type() == kByte) {
0322:           nll_loss_out_frame<scalar_t, uint8_t>(
0323:               output,
0324:               total_weight,
0325:               input,
0326:               target,
0327:               weight,
0328:               reduction,
0329:               ignore_index);
0330:         } else {
```
- **EN**: Lines 301-330 mainly cover expressions/calls, function signatures/definitions, comments/documentation. Notable symbols: nll_loss_forward_out_cpu_template, AT_DISPATCH_FLOATING_TYPES_AND2, scalar_type.
- **CN**: 第 301-330 行主要涉及表达式或调用、函数签名或实现、注释或说明。 值得关注的符号包括：nll_loss_forward_out_cpu_template, AT_DISPATCH_FLOATING_TYPES_AND2, scalar_type。

### Lines 331-360 / 第 331-360 行
```cpp
0331:           // assumed to be int64
0332:           nll_loss_out_frame<scalar_t, int64_t>(
0333:               output,
0334:               total_weight,
0335:               input,
0336:               target,
0337:               weight,
0338:               reduction,
0339:               ignore_index);
0340:         }
0341:       });
0342: }
0343: 
0344: template <typename scalar_t, typename target_t>
0345: void nll_loss_backward_out_frame(
0346:     const Tensor& grad_input,
0347:     const Tensor& grad_output,
0348:     const Tensor& input,
0349:     const Tensor& target,
0350:     const Tensor& weight,
0351:     int64_t reduction,
0352:     int64_t ignore_index,
0353:     const Tensor& total_weight) {
0354:   const auto n_dims = input.dim();
0355:   const auto n_classes = input.size(-1);
0356: 
0357:   auto target_ = target;
0358:   if (target.dim() == 0) {
0359:     target_ = target.unsqueeze(0);
0360:   }
```
- **EN**: Lines 331-360 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: nll_loss_backward_out_frame, dim, size, unsqueeze.
- **CN**: 第 331-360 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：nll_loss_backward_out_frame, dim, size, unsqueeze。

### Lines 361-390 / 第 361-390 行
```cpp
0361:   auto target_acc = target_.accessor<const target_t, 1>();
0362: 
0363:   auto weight_contiguous = optional_contiguous(weight);
0364:   const scalar_t* weight_data = optional_data<const scalar_t>(weight_contiguous);
0365: 
0366:   if (reduction == Reduction::None && n_dims == 2) {
0367:     const auto batch_size = input.size(0);
0368:     auto grad_input_acc = grad_input.accessor<scalar_t, 2>();
0369:     auto grad_output_acc = grad_output.accessor<const scalar_t, 1>();
0370:     at::parallel_for(0, batch_size, 0, [&](int64_t start, int64_t end) {
0371:       for (const auto i : c10::irange(start, end)) {
0372:         auto cur_target = target_acc[i];
0373:         if (cur_target == ignore_index) {
0374:           continue;
0375:         }
0376:         const scalar_t w =
0377:             weight_data ? weight_data[cur_target] : static_cast<scalar_t>(1);
0378:         grad_input_acc[i][cur_target] = -w * grad_output_acc[i];
0379:       }
0380:     });
0381:     return;
0382:   }
0383: 
0384:   const scalar_t total_weight_value = *total_weight.const_data_ptr<scalar_t>();
0385: 
0386:   const scalar_t grad_output_value = *grad_output.const_data_ptr<scalar_t>();
0387: 
0388:   if (input.dim() == 1) {
0389:     auto grad_input_acc = grad_input.accessor<scalar_t, 1>();
0390: 
```
- **EN**: Lines 361-390 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: optional_contiguous, size, parallel_for, irange.
- **CN**: 第 361-390 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：optional_contiguous, size, parallel_for, irange。

### Lines 391-420 / 第 391-420 行
```cpp
0391:     const auto t = target_acc[0];
0392:     if (t != ignore_index) {
0393:       TORCH_CHECK_INDEX(t >= 0 && t < n_classes, "Target ", t, " is out of bounds.");
0394:       const auto grad = -(reduction == Reduction::Mean ? grad_output_value / total_weight_value
0395:                                                        : grad_output_value);
0396:       grad_input_acc[t] = weight_data != nullptr ? weight_data[t] * grad
0397:                                                  : grad;
0398:     }
0399:   } else if (input.dim() == 2) {
0400:     auto grad_input_acc = grad_input.accessor<scalar_t, 2>();
0401:     const auto grad = -(reduction == Reduction::Mean ? grad_output_value / total_weight_value
0402:                                                      : grad_output_value);
0403: 
0404:     const auto batch_size = input.size(0);
0405: 
0406:     at::parallel_for(0, batch_size, 0, [&](int64_t start, int64_t end) {
0407:       for (const auto i : c10::irange(start, end)) {
0408:         const auto t = target_acc[i];
0409:         if (t != ignore_index) {
0410:           TORCH_CHECK_INDEX(t >= 0 && t < n_classes, "Target ", t, " is out of bounds.");
0411:           grad_input_acc[i][t] = weight_data != nullptr ? weight_data[t] * grad
0412:                                                         : grad;
0413:         }
0414:       }
0415:     });
0416:   }
0417: }
0418: 
0419: void nll_loss_backward_out_cpu_template(
0420:     const Tensor& grad_input,
```
- **EN**: Lines 391-420 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: TORCH_CHECK_INDEX, dim, size, parallel_for.
- **CN**: 第 391-420 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：TORCH_CHECK_INDEX, dim, size, parallel_for。

### Lines 421-450 / 第 421-450 行
```cpp
0421:     const Tensor& grad_output,
0422:     const Tensor& input,
0423:     const Tensor& target,
0424:     const Tensor& weight,
0425:     int64_t reduction,
0426:     int64_t ignore_index,
0427:     const Tensor& total_weight) {
0428:   grad_input.zero_();
0429: 
0430:   AT_DISPATCH_FLOATING_TYPES_AND2(
0431:       ScalarType::BFloat16,
0432:       ScalarType::Half,
0433:       input.scalar_type(),
0434:       "nll_loss_backward_out_frame",
0435:       [&] {
0436:         if (target.scalar_type() == kByte) {
0437:           nll_loss_backward_out_frame<scalar_t, uint8_t>(
0438:               grad_input,
0439:               grad_output,
0440:               input,
0441:               target,
0442:               weight,
0443:               reduction,
0444:               ignore_index,
0445:               total_weight);
0446:         } else {
0447:           // assumed to be uint64
0448:           nll_loss_backward_out_frame<scalar_t, int64_t>(
0449:               grad_input,
0450:               grad_output,
```
- **EN**: Lines 421-450 mainly cover expressions/calls, function signatures/definitions, state/variable declarations. Notable symbols: zero_, AT_DISPATCH_FLOATING_TYPES_AND2, scalar_type.
- **CN**: 第 421-450 行主要涉及表达式或调用、函数签名或实现、变量/别名声明。 值得关注的符号包括：zero_, AT_DISPATCH_FLOATING_TYPES_AND2, scalar_type。

### Lines 451-480 / 第 451-480 行
```cpp
0451:               input,
0452:               target,
0453:               weight,
0454:               reduction,
0455:               ignore_index,
0456:               total_weight);
0457:         }
0458:       });
0459: }
0460: 
0461: } // namespace
0462: 
0463: TORCH_IMPL_FUNC(nll_loss_forward_out_cpu)
0464: (const Tensor& self,
0465:  const Tensor& target,
0466:  const OptionalTensorRef weight_opt,
0467:  int64_t reduction,
0468:  int64_t ignore_index,
0469:  const Tensor& output,
0470:  const Tensor& total_weight) {
0471:   const Tensor& weight = weight_opt.getTensorRef();
0472:   nll_loss_forward_out_cpu_template(
0473:       output, total_weight, self, target, weight, reduction, ignore_index);
0474: }
0475: 
0476: TORCH_IMPL_FUNC(nll_loss_backward_out_cpu)
0477: (const Tensor& grad_output,
0478:  const Tensor& self,
0479:  const Tensor& target,
0480:  OptionalTensorRef weight_opt,
```
- **EN**: Lines 451-480 mainly cover expressions/calls, state/variable declarations, macro-based glue. Notable symbols: TORCH_IMPL_FUNC, getTensorRef, nll_loss_forward_out_cpu_template.
- **CN**: 第 451-480 行主要涉及表达式或调用、变量/别名声明、宏定义或宏调用。 值得关注的符号包括：TORCH_IMPL_FUNC, getTensorRef, nll_loss_forward_out_cpu_template。

### Lines 481-510 / 第 481-510 行
```cpp
0481:  int64_t reduction,
0482:  int64_t ignore_index,
0483:  const Tensor& total_weight,
0484:  const Tensor& grad_input
0485: ) {
0486:   const Tensor& weight = weight_opt.getTensorRef();
0487:   nll_loss_backward_out_cpu_template(
0488:       grad_input,
0489:       grad_output,
0490:       self,
0491:       target,
0492:       weight,
0493:       reduction,
0494:       ignore_index,
0495:       total_weight);
0496: }
0497: 
0498: static Tensor cross_entropy_loss_prob_target(
0499:     const Tensor& self,
0500:     const Tensor& target_,
0501:     const Tensor& weight,
0502:     int64_t reduction,
0503:     double label_smoothing) {
0504:   const auto class_dim = self.dim() == 1 ? 0 : 1;
0505:   const auto n_classes = self.size(class_dim);
0506:   TORCH_CHECK(
0507:       !weight.defined() || (weight.dim() == 1 && weight.numel() == n_classes),
0508:       "cross_entropy: weight tensor should be defined either for all ",
0509:       n_classes,
0510:       " classes or no classes"
```
- **EN**: Lines 481-510 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: getTensorRef, nll_loss_backward_out_cpu_template, cross_entropy_loss_prob_target, dim.
- **CN**: 第 481-510 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：getTensorRef, nll_loss_backward_out_cpu_template, cross_entropy_loss_prob_target, dim。

### Lines 511-540 / 第 511-540 行
```cpp
0511:       " but got weight tensor of shape: ",
0512:       weight.sizes());
0513: 
0514:   auto input = at::log_softmax(self, class_dim, self.scalar_type());
0515:   Tensor target;
0516: 
0517:   if (label_smoothing > 0.0) {
0518:     TORCH_CHECK(label_smoothing <= 1.0, "label_smoothing must be between 0.0 and 1.0. Got: ", label_smoothing);
0519:     target = target_ * (1 - label_smoothing) + label_smoothing / n_classes;
0520:   } else {
0521:     target = target_;
0522:   }
0523: 
0524:   if (weight.defined()) {
0525:     // Expand weight to the correct number of dims for broadcasting with input / target
0526:     Tensor weight_ = weight;
0527:     if (input.dim() > 1) {
0528:         auto weight_broadcast_shape = SmallBuffer<int64_t, 5>(input.dim());
0529:         std::fill(weight_broadcast_shape.begin(), weight_broadcast_shape.end(), 1);
0530:         weight_broadcast_shape[1] = weight.size(0);
0531:         weight_ = weight.view(weight_broadcast_shape);
0532:     }
0533: 
0534:     switch (reduction) {
0535:       case Reduction::Mean:
0536:         if (input.sym_numel()==0){
0537:           return -(input * target * weight_).sum().fill_(std::numeric_limits<double>::quiet_NaN());
0538:         } else {
0539:           return -(input * target * weight_).sum() / (input.sym_numel() / n_classes);
0540:         }
```
- **EN**: Lines 511-540 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: sizes, log_softmax, scalar_type, TORCH_CHECK.
- **CN**: 第 511-540 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：sizes, log_softmax, scalar_type, TORCH_CHECK。

### Lines 541-570 / 第 541-570 行
```cpp
0541:       case Reduction::Sum:
0542:         return -(input * target * weight_).sum();
0543:       case Reduction::None:
0544:         return -(input * target * weight_).sum(class_dim);
0545:       default:
0546:         TORCH_CHECK(false, "Invalid reduction type encountered in cross_entropy: ", reduction);
0547:     }
0548:   } else {
0549:     switch (reduction) {
0550:       case Reduction::Mean:
0551:         if (input.sym_numel()==0){
0552:           return -(input * target).sum().fill_(std::numeric_limits<double>::quiet_NaN());
0553:         } else {
0554:           return -(input * target).sum() / (input.sym_numel() / n_classes);
0555:         }
0556:       case Reduction::Sum:
0557:         return -(input * target).sum();
0558:       case Reduction::None:
0559:         return -(input * target).sum(class_dim);
0560:       default:
0561:         TORCH_CHECK(false, "Invalid reduction type encountered in cross_entropy: ", reduction);
0562:     }
0563:   }
0564: }
0565: 
0566: static Tensor cross_entropy_loss_label_smoothing(
0567:     const Tensor& self,
0568:     const Tensor& target,
0569:     const Tensor& weight,
0570:     int64_t reduction,
```
- **EN**: Lines 541-570 mainly cover expressions/calls, control-flow checks, return paths. Notable symbols: sum, TORCH_CHECK, sym_numel, fill_.
- **CN**: 第 541-570 行主要涉及表达式或调用、控制流逻辑、返回路径。 值得关注的符号包括：sum, TORCH_CHECK, sym_numel, fill_。

### Lines 571-600 / 第 571-600 行
```cpp
0571:     c10::SymInt ignore_index,
0572:     double label_smoothing) {
0573:     auto class_dim = self.dim() == 1 ? 0 : 1;
0574:     auto input = at::log_softmax(self, class_dim, self.scalar_type());
0575:     auto nllloss = at::nll_loss_nd_symint(input, target, weight, reduction, ignore_index);
0576: 
0577:     auto n_classes = input.sym_size(class_dim);
0578: 
0579:     Tensor smooth_loss;
0580:     if (weight.defined()) {
0581:       // Expand weight to the correct number of dims for broadcasting with input / target
0582:       auto weight_broadcast_shape = SmallBuffer<int64_t, 5>(input.dim());
0583:       std::fill(weight_broadcast_shape.begin(), weight_broadcast_shape.end(), 1);
0584:       weight_broadcast_shape[class_dim] = weight.size(0);
0585:       Tensor weight_ = weight.view(weight_broadcast_shape);
0586: 
0587:       smooth_loss = -(input * weight_).sum(class_dim);
0588:     } else {
0589:       smooth_loss = -input.sum(class_dim);
0590:     }
0591: 
0592:     auto ignore_mask = target == std::move(ignore_index);
0593:     smooth_loss.masked_fill_(ignore_mask, 0.0);
0594: 
0595:     Tensor ret;
0596:     switch (reduction) {
0597:       case Reduction::Mean:
0598:         if (weight.defined()) {
0599:           if (isTensorSubclassLike(weight)){
0600:             // we will collect weights from 0 index which is always valid
```
- **EN**: Lines 571-600 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: dim, log_softmax, scalar_type, nll_loss_nd_symint.
- **CN**: 第 571-600 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：dim, log_softmax, scalar_type, nll_loss_nd_symint。

### Lines 601-630 / 第 601-630 行
```cpp
0601:             // and mask them out if they are ignored
0602:             auto filtered_target = target.masked_fill(ignore_mask, 0);
0603:             auto tgt_weights = weight.gather(0, filtered_target.flatten());
0604:             auto weight_sum =
0605:                 tgt_weights.masked_fill_(ignore_mask.flatten(), 0).sum();
0606:             ret = smooth_loss.sum() / weight_sum;
0607:           } else {
0608:             // TODO: This code can path can be removed if #61309 is resolved
0609:             // loss is normalized by the weights to be consistent with
0610:             // nll_loss_nd
0611:             ret = smooth_loss.sum() /
0612:                 weight.gather(0, target.masked_select(~ignore_mask).flatten())
0613:                     .sum();
0614:           }
0615:         } else {
0616:           auto true_mask = ~ignore_mask;
0617:           ret = smooth_loss.sum()/ true_mask.sum();
0618:         }
0619:         break;
0620:       case Reduction::Sum:
0621:         ret = smooth_loss.sum();
0622:         break;
0623:       case Reduction::None:
0624:         ret = smooth_loss;
0625:         break;
0626:       default:
0627:         TORCH_CHECK(false, "Invalid reduction type encountered in cross_entropy: ", reduction);
0628:     }
0629:     return (1 - label_smoothing) * nllloss + ret * (label_smoothing / n_classes);
0630: }
```
- **EN**: Lines 601-630 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: masked_fill, gather, flatten, masked_fill_.
- **CN**: 第 601-630 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：masked_fill, gather, flatten, masked_fill_。

### Lines 631-660 / 第 631-660 行
```cpp
0631: 
0632: Tensor cross_entropy_loss_symint(
0633:     const Tensor& self,
0634:     const Tensor& target,
0635:     const std::optional<Tensor>& weight,
0636:     int64_t reduction,
0637:     c10::SymInt ignore_index,
0638:     double label_smoothing) {
0639:   Tensor ret;
0640:   if (self.sym_sizes() == target.sym_sizes()) {
0641:     // Assume soft targets when input and target shapes are the same
0642:     TORCH_CHECK(at::isFloatingType(target.scalar_type()),
0643:         "Expected floating point type for target with class probabilities, got ", target.scalar_type());
0644:     TORCH_CHECK(ignore_index < 0, "ignore_index is not supported for floating point target");
0645: 
0646:     // See [Note: hacky wrapper removal for optional tensor]
0647:     c10::MaybeOwned<Tensor> weight_maybe_owned = at::borrow_from_optional_tensor(weight);
0648:     const Tensor& weight_ = *weight_maybe_owned;
0649:     ret = cross_entropy_loss_prob_target(self, target, weight_, reduction, label_smoothing);
0650:   } else if (label_smoothing > 0.0) {
0651:     TORCH_CHECK(label_smoothing <= 1.0, "label_smoothing must be between 0.0 and 1.0. Got: ", label_smoothing);
0652: 
0653:     // See [Note: hacky wrapper removal for optional tensor]
0654:     c10::MaybeOwned<Tensor> weight_maybe_owned = at::borrow_from_optional_tensor(weight);
0655:     const Tensor& weight_ = *weight_maybe_owned;
0656:     ret = cross_entropy_loss_label_smoothing(self, target, weight_, reduction, std::move(ignore_index), label_smoothing);
0657:   } else {
0658:     auto class_dim = self.dim() == 1 ? 0 : 1;
0659:     ret = at::nll_loss_nd_symint(
0660:         at::log_softmax(self, class_dim, self.scalar_type()),
```
- **EN**: Lines 631-660 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: cross_entropy_loss_symint, sym_sizes, TORCH_CHECK, isFloatingType.
- **CN**: 第 631-660 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：cross_entropy_loss_symint, sym_sizes, TORCH_CHECK, isFloatingType。

### Lines 661-690 / 第 661-690 行
```cpp
0661:         target,
0662:         weight,
0663:         reduction,
0664:         std::move(ignore_index));
0665:   }
0666:   return ret;
0667: }
0668: 
0669: Tensor & nll_loss_out(const Tensor & self, const Tensor & target, const std::optional<Tensor>& weight_opt, int64_t reduction, int64_t ignore_index, Tensor & output) {
0670:   Tensor total_weight = at::empty({0}, self.options());
0671:   return std::get<0>(at::nll_loss_forward_out(output, total_weight, self, target, weight_opt, reduction, ignore_index));
0672: }
0673: 
0674: Tensor nll_loss_symint(const Tensor & self, const Tensor & target, const std::optional<Tensor>& weight_opt, int64_t reduction, c10::SymInt ignore_index) {
0675:   return std::get<0>(at::nll_loss_forward_symint(self, target, weight_opt, reduction, std::move(ignore_index)));
0676: }
0677: 
0678: Tensor nll_loss_nd_symint(
0679:     const Tensor& self,
0680:     const Tensor& target,
0681:     const std::optional<Tensor>& weight,
0682:     int64_t reduction,
0683:     c10::SymInt ignore_index) {
0684:   if (self.dim() < 1) {
0685:     TORCH_CHECK_VALUE(
0686:         false, "Expected 1 or more dimensions (got ", self.dim(), ")");
0687:   }
0688: 
0689:   if (self.dim() != 1) {
0690:     auto sizes_match = self.sym_sizes()[0].sym_eq(target.sym_sizes()[0]);
```
- **EN**: Lines 661-690 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: move, nll_loss_out, empty, options.
- **CN**: 第 661-690 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：move, nll_loss_out, empty, options。

### Lines 691-720 / 第 691-720 行
```cpp
0691:     if (TORCH_GUARD_OR_FALSE(sizes_match.sym_not())) {
0692:       // Statically known mismatch - raise ValueError for eager mode
0693:       TORCH_CHECK_VALUE(
0694:           false,
0695:           "Expected input batch_size (",
0696:           self.sym_sizes()[0],
0697:           ") to match target batch_size (",
0698:           target.sym_sizes()[0],
0699:           ").");
0700:     }
0701:     // For unbacked symbolic shapes, emit runtime check.
0702:     TORCH_SYM_CHECK(
0703:         sizes_match,
0704:         "Expected input batch_size (",
0705:         self.sym_sizes()[0],
0706:         ") to match target batch_size (",
0707:         target.sym_sizes()[0],
0708:         ").");
0709:   }
0710: 
0711:   Tensor ret;
0712:   Tensor input_ = self;
0713:   Tensor target_ = target;
0714:   if (input_.dim() == 1 || input_.dim() == 2) {
0715:     ret = at::nll_loss_symint(input_, target_, weight, reduction, std::move(ignore_index));
0716:   } else if (input_.dim() == 4) {
0717:     ret = at::nll_loss2d_symint(input_, target_, weight, reduction, std::move(ignore_index));
0718:   } else {
0719:     // dim == 3 or dim > 4
0720:     auto n = input_.sym_sizes()[0];
```
- **EN**: Lines 691-720 mainly cover function signatures/definitions, state/variable declarations, expressions/calls. Notable symbols: TORCH_GUARD_OR_FALSE, sym_not, TORCH_CHECK_VALUE, batch_size.
- **CN**: 第 691-720 行主要涉及函数签名或实现、变量/别名声明、表达式或调用。 值得关注的符号包括：TORCH_GUARD_OR_FALSE, sym_not, TORCH_CHECK_VALUE, batch_size。

### Lines 721-750 / 第 721-750 行
```cpp
0721:     auto c = input_.sym_sizes()[1];
0722:     auto out_size = input_.sym_sizes().slice(2).vec();
0723:     out_size.insert(out_size.begin(), n);
0724:     if (target_.sym_sizes().slice(1) != input_.sym_sizes().slice(2)) {
0725:       TORCH_CHECK(
0726:           false,
0727:           "Expected target size ",
0728:           SymIntArrayRef(out_size),
0729:           ", got ",
0730:           target_.sym_sizes());
0731:     }
0732:     input_ = input_.contiguous();
0733:     target_ = target_.contiguous();
0734:     // support empty batches, see #15870
0735:     if (input_.sym_numel() > 0) {
0736:       input_ = input_.view_symint({n, std::move(c), 1, -1});
0737:     } else {
0738:       input_ = input_.view_symint({n, std::move(c), 0, 0});
0739:     }
0740:     if (target_.sym_numel() > 0) {
0741:       target_ = target_.view_symint({std::move(n), 1, -1});
0742:     } else {
0743:       target_ = target_.view_symint({std::move(n), 0, 0});
0744:     }
0745:     if (reduction != Reduction::None) {
0746:       ret = at::nll_loss2d_symint(input_, target_, weight, reduction, std::move(ignore_index));
0747:     } else {
0748:       auto out =
0749:           at::nll_loss2d_symint(input_, target_, weight, reduction, std::move(ignore_index));
0750:       ret = out.view_symint(out_size);
```
- **EN**: Lines 721-750 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: sym_sizes, slice, vec, insert.
- **CN**: 第 721-750 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：sym_sizes, slice, vec, insert。

### Lines 751-756 / 第 751-756 行
```cpp
0751:     }
0752:   }
0753:   return ret;
0754: }
0755: 
0756: } // namespace at::native
```
- **EN**: Lines 751-756 mainly cover expressions/calls, return paths, namespace structuring.
- **CN**: 第 751-756 行主要涉及表达式或调用、返回路径、命名空间组织。

## Key Concepts / 关键概念
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Parallel loop scheduling  
  **CN**: 并行循环调度
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/Parallel.h>`, `<ATen/TensorIndexing.h>`, `<ATen/TensorMeta.h>`, `<ATen/TensorOperators.h>`, `<ATen/TensorUtils.h>`, `<ATen/native/cpu/utils.h>`, `<ATen/native/Resize.h>`, `<c10/util/SmallBuffer.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `AT_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
