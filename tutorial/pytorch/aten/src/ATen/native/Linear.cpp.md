# Linear.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/Linear.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Linear.
- **Purpose (CN)**: 实现或声明与 线性 相关的 ATen 原生逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/core/Tensor.h>
0003: #include <ATen/native/Resize.h>
0004: #include <ATen/native/xnnpack/Engine.h>
0005: #include <ATen/WrapDimUtilsMulti.h>
0006: #include <ATen/TensorOperators.h>
0007: #include <c10/util/irange.h>
0008: #include <c10/core/Contiguity.h>
0009: #include <c10/core/GradMode.h>
0010: #include <c10/core/SymInt.h>
0011: #include <c10/util/MaybeOwned.h>
0012: #include <ATen/TensorSubclassLikeUtils.h>
0013: 
0014: #ifndef AT_PER_OPERATOR_HEADERS
0015: #include <ATen/Functions.h>
0016: #include <ATen/NativeFunctions.h>
0017: #else
0018: #include <ATen/ops/_trilinear.h>
0019: #include <ATen/ops/_trilinear_native.h>
0020: #include <ATen/ops/add.h>
0021: #include <ATen/ops/addmm.h>
0022: #include <ATen/ops/bilinear_native.h>
0023: #include <ATen/ops/bmm.h>
0024: #include <ATen/ops/dot.h>
0025: #include <ATen/ops/einsum_native.h>
0026: #include <ATen/ops/linear_native.h>
0027: #include <ATen/ops/matmul.h>
0028: #include <ATen/ops/mkldnn_linear.h>
0029: #include <ATen/ops/mm.h>
0030: #include <ATen/ops/mul.h>
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, macro-based glue.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、宏定义或宏调用。

### Lines 31-60 / 第 31-60 行
```cpp
0031: #include <ATen/ops/tensordot_native.h>
0032: #include <ATen/ops/zeros.h>
0033: #endif
0034: 
0035: #include <cctype>
0036: #include <deque>
0037: #include <string>
0038: #include <utility>
0039: #include <vector>
0040: 
0041: namespace at::native {
0042: 
0043: // Parse environment variable "TORCH_LINEAR_FLATTEN_3D"
0044: static inline bool parseLinearFlatten3d() {
0045:   // Uninitialized value
0046:   static auto value = c10::utils::check_env("TORCH_LINEAR_FLATTEN_3D");
0047:   return value.has_value() && value.value();
0048: }
0049: 
0050: // `_flatten_nd_linear` flattens all but the last dimension of the input tensor
0051: // before passing it to linear operation
0052: static inline Tensor _flatten_nd_linear(const Tensor& input, const Tensor& weight, const Tensor& bias) {
0053:   const auto input_sizes = input.sym_sizes();
0054: 
0055:   const auto result_flattened = [&]() -> Tensor {
0056:     const auto input_ncols = input_sizes.back();
0057:     const auto input_flattened_nrows = [&]() -> c10::SymInt {
0058:       // can't use -1 in reshape because it errors when a dimension is 0
0059:       auto flattened_nrows = c10::SymInt{1};
0060:       for (const auto& size : input_sizes.slice(0, input_sizes.size() - 1)) {
```
- **EN**: Lines 31-60 mainly cover header inclusion, state/variable declarations, comments/documentation. Notable symbols: parseLinearFlatten3d, check_env, has_value, value.
- **CN**: 第 31-60 行主要涉及头文件包含、变量/别名声明、注释或说明。 值得关注的符号包括：parseLinearFlatten3d, check_env, has_value, value。

### Lines 61-90 / 第 61-90 行
```cpp
0061:         flattened_nrows *= size;
0062:       }
0063:       return flattened_nrows;
0064:     }();
0065: 
0066:     const auto input_flattened = input.view_symint({input_flattened_nrows, input_ncols});
0067:     if (weight.layout() == c10::kStrided) {
0068:       return at::addmm(bias, input_flattened, weight.t());
0069:     } else {
0070:       // weight is sparse, and addmm for sparse expects matmul lhs to be sparse,
0071:       // so we transpose the problem.
0072:       // NOTE: at::matmul handles (dense @ sparse) similarly.
0073:       const auto bias_t = (bias.dim() >= 2) ? bias.mT() : bias.unsqueeze(-1);
0074:       return at::addmm(bias_t, weight, input_flattened.t()).t();
0075:     }
0076:   }();
0077: 
0078:   // Unflatten flattened row dims
0079:   auto result_sizes = c10::SymDimVector{input_sizes.begin(), input_sizes.end()};
0080:   result_sizes.back() = result_flattened.sym_size(1);
0081:   return result_flattened.view_symint(result_sizes);
0082: }
0083: 
0084: 
0085: Tensor linear(const Tensor& input, const Tensor& weight, const std::optional<Tensor>& bias_opt) {
0086:   // _matmul_impl checks this again later, but _flatten_nd_linear does not work on scalars inputs,
0087:   // so let's try to catch this here already
0088:   const auto input_dim = input.dim();
0089:   const auto weight_dim = weight.dim();
0090:   TORCH_CHECK(input_dim != 0 && weight_dim != 0,
```
- **EN**: Lines 61-90 mainly cover state/variable declarations, comments/documentation, return paths. Notable symbols: view_symint, layout, addmm, t.
- **CN**: 第 61-90 行主要涉及变量/别名声明、注释或说明、返回路径。 值得关注的符号包括：view_symint, layout, addmm, t。

### Lines 91-120 / 第 91-120 行
```cpp
0091:               "both arguments to linear need to be at least 1D, but they are ",
0092:               input_dim, "D and ", weight_dim, "D");
0093: 
0094:   // See [Note: hacky wrapper removal for optional tensor]
0095:   auto bias = bias_opt.has_value()
0096:     ? c10::MaybeOwned<Tensor>::borrowed(*bias_opt)
0097:     : c10::MaybeOwned<Tensor>::owned(std::in_place);
0098:   if (input.is_mkldnn()) {
0099:     return at::mkldnn_linear(input, weight, *bias);
0100:   }
0101: #if defined(C10_MOBILE)
0102:   if (xnnpack::use_linear(input, weight, *bias)) {
0103:     return xnnpack::linear(input, weight, *bias);
0104:   }
0105: #endif
0106:   if (input_dim == 2 && bias->defined()) {
0107:     // Fused op is marginally faster.
0108:     return at::addmm(*bias, input, weight.t());
0109:   }
0110: 
0111:   const auto is_bias_likely_fusable = (
0112:       bias->defined() &&
0113:       // cuBLASLt: will fuse in the epilogue without copies
0114:       // when input/weight/bias are all strided.
0115:       // When weight is not strided, bias will not be fused,
0116:       // but we can still dispatch here to avoid at::matmul
0117:       // path which will probably use a very similar
0118:       // flattening optimization.
0119:       ((bias->dim() == 1 || bias->squeeze().dim() == 1) && bias->is_contiguous_or_false())
0120:   );
```
- **EN**: Lines 91-120 mainly cover comments/documentation, expressions/calls, function signatures/definitions. Notable symbols: has_value, borrowed, owned, is_mkldnn.
- **CN**: 第 91-120 行主要涉及注释或说明、表达式或调用、函数签名或实现。 值得关注的符号包括：has_value, borrowed, owned, is_mkldnn。

