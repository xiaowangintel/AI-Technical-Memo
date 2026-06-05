# XnnpackUtils.h — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/XnnpackUtils.h`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares quantized CPU operator implementations, scale/zero-point handling, and packed weights in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了量化 CPU 算子实现、scale/zero-point 处理与打包权重逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
```cpp
 1: #pragma once
 2:
 3: #ifdef USE_XNNPACK
 4: #include <cstdint>
 5:
 6: #include <ATen/core/Tensor.h>
 7: #include <ATen/native/xnnpack/Common.h>
 8:
 9: using xnnpack_operator = at::native::xnnpack::Operator;
10:
11: namespace at::native::xnnp_utils {
12:
13: /*
14:  * Return shape in the same order as the memory format
15:  * e.g. channels_last will return NHWC instead of NCHW
16:  */
17: std::vector<size_t> get_mem_format_aware_shape(const at::Tensor& in);
```
- EN: This range pulls in required headers, including `cstdint`, `ATen/core/Tensor.h`, `ATen/native/xnnpack/Common.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `cstdint`, `ATen/core/Tensor.h`, `ATen/native/xnnpack/Common.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 19-32
```cpp
19: /*
20:  * Input is always int8_t, output can be [int8_t, uint8_t].
21:  * input  + offset = output
22:  * int8_t + 128    = uint8_t
23:  * int8_t + 0      = int8_t
24:  */
25: template <typename PT>
26: void q8_copy_int8_weight_and_add_offset(const at::Tensor& in, at::Tensor& out);
27:
28: template <int kSpatialDim>
29: Tensor convert_conv_weights_to_channel_last_tensor(
30:     const at::Tensor& src,
31:     int groups,
32:     bool transpose);
```
- EN: The math and shape handling relate to convolution-style operators.
- CN: 这里的计算与形状处理与卷积类算子相关。

### Lines 34-44
```cpp
34: /*
35:  * Series of create wrapper functions to call xnn_create_[de]conv* functions.
36:  */
37: C10_ALWAYS_INLINE
38: enum xnn_status xnnp_create_convolution2d_nhwc(
39:     uint32_t pad_top,
40:     uint32_t pad_right,
41:     uint32_t pad_bottom,
42:     uint32_t pad_left,
43:     uint32_t kernel_h,
44:     uint32_t kernel_w,
```
- EN: The math and shape handling relate to convolution-style operators.
- CN: 这里的计算与形状处理与卷积类算子相关。

