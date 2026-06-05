# DistributionTemplates.h — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/DistributionTemplates.h`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares support for Distribution Templates in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了 Distribution Templates 的相关支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-22
```cpp
 1: #pragma once
 2:
 3: #include <ATen/CPUApplyUtils.h>
 4: #include <ATen/Dispatch.h>
 5: #include <ATen/Dispatch_v2.h>
 6: #include <ATen/ExpandBase.h>
 7: #include <ATen/OpMathType.h>
 8: #include <ATen/core/DistributionsHelper.h>
 9: #include <ATen/native/TensorIterator.h>
10: #include <ATen/native/cpu/Loops.h>
11: #include <mutex>
12:
13: #ifdef CPU_CAPABILITY_AVX2
14: #include <ATen/native/cpu/avx_mathfun.h>
15: #include <c10/util/irange.h>
16: #endif
17:
18:
19:
20:
21: namespace at::native::templates::cpu {
22: namespace {
```
- EN: This range pulls in required headers, including `ATen/CPUApplyUtils.h`, `ATen/Dispatch.h`, `ATen/Dispatch_v2.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/CPUApplyUtils.h`, `ATen/Dispatch.h`, `ATen/Dispatch_v2.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 24-42
```cpp
24: // ==================================================== Random ========================================================
25:
26: template<typename RNG>
27: void random_from_to_kernel(TensorIteratorBase& iter, uint64_t range, int64_t base, RNG generator) {
28:   AT_DISPATCH_V2(iter.dtype(), "random_from_to_kernel_cpu", AT_WRAP([&] {
29:     std::lock_guard<std::mutex> lock(generator->mutex_);
30:     cpu_serial_kernel(iter, [range, base, generator]() -> scalar_t {
31:       uniform_int_from_to_distribution<scalar_t> random(range, base);
32:       return random(generator);
33:     });
34:   }), kBool, kHalf, kBFloat16, AT_EXPAND(AT_ALL_TYPES), AT_EXPAND(AT_BAREBONES_UNSIGNED_TYPES));
35: }
36:
37: // This is the special kernel to handle single specific case:
38: // from(inclusive) = std::numeric_limits<int64_t>::lowest()
39: // to(exclusive) = None (= std::numeric_limits<int64_t>::max() + 1)
40: template<typename RNG>
41: void random_full_64_bits_range_kernel(TensorIteratorBase& iter, RNG generator) {
42:   AT_DISPATCH_ALL_TYPES_AND(at::ScalarType::BFloat16, iter.dtype(), "random_full_64_bits_range_kernel_cpu", [&] {
```
- EN: The main symbol in this range is `random_from_to_kernel`, `from`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `random_from_to_kernel`, `from`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 43-66
```cpp
43:     if constexpr (std::is_same_v<scalar_t, int64_t> ||
44:         std::is_same_v<scalar_t, double> ||
45:         std::is_same_v<scalar_t, float> ||
46:         std::is_same_v<scalar_t, at::BFloat16>) {
47:       std::lock_guard<std::mutex> lock(generator->mutex_);
48:       cpu_serial_kernel(iter, [generator]() -> scalar_t {
49:         uniform_int_full_range_distribution<scalar_t> random;
50:         return random(generator);
51:       });
52:     } else {
53:       TORCH_CHECK(false, "random_full_64_bits_range_kernel_cpu handles only int64, double, float and bfloat16");
54:     }
55:   });
56: }
57:
58: template<typename RNG>
59: struct RandomFromToKernel {
60:   void operator()(TensorIteratorBase& iter, uint64_t range, int64_t base, std::optional<Generator> gen) {
61:     random_from_to_kernel(iter, range, base, check_generator<RNG>(gen));
62:   }
63:   void operator()(TensorIteratorBase& iter, std::optional<Generator> gen) {
64:     random_full_64_bits_range_kernel(iter, check_generator<RNG>(gen));
65:   }
66: };
```
- EN: The main symbol in this range is `constexpr`, `cpu_serial_kernel`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `constexpr`, `cpu_serial_kernel`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 68-86
```cpp
68: template<typename RNG>
69: void random_kernel(TensorIteratorBase& iter, RNG generator) {
70:   std::lock_guard<std::mutex> lock(generator->mutex_);
71:   AT_DISPATCH_ALL_TYPES_AND3(at::ScalarType::Half, at::ScalarType::BFloat16, at::ScalarType::Bool, iter.dtype(), "random_kernel_cpu", [&] {
72:     cpu_serial_kernel(iter, [generator]() -> scalar_t {
73:       uniform_int_distribution<scalar_t> random;
74:       return random(generator);
75:     });
76:   });
77: }
78:
79: template<typename RNG>
80: struct RandomKernel {
81:   void operator()(TensorIteratorBase& iter, std::optional<Generator> gen) {
82:     random_kernel(iter, check_generator<RNG>(gen));
83:   }
84: };
85:
86: // ==================================================== Normal ========================================================
```
- EN: The main symbol in this range is `random_kernel`, `cpu_serial_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `random_kernel`, `cpu_serial_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 88-109
```cpp
 88: // Box-Muller transform on 16 elements (8 pairs of uniforms).
 89: // Primary template is scalar; float specialization uses SIMD when available.
 90: // Constructed once with mean/std so constants are not rebuilt per call.
 91: template <typename opmath_t, bool enable = true>
 92: struct NormalFill16 {
 93:   opmath_t mean_;
 94:   opmath_t std_;
 95:
 96:   NormalFill16(opmath_t mean, opmath_t std)
 97:     : mean_(mean), std_(std) {}
 98:
 99:   void operator()(opmath_t* data) const {
100:     for (const auto j : c10::irange(8)) {
101:       const opmath_t u1 = 1 - data[j]; // [0, 1) -> (0, 1] for log.
102:       const opmath_t u2 = data[j + 8];
103:       const opmath_t radius = std::sqrt(-2 * std::log(u1));
104:       const opmath_t theta = 2.0f * c10::pi<double> * u2;
105:       data[j] = std::fma(radius * std::cos(theta), std_, mean_);
106:       data[j + 8] = std::fma(radius * std::sin(theta), std_, mean_);
107:     }
108:   }
109: };
```
- EN: The main symbol in this range is `NormalFill16`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Normalization-related state, scaling, or statistics are handled here.
- CN: 这一段的主要符号是 `NormalFill16`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里处理归一化相关的状态、缩放或统计量。

### Lines 111-137
```cpp
111: #if defined(CPU_CAPABILITY_AVX2)
112:
113: template <>
114: struct NormalFill16<float, true> {
115:   __m256 mean_;
116:   __m256 std_;
117:   __m256 two_pi_ = _mm256_set1_ps(2.0f * c10::pi<double>);
118:   __m256 one_ = _mm256_set1_ps(1.0f);
119:   __m256 minus_two_ = _mm256_set1_ps(-2.0f);
120:
121:   NormalFill16(float mean, float std)
122:     : mean_(_mm256_set1_ps(mean)), std_(_mm256_set1_ps(std)) {}
123:
124:   void operator()(float* data) const {
125:     const __m256 u1 = _mm256_sub_ps(one_, _mm256_loadu_ps(data));
126:     const __m256 u2 = _mm256_loadu_ps(data + 8);
127:     // sincos256_ps and log256_ps are from avx_mathfun.h
128:     const __m256 radius = _mm256_sqrt_ps(_mm256_mul_ps(minus_two_, log256_ps(u1)));
129:     const __m256 theta = _mm256_mul_ps(two_pi_, u2);
130:     __m256 sintheta, costheta;
131:     sincos256_ps(theta, &sintheta, &costheta);
132:     const __m256 n1 = _mm256_mul_ps(radius, costheta);
133:     const __m256 n2 = _mm256_mul_ps(radius, sintheta);
134:     _mm256_storeu_ps(data, _mm256_fmadd_ps(n1, std_, mean_));
135:     _mm256_storeu_ps(data + 8, _mm256_fmadd_ps(n2, std_, mean_));
136:   }
137: };
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `NormalFill16`, which contributes directly to this file's operator logic. Normalization-related state, scaling, or statistics are handled here.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `NormalFill16`，它们直接构成本文件的算子逻辑。 这里处理归一化相关的状态、缩放或统计量。

