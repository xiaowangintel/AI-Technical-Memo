# ConvUtils.h — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/ConvUtils.h`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Conv Utils. As a header, it exposes declarations and shared helpers.
- **Purpose (CN)**: 实现或声明与 卷积、utils 相关的 ATen 原生逻辑。 作为头文件，它暴露声明与共享辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #pragma once
0002: #include <ATen/core/Tensor.h>
0003: #include <ATen/TensorUtils.h>
0004: #include <ATen/detail/CUDAHooksInterface.h>
0005: #include <ATen/native/DispatchStub.h>
0006: #include <c10/util/env.h>
0007: #include <c10/util/irange.h>
0008: 
0009: #include <utility>
0010: 
0011: namespace at::native {
0012: 
0013: using conv_depthwise2d_backward_fn = std::tuple<at::Tensor,at::Tensor>(*)(
0014:     const at::Tensor&, const at::Tensor&, const at::Tensor&, at::IntArrayRef, at::IntArrayRef,
0015:     at::IntArrayRef, at::IntArrayRef, std::array<bool, 2>);
0016: DECLARE_DISPATCH(conv_depthwise2d_backward_fn, conv_depthwise2d_backward_stub)
0017: using conv_depthwise3d_backward_fn = std::tuple<at::Tensor,at::Tensor,at::Tensor>(*)(
0018:     const at::Tensor&, const at::Tensor&, const at::Tensor&, at::IntArrayRef, at::IntArrayRef,
0019:     at::IntArrayRef, at::IntArrayRef, std::array<bool, 3>);
0020: DECLARE_DISPATCH(conv_depthwise3d_backward_fn, conv_depthwise3d_backward_stub)
0021: using cudnn_convolution_backward_fn = std::tuple<at::Tensor,at::Tensor>(*)(
0022:     const at::Tensor&, const at::Tensor&, const at::Tensor&, at::IntArrayRef, at::IntArrayRef,
0023:     at::IntArrayRef, int64_t, bool, bool, bool, std::array<bool,2>);
0024: DECLARE_DISPATCH(cudnn_convolution_backward_fn, cudnn_convolution_backward_stub)
0025: using mps_convolution_backward_fn = std::tuple<at::Tensor,at::Tensor,at::Tensor>(*)(
0026:     const at::Tensor&, const at::Tensor&, const at::Tensor&, at::IntArrayRef, at::IntArrayRef,
0027:     at::IntArrayRef, int64_t, std::array<bool,3>);
0028: DECLARE_DISPATCH(mps_convolution_backward_fn, mps_convolution_backward_stub)
0029: using cudnn_convolution_transpose_backward_fn = std::tuple<at::Tensor,at::Tensor>(*)(
0030:     const at::Tensor&, const at::Tensor&, const at::Tensor&, at::IntArrayRef, at::IntArrayRef,
```
- **EN**: Lines 1-30 mainly cover state/variable declarations, header inclusion, macro-based glue. Notable symbols: DECLARE_DISPATCH.
- **CN**: 第 1-30 行主要涉及变量/别名声明、头文件包含、宏定义或宏调用。 值得关注的符号包括：DECLARE_DISPATCH。

### Lines 31-60 / 第 31-60 行
```cpp
0031:     at::IntArrayRef, at::IntArrayRef, int64_t, bool, bool, bool, std::array<bool,2>);
0032: DECLARE_DISPATCH(cudnn_convolution_transpose_backward_fn, cudnn_convolution_transpose_backward_stub)
0033: using miopen_convolution_backward_fn = std::tuple<at::Tensor,at::Tensor,at::Tensor>(*)(
0034:     const at::Tensor&, const at::Tensor&, const at::Tensor&, at::IntArrayRef, at::IntArrayRef,
0035:     at::IntArrayRef, int64_t, bool, bool, std::array<bool,3>);
0036: DECLARE_DISPATCH(miopen_convolution_backward_fn, miopen_convolution_backward_stub)
0037: using miopen_convolution_transpose_backward_fn = std::tuple<at::Tensor,at::Tensor,at::Tensor>(*)(
0038:     const at::Tensor&, const at::Tensor&, const at::Tensor&, at::IntArrayRef, at::IntArrayRef,
0039:     at::IntArrayRef, at::IntArrayRef, int64_t, bool, bool, std::array<bool,3>);
0040: DECLARE_DISPATCH(miopen_convolution_transpose_backward_fn, miopen_convolution_transpose_backward_stub)
0041: using miopen_depthwise_convolution_backward_fn = std::tuple<at::Tensor,at::Tensor,at::Tensor>(*)(
0042:     const at::Tensor&, const at::Tensor&, const at::Tensor&, at::IntArrayRef, at::IntArrayRef,
0043:     at::IntArrayRef, int64_t, bool, bool, std::array<bool,3>);
0044: DECLARE_DISPATCH(miopen_depthwise_convolution_backward_fn, miopen_depthwise_convolution_backward_stub)
0045: using mkldnn_convolution_backward_fn = std::tuple<at::Tensor,at::Tensor,at::Tensor>(*)(
0046:     const at::Tensor&, const at::Tensor&, const at::Tensor&, at::IntArrayRef, at::IntArrayRef,
0047:     at::IntArrayRef, int64_t, std::array<bool,3>);
0048: DECLARE_DISPATCH(mkldnn_convolution_backward_fn, mkldnn_convolution_backward_stub)
0049: using mkldnn_convolution_transpose_fn = Tensor(*)(const Tensor&, const Tensor&, const std::optional<Tensor>&,
0050:     IntArrayRef, IntArrayRef, IntArrayRef, IntArrayRef, int64_t);
0051: DECLARE_DISPATCH(mkldnn_convolution_transpose_fn, mkldnn_convolution_transpose_stub)
0052: using mkldnn_convolution_transpose_backward_fn = std::tuple<at::Tensor,at::Tensor,at::Tensor>(*)(
0053:     const at::Tensor&, const at::Tensor&, const at::Tensor&, at::IntArrayRef, at::IntArrayRef,
0054:     at::IntArrayRef, at::IntArrayRef, int64_t, std::array<bool,3>);
0055: DECLARE_DISPATCH(mkldnn_convolution_transpose_backward_fn, mkldnn_convolution_transpose_backward_stub)
0056: using slow_conv_dilated2d_backward_fn = std::tuple<at::Tensor,at::Tensor,at::Tensor>(*)(
0057:     const at::Tensor&, const at::Tensor&, const at::Tensor&, at::IntArrayRef, at::IntArrayRef,
0058:     at::IntArrayRef, at::IntArrayRef, std::array<bool, 3>);
0059: DECLARE_DISPATCH(slow_conv_dilated2d_backward_fn, slow_conv_dilated2d_backward_stub)
0060: using slow_conv_dilated3d_backward_fn = std::tuple<at::Tensor,at::Tensor,at::Tensor>(*)(
```
- **EN**: Lines 31-60 mainly cover state/variable declarations, macro-based glue, expressions/calls. Notable symbols: DECLARE_DISPATCH, Tensor.
- **CN**: 第 31-60 行主要涉及变量/别名声明、宏定义或宏调用、表达式或调用。 值得关注的符号包括：DECLARE_DISPATCH, Tensor。

### Lines 61-90 / 第 61-90 行
```cpp
0061:     const at::Tensor&, const at::Tensor&, const at::Tensor&, at::IntArrayRef, at::IntArrayRef,
0062:     at::IntArrayRef, at::IntArrayRef, std::array<bool, 3>);
0063: DECLARE_DISPATCH(slow_conv_dilated3d_backward_fn, slow_conv_dilated3d_backward_stub)
0064: using slow_conv_transpose2d_backward_fn = std::tuple<at::Tensor,at::Tensor,at::Tensor>(*)(
0065:     const at::Tensor&, const at::Tensor&, const at::Tensor&, at::IntArrayRef, at::IntArrayRef,
0066:     at::IntArrayRef, at::IntArrayRef, at::IntArrayRef, std::array<bool,3>);
0067: DECLARE_DISPATCH(slow_conv_transpose2d_backward_fn, slow_conv_transpose2d_backward_stub)
0068: using slow_conv_transpose3d_backward_fn = std::tuple<at::Tensor,at::Tensor,at::Tensor>(*)(
0069:     const at::Tensor&, const at::Tensor&, const at::Tensor&, at::IntArrayRef, at::IntArrayRef,
0070:     at::IntArrayRef, at::IntArrayRef, at::IntArrayRef, std::array<bool,3>);
0071: DECLARE_DISPATCH(slow_conv_transpose3d_backward_fn, slow_conv_transpose3d_backward_stub)
0072: 
0073: namespace {
0074:   bool is_cudnnv8_heuristic_mode_b() {
0075:     static const bool is_cudnnv8_heuristic_mode_b = c10::utils::check_env("TORCH_CUDNN_USE_HEURISTIC_MODE_B") == true;
0076:     return is_cudnnv8_heuristic_mode_b;
0077:   }
0078: }
0079: 
0080: inline bool cudnnv8_enabled_check_debug() {
0081:   static bool cudnnv8_flag = c10::utils::check_env("TORCH_CUDNN_V8_API_DISABLED") != true;
0082:   static bool cudnnv8_debug = c10::utils::check_env("TORCH_CUDNN_V8_API_DEBUG") == true;
0083:   static uint8_t cudnnv8_debugcount = 0;
0084:   if (cudnnv8_debug == 1 && cudnnv8_debugcount < 10) {
0085:     TORCH_WARN("TORCH_CUDNN_V8_DEBUG ON, V8 ON: ", cudnnv8_flag, " TORCH_CUDNN_USE_HEURISTIC_MODE B: ", is_cudnnv8_heuristic_mode_b());
0086:     cudnnv8_debugcount++;
0087:   }
0088:   return cudnnv8_flag == 1;
0089: }
0090: 
```
- **EN**: Lines 61-90 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: DECLARE_DISPATCH, is_cudnnv8_heuristic_mode_b, check_env, cudnnv8_enabled_check_debug.
- **CN**: 第 61-90 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：DECLARE_DISPATCH, is_cudnnv8_heuristic_mode_b, check_env, cudnnv8_enabled_check_debug。

### Lines 91-120 / 第 91-120 行
```cpp
0091: inline bool cudnnv8_use_heur_mode_b() {
0092:   return is_cudnnv8_heuristic_mode_b();
0093: }
0094: 
0095: // Keep in sync with py::enum_ in Module.cpp
0096: enum class ConvBackend {
0097:   CudaDepthwise2d,
0098:   CudaDepthwise3d,
0099:   Cudnn,
0100:   CudnnTranspose,
0101:   Empty,
0102:   Miopen,
0103:   MiopenDepthwise,
0104:   MiopenTranspose,
0105:   Mkldnn,
0106:   MkldnnTranspose,
0107:   MkldnnEmpty,
0108:   NnpackSpatial,
0109:   Overrideable,
0110:   Slow2d,
0111:   Slow3d,
0112:   SlowDilated2d,
0113:   SlowDilated3d,
0114:   SlowTranspose2d,
0115:   SlowTranspose3d,
0116:   Winograd3x3Depthwise,
0117:   Xnnpack2d,
0118:   Mps,
0119:   MpsTranspose,
0120: };
```
- **EN**: Lines 91-120 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: cudnnv8_use_heur_mode_b, is_cudnnv8_heuristic_mode_b.
- **CN**: 第 91-120 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：cudnnv8_use_heur_mode_b, is_cudnnv8_heuristic_mode_b。

### Lines 121-150 / 第 121-150 行
```cpp
0121: 
0122: // Overload for selecting the convolution backend from the full set of convolution inputs.
0123: // This overload is exposed to python for testing, etc.
0124: TORCH_API ConvBackend select_conv_backend(
0125:     const Tensor& input, const Tensor& weight, const std::optional<Tensor>& bias_opt,
0126:     SymIntArrayRef stride, SymIntArrayRef padding, SymIntArrayRef dilation,
0127:     bool transposed, SymIntArrayRef output_padding, c10::SymInt groups, const at::OptionalSymIntArrayRef bias_sizes_opt);
0128: 
0129: TORCH_API at::MemoryFormat _determine_backend_memory_format(const Tensor& input,
0130:     const Tensor& weight,
0131:     const ConvBackend backend);
0132: 
0133: // ---------------------------------------------------------------------
0134: //
0135: // Math
0136: //
0137: // ---------------------------------------------------------------------
0138: 
0139: constexpr int input_batch_size_dim = 0;  // also grad_input
0140: constexpr int input_channels_dim = 1;
0141: constexpr int output_batch_size_dim = 0;  // also grad_output
0142: constexpr int output_channels_dim = 1;
0143: constexpr int weight_output_channels_dim = 0;
0144: constexpr int weight_input_channels_dim = 1;
0145: 
0146: // Often written as 2 + max_dim (extra dims for batch size and channels)
0147: constexpr int max_dim = 3;
0148: 
0149: // ---------------------------------------------------------------------
0150: //
```
- **EN**: Lines 121-150 mainly cover comments/documentation, state/variable declarations, expressions/calls. Notable symbols: select_conv_backend, _determine_backend_memory_format, max_dim.
- **CN**: 第 121-150 行主要涉及注释或说明、变量/别名声明、表达式或调用。 值得关注的符号包括：select_conv_backend, _determine_backend_memory_format, max_dim。

### Lines 151-180 / 第 151-180 行
```cpp
0151: // Checking
0152: //
0153: // ---------------------------------------------------------------------
0154: 
0155: // Used on pad, stride and dilation
0156: static void check_args(CheckedFrom c, IntArrayRef args, size_t expected_size, const char* arg_name)
0157: {
0158:   TORCH_CHECK(args.size() <= expected_size,
0159:            "Too many ", arg_name, " values (", args.size(), ") supplied, expecting ",
0160:            expected_size, " (while checking arguments for ", c, ")");
0161:   TORCH_CHECK(args.size() >= expected_size,
0162:            "Not enough ", arg_name, " values (", args.size(), ") supplied, expecting ",
0163:            expected_size, " (while checking arguments for ", c, ")");
0164: 
0165:   auto num_negative_values = std::count_if(args.begin(), args.end(), [](int x){return x < 0;});
0166:   if (num_negative_values > 0){
0167:     std::stringstream ss;
0168:     ss << arg_name << " should be greater than zero but got (";
0169:     std::copy(args.begin(), args.end() - 1, std::ostream_iterator<int>(ss,", "));
0170:     ss << args.back() <<  ")" << " (while checking arguments for " << c << ')';
0171:     TORCH_CHECK(false, ss.str());
0172:   }
0173: }
0174: 
0175: 
0176: // NOTE [ Convolution checks ]
0177: //
0178: // NB: For many call sites, it is not strictly necessary to check all of
0179: // these relationships (for example, for forward convolution, we compute
0180: // the size of output ourselves, so we don't actually need to check
```
- **EN**: Lines 151-180 mainly cover comments/documentation, state/variable declarations, function signatures/definitions. Notable symbols: check_args, TORCH_CHECK, size, values.
- **CN**: 第 151-180 行主要涉及注释或说明、变量/别名声明、函数签名或实现。 值得关注的符号包括：check_args, TORCH_CHECK, size, values。

### Lines 181-210 / 第 181-210 行
```cpp
0181: // output.  However, writing a single function that does everything
0182: // means we get to reuse it for both forwards and all backwards
0183: // variants, even when the set of "real" inputs varies.  The magic of
0184: // relational computing!
0185: //
0186: // (There is one downside, which is that it is slightly harder to write
0187: // error messages which are able to distinguish between real inputs
0188: // (which the user can change) and computed inputs (which the user can
0189: // only indirectly affect).  It would be an interesting exercise to
0190: // come up with a general framework to handle such situations.)
0191: inline void convolution_shape_check(
0192:     CheckedFrom c,
0193:     const TensorGeometryArg& input, const TensorGeometryArg& weight, const TensorGeometryArg& output,
0194:     IntArrayRef padding, IntArrayRef stride, IntArrayRef dilation, int64_t groups)
0195: {
0196:   check_args(c, padding, input->dim() - 2, "padding");
0197:   check_args(c, stride, padding.size(), "stride");
0198:   check_args(c, dilation, padding.size(), "dilation");
0199:   for (auto s : stride) {
0200:       TORCH_CHECK(s > 0, "Stride must be greater than 0 but got ", s);
0201:   }
0202:   for (auto d : dilation) {
0203:       TORCH_CHECK(d > 0, "Dilation must be greater than 0 but got ", d);
0204:   }
0205:   for (auto p : padding) {
0206:       TORCH_CHECK(p >= 0, "Padding must be non-negative but got ", p);
0207:   }
0208: 
0209:   // Input
0210:   checkDimRange(c, input, 3, 6 /* exclusive */);
```
- **EN**: Lines 181-210 mainly cover comments/documentation, expressions/calls, state/variable declarations. Notable symbols: inputs, convolution_shape_check, check_args, dim.
- **CN**: 第 181-210 行主要涉及注释或说明、表达式或调用、变量/别名声明。 值得关注的符号包括：inputs, convolution_shape_check, check_args, dim。

### Lines 211-240 / 第 211-240 行
```cpp
0211:   checkSize_symint(c, input, input_channels_dim, weight->size(1) * groups);
0212: 
0213:   // Weight
0214:   checkSameDim(c, input, weight);
0215: 
0216:   // TODO: check that output->size() matches output_sizes
0217:   // TODO: check that weight matches output->sizes()
0218:   checkSameDim(c, input, output);
0219: }
0220: 
0221: // NB: conv_output_size and conv_input_size are not bijections,
0222: // as conv_output_size loses information; this is why conv_input_size
0223: // takes an extra output_padding argument to resolve the ambiguity.
0224: 
0225: template <typename T>
0226: inline std::vector<T> _conv_output_size(
0227:     ArrayRef<T> input_size, ArrayRef<T> weight_size,
0228:     ArrayRef<T> padding, ArrayRef<T> stride, ArrayRef<T> dilation = ArrayRef<T>()
0229: ) {
0230:   // ASSERT(input_size.size() > 2)
0231:   // ASSERT(input_size.size() == weight_size.size())
0232:   bool has_dilation = !dilation.empty();
0233:   auto dim = input_size.size();
0234:   std::vector<T> output_size(dim);
0235:   output_size[0] = input_size[input_batch_size_dim];
0236:   output_size[1] = weight_size[weight_output_channels_dim];
0237:   for (const auto d : c10::irange(2, dim)) {
0238:     auto dilation_ = has_dilation ? dilation[d - 2] : 1;
0239:     auto kernel = dilation_ * (weight_size[d] - 1) + 1;
0240:     output_size[d] = (input_size[d] + (2 * padding[d - 2]) - kernel) / stride[d - 2] + 1;
```
- **EN**: Lines 211-240 mainly cover state/variable declarations, comments/documentation, expressions/calls. Notable symbols: checkSize_symint, size, checkSameDim, sizes.
- **CN**: 第 211-240 行主要涉及变量/别名声明、注释或说明、表达式或调用。 值得关注的符号包括：checkSize_symint, size, checkSameDim, sizes。

### Lines 241-270 / 第 241-270 行
```cpp
0241:   }
0242:   return output_size;
0243: }
0244: 
0245: inline std::vector<int64_t> conv_output_size(
0246:     IntArrayRef input_size, IntArrayRef weight_size,
0247:     IntArrayRef padding, IntArrayRef stride, IntArrayRef dilation = IntArrayRef()
0248: ) {
0249:   return _conv_output_size(input_size, weight_size, padding, stride, dilation);
0250: }
0251: 
0252: inline std::vector<c10::SymInt> conv_output_size(
0253:     SymIntArrayRef input_size, SymIntArrayRef weight_size,
0254:     SymIntArrayRef padding, SymIntArrayRef stride, SymIntArrayRef dilation = SymIntArrayRef()
0255: ) {
0256:   return _conv_output_size(input_size, weight_size, padding, stride, dilation);
0257: }
0258: 
0259: template <typename T>
0260: std::vector<T> _conv_input_size(
0261:     ArrayRef<T> output_size, ArrayRef<T> weight_size,
0262:     ArrayRef<T> padding, ArrayRef<T> output_padding, ArrayRef<T> stride, ArrayRef<T> dilation, T groups
0263: ) {
0264:   // ASSERT(output_size.size() > 2)
0265:   // ASSERT(output_size.size() == weight_size.size())
0266:   auto dim = output_size.size();
0267:   std::vector<T> input_size(dim);
0268:   input_size[0] = output_size[output_batch_size_dim];
0269:   input_size[1] = weight_size[weight_input_channels_dim] * groups;
0270:   for (const auto d : c10::irange(2, dim)) {
```
- **EN**: Lines 241-270 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: conv_output_size, IntArrayRef, _conv_output_size, SymIntArrayRef.
- **CN**: 第 241-270 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：conv_output_size, IntArrayRef, _conv_output_size, SymIntArrayRef。

### Lines 271-300 / 第 271-300 行
```cpp
0271:     auto kernel = (weight_size[d] - 1) * dilation[d - 2] + 1;
0272:     input_size[d] = (output_size[d] - 1) * stride[d - 2] - (padding[d - 2] * 2) +
0273:                      kernel + output_padding[d - 2];
0274:   }
0275:   return input_size;
0276: }
0277: 
0278: inline std::vector<c10::SymInt> conv_input_size(
0279:     SymIntArrayRef output_size, SymIntArrayRef weight_size,
0280:     SymIntArrayRef padding, SymIntArrayRef output_padding, SymIntArrayRef stride, SymIntArrayRef dilation, c10::SymInt groups
0281: ) {
0282:   return _conv_input_size(output_size, weight_size, padding, output_padding, stride, dilation, std::move(groups));
0283: }
0284: 
0285: inline std::vector<int64_t> conv_input_size(
0286:     IntArrayRef output_size, IntArrayRef weight_size,
0287:     IntArrayRef padding, IntArrayRef output_padding, IntArrayRef stride, IntArrayRef dilation, int64_t groups
0288: ) {
0289:   return _conv_input_size(output_size, weight_size, padding, output_padding, stride, dilation, groups);
0290: }
0291: 
0292: template <typename T>
0293: std::vector<T> _conv_weight_size(
0294:     ArrayRef<T> input_size, ArrayRef<T> output_size,
0295:     ArrayRef<T> padding, ArrayRef<T> output_padding, IntArrayRef stride, IntArrayRef dilation, int64_t groups
0296: ) {
0297:   auto dim = input_size.size();
0298:   std::vector<T> weight_size(dim);
0299:   weight_size[0] = output_size[1];
0300:   weight_size[1] = input_size[1] / groups;
```
- **EN**: Lines 271-300 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: conv_input_size, _conv_input_size, move, _conv_weight_size.
- **CN**: 第 271-300 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：conv_input_size, _conv_input_size, move, _conv_weight_size。

### Lines 301-330 / 第 301-330 行
```cpp
0301:   for (const auto d : c10::irange(2, dim)) {
0302:     auto kernel = input_size[d] - (output_size[d] - 1) * stride[d - 2]
0303:                + padding[d - 2] * 2 - output_padding[d - 2];
0304:     weight_size[d] = (kernel - 1) / dilation[d - 2] + 1;
0305:   }
0306:   return weight_size;
0307: }
0308: 
0309: inline std::vector<c10::SymInt> conv_weight_size(
0310:     SymIntArrayRef input_size, SymIntArrayRef output_size,
0311:     SymIntArrayRef padding, SymIntArrayRef output_padding, IntArrayRef stride, IntArrayRef dilation, int64_t groups
0312: ) {
0313:   return _conv_weight_size(input_size, output_size, padding, output_padding, stride, dilation, groups);
0314: }
0315: 
0316: inline std::vector<int64_t> conv_weight_size(
0317:     IntArrayRef input_size, IntArrayRef output_size,
0318:     IntArrayRef padding, IntArrayRef output_padding, IntArrayRef stride, IntArrayRef dilation, int64_t groups
0319: ) {
0320:   return _conv_weight_size(input_size, output_size, padding, output_padding, stride, dilation, groups);
0321: }
0322: 
0323: inline Tensor reshape_bias(int64_t dim, const Tensor& bias) {
0324:   std::vector<int64_t> shape(dim, 1);
0325:   shape[1] = -1;
0326:   return bias.reshape(shape);
0327: }
0328: 
0329: inline at::MemoryFormat cudnn_conv_suggest_memory_format(const at::Tensor& input, const at::Tensor& weight) {
0330:   // disable NHWC for float64 input.
```
- **EN**: Lines 301-330 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: irange, conv_weight_size, _conv_weight_size, reshape_bias.
- **CN**: 第 301-330 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：irange, conv_weight_size, _conv_weight_size, reshape_bias。

### Lines 331-360 / 第 331-360 行
```cpp
0331:   if (!at::detail::getCUDAHooks().compiledWithCuDNN() ||
0332:       input.scalar_type() == at::kDouble ||
0333:       weight.scalar_type() == at::kDouble) {
0334:     return at::MemoryFormat::Contiguous;
0335:   }
0336:   auto input_memory_format = input.suggest_memory_format();
0337:   auto weight_memory_format = weight.suggest_memory_format();
0338:   auto weight_ndim = weight.ndimension();
0339: 
0340:   bool can_use_cudnn_channels_last_2d = weight_ndim == 4 && (
0341:     (input_memory_format  == at::MemoryFormat::ChannelsLast) ||
0342:     (weight_memory_format == at::MemoryFormat::ChannelsLast)
0343:   );
0344:   if (can_use_cudnn_channels_last_2d) {
0345:     return at::MemoryFormat::ChannelsLast;
0346:   }
0347: 
0348:   bool can_use_cudnn_channels_last_3d = weight_ndim == 5 && (
0349:     (input_memory_format  == at::MemoryFormat::ChannelsLast3d) ||
0350:     (weight_memory_format == at::MemoryFormat::ChannelsLast3d)
0351:   );
0352:   if (can_use_cudnn_channels_last_3d) {
0353:     return at::MemoryFormat::ChannelsLast3d;
0354:   }
0355: 
0356:   return at::MemoryFormat::Contiguous;
0357: }
0358: 
0359: // controls whether emptyCache will be called following cudnn conv benchmarking
0360: TORCH_API void _cudnn_set_conv_benchmark_empty_cache(bool enable);
```
- **EN**: Lines 331-360 mainly cover function signatures/definitions, expressions/calls, state/variable declarations. Notable symbols: getCUDAHooks, compiledWithCuDNN, scalar_type, suggest_memory_format.
- **CN**: 第 331-360 行主要涉及函数签名或实现、表达式或调用、变量/别名声明。 值得关注的符号包括：getCUDAHooks, compiledWithCuDNN, scalar_type, suggest_memory_format。

### Lines 361-390 / 第 361-390 行
```cpp
0361: TORCH_API bool _cudnn_get_conv_benchmark_empty_cache();
0362: 
0363: 
0364: inline at::MemoryFormat miopen_conv_suggest_memory_format(const at::Tensor& input, const at::Tensor& weight) {
0365:   // disable NHWC for float64 input.
0366:   if (!at::detail::getCUDAHooks().compiledWithMIOpen() ||
0367:       input.scalar_type() == at::kDouble ||
0368:       weight.scalar_type() == at::kDouble) {
0369:     return at::MemoryFormat::Contiguous;
0370:   }
0371: 
0372:   // TODO: Remove PYTORCH_MIOPEN_SUGGEST_NHWC once ROCm officially supports NHWC in MIOpen
0373:   // See https://github.com/pytorch/pytorch/issues/64427.
0374:   // non static variable is used to be able to change environment variable in runtime for testing
0375:   bool suggest_nhwc = c10::utils::check_env("PYTORCH_MIOPEN_SUGGEST_NHWC").value_or(false);
0376: 
0377:   auto input_memory_format = input.suggest_memory_format();
0378:   auto weight_memory_format = weight.suggest_memory_format();
0379:   auto weight_ndim = weight.ndimension();
0380: 
0381:   bool can_use_miopen_channels_last_2d = suggest_nhwc && (weight_ndim == 4) && (
0382:     (input_memory_format  == at::MemoryFormat::ChannelsLast) ||
0383:     (weight_memory_format == at::MemoryFormat::ChannelsLast)
0384:   );
0385:   if (can_use_miopen_channels_last_2d) {
0386:     return at::MemoryFormat::ChannelsLast;
0387:   }
0388: 
0389:   bool can_use_miopen_channels_last_3d = suggest_nhwc && (weight_ndim == 5) && (
0390:     (input_memory_format  == at::MemoryFormat::ChannelsLast3d) ||
```
- **EN**: Lines 361-390 mainly cover state/variable declarations, function signatures/definitions, comments/documentation. Notable symbols: _cudnn_get_conv_benchmark_empty_cache, miopen_conv_suggest_memory_format, getCUDAHooks, compiledWithMIOpen.
- **CN**: 第 361-390 行主要涉及变量/别名声明、函数签名或实现、注释或说明。 值得关注的符号包括：_cudnn_get_conv_benchmark_empty_cache, miopen_conv_suggest_memory_format, getCUDAHooks, compiledWithMIOpen。

### Lines 391-420 / 第 391-420 行
```cpp
0391:     (weight_memory_format == at::MemoryFormat::ChannelsLast3d)
0392:   );
0393:   if (can_use_miopen_channels_last_3d) {
0394:     return at::MemoryFormat::ChannelsLast3d;
0395:   }
0396: 
0397:   return at::MemoryFormat::Contiguous;
0398: }
0399: 
0400: // deprecated, but to remove would be BC-breaking
0401: inline bool miopen_conv_use_channels_last(const at::Tensor& input, const at::Tensor& weight) {
0402:   return miopen_conv_suggest_memory_format(input, weight) != at::MemoryFormat::Contiguous;
0403: }
0404: 
0405: inline bool mkldnn_conv_use_channels_last(const at::Tensor& input, const at::Tensor& weight) {
0406: 
0407:   // disable NHWC for float64 input.
0408:   if (input.scalar_type() == at::kDouble ||
0409:       weight.scalar_type() == at::kDouble) {
0410:     return false;
0411:   }
0412: 
0413:   // disable NHWC for MkldnnCPU tensor.
0414:   if (input.is_mkldnn() || weight.is_mkldnn()) {
0415:     return false;
0416:   }
0417: 
0418:   auto input_memory_format = input.suggest_memory_format();
0419:   auto weight_memory_format = weight.suggest_memory_format();
0420: 
```
- **EN**: Lines 391-420 mainly cover state/variable declarations, return paths, expressions/calls. Notable symbols: miopen_conv_use_channels_last, miopen_conv_suggest_memory_format, mkldnn_conv_use_channels_last, scalar_type.
- **CN**: 第 391-420 行主要涉及变量/别名声明、返回路径、表达式或调用。 值得关注的符号包括：miopen_conv_use_channels_last, miopen_conv_suggest_memory_format, mkldnn_conv_use_channels_last, scalar_type。

### Lines 421-450 / 第 421-450 行
```cpp
0421:   bool can_use_mkldnn_channels_last_2d =
0422:       (input_memory_format  == at::MemoryFormat::ChannelsLast) ||
0423:       (weight_memory_format == at::MemoryFormat::ChannelsLast);
0424: 
0425:   bool can_use_mkldnn_channels_last_3d =
0426:       (input_memory_format  == at::MemoryFormat::ChannelsLast3d) ||
0427:       (weight_memory_format == at::MemoryFormat::ChannelsLast3d);
0428: 
0429:   return can_use_mkldnn_channels_last_2d || can_use_mkldnn_channels_last_3d;
0430: }
0431: 
0432: inline bool thnn_conv_use_channels_last(const at::Tensor& input, const at::Tensor& weight) {
0433: 
0434:   auto input_memory_format = input.suggest_memory_format();
0435:   auto weight_memory_format = weight.suggest_memory_format();
0436: 
0437:   bool can_use_thnn_channels_last_2d = input.device().is_cpu() && (
0438:       (input_memory_format  == at::MemoryFormat::ChannelsLast) || (
0439:        weight_memory_format == at::MemoryFormat::ChannelsLast));
0440: 
0441:   return can_use_thnn_channels_last_2d;
0442: }
0443: 
0444: inline bool xpu_conv_use_channels_last(const at::Tensor& input, const at::Tensor& weight) {
0445: 
0446:   // check layout only for xpu tensor.
0447:   if (!input.is_xpu() || !weight.is_xpu()) {
0448:     return false;
0449:   }
0450:   if (!input.defined() || input.is_sparse()) {
```
- **EN**: Lines 421-450 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: thnn_conv_use_channels_last, suggest_memory_format, device, is_cpu.
- **CN**: 第 421-450 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：thnn_conv_use_channels_last, suggest_memory_format, device, is_cpu。

### Lines 451-480 / 第 451-480 行
```cpp
0451:     // suggest channels_first
0452:     return false;
0453:   }
0454: 
0455:   auto is_channel_last = [](const at::Tensor& t) {
0456:     auto fmt = t.suggest_memory_format();
0457:     return fmt == at::MemoryFormat::ChannelsLast || fmt == at::MemoryFormat::ChannelsLast3d;
0458:   };
0459:   return is_channel_last(input) || is_channel_last(weight);
0460: }
0461: 
0462: inline bool mps_conv_use_channels_last(const at::Tensor& input, const at::Tensor& weight) {
0463: 
0464:   // check layout only for mps tensor.
0465:   if (!input.is_mps() || !weight.is_mps()) {
0466:     return false;
0467:   }
0468:   if (!input.defined() || input.is_sparse()) {
0469:     // suggest channels_first
0470:     return false;
0471:   }
0472: 
0473:   auto is_channel_last = [](const at::Tensor& t) {
0474:     auto fmt = t.suggest_memory_format();
0475:     return fmt == at::MemoryFormat::ChannelsLast || fmt == at::MemoryFormat::ChannelsLast3d;
0476:   };
0477:   return is_channel_last(input) || is_channel_last(weight);
0478: }
0479: 
0480: } // namespace at::native
```
- **EN**: Lines 451-480 mainly cover return paths, expressions/calls, state/variable declarations. Notable symbols: suggest_memory_format, is_channel_last, mps_conv_use_channels_last, is_mps.
- **CN**: 第 451-480 行主要涉及返回路径、表达式或调用、变量/别名声明。 值得关注的符号包括：suggest_memory_format, is_channel_last, mps_conv_use_channels_last, is_mps。

## Key Concepts / 关键概念
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现
- **EN**: Native operator implementation path  
  **CN**: 原生算子实现路径

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/TensorUtils.h>`, `<ATen/detail/CUDAHooksInterface.h>`, `<ATen/native/DispatchStub.h>`, `<c10/util/env.h>`, `<c10/util/irange.h>`, `<utility>`
- **Macros / 宏**: `TORCH_CHECK`, `TORCH_WARN`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
