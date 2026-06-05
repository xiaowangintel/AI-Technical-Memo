# FractionalMaxPool2d.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/FractionalMaxPool2d.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Fractional Max Pool2d.
- **Purpose (CN)**: 实现或声明与 fractional、最大值、pool2d 相关的 ATen 原生逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/core/Tensor.h>
0003: #include <ATen/Dispatch.h>
0004: #include <ATen/Parallel.h>
0005: #include <ATen/TensorMeta.h>
0006: #include <ATen/native/FractionalMaxPooling.h>
0007: #include <c10/util/irange.h>
0008: 
0009: #ifndef AT_PER_OPERATOR_HEADERS
0010: #include <ATen/Functions.h>
0011: #include <ATen/NativeFunctions.h>
0012: #else
0013: #include <ATen/ops/fractional_max_pool2d_backward_native.h>
0014: #include <ATen/ops/fractional_max_pool2d_native.h>
0015: #endif
0016: 
0017: namespace at {
0018: 
0019: namespace meta {
0020: TORCH_META_FUNC(fractional_max_pool2d) (
0021:   const at::Tensor& input,
0022:   IntArrayRef pool_size,
0023:   IntArrayRef output_size,
0024:   const at::Tensor& randomSamples
0025: ) {
0026:   TORCH_CHECK(
0027:       pool_size.size() == 2,
0028:       "fractional_max_pool2d: kernel_size must either be a single Int or tuple of Ints")
0029:   TORCH_CHECK(
0030:       output_size.size() == 2,
```
- **EN**: Lines 1-30 mainly cover header inclusion, expressions/calls, macro-based glue. Notable symbols: TORCH_META_FUNC, TORCH_CHECK, size.
- **CN**: 第 1-30 行主要涉及头文件包含、表达式或调用、宏定义或宏调用。 值得关注的符号包括：TORCH_META_FUNC, TORCH_CHECK, size。

### Lines 31-60 / 第 31-60 行
```cpp
0031:       "fractional_max_pool2d: output_size must either be a single Int or tuple of Ints")
0032:   int64_t numBatch = 1;
0033:   int64_t planeDim = 0;
0034:   int64_t heightDim = 1;
0035:   int64_t widthDim = 2;
0036:   int64_t outputH = output_size[0];
0037:   int64_t outputW = output_size[1];
0038:   int64_t poolSizeH = pool_size[0];
0039:   int64_t poolSizeW = pool_size[1];
0040: 
0041:   int64_t ndims = input.ndimension();
0042:   TORCH_CHECK(ndims == 3 || ndims == 4,
0043:               "fractional_max_pool2d(): Expected 3D or 4D tensor, but got: ", input.sizes());
0044:   for (const auto i : c10::irange(1, ndims)) {
0045:     TORCH_CHECK(input.size(i) > 0,
0046:                 "fractional_max_pool2d(): Expected input to have non-zero size for non-batch dimensions, but got",
0047:                 input.sizes(), " with dimension ", i, " being empty.");
0048:   }
0049: 
0050: 
0051:   if (ndims == 4) {
0052:     numBatch = input.size(0);
0053:     planeDim++;
0054:     heightDim++;
0055:     widthDim++;
0056:   }
0057: 
0058:   /* sizes */
0059:   int64_t numPlanes = input.size(planeDim);
0060:   int64_t inputH = input.size(heightDim);
```
- **EN**: Lines 31-60 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: ndimension, TORCH_CHECK, fractional_max_pool2d, sizes.
- **CN**: 第 31-60 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：ndimension, TORCH_CHECK, fractional_max_pool2d, sizes。

### Lines 61-90 / 第 61-90 行
```cpp
0061:   auto inputW = input.size(widthDim);
0062: 
0063:   TORCH_CHECK(outputH + poolSizeH - 1 <= inputH,
0064:     "fractional_max_pool2d(): pool height ", poolSizeH,
0065:     " too large relative to input height ", inputH);
0066:   TORCH_CHECK(outputW + poolSizeW - 1 <= inputW,
0067:     "fractional_max_pool2d(): pool width ", poolSizeW,
0068:     " too large relative to input width ", inputW);
0069: 
0070:   if (ndims == 3) {
0071:     set_output_raw_strided(0, {numPlanes, outputH, outputW}, {}, input.options());
0072:     /* indices will contain the locations for each output point */
0073:     set_output_raw_strided(1, {numPlanes, outputH, outputW}, {}, input.options().dtype(kLong));
0074:   } else {
0075:     set_output_raw_strided(0, {numBatch, numPlanes, outputH, outputW}, {}, input.options());
0076:     /* indices will contain the locations for each output point */
0077:     set_output_raw_strided(1, {numBatch, numPlanes, outputH, outputW}, {}, input.options().dtype(kLong));
0078:   }
0079: }
0080: 
0081: TORCH_META_FUNC(fractional_max_pool2d_backward)(
0082:   const at::Tensor& gradOutput_,
0083:   const at::Tensor& input,
0084:   IntArrayRef pool_size /* unused */,
0085:   IntArrayRef output_size,
0086:   const at::Tensor& indices) {
0087: 
0088:   int64_t numBatch = 1;
0089:   int planeDim = 0;
0090:   int heightDim = 1;
```
- **EN**: Lines 61-90 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: size, TORCH_CHECK, fractional_max_pool2d, set_output_raw_strided.
- **CN**: 第 61-90 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：size, TORCH_CHECK, fractional_max_pool2d, set_output_raw_strided。

### Lines 91-120 / 第 91-120 行
```cpp
0091:   int widthDim = 2;
0092: 
0093:   auto outputH = output_size[0];
0094:   auto outputW = output_size[1];
0095: 
0096:   auto ndims = input.ndimension();
0097:   if (ndims == 4) {
0098:     numBatch = input.size(0);
0099:     planeDim = 1;
0100:     heightDim++;
0101:     widthDim++;
0102:   }
0103: 
0104:   /* sizes */
0105:   auto numPlanes = input.size(planeDim);
0106:   auto inputH = input.size(heightDim);
0107:   auto inputW = input.size(widthDim);
0108: 
0109:   /* get contiguous gradOutput */
0110:   auto gradOutput = gradOutput_.contiguous();
0111: 
0112:   auto expectedOutputShape = IntArrayRef(input.sizes().data(), ndims - 2).vec();
0113:   expectedOutputShape.push_back(outputH);
0114:   expectedOutputShape.push_back(outputW);
0115:   TORCH_CHECK(gradOutput.sizes().equals(expectedOutputShape),
0116:     "fractional_max_pool2d_backward(): gradOutput sizes unexpected");
0117:   TORCH_CHECK(indices.sizes().equals(expectedOutputShape),
0118:     "fractional_max_pool2d_backward(): indices sizes unexpected");
0119: 
0120:   /* resize */
```
- **EN**: Lines 91-120 mainly cover state/variable declarations, comments/documentation, macro-based glue. Notable symbols: ndimension, size, contiguous, IntArrayRef.
- **CN**: 第 91-120 行主要涉及变量/别名声明、注释或说明、宏定义或宏调用。 值得关注的符号包括：ndimension, size, contiguous, IntArrayRef。

### Lines 121-150 / 第 121-150 行
```cpp
0121:   if (ndims == 3) {
0122:     set_output_raw_strided(0, {numPlanes, inputH, inputW}, {}, input.options());
0123:   } else {
0124:     set_output_raw_strided(0, {numBatch, numPlanes, inputH, inputW}, {}, input.options());
0125:   }
0126: }
0127: } // namespace meta
0128: 
0129: namespace native {
0130: namespace {
0131: 
0132: template <typename scalar_t>
0133: void fractional_max_pool2d_out_single_batch_frame(
0134:   const scalar_t* input,
0135:   scalar_t* output,
0136:   int64_t* indices,
0137:   const scalar_t* randomSamples,
0138:   int numPlanes,
0139:   int inputW, int inputH,
0140:   int outputW, int outputH,
0141:   int poolSizeW, int poolSizeH) {
0142:   at::parallel_for(0, numPlanes, 0, [&](int64_t start, int64_t end) {
0143:     for (const auto plane : c10::irange(start, end)) {
0144:       /* each plane contains 2 random samples, one for W and one for H */
0145:       const scalar_t* randomSamplesForPlane = randomSamples + plane * 2;
0146: 
0147:       /* Generate interval sequence */
0148:       auto sequenceW = generate_intervals<scalar_t>(
0149:           randomSamplesForPlane[0], inputW, outputW, poolSizeW);
0150:       auto sequenceH = generate_intervals<scalar_t>(
```
- **EN**: Lines 121-150 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: set_output_raw_strided, options, fractional_max_pool2d_out_single_batch_frame, parallel_for.
- **CN**: 第 121-150 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：set_output_raw_strided, options, fractional_max_pool2d_out_single_batch_frame, parallel_for。

### Lines 151-180 / 第 151-180 行
```cpp
0151:           randomSamplesForPlane[1], inputH, outputH, poolSizeH);
0152: 
0153:       /* loop over output */
0154:       const scalar_t* inputForPlane = input + plane * inputW * inputH;
0155:       scalar_t* outputForPlane = output + plane * outputW * outputH;
0156:       int64_t* indicesForPlane = indices + plane * outputW * outputH;
0157: 
0158:       for (int h = 0; h < outputH; ++h) {
0159:         int inputHStart = sequenceH[h];
0160: 
0161:         for (int w = 0; w < outputW; ++w) {
0162:           int inputWStart = sequenceW[w];
0163: 
0164:           int h2 = inputHStart, w2 = inputWStart;
0165:           scalar_t maxVal = -std::numeric_limits<scalar_t>::infinity();
0166:           int64_t maxIndex = h2 * inputW + w2;
0167: 
0168:           for (h2 = inputHStart; h2 < inputHStart + poolSizeH; ++h2) {
0169:             for (w2 = inputWStart; w2 < inputWStart + poolSizeW; ++w2) {
0170:               AT_ASSERT(h2 >= 0 && h2 < inputH);
0171:               AT_ASSERT(w2 >= 0 && w2 < inputW);
0172: 
0173:               int planeIndex = h2 * inputW + w2;
0174:               scalar_t val = inputForPlane[planeIndex];
0175:               if (val > maxVal || std::isnan(val)) {
0176:                 maxVal = val;
0177:                 maxIndex = planeIndex;
0178:               }
0179:             }
0180:           }
```
- **EN**: Lines 151-180 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: infinity, AT_ASSERT, isnan.
- **CN**: 第 151-180 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：infinity, AT_ASSERT, isnan。

### Lines 181-210 / 第 181-210 行
```cpp
0181: 
0182:           outputForPlane[h * outputW + w] = maxVal;
0183:           indicesForPlane[h * outputW + w] = maxIndex;
0184:         }
0185:       }
0186:     }
0187:   });
0188: }
0189: 
0190: template <typename scalar_t>
0191: void fractional_max_pool2d_out_frame(
0192:   const scalar_t* input,
0193:   scalar_t* output,
0194:   int64_t* indices,
0195:   const scalar_t* randomSamples,
0196:   int numBatch, int numPlanes,
0197:   int inputW, int inputH,
0198:   int outputW, int outputH,
0199:   int poolSizeW, int poolSizeH) {
0200:     if(numBatch == 1) {
0201:       fractional_max_pool2d_out_single_batch_frame<scalar_t>(
0202:         input,
0203:         output,
0204:         indices,
0205:         randomSamples,
0206:         numPlanes, inputW, inputH, outputW, outputH, poolSizeW, poolSizeH
0207:       );
0208:       return;
0209:     }
0210:     at::parallel_for(0, numBatch, 0, [&](int64_t start, int64_t end) {
```
- **EN**: Lines 181-210 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: fractional_max_pool2d_out_frame, parallel_for.
- **CN**: 第 181-210 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：fractional_max_pool2d_out_frame, parallel_for。

### Lines 211-240 / 第 211-240 行
```cpp
0211:       for (const auto batch : c10::irange(start, end)) {
0212:         fractional_max_pool2d_out_single_batch_frame<scalar_t>(
0213:           input + batch * numPlanes * inputH * inputW,
0214:           output + batch * numPlanes * outputH * outputW,
0215:           indices + batch * numPlanes * outputH * outputW,
0216:           randomSamples + batch * numPlanes * 2,
0217:           numPlanes, inputW, inputH, outputW, outputH, poolSizeW, poolSizeH);
0218:       }
0219:     });
0220:   }
0221: 
0222: template <typename scalar_t>
0223: void fractional_max_pool2d_backward_out_single_batch_frame(
0224:   scalar_t* gradInput,
0225:   const scalar_t* gradOutput,
0226:   const int64_t* indices,
0227:   int numPlanes,
0228:   int inputW, int inputH,
0229:   int outputW, int outputH) {
0230:   at::parallel_for(0, numPlanes, 0, [&](int64_t start, int64_t end) {
0231:     for (const auto plane : c10::irange(start, end)) {
0232:       scalar_t* gradInputForPlane = gradInput + plane * inputW * inputH;
0233:       const scalar_t* gradOutputForPlane = gradOutput + plane * outputW * outputH;
0234:       const int64_t* indicesForPlane = indices + plane * outputW * outputH;
0235: 
0236:       for (int h = 0; h < outputH; ++h) {
0237:         for (int w = 0; w < outputW; ++w) {
0238:           int outputIndex = h * outputW + w;
0239:           int64_t index = indicesForPlane[outputIndex];
0240:           AT_ASSERT(index >= 0 && index < static_cast<int64_t>(inputW) * inputH);
```
- **EN**: Lines 211-240 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: irange, fractional_max_pool2d_backward_out_single_batch_frame, parallel_for, AT_ASSERT.
- **CN**: 第 211-240 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：irange, fractional_max_pool2d_backward_out_single_batch_frame, parallel_for, AT_ASSERT。

### Lines 241-270 / 第 241-270 行
```cpp
0241: 
0242:           gradInputForPlane[index] += gradOutputForPlane[outputIndex];
0243:         }
0244:       }
0245:     }
0246:   });
0247: }
0248: 
0249: template <typename scalar_t>
0250: void fractional_max_pool2d_backward_out_frame(
0251:   scalar_t* gradInput,
0252:   const scalar_t* gradOutput,
0253:   const int64_t* indices,
0254:   int numBatch, int numPlanes,
0255:   int inputW, int inputH,
0256:   int outputW, int outputH) {
0257:     if(numBatch == 1) {
0258:       fractional_max_pool2d_backward_out_single_batch_frame<scalar_t>(
0259:         gradInput, gradOutput, indices,
0260:         numPlanes,
0261:         inputW, inputH, outputW, outputH
0262:       );
0263:       return;
0264:     }
0265:     at::parallel_for(0, numBatch, 0, [&](int64_t start, int64_t end) {
0266:       for (const auto batch : c10::irange(start, end)) {
0267:         fractional_max_pool2d_backward_out_single_batch_frame<scalar_t>(
0268:           gradInput + batch * numPlanes * inputH * inputW,
0269:           gradOutput + batch * numPlanes * outputH * outputW,
0270:           indices + batch * numPlanes * outputH * outputW,
```
- **EN**: Lines 241-270 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: fractional_max_pool2d_backward_out_frame, parallel_for, irange.
- **CN**: 第 241-270 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：fractional_max_pool2d_backward_out_frame, parallel_for, irange。

### Lines 271-300 / 第 271-300 行
```cpp
0271:           numPlanes, inputW, inputH, outputW, outputH);
0272:       }
0273:     });
0274: }
0275: 
0276: } // anonymous namespace
0277: 
0278: TORCH_IMPL_FUNC(fractional_max_pool2d_out_cpu) (
0279:   const at::Tensor& input_,
0280:   IntArrayRef pool_size,
0281:   IntArrayRef output_size,
0282:   const at::Tensor& randomSamples_,
0283:   const at::Tensor& output,
0284:   const at::Tensor& indices) {
0285: 
0286:   fractional_max_pool_check_shape</*ndim*/ 2>(input_, randomSamples_);
0287: 
0288:   if (output.numel() == 0) {
0289:     return;
0290:   }
0291: 
0292:   int64_t numBatch = 1;
0293:   int64_t planeDim = 0;
0294:   int64_t heightDim = 1;
0295:   int64_t widthDim = 2;
0296:   int64_t outputH = output_size[0]; // output.size(heightDim)
0297:   int64_t outputW = output_size[1]; // output.size(widthDim)
0298:   int64_t poolSizeH = pool_size[0];
0299:   int64_t poolSizeW = pool_size[1];
0300: 
```
- **EN**: Lines 271-300 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: TORCH_IMPL_FUNC, numel, size.
- **CN**: 第 271-300 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：TORCH_IMPL_FUNC, numel, size。

### Lines 301-330 / 第 301-330 行
```cpp
0301:   /* get contiguous input and samples */
0302:   auto input = input_.contiguous();
0303:   auto randomSamples = randomSamples_.contiguous();
0304: 
0305:   int64_t ndims = input.ndimension();
0306: 
0307:   if (ndims == 4) {
0308:     numBatch = input.size(0);
0309:     planeDim++;
0310:     heightDim++;
0311:     widthDim++;
0312:   }
0313: 
0314:   /* sizes */
0315:   int64_t numPlanes = input.size(planeDim);
0316:   int64_t inputH = input.size(heightDim);
0317:   int64_t inputW = input.size(widthDim);
0318: 
0319:   AT_DISPATCH_FLOATING_TYPES_AND2(
0320:     kBFloat16,
0321:     kHalf,
0322:     input.scalar_type(),
0323:     "fractional_max_pool2d_out_frame", [&] {
0324:       auto input_data = input.const_data_ptr<scalar_t>();
0325:       auto output_data = output.data_ptr<scalar_t>();
0326:       auto indices_data = indices.data_ptr<int64_t>();
0327:       auto randomSamples_data = randomSamples.const_data_ptr<scalar_t>();
0328:       fractional_max_pool2d_out_frame<scalar_t>(
0329:         input_data,
0330:         output_data,
```
- **EN**: Lines 301-330 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: contiguous, ndimension, size, AT_DISPATCH_FLOATING_TYPES_AND2.
- **CN**: 第 301-330 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：contiguous, ndimension, size, AT_DISPATCH_FLOATING_TYPES_AND2。

### Lines 331-360 / 第 331-360 行
```cpp
0331:         indices_data,
0332:         randomSamples_data,
0333:         numBatch, numPlanes,
0334:         inputW, inputH,
0335:         outputW, outputH,
0336:         poolSizeW, poolSizeH);
0337:     }
0338:   );
0339: }
0340: 
0341: TORCH_IMPL_FUNC(fractional_max_pool2d_backward_cpu) (
0342:   const at::Tensor& gradOutput_,
0343:   const at::Tensor& input,
0344:   IntArrayRef pool_size,
0345:   IntArrayRef output_size,
0346:   const at::Tensor& indices,
0347:   const at::Tensor& gradInput) {
0348: 
0349:   gradInput.zero_();
0350: 
0351:   int64_t numBatch = 1;
0352:   int planeDim = 0;
0353:   int heightDim = 1;
0354:   int widthDim = 2;
0355: 
0356:   auto outputH = output_size[0];
0357:   auto outputW = output_size[1];
0358: 
0359:   auto ndims = input.ndimension();
0360:   if (ndims == 4) {
```
- **EN**: Lines 331-360 mainly cover expressions/calls, state/variable declarations, macro-based glue. Notable symbols: TORCH_IMPL_FUNC, zero_, ndimension.
- **CN**: 第 331-360 行主要涉及表达式或调用、变量/别名声明、宏定义或宏调用。 值得关注的符号包括：TORCH_IMPL_FUNC, zero_, ndimension。

### Lines 361-390 / 第 361-390 行
```cpp
0361:     numBatch = input.size(0);
0362:     planeDim = 1;
0363:     heightDim++;
0364:     widthDim++;
0365:   }
0366: 
0367:   /* sizes */
0368:   auto numPlanes = input.size(planeDim);
0369:   auto inputH = input.size(heightDim);
0370:   auto inputW = input.size(widthDim);
0371: 
0372:   /* get contiguous gradOutput */
0373:   auto gradOutput = gradOutput_.contiguous();
0374: 
0375:   /* backprop */
0376:   AT_DISPATCH_FLOATING_TYPES_AND2(
0377:     kBFloat16,
0378:     kHalf,
0379:     input.scalar_type(), "fractional_max_pool2d_backward_out_frame", [&] {
0380:       auto gradInput_data = gradInput.data_ptr<scalar_t>();
0381:       auto gradOutput_data = gradOutput.const_data_ptr<scalar_t>();
0382:       auto indices_data = indices.const_data_ptr<int64_t>();
0383:       fractional_max_pool2d_backward_out_frame<scalar_t>(
0384:         gradInput_data,
0385:         gradOutput_data,
0386:         indices_data,
0387:         numBatch, numPlanes,
0388:         inputW, inputH,
0389:         outputW, outputH
0390:       );
```
- **EN**: Lines 361-390 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: size, contiguous, AT_DISPATCH_FLOATING_TYPES_AND2, scalar_type.
- **CN**: 第 361-390 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：size, contiguous, AT_DISPATCH_FLOATING_TYPES_AND2, scalar_type。

### Lines 391-396 / 第 391-396 行
```cpp
0391:     }
0392:   );
0393: }
0394: 
0395: } // at::native
0396: } // at
```
- **EN**: Lines 391-396 mainly cover expressions/calls, state/variable declarations.
- **CN**: 第 391-396 行主要涉及表达式或调用、变量/别名声明。

## Key Concepts / 关键概念
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Parallel loop scheduling  
  **CN**: 并行循环调度
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Native operator implementation path  
  **CN**: 原生算子实现路径

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/Parallel.h>`, `<ATen/TensorMeta.h>`, `<ATen/native/FractionalMaxPooling.h>`, `<c10/util/irange.h>`, `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/fractional_max_pool2d_backward_native.h>`, `<ATen/ops/fractional_max_pool2d_native.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `AT_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
