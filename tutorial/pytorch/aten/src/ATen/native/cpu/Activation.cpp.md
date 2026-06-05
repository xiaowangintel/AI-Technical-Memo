# Activation.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/Activation.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares native CPU activation kernels, dtype dispatch, and vectorized elementwise execution paths in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了原生 CPU 激活函数 kernel、数据类型分派以及向量化逐元素执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
 1: #define TORCH_ASSERT_NO_OPERATORS
 2: #ifndef _USE_MATH_DEFINES
 3: #define _USE_MATH_DEFINES
 4: #endif
 5:
 6: #include <ATen/native/Activation.h>
 7:
 8:
 9: #include <cmath>
10: #include <functional>
11:
12: #include <ATen/Dispatch.h>
13: #include <ATen/OpMathType.h>
14: #include <ATen/core/TensorBase.h>
15: #include <ATen/cpu/vec/functional.h>
16: #include <ATen/cpu/vec/vec.h>
17: #include <ATen/native/TensorIterator.h>
18: #include <ATen/native/cpu/Elu.h>
19: #include <ATen/native/cpu/Gelu.h>
20: #include <ATen/native/cpu/Loops.h>
21: #include <ATen/Parallel.h>
22:
23: #include <c10/core/Scalar.h>
24:
25: namespace at::native {
26:
27: namespace {
28:
29: #if defined(__GNUC__) && __GNUC__ == 14 && defined(__aarch64__) && !defined(__ARM_FEATURE_SVE)
30: // Workaround for gcc-14.2.0 ICE during RTL pass: expand when compiling for NEON
```
- EN: This range pulls in required headers, including `ATen/native/Activation.h`, `cmath`, `functional`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/native/Activation.h`, `cmath`, `functional`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 31-60
```cpp
31: __attribute__((optimize("no-tree-vectorize")))
32: #endif
33: void log_sigmoid_cpu_kernel(TensorBase &output, TensorBase &buffer, const TensorBase &input) {
34:   if (at::isReducedFloatingType(input.scalar_type())) {
35:     AT_DISPATCH_REDUCED_FLOATING_TYPES(input.scalar_type(), "log_sigmoid_cpu", [&]() {
36:     using Vec = Vectorized<scalar_t>;
37:     scalar_t* output_data = output.data_ptr<scalar_t>();
38:     scalar_t* buffer_data = buffer.data_ptr<scalar_t>();
39:     const scalar_t* input_data = input.const_data_ptr<scalar_t>();
40:     parallel_for(0, input.numel(), 1, [&] (int64_t begin, int64_t end) {
41:       int64_t size = end - begin;
42:       int64_t d = 0;
43:       for (; d < size - (size % Vec::size()); d += Vec::size()) {
44:         Vec data_vec = Vec::loadu(input_data + begin+ d);
45:         auto [data_vec0, data_vec1] = convert_to_float<scalar_t>(data_vec);
46:         Vectorized<float> min_vec = minimum(data_vec0, Vectorized<float>(float(0)));
47:         Vectorized<float> buffer_vec0 = data_vec0.abs().neg().exp();
48:         Vectorized<float> output_vec0 = min_vec - buffer_vec0.log1p();
49:         min_vec = minimum(data_vec1, Vectorized<float>(float(0)));
50:         Vectorized<float> buffer_vec1 = data_vec1.abs().neg().exp();
51:         Vectorized<float> output_vec1 = min_vec - buffer_vec1.log1p();
52:         convert_from_float<scalar_t>(buffer_vec0, buffer_vec1).store(buffer_data + begin + d);
53:         convert_from_float<scalar_t>(output_vec0, output_vec1).store(output_data + begin + d);
54:       }
55:       if (size - d > 0) {
56:         Vec data_vec = Vec::loadu(input_data + begin + d, size - d);
57:         auto [data_vec0, data_vec1] = convert_to_float<scalar_t>(data_vec);
58:         Vectorized<float> min_vec = minimum(data_vec0, Vectorized<float>(float(0)));
59:         Vectorized<float> buffer_vec0 = data_vec0.abs().neg().exp();
60:         Vectorized<float> output_vec0 = min_vec - buffer_vec0.log1p();
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `__attribute__`, `log_sigmoid_cpu_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `__attribute__`, `log_sigmoid_cpu_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 61-97
```cpp
61:         min_vec = minimum(data_vec1, Vectorized<float>(float(0)));
62:         Vectorized<float> buffer_vec1 = data_vec1.abs().neg().exp();
63:         Vectorized<float> output_vec1 = min_vec - buffer_vec1.log1p();
64:         convert_from_float<scalar_t>(buffer_vec0, buffer_vec1).store(buffer_data + begin + d, size - d);
65:         convert_from_float<scalar_t>(output_vec0, output_vec1).store(output_data + begin + d, size - d);
66:       }
67:     });
68:     });
69:   } else {
70:     AT_DISPATCH_FLOATING_TYPES(input.scalar_type(), "log_sigmoid_cpu", [&] {
71:       using Vec = Vectorized<scalar_t>;
72:       scalar_t* output_data = output.data_ptr<scalar_t>();
73:       scalar_t* buffer_data = buffer.data_ptr<scalar_t>();
74:       const scalar_t* input_data = input.const_data_ptr<scalar_t>();
75:       parallel_for(0, input.numel(), 1, [&] (int64_t begin, int64_t end) {
76:         int64_t size = end - begin;
77:         int64_t d = 0;
78:         for (; d < size - (size % Vec::size()); d += Vec::size()) {
79:           Vec data_vec = Vec::loadu(input_data + begin+ d);
80:           Vec min_vec = vec::minimum(data_vec, Vec(scalar_t(0)));
81:           Vec buffer_vec = data_vec.abs().neg().exp();
82:           Vec output_vec = min_vec - buffer_vec.log1p();
83:           buffer_vec.store(buffer_data + begin + d);
84:           output_vec.store(output_data + begin + d);
85:         }
86:         if (size - d > 0) {
87:           Vec data_vec = Vec::loadu(input_data + begin + d, size - d);
88:           Vec min_vec = vec::minimum(data_vec, Vec(scalar_t(0)));
89:           Vec buffer_vec = data_vec.abs().neg().exp();
90:           Vec output_vec = min_vec - buffer_vec.log1p();
91:           buffer_vec.store(buffer_data + begin + d, size - d);
92:           output_vec.store(output_data + begin + d, size - d);
93:         }
94:       });
95:     });
96:   }
97: }
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 99-127
```cpp
 99: void log_sigmoid_backward_cpu_kernel(TensorIterator& iter) {
100:   if (at::isReducedFloatingType(iter.dtype())) {
101:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "log_sigmoid_backward_cpu", [&]() {
102:       using Vec = Vectorized<scalar_t>;
103:       auto zero_val = float(0);
104:       auto zero_vec = Vectorized<float>(zero_val);
105:       auto one_val = float(1);
106:       auto one_vec = Vectorized<float>(one_val);
107:       cpu_kernel_vec(iter,
108:         [=](scalar_t a, scalar_t b, scalar_t c) -> scalar_t {
109:           auto in_negative = float(a) < float(0);
110:           auto max_deriv = in_negative ? float(1) : float(0);
111:           auto sign = in_negative ? float(1) : -float(1);
112:           return (max_deriv - sign * (float(b) / (float(1) + b))) * float(c);
113:         },
114:         [=](Vec a, Vec b, Vec c) -> Vec {
115:           auto [a0, a1] = convert_to_float<scalar_t>(a);
116:           auto [b0, b1] = convert_to_float<scalar_t>(b);
117:           auto [c0, c1] = convert_to_float<scalar_t>(c);
118:           auto mask = a0 < zero_vec;
119:           auto max_deriv_vec = Vectorized<float>::blendv(zero_vec, one_vec, mask);
120:           auto sign_vec = Vectorized<float>::blendv(one_vec.neg(), one_vec, mask);
121:           a0 = (max_deriv_vec - sign_vec * (b0 / (one_vec + b0))) * c0;
122:           mask = a1 < zero_vec;
123:           max_deriv_vec = Vectorized<float>::blendv(zero_vec, one_vec, mask);
124:           sign_vec = Vectorized<float>::blendv(one_vec.neg(), one_vec, mask);
125:           a1 = (max_deriv_vec - sign_vec * (b1 / (one_vec + b1))) * c1;
126:           return convert_from_float<scalar_t>(a0, a1);
127:         });
```
- EN: The main symbol in this range is `log_sigmoid_backward_cpu_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `log_sigmoid_backward_cpu_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 128-157
```cpp
128:     });
129:   } else {
130:     AT_DISPATCH_FLOATING_TYPES(iter.dtype(), "log_sigmoid_backward_cpu", [&]() {
131:     using Vec = Vectorized<scalar_t>;
132:     auto zero_val = scalar_t(0);
133:     auto zero_vec = Vec(zero_val);
134:     auto one_val = scalar_t(1);
135:     auto one_vec = Vec(one_val);
136:     cpu_kernel_vec(iter,
137:       [=](scalar_t a, scalar_t b, scalar_t c) -> scalar_t {
138:         auto in_negative = a < scalar_t(0);
139:         auto max_deriv = in_negative ? scalar_t(1) : scalar_t(0);
140:         auto sign = in_negative ? scalar_t(1) : -scalar_t(1);
141:         return (max_deriv - sign * (b / (scalar_t(1) + b))) * c;
142:       },
143:       [=](Vec a, Vec b, Vec c) -> Vec {
144:         auto mask = a < zero_vec;
145:         auto max_deriv_vec = Vec::blendv(zero_vec, one_vec, mask);
146:         auto sign_vec = Vec::blendv(one_vec.neg(), one_vec, mask);
147:         return (max_deriv_vec - sign_vec * (b / (one_vec + b))) * c;
148:       });
149:   });
150:   }
151: }
152:
153: void threshold_kernel(
154:     TensorIteratorBase& iter,
155:     const Scalar& threshold_scalar,
156:     const Scalar& value_scalar) {
157:   if (at::isReducedFloatingType(iter.dtype())) {
```
- EN: The main symbol in this range is `threshold_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `threshold_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 158-193
```cpp
158:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "threshold_cpu", [&]() {
159:       using Vec = Vectorized<float>;
160:       float threshold = threshold_scalar.to<float>();
161:       Vec threshold_v = Vec(threshold);
162:       scalar_t value = value_scalar.to<scalar_t>();
163:       Vec value_v = Vec(float(value));
164:       cpu_kernel_vec(
165:           iter,
166:           [&](scalar_t x, scalar_t other) -> scalar_t {
167:             return float(x) <= threshold ? value : other;
168:           },
169:           [&](Vectorized<scalar_t> x, Vectorized<scalar_t> other) -> Vectorized<scalar_t> {
170:             auto [x0, x1] = convert_to_float<scalar_t>(x);
171:             auto [other0, other1] = convert_to_float<scalar_t>(other);
172:             return convert_from_float<scalar_t>(Vec::blendv(other0, value_v, x0 <= threshold_v),
173:                                                 Vec::blendv(other1, value_v, x1 <= threshold_v));
174:           });
175:     });
176:   } else {
177:     AT_DISPATCH_ALL_TYPES(iter.dtype(), "threshold_cpu", [&] {
178:       using Vec = Vectorized<scalar_t>;
179:       scalar_t threshold = threshold_scalar.to<scalar_t>();
180:       Vec threshold_v = Vec(threshold);
181:       scalar_t value = value_scalar.to<scalar_t>();
182:       Vec value_v = Vec(value);
183:       cpu_kernel_vec(
184:           iter,
185:           [&](scalar_t x, scalar_t other) -> scalar_t {
186:             return x <= threshold ? value : other;
187:           },
188:           [&](Vec x, Vec other) -> Vec {
189:             return Vec::blendv(other, value_v, x <= threshold_v);
190:           });
191:     });
192:   }
193: }
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 195-223
```cpp
195: void elu_kernel(TensorIteratorBase& it, const Scalar& alpha, const Scalar& scale, const Scalar& input_scale) {
196:   if (at::isReducedFloatingType(it.common_dtype())) {
197:     AT_DISPATCH_REDUCED_FLOATING_TYPES(it.common_dtype(), "elu_cpu", [&]() {
198:       cpu_kernel_vec(
199:         it,
200:         get_scalar_elu_elementwise_func<scalar_t, float>(alpha.to<float>(), scale.to<float>(), input_scale.to<float>()),
201:         get_vectorized_elu_elementwise_func<scalar_t>(alpha.to<float>(), scale.to<float>(), input_scale.to<float>()));
202:     });
203:   } else {
204:     AT_DISPATCH_FLOATING_TYPES(it.common_dtype(), "elu_cpu", [&]() {
205:       cpu_kernel_vec(
206:           it,
207:           get_scalar_elu_elementwise_func<scalar_t>(alpha.to<scalar_t>(), scale.to<scalar_t>(), input_scale.to<scalar_t>()),
208:           get_vectorized_elu_elementwise_func<scalar_t>(alpha.to<scalar_t>(), scale.to<scalar_t>(), input_scale.to<scalar_t>()));
209:     });
210:   }
211: }
212:
213: void elu_backward_kernel(TensorIteratorBase& it, const Scalar& alpha, const Scalar& scale, const Scalar& input_scale, bool is_result) {
214:   if (at::isReducedFloatingType(it.common_dtype())) {
215:     AT_DISPATCH_REDUCED_FLOATING_TYPES(it.common_dtype(), "elu_backward_cpu", [&]() {
216:     auto negcoef = alpha.to<float>() * scale.to<float>();
217:     auto poscoef = scale.to<float>();
218:     auto negiptcoef = input_scale.to<float>();
219:     const Vectorized<float> negcoef_vec(negcoef);
220:     const Vectorized<float> negiptcoef_vec(negiptcoef);
221:     const Vectorized<float> poscoef_vec(poscoef);
222:     const Vectorized<float> zero_vec(static_cast<float>(0));
223:     cpu_kernel_vec(
```
- EN: The main symbol in this range is `elu_kernel`, `elu_backward_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `elu_kernel`, `elu_backward_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 224-253
```cpp
224:         it,
225:         [negcoef, negiptcoef, poscoef, is_result](scalar_t a, scalar_t b) -> scalar_t {
226:           if (is_result) {
227:             return float(b) <= float(0) ? float(a) * negiptcoef * (float(b) + negcoef) : float(a) * poscoef;
228:           } else {
229:             return float(b) <= float(0) ? float(a) * negiptcoef * negcoef * std::exp(float(b) * negiptcoef): float(a) * poscoef;
230:           }
231:         },
232:         [&negcoef_vec, &negiptcoef_vec, &poscoef_vec, &zero_vec, is_result](Vectorized<scalar_t> a, Vectorized<scalar_t> b) -> Vectorized<scalar_t> {
233:           auto [a0, a1] = convert_to_float<scalar_t>(a);
234:           auto [b0, b1] = convert_to_float<scalar_t>(b);
235:           auto cmp0 = (b0 > zero_vec);
236:           auto cmp1 = (b1 > zero_vec);
237:           auto get_res_masked = [&](Vectorized<float>& cmp, Vectorized<float>& a, Vectorized<float>& b) {
238:             if (is_result) {
239:               return !cmp.zero_mask() ? a * poscoef_vec :
240:                 Vectorized<float>::blendv(a * negiptcoef_vec * (b + negcoef_vec), a * poscoef_vec, cmp);
241:             } else {
242:               return Vectorized<float>::blendv(a * negiptcoef_vec * negcoef_vec * (b * negiptcoef_vec).exp(), a * poscoef_vec, cmp);
243:             }
244:           };
245:           auto res0 = get_res_masked(cmp0, a0, b0);
246:           auto res1 = get_res_masked(cmp1, a1, b1);
247:           return convert_from_float<scalar_t>(res0, res1);
248:         });
249:     });
250:   } else {
251:     AT_DISPATCH_FLOATING_TYPES(it.dtype(), "elu_backward_cpu", [&]() {
252:       using Vec = Vectorized<scalar_t>;
253:       auto negcoef = alpha.to<scalar_t>() * scale.to<scalar_t>();
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 254-284
```cpp
254:       auto poscoef = scale.to<scalar_t>();
255:       auto negiptcoef = input_scale.to<scalar_t>();
256:       const Vec negcoef_vec(negcoef);
257:       const Vec negiptcoef_vec(negiptcoef);
258:       const Vec poscoef_vec(poscoef);
259:       const Vec zero_vec(static_cast<scalar_t>(0));
260:       cpu_kernel_vec(
261:           it,
262:           [negcoef, negiptcoef, poscoef, is_result](scalar_t a, scalar_t b) -> scalar_t {
263:             if (is_result) {
264:               return b <= scalar_t(0) ? a * negiptcoef * (b + negcoef) : a * poscoef;
265:             } else {
266:               return b <= scalar_t(0) ? a * negiptcoef * negcoef * std::exp(b * negiptcoef): a * poscoef;
267:             }
268:           },
269:           [&negcoef_vec, &negiptcoef_vec, &poscoef_vec, &zero_vec, is_result](Vec a, Vec b) -> Vec {
270:             auto cmp = (b > zero_vec);
271:             if (is_result) {
272:               if (!cmp.zero_mask()) {  // only a * poscoef (which is very quick) needs to be computed
273:                 return a * poscoef_vec;
274:               } else {
275:                 return Vec::blendv(a * negiptcoef_vec * (b + negcoef_vec), a * poscoef_vec, cmp);
276:               }
277:             } else {
278:               return Vec::blendv(a * negiptcoef_vec * negcoef_vec * (b * negiptcoef_vec).exp(), a * poscoef_vec, cmp);
279:             }
280:           }
281:       );
282:     });
283:   }
284: }
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 286-314
```cpp
286: // TODO(yangxm): Add another fast kernel using formula
287: // y = 0.5x * (1 + tanh(sqrt(2/Pi) * (x + 0.044715x^3)))
288: // and the fast tanh impl from Eigen.
289: void GeluKernelImpl(TensorIteratorBase& it, GeluType approximate) {
290:   auto grain_size = at::internal::GRAIN_SIZE;
291:   // Numbers based on benchmarking.
292:   // Benchmark: benchmarks/operator_benchmarks/pt/gelu_test.py
293: #ifdef C10_MOBILE
294:   // Benchmarked on S8 US phone.
295:   // Internal benchmarking that converts operator benchmark into
296:   // a torchscript module and run that on mobile.
297:   // Same benchmark as server side.
298:   constexpr int64_t GELU_MIN_ELEMENTS_FOR_MULTI_THREADING{6144};
299: #else
300:   // Benchmarked on i9 8 core 16 thread machine.
301:   // 1 thread: cd benchmark/operator_benchmarks;
302:   //           python -m pt.gelu_test --tag_filter long --omp_num_threads 1
303:   // 2 threads: cd benchmark/operator_benchmarks;
304:   //           python -m pt.gelu_test --tag_filter long --omp_num_threads 1
305:   constexpr int64_t GELU_MIN_ELEMENTS_FOR_MULTI_THREADING{16384};
306: #endif
307:   if (it.numel() > GELU_MIN_ELEMENTS_FOR_MULTI_THREADING) {
308:     grain_size = it.numel() / at::get_num_threads();
309:   }
310:   if (approximate == GeluType::Tanh) {
311:     if (at::isReducedFloatingType(it.common_dtype())) {
312:       AT_DISPATCH_REDUCED_FLOATING_TYPES(it.common_dtype(), "GeluKernelImpl", [&]() {
313:         cpu_kernel_vec(
314:             it,
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 315-343
```cpp
315:             scalar_gelu_approximated_with_tanh<scalar_t>,
316:             vectorized_gelu_approximated_with_tanh<scalar_t>,
317:             grain_size);
318:       });
319:     } else {
320:       AT_DISPATCH_FLOATING_TYPES(
321:           it.dtype(), "GeluKernelImpl", [&]() {
322:         cpu_kernel_vec(
323:             it,
324:             scalar_gelu_approximated_with_tanh<scalar_t>,
325:             vectorized_gelu_approximated_with_tanh<scalar_t>,
326:             grain_size);
327:       });
328:     }
329:   } else {
330:     AT_DISPATCH_FLOATING_TYPES_AND2(
331:         ScalarType::Half,
332:         ScalarType::BFloat16,
333:         it.dtype(),
334:         "GeluKernelImpl",
335:         [&]() {
336:         cpu_kernel_vec(
337:             it,
338:             scalar_gelu<scalar_t>,
339:             vectorized_gelu<scalar_t>,
340:             grain_size);
341:       });
342:   }
343: }
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 345-386
```cpp
345: void GeluBackwardKernelImpl(TensorIteratorBase& it, GeluType approximate) {
346:   if (approximate == GeluType::Tanh) {
347:     if (at::isReducedFloatingType(it.common_dtype())) {
348:       AT_DISPATCH_REDUCED_FLOATING_TYPES(it.common_dtype(), "GeluBackwardKernelImpl", [&]() {
349:       auto kBetaVec = Vectorized<float>((float)(M_SQRT2 * M_2_SQRTPI * 0.5));
350:       auto kKappaVec = Vectorized<float>((float)(0.044715));
351:       auto kOneVec = Vectorized<float>((float)(1));
352:       auto kThreeVec = Vectorized<float>((float)(3));
353:       auto kPointFiveVec = Vectorized<float>((float)(0.5));
354:       cpu_kernel_vec(
355:           it,
356:           [](scalar_t dy, scalar_t x) -> scalar_t {
357:             const float kBeta = float(M_SQRT2 * M_2_SQRTPI * 0.5);
358:             const float kKappa = float(0.044715);
359:             float x_sq = float(x) * float(x);
360:             float x_cube = x_sq * float(x);
361:             float inner = kBeta * (float(x) + kKappa * x_cube);
362:             float tanh_inner = float(std::tanh(inner));
363:
364:             float left = float(0.5) * float(x);
365:             float right = float(1) + tanh_inner;
366:
367:             float left_derivative = float(0.5) * right;
368:
369:             float tanh_derivative = float(1) - tanh_inner * tanh_inner;
370:             float inner_derivative =
371:               kBeta * (float(1) + float(3) * kKappa * x_sq);
372:             float right_derivative = left * tanh_derivative * inner_derivative;
373:
374:             return float(dy) * (left_derivative + right_derivative);
375:           },
376:           [&](Vectorized<scalar_t> dy_vec, Vectorized<scalar_t> x_vec) -> Vectorized<scalar_t> {
377:             auto [x0_vec, x1_vec] = convert_to_float<scalar_t>(x_vec);
378:             auto [dy0_vec, dy1_vec] = convert_to_float<scalar_t>(dy_vec);
379:             auto x0_sq = x0_vec * x0_vec;
380:             auto x1_sq = x1_vec * x1_vec;
381:             auto x0_cube = x0_vec * x0_vec * x0_vec;
382:             auto x1_cube = x1_vec * x1_vec * x1_vec;
383:             auto inner_vec0 = kBetaVec * (x0_vec + kKappaVec * x0_cube);
384:             auto inner_vec1 = kBetaVec * (x1_vec + kKappaVec * x1_cube);
385:             auto tanh_inner_vec0 = inner_vec0.tanh();
386:             auto tanh_inner_vec1 = inner_vec1.tanh();
```
- EN: The main symbol in this range is `GeluBackwardKernelImpl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `GeluBackwardKernelImpl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 388-425
```cpp
388:             auto left_vec0 = kPointFiveVec * x0_vec;
389:             auto left_vec1 = kPointFiveVec * x1_vec;
390:             auto right_vec0 = kOneVec + tanh_inner_vec0;
391:             auto right_vec1 = kOneVec + tanh_inner_vec1;
392:
393:             auto left_derivative_vec0 = kPointFiveVec * right_vec0;
394:             auto left_derivative_vec1 = kPointFiveVec * right_vec1;
395:
396:             auto tanh_derivative_vec0 = kOneVec - tanh_inner_vec0 * tanh_inner_vec0;
397:             auto tanh_derivative_vec1 = kOneVec - tanh_inner_vec1 * tanh_inner_vec1;
398:             auto inner_derivative_vec0 = kBetaVec * (kOneVec + kThreeVec * kKappaVec * x0_sq);
399:             auto inner_derivative_vec1 = kBetaVec * (kOneVec + kThreeVec * kKappaVec * x1_sq);
400:             auto right_derivative_vec0 = left_vec0 * tanh_derivative_vec0 * inner_derivative_vec0;
401:             auto right_derivative_vec1 = left_vec1 * tanh_derivative_vec1 * inner_derivative_vec1;
402:
403:             auto res0 = dy0_vec * (left_derivative_vec0 + right_derivative_vec0);
404:             auto res1 = dy1_vec * (left_derivative_vec1 + right_derivative_vec1);
405:             return convert_from_float<scalar_t>(res0, res1);
406:           });
407:       });
408:     } else {
409:       AT_DISPATCH_FLOATING_TYPES(
410:           it.dtype(), "GeluBackwardKernelImpl", [&]() {
411:         using Vec = vec::Vectorized<scalar_t>;
412:         const Vec kBetaVec(scalar_t(M_SQRT2 * M_2_SQRTPI * 0.5));
413:         const Vec kKappaVec(scalar_t(0.044715));
414:         const Vec kOneVec(scalar_t(1));
415:         const Vec kThreeVec(scalar_t(3));
416:         const Vec kPointFiveVec(scalar_t(0.5));
417:         cpu_kernel_vec(
418:             it,
419:             [](scalar_t dy, scalar_t x) {
420:               const scalar_t kBeta = M_SQRT2 * M_2_SQRTPI * 0.5;
421:               const scalar_t kKappa = 0.044715;
422:               auto x_sq = x * x;
423:               auto x_cube = x_sq * x;
424:               auto inner = kBeta * (x + kKappa * x_cube);
425:               auto tanh_inner = std::tanh(inner);
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 427-456
```cpp
427:               auto left = scalar_t(0.5) * x;
428:               auto right = scalar_t(1) + tanh_inner;
429:
430:               auto left_derivative = scalar_t(0.5) * right;
431:
432:               auto tanh_derivative = scalar_t(1) - tanh_inner * tanh_inner;
433:               auto inner_derivative =
434:                 kBeta * (scalar_t(1) + scalar_t(3) * kKappa * x_sq);
435:               auto right_derivative = left * tanh_derivative * inner_derivative;
436:
437:               return dy * (left_derivative + right_derivative);
438:             },
439:             [&](Vec dy_vec, Vec x_vec) {
440:               auto x_sq = x_vec * x_vec;
441:               auto x_cube = x_vec * x_vec * x_vec;
442:               auto inner_vec =
443:                   kBetaVec * (x_vec + kKappaVec * x_cube);
444:               auto tanh_inner_vec = inner_vec.tanh();
445:
446:               auto left_vec = kPointFiveVec * x_vec;
447:               auto right_vec = kOneVec + tanh_inner_vec;
448:
449:               auto left_derivative_vec = kPointFiveVec * right_vec;
450:
451:               auto tanh_derivative_vec =
452:                   kOneVec - tanh_inner_vec * tanh_inner_vec;
453:               auto inner_derivative_vec =
454:                   kBetaVec * (kOneVec + kThreeVec * kKappaVec * x_sq);
455:               auto right_derivative_vec =
456:                   left_vec * tanh_derivative_vec * inner_derivative_vec;
```
- EN: The block also assembles or returns the result expected by the surrounding operator code.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 458-486
```cpp
458:               return dy_vec * (left_derivative_vec + right_derivative_vec);
459:             });
460:       });
461:     }
462:   } else {
463:     if (at::isReducedFloatingType(it.common_dtype())) {
464:       AT_DISPATCH_REDUCED_FLOATING_TYPES(it.common_dtype(), "GeluBackwardKernelImpl", [&]() {
465:       auto kAlphaVec = Vectorized<float>((float)(M_SQRT1_2));
466:       auto kBetaVec = Vectorized<float>((float)(M_2_SQRTPI * M_SQRT1_2 * 0.5));
467:       auto kOneVec = Vectorized<float>((float)(1));
468:       auto kPointFiveVec = Vectorized<float>((float)(0.5));
469:       auto kMinusPointFiveVec = Vectorized<float>((float)(-0.5));
470:       cpu_kernel_vec(
471:           it,
472:           [](scalar_t dy, scalar_t x) -> scalar_t {
473:               const float kAlpha = float(M_SQRT1_2);
474:               const float kBeta = float(M_2_SQRTPI) * float(M_SQRT1_2) * float(0.5);
475:               const float cdf =
476:                   float(0.5) * (float(1) + std::erf(float(x) * kAlpha));
477:               const float pdf = kBeta * std::exp(float(x) * float(x) * float(-0.5));
478:               return float(dy) * (cdf + float(x) * pdf);
479:           },
480:           [&](Vectorized<scalar_t> dy, Vectorized<scalar_t> x) -> Vectorized<scalar_t> {
481:               auto [x0, x1] = convert_to_float<scalar_t>(x);
482:               auto [dy0, dy1] = convert_to_float<scalar_t>(dy);
483:               auto cdf_vec0 = kPointFiveVec * (kOneVec + (x0 * kAlphaVec).erf());
484:               auto cdf_vec1 = kPointFiveVec * (kOneVec + (x1 * kAlphaVec).erf());
485:               auto pdf_vec0 = kBetaVec * (x0 * x0 * kMinusPointFiveVec).exp();
486:               auto pdf_vec1 = kBetaVec * (x1 * x1 * kMinusPointFiveVec).exp();
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 487-521
```cpp
487:               auto res0 = dy0 * (cdf_vec0 + x0 * pdf_vec0);
488:               auto res1 = dy1 * (cdf_vec1 + x1 * pdf_vec1);
489:               return convert_from_float<scalar_t>(res0, res1);
490:           });
491:       });
492:     } else {
493:       AT_DISPATCH_FLOATING_TYPES(
494:           it.dtype(), "GeluBackwardKernelImpl", [&]() {
495:         using Vec = vec::Vectorized<scalar_t>;
496:         const Vec kAlphaVec(scalar_t(M_SQRT1_2));
497:         const Vec kBetaVec(scalar_t(M_2_SQRTPI * M_SQRT1_2 * 0.5));
498:         const Vec kOneVec(scalar_t(1));
499:         const Vec kPointFiveVec(scalar_t(0.5));
500:         const Vec kMinusPointFiveVec(scalar_t(-0.5));
501:         cpu_kernel_vec(
502:             it,
503:             [](scalar_t dy, scalar_t x) {
504:               const scalar_t kAlpha = scalar_t(M_SQRT1_2);
505:               const scalar_t kBeta = M_2_SQRTPI * M_SQRT1_2 * scalar_t(0.5);
506:               const scalar_t cdf =
507:                   scalar_t(0.5) * (scalar_t(1) + std::erf(x * kAlpha));
508:               const scalar_t pdf = kBeta * std::exp(x * x * scalar_t(-0.5));
509:               return dy * (cdf + x * pdf);
510:             },
511:             [&](Vec dy_vec, Vec x_vec) {
512:               const Vec cdf_vec =
513:                   kPointFiveVec * (kOneVec + (x_vec * kAlphaVec).erf());
514:               const Vec pdf_vec =
515:                   kBetaVec * (x_vec * x_vec * kMinusPointFiveVec).exp();
516:               return dy_vec * (cdf_vec + x_vec * pdf_vec);
517:             });
518:       });
519:     }
520:   }
521: }
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 523-551
```cpp
523: void hardsigmoid_kernel(TensorIteratorBase& iter) {
524:   if (at::isReducedFloatingType(iter.dtype())) {
525:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "hardsigmoid_cpu", [&]() {
526:     const float zero(0.0f);
527:     const float three(3.0f);
528:     const float six(6.0f);
529:     using Vec = vec::Vectorized<float>;
530:     const Vec kZeroVec(zero);
531:     const Vec kThreeVec(three);
532:     const Vec kSixVec(six);
533:     cpu_kernel_vec(
534:         iter,
535:         [&](scalar_t self_val) -> scalar_t {
536:           return std::min(std::max(float(self_val) + three, zero), six) / six;
537:         },
538:         [&](vec::Vectorized<scalar_t> self_val) -> vec::Vectorized<scalar_t> {
539:           auto [self_val0, self_val1] = convert_to_float<scalar_t>(self_val);
540:           self_val0 = minimum(
541:             maximum(self_val0 + kThreeVec, kZeroVec),
542:             kSixVec
543:           ) / kSixVec;
544:           self_val1 = minimum(
545:             maximum(self_val1 + kThreeVec, kZeroVec),
546:             kSixVec
547:           ) / kSixVec;
548:           return convert_from_float<scalar_t>(self_val0, self_val1);
549:         });
550:     });
551:   } else {
```
- EN: The main symbol in this range is `hardsigmoid_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `hardsigmoid_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 552-581
```cpp
552:     AT_DISPATCH_FLOATING_TYPES(iter.dtype(), "hardsigmoid_cpu", [&] {
553:     const scalar_t zero(0.0f);
554:     const scalar_t three(3.0f);
555:     const scalar_t six(6.0f);
556:     using Vec = vec::Vectorized<scalar_t>;
557:     const Vec kZeroVec(zero);
558:     const Vec kThreeVec(three);
559:     const Vec kSixVec(six);
560:     cpu_kernel_vec(
561:         iter,
562:         [&](scalar_t self_val) {
563:           return std::min(std::max(self_val + three, zero), six) / six;
564:         },
565:         [&](Vec self_val) {
566:           return vec::minimum(
567:             vec::maximum(self_val + kThreeVec, kZeroVec),
568:             kSixVec
569:           ) / kSixVec;
570:         });
571:   });
572:   }
573: }
574:
575: void hardsigmoid_backward_kernel(TensorIteratorBase& iter) {
576:   if (at::isReducedFloatingType(iter.dtype())) {
577:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.common_dtype(), "hardsigmoid_backward", [&]() {
578:     const float zero(0.0f);
579:     const float three(3.0f);
580:     const float neg_three(-3.0f);
581:     const float one_sixth(1.0f / 6.0f);
```
- EN: The main symbol in this range is `hardsigmoid_backward_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `hardsigmoid_backward_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 582-624
```cpp
582:     using Vec = Vectorized<float>;
583:     Vec kZeroVec(0.0f);
584:     Vec kOneSixthVec(1.0f / 6.0f);
585:     cpu_kernel_vec(
586:         iter,
587:         [=](scalar_t grad_val, scalar_t self_val) -> scalar_t {
588:           return (float(self_val) > neg_three && float(self_val) < three)
589:             ? float(grad_val) * one_sixth
590:             : zero;
591:         },
592:         [=](Vectorized<scalar_t> grad_val, Vectorized<scalar_t> self_val) -> Vectorized<scalar_t> {
593:           auto [self_val0, self_val1] = convert_to_float<scalar_t>(self_val);
594:           auto [grad_val0, grad_val1] = convert_to_float<scalar_t>(grad_val);
595:           Vec gradNonZeroMask = (self_val0 > neg_three) & (self_val0 < three);
596:           self_val0 = Vec::blendv(kZeroVec, grad_val0 * kOneSixthVec, gradNonZeroMask);
597:           gradNonZeroMask = (self_val1 > neg_three) & (self_val1 < three);
598:           self_val1 = Vec::blendv(kZeroVec, grad_val1 * kOneSixthVec, gradNonZeroMask);
599:           return convert_from_float<scalar_t>(self_val0, self_val1);
600:         });
601:     });
602:   } else {
603:     AT_DISPATCH_FLOATING_TYPES(iter.dtype(), "hardsigmoid_backward", [&] {
604:     const scalar_t zero(0.0f);
605:     const scalar_t three(3.0f);
606:     const scalar_t neg_three(-3.0f);
607:     const scalar_t one_sixth(1.0f / 6.0f);
608:     using Vec = Vectorized<scalar_t>;
609:     Vec kZeroVec(0.0f);
610:     Vec kOneSixthVec(1.0f / 6.0f);
611:     cpu_kernel_vec(
612:         iter,
613:         [=](scalar_t grad_val, scalar_t self_val) {
614:           return (self_val > neg_three && self_val < three)
615:             ? grad_val * one_sixth
616:             : zero;
617:         },
618:         [=](Vec grad_val, Vec self_val) {
619:           Vec gradNonZeroMask = (self_val > neg_three) & (self_val < three);
620:           return Vec::blendv(kZeroVec, grad_val * kOneSixthVec, gradNonZeroMask);
621:         });
622:   });
623:   }
624: }
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 626-654
```cpp
626: void hardshrink_kernel(TensorIteratorBase& iter, const Scalar& lambd) {
627:     AT_DISPATCH_FLOATING_TYPES_AND2(kBFloat16, kHalf, iter.dtype(), "hardshrink_cpu", [&] {
628:     auto lambd_val = lambd.to<scalar_t>();
629:     using Vec = Vectorized<scalar_t>;
630:     cpu_kernel_vec(
631:         iter,
632:         [=](scalar_t self_val) {
633:           return (self_val >= -lambd_val && self_val <= lambd_val) ? scalar_t(0)
634:                                                                    : self_val;
635:         },
636:         [=](Vec self_val) {
637:           return Vec::blendv(self_val, Vec(0), (self_val >= -lambd_val) & (self_val <= lambd_val));
638:         });
639:   });
640: }
641:
642: void softshrink_kernel(TensorIteratorBase& iter, const Scalar& lambd) {
643:   if (at::isReducedFloatingType(iter.dtype())) {
644:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.common_dtype(), "softshrink_cpu", [&]() {
645:     auto lambd_val = lambd.to<float>();
646:     auto lambdVec = Vectorized<float>(lambd_val);
647:     cpu_kernel_vec(
648:       iter,
649:       [=](scalar_t a) -> scalar_t {
650:         return float(a) > lambd_val ? a - lambd_val
651:                 : (float(a) < -lambd_val ? a + lambd_val : float(a) * float(0));
652:       },
653:       [=](Vectorized<scalar_t> self_val) -> Vectorized<scalar_t> {
654:           auto [self_val0, self_val1] = convert_to_float<scalar_t>(self_val);
```
- EN: The main symbol in this range is `hardshrink_kernel`, `softshrink_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `hardshrink_kernel`, `softshrink_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 655-696
```cpp
655:           auto self_val_t0 = convert_from_float<scalar_t>(
656:               ((self_val0 > lambdVec) | (self_val0.isnan())) & (self_val0 - lambdVec),
657:               ((self_val1 > lambdVec) | (self_val1.isnan())) & (self_val1 - lambdVec));
658:           auto self_val_t1 = convert_from_float<scalar_t>(
659:               ((self_val0 < -lambd_val) | (self_val0.isnan())) & (self_val0 + lambdVec),
660:               ((self_val1 < -lambd_val) | (self_val1.isnan())) & (self_val1 + lambdVec));
661:           return (self_val_t0 | self_val_t1);
662:       });
663:     });
664:   } else {
665:     AT_DISPATCH_FLOATING_TYPES(iter.dtype(), "softshrink_cpu", [&]() {
666:     auto lambd_val = lambd.to<scalar_t>();
667:     auto lambdVec = Vectorized<scalar_t>(lambd_val);
668:     cpu_kernel_vec(
669:       iter,
670:       [=](scalar_t a) -> scalar_t {
671:         return a > lambd_val ? a - lambd_val : (a < -lambd_val ? a + lambd_val : a * scalar_t(0));
672:       },
673:       [=](Vectorized<scalar_t> self_val) -> Vectorized<scalar_t> {
674:           Vectorized<scalar_t> self_val_t0, self_val_t1;
675:           self_val_t0 = ((self_val > lambdVec) | (self_val.isnan())) & (self_val - lambdVec);
676:           self_val_t1 = ((self_val < -lambd_val) | (self_val.isnan())) & (self_val + lambdVec);
677:           return (self_val_t0 | self_val_t1);
678:       });
679:   });
680:   }
681: }
682:
683: void shrink_backward_kernel(TensorIteratorBase& iter, const Scalar& lambd) {
684:     AT_DISPATCH_FLOATING_TYPES_AND2(kBFloat16, kHalf, iter.dtype(), "shrink_backward_cpu", [&] {
685:     auto lambd_val = lambd.to<scalar_t>();
686:     cpu_kernel_vec(
687:         iter,
688:         [=](scalar_t grad_val, scalar_t self_val) {
689:           return (self_val >= -lambd_val && self_val <= lambd_val) ? scalar_t(0)
690:                                                                    : grad_val;
691:         },
692:         [=](Vectorized<scalar_t> grad_val, Vectorized<scalar_t> self_val) {
693:           return ((self_val < -lambd_val) | (self_val > lambd_val)) & grad_val;
694:         });
695:   });
696: }
```
- EN: The main symbol in this range is `shrink_backward_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `shrink_backward_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 698-731
```cpp
698: void hardtanh_backward_kernel(TensorIterator& iter, const Scalar& min, const Scalar& max) {
699:   if (at::isReducedFloatingType(iter.dtype())) {
700:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "hardshrink_backward_cpu", [&]() {
701:       auto min_val = min.to<float>();
702:       auto max_val = max.to<float>();
703:       cpu_kernel_vec(
704:           iter,
705:           [=](scalar_t grad_val, scalar_t self_val) -> scalar_t {
706:             return (float(self_val) <= min_val || float(self_val) >= max_val) ? scalar_t(0) : grad_val;
707:           },
708:           [=](Vectorized<scalar_t> grad_val, Vectorized<scalar_t> self_val) -> Vectorized<scalar_t> {
709:             auto [grad_val0, grad_val1] = convert_to_float<scalar_t>(grad_val);
710:             auto [self_val0, self_val1] = convert_to_float<scalar_t>(self_val);
711:             return convert_from_float<scalar_t>(
712:               ((self_val0 > min_val) & (self_val0 < max_val)) & grad_val0,
713:               ((self_val1 > min_val) & (self_val1 < max_val)) & grad_val1
714:             );
715:           });
716:     });
717:   } else {
718:     AT_DISPATCH_FLOATING_TYPES(iter.dtype(), "hardshrink_backward_cpu", [&] {
719:     auto min_val = min.to<scalar_t>();
720:     auto max_val = max.to<scalar_t>();
721:     cpu_kernel_vec(
722:         iter,
723:         [=](scalar_t grad_val, scalar_t self_val) {
724:           return (self_val <= min_val || self_val >= max_val) ? scalar_t(0) : grad_val;
725:         },
726:         [=](Vectorized<scalar_t> grad_val, Vectorized<scalar_t> self_val) {
727:           return ((self_val > min_val) & (self_val < max_val)) & grad_val;
728:         });
729:   });
730:   }
731: }
```
- EN: The main symbol in this range is `hardtanh_backward_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `hardtanh_backward_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 733-761
```cpp
733: void hardswish_kernel(TensorIterator& iter) {
734:   if (at::isReducedFloatingType(iter.dtype())) {
735:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "hardswish_cpu", [&]() {
736:     const float zero(0.0f);
737:     const float three(3.0f);
738:     const float six(6.0f);
739:     using Vec = vec::Vectorized<float>;
740:     const Vec kZeroVec(zero);
741:     const Vec kThreeVec(three);
742:     const Vec kSixVec(six);
743:     cpu_kernel_vec(
744:       iter,
745:       [&](scalar_t x) -> scalar_t {
746:         return float(x) * std::min(std::max(float(x) + three, zero), six) / six;
747:       },
748:       [&](vec::Vectorized<scalar_t> x_vec) {
749:         auto [x_vec0, x_vec1] = convert_to_float<scalar_t>(x_vec);
750:         x_vec0 = x_vec0 * minimum(
751:           maximum(x_vec0 + kThreeVec, kZeroVec),
752:           kSixVec
753:         ) / kSixVec;
754:         x_vec1 = x_vec1 * minimum(
755:           maximum(x_vec1 + kThreeVec, kZeroVec),
756:           kSixVec
757:         ) / kSixVec;
758:         return convert_from_float<scalar_t>(x_vec0, x_vec1);
759:       });
760:     });
761:   } else {
```
- EN: The main symbol in this range is `hardswish_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `hardswish_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 762-791
```cpp
762:     AT_DISPATCH_FLOATING_TYPES(iter.dtype(), "hardswish_cpu", [&]() {
763:     const scalar_t zero(0.0f);
764:     const scalar_t three(3.0f);
765:     const scalar_t six(6.0f);
766:     using Vec = vec::Vectorized<scalar_t>;
767:     const Vec kZeroVec(zero);
768:     const Vec kThreeVec(three);
769:     const Vec kSixVec(six);
770:     cpu_kernel_vec(
771:       iter,
772:       [&](scalar_t x) {
773:         return x * std::min(std::max(x + three, zero), six) / six;
774:       },
775:       [&](Vec x_vec) {
776:         return x_vec * vec::minimum(
777:           vec::maximum(x_vec + kThreeVec, kZeroVec),
778:           kSixVec
779:         ) / kSixVec;
780:       }
781:     );
782:   });
783:   }
784: }
785:
786: void hardswish_backward_kernel(TensorIterator& iter) {
787:   if (at::isReducedFloatingType(iter.dtype())) {
788:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "hardswish_backward_cpu", [&]() {
789:     const float zero(0.0f);
790:     const float three(3.0f);
791:     const float neg_three(-3.0f);
```
- EN: The main symbol in this range is `hardswish_backward_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `hardswish_backward_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 792-821
```cpp
792:     const float one_half(0.5f);
793:     using Vec = vec::Vectorized<float>;
794:     const Vec kZeroVec(zero);
795:     const Vec kThreeVec(three);
796:     const Vec kNegThreeVec(neg_three);
797:     const Vec kOneHalfVec(one_half);
798:     cpu_kernel_vec(
799:       iter,
800:       [&](scalar_t grad_val, scalar_t self_val) -> scalar_t {
801:         if (float(self_val) <= neg_three) {
802:           return zero;
803:         } else if (float(self_val) < three) {
804:           return float(grad_val) * ((float(self_val) / three) + one_half);
805:         } else {
806:           return grad_val;
807:         }
808:       },
809:       [&](vec::Vectorized<scalar_t> grad_val, vec::Vectorized<scalar_t> self_val) {
810:         auto [self_val0, self_val1] = convert_to_float<scalar_t>(self_val);
811:         auto [grad_val0, grad_val1] = convert_to_float<scalar_t>(grad_val);
812:         self_val0 = Vec::blendv(
813:           Vec::blendv(
814:             grad_val0 * ((self_val0 / kThreeVec) + kOneHalfVec),
815:             grad_val0,
816:             self_val0 >= kThreeVec
817:           ),
818:           kZeroVec,
819:           self_val0 <= kNegThreeVec
820:         );
821:         self_val1 = Vec::blendv(
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 822-851
```cpp
822:           Vec::blendv(
823:             grad_val1 * ((self_val1 / kThreeVec) + kOneHalfVec),
824:             grad_val1,
825:             self_val1 >= kThreeVec
826:           ),
827:           kZeroVec,
828:           self_val1 <= kNegThreeVec
829:         );
830:         return convert_from_float<scalar_t>(self_val0, self_val1);
831:       });
832:     });
833:   } else {
834:     AT_DISPATCH_FLOATING_TYPES(iter.dtype(), "hardswish_backward_cpu", [&]() {
835:     const scalar_t zero(0.0f);
836:     const scalar_t three(3.0f);
837:     const scalar_t neg_three(-3.0f);
838:     const scalar_t one_half(0.5f);
839:     using Vec = vec::Vectorized<scalar_t>;
840:     const Vec kZeroVec(zero);
841:     const Vec kThreeVec(three);
842:     const Vec kNegThreeVec(neg_three);
843:     const Vec kOneHalfVec(one_half);
844:     cpu_kernel_vec(
845:       iter,
846:       [&](scalar_t grad_val, scalar_t self_val) {
847:         if (self_val <= neg_three) {
848:           return zero;
849:         } else if (self_val < three) {
850:           return grad_val * ((self_val / three) + one_half);
851:         } else {
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 852-881
```cpp
852:           return grad_val;
853:         }
854:       },
855:       [&](Vec grad_val, Vec self_val) {
856:         return Vec::blendv(
857:           Vec::blendv(
858:             grad_val * ((self_val / kThreeVec) + kOneHalfVec),
859:             grad_val,
860:             self_val >= kThreeVec
861:           ),
862:           kZeroVec,
863:           self_val <= kNegThreeVec
864:         );
865:       }
866:     );
867:   });
868:   }
869: }
870:
871: void leaky_relu_kernel(TensorIteratorBase& iter, const Scalar& negval_) {
872:   if (at::isReducedFloatingType(iter.dtype())) {
873:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "leaky_relu_cpu", [&]() {
874:     auto zero_vec = Vectorized<float>((float)(0));
875:     auto one_vec = Vectorized<float>((float)(1));
876:     float negval = negval_.to<float>();
877:     Vectorized<float> negval_v = Vectorized<float>(negval);
878:     cpu_kernel_vec(
879:         iter,
880:         [&](scalar_t a) -> scalar_t {
881:           return float(a) > float(0) ? float(a) : float(a) * negval;
```
- EN: The main symbol in this range is `leaky_relu_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `leaky_relu_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 882-911
```cpp
882:         },
883:         [&](Vectorized<scalar_t> a) -> Vectorized<scalar_t> {
884:           auto [a0, a1] = convert_to_float<scalar_t>(a);
885:           auto res0 = a0 * (Vectorized<float>::blendv(negval_v, one_vec, a0 > zero_vec));
886:           auto res1 = a1 * (Vectorized<float>::blendv(negval_v, one_vec, a1 > zero_vec));
887:           return convert_from_float<scalar_t>(res0, res1);
888:         });
889:     });
890:   } else {
891:     AT_DISPATCH_FLOATING_TYPES(iter.dtype(), "leaky_relu_cpu", [&] {
892:       using Vec = Vectorized<scalar_t>;
893:       auto zero_vec = Vec((scalar_t)(0));
894:       auto one_vec = Vec((scalar_t)(1));
895:       scalar_t negval = negval_.to<scalar_t>();
896:       Vec negval_v = Vec(negval);
897:       cpu_kernel_vec(
898:           iter,
899:           [&](scalar_t a) -> scalar_t {
900:             return a > scalar_t(0) ? a : a * negval;
901:           },
902:           [&](Vec a) -> Vec {
903:             auto r = Vec::blendv(negval_v, one_vec, a > zero_vec);
904:             return a * r;
905:           });
906:     });
907:   }
908: }
909:
910: void leaky_relu_backward_kernel(TensorIteratorBase& iter, const Scalar& negval_) {
911:   if (at::isReducedFloatingType(iter.dtype())) {
```
- EN: The main symbol in this range is `leaky_relu_backward_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `leaky_relu_backward_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 912-948
```cpp
912:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "leaky_relu_backward_cpu", [&]() {
913:     auto zero_vec = Vectorized<float>((float)(0));
914:     auto one_vec = Vectorized<float>((float)(1));
915:     float negval = negval_.to<float>();
916:     Vectorized<float> negval_v = Vectorized<float>(negval);
917:     cpu_kernel_vec(
918:       iter,
919:       [&](scalar_t a, scalar_t b) -> scalar_t {
920:         return float(a) > float(0) ? float(b) : float(b) * negval;
921:       },
922:       [&](Vectorized<scalar_t> a, Vectorized<scalar_t> b) -> Vectorized<scalar_t> {
923:         auto [a0, a1] = convert_to_float<scalar_t>(a);
924:         auto [b0, b1] = convert_to_float<scalar_t>(b);
925:         auto res0 = b0 * (Vectorized<float>::blendv(negval_v, one_vec, a0 > zero_vec));
926:         auto res1 = b1 * (Vectorized<float>::blendv(negval_v, one_vec, a1 > zero_vec));
927:         return convert_from_float<scalar_t>(res0, res1);
928:       });
929:     });
930:   } else {
931:     AT_DISPATCH_FLOATING_TYPES(iter.dtype(), "leaky_relu_backward_cpu", [&] {
932:       using Vec = Vectorized<scalar_t>;
933:       auto zero_vec = Vec((scalar_t)(0));
934:       auto one_vec = Vec((scalar_t)(1));
935:       scalar_t negval = negval_.to<scalar_t>();
936:       Vec negval_v = Vec(negval);
937:       cpu_kernel_vec(
938:           iter,
939:           [&](scalar_t a, scalar_t b) -> scalar_t {
940:             return a > scalar_t(0) ? b : b * negval;
941:           },
942:           [&](Vec a, Vec b) -> Vec {
943:             auto r = Vec::blendv(negval_v, one_vec, a > zero_vec);
944:             return b * r;
945:           });
946:     });
947:   }
948: }
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 950-991
```cpp
950: void softplus_kernel(TensorIteratorBase& iter, const Scalar& beta_, const Scalar& threshold_) {
951:     if (at::isReducedFloatingType(iter.dtype())) {
952:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "softplus_cpu", [&]() {
953:       using Vec = Vectorized<float>;
954:       auto beta = beta_.to<float>();
955:       auto threshold = threshold_.to<float>();
956:       const Vec beta_vec(beta);
957:       const Vec threshold_vec(threshold);
958:       cpu_kernel_vec(
959:           iter,
960:           [beta, threshold](scalar_t a) -> scalar_t {
961:             return (float(a) * beta) > threshold ? a
962:               : static_cast<scalar_t>((std::log1p(std::exp(float(a) * beta))) / beta);
963:           },
964:           [beta_vec, threshold_vec](Vectorized<scalar_t> a) -> Vectorized<scalar_t> {
965:             auto [a0, a1] = convert_to_float<scalar_t>(a);
966:             a0 = Vec::blendv((a0 * beta_vec).exp().log1p() / beta_vec, a0, (a0 * beta_vec) > threshold_vec);
967:             a1 = Vec::blendv((a1 * beta_vec).exp().log1p() / beta_vec, a1, (a1 * beta_vec) > threshold_vec);
968:             return convert_from_float<scalar_t>(a0, a1);
969:           }
970:       );
971:     });
972:   } else {
973:     AT_DISPATCH_FLOATING_TYPES(iter.dtype(), "softplus_cpu", [&]() {
974:     using Vec = Vectorized<scalar_t>;
975:     auto beta = beta_.to<scalar_t>();
976:     auto threshold = threshold_.to<scalar_t>();
977:     const Vec beta_vec(beta);
978:     const Vec threshold_vec(threshold);
979:     cpu_kernel_vec(
980:         iter,
981:         [beta, threshold](scalar_t a) -> scalar_t {
982:           return (a * beta) > threshold ? a
983:             : static_cast<scalar_t>(std::log1p(std::exp(a * beta))) / beta;
984:         },
985:         [beta_vec, threshold_vec](Vec a) -> Vec {
986:           return Vec::blendv((a * beta_vec).exp().log1p() / beta_vec, a, (a * beta_vec) > threshold_vec);
987:         }
988:     );
989:   });
990:   }
991: }
```
- EN: The main symbol in this range is `softplus_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `softplus_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 993-1021
```cpp
 993: void softplus_backward_kernel(TensorIteratorBase& iter, const Scalar& beta_, const Scalar& threshold_) {
 994:   if (at::isReducedFloatingType(iter.dtype())) {
 995:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "softplus_backward_cpu", [&]() {
 996:     using Vec = Vectorized<float>;
 997:     auto beta = beta_.to<float>();
 998:     auto threshold = threshold_.to<float>();
 999:     const Vec beta_vec(beta);
1000:     const Vec threshold_vec(threshold);
1001:     const Vec one_vec(1.0f);
1002:     cpu_kernel_vec(
1003:         iter,
1004:         [beta, threshold](scalar_t a, scalar_t b) -> scalar_t {
1005:           float z = std::exp(float(b) * beta);
1006:           return (float(b) * beta) > threshold ? a : static_cast<scalar_t>(float(a) * z / (z + float(1.)));
1007:         },
1008:         [beta_vec, one_vec, threshold_vec](Vectorized<scalar_t> a, Vectorized<scalar_t> b) -> Vectorized<scalar_t> {
1009:           auto [a0, a1] = convert_to_float<scalar_t>(a);
1010:           auto [b0, b1] = convert_to_float<scalar_t>(b);
1011:           Vec z = (b0 * beta_vec).exp();
1012:           a0 = Vec::blendv(a0 * z / (z + one_vec), a0, (b0 * beta_vec) > threshold_vec);
1013:           z = (b1 * beta_vec).exp();
1014:           a1 = Vec::blendv(a1 * z / (z + one_vec), a1, (b1 * beta_vec) > threshold_vec);
1015:           return convert_from_float<scalar_t>(a0, a1);
1016:         });
1017:     });
1018:   } else {
1019:     AT_DISPATCH_FLOATING_TYPES(iter.dtype(), "softplus_backward_cpu", [&]() {
1020:     using Vec = Vectorized<scalar_t>;
1021:     auto beta = beta_.to<scalar_t>();
```
- EN: The main symbol in this range is `softplus_backward_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `softplus_backward_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1022-1051
```cpp
1022:     auto threshold = threshold_.to<scalar_t>();
1023:     const Vec beta_vec(beta);
1024:     const Vec threshold_vec(threshold);
1025:     const Vec one_vec(static_cast<scalar_t>(1.0));
1026:     cpu_kernel_vec(
1027:         iter,
1028:         [beta, threshold](scalar_t a, scalar_t b) -> scalar_t {
1029:           scalar_t z = std::exp(b * beta);
1030:           return (b * beta) > threshold ? a : a * z / (z + scalar_t(1.));
1031:         },
1032:         [beta_vec, one_vec, threshold_vec](Vec a, Vec b) -> Vec {
1033:           const Vec z = (b * beta_vec).exp();
1034:           return Vec::blendv(a * z / (z + one_vec), a, (b * beta_vec) > threshold_vec);
1035:         }
1036:     );
1037:   });
1038:   }
1039: }
1040:
1041: void glu_kernel(TensorIteratorBase& iter) {
1042:   if (at::isReducedFloatingType(iter.dtype())) {
1043:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "glu_cpu", [&]() {
1044:     const float float_one_val(1);
1045:     const Vectorized<float> float_one_vec(float_one_val);
1046:     cpu_kernel_vec(
1047:       iter,
1048:       [float_one_val](scalar_t a, scalar_t b) -> scalar_t {
1049:         return float(a) * (float_one_val / (float_one_val + std::exp(- float(b))));
1050:       },
1051:       [float_one_vec](Vectorized<scalar_t> a, Vectorized<scalar_t> b) -> Vectorized<scalar_t> {
```
- EN: The main symbol in this range is `glu_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `glu_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1052-1093
```cpp
1052:         auto [a0, a1] = convert_to_float<scalar_t>(a);
1053:         auto [b0, b1] = convert_to_float<scalar_t>(b);
1054:         return convert_from_float<scalar_t>(a0 * (float_one_vec / (float_one_vec + b0.neg().exp())),
1055:                                             a1 * (float_one_vec / (float_one_vec + b1.neg().exp())));
1056:       });
1057:     });
1058:   } else {
1059:     AT_DISPATCH_FLOATING_TYPES(iter.dtype(), "glu_cpu", [&] {
1060:     using Vec = Vectorized<scalar_t>;
1061:     const scalar_t one_val(1);
1062:     const Vec one_vec(one_val);
1063:     cpu_kernel_vec(
1064:       iter,
1065:       [one_val](scalar_t a, scalar_t b) -> scalar_t {
1066:         return a * (one_val / (one_val + std::exp(-b)));
1067:       },
1068:       [one_vec](Vec a, Vec b) -> Vec {
1069:         return a * (one_vec / (one_vec + b.neg().exp()));
1070:       }
1071:     );
1072:   });
1073:   }
1074: }
1075:
1076: void glu_jvp_kernel(TensorIteratorBase& iter) {
1077:   AT_DISPATCH_FLOATING_TYPES(iter.dtype(), "glu_jvp_cpu", [&] {
1078:     using Vec = Vectorized<scalar_t>;
1079:     const scalar_t one(1);
1080:     const Vec ones(one);
1081:     cpu_kernel_vec(
1082:       iter,
1083:       [one](scalar_t res, scalar_t b, scalar_t da, scalar_t db) -> scalar_t {
1084:         const auto sig_b = one / (one + std::exp(-b));
1085:         return da * sig_b + res * (db - sig_b * db);
1086:       },
1087:       [ones](Vec res, Vec b, Vec da, Vec db) -> Vec {
1088:         const auto sig_b = ones / (ones + b.neg().exp());
1089:         return da * sig_b + res * (db - sig_b * db);
1090:       }
1091:     );
1092:   });
1093: }
```
- EN: The main symbol in this range is `glu_jvp_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `glu_jvp_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1095-1130
```cpp
1095: void glu_backward_kernel(TensorIterator& iter) {
1096:   if (at::isReducedFloatingType(iter.dtype())) {
1097:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "glu_backward_cpu", [&]() {
1098:     const float float_one_val(1);
1099:     const Vectorized<float> float_one_vec(float_one_val);
1100:     cpu_kernel_vec(
1101:       iter,
1102:       [float_one_val](scalar_t a, scalar_t b, scalar_t c) -> scalar_t {
1103:         return  (float_one_val - float(a)) * float(a) * float(b) * float(c);
1104:       },
1105:       [float_one_vec](Vectorized<scalar_t> a, Vectorized<scalar_t> b, Vectorized<scalar_t> c) -> Vectorized<scalar_t> {
1106:         auto [a0, a1] = convert_to_float<scalar_t>(a);
1107:         auto [b0, b1] = convert_to_float<scalar_t>(b);
1108:         auto [c0, c1] = convert_to_float<scalar_t>(c);
1109:         a0 = (float_one_vec - a0) * a0 * b0 * c0;
1110:         a1 = (float_one_vec - a1) * a1 * b1 * c1;
1111:         return convert_from_float<scalar_t>(a0, a1);
1112:       });
1113:     });
1114:   } else {
1115:     AT_DISPATCH_FLOATING_TYPES(iter.dtype(), "glu_backward_cpu", [&] {
1116:       using Vec = Vectorized<scalar_t>;
1117:       const scalar_t one_val(1);
1118:       const Vec one_vec(one_val);
1119:       cpu_kernel_vec(
1120:         iter,
1121:         [one_val](scalar_t a, scalar_t b, scalar_t c) -> scalar_t {
1122:           return (one_val - a) * a * b * c;
1123:         },
1124:         [one_vec](Vec a, Vec b, Vec c) -> Vec {
1125:           return (one_vec - a) * a * b * c;
1126:         }
1127:       );
1128:     });
1129:   }
1130: }
```
- EN: The main symbol in this range is `glu_backward_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `glu_backward_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1132-1162
```cpp
1132: void silu_kernel(TensorIteratorBase& iter) {
1133:   if (at::isReducedFloatingType(iter.dtype())) {
1134:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "silu_cpu", [&]() {
1135:       const Vectorized<float> kOneVec(1.0f);
1136:       cpu_kernel_vec(
1137:           iter,
1138:           [](scalar_t x) -> scalar_t {
1139:             return float(x) / (1.0f + std::exp(-float(x)));
1140:           },
1141:           [kOneVec](Vectorized<scalar_t> x_vec) -> Vectorized<scalar_t> {
1142:             auto [x_vec0, x_vec1] = convert_to_float<scalar_t>(x_vec);
1143:             return convert_from_float<scalar_t>(
1144:               x_vec0 / (kOneVec + x_vec0.neg().exp()),
1145:               x_vec1 / (kOneVec + x_vec1.neg().exp()));
1146:           });
1147:     });
1148:   } else {
1149:     AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES(
1150:       iter.dtype(), "silu_cpu", [&]() {
1151:         const Vectorized<scalar_t> kOneVec(scalar_t(1));
1152:         cpu_kernel_vec(
1153:             iter,
1154:             [](scalar_t x) {
1155:               return x / (scalar_t(1) + std::exp(-x));
1156:             },
1157:             [kOneVec](Vectorized<scalar_t> x_vec) {
1158:               return x_vec / (kOneVec + x_vec.neg().exp());
1159:             });
1160:       });
1161:     }
1162: }
```
- EN: The main symbol in this range is `silu_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `silu_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1164-1205
```cpp
1164: void silu_backward_kernel(TensorIteratorBase& iter) {
1165:   if (at::isReducedFloatingType(iter.dtype())) {
1166:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "silu_backward_cpu", [&]() {
1167:     const Vectorized<float> kOneVec(1.0f);
1168:     cpu_kernel_vec(
1169:         iter,
1170:         [](scalar_t dy, scalar_t x) -> scalar_t {
1171:           const float sigmoid =
1172:               1.0f / (1.0f + std::exp(-float(x)));
1173:           return dy * sigmoid * (1.0f + x * (1.0f - sigmoid));
1174:         },
1175:         [kOneVec](Vectorized<scalar_t> dy_vec, Vectorized<scalar_t> x_vec) -> Vectorized<scalar_t> {
1176:           auto [x_vec0, x_vec1] = convert_to_float<scalar_t>(x_vec);
1177:           auto [dy_vec0, dy_vec1] = convert_to_float<scalar_t>(dy_vec);
1178:           const Vectorized<float> sigmoid0 =
1179:               kOneVec / (kOneVec + x_vec0.neg().exp());
1180:           const Vectorized<float> sigmoid1 =
1181:               kOneVec / (kOneVec + x_vec1.neg().exp());
1182:           return convert_from_float<scalar_t>(
1183:             dy_vec0 * sigmoid0 * (kOneVec + x_vec0 * (kOneVec - sigmoid0)),
1184:             dy_vec1 * sigmoid1 * (kOneVec + x_vec1 * (kOneVec - sigmoid1)));
1185:         });
1186:     });
1187:   } else {
1188:     AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES(
1189:       iter.dtype(), "silu_backward_cpu", [&]() {
1190:         const Vectorized<scalar_t> kOneVec(scalar_t(1));
1191:         cpu_kernel_vec(
1192:             iter,
1193:             [](scalar_t dy, scalar_t x) {
1194:               const scalar_t sigmoid =
1195:                   scalar_t(1) / (scalar_t(1) + std::exp(-x));
1196:               return dy * sigmoid * (scalar_t(1) + x * (scalar_t(1) - sigmoid));
1197:             },
1198:             [kOneVec](Vectorized<scalar_t> dy_vec, Vectorized<scalar_t> x_vec) {
1199:               const Vectorized<scalar_t> sigmoid =
1200:                   kOneVec / (kOneVec + x_vec.neg().exp());
1201:               return dy_vec * sigmoid * (kOneVec + x_vec * (kOneVec - sigmoid));
1202:             });
1203:       });
1204:   }
1205: }
```
- EN: The main symbol in this range is `silu_backward_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `silu_backward_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1207-1236
```cpp
1207: void mish_kernel(TensorIteratorBase& iter) {
1208:   if (at::isReducedFloatingType(iter.dtype())) {
1209:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "mish_cpu", [&]() {
1210:     cpu_kernel_vec(
1211:         iter,
1212:         [](scalar_t x) -> scalar_t{
1213:           return static_cast<scalar_t>(float(x) * std::tanh(std::log1p(std::exp(float(x)))));
1214:         },
1215:         [](Vectorized<scalar_t> x_vec) -> Vectorized<scalar_t> {
1216:           auto [x_vec0, x_vec1] = convert_to_float<scalar_t>(x_vec);
1217:           return convert_from_float<scalar_t>(
1218:             x_vec0 * x_vec0.exp().log1p().tanh(),
1219:             x_vec1 * x_vec1.exp().log1p().tanh()
1220:           );
1221:         });
1222:     });
1223:   } else {
1224:     AT_DISPATCH_FLOATING_TYPES(iter.dtype(), "mish_cpu", [&]() {
1225:         using Vec = Vectorized<scalar_t>;
1226:         cpu_kernel_vec(
1227:             iter,
1228:             [](scalar_t x) -> scalar_t{
1229:               return static_cast<scalar_t>(x * std::tanh(std::log1p(std::exp(x))));
1230:             },
1231:             [](Vec x_vec) -> Vec {
1232:               return x_vec * x_vec.exp().log1p().tanh();
1233:             });
1234:       });
1235:   }
1236: }
```
- EN: The main symbol in this range is `mish_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `mish_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1238-1266
```cpp
1238: void mish_backward_kernel(TensorIterator& iter) {
1239:   if (at::isReducedFloatingType(iter.dtype())) {
1240:     AT_DISPATCH_REDUCED_FLOATING_TYPES(iter.dtype(), "mish_backward_cpu", [&]() {
1241:     using Vec = Vectorized<float>;
1242:     const Vec kOneVec(1.0f);
1243:     cpu_kernel_vec(
1244:         iter,
1245:         [](scalar_t dy, scalar_t x) -> scalar_t {
1246:           const float sigmoid =
1247:               1.0f / (1.0f + std::exp(-float(x)));
1248:           const float tanh_softplus = std::tanh(std::log1p(std::exp(float(x))));
1249:           return dy * (tanh_softplus + x * sigmoid * (1.0f - tanh_softplus * tanh_softplus));
1250:         },
1251:         [kOneVec](Vectorized<scalar_t> dy_vec, Vectorized<scalar_t> x_vec) -> Vectorized<scalar_t> {
1252:           auto [x_vec0, x_vec1] = convert_to_float<scalar_t>(x_vec);
1253:           auto [dy_vec0, dy_vec1] = convert_to_float<scalar_t>(dy_vec);
1254:           const Vec sigmoid0 = kOneVec / (kOneVec + x_vec0.neg().exp());
1255:           const Vec sigmoid1 = kOneVec / (kOneVec + x_vec1.neg().exp());
1256:           const Vec tanh_softplus0 = x_vec0.exp().log1p().tanh();
1257:           const Vec tanh_softplus1 = x_vec1.exp().log1p().tanh();
1258:           return convert_from_float<scalar_t>(
1259:             dy_vec0 * (tanh_softplus0 + x_vec0 * sigmoid0 * (kOneVec - tanh_softplus0 * tanh_softplus0)),
1260:             dy_vec1 * (tanh_softplus1 + x_vec1 * sigmoid1 * (kOneVec - tanh_softplus1 * tanh_softplus1))
1261:           );
1262:         });
1263:     });
1264:   } else {
1265:     AT_DISPATCH_FLOATING_TYPES(iter.dtype(), "mish_backward_cpu", [&]() {
1266:         using Vec = Vectorized<scalar_t>;
```
- EN: The main symbol in this range is `mish_backward_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `mish_backward_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1267-1297
```cpp
1267:         const Vec kOneVec(scalar_t(1));
1268:         cpu_kernel_vec(
1269:             iter,
1270:             [](scalar_t dy, scalar_t x) -> scalar_t {
1271:               const scalar_t sigmoid =
1272:                   scalar_t(1) / (scalar_t(1) + std::exp(-x));
1273:               const scalar_t tanh_softplus = std::tanh(std::log1p(std::exp(x)));
1274:               return dy * (tanh_softplus + x * sigmoid * (scalar_t(1) - tanh_softplus * tanh_softplus));
1275:             },
1276:             [kOneVec](Vec dy_vec, Vec x_vec) -> Vec {
1277:               const Vec sigmoid = kOneVec / (kOneVec + x_vec.neg().exp());
1278:               const Vec tanh_softplus = x_vec.exp().log1p().tanh();
1279:               return dy_vec * (tanh_softplus + x_vec * sigmoid * (kOneVec - tanh_softplus * tanh_softplus));
1280:             });
1281:       });
1282:   }
1283: }
1284:
1285: void prelu_kernel(TensorIterator& iter) {
1286:   AT_DISPATCH_FLOATING_TYPES_AND2(kBFloat16, kHalf, iter.dtype(), "prelu_cpu", [&]() {
1287:     using Vec = Vectorized<scalar_t>;
1288:     cpu_kernel_vec(
1289:       iter,
1290:       [](scalar_t input, scalar_t weight) {
1291:         return (input > scalar_t(0)) ? input : weight * input;
1292:       },
1293:       [](Vec input, Vec weight) {
1294:         return Vec::blendv(weight * input, input, input > Vec(0));
1295:       });
1296:   });
1297: }
```
- EN: The main symbol in this range is `prelu_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `prelu_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 1299-1327
```cpp
1299: void prelu_backward_kernel(TensorIterator& iter) {
1300:   AT_DISPATCH_FLOATING_TYPES_AND2(kBFloat16, kHalf, iter.dtype(), "prelu_backward_cpu", [&]() {
1301:     cpu_kernel_multiple_outputs(iter,
1302:       [](scalar_t input, scalar_t weight, scalar_t grad) -> std::tuple<scalar_t, scalar_t> {
1303:         auto mask = input > scalar_t{0};
1304:         auto grad_input = mask ? grad : weight * grad;
1305:         auto grad_weight = mask ? scalar_t{0} : input * grad;
1306:         return {grad_input, grad_weight};
1307:       });
1308:   });
1309: }
1310:
1311: } // namespace
1312:
1313:
1314: REGISTER_DISPATCH(hardsigmoid_stub, &hardsigmoid_kernel)
1315: REGISTER_DISPATCH(hardsigmoid_backward_stub, &hardsigmoid_backward_kernel)
1316: REGISTER_DISPATCH(threshold_stub, &threshold_kernel)
1317: REGISTER_DISPATCH(leaky_relu_stub, &leaky_relu_kernel)
1318: REGISTER_DISPATCH(leaky_relu_backward_stub, &leaky_relu_backward_kernel)
1319: REGISTER_DISPATCH(prelu_stub, &prelu_kernel)
1320: REGISTER_DISPATCH(prelu_backward_stub, &prelu_backward_kernel)
1321: REGISTER_DISPATCH(hardtanh_backward_stub, &hardtanh_backward_kernel)
1322: REGISTER_DISPATCH(hardshrink_stub, &hardshrink_kernel)
1323: REGISTER_DISPATCH(softshrink_stub, &softshrink_kernel)
1324: REGISTER_DISPATCH(shrink_backward_stub, &shrink_backward_kernel)
1325:
1326: ALSO_REGISTER_AVX512_DISPATCH(log_sigmoid_cpu_stub, &log_sigmoid_cpu_kernel)
1327: ALSO_REGISTER_AVX512_DISPATCH(log_sigmoid_backward_stub, &log_sigmoid_backward_cpu_kernel)
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `prelu_backward_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `prelu_backward_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 1328-1344
```cpp
1328: ALSO_REGISTER_AVX512_DISPATCH(glu_stub, &glu_kernel)
1329: ALSO_REGISTER_AVX512_DISPATCH(glu_backward_stub, &glu_backward_kernel)
1330: ALSO_REGISTER_AVX512_DISPATCH(glu_jvp_stub, &glu_jvp_kernel)
1331: ALSO_REGISTER_AVX512_DISPATCH(elu_stub, &elu_kernel)
1332: ALSO_REGISTER_AVX512_DISPATCH(elu_backward_stub, &elu_backward_kernel)
1333: ALSO_REGISTER_AVX512_DISPATCH(GeluKernel, &GeluKernelImpl)
1334: ALSO_REGISTER_AVX512_DISPATCH(GeluBackwardKernel, &GeluBackwardKernelImpl)
1335: ALSO_REGISTER_AVX512_DISPATCH(hardswish_stub, &hardswish_kernel)
1336: ALSO_REGISTER_AVX512_DISPATCH(hardswish_backward_stub, &hardswish_backward_kernel)
1337: ALSO_REGISTER_AVX512_DISPATCH(softplus_stub, &softplus_kernel)
1338: ALSO_REGISTER_AVX512_DISPATCH(softplus_backward_stub, &softplus_backward_kernel)
1339: ALSO_REGISTER_AVX512_DISPATCH(silu_stub, &silu_kernel)
1340: ALSO_REGISTER_AVX512_DISPATCH(silu_backward_stub, &silu_backward_kernel)
1341: ALSO_REGISTER_AVX512_DISPATCH(mish_stub, &mish_kernel)
1342: ALSO_REGISTER_AVX512_DISPATCH(mish_backward_stub, &mish_backward_kernel)
1343:
1344: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Dispatcher registration / 调度器注册

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/native/Activation.h`, `ATen/Dispatch.h`, `ATen/OpMathType.h`, `ATen/core/TensorBase.h`, `ATen/cpu/vec/functional.h`, `ATen/cpu/vec/vec.h`, `ATen/native/TensorIterator.h`, `ATen/native/cpu/Elu.h`, `ATen/native/cpu/Gelu.h`, `ATen/native/cpu/Loops.h`
- c10 headers / c10 头文件: `c10/core/Scalar.h`
- Standard or third-party headers / 标准库或第三方头文件: `cmath`, `functional`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `TensorIteratorBase`, `TensorBase`, `cpu_kernel`, `cpu_kernel_vec`, `Vectorized`, `parallel_for`, `REGISTER_DISPATCH`, `AT_DISPATCH_FLOATING_TYPES`, `AT_DISPATCH_ALL_TYPES`
