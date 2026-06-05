# ScatterGatherKernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/ScatterGatherKernel.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU concatenation kernels and layout-aware copy logic in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了CPU 拼接 kernel 与面向布局的拷贝逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-38
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/native/NonEmptyUtils.h>
 3: #include <ATen/native/DispatchStub.h>
 4: #include <ATen/native/TensorIterator.h>
 5: #include <ATen/native/TensorAdvancedIndexing.h>
 6: #include <ATen/core/Tensor.h>
 7: #include <ATen/Config.h>
 8: #include <ATen/Dispatch.h>
 9: #include <ATen/NumericUtils.h>
10: #include <ATen/Parallel.h>
11: #include <ATen/native/cpu/ReduceUtils.h>
12: #include <ATen/cpu/vec/functional.h>
13: #include <ATen/cpu/vec/vec.h>
14: #include <c10/util/irange.h>
15: #ifdef USE_FBGEMM
16: #include <fbgemm/Utils.h>
17: #endif
18: #include <ATen/OpMathType.h>
19:
20: #ifndef AT_PER_OPERATOR_HEADERS
21: #include <ATen/Functions.h>
22: #include <ATen/NativeFunctions.h>
23: #else
24: #include <ATen/ops/empty.h>
25: #include <ATen/ops/zeros.h>
26: #endif
27: namespace at::native {
28:
29: namespace {
30:
31: // Implement as functors since lambdas don't get optimized.
32: class ReduceMultiply {
33: public:
34:   template <typename scalar_t>
35:   constexpr void operator() (at::opmath_type<scalar_t> * self_data, scalar_t * src_data) const {
36:     using opmath_t = at::opmath_type<scalar_t>;
37:     *self_data *= opmath_t(c10::load(src_data));
38:   }
```
- EN: This range pulls in required headers, including `ATen/native/NonEmptyUtils.h`, `ATen/native/DispatchStub.h`, `ATen/native/TensorIterator.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/native/NonEmptyUtils.h`, `ATen/native/DispatchStub.h`, `ATen/native/TensorIterator.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 40-76
```cpp
40:   constexpr void operator() (bool * self_data, bool * src_data) const {
41:     *self_data = c10::load(self_data) && c10::load(src_data);
42:   }
43: };
44: ReduceMultiply reduce_multiply;
45:
46: class ReduceAdd {
47: public:
48:   template <typename scalar_t>
49:   constexpr void operator() (at::opmath_type<scalar_t> * self_data, scalar_t * src_data) const {
50:     using opmath_t = at::opmath_type<scalar_t>;
51:     *self_data += opmath_t(c10::load(src_data));
52:   }
53: };
54: ReduceAdd reduce_add;
55:
56: class ReduceMean {
57: public:
58:   template <typename scalar_t>
59:   constexpr void operator() (at::opmath_type<scalar_t> * self_data, scalar_t * src_data) const {
60:     using opmath_t = at::opmath_type<scalar_t>;
61:     *self_data += opmath_t(c10::load(src_data));
62:   }
63: };
64: ReduceMean reduce_mean;
65:
66: class ReduceMaximum {
67: public:
68:   template <typename scalar_t>
69:   constexpr void operator() (at::opmath_type<scalar_t> * self_data, scalar_t * src_data) const {
70:     using opmath_t = at::opmath_type<scalar_t>;
71:     auto self_value = c10::load(self_data);
72:     auto src_value = c10::load(src_data);
73:     *self_data = at::_isnan<scalar_t>(src_value) ? opmath_t(src_value) : std::max(self_value, opmath_t(src_value));
74:   }
75: };
76: ReduceMaximum reduce_maximum;
```
- EN: The main symbol in this range is `ReduceAdd`, `ReduceMean`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `ReduceAdd`, `ReduceMean`，它们直接构成本文件的算子逻辑。

### Lines 78-110
```cpp
 78: class ReduceMinimum {
 79: public:
 80:   template <typename scalar_t>
 81:   constexpr void operator() (at::opmath_type<scalar_t> * self_data, scalar_t * src_data) const {
 82:     using opmath_t = at::opmath_type<scalar_t>;
 83:     auto self_value = c10::load(self_data);
 84:     auto src_value = c10::load(src_data);
 85:     *self_data = at::_isnan<scalar_t>(src_value) ? opmath_t(src_value) : std::min(self_value, opmath_t(src_value));
 86:   }
 87: };
 88: ReduceMinimum reduce_minimum;
 89:
 90: class TensorAssign {
 91: public:
 92:   template <typename scalar_t>
 93:   constexpr void operator() (at::opmath_type<scalar_t> * self_data, scalar_t * src_data) const {
 94:     using opmath_t = at::opmath_type<scalar_t>;
 95:     *self_data = opmath_t(c10::load(src_data));
 96:   }
 97: };
 98: TensorAssign tensor_assign;
 99:
100: template <bool is_scatter_like = true>
101: struct _cpu_scatter_gather_dim_loop {
102:   template <typename scalar_t, typename func_t>
103:   void operator()(
104:     at::opmath_type<scalar_t>* self_data, int64_t self_dim_stride,
105:     int64_t* index_data, int64_t index_dim_stride,
106:     scalar_t* src_data, int64_t src_dim_stride,
107:     int64_t dim, int64_t index_dim_size,
108:     int64_t index_upper_bound,
109:     func_t& f
110:   ) {
```
- EN: The main symbol in this range is `ReduceMinimum`, `TensorAssign`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `ReduceMinimum`, `TensorAssign`，它们直接构成本文件的算子逻辑。

### Lines 112-154
```cpp
112:     for (const auto i : c10::irange(index_dim_size)) {
113:       int64_t idx_dim = index_data[i * index_dim_stride];
114:       // we are not putting idx_dim in the error message because it disables
115:       // loop optimization in clang-7
116:       TORCH_CHECK(idx_dim >= 0 && idx_dim < index_upper_bound,
117:         "index ", index_data[i * index_dim_stride],
118:         " is out of bounds for dimension ", dim,
119:         " with size ", index_upper_bound
120:       );
121:
122:       f(
123:         self_data + (is_scatter_like ? idx_dim : i) * self_dim_stride,
124:         src_data + (is_scatter_like ? i : idx_dim) * src_dim_stride
125:       );
126:     }
127:   }
128:
129:   template <typename scalar_t, typename func_t>
130:   void operator()(
131:     at::opmath_type<scalar_t>* self_data, int64_t self_dim_stride,
132:     int64_t* index_data, int64_t index_dim_stride,
133:     Scalar value,
134:     int64_t dim, int64_t index_dim_size,
135:     int64_t index_upper_bound,
136:     func_t& f
137:   ) {
138:
139:     for (const auto i : c10::irange(index_dim_size)) {
140:       int64_t idx_dim = index_data[i * index_dim_stride];
141:       // we are not putting idx_dim in the error message because it disables
142:       // loop optimization in clang-7
143:       TORCH_CHECK(idx_dim >= 0 && idx_dim < index_upper_bound,
144:         "index ", index_data[i * index_dim_stride],
145:         " is out of bounds for dimension ", dim,
146:         " with size ", index_upper_bound
147:       );
148:       auto temp = value.to<scalar_t>();
149:       f(
150:         self_data + (is_scatter_like ? idx_dim : i) * self_dim_stride, &temp
151:       );
152:     }
153:   }
154: };
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 156-186
```cpp
156: inline void create_acc_buffer(Tensor& buffer, const Tensor& self, bool need_acc) {
157:   if (need_acc) {
158:     auto acc_type = at::toOpMathType(self.scalar_type());
159:     buffer = at::empty(self.sizes(), self.options().dtype(acc_type));
160:     buffer.copy_(self);
161:   } else {
162:     buffer = self;
163:   }
164: }
165:
166: template <bool is_scatter_like = true>
167: struct cpu_scatter_gather_base_kernel {
168:   template <typename func_t>
169:   void operator()(const Tensor& self, int64_t dim,
170:     const Tensor& _index, const Scalar& value,
171:     const std::string& method_name, func_t& kernel_func) {
172:
173:     Tensor buffer;
174:     Tensor index = _index.to(ScalarType::Long);
175:     bool need_acc = isReducedFloatingType(self.scalar_type());
176:     create_acc_buffer(buffer, self, need_acc);
177:
178:     auto index_sizes = ensure_nonempty_vec(index.sizes().vec());
179:     auto index_strides = ensure_nonempty_vec(index.strides().vec());
180:
181:     // `dim` is traversed in the kernel,
182:     // that is why index.stride(dim) = 0 and index.size(dim) = 1.
183:     // Also, index.size(dim) = 1 makes sure that TensorIterator.DimCounter
184:     // has the following form : (i_1,..., i_{dim-1}, 0, i_{dim+1},...,i_n).
185:     index_sizes[dim] = 1;
186:     index_strides[dim] = 0;
```
- EN: The main symbol in this range is `create_acc_buffer`, `cpu_scatter_gather_base_kernel`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `create_acc_buffer`, `cpu_scatter_gather_base_kernel`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 188-229
```cpp
188:     auto iter = TensorIteratorConfig()
189:       .check_all_same_dtype(false)
190:       .resize_outputs(false)
191:       .declare_static_shape(index.sizes(), /*squash_dims=*/dim)
192:       .add_output(buffer)
193:       .add_const_input(index)
194:       .build();
195:
196:     auto self_dim_stride = ensure_nonempty_stride(buffer, dim);
197:     auto self_dim_size = ensure_nonempty_size(buffer, dim);
198:
199:     auto index_dim_stride = ensure_nonempty_stride(index, dim);
200:     auto index_dim_size = ensure_nonempty_size(index, dim);
201:
202:     auto index_upper_bound = self_dim_size;
203:
204:     // since the index dimension is squashed, need to alter the grain size according
205:     // to keep equal granularity in parallelism.
206:     int64_t grain_size = std::max((int64_t) 1, at::internal::GRAIN_SIZE / index_dim_size);
207:
208:     AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3(
209:       ScalarType::Bool, ScalarType::Half, ScalarType::BFloat16, self.scalar_type(),
210:       "scatter_gather_scalar_cpu", [&] {
211:         constexpr auto SELF_ITER_STRIDE_IDX = 0;
212:         constexpr auto INDEX_ITER_STRIDE_IDX = 1;
213:         using opmath_t = at::opmath_type<scalar_t>;
214:         _cpu_scatter_gather_dim_loop<is_scatter_like> loop_func;
215:         auto loop = [&](char** data, const int64_t* strides, int64_t n) {
216:           auto* self_data_bytes = data[SELF_ITER_STRIDE_IDX];
217:           auto* index_data_bytes = data[INDEX_ITER_STRIDE_IDX];
218:           // we change the order of TensorIterator-dim loop
219:           // vs dim-TensorIterator loop order depending on
220:           // whether dim is the last dimension
221:           if (dim== buffer.dim() - 1) {
222:             for ([[maybe_unused]] const auto nelem : c10::irange(n)) {
223:               // dim loop is a separate code block
224:               // for better performance
225:               loop_func.template operator()<scalar_t, func_t>(
226:                 (opmath_t*)self_data_bytes, self_dim_stride,
227:                 (int64_t*)index_data_bytes, index_dim_stride,
228:                 value, dim, index_dim_size, index_upper_bound,
229:                 kernel_func);
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 231-263
```cpp
231:               self_data_bytes += strides[SELF_ITER_STRIDE_IDX];
232:               index_data_bytes += strides[INDEX_ITER_STRIDE_IDX];
233:             }
234:           }
235:           else {
236:             for (const auto i : c10::irange(index_dim_size)) {
237:               auto* self_data = self_data_bytes;
238:               auto* index_data = (char*)((int64_t*)index_data_bytes + i * index_dim_stride);
239:               for ([[maybe_unused]] const auto nelem : c10::irange(n)) {
240:                 int64_t idx_dim = *(int64_t*)index_data;
241:                 // we are not putting idx_dim in the error message because it disables
242:                 // loop optimization in clang-7
243:                 TORCH_CHECK(idx_dim >= 0 && idx_dim < index_upper_bound,
244:                             "index ", *(int64_t*)index_data,
245:                             " is out of bounds for dimension ", dim,
246:                             " with size ", index_upper_bound);
247:
248:                 auto temp = value.to<scalar_t>();
249:                 kernel_func((opmath_t*)self_data + (is_scatter_like ? idx_dim : i) * self_dim_stride, &temp);
250:
251:                 self_data += strides[SELF_ITER_STRIDE_IDX];
252:                 index_data += strides[INDEX_ITER_STRIDE_IDX];
253:               }
254:             }
255:           }
256:         };
257:         iter.for_each(loop, grain_size);
258:       }
259:     );
260:     if (need_acc) {
261:       self.copy_(buffer);
262:     }
263:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 265-293
```cpp
265:   template <typename func_t>
266:   void operator()(const Tensor& self, int64_t dim,
267:     const Tensor& _index, const Tensor& src,
268:     const std::string& method_name, func_t& kernel_func) {
269:
270:     Tensor buffer;
271:     Tensor index = _index.to(ScalarType::Long);
272:     bool need_acc = isReducedFloatingType(self.scalar_type());
273:     create_acc_buffer(buffer, self, need_acc);
274:
275:     auto iter = TensorIteratorConfig()
276:       .check_all_same_dtype(false)
277:       .resize_outputs(false)
278:       .declare_static_shape(index.sizes(), /*squash_dims=*/dim)
279:       .add_output(buffer)
280:       .add_const_input(src)
281:       .add_const_input(index)
282:       .build();
283:
284:     auto self_dim_stride = ensure_nonempty_stride(buffer, dim);
285:     auto self_dim_size = ensure_nonempty_size(buffer, dim);
286:
287:     auto index_dim_stride = ensure_nonempty_stride(index, dim);
288:     auto index_dim_size = ensure_nonempty_size(index, dim);
289:
290:     auto src_dim_stride = ensure_nonempty_stride(src, dim);
291:     auto src_dim_size = ensure_nonempty_size(src, dim);
292:
293:     auto index_upper_bound = is_scatter_like ? self_dim_size : src_dim_size;
```
- EN: TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 295-322
```cpp
295:     int64_t grain_size = std::max((int64_t) 1, at::internal::GRAIN_SIZE / index_dim_size);
296:
297:     AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3(
298:       ScalarType::Bool, ScalarType::Half, ScalarType::BFloat16, iter.dtype(1),
299:       "scatter_gather_tensor_cpu", [&] {
300:         constexpr auto SELF_ITER_STRIDE_IDX = 0;
301:         constexpr auto INDEX_ITER_STRIDE_IDX = 2;
302:         constexpr auto SRC_ITER_STRIDE_IDX = 1;
303:         using opmath_t = at::opmath_type<scalar_t>;
304:         _cpu_scatter_gather_dim_loop<is_scatter_like> loop_func;
305:         auto loop = [&](char** data, const int64_t* strides, int64_t n) {
306:           auto* self_data_bytes = data[SELF_ITER_STRIDE_IDX];
307:           auto* index_data_bytes = data[INDEX_ITER_STRIDE_IDX];
308:           auto* src_data_bytes = data[SRC_ITER_STRIDE_IDX];
309:           // we change the order of TensorIterator-dim loop
310:           // vs dim-TensorIterator loop order depending on
311:           // whether dim is the last dimension
312:           if (dim== buffer.dim() - 1) {
313:             for ([[maybe_unused]] const auto nelem : c10::irange(n)) {
314:               // dim loop is a separate code block
315:               // for better performance
316:               loop_func.template operator()<scalar_t, func_t>(
317:                  (opmath_t*)self_data_bytes, self_dim_stride,
318:                  (int64_t*)index_data_bytes, index_dim_stride,
319:                  (scalar_t*)src_data_bytes, src_dim_stride,
320:                  dim, index_dim_size, index_upper_bound,
321:                  kernel_func
322:                );
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 324-360
```cpp
324:               self_data_bytes += strides[SELF_ITER_STRIDE_IDX];
325:               index_data_bytes += strides[INDEX_ITER_STRIDE_IDX];
326:               src_data_bytes += strides[SRC_ITER_STRIDE_IDX];
327:             }
328:           }
329:           else {
330:             for (const auto i : c10::irange(index_dim_size)) {
331:               auto* self_data = self_data_bytes;
332:               auto* index_data = (char*)((int64_t*)index_data_bytes + i * index_dim_stride);
333:               auto* src_data = src_data_bytes;
334:               for ([[maybe_unused]] const auto nelem : c10::irange(n)) {
335:                 int64_t idx_dim = *(int64_t*)index_data;
336:                 // we are not putting idx_dim in the error message because it disables
337:                 // loop optimization in clang-7
338:                 TORCH_CHECK(idx_dim >= 0 && idx_dim < index_upper_bound,
339:                             "index ", *(int64_t*)index_data,
340:                             " is out of bounds for dimension ", dim,
341:                             " with size ", index_upper_bound);
342:
343:                 kernel_func(
344:                   (opmath_t*)self_data + (is_scatter_like ? idx_dim : i) * self_dim_stride,
345:                   (scalar_t*)src_data + (is_scatter_like ? i : idx_dim) * src_dim_stride);
346:
347:                 self_data += strides[SELF_ITER_STRIDE_IDX];
348:                 index_data += strides[INDEX_ITER_STRIDE_IDX];
349:                 src_data += strides[SRC_ITER_STRIDE_IDX];
350:               }
351:             }
352:           }
353:         };
354:         iter.for_each(loop, grain_size);
355:       }
356:     );
357:     if (need_acc) {
358:       self.copy_(buffer);
359:     }
360:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 362-391
```cpp
362:   void operator()(const Tensor& self, int64_t dim,
363:     const Tensor& _index, const Tensor& src,
364:     const std::string& method_name, ReduceMean& kernel_func) {
365:
366:     Tensor buffer;
367:     Tensor index = _index.to(ScalarType::Long);
368:     bool need_acc = isReducedFloatingType(self.scalar_type());
369:     create_acc_buffer(buffer, self, need_acc);
370:
371:     auto iter = TensorIteratorConfig()
372:       .check_all_same_dtype(false)
373:       .resize_outputs(false)
374:       .declare_static_shape(index.sizes(), /*squash_dims=*/dim)
375:       .add_output(buffer)
376:       .add_const_input(src)
377:       .add_const_input(index)
378:       .build();
379:
380:     auto self_dim_stride = ensure_nonempty_stride(buffer, dim);
381:     auto self_dim_size = ensure_nonempty_size(buffer, dim);
382:
383:     auto index_dim_stride = ensure_nonempty_stride(index, dim);
384:     auto index_dim_size = ensure_nonempty_size(index, dim);
385:
386:     auto src_dim_stride = ensure_nonempty_stride(src, dim);
387:     auto src_dim_size = ensure_nonempty_size(src, dim);
388:
389:     auto index_upper_bound = is_scatter_like ? self_dim_size : src_dim_size;
390:
391:     int64_t grain_size = std::max((int64_t) 1, at::internal::GRAIN_SIZE / index_dim_size);
```
- EN: TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 393-421
```cpp
393:     AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND2(
394:       ScalarType::Half, ScalarType::BFloat16, iter.dtype(1),
395:       "scatter_gather_tensor_cpu_reduce_mean", [&] {
396:         constexpr auto SELF_ITER_STRIDE_IDX = 0;
397:         constexpr auto INDEX_ITER_STRIDE_IDX = 2;
398:         constexpr auto SRC_ITER_STRIDE_IDX = 1;
399:         using opmath_t = at::opmath_type<scalar_t>;
400:         _cpu_scatter_gather_dim_loop<is_scatter_like> loop_func;
401:         auto loop = [&](char** data, const int64_t* strides, int64_t n) {
402:           auto* self_data_bytes = data[SELF_ITER_STRIDE_IDX];
403:           auto* index_data_bytes = data[INDEX_ITER_STRIDE_IDX];
404:           auto* src_data_bytes = data[SRC_ITER_STRIDE_IDX];
405:           // we change the order of TensorIterator-dim loop
406:           // vs dim-TensorIterator loop order depending on
407:           // whether dim is the last dimension
408:           if (dim== buffer.dim() - 1) {
409:             for ([[maybe_unused]] const auto nelem : c10::irange(n)) {
410:               // dim loop is a separate code block
411:               // for better performance
412:               loop_func.template operator()<scalar_t, ReduceMean>(
413:                  (opmath_t*)self_data_bytes, self_dim_stride,
414:                  (int64_t*)index_data_bytes, index_dim_stride,
415:                  (scalar_t*)src_data_bytes, src_dim_stride,
416:                  dim, index_dim_size, index_upper_bound,
417:                  kernel_func
418:                );
419:
420:               self_data_bytes += strides[SELF_ITER_STRIDE_IDX];
421:               index_data_bytes += strides[INDEX_ITER_STRIDE_IDX];
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 422-456
```cpp
422:               src_data_bytes += strides[SRC_ITER_STRIDE_IDX];
423:             }
424:           }
425:           else {
426:             for (const auto i : c10::irange(index_dim_size)) {
427:               auto* self_data = self_data_bytes;
428:               auto* index_data = (char*)((int64_t*)index_data_bytes + i * index_dim_stride);
429:               auto* src_data = src_data_bytes;
430:               for ([[maybe_unused]] const auto nelem : c10::irange(n)) {
431:                 int64_t idx_dim = *(int64_t*)index_data;
432:                 // we are not putting idx_dim in the error message because it disables
433:                 // loop optimization in clang-7
434:                 TORCH_CHECK(idx_dim >= 0 && idx_dim < index_upper_bound,
435:                             "index ", *(int64_t*)index_data,
436:                             " is out of bounds for dimension ", dim,
437:                             " with size ", index_upper_bound);
438:
439:                 kernel_func(
440:                   (opmath_t*)self_data + (is_scatter_like ? idx_dim : i) * self_dim_stride,
441:                   (scalar_t*)src_data + (is_scatter_like ? i : idx_dim) * src_dim_stride);
442:
443:                 self_data += strides[SELF_ITER_STRIDE_IDX];
444:                 index_data += strides[INDEX_ITER_STRIDE_IDX];
445:                 src_data += strides[SRC_ITER_STRIDE_IDX];
446:               }
447:             }
448:           }
449:         };
450:         iter.for_each(loop, grain_size);
451:       }
452:     );
453:     if (need_acc) {
454:       self.copy_(buffer);
455:     }
456:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 458-486
```cpp
458:   void operator()(const Tensor& self, int64_t dim,
459:     const Tensor& _index, const Tensor& src,
460:     const std::string& method_name, ReduceMaximum& kernel_func) {
461:     Tensor buffer;
462:     Tensor index = _index.to(ScalarType::Long);
463:     bool need_acc = isReducedFloatingType(self.scalar_type());
464:     create_acc_buffer(buffer, self, need_acc);
465:
466:     auto iter = TensorIteratorConfig()
467:       .check_all_same_dtype(false)
468:       .resize_outputs(false)
469:       .declare_static_shape(index.sizes(), /*squash_dims=*/dim)
470:       .add_output(buffer)
471:       .add_const_input(src)
472:       .add_const_input(index)
473:       .build();
474:
475:     auto self_dim_stride = ensure_nonempty_stride(buffer, dim);
476:     auto self_dim_size = ensure_nonempty_size(buffer, dim);
477:
478:     auto index_dim_stride = ensure_nonempty_stride(index, dim);
479:     auto index_dim_size = ensure_nonempty_size(index, dim);
480:
481:     auto src_dim_stride = ensure_nonempty_stride(src, dim);
482:     auto src_dim_size = ensure_nonempty_size(src, dim);
483:
484:     auto index_upper_bound = is_scatter_like ? self_dim_size : src_dim_size;
485:
486:     int64_t grain_size = std::max((int64_t) 1, at::internal::GRAIN_SIZE / index_dim_size);
```
- EN: TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 488-516
```cpp
488:     AT_DISPATCH_ALL_TYPES_AND3(
489:       ScalarType::Bool, ScalarType::Half, ScalarType::BFloat16, iter.dtype(1),
490:       "scatter_gather_tensor_cpu_reduce_amax", [&] {
491:         constexpr auto SELF_ITER_STRIDE_IDX = 0;
492:         constexpr auto INDEX_ITER_STRIDE_IDX = 2;
493:         constexpr auto SRC_ITER_STRIDE_IDX = 1;
494:         using opmath_t = at::opmath_type<scalar_t>;
495:         _cpu_scatter_gather_dim_loop<is_scatter_like> loop_func;
496:         auto loop = [&](char** data, const int64_t* strides, int64_t n) {
497:           auto* self_data_bytes = data[SELF_ITER_STRIDE_IDX];
498:           auto* index_data_bytes = data[INDEX_ITER_STRIDE_IDX];
499:           auto* src_data_bytes = data[SRC_ITER_STRIDE_IDX];
500:           // we change the order of TensorIterator-dim loop
501:           // vs dim-TensorIterator loop order depending on
502:           // whether dim is the last dimension
503:           if (dim== buffer.dim() - 1) {
504:             for ([[maybe_unused]] const auto nelem : c10::irange(n)) {
505:               // dim loop is a separate code block
506:               // for better performance
507:               loop_func.template operator()<scalar_t, ReduceMaximum>(
508:                  (opmath_t*)self_data_bytes, self_dim_stride,
509:                  (int64_t*)index_data_bytes, index_dim_stride,
510:                  (scalar_t*)src_data_bytes, src_dim_stride,
511:                  dim, index_dim_size, index_upper_bound,
512:                  kernel_func
513:                );
514:
515:               self_data_bytes += strides[SELF_ITER_STRIDE_IDX];
516:               index_data_bytes += strides[INDEX_ITER_STRIDE_IDX];
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 517-551
```cpp
517:               src_data_bytes += strides[SRC_ITER_STRIDE_IDX];
518:             }
519:           }
520:           else {
521:             for (const auto i : c10::irange(index_dim_size)) {
522:               auto* self_data = self_data_bytes;
523:               auto* index_data = (char*)((int64_t*)index_data_bytes + i * index_dim_stride);
524:               auto* src_data = src_data_bytes;
525:               for ([[maybe_unused]] const auto nelem : c10::irange(n)) {
526:                 int64_t idx_dim = *(int64_t*)index_data;
527:                 // we are not putting idx_dim in the error message because it disables
528:                 // loop optimization in clang-7
529:                 TORCH_CHECK(idx_dim >= 0 && idx_dim < index_upper_bound,
530:                             "index ", *(int64_t*)index_data,
531:                             " is out of bounds for dimension ", dim,
532:                             " with size ", index_upper_bound);
533:
534:                 kernel_func(
535:                   (opmath_t*)self_data + (is_scatter_like ? idx_dim : i) * self_dim_stride,
536:                   (scalar_t*)src_data + (is_scatter_like ? i : idx_dim) * src_dim_stride);
537:
538:                 self_data += strides[SELF_ITER_STRIDE_IDX];
539:                 index_data += strides[INDEX_ITER_STRIDE_IDX];
540:                 src_data += strides[SRC_ITER_STRIDE_IDX];
541:               }
542:             }
543:           }
544:         };
545:         iter.for_each(loop, grain_size);
546:       }
547:     );
548:     if (need_acc) {
549:       self.copy_(buffer);
550:     }
551:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 553-582
```cpp
553:   void operator()(const Tensor& self, int64_t dim,
554:     const Tensor& _index, const Tensor& src,
555:     const std::string& method_name, ReduceMinimum& kernel_func) {
556:
557:     Tensor buffer;
558:     Tensor index = _index.to(ScalarType::Long);
559:     bool need_acc = isReducedFloatingType(self.scalar_type());
560:     create_acc_buffer(buffer, self, need_acc);
561:
562:     auto iter = TensorIteratorConfig()
563:       .check_all_same_dtype(false)
564:       .resize_outputs(false)
565:       .declare_static_shape(index.sizes(), /*squash_dims=*/dim)
566:       .add_output(buffer)
567:       .add_const_input(src)
568:       .add_const_input(index)
569:       .build();
570:
571:     auto self_dim_stride = ensure_nonempty_stride(buffer, dim);
572:     auto self_dim_size = ensure_nonempty_size(buffer, dim);
573:
574:     auto index_dim_stride = ensure_nonempty_stride(index, dim);
575:     auto index_dim_size = ensure_nonempty_size(index, dim);
576:
577:     auto src_dim_stride = ensure_nonempty_stride(src, dim);
578:     auto src_dim_size = ensure_nonempty_size(src, dim);
579:
580:     auto index_upper_bound = is_scatter_like ? self_dim_size : src_dim_size;
581:
582:     int64_t grain_size = std::max((int64_t) 1, at::internal::GRAIN_SIZE / index_dim_size);
```
- EN: TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 584-612
```cpp
584:     AT_DISPATCH_ALL_TYPES_AND3(
585:       ScalarType::Bool, ScalarType::Half, ScalarType::BFloat16, iter.dtype(1),
586:       "scatter_gather_tensor_cpu_reduce_amin", [&] {
587:         constexpr auto SELF_ITER_STRIDE_IDX = 0;
588:         constexpr auto INDEX_ITER_STRIDE_IDX = 2;
589:         constexpr auto SRC_ITER_STRIDE_IDX = 1;
590:         using opmath_t = at::opmath_type<scalar_t>;
591:         _cpu_scatter_gather_dim_loop<is_scatter_like> loop_func;
592:         auto loop = [&](char** data, const int64_t* strides, int64_t n) {
593:           auto* self_data_bytes = data[SELF_ITER_STRIDE_IDX];
594:           auto* index_data_bytes = data[INDEX_ITER_STRIDE_IDX];
595:           auto* src_data_bytes = data[SRC_ITER_STRIDE_IDX];
596:           // we change the order of TensorIterator-dim loop
597:           // vs dim-TensorIterator loop order depending on
598:           // whether dim is the last dimension
599:           if (dim== buffer.dim() - 1) {
600:             for ([[maybe_unused]] const auto nelem : c10::irange(n)) {
601:               // dim loop is a separate code block
602:               // for better performance
603:               loop_func.template operator()<scalar_t, ReduceMinimum>(
604:                  (opmath_t*)self_data_bytes, self_dim_stride,
605:                  (int64_t*)index_data_bytes, index_dim_stride,
606:                  (scalar_t*)src_data_bytes, src_dim_stride,
607:                  dim, index_dim_size, index_upper_bound,
608:                  kernel_func
609:                );
610:
611:               self_data_bytes += strides[SELF_ITER_STRIDE_IDX];
612:               index_data_bytes += strides[INDEX_ITER_STRIDE_IDX];
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 613-648
```cpp
613:               src_data_bytes += strides[SRC_ITER_STRIDE_IDX];
614:             }
615:           }
616:           else {
617:             for (const auto i : c10::irange(index_dim_size)) {
618:               auto* self_data = self_data_bytes;
619:               auto* index_data = (char*)((int64_t*)index_data_bytes + i * index_dim_stride);
620:               auto* src_data = src_data_bytes;
621:               for ([[maybe_unused]] const auto nelem : c10::irange(n)) {
622:                 int64_t idx_dim = *(int64_t*)index_data;
623:                 // we are not putting idx_dim in the error message because it disables
624:                 // loop optimization in clang-7
625:                 TORCH_CHECK(idx_dim >= 0 && idx_dim < index_upper_bound,
626:                             "index ", *(int64_t*)index_data,
627:                             " is out of bounds for dimension ", dim,
628:                             " with size ", index_upper_bound);
629:
630:                 kernel_func(
631:                   (opmath_t*)self_data + (is_scatter_like ? idx_dim : i) * self_dim_stride,
632:                   (scalar_t*)src_data + (is_scatter_like ? i : idx_dim) * src_dim_stride);
633:
634:                 self_data += strides[SELF_ITER_STRIDE_IDX];
635:                 index_data += strides[INDEX_ITER_STRIDE_IDX];
636:                 src_data += strides[SRC_ITER_STRIDE_IDX];
637:               }
638:             }
639:           }
640:         };
641:         iter.for_each(loop, grain_size);
642:       }
643:     );
644:     if (need_acc) {
645:       self.copy_(buffer);
646:     }
647:   }
648: };
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 650-690
```cpp
650: #ifndef USE_FBGEMM
651: namespace fbgemm {
652:
653: template <typename K, typename V>
654: std::pair<K*, V*> radix_sort_parallel(
655:     K* const inp_key_buf,
656:     V* const inp_value_buf,
657:     K* const tmp_key_buf,
658:     V* const tmp_value_buf,
659:     const int64_t elements_count,
660:     const int64_t max_value) {
661:   TORCH_INTERNAL_ASSERT(false, "radix_sort_parallel: ATen not compiled with FBGEMM support");
662:   return std::make_pair(nullptr, nullptr);
663: }
664:
665: }
666: #endif
667:
668: // Note [scatter reduce optimization]
669: // ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
670: //
671: // 1. initiative: optimize `scatter_reduce` on classic PyG use-case:
672: //   `scatter_reduce` is extensively used on 'message passing' when
673: //   aggregating info.
674: //
675: //   Typically, `self` will 2D tensor and `index` is a 1D extended/broadcasted
676: //   tensor, which means that the aggregation is on rowwise and we can vectorize
677: //   on the inner dimensions.
678: //
679: // 2. implementation: map `scatter_reduce` to `spmm` reduce
680: //   in the shape of `[M, N]` * `[N, K]`, where:
681: //
682: //   M: self_dim_size
683: //   nnz: index_dim_size
684: //   K: index.numel() / index_dim_size;
685: //
686: //   step 1: convert input index to CSR format (use radix_sort to
687: //     solve write addr conflicts on `self` tensor)
688: //
689: //   step 2: spmm reduce, parallel on M and vectorize on K
690: //
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `radix_sort_parallel`, which contributes directly to this file's operator logic.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `radix_sort_parallel`，它们直接构成本文件的算子逻辑。

### Lines 692-728
```cpp
692: template <typename scalar_t, ReductionType reduce>
693: void cpu_scatter_reduce_expanded_index(const Tensor& self, const Tensor& index, const Tensor& src, bool include_self) {
694:   const int64_t* index_data = index.const_data_ptr<int64_t>();
695:   scalar_t* self_data = self.data_ptr<scalar_t>();
696:   const scalar_t* src_data = src.const_data_ptr<scalar_t>();
697:
698:   const int64_t M = ensure_nonempty_size(self, 0);
699:   const int64_t nnz = ensure_nonempty_size(index, 0);
700:   const int64_t K = index.numel() / nnz;
701:
702:   const int64_t index_upper_bound = M;
703:
704:   auto keys = std::make_unique<int64_t[]>(nnz);
705:   auto values = std::make_unique<int64_t[]>(nnz);
706:   auto keys_tmp = std::make_unique<int64_t[]>(nnz);
707:   auto values_tmp = std::make_unique<int64_t[]>(nnz);
708:   at::parallel_for(0, nnz, 1, [&](int64_t begin, int64_t end) {
709:     for (const auto i : c10::irange(begin, end)) {
710:       int64_t index = index_data[i];
711:       TORCH_CHECK(index >= 0 && index < index_upper_bound,
712:                   "index ", index,
713:                   " is out of bounds for dimension ", 0,
714:                   " with size ", index_upper_bound);
715:       keys[i] = index;
716:       values[i] = i;
717:     }
718:   });
719:
720:   int64_t* sorted_col_index_keys = nullptr;
721:   int64_t* sorted_col_index_values = nullptr;
722:   std::tie(sorted_col_index_keys, sorted_col_index_values) = fbgemm::radix_sort_parallel(
723:       keys.get(),
724:       values.get(),
725:       keys_tmp.get(),
726:       values_tmp.get(),
727:       nnz,
728:       M);
```
- EN: The main symbol in this range is `cpu_scatter_reduce_expanded_index`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `cpu_scatter_reduce_expanded_index`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 730-767
```cpp
730:   int num_threads = at::get_num_threads();
731:   std::vector<int64_t> num_uniq(num_threads, 0);
732:   at::parallel_for(1, nnz, 1, [&](int64_t begin, int64_t end) {
733:     int tid = at::get_thread_num();
734:     for(const auto i : c10::irange(begin, end)) {
735:       if (sorted_col_index_keys[i] != sorted_col_index_keys[i - 1]) {
736:         num_uniq[tid]++;
737:       }
738:     }
739:   });
740:   num_uniq[0]++;
741:   for (const auto n : c10::irange(1, num_threads)) {
742:     num_uniq[n] += num_uniq[n - 1];
743:   }
744:
745:   // in case some rows are not written into, num_nonzero_rows will be smaller than M
746:   int64_t num_nonzero_rows = num_uniq[num_threads - 1];
747:   auto row_index_tmp = std::make_unique<int64_t[]>(num_nonzero_rows);
748:   auto row_index_offset_tmp = std::make_unique<int64_t[]>(num_nonzero_rows + 1);
749:   int64_t* row_index = row_index_tmp.get();
750:   int64_t* row_index_offset = row_index_offset_tmp.get();
751:   row_index[0] = sorted_col_index_keys[0];
752:   row_index_offset[0] = 0;
753:   row_index_offset[num_nonzero_rows] = nnz;
754:
755:   at::parallel_for(1, nnz, 1, [&](int64_t begin, int64_t end) {
756:     int tid = at::get_thread_num();
757:     int64_t* t_index = row_index + ((tid == 0) ? 1 : num_uniq[tid - 1]);
758:     int64_t* t_index_offset = row_index_offset + ((tid == 0) ? 1 : num_uniq[tid - 1]);
759:     for (const auto i : c10::irange(begin, end)) {
760:       if (sorted_col_index_keys[i] != sorted_col_index_keys[i - 1]) {
761:         *t_index = sorted_col_index_keys[i];
762:         *t_index_offset = i;
763:         t_index++;
764:         t_index_offset++;
765:       }
766:     }
767:   });
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 769-798
```cpp
769:   using opmath_t = at::opmath_type<scalar_t>;
770:   Tensor buffer;
771:   opmath_t* buffer_data = nullptr;
772:   static constexpr bool need_acc = is_reduced_floating_point_v<scalar_t>;
773:   if constexpr (need_acc) {
774:     auto acc_type = at::toAccumulateType(self.scalar_type(), /*is_cuda=*/true);
775:     buffer = at::zeros({num_threads, K}, self.options().dtype(acc_type));
776:     buffer_data = buffer.data_ptr<opmath_t>();
777:   }
778:
779:   // TODO: do blocking on col dimension to reduce WR bandwidth
780:   at::parallel_for(0, num_nonzero_rows, 1, [&](int64_t begin, int64_t end) {
781:     int tid = at::get_thread_num();
782:     TORCH_CHECK(tid < num_threads,
783:                 "expect thread id smaller than ", num_threads, ", got thread id ", tid);
784:     opmath_t* buffer_ptr = nullptr;
785:
786:     for (const auto m : c10::irange(begin, end)) {
787:       int64_t row = row_index[m];
788:       int64_t off_start = row_index_offset[m];
789:       int64_t off_end = row_index_offset[m + 1];
790:       scalar_t* self_ptr = self_data + row * K;
791:       if constexpr (need_acc) {
792:         buffer_ptr = buffer_data + tid * K;
793:       } else {
794:         buffer_ptr = reinterpret_cast<opmath_t*>(self_ptr);
795:       }
796:
797:       // step 1: reinit rows in `self` if needed
798:       _init<scalar_t, reduce>(self_ptr, buffer_ptr, K, include_self);
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 800-828
```cpp
800:       // step 2: reduce
801:       for (const auto n : c10::irange(off_start, off_end)) {
802:         int64_t col = sorted_col_index_values[n];
803:         update<scalar_t, reduce>(buffer_ptr, src_data + col * K, K);
804:       }
805:       if constexpr (need_acc) {
806:         vec::convert(buffer_ptr, self_ptr, K);
807:       }
808:
809:       // step 3: finalize
810:       int64_t count = include_self ? 1 : 0;
811:       count += off_end - off_start;
812:       write<scalar_t, reduce>(self_ptr, count, K);
813:     }
814:   });
815: }
816:
817: template <typename scalar_t>
818: void cpu_gather_expanded_index_kernel(const Tensor& result, const Tensor& _index, const Tensor& self) {
819:   Tensor index = _index.to(ScalarType::Long);
820:   const int64_t* index_data = index.const_data_ptr<int64_t>();
821:   scalar_t* result_data = result.data_ptr<scalar_t>();
822:   const scalar_t* self_data = self.const_data_ptr<scalar_t>();
823:
824:   const int64_t M = ensure_nonempty_size(result, 0);
825:   const int64_t N = ensure_nonempty_size(self, 0);
826:   const int64_t K = index.numel() / M;
827:
828:   const int64_t index_upper_bound = N;
```
- EN: The main symbol in this range is `constexpr`, `cpu_gather_expanded_index_kernel`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `constexpr`, `cpu_gather_expanded_index_kernel`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 830-861
```cpp
830:   using Vec = vec::Vectorized<scalar_t>;
831:   int64_t grain_size = std::max((int64_t) 1, at::internal::GRAIN_SIZE / K);
832:   at::parallel_for(0, M, grain_size, [&](int64_t begin, int64_t end) {
833:     for (const auto m : c10::irange(begin, end)) {
834:       scalar_t* result_ptr = result_data + m * K;
835:       int64_t index = index_data[m];
836:       TORCH_CHECK(index >= 0 && index < index_upper_bound,
837:                   "index ", index,
838:                   " is out of bounds for dimension ", 0,
839:                   " with size ", index_upper_bound);
840:       const scalar_t* self_ptr = self_data + index * K;
841:       int64_t d = 0;
842:       for (; d < K - (K % Vec::size()); d += Vec::size()) {
843:         Vec out_vec = Vec::loadu(self_ptr + d);
844:         out_vec.store(result_ptr + d);
845:       }
846:       #if !defined(_MSC_VER) && !defined(COMPILING_FOR_MIN_SIZE)
847:       # pragma unroll
848:       #endif
849:       for (; d < K; d++) {
850:         result_ptr[d] = self_ptr[d];
851:       }
852:     }
853:   });
854: }
855:
856: void scatter_add_expanded_index_kernel(const Tensor& self, const Tensor& index, const Tensor& src) {
857:   AT_DISPATCH_FLOATING_TYPES_AND2(
858:     ScalarType::BFloat16, ScalarType::Half, self.scalar_type(), "scatter_add_expanded_index", [&] {
859:       cpu_scatter_reduce_expanded_index<scalar_t, ReductionType::SUM>(self, index, src, /*include_self*/true);
860:   });
861: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `scatter_add_expanded_index_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `scatter_add_expanded_index_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 863-895
```cpp
863: void scatter_reduce_expanded_index_kernel(
864:     const Tensor& self, const Tensor& index, const Tensor& src,
865:     const ReductionType& reduction, bool include_self) {
866:   AT_DISPATCH_FLOATING_TYPES_AND2(
867:     ScalarType::BFloat16, ScalarType::Half, self.scalar_type(), "scatter_reduce_expanded_index", [&] {
868:     AT_DISPATCH_REDUCTION_TYPES(reduction, [&]() {
869:       cpu_scatter_reduce_expanded_index<scalar_t, reduce>(self, index, src, include_self);
870:     });
871:   });
872: }
873:
874: void gather_expanded_index_kernel(const Tensor& result, const Tensor& self, const Tensor& index) {
875:   AT_DISPATCH_FLOATING_TYPES_AND2(
876:     ScalarType::BFloat16, ScalarType::Half, self.scalar_type(), "gather_expanded_index", [&] {
877:       cpu_gather_expanded_index_kernel<scalar_t>(result, index, self);
878:   });
879: }
880:
881: void gather_cpu_kernel(const Tensor& result, const Tensor& self, int64_t dim, const Tensor& index) {
882:   cpu_scatter_gather_base_kernel</*is_scatter_like=*/false>()(
883:     result, dim, index, self,
884:     "gather_out_cpu", tensor_assign);
885: }
886:
887: void scatter_cpu_kernel(const Tensor& self, int64_t dim, const Tensor& index, const Tensor& src) {
888:   cpu_scatter_gather_base_kernel<>()(
889:     self, dim, index, src, "scatter_cpu_", tensor_assign);
890: }
891:
892: void scatter_fill_cpu_kernel(const Tensor& self, int64_t dim, const Tensor& index, const Scalar& value) {
893:   cpu_scatter_gather_base_kernel<>()(
894:     self, dim, index, value, "scatter_fill_cpu_", tensor_assign);
895: }
```
- EN: The main symbol in this range is `scatter_reduce_expanded_index_kernel`, `gather_expanded_index_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 这一段的主要符号是 `scatter_reduce_expanded_index_kernel`, `gather_expanded_index_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 897-925
```cpp
897: void scatter_add_cpu_kernel(const Tensor& self, int64_t dim, const Tensor& index, const Tensor& src) {
898:   cpu_scatter_gather_base_kernel<>()(
899:     self, dim, index, src,
900:     "scatter_add_", reduce_add);
901: }
902:
903: void scatter_reduce_cpu_kernel(const Tensor& self, const int64_t dim, const Tensor& index,
904:                                const Tensor& src, const ReductionType& reduce) {
905:   switch (reduce) {
906:   case ReductionType::SUM :
907:     cpu_scatter_gather_base_kernel<>()(self, dim, index, src,
908:                                        "scatter_reduce_add_", reduce_add);
909:     break;
910:   case ReductionType::PROD :
911:     cpu_scatter_gather_base_kernel<>()(self, dim, index, src,
912:                                        "scatter_reduce_multiply_", reduce_multiply);
913:     break;
914:   default :
915:     break;
916:   }
917: }
918:
919: void scatter_reduce_two_cpu_kernel(const Tensor& self, const int64_t dim, const Tensor& index,
920:                                    const Tensor& src, const ReductionType& reduce) {
921:   switch (reduce) {
922:   case ReductionType::SUM :
923:     cpu_scatter_gather_base_kernel<>()(self, dim, index, src,
924:                                        "scatter_reduce_sum_", reduce_add);
925:     break;
```
- EN: The main symbol in this range is `scatter_add_cpu_kernel`, `scatter_reduce_cpu_kernel`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `scatter_add_cpu_kernel`, `scatter_reduce_cpu_kernel`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 926-959
```cpp
926:   case ReductionType::PROD :
927:     cpu_scatter_gather_base_kernel<>()(self, dim, index, src,
928:                                        "scatter_reduce_prod_", reduce_multiply);
929:     break;
930:   case ReductionType::MAX :
931:     cpu_scatter_gather_base_kernel<>()(self, dim, index, src,
932:                                        "scatter_reduce_amax_", reduce_maximum);
933:     break;
934:   case ReductionType::MIN :
935:     cpu_scatter_gather_base_kernel<>()(self, dim, index, src,
936:                                        "scatter_reduce_amin_", reduce_minimum);
937:     break;
938:   case ReductionType::MEAN :
939:     cpu_scatter_gather_base_kernel<>()(self, dim, index, src,
940:                                        "scatter_reduce_mean_", reduce_mean);
941:     break;
942:   }
943: }
944:
945: void scatter_scalar_reduce_cpu_kernel(const Tensor& self, const int64_t dim, const Tensor& index,
946:                                       const Scalar& value, const ReductionType& reduce) {
947:   switch (reduce) {
948:   case ReductionType::SUM :
949:     cpu_scatter_gather_base_kernel<>()(self, dim, index, value,
950:                                        "scatter_scalar_reduce_add_", reduce_add);
951:     break;
952:   case ReductionType::PROD :
953:     cpu_scatter_gather_base_kernel<>()(self, dim, index, value,
954:                                        "scatter_scalar_reduce_multiply_", reduce_multiply);
955:     break;
956:   default:
957:     break;
958:   }
959: }
```
- EN: The main symbol in this range is `scatter_scalar_reduce_cpu_kernel`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `scatter_scalar_reduce_cpu_kernel`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 961-976
```cpp
961: } // anonymous namespace
962:
963: REGISTER_DISPATCH(gather_stub, &gather_cpu_kernel)
964: REGISTER_DISPATCH(scatter_stub, &scatter_cpu_kernel)
965: REGISTER_DISPATCH(scatter_fill_stub, &scatter_fill_cpu_kernel)
966: REGISTER_DISPATCH(scatter_add_stub, &scatter_add_cpu_kernel)
967: REGISTER_DISPATCH(scatter_reduce_stub, &scatter_reduce_cpu_kernel)
968: REGISTER_DISPATCH(scatter_scalar_reduce_stub, &scatter_scalar_reduce_cpu_kernel)
969: REGISTER_DISPATCH(scatter_reduce_two_stub, &scatter_reduce_two_cpu_kernel)
970:
971: // fast paths for GNN usage
972: REGISTER_DISPATCH(scatter_add_expanded_index_stub, &scatter_add_expanded_index_kernel)
973: REGISTER_DISPATCH(scatter_reduce_expanded_index_stub, &scatter_reduce_expanded_index_kernel)
974: REGISTER_DISPATCH(gather_expanded_index_stub, &gather_expanded_index_kernel)
975:
976: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/native/NonEmptyUtils.h`, `ATen/native/DispatchStub.h`, `ATen/native/TensorIterator.h`, `ATen/native/TensorAdvancedIndexing.h`, `ATen/core/Tensor.h`, `ATen/Config.h`, `ATen/Dispatch.h`, `ATen/NumericUtils.h`, `ATen/Parallel.h`, `ATen/native/cpu/ReduceUtils.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `fbgemm/Utils.h`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `cpu_kernel`, `Vectorized`, `parallel_for`, `REGISTER_DISPATCH`, `AT_DISPATCH_FLOATING_TYPES`, `AT_DISPATCH_ALL_TYPES`, `Scalar`, `ScalarType`, `fbgemm`