### Lines 45-56
```cpp
45:     uint32_t stride_h,
46:     uint32_t stride_w,
47:     uint32_t dilation_h,
48:     uint32_t dilation_w,
49:     uint32_t groups,
50:     size_t group_input_channels,
51:     size_t group_output_channels,
52:     size_t ip_chan_stride,
53:     size_t op_chan_stride,
54:     int8_t izp,
55:     float ip_scale,
56:     int8_t kzp,
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 57-70
```cpp
57:     const float* k_scales,
58:     const int8_t* kernel,
59:     const int32_t* bias,
60:     int8_t ozp,
61:     float op_scale,
62:     int8_t op_min,
63:     int8_t op_max,
64:     uint32_t flags,
65:     xnn_operator_t* op,
66:     bool per_channel,
67:     bool transpose) {
68:   /* Symmetric quantization forces kzp = 0 */
69:   TORCH_CHECK(!kzp, "XNNPACK Q[SC]8 conv kernels expects kernel zero point to be zero."
70:                     "But got: ", kzp);
```
- EN: The math and shape handling relate to convolution-style operators.
- CN: 这里的计算与形状处理与卷积类算子相关。

### Lines 72-82
```cpp
72:   if (transpose) {
73:     TORCH_CHECK(!per_channel, "XNNPACK Q[SC]8 does not have a per channel deconvolution!");
74:     return xnn_create_deconvolution2d_nhwc_qs8(
75:         pad_top,        /* uint32_t output_padding_top          */
76:         pad_right,      /* uint32_t output_padding_right        */
77:         pad_bottom,     /* uint32_t output_padding_bottom       */
78:         pad_left,       /* uint32_t output_padding_left         */
79:         kernel_h,       /* uint32_t kernel_height               */
80:         kernel_w,       /* uint32_t kernel_width                */
81:         stride_h,       /* uint32_t stride_height               */
82:         stride_w,       /* uint32_t stride_width                */
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 83-94
```cpp
83:         dilation_h,     /* uint32_t dilation_height             */
84:         dilation_w,     /* uint32_t dilation_width              */
85:         groups,         /* uint32_t groups                      */
86:         group_input_channels,  /* size_t group_input_channels   */
87:         group_output_channels, /* size_t group_output_channels  */
88:         ip_chan_stride, /* size_t input_pixel_stride            */
89:         op_chan_stride, /* size_t output_pixel_stride           */
90:         izp,            /* int8_t input_zero_point              */
91:         ip_scale,       /* float input_scale                    */
92:         k_scales[0],    /* float kernel_scale                   */
93:         kernel,         /* const int8_t* kernel                 */
94:         bias,           /* const int32_t* bias                  */
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 95-106
```cpp
 95:         ozp,            /* int8_t output_zero_point             */
 96:         op_scale,       /* float output_scale                   */
 97:         op_min,         /* int8_t output_min                    */
 98:         op_max,         /* int8_t output_max                    */
 99:         flags,          /* uint32_t flags                       */
100:         nullptr,        /* xnn_caches_t caches                  */
101:         nullptr,        /* xnn_weights_cache_t weights_cache    */
102:         op);            /* xnn_operator_t* deconvolution_op_out */
103:
104:   }
105:
106:   if (!per_channel) {
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这里的计算与形状处理与卷积类算子相关。

### Lines 107-118
```cpp
107:     return xnn_create_convolution2d_nhwc_qs8(
108:         pad_top,        /* uint32_t input_padding_top         */
109:         pad_right,      /* uint32_t input_padding_right       */
110:         pad_bottom,     /* uint32_t input_padding_bottom      */
111:         pad_left,       /* uint32_t input_padding_left        */
112:         kernel_h,       /* uint32_t kernel_height             */
113:         kernel_w,       /* uint32_t kernel_width              */
114:         stride_h,       /* uint32_t subsampling_height        */
115:         stride_w,       /* uint32_t subsampling_width         */
116:         dilation_h,     /* uint32_t dilation_height           */
117:         dilation_w,     /* uint32_t dilation_width            */
118:         groups,         /* uint32_t groups                    */
```
- EN: The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 119-130
```cpp
119:         group_input_channels,  /* size_t group_input_channels */
120:         group_output_channels, /* size_t group_output_channels*/
121:         ip_chan_stride, /* size_t input_channel_stride        */
122:         op_chan_stride, /* size_t output_channel_stride       */
123:         izp,            /* int8_t input_zero_point            */
124:         ip_scale,       /* float input_scale                  */
125:         k_scales[0],    /* float kernel_scale                 */
126:         kernel,         /* const int8_t* kernel               */
127:         bias,           /* const int32_t* bias                */
128:         ozp,            /* int8_t output_zero_point           */
129:         op_scale,       /* float output_scale                 */
130:         op_min,         /* int8_t output_min                  */
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 131-142
```cpp
131:         op_max,         /* int8_t output_max                  */
132:         flags,          /* uint32_t flags                     */
133:         nullptr,        /* xnn_caches_t caches                */
134:         nullptr,        /* xnn_weights_cache_t weights_cache    */
135:         op);            /* xnn_operator_t* convolution_op_out */
136:   } else { /* per_channel */
137:     return xnn_create_convolution2d_nhwc_qs8_qc8w(
138:         pad_top,        /* uint32_t input_padding_top         */
139:         pad_right,      /* uint32_t input_padding_right       */
140:         pad_bottom,     /* uint32_t input_padding_bottom      */
141:         pad_left,       /* uint32_t input_padding_left        */
142:         kernel_h,       /* uint32_t kernel_height             */
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 143-154
```cpp
143:         kernel_w,       /* uint32_t kernel_width              */
144:         stride_h,       /* uint32_t subsampling_height        */
145:         stride_w,       /* uint32_t subsampling_width         */
146:         dilation_h,     /* uint32_t dilation_height           */
147:         dilation_w,     /* uint32_t dilation_width            */
148:         groups,         /* uint32_t groups                    */
149:         group_input_channels,  /* size_t group_input_channels */
150:         group_output_channels, /* size_t group_output_channels*/
151:         ip_chan_stride, /* size_t input_channel_stride        */
152:         op_chan_stride, /* size_t output_channel_stride       */
153:         izp,            /* int8_t input_zero_point            */
154:         ip_scale,       /* float input_scale                  */
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 155-167
```cpp
155:         k_scales,       /* const float* kernel_scale          */
156:         kernel,         /* const int8_t* kernel               */
157:         bias,           /* const int32_t* bias                */
158:         ozp,            /* int8_t output_zero_point           */
159:         op_scale,       /* float output_scale                 */
160:         op_min,         /* int8_t output_min                  */
161:         op_max,         /* int8_t output_max                  */
162:         flags,          /* uint32_t flags                     */
163:         nullptr,        /* xnn_caches_t caches                */
164:         nullptr,        /* xnn_weights_cache_t weights_cache    */
165:         op);            /* xnn_operator_t* convolution_op_out */
166:   }
167: }
```
- EN: The math and shape handling relate to convolution-style operators.
- CN: 这里的计算与形状处理与卷积类算子相关。

### Lines 169-179
```cpp
169: /*
170:  * Series of reshape wrapper functions to call xnn_reshape_[de]conv* functions.
171:  */
172: C10_ALWAYS_INLINE
173: enum xnn_status xnnp_reshape_convolution2d_nhwc(
174:     xnn_operator_t op,
175:     size_t batch,
176:     size_t in_h,
177:     size_t in_w,
178:     pthreadpool_t pt_pool,
179:     bool per_channel = false,
```
- EN: The math and shape handling relate to convolution-style operators. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这里的计算与形状处理与卷积类算子相关。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 180-195
```cpp
180:     bool transpose = false,
181:     uint32_t adj_h = 0,
182:     uint32_t adj_w = 0) {
183:   if(transpose) {
184:     TORCH_CHECK(!per_channel, "XNNPACK Q[SC]8 does not have a per channel deconvolution!");
185:     return xnn_reshape_deconvolution2d_nhwc_qs8(
186:         op,       /* xnn_operator_t deconvolution_op */
187:         batch,    /* size_t batch_size               */
188:         in_h,     /* size_t input_height             */
189:         in_w,     /* size_t input_width              */
190:         adj_h,    /* uint32_t adjustment_height      */
191:         adj_w,    /* uint32_t adjustment_width       */
192:         nullptr,  /* size_t* output_height_out       */
193:         nullptr,  /* size_t* output_width_out        */
194:         pt_pool); /* pthreadpool_t threadpool        */
195:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 197-207
```cpp
197:   size_t workspace_size = SIZE_MAX;
198:   size_t workspace_alignment = SIZE_MAX;
199:
200:   if (!per_channel) {
201:     return xnn_reshape_convolution2d_nhwc_qs8(
202:         op,       /* xnn_operator_t convolution_op */
203:         batch,    /* size_t batch_size             */
204:         in_h,     /* size_t input_height           */
205:         in_w,     /* size_t input_width            */
206:         &workspace_size, /* size_t* workspace_size */
207:         &workspace_alignment, /* size_t* workspace_alignment */
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 208-223
```cpp
208:         nullptr,  /* size_t* output_height_out     */
209:         nullptr,  /* size_t* output_width_out      */
210:         pt_pool); /* pthreadpool_t threadpool      */
211:   } else { /* per_channel */
212:     return xnn_reshape_convolution2d_nhwc_qs8_qc8w(
213:         op,       /* xnn_operator_t convolution_op */
214:         batch,    /* size_t batch_size             */
215:         in_h,     /* size_t input_height           */
216:         in_w,     /* size_t input_width            */
217:         &workspace_size, /* size_t* workspace_size */
218:         &workspace_alignment, /* size_t* workspace_alignment */
219:         nullptr,  /* size_t* output_height_out     */
220:         nullptr,  /* size_t* output_width_out      */
221:         pt_pool); /* pthreadpool_t threadpool      */
222:   }
223: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 226-237
```cpp
226: /*
227:  * Series of setup wrapper functions to call xnn_setup_[de]conv* functions.
228:  */
229: C10_ALWAYS_INLINE
230: enum xnn_status xnnp_setup_convolution2d_nhwc(
231:     xnn_operator_t op,
232:     const int8_t* inp,
233:     int8_t* outp,
234:     bool per_channel = false,
235:     bool transpose = false) {
236:   if(transpose) {
237:     TORCH_CHECK(!per_channel, "XNNPACK Q[SC]8 does not have a per channel deconvolution!");
```
- EN: The main symbol in this range is `xnnp_setup_convolution2d_nhwc`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `xnnp_setup_convolution2d_nhwc`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这里的计算与形状处理与卷积类算子相关。

