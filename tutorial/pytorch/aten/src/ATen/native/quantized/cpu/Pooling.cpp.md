# Pooling.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/Pooling.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU pooling kernels and pooled-output shape or reduction logic in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 池化 kernel，以及池化输出形状或归约逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/Context.h>
 4: #include <ATen/Dispatch.h>
 5: #include <ATen/Parallel.h>
 6: #include <torch/library.h>
 7: #include <ATen/native/Pool.h>
 8: #include <ATen/native/MaxPooling.h>
 9: #include <ATen/quantized/Quantizer.h>
10: #include <ATen/native/quantized/cpu/QuantizedOps.h>
11: #include <ATen/native/quantized/cpu/init_qnnpack.h>
12: #include <ATen/native/quantized/cpu/QnnpackUtils.h>
13: #include <c10/util/irange.h>
14: #include <caffe2/utils/threadpool/pthreadpool-cpp.h>
15:
16: #ifndef AT_PER_OPERATOR_HEADERS
17: #include <ATen/Functions.h>
18: #include <ATen/NativeFunctions.h>
19: #else
20: #include <ATen/ops/empty.h>
21: #include <ATen/ops/_empty_affine_quantized.h>
22: #include <ATen/ops/quantized_max_pool1d.h>
23: #include <ATen/ops/quantized_max_pool1d_native.h>
24: #include <ATen/ops/quantized_max_pool2d.h>
25: #include <ATen/ops/quantized_max_pool2d_native.h>
26: #include <ATen/ops/quantized_max_pool3d_native.h>
27: #endif
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/Context.h`, `ATen/Dispatch.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/Context.h`, `ATen/Dispatch.h`。 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 29-47
```cpp
29: #include <algorithm>
30: #include <vector>
31:
32: namespace at::native {
33:
34: DEFINE_DISPATCH(qmaxpool_2d_nhwc_stub);
35: DEFINE_DISPATCH(qmaxpool_3d_nthwc_stub);
36:
37: namespace {
38:
39: /* Computes the spatial 2D max pooling with dilation.
40:
41: Argument description in the argument list.
42: */
43: template <typename T>
44: void spatial_dilated_max_pooling(
45:     const T* iData,
46:     int64_t iC, // input/output channels
47:     int64_t iH,
```
- EN: This range pulls in required headers, including `algorithm`, `vector`. The code enters or documents the namespace scope used by ATen native CPU operators. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段引入了所需头文件，例如 `algorithm`, `vector`。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 48-72
```cpp
48:     int64_t iW, // input sizes
49:     int64_t oH,
50:     int64_t oW, // output sizes
51:     int64_t kH,
52:     int64_t kW, // kernel size
53:     int64_t sH,
54:     int64_t sW, // strides
55:     int64_t pH,
56:     int64_t pW, // padding
57:     int64_t dH,
58:     int64_t dW, // dilation
59:     T* oData) { // output arrays (data and max-index)
60:   at::parallel_for(0, iC, 0, [&](int64_t start, int64_t end) {
61:     for (const auto p : c10::irange(start, end)) {
62:       const T* i_p = iData + p * iW * iH;
63:       for (int64_t row = 0; row < oH; ++row) {
64:         for (int64_t col = 0; col < oW; ++col) {
65:           int64_t h_start = row * sH - pH;
66:           int64_t w_start = col * sW - pW;
67:           int64_t h_end = std::min(h_start + (kH - 1) * dH + 1, iH);
68:           int64_t w_end = std::min(w_start + (kW - 1) * dW + 1, iW);
69:           while (h_start < 0)
70:             h_start += dH;
71:           while (w_start < 0)
72:             w_start += dW;
```
- EN: The main symbol in this range is `arrays`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `arrays`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 74-94
```cpp
74:           // local pointers
75:           T* o_p = oData + p * oW * oH + row * oW + col;
76:
77:           // local max
78:           auto max_val = std::numeric_limits<typename T::underlying>::lowest();
79:           int64_t tcntr = 0; // center point
80:           for (int64_t y = h_start; y < h_end; y += dH) {
81:             for (int64_t x = w_start; x < w_end; x += dW) {
82:               tcntr = y * iW + x;
83:               auto val = (i_p + tcntr)->val_;
84:               if (val > max_val) {
85:                 max_val = val;
86:               }
87:             }
88:           }
89:           *o_p = T(max_val); // Output.
90:         }
91:       }
92:     }
93:   });
94: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 96-114
```cpp
 96: template <typename T>
 97: void spatial_dilated_max_pooling3d(
 98:     const T* qxd,
 99:     int64_t nbatch,
100:     int64_t iC, // input/output channels
101:     int64_t iT,
102:     int64_t iH,
103:     int64_t iW, // input sizes
104:     int64_t oT,
105:     int64_t oH,
106:     int64_t oW, // output sizes
107:     int64_t kT,
108:     int64_t kH,
109:     int64_t kW, // kernel size
110:     int64_t sT,
111:     int64_t sH,
112:     int64_t sW, // strides
113:     int64_t pT,
114:     int64_t pH,
```
- EN: The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 115-134
```cpp
115:     int64_t pW, // padding
116:     int64_t dT,
117:     int64_t dH,
118:     int64_t dW, // dilation
119:     T* qyd) { // output arrays (data and max-index)
120:   // TODO: Further optimize the performance suggested by @mingfeima. Parallel on NCTH and cache the output indices from W.
121:   // Handle each bs
122:   int64_t oC = iC;
123:   int64_t parallel_dim = nbatch * iC;
124:   at::parallel_for(0, parallel_dim, 0, [&](int64_t start, int64_t end) {
125:     for (const auto p : c10::irange(start, end)) {
126:
127:       int64_t batch_idx = p / iC;
128:       int64_t channel_idx = p - batch_idx * iC;
129:
130:       auto* iData = qxd + batch_idx * iC * iT * iH * iW;
131:       auto* oData = qyd + batch_idx * oC * oT * oH * oW;
132:
133:       // Handle each Channel
134:       int64_t time, row, col;
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 135-155
```cpp
135:       const T* i_p = iData + channel_idx * iT * iW * iH;
136:       for (time = 0; time < oT; ++time) {
137:         for (row = 0; row < oH; ++row) {
138:           for (col = 0; col < oW; ++col) {
139:             // Handle each output element
140:             int64_t t_start = time * sT - pT;
141:             int64_t h_start = row * sH - pH;
142:             int64_t w_start = col * sW - pW;
143:             int64_t t_end = std::min(t_start + (kT - 1) * dT + 1, iT);
144:             int64_t h_end = std::min(h_start + (kH - 1) * dH + 1, iH);
145:             int64_t w_end = std::min(w_start + (kW - 1) * dW + 1, iW);
146:
147:             while (t_start < 0)
148:               t_start += dT;
149:             while (h_start < 0)
150:               h_start += dH;
151:             while (w_start < 0)
152:               w_start += dW;
153:
154:             // local pointers
155:             T* o_p = oData + channel_idx * oT * oH * oW  + time * oH * oW  + row * oW + col;
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 157-177
```cpp
157:             // local max
158:             auto max_val = std::numeric_limits<typename T::underlying>::lowest();
159:             int64_t tcntr = 0; // center point
160:             for (int64_t t = t_start; t < t_end; t += dT) {
161:               for (int64_t y = h_start; y < h_end; y += dH) {
162:                 for (int64_t x = w_start; x < w_end; x += dW) {
163:                   tcntr = t * iH * iW + y * iW + x;
164:                   auto val = (i_p + tcntr)->val_;
165:                   if (val > max_val) {
166:                     max_val = val;
167:                   }
168:                 }
169:               }
170:             }
171:             *o_p = T(max_val); // Output.
172:           }
173:         }
174:       }
175:     }
176:   });
177: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 179-201
```cpp
179: template <typename Q>
180: Tensor q_maxpool_2d(
181:     Tensor qx, // Input Tensor (Quantized)
182:     int64_t kH,
183:     int64_t kW, // kernel size
184:     int64_t sH,
185:     int64_t sW, // strides
186:     int64_t pH,
187:     int64_t pW, // padding
188:     int64_t dH,
189:     int64_t dW,
190:     bool ceil_mode) { // dilation
191:   // Check input dimensions.
192:   TORCH_CHECK(kH > 0 && kW > 0, "kernel_size should be greater than zero.");
193:   TORCH_CHECK(sH > 0 && sW > 0, "strides should be greater than zero.");
194:   TORCH_CHECK(
195:       dH > 0 && dW > 0,
196:       "dilation should be greater than zero. "
197:       "Got (",
198:       dH,
199:       ", ",
200:       dW,
201:       ")");
```
- EN: The main symbol in this range is `q_maxpool_2d`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段的主要符号是 `q_maxpool_2d`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 203-227
```cpp
203:   int ndim = qx.dim();
204:   TORCH_CHECK(
205:       ndim == 3 || ndim == 4, "Expecting the input tensor of rank 3 or 4.");
206:   int dimc = 0;
207:   int dimh = 1;
208:   int dimw = 2;
209:   int nbatch = 1;
210:   if (ndim == 4) { // Includes batches
211:     ++dimc;
212:     ++dimh;
213:     ++dimw;
214:     nbatch = qx.size(0);
215:   }
216:
217:   // Check if inputs are valid.
218:   int64_t iC = qx.size(dimc);
219:   int64_t iH = qx.size(dimh);
220:   int64_t iW = qx.size(dimw);
221:   TORCH_CHECK(iC > 0 && iH > 0 && iW > 0, "input dimensions must be non-zero.");
222:   TORCH_CHECK(
223:       (ndim == 3 || ndim == 4),
224:       "non-empty 3D or 4D input tensor is expected.");
225:   TORCH_CHECK(
226:       kH / 2 >= pH && kW / 2 >= pW,
227:       "padding should be smaller than half of kernel_size.");
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 229-247
```cpp
229:   // Check output dimensions.
230:   int64_t oC = iC;
231:   int64_t oH = pooling_output_shape(iH, kH, pH, sH, dH, ceil_mode);
232:   int64_t oW = pooling_output_shape(iW, kW, pW, sW, dW, ceil_mode);
233:   TORCH_CHECK(oH > 0 && oW > 0,
234:               "Given input size: (",
235:               iC, "x", iH, "x", iW,
236:               "). Calculated output size: (",
237:               oC, "x", oH, "x", oW,
238:               "). Output size is too small.");
239:
240:   std::vector<int64_t> oSizes;
241:   if (ndim == 3) {
242:     oSizes = {oC, oH, oW};
243:   } else {
244:     oSizes = {nbatch, oC, oH, oW};
245:   }
246:
247:   if (qx.is_contiguous(c10::MemoryFormat::ChannelsLast)) {
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 248-267
```cpp
248:     // Fast path case for channels-last case.
249:     // In this case, we can preserve the data layout in memory
250:     // as well as use a loop nest that is more amenable to
251:     // vectorization.
252:     Tensor qy;
253:     if constexpr(std::is_same_v<Q, uint8_t>) {
254:       qy = at::empty(
255:         oSizes,
256:         qx.options()
257:           .device(c10::kCPU)
258:           .dtype(qx.scalar_type())
259:           .memory_format(c10::MemoryFormat::ChannelsLast));
260:     } else {
261:       qy = at::_empty_affine_quantized(
262:           oSizes,
263:           qx.options()
264:             .dtype(toQIntType(qx.scalar_type()))
265:             .memory_format(qx.suggest_memory_format()),
266:           qx.q_scale(),
267:           qx.q_zero_point(),
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 268-287
```cpp
268:           std::nullopt);
269:     }
270:     qmaxpool_2d_nhwc_stub(qx.device().type(), qx, iC, iH, iW, oH, oW, kH, kW, sH, sW, pH, pW, dH, dW, qy);
271:     return qy;
272:   } else {
273:     Tensor qy;
274:     if constexpr(!std::is_same_v<Q, uint8_t>) {
275:       qy = at::_empty_affine_quantized(
276:               oSizes,
277:               qx.options().dtype(toQIntType(qx.scalar_type())),
278:               qx.q_scale(),
279:               qx.q_zero_point());
280:       auto qx_contig = qx.contiguous();
281:       auto qxd = qx_contig.data_ptr<Q>();
282:       auto qyd = qy.data_ptr<Q>();
283:       if (ndim == 3 || nbatch == 1) {
284:         auto* iData = qxd;
285:         auto* oData = qyd;
286:         spatial_dilated_max_pooling<Q>(
287:             iData,
```
- EN: The main symbol in this range is `constexpr`, `qmaxpool_2d_nhwc_stub`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `constexpr`, `qmaxpool_2d_nhwc_stub`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 288-307
```cpp
288:             iC,
289:             iH,
290:             iW,
291:             oH,
292:             oW,
293:             kH,
294:             kW,
295:             sH,
296:             sW,
297:             pH,
298:             pW,
299:             dH,
300:             dW,
301:             oData);
302:       } else {
303:         at::parallel_for(0, nbatch, 0, [&](int64_t start, int64_t end) {
304:           for (const auto p : c10::irange(start, end)) {
305:             auto* iData = qxd + p * iC * iW * iH;
306:             auto* oData = qyd + p * oC * oW * oH;
307:             spatial_dilated_max_pooling<Q>(
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 308-327
```cpp
308:                 iData,
309:                 iC,
310:                 iH,
311:                 iW,
312:                 oH,
313:                 oW,
314:                 kH,
315:                 kW,
316:                 sH,
317:                 sW,
318:                 pH,
319:                 pW,
320:                 dH,
321:                 dW,
322:                 oData);
323:           }
324:         });
325:       }
326:     } else {
327:       // If qx is uint8 and contiguous memory format,
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 328-347
```cpp
328:       // Use the channels_last implementation and convert qy back to contiguous.
329:       qy = at::empty(
330:         oSizes,
331:         qx.options()
332:           .device(c10::kCPU)
333:           .dtype(qx.scalar_type())
334:           .memory_format(c10::MemoryFormat::ChannelsLast));
335:       auto qx_nhwc = qx.contiguous(c10::MemoryFormat::ChannelsLast);
336:       qmaxpool_2d_nhwc_stub(qx_nhwc.device().type(), qx_nhwc, iC, iH, iW, oH, oW, kH, kW, sH, sW, pH, pW, dH, dW, qy);
337:       qy = qy.contiguous();
338:     }
339:     return qy;
340:   }
341: }
342:
343: template <typename Q>
344: Tensor q_maxpool_3d(
345:     Tensor qx, // Input Tensor (Quantized)
346:     int64_t kT,
347:     int64_t kH,
```
- EN: The main symbol in this range is `qmaxpool_2d_nhwc_stub`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `qmaxpool_2d_nhwc_stub`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 348-374
```cpp
348:     int64_t kW, // kernel size
349:     int64_t sT,
350:     int64_t sH,
351:     int64_t sW, // strides
352:     int64_t pT,
353:     int64_t pH,
354:     int64_t pW, // padding
355:     int64_t dT,
356:     int64_t dH,
357:     int64_t dW,
358:     bool ceil_mode) { // dilation
359:   // Check input dimensions.
360:   TORCH_CHECK(kT > 0 && kH > 0 && kW > 0, "kernel_size should be greater than zero.");
361:   TORCH_CHECK(sT > 0 && sH > 0 && sW > 0, "strides should be greater than zero.");
362:   TORCH_CHECK(
363:       dT > 0 && dH > 0 && dW > 0,
364:       "dilation should be greater than zero. "
365:       "Got (",
366:       dT,
367:       ", ",
368:       dH,
369:       ", ",
370:       dW,
371:       ")");
372:   int ndim = qx.dim();
373:   // TODO leslie: Support non batch mode input when input is THWC which is 4-d tensor.
374:   TORCH_CHECK(ndim == 5, "Expecting the input tensor of rank 5.");
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 376-402
```cpp
376:   // act: n, c, t, h, w
377:   int dimc = 1;
378:   int dimt = 2;
379:   int dimh = 3;
380:   int dimw = 4;
381:   int nbatch = qx.size(0);
382:   // Check if inputs are valid.
383:   int64_t iC = qx.size(dimc);
384:   int64_t iT = qx.size(dimt);
385:   int64_t iH = qx.size(dimh);
386:   int64_t iW = qx.size(dimw);
387:   TORCH_CHECK(iC > 0 && iT > 0 && iH > 0 && iW > 0, "input dimensions must be non-zero.");
388:   TORCH_CHECK(
389:       kT / 2 >= pT && kH / 2 >= pH && kW / 2 >= pW,
390:       "padding should be smaller than half of kernel_size.");
391:
392:   // Check output dimensions.
393:   int64_t oC = iC;
394:   int64_t oT = pooling_output_shape(iT, kT, pT, sT, dT, ceil_mode);
395:   int64_t oH = pooling_output_shape(iH, kH, pH, sH, dH, ceil_mode);
396:   int64_t oW = pooling_output_shape(iW, kW, pW, sW, dW, ceil_mode);
397:   TORCH_CHECK(oT > 0 && oH > 0 && oW > 0,
398:               "Given input size: (",
399:               iC, "t", iT , "x", iH, "x", iW,
400:               "). Calculated output size: (",
401:               oC, "t", oT , "x", oH, "x", oW,
402:               "). Output size is too small.");
```
- EN: The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 404-429
```cpp
404:   std::vector<int64_t> oSizes = {nbatch, oC, oT, oH, oW};
405:
406:   if (qx.is_contiguous(c10::MemoryFormat::ChannelsLast3d)) {
407:     // Fast path case for channels-last case.
408:     // In this case, we can preserve the data layout in memory
409:     // as well as use a loop nest that is more amenable to
410:     // vectorization.
411:     Tensor qy = at::_empty_affine_quantized(
412:         oSizes,
413:         qx.options()
414:           .dtype(toQIntType(qx.scalar_type()))
415:           .memory_format(qx.suggest_memory_format()),
416:         qx.q_scale(),
417:         qx.q_zero_point(),
418:         std::nullopt);
419:     qmaxpool_3d_nthwc_stub(qx.device().type(), qx, iC, iT, iH, iW, oT, oH, oW, kT, kH, kW, sT, sH, sW, pT, pH, pW, dT, dH, dW, qy);
420:     return qy;
421:   } else {
422:     Tensor qy = at::_empty_affine_quantized(
423:       oSizes,
424:       qx.options().dtype(toQIntType(qx.scalar_type())),
425:       qx.q_scale(),
426:       qx.q_zero_point());
427:     auto qx_contig = qx.contiguous();
428:     auto qxd = qx_contig.data_ptr<Q>();
429:     auto qyd = qy.data_ptr<Q>();
```
- EN: The main symbol in this range is `qmaxpool_3d_nthwc_stub`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `qmaxpool_3d_nthwc_stub`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 431-453
```cpp
431:     spatial_dilated_max_pooling3d<Q>(
432:         qxd,
433:         nbatch,
434:         iC,
435:         iT,
436:         iH,
437:         iW,
438:         oT,
439:         oH,
440:         oW,
441:         kT,
442:         kH,
443:         kW,
444:         sT,
445:         sH,
446:         sW,
447:         pT,
448:         pH,
449:         pW,
450:         dT,
451:         dH,
452:         dW,
453:         qyd);
```
- EN: The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 455-476
```cpp
455:     return qy;
456:   }
457: }
458: } // namespace
459:
460: namespace {
461: void check_maxpool2d_params(
462:     IntArrayRef kernel_size,
463:     IntArrayRef stride,
464:     IntArrayRef padding,
465:     IntArrayRef dilation) {
466:   TORCH_CHECK(kernel_size.size() == 1 || kernel_size.size() == 2,
467:               "Expected 1d or 2d kernel size, got ", kernel_size.size());
468:   TORCH_CHECK(stride.empty() || stride.size() == 2,
469:               "Expected no strides or 2d strides, got", stride.size());
470:   TORCH_CHECK(padding.size() == 1 || padding.size() == 2,
471:               "Expected 1d or 2d padding, got ", padding.size());
472:   TORCH_CHECK(dilation.size() == 1 || dilation.size() == 2,
473:               "Expected 1d or 2d dilation, got ", dilation.size());
474:   TORCH_CHECK(dilation.allMatch([](const auto& ele) { return ele >= 1L; }),
475:               "Expected dilation >= 1");
476: }
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `check_maxpool2d_params`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `check_maxpool2d_params`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 478-500
```cpp
478: void check_maxpool3d_params(
479:     IntArrayRef kernel_size,
480:     IntArrayRef stride,
481:     IntArrayRef padding,
482:     IntArrayRef dilation) {
483:   TORCH_CHECK(kernel_size.size() == 3, "Expected 3d kernel size, got ", kernel_size.size());
484:   TORCH_CHECK(stride.empty() || stride.size() == 3,
485:               "Expected no strides or 3d strides, got", stride.size());
486:   TORCH_CHECK(padding.size() == 3, "Expected 3d padding, got ", padding.size());
487:   TORCH_CHECK(dilation.size() == 3, "Expected 1d or 3d dilation, got ", dilation.size());
488:   TORCH_CHECK(dilation.allMatch([](const auto& ele) { return ele >= 1L; }),
489:               "Expected dilation >= 1");
490: }
491:
492: #ifdef USE_PYTORCH_QNNPACK
493:  static Tensor qnnpack_maxpool2d(
494:      Tensor input,
495:      IntArrayRef kernel_size,
496:      IntArrayRef stride,
497:      IntArrayRef padding,
498:      IntArrayRef dilation,
499:      bool ceil_mode) {
500:    Tensor qy;
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `check_maxpool3d_params`, `qnnpack_maxpool2d`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `check_maxpool3d_params`, `qnnpack_maxpool2d`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 502-521
```cpp
502:    TORCH_CHECK(
503:        input.ndimension() == 4,
504:        "qnnpack_maxpool2d(): Expected input to be 4-dimensional: got ",
505:        input.ndimension());
506:    TORCH_CHECK(
507:        kernel_size.size() == 2,
508:        "qnnpack_maxpool2d(): Expected kernel_size to be 2-dimensional: got ",
509:        kernel_size.size());
510:    TORCH_CHECK(
511:        stride.size() == 2,
512:        "qnnpack_maxpool2d(): Expected stride to be 2-dimensional: got ",
513:        stride.size());
514:    TORCH_CHECK(
515:        dilation.size() == 2,
516:        "qnnpack_maxpool2d(): Expected dilation to be 2-dimensional: got ",
517:        dilation.size());
518:    TORCH_CHECK(
519:        padding.size() == 2,
520:        "qnnpack_maxpool2d(): Expected padding to be 2-dimensional: got ",
521:        padding.size());
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 523-541
```cpp
523:    int64_t batch_size = input.size(0);
524:    int64_t inC = input.size(1);
525:    int64_t inH = input.size(2);
526:    int64_t inW = input.size(3);
527:    Tensor input_contig = input.contiguous(MemoryFormat::ChannelsLast);
528:
529:    initQNNPACK();
530:    const auto scale = input_contig.q_scale();
531:    const auto zero_point = input_contig.q_zero_point();
532:    pytorch_qnnp_operator_t qnnpack_operator{nullptr};
533:
534:    int64_t padH = padding[0];
535:    int64_t padW = padding[1];
536:    int64_t kH = kernel_size[0];
537:    int64_t kW = kernel_size[1];
538:    int64_t strideH = stride[0];
539:    int64_t strideW = stride[1];
540:    int64_t dilationH = dilation[0];
541:    int64_t dilationW = dilation[1];
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 543-567
```cpp
543:    TORCH_CHECK(
544:        kH > 0 && kW > 0,
545:        "qnnpack_maxpool2d(): kernel_size should be greater than zero.");
546:    TORCH_CHECK(
547:        strideH > 0 && strideW > 0,
548:        "qnnpack_maxpool2d(): strides should be greater than zero.");
549:
550:    const pytorch_qnnp_status createStatus =
551:        pytorch_qnnp_create_max_pooling2d_nhwc_u8(
552:            padH /* input_padding_height */,
553:            padW /* input_padding_width */,
554:            kH /* pooling height */,
555:            kW /* pooling width */,
556:            strideH /* stride height */,
557:            strideW /* stride width */,
558:            dilationH /* dilation height */,
559:            dilationW /* dilation width */,
560:            inC /* input channels */,
561:            std::numeric_limits<uint8_t>::min() /* output min */,
562:            std::numeric_limits<uint8_t>::max() /* output max */,
563:            0 /* flags */,
564:            &qnnpack_operator);
565:    TORCH_INTERNAL_ASSERT(
566:        createStatus == pytorch_qnnp_status_success,
567:        "failed to create QNNPACK MaxPool operator");
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 569-588
```cpp
569:    int64_t outC = inC;
570:    int64_t outH =
571:        pooling_output_shape(inH, kH, padH, strideH, dilationH, ceil_mode);
572:    int64_t outW =
573:        pooling_output_shape(inW, kW, padW, strideW, dilationW, ceil_mode);
574:
575:    TORCH_CHECK(
576:        outH > 0 && outW > 0,
577:        "qnnpack_maxpool2d(): the resulting output Tensor size should be >= 0");
578:
579:    std::unique_ptr<pytorch_qnnp_operator, QnnpackOperatorDeleter>
580:        qnnpack_uniq_ptr(qnnpack_operator);
581:
582:    // NHWC output
583:    qy = at::_empty_affine_quantized(
584:        {batch_size, outC, outH, outW},
585:        at::device(kCPU).dtype(kQUInt8),
586:        scale,
587:        zero_point,
588:        MemoryFormat::ChannelsLast);
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 590-614
```cpp
590:    const pytorch_qnnp_status setupStatus =
591:        pytorch_qnnp_setup_max_pooling2d_nhwc_u8(
592:            qnnpack_operator /* max pooling */,
593:            batch_size /* batch size */,
594:            inH /* input height */,
595:            inW /* input width */,
596:            (uint8_t*)input_contig.data_ptr<c10::quint8>() /* input */,
597:            inC /* input_pixel_stride */,
598:            (uint8_t*)qy.data_ptr<c10::quint8>() /* output data */,
599:            outC /* output_pixel_stride */,
600:            nullptr /* thread pool */);
601:    TORCH_INTERNAL_ASSERT(
602:        setupStatus == pytorch_qnnp_status_success,
603:        "failed to setup QNNPACK MaxPool operator");
604:
605:    pthreadpool_t threadpool = caffe2::pthreadpool_();
606:    const pytorch_qnnp_status runStatus =
607:        pytorch_qnnp_run_operator(qnnpack_operator, threadpool);
608:    TORCH_INTERNAL_ASSERT(
609:        runStatus == pytorch_qnnp_status_success,
610:        "failed to run QNNPACK MaxPool operator");
611:    return qy.contiguous(input.suggest_memory_format());
612:  }
613:  #endif
614: }  // namespace
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 616-634
```cpp
616: // at::native functions for the native_functions.yaml
617: Tensor quantized_max_pool2d(
618:     const Tensor& qx,
619:     IntArrayRef kernel_size,
620:     IntArrayRef stride,
621:     IntArrayRef padding,
622:     IntArrayRef dilation,
623:     bool ceil_mode) {
624:   check_maxpool2d_params(
625:       kernel_size,
626:       stride,
627:       padding,
628:       dilation);
629:   if (stride.empty()) {
630:     stride = kernel_size;
631:   }
632: #ifdef USE_PYTORCH_QNNPACK
633:   if (at::globalContext().qEngine() == at::QEngine::QNNPACK && qx.scalar_type() == kQUInt8 && !ceil_mode) {
634:     return qnnpack_maxpool2d(qx, kernel_size, stride, padding, dilation, ceil_mode);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `quantized_max_pool2d`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `quantized_max_pool2d`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 635-654
```cpp
635:   }
636: #endif
637:   Tensor qy;
638:   AT_DISPATCH_QINT_TYPES_AND(ScalarType::Byte, qx.scalar_type(), "max_pool2d", [&]() {
639:     qy = q_maxpool_2d<scalar_t>(
640:         qx,
641:         kernel_size[0],
642:         kernel_size[1],
643:         stride[0],
644:         stride[1],
645:         padding[0],
646:         padding[1],
647:         dilation[0],
648:         dilation[1],
649:         ceil_mode);
650:   });
651:   return qy;
652: }
653:
654: Tensor quantized_max_pool3d(
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 运行时分派会根据数据类型或量化标量类别选择专用实现。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 655-674
```cpp
655:     const Tensor& qx,
656:     IntArrayRef kernel_size,
657:     IntArrayRef stride,
658:     IntArrayRef padding,
659:     IntArrayRef dilation,
660:     bool ceil_mode) {
661:   check_maxpool3d_params(
662:       kernel_size,
663:       stride,
664:       padding,
665:       dilation);
666:   if (stride.empty()) {
667:     stride = kernel_size;
668:   }
669: #ifdef USE_PYTORCH_QNNPACK
670:   TORCH_CHECK(at::globalContext().qEngine() != at::QEngine::QNNPACK,
671:               "QNNPACK backend doesn't support of quantized_max_pool3d");
672: #endif
673:   Tensor qy;
674:   AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "max_pool3d", [&]() {
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 675-694
```cpp
675:     qy = q_maxpool_3d<scalar_t>(
676:         qx,
677:         kernel_size[0],
678:         kernel_size[1],
679:         kernel_size[2],
680:         stride[0],
681:         stride[1],
682:         stride[2],
683:         padding[0],
684:         padding[1],
685:         padding[2],
686:         dilation[0],
687:         dilation[1],
688:         dilation[2],
689:         ceil_mode);
690:   });
691:   return qy;
692: }
693:
694: // Quantized max_pool1d is a special case of the max_pool2d, with one of the
```
- EN: The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 695-720
```cpp
695: // dimensions and kernels removed.
696: Tensor quantized_max_pool1d(
697:     const Tensor& qx,
698:     IntArrayRef kernel_size,
699:     IntArrayRef stride,
700:     IntArrayRef padding,
701:     IntArrayRef dilation,
702:     bool ceil_mode) {
703:   check_max_pool1d(qx, kernel_size, stride, padding, dilation, ceil_mode);
704:   // (C, L) -> (C, 1, L) => kSqueezeDim = 1
705:   // (N, C, L) -> (N, C, 1, L) => kSqueezeDim = 2
706:   const int32_t kSqueezeDim = qx.dim() - 1;
707:   const auto qx_unsqueeze = qx.unsqueeze(kSqueezeDim);
708:   if (stride.empty()) {
709:     stride = kernel_size;
710:   }
711:   auto qy = at::quantized_max_pool2d(
712:     qx.unsqueeze(kSqueezeDim),
713:     {1, kernel_size[0]},
714:     {1, stride[0]},
715:     {0, padding[0]},
716:     {1, dilation[0]},
717:     ceil_mode);
718:   qy = qy.squeeze(kSqueezeDim);
719:   return qy;
720: }
```
- EN: The main symbol in this range is `quantized_max_pool1d`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `quantized_max_pool1d`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 722-747
```cpp
722: // Keep the registry in the anonymous namespace.
723: namespace {
724: template <uint32_t kSpatialDim>
725: class QMaxPool_arr_args final {
726:  public:
727:   static Tensor run(
728:       const Tensor& qx,
729:       std::vector<int64_t> kernel_size,
730:       std::vector<int64_t> stride,
731:       std::vector<int64_t> padding,
732:       std::vector<int64_t> dilation,
733:       bool ceil_mode) {
734:     if (!qx.is_quantized() && kSpatialDim == 2 && qx.scalar_type() == c10::ScalarType::Byte){
735:       return at::native::quantized_max_pool2d(qx, kernel_size, stride, padding,
736:                                       dilation, ceil_mode);
737:     }
738:     if (kSpatialDim == 1) {
739:       return at::quantized_max_pool1d(qx, kernel_size, stride, padding,
740:                                       dilation, ceil_mode);
741:     } else if (kSpatialDim == 2) {
742:       return at::quantized_max_pool2d(qx, kernel_size, stride, padding,
743:                                       dilation, ceil_mode);
744:     }
745:     TORCH_CHECK(false, "MaxPool", kSpatialDim, "D is not supported.");
746:   }
747: };
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `run`, `QMaxPool_arr_args`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `run`, `QMaxPool_arr_args`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 749-759
```cpp
749: TORCH_LIBRARY_IMPL(quantized, QuantizedCPU, m) {
750:   m.impl(TORCH_SELECTIVE_NAME("quantized::max_pool1d"), TORCH_FN(QMaxPool_arr_args<1>::run));
751:   m.impl(TORCH_SELECTIVE_NAME("quantized::max_pool2d"), TORCH_FN(QMaxPool_arr_args<2>::run));
752: }
753:
754: TORCH_LIBRARY_IMPL(quantized, CPU, m) {
755:   m.impl(TORCH_SELECTIVE_NAME("quantized::max_pool2d"), TORCH_FN(QMaxPool_arr_args<2>::run));
756: }
757:
758: } // namespace
759: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. Quantization-specific data handling or packed-parameter logic is active in this range. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段涉及量化数据处理或打包参数逻辑。 这段代码参与池化窗口遍历或池化输出形状计算。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- CPU parallelism / CPU 并行
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Pooling reductions / 池化归约
- Dispatcher registration / 调度器注册

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Context.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`, `ATen/native/Pool.h`, `ATen/native/MaxPooling.h`, `ATen/quantized/Quantizer.h`, `ATen/native/quantized/cpu/QuantizedOps.h`, `ATen/native/quantized/cpu/init_qnnpack.h`, `ATen/native/quantized/cpu/QnnpackUtils.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `torch/library.h`, `caffe2/utils/threadpool/pthreadpool-cpp.h`, `algorithm`, `vector`
- Key helper symbols / 关键辅助符号: `parallel_for`, `Scalar`, `ScalarType`, `qnnpack`
