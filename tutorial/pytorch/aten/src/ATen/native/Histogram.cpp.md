# Histogram.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/Histogram.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Histogram. It also wires backend dispatch paths.
- **Purpose (CN)**: 实现或声明与 histogram 相关的 ATen 原生逻辑。 它还负责连接不同后端的调度路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/core/Tensor.h>
0003: #include <ATen/Dispatch.h>
0004: 
0005: #include <ATen/native/Histogram.h>
0006: #include <ATen/native/Resize.h>
0007: 
0008: #ifndef AT_PER_OPERATOR_HEADERS
0009: #include <ATen/Functions.h>
0010: #include <ATen/NativeFunctions.h>
0011: #else
0012: #include <ATen/ops/_histogramdd_bin_edges.h>
0013: #include <ATen/ops/_histogramdd_bin_edges_native.h>
0014: #include <ATen/ops/_histogramdd_from_bin_cts.h>
0015: #include <ATen/ops/_histogramdd_from_bin_cts_native.h>
0016: #include <ATen/ops/_histogramdd_from_bin_tensors.h>
0017: #include <ATen/ops/_histogramdd_from_bin_tensors_native.h>
0018: #include <ATen/ops/aminmax.h>
0019: #include <ATen/ops/empty.h>
0020: #include <ATen/ops/histc_native.h>
0021: #include <ATen/ops/histogram_native.h>
0022: #include <ATen/ops/histogramdd_native.h>
0023: #include <ATen/ops/linspace.h>
0024: #endif
0025: 
0026: #include <cmath>
0027: #include <numeric>
0028: #include <tuple>
0029: #include <vector>
0030: #include <functional>
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, macro-based glue.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、宏定义或宏调用。

### Lines 31-60 / 第 31-60 行
```cpp
0031: #include <c10/util/ArrayRef.h>
0032: #include <c10/core/ScalarType.h>
0033: #include <c10/core/DefaultDtype.h>
0034: #include <c10/util/irange.h>
0035: 
0036: /* Implements a numpy-like histogramdd function running on cpu
0037:  * https://numpy.org/doc/stable/reference/generated/numpy.histogramdd.html
0038:  *
0039:  * See the docstr for torch.histogramdd in torch/functional.py for further explanation.
0040:  *
0041:  * - torch.histogramdd(input, bins, range=None, weight=None, density=False)
0042:  *   input     - tensor with shape (M, N). input is interpreted as M coordinates in N-dimensional space.
0043:  *               If a tensor with more than 2 dimensions is passed, all but the last dimension will be flattened.
0044:  *   bins      - int[] of length N or tensor list of length N. If int[], defines the number of equal-width bins
0045:  *               in each dimension. If tensor list, defines the sequences of bin edges, including rightmost edges,
0046:  *               for each dimension.
0047:  *   range     - float[] of length 2 * N, optional. If specified, defines the leftmost and rightmost bin edges
0048:  *               for each dimension.
0049:  *   weight    - tensor, optional. If provided, weight should have the same shape as input excluding its last dimension.
0050:  *               Each N-dimensional value in input contributes its associated weight towards its bin's result.
0051:  *               If weight is not specified, each value has weight 1 by default.
0052:  *   density   - bool, optional. If false (default), the result will contain the total count (weight) in each bin.
0053:  *               If True, each count (weight) is divided by the total count (total weight), then divided by the
0054:  *               volume of its associated bin.
0055:  *
0056:  * Returns:
0057:  *   hist      - N-dimensional tensor containing the values of the histogram.
0058:  *   bin_edges - tensor list of length N containing the edges of the histogram bins in each dimension.
0059:  *               Bins include their left edge and exclude their right edge, with the exception of the
0060:  *               rightmost bin in each dimension which includes both of its edges.
```
- **EN**: Lines 31-60 mainly cover comments/documentation, header inclusion. Notable symbols: histogramdd, shape, false, count.
- **CN**: 第 31-60 行主要涉及注释或说明、头文件包含。 值得关注的符号包括：histogramdd, shape, false, count。

