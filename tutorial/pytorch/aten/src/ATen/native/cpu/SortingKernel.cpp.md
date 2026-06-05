# SortingKernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/SortingKernel.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU sorting, selection, and ordering logic in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了CPU 排序、选择与顺序处理逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
 1: #define TORCH_ASSERT_NO_OPERATORS
 2:
 3: #include <limits>
 4:
 5: #include <ATen/native/Sorting.h>
 6: #include <ATen/core/TensorBase.h>
 7: #include <ATen/Dispatch.h>
 8: #include <ATen/Dispatch_v2.h>
 9: #include <ATen/Parallel.h>
10: #include <ATen/NumericUtils.h>
11: #include <ATen/TensorIterator.h>
12: #include <ATen/cpu/vec/functional.h>
```
- EN: This range pulls in required headers, including `limits`, `ATen/native/Sorting.h`, `ATen/core/TensorBase.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段引入了所需头文件，例如 `limits`, `ATen/native/Sorting.h`, `ATen/core/TensorBase.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 13-25
```cpp
13: #include <ATen/cpu/vec/vec.h>
14: #include <ATen/native/StridedRandomAccessor.h>
15: #include <ATen/native/CompositeRandomAccessor.h>
16: #include <ATen/native/TopKImpl.h>
17: #include <c10/core/WrapDimMinimal.h>
18: #include <c10/util/irange.h>
19: #ifdef USE_FBGEMM
20: #include <fbgemm/Utils.h>
21: #endif
22:
23: namespace at::native {
24:
25: namespace {
```
- EN: This range pulls in required headers, including `ATen/cpu/vec/vec.h`, `ATen/native/StridedRandomAccessor.h`, `ATen/native/CompositeRandomAccessor.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/cpu/vec/vec.h`, `ATen/native/StridedRandomAccessor.h`, `ATen/native/CompositeRandomAccessor.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 27-40
```cpp
27: template <typename func_t>
28: void _dim_apply(
29:     const TensorBase &values,
30:     const TensorBase &indices,
31:     int64_t dim,
32:     const std::string& method_name,
33:     const func_t& f) {
34:   auto iter = TensorIteratorConfig()
35:     .check_all_same_dtype(false)
36:     .resize_outputs(false)
37:     .declare_static_shape(values.sizes(), /*squash_dims=*/dim)
38:     .add_output(values)
39:     .add_output(indices)
40:     .build();
```
- EN: The main symbol in this range is `_dim_apply`, which contributes directly to this file's operator logic. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段的主要符号是 `_dim_apply`，它们直接构成本文件的算子逻辑。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 42-54
```cpp
42:   auto values_dim_stride = values.stride(dim);
43:   auto indices_dim_stride = indices.stride(dim);
44:   auto dim_size = values.size(dim);
45:
46:   AT_DISPATCH_V2(
47:     iter.dtype(), "sorting_kernel_method_name", AT_WRAP([&] {
48:       auto loop = [&](char** data, const int64_t* strides, int64_t n) {
49:         auto* values_data_bytes = data[0];
50:         auto* indices_data_bytes = data[1];
51:
52:         if(values_data_bytes==nullptr || indices_data_bytes==nullptr){
53:           return;
54:         }
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 56-66
```cpp
56:         for ([[maybe_unused]] const auto i : c10::irange(n)) {
57:           f(reinterpret_cast<scalar_t*>(values_data_bytes),
58:             values_dim_stride,
59:             reinterpret_cast<int64_t*>(indices_data_bytes),
60:             indices_dim_stride,
61:             dim_size);
62:
63:           values_data_bytes += strides[0];
64:           indices_data_bytes += strides[1];
65:         }
66:       };
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 68-81
```cpp
68:       int64_t grain_size = internal::GRAIN_SIZE / std::max(int64_t{1}, dim_size);
69:       iter.for_each(loop, /*grain_size=*/grain_size);
70:     }), kBool, kHalf, kBFloat16, AT_EXPAND(AT_ALL_TYPES), AT_EXPAND(AT_BAREBONES_UNSIGNED_TYPES)
71:   );
72: }
73:
74: template <typename scalar_t>
75: struct KeyValueCompAsc {
76:   template <typename LHS, typename RHS>
77:   constexpr bool operator()(LHS lhs, RHS rhs) const {
78:     return (!_isnan<scalar_t>(get<0>(lhs)) && _isnan<scalar_t>(get<0>(rhs)))
79:       || (get<0>(lhs) < get<0>(rhs));
80:   }
81: };
```
- EN: The main symbol in this range is `KeyValueCompAsc`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `KeyValueCompAsc`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 83-93
```cpp
83: template <typename scalar_t>
84: struct KeyValueCompDesc {
85:   template <typename LHS, typename RHS>
86:   constexpr bool operator()(LHS lhs, RHS rhs) const {
87:     return (_isnan<scalar_t>(get<0>(lhs)) && !_isnan<scalar_t>(get<0>(rhs)))
88:       || (get<0>(lhs) > get<0>(rhs));
89:   }
90: };
91:
92: #ifdef USE_FBGEMM
93: bool can_use_radix_sort(const TensorBase& values, const bool descending) {
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `can_use_radix_sort`, `KeyValueCompDesc`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `can_use_radix_sort`, `KeyValueCompDesc`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 94-107
```cpp
 94:   // radix_sort can be used only for 1D data
 95:   if (values.dim() != 1) return false;
 96:   // radix_sort sorts in ascending order
 97:   if (descending) return false;
 98:   // radix_sort works for integer values
 99:   if (!at::isIntegralType(values.scalar_type(), /*includeBool=*/false)) return false;
100:   // performance improvements are visible for bigger tensor sizes, when radix_sort
101:   // is accelerated with OpenMP
102:   if (values.numel() < at::internal::GRAIN_SIZE || !fbgemm::is_radix_sort_accelerated_with_openmp()) return false;
103:   // TODO(DamianSzwichtenberg): radix_sort is a stable sorting algorithm,
104:   // should we check here, whether stable is set to true?
105:
106:   return true;
107: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 109-119
```cpp
109: void parallel_sort1d_kernel(
110:     const TensorBase& values,
111:     const TensorBase& indices) {
112:   AT_DISPATCH_INTEGRAL_TYPES(values.scalar_type(), "parallel_sort1d_kernel", [&] {
113:     const auto elements = values.numel();
114:     auto* const keys = values.data_ptr<scalar_t>();
115:     auto* const vals = indices.data_ptr<int64_t>();
116:     std::vector<scalar_t> tmp_keys(elements);
117:     std::vector<int64_t> tmp_vals(elements);
118:     const scalar_t* sorted_keys = nullptr;
119:     const int64_t* sorted_vals = nullptr;
```
- EN: The main symbol in this range is `parallel_sort1d_kernel`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 这一段的主要符号是 `parallel_sort1d_kernel`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 120-131
```cpp
120:     std::tie(sorted_keys, sorted_vals) = fbgemm::radix_sort_parallel(
121:         keys,
122:         vals,
123:         tmp_keys.data(),
124:         tmp_vals.data(),
125:         elements,
126:         std::numeric_limits<scalar_t>::max(),
127:         values.scalar_type() != ScalarType::Byte);
128:
129:     const bool sorted_in_place = keys == sorted_keys;
130:     if (!sorted_in_place) {
131:       const auto num_threads = at::get_num_threads();
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 132-143
```cpp
132:       at::parallel_for(0, elements, elements / num_threads, [&](int64_t begin, int64_t end) {
133:         const auto job_size = end - begin;
134:         vec::map([](vec::Vectorized<scalar_t> x) -> vec::Vectorized<scalar_t> { return x; }, keys + begin, sorted_keys + begin, job_size);
135:         vec::map([](vec::Vectorized<int64_t> x) -> vec::Vectorized<int64_t> { return x; }, vals + begin, sorted_vals + begin, job_size);
136:       });
137:     }
138:   });
139: }
140: #endif
141:
142: template <typename scalar_t, typename value_accessor_t, typename indices_accessor_t>
143: inline void sort_kernel_impl(const value_accessor_t& value_accessor,
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `sort_kernel_impl`, which contributes directly to this file's operator logic. Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `sort_kernel_impl`，它们直接构成本文件的算子逻辑。 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 144-155
```cpp
144:             const indices_accessor_t& indices_accessor,
145:             int64_t dim_size, bool descending, bool stable) {
146:   auto composite_accessor = CompositeRandomAccessorCPU<
147:     value_accessor_t, indices_accessor_t
148:   >(value_accessor, indices_accessor);
149:   if (descending) {
150:     if (stable) {
151:       std::stable_sort(composite_accessor, composite_accessor + dim_size,
152:         KeyValueCompDesc<scalar_t>());
153:     } else {
154:       std::sort(composite_accessor, composite_accessor + dim_size,
155:         KeyValueCompDesc<scalar_t>());
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 156-166
```cpp
156:     }
157:   } else {
158:     if (stable) {
159:       std::stable_sort(composite_accessor, composite_accessor + dim_size,
160:         KeyValueCompAsc<scalar_t>());
161:     } else {
162:       std::sort(composite_accessor, composite_accessor + dim_size,
163:         KeyValueCompAsc<scalar_t>());
164:     }
165:   }
166: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 168-178
```cpp
168: void sort_kernel(
169:     const TensorBase& self,
170:     const TensorBase& values,
171:     const TensorBase& indices,
172:     int64_t dim,
173:     bool descending,
174:     bool stable) {
175:   dim = maybe_wrap_dim(dim, values.dim());
176:   _fill_indices(indices, dim);
177:   if (self.stride(dim) == 0) {
178:     // check if stride is zero
```
- EN: The main symbol in this range is `sort_kernel`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段的主要符号是 `sort_kernel`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 179-190
```cpp
179:     // https://github.com/pytorch/pytorch/issues/91420
180:     return;
181:   }
182: #ifdef USE_FBGEMM
183:   if (can_use_radix_sort(values, descending)) {
184:     parallel_sort1d_kernel(values, indices);
185:     return;
186:   }
187: #endif
188:   _dim_apply(
189:     values, indices, dim,
190:     "sort_cpu", [&](
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `parallel_sort1d_kernel`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `parallel_sort1d_kernel`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 191-202
```cpp
191:       auto* values, int64_t values_dim_stride,
192:       auto* indices, int64_t indices_dim_stride,
193:       int64_t dim_size
194:     ) {
195:       using scalar_t = std::remove_pointer_t<decltype(values)>;
196:       if (values_dim_stride == 1 && indices_dim_stride == 1) {
197:         sort_kernel_impl<
198:           scalar_t, decltype(values), decltype(indices)
199:         >(values, indices, dim_size, descending, stable);
200:       } else if (values_dim_stride == 1 && indices_dim_stride != 1) {
201:         auto indices_accessor = StridedRandomAccessor<int64_t>(
202:           indices, indices_dim_stride);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 203-214
```cpp
203:         sort_kernel_impl<
204:           scalar_t, decltype(values), decltype(indices_accessor)
205:         >(values, indices_accessor, dim_size, descending, stable);
206:       } else if (values_dim_stride != 1 && indices_dim_stride == 1) {
207:         auto values_accessor = StridedRandomAccessor<scalar_t>(
208:           values, values_dim_stride);
209:         sort_kernel_impl<
210:           scalar_t, decltype(values_accessor), decltype(indices)
211:         >(values_accessor, indices, dim_size, descending, stable);
212:       } else {
213:         auto values_accessor = StridedRandomAccessor<scalar_t>(
214:           values, values_dim_stride);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 215-226
```cpp
215:         auto indices_accessor = StridedRandomAccessor<int64_t>(
216:           indices, indices_dim_stride);
217:         sort_kernel_impl<
218:           scalar_t, decltype(values_accessor), decltype(indices_accessor)
219:         >(values_accessor, indices_accessor, dim_size, descending, stable);
220:       }
221:     }
222:   );
223: }
224:
225: void topk_kernel(
226:     const TensorBase &values,
```
- EN: The main symbol in this range is `topk_kernel`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `topk_kernel`，它们直接构成本文件的算子逻辑。

### Lines 227-241
```cpp
227:     const TensorBase &indices,
228:     const TensorBase &self,
229:     int64_t k,
230:     int64_t dim,
231:     bool largest,
232:     bool sorted) {
233:   auto sizes = self.sizes();
234:   auto iter = TensorIteratorConfig()
235:     .check_all_same_dtype(false)
236:     .resize_outputs(false)
237:     .declare_static_shape(sizes, /*squash_dims=*/dim)
238:     .add_output(values)
239:     .add_output(indices)
240:     .add_const_input(self)
241:     .build();
```
- EN: TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 243-258
```cpp
243:   auto mode_values_stride = values.strides()[dim];
244:   auto mode_indices_stride = indices.strides()[dim];
245:   auto tmp_values_stride = self.strides()[dim];
246:
247:   AT_DISPATCH_ALL_TYPES_AND2(ScalarType::BFloat16, ScalarType::Half, self.scalar_type(), "topk_cpu", [&] {
248:     auto loop = [&](char** data, const int64_t* strides, int64_t n) {
249:       if (self.scalar_type() == ScalarType::BFloat16) {
250:         return topk_impl_loop<scalar_t, float>(
251:             mode_values_stride, mode_indices_stride, tmp_values_stride,
252:             k, sizes[dim], largest, sorted, data, strides, n);
253:       } else {
254:         return topk_impl_loop<scalar_t, scalar_t>(
255:             mode_values_stride, mode_indices_stride, tmp_values_stride,
256:             k, sizes[dim], largest, sorted, data, strides, n);
257:       }
258:     };
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 260-270
```cpp
260:     int64_t grain_size = internal::GRAIN_SIZE / std::max(int64_t{1}, sizes[dim]);
261:     iter.for_each(loop, /*grain_size=*/grain_size);
262:   });
263: }
264:
265: } // anonymous namespace
266:
267: REGISTER_DISPATCH(sort_stub, &sort_kernel)
268: REGISTER_DISPATCH(topk_stub, &topk_kernel)
269:
270: } //at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Dispatcher registration / 调度器注册

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/native/Sorting.h`, `ATen/core/TensorBase.h`, `ATen/Dispatch.h`, `ATen/Dispatch_v2.h`, `ATen/Parallel.h`, `ATen/NumericUtils.h`, `ATen/TensorIterator.h`, `ATen/cpu/vec/functional.h`, `ATen/cpu/vec/vec.h`, `ATen/native/StridedRandomAccessor.h`
- c10 headers / c10 头文件: `c10/core/WrapDimMinimal.h`, `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `limits`, `fbgemm/Utils.h`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `TensorBase`, `Vectorized`, `parallel_for`, `REGISTER_DISPATCH`, `AT_DISPATCH_ALL_TYPES`, `Scalar`, `ScalarType`, `fbgemm`
