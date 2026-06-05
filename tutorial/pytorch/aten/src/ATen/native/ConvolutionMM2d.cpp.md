# ConvolutionMM2d.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/ConvolutionMM2d.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Convolution MM2d.
- **Purpose (CN)**: 实现或声明与 卷积、mm2d 相关的 ATen 原生逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/core/Tensor.h>
0003: #include <ATen/Dispatch.h>
0004: #include <ATen/Parallel.h>
0005: #include <ATen/TensorUtils.h>
0006: #include <ATen/div_rtn.h>
0007: #include <ATen/native/ConvUtils.h>
0008: #include <ATen/native/CPUBlas.h>
0009: #include <ATen/native/Unfold2d.h>
0010: #include <c10/util/irange.h>
0011: 
0012: #ifndef AT_PER_OPERATOR_HEADERS
0013: #include <ATen/Functions.h>
0014: #include <ATen/NativeFunctions.h>
0015: #else
0016: #include <ATen/ops/_slow_conv2d_backward_native.h>
0017: #include <ATen/ops/_slow_conv2d_forward.h>
0018: #include <ATen/ops/_slow_conv2d_forward_native.h>
0019: #include <ATen/ops/empty.h>
0020: #include <ATen/ops/sum.h>
0021: #include <ATen/ops/thnn_conv2d_native.h>
0022: #endif
0023: 
0024: namespace at::native {
0025: 
0026: namespace {
0027: 
0028: Tensor compute_columns2d(
0029:     const Tensor& input,
0030:     IntArrayRef padding,
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, expressions/calls. Notable symbols: compute_columns2d.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、表达式或调用。 值得关注的符号包括：compute_columns2d。

### Lines 31-60 / 第 31-60 行
```cpp
0031:     IntArrayRef stride,
0032:     IntArrayRef kernel_size,
0033:     bool is_channels_last) {
0034:   const int64_t kernel_height = kernel_size[0];
0035:   const int64_t kernel_width = kernel_size[1];
0036:   const int64_t pad_height = padding[0];
0037:   const int64_t pad_width = padding[1];
0038:   const int64_t stride_height = stride[0];
0039:   const int64_t stride_width = stride[1];
0040:   const int64_t batch_size = input.size(0);
0041:   const int64_t n_input_plane = input.size(1);
0042:   const int64_t input_height = input.size(2);
0043:   const int64_t input_width = input.size(3);
0044:   const int64_t output_height = (input_height + 2 * pad_height - kernel_height) / stride_height + 1;
0045:   const int64_t output_width =  (input_width + 2 * pad_width - kernel_width) / stride_width + 1;
0046: 
0047:   Tensor columns;
0048:   if ((kernel_height == 1) && (stride_height == 1) && (pad_height == 0) &&
0049:       (kernel_width == 1) && (stride_width == 1) && (pad_width == 0)) {
0050:     // Columns are just a view on the input for the 1x1 kernel special case.
0051:     if (is_channels_last) {
0052:       columns = input.as_strided({batch_size, output_height * output_width, n_input_plane},
0053:           {output_height * output_width * n_input_plane, n_input_plane, 1}).detach();
0054:     } else {
0055:       columns = input.view({batch_size, n_input_plane, output_height * output_width}).detach();
0056:     }
0057:   } else {
0058:     int64_t row = is_channels_last ?
0059:         output_height * output_width : n_input_plane * kernel_height * kernel_width;
0060:     int64_t col = is_channels_last ?
```
- **EN**: Lines 31-60 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: size, as_strided, detach, view.
- **CN**: 第 31-60 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：size, as_strided, detach, view。

### Lines 61-90 / 第 61-90 行
```cpp
0061:         kernel_height * kernel_width * n_input_plane : output_height * output_width;
0062:     columns = at::empty({batch_size, row, col}, input.options());
0063:     AT_DISPATCH_ALL_TYPES_AND2(kBFloat16, kHalf, input.scalar_type(), "slow_conv2d_cpu", [&]{
0064:       auto input_a = input.accessor<const scalar_t, 4>();
0065:       auto columns_a = columns.accessor<scalar_t, 3>();
0066: 
0067:       at::parallel_for(0, batch_size, 0, [&](int64_t start, int64_t end) {
0068:         for (const auto t : c10::irange(start, end)) {
0069:           auto input_t = input_a[t];
0070:           auto columns_t = columns_a[t];
0071:           unfolded2d_copy_stub(
0072:               kCPU,
0073:               c10::CppTypeToScalarType<scalar_t>::value,
0074:               columns_t.data(),
0075:               input_t.data(),
0076:               kernel_height,
0077:               kernel_width,
0078:               stride_height,
0079:               stride_width,
0080:               pad_height,
0081:               pad_width,
0082:               n_input_plane,
0083:               input_height,
0084:               input_width,
0085:               output_height,
0086:               output_width,
0087:               is_channels_last);
0088:         }
0089:       });
0090:     });
```
- **EN**: Lines 61-90 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: empty, options, AT_DISPATCH_ALL_TYPES_AND2, scalar_type.
- **CN**: 第 61-90 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：empty, options, AT_DISPATCH_ALL_TYPES_AND2, scalar_type。

### Lines 91-120 / 第 91-120 行
```cpp
0091:   }
0092: 
0093:   return columns.contiguous();
0094: }
0095: 
0096: inline void slow_conv2d_shape_check(
0097:     const Tensor& input,
0098:     const Tensor& grad_output,
0099:     const Tensor& weight,
0100:     const Tensor& bias,
0101:     int64_t kernel_height,
0102:     int64_t kernel_width,
0103:     int64_t stride_height,
0104:     int64_t stride_width,
0105:     int64_t pad_height,
0106:     int64_t pad_width,
0107:     bool weight_optional) {
0108:   TORCH_CHECK(
0109:       kernel_width > 0 && kernel_height > 0,
0110:       "kernel size should be greater than zero, but got kernel_height: ",
0111:       kernel_height,
0112:       " kernel_width: ",
0113:       kernel_width);
0114:   TORCH_CHECK(
0115:       stride_width > 0 && stride_height > 0,
0116:       "stride should be greater than zero, but got stride_height: ",
0117:       stride_height,
0118:       " stride_width: ",
0119:       stride_width);
0120: 
```
- **EN**: Lines 91-120 mainly cover expressions/calls, state/variable declarations, macro-based glue. Notable symbols: contiguous, slow_conv2d_shape_check, TORCH_CHECK.
- **CN**: 第 91-120 行主要涉及表达式或调用、变量/别名声明、宏定义或宏调用。 值得关注的符号包括：contiguous, slow_conv2d_shape_check, TORCH_CHECK。

### Lines 121-150 / 第 121-150 行
```cpp
0121:   if (weight.defined()) {
0122:     TORCH_CHECK(
0123:         weight.numel() > 0 && (weight.dim() == 2 || weight.dim() == 4),
0124:         "non-empty 2D or 4D weight tensor expected, but got: ",
0125:         weight.sizes());
0126:     if (bias.defined()) {
0127:       check_dim_size(bias, 1, 0, weight.size(0));
0128:     }
0129:   } else {
0130:     TORCH_CHECK(weight_optional, "weight tensor is undefined");
0131:   }
0132: 
0133:   const int64_t ndim = input.dim();
0134:   const int64_t dim_planes = 1;
0135:   const int64_t dim_height = 2;
0136:   const int64_t dim_width = 3;
0137: 
0138:   // Allow for empty batch size and channel size but not other dimensions
0139:   TORCH_CHECK(ndim == 4, "Expected 4D input tensor, but got: ", input.sizes());
0140:   for (const auto dim : c10::irange(2, ndim)) {
0141:     TORCH_CHECK(input.size(dim) != 0,
0142:                 "Expected non-zero size for input dimension ", dim,
0143:                 ", but got input shape: ", input.sizes(), ". Only the batch and channel dimensions support size 0.");
0144:   }
0145: 
0146:   const int64_t input_height = input.size(dim_height);
0147:   const int64_t input_width = input.size(dim_width);
0148: 
0149:   const int64_t exact_input_height = input_height + 2 * pad_height;
0150:   const int64_t exact_input_width = input_width + 2 * pad_width;
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: defined, TORCH_CHECK, numel, dim.
- **CN**: 第 121-150 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：defined, TORCH_CHECK, numel, dim。

### Lines 151-180 / 第 151-180 行
```cpp
0151: 
0152:   TORCH_CHECK(
0153:       exact_input_height >= kernel_height && exact_input_width >= kernel_width,
0154:       "Calculated padded input size per channel: (",
0155:       exact_input_height,
0156:       " x ",
0157:       exact_input_width,
0158:       "). ",
0159:       "Kernel size: (",
0160:       kernel_height,
0161:       " x ",
0162:       kernel_width,
0163:       "). Kernel size can't be greater than actual input size");
0164: 
0165:   const int64_t output_height =
0166:       div_rtn<int64_t>(exact_input_height - kernel_height, stride_height) + 1;
0167:   const int64_t output_width =
0168:       div_rtn<int64_t>(exact_input_width - kernel_width, stride_width) + 1;
0169: 
0170:   TORCH_CHECK(
0171:       output_width >= 1 && output_height >= 1,
0172:       "Given input size per channel: (",
0173:       input_height,
0174:       " x ",
0175:       input_width,
0176:       "). "
0177:       "Calculated output size per channel: (",
0178:       output_height,
0179:       " x ",
0180:       output_width,
```
- **EN**: Lines 151-180 mainly cover expressions/calls, state/variable declarations, macro-based glue. Notable symbols: TORCH_CHECK, channel:, size:.
- **CN**: 第 151-180 行主要涉及表达式或调用、变量/别名声明、宏定义或宏调用。 值得关注的符号包括：TORCH_CHECK, channel:, size:。

### Lines 181-210 / 第 181-210 行
```cpp
0181:       "). Output size is too small");
0182: 
0183:   if (weight.defined()) {
0184:     int64_t n_input_plane = weight.size(1);
0185:     if (weight.dim() == 2) {
0186:       n_input_plane /= kernel_height;
0187:       n_input_plane /= kernel_width;
0188:     }
0189:     if (input.size(1) != 0) {
0190:       check_dim_size(input, ndim, dim_planes, n_input_plane);
0191:     }
0192:   }
0193: 
0194:   if (grad_output.defined()) {
0195:     if (weight.defined()) {
0196:       int64_t n_output_plane = weight.size(0);
0197:       check_dim_size(grad_output, ndim, dim_planes, n_output_plane);
0198:     } else if (bias.defined()) {
0199:       TORCH_CHECK(bias.numel() > 0, "non-empty bias tensor expected");
0200:       const int64_t n_output_plane = bias.dim() == 0 ? 1 : bias.size(0);
0201:       check_dim_size(grad_output, ndim, dim_planes, n_output_plane);
0202:     }
0203:     check_dim_size(grad_output, ndim, dim_height, output_height);
0204:     check_dim_size(grad_output, ndim, dim_width, output_width);
0205:   }
0206: }
0207: 
0208: inline Tensor view_weight_2d(const Tensor& weight_,
0209:     at::MemoryFormat memory_format = at::MemoryFormat::Contiguous) {
0210:   Tensor weight = weight_.contiguous(memory_format);
```
- **EN**: Lines 181-210 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: defined, size, dim, check_dim_size.
- **CN**: 第 181-210 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：defined, size, dim, check_dim_size。

### Lines 211-240 / 第 211-240 行
```cpp
0211:   if (weight.dim() == 4) {
0212:     const int64_t s1 = weight.size(0);
0213:     const int64_t s2 = weight.size(1) * weight.size(2) * weight.size(3);
0214:     return memory_format == at::MemoryFormat::ChannelsLast
0215:         ? weight.as_strided({s1, s2}, {s2, 1}) // CL: view as {oc, kh*kw*ic}
0216:         : weight.view({s1, s2}); // CF: view as {oc, ic*kh*kw}
0217:   } else {
0218:     return weight;
0219:   }
0220: }
0221: 
0222: template <typename scalar_t>
0223: void slow_conv2d_update_output_frame(
0224:     TensorAccessor<const scalar_t, 3> input,
0225:     TensorAccessor<scalar_t, 3> output,
0226:     TensorAccessor<const scalar_t, 2> weight,
0227:     bool has_bias,
0228:     TensorAccessor<scalar_t, 2> finput,
0229:     int64_t kernel_height,
0230:     int64_t kernel_width,
0231:     int64_t stride_height,
0232:     int64_t stride_width,
0233:     int64_t pad_height,
0234:     int64_t pad_width,
0235:     int64_t n_input_plane,
0236:     int64_t input_height,
0237:     int64_t input_width,
0238:     int64_t n_output_plane,
0239:     int64_t output_height,
0240:     int64_t output_width,
```
- **EN**: Lines 211-240 mainly cover expressions/calls, function signatures/definitions, state/variable declarations. Notable symbols: dim, size, as_strided, view.
- **CN**: 第 211-240 行主要涉及表达式或调用、函数签名或实现、变量/别名声明。 值得关注的符号包括：dim, size, as_strided, view。

### Lines 241-270 / 第 241-270 行
```cpp
0241:     bool is_channels_last) {
0242:   const int beta = has_bias ? 1 : 0;
0243: 
0244:   // Compute out = weight * input
0245:   // Note gemm expects fortran order, so all 3 matrices are transposed.
0246:   // Swapping argument order cancels this, since C == AB <=> T(C) == T(B)T(A)
0247:   if (is_channels_last) {
0248:     const int64_t m = n_output_plane;
0249:     const int64_t n = output_height * output_width;
0250:     const int64_t k = n_input_plane * kernel_height * kernel_width;
0251: 
0252:     const int64_t lda = k;
0253:     const int64_t ldb = k;
0254:     const int64_t ldc = m;
0255: 
0256:     at::native::cpublas::gemm(
0257:         TransposeType::Transpose,
0258:         TransposeType::NoTranspose,
0259:         m, n, k,
0260:         static_cast<scalar_t>(1),
0261:         weight.data(), lda,
0262:         finput.data(), ldb,
0263:         static_cast<scalar_t>(beta),
0264:         output.data(), ldc);
0265:   } else {
0266:     const int64_t m = output_height * output_width;
0267:     const int64_t n = n_output_plane;
0268:     const int64_t k = n_input_plane * kernel_height * kernel_width;
0269: 
0270:     const int64_t lda = m;
```
- **EN**: Lines 241-270 mainly cover state/variable declarations, function signatures/definitions, expressions/calls. Notable symbols: T, gemm, data.
- **CN**: 第 241-270 行主要涉及变量/别名声明、函数签名或实现、表达式或调用。 值得关注的符号包括：T, gemm, data。

### Lines 271-300 / 第 271-300 行
```cpp
0271:     const int64_t ldb = k;
0272:     const int64_t ldc = m;
0273: 
0274:     at::native::cpublas::gemm(
0275:         TransposeType::NoTranspose,
0276:         TransposeType::NoTranspose,
0277:         m, n, k,
0278:         static_cast<scalar_t>(1),
0279:         finput.data(), lda,
0280:         weight.data(), ldb,
0281:         static_cast<scalar_t>(beta),
0282:         output.data(), ldc);
0283:   }
0284: }
0285: 
0286: template <typename scalar_t>
0287: void slow_conv2d_backward_update_grad_input_frame(
0288:     TensorAccessor<scalar_t, 3> grad_input,
0289:     TensorAccessor<const scalar_t, 3> grad_output,
0290:     TensorAccessor<const scalar_t, 2> weight,
0291:     scalar_t *fgrad_input,
0292:     int64_t kernel_height,
0293:     int64_t kernel_width,
0294:     int64_t stride_height,
0295:     int64_t stride_width,
0296:     int64_t pad_height,
0297:     int64_t pad_width,
0298:     bool is_channels_last) {
0299:   // Compute fgrad_input = weight.T * grad_output.reshape({grad_output.shape(0), -1})
0300:   // Note gemm expects fortran order, so all 3 matrices are transposed.
```
- **EN**: Lines 271-300 mainly cover expressions/calls, function signatures/definitions, state/variable declarations. Notable symbols: gemm, data, slow_conv2d_backward_update_grad_input_frame, reshape.
- **CN**: 第 271-300 行主要涉及表达式或调用、函数签名或实现、变量/别名声明。 值得关注的符号包括：gemm, data, slow_conv2d_backward_update_grad_input_frame, reshape。

### Lines 301-330 / 第 301-330 行
```cpp
0301:   // Swapping argument order cancels this, since C == AB <=> T(C) == T(B)T(A)
0302:   if (is_channels_last) {
0303:     const int64_t m = weight.size(1);
0304:     const int64_t n = grad_output.size(1) * grad_output.size(2);
0305:     const int64_t k = weight.size(0);
0306: 
0307:     const int64_t lda = m;
0308:     const int64_t ldb = k;
0309:     const int64_t ldc = m;
0310: 
0311:     at::native::cpublas::gemm(
0312:         TransposeType::NoTranspose,
0313:         TransposeType::NoTranspose,
0314:         m, n, k,
0315:         static_cast<scalar_t>(1),
0316:         weight.data(), lda,
0317:         grad_output.data(), ldb,
0318:         static_cast<scalar_t>(0),
0319:         fgrad_input, ldc);
0320:   } else {
0321:     const int64_t m = grad_output.size(1) * grad_output.size(2);
0322:     const int64_t n = weight.size(1);
0323:     const int64_t k = weight.size(0);
0324: 
0325:     const int64_t lda = m;
0326:     const int64_t ldb = n;
0327:     const int64_t ldc = m;
0328: 
0329:     at::native::cpublas::gemm(
0330:         TransposeType::NoTranspose,
```
- **EN**: Lines 301-330 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: T, size, gemm, data.
- **CN**: 第 301-330 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：T, size, gemm, data。

### Lines 331-360 / 第 331-360 行
```cpp
0331:         TransposeType::Transpose,
0332:         m, n, k,
0333:         static_cast<scalar_t>(1),
0334:         grad_output.data(), lda,
0335:         weight.data(), ldb,
0336:         static_cast<scalar_t>(0),
0337:         fgrad_input, ldc);
0338:   }
0339: 
0340:   unfolded2d_acc_stub(
0341:       kCPU,
0342:       c10::CppTypeToScalarType<scalar_t>::value,
0343:       fgrad_input,
0344:       grad_input.data(),
0345:       kernel_height,
0346:       kernel_width,
0347:       stride_height,
0348:       stride_width,
0349:       pad_height,
0350:       pad_width,
0351:       grad_input.size(0),
0352:       grad_input.size(1),
0353:       grad_input.size(2),
0354:       grad_output.size(1),
0355:       grad_output.size(2),
0356:       is_channels_last);
0357: }
0358: 
0359: void slow_conv2d_backward_out_cpu_template(
0360:     Tensor& grad_input,
```
- **EN**: Lines 331-360 mainly cover expressions/calls, function signatures/definitions, state/variable declarations. Notable symbols: data, unfolded2d_acc_stub, size, slow_conv2d_backward_out_cpu_template.
- **CN**: 第 331-360 行主要涉及表达式或调用、函数签名或实现、变量/别名声明。 值得关注的符号包括：data, unfolded2d_acc_stub, size, slow_conv2d_backward_out_cpu_template。

### Lines 361-390 / 第 361-390 行
```cpp
0361:     const Tensor& grad_output_,
0362:     const Tensor& input_,
0363:     const Tensor& weight_,
0364:     IntArrayRef kernel_size,
0365:     IntArrayRef stride,
0366:     IntArrayRef padding) {
0367:   const int64_t kernel_height = kernel_size[0];
0368:   const int64_t kernel_width = kernel_size[1];
0369:   const int64_t pad_height = padding[0];
0370:   const int64_t pad_width = padding[1];
0371:   const int64_t stride_height = stride[0];
0372:   const int64_t stride_width = stride[1];
0373: 
0374:   bool use_channels_last = thnn_conv_use_channels_last(input_, weight_);
0375:   auto memory_format = use_channels_last ? at::MemoryFormat::ChannelsLast : at::MemoryFormat::Contiguous;
0376: 
0377:   const Tensor weight = view_weight_2d(weight_, memory_format);
0378:   slow_conv2d_shape_check(
0379:       input_,
0380:       grad_output_,
0381:       weight,
0382:       Tensor(),
0383:       kernel_height,
0384:       kernel_width,
0385:       stride_height,
0386:       stride_width,
0387:       pad_height,
0388:       pad_width,
0389:       false);
0390: 
```
- **EN**: Lines 361-390 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: thnn_conv_use_channels_last, view_weight_2d, slow_conv2d_shape_check, Tensor.
- **CN**: 第 361-390 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：thnn_conv_use_channels_last, view_weight_2d, slow_conv2d_shape_check, Tensor。

### Lines 391-420 / 第 391-420 行
```cpp
0391:   const Tensor input = input_.contiguous(memory_format);
0392: 
0393:   // Compute shape of columnized data excluding batch dim.
0394:   const int64_t batch_size = input.size(0);
0395:   const int64_t n_input_plane = input.size(1);
0396:   const int64_t input_height = input.size(2);
0397:   const int64_t input_width = input.size(3);
0398:   const int64_t output_height = (input_height + 2 * pad_height - kernel_height) / stride_height + 1;
0399:   const int64_t output_width = (input_width + 2 * pad_width - kernel_width) / stride_width + 1;
0400:   const int64_t fgrad_input_size = n_input_plane * kernel_height * kernel_width * output_height * output_width;
0401: 
0402:   const Tensor grad_output = grad_output_.contiguous(memory_format);
0403:   grad_input.resize_as_(input, memory_format);
0404:   grad_input.zero_();
0405:   TORCH_CHECK(grad_input.is_contiguous(memory_format), "slow_conv2d: grad_input must be contiguous");
0406: 
0407:   AT_DISPATCH_FLOATING_TYPES_AND2(
0408:       kBFloat16, kHalf, input.scalar_type(), "slow_conv2d_cpu_grad_input", [&] {
0409:     auto grad_output_a = grad_output.accessor<const scalar_t, 4>();
0410:     auto grad_input_a = grad_input.accessor<scalar_t, 4>();
0411:     auto weight_a = weight.accessor<const scalar_t, 2>();
0412: 
0413:     at::parallel_for(0, batch_size, 0, [&](int64_t start, int64_t end) {
0414:       auto fgrad_input = std::make_unique<scalar_t[]>(fgrad_input_size);
0415:       for (const auto t : c10::irange(start, end)) {
0416:         auto grad_input_t = grad_input_a[t];
0417:         auto grad_output_t = grad_output_a[t];
0418:         slow_conv2d_backward_update_grad_input_frame(
0419:             grad_input_t,
0420:             grad_output_t,
```
- **EN**: Lines 391-420 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: contiguous, size, resize_as_, zero_.
- **CN**: 第 391-420 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：contiguous, size, resize_as_, zero_。

### Lines 421-450 / 第 421-450 行
```cpp
0421:             weight_a,
0422:             fgrad_input.get(),
0423:             kernel_height,
0424:             kernel_width,
0425:             stride_height,
0426:             stride_width,
0427:             pad_height,
0428:             pad_width,
0429:             use_channels_last);
0430:       }
0431:     });
0432:   });
0433: }
0434: 
0435: template <typename scalar_t>
0436: void slow_conv2d_backward_weight_frame(
0437:     TensorAccessor<scalar_t, 2> grad_weight,
0438:     TensorAccessor<const scalar_t, 3> grad_output,
0439:     TensorAccessor<const scalar_t, 2> finput,
0440:     bool is_channels_last) {
0441:   // Compute grad_weight += grad_output.reshape({grad_output.shape(0), -1}) * finput.T
0442:   // Note gemm expects fortran order, so all 3 matrices are transposed.
0443:   // Swapping argument order cancels this, since C == AB <=> T(C) == T(B)T(A)
0444:   if (is_channels_last) {
0445:     const int64_t m = finput.size(1);
0446:     const int64_t n = grad_output.size(0);
0447:     const int64_t k = grad_output.size(1) * grad_output.size(2);
0448: 
0449:     const int64_t lda = m;
0450:     const int64_t ldb = n;
```
- **EN**: Lines 421-450 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: get, slow_conv2d_backward_weight_frame, reshape, shape.
- **CN**: 第 421-450 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：get, slow_conv2d_backward_weight_frame, reshape, shape。

### Lines 451-480 / 第 451-480 行
```cpp
0451:     const int64_t ldc = m;
0452: 
0453:     at::native::cpublas::gemm(
0454:         TransposeType::NoTranspose,
0455:         TransposeType::Transpose,
0456:         m, n, k,
0457:         static_cast<scalar_t>(1),
0458:         finput.data(), lda,
0459:         grad_output.data(), ldb,
0460:         static_cast<scalar_t>(1),
0461:         grad_weight.data(), ldc);
0462:   } else {
0463:     const int64_t m = finput.size(0);
0464:     const int64_t n = grad_output.size(0);
0465:     const int64_t k = grad_output.size(1) * grad_output.size(2);
0466: 
0467:     const int64_t lda = k;
0468:     const int64_t ldb = k;
0469:     const int64_t ldc = m;
0470: 
0471:     at::native::cpublas::gemm(
0472:         TransposeType::Transpose,
0473:         TransposeType::NoTranspose,
0474:         m, n, k,
0475:         static_cast<scalar_t>(1),
0476:         finput.data(), lda,
0477:         grad_output.data(), ldb,
0478:         static_cast<scalar_t>(1),
0479:         grad_weight.data(), ldc);
0480:   }
```
- **EN**: Lines 451-480 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: gemm, data, size.
- **CN**: 第 451-480 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：gemm, data, size。

### Lines 481-510 / 第 481-510 行
```cpp
0481: }
0482: 
0483: void slow_conv2d_backward_weight_out_cpu_template(
0484:     Tensor& grad_weight,
0485:     const Tensor& input,
0486:     const Tensor& grad_output_,
0487:     IntArrayRef kernel_size,
0488:     IntArrayRef stride,
0489:     IntArrayRef padding) {
0490:   const int64_t kernel_height = kernel_size[0];
0491:   const int64_t kernel_width = kernel_size[1];
0492:   const int64_t pad_height = padding[0];
0493:   const int64_t pad_width = padding[1];
0494:   const int64_t stride_height = stride[0];
0495:   const int64_t stride_width = stride[1];
0496: 
0497:   bool use_channels_last = thnn_conv_use_channels_last(input, grad_weight);
0498:   auto memory_format = use_channels_last ? at::MemoryFormat::ChannelsLast : at::MemoryFormat::Contiguous;
0499: 
0500:   TORCH_CHECK(grad_weight.is_contiguous(memory_format), "slow_conv2d: grad_weight must be contiguous");
0501:   Tensor grad_weight_2d = view_weight_2d(grad_weight, memory_format);
0502: 
0503:   slow_conv2d_shape_check(
0504:       input,
0505:       grad_output_,
0506:       grad_weight_2d,
0507:       {},
0508:       kernel_height,
0509:       kernel_width,
0510:       stride_height,
```
- **EN**: Lines 481-510 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: slow_conv2d_backward_weight_out_cpu_template, thnn_conv_use_channels_last, TORCH_CHECK, is_contiguous.
- **CN**: 第 481-510 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：slow_conv2d_backward_weight_out_cpu_template, thnn_conv_use_channels_last, TORCH_CHECK, is_contiguous。

### Lines 511-540 / 第 511-540 行
```cpp
0511:       stride_width,
0512:       pad_height,
0513:       pad_width,
0514:       true);
0515: 
0516:   auto grad_output = grad_output_.contiguous(memory_format);
0517:   Tensor finput = compute_columns2d(input, padding, stride, kernel_size, use_channels_last);
0518: 
0519:   const int64_t batch_size = input.size(0);
0520: 
0521:   AT_DISPATCH_FLOATING_TYPES_AND2(
0522:       kBFloat16, kHalf, input.scalar_type(), "slow_conv2d_cpu_grad_weight", [&] {
0523:     auto grad_output_a = grad_output.accessor<const scalar_t, 4>();
0524:     auto grad_weight_2d_a = grad_weight_2d.accessor<scalar_t, 2>();
0525:     auto finput_a = finput.accessor<const scalar_t, 3>();
0526: 
0527:     for (const auto t : c10::irange(batch_size)) {
0528:       auto grad_output_t = grad_output_a[t];
0529:       auto finput_t = finput_a[t];
0530: 
0531:       slow_conv2d_backward_weight_frame(
0532:           grad_weight_2d_a, grad_output_t, finput_t, use_channels_last);
0533:     }
0534:   });
0535: }
0536: 
0537: } // namespace
0538: 
0539: Tensor& slow_conv2d_forward_out_cpu(
0540:     const Tensor& self,
```
- **EN**: Lines 511-540 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: contiguous, compute_columns2d, size, AT_DISPATCH_FLOATING_TYPES_AND2.
- **CN**: 第 511-540 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：contiguous, compute_columns2d, size, AT_DISPATCH_FLOATING_TYPES_AND2。

### Lines 541-570 / 第 541-570 行
```cpp
0541:     const Tensor& weight_,
0542:     IntArrayRef kernel_size, const std::optional<Tensor>& bias_opt,
0543:     IntArrayRef stride,
0544:     IntArrayRef padding,
0545:     Tensor& output) {
0546:   // See [Note: hacky wrapper removal for optional tensor]
0547: 
0548:   TORCH_CHECK(kernel_size.size() == 2, "2D kernel_size expected");
0549:   TORCH_CHECK(stride.size() == 2, "2D stride expected");
0550:   TORCH_CHECK(padding.size() == 2, "2D padding expected");
0551: 
0552:   c10::MaybeOwned<Tensor> bias_maybe_owned = at::borrow_from_optional_tensor(bias_opt);
0553:   const Tensor& bias = *bias_maybe_owned;
0554: 
0555:   const int64_t kernel_height = kernel_size[0];
0556:   const int64_t kernel_width = kernel_size[1];
0557:   const int64_t pad_height = padding[0];
0558:   const int64_t pad_width = padding[1];
0559:   const int64_t stride_height = stride[0];
0560:   const int64_t stride_width = stride[1];
0561: 
0562:   bool use_channels_last = thnn_conv_use_channels_last(self, weight_);
0563:   auto memory_format = use_channels_last ? at::MemoryFormat::ChannelsLast : at::MemoryFormat::Contiguous;
0564: 
0565:   const Tensor weight_2d = view_weight_2d(weight_, memory_format);
0566: 
0567:   slow_conv2d_shape_check(
0568:       self,
0569:       Tensor(),
0570:       weight_2d,
```
- **EN**: Lines 541-570 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: TORCH_CHECK, size, borrow_from_optional_tensor, thnn_conv_use_channels_last.
- **CN**: 第 541-570 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：TORCH_CHECK, size, borrow_from_optional_tensor, thnn_conv_use_channels_last。

### Lines 571-600 / 第 571-600 行
```cpp
0571:       bias,
0572:       kernel_height,
0573:       kernel_width,
0574:       stride_height,
0575:       stride_width,
0576:       pad_height,
0577:       pad_width,
0578:       false);
0579: 
0580:   const Tensor input = self.contiguous(memory_format);
0581:   const int64_t batch_size = input.size(0);
0582:   const int64_t n_input_plane = input.size(1);
0583:   const int64_t input_height = input.size(2);
0584:   const int64_t input_width = input.size(3);
0585:   const int64_t n_output_plane = weight_2d.size(0);
0586:   const int64_t output_height = (input_height + 2 * pad_height - kernel_height) / stride_height + 1;
0587:   const int64_t output_width = (input_width + 2 * pad_width - kernel_width) / stride_width + 1;
0588: 
0589:   Tensor finput = compute_columns2d(input, padding, stride, kernel_size, use_channels_last);
0590:   output.resize_({batch_size, n_output_plane, output_height, output_width}, memory_format);
0591:   if (bias.defined()) {
0592:     output.copy_(bias.reshape({-1, 1, 1}));
0593:   }
0594:   TORCH_CHECK(output.is_contiguous(memory_format), "slow_conv2d output tensor must be contiguous");
0595: 
0596:   AT_DISPATCH_ALL_TYPES_AND2(kBFloat16, kHalf, input.scalar_type(), "slow_conv2d_cpu", [&]{
0597:     auto input_a = input.accessor<const scalar_t, 4>();
0598:     auto output_a = output.accessor<scalar_t, 4>();
0599:     auto finput_a = finput.accessor<scalar_t, 3>();
0600:     auto weight_2d_a = weight_2d.accessor<const scalar_t, 2>();
```
- **EN**: Lines 571-600 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: contiguous, size, compute_columns2d, resize_.
- **CN**: 第 571-600 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：contiguous, size, compute_columns2d, resize_。

### Lines 601-630 / 第 601-630 行
```cpp
0601: 
0602:     at::parallel_for(0, batch_size, 0, [&](int64_t start, int64_t end) {
0603:       for (const auto t : c10::irange(start, end)) {
0604:         auto input_t = input_a[t];
0605:         auto output_t = output_a[t];
0606:         auto finput_t = finput_a[t];
0607:         slow_conv2d_update_output_frame(
0608:             input_t,
0609:             output_t,
0610:             weight_2d_a,
0611:             bias.defined(),
0612:             finput_t,
0613:             kernel_height,
0614:             kernel_width,
0615:             stride_height,
0616:             stride_width,
0617:             pad_height,
0618:             pad_width,
0619:             n_input_plane,
0620:             input_height,
0621:             input_width,
0622:             n_output_plane,
0623:             output_height,
0624:             output_width,
0625:             use_channels_last);
0626:       }
0627:     });
0628:   });
0629: 
0630:   return output;
```
- **EN**: Lines 601-630 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: parallel_for, irange, slow_conv2d_update_output_frame, defined.
- **CN**: 第 601-630 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：parallel_for, irange, slow_conv2d_update_output_frame, defined。

### Lines 631-660 / 第 631-660 行
```cpp
0631: }
0632: 
0633: Tensor slow_conv2d_forward_cpu(
0634:     const Tensor& self,
0635:     const Tensor& weight,
0636:     IntArrayRef kernel_size, const std::optional<Tensor>& bias_opt,
0637:     IntArrayRef stride,
0638:     IntArrayRef padding) {
0639:   // See [Note: hacky wrapper removal for optional tensor]
0640:   c10::MaybeOwned<Tensor> bias_maybe_owned = at::borrow_from_optional_tensor(bias_opt);
0641:   const Tensor& bias = *bias_maybe_owned;
0642: 
0643:   auto output = at::empty({0}, self.options());
0644:   at::native::slow_conv2d_forward_out_cpu(
0645:       self,
0646:       weight,
0647:       kernel_size,
0648:       bias,
0649:       stride,
0650:       padding,
0651:       output);
0652: 
0653:   return output;
0654: }
0655: 
0656: std::tuple<Tensor&, Tensor&, Tensor&> slow_conv2d_backward_out_cpu(
0657:     const Tensor& grad_output,
0658:     const Tensor& self,
0659:     const Tensor& weight,
0660:     IntArrayRef kernel_size,
```
- **EN**: Lines 631-660 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: slow_conv2d_forward_cpu, borrow_from_optional_tensor, empty, options.
- **CN**: 第 631-660 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：slow_conv2d_forward_cpu, borrow_from_optional_tensor, empty, options。

### Lines 661-690 / 第 661-690 行
```cpp
0661:     IntArrayRef stride,
0662:     IntArrayRef padding,
0663:     Tensor& grad_input,
0664:     Tensor& grad_weight,
0665:     Tensor& grad_bias) {
0666:   if (grad_input.defined()) {
0667:     slow_conv2d_backward_out_cpu_template(
0668:         grad_input,
0669:         grad_output,
0670:         self,
0671:         weight,
0672:         kernel_size,
0673:         stride,
0674:         padding);
0675:   }
0676: 
0677:   if (grad_bias.defined()) {
0678:     at::sum_out(grad_bias, grad_output, IntArrayRef{0, 2, 3});
0679:   }
0680: 
0681:   if (grad_weight.defined()) {
0682:     grad_weight.resize_(weight.sizes(), weight.suggest_memory_format());
0683:     grad_weight.zero_();
0684:     slow_conv2d_backward_weight_out_cpu_template(
0685:         grad_weight,
0686:         self,
0687:         grad_output,
0688:         kernel_size,
0689:         stride,
0690:         padding);
```
- **EN**: Lines 661-690 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: defined, slow_conv2d_backward_out_cpu_template, sum_out, resize_.
- **CN**: 第 661-690 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：defined, slow_conv2d_backward_out_cpu_template, sum_out, resize_。

### Lines 691-720 / 第 691-720 行
```cpp
0691:   }
0692: 
0693:   return std::tuple<Tensor&, Tensor&, Tensor&>(
0694:       grad_input, grad_weight, grad_bias);
0695: }
0696: 
0697: std::tuple<Tensor, Tensor, Tensor> slow_conv2d_backward_cpu(
0698:     const Tensor& grad_output,
0699:     const Tensor& self,
0700:     const Tensor& weight,
0701:     IntArrayRef kernel_size,
0702:     IntArrayRef stride,
0703:     IntArrayRef padding,
0704:     std::array<bool, 3> output_mask) {
0705:   Tensor grad_input;
0706:   Tensor grad_weight;
0707:   Tensor grad_bias;
0708: 
0709:   if (output_mask[0]) {
0710:     grad_input = at::empty({0}, grad_output.options());
0711:   }
0712: 
0713:   if (output_mask[1]) {
0714:     grad_weight = at::empty({0}, grad_output.options());
0715:   }
0716: 
0717:   if (output_mask[2]) {
0718:     grad_bias = at::empty({0}, grad_output.options());
0719:   }
0720: 
```
- **EN**: Lines 691-720 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: slow_conv2d_backward_cpu, empty, options.
- **CN**: 第 691-720 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：slow_conv2d_backward_cpu, empty, options。

### Lines 721-750 / 第 721-750 行
```cpp
0721:   at::native::slow_conv2d_backward_out_cpu(
0722:       grad_output,
0723:       self,
0724:       weight,
0725:       kernel_size,
0726:       stride,
0727:       padding,
0728:       grad_input,
0729:       grad_weight,
0730:       grad_bias);
0731: 
0732:   return std::make_tuple(grad_input, grad_weight, grad_bias);
0733: }
0734: 
0735: Tensor & thnn_conv2d_out(const Tensor & self, const Tensor & weight, IntArrayRef kernel_size, const std::optional<Tensor>& bias_opt, IntArrayRef stride, IntArrayRef padding, Tensor & output) {
0736:   // See [Note: hacky wrapper removal for optional tensor]
0737:   c10::MaybeOwned<Tensor> bias_maybe_owned = at::borrow_from_optional_tensor(bias_opt);
0738:   const Tensor& bias = *bias_maybe_owned;
0739: 
0740:   return at::_slow_conv2d_forward_out(output, self, weight, kernel_size, bias, stride, padding);
0741: }
0742: 
0743: Tensor thnn_conv2d(const Tensor & self, const Tensor & weight, IntArrayRef kernel_size, const std::optional<Tensor>& bias_opt, IntArrayRef stride, IntArrayRef padding) {
0744:   // See [Note: hacky wrapper removal for optional tensor]
0745:   c10::MaybeOwned<Tensor> bias_maybe_owned = at::borrow_from_optional_tensor(bias_opt);
0746:   const Tensor& bias = *bias_maybe_owned;
0747: 
0748:   return at::_slow_conv2d_forward(self, weight, kernel_size, bias, stride, padding);
0749: }
0750: 
```
- **EN**: Lines 721-750 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: slow_conv2d_backward_out_cpu, make_tuple, thnn_conv2d_out, borrow_from_optional_tensor.
- **CN**: 第 721-750 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：slow_conv2d_backward_out_cpu, make_tuple, thnn_conv2d_out, borrow_from_optional_tensor。

### Lines 751-751 / 第 751-751 行
```cpp
0751: } // namespace at::native
```
- **EN**: Lines 751-751 mainly cover namespace structuring.
- **CN**: 第 751-751 行主要涉及命名空间组织。

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
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/Parallel.h>`, `<ATen/TensorUtils.h>`, `<ATen/div_rtn.h>`, `<ATen/native/ConvUtils.h>`, `<ATen/native/CPUBlas.h>`, `<ATen/native/Unfold2d.h>`, `<c10/util/irange.h>`, `<ATen/Functions.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `AT_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
