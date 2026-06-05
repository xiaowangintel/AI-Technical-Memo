# moments_utils.h — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/moments_utils.h`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares support for moments utils in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了 moments utils 的相关支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
 1: #pragma once
 2:
 3: #include <array>
 4: #include <cstring>
 5: #include <utility>
 6:
 7: #include <ATen/Parallel.h>
 8: #include <ATen/OpMathType.h>
 9: #include <ATen/cpu/vec/vec.h>
10: #include <ATen/native/cpu/utils.h>
11: #include <c10/util/irange.h>
12:
13: namespace at::native {
14: inline namespace CPU_CAPABILITY {
```
- EN: This range pulls in required headers, including `array`, `cstring`, `utility`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `array`, `cstring`, `utility`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 16-26
```cpp
16: template<typename T> using opmath_t = at::opmath_type<T>;
17:
18: constexpr int64_t kChunkSize = 16;
19:
20: template <typename T>
21: void AddMoments(
22:     int64_t m0_add,
23:     const T& m1_add,
24:     const T& m2_add,
25:     int64_t& m0,
26:     T& m1,
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 27-38
```cpp
27:     T& m2) {
28:   const int64_t n = m0 + m0_add;
29:   const T c = n == 0 ? static_cast<T>(0) : static_cast<T>(m0_add) / static_cast<T>(n);
30:   const T delta = m1_add - m1;
31:   m1 += c * delta;
32:   m2 += m2_add + delta * delta * c * static_cast<T>(m0);
33:   m0 = n;
34: }
35:
36: template <typename T>
37: C10_ALWAYS_INLINE void AddMomentsVec(
38:     int64_t m0_add,
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 39-55
```cpp
39:     const vec::Vectorized<T>& m1_add,
40:     const vec::Vectorized<T>& m2_add,
41:     int64_t& m0,
42:     vec::Vectorized<T>& m1,
43:     vec::Vectorized<T>& m2) {
44:   using Vec = vec::Vectorized<T>;
45:   const int64_t n = m0 + m0_add;
46:   const T c = n == 0 ? static_cast<T>(0) : static_cast<T>(m0_add) / static_cast<T>(n);
47:   const Vec c_vec(c);
48:   const Vec delta = m1_add - m1;
49:   const Vec m2_tmp = m2 + m2_add;
50:   const Vec c_vec_delta = c_vec * delta;
51:   const Vec m0_delta = delta * Vec(static_cast<T>(m0));
52:   m1 = m1 + c_vec_delta;
53:   m2 = fmadd(m0_delta, c_vec_delta, m2_tmp);
54:   m0 = n;
55: }
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 57-67
```cpp
57: template <typename T>
58: inline std::enable_if_t<std::is_same_v<T, opmath_t<T>>, void>
59: UpdateMomentsVec(
60:     int64_t m0,
61:     const T* X_ptr,
62:     const std::array<vec::Vectorized<opmath_t<T>>, kChunkSize>& c_vecs,
63:     int64_t& m0_stk0,
64:     vec::Vectorized<opmath_t<T>>& m1_stk0,
65:     vec::Vectorized<opmath_t<T>>& m2_stk0) {
66:   using Vec = vec::Vectorized<opmath_t<T>>;
67:   Vec m1_vec(0);
```
- EN: The main symbol in this range is `UpdateMomentsVec`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这一段的主要符号是 `UpdateMomentsVec`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 68-78
```cpp
68:   Vec m2_vec(0);
69:   for (const auto j : c10::irange(m0)) {
70:     const Vec x_vec = Vec::loadu(X_ptr + j * Vec::size());
71:     const Vec tmpVec = c_vecs[j];
72:     const Vec delta_vec = x_vec - m1_vec;
73:     m1_vec = fmadd(tmpVec, delta_vec, m1_vec);
74:     const Vec tmpVec2 = x_vec - m1_vec;
75:     m2_vec = fmadd(delta_vec, tmpVec2, m2_vec);
76:   }
77:   AddMomentsVec(m0, m1_vec, m2_vec, m0_stk0, m1_stk0, m2_stk0);
78: }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 80-90
```cpp
80: // each bfloat16/half vector will be converted to two float vectors,
81: // and accumulated successively on m1_stk0/m2_stk0.
82: template <typename T>
83: inline std::enable_if_t<!std::is_same_v<T, at::opmath_type<T>>, void>
84: UpdateMomentsVec(
85:     int64_t m0,
86:     const T* X_ptr,
87:     const std::array<vec::Vectorized<at::opmath_type<T>>, kChunkSize>& c_vecs,
88:     int64_t& m0_stk0,
89:     vec::Vectorized<at::opmath_type<T>>& m1_stk0,
90:     vec::Vectorized<at::opmath_type<T>>& m2_stk0) {
```
- EN: The main symbol in this range is `UpdateMomentsVec`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `UpdateMomentsVec`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 这里的计算与形状处理与卷积类算子相关。

### Lines 91-102
```cpp
 91:   using Vec = vec::Vectorized<T>;
 92:   using fVec = vec::Vectorized<at::opmath_type<T>>;
 93:   fVec m1_fvec0(0), m1_fvec1(0);
 94:   fVec m2_fvec0(0), m2_fvec1(0);
 95:   for (const auto j : c10::irange(m0)) {
 96:     const Vec x_bvec = Vec::loadu(X_ptr + j * Vec::size());
 97:     const fVec tmpVec = c_vecs[j];
 98:     auto [x_fvec0, x_fvec1] = convert_to_float<T>(x_bvec);
 99:     const fVec delta_fvec0 = x_fvec0 - m1_fvec0;
100:     const fVec delta_fvec1 = x_fvec1 - m1_fvec1;
101:     m1_fvec0 = fmadd(delta_fvec0, tmpVec, m1_fvec0);
102:     m1_fvec1 = fmadd(delta_fvec1, tmpVec, m1_fvec1);
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The math and shape handling relate to convolution-style operators.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这里的计算与形状处理与卷积类算子相关。

### Lines 103-118
```cpp
103:     const fVec delta_fvec2 = x_fvec0 - m1_fvec0;
104:     const fVec delta_fvec3 = x_fvec1 - m1_fvec1;
105:     m2_fvec0 = fmadd(delta_fvec0, delta_fvec2, m2_fvec0);
106:     m2_fvec1 = fmadd(delta_fvec1, delta_fvec3, m2_fvec1);
107:   }
108:   AddMomentsVec(m0, m1_fvec0, m2_fvec0, m0_stk0, m1_stk0, m2_stk0);
109:   AddMomentsVec(m0, m1_fvec1, m2_fvec1, m0_stk0, m1_stk0, m2_stk0);
110: }
111:
112: // Compute rowwise moments by Welford algorithm and cascade sum to improve
113: // numerical stability.
114: // https://en.wikipedia.org/wiki/Algorithms_for_calculating_variance
115: // https://en.wikipedia.org/wiki/Pairwise_summation
116: template <typename T, int64_t kMaxDepth>
117: std::pair<opmath_t<T>, opmath_t<T>> RowwiseMomentsImpl(const T* X, int64_t N, int64_t ddof = 0) {
118:   using math_t = opmath_t<T>;
```
- EN: The main symbol in this range is `RowwiseMomentsImpl`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `RowwiseMomentsImpl`，它们直接构成本文件的算子逻辑。

### Lines 120-132
```cpp
120:   constexpr int64_t kVecSize = vec::Vectorized<T>::size();
121:   constexpr int64_t kAccVecSize = vec::Vectorized<math_t>::size();
122:   const int64_t n = N / kVecSize;
123:   const int64_t m = divup(n, kChunkSize);
124:   const int64_t depth = utils::CeilLog2(m);
125:
126:   using Vec = vec::Vectorized<math_t>;
127:   const Vec kZeroVec(math_t(0));
128:   std::array<int64_t, kMaxDepth> m0_stk = {{0}};
129:   std::array<Vec, kMaxDepth> m1_stk;
130:   m1_stk.fill(kZeroVec);
131:   std::array<Vec, kMaxDepth> m2_stk;
132:   m2_stk.fill(kZeroVec);
```
- EN: Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 134-144
```cpp
134:   for (const auto i : c10::irange(m)) {
135:     const T* X_ptr = X + i * kChunkSize * kVecSize;
136:     const int64_t m0 = std::min(kChunkSize, n - i * kChunkSize);
137:     static std::array<Vec, kChunkSize> c_vecs = ([]() {
138:       std::array<Vec, kChunkSize> result;
139:       for (const auto i : c10::irange(kChunkSize)) {
140:         result[i] = Vec(math_t(1) / static_cast<math_t>(i + 1));
141:       }
142:       return result;
143:     })();
144:     UpdateMomentsVec(m0, X_ptr, c_vecs, m0_stk[0], m1_stk[0], m2_stk[0]);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 146-156
```cpp
146:     int64_t mask = i + 1;
147:     for (int64_t j = 1; j < depth && (mask & 1) == 0; ++j) {
148:       AddMomentsVec(
149:           m0_stk[j - 1],
150:           m1_stk[j - 1],
151:           m2_stk[j - 1],
152:           m0_stk[j],
153:           m1_stk[j],
154:           m2_stk[j]);
155:       m0_stk[j - 1] = 0;
156:       m1_stk[j - 1] = kZeroVec;
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 157-169
```cpp
157:       m2_stk[j - 1] = kZeroVec;
158:       mask >>= 1;
159:     }
160:   }
161:   for (const auto i : c10::irange(1, depth)) {
162:     AddMomentsVec(
163:         m0_stk[i], m1_stk[i], m2_stk[i], m0_stk[0], m1_stk[0], m2_stk[0]);
164:   }
165:
166:   std::array<math_t, kAccVecSize> m1_arr{};
167:   std::array<math_t, kAccVecSize> m2_arr{};
168:   m1_stk[0].store(m1_arr.data());
169:   m2_stk[0].store(m2_arr.data());
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 171-185
```cpp
171:   int64_t m0 = 0;
172:   math_t m1 = 0;
173:   math_t m2 = 0;
174:   for (int64_t i = n * kVecSize; i < N; ++i) {
175:     math_t x = static_cast<math_t>(X[i]);
176:     const math_t delta = x - m1;
177:     ++m0;
178:     m1 += delta / static_cast<math_t>(m0);
179:     m2 += delta * (x - m1);
180:   }
181:   // for BFloat16, each vector in m1_arr/m2_arr holds 2*n accumulated result
182:   int64_t m0_add = n * kVecSize / kAccVecSize;
183:   for (const auto i : c10::irange(kAccVecSize)) {
184:     AddMoments(m0_add, m1_arr[i], m2_arr[i], m0, m1, m2);
185:   }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 187-197
```cpp
187:   return std::make_pair(m1, m2 / static_cast<math_t>(N - ddof));
188: }
189:
190: template <typename T>
191: std::pair<opmath_t<T>, opmath_t<T>> RowwiseMoments(const T* X, int64_t N, int64_t ddof = 0) {
192:   using Vec = vec::Vectorized<T>;
193:   constexpr int64_t kVecSize = Vec::size();
194:   const int64_t n = N / kVecSize;
195:   const int64_t m = divup(n, kChunkSize);
196:   const int64_t depth = utils::CeilLog2(m);
197:   if (depth <= 4) {
```
- EN: The main symbol in this range is `RowwiseMoments`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `RowwiseMoments`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 198-208
```cpp
198:     return RowwiseMomentsImpl<T, 4>(X, N, ddof);
199:   } else if (depth <= 8) {
200:     return RowwiseMomentsImpl<T, 8>(X, N, ddof);
201:   } else if (depth <= 16) {
202:     return RowwiseMomentsImpl<T, 16>(X, N, ddof);
203:   } else if (depth <= 32) {
204:     return RowwiseMomentsImpl<T, 32>(X, N, ddof);
205:   } else {
206:     return RowwiseMomentsImpl<T, 64>(X, N, ddof);
207:   }
208: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 210-211
```cpp
210: } // namespace CPU_CAPABILITY
211: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- SIMD vectorization / SIMD 向量化
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Shape/container bookkeeping / 形状与容器管理
- Namespace-scoped helper structure / 命名空间内辅助结构

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/Parallel.h`, `ATen/OpMathType.h`, `ATen/cpu/vec/vec.h`, `ATen/native/cpu/utils.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `array`, `cstring`, `utility`
- Key helper symbols / 关键辅助符号: `Vectorized`