### Lines 121-150 / 第 121-150 行
```cpp
0121:   if (is_bias_likely_fusable && !input.is_xla()) {
0122:     // Also hit the fused path for contiguous nD input, if not using xla
0123:     // backend. Reshaping/flattening has some performance implications on xla.
0124:     if (input.is_contiguous_or_false()) {
0125:       return _flatten_nd_linear(input, weight, *bias);
0126:     } else if (parseLinearFlatten3d()) {
0127:       // If user forces flattening via env var
0128:       const Tensor input_cont = input.contiguous();
0129:       return _flatten_nd_linear(input_cont, weight, *bias);
0130:     }
0131:   }
0132:   auto output = at::matmul(input, weight.t());
0133:   if (bias->defined()) {
0134:     // for composite compliance use out-of-place version of `add`
0135:     if (isTensorSubclassLike(*bias) ||
0136:         bias->_fw_grad(/*level*/ 0).defined()) {
0137:       output = at::add(output, *bias);
0138:     } else {
0139:       output.add_(*bias);
0140:     }
0141:   }
0142:   return output;
0143: }
0144: 
0145: Tensor& linear_out(const Tensor& input, const Tensor& weight, const std::optional<Tensor>& bias_opt, Tensor& output) {
0146:   TORCH_CHECK(!input.is_mkldnn(), "linear doesn't support out for MKLDNN tensors");
0147:   // See [Note: hacky wrapper removal for optional tensor]
0148:   auto bias = bias_opt.has_value()
0149:               ? c10::MaybeOwned<Tensor>::borrowed(*bias_opt)
0150:               : c10::MaybeOwned<Tensor>::owned(std::in_place);
```
- **EN**: Lines 121-150 mainly cover function signatures/definitions, comments/documentation, state/variable declarations. Notable symbols: is_xla, is_contiguous_or_false, _flatten_nd_linear, parseLinearFlatten3d.
- **CN**: 第 121-150 行主要涉及函数签名或实现、注释或说明、变量/别名声明。 值得关注的符号包括：is_xla, is_contiguous_or_false, _flatten_nd_linear, parseLinearFlatten3d。

### Lines 151-180 / 第 151-180 行
```cpp
0151: 
0152:   if (input.dim() == 2 && bias->defined()) {
0153:     // Fused op is marginally faster.
0154:     return at::addmm_out(output, *bias, input, weight.t());
0155:   }
0156:   output = at::matmul_out(output, input, weight.t());
0157:   if (bias->defined()) {
0158:     output.add_(*bias);
0159:   }
0160:   return output;
0161: }
0162: 
0163: // sumproduct_pair computes `(left*right).sum(sumdims)` by means of permutation and
0164: // batch matrix multiplication
0165: // its main purpose is to provide a pairwise reduction for einsum
0166: static Tensor sumproduct_pair(const Tensor& left_, const Tensor& right_, IntArrayRef sum_dims_, bool keepdim) {
0167:   // assumes that tensors have been pre-unsqueezed (so that all dimensions match - after broadcasting)
0168:   // but makes no other assumptions on the order of dimensions
0169:   TORCH_CHECK(left_.dim()==right_.dim(), "number of dimensions must match");
0170:   if (sum_dims_.empty())
0171:     return at::mul(left_, right_);
0172:   int64_t dim = left_.dim();
0173:   auto sum_dims = at::dim_list_to_bitset(sum_dims_, dim);
0174:   // dimensions that will be part of the output (i.e. not summed over) in three vectors:
0175:   // dims in lro appear in left, right and output, similarly, lo: left and output, ro: right and output
0176:   // also the sizes are kept track of for reshaping
0177:   std::vector<int64_t> lro, lo, ro;
0178:   SymInt lro_size = 1, lo_size = 1, ro_size = 1, sum_size = 1;
0179:   Tensor left = left_;
0180:   Tensor right = right_;
```
- **EN**: Lines 151-180 mainly cover comments/documentation, state/variable declarations, control-flow checks. Notable symbols: dim, defined, addmm_out, t.
- **CN**: 第 151-180 行主要涉及注释或说明、变量/别名声明、控制流逻辑。 值得关注的符号包括：dim, defined, addmm_out, t。

### Lines 181-210 / 第 181-210 行
```cpp
0181:   for (const auto i : c10::irange(dim)) {
0182:     auto sl = TORCH_GUARD_OR_TRUE(left.sym_size(i).sym_ne(1));
0183:     auto sr = TORCH_GUARD_OR_TRUE(right.sym_size(i).sym_ne(1));
0184:     if (sum_dims[i]) { // first dimensions that will be summed over after multiplication
0185:       if (sl && sr) {  // dimensions nontrivially in both left and right must be of the same size
0186:         TORCH_SYM_CHECK(left.sym_size(i).sym_eq(right.sym_size(i)), "non-broadcast dimensions must match");
0187:         sum_size *= left.sym_size(i);
0188:       } else if (sl) { // if it is only in one of left and right, we can sum right away
0189:         left = left.sum(i, true);
0190:       } else if (sr) {
0191:         right = right.sum(i, true);
0192:       }
0193:     } else if (sl && sr) { // now deal with dimensions that will be in the output
0194:       // dimensions nontrivially in both left and right must be of the same size
0195:       TORCH_SYM_CHECK(left.sym_size(i).sym_eq(right.sym_size(i)), "non-broadcast dimensions must match");
0196:       lro.push_back(i);
0197:       lro_size *= left.sym_size(i);
0198:     } else if (sl) { // keep track of dimensions appearing only once
0199:       lo.push_back(i);
0200:       lo_size *= left.sym_size(i);
0201:     } else {
0202:       ro.push_back(i);
0203:       ro_size *= right.sym_size(i);
0204:     }
0205:   }
0206:   // we now work with the following permutations / shapes.
0207:   // the pipeline is permute inputs -> reshape inputs -> batch matrix mul -> reshape(view) output -> permute output
0208:   // output: "lro, lo, 1-for-summed-dims, ro" with original shape dimensions
0209:   // left:   "lro, lo, summed" permuted with lpermutation and the three flattened
0210:   // right:  "lro, summed, ro" permuted with rpermutation and the three flattened
```
- **EN**: Lines 181-210 mainly cover state/variable declarations, comments/documentation, function signatures/definitions. Notable symbols: irange, TORCH_GUARD_OR_TRUE, sym_size, sym_ne.
- **CN**: 第 181-210 行主要涉及变量/别名声明、注释或说明、函数签名或实现。 值得关注的符号包括：irange, TORCH_GUARD_OR_TRUE, sym_size, sym_ne。

