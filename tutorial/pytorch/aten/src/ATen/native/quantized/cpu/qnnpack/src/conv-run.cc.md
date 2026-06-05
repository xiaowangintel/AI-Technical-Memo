# conv-run.cc — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/conv-run.cc`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `conv-run.cc`.
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `conv-run.cc` 展开。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行

```cpp
0001: #include <qnnpack/indirection.h>
0002: #include <qnnpack/log.h>
0003: #include <qnnpack/operator.h>
0004: #include <qnnpack/pack.h>
0005: #include <qnnpack_func.h>
0006: #include <cstring>
0007: #include <memory>
0008: #include <numeric>
0009: 
0010: namespace qnnpack {
0011: 
0012: struct q8gemm_xzp_context {
0013:   size_t k;
0014:   size_t k_stride;
0015:   size_t n;
0016:   size_t n_stride;
0017:   const uint8_t* a;
0018:   size_t a_stride;
0019:   const void* packed_w;
0020:   uint8_t* c;
0021:   size_t c_stride;
0022:   const int32_t* a_sum;
0023:   size_t groups;
0024:   size_t batch_size;
0025:   size_t a_sum_stride;
0026:   union pytorch_qnnp_q31_requantization_params requantization_params;
0027:   const pytorch_q8gemm_xzp_ukernel_function ukernel;
0028: };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8gemm_xzp_context`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8gemm_xzp_context`。

### Lines 29-51 / 第 29-51 行

```cpp
0029: static void compute_q8gemm_xzp(
0030:     const struct q8gemm_xzp_context context[1],
0031:     size_t group_index,
0032:     size_t pixel_index,
0033:     size_t mr_block_start,
0034:     size_t nr_block_start,
0035:     size_t group_range /* always 1 */,
0036:     size_t pixel_range,
0037:     size_t mr_block_size,
0038:     size_t nr_block_size) {
0039:   const size_t k = context->k;
0040:   const size_t k_stride = context->k_stride;
0041:   const size_t n = context->n;
0042:   const size_t n_stride = context->n_stride;
0043:   const uint8_t* a = context->a;
0044:   const size_t a_stride = context->a_stride;
0045:   const void* packed_w = context->packed_w;
0046:   uint8_t* c = context->c;
0047:   const size_t c_stride = context->c_stride;
0048:   const int32_t* a_sum = context->a_sum;
0049:   const size_t groups = context->groups;
0050:   const size_t a_sum_stride = context->a_sum_stride;
0051: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8gemm_xzp_context`, `compute_q8gemm_xzp`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8gemm_xzp_context`, `compute_q8gemm_xzp`。

### Lines 52-79 / 第 52-79 行

```cpp
0052:   context->ukernel(
0053:       mr_block_size,
0054:       nr_block_size,
0055:       k,
0056:       a + (pixel_index + mr_block_start) * a_stride + group_index * k,
0057:       a_stride,
0058:       a_sum + pixel_index * groups + group_index * a_sum_stride +
0059:           mr_block_start,
0060:       (const void*)((uintptr_t)packed_w + (nr_block_start + group_index * n_stride) * (k_stride * sizeof(uint8_t) + sizeof(int32_t))),
0061:       c + (pixel_index + mr_block_start) * c_stride + nr_block_start +
0062:           group_index * n,
0063:       c_stride,
0064:       &context->requantization_params);
0065: }
0066: 
0067: struct q8gemm_context {
0068:   size_t k;
0069:   size_t k_stride;
0070:   size_t n;
0071:   size_t n_stride;
0072:   const uint8_t* a;
0073:   size_t a_stride;
0074:   const uint8_t* packed_w;
0075:   uint8_t* c;
0076:   size_t c_stride;
0077:   union pytorch_qnnp_conv_quantization_params quantization_params;
0078:   const pytorch_q8gemm_ukernel_function ukernel;
0079: };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8gemm_context`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8gemm_context`。

### Lines 80-99 / 第 80-99 行

```cpp
0080: static void compute_q8gemm(
0081:     const struct q8gemm_context context[1],
0082:     size_t group_index,
0083:     size_t pixel_index,
0084:     size_t mr_block_start,
0085:     size_t nr_block_start,
0086:     size_t group_range /* always 1 */,
0087:     size_t pixel_range,
0088:     size_t mr_block_size,
0089:     size_t nr_block_size) {
0090:   const size_t k = context->k;
0091:   const size_t k_stride = context->k_stride;
0092:   const size_t n = context->n;
0093:   const size_t n_stride = context->n_stride;
0094:   const uint8_t* a = context->a;
0095:   const size_t a_stride = context->a_stride;
0096:   const void* packed_w = context->packed_w;
0097:   uint8_t* c = context->c;
0098:   const size_t c_stride = context->c_stride;
0099: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8gemm_context`, `compute_q8gemm`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8gemm_context`, `compute_q8gemm`。

### Lines 100-130 / 第 100-130 行

```cpp
0100:   const size_t output_channel_index = nr_block_start + group_index * n;
0101:   context->ukernel(
0102:       mr_block_size,
0103:       nr_block_size,
0104:       k,
0105:       a + (pixel_index + mr_block_start) * a_stride + group_index * k,
0106:       a_stride,
0107:       (const void*)((uintptr_t)packed_w + (nr_block_start + group_index * n_stride) * (k_stride * sizeof(uint8_t) + sizeof(int32_t))),
0108:       c + (pixel_index + mr_block_start) * c_stride + nr_block_start +
0109:           group_index * n,
0110:       c_stride,
0111:       output_channel_index,
0112:       &context->quantization_params);
0113: }
0114: 
0115: struct q8conv_context {
0116:   size_t bs;
0117:   size_t ks;
0118:   size_t kc;
0119:   size_t kc_stride;
0120:   size_t m;
0121:   size_t m_stride;
0122:   size_t n;
0123:   size_t n_stride;
0124:   const uint8_t** indirect_a;
0125:   const void* packed_w;
0126:   uint8_t* c;
0127:   size_t c_stride;
0128:   union pytorch_qnnp_conv_quantization_params quantization_params;
0129:   const pytorch_q8conv_ukernel_function ukernel;
0130: };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8conv_context`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8conv_context`。

### Lines 131-153 / 第 131-153 行

```cpp
0131: static void compute_q8conv(
0132:     const struct q8conv_context context[1],
0133:     size_t group_index,
0134:     size_t image_index,
0135:     size_t mr_block_start,
0136:     size_t nr_block_start,
0137:     size_t group_range /* always 1 */,
0138:     size_t image_range /* always 1 */,
0139:     size_t mr_block_size,
0140:     size_t nr_block_size) {
0141:   const size_t bs = context->bs;
0142:   const size_t ks = context->ks;
0143:   const size_t kc = context->kc;
0144:   const size_t kc_stride = context->kc_stride;
0145:   const size_t m = context->m;
0146:   const size_t m_stride = context->m_stride;
0147:   const size_t n = context->n;
0148:   const size_t n_stride = context->n_stride;
0149:   const uint8_t** indirect_a = context->indirect_a;
0150:   const void* packed_w = context->packed_w;
0151:   uint8_t* c = context->c;
0152:   const size_t c_stride = context->c_stride;
0153: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8conv_context`, `compute_q8conv`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8conv_context`, `compute_q8conv`。

### Lines 154-180 / 第 154-180 行

```cpp
0154:   const size_t output_channel_index = group_index * n + nr_block_start;
0155:   context->ukernel(
0156:       mr_block_size,
0157:       nr_block_size,
0158:       kc,
0159:       ks,
0160:       indirect_a +
0161:           (mr_block_start + (image_index + group_index * bs) * m_stride) * ks,
0162:       (const void*)((uintptr_t)packed_w + (nr_block_start + group_index * n_stride) * (kc_stride * sizeof(uint8_t) + sizeof(int32_t))),
0163:       c + (mr_block_start + image_index * m) * c_stride + group_index * n +
0164:           nr_block_start,
0165:       c_stride,
0166:       output_channel_index,
0167:       &context->quantization_params);
0168: }
0169: 
0170: struct q8sum_rows_context {
0171:   const uint8_t* a;
0172:   size_t groups;
0173:   size_t m;
0174:   size_t k;
0175:   size_t a_stride;
0176:   const int32_t multiplier;
0177:   int32_t* a_sum;
0178:   size_t a_sum_stride;
0179:   const pytorch_q8sum_rows_ukernel_function ukernel;
0180: };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8sum_rows_context`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8sum_rows_context`。

### Lines 181-207 / 第 181-207 行

```cpp
0181: static void compute_sum_rows(
0182:     const struct q8sum_rows_context context[1],
0183:     size_t group_index,
0184:     size_t batch_index,
0185:     size_t block_start,
0186:     size_t group_range /* always 1 */,
0187:     size_t batch_range /* always 1 */,
0188:     size_t block_size) {
0189:   const uint8_t* a = context->a;
0190:   const size_t groups = context->groups;
0191:   const size_t m = context->m;
0192:   const size_t k = context->k;
0193:   const size_t a_stride = context->a_stride;
0194:   const int32_t multiplier = context->multiplier;
0195:   int32_t* a_sum = context->a_sum;
0196:   const size_t a_sum_stride = context->a_sum_stride;
0197: 
0198:   context->ukernel(
0199:       a + batch_index * m * a_stride + group_index * k + block_start * a_stride,
0200:       min(block_size, m - block_start),
0201:       k,
0202:       a_stride,
0203:       multiplier,
0204:       a_sum + batch_index * groups * a_sum_stride + group_index * a_sum_stride +
0205:           block_start);
0206: }
0207: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8sum_rows_context`, `compute_sum_rows`, `min`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8sum_rows_context`, `compute_sum_rows`, `min`。

### Lines 208-240 / 第 208-240 行

```cpp
0208: struct q8dwconv2d_context {
0209:   size_t groups;
0210:   size_t group_stride;
0211:   const uint8_t** indirection_buffer;
0212:   size_t indirection_buffer_row_stride;
0213:   size_t indirection_buffer_col_stride;
0214:   const void* packed_weights;
0215:   uint8_t* output;
0216:   size_t output_height;
0217:   size_t output_width;
0218:   size_t output_row_stride;
0219:   size_t output_col_increment;
0220:   union pytorch_qnnp_conv_quantization_params quantization_params;
0221:   const pytorch_q8dwconv2d_up_ukernel_function unipass_ukernel;
0222:   const pytorch_q8dwconv2d_mp_ukernel_function multipass_ukernel;
0223: };
0224: 
0225: struct q8dwconv3d_context {
0226:   size_t groups;
0227:   size_t group_stride;
0228:   const uint8_t** indirection_buffer;
0229:   size_t indirection_buffer_slice_stride;
0230:   size_t indirection_buffer_row_stride;
0231:   size_t indirection_buffer_col_stride;
0232:   const void* packed_weights;
0233:   uint8_t* output;
0234:   size_t output_depth;
0235:   size_t output_height;
0236:   size_t output_width;
0237:   size_t output_slice_stride;
0238:   union pytorch_qnnp_conv_quantization_params quantization_params;
0239:   const pytorch_q8dwconv3d_mp_ukernel_function multipass_ukernel;
0240: };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8dwconv2d_context`, `q8dwconv3d_context`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8dwconv2d_context`, `q8dwconv3d_context`。

### Lines 241-261 / 第 241-261 行

```cpp
0241: 
0242: static void compute_dwconv2d_unipass(
0243:     const struct q8dwconv2d_context context[1],
0244:     size_t image,
0245:     size_t output_y) {
0246:   const size_t output_height = context->output_height;
0247: 
0248:   context->unipass_ukernel(
0249:       context->groups,
0250:       context->output_width,
0251:       context->indirection_buffer +
0252:           (image * output_height + output_y) *
0253:               context->indirection_buffer_row_stride,
0254:       context->packed_weights,
0255:       context->output +
0256:           (image * output_height + output_y) * context->output_row_stride,
0257:       context->indirection_buffer_col_stride,
0258:       context->output_col_increment,
0259:       &context->quantization_params);
0260: }
0261: static void compute_dwconv2d_multiipass(
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8dwconv2d_context`, `compute_dwconv2d_unipass`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8dwconv2d_context`, `compute_dwconv2d_unipass`。

### Lines 262-286 / 第 262-286 行

```cpp
0262:     const struct q8dwconv2d_context context[1],
0263:     size_t image,
0264:     size_t output_y) {
0265:   const size_t output_height = context->output_height;
0266:   PYTORCH_QNNP_ALIGN(16)
0267: #ifdef _MSC_VER
0268:   int32_t* multipass_acc = (int32_t*)_malloca(sizeof(int32_t) * context->group_stride);
0269: #else
0270:   int32_t multipass_acc[context->group_stride];
0271: #endif
0272: 
0273:   context->multipass_ukernel(
0274:       context->groups,
0275:       context->output_width,
0276:       context->indirection_buffer +
0277:           (image * output_height + output_y) *
0278:               context->indirection_buffer_row_stride,
0279:       context->packed_weights,
0280:       multipass_acc,
0281:       context->output +
0282:           (image * output_height + output_y) * context->output_row_stride,
0283:       context->indirection_buffer_col_stride,
0284:       context->output_col_increment,
0285:       &context->quantization_params);
0286: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8dwconv2d_context`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8dwconv2d_context`。

### Lines 287-320 / 第 287-320 行

```cpp
0287: #ifdef _MSC_VER
0288:   _freea(multipass_acc);
0289: #endif
0290: }
0291: 
0292: static void compute_dwconv3d_multiipass(
0293:     const struct q8dwconv3d_context context[1],
0294:     size_t image,
0295:     size_t output_z) {
0296:   const size_t output_depth = context->output_depth;
0297:   PYTORCH_QNNP_ALIGN(16)
0298: #ifdef _MSC_VER
0299:   int32_t* multipass_acc =
0300:       (int32_t*)_malloca(sizeof(int32_t) * context->group_stride);
0301: #else
0302:   int32_t multipass_acc[context->group_stride];
0303: #endif
0304: 
0305:   context->multipass_ukernel(
0306:       context->groups,
0307:       context->output_height,
0308:       context->output_width,
0309:       context->indirection_buffer +
0310:           (image * output_depth + output_z) *
0311:               context->indirection_buffer_slice_stride,
0312:       context->packed_weights,
0313:       multipass_acc,
0314:       context->output +
0315:           (image * output_depth + output_z) * context->output_slice_stride,
0316:       context->indirection_buffer_row_stride,
0317:       context->indirection_buffer_col_stride,
0318:       0,
0319:       &context->quantization_params);
0320: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8dwconv3d_context`, `_freea`, `compute_dwconv3d_multiipass`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8dwconv3d_context`, `_freea`, `compute_dwconv3d_multiipass`。

### Lines 321-356 / 第 321-356 行

```cpp
0321: #ifdef _MSC_VER
0322:   _freea(multipass_acc);
0323: #endif
0324: }
0325: 
0326: struct QnnpackDeleter {
0327:   void operator()(pytorch_qnnp_operator_t op) {
0328:     pytorch_qnnp_delete_operator(op);
0329:   }
0330: };
0331: 
0332: enum pytorch_qnnp_status qnnpackConv(
0333:     const pytorch_qnnp_operator_t convolution,
0334:     void* packed_weights,
0335:     const size_t batch_size,
0336:     const size_t input_depth,
0337:     const size_t input_height,
0338:     const size_t input_width,
0339:     const uint8_t input_zero_point,
0340:     const uint8_t* input,
0341:     const uint8_t* kernel_zero_points,
0342:     const float* requantization_scales,
0343:     const uint8_t output_zero_point,
0344:     const uint8_t output_min,
0345:     const uint8_t output_max,
0346:     uint8_t* output,
0347:     pthreadpool_t threadpool) {
0348:   const size_t groups = convolution->groups;
0349:   const size_t input_pixel_stride = convolution->group_input_channels * groups;
0350:   const size_t output_pixel_stride =
0351:       convolution->group_output_channels * groups;
0352:   const size_t kernel_width = convolution->kernel_width;
0353:   const size_t kernel_height = convolution->kernel_height;
0354:   const size_t kernel_depth = convolution->kernel_depth;
0355:   const size_t kernel_size = kernel_height * kernel_width * kernel_depth;
0356: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `QnnpackDeleter`, `pytorch_qnnp_status`, `_freea`, `pytorch_qnnp_delete_operator`, `qnnpackConv`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`QnnpackDeleter`, `pytorch_qnnp_status`, `_freea`, `pytorch_qnnp_delete_operator`, `qnnpackConv`。

### Lines 357-380 / 第 357-380 行

```cpp
0357:   if (batch_size == 0) {
0358:     // If no batches, return
0359:     return pytorch_qnnp_status_success;
0360:   }
0361: 
0362:   union pytorch_qnnp_q31_requantization_params requantization_params {};
0363:   union pytorch_qnnp_conv_quantization_params conv_quantization_params {};
0364:   if (convolution->ukernel_type == pytorch_qnnp_ukernel_type_xzp_gemm) {
0365:     requantization_params = pytorch_qnnp_compute_requantization_params(
0366:         // Note. XZP kernels are not changed for per channel quant.
0367:         requantization_scales[0],
0368:         output_zero_point,
0369:         output_min,
0370:         output_max);
0371:   } else {
0372:     conv_quantization_params = pytorch_qnnp_compute_conv_quantization_params(
0373:         input_zero_point,
0374:         kernel_zero_points,
0375:         requantization_scales,
0376:         output_zero_point,
0377:         output_min,
0378:         output_max);
0379:   }
0380: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 381-401 / 第 381-401 行

```cpp
0381:   // Convolution op caches a few things.
0382:   // We need to check if the corresponding values on this
0383:   // invocation is same as cached values.
0384:   // If so we can skip setup step.
0385:   if (convolution->input != input || convolution->batch_size != batch_size ||
0386:       convolution->input_depth != input_depth ||
0387:       convolution->input_height != input_height ||
0388:       convolution->input_width != input_width ||
0389:       convolution->input_pixel_stride != input_pixel_stride) {
0390:     pytorch_qnnp_status status = pytorch_qnnp_setup_convolution_ndhwc_q8(
0391:         convolution,
0392:         batch_size,
0393:         input_depth,
0394:         input_height,
0395:         input_width,
0396:         input,
0397:         input_pixel_stride,
0398:         output,
0399:         output_pixel_stride,
0400:         threadpool);
0401:     if (status != pytorch_qnnp_status_success) {
```

- **EN:** This block handles conditional branches and special cases. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：无明显局部符号。

### Lines 402-421 / 第 402-421 行

```cpp
0402:       pytorch_qnnp_log_error(
0403:           "failed to run convolution op setup to setup indirection buffer.");
0404:       return status;
0405:     }
0406:   }
0407: 
0408:   const size_t output_size = convolution->output_height *
0409:       convolution->output_width * convolution->output_depth;
0410: 
0411:   switch (convolution->ukernel_type) {
0412:     case pytorch_qnnp_ukernel_type_dwconv: {
0413:       const uint32_t cr = pytorch_qnnp_params.q8dw9.cr;
0414:       const size_t group_stride = (groups + (cr - 1)) & -cr;
0415: 
0416:       const size_t step_height = convolution->step_height;
0417:       const size_t step_width = convolution->step_width;
0418: 
0419:       switch (kernel_size) {
0420:         case 9: {
0421:           struct q8dwconv2d_context context = {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8dwconv2d_context`, `pytorch_qnnp_log_error`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8dwconv2d_context`, `pytorch_qnnp_log_error`。

### Lines 422-444 / 第 422-444 行

```cpp
0422:               .groups = groups,
0423:               .group_stride = group_stride,
0424:               .indirection_buffer =
0425:                   (const uint8_t**)convolution->indirection_buffer,
0426:               .indirection_buffer_row_stride = step_height,
0427:               .indirection_buffer_col_stride =
0428:                   kernel_height * step_width * sizeof(void*),
0429:               .packed_weights = packed_weights,
0430:               .output = output,
0431:               .output_height = convolution->output_height,
0432:               .output_width = convolution->output_width,
0433:               .output_row_stride =
0434:                   convolution->output_width * output_pixel_stride,
0435:               .output_col_increment =
0436:                   (output_pixel_stride - groups) * sizeof(uint8_t),
0437:               .quantization_params = conv_quantization_params,
0438:               .unipass_ukernel = convolution->per_channel
0439:                   ? pytorch_qnnp_params.q8dw9.updw_per_channel
0440:                   : pytorch_qnnp_params.q8dw9.updw,
0441:               .multipass_ukernel = convolution->per_channel
0442:                   ? pytorch_qnnp_params.q8dw25.mpdw_per_channel
0443:                   : pytorch_qnnp_params.q8dw25.mpdw,
0444:           };
```

- **EN:** This block manages memory allocation, buffers, or ownership boundaries. Key symbols: no prominent local symbols.
- **CN:** 该代码块管理内存分配、缓冲区或所有权边界。关键符号：无明显局部符号。

### Lines 445-477 / 第 445-477 行

```cpp
0445:           pthreadpool_compute_2d(
0446:               threadpool,
0447:               (pthreadpool_function_2d_t)compute_dwconv2d_unipass,
0448:               &context,
0449:               batch_size,
0450:               convolution->output_height);
0451:           break;
0452:         }
0453:         case 25: {
0454:           struct q8dwconv2d_context context = {
0455:               .groups = groups,
0456:               .group_stride = group_stride,
0457:               .indirection_buffer =
0458:                   (const uint8_t**)convolution->indirection_buffer,
0459:               .indirection_buffer_row_stride = step_height,
0460:               .indirection_buffer_col_stride =
0461:                   kernel_height * step_width * sizeof(void*),
0462:               .packed_weights = packed_weights,
0463:               .output = output,
0464:               .output_height = convolution->output_height,
0465:               .output_width = convolution->output_width,
0466:               .output_row_stride =
0467:                   convolution->output_width * output_pixel_stride,
0468:               .output_col_increment =
0469:                   (output_pixel_stride - groups) * sizeof(uint8_t),
0470:               .quantization_params = conv_quantization_params,
0471:               .unipass_ukernel = convolution->per_channel
0472:                   ? pytorch_qnnp_params.q8dw9.updw_per_channel
0473:                   : pytorch_qnnp_params.q8dw9.updw,
0474:               .multipass_ukernel = convolution->per_channel
0475:                   ? pytorch_qnnp_params.q8dw25.mpdw_per_channel
0476:                   : pytorch_qnnp_params.q8dw25.mpdw,
0477:           };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8dwconv2d_context`, `pthreadpool_compute_2d`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8dwconv2d_context`, `pthreadpool_compute_2d`。

### Lines 478-506 / 第 478-506 行

```cpp
0478:           pthreadpool_compute_2d(
0479:               threadpool,
0480:               (pthreadpool_function_2d_t)compute_dwconv2d_multiipass,
0481:               &context,
0482:               batch_size,
0483:               convolution->output_height);
0484:           break;
0485:         }
0486:         case 27: {
0487:           struct q8dwconv3d_context context = {
0488:               .groups = groups,
0489:               .group_stride = group_stride,
0490:               .indirection_buffer =
0491:                   (const uint8_t**)convolution->indirection_buffer,
0492:               .indirection_buffer_slice_stride =
0493:                   step_height * convolution->output_height,
0494:               .indirection_buffer_row_stride = step_height * sizeof(void*),
0495:               .indirection_buffer_col_stride =
0496:                   kernel_height * kernel_depth * step_width * sizeof(void*),
0497:               .packed_weights = packed_weights,
0498:               .output = output,
0499:               .output_depth = convolution->output_depth,
0500:               .output_height = convolution->output_height,
0501:               .output_width = convolution->output_width,
0502:               .output_slice_stride = convolution->output_height *
0503:                   convolution->output_width * output_pixel_stride,
0504:               .quantization_params = conv_quantization_params,
0505:               .multipass_ukernel = pytorch_qnnp_params.q8dw27.mpdw,
0506:           };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8dwconv3d_context`, `pthreadpool_compute_2d`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8dwconv3d_context`, `pthreadpool_compute_2d`。

### Lines 507-528 / 第 507-528 行

```cpp
0507:           pthreadpool_compute_2d(
0508:               threadpool,
0509:               (pthreadpool_function_2d_t)compute_dwconv3d_multiipass,
0510:               &context,
0511:               batch_size,
0512:               convolution->output_depth);
0513:           break;
0514:         }
0515:         default:
0516:           PYTORCH_QNNP_UNREACHABLE;
0517:       }
0518:       break;
0519:     }
0520:     case pytorch_qnnp_ukernel_type_xzp_gemm: {
0521:       const size_t group_input_channels = convolution->group_input_channels;
0522:       const size_t group_output_channels = convolution->group_output_channels;
0523:       const uint32_t mr = pytorch_qnnp_params.q8conv_xzp.mr;
0524:       const uint32_t nr = pytorch_qnnp_params.q8conv_xzp.nr;
0525:       const uint32_t kr = pytorch_qnnp_params.q8conv_xzp.kr;
0526:       const size_t k_stride = (group_input_channels + (kr - 1)) & -kr;
0527:       const size_t n_stride = (group_output_channels + (nr - 1)) & -nr;
0528: 
```

- **EN:** This block implements local helper logic for `conv-run`. Key symbols: `pthreadpool_compute_2d`.
- **CN:** 该代码块实现与 `conv-run` 相关的局部辅助逻辑。关键符号：`pthreadpool_compute_2d`。

### Lines 529-554 / 第 529-554 行

```cpp
0529:       /* compute input row sum */
0530:       const size_t input_size = input_depth * input_height * input_width;
0531:       int32_t* a_sum = (int32_t*)realloc(
0532:           convolution->a_sum,
0533:           sizeof(int32_t) * batch_size * groups * input_size);
0534:       if (a_sum == nullptr) {
0535:         pytorch_qnnp_log_error(
0536:             "failed to allocate %zu bytes for row sum data",
0537:             sizeof(int32_t) * batch_size * groups * input_size);
0538:         return pytorch_qnnp_status_out_of_memory;
0539:       }
0540:       convolution->a_sum = a_sum;
0541:       struct q8sum_rows_context context = {
0542:           .a = input,
0543:           .groups = groups,
0544:           .m = input_size,
0545:           .k = convolution->group_input_channels,
0546:           .a_stride = input_pixel_stride,
0547:           // XZP kernels are not supporting per channel quant.
0548:           // We dont really use XZP kernels ATM.
0549:           // Thus assigning the zero point of first channel.
0550:           .multiplier = (int32_t)-kernel_zero_points[0],
0551:           .a_sum = a_sum,
0552:           .a_sum_stride = input_size,
0553:           .ukernel = pytorch_qnnp_params.q8sum_rows.sum_rows,
0554:       };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8sum_rows_context`, `pytorch_qnnp_log_error`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8sum_rows_context`, `pytorch_qnnp_log_error`。

### Lines 555-582 / 第 555-582 行

```cpp
0555:       pthreadpool_compute_3d_tiled(
0556:           threadpool,
0557:           (pthreadpool_function_3d_tiled_t)compute_sum_rows,
0558:           &context,
0559:           groups,
0560:           batch_size,
0561:           input_size,
0562:           1,
0563:           1,
0564:           pytorch_qnnp_params.q8sum_rows.m);
0565: 
0566:       struct q8gemm_xzp_context q8gemm_xzp_context = {
0567:           .k = convolution->group_input_channels,
0568:           .k_stride = k_stride,
0569:           .n = convolution->group_output_channels,
0570:           .n_stride = n_stride,
0571:           .a = input,
0572:           .a_stride = input_pixel_stride,
0573:           .packed_w = packed_weights,
0574:           .c = output,
0575:           .c_stride = output_pixel_stride,
0576:           .a_sum = a_sum,
0577:           .groups = groups,
0578:           .batch_size = batch_size,
0579:           .a_sum_stride = input_size,
0580:           .requantization_params = requantization_params,
0581:           .ukernel = pytorch_qnnp_params.q8conv_xzp.gemm,
0582:       };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8gemm_xzp_context`, `pthreadpool_compute_3d_tiled`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8gemm_xzp_context`, `pthreadpool_compute_3d_tiled`。

### Lines 583-605 / 第 583-605 行

```cpp
0583:       pthreadpool_compute_4d_tiled(
0584:           threadpool,
0585:           (pthreadpool_function_4d_tiled_t)compute_q8gemm_xzp,
0586:           &q8gemm_xzp_context,
0587:           groups,
0588:           batch_size * input_size,
0589:           input_size,
0590:           group_output_channels,
0591:           1,
0592:           input_size,
0593:           mr,
0594:           nr);
0595:       break;
0596:     }
0597:     case pytorch_qnnp_ukernel_type_gemm: {
0598:       const size_t group_input_channels = convolution->group_input_channels;
0599:       const size_t group_output_channels = convolution->group_output_channels;
0600:       const uint32_t mr = pytorch_qnnp_params.q8conv.mr;
0601:       const uint32_t nr = pytorch_qnnp_params.q8conv.nr;
0602:       const uint32_t kr = pytorch_qnnp_params.q8conv.kr;
0603:       const size_t k_stride = (group_input_channels + (kr - 1)) & -kr;
0604:       const size_t n_stride = (group_output_channels + (nr - 1)) & -nr;
0605: 
```

- **EN:** This block implements local helper logic for `conv-run`. Key symbols: `pthreadpool_compute_4d_tiled`.
- **CN:** 该代码块实现与 `conv-run` 相关的局部辅助逻辑。关键符号：`pthreadpool_compute_4d_tiled`。

### Lines 606-643 / 第 606-643 行

```cpp
0606:       struct q8gemm_context q8gemm_context = {
0607:           .k = convolution->group_input_channels,
0608:           .k_stride = k_stride,
0609:           .n = convolution->group_output_channels,
0610:           .n_stride = n_stride,
0611:           .a = input,
0612:           .a_stride = input_pixel_stride,
0613:           .packed_w = (uint8_t*)packed_weights,
0614:           .c = output,
0615:           .c_stride = output_pixel_stride,
0616:           .quantization_params = conv_quantization_params,
0617:           .ukernel = pytorch_qnnp_params.q8conv.gemm,
0618:       };
0619: 
0620:       pthreadpool_compute_4d_tiled(
0621:           threadpool,
0622:           (pthreadpool_function_4d_tiled_t)compute_q8gemm,
0623:           &q8gemm_context,
0624:           groups,
0625:           batch_size * output_size,
0626:           output_size,
0627:           group_output_channels,
0628:           1,
0629:           output_size,
0630:           mr,
0631:           nr);
0632:       break;
0633:     }
0634:     case pytorch_qnnp_ukernel_type_conv: {
0635:       const size_t group_input_channels = convolution->group_input_channels;
0636:       const size_t group_output_channels = convolution->group_output_channels;
0637:       const uint32_t mr = pytorch_qnnp_params.q8conv.mr;
0638:       const uint32_t nr = pytorch_qnnp_params.q8conv.nr;
0639:       const uint32_t kr = pytorch_qnnp_params.q8conv.kr;
0640:       const size_t k_stride = (group_input_channels + (kr - 1)) & -kr;
0641:       const size_t n_stride = (group_output_channels + (nr - 1)) & -nr;
0642:       const size_t m_stride = round_up(output_size, mr);
0643: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8gemm_context`, `pthreadpool_compute_4d_tiled`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8gemm_context`, `pthreadpool_compute_4d_tiled`。

### Lines 644-680 / 第 644-680 行

```cpp
0644:       struct q8conv_context q8conv_context = {
0645:           .bs = batch_size,
0646:           .ks = kernel_size,
0647:           .kc = group_input_channels,
0648:           .kc_stride = k_stride * kernel_size,
0649:           .m = output_size,
0650:           .m_stride = m_stride,
0651:           .n = group_output_channels,
0652:           .n_stride = n_stride,
0653:           .indirect_a = (const uint8_t**)convolution->indirection_buffer,
0654:           .packed_w = packed_weights,
0655:           .c = output,
0656:           .c_stride = output_pixel_stride,
0657:           .quantization_params = conv_quantization_params,
0658:           .ukernel = pytorch_qnnp_params.q8conv.conv,
0659:       };
0660: 
0661:       pthreadpool_compute_4d_tiled(
0662:           threadpool,
0663:           (pthreadpool_function_4d_tiled_t)compute_q8conv,
0664:           &q8conv_context,
0665:           groups,
0666:           batch_size,
0667:           output_size,
0668:           group_output_channels,
0669:           1,
0670:           1,
0671:           mr,
0672:           nr);
0673:       break;
0674:     }
0675:     default: {
0676:       pytorch_qnnp_log_error("Invalid kernel type. QNNPACK convolution run failed.");
0677:       PYTORCH_QNNP_UNREACHABLE;
0678:     }
0679:   }
0680:   return pytorch_qnnp_status_success;
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8conv_context`, `pthreadpool_compute_4d_tiled`, `pytorch_qnnp_log_error`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8conv_context`, `pthreadpool_compute_4d_tiled`, `pytorch_qnnp_log_error`。

### Lines 681-682 / 第 681-682 行

```cpp
0681: }
0682: } // namespace qnnpack
```

- **EN:** Opens or closes namespace scopes so the remaining declarations stay inside the intended ATen/backend module.
- **CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 ATen/后端模块之内。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Linear algebra backend integration** — 线性代数后端集成
- **Memory allocation strategy** — 内存分配策略
- **Runtime validation and invariants** — 运行时校验与不变量
- **Parallel scan/reduction behavior** — 并行扫描/归约行为
- **Core symbols: q8gemm_xzp_context, q8gemm_context, q8conv_context, q8sum_rows_context, q8dwconv2d_context, q8dwconv3d_context, QnnpackDeleter, pytorch_qnnp_status** — 核心符号：q8gemm_xzp_context、q8gemm_context、q8conv_context、q8sum_rows_context、q8dwconv2d_context、q8dwconv3d_context、QnnpackDeleter、pytorch_qnnp_status

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `qnnpack/indirection.h`, `qnnpack/log.h`, `qnnpack/operator.h`, `qnnpack/pack.h`, `qnnpack_func.h`, `cstring`, `memory`, `numeric`
- **Namespaces / 命名空间**: `qnnpack`
- **Representative symbols / 代表性符号**: `q8gemm_xzp_context`, `q8gemm_context`, `q8conv_context`, `q8sum_rows_context`, `q8dwconv2d_context`, `q8dwconv3d_context`, `QnnpackDeleter`, `pytorch_qnnp_status`, `compute_q8gemm_xzp`, `compute_q8gemm`, `compute_q8conv`, `compute_sum_rows`, `...`
