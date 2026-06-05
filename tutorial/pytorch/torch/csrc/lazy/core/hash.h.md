# hash.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/core/hash.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
  - CN: 实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-39
```cpp
 1 | /**
 2 |  * Hash utils in this file is adapted from PyTorch/XLA
 3 |  * https://github.com/pytorch/xla/blob/e0e5f937a0ba8d904f9608137dc8c51ba439df2d/third_party/xla_client/util.h
 4 |  */
 5 | #pragma once
 6 | 
 7 | #include <ATen/Tensor.h>
 8 | #include <c10/core/Scalar.h>
 9 | #include <c10/util/int128.h>
10 | #include <torch/csrc/Export.h>
11 | #include <cstring>
12 | #include <set>
13 | #include <string>
14 | #include <string_view>
15 | #include <vector>
16 | 
17 | namespace torch::lazy {
18 | 
19 | using size_t = std::size_t;
20 | 
21 | class TORCH_API hash_t : public c10::uint128 {
22 |  public:
23 |   // Switch from typedef hash_t = uint128 to provide explicit casters
24 |   hash_t(int8_t val) : uint128(static_cast<uint32_t>(val)) {}
25 |   hash_t(int16_t val) : uint128(static_cast<uint32_t>(val)) {}
26 |   hash_t(int32_t val) : uint128(static_cast<uint32_t>(val)) {}
27 |   hash_t(int64_t val) : uint128(static_cast<uint64_t>(val)) {}
28 |   hash_t(uint32_t val) : uint128(val) {}
29 |   hash_t(uint64_t val) : uint128(val) {}
30 |   hash_t(uint128 val) : uint128(val) {}
31 |   hash_t(uint64_t top, uint64_t bottom) : uint128(top, bottom) {}
32 |   hash_t() = default;
33 | };
34 | 
35 | // Std* functions use 64-bit hash
36 | size_t TORCH_API StdDataHash(const void* data, size_t size);
37 | 
38 | size_t TORCH_API StdHashCombine(uintmax_t a, uintmax_t b);
39 | 
```
- EN: Brings in project headers such as `<ATen/Tensor.h>`, `<c10/core/Scalar.h>`, `<c10/util/int128.h>`, `<torch/csrc/Export.h>` and system or third-party headers such as `<cstring>`, `<set>`, `<string>`, `<string_view>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `hash_t` that structure the state handled by this file.
- CN: 这里引入了项目头文件，例如 `<ATen/Tensor.h>`、`<c10/core/Scalar.h>`、`<c10/util/int128.h>`、`<torch/csrc/Export.h>`以及系统或第三方头文件，例如 `<cstring>`、`<set>`、`<string>`、`<string_view>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `hash_t` 等数据抽象，用来组织本文件处理的状态。

