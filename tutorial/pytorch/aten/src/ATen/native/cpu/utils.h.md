# utils.h — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/utils.h`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares support for utils in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了 utils 的相关支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
 1: #pragma once
 2:
 3: #include <ATen/Parallel.h>
 4: #include <ATen/core/TensorAccessor.h>
 5: #include <ATen/cpu/vec/vec.h>
 6: #include <c10/util/llvmMathExtras.h>
 7:
 8: #ifdef USE_FBGEMM
 9: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wextra-semi")
10: #include <fbgemm/Fbgemm.h>
11: C10_DIAGNOSTIC_POP()
12: #endif
```
- EN: This range pulls in required headers, including `ATen/Parallel.h`, `ATen/core/TensorAccessor.h`, `ATen/cpu/vec/vec.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `ATen/Parallel.h`, `ATen/core/TensorAccessor.h`, `ATen/cpu/vec/vec.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 14-24
```cpp
14: namespace at::native {
15:
16: template <typename T>
17: inline void _store(T* dst, at::vec::Vectorized<T> src) {
18:   src.store(dst);
19: }
20:
21: inline void _store(at::BFloat16* dst, at::vec::Vectorized<float> src) {
22:   auto res = at::vec::convert_float_bfloat16(src, src);
23:   res.store(dst, at::vec::Vectorized<float>::size());
24: }
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `_store`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `_store`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 26-36
```cpp
26: inline void _store(at::Half* dst, at::vec::Vectorized<float> src) {
27:   auto res = at::vec::convert_float_half(src, src);
28:   res.store(dst, at::vec::Vectorized<float>::size());
29: }
30:
31: inline namespace CPU_CAPABILITY {
32:
33: template <typename T>
34: inline T data_index_init(T offset) {
35:   return offset;
36: }
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `_store`, `data_index_init`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `_store`, `data_index_init`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 38-47
```cpp
38: template <typename T, typename... Args>
39: inline T data_index_init(T offset, T& x, const T& X, Args&&... args) {
40:   offset = data_index_init(offset, std::forward<Args>(args)...);
41:   x = offset % X;
42:   return offset / X;
43: }
44:
45: inline bool data_index_step() {
46:   return true;
47: }
```
- EN: The main symbol in this range is `data_index_init`, `data_index_step`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `data_index_init`, `data_index_step`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 49-59
```cpp
49: template <typename T, typename... Args>
50: inline bool data_index_step(T& x, const T& X, Args&&... args) {
51:   if (data_index_step(std::forward<Args>(args)...)) {
52:     x = ((x + 1) == X) ? 0 : (x + 1);
53:     return x == 0;
54:   }
55:   return false;
56: }
57:
58: // Helper struct for bfloat16/float16 vectorization
59: // Useful when you need float as immediate dtype or accumulate dtype
```
- EN: The main symbol in this range is `data_index_step`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `data_index_step`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 60-71
```cpp
60: using namespace vec;
61: struct Vec2 {
62:   Vectorized<float> val0, val1;
63:   Vec2(Vectorized<float> v0, Vectorized<float> v1) : val0(v0), val1(v1) {}
64:   Vec2(float v) : val0(v), val1(v) {}
65:   static Vec2 loadu(const BFloat16* ptr) {
66:     auto [v0, v1] = convert_bfloat16_float(Vectorized<BFloat16>::loadu(ptr));
67:     return {v0, v1};
68:   }
69:   static Vec2 loadu(const Half* ptr) {
70:     auto [v0, v1] = convert_half_float(Vectorized<Half>::loadu(ptr));
71:     return {v0, v1};
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `Vec2`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `Vec2`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 72-83
```cpp
72:   }
73:   static Vec2 loadu(const float* ptr) {
74:     return {Vectorized<float>::loadu(ptr), Vectorized<float>::loadu(ptr + Vectorized<float>::size())};
75:   }
76:   void store(BFloat16* ptr) const {
77:     Vectorized<BFloat16> val = convert_float_bfloat16(val0, val1);
78:     val.store(ptr);
79:   }
80:   void store(Half* ptr) const {
81:     Vectorized<Half> val = convert_float_half(val0, val1);
82:     val.store(ptr);
83:   }
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code. The math and shape handling relate to convolution-style operators.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。 这里的计算与形状处理与卷积类算子相关。

### Lines 84-99
```cpp
84:   void store(float* ptr) const {
85:     val0.store(ptr);
86:     val1.store(ptr + Vectorized<float>::size());
87:   }
88: };
89: inline Vec2 operator+(const Vec2& a, const Vec2& b) { return {a.val0 + b.val0, a.val1 + b.val1}; }
90: inline Vec2 operator*(const Vec2& a, const Vec2& b) { return {a.val0 * b.val0, a.val1 * b.val1}; }
91: inline Vec2 operator-(const Vec2& a, const Vec2& b) { return {a.val0 - b.val0, a.val1 - b.val1}; }
92: inline Vec2 operator/(const Vec2& a, const Vec2& b) { return {a.val0 / b.val0, a.val1 / b.val1}; }
93: inline Vec2 maximum(const Vec2& a, const Vec2& b) { return {vec::maximum(a.val0, b.val0), vec::maximum(a.val1, b.val1)}; }
94: inline Vec2 minimum(const Vec2& a, const Vec2& b) { return {vec::minimum(a.val0, b.val0), vec::minimum(a.val1, b.val1)}; }
95:
96: template <typename scalar_t> struct VectorizedType { using type = Vectorized<scalar_t>; };
97: template <> struct VectorizedType<BFloat16> { using type = Vec2; };
98: template <> struct VectorizedType<Half> { using type = Vec2; };
99: template <typename scalar_t> using VecType = typename VectorizedType<scalar_t>::type;
```
- EN: The main symbol in this range is `VectorizedType`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `VectorizedType`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 101-113
```cpp
101: // Helper for mixed data type parameter Vec::load
102: inline std::tuple<Vectorized<float>, Vectorized<float>> load2f(const BFloat16* ptr) {
103:   return convert_bfloat16_float(Vectorized<BFloat16>::loadu(ptr));
104: }
105:
106: inline std::tuple<Vectorized<float>, Vectorized<float>> load2f(const Half* ptr) {
107:   return convert_half_float(Vectorized<Half>::loadu(ptr));
108: }
109:
110: inline std::tuple<Vectorized<float>, Vectorized<float>> load2f(const float* ptr) {
111:   using Vec = Vectorized<float>;
112:   return std::make_tuple(Vec::loadu(ptr), Vec::loadu(ptr + Vec::size()));
113: }
```
- EN: The main symbol in this range is `load2f`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `load2f`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 115-130
```cpp
115: inline std::tuple<Vectorized<float>, Vectorized<float>> load2f(const BFloat16* ptr, int64_t count) {
116:   return convert_bfloat16_float(Vectorized<BFloat16>::loadu(ptr, count));
117: }
118:
119: inline std::tuple<Vectorized<float>, Vectorized<float>> load2f(const Half* ptr, int64_t count) {
120:   return convert_half_float(Vectorized<Half>::loadu(ptr, count));
121: }
122:
123: inline std::tuple<Vectorized<float>, Vectorized<float>> load2f(const float* ptr, int64_t count) {
124:   using Vec = Vectorized<float>;
125:   if (count > Vec::size()) {
126:   return std::make_tuple(Vec::loadu(ptr), Vec::loadu(ptr + Vec::size(), count - Vec::size()));
127:   } else {
128:     return std::make_tuple(Vec::loadu(ptr, count), Vec(0));
129:   }
130: }
```
- EN: The main symbol in this range is `load2f`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `load2f`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 132-144
```cpp
132: } // namespace
133:
134: namespace utils {
135:
136: template <typename T>
137: T CeilLog2(const T& x) {
138:   if (x <= 2) {
139:     return 1;
140:   }
141:   // Last set bit is floor(log2(x)), floor + 1 is ceil
142:   // except when x is an exact powers of 2, so subtract 1 first
143:   return static_cast<T>(llvm::findLastSet(static_cast<uint64_t>(x) - 1)) + 1;
144: }
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `CeilLog2`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `CeilLog2`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 146-156
```cpp
146: // matrix transpose:
147: //   src has shape of M by N, with leading dimension of ld_src
148: //   dst has shape of N by M, with leading dimension of ld_dst
149: template <typename T>
150: inline void transpose(int64_t M, int64_t N, const T* src, int64_t ld_src, T* dst, int64_t ld_dst) {
151:   for (int64_t j = 0; j < N; j++) {
152:     for (int64_t i = 0; i < M; i++) {
153:       dst[j * ld_dst + i] = c10::load(&(src[i * ld_src + j]));
154:     }
155:   }
156: }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 158-169
```cpp
158: #ifdef USE_FBGEMM
159: template <>
160: inline void transpose<float>(int64_t M, int64_t N, const float* src, int64_t ld_src, float* dst, int64_t ld_dst) {
161:   TORCH_CHECK(fbgemm::fbgemmSupportedCPU(), "Your CPU does not support FBGEMM.");
162:   fbgemm::transpose_simd<float>(M, N, src, ld_src, dst, ld_dst);
163: }
164:
165: template <>
166: inline void transpose<uint16_t>(int64_t M, int64_t N, const uint16_t* src, int64_t ld_src, uint16_t* dst, int64_t ld_dst) {
167:   TORCH_CHECK(fbgemm::fbgemmSupportedCPU(), "Your CPU does not support FBGEMM.");
168:   fbgemm::transpose_simd<uint16_t>(M, N, src, ld_src, dst, ld_dst);
169: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `transpose<float>`, `transpose<uint16_t>`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `transpose<float>`, `transpose<uint16_t>`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 171-184
```cpp
171: template <>
172: inline void transpose<uint8_t>(int64_t M, int64_t N, const uint8_t* src, int64_t ld_src, uint8_t* dst, int64_t ld_dst) {
173:   TORCH_CHECK(fbgemm::fbgemmSupportedCPU(), "Your CPU does not support FBGEMM.");
174:   fbgemm::transpose_simd<uint8_t>(M, N, src, ld_src, dst, ld_dst);
175: }
176: #endif
177:
178: template <typename index_t, typename F>
179: inline void parallel_sparse_csr(
180:     const TensorAccessor<index_t, 1>& crow_acc,
181:     const int64_t M,
182:     const int64_t nnz,
183:     const F& f) {
184:   TORCH_CHECK(crow_acc.size(0) == M + 1);
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `transpose<uint8_t>`, `parallel_sparse_csr`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `transpose<uint8_t>`, `parallel_sparse_csr`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 186-196
```cpp
186:   // directly parallel on `M` may lead to load imbalance,
187:   // statically determine thread partition here to average payload
188:   // for each thread.
189:   int num_threads = at::get_num_threads();
190:   std::vector<int64_t> thread_splits(num_threads + 1, M);
191:
192:   int64_t thread_averge_payload = std::max((int64_t)1, divup(nnz, num_threads));
193:
194:   thread_splits[0] = 0;
195:   int64_t sum = 0;
196:   int64_t t = 1;
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 197-208
```cpp
197:   for (const auto m : c10::irange(M)) {
198:     int64_t row_start = crow_acc[m];
199:     int64_t row_end = crow_acc[m + 1];
200:     sum += row_end - row_start;
201:     if (sum > t * thread_averge_payload) {
202:       thread_splits[t] = m;
203:       t++;
204:     }
205:   }
206:   // need to restore the last index,
207:   // due to rounding error when calculating `thread_averge_payload`.
208:   thread_splits[num_threads] = M;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 210-220
```cpp
210:   at::parallel_for(0, num_threads, 1, [&](int64_t cbegin, int64_t cend) {
211:     int tid = at::get_thread_num();
212:     int64_t begin = thread_splits[tid];
213:     int64_t end = thread_splits[tid + 1];
214:     f(begin, end);
215:   });
216: }
217:
218: } // namespace utils
219:
220: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

## Key Concepts / 关键概念

- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/Parallel.h`, `ATen/core/TensorAccessor.h`, `ATen/cpu/vec/vec.h`
- c10 headers / c10 头文件: `c10/util/llvmMathExtras.h`
- Standard or third-party headers / 标准库或第三方头文件: `fbgemm/Fbgemm.h`
- Key helper symbols / 关键辅助符号: `Vectorized`, `parallel_for`, `fbgemm`
