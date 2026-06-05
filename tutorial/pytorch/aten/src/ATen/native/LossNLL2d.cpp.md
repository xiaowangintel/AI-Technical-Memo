# LossNLL2d.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/LossNLL2d.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Loss NLL2d.
- **Purpose (CN)**: 实现或声明与 损失函数、nll2d 相关的 ATen 原生逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/core/Tensor.h>
0003: #include <ATen/Dispatch.h>
0004: #include <ATen/Parallel.h>
0005: #include <ATen/native/cpu/utils.h>
0006: #include <ATen/native/Resize.h>
0007: #include <c10/util/irange.h>
0008: 
0009: #ifndef AT_PER_OPERATOR_HEADERS
0010: #include <ATen/Functions.h>
0011: #include <ATen/NativeFunctions.h>
0012: #else
0013: #include <ATen/ops/empty.h>
0014: #include <ATen/ops/nll_loss2d_backward_native.h>
0015: #include <ATen/ops/nll_loss2d_forward.h>
0016: #include <ATen/ops/nll_loss2d_forward_native.h>
0017: #include <ATen/ops/nll_loss2d_native.h>
0018: #include <ATen/ops/zeros_like.h>
0019: 
0020: #include <utility>
0021: #endif
0022: 
0023: namespace at::native {
0024: 
0025: namespace {
0026: 
0027: // Returns a contiguous tensor if the source tensor
0028: // is defined. Otherwise returns the undefined
0029: // source tensor unmodified.
0030: inline Tensor optional_contiguous(const Tensor& source) {
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, comments/documentation. Notable symbols: optional_contiguous.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、注释或说明。 值得关注的符号包括：optional_contiguous。

### Lines 31-60 / 第 31-60 行
```cpp
0031:   return source.defined() ? source.contiguous() : source;
0032: }
0033: 
0034: // Returns the address of the first element of a tensor
0035: // or nullptr if the tensor is undefined.
0036: template <typename scalar_t>
0037: inline scalar_t* optional_data(const Tensor& source) {
0038:   if constexpr (std::is_const_v<scalar_t>) {
0039:     return source.defined() ? source.const_data_ptr<scalar_t>() : nullptr;
0040:   } else {
0041:     return source.defined() ? source.data_ptr<scalar_t>() : nullptr;
0042:   }
0043: }
0044: 
0045: inline void check_inputs_nll_loss2d(
0046:     const Tensor& input,
0047:     const Tensor& target,
0048:     const Tensor& weight) {
0049:   TORCH_CHECK(
0050:       target.dim() == 3,
0051:       "only batches of spatial targets supported (3D tensors)"
0052:       " but got targets of dimension: ",
0053:       target.dim());
0054:   TORCH_CHECK(
0055:       input.dim() == 4,
0056:       "only batches of spatial inputs supported (4D tensors), "
0057:       "but got input of dimension: ",
0058:       input.dim());
0059:   TORCH_CHECK(
0060:       target.scalar_type() == kLong || target.scalar_type() == kByte,
```
- **EN**: Lines 31-60 mainly cover expressions/calls, function signatures/definitions, state/variable declarations. Notable symbols: defined, contiguous, optional_data, constexpr.
- **CN**: 第 31-60 行主要涉及表达式或调用、函数签名或实现、变量/别名声明。 值得关注的符号包括：defined, contiguous, optional_data, constexpr。

### Lines 61-90 / 第 61-90 行
```cpp
0061:       "expected target dtype to be Long or Byte, but got ",
0062:       target.scalar_type());
0063:   TORCH_CHECK(
0064:       !weight.defined() || weight.numel() == input.size(1),
0065:       "weight tensor should be defined either for all or no classes");
0066: 
0067:   const int64_t input0 = input.size(0);
0068:   const int64_t input2 = input.size(2);
0069:   const int64_t input3 = input.size(3);
0070:   const int64_t target0 = target.size(0);
0071:   const int64_t target1 = target.size(1);
0072:   const int64_t target2 = target.size(2);
0073:   TORCH_CHECK(
0074:       input0 == target0 && input2 == target1 && input3 == target2,
0075:       "size mismatch (got input: ",
0076:       input.sizes(),
0077:       " , target: ",
0078:       target.sizes());
0079: }
0080: 
0081: inline void check_gradout_shape_nll_loss2d(
0082:     const Tensor& grad_output,
0083:     const Tensor& target) {
0084:   TORCH_CHECK(
0085:       grad_output.dim() == 3,
0086:       "grad_output must have same dimension as target (3) but got dimension: ",
0087:       grad_output.sizes());
0088: 
0089:   const int64_t grad_output0 = grad_output.size(0);
0090:   const int64_t grad_output1 = grad_output.size(1);
```
- **EN**: Lines 61-90 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: scalar_type, TORCH_CHECK, defined, numel.
- **CN**: 第 61-90 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：scalar_type, TORCH_CHECK, defined, numel。

### Lines 91-120 / 第 91-120 行
```cpp
0091:   const int64_t grad_output2 = grad_output.size(2);
0092:   const int64_t target0 = target.size(0);
0093:   const int64_t target1 = target.size(1);
0094:   const int64_t target2 = target.size(2);
0095:   TORCH_CHECK(
0096:       grad_output0 == target0 && grad_output1 == target1 &&
0097:           grad_output2 == target2,
0098:       "size mismatch (got grad_output: ",
0099:       grad_output.sizes(),
0100:       " target: ",
0101:       target.sizes());
0102: }
0103: 
0104: 
0105: template <typename scalar_t>
0106: void nll_loss2d_forward_out_frame(
0107:     Tensor& output,
0108:     Tensor& total_weight,
0109:     const Tensor& input,
0110:     const Tensor& target,
0111:     const Tensor& weight,
0112:     int64_t reduction,
0113:     int64_t ignore_index) {
0114:   const int64_t n_classes = input.size(1);
0115: 
0116:   scalar_t* total_weight_data = total_weight.data_ptr<scalar_t>();
0117:   *total_weight_data = 0;
0118: 
0119:   auto weight_contiguous = optional_contiguous(weight);
0120:   const scalar_t* weight_data = optional_data<const scalar_t>(weight_contiguous);
```
- **EN**: Lines 91-120 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: size, TORCH_CHECK, mismatch, sizes.
- **CN**: 第 91-120 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：size, TORCH_CHECK, mismatch, sizes。

### Lines 121-150 / 第 121-150 行
```cpp
0121: 
0122:   if (reduction == Reduction::None) {
0123:     const int64_t batch_size = input.size(0);
0124:     const int64_t H = input.size(2);
0125:     const int64_t W = input.size(3);
0126: 
0127:     at::native::resize_output(output, {batch_size, H, W});
0128:     auto input_acc = input.accessor<const scalar_t, 4>();
0129:     auto output_acc = output.accessor<scalar_t, 3>();
0130:     auto target_acc = target.accessor<const int64_t, 3>();
0131: 
0132:     at::parallel_for(0, batch_size, 0, [&](int64_t start, int64_t end) {
0133:       for (const auto b : c10::irange(start, end)) {
0134:         for (const auto h : c10::irange(H)) {
0135:           for (const auto w : c10::irange(W)) {
0136:             const int64_t cur_target = target_acc[b][h][w];
0137: 
0138:             if (cur_target == ignore_index) {
0139:               output_acc[b][h][w] = static_cast<scalar_t>(0);
0140:               continue;
0141:             }
0142: 
0143:             TORCH_CHECK_INDEX(
0144:                 cur_target >= 0 && cur_target < n_classes,
0145:                 "Target ",
0146:                 cur_target,
0147:                 " is out of bounds.");
0148: 
0149:             // load optional weight value
0150:             const scalar_t cur_weight = weight_data != nullptr
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: size, resize_output, parallel_for, irange.
- **CN**: 第 121-150 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：size, resize_output, parallel_for, irange。

### Lines 151-180 / 第 151-180 行
```cpp
0151:                 ? weight_data[cur_target]
0152:                 : static_cast<scalar_t>(1);
0153:             output_acc[b][h][w] = -input_acc[b][cur_target][h][w] * cur_weight;
0154:           }
0155:         }
0156:       }
0157:     });
0158: 
0159:     return;
0160:   }
0161: 
0162:   // produce scalar outputs for the reduction case
0163:   at::native::resize_output(output, {});
0164: 
0165:   if (target.numel() == 0) {
0166:     // Here target (and input) have zero elements
0167:     // Mean reduction on empty tensors produces NaN. See the discussion in
0168:     // https://github.com/pytorch/pytorch/pull/64572#issuecomment-926504162
0169:     if (reduction == Reduction::Mean) {
0170:       output.fill_(std::numeric_limits<double>::quiet_NaN());
0171:     } else {
0172:       output.zero_();
0173:     }
0174:     total_weight.zero_();
0175:     return;
0176:   }
0177: 
0178:   auto input_contiguous = input.contiguous();
0179:   auto target_contiguous = target.contiguous();
0180: 
```
- **EN**: Lines 151-180 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: resize_output, numel, target, fill_.
- **CN**: 第 151-180 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：resize_output, numel, target, fill_。

### Lines 181-210 / 第 181-210 行
```cpp
0181:   const scalar_t* input_data = input_contiguous.const_data_ptr<scalar_t>();
0182:   const int64_t* target_data = target_contiguous.const_data_ptr<int64_t>();
0183: 
0184:   const int64_t batch_size = input.size(0);
0185:   const int64_t map_size = input.size(2) * input.size(3);
0186:   const int64_t sample_size = map_size * n_classes;
0187:   const int64_t numiter = batch_size * map_size;
0188: 
0189:   constexpr int64_t cascade_sum_num_levels = 8;
0190:   // NOLINTNEXTLINE(cppcoreguidelines-avoid-c-arrays,modernize-avoid-c-arrays)
0191:   scalar_t weight_partial_sums[cascade_sum_num_levels] = {0};
0192:   // NOLINTNEXTLINE(cppcoreguidelines-avoid-c-arrays,modernize-avoid-c-arrays)
0193:   scalar_t loss_partial_sums[cascade_sum_num_levels] = {0};
0194:   const int64_t level_power =
0195:       std::max(static_cast<int64_t>(4), utils::CeilLog2(numiter) / cascade_sum_num_levels);
0196:   const int64_t level_step = (1 << level_power);
0197:   const int64_t level_mask = level_step - 1;
0198: 
0199:   int64_t num_ignored = 0;
0200:   for (const auto b : c10::irange(batch_size)) {
0201:     for (const auto elem : c10::irange(map_size)) {
0202:       const int64_t cur_target = target_data[b * map_size + elem];
0203:       if (cur_target == ignore_index) {
0204:         ++num_ignored;
0205:         continue;
0206:       }
0207: 
0208:       TORCH_CHECK_INDEX(
0209:           cur_target >= 0 && cur_target < n_classes,
0210:           "Target ",
```
- **EN**: Lines 181-210 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: size, NOLINTNEXTLINE, max, CeilLog2.
- **CN**: 第 181-210 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：size, NOLINTNEXTLINE, max, CeilLog2。

### Lines 211-240 / 第 211-240 行
```cpp
0211:           cur_target,
0212:           " is out of bounds.");
0213: 
0214:       const auto data = input_data[b * sample_size + cur_target * map_size + elem];
0215:       if (weight_data) {
0216:         const scalar_t weight_val = weight_data[cur_target];
0217:         loss_partial_sums[0] -= data * weight_val;
0218:         weight_partial_sums[0] += weight_val;
0219:       } else {
0220:         loss_partial_sums[0] -= data;
0221:       }
0222: 
0223:       const int64_t linear_idx = b * map_size + elem;
0224:       for (int64_t j = 0; j + 1 < cascade_sum_num_levels; ++j) {
0225:         const auto mask = (level_mask << (j * level_power));
0226:         if (C10_LIKELY((linear_idx & mask) != 0)) {
0227:           break;
0228:         }
0229: 
0230:         weight_partial_sums[j + 1] += weight_partial_sums[j];
0231:         loss_partial_sums[j + 1] += loss_partial_sums[j];
0232: 
0233:         weight_partial_sums[j] = 0;
0234:         loss_partial_sums[j] = 0;
0235:       }
0236:     }
0237:   }
0238: 
0239: 
0240:   const scalar_t total_weight_val = !weight_data ?
```
- **EN**: Lines 211-240 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: C10_LIKELY.
- **CN**: 第 211-240 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：C10_LIKELY。

### Lines 241-270 / 第 241-270 行
```cpp
0241:     static_cast<scalar_t>(numiter - num_ignored) :
0242:     std::accumulate(std::begin(weight_partial_sums),
0243:                     std::end(weight_partial_sums),
0244:                     scalar_t{0});
0245: 
0246:   scalar_t output_val = std::accumulate(std::begin(loss_partial_sums),
0247:                                         std::end(loss_partial_sums),
0248:                                         scalar_t{0});
0249: 
0250:   if (reduction == Reduction::Mean) {
0251:     output_val /= total_weight_val;
0252:   }
0253: 
0254:   *total_weight_data = total_weight_val;
0255:   *output.data_ptr<scalar_t>() = output_val;
0256: }
0257: 
0258: void nll_loss2d_forward_out_cpu_template(
0259:     Tensor& output,
0260:     Tensor& total_weight,
0261:     const Tensor& input,
0262:     const Tensor& target,
0263:     const Tensor& weight,
0264:     int64_t reduction,
0265:     int64_t ignore_index) {
0266:   check_inputs_nll_loss2d(input, target, weight);
0267:   total_weight.resize_({});
0268: 
0269:   AT_DISPATCH_FLOATING_TYPES_AND2(
0270:       ScalarType::BFloat16,
```
- **EN**: Lines 241-270 mainly cover expressions/calls, function signatures/definitions, state/variable declarations. Notable symbols: accumulate, begin, end, nll_loss2d_forward_out_cpu_template.
- **CN**: 第 241-270 行主要涉及表达式或调用、函数签名或实现、变量/别名声明。 值得关注的符号包括：accumulate, begin, end, nll_loss2d_forward_out_cpu_template。

### Lines 271-300 / 第 271-300 行
```cpp
0271:       ScalarType::Half,
0272:       input.scalar_type(),
0273:       "nll_loss2d_forward_out_frame",
0274:       [&] {
0275:         nll_loss2d_forward_out_frame<scalar_t>(
0276:             output,
0277:             total_weight,
0278:             input,
0279:             target,
0280:             weight,
0281:             reduction,
0282:             ignore_index);
0283:       });
0284: }
0285: 
0286: template <typename scalar_t>
0287: void nll_loss2d_backward_out_frame(
0288:     Tensor& grad_input,
0289:     const Tensor& grad_output,
0290:     const Tensor& input,
0291:     const Tensor& target,
0292:     const Tensor& weight,
0293:     int64_t reduction,
0294:     int64_t ignore_index,
0295:     const Tensor& total_weight) {
0296:   auto weight_contiguous = optional_contiguous(weight);
0297:   const scalar_t* weight_data = optional_data<const scalar_t>(weight_contiguous);
0298: 
0299:   if (reduction == at::Reduction::None) {
0300:     check_gradout_shape_nll_loss2d(grad_output, target);
```
- **EN**: Lines 271-300 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: scalar_type, nll_loss2d_backward_out_frame, optional_contiguous, check_gradout_shape_nll_loss2d.
- **CN**: 第 271-300 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：scalar_type, nll_loss2d_backward_out_frame, optional_contiguous, check_gradout_shape_nll_loss2d。

### Lines 301-330 / 第 301-330 行
```cpp
0301: 
0302:     const int64_t batch_size = input.size(0);
0303:     const int64_t H = input.size(2);
0304:     const int64_t W = input.size(3);
0305: 
0306:     auto grad_input_acc = grad_input.accessor<scalar_t, 4>();
0307:     auto grad_output_acc = grad_output.accessor<const scalar_t, 3>();
0308:     auto target_acc = target.accessor<const int64_t, 3>();
0309: 
0310:     at::parallel_for(0, batch_size, 0, [&](int64_t start, int64_t end) {
0311:       for (const auto b : c10::irange(start, end)) {
0312:         for (const auto h : c10::irange(H)) {
0313:           for (const auto w : c10::irange(W)) {
0314:             const int64_t cur_target = target_acc[b][h][w];
0315:             if (cur_target == ignore_index) {
0316:               continue;
0317:             }
0318:             const scalar_t value =
0319:                 -(weight_data ? weight_data[cur_target]
0320:                               : static_cast<scalar_t>(1));
0321:             const scalar_t grad_output_value = grad_output_acc[b][h][w];
0322:             grad_input_acc[b][cur_target][h][w] = value * grad_output_value;
0323:           }
0324:         }
0325:       }
0326:     });
0327: 
0328:     return;
0329:   }
0330: 
```
- **EN**: Lines 301-330 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: size, parallel_for, irange.
- **CN**: 第 301-330 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：size, parallel_for, irange。

### Lines 331-360 / 第 331-360 行
```cpp
0331:   const scalar_t total_weight_value = *total_weight.const_data_ptr<scalar_t>();
0332: 
0333:   TORCH_CHECK(
0334:       grad_output.dim() <= 1 && grad_output.numel() == 1,
0335:       "Expected a single element grad_output tensor, but got: ",
0336:       grad_output.sizes());
0337: 
0338:   const scalar_t grad_output_value = *grad_output.const_data_ptr<scalar_t>();
0339: 
0340:   const auto target_contiguous = target.contiguous();
0341:   const int64_t* target_data = target_contiguous.const_data_ptr<int64_t>();
0342: 
0343:   scalar_t* grad_input_data = grad_input.mutable_data_ptr<scalar_t>();
0344: 
0345:   const int64_t batch_size = input.size(0);
0346:   const int64_t n_classes = input.size(1);
0347:   const int64_t map_size = input.size(2) * input.size(3);
0348:   const int64_t sample_size = map_size * n_classes;
0349: 
0350:   const auto grad = -(reduction == Reduction::Mean ? grad_output_value / total_weight_value
0351:                                                    : grad_output_value);
0352: 
0353:   at::parallel_for(0, batch_size, 0, [&](int64_t start, int64_t end) {
0354:     for (const auto b : c10::irange(start, end)) {
0355:       for (const auto elem : c10::irange(map_size)) {
0356:         const int64_t t = target_data[b * map_size + elem];
0357: 
0358:         if (t != ignore_index) {
0359:           TORCH_CHECK_INDEX(t >= 0 && t < n_classes, "Target ", t, " is out of bounds.");
0360: 
```
- **EN**: Lines 331-360 mainly cover state/variable declarations, control-flow checks, macro-based glue. Notable symbols: TORCH_CHECK, dim, numel, sizes.
- **CN**: 第 331-360 行主要涉及变量/别名声明、控制流逻辑、宏定义或宏调用。 值得关注的符号包括：TORCH_CHECK, dim, numel, sizes。

### Lines 361-390 / 第 361-390 行
```cpp
0361:           const int64_t index = b * sample_size + t * map_size + elem;
0362:           grad_input_data[index] = weight_data != nullptr ? weight_data[t] * grad
0363:                                                           : grad;
0364:         }
0365:       }
0366:     }
0367:   });
0368: }
0369: 
0370: void nll_loss2d_backward_out_cpu_template(
0371:     Tensor& grad_input,
0372:     const Tensor& grad_output,
0373:     const Tensor& input,
0374:     const Tensor& target,
0375:     const Tensor& weight,
0376:     int64_t reduction,
0377:     int64_t ignore_index,
0378:     const Tensor& total_weight) {
0379:   check_inputs_nll_loss2d(input, target, weight);
0380:   grad_input.resize_as_(input);
0381:   grad_input.zero_();
0382:   TORCH_CHECK(grad_input.is_contiguous(), "grad_input must be contiguous");
0383:   TORCH_CHECK(
0384:       total_weight.numel() == 1,
0385:       "expected total_weight to be a single element tensor, got: ",
0386:       total_weight.sizes(),
0387:       " (",
0388:       total_weight.numel(),
0389:       " elements)");
0390: 
```
- **EN**: Lines 361-390 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: nll_loss2d_backward_out_cpu_template, check_inputs_nll_loss2d, resize_as_, zero_.
- **CN**: 第 361-390 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：nll_loss2d_backward_out_cpu_template, check_inputs_nll_loss2d, resize_as_, zero_。

### Lines 391-420 / 第 391-420 行
```cpp
0391:   AT_DISPATCH_FLOATING_TYPES_AND2(
0392:       ScalarType::BFloat16,
0393:       ScalarType::Half,
0394:       input.scalar_type(),
0395:       "nll_loss2d_backward_out_frame",
0396:       [&] {
0397:         nll_loss2d_backward_out_frame<scalar_t>(
0398:             grad_input,
0399:             grad_output,
0400:             input,
0401:             target,
0402:             weight,
0403:             reduction,
0404:             ignore_index,
0405:             total_weight);
0406:       });
0407: }
0408: 
0409: } // namespace
0410: 
0411: std::tuple<Tensor&, Tensor&> nll_loss2d_forward_out_cpu(const Tensor& self,
0412:     const Tensor& target, const std::optional<Tensor>& weight_opt,
0413:     int64_t reduction,
0414:     int64_t ignore_index,
0415:     Tensor& output,
0416:     Tensor& total_weight) {
0417:   // See [Note: hacky wrapper removal for optional tensor]
0418:   c10::MaybeOwned<Tensor> weight_maybe_owned = at::borrow_from_optional_tensor(weight_opt);
0419:   const Tensor& weight = *weight_maybe_owned;
0420: 
```
- **EN**: Lines 391-420 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: AT_DISPATCH_FLOATING_TYPES_AND2, scalar_type, nll_loss2d_forward_out_cpu, borrow_from_optional_tensor.
- **CN**: 第 391-420 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：AT_DISPATCH_FLOATING_TYPES_AND2, scalar_type, nll_loss2d_forward_out_cpu, borrow_from_optional_tensor。

### Lines 421-450 / 第 421-450 行
```cpp
0421:   nll_loss2d_forward_out_cpu_template(
0422:       output, total_weight, self, target, weight, reduction, ignore_index);
0423:   return std::tuple<Tensor&, Tensor&>(output, total_weight);
0424: }
0425: 
0426: std::tuple<Tensor, Tensor> nll_loss2d_forward_cpu(
0427:     const Tensor& self,
0428:     const Tensor& target, const std::optional<Tensor>& weight_opt,
0429:     int64_t reduction,
0430:     int64_t ignore_index) {
0431:   auto output = at::empty({0}, self.options());
0432:   auto total_weight = at::empty({0}, self.options());
0433:   at::native::nll_loss2d_forward_out_cpu(
0434:       self, target, weight_opt, reduction, ignore_index, output, total_weight);
0435:   return std::make_tuple(std::move(output), std::move(total_weight));
0436: }
0437: 
0438: Tensor& nll_loss2d_backward_out_cpu(const Tensor& grad_output,
0439:     const Tensor& self,
0440:     const Tensor& target, const std::optional<Tensor>& weight_opt,
0441:     int64_t reduction,
0442:     int64_t ignore_index,
0443:     const Tensor& total_weight,
0444:     Tensor& grad_input) {
0445:   // See [Note: hacky wrapper removal for optional tensor]
0446:   c10::MaybeOwned<Tensor> weight_maybe_owned = at::borrow_from_optional_tensor(weight_opt);
0447:   const Tensor& weight = *weight_maybe_owned;
0448: 
0449:   nll_loss2d_backward_out_cpu_template(
0450:       grad_input,
```
- **EN**: Lines 421-450 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: nll_loss2d_forward_out_cpu_template, nll_loss2d_forward_cpu, empty, options.
- **CN**: 第 421-450 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：nll_loss2d_forward_out_cpu_template, nll_loss2d_forward_cpu, empty, options。

### Lines 451-480 / 第 451-480 行
```cpp
0451:       grad_output,
0452:       self,
0453:       target,
0454:       weight,
0455:       reduction,
0456:       ignore_index,
0457:       total_weight);
0458:   return grad_input;
0459: }
0460: 
0461: Tensor nll_loss2d_backward_cpu(
0462:     const Tensor& grad_output,
0463:     const Tensor& self,
0464:     const Tensor& target, const std::optional<Tensor>& weight_opt,
0465:     int64_t reduction,
0466:     int64_t ignore_index,
0467:     const Tensor& total_weight) {
0468:   auto grad_input = at::zeros_like(self);
0469:   at::native::nll_loss2d_backward_out_cpu(
0470:       grad_output,
0471:       self,
0472:       target,
0473:       weight_opt,
0474:       reduction,
0475:       ignore_index,
0476:       total_weight,
0477:       grad_input);
0478:   return grad_input;
0479: }
0480: 
```
- **EN**: Lines 451-480 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: nll_loss2d_backward_cpu, zeros_like, nll_loss2d_backward_out_cpu.
- **CN**: 第 451-480 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：nll_loss2d_backward_cpu, zeros_like, nll_loss2d_backward_out_cpu。

### Lines 481-490 / 第 481-490 行
```cpp
0481: Tensor & nll_loss2d_out(const Tensor & self, const Tensor & target, const std::optional<Tensor>& weight_opt, int64_t reduction, int64_t ignore_index, Tensor & output) {
0482:   Tensor total_weight = at::empty({0}, self.options());
0483:   return std::get<0>(at::nll_loss2d_forward_out(output, total_weight, self, target, weight_opt, reduction, ignore_index));
0484: }
0485: 
0486: Tensor nll_loss2d_symint(const Tensor & self, const Tensor & target, const std::optional<Tensor>& weight_opt, int64_t reduction, c10::SymInt ignore_index) {
0487:   return std::get<0>(at::nll_loss2d_forward_symint(self, target, weight_opt, reduction, std::move(ignore_index)));
0488: }
0489: 
0490: } // namespace at::native
```
- **EN**: Lines 481-490 mainly cover function signatures/definitions, return paths, expressions/calls. Notable symbols: nll_loss2d_out, empty, options, nll_loss2d_forward_out.
- **CN**: 第 481-490 行主要涉及函数签名或实现、返回路径、表达式或调用。 值得关注的符号包括：nll_loss2d_out, empty, options, nll_loss2d_forward_out。

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
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/Parallel.h>`, `<ATen/native/cpu/utils.h>`, `<ATen/native/Resize.h>`, `<c10/util/irange.h>`, `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/empty.h>`, `<ATen/ops/nll_loss2d_backward_native.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `AT_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