### Lines 211-240 / 第 211-240 行
```cpp
0211:   // then the permuted output is a view of bmm(left, right)
0212:   // finally, opermutation reverts the permutation to the original order of dimensions
0213:   // By default the output is "lro, lo, 1-for-summed-dims, ro" with original shape dimensions.
0214:   // However, if all dimensions from the right operand appear before those from the left
0215:   // operand in the final output, we can swap the operands so that bmm directly produces
0216:   // the result in the correct memory order.
0217: 
0218:   bool swap_lo_ro = !lo.empty() && !ro.empty() && ro.back() < lo.front();
0219:   if (swap_lo_ro) {
0220:     std::swap(left, right);
0221:     std::swap(lo, ro);
0222:     std::swap(lo_size, ro_size);
0223:   }
0224:   auto out_num_dim = lro.size() + lo.size() + sum_dims_.size() + ro.size();
0225:   std::vector<SymInt> out_size;
0226:   out_size.reserve(out_num_dim);
0227:   for (auto& d : lro) out_size.push_back(left.sym_size(d));
0228:   for (auto& d : lo) out_size.push_back(left.sym_size(d));
0229:   for (auto& d : sum_dims_) { out_size.emplace_back(1); (void)d; }; // avoid warning about not using d
0230:   for (auto& d : ro) out_size.push_back(right.sym_size(d));
0231: 
0232:   std::vector<int64_t> lpermutation(lro);
0233:   lpermutation.insert(lpermutation.end(), lo.begin(), lo.end());
0234:   lpermutation.insert(lpermutation.end(), sum_dims_.begin(), sum_dims_.end());
0235:   lpermutation.insert(lpermutation.end(), ro.begin(), ro.end());
0236: 
0237:   std::vector<int64_t> rpermutation(lro);
0238:   rpermutation.insert(rpermutation.end(), sum_dims_.begin(), sum_dims_.end());
0239:   rpermutation.insert(rpermutation.end(), ro.begin(), ro.end());
0240:   rpermutation.insert(rpermutation.end(), lo.begin(), lo.end());
```
- **EN**: Lines 211-240 mainly cover state/variable declarations, comments/documentation, control-flow checks. Notable symbols: bmm, empty, back, front.
- **CN**: 第 211-240 行主要涉及变量/别名声明、注释或说明、控制流逻辑。 值得关注的符号包括：bmm, empty, back, front。

### Lines 241-270 / 第 241-270 行
```cpp
0241: 
0242:   std::vector<int64_t> opermutation(out_num_dim, -1);
0243:   {
0244:     int64_t i = 0;
0245: 
0246:     for (auto it = lro.cbegin(); it != lro.cend(); i++, it++) {
0247:       opermutation[*it] = i;
0248:     }
0249:     for (auto it = lo.cbegin(); it != lo.cend(); i++, it++) {
0250:       opermutation[*it] = i;
0251:     }
0252:     for (auto it = sum_dims_.cbegin(); it != sum_dims_.cend(); i++, it++) {
0253:       opermutation[*it] = i;
0254:     }
0255:     for (auto it = ro.cbegin(); it != ro.cend(); i++, it++) {
0256:       opermutation[*it] = i;
0257:     }
0258:   }
0259: 
0260:   // now we can execute the operations above
0261:   left = left.permute(lpermutation).reshape_symint({lro_size, std::move(lo_size), sum_size});
0262:   right = right.permute(rpermutation).reshape_symint({std::move(lro_size), std::move(sum_size), std::move(ro_size)});
0263:   Tensor result = at::bmm(left, right);
0264:   result = result.view_symint(out_size).permute(opermutation);
0265: 
0266:   // finally squeeze summed dimensions if desired
0267:   if (! keepdim) {
0268:     auto sizes = result.sizes().vec();
0269:     for (auto i = dim-1; i>=0; i--) {
0270:       if (sum_dims[i]) {
```
- **EN**: Lines 241-270 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: opermutation, cbegin, cend, permute.
- **CN**: 第 241-270 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：opermutation, cbegin, cend, permute。

### Lines 271-300 / 第 271-300 行
```cpp
0271:         sizes.erase(sizes.begin() + i);
0272:       }
0273:     }
0274:     result = result.view(sizes);
0275:   }
0276:   return result;
0277: }
0278: 
0279: // There are roughly three parts to computing einsum:
0280: // 1. Parse equation to extract the labels for each input operand and output
0281: // 2. Unsqueeze missing dimensions from input operands and permute to align them
0282: // 3. Compute result by multiplying input operands and summing contraction
0283: //    dimensions. We do the last part by reducing to bmm.
0284: // If a path is specified, we reduce in the order specified by the path, else we
0285: // default to going left => right. The path is a list of indices processed the same
0286: // way as opt-einsum: https://optimized-einsum.readthedocs.io/en/stable/path_finding.html#format-of-the-path
0287: Tensor einsum(std::string_view equation, TensorList operands, at::OptionalIntArrayRef path) {
0288:   TORCH_CHECK(!operands.empty(), "einsum(): must provide at least one operand");
0289:   const auto num_ops = operands.size();
0290: 
0291:   if (path.has_value()) {
0292:     const auto path_size = num_ops == 1 ? 1 : (num_ops - 1) * 2;
0293:     TORCH_CHECK(
0294:         path->size() == path_size,
0295:         "einsum(): expected contraction path given in path parameter to have size ",
0296:         path_size,
0297:         " but got ",
0298:         path->size());
0299:   }
0300: 
```
- **EN**: Lines 271-300 mainly cover comments/documentation, expressions/calls, state/variable declarations. Notable symbols: erase, begin, view, einsum.
- **CN**: 第 271-300 行主要涉及注释或说明、表达式或调用、变量/别名声明。 值得关注的符号包括：erase, begin, view, einsum。

### Lines 301-330 / 第 301-330 行
```cpp
0301:   // Labels must be in range [A-Za-z]
0302:   constexpr uint8_t NUM_OF_LETTERS = 'z' - 'a' + 1;
0303:   constexpr uint8_t TOTAL_LABELS = NUM_OF_LETTERS * 2;
0304: 
0305:   // Code used to identify ELLIPSIS ("...")
0306:   constexpr uint8_t ELLIPSIS = TOTAL_LABELS;
0307: 
0308:   // Convert label in [A-Za-z] to subscript in [0, TOTAL_LABELS)
0309:   auto label_to_subscript = [=](unsigned char label) -> uint8_t {
0310:     return std::isupper(label) ? label - 'A' : label - 'a' + NUM_OF_LETTERS;
0311:   };
0312: 
0313: #ifndef STRIP_ERROR_MESSAGES
0314:   // Convert subscript in [0, TOTAL_LABELS) to label in [A-Za-z]
0315:   auto subscript_to_label = [=](uint8_t s) -> unsigned char {
0316:     return s < NUM_OF_LETTERS ? s + 'A' : s + 'a' - NUM_OF_LETTERS;
0317:   };
0318: #endif
0319: 
0320:   // Find arrow (->) to split equation into lhs and rhs
0321:   const auto arrow_pos = equation.find("->");
0322:   const auto lhs = equation.substr(0, arrow_pos);
0323: 
0324:   // Convert labels for input operands into an index in [0, 52) and store
0325:   // them in op_labels for each operand along with ELLIPSIS if present.
0326:   std::vector<std::vector<uint8_t>> op_labels(num_ops);
0327:   bool ell_in_input = false;
0328:   std::size_t curr_op = 0;
0329:   for (std::size_t i = 0; i < lhs.length(); ++i) {
0330:     const unsigned char label = lhs[i];
```
- **EN**: Lines 301-330 mainly cover state/variable declarations, comments/documentation, function signatures/definitions. Notable symbols: ELLIPSIS, isupper, arrow, find.
- **CN**: 第 301-330 行主要涉及变量/别名声明、注释或说明、函数签名或实现。 值得关注的符号包括：ELLIPSIS, isupper, arrow, find。

