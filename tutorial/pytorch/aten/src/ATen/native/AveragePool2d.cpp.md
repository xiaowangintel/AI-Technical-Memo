# AveragePool2d.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/AveragePool2d.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Average Pool2d. It also wires backend dispatch paths.
- **Purpose (CN)**: 实现或声明与 平均、pool2d 相关的 ATen 原生逻辑。 它还负责连接不同后端的调度路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/core/Tensor.h>
0003: #include <ATen/ScalarOps.h>
0004: #include <ATen/native/Pool.h>
0005: 
0006: #ifndef AT_PER_OPERATOR_HEADERS
0007: #include <ATen/Functions.h>
0008: #include <ATen/NativeFunctions.h>
0009: #else
0010: #include <ATen/ops/avg_pool2d_backward_native.h>
0011: #include <ATen/ops/avg_pool2d_native.h>
0012: #endif
0013: 
0014: namespace at::meta {
0015: using namespace ::at::native;
0016: 
0017: TORCH_PRECOMPUTE_META_FUNC(avg_pool2d)
0018: (const Tensor& input,
0019:  IntArrayRef kernel_size,
0020:  IntArrayRef stride,
0021:  IntArrayRef padding,
0022:  bool ceil_mode,
0023:  bool count_include_pad,
0024:  std::optional<int64_t> divisor_override) {
0025:   // #20866, #22032: Guarantee this for the official C++ API?
0026:   TORCH_CHECK(kernel_size.size() == 1 || kernel_size.size() == 2,
0027:     "avg_pool2d: kernel_size must either be a single int, or a tuple of two ints");
0028:   const int64_t kH_val = kernel_size[0];
0029:   const int64_t kW_val = kernel_size.size() == 1 ? kH_val : kernel_size[1];
0030:   TORCH_CHECK(kH_val > 0 && kH_val <= std::numeric_limits<int>::max() &&
```
- **EN**: Lines 1-30 mainly cover header inclusion, expressions/calls, macro-based glue. Notable symbols: TORCH_PRECOMPUTE_META_FUNC, TORCH_CHECK, size, max.
- **CN**: 第 1-30 行主要涉及头文件包含、表达式或调用、宏定义或宏调用。 值得关注的符号包括：TORCH_PRECOMPUTE_META_FUNC, TORCH_CHECK, size, max。

### Lines 31-60 / 第 31-60 行
```cpp
0031:               kW_val > 0 && kW_val <= std::numeric_limits<int>::max(),
0032:     "integer out of range");
0033:   const int kH = static_cast<int>(kH_val);
0034:   const int kW = static_cast<int>(kW_val);
0035: 
0036:   TORCH_CHECK(stride.empty() || stride.size() == 1 || stride.size() == 2,
0037:     "avg_pool2d: stride must either be omitted, a single int, or a tuple of two ints");
0038:   const int64_t dH_val = stride.empty() ? kH_val : stride[0];
0039:   const int64_t dW_val = stride.empty() ? kW_val : (stride.size() == 1 ? dH_val : stride[1]);
0040:   if (!stride.empty()) {
0041:     // Check for negative values (invalid) and overflow (values > INT_MAX)
0042:     // Zero stride is validated later in pooling_output_shape with proper error message
0043:     if (dH_val < 0 || dH_val > std::numeric_limits<int>::max() ||
0044:         dW_val < 0 || dW_val > std::numeric_limits<int>::max()) {
0045:       TORCH_CHECK(false, "integer out of range");
0046:     }
0047:   }
0048:   const int dH = static_cast<int>(dH_val);
0049:   const int dW = static_cast<int>(dW_val);
0050: 
0051:   TORCH_CHECK(padding.size() == 1 || padding.size() == 2,
0052:     "avg_pool2d: padding must either be a single int, or a tuple of two ints");
0053:   const int64_t padH_val = padding[0];
0054:   const int64_t padW_val = padding.size() == 1 ? padH_val : padding[1];
0055:   // note: negative padding is checked later in pool2d_shape_check with "pad must be non-negative" error
0056:   TORCH_CHECK(padH_val <= std::numeric_limits<int>::max() &&
0057:               padW_val <= std::numeric_limits<int>::max(),
0058:     "integer out of range");
0059:   const int padH = static_cast<int>(padH_val);
0060:   const int padW = static_cast<int>(padW_val);
```
- **EN**: Lines 31-60 mainly cover state/variable declarations, macro-based glue, function signatures/definitions. Notable symbols: max, TORCH_CHECK, empty, size.
- **CN**: 第 31-60 行主要涉及变量/别名声明、宏定义或宏调用、函数签名或实现。 值得关注的符号包括：max, TORCH_CHECK, empty, size。

### Lines 61-90 / 第 61-90 行
```cpp
0061: 
0062:   TORCH_CHECK(!divisor_override.has_value() || divisor_override.value() != 0,
0063:     "divisor must be not zero");
0064: 
0065:   const int64_t nbatch = input.ndimension() == 4 ? input.size(-4) : 1;
0066:   const int64_t nInputPlane = input.size(-3);
0067:   const int64_t inputHeight = input.size(-2);
0068:   const int64_t inputWidth = input.size(-1);
0069: 
0070:   const int64_t outputHeight = pooling_output_shape<int64_t>(
0071:       inputHeight, kH, padH, dH, 1, ceil_mode);
0072:   const int64_t outputWidth =
0073:       pooling_output_shape<int64_t>(inputWidth, kW, padW, dW, 1, ceil_mode);
0074: 
0075:   auto memory_format = input.suggest_memory_format();
0076:   pool2d_shape_check(
0077:       input,
0078:       kH,
0079:       kW,
0080:       dH,
0081:       dW,
0082:       padH,
0083:       padW,
0084:       1,
0085:       1,
0086:       nInputPlane,
0087:       inputHeight,
0088:       inputWidth,
0089:       outputHeight,
0090:       outputWidth,
```
- **EN**: Lines 61-90 mainly cover expressions/calls, state/variable declarations, macro-based glue. Notable symbols: TORCH_CHECK, has_value, value, ndimension.
- **CN**: 第 61-90 行主要涉及表达式或调用、变量/别名声明、宏定义或宏调用。 值得关注的符号包括：TORCH_CHECK, has_value, value, ndimension。

### Lines 91-120 / 第 91-120 行
```cpp
0091:       memory_format);
0092: 
0093:   /* resize output */
0094:   if (input.ndimension() == 3) {
0095:     set_output_raw_strided(
0096:         0,
0097:         {nInputPlane,
0098:          outputHeight,
0099:          outputWidth},
0100:         {},
0101:         input.options());
0102:   }
0103:   else {
0104:     set_output_raw_strided(
0105:         0,
0106:         {nbatch,
0107:          nInputPlane,
0108:          outputHeight,
0109:          outputWidth},
0110:         {},
0111:         input.options().memory_format(memory_format));
0112:   }
0113: 
0114:   return TORCH_PRECOMPUTE_STRUCT(avg_pool2d)().set_kH(kH).set_kW(kW).set_dH(dH).set_dW(dW).set_padH(padH).set_padW(padW);
0115: }
0116: 
0117: TORCH_META_FUNC(avg_pool2d_backward) (
0118:   const Tensor& gradOutput_,
0119:   const Tensor& input,
0120:   IntArrayRef kernel_size,
```
- **EN**: Lines 91-120 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: ndimension, set_output_raw_strided, options, memory_format.
- **CN**: 第 91-120 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：ndimension, set_output_raw_strided, options, memory_format。

### Lines 121-150 / 第 121-150 行
```cpp
0121:   IntArrayRef stride,
0122:   IntArrayRef padding,
0123:   bool ceil_mode,
0124:   bool count_include_pad,
0125:   std::optional<int64_t> divisor_override
0126: ) {
0127:   // #20866, #22032: Guarantee this for the official C++ API?
0128:   TORCH_CHECK(kernel_size.size() == 1 || kernel_size.size() == 2,
0129:     "avg_pool2d: kernel_size must either be a single int, or a tuple of two ints");
0130:   const int64_t kH_val = kernel_size[0];
0131:   const int64_t kW_val = kernel_size.size() == 1 ? kH_val : kernel_size[1];
0132:   TORCH_CHECK(kH_val > 0 && kH_val <= std::numeric_limits<int>::max() &&
0133:               kW_val > 0 && kW_val <= std::numeric_limits<int>::max(),
0134:     "integer out of range");
0135:   const int kH = static_cast<int>(kH_val);
0136:   const int kW = static_cast<int>(kW_val);
0137: 
0138:   TORCH_CHECK(stride.empty() || stride.size() == 1 || stride.size() == 2,
0139:     "avg_pool2d: stride must either be omitted, a single int, or a tuple of two ints");
0140:   const int64_t dH_val = stride.empty() ? kH_val : stride[0];
0141:   const int64_t dW_val = stride.empty() ? kW_val : (stride.size() == 1 ? dH_val : stride[1]);
0142:   if (!stride.empty()) {
0143:     // Check for negative values (invalid) and overflow (values > INT_MAX)
0144:     // Zero stride is validated later in pooling_output_shape with proper error message
0145:     if (dH_val < 0 || dH_val > std::numeric_limits<int>::max() ||
0146:         dW_val < 0 || dW_val > std::numeric_limits<int>::max()) {
0147:       TORCH_CHECK(false, "integer out of range");
0148:     }
0149:   }
0150:   const int dH = static_cast<int>(dH_val);
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: TORCH_CHECK, size, max, empty.
- **CN**: 第 121-150 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：TORCH_CHECK, size, max, empty。

### Lines 151-180 / 第 151-180 行
```cpp
0151:   const int dW = static_cast<int>(dW_val);
0152: 
0153:   TORCH_CHECK(padding.size() == 1 || padding.size() == 2,
0154:     "avg_pool2d: padding must either be a single int, or a tuple of two ints");
0155:   const int64_t padH_val = padding[0];
0156:   const int64_t padW_val = padding.size() == 1 ? padH_val : padding[1];
0157:   // note: negative padding is checked later in avg_pool2d_backward_shape_check with "pad must be non-negative" error
0158:   TORCH_CHECK(padH_val <= std::numeric_limits<int>::max() &&
0159:               padW_val <= std::numeric_limits<int>::max(),
0160:     "integer out of range");
0161:   const int padH = static_cast<int>(padH_val);
0162:   const int padW = static_cast<int>(padW_val);
0163: 
0164:   TORCH_CHECK(!divisor_override.has_value() || divisor_override.value() != 0, "divisor must be not zero");
0165: 
0166:   /* sizes */
0167:   const int64_t nbatch = input.ndimension() == 4 ? input.size(-4) : 1;
0168:   const int64_t nInputPlane = input.size(-3); // number of channels (or colors)
0169:   const int64_t inputHeight = input.size(-2);
0170:   const int64_t inputWidth = input.size(-1);
0171:   const int64_t outputWidth = pooling_output_shape<int64_t>(inputWidth, kW, padW, dW, 1, ceil_mode);
0172:   const int64_t outputHeight = pooling_output_shape<int64_t>(inputHeight, kH, padH, dH, 1, ceil_mode);
0173: 
0174:   auto memory_format = input.suggest_memory_format();
0175:   avg_pool2d_backward_shape_check(
0176:     input,
0177:     gradOutput_,
0178:     nbatch,
0179:     kH, kW, dH, dW, padH, padW,
0180:     nInputPlane,
```
- **EN**: Lines 151-180 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: TORCH_CHECK, size, max, has_value.
- **CN**: 第 151-180 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：TORCH_CHECK, size, max, has_value。

### Lines 181-210 / 第 181-210 行
```cpp
0181:     inputHeight, inputWidth,
0182:     outputHeight, outputWidth,
0183:     memory_format);
0184: 
0185:   /* resize output */
0186:   set_output_raw_strided(0, input.sizes(), {}, input.options().memory_format(memory_format));
0187: }
0188: 
0189: } // namespace at::meta
0190: 
0191: namespace at::native {
0192: 
0193: TORCH_IMPL_FUNC(avg_pool2d_out_cpu)
0194: (const Tensor& input,
0195:  int64_t kH,
0196:  int64_t kW,
0197:  int64_t dH,
0198:  int64_t dW,
0199:  int64_t padH,
0200:  int64_t padW,
0201:  bool ceil_mode,
0202:  bool count_include_pad,
0203:  std::optional<int64_t> divisor_override,
0204:  const Tensor& output) {
0205:   avg_pool2d_kernel(
0206:       kCPU,
0207:       output,
0208:       input,
0209:       kW,
0210:       kH,
```
- **EN**: Lines 181-210 mainly cover expressions/calls, state/variable declarations, namespace structuring. Notable symbols: set_output_raw_strided, sizes, options, memory_format.
- **CN**: 第 181-210 行主要涉及表达式或调用、变量/别名声明、命名空间组织。 值得关注的符号包括：set_output_raw_strided, sizes, options, memory_format。

### Lines 211-240 / 第 211-240 行
```cpp
0211:       dW,
0212:       dH,
0213:       padW,
0214:       padH,
0215:       count_include_pad,
0216:       divisor_override);
0217: }
0218: 
0219: TORCH_IMPL_FUNC(avg_pool2d_backward_out_cpu) (
0220:   const Tensor& gradOutput,
0221:   const Tensor& input,
0222:   IntArrayRef kernel_size,
0223:   IntArrayRef stride,
0224:   IntArrayRef padding,
0225:   bool ceil_mode,
0226:   bool count_include_pad,
0227:   std::optional<int64_t> divisor_override,
0228:   const Tensor& gradInput
0229: ) {
0230:   const int kH = safe_downcast<int, int64_t>(kernel_size[0]);
0231:   const int kW = kernel_size.size() == 1 ? kH : safe_downcast<int, int64_t>(kernel_size[1]);
0232: 
0233:   const int dH = stride.empty() ? kH : safe_downcast<int, int64_t>(stride[0]);
0234:   const int dW = stride.empty() ? kW :
0235:                  stride.size() == 1 ? dH : safe_downcast<int, int64_t>(stride[1]);
0236: 
0237:   const int padH = safe_downcast<int, int64_t>(padding[0]);
0238:   const int padW = padding.size() == 1 ? padH : safe_downcast<int, int64_t>(padding[1]);
0239: 
0240:   TORCH_CHECK(!divisor_override.has_value() || divisor_override.value() != 0, "divisor must be not zero");
```
- **EN**: Lines 211-240 mainly cover expressions/calls, state/variable declarations, macro-based glue. Notable symbols: TORCH_IMPL_FUNC, size, empty, TORCH_CHECK.
- **CN**: 第 211-240 行主要涉及表达式或调用、变量/别名声明、宏定义或宏调用。 值得关注的符号包括：TORCH_IMPL_FUNC, size, empty, TORCH_CHECK。

### Lines 241-257 / 第 241-257 行
```cpp
0241: 
0242:   TORCH_CHECK(input.dtype() == gradOutput.dtype(),
0243:     "expected dtype ", input.dtype(), " for `gradOutput` but got dtype ", gradOutput.dtype());
0244: 
0245:   /* zero the gradient */
0246:   gradInput.zero_();
0247: 
0248:   avg_pool2d_backward_kernel(
0249:       kCPU, gradInput, gradOutput,
0250:       kW, kH, dW, dH, padW, padH,
0251:       count_include_pad, divisor_override);
0252: }
0253: 
0254: DEFINE_DISPATCH(avg_pool2d_kernel);
0255: DEFINE_DISPATCH(avg_pool2d_backward_kernel);
0256: 
0257: } // namespace at::native
```
- **EN**: Lines 241-257 mainly cover expressions/calls, macro-based glue, state/variable declarations. Notable symbols: TORCH_CHECK, dtype, zero_, avg_pool2d_backward_kernel.
- **CN**: 第 241-257 行主要涉及表达式或调用、宏定义或宏调用、变量/别名声明。 值得关注的符号包括：TORCH_CHECK, dtype, zero_, avg_pool2d_backward_kernel。

## Key Concepts / 关键概念
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Dispatch stub definition  
  **CN**: 调度桩定义
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Scalar/tensor mixed arithmetic  
  **CN**: 标量与张量混合运算
- **EN**: Native operator implementation path  
  **CN**: 原生算子实现路径

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/ScalarOps.h>`, `<ATen/native/Pool.h>`, `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/avg_pool2d_backward_native.h>`, `<ATen/ops/avg_pool2d_native.h>`
- **Macros / 宏**: `TORCH_CHECK`, `DEFINE_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `std::`
