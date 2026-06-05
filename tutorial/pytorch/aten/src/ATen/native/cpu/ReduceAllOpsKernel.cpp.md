# ReduceAllOpsKernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/ReduceAllOpsKernel.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares support for Reduce All Ops Kernel in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了 Reduce All Ops Kernel 的相关支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/native/ReduceOps.h>
 4: #include <ATen/native/ReduceAllOps.h>
 5: #include <ATen/native/ReduceOpsUtils.h>
 6:
 7: #include <ATen/Dispatch.h>
 8: #include <ATen/Parallel.h>
 9: #include <ATen/TensorIterator.h>
10: #include <ATen/OpMathType.h>
11:
12: #include <ATen/native/cpu/Loops.h>
13: #include <ATen/native/cpu/zmath.h>
14: #include <ATen/cpu/vec/functional.h>
15: #include <ATen/cpu/vec/vec.h>
16: #include <c10/util/irange.h>
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/native/ReduceOps.h`, `ATen/native/ReduceAllOps.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/native/ReduceOps.h`, `ATen/native/ReduceAllOps.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 18-28
```cpp
18: namespace at::native {
19: namespace {
20:
21: using namespace vec;
22:
23: template <typename scalar_t, typename func_t, typename vec_func_t>
24: inline void reduce_all_impl_vec(
25:     Tensor& output,
26:     const Tensor& input,
27:     const scalar_t ident_v,
28:     func_t op,
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 29-43
```cpp
29:     vec_func_t vop) {
30:   using Vec = Vectorized<opmath_type<scalar_t>>;
31:   const int64_t input_numel = input.numel();
32:   auto input_data = input.const_data_ptr<scalar_t>();
33:   // NOTE: parallel_reduce not support bool type
34:   scalar_t result = at::parallel_reduce(0, input_numel, internal::GRAIN_SIZE, ident_v,
35:     [&](int64_t start, int64_t end, const scalar_t /*ident*/) -> scalar_t {
36:       scalar_t partial_out = vec::reduce_all<scalar_t>(
37:         [=](Vec x, Vec y) { return vop(x, y); },
38:         input_data + start,
39:         end - start);
40:       return partial_out;
41:     }, op);
42:   output.fill_(result);
43: }
```
- EN: The main symbol in this range is `reduce_all<scalar_t>`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `reduce_all<scalar_t>`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 45-55
```cpp
45: // For operation not support in avx/avx2
46: template <typename scalar_t, typename func_t>
47: inline void reduce_all_impl(
48:     Tensor& output,
49:     const Tensor& input,
50:     const scalar_t ident_v,
51:     func_t op) {
52:   const int64_t input_numel = input.numel();
53:   auto input_data = input.const_data_ptr<scalar_t>();
54:   scalar_t result = at::parallel_reduce(0, input_numel, internal::GRAIN_SIZE, ident_v,
55:     [&](int64_t start, int64_t end, const scalar_t ident) -> scalar_t {
```
- EN: The main symbol in this range is `reduce_all_impl`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `reduce_all_impl`，它们直接构成本文件的算子逻辑。

### Lines 56-67
```cpp
56:       scalar_t partial_out = ident;
57:       for (const auto i : c10::irange(start, end)) {
58:          partial_out = op(partial_out, input_data[i]);
59:       }
60:       return partial_out;
61:     }, op);
62:   output.fill_(result);
63: }
64:
65: void min_all_kernel_impl(Tensor& result, const Tensor& input) {
66:   if (input.scalar_type() == ScalarType::Bool) {
67:     TensorIterator iter = TensorIteratorConfig()
```
- EN: The main symbol in this range is `min_all_kernel_impl`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `min_all_kernel_impl`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 68-79
```cpp
68:       .add_input(input)
69:       .build();
70:     bool result_data  = true;
71:     cpu_serial_kernel(iter, [&](const bool a) -> void {
72:       result_data = result_data && a;
73:     });
74:     result.fill_(result_data);
75:   } else if(input.scalar_type() == ScalarType::Long) {
76:     // for int64_t, vectorized implementation have performance issue,
77:     // just use scalar path
78:     reduce_all_impl<int64_t>(result, input, upper_bound<int64_t>(),
79:       [=](int64_t a, int64_t b) -> int64_t { return min_impl(a, b); });
```
- EN: The main symbol in this range is `cpu_serial_kernel`, `min_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `cpu_serial_kernel`, `min_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 80-91
```cpp
80:   } else {
81:     AT_DISPATCH_ALL_TYPES_AND2(kHalf, kBFloat16, input.scalar_type(), "min_all", [&] {
82:       using Vec = Vectorized<opmath_type<scalar_t>>;
83:       reduce_all_impl_vec<scalar_t>(result, input, upper_bound<scalar_t>(),
84:         [=] (scalar_t a , scalar_t b) -> scalar_t { return min_impl(a, b); },
85:         [=](Vec a, Vec b) -> Vec { return minimum(a, b); });
86:     });
87:   }
88: }
89:
90: void max_all_kernel_impl(Tensor& result, const Tensor& input) {
91:   if (input.scalar_type() == ScalarType::Bool) {
```
- EN: The main symbol in this range is `max_all_kernel_impl`, `min_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `max_all_kernel_impl`, `min_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 92-103
```cpp
 92:     TensorIterator iter = TensorIteratorConfig()
 93:       .add_input(input)
 94:       .build();
 95:     bool result_data  = false;
 96:     cpu_serial_kernel(iter, [&](const bool a) -> void {
 97:       result_data = result_data || a;
 98:     });
 99:     result.fill_(result_data);
100:   } else if (input.scalar_type() == ScalarType::Long) {
101:     // for int64_t, vectorized implementation have performance issue,
102:     // just use scalar path
103:     reduce_all_impl<int64_t>(result, input, lower_bound<int64_t>(),
```
- EN: The main symbol in this range is `cpu_serial_kernel`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `cpu_serial_kernel`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 104-115
```cpp
104:       [=](int64_t a, int64_t b) -> int64_t { return max_impl(a, b); });
105:   } else {
106:     AT_DISPATCH_ALL_TYPES_AND2(kHalf, kBFloat16, input.scalar_type(), "max_all", [&] {
107:       using Vec = Vectorized<opmath_type<scalar_t>>;
108:       reduce_all_impl_vec<scalar_t>(result, input, lower_bound<scalar_t>(),
109:         [=] (scalar_t a , scalar_t b) -> scalar_t { return max_impl(a, b); },
110:         [=](Vec a, Vec b) -> Vec { return maximum(a, b); });
111:     });
112:   }
113: }
114:
115: // For operation not support in avx/avx2
```
- EN: The main symbol in this range is `max_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `max_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 116-127
```cpp
116: template <typename scalar_t, typename func_t1, typename func_t2>
117: inline void reduce_all_impl_two_outputs(
118:     Tensor& output1,
119:     Tensor& output2,
120:     const Tensor& input,
121:     const std::pair<scalar_t, scalar_t>& ident_v,
122:     func_t1 reduce_chunk_func,
123:     func_t2 reduce_acc_func) {
124:   using scalar_t_pair = std::pair<scalar_t, scalar_t>;
125:   const int64_t input_numel = input.numel();
126:   auto input_data = input.const_data_ptr<scalar_t>();
127:   scalar_t_pair result = at::parallel_reduce(0, input_numel, internal::GRAIN_SIZE, ident_v,
```
- EN: The main symbol in this range is `reduce_all_impl_two_outputs`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `reduce_all_impl_two_outputs`，它们直接构成本文件的算子逻辑。

### Lines 128-139
```cpp
128:     [&](int64_t start, int64_t end, const scalar_t_pair& ident) -> scalar_t_pair {
129:       scalar_t_pair partial_out(ident);
130:       for (const auto i : c10::irange(start, end)) {
131:          partial_out = reduce_chunk_func(partial_out, input_data[i]);
132:       }
133:       return partial_out;
134:     },
135:     reduce_acc_func
136:   );
137:   output1.fill_(result.first);
138:   output2.fill_(result.second);
139: }
```
- EN: The main symbol in this range is `partial_out`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `partial_out`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 141-151
```cpp
141: template <typename scalar_t, typename func_t, typename vec_func_t1, typename vec_func_t2>
142: inline void reduce_all_impl_vec_two_outputs(
143:     Tensor& output1,
144:     Tensor& output2,
145:     const Tensor& input,
146:     const std::pair<scalar_t, scalar_t>& ident_v,
147:     func_t reduce_acc_func,
148:     vec_func_t1 reduce_chunk_func1,
149:     vec_func_t2 reduce_chunk_func2) {
150:   using Vec = Vectorized<opmath_type<scalar_t>>;
151:   using scalar_t_pair = std::pair<scalar_t, scalar_t>;
```
- EN: The main symbol in this range is `reduce_all_impl_vec_two_outputs`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `reduce_all_impl_vec_two_outputs`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 152-168
```cpp
152:   const int64_t input_numel = input.numel();
153:   auto input_data = input.const_data_ptr<scalar_t>();
154:   // NOTE: parallel_reduce not support bool type
155:   std::pair<scalar_t, scalar_t> result = at::parallel_reduce(0, input_numel, internal::GRAIN_SIZE, ident_v,
156:     [&](int64_t start, int64_t end, const scalar_t_pair& /* ident */) -> scalar_t_pair {
157:     scalar_t_pair partial_out = vec::reduce2_all<scalar_t>(
158:         [=](Vec x, Vec y) { return reduce_chunk_func1(x, y); },
159:         [=](Vec x, Vec y) { return reduce_chunk_func2(x, y); },
160:         input_data + start,
161:         end - start);
162:       return partial_out;
163:     },
164:     reduce_acc_func
165:   );
166:   output1.fill_(result.first);
167:   output2.fill_(result.second);
168: }
```
- EN: The main symbol in this range is `reduce2_all<scalar_t>`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `reduce2_all<scalar_t>`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 170-180
```cpp
170: void aminmax_allreduce_kernel(
171:     const Tensor& input,
172:     Tensor& min_result,
173:     Tensor& max_result) {
174:   if (input.scalar_type() == ScalarType::Bool) {
175:     TensorIterator iter = TensorIteratorConfig()
176:       .add_input(input)
177:       .build();
178:     bool min_result_data = true;
179:     bool max_result_data = false;
180:     cpu_serial_kernel(iter, [&](const bool a) -> void {
```
- EN: The main symbol in this range is `aminmax_allreduce_kernel`, `cpu_serial_kernel`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `aminmax_allreduce_kernel`, `cpu_serial_kernel`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 181-192
```cpp
181:       min_result_data = min_result_data && a;
182:       max_result_data = max_result_data || a;
183:     });
184:     min_result.fill_(min_result_data);
185:     max_result.fill_(max_result_data);
186:   } else if (input.scalar_type() == ScalarType::Long) {
187:     // for int64_t, vectorized implementation have performance issue,
188:     // just use scalar path
189:     using int64_t_pair = std::pair<int64_t, int64_t>;
190:     reduce_all_impl_two_outputs<int64_t>(min_result, max_result, input,
191:       int64_t_pair(upper_bound<int64_t>(), lower_bound<int64_t>()),
192:       // reduce over chunk
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 193-204
```cpp
193:       [=](int64_t_pair a, int64_t b) -> int64_t_pair {
194:         return int64_t_pair(min_impl(a.first, b), max_impl(a.second, b));
195:       },
196:       // combine two inputs
197:       [=](int64_t_pair a, int64_t_pair b) -> int64_t_pair {
198:         return int64_t_pair(min_impl(a.first, b.first), max_impl(a.second, b.second));
199:       }
200:     );
201:   } else {
202:     AT_DISPATCH_ALL_TYPES_AND2(kBFloat16, kHalf, input.scalar_type(), "aminmax_cpu", [&] {
203:       using Vec = Vectorized<opmath_type<scalar_t>>;
204:       using scalar_t_pair = std::pair<scalar_t, scalar_t>;
```
- EN: The main symbol in this range is `min_impl`, `max_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `min_impl`, `max_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 205-219
```cpp
205:       reduce_all_impl_vec_two_outputs<scalar_t>(
206:         min_result,
207:         max_result,
208:         input,
209:         scalar_t_pair(upper_bound<scalar_t>(), lower_bound<scalar_t>()),
210:         [=] (scalar_t_pair a , scalar_t_pair b) -> scalar_t_pair {
211:           return scalar_t_pair(
212:             min_impl(a.first, b.first), max_impl(a.second, b.second));
213:         },
214:         [=](Vec a, Vec b) -> Vec { return minimum(a, b); },
215:         [=](Vec a, Vec b) -> Vec { return maximum(a, b); }
216:       );
217:     });
218:   }
219: }
```
- EN: The main symbol in this range is `min_impl`, `max_impl`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `min_impl`, `max_impl`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 221-227
```cpp
221: } // namespace
222:
223: REGISTER_DISPATCH(min_all_stub, &min_all_kernel_impl)
224: REGISTER_DISPATCH(max_all_stub, &max_all_kernel_impl)
225: REGISTER_DISPATCH(aminmax_allreduce_stub, &aminmax_allreduce_kernel)
226:
227: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- SIMD vectorization / SIMD 向量化
- Dispatcher registration / 调度器注册
- Low-level memory access / 底层内存访问
- Scalar and dtype abstractions / 标量与数据类型抽象

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/native/ReduceOps.h`, `ATen/native/ReduceAllOps.h`, `ATen/native/ReduceOpsUtils.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`, `ATen/TensorIterator.h`, `ATen/OpMathType.h`, `ATen/native/cpu/Loops.h`, `ATen/native/cpu/zmath.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `Vectorized`, `REGISTER_DISPATCH`, `AT_DISPATCH_ALL_TYPES`, `Scalar`, `ScalarType`