### Lines 331-360 / 第 331-360 行
```cpp
0331:     switch (label) {
0332:       case ' ':
0333:         // Ignore spaces
0334:         break;
0335: 
0336:       case '.':
0337:         TORCH_CHECK(
0338:             // Only one ellipsis per operand can be given
0339:             !ell_in_input,
0340:             "einsum(): found \'.\' for operand ",
0341:             curr_op,
0342:             " for which an ellipsis was already found");
0343:         TORCH_CHECK(
0344:             // Ensure it's a valid ellipsis
0345:             i + 2 < lhs.length() && lhs[++i] == '.' && lhs[++i] == '.',
0346:             "einsum(): found \'.\' for operand ",
0347:             curr_op,
0348:             " that is not part of any ellipsis");
0349:         op_labels[curr_op].push_back(ELLIPSIS);
0350:         ell_in_input = true;
0351:         break;
0352: 
0353:       case ',':
0354:         // Move onto next operand
0355:         ++curr_op;
0356:         TORCH_CHECK(
0357:             curr_op < num_ops,
0358:             "einsum(): fewer operands were provided than specified in the equation");
0359:         ell_in_input = false;
0360:         break;
```
- **EN**: Lines 331-360 mainly cover state/variable declarations, control-flow checks, comments/documentation. Notable symbols: TORCH_CHECK, einsum, length, push_back.
- **CN**: 第 331-360 行主要涉及变量/别名声明、控制流逻辑、注释或说明。 值得关注的符号包括：TORCH_CHECK, einsum, length, push_back。

### Lines 361-390 / 第 361-390 行
```cpp
0361: 
0362:       default:
0363:         // Parse label
0364:         TORCH_CHECK(
0365:             std::isalpha(label),
0366:             "einsum(): invalid subscript given at index ",
0367:             i,
0368:             " in the equation string, subscripts must be in [a-zA-Z]");
0369:         op_labels[curr_op].push_back(label_to_subscript(label));
0370:     }
0371:   }
0372: 
0373:   TORCH_CHECK(
0374:       curr_op == num_ops - 1,
0375:       "einsum(): more operands were provided than specified in the equation");
0376: 
0377:   std::vector<int64_t> label_count(TOTAL_LABELS, 0);
0378: 
0379:   // The maximum number of dimensions covered by any ellipsis, needed when
0380:   // unsqueezing missing dimensions from operands to permute and broadcast
0381:   int64_t ell_num_dim = 0;
0382: 
0383:   // Compute label frequency and number of dimensions covered by ellipsis
0384:   // We do this after parsing labels to make it more readable and simpler
0385:   // to compute the number of dimensions covered by ellipsis.
0386:   for(const auto i : c10::irange(num_ops)) {
0387:     const auto& operand = operands[i];
0388:     const auto labels = op_labels[i];
0389:     const auto ndims = operand.dim();
0390:     int64_t nlabels = static_cast<int64_t>(labels.size());
```
- **EN**: Lines 361-390 mainly cover state/variable declarations, comments/documentation, expressions/calls. Notable symbols: TORCH_CHECK, isalpha, einsum, push_back.
- **CN**: 第 361-390 行主要涉及变量/别名声明、注释或说明、表达式或调用。 值得关注的符号包括：TORCH_CHECK, isalpha, einsum, push_back。

### Lines 391-420 / 第 391-420 行
```cpp
0391:     bool has_ellipsis = false;
0392: 
0393:     for (const auto& label : labels) {
0394:       if (label == ELLIPSIS) {
0395:         --nlabels;
0396:         has_ellipsis = true;
0397:         ell_num_dim = std::max(ell_num_dim, ndims - nlabels);
0398:       } else {
0399:         ++label_count[label];
0400:       }
0401:     }
0402: 
0403:     TORCH_CHECK(
0404:         has_ellipsis ? nlabels <= ndims : nlabels == ndims,
0405:         "einsum(): the number of subscripts in the equation (",
0406:         nlabels,
0407:         has_ellipsis ? ") is more than the number of dimensions ("
0408:                      : ") does not match the number of dimensions (",
0409:         ndims,
0410:         ") for operand ",
0411:         i,
0412:         has_ellipsis ? "" : " and no ellipsis was given");
0413:   }
0414: 
0415:   // We want to align the dimensions of every input tensor to have
0416:   // shape out_dims + sum_dims. For this, we create a mapping of label
0417:   // to index into the permuted shape.
0418:   std::vector<int64_t> label_perm_index(TOTAL_LABELS, -1);
0419: 
0420:   // Current index in the permuted shape
```
- **EN**: Lines 391-420 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: max, TORCH_CHECK, einsum, equation.
- **CN**: 第 391-420 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：max, TORCH_CHECK, einsum, equation。

### Lines 421-450 / 第 421-450 行
```cpp
0421:   int64_t perm_index = 0;
0422: 
0423:   // Start index of ellipsis dimensions in the permuted shape
0424:   int64_t ell_index = 0;
0425:   bool ell_in_output = false;
0426: 
0427:   if (arrow_pos == std::string::npos) {
0428:     // Implicit output is ellipsis (...) + labels seen only once
0429:     perm_index = ell_num_dim;
0430:     // ell_in_output is used to stop us from reducing ellipses dims later
0431:     ell_in_output = true;
0432:     for (const auto label : c10::irange(TOTAL_LABELS)) {
0433:       if (label_count[label] == 1) {
0434:         label_perm_index[label] = perm_index++;
0435:       }
0436:     }
0437:   } else {
0438:     // Parse explicit output
0439:     const auto rhs = equation.substr(arrow_pos + 2);
0440:     for (std::size_t i = 0; i < rhs.length(); ++i) {
0441:       const unsigned char label = rhs[i];
0442:       switch (label) {
0443:         case ' ':
0444:           // Ignore spaces
0445:           break;
0446: 
0447:         case '.':
0448:           TORCH_CHECK(
0449:               // There can only be one ellipsis in the output
0450:               !ell_in_output,
```
- **EN**: Lines 421-450 mainly cover state/variable declarations, control-flow checks, comments/documentation. Notable symbols: ellipsis, irange, substr, length.
- **CN**: 第 421-450 行主要涉及变量/别名声明、控制流逻辑、注释或说明。 值得关注的符号包括：ellipsis, irange, substr, length。

### Lines 451-480 / 第 451-480 行
```cpp
0451:               "einsum(): found \'.\' for output but an ellipsis (...) was already found");
0452:           TORCH_CHECK(
0453:               // Ensure ellipsis is correct
0454:               i + 2 < rhs.length() && rhs[++i] == '.' && rhs[++i] == '.',
0455:               "einsum(): found \'.\' for output that is not part of any ellipsis (...)");
0456:           ell_index = perm_index;
0457:           perm_index += ell_num_dim;
0458:           ell_in_output = true;
0459:           break;
0460: 
0461:         default:
0462:           TORCH_CHECK(
0463:               std::isalpha(label),
0464:               "einsum(): invalid subscript given at index ",
0465:               lhs.size() + 2 + i,
0466:               " in the equation string, subscripts must be in [a-zA-Z]");
0467:           const auto index = label_to_subscript(label);
0468:           TORCH_CHECK(
0469:               // Ensure label appeared at least once for some input operand and at
0470:               // most once for the output
0471:               label_count[index] > 0 && label_perm_index[index] == -1,
0472:               "einsum(): output subscript ",
0473:               label,
0474:               label_perm_index[index] > -1
0475:                   ? " appears more than once in the output"
0476:                   : " does not appear in the equation for any input operand");
0477:           label_perm_index[index] = perm_index++;
0478:       }
0479:     }
0480:   }
```
- **EN**: Lines 451-480 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: einsum, ellipsis, TORCH_CHECK, length.
- **CN**: 第 451-480 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：einsum, ellipsis, TORCH_CHECK, length。

