# DistributionKernels.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/DistributionKernels.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares support for Distribution Kernels in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了 Distribution Kernels 的相关支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/CPUGeneratorImpl.h>
 3: #include <ATen/Dispatch.h>
 4: #include <ATen/Generator.h>
 5: #include <ATen/core/DistributionsHelper.h>
 6: #include <ATen/native/Distributions.h>
 7: #include <ATen/native/cpu/DistributionTemplates.h>
 8:
 9: #include <ATen/native/UnaryOps.h>
10:
11: #ifndef AT_PER_OPERATOR_HEADERS
12: #include <ATen/Functions.h>
13: #else
14: #include <ATen/ops/empty.h>
15: #endif
```
- EN: This range pulls in required headers, including `ATen/CPUGeneratorImpl.h`, `ATen/Dispatch.h`, `ATen/Generator.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段引入了所需头文件，例如 `ATen/CPUGeneratorImpl.h`, `ATen/Dispatch.h`, `ATen/Generator.h`。 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 17-28
```cpp
17: #include <cmath>
18: #include <limits>
19: #include <type_traits>
20:
21: // Disable MKL rng until https://github.com/pytorch/pytorch/issues/132395 is addressed
22: #if AT_MKL_ENABLED() && defined(FBCODE_CAFFE2)
23: #include <mkl.h>
24: #include <cpuinfo.h>
25: #endif
26:
27: namespace at::native {
28: namespace {
```
- EN: This range pulls in required headers, including `cmath`, `limits`, `type_traits`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `cmath`, `limits`, `type_traits`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 30-45
```cpp
30: void cauchy_kernel(TensorIteratorBase& iter, double median, double sigma, std::optional<Generator> gen) {
31:   CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
32:   templates::cpu::cauchy_kernel(iter, median, sigma, generator);
33: }
34:
35: void bernoulli_tensor_kernel(const TensorBase &self, const TensorBase &p_, std::optional<Generator> gen) {
36:   CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
37:   templates::cpu::bernoulli_kernel(self, p_, generator);
38: }
39:
40: // Disable MKL rng until https://github.com/pytorch/pytorch/issues/132395 is addressed
41: #if !AT_MKL_ENABLED() || (AT_MKL_ENABLED() && !defined(FBCODE_CAFFE2))
42: void bernoulli_scalar_kernel_default(const TensorBase &self, double p, std::optional<Generator> gen) {
43:   CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
44:   templates::cpu::bernoulli_kernel(self, p, generator);
45: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `cauchy_kernel`, `bernoulli_tensor_kernel`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `cauchy_kernel`, `bernoulli_tensor_kernel`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 47-60
```cpp
47: void bernoulli_scalar_kernel(const TensorBase &self, double p, std::optional<Generator> gen) {
48:   bernoulli_scalar_kernel_default(self, p, gen);
49: }
50: #else
51: void bernoulli_scalar_kernel(const TensorBase &self, double p, std::optional<Generator> gen) {
52:   CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
53:   int64_t seed;
54:   {
55:     // See Note [Acquire lock when using random generators]
56:     std::lock_guard<std::mutex> lock(generator->mutex_);
57:     seed = generator->random();
58:   }
59:   int64_t n = self.numel();
60:   bool contig = self.is_contiguous();
```
- EN: The main symbol in this range is `bernoulli_scalar_kernel`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `bernoulli_scalar_kernel`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 62-72
```cpp
62:   AT_DISPATCH_ALL_TYPES_AND3(at::ScalarType::Bool, at::ScalarType::BFloat16, at::ScalarType::Half,
63:   self.scalar_type(), "bernoulli_scalar_cpu_", [&] {
64:     at::Tensor tmp_int_tensor;
65:     if (std::is_same_v<scalar_t, int> && contig) {
66:       tmp_int_tensor = self;
67:     } else {
68:       tmp_int_tensor = at::empty(self.sizes(), self.options().dtype(at::kInt));
69:     }
70:
71:     scalar_t *self_ptr = self.data_ptr<scalar_t>();
72:     int *sample_int_ptr = tmp_int_tensor.data_ptr<int>();
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 74-84
```cpp
74:     auto sample = [&](int64_t begin, int64_t end) {
75:       int64_t len = end - begin;
76:       if (len > 0) {
77:         VSLStreamStatePtr stream;
78:         vslNewStream(&stream, VSL_BRNG_MCG31, seed);
79:         vslSkipAheadStream(stream, begin);
80:         viRngBernoulli(VSL_RNG_METHOD_BERNOULLI_ICDF, stream, len,
81:           sample_int_ptr + begin, p);
82:         vslDeleteStream(&stream);
83:
84:         // vectorized copy if using buffer and contiguous, i.e., being non-int
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 85-96
```cpp
85:         // type and contiguous
86:         if (!std::is_same_v<scalar_t, int> && contig) {
87:           scalar_t *self_seg = self_ptr + begin;
88:           int* tmp_seg = sample_int_ptr + begin;
89:           at::vec::convert<int, scalar_t>(tmp_seg, self_seg, len);
90:         }
91:       }
92:     };
93:
94:     parallel_for(0, n, /* grain_size= */ 800, sample);
95:
96:     // copy_ if using buffer and non contiguous
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 97-107
```cpp
 97:     if (!contig) {
 98:       OptionalTensorRef(self)->copy_(tmp_int_tensor);
 99:     }
100:   });
101: }
102: #endif
103:
104: void exponential_kernel_default(TensorIteratorBase& iter, double lambda, std::optional<Generator> gen) {
105:   CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
106:   templates::cpu::exponential_kernel(iter, lambda, generator);
107: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `exponential_kernel_default`, `exponential_kernel`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `exponential_kernel_default`, `exponential_kernel`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 109-119
```cpp
109: // Disable MKL rng until https://github.com/pytorch/pytorch/issues/132395 is addressed
110: #if (!AT_MKL_ENABLED() || defined(FBCODE_CAFFE2) || 1)
111: void exponential_kernel(TensorIteratorBase& iter, double lambda, std::optional<Generator> gen) {
112:   exponential_kernel_default(iter, lambda, gen);
113: }
114: #else
115: void exponential_kernel(TensorIteratorBase &iter, double lambda, std::optional<Generator> gen) {
116:   TORCH_CHECK(isFloatingType(iter.dtype()), "Exponential distribution is a continuous probability distribution. dtype must be a floating point but you specified ", iter.dtype());
117:
118:   Tensor self = iter.tensor(0);
119:   if (lambda > 0 && !std::isinf(lambda) && !std::isnan(lambda)) {
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `exponential_kernel`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `exponential_kernel`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 120-131
```cpp
120:     CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
121:     int64_t seed;
122:     {
123:       // See Note [Acquire lock when using random generators]
124:       std::lock_guard<std::mutex> lock(generator->mutex_);
125:       if (self.scalar_type() == at::kDouble)
126:         seed = generator->random64();
127:       else
128:         seed = generator->random();
129:     }
130:     int64_t n = self.numel();
131:     bool contig = self.is_contiguous();
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 133-146
```cpp
133:     AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, self.scalar_type(), "exponential_cpu", [&] {
134:       at::Tensor tmp_tensor;
135:       constexpr bool is_df = std::is_same_v<scalar_t, float> || std::is_same_v<scalar_t, double>;
136:       if (is_df && contig) {
137:         tmp_tensor = self;
138:       } else if (std::is_same_v<scalar_t, double>) {
139:         tmp_tensor = at::empty(self.sizes(), self.options().dtype(at::kDouble));
140:       } else {
141:         tmp_tensor = at::empty(self.sizes(), self.options().dtype(at::kFloat));
142:       }
143:
144:       scalar_t *self_ptr = self.data_ptr<scalar_t>();
145:       using tmp_scalar_t = typename std::conditional_t<std::is_same_v<scalar_t, double>, double, float>;
146:       tmp_scalar_t *sample_ptr = tmp_tensor.data_ptr<tmp_scalar_t>();
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 148-159
```cpp
148:       // Intel MKL vRngExponential variate originally does not exclude 0.
149:       // However, to align with pytorch exponential variate definition which excludes 0,
150:       // we shift the MKL vRngExponential distribution location by adding a very small constant, eps.
151:       // If X ~ Exp(lambda), then E(X) = 1/lambda, and V(X) = 1/lambda**2.
152:       // If Y = X + eps, where eps ~= 0, then E(Y) = (1/lambda) + eps, and V(Y) = 1/lambda**2.
153:       // If eps is very small, the two distributions are indistinguishable, and are almost identical.
154:       // The detail of location-shifted MKL vRngExponential is as follows.
155:       // PDF:         f(x) = lambda * exp( -lambda * (x - eps) )
156:       // CDF:         F(x) = 1 - exp( -lambda * (x - eps) )
157:       // Mean:        E[X+eps] = (1/lambda) + eps
158:       // Variance:    V[X+eps] = 1/lambda**2
159:       auto eps = std::numeric_limits<tmp_scalar_t>::min();
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 161-171
```cpp
161:       auto sample = [&](int64_t begin, int64_t end) {
162:         int64_t len = end - begin;
163:         if (len > 0) {
164:           VSLStreamStatePtr stream;
165:           if constexpr (std::is_same_v<scalar_t, double>) {
166:             vslNewStream(&stream, VSL_BRNG_MCG31, seed);
167:             vslSkipAheadStream(stream, begin);
168:             vdRngExponential(VSL_RNG_METHOD_EXPONENTIAL_ICDF, stream, len,
169:               (double *)(sample_ptr + begin), eps, 1./lambda);
170:             vslDeleteStream(&stream);
171:           } else {
```
- EN: The main symbol in this range is `constexpr`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `constexpr`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 172-185
```cpp
172:             vslNewStream(&stream, VSL_BRNG_MCG31, seed);
173:             vslSkipAheadStream(stream, begin);
174:             vsRngExponential(VSL_RNG_METHOD_EXPONENTIAL_ICDF, stream, len,
175:               (float *) (sample_ptr + begin), eps, 1./lambda);
176:             vslDeleteStream(&stream);
177:           }
178:           // vectorized copy if using buffer and contiguous
179:           if (!is_df && contig) {
180:             scalar_t *self_seg = self_ptr + begin;
181:             tmp_scalar_t *tmp_seg = sample_ptr + begin;
182:             at::vec::convert<tmp_scalar_t, scalar_t>(tmp_seg, self_seg, len);
183:           }
184:         }
185:       };
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The math and shape handling relate to convolution-style operators.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这里的计算与形状处理与卷积类算子相关。

### Lines 187-199
```cpp
187:       parallel_for(0, n, /* grain_size= */ 800, sample);
188:
189:       // copy_ if using buffer and non contiguous
190:       if (!contig) {
191:         self.copy_(tmp_tensor);
192:       }
193:     });
194:   } else {
195:     // The situation of inf and nan, move to using the default version
196:     exponential_kernel_default(iter, lambda, gen);
197:   }
198: }
199: #endif
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 201-214
```cpp
201: void geometric_kernel(TensorIteratorBase& iter, double p, std::optional<Generator> gen) {
202:   CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
203:   templates::cpu::geometric_kernel(iter, p, generator);
204: }
205:
206: void log_normal_kernel(TensorIteratorBase& iter, double mean, double std, std::optional<Generator> gen) {
207:   CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
208:   templates::cpu::log_normal_kernel(iter, mean, std, generator);
209: }
210:
211: void uniform_kernel(TensorIteratorBase& iter, double from, double to, std::optional<Generator> gen) {
212:   CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
213:   templates::cpu::uniform_kernel(iter, from, to, generator);
214: }
```
- EN: The main symbol in this range is `geometric_kernel`, `log_normal_kernel`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Normalization-related state, scaling, or statistics are handled here.
- CN: 这一段的主要符号是 `geometric_kernel`, `log_normal_kernel`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 这里处理归一化相关的状态、缩放或统计量。

### Lines 216-229
```cpp
216: void normal_kernel(const TensorBase &self, double mean, double std, std::optional<Generator> gen) {
217:   CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
218:   templates::cpu::normal_kernel(self, mean, std, generator);
219: }
220:
221: void random_from_to_kernel(TensorIteratorBase& iter, uint64_t range, int64_t base, std::optional<Generator> gen) {
222:   CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
223:   templates::cpu::random_from_to_kernel(iter, range, base, generator);
224: }
225:
226: void random_kernel(TensorIteratorBase& iter, std::optional<Generator> gen) {
227:   CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
228:   templates::cpu::random_kernel(iter, generator);
229: }
```
- EN: The main symbol in this range is `normal_kernel`, `random_from_to_kernel`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Normalization-related state, scaling, or statistics are handled here.
- CN: 这一段的主要符号是 `normal_kernel`, `random_from_to_kernel`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 这里处理归一化相关的状态、缩放或统计量。

### Lines 231-241
```cpp
231: // This is the special kernel to handle single specific case:
232: // from(inclusive) = std::numeric_limits<int64_t>::lowest()
233: // to(exclusive) = None (= std::numeric_limits<int64_t>::max() + 1)
234: void random_full_64_bits_range_kernel(TensorIteratorBase& iter, std::optional<Generator> gen) {
235:   CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
236:   templates::cpu::random_full_64_bits_range_kernel(iter, generator);
237: }
238:
239: } // namespace (anonymous)
240:
241: REGISTER_DISPATCH(bernoulli_tensor_stub, &bernoulli_tensor_kernel)
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `from`, `random_full_64_bits_range_kernel`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `from`, `random_full_64_bits_range_kernel`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 242-253
```cpp
242: REGISTER_DISPATCH(bernoulli_scalar_stub, &bernoulli_scalar_kernel)
243: REGISTER_DISPATCH(cauchy_stub, &cauchy_kernel)
244: REGISTER_DISPATCH(exponential_stub, &exponential_kernel)
245: REGISTER_DISPATCH(geometric_stub, &geometric_kernel)
246: REGISTER_DISPATCH(log_normal_stub, &log_normal_kernel)
247: REGISTER_DISPATCH(normal_stub, &normal_kernel)
248: REGISTER_DISPATCH(uniform_stub, &uniform_kernel)
249: REGISTER_DISPATCH(random_from_to_stub, &random_from_to_kernel)
250: REGISTER_DISPATCH(random_full_64_bits_range_stub, &random_full_64_bits_range_kernel)
251: REGISTER_DISPATCH(random_stub, &random_kernel)
252:
253: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. Normalization-related state, scaling, or statistics are handled here.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这里处理归一化相关的状态、缩放或统计量。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Normalization statistics / 归一化统计

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/CPUGeneratorImpl.h`, `ATen/Dispatch.h`, `ATen/Generator.h`, `ATen/core/DistributionsHelper.h`, `ATen/native/Distributions.h`, `ATen/native/cpu/DistributionTemplates.h`, `ATen/native/UnaryOps.h`, `ATen/Functions.h`, `ATen/ops/empty.h`
- Standard or third-party headers / 标准库或第三方头文件: `cmath`, `limits`, `type_traits`, `mkl.h`, `cpuinfo.h`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `TensorIteratorBase`, `TensorBase`, `parallel_for`, `REGISTER_DISPATCH`, `AT_DISPATCH_FLOATING_TYPES`, `AT_DISPATCH_ALL_TYPES`, `Scalar`, `ScalarType`