### Lines 239-249
```cpp
239:     return xnn_setup_deconvolution2d_nhwc_qs8(
240:         op,       /* xnn_operator_t deconvolution_op */
241:         inp,      /* const int8_t* input             */
242:         outp);    /* int8_t* output                  */
243:   }
244:
245:   if (!per_channel) {
246:     return xnn_setup_convolution2d_nhwc_qs8(
247:         op,       /* xnn_operator_t deconvolution_op */
248:         nullptr,  /* void workspace                  */
249:         inp,      /* const int8_t* input             */
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 250-261
```cpp
250:         outp);    /* int8_t* output                  */
251:   } else { /* per_channel */
252:     return xnn_setup_convolution2d_nhwc_qs8_qc8w(
253:         op,       /* xnn_operator_t deconvolution_op */
254:         nullptr,  /* void workspace                  */
255:         inp,      /* const int8_t* input             */
256:         outp);    /* int8_t* output                  */
257:   }
258: }
259:
260:
261: /*
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 262-273
```cpp
262:  * Series of wrapper functions to call xnn_create* and xnn_setup*
263:  * functions for linear
264:  */
265: C10_ALWAYS_INLINE
266: enum xnn_status xnnp_create_fully_connected_nc(
267:     size_t input_channels,
268:     size_t output_channels,
269:     size_t input_stride,
270:     size_t output_stride,
271:     int8_t input_zero_point,
272:     float input_scale,
273:     int8_t kernel_zero_point,
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 274-285
```cpp
274:     float kernel_scale,
275:     const int8_t* kernel,
276:     const int32_t* bias,
277:     int8_t output_zero_point,
278:     float output_scale,
279:     int8_t output_min,
280:     int8_t output_max,
281:     uint32_t flags,
282:     xnn_operator_t* fully_connected_op_out) {
283:   /* Symmetric quantization forces kzp = 0 */
284:   TORCH_CHECK(!kernel_zero_point, "XNNPACK QS8 linear kernel expects kernel zero point to be zero."
285:                     "But got: ", kernel_zero_point);
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 286-297
```cpp
286:   return xnn_create_fully_connected_nc_qs8(
287:       input_channels,          /* size_t input_channels                  */
288:       output_channels,         /* size_t output_channels                 */
289:       input_stride,            /* size_t input_stride                    */
290:       output_stride,           /* size_t output_stride                   */
291:       input_zero_point,        /* int8_t input_zero_point                */
292:       input_scale,             /* float input_scale                      */
293:       kernel_scale,            /* float kernel_scale                     */
294:       kernel,                  /* const int8_t* kernel                   */
295:       bias,                    /* const int32_t* bias                    */
296:       output_zero_point,       /* int8_t output_zero_point               */
297:       output_scale,            /* float output_scale                     */
```
- EN: The block also assembles or returns the result expected by the surrounding operator code.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 298-309
```cpp
298:       output_min,              /* int8_t output_min                      */
299:       output_max,              /* int8_t output_max                      */
300:       flags,                   /* uint32_t flags                         */
301:       nullptr,                 /* xnn_caches_t caches                    */
302:       nullptr,                 /* xnn_weights_cache_t                    */
303:       fully_connected_op_out); /* xnn_operator_t* fully_connected_op_out */
304: }
305:
306: C10_ALWAYS_INLINE
307: enum xnn_status xnnp_reshape_fully_connected_nc(
308:     xnn_operator_t fully_connected_op,
309:     size_t batch_size,
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 310-321
```cpp
310:     pthreadpool_t threadpool) {
311:   return xnn_reshape_fully_connected_nc_qs8(
312:       fully_connected_op, /* xnn_operator_t fully_connected_op */
313:       batch_size,         /* size_t batch_size                 */
314:       threadpool);        /* pthreadpool_t threadpool          */
315: }
316:
317: C10_ALWAYS_INLINE
318: enum xnn_status xnnp_setup_fully_connected_nc(
319:     xnn_operator_t fully_connected_op,
320:     const int8_t* input,
321:     int8_t* output) {
```
- EN: The main symbol in this range is `xnnp_setup_fully_connected_nc`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段的主要符号是 `xnnp_setup_fully_connected_nc`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 322-331
```cpp
322:   return xnn_setup_fully_connected_nc_qs8(
323:       fully_connected_op, /* xnn_operator_t fully_connected_op */
324:       input,              /* const int8_t* input               */
325:       output              /* int8_t* output                    */
326:     );
327: }
328:
329: } // namespace at::native::xnnp_utils
330:
331: #endif // USE_XNNPACK
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 该代码块还会组装或返回外围算子代码所需的结果。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Pooling reductions / 池化归约
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/native/xnnpack/Common.h`
- Standard or third-party headers / 标准库或第三方头文件: `cstdint`