### Lines 481-510 / 第 481-510 行
```cpp
0481: 
0482:   // Save number of dimensions in output before adding contraction dims (dims to sum out)
0483:   const int64_t out_num_dim = perm_index;
0484: 
0485:   // If ellipsis is not part of the output, add to contraction dimensions
0486:   if (!ell_in_output) {
0487:     ell_index = perm_index;
0488:     perm_index += ell_num_dim;
0489:   }
0490: 
0491:   // Add contraction labels (labels not present in output)
0492:   for (const auto label : c10::irange(TOTAL_LABELS)) {
0493:     if (label_count[label] > 0 && label_perm_index[label] == -1) {
0494:       label_perm_index[label] = perm_index++;
0495:     }
0496:   }
0497: 
0498:   // Next: we check the sizes, take diagonals for repeated labels, unsqueeze
0499:   // missing dimensions so all operands have the same dimensions and permute
0500:   // the operands to align the dimensions following the indices computed above.
0501:   // We also count how many operands have dimension with size != 1 for each
0502:   // label used to identify which dimensions can be contracted.
0503:   std::vector<SymInt> label_size(TOTAL_LABELS, 1);
0504:   std::vector<SymInt> ell_sizes(ell_num_dim, 1);
0505:   std::vector<uint64_t> dim_counts(perm_index, 0);
0506:   std::deque<Tensor> ops;
0507:   for (const auto i : irange(num_ops)) {
0508:     auto op = operands[i];
0509:     std::vector<int64_t> permutation(perm_index, -1);
0510:     std::int64_t dim = 0;
```
- **EN**: Lines 481-510 mainly cover state/variable declarations, comments/documentation, control-flow checks. Notable symbols: dims, labels, irange, label_size.
- **CN**: 第 481-510 行主要涉及变量/别名声明、注释或说明、控制流逻辑。 值得关注的符号包括：dims, labels, irange, label_size。

### Lines 511-540 / 第 511-540 行
```cpp
0511:     for (const auto s : op_labels[i]) {
0512:       if (s == ELLIPSIS) {
0513:         // Iterate over each dimension covered by ellipsis
0514:         const auto ndim = operands[i].ndimension() - (static_cast<int64_t>(op_labels[i].size()) - 1);
0515:         for (auto j = ell_num_dim - ndim; j < ell_num_dim; ++j) {
0516:           if (TORCH_GUARD_OR_TRUE(op.sym_size(dim).sym_ne(1))) {
0517:             // Update ellipsis size
0518:             TORCH_SYM_CHECK(
0519:                 ell_sizes[j].sym_eq(1).sym_or(ell_sizes[j].sym_eq(op.sym_size(dim))),
0520:                 "einsum(): dimension ",
0521:                 dim,
0522:                 " covered by ellipsis in operand ",
0523:                 i,
0524:                 "has size ",
0525:                 op.size(dim),
0526:                 " which does not broadcast with previously seen ellipsis with size ",
0527:                 ell_sizes[j],
0528:                 " for the respective dimension");
0529:             ell_sizes[j] = op.sym_size(dim);
0530:             ++dim_counts[ell_index + j];
0531:           }
0532:           permutation[ell_index + j] = dim++;
0533:         }
0534:       } else if (permutation[label_perm_index[s]] == -1) {
0535:         if (TORCH_GUARD_OR_TRUE(op.sym_size(dim).sym_ne(1))) {
0536:           // Update subscript
0537:           TORCH_SYM_CHECK(
0538:               label_size[s].sym_eq(1).sym_or(label_size[s].sym_eq(op.sym_size(dim))),
0539:               "einsum(): subscript ",
0540:               subscript_to_label(s),
```
- **EN**: Lines 511-540 mainly cover expressions/calls, function signatures/definitions, control-flow checks. Notable symbols: ndimension, size, TORCH_GUARD_OR_TRUE, sym_size.
- **CN**: 第 511-540 行主要涉及表达式或调用、函数签名或实现、控制流逻辑。 值得关注的符号包括：ndimension, size, TORCH_GUARD_OR_TRUE, sym_size。

### Lines 541-570 / 第 541-570 行
```cpp
0541:               " has size ",
0542:               op.sym_size(dim),
0543:               " for operand ",
0544:               i,
0545:               " which does not broadcast with previously seen size ",
0546:               label_size[s]);
0547:           label_size[s] = op.sym_size(dim);
0548:           ++dim_counts[label_perm_index[s]];
0549:         }
0550:         permutation[label_perm_index[s]] = dim++;
0551:       } else {
0552:         // Repeated label, take diagonal
0553:         const auto prev_dim = permutation[label_perm_index[s]];
0554:         TORCH_CHECK(
0555:           op.sym_size(dim) == op.sym_size(prev_dim),
0556:             "einsum(): subscript ",
0557:             subscript_to_label(s),
0558:             " is repeated for operand ",
0559:             i,
0560:             " but the sizes don't match, ",
0561:             op.sym_size(dim),
0562:             " != ",
0563:             op.sym_size(prev_dim));
0564:         op = op.diagonal(0, prev_dim, dim).movedim(-1, prev_dim);
0565:       }
0566:     }
0567: 
0568:     // Add dimensions for missing labels
0569:     for (auto& val : permutation) {
0570:       if (val == -1) {
```
- **EN**: Lines 541-570 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: sym_size, TORCH_CHECK, einsum, subscript_to_label.
- **CN**: 第 541-570 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：sym_size, TORCH_CHECK, einsum, subscript_to_label。

### Lines 571-600 / 第 571-600 行
```cpp
0571:         op = op.unsqueeze(dim);
0572:         val = dim++;
0573:       }
0574:     }
0575:     ops.emplace_back(op.permute(permutation));
0576:   }
0577: 
0578:   const auto contract_path = path.value_or(std::vector<int64_t>{});
0579:   auto it = contract_path.begin();
0580: 
0581:   // Contract
0582:   while (ops.size() > 1) {
0583:     int64_t i = 0;
0584:     int64_t j = 1;
0585: 
0586:     if (path.has_value()) {
0587:       i = *it++;
0588:       j = *it++;
0589:       if (j < i) {
0590:         std::swap(i, j);
0591:       }
0592: 
0593:       TORCH_CHECK(
0594:           i != j && i >= 0 && j < static_cast<int64_t>(ops.size()),
0595:           "einsum(): invalid contraction (",
0596:           i,
0597:           ", ",
0598:           j,
0599:           i == j ? ") cannot contract an operand with itself"
0600:                  : ") operand index is out of bounds");
```
- **EN**: Lines 571-600 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: unsqueeze, emplace_back, permute, value_or.
- **CN**: 第 571-600 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：unsqueeze, emplace_back, permute, value_or。