### Lines 139-163
```cpp
139: #elif defined(__VSX__) || defined(CPU_CAPABILITY_VSX)
140:
141: template <>
142: struct NormalFill16<float, Vectorized<float>::size() == 8> {
143:   using Vec = Vectorized<float>;
144:   Vec mean_;
145:   Vec std_;
146:   Vec two_pi_ = Vec(2.0f * c10::pi<double>);
147:   Vec one_ = Vec(1.0f);
148:   Vec minus_two_ = Vec(-2.0f);
149:
150:   NormalFill16(float mean, float std)
151:     : mean_(mean), std_(std) {}
152:
153:   void operator()(float* data) const {
154:     Vec u1 = one_ - Vec::loadu(data);
155:     Vec u2 = Vec::loadu(data + 8);
156:     Vec radius = (minus_two_ * u1.log()).sqrt();
157:     Vec theta = two_pi_ * u2;
158:     Vec output1 = radius * theta.cos() * std_ + mean_;
159:     Vec output2 = radius * theta.sin() * std_ + mean_;
160:     output1.store(data);
161:     output2.store(data + 8);
162:   }
163: };
```
- EN: The main symbol in this range is `NormalFill16`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Normalization-related state, scaling, or statistics are handled here.
- CN: 这一段的主要符号是 `NormalFill16`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 这里处理归一化相关的状态、缩放或统计量。