### Lines 61-90 / 第 61-90 行
```cpp
0061:  *
0062:  * Restrictions are defined in histogram_check_inputs() and in select_outer_bin_edges().
0063:  */
0064: 
0065: namespace at::native {
0066: 
0067: DEFINE_DISPATCH(histogramdd_stub);
0068: DEFINE_DISPATCH(histogramdd_linear_stub);
0069: DEFINE_DISPATCH(histogram_select_outer_bin_edges_stub);
0070: 
0071: namespace {
0072: 
0073: /* Checks properties of input tensors input, bins, and weight.
0074:  */
0075: void histogramdd_check_inputs(const Tensor& input, const TensorList& bins, const std::optional<Tensor>& weight) {
0076:     TORCH_CHECK(input.dim() >= 2, "torch.histogramdd: input tensor should have at least 2 dimensions, but got ",
0077:                 input.dim());
0078: 
0079:     const int64_t N = input.size(-1);
0080: 
0081:     TORCH_CHECK(static_cast<int64_t>(bins.size()) == N, "torch.histogramdd: expected ", N, " sequences of bin edges for a ", N,
0082:                 "-dimensional histogram but got ", bins.size());
0083: 
0084:     auto input_dtype = input.dtype();
0085:     for (const auto dim : c10::irange(N)) {
0086:         const Tensor& dim_bins = bins[dim];
0087: 
0088:         auto bins_dtype = dim_bins.dtype();
0089:         TORCH_CHECK(input_dtype == bins_dtype, "torch.histogramdd: input tensor and bins tensors should",
0090:                 " have the same dtype, but got input with dtype ", input_dtype,
```
- **EN**: Lines 61-90 mainly cover macro-based glue, state/variable declarations, comments/documentation. Notable symbols: histogram_check_inputs, select_outer_bin_edges, DEFINE_DISPATCH, histogramdd_check_inputs.
- **CN**: 第 61-90 行主要涉及宏定义或宏调用、变量/别名声明、注释或说明。 值得关注的符号包括：histogram_check_inputs, select_outer_bin_edges, DEFINE_DISPATCH, histogramdd_check_inputs。

### Lines 91-120 / 第 91-120 行
```cpp
0091:                 " and bins for dimension ", dim, " with dtype ", bins_dtype);
0092: 
0093:         const int64_t dim_bins_dim = dim_bins.dim();
0094:         TORCH_CHECK(dim_bins_dim == 1, "torch.histogramdd: bins tensor should have one dimension,",
0095:                 " but got ", dim_bins_dim, " dimensions in the bins tensor for dimension ", dim);
0096: 
0097:         const int64_t numel = dim_bins.numel();
0098:         TORCH_CHECK(numel > 0, "torch.histogramdd: bins tensor should have at least 1 element,",
0099:                 " but got ", numel, " elements in the bins tensor for dimension ", dim);
0100:     }
0101: 
0102:     if (weight.has_value()) {
0103:         TORCH_CHECK(input.dtype() == weight.value().dtype(), "torch.histogramdd: if weight tensor is provided,"
0104:                 " input tensor and weight tensor should have the same dtype, but got input(", input.dtype(), ")",
0105:                 ", and weight(", weight.value().dtype(), ")");
0106: 
0107:         /* If a weight tensor is provided, we expect its shape to match that of
0108:          * the input tensor excluding its innermost dimension N.
0109:          */
0110:         auto input_sizes = input.sizes().vec();
0111:         input_sizes.pop_back();
0112: 
0113:         auto weight_sizes = weight.value().sizes().vec();
0114:         if (weight_sizes.empty()) {
0115:             // correctly handle scalars
0116:             weight_sizes = {1};
0117:         }
0118: 
0119:         TORCH_CHECK(input_sizes == weight_sizes, "torch.histogramdd: if weight tensor is provided it should have"
0120:                 " the same shape as the input tensor excluding its innermost dimension, but got input with shape ",
```
- **EN**: Lines 91-120 mainly cover state/variable declarations, macro-based glue, comments/documentation. Notable symbols: dim, TORCH_CHECK, numel, has_value.
- **CN**: 第 91-120 行主要涉及变量/别名声明、宏定义或宏调用、注释或说明。 值得关注的符号包括：dim, TORCH_CHECK, numel, has_value。