### Lines 601-630 / 第 601-630 行
```cpp
0601:     }
0602: 
0603:     auto a = ops[i];
0604:     auto b = ops[j];
0605:     ops.erase(ops.begin() + j);
0606:     ops.erase(ops.begin() + i);
0607: 
0608:     // Collect dimensions that can be summed now
0609:     std::vector<int64_t> sum_dims;
0610:     SmallVector<int64_t, 5> a_dims_to_sum;
0611:     SmallVector<int64_t, 5> b_dims_to_sum;
0612:     for (auto dim = out_num_dim; dim < perm_index; ++dim) {
0613:       auto sa = TORCH_GUARD_OR_TRUE(a.sym_size(dim).sym_ne(1));
0614:       auto sb = TORCH_GUARD_OR_TRUE(b.sym_size(dim).sym_ne(1));
0615: 
0616:       if (sa && sb) {
0617:         // if both a and b are equal, or we can't tell that its a broadcast for sure,
0618:         // we assume non-broadcast.
0619:         TORCH_SYM_CHECK(a.sym_size(dim).sym_eq(b.sym_size(dim)), "non-broadcast dimensions must match");
0620:         if (--dim_counts[dim] == 1) {
0621:           sum_dims.push_back(dim);
0622:           dim_counts[dim] = 0;
0623:         }
0624:       } else if (dim_counts[dim] == 1) {
0625:         if (sa) {
0626:           a_dims_to_sum.push_back(dim);
0627:           dim_counts[dim] = 0;
0628:         } else if (sb) {
0629:           b_dims_to_sum.push_back(dim);
0630:           dim_counts[dim] = 0;
```
- **EN**: Lines 601-630 mainly cover state/variable declarations, control-flow checks, comments/documentation. Notable symbols: erase, begin, TORCH_GUARD_OR_TRUE, sym_size.
- **CN**: 第 601-630 行主要涉及变量/别名声明、控制流逻辑、注释或说明。 值得关注的符号包括：erase, begin, TORCH_GUARD_OR_TRUE, sym_size。

### Lines 631-660 / 第 631-660 行
```cpp
0631:         }
0632:       }
0633:     }
0634: 
0635:     // Sum multiple dims at a time to minimize the number of kernel calls to sum
0636:     if (!a_dims_to_sum.empty()) {
0637:       a = a.sum(a_dims_to_sum, true);
0638:     }
0639:     if (!b_dims_to_sum.empty()) {
0640:       b = b.sum(b_dims_to_sum, true);
0641:     }
0642: 
0643:     if (path.has_value()) {
0644:       ops.emplace_back(sumproduct_pair(a, b, sum_dims, true));
0645:     } else {
0646:       ops.emplace_front(sumproduct_pair(a, b, sum_dims, true));
0647:     }
0648:   }
0649: 
0650:   // Sum out contraction dims
0651:   if (perm_index - out_num_dim > 0) {
0652:     // if there were ops to contract, we would have already done so
0653:     // in the previous loop and all the dims to sum are now 1
0654:     // NB: use view instead of squeeze (or sum) for faster (mps) performance
0655:     if (num_ops > 1) {
0656:       auto sizes = ops[0].sym_sizes().vec();
0657:       for (auto dim = perm_index - 1; dim >= out_num_dim; --dim) {
0658:         sizes.erase(sizes.begin() + dim);
0659:       }
0660:       return ops[0].view_symint(sizes);
```
- **EN**: Lines 631-660 mainly cover expressions/calls, control-flow checks, state/variable declarations. Notable symbols: empty, sum, has_value, emplace_back.
- **CN**: 第 631-660 行主要涉及表达式或调用、控制流逻辑、变量/别名声明。 值得关注的符号包括：empty, sum, has_value, emplace_back。

### Lines 661-690 / 第 661-690 行
```cpp
0661:     } else {
0662:       std::vector<int64_t> sum_dims(perm_index - out_num_dim);
0663:       std::iota(sum_dims.begin(), sum_dims.end(), out_num_dim);
0664:       return ops[0].sum(sum_dims);
0665:     }
0666:   }
0667: 
0668:   return std::move(ops[0]);
0669: }
0670: 
0671: // _trilinear computes a trilinear einstein sum with an unrolled dimension
0672: // the result is `(i1.unsqueeze(expand1)*i2.unsqueeze(expand2)*i2.unsqueeze(expand3)).sum(sumdim)`
0673: // the computation is unrolled in the unroll_dim dimension
0674: // its main purpose is to unify the computations in bilinear and bilinear_backward
0675: Tensor _trilinear(const Tensor& i1_, const Tensor& i2_, const Tensor& i3_,
0676:                   IntArrayRef expand1_, IntArrayRef expand2_, IntArrayRef expand3_,
0677:                   IntArrayRef sumdim_, int64_t unroll_dim) {
0678:   int64_t total_dim = i1_.dim()+expand1_.size();
0679:   TORCH_CHECK((unroll_dim >= 0) && (unroll_dim < total_dim), "unroll_dim must be in [0,", total_dim-1, "]");
0680:   auto expand1 = at::dim_list_to_bitset(expand1_, total_dim);
0681:   auto expand2 = at::dim_list_to_bitset(expand2_, total_dim);
0682:   auto expand3 = at::dim_list_to_bitset(expand3_, total_dim);
0683:   auto sumdim  = at::dim_list_to_bitset(sumdim_,  total_dim);
0684:   Tensor i1 = i1_;
0685:   Tensor i2 = i2_;
0686:   Tensor i3 = i3_;
0687:   std::vector<c10::SymInt> output_size;
0688:   std::vector<int64_t> sum_dims_12, sum_dims_23;
0689:   int64_t unroll_size = -1;
0690:   // asserts...
```
- **EN**: Lines 661-690 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: sum_dims, iota, begin, end.
- **CN**: 第 661-690 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：sum_dims, iota, begin, end。

### Lines 691-720 / 第 691-720 行
```cpp
0691:   for (const auto i : c10::irange(total_dim)) {
0692:     c10::SymInt s = 0;
0693:     if (expand1[i]) {
0694:       i1 = i1.unsqueeze(i);
0695:     } else  {
0696:       s = i1.sym_size(i);
0697:     }
0698:     if (expand2[i]) {
0699:       i2 = i2.unsqueeze(i);
0700:     } else  {
0701:       s = i2.sym_size(i);
0702:     }
0703:     if (expand3[i]) {
0704:       i3 = i3.unsqueeze(i);
0705:       if (sumdim[i] && (i != unroll_dim))
0706:         sum_dims_12.push_back(i);
0707:     } else  {
0708:       s = i3.sym_size(i);
0709:       if (sumdim[i] && (i != unroll_dim))
0710:         sum_dims_23.push_back(i);
0711:     }
0712:     output_size.push_back(sumdim[i] ? 1 : s);
0713:     if (i == unroll_dim)
0714:       unroll_size = s.guard_int(__FILE__, __LINE__);
0715:   }
0716:   int64_t slicemul1 = (expand1[unroll_dim] ? 0 : 1);
0717:   int64_t slicemul2 = (expand2[unroll_dim] ? 0 : 1);
0718:   int64_t slicemul3 = (expand3[unroll_dim] ? 0 : 1);
0719: 
0720:   auto output = at::zeros_symint(output_size, i1.options());
```
- **EN**: Lines 691-720 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: irange, unsqueeze, sym_size, push_back.
- **CN**: 第 691-720 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：irange, unsqueeze, sym_size, push_back。