### Lines 165-183
```cpp
165: #endif
166:
167: template <typename scalar_t, typename RNG>
168: void normal_fill(const TensorBase &self, double mean, double std, RNG generator) {
169:   using opmath_t = at::opmath_type<scalar_t>;
170:   scalar_t *data = self.data_ptr<scalar_t>();
171:   auto size = self.numel();
172:   std::lock_guard<std::mutex> lock(generator->mutex_);
173:   at::uniform_real_distribution<opmath_t> uniform(0, 1);
174:   NormalFill16<opmath_t> normal_fill_16(
175:       static_cast<opmath_t>(mean), static_cast<opmath_t>(std));
176:
177:   if constexpr (std::is_same_v<scalar_t, opmath_t>) {
178:     // float/double: generate uniform samples directly into the output buffer,
179:     // then apply Box-Muller in-place.
180:     for (const auto i : c10::irange(size)) {
181:       data[i] = uniform(generator);
182:     }
183:     for (int64_t i = 0; i < size - 15; i += 16) {
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `normal_fill`, `constexpr`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `normal_fill`, `constexpr`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 184-203
```cpp
184:       normal_fill_16(data + i);
185:     }
186:     // Recompute the last 16 values.
187:     if (size % 16 != 0) {
188:       data = data + size - 16;
189:       for (const auto i : c10::irange(16)) {
190:         data[i] = uniform(generator);
191:       }
192:       normal_fill_16(data);
193:     }
194:   } else {
195:     // bf16/fp16: generate in opmath_t precision using a stack buffer,
196:     // apply Box-Muller, then cast down to scalar_t.
197:     opmath_t buf[16];
198:     for (int64_t i = 0; i < size - 15; i += 16) {
199:       for (const auto j : c10::irange(16)) {
200:         buf[j] = uniform(generator);
201:       }
202:       normal_fill_16(buf);
203:       for (const auto j : c10::irange(16)) {
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Normalization-related state, scaling, or statistics are handled here.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里处理归一化相关的状态、缩放或统计量。

### Lines 204-223
```cpp
204:         data[i + j] = static_cast<scalar_t>(buf[j]);
205:       }
206:     }
207:     // Recompute the last 16 values.
208:     if (size % 16 != 0) {
209:       int64_t offset = size - 16;
210:       for (const auto j : c10::irange(16)) {
211:         buf[j] = uniform(generator);
212:       }
213:       normal_fill_16(buf);
214:       for (const auto j : c10::irange(16)) {
215:         data[offset + j] = static_cast<scalar_t>(buf[j]);
216:       }
217:     }
218:   }
219: }
220:
221: template<typename RNG>
222: void normal_kernel(const TensorBase &self, double mean, double std, RNG generator) {
223:   auto size = self.numel();
```
- EN: The main symbol in this range is `normal_kernel`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `normal_kernel`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 224-243
```cpp
224:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, self.scalar_type(), "normal_kernel_cpu", [&] {
225:     if (size >= 16 && self.is_contiguous()) {
226:       normal_fill<scalar_t>(self, mean, std, generator);
227:     } else {
228:       auto iter = TensorIterator::borrowing_nullary_op(self);
229:       std::lock_guard<std::mutex> lock(generator->mutex_);
230:       cpu_serial_kernel(iter, [mean, std, generator]() -> scalar_t {
231:         at::normal_distribution<double> normal(mean, std);
232:         return static_cast<scalar_t>(normal(generator));
233:       });
234:     }
235:   });
236: }
237:
238: template<typename RNG>
239: struct NormalKernel {
240:   void operator()(Tensor& self, double mean, double std, std::optional<Generator> gen) {
241:     normal_kernel(self, mean, std, check_generator<RNG>(gen));
242:   }
243: };
```
- EN: The main symbol in this range is `cpu_serial_kernel`, `normal_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `cpu_serial_kernel`, `normal_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 245-263
```cpp
245: // ==================================================== Uniform =======================================================
246:
247: template<typename RNG>
248: void uniform_kernel(TensorIteratorBase& iter, double from_, double to_, RNG generator) {
249:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, iter.dtype(), "uniform_kernel_cpu", [&]() {
250:     std::lock_guard<std::mutex> lock(generator->mutex_);
251:     using opmath_t = at::opmath_type<scalar_t>;
252:     auto from = static_cast<opmath_t>(from_);
253:     auto to = static_cast<opmath_t>(to_);
254:     auto to_scalar = static_cast<scalar_t>(to_);
255:     auto from_scalar = static_cast<scalar_t>(from_);
256:     at::uniform_real_distribution<opmath_t> uniform(from, to);
257:     cpu_serial_kernel(iter, [&uniform, generator, to_scalar, from_scalar]() -> scalar_t {
258:       auto value = static_cast<scalar_t>(uniform(generator));
259:       // Clamp if the float→scalar_t cast rounded up to the upper bound
260:       return value == to_scalar ? from_scalar : value;
261:     });
262:   });
263: }
```
- EN: The main symbol in this range is `uniform_kernel`, `cpu_serial_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `uniform_kernel`, `cpu_serial_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 265-283
```cpp
265: template<typename RNG>
266: struct UniformKernel {
267:   void operator()(TensorIteratorBase& iter, double from, double to, std::optional<Generator> gen) {
268:     uniform_kernel(iter, from, to, check_generator<RNG>(gen));
269:   }
270: };
271:
272: // ==================================================== Cauchy ========================================================
273:
274: template<typename RNG>
275: void cauchy_kernel(TensorIteratorBase& iter, double median, double sigma, RNG generator) {
276:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, iter.dtype(), "cauchy_cpu", [&]() {
277:     std::lock_guard<std::mutex> lock(generator->mutex_);
278:     at::cauchy_distribution<double> cauchy(median, sigma);
279:     cpu_serial_kernel(iter, [&cauchy, generator]() -> scalar_t {
280:       return static_cast<scalar_t>(cauchy(generator));
281:     });
282:   });
283: }
```
- EN: The main symbol in this range is `cauchy_kernel`, `uniform_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `cauchy_kernel`, `uniform_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 285-303
```cpp
285: template<typename RNG>
286: struct CauchyKernel {
287:   void operator()(TensorIteratorBase& iter, double median, double sigma, std::optional<Generator> gen) {
288:     cauchy_kernel(iter, median, sigma, check_generator<RNG>(gen));
289:   }
290: };
291:
292: // ================================================== LogNormal =======================================================
293:
294: template<typename RNG>
295: void log_normal_kernel(TensorIteratorBase& iter, double mean, double std, RNG generator) {
296:   AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, iter.dtype(), "log_normal_cpu", [&]() {
297:     std::lock_guard<std::mutex> lock(generator->mutex_);
298:     at::lognormal_distribution<double> logNormal(mean, std);
299:     cpu_serial_kernel(iter, [&logNormal, generator]() -> scalar_t {
300:       return static_cast<scalar_t>(logNormal(generator));
301:     });
302:   });
303: }
```
- EN: The main symbol in this range is `log_normal_kernel`, `cauchy_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `log_normal_kernel`, `cauchy_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 305-323
```cpp
305: template<typename RNG>
306: struct LogNormalKernel {
307:   void operator()(TensorIteratorBase& iter, double mean, double std, std::optional<Generator> gen) {
308:     log_normal_kernel(iter, mean, std, check_generator<RNG>(gen));
309:   }
310: };
311:
312: // =================================================== Geometric ======================================================
313:
314: template<typename RNG>
315: void geometric_kernel(TensorIteratorBase& iter, double p, RNG generator) {
316:   AT_DISPATCH_ALL_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, iter.dtype(), "geometric_cpu", [&]() {
317:     std::lock_guard<std::mutex> lock(generator->mutex_);
318:     at::geometric_distribution<double> geometric(p);
319:     cpu_serial_kernel(iter, [&geometric, generator]() -> scalar_t {
320:       return static_cast<scalar_t>(geometric(generator));
321:     });
322:   });
323: }
```
- EN: The main symbol in this range is `geometric_kernel`, `log_normal_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `geometric_kernel`, `log_normal_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 325-344
```cpp
325: template<typename RNG>
326: struct GeometricKernel {
327:   void operator()(TensorIteratorBase& iter, double p, std::optional<Generator> gen) {
328:     geometric_kernel(iter, p, check_generator<RNG>(gen));
329:   }
330: };
331:
332: // ================================================== Exponential =====================================================
333:
334: template<typename RNG>
335: void exponential_kernel(TensorIteratorBase& iter, double lambda, RNG generator) {
336:   TORCH_CHECK(isFloatingType(iter.dtype()), "Exponential distribution is a continuous probability distribution. dtype must be a floating point but you specified ", iter.dtype());
337:   AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, iter.dtype(), "exponential_cpu", [&]() {
338:     std::lock_guard<std::mutex> lock(generator->mutex_);
339:     at::exponential_distribution<double> exponential(lambda);
340:     cpu_serial_kernel(iter, [&exponential, generator]() -> scalar_t {
341:       return static_cast<scalar_t>(exponential(generator));
342:     });
343:   });
344: }
```
- EN: The main symbol in this range is `exponential_kernel`, `geometric_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `exponential_kernel`, `geometric_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 346-364
```cpp
346: template<typename RNG>
347: struct ExponentialKernel {
348:   void operator()(TensorIteratorBase& iter, double lambda, std::optional<Generator> gen) {
349:     exponential_kernel(iter, lambda, check_generator<RNG>(gen));
350:   }
351: };
352:
353: // ================================================== Bernoulli =======================================================
354:
355: template<typename RNG>
356: void bernoulli_kernel(const TensorBase &self, const TensorBase &p_, RNG generator) {
357:   AT_DISPATCH_ALL_TYPES_AND3(at::ScalarType::Bool, at::ScalarType::BFloat16, at::ScalarType::Half,
358:   self.scalar_type(), "bernoulli_tensor_cpu_self_", [&] {
359:     // See Note [Acquire lock when using random generators]
360:     std::lock_guard<std::mutex> lock(generator->mutex_);
361:     using self_t = scalar_t;
362:     auto p_cpu = p_.to(kCPU);
363:     auto p = expand_inplace(self, p_cpu);
364:     auto iter = TensorIteratorConfig()
```
- EN: The main symbol in this range is `bernoulli_kernel`, `exponential_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `bernoulli_kernel`, `exponential_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 365-385
```cpp
365:         .add_output(self)
366:         .add_const_input(*p)
367:         .check_all_same_dtype(false)
368:         .build();
369:     if (p->scalar_type() == kDouble) {
370:       cpu_serial_kernel(iter, [&](const double p_val) -> self_t {
371:         at::bernoulli_distribution<double> bernoulli(p_val);
372:         return static_cast<self_t>(bernoulli(generator));
373:       });
374:     } else {
375:       AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::BFloat16, at::ScalarType::Half,
376:       p->scalar_type(), "bernoulli_tensor_cpu_p_", [&] {
377:         using p_t = scalar_t;
378:         cpu_serial_kernel(iter, [&](const p_t p_val) -> self_t {
379:           at::bernoulli_distribution<float> bernoulli(p_val);
380:           return static_cast<self_t>(bernoulli(generator));
381:         });
382:       });
383:     }
384:   });
385: }
```
- EN: The main symbol in this range is `cpu_serial_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `cpu_serial_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 387-409
```cpp
387: template<typename RNG>
388: void bernoulli_kernel(const TensorBase &self, double p, RNG generator) {
389:   AT_DISPATCH_ALL_TYPES_AND3(at::ScalarType::Bool, at::ScalarType::BFloat16, at::ScalarType::Half,
390:   self.scalar_type(), "bernoulli_scalar_cpu_", [&] {
391:     // See Note [Acquire lock when using random generators]
392:     std::lock_guard<std::mutex> lock(generator->mutex_);
393:     auto iter = TensorIterator::borrowing_nullary_op(self);
394:     cpu_serial_kernel(iter, [p, generator]() -> scalar_t {
395:       at::bernoulli_distribution<double> bernoulli(p);
396:       return static_cast<scalar_t>(bernoulli(generator));
397:     });
398:   });
399: }
400:
401: template<typename RNG>
402: struct BernoulliKernel {
403:   void operator()(const TensorBase &self, double p, std::optional<Generator> gen) {
404:     bernoulli_kernel(self, p, check_generator<RNG>(gen));
405:   }
406:   void operator()(const TensorBase &self, const TensorBase &p_, std::optional<Generator> gen) {
407:     bernoulli_kernel(self, p_, check_generator<RNG>(gen));
408:   }
409: };
```
- EN: The main symbol in this range is `bernoulli_kernel`, `cpu_serial_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `bernoulli_kernel`, `cpu_serial_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 411-411
```cpp
411: }}
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- SIMD vectorization / SIMD 向量化
- Normalization statistics / 归一化统计
- Low-level memory access / 底层内存访问
- Scalar and dtype abstractions / 标量与数据类型抽象

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/CPUApplyUtils.h`, `ATen/Dispatch.h`, `ATen/Dispatch_v2.h`, `ATen/ExpandBase.h`, `ATen/OpMathType.h`, `ATen/core/DistributionsHelper.h`, `ATen/native/TensorIterator.h`, `ATen/native/cpu/Loops.h`, `ATen/native/cpu/avx_mathfun.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `mutex`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `TensorIteratorBase`, `TensorBase`, `Vectorized`, `AT_DISPATCH_FLOATING_TYPES`, `AT_DISPATCH_ALL_TYPES`, `Scalar`, `ScalarType`