### Lines 40-76
```cpp
40 | // Other functions are all 128-bit
41 | hash_t TORCH_API HashBlock(const void* data, size_t n, const hash_t& seed);
42 | 
43 | hash_t TORCH_API DataHash(const void* data, size_t size);
44 | 
45 | hash_t TORCH_API HashCombine(const hash_t& a, const hash_t& b);
46 | 
47 | size_t TORCH_API HashReduce(const hash_t& a);
48 | 
49 | // Returns a string representation of a hash
50 | std::string TORCH_API HashToString(const hash_t& a);
51 | 
52 | struct HashReducer {
53 |   size_t operator()(const hash_t& value) const {
54 |     return HashReduce(value);
55 |   }
56 | };
57 | 
58 | static inline hash_t StringHash(const char* data) {
59 |   return DataHash(data, std::strlen(data));
60 | }
61 | 
62 | // Automatic templated implementation for 'arithmetic' types
63 | template <typename T, std::enable_if_t<std::is_arithmetic_v<T>>* = nullptr>
64 | hash_t Hash(const T& value) {
65 |   return DataHash(&value, sizeof(value));
66 | }
67 | 
68 | // added because on macos builds the vector<bool> specialization
69 | // breaks falling through to the templated arithmetic types above
70 | hash_t TORCH_API Hash(const std::vector<bool>& value);
71 | 
72 | // Specialized implementations for proprietary types
73 | static inline hash_t Hash(const c10::ScalarType& value) {
74 |   return DataHash(&value, sizeof(value));
75 | }
76 | 
```
- EN: Defines or extends data abstractions such as `HashReducer` that structure the state handled by this file. Implements routines such as `HashBlock`, `DataHash`, `HashCombine`, `HashReduce`, `HashToString` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 定义或扩展了 `HashReducer` 等数据抽象，用来组织本文件处理的状态。 实现了 `HashBlock`、`DataHash`、`HashCombine`、`HashReduce`、`HashToString` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 77-107
```cpp
 77 | static inline hash_t Hash(const c10::MemoryFormat& value) {
 78 |   return DataHash(&value, sizeof(value));
 79 | }
 80 | 
 81 | static inline hash_t Hash(const c10::DeviceType& value) {
 82 |   return DataHash(&value, sizeof(value));
 83 | }
 84 | 
 85 | static inline hash_t Hash(const c10::Device& value) {
 86 |   return HashCombine(Hash(value.type()), Hash(value.index()));
 87 | }
 88 | 
 89 | static inline hash_t Hash(const c10::Layout& value) {
 90 |   return DataHash(&value, sizeof(value));
 91 | }
 92 | 
 93 | static inline hash_t Hash(const c10::Scalar& value) {
 94 |   switch (value.type()) {
 95 |     case c10::ScalarType::ComplexDouble:
 96 |       return Hash(value.toComplexDouble());
 97 |     case c10::ScalarType::Double:
 98 |       return Hash(value.toDouble());
 99 |     case c10::ScalarType::Long:
100 |       return Hash(value.toLong());
101 |     case c10::ScalarType::Bool:
102 |       return Hash(value.toBool());
103 |     default:
104 |       TORCH_INTERNAL_ASSERT(false, "Unknown scalar type.", value.type());
105 |   }
106 | }
107 | 
```
- EN: Implements routines such as `Hash`, `DataHash`, `HashCombine`, `TORCH_INTERNAL_ASSERT` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `Hash`、`DataHash`、`HashCombine`、`TORCH_INTERNAL_ASSERT` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 108-147
```cpp
108 | static inline hash_t TensorHash(const at::Tensor& tensor) {
109 |   at::Tensor ctensor = tensor.contiguous();
110 |   int64_t size = ctensor.numel() * ctensor.element_size();
111 |   switch (ctensor.scalar_type()) {
112 |     case at::ScalarType::Bool:
113 |       return DataHash(ctensor.const_data_ptr<bool>(), size);
114 |     case at::ScalarType::Byte:
115 |       return DataHash(ctensor.const_data_ptr<uint8_t>(), size);
116 |     case at::ScalarType::Char:
117 |       return DataHash(ctensor.const_data_ptr<int8_t>(), size);
118 |     case at::ScalarType::Short:
119 |       return DataHash(ctensor.const_data_ptr<int16_t>(), size);
120 |     case at::ScalarType::Int:
121 |       return DataHash(ctensor.const_data_ptr<int32_t>(), size);
122 |     case at::ScalarType::Long:
123 |       return DataHash(ctensor.const_data_ptr<int64_t>(), size);
124 |     case at::ScalarType::Float:
125 |       return DataHash(ctensor.const_data_ptr<float>(), size);
126 |     case at::ScalarType::Double:
127 |       return DataHash(ctensor.const_data_ptr<double>(), size);
128 |     case at::ScalarType::BFloat16:
129 |       return DataHash(ctensor.const_data_ptr<at::BFloat16>(), size);
130 |     case at::ScalarType::Half:
131 |       return DataHash(ctensor.const_data_ptr<at::Half>(), size);
132 |     case at::ScalarType::ComplexFloat:
133 |       return DataHash(ctensor.const_data_ptr<c10::complex<float>>(), size);
134 |     case at::ScalarType::ComplexDouble:
135 |       return DataHash(ctensor.const_data_ptr<c10::complex<double>>(), size);
136 |     case at::ScalarType::UInt16:
137 |       return DataHash(ctensor.const_data_ptr<uint16_t>(), size);
138 |     case at::ScalarType::UInt32:
139 |       return DataHash(ctensor.const_data_ptr<uint32_t>(), size);
140 |     case at::ScalarType::UInt64:
141 |       return DataHash(ctensor.const_data_ptr<uint64_t>(), size);
142 |     default:
143 |       TORCH_INTERNAL_ASSERT(
144 |           false, "Unsupported scalar type:", ctensor.scalar_type());
145 |   }
146 | }
147 | 
```
- EN: Implements routines such as `TensorHash`, `DataHash`, `TORCH_INTERNAL_ASSERT` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `TensorHash`、`DataHash`、`TORCH_INTERNAL_ASSERT` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 148-187
```cpp
148 | static inline hash_t Hash(const std::string& value) {
149 |   return DataHash(value.data(), value.size());
150 | }
151 | 
152 | static inline hash_t Hash(const std::string_view& value) {
153 |   return DataHash(value.data(), value.size());
154 | }
155 | 
156 | static inline hash_t Hash(const at::Generator& value) {
157 |   return TensorHash(value.get_state());
158 | }
159 | 
160 | // Taken from glibc's implementation of hashing optionals,
161 | // we want to include a contribution to the hash to distinguish
162 | // cases where one or another option was null, but we hope it doesn't
163 | // collide with an actually scalar value.
164 | //
165 | // Use an arbitrary randomly-selected 64-bit integer rather than a
166 | // small constant that we then hash at runtime so we don't have to
167 | // repeatedly hash a constant at runtime.
168 | // NOLINTNEXTLINE(*-narrowing-conversions)
169 | static const int64_t kNullOpt = 0x8655d738f3678dda;
170 | 
171 | // Hashing for std::optional types contributes to hash
172 | // for optionals with null value, important to distinguish
173 | // between <nullopt, non-nullopt> and <non-nullopt, nullopt> cases
174 | template <typename T>
175 | hash_t Hash(const std::optional<T>& value) {
176 |   if (value.has_value()) {
177 |     return Hash(value.value());
178 |   } else {
179 |     return kNullOpt;
180 |   }
181 | }
182 | 
183 | // Hashing of containers
184 | // Forward declare to allow hashes of vectors of vectors to work.
185 | template <typename T>
186 | hash_t ContainerHash(const T& values);
187 | 
```
- EN: Implements routines such as `Hash`, `DataHash`, `TensorHash`, `ContainerHash` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `Hash`、`DataHash`、`TensorHash`、`ContainerHash` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 188-221
```cpp
188 | template <typename T>
189 | hash_t Hash(const std::vector<T>& values) {
190 |   return ContainerHash(values);
191 | }
192 | 
193 | // Need a special case for std::optional<container>?
194 | template <typename T>
195 | hash_t Hash(const std::optional<std::vector<T>>& value) {
196 |   if (value.has_value()) {
197 |     return ContainerHash(value.value());
198 |   } else {
199 |     return kNullOpt;
200 |   }
201 | }
202 | 
203 | template <typename T>
204 | hash_t Hash(const std::set<T>& values) {
205 |   return ContainerHash(values);
206 | }
207 | 
208 | template <typename T, typename S>
209 | hash_t Hash(const std::pair<T, S>& values) {
210 |   return HashCombine(Hash(values.first), Hash(values.second));
211 | }
212 | 
213 | static inline hash_t Hash(const hash_t& value) {
214 |   return value;
215 | }
216 | 
217 | template <typename T>
218 | hash_t Hash(c10::ArrayRef<T> values) {
219 |   return ContainerHash(values);
220 | }
221 | 
```
- EN: Implements routines such as `Hash`, `ContainerHash`, `HashCombine` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `Hash`、`ContainerHash`、`HashCombine` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 222-242
```cpp
222 | template <typename T>
223 | hash_t ContainerHash(const T& values) {
224 |   hash_t h(static_cast<uint64_t>(0x85ebca77c2b2ae63));
225 |   for (const auto& value : values) {
226 |     h = HashCombine(h, Hash(value));
227 |   }
228 |   return h;
229 | }
230 | 
231 | // Varargs hashing
232 | template <typename T = void>
233 | hash_t MHash() {
234 |   return hash_t(static_cast<uint64_t>(0x165667b19e3779f9));
235 | }
236 | 
237 | template <typename T, typename... Targs>
238 | hash_t MHash(T value, Targs... Fargs) {
239 |   return HashCombine(Hash(value), MHash(Fargs...));
240 | }
241 | 
242 | } // namespace torch::lazy
```
- EN: Implements routines such as `ContainerHash`, `h`, `MHash`, `hash_t`, `HashCombine` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `ContainerHash`、`h`、`MHash`、`hash_t`、`HashCombine` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `hash_t`, `HashReducer`.
  - CN: `hash_t`、`HashReducer`。
- **Important routines / 重要例程**
  - EN: `StdDataHash`, `StdHashCombine`, `HashBlock`, `DataHash`, `HashCombine`, `HashReduce`, `HashToString`, `operator`.
  - CN: `StdDataHash`、`StdHashCombine`、`HashBlock`、`DataHash`、`HashCombine`、`HashReduce`、`HashToString`、`operator`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<ATen/Tensor.h>`, `<c10/core/Scalar.h>`, `<c10/util/int128.h>`, `<torch/csrc/Export.h>`
- External includes / 外部头文件: `<cstring>`, `<set>`, `<string>`, `<string_view>`, `<vector>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