### Lines 721-750 / 第 721-750 行
```cpp
0721: 
0722:   // Three conditionals are necessary since this function is meant to work for both
0723:   // forward and backward, which changes the dimensions of the inputs.
0724:   // Note that if output has zero elems is because (at least) one of i1, i2, i3 has zero elems.
0725:   if (i1.sym_numel() != 0 && i2.sym_numel() != 0 && i3.sym_numel() != 0) {
0726:     if (! sumdim[unroll_dim]) {
0727:       for (const auto k : c10::irange(unroll_size)) {
0728:         Tensor buf = at::native::sumproduct_pair(i1.narrow(unroll_dim, k * slicemul1, 1),
0729:                                                  i2.narrow(unroll_dim, k * slicemul2, 1),
0730:                                                  sum_dims_12, true);
0731:         buf = at::native::sumproduct_pair(buf, i3.narrow(unroll_dim, k * slicemul3, 1), sum_dims_23, true);
0732:         output.narrow(unroll_dim, k, 1).add_(buf);
0733:       }
0734:     }
0735:     else {
0736:       for (const auto k : c10::irange(unroll_size)) {
0737:         Tensor buf = at::native::sumproduct_pair(i1.narrow(unroll_dim, k*slicemul1, 1),
0738:                                                  i2.narrow(unroll_dim, k*slicemul2, 1), sum_dims_12, true);
0739:         buf = at::native::sumproduct_pair(buf, i3.narrow(unroll_dim, k*slicemul3, 1), sum_dims_23, true);
0740:         output.add_(buf);
0741:       }
0742:     }
0743:   }
0744:   for (int64_t i = output.dim()-1; i >= 0; i--)
0745:     if (sumdim[i])
0746:       output.squeeze_(i);
0747:   return output;
0748: }
0749: 
0750: Tensor bilinear(const Tensor& input1, const Tensor& input2, const Tensor& weight, const std::optional<Tensor>& bias_opt) {
```
- **EN**: Lines 721-750 mainly cover control-flow checks, state/variable declarations, expressions/calls. Notable symbols: because, sym_numel, irange, sumproduct_pair.
- **CN**: 第 721-750 行主要涉及控制流逻辑、变量/别名声明、表达式或调用。 值得关注的符号包括：because, sym_numel, irange, sumproduct_pair。

### Lines 751-780 / 第 751-780 行
```cpp
0751:   // See [Note: hacky wrapper removal for optional tensor]
0752:   c10::MaybeOwned<Tensor> bias_maybe_owned = at::borrow_from_optional_tensor(bias_opt);
0753:   const Tensor& bias = *bias_maybe_owned;
0754:   if (bias.defined()) {
0755:     TORCH_CHECK(
0756:         input1.dtype() == input2.dtype() && input1.dtype() == weight.dtype() &&
0757:             input1.dtype() == bias.dtype(),
0758:         "All tensors must have the same dtype, got input1: ",
0759:         input1.dtype(),
0760:         ", input2: ",
0761:         input2.dtype(),
0762:         ", weight: ",
0763:         weight.dtype(),
0764:         ", bias: ",
0765:         bias.dtype());
0766:   } else {
0767:     TORCH_CHECK(
0768:         input1.dtype() == input2.dtype() && input1.dtype() == weight.dtype(),
0769:         "All tensors must have the same dtype, got input1: ",
0770:         input1.dtype(),
0771:         ", input2: ",
0772:         input2.dtype(),
0773:         ", weight: ",
0774:         weight.dtype());
0775:   }
0776: 
0777:   TORCH_CHECK(input1.dim() == input2.dim(), "bilinear(): input dimensions do not match: got ", input1.dim(), " and ", input2.dim());
0778:   for (const auto i : c10::irange(input1.dim() - 1)) {
0779:     TORCH_CHECK(input1.sym_size(i) == input2.sym_size(i),
0780:               "bilinear(): input batch dimensions do not match at dim ", i, ": got ", input1.sym_size(i), " and ", input2.sym_size(i));
```
- **EN**: Lines 751-780 mainly cover function signatures/definitions, expressions/calls, state/variable declarations. Notable symbols: borrow_from_optional_tensor, defined, TORCH_CHECK, dtype.
- **CN**: 第 751-780 行主要涉及函数签名或实现、表达式或调用、变量/别名声明。 值得关注的符号包括：borrow_from_optional_tensor, defined, TORCH_CHECK, dtype。

### Lines 781-810 / 第 781-810 行
```cpp
0781:   }
0782:   TORCH_CHECK(input1.sym_size(input1.dim() - 1) == weight.sym_size(1),
0783:             "bilinear(): input1 size does not match weight size: got ",
0784:             input1.sym_size(input1.dim() - 1), " but expected ", weight.sym_size(1));
0785:   TORCH_CHECK(input2.sym_size(input2.dim() - 1) == weight.sym_size(2),
0786:             "bilinear(): input2 size does not match weight size: got ",
0787:             input2.sym_size(input2.dim() - 1), " but expected ", weight.sym_size(2));
0788:   TORCH_CHECK(!bias.defined() || bias.sym_size(0) == weight.sym_size(0),
0789:             "bilinear(): bias size does not match weight size: got ",
0790:             bias.sym_size(0), " but expected ", weight.sym_size(0));
0791: 
0792:   std::vector<c10::SymInt> output_size;
0793:   auto size1 = input1.sym_sizes();
0794:   output_size.insert(output_size.end(), size1.begin(), size1.end() - 1);
0795:   output_size.push_back(weight.sym_size(0));
0796:   auto input1_flattened = input1.reshape_symint({-1, input1.sym_size(-1)});
0797:   auto input2_flattened = input2.reshape_symint({-1, input2.sym_size(-1)});
0798:   Tensor output = at::_trilinear(input1_flattened, weight, input2_flattened, {1,3}, {0}, {1,2}, {2,3}).reshape_symint(output_size);
0799:   if (bias.defined()) {
0800:     output = output + bias;
0801:   }
0802:   return output;
0803: }
0804: 
0805: // implements tensordot, a matrix-multiplication-like contraction, but the dimensions given
0806: // in the two dimension lists
0807: Tensor tensordot(const Tensor& input1, const Tensor& input2, IntArrayRef dims1, IntArrayRef dims2) {
0808:   TORCH_CHECK(dims1.size() == dims2.size(), "both dimension lists should have same length");
0809:   TORCH_CHECK(input1.scalar_type() == input2.scalar_type(), "both inputs should have same dtype");
0810:   SymInt csize = 1;  // total size of the contracted dimensions
```
- **EN**: Lines 781-810 mainly cover state/variable declarations, macro-based glue, expressions/calls. Notable symbols: TORCH_CHECK, sym_size, dim, bilinear.
- **CN**: 第 781-810 行主要涉及变量/别名声明、宏定义或宏调用、表达式或调用。 值得关注的符号包括：TORCH_CHECK, sym_size, dim, bilinear。