### Lines 121-150 / 第 121-150 行
```cpp
0121:                 input.sizes(), " and weight with shape ", weight.value().sizes());
0122:     }
0123: }
0124: 
0125: /* Checks properties of output tensors hist and bin_edges, then resizes them.
0126:  */
0127: void histogramdd_prepare_out(const Tensor& input, const std::vector<int64_t>& bin_ct,
0128:         const Tensor& hist, const TensorList& bin_edges) {
0129:     const int64_t N = input.size(-1);
0130: 
0131:     TORCH_INTERNAL_ASSERT((int64_t)bin_ct.size() == N);
0132:     TORCH_INTERNAL_ASSERT((int64_t)bin_edges.size() == N);
0133: 
0134:     TORCH_CHECK(input.dtype() == hist.dtype(), "torch.histogram: input tensor and hist tensor should",
0135:             " have the same dtype, but got input ", input.dtype(), " and hist ", hist.dtype());
0136: 
0137:     for (const auto dim : c10::irange(N)) {
0138:         TORCH_CHECK(input.dtype() == bin_edges[dim].dtype(), "torch.histogram: input tensor and bin_edges tensor should",
0139:                 " have the same dtype, but got input ", input.dtype(), " and bin_edges ", bin_edges[dim].dtype(),
0140:                 " for dimension ", dim);
0141: 
0142:         TORCH_CHECK(bin_ct[dim] > 0,
0143:                 "torch.histogram(): bins must be > 0, but got ", bin_ct[dim], " for dimension ", dim);
0144: 
0145:         at::native::resize_output(bin_edges[dim], bin_ct[dim] + 1);
0146:     }
0147: 
0148:     at::native::resize_output(hist, bin_ct);
0149: }
0150: 
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: sizes, value, histogramdd_prepare_out, size.
- **CN**: 第 121-150 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：sizes, value, histogramdd_prepare_out, size。

### Lines 151-180 / 第 151-180 行
```cpp
0151: void histogramdd_prepare_out(const Tensor& input, TensorList bins,
0152:         const Tensor& hist, const TensorList& bin_edges) {
0153:     std::vector<int64_t> bin_ct(bins.size());
0154:     std::transform(bins.begin(), bins.end(), bin_ct.begin(), [](Tensor t) { return t.numel() - 1; });
0155:     histogramdd_prepare_out(input, bin_ct, hist, bin_edges);
0156: }
0157: 
0158: /* Determines the outermost bin edges. For simplicity when calling into aminmax,
0159:  * assumes that input has already been reshaped to (M, N).
0160:  */
0161: std::pair<std::vector<double>, std::vector<double>>
0162: select_outer_bin_edges(const Tensor& input, std::optional<c10::ArrayRef<double>> range) {
0163:     TORCH_INTERNAL_ASSERT(input.dim() == 2, "expected input to have shape (M, N)");
0164:     const int64_t N = input.size(-1);
0165: 
0166:     // Default ranges for empty input matching numpy.histogram's default
0167:     std::vector<double> leftmost_edges(N, 0.);
0168:     std::vector<double> rightmost_edges(N, 1.);
0169: 
0170:     if (range.has_value()) {
0171:         // range is specified
0172:         TORCH_CHECK((int64_t)range.value().size() == 2 * N, "torch.histogramdd: for a ", N, "-dimensional histogram",
0173:                 " range should have ", 2 * N, " elements, but got ", range.value().size());
0174: 
0175:         for (const auto dim : c10::irange(N)) {
0176:             leftmost_edges[dim] = range.value()[2 * dim];
0177:             rightmost_edges[dim] = range.value()[2 * dim + 1];
0178:         }
0179:     } else if (input.numel() > 0) {
0180:         // non-empty input
```
- **EN**: Lines 151-180 mainly cover state/variable declarations, comments/documentation, expressions/calls. Notable symbols: histogramdd_prepare_out, bin_ct, size, transform.
- **CN**: 第 151-180 行主要涉及变量/别名声明、注释或说明、表达式或调用。 值得关注的符号包括：histogramdd_prepare_out, bin_ct, size, transform。

### Lines 181-210 / 第 181-210 行
```cpp
0181: 
0182:         histogram_select_outer_bin_edges_stub(input.device().type(), input, N, leftmost_edges, rightmost_edges);
0183:     }
0184: 
0185:     for (const auto dim : c10::irange(N)) {
0186:         double leftmost_edge = leftmost_edges[dim];
0187:         double rightmost_edge = rightmost_edges[dim];
0188: 
0189:         TORCH_CHECK(std::isfinite(leftmost_edge) && std::isfinite(rightmost_edge),
0190:                 "torch.histogramdd: dimension ", dim, "'s range [",
0191:                 leftmost_edge, ", ", rightmost_edge, "] is not finite");
0192: 
0193:         TORCH_CHECK(leftmost_edge <= rightmost_edge, "torch.histogramdd: min should not exceed max, but got",
0194:                 " min ", leftmost_edge, " max ", rightmost_edge, " for dimension ", dim);
0195: 
0196:         // Expand empty range to match numpy behavior and avoid division by 0 in normalization
0197:         if (leftmost_edge == rightmost_edge) {
0198:             leftmost_edges[dim] -= 0.5;
0199:             rightmost_edges[dim] += 0.5;
0200:         }
0201:     }
0202: 
0203:     return std::make_pair(leftmost_edges, rightmost_edges);
0204: }
0205: 
0206: 
0207: /* Bin edges correction based on the precision representation.
0208:  * To maintain the backward compatibility we take max(std::nextafter<>, +1)
0209:  * and min(std::nextafter<>, -1) for scalar types. For other types +/- 1 as usual.
0210:  */
```
- **EN**: Lines 181-210 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: histogram_select_outer_bin_edges_stub, device, type, irange.
- **CN**: 第 181-210 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：histogram_select_outer_bin_edges_stub, device, type, irange。

### Lines 211-240 / 第 211-240 行
```cpp
0211: void bins_edges_correction(const ScalarType& t, double &leftmost_edge, double &rightmost_edge)
0212: {
0213: #define UPDATE_WITH_LIMIT(real_type, scalartype) \
0214:   case ScalarType::scalartype:                   \
0215:     leftmost_edge = std::min(                    \
0216:         static_cast<double>(                     \
0217:             std::nexttoward(                     \
0218:                 static_cast<real_type>(leftmost_edge),   \
0219:                 std::numeric_limits<real_type>::lowest() \
0220:             )                                    \
0221:         ),                                       \
0222:         leftmost_edge - 1.                       \
0223:     );                                           \
0224:     rightmost_edge = std::max(                   \
0225:         static_cast<double>(                     \
0226:             std::nexttoward(                     \
0227:                 static_cast<real_type>(rightmost_edge), \
0228:                 std::numeric_limits<real_type>::max()   \
0229:             )                                    \
0230:         ),                                       \
0231:         rightmost_edge + 1.                      \
0232:     );                                           \
0233:     break;
0234: 
0235:     switch (t) {
0236:         UPDATE_WITH_LIMIT(double, Double)
0237:         UPDATE_WITH_LIMIT(float, Float)
0238:         default:
0239:             // Fallback to the default behavior for other types
0240:             leftmost_edge -= 1;
```
- **EN**: Lines 211-240 mainly cover expressions/calls, function signatures/definitions, macro-based glue. Notable symbols: bins_edges_correction, UPDATE_WITH_LIMIT, min, nexttoward.
- **CN**: 第 211-240 行主要涉及表达式或调用、函数签名或实现、宏定义或宏调用。 值得关注的符号包括：bins_edges_correction, UPDATE_WITH_LIMIT, min, nexttoward。

### Lines 241-270 / 第 241-270 行
```cpp
0241:             rightmost_edge += 1;
0242:     }
0243: #undef UPDATE_WITH_LIMIT
0244: }
0245: 
0246: /* histc's version of the logic for outermost bin edges.
0247:  */
0248: std::pair<double, double> histc_select_outer_bin_edges(const Tensor& input,
0249:         const Scalar& min, const Scalar& max) {
0250:     double leftmost_edge = min.to<double>();
0251:     double rightmost_edge = max.to<double>();
0252: 
0253:     if (leftmost_edge == rightmost_edge && input.numel() > 0) {
0254:         auto extrema = aminmax(input);
0255:         leftmost_edge = std::get<0>(extrema).item<double>();
0256:         rightmost_edge = std::get<1>(extrema).item<double>();
0257:     }
0258: 
0259:     if (leftmost_edge == rightmost_edge) {
0260:         bins_edges_correction(input.dtype().toScalarType(), leftmost_edge, rightmost_edge);
0261:     }
0262: 
0263:     TORCH_CHECK(!(std::isinf(leftmost_edge) || std::isinf(rightmost_edge) ||
0264:             std::isnan(leftmost_edge) || std::isnan(rightmost_edge)),
0265:             "torch.histc: range of [", leftmost_edge, ", ", rightmost_edge, "] is not finite");
0266: 
0267:     TORCH_CHECK(leftmost_edge < rightmost_edge, "torch.histc: max must be larger than min");
0268: 
0269:     return std::make_pair(leftmost_edge, rightmost_edge);
0270: }
```
- **EN**: Lines 241-270 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: histc_select_outer_bin_edges, numel, aminmax, bins_edges_correction.
- **CN**: 第 241-270 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：histc_select_outer_bin_edges, numel, aminmax, bins_edges_correction。

### Lines 271-300 / 第 271-300 行
```cpp
0271: 
0272: } // namespace
0273: 
0274: static std::vector<Tensor> allocate_bin_edges_tensors(const Tensor& self) {
0275:     TORCH_CHECK(self.dim() >= 2, "torch.histogramdd: input tensor should have at least 2 dimensions");
0276:     const int64_t N = self.size(-1);
0277:     std::vector<Tensor> bin_edges_out(N);
0278:     for (const auto dim : c10::irange(N)) {
0279:         bin_edges_out[dim] = at::empty({0}, self.options(), MemoryFormat::Contiguous);
0280:     }
0281:     return bin_edges_out;
0282: }
0283: 
0284: /* Versions of histogramdd in which bins is a Tensor[] defining the sequences of bin edges.
0285:  */
0286: static Tensor& histogramdd_out(const Tensor& self, TensorList bins,
0287:         const std::optional<Tensor>& weight, bool density,
0288:         Tensor& hist, TensorList& bin_edges) {
0289:     histogramdd_check_inputs(self, bins, weight);
0290:     histogramdd_prepare_out(self, bins, hist, bin_edges);
0291: 
0292:     for (const auto dim : c10::irange(bins.size())) {
0293:         bin_edges[dim].copy_(bins[dim]);
0294:     }
0295: 
0296:     histogramdd_stub(self.device().type(), self, weight, density, hist, bin_edges);
0297:     return hist;
0298: }
0299: 
0300: Tensor _histogramdd(const Tensor& self, TensorList bins,
```
- **EN**: Lines 271-300 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: allocate_bin_edges_tensors, TORCH_CHECK, dim, size.
- **CN**: 第 271-300 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：allocate_bin_edges_tensors, TORCH_CHECK, dim, size。

### Lines 301-330 / 第 301-330 行
```cpp
0301:         const std::optional<Tensor>& weight, bool density) {
0302:     Tensor hist = at::empty({0}, self.options(), MemoryFormat::Contiguous);
0303:     std::vector<Tensor> bin_edges_out = allocate_bin_edges_tensors(self);
0304:     TensorList bin_edges_out_tl(bin_edges_out);
0305: 
0306:     histogramdd_out(self, bins, weight, density, hist, bin_edges_out_tl);
0307:     return hist;
0308: }
0309: 
0310: /* Versions of histogramdd in which bins is an int[]
0311:  * defining the number of bins in each dimension.
0312:  */
0313: static std::vector<Tensor>& histogramdd_bin_edges_out(const Tensor& self, IntArrayRef bin_ct,
0314:         std::optional<c10::ArrayRef<double>> range,
0315:         const std::optional<Tensor>& weight, bool density,
0316:         std::vector<Tensor>& bin_edges_out) {
0317:     TensorList bin_edges_out_tl(bin_edges_out);
0318: 
0319:     const int64_t N = self.size(-1);
0320:     const int64_t M = std::accumulate(self.sizes().begin(), self.sizes().end() - 1,
0321:             static_cast<int64_t>(1), std::multiplies<int64_t>());
0322:     Tensor reshaped_self = self.reshape({ M, N });
0323: 
0324:     auto outer_bin_edges = select_outer_bin_edges(reshaped_self, range);
0325: 
0326:     const int64_t bin_size = bin_ct.size();
0327:     TORCH_CHECK(
0328:         N == bin_size,
0329:         "histogramdd: The size of bins must be equal to the innermost dimension of the input.");
0330:     for (const auto dim : c10::irange(N)) {
```
- **EN**: Lines 301-330 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: empty, options, allocate_bin_edges_tensors, bin_edges_out_tl.
- **CN**: 第 301-330 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：empty, options, allocate_bin_edges_tensors, bin_edges_out_tl。

### Lines 331-360 / 第 331-360 行
```cpp
0331:         at::linspace_out(bin_edges_out[dim], outer_bin_edges.first[dim], outer_bin_edges.second[dim],
0332:                 bin_ct[dim] + 1);
0333:     }
0334: 
0335:     return bin_edges_out;
0336: }
0337: 
0338: std::vector<Tensor> histogramdd_bin_edges(const Tensor& self, IntArrayRef bin_ct,
0339:         std::optional<c10::ArrayRef<double>> range,
0340:         const std::optional<Tensor>& weight, bool density) {
0341:     std::vector<Tensor> bin_edges_out = allocate_bin_edges_tensors(self);
0342:     return histogramdd_bin_edges_out(self, bin_ct, range, weight, density, bin_edges_out);
0343: }
0344: 
0345: static Tensor& histogramdd_out(const Tensor& self, IntArrayRef bin_ct,
0346:         std::optional<c10::ArrayRef<double>> range,
0347:         const std::optional<Tensor>& weight, bool density,
0348:         Tensor& hist, TensorList& bin_edges) {
0349:     std::vector<Tensor> bins = histogramdd_bin_edges(self, bin_ct, range, weight, density);
0350: 
0351:     histogramdd_check_inputs(self, bins, weight);
0352:     histogramdd_prepare_out(self, bins, hist, bin_edges);
0353: 
0354:     for (const auto dim : c10::irange(bins.size())) {
0355:         bin_edges[dim].copy_(bins[dim]);
0356:     }
0357: 
0358:     histogramdd_linear_stub(self.device().type(), self, weight, density, hist, bin_edges, true);
0359:     return hist;
0360: }
```
- **EN**: Lines 331-360 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: linspace_out, histogramdd_bin_edges, allocate_bin_edges_tensors, histogramdd_bin_edges_out.
- **CN**: 第 331-360 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：linspace_out, histogramdd_bin_edges, allocate_bin_edges_tensors, histogramdd_bin_edges_out。

### Lines 361-390 / 第 361-390 行
```cpp
0361: 
0362: Tensor _histogramdd(const Tensor& self, IntArrayRef bin_ct,
0363:         std::optional<c10::ArrayRef<double>> range,
0364:         const std::optional<Tensor>& weight, bool density) {
0365:     Tensor hist = at::empty({0}, self.options(), MemoryFormat::Contiguous);
0366:     std::vector<Tensor> bin_edges_out = allocate_bin_edges_tensors(self);
0367:     TensorList bin_edges_out_tl(bin_edges_out);
0368: 
0369:     histogramdd_out(self, bin_ct, range, weight, density, hist, bin_edges_out_tl);
0370:     return hist;
0371: }
0372: 
0373: /* Versions of histogram in which bins is a Tensor defining the sequence of bin edges.
0374:  */
0375: std::tuple<Tensor&, Tensor&>
0376: histogram_out(const Tensor& self, const Tensor& bins,
0377:         const std::optional<Tensor>& weight, bool density,
0378:         Tensor& hist, Tensor& bin_edges) {
0379:     Tensor reshaped_self = self.reshape({ self.numel(), 1 });
0380:     std::optional<Tensor> reshaped_weight = weight.has_value()
0381:         ? weight.value().reshape({ weight.value().numel() }) : weight;
0382:     TensorList bins_in = bins;
0383:     TensorList bins_out = bin_edges;
0384: 
0385:     histogramdd_out(reshaped_self, bins_in, reshaped_weight, density, hist, bins_out);
0386: 
0387:     return std::forward_as_tuple(hist, bin_edges);
0388: }
0389: 
0390: std::tuple<Tensor, Tensor>
```
- **EN**: Lines 361-390 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: _histogramdd, empty, options, allocate_bin_edges_tensors.
- **CN**: 第 361-390 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：_histogramdd, empty, options, allocate_bin_edges_tensors。

### Lines 391-420 / 第 391-420 行
```cpp
0391: histogram(const Tensor& self, const Tensor& bins,
0392:         const std::optional<Tensor>& weight, bool density) {
0393:     Tensor hist = at::empty({0}, self.options(), MemoryFormat::Contiguous);
0394:     Tensor bin_edges = at::empty({0}, bins.options(), MemoryFormat::Contiguous);
0395:     return histogram_out(self, bins, weight, density, hist, bin_edges);
0396: }
0397: 
0398: /* Versions of histogram in which bins is an integer specifying the number of equal-width bins.
0399:  */
0400: std::tuple<Tensor&, Tensor&>
0401: histogram_out(const Tensor& self, int64_t bin_ct, std::optional<c10::ArrayRef<double>> range,
0402:         const std::optional<Tensor>& weight, bool density,
0403:         Tensor& hist, Tensor& bin_edges) {
0404:     Tensor reshaped_self = self.reshape({ self.numel(), 1 });
0405:     std::optional<Tensor> reshaped_weight = weight.has_value()
0406:         ? weight.value().reshape({ weight.value().numel() }) : weight;
0407:     TensorList bins_in = bin_edges;
0408:     TensorList bins_out = bin_edges;
0409: 
0410:     histogramdd_prepare_out(reshaped_self, std::vector<int64_t>{bin_ct}, hist, bins_out);
0411:     auto outer_bin_edges = select_outer_bin_edges(reshaped_self, range);
0412:     at::linspace_out(bin_edges, outer_bin_edges.first[0], outer_bin_edges.second[0], bin_ct + 1);
0413: 
0414:     histogramdd_check_inputs(reshaped_self, bins_in, reshaped_weight);
0415: 
0416:     histogramdd_linear_stub(reshaped_self.device().type(), reshaped_self, reshaped_weight, density, hist, bin_edges, true);
0417:     return std::forward_as_tuple(hist, bin_edges);
0418: }
0419: 
0420: std::tuple<Tensor, Tensor>
```
- **EN**: Lines 391-420 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: histogram, empty, options, histogram_out.
- **CN**: 第 391-420 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：histogram, empty, options, histogram_out。

### Lines 421-450 / 第 421-450 行
```cpp
0421: histogram(const Tensor& self, int64_t bin_ct, std::optional<c10::ArrayRef<double>> range,
0422:         const std::optional<Tensor>& weight, bool density) {
0423:     Tensor hist = at::empty({0}, self.options(), MemoryFormat::Contiguous);
0424:     Tensor bin_edges_out = at::empty({0}, self.options());
0425:     return histogram_out(self, bin_ct, range, weight, density, hist, bin_edges_out);
0426: }
0427: 
0428: /* Narrowed interface for the legacy torch.histc function.
0429:  */
0430: Tensor& histogram_histc_out(const Tensor& self, int64_t bin_ct,
0431:         const Scalar& min, const Scalar& max, Tensor& hist) {
0432:     Tensor bin_edges = at::empty({0}, self.options());
0433: 
0434:     Tensor reshaped = self.reshape({ self.numel(), 1 });
0435:     TensorList bins_in = bin_edges;
0436:     TensorList bins_out = bin_edges;
0437: 
0438:     histogramdd_prepare_out(reshaped, std::vector<int64_t>{bin_ct}, hist, bins_out);
0439: 
0440:     auto outer_bin_edges = histc_select_outer_bin_edges(self, min, max);
0441:     at::linspace_out(bin_edges, outer_bin_edges.first, outer_bin_edges.second, bin_ct + 1);
0442: 
0443:     histogramdd_check_inputs(reshaped, bins_in, {});
0444: 
0445:     histogramdd_linear_stub(reshaped.device().type(), reshaped,
0446:             std::optional<Tensor>(), false, hist, bin_edges, false);
0447:     return hist;
0448: }
0449: 
0450: Tensor histogram_histc(const Tensor& self, int64_t bin_ct,
```
- **EN**: Lines 421-450 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: histogram, empty, options, histogram_out.
- **CN**: 第 421-450 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：histogram, empty, options, histogram_out。

### Lines 451-480 / 第 451-480 行
```cpp
0451:         const Scalar& min, const Scalar& max) {
0452:     Tensor hist = at::empty({0}, self.options(), MemoryFormat::Contiguous);
0453:     return histogram_histc_out(self, bin_ct, min, max, hist);
0454: }
0455: 
0456: std::tuple<Tensor, std::vector<Tensor>> histogramdd(
0457:     const Tensor &self, TensorList bins, std::optional<ArrayRef<double>> /*range*/,
0458:     const std::optional<Tensor> &weight, bool density) {
0459:   auto hist = at::_histogramdd_from_bin_tensors(self, bins, weight, density);
0460:   return std::tuple<Tensor, std::vector<Tensor>>{
0461:       std::move(hist), bins.vec()};
0462: }
0463: 
0464: std::tuple<Tensor, std::vector<Tensor>> histogramdd(
0465:     const Tensor &self, IntArrayRef bins, std::optional<ArrayRef<double>> range,
0466:     const std::optional<Tensor> &weight, bool density) {
0467:   auto bin_edges = at::_histogramdd_bin_edges(self, bins, range, weight, density);
0468:   auto hist = at::_histogramdd_from_bin_cts(self, bins, range, weight, density);
0469:   return std::tuple<Tensor, std::vector<Tensor>>{
0470:       std::move(hist), std::move(bin_edges)};
0471: }
0472: 
0473: std::tuple<Tensor, std::vector<Tensor>> histogramdd(
0474:     const Tensor &self, int64_t bins, std::optional<ArrayRef<double>> range,
0475:     const std::optional<Tensor> &weight, bool density) {
0476:   DimVector bins_v(self.size(-1), bins);
0477:   return at::native::histogramdd(self, bins_v, range, weight, density);
0478: }
0479: 
0480: } // namespace at::native
```
- **EN**: Lines 451-480 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: empty, options, histogram_histc_out, histogramdd.
- **CN**: 第 451-480 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：empty, options, histogram_histc_out, histogramdd。

## Key Concepts / 关键概念
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Dispatch stub definition  
  **CN**: 调度桩定义
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现
- **EN**: Scalar/tensor mixed arithmetic  
  **CN**: 标量与张量混合运算

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/native/Histogram.h>`, `<ATen/native/Resize.h>`, `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/_histogramdd_bin_edges.h>`, `<ATen/ops/_histogramdd_bin_edges_native.h>`, `<ATen/ops/_histogramdd_from_bin_cts.h>`, `<ATen/ops/_histogramdd_from_bin_cts_native.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `DEFINE_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
