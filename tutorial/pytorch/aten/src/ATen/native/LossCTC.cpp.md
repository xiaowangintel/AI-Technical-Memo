# LossCTC.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/LossCTC.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Loss CTC.
- **Purpose (CN)**: 实现或声明与 损失函数、ctc 相关的 ATen 原生逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: // Copyright (c) 2018 MathInf GmbH, Thomas Viehmann
0002: // Licensed under the BSD-3-Clause license
0003: // This is the CPU implementation of the Connectionist Temporal Loss.
0004: // We mostly follow Graves.
0005: // 1. Graves et al.: http://www.cs.toronto.edu/~graves/icml_2006.pdf
0006: // We use the equations from above link, but note that [1] has 1-based indexing and we (of course) use 0-based.
0007: // Graves et al. call the probabilities y, we use log_probs (also calling them inputs)
0008: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0009: 
0010: #include <ATen/core/Tensor.h>
0011: #include <ATen/Dispatch.h>
0012: #include <ATen/Parallel.h>
0013: #include <ATen/TensorIterator.h>
0014: #include <ATen/TensorOperators.h>
0015: #include <ATen/native/Fill.h>
0016: #include <c10/util/irange.h>
0017: #include <ATen/TensorSubclassLikeUtils.h>
0018: 
0019: #ifndef AT_PER_OPERATOR_HEADERS
0020: #include <ATen/Functions.h>
0021: #include <ATen/NativeFunctions.h>
0022: #else
0023: #include <ATen/ops/_ctc_loss.h>
0024: #include <ATen/ops/_ctc_loss_backward.h>
0025: #include <ATen/ops/_ctc_loss_backward_native.h>
0026: #include <ATen/ops/_ctc_loss_native.h>
0027: #include <ATen/ops/_cudnn_ctc_loss.h>
0028: #include <ATen/ops/_use_cudnn_ctc_loss.h>
0029: #include <ATen/ops/ctc_loss_native.h>
0030: #include <ATen/ops/empty.h>
```
- **EN**: Lines 1-30 mainly cover header inclusion, comments/documentation, conditional compilation. Notable symbols: Copyright, we, log_probs.
- **CN**: 第 1-30 行主要涉及头文件包含、注释或说明、预处理条件。 值得关注的符号包括：Copyright, we, log_probs。

### Lines 31-60 / 第 31-60 行
```cpp
0031: #include <ATen/ops/empty_like.h>
0032: #include <ATen/ops/full_like.h>
0033: #include <ATen/ops/tensor.h>
0034: #include <ATen/ops/where.h>
0035: #include <ATen/ops/zeros.h>
0036: #include <ATen/ops/_use_miopen_ctc_loss.h>
0037: #include <ATen/ops/miopen_ctc_loss.h>
0038: #endif
0039: 
0040: #include <type_traits>
0041: #include <utility>
0042: 
0043: namespace at::native {
0044: 
0045: namespace {
0046: 
0047: // this ad-hoc converts from targets (l in [1]) to augmented targets (l' in [1]) note that no bound-checking is done
0048: template<typename target_t>
0049: inline int64_t get_target_prime(target_t* target, int64_t offset, int64_t stride, int64_t idx, int64_t BLANK) {
0050:   if (idx % 2 == 0) {
0051:     return BLANK;
0052:   } else {
0053:     return target[offset + stride * (idx / 2)];
0054:   }
0055: }
0056: 
0057: template<typename scalar_t, ScalarType target_scalar_type>
0058: std::tuple<Tensor, Tensor, size_t, std::vector<int64_t>> ctc_loss_allocate_outputs(const Tensor& log_probs, const Tensor& targets, IntArrayRef input_lengths, IntArrayRef target_lengths, int64_t BLANK) {
0059:   // log_probs: input_len x batch_size x num_labels
0060:   // targets [int64]: batch_size x target_length OR sum(target_lengths)
```
- **EN**: Lines 31-60 mainly cover header inclusion, comments/documentation, namespace structuring. Notable symbols: targets, get_target_prime, ctc_loss_allocate_outputs, sum.
- **CN**: 第 31-60 行主要涉及头文件包含、注释或说明、命名空间组织。 值得关注的符号包括：targets, get_target_prime, ctc_loss_allocate_outputs, sum。

### Lines 61-90 / 第 61-90 行
```cpp
0061: 
0062:   CheckedFrom c = "ctc_loss_allocate_outputs";
0063:   auto log_probs_arg = TensorArg(log_probs, "log_probs", 1);
0064:   auto targets_arg = TensorArg(targets, "targets", 2);
0065:   checkScalarType(c, targets_arg, target_scalar_type);
0066:   checkDim(c, log_probs_arg, 3);
0067:   checkDimRange(c, targets_arg, 1, 3);
0068: 
0069:   int64_t batch_size = log_probs.size(1);
0070:   int64_t num_labels = log_probs.size(2);
0071:   TORCH_CHECK((0 <= BLANK) && (BLANK < num_labels), "blank must be in label range");
0072:   TORCH_CHECK((int64_t) input_lengths.size() == batch_size, "input_lengths must be of size batch_size");
0073:   TORCH_CHECK((int64_t) target_lengths.size() == batch_size, "target_lengths must be of size batch_size");
0074: 
0075:   size_t tg_target_stride = 0;
0076:   int64_t max_target_length = 0;
0077:   std::vector<int64_t> tg_batch_offsets(batch_size);
0078:   if (targets.dim() == 1) { // concatenated targets
0079:     int64_t pos = 0;
0080:     for (const auto i : c10::irange(batch_size)) {
0081:       TORCH_CHECK(target_lengths[i] >= 0,
0082:                   "Expected target_lengths to have value at least ", 0, ", but got value ", target_lengths[i],
0083:                   " (while checking arguments for ", c, ")");
0084:       tg_batch_offsets[i] = pos;
0085:       pos += target_lengths[i];
0086:       if (max_target_length < target_lengths[i])
0087:          max_target_length = target_lengths[i];
0088:     }
0089:     tg_target_stride = targets.stride(0);
0090:     checkSize(c, targets_arg, 0, pos);
```
- **EN**: Lines 61-90 mainly cover state/variable declarations, macro-based glue, control-flow checks. Notable symbols: TensorArg, checkScalarType, checkDim, checkDimRange.
- **CN**: 第 61-90 行主要涉及变量/别名声明、宏定义或宏调用、控制流逻辑。 值得关注的符号包括：TensorArg, checkScalarType, checkDim, checkDimRange。

### Lines 91-120 / 第 91-120 行
```cpp
0091:   }
0092:   else { // batch x max_target_length
0093:     // dim is 2
0094:     int64_t tg_batch_stride = targets.stride(0);
0095:     for (const auto i : c10::irange(batch_size)) {
0096:       TORCH_CHECK(target_lengths[i] >= 0,
0097:                   "Expected target_lengths to have value at least ", 0, ", but got value ", target_lengths[i],
0098:                   " (while checking arguments for ", c, ")");
0099:       tg_batch_offsets[i] = i * tg_batch_stride;
0100:       if (max_target_length < target_lengths[i])
0101:         max_target_length = target_lengths[i];
0102:     }
0103:     tg_target_stride = targets.stride(1);
0104:     checkSize(c, targets_arg, 0, batch_size);
0105:     TORCH_CHECK(targets.size(1) >= max_target_length,
0106:              "Expected tensor to have size at least ", max_target_length, " at dimension 1, but got size ", targets.size(1), " for ", targets_arg,
0107:              " (while checking arguments for ", c, ")");
0108:   }
0109:   int64_t max_input_length = log_probs.size(0);
0110:   for (const auto b : c10::irange(batch_size)) {
0111:     TORCH_CHECK(input_lengths[b] >= 0,
0112:              "Expected input_lengths to have value at least ", 0, ", but got value ", input_lengths[b],
0113:              " (while checking arguments for ", c, ")");
0114:     TORCH_CHECK(input_lengths[b] <= max_input_length,
0115:              "Expected input_lengths to have value at most ", max_input_length, ", but got value ", input_lengths[b],
0116:              " (while checking arguments for ", c, ")");
0117:   }
0118: 
0119:   Tensor log_alpha = at::empty({batch_size, log_probs.size(0), 2*max_target_length+1}, log_probs.options());
0120:   Tensor neg_log_likelihood = at::empty({batch_size}, log_probs.options());
```
- **EN**: Lines 91-120 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: stride, irange, TORCH_CHECK, checkSize.
- **CN**: 第 91-120 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：stride, irange, TORCH_CHECK, checkSize。

### Lines 121-150 / 第 121-150 行
```cpp
0121: 
0122:   return std::make_tuple(neg_log_likelihood, log_alpha, tg_target_stride, tg_batch_offsets);
0123: }
0124: 
0125: // This kernel is a relatively straightforward implementation of the alpha calculation in the forward backward algorithm (section 4.1).
0126: // A (minor) twist is that we are using log-calculations to enhance numerical stability (log_probs and log_alpha).
0127: // The function returns the loss and the alphas, the alphas are kept for the backward step. The wrapper (ctc_loss below) hides
0128: // the alphas from the user by only returning the loss.
0129: template<typename scalar_t, ScalarType target_scalar_type>
0130: std::tuple<Tensor, Tensor> ctc_loss_cpu_template(const Tensor& log_probs, const Tensor& targets, IntArrayRef input_lengths, IntArrayRef target_lengths, int64_t BLANK) {
0131:   TORCH_CHECK(log_probs.numel() > 0, "log_probs tensor must not be empty");
0132:   // log_probs: input_len x batch_size x num_labels
0133:   // targets [int64]: batch_size x target_length OR sum(target_lengths)
0134:   constexpr scalar_t neginf = -std::numeric_limits<scalar_t>::infinity();
0135:   using target_t = typename std::conditional_t<target_scalar_type == kInt, int, int64_t>;
0136: 
0137:   Tensor neg_log_likelihood, log_alpha;
0138:   size_t tg_target_stride;
0139:   std::vector<int64_t> tg_batch_offsets;
0140: 
0141:   if (targets.scalar_type() == kLong) {
0142:     std::tie(neg_log_likelihood, log_alpha,tg_target_stride, tg_batch_offsets) =
0143:         ctc_loss_allocate_outputs<scalar_t, kLong>(
0144:             log_probs, targets, input_lengths, target_lengths, BLANK);
0145:   } else {
0146:     std::tie(neg_log_likelihood, log_alpha, tg_target_stride, tg_batch_offsets) =
0147:         ctc_loss_allocate_outputs<scalar_t, kInt>(
0148:             log_probs, targets, input_lengths, target_lengths, BLANK);
0149:   }
0150: 
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, comments/documentation, expressions/calls. Notable symbols: make_tuple, algorithm, A, stability.
- **CN**: 第 121-150 行主要涉及变量/别名声明、注释或说明、表达式或调用。 值得关注的符号包括：make_tuple, algorithm, A, stability。

### Lines 151-180 / 第 151-180 行
```cpp
0151:   int64_t batch_size = log_probs.size(1);
0152:   auto lpp  = log_probs.permute({1,0,2});
0153:   auto log_probs_a_global = lpp.accessor<const scalar_t, 3>();
0154:   auto log_alpha_a_global = log_alpha.accessor<scalar_t, 3>();
0155:   auto targets_data = targets.const_data_ptr<target_t>();
0156:   auto neg_log_likelihood_a = neg_log_likelihood.accessor<scalar_t, 1>();
0157: 
0158:   // alpha calculation for the first row, the three equations for alpha_1 above eq (6)
0159:   // first the default
0160:   log_alpha.narrow(1, 0, 1).fill_(neginf);
0161:   at::parallel_for(0, batch_size, 0, [&](int64_t start, int64_t end) {
0162:     for (const auto b : c10::irange(start, end)) {
0163:       int64_t input_length = input_lengths[b];
0164:       int64_t target_length = target_lengths[b];
0165:       auto log_probs_a = log_probs_a_global[b];
0166:       auto log_alpha_a = log_alpha_a_global[b];
0167:       int64_t tg_batch_offset = tg_batch_offsets[b];
0168: 
0169:       if (input_length == 0) {
0170:         scalar_t log_likelihood = target_length == 0 ? 0 : neginf;
0171:         neg_log_likelihood_a[b] = -log_likelihood;
0172:         continue;
0173:       }
0174: 
0175:       // the first two items of alpha_t above eq (6)
0176:       log_alpha_a[0][0] = log_probs_a[0][BLANK];
0177:       if (target_length > 0)
0178:         log_alpha_a[0][1] = log_probs_a[0][get_target_prime(targets_data, tg_batch_offset, tg_target_stride, 1, BLANK)];
0179: 
0180:       // now the loop over the inputs
```
- **EN**: Lines 151-180 mainly cover state/variable declarations, comments/documentation, control-flow checks. Notable symbols: size, permute, eq, narrow.
- **CN**: 第 151-180 行主要涉及变量/别名声明、注释或说明、控制流逻辑。 值得关注的符号包括：size, permute, eq, narrow。

### Lines 181-210 / 第 181-210 行
```cpp
0181:       for (const auto t : c10::irange(1, input_length)) {
0182:         for (const auto s : c10::irange(2*target_length+1)) {
0183:           auto current_target_prime = get_target_prime(targets_data, tg_batch_offset, tg_target_stride, s, BLANK);
0184:           // this loop over s could be parallel/vectorized, too, but the required items are one index apart
0185:           // alternatively, one might consider moving s to the outer loop to cache current_target_prime more (but then it needs to be descending)
0186:           // for the cuda implementation, that gave a speed boost.
0187:           // This is eq (6) and (7), la1,2,3 are the three summands. We keep track of the maximum for the logsumexp calculation.
0188: 
0189:           scalar_t la1 = log_alpha_a[t-1][s];
0190:           scalar_t lamax = la1;
0191:           scalar_t la2, la3;
0192:           if (s > 0) {
0193:             la2 = log_alpha_a[t-1][s-1];
0194:             if (la2 > lamax)
0195:               lamax = la2;
0196:           } else {
0197:             la2 = neginf;
0198:           }
0199:           if ((s > 1) && (get_target_prime(targets_data, tg_batch_offset, tg_target_stride, s-2, BLANK) !=
0200:                           current_target_prime)) {
0201:             la3 = log_alpha_a[t-1][s-2];
0202:             if (la3 > lamax)
0203:               lamax = la3;
0204:           } else {
0205:             la3 = neginf;
0206:           }
0207:           if (lamax == neginf) // cannot do neginf-neginf
0208:             lamax = 0;
0209:           // this is the assignment of eq (6)
0210:           log_alpha_a[t][s] = std::log(std::exp(la1-lamax)+std::exp(la2-lamax)+std::exp(la3-lamax))+lamax + log_probs_a[t][current_target_prime];
```
- **EN**: Lines 181-210 mainly cover state/variable declarations, control-flow checks, comments/documentation. Notable symbols: irange, get_target_prime, more, eq.
- **CN**: 第 181-210 行主要涉及变量/别名声明、控制流逻辑、注释或说明。 值得关注的符号包括：irange, get_target_prime, more, eq。

### Lines 211-240 / 第 211-240 行
```cpp
0211:         }
0212:       }
0213:       // the likelihood is the sum of the last two alphas, eq (8), the loss is the negative log likelihood
0214:       if (target_length == 0) {
0215:         // if the target is empty then there is no preceding BLANK state and hence there is no path to merge
0216:         neg_log_likelihood_a[b] = -log_alpha_a[input_length-1][0];
0217:       } else {
0218:         scalar_t l1 = log_alpha_a[input_length-1][target_length*2];
0219:         scalar_t l2 = log_alpha_a[input_length-1][target_length*2-1];
0220:         scalar_t m = std::max(l1, l2);
0221:         m = ((m == neginf) ? 0 : m);
0222:         scalar_t log_likelihood = std::log(std::exp(l1-m)+std::exp(l2-m))+m;
0223:         neg_log_likelihood_a[b] = -log_likelihood;
0224:       }
0225:     }
0226:   });
0227: 
0228:   return std::make_tuple(neg_log_likelihood, log_alpha);
0229: }
0230: 
0231: // This is the backward. It consists of two phases:
0232: // a) computing the beta analogous to the alphas in the forward (backward half of the forward-backward algorithm) (eq (10) and (11))
0233: // b) collecting the per-activation characters for all s and wrapping the gradient (eq (16), the collection is the sum)
0234: template<typename scalar_t, ScalarType target_scalar_type>
0235: Tensor ctc_loss_backward_cpu_template(const Tensor& grad_out, const Tensor& log_probs, const Tensor& targets, IntArrayRef input_lengths, IntArrayRef target_lengths,
0236:                                       const Tensor& neg_log_likelihood, const Tensor& log_alpha, int64_t BLANK, bool zero_infinity) {
0237:   constexpr scalar_t neginf = -std::numeric_limits<scalar_t>::infinity();
0238:   using target_t = typename std::conditional_t<target_scalar_type == kInt, int, int64_t>;
0239:   int64_t max_input_length = log_probs.size(0);
0240:   int64_t batch_size = log_probs.size(1);
```
- **EN**: Lines 211-240 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: eq, max, log, exp.
- **CN**: 第 211-240 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：eq, max, log, exp。

### Lines 241-270 / 第 241-270 行
```cpp
0241:   int64_t num_labels = log_probs.size(2);
0242:   Tensor grad = at::full_like(log_probs, neginf, LEGACY_CONTIGUOUS_MEMORY_FORMAT); // at this point, this is log of empty sum
0243: 
0244:   // The admin bits. We don't do much checking and assume that the forward did.
0245:   int64_t tg_target_stride = 0;
0246:   int64_t max_target_length = 0;
0247:   std::vector<int64_t> tg_batch_offsets(batch_size);
0248: 
0249:   if (targets.dim() == 1) { // concatenated targets
0250:     int64_t pos = 0;
0251:     max_target_length = 0;
0252:     for (const auto i : c10::irange(batch_size)) {
0253:       tg_batch_offsets[i] = pos;
0254:       pos += target_lengths[i];
0255:       if (max_target_length < target_lengths[i])
0256:         max_target_length = target_lengths[i];
0257:     }
0258:     tg_target_stride = targets.stride(0);
0259:   }
0260:   else { // batch x max_target_length
0261:     // dim is 2
0262:     int64_t tg_batch_stride = targets.stride(0);
0263:     for (const auto i : c10::irange(batch_size)) {
0264:       tg_batch_offsets[i] = i * tg_batch_stride;
0265:     }
0266:     tg_target_stride = targets.stride(1);
0267:     max_target_length = targets.size(1);
0268:   }
0269: 
0270:   Tensor log_beta = at::empty_like(log_alpha, LEGACY_CONTIGUOUS_MEMORY_FORMAT);  // could be optimized to use only 2 rows
```
- **EN**: Lines 241-270 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: size, full_like, tg_batch_offsets, dim.
- **CN**: 第 241-270 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：size, full_like, tg_batch_offsets, dim。

### Lines 271-300 / 第 271-300 行
```cpp
0271:   auto lpp  = log_probs.permute({1,0,2});
0272:   auto log_probs_a_global = lpp.accessor<const scalar_t, 3>();
0273:   auto log_alpha_a_global = log_alpha.accessor<const scalar_t, 3>();
0274:   auto log_beta_a_global = log_beta.accessor<scalar_t, 3>();
0275:   auto gp = grad.permute({1,0,2});
0276:   auto grad_a_global = gp.accessor<scalar_t, 3>();
0277:   auto targets_data = targets.const_data_ptr<target_t>();
0278:   auto grad_out_a = grad_out.accessor<const scalar_t, 1>();
0279: 
0280:   auto create_fill_iterator = [](const Tensor& tensor, IntArrayRef squash_dims) {
0281:     return TensorIteratorConfig()
0282:         .set_check_mem_overlap(false)  // Fill is idempotent, so overlap is okay
0283:         .check_all_same_dtype(false)
0284:         .add_output(tensor)
0285:         .resize_outputs(false)
0286:         .declare_static_shape(tensor.sizes(), squash_dims)
0287:         .build();
0288:   };
0289:   const auto fill_iter = create_fill_iterator(grad, /*squash_dims=*/1);
0290:   const auto fill_1d_iter = create_fill_iterator(grad, /*squash_dims=*/{0, 1});
0291:   const auto fill_log_beta_1d_iter = create_fill_iterator(log_beta, /*squash_dims=*/{0, 1});
0292: 
0293:   at::parallel_for(0, batch_size, 0, [&](int64_t start, int64_t end) {
0294:     TensorIterator fill_iter_local(fill_iter);
0295:     TensorIterator fill_1d_iter_local(fill_1d_iter);
0296:     TensorIterator fill_log_beta_1d_iter_local(fill_log_beta_1d_iter);
0297: 
0298:     for (const auto b : c10::irange(start, end)) {
0299:       scalar_t nll = neg_log_likelihood.accessor<scalar_t, 1>()[b];
0300:       auto grad_a = grad_a_global[b];
```
- **EN**: Lines 271-300 mainly cover state/variable declarations, function signatures/definitions, return paths. Notable symbols: permute, TensorIteratorConfig, set_check_mem_overlap, check_all_same_dtype.
- **CN**: 第 271-300 行主要涉及变量/别名声明、函数签名或实现、返回路径。 值得关注的符号包括：permute, TensorIteratorConfig, set_check_mem_overlap, check_all_same_dtype。

### Lines 301-330 / 第 301-330 行
```cpp
0301:       if (zero_infinity && nll == std::numeric_limits<scalar_t>::infinity()) {
0302:         // grad_batch.zero_();
0303:         fill_iter_local.unsafe_replace_operand(0, grad_a.data());
0304:         fill_stub(kCPU, fill_iter_local, 0);
0305:         continue;
0306:       }
0307: 
0308:       auto log_probs_a = log_probs_a_global[b];
0309:       auto log_alpha_a = log_alpha_a_global[b];
0310:       auto log_beta_a = log_beta_a_global[b];
0311:       int64_t input_length = input_lengths[b];
0312:       int64_t target_length = target_lengths[b];
0313:       int64_t tg_batch_offset = tg_batch_offsets[b];
0314: 
0315:       // the initialization of beta before eq (10)
0316:       // here we do the fill for each batch item separately, as the input lengths will differ, so the t in which
0317:       // we start varies
0318:       if (input_length > 0) {
0319:         // log_beta.select(0, b).select(1, input_length-1).fill_(neginf);
0320:         fill_log_beta_1d_iter_local.unsafe_replace_operand(
0321:             0, log_beta_a[input_length - 1].data());
0322:         fill_stub(kCPU, fill_log_beta_1d_iter_local, neginf);
0323: 
0324:         log_beta_a[input_length-1][2*target_length] = log_probs_a[input_length-1][BLANK];
0325:         grad_a[input_length-1][BLANK] = log_alpha_a[input_length-1][2*target_length] + log_beta_a[input_length-1][2*target_length];
0326: 
0327:         if (target_length > 0) {
0328:           auto current_target_prime = get_target_prime(targets_data, tg_batch_offset, tg_target_stride, 2*target_length-1, BLANK);
0329:           log_beta_a[input_length-1][2*target_length-1] = log_probs_a[input_length-1][current_target_prime];
0330: 
```
- **EN**: Lines 301-330 mainly cover state/variable declarations, comments/documentation, control-flow checks. Notable symbols: infinity, zero_, unsafe_replace_operand, data.
- **CN**: 第 301-330 行主要涉及变量/别名声明、注释或说明、控制流逻辑。 值得关注的符号包括：infinity, zero_, unsafe_replace_operand, data。

### Lines 331-360 / 第 331-360 行
```cpp
0331:           // the first two are a blank and a non-blank, so we know they are different and we don't need to do log+
0332:           grad_a[input_length-1][current_target_prime] = log_alpha_a[input_length-1][2*target_length-1] + log_beta_a[input_length-1][2*target_length-1];
0333:         }
0334:       }
0335: 
0336:       // now loop applying eq (10) / (11)
0337:       for (int64_t t=input_length-2; t>=0; t--) {
0338:         // this loop over s could be parallel/vectorized and doesn't really need to be descending...
0339:         // alternatively, one might consider moving s to the outer loop to cache current_target_prime more (but then it needs to be descending)
0340:         // for the cuda implementation, that gave a speed boost.
0341:         for (int64_t s=2*target_length; s>=0; s--) {
0342:           scalar_t lb1 = log_beta_a[t+1][s];
0343:           scalar_t lbmax = lb1;
0344:           scalar_t lb2, lb3;
0345:           auto current_target_prime = get_target_prime(targets_data, tg_batch_offset, tg_target_stride, s, BLANK);
0346:           if (s < 2*target_length) {
0347:             lb2 = log_beta_a[t+1][s+1];
0348:             if (lb2 > lbmax)
0349:               lbmax = lb2;
0350:           } else {
0351:             lb2 = neginf;
0352:           }
0353:           if ((s < 2*target_length-1) && (get_target_prime(targets_data, tg_batch_offset, tg_target_stride, s+2, BLANK) !=
0354:                                           current_target_prime)) {
0355:             lb3 = log_beta_a[t+1][s+2];
0356:             if (lb3 > lbmax)
0357:               lbmax = lb3;
0358:           } else {
0359:             lb3 = neginf;
0360:           }
```
- **EN**: Lines 331-360 mainly cover state/variable declarations, control-flow checks, comments/documentation. Notable symbols: eq, more, get_target_prime.
- **CN**: 第 331-360 行主要涉及变量/别名声明、控制流逻辑、注释或说明。 值得关注的符号包括：eq, more, get_target_prime。

### Lines 361-390 / 第 361-390 行
```cpp
0361:           if (lbmax == neginf)
0362:             lbmax = 0;
0363: 
0364:           log_beta_a[t][s] = std::log(std::exp(lb1-lbmax)+std::exp(lb2-lbmax)+std::exp(lb3-lbmax))+lbmax + log_probs_a[t][current_target_prime];
0365:           // one might check whether one can vectorize this better when done after the t-loop...
0366:           // now that we have beta, we fill in the sum of alpha*beta in eq (16)
0367:           // in contrast to the cuda implementation, we only parallelize over the batch, so we don't have a concurrency
0368:           // issue (several s can map to the same target character)
0369:           // collected[b, t, target'[s]] "log+=" log_alpha[t, s]+log_beta[t, s]
0370:           scalar_t log_alpha_beta =  log_alpha_a[t][s] + log_beta_a[t][s];
0371:           scalar_t &lcab = grad_a[t][current_target_prime];
0372:           if (lcab == neginf) {
0373:             lcab = log_alpha_beta;
0374:           } else {
0375:             scalar_t max = std::max(lcab, log_alpha_beta);
0376:             lcab = std::log(std::exp(lcab-max)+std::exp(log_alpha_beta-max))+max;
0377:           }
0378:         }
0379:       }
0380: 
0381:       // now grad has the sum of eq (16)
0382:       // now we wrap up the calculation by adding in the remaining items of eq (16)
0383:       // this could be a great target for further vectorization.
0384:       // grad is the output gradient, nll is the loss. Note that the likelihood -nll is the Z of eq (16)
0385:       scalar_t gr = grad_out_a[b];
0386:       for (const auto t : c10::irange(input_length)) { // or go for the full thing?
0387:         for (const auto c : c10::irange(num_labels)) {
0388:           scalar_t& res = grad_a[t][c];
0389:           scalar_t lp = log_probs_a[t][c];
0390:           res = (std::exp(lp)-std::exp(res + nll - lp)) * gr;
```
- **EN**: Lines 361-390 mainly cover state/variable declarations, comments/documentation, control-flow checks. Notable symbols: log, exp, eq, issue.
- **CN**: 第 361-390 行主要涉及变量/别名声明、注释或说明、控制流逻辑。 值得关注的符号包括：log, exp, eq, issue。

### Lines 391-420 / 第 391-420 行
```cpp
0391:         }
0392:       }
0393: 
0394:       // zero the remainder
0395:       for (auto l : c10::irange(input_length, max_input_length)) {
0396:         // grad_batch.select(0, l).zero_();
0397:         fill_1d_iter_local.unsafe_replace_operand(0, grad_a[l].data());
0398:         fill_stub(kCPU, fill_1d_iter_local, 0);
0399:       }
0400:     }
0401:   });
0402:   return grad;
0403: }
0404: 
0405: } // namespace
0406: 
0407: std::tuple<Tensor, Tensor> ctc_loss_meta(const Tensor& log_probs, const Tensor& targets, IntArrayRef input_lengths, IntArrayRef target_lengths, int64_t BLANK, bool zero_infinity) {
0408:   (void)zero_infinity; // only used for backwards
0409:   return AT_DISPATCH_FLOATING_TYPES(
0410:       log_probs.scalar_type(), "ctc_loss_meta", [&] {
0411:         Tensor neg_log_likelihood, log_alpha;
0412:         if (targets.scalar_type() == kLong) {
0413:           std::tie(neg_log_likelihood, log_alpha, std::ignore, std::ignore) =  ctc_loss_allocate_outputs<scalar_t, kLong>(
0414:               log_probs, targets, input_lengths, target_lengths, BLANK);
0415:         } else {
0416:           std::tie(neg_log_likelihood, log_alpha, std::ignore, std::ignore) = ctc_loss_allocate_outputs<scalar_t, kInt>(
0417:               log_probs, targets, input_lengths, target_lengths, BLANK);
0418:         }
0419:         return std::make_tuple(neg_log_likelihood, log_alpha);
0420:       });
```
- **EN**: Lines 391-420 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: irange, select, zero_, unsafe_replace_operand.
- **CN**: 第 391-420 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：irange, select, zero_, unsafe_replace_operand。

### Lines 421-450 / 第 421-450 行
```cpp
0421: }
0422: 
0423: std::tuple<Tensor, Tensor> ctc_loss_cpu(const Tensor& log_probs, const Tensor& targets, IntArrayRef input_lengths, IntArrayRef target_lengths, int64_t BLANK, bool zero_infinity) {
0424:   (void)zero_infinity; // only used for backwards
0425:   return AT_DISPATCH_FLOATING_TYPES(log_probs.scalar_type(), "ctc_loss_cpu", [&] {
0426:       if (targets.scalar_type() == kLong) {
0427:         return ctc_loss_cpu_template<scalar_t, kLong>(log_probs, targets, input_lengths, target_lengths, BLANK);
0428:       } else {
0429:         return ctc_loss_cpu_template<scalar_t, kInt>(log_probs, targets, input_lengths, target_lengths, BLANK);
0430:       }
0431:   });
0432: }
0433: 
0434: 
0435: std::tuple<Tensor, Tensor> ctc_loss_tensor(const Tensor& log_probs, const Tensor& targets, const Tensor& input_lengths, const Tensor& target_lengths, int64_t BLANK, bool zero_infinity) {
0436:   TORCH_CHECK(isIntegralType(input_lengths.scalar_type(), /*includeBool=*/false), "input_lengths must be integral");
0437:   TORCH_CHECK(isIntegralType(target_lengths.scalar_type(), /*includeBool=*/false), "target_lengths must be integral");
0438: 
0439:   Tensor ilc = input_lengths.to(Device(at::kCPU), at::kLong).contiguous();
0440:   Tensor tlc = target_lengths.to(Device(at::kCPU), at::kLong).contiguous();
0441:   IntArrayRef il(ilc.const_data_ptr<int64_t>(), ilc.numel());
0442:   IntArrayRef tl(tlc.const_data_ptr<int64_t>(), tlc.numel());
0443: 
0444:   return at::_ctc_loss(log_probs, targets, il, tl, BLANK, zero_infinity);
0445: }
0446: 
0447: Tensor ctc_loss_backward_cpu(const Tensor& grad, const Tensor& log_probs, const Tensor& targets, IntArrayRef input_lengths, IntArrayRef target_lengths,
0448:                              const Tensor& neg_log_likelihood, const Tensor& log_alpha, int64_t BLANK, bool zero_infinity) {
0449:   return AT_DISPATCH_FLOATING_TYPES(log_probs.scalar_type(), "ctc_loss_backward_cpu", [&] {
0450:       if (targets.scalar_type() == kLong) {
```
- **EN**: Lines 421-450 mainly cover expressions/calls, function signatures/definitions, return paths. Notable symbols: ctc_loss_cpu, AT_DISPATCH_FLOATING_TYPES, scalar_type, ctc_loss_tensor.
- **CN**: 第 421-450 行主要涉及表达式或调用、函数签名或实现、返回路径。 值得关注的符号包括：ctc_loss_cpu, AT_DISPATCH_FLOATING_TYPES, scalar_type, ctc_loss_tensor。

### Lines 451-480 / 第 451-480 行
```cpp
0451:         return ctc_loss_backward_cpu_template<scalar_t,kLong>(grad, log_probs, targets, input_lengths, target_lengths, neg_log_likelihood, log_alpha, BLANK, zero_infinity);
0452:       } else {
0453:         return ctc_loss_backward_cpu_template<scalar_t,kInt>(grad, log_probs, targets, input_lengths, target_lengths, neg_log_likelihood, log_alpha, BLANK, zero_infinity);
0454:       }
0455:   });
0456: }
0457: 
0458: Tensor ctc_loss_backward_tensor(
0459:     const Tensor& grad,
0460:     const Tensor& log_probs,
0461:     const Tensor& targets,
0462:     const Tensor& input_lengths,
0463:     const Tensor& target_lengths,
0464:     const Tensor& neg_log_likelihood,
0465:     const Tensor& log_alpha,
0466:     int64_t BLANK,
0467:     bool zero_infinity) {
0468:   TORCH_CHECK(
0469:       isIntegralType(input_lengths.scalar_type(), /*includeBool=*/false),
0470:       "input_lengths must be integral");
0471:   TORCH_CHECK(isIntegralType(target_lengths.scalar_type(), /*includeBool=*/false), "target_lengths must be integral");
0472: 
0473:   Tensor ilc = input_lengths.to(Device(at::kCPU), at::kLong).contiguous();
0474:   Tensor tlc = target_lengths.to(Device(at::kCPU), at::kLong).contiguous();
0475:   IntArrayRef il(ilc.const_data_ptr<int64_t>(), ilc.numel());
0476:   IntArrayRef tl(tlc.const_data_ptr<int64_t>(), tlc.numel());
0477:   return at::_ctc_loss_backward(grad, log_probs, targets, il, tl, neg_log_likelihood, log_alpha, BLANK, zero_infinity);
0478: }
0479: 
0480: namespace {
```
- **EN**: Lines 451-480 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: ctc_loss_backward_tensor, TORCH_CHECK, isIntegralType, scalar_type.
- **CN**: 第 451-480 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：ctc_loss_backward_tensor, TORCH_CHECK, isIntegralType, scalar_type。

### Lines 481-510 / 第 481-510 行
```cpp
0481: 
0482: Tensor get_clamped_target_length(
0483:     IntArrayRef target_lengths,
0484:     const TensorOptions& options) {
0485:   return at::tensor(target_lengths, options).clamp_min(1);
0486: }
0487: 
0488: Tensor get_clamped_target_length(
0489:     const Tensor & target_lengths,
0490:     const TensorOptions& options) {
0491:   return target_lengths.clamp_min(1);
0492: }
0493: 
0494: // this wrapper function dispatches to the native and cudnn/miopen implementations and hides the alpha/grad from the user (by just returning the loss)
0495: // the gradient is implemented for _cudnn_ctc_loss, miopen_ctc_loss (just in derivatives.yaml) and _ctc_loss and this function has automatic gradients
0496: // it also handles the reduction if desired
0497: template <typename LengthsType>
0498: Tensor ctc_loss_impl(const Tensor& log_probs_, const Tensor& targets, LengthsType input_lengths, LengthsType target_lengths, int64_t BLANK, int64_t reduction, bool zero_infinity) {
0499:   auto is_batched = log_probs_.dim() == 3;
0500:   Tensor log_probs = is_batched ? log_probs_ : log_probs_.unsqueeze(1);
0501: 
0502:   Tensor res;
0503: 
0504:   // cuDNN CTC Loss (returns false on non-CUDA builds)
0505:   bool use_cudnn =
0506:       (log_probs.device().type() == at::kCUDA) &&
0507:       at::_use_cudnn_ctc_loss(
0508:           log_probs, targets, input_lengths, target_lengths, BLANK);
0509: 
0510:   // MIOpen CTC Loss (returns false on non-ROCm builds)
```
- **EN**: Lines 481-510 mainly cover expressions/calls, comments/documentation, function signatures/definitions. Notable symbols: get_clamped_target_length, tensor, clamp_min, user.
- **CN**: 第 481-510 行主要涉及表达式或调用、注释或说明、函数签名或实现。 值得关注的符号包括：get_clamped_target_length, tensor, clamp_min, user。

### Lines 511-540 / 第 511-540 行
```cpp
0511:   bool use_miopen = false;
0512:   Tensor targets_cpu;
0513:   if (log_probs.device().type() == at::kCUDA) {
0514:     targets_cpu = targets.device().type() == at::kCPU
0515:         ? targets.to(at::kInt)
0516:         : targets.to(Device(at::kCPU), at::kInt);
0517:     use_miopen = at::_use_miopen_ctc_loss(log_probs, targets_cpu, input_lengths, target_lengths, BLANK);
0518:   }
0519: 
0520:   if (use_cudnn) {
0521:     // non-deterministic ctc loss on cudnn disabled due to inconsistent results
0522:     // see: https://github.com/pytorch/pytorch/issues/21680
0523:     res = std::get<0>(at::_cudnn_ctc_loss(log_probs, targets, input_lengths, target_lengths, BLANK, /*deterministic=*/true, zero_infinity));
0524:   } else if (use_miopen) {
0525:     // MIOpen CTC Loss only supports deterministic algorithm
0526:     res = std::get<0>(at::miopen_ctc_loss(log_probs, targets_cpu, input_lengths, target_lengths, BLANK, /*deterministic=*/true, zero_infinity));
0527:   } else {
0528:     // if the targets are on CPU (which you need for cuDNN/MIOpen), move them to
0529:     // GPU as a service for the user
0530:     res = std::get<0>(at::_ctc_loss(
0531:         log_probs,
0532:         targets.to(log_probs.device(), kLong),
0533:         input_lengths,
0534:         target_lengths,
0535:         BLANK,
0536:         zero_infinity));
0537:     if (zero_infinity) {
0538:       res = at::where(res == Scalar(std::numeric_limits<double>::infinity()), at::zeros({}, res.options()), res);
0539:     }
0540:   }
```
- **EN**: Lines 511-540 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: device, type, to, Device.
- **CN**: 第 511-540 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：device, type, to, Device。

### Lines 541-570 / 第 541-570 行
```cpp
0541:   if (reduction == at::Reduction::Mean) {
0542:     auto target_lengths_t = get_clamped_target_length(target_lengths, res.options());
0543:     return (res / target_lengths_t).mean();
0544:   } else if (reduction == at::Reduction::Sum) {
0545:     return res.sum();
0546:   }
0547:   return is_batched ? std::move(res) : res.squeeze(0);
0548: }
0549: 
0550: } // namespace
0551: 
0552: Tensor ctc_loss(const Tensor& log_probs_, const Tensor& targets, IntArrayRef input_lengths, IntArrayRef target_lengths, int64_t BLANK, int64_t reduction, bool zero_infinity) {
0553:   return ctc_loss_impl(log_probs_, targets, input_lengths, target_lengths, BLANK, reduction, zero_infinity);
0554: }
0555: 
0556: // Convenience function accepting Tensors
0557: Tensor ctc_loss(const Tensor& log_probs, const Tensor& targets, const Tensor& input_lengths, const Tensor& target_lengths, int64_t BLANK, int64_t reduction, bool zero_infinity) {
0558:   // we don't want to convert to IntArrayRef if we can dispatch to cuDNN/MIOpen (this allows graph-capturable ctc_loss)
0559:   // cuDNN CTC Loss (returns false on non-CUDA builds)
0560:   bool use_cudnn =
0561:       (log_probs.device().type() == at::kCUDA) &&
0562:       at::_use_cudnn_ctc_loss(
0563:           log_probs, targets, input_lengths, target_lengths, BLANK);
0564:   // MIOpen CTC Loss (returns false on non-ROCm builds)
0565:   bool use_miopen = false;
0566:   if (log_probs.device().type() == at::kCUDA) {
0567:     Tensor targets_check = targets.device().type() == at::kCPU
0568:         ? targets.to(at::kInt)
0569:         : targets.to(Device(at::kCPU), at::kInt);
0570:     use_miopen = at::_use_miopen_ctc_loss(
```
- **EN**: Lines 541-570 mainly cover function signatures/definitions, expressions/calls, state/variable declarations. Notable symbols: get_clamped_target_length, options, mean, sum.
- **CN**: 第 541-570 行主要涉及函数签名或实现、表达式或调用、变量/别名声明。 值得关注的符号包括：get_clamped_target_length, options, mean, sum。

### Lines 571-591 / 第 571-591 行
```cpp
0571:         log_probs, targets_check, input_lengths, target_lengths, BLANK);
0572:   }
0573:   bool use_accelerated = use_cudnn || use_miopen;
0574:   if (at::areAnyTensorSubclassLike(
0575:           {log_probs, targets, input_lengths, target_lengths}) || use_accelerated) {
0576:     // Composite Compliant path for TensorSubclasses
0577:     return ctc_loss_impl(log_probs, targets, input_lengths, target_lengths, BLANK, reduction, zero_infinity);
0578:   }
0579:   // Fast path (which accesses data_ptr) and less operator dispatches for
0580:   // regular tensors
0581:   TORCH_CHECK(isIntegralType(input_lengths.scalar_type(), /*includeBool=*/false), "input_lengths must be integral");
0582:   TORCH_CHECK(isIntegralType(target_lengths.scalar_type(), /*includeBool=*/false), "target_lengths must be integral");
0583: 
0584:   Tensor ilc = input_lengths.to(Device(at::kCPU), at::kLong).contiguous();
0585:   Tensor tlc = target_lengths.to(Device(at::kCPU), at::kLong).contiguous();
0586:   IntArrayRef il(ilc.const_data_ptr<int64_t>(), ilc.numel());
0587:   IntArrayRef tl(tlc.const_data_ptr<int64_t>(), tlc.numel());
0588:   return at::native::ctc_loss(log_probs, targets, il, tl, BLANK, reduction, zero_infinity);
0589: }
0590: 
0591: } // at::native
```
- **EN**: Lines 571-591 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: areAnyTensorSubclassLike, ctc_loss_impl, path, TORCH_CHECK.
- **CN**: 第 571-591 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：areAnyTensorSubclassLike, ctc_loss_impl, path, TORCH_CHECK。

## Key Concepts / 关键概念
- **EN**: TensorIterator-driven traversal  
  **CN**: 基于 TensorIterator 的遍历
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Parallel loop scheduling  
  **CN**: 并行循环调度
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/Parallel.h>`, `<ATen/TensorIterator.h>`, `<ATen/TensorOperators.h>`, `<ATen/native/Fill.h>`, `<c10/util/irange.h>`, `<ATen/TensorSubclassLikeUtils.h>`, `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `AT_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