### Lines 811-840 / 第 811-840 行
```cpp
0811:   Tensor t1 = input1;
0812:   Tensor t2 = input2;
0813:   for (const auto i : c10::irange(dims1.size())) {
0814:     SymInt s1 = input1.sym_size(dims1[i]);
0815:     SymInt s2 = input2.sym_size(dims2[i]);
0816:     if (s2 == 1) { // broadcasted dimensions can be summed right away
0817:       t1 = t1.sum(dims1[i], true, t1.scalar_type());
0818:     } else if (s1 == 1) {
0819:       t2 = t2.sum(dims2[i], true, t2.scalar_type());
0820:     } else {
0821:       TORCH_CHECK(s1 == s2, "contracted dimensions need to match, but first has size ", s1, " in dim ", dims1[i],
0822:                " and second has size ", s2, " in dim ", dims2[i]);
0823:       csize *= s1;
0824:     }
0825:   }
0826: 
0827:   auto cdims1 = at::dim_list_to_bitset(dims1, input1.dim());
0828:   auto cdims2 = at::dim_list_to_bitset(dims2, input2.dim());
0829:   std::vector<int64_t> p1, p2;  // p1, p2: input permutations
0830:   std::vector<SymInt> rsizes;  // rsizes: sizes of the result
0831:   p1.reserve(input1.dim());
0832:   p2.reserve(input2.dim());
0833:   rsizes.reserve(input1.dim() + input2.dim() - static_cast<int64_t>(dims1.size()));
0834:   SymInt size1 = 1; // number of non-contracted elements in input1
0835:   SymInt size2 = 1; // number of non-contracted elements in input2
0836: 
0837:   // fill the permutations and compute sizes
0838:   for (const auto i : c10::irange(input1.dim())) {
0839:     if (! cdims1[i]) {
0840:       p1.emplace_back(i);
```
- **EN**: Lines 811-840 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: irange, size, sym_size, sum.
- **CN**: 第 811-840 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：irange, size, sym_size, sum。

### Lines 841-870 / 第 841-870 行
```cpp
0841:       size1 *= t1.sym_size(i);
0842:       rsizes.emplace_back(t1.sym_size(i));
0843:     }
0844:   }
0845:   for (const auto x : dims1) {
0846:     p1.emplace_back(x);
0847:   }
0848:   for (const auto x : dims2) {
0849:     p2.emplace_back(x);
0850:   }
0851:   for (const auto i : c10::irange(input2.dim())) {
0852:     if (! cdims2[i]) {
0853:       p2.emplace_back(i);
0854:       size2 *= t2.sym_size(i);
0855:       rsizes.emplace_back(t2.sym_size(i));
0856:     }
0857:   }
0858: 
0859:   // Full contraction (size1 == 1 and size2 == 1) is much faster when done with dot ...
0860:   // TODO(@nikitaved): there are other cases where dot outperforms gemms,
0861:   // like, for example, when the non-contracted dims are relatively small.
0862:   // NOTE(@nikitaved): contract with gemm when on MPS,
0863:   // otherwise issues with the tests xpassing/xfailing
0864:   // when enabling the fast-path with dot.
0865:   // TODO: resolve that
0866:   if ((t1.device().type() == at::kMPS || t2.device().type() == at::kMPS) || size1 != 1 || size2 != 1) {
0867:     // permute and reshape for matrix multiplication
0868:     t1 = t1.permute(p1).reshape_symint({size1, csize});
0869:     t2 = t2.permute(p2).reshape_symint({csize, size2});
0870:     // multiply and reshape to target size
```
- **EN**: Lines 841-870 mainly cover state/variable declarations, comments/documentation, expressions/calls. Notable symbols: sym_size, emplace_back, irange, dim.
- **CN**: 第 841-870 行主要涉及变量/别名声明、注释或说明、表达式或调用。 值得关注的符号包括：sym_size, emplace_back, irange, dim。

### Lines 871-900 / 第 871-900 行
```cpp
0871:     return at::mm(t1, t2).reshape_symint(rsizes);
0872:   } else {
0873:     // permute to align for contraction
0874:     t1 = t1.permute(p1);
0875:     t2 = t2.permute(p2);
0876: 
0877:     if (t1.is_contiguous() && t2.is_contiguous()) {
0878:       // If t1 and t2 are both contiguous, then flatten is a view,
0879:       // then dot is the method of choice
0880:       return at::dot(t1.flatten(), t2.flatten()).reshape_symint(rsizes);
0881:     } else {
0882:       // Otherwise mul + sum can be faster as it avoids at most 2x contiguous() calls
0883:       // NOTE: t1.dtype == t2.dtype -- check above
0884:       return (t1.squeeze() * t2.squeeze()).sum(t1.scalar_type()).reshape_symint(rsizes);
0885:     }
0886:   }
0887: }
0888: 
0889: Tensor &tensordot_out(const Tensor& input1, const Tensor& input2, IntArrayRef dims1, IntArrayRef dims2, Tensor& result) {
0890:   Tensor result_tmp = at::native::tensordot(input1, input2, dims1, dims2);
0891:   auto result_dtype = result_tmp.scalar_type();
0892:   auto output_tensor_dtype = result.scalar_type();
0893:   auto output_device = result.device();
0894:   auto input1_device = input1.device();
0895:   auto input2_device = input2.device();
0896: 
0897:   if(result.defined()) {
0898:     TORCH_CHECK(
0899:       !(result.requires_grad() && at::GradMode::is_enabled() && result.sizes() != result_tmp.sizes()),
0900:       "tensordot(): the 'out' tensor was specified and requires gradients, and its shape does not match the expected result. "
```
- **EN**: Lines 871-900 mainly cover state/variable declarations, function signatures/definitions, comments/documentation. Notable symbols: mm, reshape_symint, permute, is_contiguous.
- **CN**: 第 871-900 行主要涉及变量/别名声明、函数签名或实现、注释或说明。 值得关注的符号包括：mm, reshape_symint, permute, is_contiguous。

### Lines 901-921 / 第 901-921 行
```cpp
0901:       "Either remove the 'out' argument, ensure it does not require gradients, or make sure its shape matches the expected output."
0902:     );
0903:   }
0904:   // check if the input & output tensors are on the same device.
0905:   TORCH_CHECK(
0906:     (output_device == input1_device) && (input1_device == input2_device),
0907:     "tensordot: Expected the output and input tensors to be on the "
0908:     "same device, but got the output tensor on ", output_device,
0909:     ", input tensor a on ", input1_device, ", and input tensor b on ", input2_device);
0910:   // check if the computed result has the same dtype as the out tensor
0911:   // (because tensordot does not support type promotion)
0912:   TORCH_CHECK(
0913:     result_dtype == output_tensor_dtype, "tensordot",
0914:     ": Expected the output tensor to have dtype ", result_dtype,
0915:     ", but got an output tensor with dtype ", output_tensor_dtype);
0916:   at::native::resize_output(result, result_tmp.sizes());
0917:   result.copy_(result_tmp);
0918:   return result;
0919: }
0920: 
0921: }  // namespace at::native
```
- **EN**: Lines 901-921 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: TORCH_CHECK, resize_output, sizes, copy_.
- **CN**: 第 901-921 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：TORCH_CHECK, resize_output, sizes, copy_。

## Key Concepts / 关键概念
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现
- **EN**: Native operator implementation path  
  **CN**: 原生算子实现路径

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/native/Resize.h>`, `<ATen/native/xnnpack/Engine.h>`, `<ATen/WrapDimUtilsMulti.h>`, `<ATen/TensorOperators.h>`, `<c10/util/irange.h>`, `<c10/core/Contiguity.h>`, `<c10/core/GradMode.h>`, `<c10/core/SymInt.h>`, `<c10/util/MaybeOwned.h>` ...
- **Macros / 宏**: `TORCH_CHECK`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
