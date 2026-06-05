# DistanceOpsKernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/DistanceOpsKernel.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares support for Distance Ops Kernel in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了 Distance Ops Kernel 的相关支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/native/Distance.h>
 3:
 4: #include <algorithm>
 5:
 6: #include <ATen/core/Tensor.h>
 7: #include <ATen/Dispatch.h>
 8: #include <ATen/Parallel.h>
 9: #include <ATen/TensorIterator.h>
10: #include <ATen/cpu/vec/functional.h>
11: #include <c10/util/irange.h>
12:
13: namespace at::native {
14: namespace {
15:
16: template<typename scalar_t>
17: struct Dist {
18:   using Vec = vec::Vectorized<scalar_t>;
19:
20:   // Depending on the value of the pnorm, there are specific implementations
```
- EN: This range pulls in required headers, including `ATen/native/Distance.h`, `algorithm`, `ATen/core/Tensor.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/native/Distance.h`, `algorithm`, `ATen/core/Tensor.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 21-42
```cpp
21:   // that are much faster than std::pow(std::abs(a - b), p), but have the same
22:   // standard loop code for how to process the input vector. To reuse the main
23:   // outside loop while still guaranteeing that the compiler inlines every
24:   // different function on p, we break the inner norm logic into structs with
25:   // static functions that represent what's done differently, and template the
26:   // outer loop on those structs.
27:   //
28:   // The four functions are:
29:   //     map :      This tells how to modify (a - b) to form the component that
30:   //                gets summed.
31:   //     red :      This tells how to sum the result of map up. This is
32:   //                separate because the inf norm actually uses max instead of
33:   //                sum.
34:   //     finish :   This tells what to do with the aggregated value to compute
35:   //                the norm. Generally this is the result of val ^ (1 / p).
36:   //     backward : This is the gradient for that norm. Arguments are pretty
37:   //                self explanatory.
38:   //
39:   // There are a few cases where these aren't used. The 0 norm has no backward,
40:   // because it's always 0, so that's shortcircuited earlier. There's a special
41:   // implementation of the general backward pass when p is less than two, so
42:   // there's a struct with only a backward pass for this case.
```
- EN: The main symbol in this range is `with`, which contributes directly to this file's operator logic. Normalization-related state, scaling, or statistics are handled here.
- CN: 这一段的主要符号是 `with`，它们直接构成本文件的算子逻辑。 这里处理归一化相关的状态、缩放或统计量。

### Lines 44-65
```cpp
44:   // TODO This is an inefficient way to compite sign, and can be much faster
45:   // using native SSE instructions that should be added to Vectorized.
46:   static inline Vec sign(Vec val) {
47:     return vec::minimum(vec::maximum(Vec(0), val.ceil()), Vec(1)) +
48:       vec::minimum(vec::maximum(Vec(-1), val.floor()), Vec(0));
49:   }
50:
51:   static inline Vec abs(Vec val) {
52:     return val.abs();
53:   }
54:
55:   static inline scalar_t abs(scalar_t val) {
56:     return std::abs(val);
57:   }
58:
59:   static inline Vec ceil(Vec val) {
60:     return val.ceil();
61:   }
62:
63:   static inline scalar_t ceil(scalar_t val) {
64:     return std::ceil(val);
65:   }
```
- EN: The main symbol in this range is `sign`, `ceil`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `sign`, `ceil`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 67-85
```cpp
67:   static inline Vec min(Vec val, scalar_t other) {
68:     return vec::minimum(val, Vec(other));
69:   }
70:
71:   static inline scalar_t min(scalar_t val, scalar_t other) {
72:     return std::min(val, other);
73:   }
74:
75:   static inline Vec max(Vec val, Vec other) {
76:     return vec::maximum(val, other);
77:   }
78:
79:   static inline scalar_t max(scalar_t val, scalar_t other) {
80:     return std::max(val, other);
81:   }
82:
83:   static inline Vec pow(Vec val, Vec p) {
84:     return val.pow(p);
85:   }
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 87-106
```cpp
 87:   static inline scalar_t pow(scalar_t val, scalar_t p) {
 88:     return std::pow(val, p);
 89:   }
 90:
 91:   // Zero norm
 92:   template<typename data_t>
 93:   struct zdist_calc {
 94:     static inline data_t map(const data_t& diff, const data_t& p) { return min(ceil(abs(diff)), 1); }
 95:     static inline data_t red(const data_t& agg, const data_t& up) { return agg + up; }
 96:     static inline scalar_t finish(const scalar_t agg, const scalar_t /*p*/) { return agg; }
 97:   };
 98:
 99:   // One norm
100:   template<typename data_t>
101:   struct odist_calc {
102:     static inline data_t map(const data_t& diff, const data_t& p) { return diff; }
103:     static inline data_t red(const data_t& agg, const data_t& up) { return agg + up; }
104:     static inline scalar_t finish(const scalar_t agg, const scalar_t /*p*/) { return agg; }
105:     static inline Vec backward(const Vec& diff, const scalar_t grad, const scalar_t /*dist*/, const Vec& /*p*/) { return Vec(grad) * sign(diff); }
106:   };
```
- EN: The main symbol in this range is `map`, `red`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Normalization-related state, scaling, or statistics are handled here.
- CN: 这一段的主要符号是 `map`, `red`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这里处理归一化相关的状态、缩放或统计量。

### Lines 108-134
```cpp
108:   // Special general pnorm derivative if p is less than two
109:   struct lttdist_calc {
110:     static inline Vec backward(const Vec& diff, const scalar_t grad, const scalar_t dist, const Vec& p) {
111:       Vec result = (dist == 0.0) ? Vec(0) : (sign(diff) * diff.abs().pow(p - Vec(1)) * Vec(grad) / Vec(dist).pow(p - Vec(1)));
112:       result = Vec::blendv(result, Vec(0), (diff == Vec(0)) & (p < Vec(1)));
113:       return result;
114:     }
115:   };
116:
117:   // Two norm
118:   template<typename data_t>
119:   struct tdist_calc {
120:     // TODO This can probably use fused add multiply to get better perf
121:     static inline data_t map(const data_t& diff, const data_t& p) { return diff * diff; }
122:     static inline data_t red(const data_t& agg, const data_t& up) { return agg + up; }
123:     static inline scalar_t finish(const scalar_t agg, const scalar_t p) { return std::sqrt(agg); }
124:     static inline Vec backward(const Vec& diff, const scalar_t grad, const scalar_t dist, const Vec& p) { return dist == 0.0 ? Vec(0) : Vec(grad) * diff / Vec(dist); }
125:   };
126:
127:   // General p norm
128:   template<typename data_t>
129:   struct pdist_calc {
130:     static inline data_t map(const data_t& diff, const data_t& p) { return pow(diff, p); }
131:     static inline data_t red(const data_t& agg, const data_t& up) { return agg + up; }
132:     static inline scalar_t finish(const scalar_t agg, const scalar_t p) { return std::pow(agg, 1.0 / p); }
133:     static inline Vec backward(const Vec& diff, const scalar_t grad, const scalar_t dist, const Vec& p) { return dist == 0.0 ? Vec(0) : diff * diff.abs().pow(p - Vec(2)) * Vec(grad) / Vec(dist).pow(p - Vec(1)); }
134:   };
```
- EN: The main symbol in this range is `backward`, `map`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Normalization-related state, scaling, or statistics are handled here.
- CN: 这一段的主要符号是 `backward`, `map`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这里处理归一化相关的状态、缩放或统计量。

### Lines 136-155
```cpp
136:   // Inf norm
137:   template<typename data_t>
138:   struct idist_calc {
139:     static inline data_t map(const data_t& diff, const data_t& p) { return diff; }
140:     static inline data_t red(const data_t& agg, const data_t& up) { return max(agg, up); }
141:     static inline scalar_t finish(const scalar_t agg, const scalar_t p) { return agg; }
142:     // TODO This backward pass uses a very complex expression to compute (diff
143:     // == dist) that could be much faster if using SSE instructions.
144:     static inline Vec backward(const Vec& diff, const scalar_t grad, const scalar_t dist, const Vec& p) { return Vec(grad) * sign(diff) * (Vec(1) - vec::minimum(Vec(1), (diff.abs() - Vec(dist)).abs().ceil())); }
145:   };
146:
147:   template <typename F>
148:   static void run_parallel_pdist(Tensor& result, const Tensor& self, const scalar_t p) {
149:     const scalar_t * const self_start = self.const_data_ptr<scalar_t>();
150:     const scalar_t * const self_end = self_start + self.numel();
151:     int64_t n = self.size(0);
152:     int64_t m = self.size(1);
153:
154:     scalar_t * const res_start = result.data_ptr<scalar_t>();
155:     int64_t combs = result.numel(); // n * (n - 1) / 2
```
- EN: The main symbol in this range is `map`, `red`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `map`, `red`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 157-176
```cpp
157:     // We conceptually iterate over tuples of (i, j, k) where i is the first
158:     // vector from the input, j is the second, and k is the result index. This
159:     // parallelizes over the range of k and infers what i and j are from the
160:     // value of k.
161:     parallel_for(0, combs, internal::GRAIN_SIZE / (16 * m), [p, self_start, self_end, n, m, res_start](int64_t k, int64_t end) {
162:       const Vec pvec(p);
163:       double n2 = static_cast<double>(n) - .5;
164:       // The -1 accounts for floating point truncation issues
165:       int64_t i = static_cast<int64_t>((n2 - std::sqrt(n2 * n2 - 2.0 * static_cast<double>(k) - 1.0)));
166:       int64_t j = k - n * i + i * (i + 1) / 2 + i + 1;
167:
168:       const scalar_t * self_i = self_start + i * m;
169:       const scalar_t * self_j = self_start + j * m;
170:       scalar_t * res = res_start + k;
171:       const scalar_t * const res_end = res_start + end;
172:
173:       while (res != res_end) {
174:         *res = F::finish(vec::map2_reduce_all<scalar_t>(
175:           [&pvec](Vec a, Vec b) { return F::map((a - b).abs(), pvec); },
176:           F::red, self_i, self_j, m), p);
```
- EN: The main symbol in this range is `of`, `finish`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `of`, `finish`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 178-201
```cpp
178:         res += 1;
179:         self_j += m;
180:         if (self_j == self_end) {
181:           self_i += m;
182:           self_j = self_i + m;
183:         }
184:       }
185:     });
186:   }
187:
188:   // Assumes self is nonempty, contiguous, and 2D
189:   static void apply_pdist(Tensor& result, const Tensor& self, const scalar_t p) {
190:     if (p == 0.0) {
191:       run_parallel_pdist<zdist_calc<Vec>>(result, self, p);
192:     } else if (p == 1.0) {
193:       run_parallel_pdist<odist_calc<Vec>>(result, self, p);
194:     } else if (p == 2.0) {
195:       run_parallel_pdist<tdist_calc<Vec>>(result, self, p);
196:     } else if (std::isinf(p)) {
197:       run_parallel_pdist<idist_calc<Vec>>(result, self, p);
198:     } else {
199:       run_parallel_pdist<pdist_calc<Vec>>(result, self, p);
200:     }
201:   }
```
- EN: The main symbol in this range is `apply_pdist`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `apply_pdist`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 203-225
```cpp
203:   template <typename F>
204:   static void run_parallel_cdist(Tensor& result, const Tensor& t1, const Tensor& t2, const scalar_t p) {
205:     const scalar_t * const t1_start = t1.const_data_ptr<scalar_t>();
206:     const scalar_t * const t2_start = t2.const_data_ptr<scalar_t>();
207:     int64_t d = t1.size(0);
208:     int64_t r1 = t1.size(-2);
209:     int64_t r2 = t2.size(-2);
210:     int64_t m = t1.size(-1);
211:
212:     scalar_t * const res_start = result.data_ptr<scalar_t>();
213:     int64_t combs = r1 * r2;
214:     int64_t size1 = r1 * m;
215:     int64_t size2 = r2 * m;
216:
217:     parallel_for(0, combs * d, internal::GRAIN_SIZE / (16 * m), [=](int64_t start, int64_t end) {
218:       scalar_t * res = res_start + start;
219:       const scalar_t * const res_end = res_start + end;
220:       int64_t l = start / combs;
221:       int64_t k = start % combs;
222:       int64_t i = k / r2;
223:       int64_t j = k % r2;
224:       i = i * m;
225:       j = j * m;
```
- EN: The main symbol in this range is `run_parallel_cdist`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 这一段的主要符号是 `run_parallel_cdist`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 227-251
```cpp
227:       while (res != res_end) {
228:         const scalar_t * self_i = t1_start + size1 * l + i;
229:         const scalar_t * self_j = t2_start + size2 * l + j;
230:
231:         scalar_t agg = 0;
232:         for (const auto x : c10::irange(m)) {
233:           scalar_t a = *(self_i + x);
234:           scalar_t b = *(self_j + x);
235:           agg = F::red(agg, F::map(std::abs(a-b), p));
236:         }
237:         *res = F::finish(agg, p);
238:
239:         res += 1;
240:         j += m;
241:         if (j == size2) {
242:           j = 0;
243:           i += m;
244:           if (i == size1) {
245:             i = 0;
246:             l += 1;
247:           }
248:         }
249:       }
250:     });
251:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 253-273
```cpp
253:   static void apply_cdist(Tensor& result, const Tensor& x1, const Tensor& x2, const scalar_t p) {
254:     if (p == 0.0) {
255:       run_parallel_cdist<zdist_calc<scalar_t>>(result, x1, x2, p);
256:     } else if (p == 1.0) {
257:       run_parallel_cdist<odist_calc<scalar_t>>(result, x1, x2, p);
258:     } else if (p == 2.0) {
259:       run_parallel_cdist<tdist_calc<scalar_t>>(result, x1, x2, p);
260:     } else if (std::isinf(p)) {
261:       run_parallel_cdist<idist_calc<scalar_t>>(result, x1, x2, p);
262:     } else {
263:       run_parallel_cdist<pdist_calc<scalar_t>>(result, x1, x2, p);
264:     }
265:   }
266:
267:   // This does a backward pass down a Vec column of the input
268:   template <typename F>
269:   static void backward_down_column_pdist(const scalar_t * self_i, scalar_t * res_i, const scalar_t * grad_k, const scalar_t * dist_k, const Vec& pvec, int64_t n, int64_t m, int64_t gs, int64_t count = Vec::size()) {
270:     for (const scalar_t * const self_end = self_i + m * n; self_i != self_end - m; self_i += m, res_i += m) {
271:
272:       const Vec self_vec_i = Vec::loadu(self_i, count);
273:       Vec res_vec_i = Vec::loadu(res_i, count);
```
- EN: The main symbol in this range is `apply_cdist`, `backward_down_column_pdist`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `apply_cdist`, `backward_down_column_pdist`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 275-296
```cpp
275:       const scalar_t * self_j = self_i + m;
276:       scalar_t * res_j = res_i + m;
277:       for (; self_j != self_end; self_j += m, res_j += m, grad_k += gs, dist_k += 1) {
278:         const Vec self_vec_j = Vec::loadu(self_j, count);
279:         Vec res_vec_j = Vec::loadu(res_j, count);
280:
281:         Vec res = F::backward(self_vec_i - self_vec_j, *grad_k, *dist_k, pvec);
282:         res_vec_i = res_vec_i + res;
283:         res_vec_j = res_vec_j - res;
284:
285:         res_vec_j.store(res_j, count);
286:       }
287:
288:       res_vec_i.store(res_i, count);
289:     }
290:   }
291:
292:   template <typename F>
293:   static void run_backward_parallel_pdist(Tensor& result, const Tensor & grad, const Tensor & self, const scalar_t p, const Tensor& dist) {
294:     const int64_t n = self.size(0);
295:     const int64_t m = self.size(1);
296:     const int64_t gs = grad.stride(0);
```
- EN: The main symbol in this range is `run_backward_parallel_pdist`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `run_backward_parallel_pdist`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 298-320
```cpp
298:     const scalar_t * const grad_start = grad.const_data_ptr<scalar_t>();
299:     const scalar_t * const dist_start = dist.const_data_ptr<scalar_t>();
300:     const scalar_t * const self_start = self.const_data_ptr<scalar_t>();
301:     scalar_t * const res_start = result.data_ptr<scalar_t>();
302:
303:     // The only way to parallelize and avoid locking requires parallelizing
304:     // over the columns of the input, i.e. we compute the gradient for the
305:     // first section of each vector independently of the second section, etc.
306:     at::parallel_for(0, m / Vec::size(), internal::GRAIN_SIZE / (8 * n * n), [p, n, m, gs, grad_start, dist_start, self_start, res_start](int64_t l, int64_t end) {
307:       const Vec pvec(p);
308:
309:       const scalar_t * self_l = self_start + l * Vec::size();
310:       scalar_t * res_l = res_start + l * Vec::size();
311:
312:       for (const scalar_t * const res_end = res_start + end * Vec::size(); res_l != res_end; self_l += Vec::size(), res_l += Vec::size()) {
313:         backward_down_column_pdist<F>(self_l, res_l, grad_start, dist_start, pvec, n, m, gs);
314:       }
315:     });
316:     const int64_t remainder = m % Vec::size();
317:     if (remainder) {
318:       backward_down_column_pdist<F>(self_start + (m - remainder), res_start + (m - remainder), grad_start, dist_start, Vec(p), n, m, gs, remainder);
319:     }
320:   }
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 322-340
```cpp
322:   // Assumes self is nonempty, contiguous, and 2D and dist is also contiguous
323:   static void apply_backward_pdist(Tensor& result, const Tensor& grad, const Tensor& self, const double p, const Tensor& dist) {
324:     result.fill_(0);
325:     if (p == 0.0) {
326:     } else if (p == 1.0) {
327:       run_backward_parallel_pdist<odist_calc<Vec>>(result, grad, self, p, dist);
328:     } else if (p < 2.0) {
329:       run_backward_parallel_pdist<lttdist_calc>(result, grad, self, p, dist);
330:     } else if (p == 2.0) {
331:       run_backward_parallel_pdist<tdist_calc<Vec>>(result, grad, self, p, dist);
332:     } else if (std::isinf(p)) {
333:       run_backward_parallel_pdist<idist_calc<Vec>>(result, grad, self, p, dist);
334:     } else {
335:       run_backward_parallel_pdist<pdist_calc<Vec>>(result, grad, self, p, dist);
336:     }
337:   }
338:
339:   static void apply_backward_cdist(Tensor& result, const Tensor& grad, const Tensor& x1, const Tensor& x2, const double p, const Tensor& dist) {
340:     result.fill_(0);
```
- EN: The main symbol in this range is `apply_backward_pdist`, `apply_backward_cdist`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `apply_backward_pdist`, `apply_backward_cdist`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 341-367
```cpp
341:     if (p == 0.0) {
342:     } else if (p == 1.0) {
343:       run_backward_parallel_cdist<odist_calc<Vec>>(result, grad, x1, x2, p, dist);
344:     } else if (p < 2.0) {
345:       run_backward_parallel_cdist<lttdist_calc>(result, grad, x1, x2, p, dist);
346:     } else if (p == 2.0) {
347:       run_backward_parallel_cdist<tdist_calc<Vec>>(result, grad, x1, x2, p, dist);
348:     } else if (std::isinf(p)) {
349:       run_backward_parallel_cdist<idist_calc<Vec>>(result, grad, x1, x2, p, dist);
350:     } else {
351:       run_backward_parallel_cdist<pdist_calc<Vec>>(result, grad, x1, x2, p, dist);
352:     }
353:   }
354:
355:
356:   template <typename F>
357:   static void run_backward_parallel_cdist(Tensor& result, const Tensor & grad, const Tensor & t1, const Tensor & t2, const scalar_t p, const Tensor& dist) {
358:     const int64_t r1 = t1.size(-2);
359:     const int64_t r2 = t2.size(-2);
360:     const int64_t m = t1.size(-1);
361:     const int64_t d = result.size(0);
362:     const int64_t l1_size = r1 * m;
363:     const int64_t l2_size = r2 * m;
364:     //current implementation supports only tensor that can be collapsed to 1D. However, to avoid checking if grad satisfies this assumption,
365:     //we call .contiguous() on grad before backward, thus stride is guaranteed to be 1
366:     //don't use grad.stride(-1), because if last dimension is 1, stride can be bogus.
367:     const int64_t gs = 1;
```
- EN: The main symbol in this range is `run_backward_parallel_cdist`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `run_backward_parallel_cdist`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 369-390
```cpp
369:     const scalar_t * const grad_start = grad.const_data_ptr<scalar_t>();
370:     const scalar_t * const dist_start = dist.const_data_ptr<scalar_t>();
371:     const scalar_t * const t1_start = t1.const_data_ptr<scalar_t>();
372:     const scalar_t * const t2_start = t2.const_data_ptr<scalar_t>();
373:     scalar_t * const res_start = result.data_ptr<scalar_t>();
374:
375:     at::parallel_for(0, m / Vec::size(), internal::GRAIN_SIZE / (16 * r1), [=](int64_t l, int64_t end) {
376:       const Vec pvec(p);
377:
378:       const scalar_t * i = t1_start + l * Vec::size();
379:       const scalar_t * j = t2_start + l * Vec::size();
380:       scalar_t * res_l = res_start + l * Vec::size();
381:
382:       for (const scalar_t * const res_end = res_start + end * Vec::size(); res_l != res_end; i += Vec::size(), j += Vec::size(), res_l += Vec::size()) {
383:         backward_down_column_cdist<F>(i, j, res_l, grad_start, dist_start, pvec, r1, r2, m, d, gs, l1_size, l2_size);
384:       }
385:     });
386:     const int64_t remainder = m % Vec::size();
387:     if (remainder) {
388:       backward_down_column_cdist<F>(t1_start + (m - remainder), t2_start + (m - remainder), res_start + (m - remainder), grad_start, dist_start, Vec(p), r1, r2, m, d, gs, l1_size, l2_size, remainder);
389:     }
390:   }
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 392-414
```cpp
392:   template <typename F>
393:   static void backward_down_column_cdist(const scalar_t * t1, const scalar_t * t2, scalar_t * res, const scalar_t * grad_k, const scalar_t * dist_k, const Vec& pvec, int64_t r1, int64_t r2, int64_t m, int64_t d, int64_t gs, int64_t l1_size, int64_t l2_size, int64_t count = Vec::size()) {
394:     const scalar_t * t1_end = t1 + l1_size;
395:     const scalar_t * t2_end = t2 + l2_size;
396:
397:     for ([[maybe_unused]] const auto l : c10::irange(d)) {
398:       for (; t1 != t1_end; t1 += m, res += m) {
399:         const Vec vec_t1 = Vec::loadu(t1, count);
400:         Vec res_vec = Vec::loadu(res, count);
401:
402:         for (const scalar_t * t2_curr = t2; t2_curr != t2_end; t2_curr += m, grad_k += gs, dist_k += 1) {
403:           const Vec vec_t2 = Vec::loadu(t2_curr, count);
404:           Vec res = F::backward(vec_t1 - vec_t2, *grad_k, *dist_k, pvec);
405:           res_vec = res_vec + res;
406:         }
407:
408:         res_vec.store(res, count);
409:       }
410:       t1_end += l1_size;
411:       t2_end += l2_size;
412:       t2 += l2_size;
413:     }
414:   }
```
- EN: The main symbol in this range is `backward_down_column_cdist`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `backward_down_column_cdist`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 416-434
```cpp
416: };
417:
418: void pdist_forward_kernel_impl(Tensor& result, const Tensor& self, const double p) {
419:   AT_DISPATCH_FLOATING_TYPES(self.scalar_type(), "pdist", [&] {
420:     Dist<scalar_t>::apply_pdist(result, self, p);
421:   });
422: }
423:
424: void pdist_backward_kernel_impl(Tensor& result, const Tensor& grad, const Tensor& self, const double p, const Tensor& dist) {
425:   AT_DISPATCH_FLOATING_TYPES(self.scalar_type(), "pdist_backward", [&] {
426:     Dist<scalar_t>::apply_backward_pdist(result, grad, self, p, dist);
427:   });
428: }
429:
430: void cdist_kernel_impl(Tensor& result, const Tensor& x1, const Tensor& x2, const double p) {
431:   AT_DISPATCH_FLOATING_TYPES(result.scalar_type(), "cdist", [&] {
432:     Dist<scalar_t>::apply_cdist(result, x1, x2, p);
433:   });
434: }
```
- EN: The main symbol in this range is `pdist_forward_kernel_impl`, `pdist_backward_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 这一段的主要符号是 `pdist_forward_kernel_impl`, `pdist_backward_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 436-450
```cpp
436: void cdist_backward_kernel_impl(Tensor& result, const Tensor& grad, const Tensor& x1, const Tensor& x2, const double p, const Tensor& dist) {
437:   AT_DISPATCH_FLOATING_TYPES(result.scalar_type(), "cdist_backward", [&] {
438:     Dist<scalar_t>::apply_backward_cdist(result, grad, x1, x2, p, dist);
439:   });
440: }
441:
442:
443: }  // anonymous namespace
444:
445: REGISTER_DISPATCH(pdist_forward_stub, &pdist_forward_kernel_impl)
446: REGISTER_DISPATCH(pdist_backward_stub, &pdist_backward_kernel_impl)
447: REGISTER_DISPATCH(cdist_stub, &cdist_kernel_impl)
448: REGISTER_DISPATCH(cdist_backward_stub, &cdist_backward_kernel_impl)
449:
450: }  // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `cdist_backward_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `cdist_backward_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Normalization statistics / 归一化统计
- Dispatcher registration / 调度器注册

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/native/Distance.h`, `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`, `ATen/TensorIterator.h`, `ATen/cpu/vec/functional.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `algorithm`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `Vectorized`, `parallel_for`, `REGISTER_DISPATCH`, `AT_DISPATCH_FLOATING_TYPES`
