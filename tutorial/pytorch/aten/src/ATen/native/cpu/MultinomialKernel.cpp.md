# MultinomialKernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/MultinomialKernel.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares support for Multinomial Kernel in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了 Multinomial Kernel 的相关支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3:
 4: #include <ATen/Dispatch.h>
 5: #include <ATen/core/DistributionsHelper.h>
 6: #include <ATen/native/Copy.h>
 7: #include <ATen/native/TensorIterator.h>
 8: #include <ATen/native/UnaryOps.h>
 9: #include <ATen/native/cpu/Loops.h>
10: #include <c10/util/irange.h>
11:
12: #ifndef AT_PER_OPERATOR_HEADERS
13: #include <ATen/Functions.h>
14: #else
15: #include <ATen/ops/empty.h>
16: #endif
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/core/DistributionsHelper.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/core/DistributionsHelper.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 18-31
```cpp
18: namespace at::native {
19: namespace {
20:
21: template <typename scalar_t>
22: typename std::enable_if_t<!is_reduced_floating_point_v<scalar_t>, void>
23: multinomial_with_replacement_apply(
24:     Tensor& result,
25:     const Tensor& self,
26:     const int64_t n_sample,
27:     std::optional<Generator> generator) {
28:   auto gen = get_generator_or_default<CPUGeneratorImpl>(
29:       generator, detail::getDefaultCPUGenerator());
30:   // See Note [Acquire lock when using random generators]
31:   std::lock_guard<std::mutex> lock(gen->mutex_);
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `multinomial_with_replacement_apply`, which contributes directly to this file's operator logic.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `multinomial_with_replacement_apply`，它们直接构成本文件的算子逻辑。

### Lines 33-44
```cpp
33:   int64_t n_categories = self.size(-1);
34:   int64_t n_dist = self.dim() > 1 ? self.size(-2) : 1;
35:
36:   /* cumulative probability distribution vector */
37:   Tensor cum_dist = at::empty({n_categories}, self.options());
38:
39:   const scalar_t* const self_ptr = self.const_data_ptr<scalar_t>();
40:   scalar_t* const cum_dist_ptr = cum_dist.data_ptr<scalar_t>();
41:   int64_t* const result_ptr = result.data_ptr<int64_t>();
42:
43:   auto self_stride_0 = self.dim() > 1 ? self.stride(-2) : 0;
44:   auto self_stride_1 = self.stride(-1);
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 46-56
```cpp
46:   auto cum_dist_stride_0 = cum_dist.stride(0);
47:
48:   auto result_dist_stride_0 = result.dim() > 1 ? result.stride(-2) : 0;
49:   auto result_dist_stride_1 = result.stride(-1);
50:
51:   for (const auto i : c10::irange(n_dist)) {
52:     /* Get normalized cumulative distribution from prob distribution */
53:     scalar_t sum = 0;
54:     for (const auto j : c10::irange(n_categories)) {
55:       scalar_t val = self_ptr[i * self_stride_0 + j * self_stride_1];
56:       TORCH_CHECK(
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Normalization-related state, scaling, or statistics are handled here.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里处理归一化相关的状态、缩放或统计量。

### Lines 57-69
```cpp
57:           val >= 0,
58:           "invalid multinomial distribution (encountering probability entry < 0)");
59: // NB: std::isfinite doesn't bode well with libc++ for half datatypes,
60: // so we manually cast it to a double and perform the check.
61: #if defined(_LIBCPP_VERSION)
62:       TORCH_CHECK(
63:           std::isfinite(static_cast<double>(val)),
64:           "invalid multinomial distribution (encountering probability entry = infinity or NaN)");
65: #else
66:       TORCH_CHECK(
67:           std::isfinite(val),
68:           "invalid multinomial distribution (encountering probability entry = infinity or NaN)");
69: #endif
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 71-85
```cpp
71:       sum += val;
72:       cum_dist_ptr[j * cum_dist_stride_0] = sum;
73:     }
74:
75:     TORCH_CHECK(
76:         sum > 0,
77:         "invalid multinomial distribution (sum of probabilities <= 0)");
78:
79:     /* normalize cumulative probability distribution so that last val is 1
80:     i.e. doesn't assume original self row sums to one */
81:     if ((sum > 0) || ((sum < 1.00001) && (sum > 0.99999))) {
82:       for (const auto j : c10::irange(n_categories)) {
83:         cum_dist_ptr[j * cum_dist_stride_0] /= sum;
84:       }
85:     }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Normalization-related state, scaling, or statistics are handled here.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里处理归一化相关的状态、缩放或统计量。

### Lines 87-96
```cpp
87:     for (const auto j : c10::irange(n_sample)) {
88:       /* sample a probability mass from a uniform distribution */
89:       at::uniform_real_distribution<double> uniform(0, 1);
90:       double uniform_sample = uniform(gen);
91:       /* Do a binary search for the slot in which the prob falls
92:       ie cum_dist[row][slot-1] < uniform_prob < cum_distr[row][slot] */
93:       int64_t left_pointer = 0;
94:       int64_t right_pointer = n_categories;
95:       /* Make sure the last cumulative distribution bucket sums to 1 */
96:       cum_dist_ptr[(n_categories - 1) * cum_dist_stride_0] = 1;
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 98-107
```cpp
 98:       while (right_pointer - left_pointer > 0) {
 99:         int64_t mid_pointer = left_pointer + (right_pointer - left_pointer) / 2;
100:         scalar_t cum_prob = cum_dist_ptr[mid_pointer * cum_dist_stride_0];
101:         if (cum_prob < uniform_sample) {
102:           left_pointer = mid_pointer + 1;
103:         } else {
104:           right_pointer = mid_pointer;
105:         }
106:       }
107:       auto sample_idx = left_pointer;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 109-119
```cpp
109:       /* store in result tensor (will be incremented for lua compat by wrapper)
110:        */
111:       result_ptr[i * result_dist_stride_0 + j * result_dist_stride_1] =
112:           sample_idx;
113:     }
114:   }
115: }
116:
117: template <typename scalar_t>
118: typename std::enable_if_t<is_reduced_floating_point_v<scalar_t>, void>
119: multinomial_with_replacement_apply(
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 120-133
```cpp
120:     Tensor& result,
121:     const Tensor& self,
122:     const int64_t n_sample,
123:     std::optional<Generator> generator) {
124:   auto gen = get_generator_or_default<CPUGeneratorImpl>(
125:       generator, detail::getDefaultCPUGenerator());
126:   // See Note [Acquire lock when using random generators]
127:   std::lock_guard<std::mutex> lock(gen->mutex_);
128:
129:   int64_t n_categories = self.size(-1);
130:   int64_t n_dist = self.dim() > 1 ? self.size(-2) : 1;
131:
132:   /* cumulative probability distribution vector */
133:   Tensor cum_dist = at::empty({n_categories}, self.options().dtype(kFloat));
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 135-145
```cpp
135:   const scalar_t* const self_ptr = self.const_data_ptr<scalar_t>();
136:   float* const cum_dist_ptr = cum_dist.data_ptr<float>();
137:   int64_t* const result_ptr = result.data_ptr<int64_t>();
138:
139:   auto self_stride_0 = self.dim() > 1 ? self.stride(-2) : 0;
140:   auto self_stride_1 = self.stride(-1);
141:
142:   auto cum_dist_stride_0 = cum_dist.stride(0);
143:
144:   auto result_dist_stride_0 = result.dim() > 1 ? result.stride(-2) : 0;
145:   auto result_dist_stride_1 = result.stride(-1);
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 147-157
```cpp
147:   for (const auto i : c10::irange(n_dist)) {
148:     /* Get normalized cumulative distribution from prob distribution */
149:     float sum = 0;
150:     for (const auto j : c10::irange(n_categories)) {
151:       float val = self_ptr[i * self_stride_0 + j * self_stride_1];
152:       TORCH_CHECK(
153:           val >= 0,
154:           "invalid multinomial distribution (encountering probability entry < 0)");
155: // NB: std::isfinite doesn't bode well with libc++ for half datatypes,
156: // so we manually cast it to a double and perform the check.
157: #if defined(_LIBCPP_VERSION)
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Normalization-related state, scaling, or statistics are handled here.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里处理归一化相关的状态、缩放或统计量。

### Lines 158-169
```cpp
158:       TORCH_CHECK(
159:           std::isfinite(static_cast<double>(val)),
160:           "invalid multinomial distribution (encountering probability entry = infinity or NaN)");
161: #else
162:       TORCH_CHECK(
163:           std::isfinite(val),
164:           "invalid multinomial distribution (encountering probability entry = infinity or NaN)");
165: #endif
166:
167:       sum += val;
168:       cum_dist_ptr[j * cum_dist_stride_0] = sum;
169:     }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 171-181
```cpp
171:     TORCH_CHECK(
172:         sum > 0,
173:         "invalid multinomial distribution (sum of probabilities <= 0)");
174:
175:     /* normalize cumulative probability distribution so that last val is 1
176:     i.e. doesn't assume original self row sums to one */
177:     if ((sum > 0) || ((sum < 1.00001) && (sum > 0.99999))) {
178:       for (const auto j : c10::irange(n_categories)) {
179:         cum_dist_ptr[j * cum_dist_stride_0] /= sum;
180:       }
181:     }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Normalization-related state, scaling, or statistics are handled here.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里处理归一化相关的状态、缩放或统计量。

### Lines 183-192
```cpp
183:     for (const auto j : c10::irange(n_sample)) {
184:       /* sample a probability mass from a uniform distribution */
185:       at::uniform_real_distribution<double> uniform(0, 1);
186:       double uniform_sample = uniform(gen);
187:       /* Do a binary search for the slot in which the prob falls
188:       ie cum_dist[row][slot-1] < uniform_prob < cum_distr[row][slot] */
189:       int64_t left_pointer = 0;
190:       int64_t right_pointer = n_categories;
191:       /* Make sure the last cumulative distribution bucket sums to 1 */
192:       cum_dist_ptr[(n_categories - 1) * cum_dist_stride_0] = 1;
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 194-203
```cpp
194:       while (right_pointer - left_pointer > 0) {
195:         int64_t mid_pointer = left_pointer + (right_pointer - left_pointer) / 2;
196:         float cum_prob = cum_dist_ptr[mid_pointer * cum_dist_stride_0];
197:         if (cum_prob < uniform_sample) {
198:           left_pointer = mid_pointer + 1;
199:         } else {
200:           right_pointer = mid_pointer;
201:         }
202:       }
203:       auto sample_idx = left_pointer;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 205-215
```cpp
205:       /* store in result tensor (will be incremented for lua compat by wrapper)
206:        */
207:       result_ptr[i * result_dist_stride_0 + j * result_dist_stride_1] =
208:           sample_idx;
209:     }
210:   }
211: }
212:
213: void multinomial_with_replacement_kernel_impl(
214:     Tensor& result,
215:     const Tensor& self,
```
- EN: The main symbol in this range is `multinomial_with_replacement_kernel_impl`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `multinomial_with_replacement_kernel_impl`，它们直接构成本文件的算子逻辑。

### Lines 216-227
```cpp
216:     const int64_t n_sample,
217:     std::optional<Generator> gen) {
218:   AT_DISPATCH_FLOATING_TYPES_AND2(
219:       kHalf, kBFloat16, self.scalar_type(), "multinomial", [&] {
220:         multinomial_with_replacement_apply<scalar_t>(
221:             result, self, n_sample, gen);
222:       });
223: }
224: } // namespace
225:
226: REGISTER_DISPATCH(
227:     multinomial_with_replacement_stub,
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 228-229
```cpp
228:     &multinomial_with_replacement_kernel_impl)
229: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- Normalization statistics / 归一化统计
- Dispatcher registration / 调度器注册
- Low-level memory access / 底层内存访问

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/core/DistributionsHelper.h`, `ATen/native/Copy.h`, `ATen/native/TensorIterator.h`, `ATen/native/UnaryOps.h`, `ATen/native/cpu/Loops.h`, `ATen/Functions.h`, `ATen/ops/empty.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `REGISTER_DISPATCH`, `AT_DISPATCH_FLOATING_TYPES`
